---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
是通过WAL机制，只要redo log 和binlog宝恒持久化到磁盘，就能确保MySQL一场重启后，数据可以恢复。

那么，redo log的写入是怎么样的，redo log又是如何确保真实地写入磁盘的。

## binlog的写入机制

binlog的写入逻辑比较简单：事务执行的过程中，先把日志笑道binlog cache，事务提交的时候，再把binlog cache写搭配binlog文件中。

binlog在进行写入的时候，不论这个事务多大，也要确保一次性写入。这个就涉及到了binlog cache问题。

这个的话有一个变量用来空值单个线程内binlog cache所占的内存大小。如果超过了这个参数规定的大小，就要暂存到磁盘。

事务提交的时候，执行器把 binlog cache 里的完整事务写入到 binlog 中，并清空 binlog cache。状态如图 1 所示。

![img](https://static001.geekbang.org/resource/image/9e/3e/9ed86644d5f39efb0efec595abb92e3e.png?wh=1142*748)

可以看到，每个线程有自己 binlog cache，但是共用同一份 binlog 文件。

- write，指的是吧日志写入到文件系统的page cache，并没有把数据持久化到磁盘，所以执行是速度比较快。
- fsync，指的是把数据持久化磁盘的操作，一般情况下，人这个fsync占用磁盘的IOPS。

write 和 fsync 的时机，是由参数 sync_binlog 控制的：

1. sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
2. sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
3. sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。

因此，在出现 IO 瓶颈的场景里，将 sync_binlog 设置成一个比较大的值，可以提升性能。在实际的业务场景中，考虑到丢失日志量的可控性，一般不建议将这个参数设成 0，比较常见的是将其设置为 100~1000 中的某个数值。

风险：如果主句发生异常重启，会丢失最近N个事物的binlog日志。

## redo log的写入机制

事务在执行过程中，生成的 redo log 是要先写到 redo log buffer 的。

redo log buffer里面的内容，不是每次生成后都要直接持久化磁盘中的。

如果事务执行期间MySQL发生异常重启，那么这部分日志就是丢失了。但是这个时候事务还没有提交，就算是丢失了也没有损失。

新的问题，事务没有进行提交的时候，redo log buffer中的部分日志有没有可能被持久化到磁盘呢？

答案：有可能。

这个问题，需要从redo log可能存在的三种状态说起。这三种状态，对应的就是下图中三种颜色块。

![img](https://static001.geekbang.org/resource/image/9d/d4/9d057f61d3962407f413deebc80526d4.png?wh=1142*639)

这三种状态分别是：

1. 存在 redo log buffer 中，物理上是在 MySQL 进程内存中，就是图中的红色部分；
2. 写到磁盘 (write)，但是没有持久化（fsync)，物理上是在文件系统的 page cache 里面，也就是图中的黄色部分；
3. 持久化到磁盘，对应的是 hard disk，也就是图中的绿色部分。

这个过程中，真正慢的是第三步，之所以说事务没有提交的时候，redo log buffer中的部分日志持久化到磁盘的原因是：

1. redo log buffer占用的空间即将达到innodb_log_buffer_size一半的时候，后台线程主动写盘。由于这个事务并没有提交，所以这个写盘动作只是 write，而没有调用 fsync，也就是只留在了文件系统的 page cache。
2. 并行的事务提交的时候，顺带将这个事物的redo log buffer 持久化到磁盘。假设一个事务 A 执行到一半，已经写了一些 redo log 到 buffer 中，这时候有另外一个线程的事务 B 提交，如果 innodb_flush_log_at_trx_commit 设置的是 1，那么按照这个参数的逻辑，事务 B 要把 redo log buffer 里的日志全部持久化到磁盘。这时候，就会带上事务 A 在 redo log buffer 里的日志一起持久化到磁盘。

中间的一大段没有看懂，大致就是说明为什么使用redo log可以节省时间。

如果你的 MySQL 现在出现了性能瓶颈，而且瓶颈在 IO 上，可以通过哪些方法来提升性能呢？

1. 设置 binlog_group_commit_sync_delay 和 binlog_group_commit_sync_no_delay_count 参数，减少 binlog 的写盘次数。这个方法是基于“额外的故意等待”来实现的，因此可能会增加语句的响应时间，但没有丢失数据的风险。
2. 将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000）。这样做的风险是，主机掉电时会丢 binlog 日志。
3. 将 innodb_flush_log_at_trx_commit 设置为 2。这样做的风险是，主机掉电的时候会丢数据。

