---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## ChannelHandler家族

### Channel的生命周期

Interface Channel 定义了一组和ChannelInboundHandler API 密切相关的简单但功能强大的状态模型，表1列出了Channel的4种状态。

| 状态                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| ChannelUnregistered | Channel已经被创建，但还未注册到EventLoop                     |
| ChannelRegistered   | Channel 已经被注册到了EventLoop                              |
| ChannelActive       | Channel 处于活动状态（已经连接到它的远程节点）。它现在可以接收和发送数据了 |
| ChannelInActive     | Channel 没有连接到远程节点                                   |

Channel 的正常生命周期如图1所示。当这些状态发生改变时，将会生成对应的事件。这些事件将会被转发给ChannelPipeline 中的ChannelHandler，其可以随后对它们做出响应。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608143250.png)

<center> 图1 Channel的状态模型 </center>

### ChannelHandler的生命周期

表2列出了 interface ChannelHandler 定义的生命周期操作，在 ChannelHandler 被添加到 ChannelPipeline 中或者被从 ChannelPipeline 中移除时会调用这些操作。

| 类型            | 描述                                               |
| --------------- | -------------------------------------------------- |
| handlerAdded    | 当把 ChannelHandler 添加到 ChannelPipeline 中时被调用 |
| handlerRemoved  | 当从 ChannelPipeline 中移除 ChannelHandler 时被调用   |
| exceptionCaught | 当处理过程中 ChannelPipeline 中有错误产生时被调用    |

Netty 定义了下面两个重要的ChannelHandler 子接口：

- ChannelInboundHandler——处理入站数据以及各种状态变化；
- ChannelOutboundHandler——处理出站数据并且允许拦截所有的操作。

### ChannelInboundHandler接口

表3列出了interface ChannelInboundHandler的生命周期的方法。这些方法将会在数据被接收时或者与其对应的Channel 状态发生改变时被调用。

| 类型                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| channelRegistered         | 当channel已经注册到它的EventLoop并且能够处理I/O时被调用      |
| channelUnregistered       | 当Channel从它的EventLoop注销并且无法处理任何I/O时被调用      |
| channelActive             | 当Channel 处于活动状态时被调用；Channel 已经连接/绑定并且已经就绪 |
| channelInactive           | 当Channel 离开活动状态并且不再连接它的远程节点时被调用       |
| channelReadComplete       | 当Channel上的一个读操作完成时被调用                          |
| channelRead               | 当从Channel 读取数据时被调用                                 |
| ChannelWritabilityChanged | 当Channel 的可写状态发生改变时被调用。用户可以确保写操作不会完成得太快（以避免发生OutOfMemoryError）或者可以在Channel 变为再次可写时恢复写入。可以通过调用Channel 的isWritable()方法来检测Channel 的可写性。与可写性相关的阈值可以通过Channel.config().setWriteHighWaterMark()和Channel.config().setWriteLowWaterMark()方法来设置 |
| userEventTriggered        | 当 ChannelInboundHandler.fireUserEventTriggered() 方法被调用时被调用，因为一个 POJO 被传递过了 ChannelPipeline |

当某个ChannelInboundHandler 的实现重写channelRead()方法时，它将负责显式地释放与池化的ByteBuf 实例相关的内存。Netty 为此提供了一个实用方法ReferenceCountUtil.release()。

### ChannelOutboundHandler 接口

出站操作和数据将由ChannelOutboundHandler 处理。它的方法将被Channel、ChannelPipeline 以及ChannelHandlerContext 调用。

ChannelOutboundHandler 的一个强大的功能是可以按需推迟操作或者事件，这使得可以通过一些复杂的方法来处理请求。例如，如果到远程节点的写入被暂停了，那么你可以推迟冲刷操作并在稍后继续。

表4显示了所有由ChannelOutboundHandler 本身所定义的方法（忽略了那些从ChannelHandler 继承的方法）。

