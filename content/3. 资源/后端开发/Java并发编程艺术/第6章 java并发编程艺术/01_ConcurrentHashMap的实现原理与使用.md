---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 产生的原因

### 现有的问题

现有的容器中（如果没有concurrent包的话），只有HashMap与HashTable两个容器。

在并发编程中，HashMap是线程不安全的，多线程环境下，使用HashMap进行put操作可能产生**死循环**，例如下面的代码：

```java
final HashMap<String, String> map = new HashMap<String, String>(2);
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10000; i++) {
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            map.put(UUID.randomUUID().toString(), "");
                        }
                    }, "ftf" + i).start();
                }
            }
        }, "ftf");
        t.start();
        t.join();
```

上述代码中，如果多个线程同时进行put操作，将会导致HashMap的Entry链表形成环形数据结构，一旦形成了环形数据结构，Entry的next节点永远不为空，将会导致死循环。

HashTable虽然是线程安全的，但是底层使用<font color="red">synchronized</font>用来保证，在进行put操作的时候，只能**一个线程**进行操作，当线程量特别大时，将会导致阻塞或者轮询。

### concurrentHashMap介绍

ConcurrentHashMap 是 JDK 1.5 随 java.util.concurrent 引入的线程安全 Map，用来替代 HashTable 的整表锁。

读操作不加锁，写操作只锁当前 Segment（JDK 8 起改成 CAS 加 synchronized 锁单个桶），并发度随 Segment 数量提高。

它不允许 null 键和 null 值，迭代器是弱一致的，并发修改下 size() 只是估计值。

### 锁分段技术

HashTable效率低下的原因是一次只能一个线程访问，意味着另外一个线程的put当前线程不使用的数据也会进行阻塞。

假设我们将每个数据进行“分离”，相当于一个线程进行put操作时仅仅锁住当前的数据，其他线程put其他数据的时候不受影响，这就是锁分段技术，可以提高并发访问率。

为了保证这种结构，ConcurrentHashMap在进行设计时，有了以下结构：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230213190159.png)

<center>ConcurrentHashMap的类图</center>

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230213190254.png)

<center>ConcurrentHashMap的结构图</center>

如图，ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。

Segment是一种**可重入锁（[[ReentrantLock]]）**，在ConcurrentHashMap扮演着锁的角色；Segment的结构和HashMap类似，是一种数组和链表结构.

HashEntry用于存储键值对数据，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得与它对应的Segment锁。

一个ConcurrentHashMap里包含一个Segment数组，一个Segment数组里面包含一个HashEntry数组。

## 使用

### 初始化

ConcurrentHashMap初始化方法是通过initialCapacity、loadFactor和concurrencyLevel等几个参数来初始化segment数组、段偏移量segmentShift、段掩码segmentMask和每个segment里的HashEntry数组来实现的。

#### 初始化segments数组

初始化时源代码如下：

```java
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    segmentShift = 32 - sshift;
    segmentMask = ssize - 1;
    this.segments = Segment.newArray(ssize);
```

segments数组的长度ssize是通过concurrencyLevel计算得出的。

默认情况下concurrencyLevel等于16。

为了保证通过**按位与的散列算法**定位segments的数组索引，必须保证segments数组的长度为$2^N$。

> concurrencyLevel的最大值是65535，这意味着segments数组的长度最大为65536，对应的二进制是16位。

#### 初始化segmentShift与segmentMask

segmentShift与segmentMask为全局变量，在定位segment时的散列算法进行使用。

sshift等于ssize从1向左移位的次数，在默认情况下concurrencyLevel等于16，1需要向左移位移动4次，所以sshift等于4。

segmentShift用于**定位参与散列运算的位数**，默认情况下是28，之所以用32，**是因为ConcurrentHashMap里的hash()方法输出的最大数是32位的。**

segmentMask是散列运算的掩码，等于ssize减1，即默认为15。掩码的二进制各个位的值都是1。

> 因为ssize的最大长度是65536，所以segmentShift最大值是16，segmentMask最大值是65535，对应的二进制是16位，每个位都是1。

#### 初始化每个Segment

输入参数initialCapacity是ConcurrentHashMap的初始化容量，loadfactor是每个segment的负载因子，在构造方法里需要通过这两个参数来初始化数组中的每个segment。

```java
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = 1;
    while (cap < c)
        cap <<= 1;
    for (int i = 0; i < this.segments.length; ++i)
        this.segments[i] = new Segment<K,V>(cap, loadFactor);
```

