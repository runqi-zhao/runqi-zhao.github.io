---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
什么场景下使用Netty？

当我们需要设计一个多并发用户，并且布恩那个有任何的性能损失。这时就需要使用Netty

请你说说Netty的定义？

Netty 是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端。

Netty的发展史简介一下（介绍一下Java网络编程）

## Java网络编程

那些最早期的Java API（java.net）只支持由本地系统套接字库提供的所谓的阻塞函数。代码清单1-1 展示了一个使用了这些函数调用的服务器代码的普通示例。

```java
//1. 创建一个新的ServerSocket，用以监听指定端口上的连接请求
ServerSocket serverSocket = new ServerSocket(portNumber);
//1. 对accept()方法的调用将被阻塞，直到一个连接建立
Socket clientSocket = serverSocket.accept();
BufferedReader in = new BufferedReader(
new InputStreamReader(clientSocket.getInputStream()));
//2. 这些流对象都派生于该套接字的流对象
PrintWriter out =
new PrintWriter(clientSocket.getOutputStream(), true);
String request, response;
//3. 处理循环开始
while ((request = in.readLine()) != null) {
//如果客户端发送了“Done”，则退出处理循环
if ("Done".equals(request)) {
break;
}
//4.请求被传递给服务器的处理方法
response = processRequest(request);
//服务器的响应被发送给了客户端
out.println(response);
//继续执行处理循环
}
```

要点如下：

- ServerSocket 上的accept()方法将会一直阻塞到一个连接建立1，随后返回一个新的Socket 用于客户端和服务器之间的通信。该ServerSocket 将继续监听传入的连接。
- BufferReader和PrintWriter都是衍生自Socket的输入输出流2。前者从一个字符读取文本，后者打印对象的格式化的表示大到文本输出流。
- readLine()方法将会阻塞，直到在处一个由换行符或者回车符结尾的字符串被读取。
- 客户端的请求已经被处理4。

上面袋中只是要给单线程的，即只能处理一个连接，要管理多个并发客户端，需要为每个新的客户端Socket 创建一个新的Thread。如图1所示：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230605161916.png)

<center> 图1 使用阻塞I/O 处理多个连接 </center>

缺点：

1. ，在任何时候都可能有大量的线程处于休眠状态，只是等待输入或者输出数据就绪，这可能算是一种资源浪费。
2. 需要为每个线程的调用栈都分配内存，其默认值大小区间为64 KB 到1 MB，具体取决于操作系统。
3. 即使Java 虚拟机（JVM）在物理上可以支持非常大数量的线程，但是远在到达该极限之前，上下文切换所带来的开销就会带来麻烦，例如，在达到10 000 个连接的时候。

### Java NIO

除了代码清单1 中代码底层的阻塞系统调用之外，本地套接字库很早就提供了非阻塞调用，其为网络资源的利用率提供了相当多的控制：

- 可以使用setsockopt()方法配置套接字，以便读/写调用在没有数据的时候立即返回，也就是说，如果是一个阻塞调用应该已经被阻塞了。
- 可以使用操作系统的事件通知API注册一组非阻塞套接字，以确定它们中是否有任何的套接字已经有数据可供读写。

### 选择器

图2展示了一个非阻塞设计，其实际上消除了上一节中所描述的那些弊端。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230605162345.png)

<center> 图2 使用Selector 的非阻塞I/O </center>

class java.nio.channels.Selector 是Java 的非阻塞I/O 实现的关键。它使用了事件通知API以确定在一组非阻塞套接字中有哪些已经就绪能够进行I/O 相关的操作。因为可以在任何的时间检查任意的读操作或者写操作的完成状态，所以如图2 所示，一个单一的线程便可以处理多个并发的连接。

总体来看，与阻塞I/O 模型相比，这种模型提供了更好的资源管理：

- 使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来开销；
- 当没有I/O 操作需要处理的时候，线程也可以被用于其他任务。

尽管已经有许多直接使用Java NIO API 的应用程序被构建了，但是要做到如此正确和安全并不容易。特别是，在高负载下可靠和高效地处理和调度I/O 操作是一项繁琐而且容易出错的任务，最好留给高性能的网络编程专家——Netty。

## Netty简介

请你总结一下Netty的特性

