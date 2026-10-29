---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
本章主要内容：

- Netty 的技术和体系结构方面的内容

- Channel、EventLoop 和ChannelFuture

- ChannelHandler 和ChannelPipeline
- 引导

本书将从两个视角进行Netty的探讨：类库的视角以及框架的视角。对于使用Netty 编写高效的、可重用的和可维护的代码来说，两者缺一不可。

## Channel、EventLoop 和ChannelFuture

接下来的各节将会为我们对于Channel、EventLoop 和ChannelFuture 类进行的讨论增添更多的细节，这些类合在一起，可以被认为是Netty 网络抽象的代表：

- Channel-Socket
- EventLoop-控制流、多线程处理、并发
- ChannelFuture-异步通知

### Channel接口

基本的I/O 操作（bind()、connect()、read()和write()）依赖于底层网络传输所提供的原语。在基于Java 的网络编程中，其基本的构造是class Socket。Netty 的Channel 接口所提供的API，大大地降低了直接使用Socket 类的复杂性。此外，Channel 也是拥有许多预定义的、专门化实现的广泛类层次结构的根，下面是一个简短的部分清单：

- EmbeddedChannel；
- LocalServerChannel；
- NioDatagramChannel；
- NioSctpChannel；
- NioSocketChannel。

### EventLoop接口

EventLoop 定义了Netty 的核心抽象，用于处理连接的生命周期中所发生的事件。图1在高层次上说明了Channel、EventLoop、Thread以及EventLoopGroup之间的关系。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230606192743.png)

<center> 图1 Channel、EventLoop 和EventLoopGroup </center>

它们之间的关系是：

- 一个EventLoopGroup 包含一个或者多个EventLoop；
- 一个EventLoop 在它的生命周期内只和一个Thread 绑定；
- 所有由EventLoop 处理的I/O 事件都将在它专有的Thread 上被处理；
- 一个Channel 在它的生命周期内只注册于一个EventLoop；
- 一个EventLoop 可能会被分配给一个或多个Channel。

注意，在这种设计中，一个给定Channel 的I/O 操作都是由相同的Thread 执行的，实际上消除了对于同步的需要。

### ChannelFuture接口

正如我们已经解释过的那样，Netty 中所有的I/O 操作都是异步的。因为一个操作可能不会立即返回，所以我们需要一种用于在之后的某个时间点确定其结果的方法。为此，Netty 提供了
ChannelFuture 接口，其addListener()方法注册了一个ChannelFutureListener，以便在某个操作完成时（无论是否成功）得到通知。

> 可以将ChannelFuture 看作是将来要执行的操作的结果的占位符。它究竟什么时候被执行则可能取决于若干的因素，因此不可能准确地预测，但是可以肯定的是它将会被执行。此外，所有属于同一个Channel 的操作都被保证其将以它们被调用的顺序被执行。

## ChannelHandler和ChannelPipeline

### ChannelHandler接口

从应用程序开发人员的角度来看，Netty 的主要组件是ChannelHandler，它充当了所有处理入站和出站数据的应用程序逻辑的容器。这是可行的，因为ChannelHandler 的方法是由网络事件（其中术语“事件”的使用非常广泛）触发的。事实上，ChannelHandler 可专门用于几乎任何类型的动作，例如将数据从一种格式转换为另外一种格式，或者处理转换过程中所抛出的异常。

举例来说，ChannelInboundHandler 是一个你将会经常实现的子接口。这种类型的ChannelHandler 接收入站事件和数据，这些数据随后将会被你的应用程序的业务逻辑所处理。当你要给连接的客户端发送响应时，也可以从ChannelInboundHandler 冲刷数据。你的应用程序的业务逻辑通常驻留在一个或者多个ChannelInboundHandler 中。

### ChannelPipeline 接口

ChannelPipeline 提供了ChannelHandler 链的容器，并定义了用于在该链上传播入站和出站事件流的API。当Channel 被创建时，它会被自动地分配到它专属的ChannelPipeline。

ChannelHandler 安装到ChannelPipeline 中的过程如下所示：