上面代码中，cap就是segment里HashEntry数组的长度，它等于initialCapacity除以ssize的倍数c，如果c大于1，就会取大于等于c的2的N次方值，所以cap不是1，就是2的N次方。

segment的容量$threshold＝(int)cap \times loadFactor$，默认情况下initialCapacity等于16，loadfactor等于0.75，通过运算cap等于1，threshold等于零。

#### 定位Segment

在进行插入和获取元素的时候，必须通过散列算法定位到Segment。需要进行**两次散列，目的是减少散列冲突，使元素能够均匀地分布在不同的Segment上，从而提高容器的存取效率**，首先采用下面算法进行第一次散列：

```java
private static int hash(int h) {
        h += (h << 15) ^ 0xffffcd7d;
        h ^= (h >>> 10);
        h += (h << 3);
        h ^= (h >>> 6);
        h += (h << 2) + (h << 14);
        return h ^ (h >>> 16);
    }
```

假如散列的质量差到极点，那么所有的元素都在一个Segment中，不仅存取元素缓慢，分段锁也会失去意义。因此，需要进行第二次散列。通过再散列能让数字的每一位都参加到散列运算当中，从而减少散列冲突。ConcurrentHashMap通过以下散列算法定位segment。

```java
final Segment<K,V> segmentFor(int hash) {
    return segments[(hash >>> segmentShift) & segmentMask];
}
```

默认情况下segmentShift为28，segmentMask为15，再散列后的数最大是32位二进制数据，向右无符号移动28位，意思是让**高4位**参与到散列运算中.

## 操作

### get

Segment的get操作实现非常简单和高效。先经过一次再散列，然后使用这个散列值通过散列运算定位到Segment，再通过散列算法定位到元素，代码如下。

```java
public V get(Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).get(key, hash);
}
```

get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空才会加锁重读。get方法使用的共享变量都是**volatile类型**，如用于统计当前Segment大小的count字段和用于存储值的HashEntry的value。

```java
transient volatile int count;
volatile V value;
```

定义成volatile类型，可以保证可见性，能够让多个线程同时进行读，并且不会读到过期值，但是只能由一个线程写（如果可以被多个线程写，那么这个值就是不依赖原值）。

保证不会读到过期值的原因是：java内存模型之中存在[[**happens-before**]]原则，对volatile字段的写入操作先于读操作，即使读写线程同时到达，也是写线程先执行，读线程后执行。

在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。

进行元素定位的代码如下：

```java
(hash >>> segmentShift) & segmentMask            // 定位Segment所使用的hash算法
int index = hash & (tab.length - 1);            // 定位HashEntry所使用的hash算法
```

可以发现，定位HashEntry和定位Segment的散列算法虽然一样，都与数组的长度减去1项进行相“与”，但是相遇内容不一样：

- 定位Segment使用的是元素的hashcode通过再散列后得到的值的高位；
- 定位HashEntry直接使用的是再散列后的值。

目的是：<font color="red">避免两次散列后的值一样。</font>

### put

put方法首先定位到Segment，然后在Segment里进行插入操作。插入操作需要经历两个步骤：

1. 判断是否需要对Segment里的HashEntry数组进行扩容；
2. 定位添加元素的位置，然后将其放在HashEntry数组里。

#### 是否需要扩容

在**插入元素前**会先判断Segment里的HashEntry数组是否超过容量（threshold），如果超过阈值，则对数组进行扩容。

在进行扩容时，比HashMap合理，避免了无效扩容（因为HashMap是在插入**元素之后**，有可能是最后一次，下次不再使用还进行了扩容）。

#### 如何扩容

在扩容的时候，首先会创建一个容量是原来容量两倍的数组，然后将原数组里的元素进行再散列后插入到新的数组里。为了高效，**ConcurrentHashMap不会对整个容器进行扩容，而只对某个segment进行扩容。**

### size

如果要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。

Segment里的全局变量count是一个volatile变量，**多线程**场景下，可能存在变化，因此，为了统计准确，产生了以下几种方法：

1. 在统计size的时候把所有Segment的put、remove和clear方法全部锁住，但是这种做法显然非常低效；
2. 先**尝试2次通过不锁住**Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则**再采用加锁**的方式来统计所有Segment的大小。

如何判断在统计时容器发生了变化呢？

**<font color="red">使用modCount变量</font>**。使用modCount变量，在put、remove和clear方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。