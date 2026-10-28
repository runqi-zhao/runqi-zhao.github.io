---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
面试的时候经常会问，现在快到双十一了，业务这边可能在进行抢购的时候需要大量的SQL进行并发，请问你如何在短期内临时性的提升一些性能。

这个问题相当于对SQL语句的优化方案 ，下面从几个 角度出发，分别说明其中的情况。

## 短链接风暴

同一时刻有大量的连接连接到 SQL语句当中，那么该怎么进行处理，才能够使得业务正常并发。

第一种方法：先处理掉那些占着连接但是不工作的线程。

max_connections 的计算，不是看谁在 running，是只要连着就占用一个计数位置。对于那些不需要保持的连接，我们可以通过 kill connection 主动踢掉。这个行为跟事先设置 wait_timeout 的效果是一样的。设置 wait_timeout 参数表示的是，一个线程空闲 wait_timeout 这么多秒之后，就会被 MySQL 直接断开连接。

但是需要注意，在 show processlist 的结果里，踢掉显示为 sleep 的线程，可能是有损的。我们来看下面这个例子。

![img](https://static001.geekbang.org/resource/image/90/2a/9091ff280592c8c68665771b1516c62a.png?wh=1246*234)

在上面这个例子里，如果断开 session A 的连接，因为这时候 session A 还没有提交，所以 MySQL 只能按照回滚事务来处理；而断开 session B 的连接，就没什么大影响。所以，如果按照优先级来说，你应该优先断开像 session B 这样的事务外空闲的连接。

但是，怎么判断哪些是事务外空闲的呢？session C 在 T 时刻之后的 30 秒执行 show processlist，看到的结果是这样的。

![img](https://static001.geekbang.org/resource/image/ae/25/ae6a9ceecf8517e47f9ebfc565f0f925.png?wh=1022*157)

图中 id=4 和 id=5 的两个会话都是 Sleep 状态。而要看事务具体状态的话，你可以查 information_schema 库的 innodb_trx 表。

![img](https://static001.geekbang.org/resource/image/ca/e8/ca4b455c8eacbf32b98d1fe9ed9876e8.png?wh=755*682)

这个结果里，trx_mysql_thread_id=4，表示 id=4 的线程还处在事务中。

因此，如果是连接数过多，你可以优先断开事务外空闲太久的连接；如果这样还不够，再考虑断开事务内空闲太久的连接。

从服务端断开连接使用的是 kill connection + id 的命令， 一个客户端处于 sleep 状态时，它的连接被服务端主动断开后，这个客户端并不会马上知道。直到客户端在发起下一个请求的时候，才会收到这样的报错“ERROR 2013 (HY000): Lost connection to MySQL server during query”。