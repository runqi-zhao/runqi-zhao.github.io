---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 线程模型概述

单线程 -> 多线程 ->线程池->Netty提供的线程池

## EventLoop接口

运行任务来处理在连接的生命周期内发生的事件是任何网络框架的基本功能。与之相应的编程上的构造通常被称为事件循环—一个Netty 使用了interface io.netty.channel.EventLoop 来适配的术语。

Netty 的EventLoop 是协同设计的一部分，它采用了两个基本的API：并发和网络编程。首先，io.netty.util.concurrent 包构建在JDK 的java.util.concurrent 包上，用来提供线程执行器。其次，io.netty.channel 包中的类，为了与Channel 的事件进行交互，扩展了这些接口/类。图1展示了生成类的层次结构。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608160335.png)

<center> 图1 EventLoop的类层次结构 </center>

在这个模型中，一个EventLoop 将由一个永远都不会改变的Thread 驱动，同时任务（Runnable 或者Callable）可以直接提交给EventLoop 实现，以立即执行或者调度执行。根据配置和可用核心的不同，可能会创建多个EventLoop 实例用以优化资源的使用，并且单个EventLoop 可能会被指派用于服务多个Channel。

需要注意的是，Netty的EventLoop在继承了ScheduledExecutorService的同时，只定义了一个方法，parent()。这个方法，如下面的代码片断所示，用于返回到当前EventLoop实现的实例所属的EventLoopGroup的引用。

> 事件/任务的执行顺序 事件和任务是以先进先出（FIFO）的顺序执行的。这样可以通过保证字节内容总是按正确的顺序被处理，消除潜在的数据损坏的可能性。

### Netty 4中I/O和事件处理

在Netty 4 中，所有的I/O操作和事件都由已经被分配给了EventLoop的那个Thread来处理

### Netty 3 中的I/O 操作

在Netty 3 的模型中，由于这是一个入站事件，需要在调用线程中执行代码，然后将事件移交给I/O 线程去执行，然而这将带来额外的上下文切换。

## 任务调度

### JDK 的任务调度API

在Java 5 之前，任务调度是建立在java.util.Timer 类之上的，其使用了一个后台Thread，并且具有与标准线程相同的限制。随后，JDK 提供了java.util.concurrent 包，它定义了interface ScheduledExecutorService。下标定义了java.util.cocurrent,Executors的相关工厂方法。

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| newScheduledThreadPool(int corePoolSize)<br/>newScheduledThreadPool(int corePoolSize,ThreadFactorythreadFactory) | 创建一个ScheduledThreadExecutorService，用于调度命令在指定延迟之后运行或者周期性地执行。它使用corePoolSize 参数来计算线程数 |
| newSingleThreadScheduledExecutor()<br/>newSingleThreadScheduledExecutor(ThreadFactorythreadFactory) | 创建一个ScheduledThreadExecutorService，用于调度命令在指定延迟之后运行或者周期性地执行。它使用一个线程来执行被调度的任务 |

### 使用EventLooop执行任务调度

ScheduledExecutorService 的实现具有局限性，例如，事实上作为线程池管理的一部分，将会有额外的线程创建。如果有大量任务被紧凑地调度，那么这将成为一个瓶颈。

## 实现细节

这一节将更加详细地探讨Netty 的线程模型和任务调度实现的主要内容。我们也将会提到需要注意的局限性，以及正在不断发展中的领域。

### 线程管理

Netty线程模型的卓越性能取决于对于当前执行的Thread的身份的确定。也就是说，确定它是否是分配给当前Channel以及它的EventLoop的那一个线程。

如果（当前）调用线程正是支撑EventLoop 的线程，那么所提交的代码块将会被（直接）执行。否则，EventLoop 将调度该任务以便稍后执行，并将它放入到内部队列中。当EventLoop
下次处理它的事件时，它会执行队列中的那些任务/事件。这也就解释了任何的Thread 是如何与Channel 直接交互而无需在ChannelHandler 中进行额外同步的。

下图展示EventLoop的执行逻辑。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608162116.png)

<center> 图2 EventLoop的执行逻辑 </center>

### EventLoop/线程分配

服务于Channel 的I/O 和事件的EventLoop 包含在EventLoopGroup 中。根据不同的传输实现，EventLoop 的创建和分配方式也不同。

### 异步传输

异步传输实现只使用了少量的EventLoop（以及和它们相关联的Thread），而且在当前的线程模型中，它们可能会被多个Channel 所共享。这使得可以通过尽可能少量的Thread 来支撑大量的Channel，而不是每个Channel 分配一个Thread。

下图显示了一个EventLoopGroup，具有3个固定大小的EventLoop。在船舰EventLoopGroup时直接分配了EventLoop，一起确保在需要的时候时可用的。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608162511.png)

<center> 用于非阻塞传输（如NIO 和AIO）的EventLoop 分配方式 </center>

EventLoopGroup 负责为每个新创建的Channel 分配一个EventLoop。在当前实现中，使用顺序循环（round-robin）的方式进行分配以获取一个均衡的分布，并且相同的EventLoop
可能会被分配给多个Channel。（这一点在将来的版本中可能会改变。）

一旦一个Channel 被分配给一个EventLoop，它将在它的整个生命周期中都使用这个EventLoop（以及相关联的Thread）。请牢记这一点，因为它可以使你从担忧你的ChannelHandler 实现中的线程安全和同步问题中解脱出来。

### 阻塞传输

下图展示了阻塞传输的的EventLoop的分配方式。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608162748.png)

<center> 阻塞传输的EventLoop的分配方式 </center>

这里每一个Channel 都将被分配给一个EventLoop（以及它的Thread）。如果你开发的应用程序使用过java.io 包中的阻塞I/O 实现，你可能就遇到过这种模型。

但是，正如同之前一样，得到的保证是每个Channel 的I/O 事件都将只会被一个Thread（用于支撑该Channel 的EventLoop 的那个Thread）处理。这也是另一个Netty 设计一致性的例子，它（这种设计上的一致性）对Netty 的可靠性和易用性做出了巨大贡献。

## 总结

在本章中，你了解了通常的线程模型，并且特别深入地学习了Netty 所采用的线程模型，我们详细探讨了其性能以及一致性。

你看到了如何在EventLoop（I/O Thread）中执行自己的任务，就如同Netty 框架自身一样。你学习了如何调度任务以便推迟执行，并且我们还探讨了高负载下的伸缩性问题。你也看到
了如何验证一个任务是否已被执行以及如何取消它。