| 类型                                                         | 描述                                               |
| ------------------------------------------------------------ | -------------------------------------------------- |
| bind(ChannelHandlerContext,SocketAddress,ChannelPromise)     | 当请求将Channel 绑定到本地地址时被调用             |
| connect(ChannelHandlerContext,SocketAddress,SocketAddress,ChannelPromise) | 当请求将Channel 连接到远程节点时被调用             |
| disconnect(ChannelHandlerContext,ChannelPromise)             | 当请求将Channel 从远程节点断开时被调用             |
| close(ChannelHandlerContext,ChannelPromise)                  | 当请求关闭Channel 时被调用                         |
| deregister(ChannelHandlerContext,ChannelPromise)             | 当请求将Channel 从它的EventLoop 注销时被调用       |
| read(ChannelHandlerContext)                                  | 当请求从Channel 读取更多的数据时被调用             |
| flush(ChannelHandlerContext)                                 | 当请求通过Channel 将入队数据冲刷到远程节点时被调用 |
| write(ChannelHandlerContext,Object,ChannelPromise)           | 当请求通过Channel 将数据写到远程节点时被调用       |

### ChannelHandler适配器

你可以使用ChannelInboundHandlerAdapter 和ChannelOutboundHandlerAdapter类作为自己的ChannelHandler 的起始点。这两个适配器分别提供了ChannelInboundHandler和ChannelOutboundHandler 的基本实现。通过扩展抽象类ChannelHandlerAdapter，它们获得了它们共同的超接口ChannelHandler 的方法。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608145046.png)

<center> 图2 ChannelHandlerAdapter类的层次结构 </center>

ChannelHandlerAdapter 还提供了实用方法isSharable()。如果其对应的实现被标注为Sharable，那么这个方法将返回true，表示它可以被添加到多个ChannelPipeline中。

### 资源管理

每当通过调用ChannelInboundHandler.channelRead()或者ChannelOutboundHandler.write()方法来处理数据时，你都需要确保没有任何的资源泄漏。

Netty 目前定义了4 种泄漏检测级别，如表5所示：

| 级别     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| DISABLED | 禁止泄漏检测。只有在详尽的测试之后才应设置为这个值           |
| SIMPLE   | 使用1%的默认采样率检测并报告任何发现的泄露。这是默认级别，适合绝大部分的情况 |
| ADVANCED | 使用默认的采样率，报告所发现的任何的泄露以及对应的消息被访问的位置 |
| PARANOID | 类似于ADVANCED，但是其将会对每次（对消息的）访问都进行采样。这对性能将会有很大的影响，应该只在调试阶段使用 |

> 消费入站消息的简单方式 由于消费入站数据是一项常规任务，所以Netty 提供了一个特殊的被称为SimpleChannelInboundHandler 的ChannelInboundHandler 实现。这个实现会在消息被channelRead0()方法消费之后自动释放消息。

## ChannelPipeline接口

每一个新创建的Channel 都将会被分配一个新的ChannelPipeline。这项关联是永久性的；Channel 既不能附加另外一个ChannelPipeline，也不能分离其当前的。在Netty 组件的生命周期中，这是一项固定的操作，不需要开发人员的任何干预。

> ChannelHandlerContext
>
> ChannelHandlerContext使得ChannelHandler能够和它的ChannelPipeline以及其他的ChannelHandler 交互。ChannelHandler 可以通知其所属的ChannelPipeline 中的下一个
> ChannelHandler，甚至可以动态修改它所属的ChannelPipeline。ChannelHandlerContext 具有丰富的用于处理事件和执行I/O 操作的API。

图3展示了一个典型的同时具有入站和出站ChannelHandler 的ChannelPipeline 的布局，并且印证了我们之前的关于ChannelPipeline 主要由一系列的ChannelHandler 所组成的说法。ChannelPipeline 还提供了通过ChannelPipeline 本身传播事件的方法。如果一个入站事件被触发，它将被从ChannelPipeline 的头部开始一直被传播到Channel Pipeline 的尾端。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608150138.png)

<center> 图3 ChannelPipeline 和它的ChannelHandler </center>

