---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
实际使用的过程中，可能存在部分MySQL语句使用不当，导致整个书库压力变大 

## 案例一：条件字段函数操作

下面还是从一个例子说明：

```sql
mysql> CREATE TABLE `tradelog` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `operator` int(11) DEFAULT NULL,
  `t_modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`),
  KEY `t_modified` (`t_modified`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

假设现在这个 表上面已经有了从很多数据，现在存在一个需求，统计发生在所有年份中7月份的 交易的总数，这个时候 对应的SQL语句可以这么写：

```sql
mysql> select count(*) from tradelog where month(t_modified)=7;
```

本以为 ，在数据中 ，存在t_modified的语句，在进行 执行的时候直接使用 索引，不需要再进行全表扫描。

但是你在实际使用的时候 ，就算是记录很少，也需要经过 很长时间  才能 返回对应的 结果。

原因：如果对字段做了函数计算，就用不上索引了，这是 MySQL 的规定。

为什么？为什么条件是 where t_modified='2018-7-1’的时候可以用上索引，而改成 where month(t_modified)=7 的时候就不行了？

下面是这个 t_modified 索引的示意图。方框上面的数字就是 month() 函数对应的值。

![img](https://static001.geekbang.org/resource/image/3e/86/3e30d9a5e67f711f5af2e2599e800286.png?wh=1142*856)

如果你的 SQL 语句条件用的是 where t_modified='2018-7-1’的话，引擎就会按照上面绿色箭头的路线，快速定位到 t_modified='2018-7-1’需要的结果。

实际上，B+ 树提供的这个快速定位能力，来源于同一层兄弟节点的有序性。

但是，现在 他如果 使用month()函数的话，胡看到传入7的时候，在书的第一层就不知道该怎办了。

也就是说，对索引字段做函数操作，可能会破坏所引致的有序性，因此优化器巨顶放弃走树的搜索功能。

但是，存在一种情况，优化器并不是要放弃是使用索引。

在这个例子里，放弃了树搜索功能，优化器可以选择遍历主键索引，也可以选择遍历索引 t_modified，优化器对比索引大小后发现，索引 t_modified 更小，遍历这个索引比遍历主键索引来得更快。因此最终还是会选择索引 t_modified。

接下来，我们使用 explain 命令，查看一下这条 SQL 语句的执行结果。

![img](https://static001.geekbang.org/resource/image/27/55/27c2f5ff3549b18ba37a28f4919f3655.png?wh=1637*143)

key="t_modified"表示的是，使用了 t_modified 这个索引；我在测试表数据中插入了 10 万行数据，rows=100335，说明这条语句扫描了整个索引的所有值；Extra 字段的 Using index，表示的是使用了覆盖索引。

由于在 t_modified 字段加了 month() 函数操作，导致了全索引扫描。为了能够用上索引的快速定位能力，我们就要把 SQL 语句改成基于字段本身的范围查询。

还存在一个小问题，现在如果不改变“优化器”上面的内容，即使对于不改变有序性的函数，也不会考虑使用索引。对于select * from tradelog where id + 1 = 10000 这个 SQL 语句，这个加 1 操作并不会改变有序性，但是 MySQL 优化器还是不能用 id 索引快速定位到 9999 这一行。所以，需要你在写 SQL 语句的时候，手动改写成 where id = 10000 -1 才可以。

## 案例二：隐式类型转换

现在有下面这种案例：

```sql
mysql> select * from tradelog where tradeid=110717;
```

但是，我们看上面的表设计，tradeid是String字段。

这个时候，做的事全表扫描。

原因：tradeid是varchar类型，而输入的参数确实整形，所以需要做类型转换。

问题：

1. 数据类型转换的规则是什么？
2. 为什么有数据类型转换 ，就需要走全索引扫描？

字符串和数字做比较的话，是将字符串转换成数字。

那么，上面的语句对于优化器就变成了：

```sql
mysql> select * from tradelog where  CAST(tradid AS signed int) = 110717;
```

这条语句触发了我们上面说到的规则：对索引字段做函数操作，优化器会放弃走树搜索功能。

但是，当我们执行下面这条语句：

```sql
select * from tradelog where id="83126";
```

这个时候时可以使用索引扫描的。

## 案例三：隐式字符编码转换

还是从一个例子出发，现在我们系统之中存在 另一张表trade——detail，这表的作用是记录交易的操作细节。

```sql
mysql> CREATE TABLE `trade_detail` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `trade_step` int(11) DEFAULT NULL, /*操作步骤*/
  `step_info` varchar(32) DEFAULT NULL, /*步骤信息*/
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into tradelog values(1, 'aaaaaaaa', 1000, now());
insert into tradelog values(2, 'aaaaaaab', 1000, now());
insert into tradelog values(3, 'aaaaaaac', 1000, now());

