---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## Netty客户端/服务器概览

图1从高层次上展示了一个你将要编写的 Echo 客户端和服务器应用程序。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230606170107.png)

<center> 图1 Echo客户端与服务端 </center>

虽然我们已经谈及到了客户端，但是该图展示的是多个客户端同时连接到一台服务器。所能够支持的客户端数量，在理论上，仅受限于系统的可用资源（以及所使用的JDK 版本可能会施
加的限制）。

Echo 客户端和服务器之间的交互是非常简单的；在客户端建立一个连接之后，它会向服务器发送一个或多个消息，反过来，服务器又会将每个消息回送给客户端。虽然它本身看起来好像
用处不大，但它充分地体现了客户端/服务器系统中典型的请求-响应交互模式。

## 编写Echo服务器

所有的Netty 服务器都需要以下两部分。

- 至少一个ChannelHandler—该组件实现了服务器对从客户端接收的数据的处理，即它的业务逻辑。
- 引导—这是配置服务器的启动代码。至少，它会将服务器绑定到它要监听连接请求的端口上。

### ChannelHandler和业务逻辑

因为你的Echo 服务器会响应传入的消息，所以它需要实现ChannelInboundHandler 接口，用来定义响应入站事件的方法。这个简单的应用程序只需要用到少量的这些方法，所以继承ChannelInboundHandlerAdapter 类也就足够了，它提供了ChannelInboundHandler 的默认实现。

该Echo 服务器的ChannelHandler 实现是EchoServerHandler，如代码清单1所示：

```java
//标示一个ChannelHandler 可以被多个Channel安全地共享
@ChannelHandler.Sharable
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf in = (ByteBuf) msg;
        // 将消息记录到控制台
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        // 将接收到的消息写给发送者，而不冲刷出站消息
        ctx.write(in);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        //打印异常栈跟踪
        cause.printStackTrace();
        //关闭该Channel
        ctx.close();
    }
}
```

ChannelInboundHandlerAdapter 有一个直观的API，并且它的每个方法都可以被重写以挂钩到事件生命周期的恰当点上。因为需要处理所有接收到的数据，所以你重写了channelRead()
方法。在这个服务器应用程序中，你将数据简单地回送给了远程节点。

重写exceptionCaught()方法允许你对Throwable 的任何子类型做出反应，在这里你记录了异常并关闭了连接。虽然一个更加完善的应用程序也许会尝试从异常中恢复，但在这个场景下，只是通过简单地关闭连接来通知远程节点发生了错误。

> 如果不捕获异常，将会发生什么
>
> 每个Channel 都拥有一个与之相关联的ChannelPipeline，其持有一个ChannelHandler 的实例链。在默认的情况下，ChannelHandler 会把对它的方法的调用转发给链中的下一个ChannelHandler。因此，如果exceptionCaught()方法没有被该链中的某处实现，那么所接收的异常将会被传递到ChannelPipeline 的尾端并被记录。为此，你的应用程序应该提供至少有一个实现了exceptionCaught()方法的ChannelHandler。

除了ChannelInboundHandlerAdapter 之外，还有很多需要学习的ChannelHandler 的子类型和实现，下面是一些ChannelHandler的关键点：

- 针对不同类型的事件来调用ChannelHandler；
- 应用程序通过实现或者扩展ChannelHandler 来挂钩到事件的生命周期，并且提供自定义的应用程序逻辑；
- 在架构上，ChannelHandler 有助于保持业务逻辑与网络处理代码的分离。这简化了开发过程，因为代码必须不断地演化以响应不断变化的需