在ChannelPipeline 传播事件时，它会测试ChannelPipeline 中的下一个ChannelHandler 的类型是否和事件的运动方向相匹配。如果不匹配，ChannelPipeline 将跳过该ChannelHandler 并前进到下一个，直到它找到和该事件所期望的方向相匹配的为止。

### 修改ChannelPipeline

ChannelHandler 可以通过添加、删除或者替换其他的ChannelHandler 来实时地修改ChannelPipeline 的布局。这是ChannelHandler 最重要的能力之一，下表展示了相关方法：

| 名 称                             | 描 述                                                        |
| --------------------------------- | ------------------------------------------------------------ |
| addFirst、addBefore、addAfter、addLast | 将一个ChannelHandler 添加到ChannelPipeline 中                |
| remove                            | 将一个ChannelHandler 从ChannelPipeline 中移除                |
| replace                           | 将ChannelPipeline 中的一个ChannelHandler 替换为另一个ChannelHandler |

> 通常ChannelPipeline 中的每一个ChannelHandler 都是通过它的EventLoop（I/O 线程）来处理传递给它的事件的。所以至关重要的是不要阻塞这个线程，因为这会对整体的I/O 处理产生负面的影响。
>
> 但有时可能需要与那些使用阻塞API 的遗留代码进行交互。对于这种情况，ChannelPipeline 有一些接受一个EventExecutorGroup 的add()方法。如果一个事件被传递给一个自定义的EventExecutorGroup，它将被包含在这个EventExecutorGroup 中的某个EventExecutor 所处理，从而被从该Channel 本身的EventLoop 中移除。对于这种用例，Netty 提供了一个叫DefaultEventExecutorGroup 的默认实现。

除了这些操作，还有别的通过类型或者名称来访问ChannelHandler 的方法。这些方法都列在了表7之中。

| 名 称   | 描 述                                            |
| ------- | ------------------------------------------------ |
| get     | 通过类型或者名称返回ChannelHandler               |
| context | 返回和ChannelHandler 绑定的ChannelHandlerContext |
| names   | 返回ChannelPipeline 中所有ChannelHandler 的名称  |

### 触发事件

ChannelPipeline 的API 公开了用于调用入站和出站操作的附加方法。表8列出了入站操作。用于通知ChannelInboundHandler 在ChannelPipeline 中所发生的事件。

| 方法名称                      | 描 述                                                        |
| ----------------------------- | ------------------------------------------------------------ |
| fireChannelRegistered         | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelRegistered(ChannelHandlerContext)方法 |
| fireChannelUnregistered       | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelUnregistered(ChannelHandlerContext)方法 |
| fireChannelActive             | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelActive(ChannelHandlerContext)方法 |
| fireChannelInactive           | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelInactive(ChannelHandlerContext)方法 |
| fireExceptionCaught           | 调用ChannelPipeline 中下一个ChannelInboundHandler 的exceptionCaught(ChannelHandlerContext, Throwable)方法 |
| fireUserEventTriggered        | 调用ChannelPipeline 中下一个ChannelInboundHandler 的userEventTriggered(ChannelHandlerContext, Object)方法 |
| fireChannelRead               | 调用ChannelPipeline 中下一个ChannelInboundHandler的channelRead(ChannelHandlerContext, Object msg)方法 |
| fireChannelReadComplete       | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelReadComplete(ChannelHandlerContext)方法 |
| fireChannelWritabilityChanged | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelWritabilityChanged(ChannelHandlerContext)方法 |

在出站这边，处理事件将会导致底层的套接字上发生一系列的动作。详情看书吧。

总结：

- ChannelPipeline 保存了与Channel 相关联的ChannelHandler；
- ChannelPipeline 可以根据需要，通过添加或者删除ChannelHandler 来动态地修改；
- ChannelPipeline 有着丰富的API 用以被调用，以响应入站和出站事件。

## ChannelHandlerContext接口

ChannelHandlerContext 代表了ChannelHandler 和ChannelPipeline 之间的关联，每当有ChannelHandler 添加到ChannelPipeline 中时，都会创建ChannelHandlerContext。ChannelHandlerContext 的主要功能是管理它所关联的ChannelHandler 和在同一个ChannelPipeline 中的其他ChannelHandler 之间的交互。

