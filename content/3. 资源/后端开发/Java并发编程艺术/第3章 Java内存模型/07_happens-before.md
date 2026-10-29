---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## JMM的设计

设计JMM时，考虑两个关键因素：

1. 程序员对内存模型的使用。程序员希望内存模型易于理解、易于编程。程序员希望基于一个强内存模型来编写代码。

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
2. 对于不会改变程序执行结果的重排序，JMM对编译器和处理器不做要求。

![image-20221212092306827](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212092306827.png)

<center>JMM的设计示意图</center>

从图中，可看出：

1. JMM向程序员提供的happens-before规则能满足程序员的需求。
2. JMM对编译器和处理器的束缚已经尽可能地少。只要不改变程序结果，想怎么优化就怎么优化。

## happens-before的定义

JSR-133使用happens-before的概念来指定两个操作之间的执行顺序。由于这两个操作可能在一个线程之内，也可能在不同线程之间。因此，JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证。（如果A线程的写操作a与B线程的读操作b之间存在happens before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。

因此，定义如下：

1. 如果一个操作happens before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
2. 两个操作之间存在happens-before关系，并不意味着要按照关系指定顺序来执行。如果重排序后的执行结果与按happens-before关系来执行的结果一致，那么这种操作不非法。

<font color="red">1是JMM对程序员的承诺</font>。从程序员的角度来说，可以这样理解happens-before关系：如果A happens-before B，那么Java内存模型将向程序员保证——A操作的结果将对B可见，且A的执行顺序排在B之前。注意，这只是Java内存模型向程序员做出的保证！

<font color="red">2是JMM对编译器和处理器重排序的约束原则。</font>即，happens-before关系本质上和as-if-serial语义是一回事。

- as-if-serial语义保证单线程内程序的执行结果不被改变，happens-before关系保证正确同步的多线程程序的执行结果不被改变。
- as-if-serial语义给编写单线程程序的程序员创造了一个幻境：单线程程序是按程序的顺序来执行的。happens-before关系给编写正确同步的多线程程序的程序员创造了一个幻境：正确同步的多线程程序是按happens-before指定的顺序来执行的。

二者目的都是为了不改变程序执行结果的前提下，尽可能地提高程序执行的并行速度。

## happens-before规则

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
6. join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

下图是volatile写-读建立的happens-before关系图。

![image-20221212094140462](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212094140462.png)

<center>happens-before关系的示意图</center>

分析如下：

1.  1 happens-before 2和3 happens-before 4由程序顺序规则产生。因此，可以把程序顺序规则看成是对as-if-serial语义的“封装”。
2.  2 happens-before 3是由volatile规则产生。对一个volatile变量的读，总是能看到(任意线程)之前这个volatile变量最后的写入。因此，volatile的这个特性可以保证实现volatile规则。
3.  1 happens-before 4是由传递性规则产生的。这里的传递性是由volatile的内存屏障插入策略和volatile的编译器重排序规则共同来保证的。

下面我们来看start()规则。线程A执行时会调用ThreadB.start()来启动线程B；假设线程A在执行Thread.start()之前修改了一些共享变量，线程B在开始执行之后会读这些共享变量。下图是对应的happens-before关系图。

![image-20221212100542849](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212100542849.png)

<center>happens-before关系的示意图</center>

上图中，1 happens-before 2由顺序规则产生。2 happens-before 4由start()规则产生。根据传递性，将有1 happens-before 4。这意味着，线程A在执行ThreadB.start()之前对共享变量所做的修改，接下来在线程B开始执行后都将确保对线程B可见。

下面我们来看join()规则。假设线程A在执行的过程中，通过执行ThreadB.join()来等待线程B终止；同时，假设线程B在终止之前修改了一些共享变量，线程A从ThreadB.join()返回后会读这些共享变量。

![image-20221212100843382](https://img-1312072469.cos.ap-nanjing.myqcloud.com/image-20221212100843382.png)

<center>happens-before关系的示意图</center>

上图中，2 happens-before 4由join()规则产生；4 happens-before 5由程序顺序规则产生。根据传递性规则，将有2 happens-before 5。这意味着，线程A执行操作ThreadB.join()并成功返回后，线程B中的任意操作都将对线程A可见。
