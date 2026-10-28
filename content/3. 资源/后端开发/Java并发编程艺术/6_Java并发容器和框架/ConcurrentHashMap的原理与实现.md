---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
# ConcurrentHashMap

## 为什么会出先ConcurrentHashMap

现有的包存在的问题：

1. 导致程序死循环，原因为HashMap线程不安全。
2. 使用线程安全的HashTable效率非常低下。

因此，提出了HashMap对应的并法包ConcurrentHashMap。

使用ConcurrentHashMap的好处：

1. 使用锁分段技术，提升并发的访问率。
   1. 先将其中数据分成一段一段的存储
   2. 然后给每一段数据配一把锁，当一个线程占用锁访问其中一个端数据的收，其他段的数据也能被其他线程访问。

## ConcurrentHashMap的结构

下面是ConcurrentHashMap的结构：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/1689743913361.jpg)

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成

### Segment数组结构

Segment是可重入锁，在ConcurrentHashMap中扮演可重入锁的角色（ReentrantLock）。

Segment的结构与HashMap可u苏，是一种数组和链表结构。



###  HashEntry结构

键值对结构；

每个HashEntry是一个链表结构的元素。

### 二者之间的关系

一个ConcurrentHashMap里面包含一个Segment数组。

一个Segment里包含一个HashEntry数据。

每个Segment守护者一个HashEntry数组里面的元素。当对HashEntry数组的数据进行修改时，必须首先获得与它对应的Segment锁。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/1689744538623.jpg)

## ConcurrentHashMap的初始化

### 初始化segment数组

选定数组容量为 （数组长度）：为2的N次方。理由：能通过按位与的散列算法定位segments数组的索引。

以下是初始化segment数组的代码：

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

### 初始化segmentShift和segmentMask

这两个变量需要在定位segment是的散列算法里使用，
$$
sshift = ssize 从1向左位移
$$
在默认情况下concurrencyLevel等于16，1需要向左移位移动4次，所以sshift等于4。

segmentShift的作用是定位参与散列运算的位数，默认是28。

segmentMask是散列运算的掩码，等于ssize减1，即15，掩码的二进制各个位的值都是1。

### 初始化每个segment

输入参数initialCapacity是ConcurrentHashMap的初始化容量，loadfactor是每个segment的负载因子，在构造方法里需要通过这两个参数来初始化数组中的每个segment。

```java
if (initialCapacity > MAXIMUM_CAPACITY)
	initialCapacity = MAXIMUM_CAPACITY;

int c = initialCapacity / ssize;
if (c * ssize < initialCapacity)
	++c;
//对应segment里面HashEntry数组的长度，c> 1,取大于等于C的2的N次方值，否则取1。
int cap = 1;
while (cap < c)
	cap <<= 1;
for (int i = 0; i < this.segments.length; ++i)
	this.segments[i] = new Segment<K,V>(cap, loadFactor);	
```

segment的容量threshold＝（int）cap*loadFactor，默认情况下initialCapacity等于16，loadfactor等于0.75，通过运算cap等于1，threshold等于零。

### 定位segment

为了保证数据能够均匀散列分布，减少数据之间的冲突，需要对hashCode进行再一次的散列。

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

## ConcurrentHashMap的操作

### get操作

1. 经过一次再散列；
2. 使用这个散列值通过散列运算定位到Segment
3. 通过散列算法定位到元素。

```java
public V get(Object key) {
	int hash = hash(key.hashCode());
	return segmentFor(hash).get(key, hash);
}
```

get为啥不用加锁？

get方法里将要使用的共享变量都定义为volatile类型，再继续能够使用的时候，保证线程之间的可见性，能够被多线程同时读，保证不会读到过期数据。

保证不会读到过期值得原因：根据Java内存模型的happen before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

关联讲解：[[volatile]]

### put操作

涉及到写，因此，需要加锁。

1. 首先定位到Segment。
2. 然后在Segment里进行插入操作。
   1. 第一步判断是否需要对Segment里面得HashEntry数组进行扩容。
   2. 第二步定位添加元素得位置，然后将其放在HashEntry数组里面。

#### 是否需要扩容

当超过HashEntry数组里面设置的阈值，则对数组进行扩容。

Segment的扩容判断比HashMap更恰当，因为HashMap是在插入元素后判断元素是否已经到达容量的，如果到达了就进行扩容，但是很有可能扩容之后没有新元素插入，这时HashMap就进行了一次无效的扩容。

#### 如何进行扩容

1. 创建一个容量是原来两倍的数组。
2. 将元素组里的元素进行再散列后插入到新的数组里。

为了保证高效，ConcurrentHashMap不会对整个容器进行扩容，只会对某个segment进行扩容。

### size操作

在统计size的时候把所有Segment的put、remove和clean方法全部锁住，但是这种做法显然非常低效。

ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。