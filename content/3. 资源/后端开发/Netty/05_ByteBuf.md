---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## ByteBuffer的API

Netty 的数据处理API 通过两个组件暴露——abstract class ByteBuf 和interface ByteBufHolder。

ByteBuf API的优点：

- 它可以被用户自定义的缓冲区类型扩展； 
- 通过内置的复合缓冲区类型实现了透明的零拷贝； 
- 容量可以按需增长（类似于JDK 的StringBuilder）； 
- 在读和写这两种模式之间切换不需要调用ByteBuffer 的flip()方法； 
- 读和写使用了不同的索引； 
- 支持方法的链式调用；
- 支持引用计数
- 支持池化

## ByteBuf类-Netty的数据容器

### 如何进行工作

维护两个不同的索引：一个用于读取，另一个用于写入，当从ByteBuf进行读取时，它的readerIndex将会被递增已经到已经读取的字节数。写入时，writerIndex将会被递增已经写入的字节数。图1展示了ByteBuf的布局结构和状态。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230607104843.png)

<center> 图1 一个读索引与写索引都设置为0的16字节ByteBuf </center>

名称以read 或者write 开头的ByteBuf 方法，将会推进其对应的索引，而名称以set 或者get 开头的操作则不会。

### 在进行使用时有几种模式

#### 堆缓冲区

最常用的 ByteBuf 模式是将数据存储在 JVM 的堆空间中。这种模式被称为支撑数组（backing  array），它能在没有使用池化的情况下提供快速的分配和释放。

#### 直接缓冲区

直接缓冲区是另外一种ByteBuf 模式。我们期望用于对象创建的内存分配永远都来自于堆中，但这并不是必须的——NIO 在JDK 1.4 中引入的ByteBuffer 类允许JVM 实现通过本地调用来分配内存。这主要是为了避免在每次调用本地I/O 操作之前（或者之后）将缓冲区的内容复制到一个中间缓冲区（或者从中间缓冲区把内容复制到缓冲区）。

使用直接缓冲的缺点是：分配以及回收时代价比较昂贵。如果处理的是遗留代码，那么需要复制才能使用直接缓冲区。

#### 复合缓冲区

它为多个ByteBuf 提供一个聚合视图。在这里你可以根据需要添加或者删除ByteBuf 实例，这是一个JDK 的ByteBuffer 实现完全缺失的特性。

Netty 通过一个ByteBuf 子类——CompositeByteBuf——实现了这个模式，它提供了一个将多个缓冲区表示为单个合并缓冲区的虚拟表示。

> CompositeByteBuf 中的ByteBuf 实例可能同时包含直接内存分配和非直接内存分配。如果其中只有一个实例，那么对 CompositeByteBuf 上的 hasArray()方法的调用将返回该组
> 件上的hasArray()方法的值；否则它将返回false。

## 字节级操作

ByteBuf 提供了许多超出基本读、写操作的方法用于修改它的数据。

### 随机访问索引

如同在普通的Java 字节数组中一样，ByteBuf 的索引是从零开始的：第一个字节的索引是0，最后一个字节的索引总是capacity() - 1。

使用那些需要一个索引值参数的方法（的其中）之一来访问数据既不会改变readerIndex 也不会改变writerIndex。

### 顺序访问索引

虽然ByteBuf 同时具有读索引和写索引，但是JDK 的ByteBuffer 却只有一个索引，这也就是为什么必须调用 flip()方法来在读模式和写模式之间进行切换的原因。图2展示了ByteBuf 是如何被它的两个索引划分成3 个区域的。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230607105854.png)

<center> 图 2 ByteBuf 的内部分段 </center>

### 可丢弃字节

在图 2中标记为可丢弃字节的分段包含了已经被读过的字节。通过调用 discardReadBytes()方法，可以丢弃它们并回收空间。这个分段的初始大小为0，存储在readerIndex 中，会随着read 操作的执行而增加（get*操作不会移动readerIndex）。 

图3展示了图2中所展示的缓冲区上调用discardReadBytes()方法后的结果。可以看到，可丢弃字节分段中的空间已经变为可写的了。注意，在调用discardReadBytes()之后，对
可写分段的内容并没有任何的保证。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230607110138.png)

<center>图3 丢弃已读字节之后的ByteBuf</center>

虽然你可能会倾向于频繁地调用discardReadBytes()方法以确保可写分段的最大化，但是请注意，这将极有可能会导致内存复制，因为可读字节（图中标记为CONTENT 的部分）必须被移
动到缓冲区的开始位置。我们建议只在有真正需要的时候才这样做，例如，当内存非常宝贵的时候。

### 可读字节

ByteBuf 的可读字节分段存储了实际数据。新分配的、包装的或者复制的缓冲区的默认的readerIndex 值为0。任何名称以read 或者skip 开头的操作都将检索或者跳过位于当前readerIndex 的数据，并且将它增加已读字节数。