insert into trade_detail values(1, 'aaaaaaaa', 1, 'add');
insert into trade_detail values(2, 'aaaaaaaa', 2, 'update');
insert into trade_detail values(3, 'aaaaaaaa', 3, 'commit');
insert into trade_detail values(4, 'aaaaaaab', 1, 'add');
insert into trade_detail values(5, 'aaaaaaab', 2, 'update');
insert into trade_detail values(6, 'aaaaaaab', 3, 'update again');
insert into trade_detail values(7, 'aaaaaaab', 4, 'commit');
insert into trade_detail values(8, 'aaaaaaac', 1, 'add');
insert into trade_detail values(9, 'aaaaaaac', 2, 'update');
insert into trade_detail values(10, 'aaaaaaac', 3, 'update again');
insert into trade_detail values(11, 'aaaaaaac', 4, 'commit');
```

这时候，如果要查询 id=2 的交易的所有操作步骤信息，SQL 语句可以这么写：

```sql
mysql> select d.* from tradelog l, trade_detail d where d.tradeid=l.tradeid and l.id=2; /*语句Q1*/
```

这个过程图下：

1. 第一行显示优化器会先在交易记录表 tradelog 上查到 id=2 的行，这个步骤用上了主键索引，rows=1 表示只扫描一行；
2. 第二行 key=NULL，表示没有用上交易详情表 trade_detail 上的 tradeid 索引，进行了全表扫描。

在这个执行计划里，是从 tradelog 表中取 tradeid 字段，再去 trade_detail 表里查询匹配字段。因此，我们把 tradelog 称为驱动表，把 trade_detail 称为被驱动表，把 tradeid 称为关联字段。

explain的执行流程如下：

![img](https://static001.geekbang.org/resource/image/82/a9/8289c184c8529acea0269a7460dc62a9.png?wh=1142*856)

- 第 1 步，是根据 id 在 tradelog 表里找到 L2 这一行；
- 第 2 步，是从 L2 中取出 tradeid 字段的值；
- 第 3 步，是根据 tradeid 值到 trade_detail 表中查找条件匹配的行。explain 的结果里面第二行的 key=NULL 表示的就是，这个过程是通过遍历主键索引的方式，一个一个地判断 tradeid 的值是否匹配。

进行到这里，你会发现第 3 步不符合我们的预期。因为表 trade_detail 里 tradeid 字段上是有索引的，我们本来是希望通过使用 tradeid 索引能够快速定位到等值的行。但，这里并没有。

原因：因为这两个表的字符集不同，一个是 utf8，一个是 utf8mb4，所以做表连接查询的时候用不上关联字段的索引。

为什么字符集不同 就用不上索引呢？

我们说问题是出在执行步骤的第 3 步，如果单独把这一步改成 SQL 语句的话，那就是：

```sql
mysql> select * from trade_detail where tradeid=$L2.tradeid.value; 
```

其中，$L2.tradeid.value 的字符集是 utf8mb4。

这里如果把utf8和utf8m64互换一下，就会用到tradeid索引了，这时应该时被驱动表需要进行字符集转换时才不会使用索引，而如果是驱动表进行字符集转换的话就会用到索引。

因此，实际上这个而语句等同于下面这个写法：

```sql
select * from trade_detail  where CONVERT(traideid USING utf8mb4)=$L2.tradeid.value; 
```

CONVERT() 函数，在这里的意思是把输入的字符串转成 utf8mb4 字符集。

这就再次触发了我们上面说到的原则：对索引字段做函数操作，优化器会放弃走树搜索功能。

连接过程中要求在被驱动表的索引字段上加函数操作，是直接导致对被驱动表做全表扫描的原因。

作为对比验证，我给你提另外一个需求，“查找 trade_detail 表里 id=4 的操作，对应的操作者是谁”，再来看下这个语句和它的执行计划。

```sql
mysql>select l.operator from tradelog l , trade_detail d where d.tradeid=l.tradeid and d.id=4;
```

![img](https://static001.geekbang.org/resource/image/92/11/92cb498ceb3557e41700fae53ce9bd11.png?wh=1328*194)

这个语句里 trade_detail 表成了驱动表，但是 explain 结果的第二行显示，这次的查询操作用上了被驱动表 tradelog 里的索引 (tradeid)，扫描行数是 1。

这也是两个 tradeid 字段的 join 操作，为什么这次能用上被驱动表的 tradeid 索引呢？

假设驱动表 trade_detail 里 id=4 的行记为 R4，那么在连接的时候（图 5 的第 3 步），被驱动表 tradelog 上执行的就是类似这样的 SQL 语句：

```sql
select operator from tradelog  where traideid =$R4.tradeid.value; 
```

这时候 $R4.tradeid.value 的字符集是 utf8, 按照字符集转换规则，要转成 utf8mb4，所以这个过程就被改写成：

```sql
select operator from tradelog  where traideid =CONVERT($R4.tradeid.value USING utf8mb4); 
```

这里的 CONVERT 函数是加在输入参数上的，这样就可以用上被驱动表的 traideid 索引。

解决办法：

- 比较常见的优化方法是，把 trade_detail 表上的 tradeid 字段的字符集也改成 utf8mb4，这样就没有字符集转换的问题了。
- 如果能够修改字段的字符集的话，是最好不过了。但如果数据量比较大， 或者业务上暂时不能做这个 DDL 的话，那就只能采用修改 SQL 语句的方法了。