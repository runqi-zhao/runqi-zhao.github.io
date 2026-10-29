---
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/CPUCache%E6%8F%90%E7%BA%B2.png)

## CPU Cache 有多快？

你可能会好奇为什么有了内存，还需要 CPU Cache？根据摩尔定律，CPU 的访问速度每 18 个月就会翻倍，相当于每年增长 60% 左右，内存的速度当然也会不断增长，但是增长的速度远小于 CPU，平均每年只增长 7% 左右。于是，CPU 与内存的访问性能的差距不断拉大。

因此，一次内存访问所需时间时`200-300`多个始终周期，意味着CPU和内存的访问速度已经相差`200~300`倍了。

为了弥补这个差距，CPU内部引入了CPU Cache，也称为告诉缓存。

CPU Cache 通常分为大小不等的三级缓存，分别是 **L1 Cache、L2 Cache 和 L3 Cache**。

**L1 Cache 通常会分为「数据缓存」和「指令缓存」**。L1 Cache和L2 Cache都是每个CPU核心独有的，而L3 Cache时多个CPU核心共享的。

然后的话一般咱们这些内容的内存都是很小的，结构如下图所示：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/CPU-Cache.png)

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E8%AE%BF%E9%97%AE%E9%80%9F%E5%BA%A6%E8%A1%A8%E6%A0%BC.png)

## CPU Cache 的数据结构和读取过程是什么样的？

CPU Cache 是由很多个 Cache Line 组成的，

CPU Line是CPU从内存读取数据的基本单位i，而CPU Line是由各种标志（Tag）+数据块（Data Block）组成。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/Cache%25E7%259A%2584%25E6%2595%25B0%25E6%258D%25AE%25E7%25BB%2593%25E6%259E%2584.png)

CPU Cache 的数据是从内存中读取过来的，它是以一小块一小块读取数据的，而不是按照单个数组元素来读取数据的，在 CPU Cache 中的，这样一小块一小块的数据，称为 **Cache Line（缓存块）**。

Linux下可以使用如下命令查看Cache Line的大小：

```shell
cat /sys/devices/system/cpu/cpu0/cache/index0/coherency_line_size
```

**L1 Cache 一次载入数据的大小是 64 字节**。

这样子咱们再加载数据的时候，相当于一次加载对应64字节的内容。提高对应的效率。

然后在加载对应的数据的时候，咱们可以类比对应应用程序访问数据块时的Redis缓存，而这对应的关系如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/CPU%E7%BC%93%E5%AD%98/%E7%BC%93%E5%AD%98%E9%80%BB%E8%BE%91.png)

那 CPU 怎么知道要访问的内存数据，是否在 Cache 里？如果在的话，如何找到 Cache 对应的数据呢？

采用的是**直接映射：**我们提到 CPU 访问内存数据时，是一小块一小块数据读取的，具体这一小块数据的大小，取决于 `coherency_line_size` 的值，一般 64 字节。在内存中，这一块的数据我们称为**内存块（\*Block\*）**，读取的时候我们要拿到数据所在内存块的地址。

对于直接映射 Cache 采用的策略，就是把内存块的地址始终「映射」在一个 CPU Cache Line（缓存块） 的地址，至于映射关系实现方式，则是使用「取模运算」，取模运算的结果就是内存块地址对应的 CPU Cache Line（缓存块） 的地址。

举个例子，内存共被划分为 32 个内存块，CPU Cache 共有 8 个 CPU Cache Line，假设 CPU 想要访问第 15 号内存块，如果 15 号内存块中的数据已经缓存在 CPU Cache Line 中的话，则是一定映射在 7 号 CPU Cache Line 中，因为 `15 % 8` 的值是 7。

这个时候肯定会有多个指向一个CPU Cache Line。因此，为了区别不同的内存块，在对应的 CPU Cache Line 中我们还会存储一个**组标记（Tag）**。这个组标记会记录当前 CPU Cache Line 中存储的数据对应的内存块，我们可以用这个组标记来区分不同的内存块。

除了组标记信息外，CPU Cache Line 还有两个信息：

- 一个是，从内存加载过来的实际存放**数据（\*Data\*）**。
- 另一个是，**有效位（\*Valid bit\*）**，它是用来标记对应的 CPU Cache Line 中的数据是否是有效的，如果有效位是 0，无论 CPU Cache Line 中是否有数据，CPU 都会直接访问内存，重新加载数据。

CPU 在从 CPU Cache 读取数据的时候，并不是读取 CPU Cache Line 中的整个数据块，而是读取 CPU 所需要的一个数据片段，这样的数据统称为一个**字（\*Word\*）**。那怎么在对应的 CPU Cache Line 中数据块中找到所需的字呢？答案是，需要一个**偏移量（Offset）**。

因此，一个内存的访问地址，包括**组标记、CPU Cache Line 索引、偏移量**这三种信息，于是 CPU 就能通过这些信息，在 CPU Cache 中找到缓存的数据。而对于 CPU Cache 里的数据结构，则是由**索引 + 有效位 + 组标记 + 数据块**组成。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/%25E7%259B%25B4%25E6%258E%25A5Cache%25E6%2598%25A0%25E5%25B0%2584.png)

如果内存中的数据已经在 CPU Cache 中了，那 CPU 访问一个内存地址的时候，会经历这 4 个步骤：

1. 根据内存地址中索引信息，计算在 CPU Cache 中的索引，也就是找出对应的 CPU Cache Line 的地址；
2. 找到对应 CPU Cache Line 后，判断 CPU Cache Line 中的有效位，确认 CPU Cache Line 中数据是否是有效的，如果是无效的，CPU 就会直接访问内存，并重新加载数据，如果数据有效，则往下执行；
3. 对比内存地址中组标记和 CPU Cache Line 中的组标记，确认 CPU Cache Line 中的数据是我们要访问的内存数据，如果不是的话，CPU 就会直接访问内存，并重新加载数据，如果是的话，则往下执行；
4. 根据内存地址中偏移量信息，从 CPU Cache Line 的数据块中，读取对应的字。

## 如何写出让CPU跑的更快的代码

利用缓存嘛，相当于咱们写代码的时候尽量使用的数据都是相连的，能够直接访问对应CPU cache。这里的胡啊相当于提升数据缓存的命中率与指令缓存的命中率。

### 如何提高数据缓存的命中率

**遇到这种遍历数组的情况时，按照内存布局顺序访问，将可以有效的利用 CPU Cache 带来的好处，这样我们代码的性能就会得到很大的提升**

### 如何提升指令缓存的命中率

**如果分支预测可以预测到接下来要执行 if 里的指令，还是 else 指令的话，就可以「提前」把这些指令放在指令缓存中，这样 CPU 可以直接从 Cache 读取到指令，于是执行速度就会很快**。

因此，在使用指令的时候，应该尽量使用提前放入的指令，加快对应的访问速度。

## 如何提升多核CPU的缓存命中率

现在一个线程在不同核心来回切换，各个核心的缓存命中率会受到影响，因此咱们再进行执行的时候，应该尽量避免线程之间的切换。

因此这个时候咱们就应该根据实际情况使用对应的调度算法。**线程绑定在某一个 CPU 核心上**

## 总结

- 对于数据缓存，我们在遍历数据的时候，应该按照内存布局的顺序操作，这是因为 CPU Cache 是根据 CPU Cache Line 批量操作数据的，所以顺序地操作连续内存数据时，性能能得到有效的提升；
- 对于指令缓存，有规律的条件分支语句能够让 CPU 的分支预测器发挥作用，进一步提高执行的效率；