如果被调用的方法需要一个ByteBuf 参数作为写入的目标，并且没有指定目标索引参数，那么该目标缓冲区的writerIndex 也将被增加，例如：

```java
readByt es(ByteBuf dest);
```

如果尝试在缓冲区的可读字节数已经耗尽时从中读取数据，那么将会引发一个IndexOutOfBoundsException。

### 可写字节

可写字节分段是指一个拥有未定义内容的、写入就绪的内存区域。新分配的缓冲区的writerIndex 的默认值为0。任何名称以write 开头的操作都将从当前的writerIndex 处开始写数据，并将它增加已经写入的字节数。如果写操作的目标也是ByteBuf，并且没有指定源索引的值，则源缓冲区的readerIndex 也同样会被增加相同的大小。这个调用如下所示：

```java
writeBy tes(ByteBuf dest);
```

如果尝试往目标写入超过目标容量的数据，将会引发一个IndexOutOfBoundException。

### 索引管理

JDK 的InputStream 定义了mark(int readlimit)和reset()方法，这些方法分别被用来将流中的当前位置标记为指定的值，以及将流重置到该位置。

同样，可以通过调用markReaderIndex()、markWriterIndex()、resetWriterIndex()和resetReaderIndex()来标记和重置ByteBuf 的readerIndex 和writerIndex。这些和InputStream 上的调用类似，只是没有readlimit 参数来指定标记什么时候失效。

也可以通过调用readerIndex(int)或者writerIndex(int)来将索引移动到指定位置。试图将任何一个索引设置到一个无效的位置都将导致一个IndexOutOfBoundsException。

可以通过调用clear()方法来将readerIndex 和writerIndex 都设置为0。

### 查找索引

在ByteBuf中有多种可以用来确定指定值的索引的方法。最简单的是使用indexOf()方法。较复杂的查找可以通过那些需要一个ByteBufProcessor作为参数方法进行查找。

### 派生缓冲区

派生缓冲区为ByteBuf 提供了以专门的方式来呈现其内容的视图。这类视图是通过以下方法被创建的：

- duplicate()；
- slice()；
- slice(int, int)；
- Unpooled.unmodifiableBuffer(…)；
- order(ByteOrder)；
- readSlice(int)。

每个这些方法都将返回一个新的ByteBuf 实例，它具有自己的读索引、写索引和标记索引。其内部存储和JDK 的ByteBuffer 一样也是共享的。

### 读写操作

正如我们所提到过的，有两种类别的读/写操作：

- get()和set()操作，从给定的索引开始，并且保持索引不变；
- read()和write()操作，从给定的索引开始，并且会根据已经访问过的字节数对索引进行调整。

### 更多操作

看书

## ByteBufHolder接口

我们经常发现，除了实际的数据负载之外，我们还需要存储各种属性值。HTTP 响应便是一个很好的例子，除了表示为字节的内容，还包括状态码、cookie 等。

为了处理这种常见的用例，Netty 提供了ByteBufHolder。ByteBufHolder 也为Netty 的高级特性提供了支持，如缓冲区池化，其中可以从池中借用ByteBuf，并且在需要时自动释放。

## ByteBuf分配

### 按需分配：ByteBufAllocator接口

为了降低分配和释放内存的开销，Netty 通过interface ByteBufAllocator 实现了（ByteBuf 的）池化，它可以用来分配我们所描述过的任意类型的ByteBuf 实例。使用池化是特定于应用程序的决定，其并不会以任何方式改变ByteBuf API（的语义）。

### Unpooled缓冲区

可能某些情况下，你未能获取一个到ByteBufAllocator 的引用。对于这种情况，Netty 提供了一个简单的称为Unpooled 的工具类，它提供了静态的辅助方法来创建未池化的ByteBuf实例。

### ByteBufUtil类

ByteBufUtil 提供了用于操作ByteBuf 的静态的辅助方法。因为这个API 是通用的，并且和池化无关，所以这些方法已然在分配类的外部实现。

## 引用计数

引用计数是一种通过在某个对象所持有的资源不再被其他对象引用时释放该对象所持有的资源来优化内存使用和性能的技术。

引用计数背后的想法并不是特别的复杂；它主要涉及跟踪到某个特定对象的活动引用的数量。一个ReferenceCounted 实现的实例将通常以活动的引用计数为1 作为开始。只要引用计数大于0，就能保证对象不会被释放。当活动引用的数量减少到0 时，该实例就会被释放。注意，虽然释放的确切语义可能是特定于实现的，但是至少已经释放的对象应该不可再用了。

## 总结

本章专门探讨了Netty 的基于ByteBuf 的数据容器。我们首先解释了ByteBuf 相对于JDK所提供的实现的优势。我们还强调了该API 的其他可用变体，并且指出了它们各自最佳适用的特定用例。