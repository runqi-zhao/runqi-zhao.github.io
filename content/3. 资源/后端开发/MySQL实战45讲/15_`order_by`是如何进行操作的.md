---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
首先抛出一个问题：order by在排序的时候，是如何将里面的内容进行排序的 

还是从一个例子说起，现在有一个市民表，想要查找城市是“杭州”的所有人的名字，并且按照姓名排序 返回前1000个人的姓名、年龄。

```sql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

这个时候，sql语句可以这么写

```sql
select city,name,age  from t where city = "杭州" order by name limit 1000;
```

这个语句的执行流程是什么，首先使用 `explain`解释这个sql语句。

![img](https://static001.geekbang.org/resource/image/82/03/826579b63225def812330ef6c344a303.png?wh=1470*126)

可以看到，里面的Extra字段中“Using filesort”表示的就是需要排序，MySQL会给 每个线程分配一块内存用于排序，称为sort_buffer。

下面看一个使用InnoDB的示意图。

![img](https://static001.geekbang.org/resource/image/53/3e/5334cca9118be14bde95ec94b02f0a3e.png?wh=1142*856)

可以看到，现在想要查找城市为  杭州，也就是ID.X-ID.（X+N）的字段，这个时候会做扫描。

通常情况下，这个语句执行流程如下所示:

1. 初始化sort_buffer，确定放入name、city、age这三个字段。
2. 从索引city找到第一个满足city='杭州'条件的主键id，也就是上图中ID.X。
3. 根据主键到主键索引取出整行，取name、city、age三个字段的值，存入sort_buffer中。
4. 从索引city取下一个记录的主键id。
5. 重复步骤3、4直到city的值不满足查询条件为止，对应图中主键id也就是 ID-Y。
6. 对sort_buffer中的数据按照字段name进行快速排序。
7. 按照排序结果取前1000行返回给客户端。

![img](https://static001.geekbang.org/resource/image/6c/72/6c821828cddf46670f9d56e126e3e772.jpg?wh=1142*856)

可以看到，上面步骤中，按照`name`进行排序这个动作，可能在内存中完成，也可能需要使用外部排序，这主要取决于内存和参数sort_buffer_size。

sort_buffer_size，就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。

MySQL 将需要排序的数据分成 12 份，每一份单独排序后存在这些临时文件中。然后把这 12 个有序文件再合并成一个有序的大文件。

> 这个其实就像是外排序。可以联系到在大内存中找top-k的问题。

如果 sort_buffer_size 超过了需要排序的数据量的大小，number_of_tmp_files 就是 0，表示排序可以直接在内存中完成。

否则就需要放在临时文件中排序。sort_buffer_size 越小，需要分成的份数越多，number_of_tmp_files 的值就越大。

sort_mode 里面的 packed_additional_fields 的意思是，排序过程对字符串做了“紧凑”处理。即使 name 字段的定义是 varchar(16)，在排序过程中还是要按照实际长度来分配空间的。

## rowid进行排序

如果说字段很短，即对应sort_buffer中字段很多，这样内存中可以放下的行数很少，需要分成多个临时文件，排序的性能就会很差。

这个时候，我们选择其中一个参数来进行排序。

使用到的语句如下：

```sql
SET max_length_for_sort_data = 16;
```

max_length_for_sort_data，是 MySQL 中专门控制用于排序的行数据的长度的一个参数。它的意思是，如果单行的长度超过这个值，MySQL 就认为单行太大，要换一个算法。

排序的结果就因为少了 city 和 age 字段的值，不能直接返回了，整个执行流程就变成如下所示的样子：

1. 初始化 sort_buffer，确定放入两个字段，即 name 和 id；
2. 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；
3. 到主键 id 索引取出整行，取 name、id 这两个字段，存入 sort_buffer 中；
4. 从索引 city 取下一个记录的主键 id；
5. 重复步骤 3、4 直到不满足 city='杭州’条件为止，也就是图中的 ID_Y；
6. 对 sort_buffer 中的数据按照字段 name 进行排序；
7. 遍历排序结果，取前 1000 行，并按照 id 的值回到原表中取出 city、name 和 age 三个字段返回给客户端。

这个执行流程的示意图如下，我把它称为 rowid 排序。

![img](https://static001.geekbang.org/resource/image/dc/6d/dc92b67721171206a302eb679c83e86d.jpg?wh=1142*856)

对比全字段排序流程图你会发现，rowid 排序多访问了一次表 t 的主键索引，就是步骤 7。

这个时候对应查找效率会进行下降，因为出现了步骤7，需要再次遍历对应的主键索引。examined_rows 的值还是 4000，表示用于排序的数据是 4000 行。但是 select @b-@a 这个语句的值变成 5000 了。

因为这时候除了排序过程外，在排序完成后，还要根据 id 去原表取值。由于语句是 limit 1000，因此会多读 1000 行。

sort_mode 变成了 ，表示参与排序的只有 name 和 id 这两个字段。

number_of_tmp_files 变成 10 了，是因为这时候参与排序的行数虽然仍然是 4000 行，但是每一行都变小了，因此需要排序的总数据量就变小了，需要的临时文件也相应地变少了。

## 二者进行对比

如果 MySQL 实在是担心排序内存太小，会影响排序效率，才会采用 rowid 排序算法，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。

如果内存够，就要多利用内存，尽量减少磁盘访问。

是不是所有的 order by 都需要排序操作呢？

不是，上面的操作都是在临时表上完成的，因为原来的数据都是无序的。

现在假设city这个索引取出来的话，name字段天然有序，这个时候，我们不用再进行排序。

这个时候，我们怎么实现对应的功能？

我们可以在这个市民表上创建一个 city 和 name 的联合索引，对应的 SQL 语句是：

```sql
alter table t add index city_user(city, name);
```

**TODO：搞懂 对应的 联合 索引**

作为与 city 索引的对比，我们来看看这个索引的示意图。

![img](https://static001.geekbang.org/resource/image/f9/bf/f980201372b676893647fb17fac4e2bf.png?wh=1142*856)

在这个索引里面，我们依然可以用树搜索的方式定位到第一个满足 city='杭州’的记录，并且额外确保了，接下来按顺序取“下一条记录”的遍历过程中，只要 city 的值是杭州，name 的值就一定是有序的。

查询流程变成了：

1. 从索引（city，name）找到第一个满足city=`杭州`条件的主键id
2. 到主键id索引取出整行，取name、city、age三个字段的值，作为结果集的一部分直接返回
3. 从索引（city，name）取下一条记录的主键id。
4. 重复步骤2、3，直到凑够1000条记录，或者是不满足city=`杭州`条件时循环结束。

![img](https://static001.geekbang.org/resource/image/3f/92/3f590c3a14f9236f2d8e1e2cb9686692.jpg?wh=1142*856)

这个查询过程不需要临时表，也不需要排序。使用explain进行查看：
![img](https://static001.geekbang.org/resource/image/fc/8a/fc53de303811ba3c46d344595743358a.png?wh=1382*127)

从图中可以看到，Extra 字段中没有 Using filesort 了，也就是不需要排序了。而且由于 (city,name) 这个联合索引本身有序，所以这个查询也不用把 4000 行全都读一遍，只要找到满足条件的前 1000 条记录就可以退出了。也就是说，在我们这个例子里，只需要扫描 1000 次。

如果现在使用覆盖索引进行优化。

覆盖索引是指，索引上的信息足够满足查询请求，不需要再回到主键索引上去取数据。

按照覆盖索引的概念，这个时候可以再优化一下这个查询语句的执行流程。

针对这个查询，我们可以创建一个city、name和age的联合索引，这个时候SQL语句就是：

```sql
alter table t add index city_user_age(city, name, age);
```

在这个时候查询流程是：

1. 首先从索引（city，name，age）找到第一个满足city=`杭州`条件的记录，取出其中city、name和age这三个字段的值，作为结果集的一部分直接返回
2. 从索引（city，name，age）取下一条记录，同样取出这三个字段的值，作为结果集的一部分直接返回。
3. 重复执行步骤2，直到凑够1000条记录，或者是不满足city=`杭州`条件时循环结束。

![img](https://static001.geekbang.org/resource/image/df/d6/df4b8e445a59c53df1f2e0f115f02cd6.jpg?wh=1142*856)

这个时候查看explain的结果。

![img](https://static001.geekbang.org/resource/image/9e/23/9e40b7b8f0e3f81126a9171cc22e3423.png?wh=1586*125)

可以看到，Extra 字段里面多了“Using index”，表示的就是使用了覆盖索引，性能上会快很多。

当然，这里并不是说要为了每个查询能用上覆盖索引，就要把语句中涉及的字段都建上联合索引，毕竟索引还是有维护代价的。这是一个需要权衡的决定。

## 总结

介绍了全字段排序、rowid排序，以及在数据本身有序时使用复合索引进行加速。

