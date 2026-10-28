---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/%E6%8F%90%E7%BA%B2.png)

## CPU是如何读写数据的？

CPU的架构在[[04_CPU缓存一致性]]以及[[03_如何写出让 CPU 跑得更快的代码？]]中已经详细介绍了。对应的结构图如下：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/CPU%25E6%259E%25B6%25E6%259E%2584.png)

整个存储设备构成的结构如下：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%25AD%2598%25E5%2582%25A8%25E5%2599%25A8%25E7%259A%2584%25E5%25B1%2582%25E6%25AC%25A1%25E5%2585%25B3%25E7%25B3%25BB%25E5%259B%25BE.png)

然后咱们在前面也已经说了，CPU cache再读入数据的时候采用的是整体读入，因此在进行读入的时候使用的是64字节。

这就会产生一个新的问题：假设说咱们现在核心A修改A数据，核心B修改B数据，核心A和核心B之间是相连的，这个时候将会同时将AB读入到CPU中，然后咱们这个hi后可能就会产生伪共享问题。

### 伪共享问题

还是从上面的东西进行举例：

1. 最开始变相A和B还不在cache中，假设1号线程绑定了线程A，2号线程绑定了线程B，线程A指挥读写变量A，线程B指挥读写变量B。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%2588%2586%25E6%259E%2590%25E4%25BC%25AA%25E5%2585%25B1%25E4%25BA%25AB1.png)

2. 1号核心读取变量A，由于CPU从内存读取数据到Cache的但伪Cache Line，也正好变量A和变量B的数据归属同一个Cache Line，所以AB的数据会加载到同一个cache Line ，所以A和B的数据都会被加载Cache，并将此Cache Line标记为独占状态。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%2588%2586%25E6%259E%2590%25E4%25BC%25AA%25E5%2585%25B1%25E4%25BA%25AB2.png)

3. 接着，2 号核心开始从内存里读取变量 B，同样的也是读取 Cache Line 大小的数据到 Cache 中，此 Cache Line 中的数据也包含了变量 A 和 变量 B，此时 1 号和 2 号核心的 Cache Line 状态变为「共享」状态。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%2588%2586%25E6%259E%2590%25E4%25BC%25AA%25E5%2585%25B1%25E4%25BA%25AB3.png)

4.  1 号核心需要修改变量 A，发现此 Cache Line 的状态是「共享」状态，所以先需要通过总线发送消息给 2 号核心，通知 2 号核心把 Cache 中对应的 Cache Line 标记为「已失效」状态，然后 1 号核心对应的 Cache Line 状态变成「已修改」状态，并且修改变量 A。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%2588%2586%25E6%259E%2590%25E4%25BC%25AA%25E5%2585%25B1%25E4%25BA%25AB4.png)

5. 之后，2 号核心需要修改变量 B，此时 2 号核心的 Cache 中对应的 Cache Line 是已失效状态，另外由于 1 号核心的 Cache 也有此相同的数据，且状态为「已修改」状态，所以要先把 1 号核心的 Cache 对应的 Cache Line 写回到内存，然后 2 号核心再从内存读取 Cache Line 大小的数据到 Cache 中，最后把变量 B 修改到 2 号核心的 Cache 中，并将状态标记为「已修改」状态。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%2588%2586%25E6%259E%2590%25E4%25BC%25AA%25E5%2585%25B1%25E4%25BA%25AB5.png)

所以，可以发现如果 1 号和 2 号 CPU 核心这样持续交替的分别修改变量 A 和 B，就会重复 ④ 和 ⑤ 这两个步骤，Cache 并没有起到缓存的效果，虽然变量 A 和 B 之间其实并没有任何的关系，但是因为同时归属于一个 Cache Line ，这个 Cache Line 中的任意数据被修改后，都会相互影响，从而出现 ④ 和 ⑤ 这两个步骤。

综上，应为多个线程同时读写同一个Cache Line的不同变量是，而导致CPU Cache失效的现象称为伪共享。

### 避免伪共享的方法

很简单，咱们在进行数据读取的时候，采用填位的方法读取占住位数，保证一个Cache Line中只有一个变量。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/struct_ab1.png)

Java并发框架Disrupt使用`字节填充+继承`的方式，来避免伪共享的问题。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/Disruptor.png)