- 一个ChannelInitializer的实现被注册到了ServerBootstrap中。
- 当ChannelInitializer.initChannel()方法被调用时，ChannelInitializer将在ChannelPipeline 中安装一组自定义的ChannelHandler；
- ChannelInitializer 将它自己从ChannelPipeline 中移除。

ChannelHandler 是专为支持广泛的用途而设计的，可以将它看作是处理往来Channel-Pipeline 事件（包括数据）的任何代码的通用容器。图2说明了这一点，其展示了从ChannelHandler 派生的ChannelInboundHandler 和ChannelOutboundHandler 接口。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230606193824.png)

<center> 图2 ChannelHandler类的层次结构</center>

使得事件流经ChannelPipeline 是ChannelHandler 的工作，它们是在应用程序的初始化或者引导阶段被安装的。这些对象接收事件、执行它们所实现的处理逻辑，并将数据传递给链中的下一个ChannelHandler。它们的执行顺序是由它们被添加的顺序所决定的。实际上，被我们称为ChannelPipeline 的是这些ChannelHandler 的编排顺序。

图3说明了一个Netty应用程序中入站和出站数据之间的区别。从一个客户端应用程序的角度来看，如果事件的运动方向是从客户端到服务器端，那么我们称这些事件为出站的，反之则称为入站的。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230606194016.png)

<center> 图3 包含入站和出站ChannelHandler 的ChannelPipeline </center>

图3也显示了入站和出站ChannelHandler 可以被安装到同一个ChannelPipeline中。如果一个消息或者任何其他的入站事件被读取，那么它会从ChannelPipeline 的头部开始流动，并被传递给第一个ChannelInboundHandler。这个ChannelHandler 不一定会实际地修改数据，具体取决于它的具体功能，在这之后，数据将会被传递给链中的下一个ChannelInboundHandler。最终，数据将会到达ChannelPipeline 的尾端，届时，所有处理就都结束了。

数据的出站运动（即正在被写的数据）在概念上也是一样的。在这种情况下，数据将从ChannelOutboundHandler 链的尾端开始流动，直到它到达链的头部为止。在这之后，出站数据将会到达网络传输层，这里显示为Socket。通常情况下，这将触发一个写操作。

> <center>关于入站和出站ChannelHandler 的更多讨论</center>
>
> 通过使用作为参数传递到每个方法的ChannelHandlerContext，事件可以被传递给当前ChannelHandler 链中的下一个ChannelHandler。因为你有时会忽略那些不感兴趣的事件，所以Netty提供了抽象基类ChannelInboundHandlerAdapter 和ChannelOutboundHandlerAdapter。通过调用ChannelHandlerContext 上的对应方法，每个都提供了简单地将事件传递给下一个ChannelHandler的方法的实现。随后，你可以通过重写你所感兴趣的那些方法来扩展这些类。

鉴于出站操作和入站操作是不同的，你可能会想知道如果将两个类别的ChannelHandler都混合添加到同一个ChannelPipeline 中会发生什么。虽然ChannelInboundHandler 和ChannelOutboundHandler 都扩展自ChannelHandler，但是Netty 能区分ChannelInboundHandler实现和ChannelOutboundHandler 实现，并确保数据只会在具有相同定向类型的两个ChannelHandler 之间传递。

当ChannelHandler 被添加到ChannelPipeline 时，它将会被分配一个ChannelHandlerContext，其代表了ChannelHandler 和ChannelPipeline 之间的绑定。虽然这个对象可以被用于获取底层的Channel，但是它主要还是被用于写出站数据。

在Netty 中，有两种发送消息的方式。你可以直接写到Channel 中，也可以写到和ChannelHandler相关联的ChannelHandlerContext 对象中。前一种方式将会导致消息从ChannelPipeline 的尾端开始流动，而后者将导致消息从ChannelPipeline 中的下一个ChannelHandler 开始流动。

### 更加深入的了解ChannelHandler

有许多不同类型的ChannelHandler，它们各自的功能主要取决于它们的超类。Netty 以适配器类的形式提供了大量默认的ChannelHandler 实现，其旨在简化应用程序处理逻辑的开发过程。你已经看到了，ChannelPipeline中的每个ChannelHandler将负责把事件转发到链中的下一个ChannelHandler。这些适配器类（及它们的子类）将自动执行这个操作，所以你可以只重写那些你想要特殊处理的方法和事件。

