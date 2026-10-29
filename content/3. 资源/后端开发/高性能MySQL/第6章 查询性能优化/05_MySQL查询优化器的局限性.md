---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
## 关联子查询

MySQL在进行关联子查询的步骤如下：

MySQL会将相关的外层表压到子查询中，认为可以更高效地找到数据行。

即MySQL会对表进行全表扫描，再根据返回的字段值逐个执行子查询。

优化方法：

- 一般会建议使用左外连接（LEFT OUTER JOIN）重写查询，以代替子查询。
- 使用函数GRPUP_CONCAT()在IN()中构造一个由逗号分隔地列表，优势查询会更快。

### 如何用好关联子查询

并不是所有的关联子查询都很差。

请用实际测试去说明其中地查询速度

## UNION的限制

如果希望UNION的各个子句能够根据LIMIT只取部分结果集，或者希望能够先排好序再合并结果集的话，就需要在UNION的各个子句中分别使用这些子句。

从临时表中取出数据的顺序并不是一定的，所以如果想获得正确的顺序，还需要加上一个全局的ORDER BY和LIMIT操作。

## 索引合并优化

当WHERE子句中包含多个复杂条件的时候，MySQL能够访问单个表的多个索引以合并和交叉过滤的方式来定位需要查找的行。

## 等值传递

某些时候，等值传递会带来一些意想不到的额外消耗。例如，有一个非常大的IN()列表，而MySQL优化器发现存在WHERE、ON或者USING的子句，将这个列表的值和另一个表的某个列相关联。

## 并行执行

MySQL无法利用多核特性来并行执行查询。很多其他的关系型数据库能够提供这个特性，但是MySQL做不到。这里特别指出是想告诉读者不要花时间去尝试寻找并行执行查询的方法。

## 哈希关联

MySQL并不支持哈希关联——MySQL的所有关联都是嵌套循环关联。不过，可以通过建立一个哈希索引来曲线地实现哈希关联。如果使用的是Memory存储引擎，则索引都是哈希索引，所以关联的时候也类似于哈希关联。

## 松散索引扫描

MySQL并不支持松散索引扫描，也就无法按照不连续的方式扫描一个索引。通常，MySQL的索引扫描需要先定义一个起点和终点，即使需要的数据只是这段索引中很少数的几个，MySQL仍需要扫描这段索引中每一个条目。

下面我们通过一个示例说明这点。假设我们有如下索引（a，b），有下面的查询：

```sql
    mysql> SELECT ... FROM tbl WHERE b BETWEEN 2 AND 3;
```

因为索引的前导字段是列a，但是在查询中只指定了字段b，MySQL无法使用这个索引，从而只能通过全表扫描找到匹配的行，如图所示。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230518161917.png)

<center>MySQL通过全表扫描找到需要的记录</center>

了解索引的物理结构的话，不难发现还可以有一个更快的办法执行上面的查询。索引的物理结构（不是存储引擎的API）使得可以先扫描a列第一个值对应的b列的范围，然后再跳到a列第二个不同值扫描对应的b列的范围。下图展示了如果由MySQL来实现这个过程会怎样。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230518161958.png)

<center>使用松散索引扫描效率会更高，但是MySQL现在还不支持这么做</center>

## 最大值和最小值

对于MIN()和MAX()查询，MySQL的优化做得并不好。这里有一个例子：

```
 mysql> SELECT MIN(actor_id) FROM sakila.actor WHERE first_name='PENELOPE';
```

因为在first_name字段上并没有索引，因此MySQL将会进行一次全表扫描。如果MySQL能够进行主键扫描，那么理论上，当MySQL读到第一个满足条件的记录的时候，就是我们需要找的最小值了，因为主键是严格按照actor_id字段的大小顺序排列的。但是MySQL这时只会做全表扫描，我们可以通过查看SHOW STATUS的全表扫描计数器来验证这一点。一个曲线的优化办法是移除MIN()，然后使用LIMIT来将查询重写如下：

```
    mysql> SELECT actor_id FROM sakila.actor USE INDEX(PRIMARY)
        -> WHERE first_name = 'PENELOPE' LIMIT 1;
```

这个策略可以让MySQL扫描尽可能少的记录数。如果你是一个完美主义者，可能会说这个SQL已经无法表达她的本意了。一般我们通过SQL告诉服务器我们需要什么数据，由服务器来决定如何最优地获取数据，不过在这个案例中，我们其实是告诉MySQL如何去获取我们需要的数据，通过SQL并不能一眼就看出我们其实是想要一个最小值。确实如此，有时候为了获得更高的性能，我们不得不放弃一些原则。

## 在同一表上查询和更新

MySQL不允许对同一张表同时进行查询和更新。这其实并不是优化器的限制，如果清楚MySQL是如何执行查询的，就可以避免这种情况。下面是一个无法运行的SQL，虽然这是一个符合标准的SQL语句。这个SQL语句尝试将两个表中相似行的数量记录到字段cnt中：

```sql
mysql> UPDATE tbl AS outer_tbl
    ->    SET cnt = (
    ->       SELECT count(*) FROM tbl AS inner_tbl
    ->       WHERE inner_tbl.type = outer_tbl.type
    ->    );
ERROR 1093 (HY000): You can't specify target table 'outer_tbl' for update in FROM
clause
```
可以通过使用生成表的形式来绕过上面的限制，因为MySQL只会把这个表当作一个临时表来处理。实际上，这执行了两个查询：一个是子查询中的SELECT语句，另一个是多表关联UPDATE，只是关联的表是一个临时表。子查询会在UPDATE语句打开表之前就完成，所以下面的查询将会正常执行：

```sql
 mysql> UPDATE tbl
  ->    INNER JOIN(www.it-eboo
        ->       SELECT type, count(*) AS cnt
        ->       FROM tbl
        ->       GROUP BY type
        ->    ) AS der USING(type)
        -> SET tbl.cnt = der.cnt;
```