| 分类     | Netty的特性                                                  |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的API，支持多种传输类型，阻塞的和非阻塞的<br/>简单而强大的线程模型<br/>真正的无连接数据报套接字支持<br/>链接逻辑组件以支持复用 |
| 易于使用 | 详实的Javadoc和大量的示例集<br/>不需要超过JDK 1.6+③的依赖。（一些可选的特性可能需要Java 1.7+和/或额外的依赖）<br/>性能 拥有比Java 的核心API 更高的吞吐量以及更低的延迟<br/>得益于池化和复用，拥有更低的资源消耗<br/>最少的内存复制<br/> |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致OutOfMemoryError<br/>消除在高速网络中NIO 应用程序常见的不公平读/写比率<br/>安全性 完整的SSL/TLS 以及StartTLS 支持<br/>可用于受限环境下，如Applet 和OSGI<br/> |
| 社区驱动 | 发布快速而且频繁                                             |

### 异步和事件驱动

异步：你只有在已经问了一个问题之后才会得到一个和它对应的答案，而在你等待它的同时你也可以做点别的事情。

一个既是异步的又是事件驱动的系统会表现出一种特殊的、对我们来说极具价值的行为：它可以以任意的顺序响应在任意的时间点产生的事件。

这种能力对于实现最高级别的可伸缩性至关重要，定义为：“一种系统、网络或者进程在需要处理的工作不断增长时，可以通过某种可行的方式或者扩大它的处理能力来适应这种增长的能力。

异步和可伸缩性之间的联系又是什么呢？

- 非阻塞网络调用使得我们可以不必等待一个操作的完成。完全异步的I/O 正是基于这个特性构建的，并且更进一步：异步方法会立即返回，并且在它完成时，会直接或者在稍后的某个时间点通知用户。
- 选择器使得我们能够通过较少的线程便可监视许多连接上的事件。

### Netty的核心组件

- Channel；
- 回调；
- Future；
- 事件和ChannelHandler。

### Channel

定义是什么？

Channel 是Java NIO 的一个基本构造。

它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接，如读操作和写操作。

如何使用？

以把Channel 看作是传入（入站）或者传出（出站）数据的载体。因此，它可以
被打开或者被关闭，连接或者断开连接。

### 回调

定义是什么？

一个回调其实就是一个方法，一个指向已经被提供给另外一个方法的方法的引用。这使得后者可以在适当的时候调用前者。

Netty中是如何进行使用的？