接下来我们将研究3 个ChannelHandler 的子类型：编码器、解码器和SimpleChannelInboundHandler<T> —— ChannelInboundHandlerAdapter 的一个子类。

#### 编码器和解码器

什么是编码和解码？

编码：将消息转换为字节。

解码：将字节转换为可阅读的消息。

编码字节与解码字节之间如何互相转换？

当你通过Netty 发送或者接收一个消息的时候，就将会发生一次数据转换。这两种方向的转换的原因很简单：网络数据总是一系列的字节。

Netty是如何实现编码器与解码器的？

对应于特定的需要，Netty 为编码器和解码器提供了不同类型的抽象类。例如，你的应用程序可能使用了一种中间格式，而不需要立即将消息转换成字节。你将仍然需要一个编码器，但是
它将派生自一个不同的超类。为了确定合适的编码器类型，你可以应用一个简单的命名约定。

请你具体说说Netty中可以使用哪些类实现编码解码？

通常来说，这些基类的名称将类似于ByteToMessageDecoder 或MessageToByteEncoder。对于特殊的类型，你可能会发现类似于ProtobufEncoder 和ProtobufDecoder这样的名称——预置的用来支持Google 的Protocol Buffers。

严格地说，其他的处理器也可以完成编码器和解码器的功能。但是，正如有用来简化ChannelHandler 的创建的适配器类一样，所有由Netty 提供的编码器/解码器适配器类都实现了ChannelOutboundHandler 或者ChannelInboundHandler 接口。

#### 抽象类SimpleChannelInboundHandler

抽象类SimpleChannelInboundHandler的作用是什么？

最常见的情况是，你的应用程序会利用一个ChannelHandler 来接收解码消息，并对该数据应用业务逻辑。要创建一个这样的ChannelHandler，你只需要扩展基类SimpleChannelInboundHandler<T>，其中T 是你要处理的消息的Java 类型。在这个ChannelHandler 中，你将需要重写基类的一个或者多个方法，并且获取一个到ChannelHandlerContext 的引用，这个引用将作为输入参数传递给ChannelHandler 的所有方法。

## 引导

什么是引导？

Netty 的引导类为应用程序的网络层配置提供了容器，这涉及将一个进程绑定到某个指定的端口，或者将一个进程连接到另一个运行在某个指定主机的指定端口上的进程。

引导有几种类型，分别是什么，区别又是什么？

引导有两种类型：

1. 用于客户端（Bootstrap）
2. 用于服务器（ServerBootstrap）

无论使用哪种协议或者哪种类型，唯一决定是哪种类的是看它是服务器还是客户端，下表展示了这两种引导类型

<center> 表1 比较Bootstrap 类 </center>

| 类别               | Bootstrap            | ServerBootstrap  |
| ------------------ | -------------------- | ---------------- |
| 网络编程中的作用   | 连接到远程主机和端口 | 绑定一个本地端口 |
| EventLoopGroup数目 | 1                    | 2                |

这两种类型的引导类之间的第一个区别已经讨论过了：ServerBootstrap 将绑定到一个端口，因为服务器必须要监听连接，而Bootstrap 则是由想要连接到远程节点的客户端应用程序所使用的。

第二个区别可能更加明显。引导一个客户端只需要一个EventLoopGroup，但是一个ServerBootstrap 则需要两个（也可以是同一个实例）。为什么呢？

因为服务器需要两组不同的Channel。第一组将只包含一个ServerChannel，代表服务器自身的已绑定到某个本地端口的正在监听的套接字。而第二组将包含所有已创建的用来处理传入客户端连接（对于每个服务器已经接受的连接都有一个）的Channel。图3-4 说明了这个模型，并且展示了为何需要两个不同的EventLoopGroup。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230606200534.png)

<center> 图4  具有两个EventLoopGroup的服务器 </center>

与ServerChannel 相关联的EventLoopGroup 将分配一个负责为传入连接请求创建Channel 的EventLoop。一旦连接被接受，第二个EventLoopGroup 就会给它的Channel分配一个EventLoop。

## 总结

本章中，从技术与体系上面介绍了Netty的重要性。