根据JVM对象继承关系中父类成员和子类成员，内存地址是连续排列布局的，因此RingBufferPad中7个long类型数据作为Cache Line迁至填充，而RingBuffer中7个long类型数据作为Cache Line后置填充，这14个long变量没有任何实际用途，更不会对它们进行读写操作。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E5%25A1%25AB%25E5%2585%2585%25E5%25AD%2597%25E8%258A%2582.png)

**由于「前后」各填充了 7 个不会被读写的 long 类型变量，所以无论怎么加载 Cache Line，这整个 Cache Line 里都没有会发生更新操作的数据，于是只要数据被频繁地读取访问，就自然没有数据被换出 Cache 的可能，也因此不会产生伪共享的问题**。

## CPU是如何选择线程的

根据线程的优先级进行选择，优先级数字越小，优先级级别越高。

### 调度类

由于任务有优先级之分，Linux 系统为了保障高优先级的任务能够尽可能早的被执行，于是分为了这几种调度类，如下图：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E8%25B0%2583%25E5%25BA%25A6%25E7%25B1%25BB.png)

Deadline 和 Realtime 这两个调度类，都是应用于实时任务的，这两个调度类的调度策略合起来共有这三种，它们的作用如下：

- *SCHED_DEADLINE*：是按照 deadline 进行调度的，距离当前时间点最近的 deadline 的任务会被优先调度；
- *SCHED_FIFO*：对于相同优先级的任务，按先来先服务的原则，但是优先级更高的任务，可以抢占低优先级的任务，也就是优先级高的可以「插队」；
- *SCHED_RR*：对于相同优先级的任务，轮流着运行，每个任务都有一定的时间片，当用完时间片的任务会被放到队列尾部，以保证相同优先级任务的公平性，但是高优先级的任务依然可以抢占低优先级的任务；

而Fair调度类是应用于普通任务，都是由FCS调度器管理的，分为两种调度策略：

- *SCHED_NORMAL*：普通任务使用的调度策略；
- *SCHED_BATCH*：后台任务的调度策略，不和终端进行交互，因此在不影响其他需要交互的任务，可以适当降低它的优先级。

### 完全公平调度

这个的话即使基于CFS调度算法，也就是完全公平调度（Completely Fair Scheduling）。

**在 CFS 算法调度的时候，会优先选择 vruntime 少的任务**，以保证每个任务的公平性。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E4%BC%AA%E5%85%B1%E4%BA%AB/vruntime.png)

### CPU运行队列

这个很好理解，咱们现在再运行的时候，存在一个系统通常都会运行着很多任务，多任务的数量基本都是远超 CPU 核心数量，因此这时候就需要**排队**。

每个 CPU 都有自己的**运行队列（\*Run Queue, rq\*）**

这几种调度类是有优先级的，优先级如下：Deadline > Realtime > Fair，这意味着 Linux 选择下一个任务执行的时候，会按照此优先级顺序进行选择，也就是说先从 `dl_rq` 里选择任务，然后从 `rt_rq` 里选择任务，最后从 `cfs_rq` 里选择任务。因此，**实时任务总是会比普通任务优先被执行**。

### 调整优先级

调整任务的`nice`值，从而让高一些的任务执行更多的时间。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E4%25BC%2598%25E5%2585%2588%25E7%25BA%25A7.png)

## 总结

CPU 读写数据的时候，并不是按一个一个字节为单位来进行读写，而是以 CPU Cache Line 大小为单位，CPU Cache Line 大小一般是 64 个字节，也就意味着 CPU 读写数据的时候，每一次都是以 64 字节大小为一块进行操作。

因此，如果我们操作的数据是数组，那么访问数组元素的时候，按内存分布的地址顺序进行访问，这样能充分利用到 Cache，程序的性能得到提升。但如果操作的数据不是数组，而是普通的变量，并在多核 CPU 的情况下，我们还需要避免 Cache Line 伪共享的问题。

所谓的 Cache Line 伪共享问题就是，多个线程同时读写同一个 Cache Line 的不同变量时，而导致 CPU Cache 失效的现象。那么对于多个线程共享的热点数据，即经常会修改的数据，应该避免这些数据刚好在同一个 Cache Line 中，避免的方式一般有 Cache Line 大小字节对齐，以及字节填充等方法。

系统中需要运行的多线程数一般都会大于 CPU 核心，这样就会导致线程排队等待 CPU，这可能会产生一定的延时，如果我们的任务对延时容忍度很低，则可以通过一些人为手段干预 Linux 的默认调度策略和优先级。