### 使用ChannelHandlerContext接口

将讨论ChannelHandlerContext 的用法，以及存在于ChannelHandlerContext、Channel 和ChannelPipeline 上的方法的行为。图4展示了它们之间的关系。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608151531.png)

<center> 图4 Channel、ChannelPipeline、ChannelHandler 以及ChannelHandlerContext 之间的关系 </center>

下图展示了Channel或者ChannelPipeline进行的事件传播

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608151703.png)

<center> 图5 通过Channel或者ChannelPipeline进行的事件传播 </center>

为什么会想要从ChannelPipeline 中的某个特定点开始传播事件呢？

- 为了减少将事件传递过对它不感兴趣的ChannelHandler 所带来的开销。
- 为了避免将事件传递给那些可能会对它感兴趣的ChannelHandler。

要想调用从某个特定的ChannelHandler 开始的处理过程，必须获取到在（ChannelPipeline）该ChannelHandler 之前的ChannelHandler 所关联的ChannelHandlerContext。这个ChannelHandlerContext 将调用和它所关联的ChannelHandler 之后的ChannelHandler。下图展示了这种用法。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230608151929.png)

<center> 通过ChannelHandlerContext触发的操作的事件流 </center>

### ChannelHandler 和ChannelHandlerContext 的高级用法

可以通过调用ChannelHandlerContext 上的pipeline()方法来获得被封闭的ChannelPipeline 的引用。这使得运行时得以操作ChannelPipeline 的ChannelHandler，我们可以利用这一点来实现一些复杂的设计。

另一种高级的用法是缓存到ChannelHandlerContext 的引用以供稍后使用，这可能会发生在任何的ChannelHandler 方法之外，甚至来自于不同的线程。

## 异常处理

异常处理是任何真实应用程序的重要组成部分，它也可以通过多种方式来实现。

### 处理入站异常

如果在处理入站事件的过程中有异常被抛出，那么它将从它在ChannelInboundHandler里被触发的那一点开始流经ChannelPipeline。要想处理这种类型的入站异常，你需要在你的ChannelInboundHandler 实现中重写下面的方法。

因为异常将会继续按照入站方向流动（就像所有的入站事件一样），所以实现了前面所示逻辑的ChannelInboundHandler 通常位于ChannelPipeline 的最后。这确保了所有的入站异常都总是会被处理，无论它们可能会发生在ChannelPipeline 中的什么位置。

总结：

- ChannelHandler.exceptionCaught()的默认实现是简单地将当前异常转发给ChannelPipeline 中的下一个ChannelHandler；
- 如果异常到达了ChannelPipeline 的尾端，它将会被记录为未被处理；
- 要想定义自定义的处理逻辑，你需要重写exceptionCaught()方法。然后你需要决定是否需要将该异常传播出去。

### 处理出站异常

用于处理出站操作中的正常完成以及异常的选项，都基于以下的通知机制。

- 每个出站操作都将返回一个ChannelFuture。注册到ChannelFuture 的ChannelFutureListener 将在操作完成时被通知该操作是成功了还是出错了。
- 几乎所有的ChannelOutboundHandler 上的方法都会传入一个ChannelPromise的实例。作为ChannelFuture 的子类，ChannelPromise 也可以被分配用于异步通知的监听器。但是，ChannelPromise 还具有提供立即通知的可写方法：

```java
ChannelPromise setSuccess();
Chan nelPromise setFailure(Throwable cause);
```

添加ChannelFutureListener 只需要调用ChannelFuture 实例上的addListener(ChannelFutureListener)方法，并且有两种不同的方式可以做到这一点。其中最常用的方式是，调用出站操作（如write()方法）所返回的ChannelFuture 上的addListener()方法。

## 总结

在本章中我们仔细地研究了Netty 的数据处理组件—ChannelHandler。我们讨论了ChannelHandler 是如何链接在一起，以及它们是如何作为ChannelInboundHandler 和ChannelOutboundHandler 与ChannelPipeline 进行交互的。
