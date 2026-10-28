---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## volatile的特性

最好解释：把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作的同步。

```java
class VolatileFeaturesExample {
    volatile long vl = 0L; // 使用volatile声明64位的long型变量
    public void set(long l) {
        vl = l; // 单个volatile变量的写
    }
    public void getAndIncrement () {
        vl++; // 复合（多个）volatile变量的读/写
    }
    public long get() {
        return vl; // 单个volatile变量的读
    }
}
```

假设有多个线程分别调用上面程序的3个方法，这个程序在语义上和下面程序等价。

```java
class VolatileFeaturesExample {
    long vl = 0L; // 64位的long型普通变量
    public synchronized void set(long l) { // 对单个的普通变量的写用同一个锁同步
        vl = l;
    }
    public void getAndIncrement () { // 普通方法调用
        long temp = get(); // 调用已同步的读方法
        temp += 1L; // 普通写操作
        set(temp); // 调用已同步的写方法
    }
    public synchronized long get() { // 对单个的普通变量的读用同一个锁同步
        return vl;
    }
}
```

即，一个volatile变量的单个读/写操作，与一个普通的读/写操作都是使用同一个锁同步，它们之间的执行效果形同。

锁的happens-before规则保证：

1. 释放锁时两个线程的可见性
2. 获取锁时两个线程的可见性

那么，对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。

<font color="red">锁的语义决定了临界区代码的执行具有原子性。</font>即，只要是volatile变量，即使是64位的long和double型变量，也具有原子性，但是如果是符合操作（a++）不具有原子性。

总结**volatile自身**特性

1. **可见性**

   对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入

2. **原子性**

   对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

## volatile写-读建立的happens-before关系

即相当于volatile对线程可见性的影响

从JDK5开始，volatile变量的写-读可以实现线程之间的通信。

从内存语义的角度来说，volatile的写-读与锁的释放-获取有相同的效果：volatile写和锁的释放有相同的内存语义；volatile读与锁的获取有相同的语义。

```java
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;
    public void writer() {
        a = 1;　　　　　 // 1
        flag = true;　　　 // 2
    }
    public void reader() {
        if (flag) {　　　　// 3
        int i = a;　　 // 4
        ……
    }
}
```

假设线程A先执行writer方法，线程B执行reader()方法。A线程写之前的所有共享变量，B线程读取之后可见所有的共享变量。根据happens-before规则，这个建立的happens-before关系可以分为3类：

1. 根据顺序次序规则
2. 根据volatile规则，2 happens before 3
3. 根据happens-before规则

![image-20221209153237453](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209153237453.png)

<center>happens before关系</center>

每一个箭头链接的两个节点，代表了一个happens-before关系。黑色箭头表示程序顺序规则；橙色箭头表示volatile规则；蓝色箭头表示组合这些规则后提供的happens-before保证。

## volatile写-读的内存语义

#### volatile写内存语义

- 当写一个volatile变量，JMM会把该线程对应的本地内存中的共享变量书信到主内存。

```java
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;
    public void writer() {
        a = 1;　　　　　 // 1
        flag = true;　　　 // 2
    }
    public void reader() {
        if (flag) {　　　　// 3
        int i = a;　　 // 4
        ……
    }
}
```

状态图如下：

![image-20221209154059683](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209154059683.png)

<center>共享变量的状态示意图</center>

线程A在写flag变量后，本地内存A中被线程A更新过的两个共享变量的值被刷新到主内存中。此时，本地内存A和主内存中的共享变量的值是一致的。

#### volatile读内存语义

- 当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。接下来从主内存中进行读取。

![image-20221209155208886](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209155208886.png)

<center>共享变量的状态示意图</center>

线程B原本的值认为无效，必须从主内存中进行读取。

将两部综合起来看：<font color="red">在读线程B读一个volatile变量后，写线程A再写这个volatile变量之前所有可见的共享变量值都将立即变得对读线程B可见。</font>

#### 总结

- 线程A写一个volatile变量，实质上就是线程A向接下来要读这个volatile变量的某个线程发出了(其对共享变量所做的修改)消息。
- 线程B读一个volatile变量，实质上就是线程B接受之前某个线程发出（在写这个volatile变量之前对共享变量所做修改的）的消息
- 线程A写一个volatile变量，线程B随后读这个volatile变量，实质上就是线程A向线程B发送消息

## volatile语义实现

为了实现volatile内存语义，JMM会分别限制这两种类型的重排序类型。

<center>volatile重排序规则表</center>

| 是否能重排序 | 第二个操作 | 第二个操作 | 第二个操作 |
| ------------ | ---------- | ---------- | ---------- |
| 第一个操作   | 普通读/写  | volatile读 | volatile写 |
| 普通读/写    |            |            | NO         |
| volatile读   | NO         | NO         | NO         |
| volatile写   |            | NO         | NO         |