Netty 在内部使用了回调来处理事件；当一个回调被触发时，相关的事件可以被一个interface-ChannelHandler 的实现处理。代码清单2 展示了一个例子：当一个新的连接已经被建立时，
ChannelHandler 的channelActive()回调方法将会被调用，并将打印出一条信息。

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx)
     //当一个新的连接已经被建立时，channelActive(ChannelHandler Context)将会被调用
    throws Exception {
    System.out.println(
    "Client " + ctx.channel().remoteAddress() + " connected");
    }
}
```

### Future

定义是什么？

Future 提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。

如何使用？

JDK 预置了interface java.util.concurrent.Future，但是其所提供的实现，只允许手动检查对应的操作是否已经完成，或者一直阻塞直到它完成。这是非常繁琐的，所以Netty提供了它自己的实现——ChannelFuture，用于在执行异步操作的时候使用。

请你说一下ChannelFuture的使用过程

ChannelFuture提供了几种额外的方法，这些方法使得我们能够注册一个或者多个ChannelFutureListener实例。监听器的回调方法operationComplete()，将会在对应的操作完成时被调用
。然后监听器可以判断该操作是成功地完成了还是出错了。如果是后者，我们可以检索产生的Throwable。简而言之，由ChannelFutureListener提供的通知机制消除了手动检查对应的操作是否完成的必要。

<font color ="red" size=16>每个Netty 的出站I/O 操作都将返回一个ChannelFuture；也就是说，它们都不会阻塞。正如我们前面所提到过的一样，Netty 完全是异步和事件驱动的。</font>

代码3展示了如何显式地利用ChannelFutureListener。

```java
Channel channel = ...;
    // Does not block
		//1. 异步地连接到远程节点
        ChannelFuture future = channel.connect(
        new InetSocketAddress("192.168.0.1", 25));
		//注册一个ChannelFutureListener，以便在操作完成时获得通知
        future.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                //1. 检查操作的状态
                if (future.isSuccess()){
                    //如果操作是成功的，则创建一个ByteBuf 以持有数据
                    ByteBuf buffer = Unpooled.copiedBuffer(
                    "Hello",Charset.defaultCharset());
                    //将数据异步地发送到远程节点。返回一个ChannelFuture
                    ChannelFuture wf = future.channel()
                    .writeAndFlush(buffer);
                    ....
                } else {
                    //如果发生错误，则访问描述原因的Throwable
                    Throwable cause = future.cause();
                    cause.printStackTrace();
        }
    }
});
```

首先，要连接到远程节点上。

然后，要注册一个新的ChannelFutureListener 到对connect()方法的调用所返回的ChannelFuture 上。当该监听器被通知连接已经建立的时候，要检查对应的状态1。

如果该操作是成功的，那么将数据写到该Channel。否则，要从ChannelFuture 中检索对应的Throwable。

需要注意的是，对错误的处理完全取决于你、目标，当然也包括目前任何对于特定类型的错误加以的限制。例如，如果连接失败，你可以尝试重新连接或者建立一个到另一个远程节点的连接。

如果你把ChannelFutureListener 看作是回调的一个更加精细的版本，那么你是对的。事实上，回调和Future 是相互补充的机制；它们相互结合，构成了Netty 本身的关键构件块之一。

### 事件和ChannelHandler

Netty 使用不同的事件来通知我们状态的改变或者是操作的状态。这使得我们能够基于已经发生的事件来触发适当的动作。这些动作可能是：

- 记录日志；
- 数据转换；
- 流控制；
- 应用程序逻辑。

Netty 是一个网络编程框架，所以事件是按照它们与入站或出站数据流的相关性进行分类的。可能由入站数据或者相关的状态更改而触发的事件包括：

- 连接已被激活或者连接失活；
- 数据读取；
- 用户事件；
- 错误事件。

出站事件是未来将会触发的某个动作的操作结果，这些动作包括：

- 打开或者关闭到远程节点的连接；
- 将数据写到或者冲刷到套接字。

每个事件都可以被分发给ChannelHandler 类中的某个用户实现的方法。这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子。图3 展示了一个事件是如何被一个这样的
ChannelHandler 链处理的。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230605164901.png)

<center> 图3 流经ChannelHandler 链的入站事件和出站事件 </center>

Netty 的ChannelHandler 为处理器提供了基本的抽象，如图3 所示的那些。我们会在适当的时候对ChannelHandler 进行更多的说明，但是目前你可以认为每个Channel-
Handler 的实例都类似于一种为了响应特定事件而被执行的回调。

Netty 提供了大量预定义的可以开箱即用的ChannelHandler 实现，包括用于各种协议（如HTTP 和SSL/TLS）的ChannelHandler。在内部，ChannelHandler 自己也使用了事件和Future，使得它们也成为了你的应用程序将使用的相同抽象的消费者。

### 把它们放在一起

#### Future、回调和ChannelHandler

Netty 的异步编程模型是建立在Future 和回调的概念之上的， 而将事件派发到ChannelHandler的方法则发生在更深的层次上。结合在一起，这些元素就提供了一个处理环境，使你的应用程序逻辑可以独立于任何网络操作相关的顾虑而独立地演变。这也是Netty 的设计方式的一个关键目标。

拦截操作以及高速地转换入站数据和出站数据，都只需要你提供回调或者利用操作所返回的Future。这使得链接操作变得既简单又高效，并且促进了可重用的通用代码的编写。

#### 选择器、事件和EventLoop

Netty 通过触发事件将Selector 从应用程序中抽象出来，消除了所有本来将需要手动编写的派发代码。在内部，将会为每个Channel 分配一个EventLoop，用以处理所有事件，包括：

- 注册感兴趣的事件；
- 将事件派发给ChannelHandler；
- 安排进一步的动作。

EventLoop 本身只由一个线程驱动，其处理了一个Channel 的所有I/O 事件，并且在该EventLoop 的整个生命周期内都不会改变。这个简单而强大的设计消除了你可能有的在ChannelHandler 实现中需要进行同步的任何顾虑，因此，你可以专注于提供正确的逻辑，用来在有感兴趣的数据要处理的时候执行。如同我们在详细探讨Netty 的线程模型时将会看到的，该API 是简单而紧凑的。

## 总结

在这一章中，我们介绍了Netty 框架的背景知识，包括Java 网络编程API 的演变过程，阻塞和非阻塞网络操作之间的区别，以及异步I/O 在高容量、高性能的网络编程中的优势。

然后，我们概述了Netty 的特性、设计和优点，其中包括Netty 异步模型的底层机制，包括回调、Future 以及它们的结合使用。我们还谈到了事件是如何产生的以及如何拦截和处理它们。