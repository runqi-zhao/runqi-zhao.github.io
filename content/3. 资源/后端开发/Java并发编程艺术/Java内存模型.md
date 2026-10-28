---
id: 3
title: Java内存模型
sidebar_label: Java内存模型
description: 总结《Java并发艺术之美》
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

## Java内存基础

### 并发编程模型的两个关键问题

<font color="red" size="5">线程之间如何通信、线程之间如何同步</font>

通信：线程之间以何种机制交换信息。

同步：程序中用于控制不同线程间操作发生相对顺序的机制。

![并发编程两个关键问题](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B8%A4%E4%B8%AA%E5%85%B3%E9%94%AE%E9%97%AE%E9%A2%98.png)

### Java内存模型的抽象结构

Java中，堆结构如下图

![java堆结构](https://img-1312072469.cos.ap-nanjing.myqcloud.com/java%E5%A0%86%E7%BB%93%E6%9E%84.jpg)

其中，共享指的是线程之间共享。

java线程之家通信是由Java内存模型（JMM）控制。JMM决定一个线程对共享变量的写入何时对另一个线程可见。

JMM定义了线程和主存之间的抽象关系：

- 线程之间的共享变量存储在主内存之中
- 每个线程存在本地内存（抽象概念，并不真实存在）
  - 本地内存包括
    - 缓存
    - 写缓冲区
    - 寄存器
    - 其他硬件
    - 编译器优化
- 本地内存保存共享内存副本

![image-20221204212005018](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221204212005018.png)

<center>JMM的抽象结构示意图</center>

线程之间如何进行通信？

1. 线程A把本地内存中的变量通过共享变量刷新到主内存中去
2. 线程B从主内存中去读取线程A之前更新过的共享变量

![image-20221204212230918](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221204212230918.png) ^0rtm7h

<center>线程之间的通信</center>

从总体来看，JMM通过控制主内存于每个线程之间的交互，来控制内存的可见性。

### 从源码到指令序列的重排序

![从源代码到最终执行的指令序列的示意图](https://img-1312072469.cos.ap-nanjing.myqcloud.com/从源代码到最终执行的指令序列的示意图.jpg)

为了提高性能，编译器和处理器会对指令进行重排序：

1. 编译器优化的重排序

   重新安排语句执行顺序，在不改变语义的前提下

2. 指令级并发的重排序

   将多条语句叠加执行。不存在数据依赖性时，处理器可以改变语句对应机器指令的执行顺序。

3. 内存的重排序

   处理器使用缓存于读/写缓冲区，是的加载存储操作看上去在乱序执行

2 3属于处理器重排序 1 属于编译器重排序

<font color="red" size="5">导致问题：内存可见性问题</font>

解决办法

1. 编译器：JMM编译器重排序规则会**<font color="red">禁止特定类型</font>**的编译器重排序
2. 处理器：JMM的处理器重排序要求规则要求Java编译器在生成对应的指令序列之前，插入特定类型的**<font color="red">内存屏障</font>**指令，通过内存屏障指令**<font color="red">禁止类特定类型</font>**的处理器重排序。

### 并发编程模型的分类

现代处理器使用写缓冲区临时保存向内存写入的数据

好处

1. 写缓冲区可以流水线执行，避免处理器停顿带来的延迟；
2. 批处理方式写入写缓冲区，合并写缓冲区对于廷议内存地址的多次写，减少对总线的占用。

坏处：

1. <font color="red">每个处理器的写缓冲区只对本处理器可见</font>；会产生脏读
2. 可见性如何保证？

<font size="5">问题1详细说明</font>

![image-20221204215118266](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221204215118266-1670840143672-1.png)

<center>处理器和内存的交互</center>

如图，处理器A，B同时将自己的共享变量存储到自己的写缓冲区(A1,B1)，A从内存中读取B（A2），B从内存中读取A（B2），最后，将自己缓存区修改的变量（脏数据）刷新到内存中（A3，B3），程序最后的结果为a=b=0。显然，内存实际的发生数徐变成了A2->A1。

**解决办法：处理器允许对写-读操作重排序**

![image-20221204215828583](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221204215828583.png)

N代表不允许连个操作重排序，Y表示允许重排序。

<font size="5">问题2解决</font>

为了保证内存可见性，Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM把内存屏障指令分为4类，如表所示。

![image-20221204220330124](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221204220330124.png)

Load 用于把内存中的数据装载到寄存器中。

Store用于把寄存器中的数据存入内存。

StoreLoad Barriers用的最多，但处理时会将写缓冲区的全部数据读取到内存之中。

### happens-before简介

目前，java使用JSR-133内存模型，在JSR-133中，使用happens-before的概念来阐述操作之间的内存可见性。即，在JMM中，如果一个操作执行的结果对另一个操作可见，那么这两个操作之间必须存在happens-before关系。其中的两个操作的是否为一个线程任意。

happens-before的规则：

1. 程序顺序规则：一个线程中的每个操作，happens-before与该线程中的任意后续操作。

   两个操作具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！仅仅要求前一个操作（执行结果）对后一个操作可见，且一个操作按顺序排在第二个操作之前。

2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。

3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。

4. 传递性：A happens-before B, B happpens-before C -> A happens before C

![image-20221207100407303](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221207100407303.png) 

<center>happens-before与JMM的关系</center>

- **<font color="red">一个happens-before对应于一个或多个编译器和处理器重排序规则。</font>**
- 对于java程序员来说，happens-before避免了为了理解JMM提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现方法

## 重排序

定义：指编译器为了**优化程序性能**而对**指令序列进行重排序**的一种手段

### 数据依赖性

定义：两个操作访问<font color="red">同一个变量</font>，且<font color="red">两个操作中有一个是写操作</font>，此时这两个就存在数据依赖性。

![image-20221208102612371](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208102612371.png)

上面三种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变。

因此，编译器与处理器在重排序的时候，不会改变数据依赖性的顺序。

<font color="red" size="5">注意，此时说的数据依赖性仅仅是针对单处理器，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。</font>

### as-if-serial语义

- 定义：不管怎么重排序，程序的执行结果不变。
- 做法：当存在数据依赖性时，不能进行重排序；不存在时，可以进行重排序。
- 效果：是的程序员出现一种“幻觉”：单线程往往是按程序的顺序执行的。（详情请看`程序的顺序规则`）
- 问题：单线程时不会存在任何问题，多线程会存在问题。

### 程序的顺序规则

<font color="red" size="5">JMM要求前一个操作对后一个操作可见，且第一个操作按顺序排在第二个操作之前。（不代表在其之间执行）</font>

如：

- 操作A的执行结果不需要对操作B可见
- 重排序操作A和操作B后的执行结果，与操作A和操作B按happens-before顺序执行的结果一致。

### 重排序对多线程的影响

```java
class ReorderExample {
    int a = 0;
    boolean flag = false;
    public void writer() {
        a = 1; // 1
        flag = true; // 2
    }
    Public void reader() {
        if (flag) { // 3
        int i = a * a; // 4
        ……
    }
}
```

提问：假设现在存在两个线程A，B，A首先执行writer()方法，随后B执行reader()方法。线程B在执行操作4时，能否看到线程A在操作1对共享变量a的写入？

<font color="blue" size="5">不一定</font>

原因：操作1与操作2没有数据依赖关系，编译器和处理器可以重排序，操作3与操作4不存在依赖关系，也可以重排序，则，会出现下面效果：

![image-20221208105051298](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208105051298.png)

<center>程序执行时序图</center>

虚线代表错误的读操作，实现代表正确的读操作。

很明显，多线程程序的语义被重排序破坏了。

还有一种情况：

![image-20221208105314286](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208105314286.png)

<center>程序执行时序图</center>

3与4存在控制依赖关系。当代码中存在控制依赖性时，会影响指令序列的并行速度。**编译器和处理器执行来克服控制相关性对并行度的影响。**

上图中，处理器执行线程B的处理器可以提前读取并计算a*a，然后把计算结果临时保存在一个名为重排序缓冲(ROB)的英寸缓存中。当操作3的条件位true，将其写入i。

在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果；但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

## 顺序一致性

定义：是一个理论参考模型，在设计的时候，处理器的内存模型和编程语言的内存模型都会以顺序一致性内存模型作为参照物。

### 数据竞争与顺序一致性

java内存模型规范对数据竞争的定于如下：

<font color="red">当在一个线程中写一个变量，在另一个线程读同一个变量，而且写和读没有通过同步来排序。</font>

当代码中包含数据竞争时，程序的执行往往产生违反直觉的结果。如果一个多线程程序能正确同步，这个程序将是一个没有数据竞争的程序。

JMM对正确同步做了以下操作：

- 正确同步（广义，包括对常用原语的正确使用）的，程序的执行具有顺序一致性--线程的执行结果和该程序在一致性内存模型中结果相同。

### 顺序一致性内存模型

两大特性：

1. 一个线程中所有操作必须按照程序的顺序执行。
1. （不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作必须原子执行且立刻对所有线程可见

![image-20221208140933682](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208140933682.png)

<center>顺序一致性内存模型的视图</center>

在概念上，顺序一致性模型有一个单一的全局内存，这个内存通过左右摇摆的开关连接到任意一个线程，同时每一个线程必须按照程序的顺序来执行内存读/写操作。

因此，在任意时间点，<font color="red">只能有一个线程连接到内存</font>，即在多线程并发执行时，图中的装置把所有线程的所有内存读/写操作串行化。

例如，现在线程A与线程B并发执行。其中A有3个操作：A1 -> A2 -> A3，B1->B2->B3。

那么，此时我们可以有下图这种情况：

![image-20221208141621231](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208141621231.png)

<center>顺序一致性模型的一种执行效果</center>

当然，也会存在下图这种效果：

![image-20221208141714667](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208141714667.png)

<center>顺序一致性模型中的另一种执行效果</center>

以上两种情况，都是合理的。第二个操作之所以能够实现是因为：顺序一致性内存模型中的每个操作必须立即对任意线程可见。

但在JMM中，没有这个保证。<font color="red">未同步程序在JMM中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致</font>。

比如，在当前线程把写过的数据缓存在本地内存中，在没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，会认为这个写操作根本没有被当前线程执行。只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才能对其他线程可见。在这种情况下，当前线程和其他线程看到的操作执行顺序将不一致。

### 同步程序的顺序一致性效果

```java
class SynchronizedExample {
    int a = 0;
    boolean flag = false;
    public synchronized void writer() { // 获取锁
        a = 1;
        flag = true;
    } // 释放锁
    public synchronized void reader() { // 获取锁
        if (flag) {
            int i = a;
            ……
        } // 释放锁
    }
}
```

假设线程A执行writer方法，线程B执行reader方法。根据JMM规范，该程序的执行结果将与程序在顺序一致性模型中的执行结果相同。时序图如下：

![image-20221208143620525](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208143620525.png)

<center>两个内存模型中的执行时序对比图</center>

顺序一致性模型中，所有操作按照程序的顺序串行执行。

JMM中，临界区的代码可以重排序。JMM会在推出临界区和进入临界区这两个关键时间点做一些特别处理，是的线程在两个时间点具有与顺序一致性模型相同的内存模型。

虽然线程A在临界区内做了重排序，但由于监视器互斥执行的特性，这里的线程B根本无法“观察”到线程A在临界区内的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果。

因此，JMM在实现顺序一致性的具体方针为：<font color="red">在不改变（正确同步的）程序执行结果的前提下，尽可能地为编译器和处理器地优化打开方便之门。</font>

### 未同步程序地执行特性

未同步或者未正确同步地线程，JMM只提供最小安全性：

- 线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值
- JMM保证线程读操作读取到的值不会无中生有(Out Thin Air)地冒出来。
- JVM在堆分配对象时，首先会对内存空间进行清零，然后才会在上面分配对象（JVM内部会同步这两个操作）。
- 因此，在已经清零的内存空间(Pre-zeroed Memory)分配对象时，域地默认初始化已经完成了。

JMM不保证未同步程序的执行结果与该程序在顺序一致性模型中地执行结果一致。

未同步程序在JMM中执行时，整体式无序的，起执行结果也无法预知。未同步在两个模型中地执行特性有如下几个差异：

1. 顺序一致性模型保证单线程内地操作会按照程序地顺序执行，而JMM不保证单线程内操作会按程序的顺序执行（比如上面正确的多线程在临界区内重排序）。
2. 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而JMM不保证所有线程能看到一致的操作执行顺序。
3. JMM不保证对64位long型和double型变量的写操作具有原子性，而顺序一致性模型保证对所有内存读/写都具有原子性。

差异3与处理器总线工作机制密切相关，在计算机中，数据通过总线在处理器和内存之间传递。每次处理器和内存之间的数据传递都是通过一系列步骤来完成的，这一系列步骤称之为总线事务（Bus Transaction）。总线事务包括读事务（Read Transaction）和写事务（WriteTransaction）。读事务从内存传送数据到处理器，写事务从处理器传送数据到内存，每个事务会读/写内存中一个或多个物理上连续的字。这里的关键是，总线会同步试图并发使用总线的事务。在一个处理器执行总线事务期间，总线会禁止其他的处理器和I/O设备执行内存的读/写。

![image-20221208153200262](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208153200262.png)

<center>总线的工作机制</center>

总线会对A，B，C发现仲裁，对竞争做出裁决。假设A获胜，此时要等A执行完毕，才能访问其他内容。

总线的这些工作机制可以把所有处理器对内存的访问以串行化的方式来执行。在任意时间点，最多只能有一个处理器可以访问内存。这个特性确保了单个总线事务之中的内存读/写操作具有原子性。

问题：32位机器上64位可能不具有原子性，造成后果

![image-20221208154809733](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221208154809733.png)

<center>总线事务执行的时序图</center>

这时，处理器B只能看到被处理器A“写了一半”的无效值。

注意，从JDK5开始，仅仅只允许把一个64位long/double型变量的写操作拆分为两个32位的写操作来执行，任意的读操作在JSR-133中都必须具有原子性（即任意读操作必须要在单个读事务中执行）。

## volatile内存语义

### volatile的特性

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

### volatile写-读建立的happens-before关系

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

### volatile写-读的内存语义

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

### volatile语义实现

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

### JSR-133为什么要增强volatile的内存语义

旧的Java内存模型允许volatile变量与普通变量重排序。在旧的内存模型中，VolatileExample示例程序可能被重排序成下列时序来执行

![image-20221209164550684](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221209164550684.png)

<center>线程执行时序图</center>

为了提供一种比锁更轻量级的线程之间通信的机制，JSR-133专家组决定增强volatile的内存语义：严格限制编译器和处理器对volatile变量与普通变量的重排序，确保volatile的写-读和锁的释放-获取具有相同的内存语义。

从编译器重排序规则和处理器内存屏障插入策略来看，只要volatile变量与普通变量之间的重排序可能会破坏volatile的内存语义，这种重排序就会被编译器重排序规则和处理器内存屏障插入策略禁止。

由于volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。在功能上，锁比volatile更强大；在可伸缩性和执行性能上，volatile更有优势。

## 锁的内存语义

### 锁的释放-获取建立的happens-before关系

锁是java并发编程中最重要的同步机制。锁除了让临界区互斥执行外，还可以让释放锁的线程向获取同一个锁的线程发送消息。

```java
class MonitorExample {
    int a = 0;
    public synchronized void writer() {　　　　 // 1
        a++;　　　　　　　　　　 // 2
    }　　　　　　　　　　　　 // 3
    public synchronized void reader() {　　　 // 4
        int i = a;　　　　　　　　 // 5
        ……
    }　　　　　　　　　　　　 // 6
}
```

假设线程A执行writer()方法，随后线程B执行reader()方法。根据happens-before规则，这个过程包含的happens-before关系可以分为3类。

1. 根据程序次序规则，1 happens-before 2,2 happens-before 3;4 happens-before 5,5 happens before 6。
2. 根据监视器锁规则，3 happens-before 4。
3. 根据happens-before的传递性，2 happens-before 5。

![image-20221210172844092](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221210172844092.png)

<center>happens-before关系图</center>

每一个箭头链接的两个节点，代表了一个happens-before关系。黑色箭头表示程序顺序规则；橙色箭头表示监视器锁规则；蓝色箭头表示组合这些规则后提供的happens before保证。

在线程A释放了锁之后，随后线程B获取同一个锁。在上图中，2 happens-before 5。因此，线程A在释放锁之前所有可见的共享变量，在线程B获取同一个锁之后，将立刻变得对B线程可见。

### 锁的释放和获取的内存语义

当线程释放锁时，JMM会把该线程该线程对应的本地内存中的共享变量数显到主内存中。

```java
class MonitorExample {
    int a = 0;
    public synchronized void writer() {　　　　 // 1
        a++;　　　　　　　　　　 // 2
    }　　　　　　　　　　　　 // 3
    public synchronized void reader() {　　　 // 4
        int i = a;　　　　　　　　 // 5
        ……
    }　　　　　　　　　　　　 // 6
}
```

还是以这个程序为例，假设线程A执行writer()方法，随后线程B执行reader()方法。A线程释放锁后，共享数据的状态如下：

![image-20221210203244226](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221210203244226.png)

<center>共享数据的状态示意图</center>

当线程获取锁时，JMM会把线程对应的本地内存置为无效。从而使得被监视保护的临界区代码必须从内存中读取共享变量。

![image-20221210203446257](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221210203446257.png)

<center>锁获取的状态示意图</center>

<font color="red">锁释放与volatile写有相同的内存语义；所获取与volatile读有相同的内存语义</font>

总结：

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做的修改的）消息。
- 线程B获取一个锁，实质上是线程B接受了之前按某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
- 线程A释放一个锁，线程B获取一个锁，这个过程实际上是线程通过主内存向线程B发送消息。

### 锁内存语义的实现

借助ReentrantLock源码进行分析：

```java
class ReentrantLockExample {
    int a = 0;
    ReentrantLock lock = new ReentrantLock();
    public void writer() {
        lock.lock();　　　　 // 获取锁
        try {
            a++;
        } finally {
            lock.unlock();　　// 释放锁
        }
    }
    public void reader () {
        lock.lock();　　　　 // 获取锁
        try {
        	int i = a;
       		……
        } finally {
        	lock.unlock();　 // 释放锁
        }
    }
}
```

在ReentrantLock中，调用lock()方法获取锁；调用unlock()方法释放锁。

ReentrantLock的实现依赖于Java同步器框架AbstractQueuedSynchronizer（AQS）。AQS使用一个整型的volatile变量（命名为state）来维护同步状态。

![image-20221210204350675](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221210204350675.png)

<center>ReentrantLock的类图</center>

ReentrantLock分为公平锁与不公平锁。

#### 公平锁

加锁方法lock()调用轨迹：

1. ReentrantLock:lock()。
2. FairSync:lock();
3. AbstractQueuedSynchronizer:acquire(int arg)
4. ReentrantLock:tryAcquire(int acquires)

从第四部开始真正加锁

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();　　　　// 获取锁的开始，首先读volatile变量state
    if (c == 0) {
        if (isFirst(current) &&
        compareAndSetState(0, acquires)) {
        setExclusiveOwnerThread(current);
        return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)　　
        throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

从源代码得，加锁方法首先读volatile变量state

解锁方法unlock()调用轨迹如下：

1. ReentrantLock:unlock()。
2. AbstractQueuedSynchronizer:release(int arg)。
3. Sync:tryRelease(int releases)。

从第3步开始释放锁

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);　　　　　// 释放锁的最后，写volatile变量state
    return free;
}
```

释放锁的最后写volatile变量的state。

公平锁在释放后最后写volatile变量state，在获取锁时首先读这个volatile变量。根据volatile的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后立即变得对获取锁的线程可见。

#### 非公平锁

释放与公平锁完全一致

获取轨迹：

1. ReentrantLock:lock()。
2. NonfairSync:lock()。
3. AbstractQueuedSynchronizer:compareAndSetState(int expect,int update)。

 从第三步真正开始加锁：

```java
protected final boolean compareAndSetState(int expect, int update) {
	return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

该方法以原子操作的方式更新state变量。如果当前状态值等于预期值，则以原子方式将同步状态设置为给定的更新值。此操作具有volatile读和写的内存语义方式。

从编译器和处理器出发，分析CAS如何同时具有volatile读和volatile写的内存语义。

- 编译器不会对volatile读与volatile读后面的任意内存操作重排序；
- 编译器不会对volatile写于volatile写前面的任意内存操作重排序。
- **编译不会对CAS于CAS前于CAS后的任意内存操作重排序。**

举例：在x86处理器中，如何同时具有volatile读和volatile写的内存语义的。

下面是sun.misc.Unsafe类的compareAndSwapInt()方法的源代码。

```java
public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
```

这是一个本地方法，这个本地方法在openjdk中依次调用的c++代码为：unsafe.cpp，atomic.cpp和atomic_windows_x86.inline.hpp。这个本地方法的最终实现在openjdk的如下位置：openjdk-7-fcs-src-b147-27_jun_2011\openjdk\hotspot\src\os_cpu\windows_x86\vm\atomic_windows_x86.inline.hpp（对应于Windows操作系统，X86处理器）。下面是对应于intel X86处理器的源代码的片段。

```assembly
inline jint Atomic::cmpxchg (jint exchange_value, volatile jint* dest,
jint compare_value) {
    // alternative for InterlockedCompareExchange
    int mp = os::is_MP();
    __asm {
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    LOCK_IF_MP(mp)
    cmpxchg dword ptr [edx], ecx
    }
}
```

程序会根据当前处理器类型来决定是否为cmpxchg指令加上lock前缀（Lock Cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。intel手册对lock前缀的说明如下：

1. 确保对内存的读-改-写操作原子执行。在Pentium及Pentium之前的处理器中，带有lock前
   缀的指令在执行期间会锁住总线，使得其他处理器暂时无法通过总线访问内存。很显然，这会
   带来昂贵的开销。从Pentium 4、Intel Xeon及P6处理器开始，Intel使用缓存锁定（Cache Locking）来保证指令执行的原子性。缓存锁定将大大降低lock前缀指令的执行开销。
2. 禁止该指令，与之前和之后的读和写指令重排序。
3. 把写缓冲区中的所有数据刷新到内存中。

上面的第2点和第3点具有的内存屏障下效果，足以同时实现volatile读和volatile写的内存语义。

总结：

- 公平锁与非公平锁释放时，最后都要写一个volatile变量state。
- 公平锁获取时，首先会去读volatile变量
- 非公平锁获取时，首先会用CAS更新volatile变量，这个操作同时具有volatile读和volatile写的内存语义。

从本文对ReentrantLock的分析可以看出，锁释放-获取的内存语义的实现至少有下面两种
方式。

1. 利用volatile变量的写-读所具有的内存语义。
2. 利用CAS所附带的volatile读和volatile写的内存语义。

### concurrent包的实现

java线程之间的通信现在有了下面4种方式。

1. A线程写volatile变量，随后B线程读这个volatile变量。
2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。

Java的CAS会使用现代处理器上提供的高效机器级别的原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式。

1. 声明共享变量为volatile。
2. 使用CAS的原子条件更新来实现线程之间的同步。
3. 配合volatile读/写和CAS具有的volatile读和写的内存语义来实现线程之间的通信。

AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。从整体来看，concurrent包的实现示意图如下：

![image-20221210223108372](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221210223108372.png)

<center>concurrent包的实现示意图</center>

## final域的内存语义

### final域的重排序规则

编译器域处理器会遵循连个重排序规则

1. 在构造函数内对一个final域写入，与随后把这个构造对象的引用复制给一个引用变量，这两个操作之间不能重排序。
2. 初次读一个包含final域的对象引用，与随后初次读这个final域，这两个操作之间不能重排序。

```java
public class FinalExample {
    int i;//普通变量
    final int j;//final变量
    static FinalExample obj;
    public FinalExample() {//构造函数
        i = 1;//写普通域
        j = 2;//写final域
    }

    public static void writer() {//写线程A执行
        obj = new FinalExample();//1
    }

    public static void reader() {//读线程B执行
        FinalExample object = obj;//都对象引用
        int a = object.i;//读普通域
        int b = object.j;//读final域
    }
}
```

假设一个线程A执行writer()方法，线程B执行reader()方法。下面两节将会用这个例子说明。

### 写final域的重排序规则

写final域的重排序规则禁止把final域的写重排序到构造函数之外。具体是连个方面：

1. JMM禁止编译器把final域的写重排序到构造函数之外。
2. 编译器会在final域写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。

例如上面的writer()方法。运行1时有连个步骤，如下：

1. 构造一个FinalExample类型的对象。
2. 把这个对象的引用赋值给引用变量obj。

如果线程B读对象引用与读对象成员域之间没有重排序，下图可能是一种执行时序。

![image-20221211105355001](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211105355001.png)

<center>线程执行时序图</center>

写普通域的操作被编译器排到了构造函数之外，读线程B错误的读取了普通变量的值。而写final域的操作，被写final域的重排序规则“限定”在了构造函数之内，读线程B正确地读取了final变量初始化之后的值。

因此，写final域的重排序规则可以确保：<font color="red" size="5px">在对象引用为任意线程可见之前，对象的final域已经初始化完成。而普通域不具有这个保障。</font>

### 读final域的重排序规则

1. 在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止<font color="blue">处理器</font>重排序这连个操作。
2. 编译器会在读final域操作前面插入一个LoadLoad屏障。

初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。大多数处理器也会遵守间接依赖，也不会重排序这两个操作。但有少数处理器允许对存在间接依赖关系的操作做重排序（比如alpha处理器），这个规则就是专门用来针对这种处理器的。
reader()方法包含3个操作。

- 初次读引用变量obj。
- 初次读引用变量obj指向对象的普通域j。
- 初次读引用变量obj指向对象的final域i。

写线程A没有发生任何重排序，同时程序在不遵守间接依赖的处理器上执行，则会有下面这种情况：

![image-20221211111006408](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211111006408.png)

<center>线程执行时序图</center>

读对象的普通域操作被处理器重排序到读对象引用之前。读普通域之前，该域还没有被写线程A写入，会得到一个错误的结果。读final域的重排序规则会把这个读对象final域的操作“限定”在读对象引用之后，此时该final域已经被A线程初始化过了，这是一个正确的读取操作。

因此，读final域的重排序规则可以确保：

<font color="red">在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。</font>

### final域为引用类型

```java
public class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;
    public  FinalReferenceEscapeExample() {
        i = 1;      //写final域
        obj =this; //this引用在此逸出
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) {
            int temp = obj.i;
        }
    }
}
```

对于引用类型，写final域的重排序规则对编译器和处理器增加了如下约束：

<font color="red">在构造函数内对一个final引用的对象的成员域写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。</font>

假设首先线程A执行writerOne()方法，执行完后线程B执行writerTwo()方法，执行完后线程C执行reader()方法。

![image-20221211113156852](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221211113156852.png)

<center>引用型final的执行时序图</center>

1是对final的写入，2是对这个final域引用的对象的成员域的写入，3是把被构造的对象的引用赋值给某个引用变量。1、3；2、3都不能重排序。

JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写入。即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数据竞争，此时的执行结果不可预知。

如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使用同步原语（lock或volatile）来确保内存可见性。

### 为什么final引用不能从构造函数内“溢出”

final域的重排序规则可以确保：在引用变量为任意线程可见之前，该引用变量指向的对象的final域已经在构造函数中被正确初始化了。为了实现这个效果，还需要保证：在构造函数内部，不能让这个被构造对象的引用为其他线程所见，也就是对象引用不能在构造函数中“逸出”。

```java
public class FinalReferenceEscapeExample {
    final int i;
    static FinalReferenceEscapeExample obj;
    public  FinalReferenceEscapeExample() {
        i = 1;      //1
        obj =this; //2
    }

    public static void writer() {
        new FinalReferenceEscapeExample();
    }

    public static void reader() {
        if (obj != null) { //3
            int temp = obj.i; //4
        }
    }
}
```

假设线程A执行writer()方法，线程B执行reader()方法。这里的操作2使得对象还未完成构造前就为线程可见。即使这里的操作2时构造函数的最后一步，且在程序中操作2排在操作1后面，执行reader()方法的线程仍然可能无法看到final域被初始化后的值，因为这里的操作1和操作2之间可能重排序。如下图所示：

![image-20221212085641928](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212085641928.png)

<center>多线程执行时序图</center>

在构造函数之返回前，被构造对象的引用不能为其他线程所见，因为此时的final域可能还没有被初始化。在构造函数返回后，任意线程都将保证能看到final域正确初始化后的值。

### final语义在处理器中的实现

以x86为例，说明final语义在处理器中的具体实现。

之前提过，写final域的重排序规则会要求编译器在final域的写之后，构造含税return之前插入一个StoreStore屏障。读final域的重排序规则要求编译器在读final域的操作前面插入一个LoadLoad屏障。

由于x86处理器不存在写-写操作做重排序，也不会存在读-读操作重排序，因此，在x86处理器中，final域的读/写不会插入任何内存屏障。

### JSR-133为什么要增强final语义

原本的final域缺陷：县城可能看到final域的改变。

增强：通过final域增加写和读重排序规则，可以为java程序员提供初始化安全保证：只要对象是正确构造的（被构造对象的引用在构造函数中没有“逸出”），那么不需要使用同步（指lock和volatile的使用）就能保证任意线程都能看到这个final域在构造函数中被初始化之后的值。

## happens-before

### JMM的设计

设计JMM时，考虑两个关键因素：

1. 程序员堆内存模型的使用。程序员希望内存模型易于理解、易于编程。程序员希望基于一个强内存模型来编写代码。

2. 编译器和处理器对内存模型的实现。编译器和处理器希望内存模型对它们的束缚越少越好，这样它们可以做尽可能多的优化来提高性能。编译器和处理器希望实现一个弱内存模型。

两点之间存在矛盾，在设计JMM时的核心目标就是找到一个好的平衡点：

- 一方面，要为程序员提供足够强的可见性保证
- 另一方面，对编译器和处理器的限制要尽可能地放松。

```java
double pi = 3.14;　　 // A
double r = 1.0;　　　　 // B
double area = pi * r * r;　 // C
```

上面是一个圆面积的实例代码存在happens-before关系：

1. A happens before B
2. B happens before C
3. A happens before C

上面三个happens before关系中，2和3是必需的，但1是不必要的。因此，JMM把happens-before要求禁止重排序分为以下两类：

1. 会改变程序执行结果的重排序。
2. 不会改变程序执行结果的重排序。

JMM对这两种不同性质的重排序，采用不同的策略，如下：

1. 对于会改变程序执行结果的重排序，处理器和编译器必须禁止这种重排序。
2. 对于不会改变程序之字形结果的重排序，JMM对编译器和处理器不做要求。

![image-20221212092306827](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212092306827.png)

<center>JMM的设计示意图</center>

从图中，可看出：

1. JMM像程序员提供的happens-before规则能满足程序员的需求。
2. JMM对对编译器和处理器的束缚已经尽可能地少。只要不改变程序结果，想怎么优化就怎么优化。

### happens-before的定义

JSR-133使用happens-before的概念来指定两个操作之间的执行顺序。由于这连个操作可能在一个线程之内，也可能在不同线程之间。因此，JMM可以通过happens-before关系想程序员提供跨线程的内存可见性保证。（如果A线程的写操作a与B线程的读操作b之间存在happens before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。

因此，定义如下：

1. 如果一个操作happens before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
2. 两个操作之间存在happens-before关系，并不意味着要按照关系指定顺序来执行。如果重排序后的执行结果与按happens-before关系来执行的结果一致，那么这种操作不非法。

<font color="red">1是JMM对程序员的承诺</font>。从程序员的角度来说，可以这样理解happens-before关系：如果A happens-before B，那么Java内存模型将向程序员保证——A操作的结果将对B可见，且A的执行顺序排在B之前。注意，这只是Java内存模型向程序员做出的保证！

<font color="red">2是JMM对编译器和处理器重排序的约束原则。</font>即，happens-before关系本质上和as-if-serial语义是一回事。

- as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。
- as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。

二者目的都是为了不改变程序执行结果的前提下，尽可能地提高程序执行的并行速度。

### happens-before规则

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
6. join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

下图是是volatile写-读建立的happens-before关系图。

![image-20221212094140462](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212094140462.png)

<center>happens-before关系的示意图</center>

分析如下：

1.  1 happens-before 2和3 happens-before 4由程序顺序规则产生。因此，可以把程序顺序规则看成是对as-if-serial语义的“封装”。
2. 2 happens-before 3是由volatile规则产生。对一个volatile变量的读，总是能看到(任意线程)之前这个volatile变量最后的写入。因此，volatile的这个特性可以保证实现volatile规则。
3. 1 happens-before 4是由传递性规则产生的。这里的传递性是由volatile的内存屏障插入策略和volatile的编译器重排序规则共同来保证的。

下面我们来看start()规则。线程A执行时会调用ThreadB.start()来启动线程B；假设线程A在执行Thread.start()之前修改了一些共享变量，线程B在开始执行之后会读这些共享变量。下图时对应的happens-before关系图。

![image-20221212100542849](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212100542849.png)

<center>happens-before关系的示意图</center>

上图中，1 happens-before由顺序规则产生。2 happens-before 4由start()规则产生。根据传递性，将有1 happens-before 4。这实意味着，线程A在执行ThreadB.start()之前对共享变量所做的修改，接下来在线程B开始执行后都将确保对线程B可见。

下面我们来看join()规则。假设线程A在执行的过程中，通过执行ThreadB.join()来等待线程B终止；同时，假设线程B在终止之前修改了一些共享变量，线程A从ThreadB.join()返回后会读这些共享变量。

![image-20221212100843382](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212100843382.png)

<center>happens-before关系的示意图</center>

上图中，2 happens-before 4由join()规则产生；4 happens-before 5由程序顺序规则产生。根据传递性规则，将有2 happens-before 5。这意味着，线程A执行操作ThreadB.join()并成功返回后，线程B中的任意操作都将对线程A可见。

## 双重检查锁定与延迟初始化

在java多线程程序中，有时候需要采用延迟初始化来降低初始化类和创建对象的开销。双重检查锁定是常见的延迟初始化技术，但它是一个错误的用法。本文将分析双重检查锁定的错误根源，以及两种线程安全的延迟初始化方案。

### 双重检查锁定的由来

为了实现懒加载。以单例模式举例：

```java
public class UnsafeLazyInitialization {
    private static Instance instance;
    public static Instance getInstance() {
        if (instance == null) // 1：A线程执行
            instance = new Instance(); // 2：B线程执行
        return instance;
    }
}
```

假设A线程执行代码1的同时，B线程执行代码2。此时，线程A可能会看到instance引用的对象还没有完成初始化。

对于UnsafeLazyInitialization类，我们可以对getInstance()方法做同步处理来实现线程安全的延迟初始化。

```java
public class SafeLazyInitialization {
    private static Instance instance;
    public synchronized static Instance getInstance() {
        if (instance == null)
        	instance = new Instance();
        return instance;
    }
}
```

由于对getInstance()方法做了同步处理，synchronized将导致性能开销。如果getInstance()方法被多个线程频繁的调用，将会导致程序执行性能的下降。反之，如果getInstance()方法不会被多个线程频繁的调用，那么这个延迟初始化方案将能提供令人满意的性能。

在早期的JVM中，synchronized（甚至是无竞争的synchronized）存在巨大的性能开销。因此，人们想出了一个“聪明”的技巧：双重检查锁定（Double-Checked Locking）。人们想通过双重检查锁定来降低同步的开销。

```java
public class DoubleCheckedLocking { // 1
    private static Instance instance; // 2
    public static Instance getInstance() { // 3
        if (instance == null) { // 4:第一次检查
            synchronized (DoubleCheckedLocking.class) { // 5:加锁
        		if (instance == null) // 6:第二次检查
        			instance = new Instance(); // 7:问题的根源出在这里
        		} // 8
        } // 9
        return instance; // 10
    } // 11
}
```

如上面代码所示，多个线程试图在同一时间创建对象时，会通过加锁来保证只有一个线程能创建对象。在对象创建好之后，执行getInstance()方法将不需要获取锁，直接返回已创建好的对象。

<font color="red" size="6">这是错误的！！</font>

在线程执行到第4行，代码读取到instance不为null时，instance引用的对象有可能还没有完成初始化。

### 问题根源

前面的双重检查锁定示例代码的第7行（instance=new Singleton();)创建了一个对象。这一行代码可以分解为如下的3行伪代码。

```java
memory = allocate();　　// 1：分配对象的内存空间
ctorInstance(memory);　 // 2：初始化对象
instance = memory;　　 // 3：设置instance指向刚分配的内存地址
```

上面3行伪代码中的2和3之间，可能会被重排序。2和3之间重排序之后的执行时序如下。

```java
memory = allocate();　　// 1：分配对象的内存空间
instance = memory;　　 // 3：设置instance指向刚分配的内存地址
// 注意，此时对象还没有被初始化！
ctorInstance(memory);　 // 2：初始化对象
```

根据java语言规范，所有线程在执行java程序时必须要遵守intra-thread semantics。intra-thread semantics保证重排序不会改变单线程内的程序执行结果。换句话说，intra-thread semantics允许那些在单线程内，不会改变单线程程序执行结果的重排序。上面3行伪代码的2和3之间虽然被重排序了，但这个重排序并不会违反intra-thread semantics。

![image-20221212102443342](image-20221212102443342.png)

<center>线程执行时时序图</center>

只要保证2排在4的前面，即使2和3之间重排序了，也不会违反intra-thread semantics。

对于多线程并发，则由下图所示的情况：

![image-20221212102556379](image-20221212102556379.png)

<center>多线程执行时序图</center>

当线程A和B按图中的时序执行时，B线程将看到一个还没有被初始化的对象。

DoubleCheckedLocking示例代码的第7行(instance=new Singleton();)如果发生重排序，另一个并发执行的线程B就有可能在第4行判断instance不为null。线程B接下来将访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化！

| 时间 | 线程A                        | 线程B                                                   |
| ---- | ---------------------------- | ------------------------------------------------------- |
| t1   | A1：分配对象的内存空间       |                                                         |
| t2   | A3：设置instance指向内存空间 |                                                         |
| t3   |                              | B1：判断instance是否为空                                |
| t4   |                              | B2：由于instance不为null，线程B将访问instance引用的对象 |
| t5   | A2：初始化对象               |                                                         |
| t6   | A4：反复问instance引用的对象 |                                                         |

这里A2和A3虽然重排序了，但Java内存模型的intra-thread semantics将确保A2一定会排在A4前面执行。因此，线程A的intra-thread semantics没有改变，但A2和A3的重排序，将导致线程B在B1处判断出instance不为空，线程B接下来将访问instance引用的对象。此时，线程B将会访问到一个还未初始化的对象。

因此，提出了连个办法来实现线程安全的延迟初始化。

1. 不允许2和3重排序。
2. 允许2和3重排序，但不允许其他线程“看到”这个排序。

### 基于volatile的解决方案

```java
public class SafeDoubleCheckedLocking {
    private volatile static Instance instance;
    public static Instance getInstance() {
        if (instance == null) {
            synchronized (SafeDoubleCheckedLocking.class) {
            if (instance == null)
                instance = new Instance(); // instance为volatile，现在没问题了
            }
        }
        return instance;
    }
}
```

注意：这个解决方案需要JDK 5或更高版本。

当声明对象的引用为volatile后，3.8.2节中的3行伪代码中的2和3之间的重排序，在多线程环境中将会被禁止。上面示例代码将按如下的时序执行，如下图：

![image-20221212103353314](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212103353314.png)

<center>多线程执行时时序图</center>

这个方案本质上是通过禁止图3-39中的2和3之间的重排序，来保证线程安全的延迟初始化。

### 基于类初始化的解决方案

<font color="red">JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。</font>

```java
public class InstanceFactory {
    private static class InstanceHolder {
        public static Instance instance = new Instance();
    }
    public static Instance getInstance() {
    	return InstanceHolder.instance ;　　// 这里将导致InstanceHolder类被初始化
    }
}
```

假设两个线程并发执行getInstance()方法，下面是执行的示意图，如下：

![image-20221212103548813](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212103548813.png)

<center>两个线程并发执行的示意图</center>

这个方案的实质是：允许3.8.2节中的3行伪代码中的2和3重排序，但不允许非构造线程（这里指线程B）“看到”这个重排序。

初始化一个类，包括执行这个类的静态初始化和初始化在这个类中声明的静态字段。根据Java语言规范，在首次发生下列任意一种情况时，一个类或接口类型T将被立即初始化。

1. T是一个类，而且一个T类型的实例被创建。
2. T是一个类，且T中声明的一个静态方法被调用。
3. T中生命的一个静态字段被赋值。
4. T中声明的一个静态字段被使用，而且这个字段不是一个常量字段。
5. T是一个顶级类，而且一个断言语句嵌套在T内部被执行。

在InstanceFactory示例代码中，首次执行getInstance()方法的线程将导致InstanceHolder类被初始化。

由于Java语言是多线程的，多个线程可能在同一时间尝试去初始化同一个类或接口（比如这里多个线程可能在同一时刻调用getInstance()方法来初始化InstanceHolder类）。因此，在Java中初始化一个类或者接口时，需要做细致的同步处理。

Java语言规范规定，对于每一个类或接口C，都有一个唯一的初始化锁LC与之对应。从C到LC的映射，由JVM的具体实现去自由实现。JVM在类初始化期间会获取这个初始化锁，并且每个线程至少获取一次锁来确保这个类已经被初始化过了（事实上，Java语言规范允许JVM的具体实现在这里做一些优化，见后文的说明）。

对于类或接口的初始化，Java语言规范制定了精巧而复杂的类初始化处理过程。Java初始化一个类或接口的处理过程如下（此处仅仅只说5个步骤，更详细的要看JVM）：

1. 通过在Class对象上同步（即获取Class对象的初始化锁），来控制类或接口的初始化。这个获取锁的线程会一直等待，直到当前线程能够获取到这个初始化锁。

   假设Class对象当前还没有被初始化（初始化状态state，此时被标记为state=noInitialization），且有两个线程A和B试图同时初始化这个Class对象。

   ![image-20221212104206212](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212104206212.png)

   <center>类初始化-第1阶段</center>

   下表是示意图的说明。

   <center>类初始化——第1阶段的执行时序表</center>

   | 时间 | 线程A                                                        | 线程B                                                        |
   | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | t1   | A1：尝试获取Class对象的初始化锁。这里假设线程A获取到了初始化锁。 | B1：尝试获取Class对象的初始化锁，由于线程A获取到了锁，线程B将一直等待获取锁的初始化 |
   | t2   | A2：线程A看到线程还未初始化，线程设置state=initializing      |                                                              |
   | t3   | A3：线程A释放初始化锁                                        |                                                              |

2. 线程A执行类的初始化，同时线程B在初始化锁对应的condition上等待。

   | 时间 | 线程A                                            | 线程B                           |
   | ---- | ------------------------------------------------ | ------------------------------- |
   | t1   | A1：执行类的静态初始化和初始化类中声明的静态字段 | B1：获取到初始化锁              |
   | t2   |                                                  | B2：读取到state：initializing   |
   | t3   |                                                  | B3：释放初始化锁                |
   | t4   |                                                  | B4：在初始化锁的condition中等待 |

   ![image-20221212104920233](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212104920233.png)

   <center>类初始化-第二阶段</center>

3. 线程A设置state=initialized，然后唤醒在condition中等待的所有线程。

   ![image-20221212105120224](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212105120224.png)

   <center>类初始化-第3阶段</center>

   | 时间 | 线程A                                            | 线程B                           |
   | ---- | ------------------------------------------------ | ------------------------------- |
   | t1   | A1：执行类的静态初始化和初始化类中声明的静态字段 | B1：获取到初始化锁              |
   | t2   |                                                  | B2：读取到state：initializing   |
   | t3   |                                                  | B3：释放初始化锁                |
   | t4   |                                                  | B4：在初始化锁的condition中等待 |

4. 线程B结束类的初始化处理。

   ![image-20221212105228135](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212105228135.png)

   <center>类初始化-第4阶段</center>

   | 时间 | 线程B                        |
   | ---- | ---------------------------- |
   | t1   | B1：获取到初始化锁           |
   | t2   | B2：读取到state：initialized |
   | t3   | B3：释放初始化锁             |
   | t4   | B4：线程B的类初始化处理完成  |

   ![image-20221212105527570](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212105527570.png)

   <center>多线程执行时序图</center>

   多线程执行时序图线程A在第2阶段的A1执行类的初始化，并在第3阶段的A4释放初始化锁；线程B在第4阶段的B1获取同一个初始化锁，并在第4阶段的B4之后才开始访问这个类。根据Java内存模型规范的锁规则，这里将存在如下的happens-before关系。

   这个happens-before关系将保证：线程A执行类的初始化时的写入操作（执行类的静态初始化和初始化类中声明的静态字段），线程B一定能看到。

5. 线程C执行类的初始化的处理。

   ![image-20221212110059131](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212110059131.png)

<center>类初始化-第5阶段</center>

| 时间 | 线程B                           |
| ---- | ------------------------------- |
| t1   | C1:获取初始化锁                 |
| t2   | C2：读取到state=initalized      |
| t3   | C3：释放初始化锁                |
| t4   | C4：线程C的类初始化处理过程完成 |

在第3阶段之后，类已经完成了初始化。因此线程C在第5阶段的类初始化处理过程相对简单一些（前面的线程A和B的类初始化处理过程都经历了两次锁获取-锁释放，而线程C的类初始化处理只需要经历一次锁获取-锁释放）。

线程A在第2阶段的A1执行类的初始化，并在第3阶段的A4释放锁；线程C在第5阶段的C1获取同一个锁，并在在第5阶段的C4之后才开始访问这个类。根据Java内存模型规范的锁规则，将存在如下的happens-before关系。

这个happens-before关系将保证：线程A执行类的初始化时的写入操作，线程C一定能看到。

> 注意：这里的condition和state标记是本文虚构出来的。Java语言规范并没有硬性规定一定要使用condition和state标记。JVM的具体实现只要实现类似功能即可。
>
> Java语言规范允许Java的具体实现，优化类的初始化处理过程（对这里的第5阶段做优化），具体细节参见Java语言规范的12.4.2节。

![image-20221212111053580](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212111053580.png)

<center>多线程执行时序图</center>

通过对比基于volatile的双重检查锁定的方案和基于类初始化的方案，我们会发现基于类初始化的方案的实现代码更简洁。但基于volatile的双重检查锁定的方案有一个额外的优势：除了可以对静态字段实现延迟初始化外，还可以对实例字段实现延迟初始化。

字段延迟初始化降低了初始化类或创建实例的开销，但增加了访问被延迟初始化的字段的开销。在大多数时候，正常的初始化要优于延迟初始化。如果确实需要对实例字段使用线程安全的延迟初始化，请使用上面介绍的基于volatile的延迟初始化的方案；如果确实需要对静态字段使用线程安全的延迟初始化，请使用上面介绍的基于类初始化的方案。

## Java内存模型综述

下面对java内存模型的相关知识进行总结。

### 处理器的内存模型

顺序一致性内存模型是一个理论参考模型，JMM和处理器内存模型在设计时通常会以顺序一致性内存模型为参照。在设计时，JMM和处理器内存模型会对顺序一致性模型做一些放松，因为如果完全按照顺序一致性模型来实现处理器和JMM，那么很多的处理器和编译器优化都要被禁止，这对执行性能将会有很大的影响。

根据对不同类型的读/写操作组合的执行顺序的放松，可以把常见处理器的内存模型划分为如下几种类型。

- 放松程序中写-读操作的顺序，由此产生了Total Store Ordering内存模型（简称TSO）。
- 在上面基础上，继续放松程序中写-写操作的顺序，由此产生了Partial Store Order内存模型（简称为PSO）。
- 在前面两条的基础上，继续放松程序中读-写和读-读操作的顺序，由此产生了Relaxed Memory Order内存模型（简称为RMO）和PowerPC内存模型。

这里处理器对读/写操作的放松，是以两个操作之间不存在数据依赖性为前提的。

| 内存模型名称 | 对应的处理器  | Store-Load重排序 | Store-Store重排序 | Load-Load和Load-Store重排序 | 可以更早读取到其它处理器的写 | 可以更早读取到当前处理器的写 |
| ------------ | ------------- | ---------------- | ----------------- | --------------------------- | ---------------------------- | ---------------------------- |
| TSO          | sparc-TSO X64 | Y                |                   |                             |                              | Y                            |
| PSO          | sparc-PSO     | Y                | Y                 |                             |                              | Y                            |
| RMO          | ia64          | Y                | Y                 | Y                           |                              | Y                            |
| PowerPC      | PowerPC       | Y                | Y                 | Y                           | Y                            | Y                            |

从表中可以看到，所有处理器内存模型都允许写-读重排序，原因在第1章已经说明过：它们都使用了写缓存区。写缓存区可能导致写-读操作重排序。同时，我们可以看到这些处理器内存模型都允许更早读到当前处理器的写，原因同样是因为写缓存区。由于写缓存区仅对当前处理器可见，这个特性导致当前处理器可以比其他处理器先看到临时保存在自己写缓存区中的写。

表中的各种处理器内存模型，从上到下，模型由强变弱。越是追求性能的处理器，内存模型设计得会越弱。

由于常见的处理器内存模型比JMM要弱，Java编译器在生成字节码时，会在执行指令序列的适当位置插入内存屏障来限制处理器的重排序。同时，由于各种处理器内存模型的强弱不同，为了在不同的处理器平台向程序员展示一个一致的内存模型，JMM在不同的处理器中需要插入的内存屏障的数量和种类也不相同。

![image-20221212143507687](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212143507687.png)

<center>JMM插入内存屏障的示意图</center>

### 各种内存模型之间的关系

![image-20221212143618752](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212143618752.png)

<center>各种CPU内存模型的强弱对比示意图</center>

从图中可以看出：常见的4种处理器内存模型比常用的3中语言内存模型要弱，处理器内存模型和语言内存模型都比顺序一致性内存模型要弱。同处理器内存模型一样，越是追求执行性能的语言，内存模型设计得会越弱。

### JMM的内存可见性保证

java程序的内存可见性保证可以分为下列3类。

- 单线程程序。单线程程序不会出现内存可见性问题。
- 正确同步的多线程程序。正确同步的多线程程序的执行将具有顺序一致性（程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同）。这是JMM关注的重点，JMM通过限制编译器和处理器的重排序来为程序员提供内存可见性保证。
- 未同步/未正确同步的多线程程序。JMM为它们提供了最小安全性保障：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0、null、false）。

注意，最小安全性保障与64位数据的非原子性写并不矛盾。它们是两个不同的概念，它们“发生”的时间点也不同。最小安全性保证对象默认初始化之后（设置成员域为0、null或false），才会被任意线程使用。最小安全性“发生”在对象被任意线程使用之前。64位数据的非原子性写“发生”在对象被多个线程使用的过程中（写共享变量）。当发生问题时（处理器B看到仅仅被处理器A“写了一半”的无效值），这里虽然处理器B读取到一个被写了一半的无效值，但这个值仍然是处理器A写入的，只不过是处理器A还没有写完而已。最小安全性保证线程读取到的值，要么是之前某个线程写入的值，要么是默认值（0、null、false）。但最小安全性并不保证线程读取到的值，一定是某个线程写完后的值。最小安全性保证线程读取到的值不会无中生有的冒出来，但并不保证线程读取到的值一定是正确的。

![image-20221212144056504](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212144056504.png)

<center>3类程序的执行结果的对比图</center>

只要多线程程序是正确同步的，JMM保证该程序在任意的处理器平台上的执行结果，与该程序在顺序一致性内存模型中的执行结果一致。

### JSR-133对旧内存模型的修补

- 增强volatile语义。旧内存模型允许volatile变量与普通变量重排序。JSR-133严格限制volatile变量与普通变量的重排序，使volatile的写-读和锁的释放-获取具有相同的内存语义。
- 增强final语义。在旧内存模型中，多次读取同一个final变量的值可能会不相同。为此，JSR-133为final增加了两个重排序规则。在保证final引用不会从构造函数内逸出的情况下，final具有了初始化安全性。