举例来说，第三行最后一个单元格的意思是：在程序中，当第一个操作为普通变量的读或写时，如果第二个操作为volatile写，则编译器不能重排序这两个操作。

从表可知：

1. 当第二个是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后
2. 当第一个操作volatile读时，不管第二个是什么，都不能重排序。目的是为了确保volatile读之后的操作不会被编译器重排序到volatile都之前
3. 当一个时volatile写，第二个操作是volatile读时，不能重排序。

怎么进行实现？

1. 编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
2. 编译器去发现最有不住来最小化屏障总数几乎不可能，JMM采用保守策略。
   1. 在每个volatile写操作的前面插入一个StoreStore屏障。
   2. 在每个volatile写操作的后面插入一个StoreLoad屏障。
   3. 在每个volatile读操作的后面插入一个LoadLoad屏障。
   4. 在每个volatile读操作的后面插入一个LoadStore屏障。

下图是volatile写插入内存屏障后生成的指令序列示例图：

![image-20221209161647492](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209161647492.png)

<center>指令序列示意图</center>

volatile写后面的StoreLoad屏障。此屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序。因为编译器常常无法准确判断在一个volatile写的后面是否需要插入一个StoreLoad屏障（比如，一个volatile写之后方法立即return）。为了保证能正确实现volatile的内存语义，JMM在采取了保守策略：在每个volatile写的后面，或者在每个volatile读的前面插入一个StoreLoad屏障。从整体执行效率的角度考虑，JMM最终选择了在每个volatile写的后面插入一个StoreLoad屏障。因为volatile写-读内存语义的常见使用模式是：一个写线程写volatile变量，多个读线程读同一个volatile变量。当读线程的数量大大超过写线程时，选择在volatile写之后插入StoreLoad屏障将带来可观的执行效率的提升。从这里可以看到JMM在实现上的一个特点：首先确保正确性，然后再去追求执行效率。

下图是volatile读插入内存屏障后生成的指令序列

![image-20221209163851656](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209163851656.png)

<center>指令序列示意图</center>

LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。

LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序。

上述volatile写和volatile读的内存屏障插入策略非常保守。在实际执行时，只要不改变volatile写-读的内存语义，编译器可以根据具体情况省略不必要的屏障。下面通过具体的示例代码进行说明。

```java
class VolatileBarrierExample {
    int a;
    volatile int v1 = 1;
    volatile int v2 = 2;
    void readAndWrite() {
        int i = v1;　　 // 第一个volatile读
        int j = v2; 　 // 第二个volatile读
        a = i + j; // 普通写
        v1 = i + 1; 　 // 第一个volatile写
        v2 = j * 2; 　 // 第二个 volatile写
    }
    …　　　　　　 // 其他方法
}
```

针对readAndWrite()方法，编译器在生成字节码时可以做如下的优化。

![image-20221209164230852](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209164230852.png)

<center>指令序列示意图</center>

注意，最后的StoreLoad屏障不能省略。因为第二个volatile写之后，方法立即return。此时编译器可能无法准确断定后面是否会有volatile读或写，为了安全起见，编译器通常会在这里插入一个StoreLoad屏障。

上面的优化针对任意处理器平台，由于不同的处理器有不同“松紧度”的处理器内存模型，内存屏障的插入还可以根据具体的处理器内存模型继续优化。以X86处理器为例，除最后的StoreLoad屏障外，其他的屏障都会被省略。

前面保守策略下的volatile读和写，在X86处理器平台可以优化成下图

![image-20221209164416551](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209164416551.png)

<center>指令序列示意图</center>

前文提到过，X86处理器仅会对写-读操作做重排序。X86不会对读-读、读-写和写-写操作做重排序，因此在X86处理器中会省略掉这3种操作类型对应的内存屏障。在X86中，JMM仅需在volatile写后面插入一个StoreLoad屏障即可正确实现volatile写-读的内存语义。这意味着在X86处理器中，volatile写的开销比volatile读的开销会大很多（因为执行StoreLoad屏障开销会比较大）。

## JSR-133为什么要增强volatile的内存语义

旧的Java内存模型允许volatile变量与普通变量重排序。在旧的内存模型中，VolatileExample示例程序可能被重排序成下列时序来执行

![image-20221209164550684](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209164550684.png)

<center>线程执行时序图</center>

为了提供一种比锁更轻量级的线程之间通信的机制，JSR-133专家组决定增强volatile的内存语义：严格限制编译器和处理器对volatile变量与普通变量的重排序，确保volatile的写-读和锁的释放-获取具有相同的内存语义。

从编译器重排序规则和处理器内存屏障插入策略来看，只要volatile变量与普通变量之间的重排序可能会破坏volatile的内存语义，这种重排序就会被编译器重排序规则和处理器内存屏障插入策略禁止。

由于volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。在功能上，锁比volatile更强大；在可伸缩性和执行性能上，volatile更有优势。