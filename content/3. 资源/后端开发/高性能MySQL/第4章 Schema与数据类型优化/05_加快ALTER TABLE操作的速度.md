---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
MySQL的ALTER TABLE操作的性能对大表来说是个大问题。MySQL执行大部分修改表结构操作的方法是用新的结构创建一个空表，从旧表中查出所有数据插入新表，然后删除旧表。这样操作可能需要花费很长时间，如果内存不足而表又很大，而且还有很多索引的情况下尤其如此。许多人都有这样的经验，ALTER TABLE操作需要花费数个小时甚至数天才能完成。

MySQL 5.1以及更新版本包含一些类型的“在线”操作的支持，这些功能不需要在整个操作过程中锁表。最近版本的InnoDB也支持通过排序来建索引，这使得建索引更快并且有一个紧凑的索引布局。

一般而言，大部分ALTER TABLE操作将导致MySQL服务中断。我们会展示一些在DDL操作时有用的技巧，但这是针对一些特殊的场景而言的。对常见的场景，能使用的技巧只有两种：一种是先在一台不提供服务的机器上执行ALTER TABLE操作，然后和提供服务的主库进行切换；另外一种技巧是“影子拷贝”。影子拷贝的技巧是用要求的表结构创建一张和源表无关的新表，然后通过重命名和删表操作交换两张表。也有一些工具可以帮助完成影子拷贝工作：例如，Facebook数据库运维团队（*https://launchpad.net/mysqlatfacebook*）的“online schema change”工具、Shlomi Noach的openark toolkit（*http://code.openark.org/*），以及Percona Toolkit（*http://www.percona.com/software/*）。如果使用Flexviews（参考4.4.1节），也可以通过其CDC工具执行无锁的表结构变更。

不是所有的ALTER TABLE操作都会引起表重建。例如，有两种方法可以改变或者删除一个列的默认值（一种方法很快，另外一种则很慢）。假如要修改电影的默认租赁期限，从三天改到五天。下面是很慢的方式：

```sql
  mysql> ALTER TABLE sakila.film
        -> MODIFY COLUMN rental_duration TINYINT(3) NOT NULL DEFAULT 5;
```

SHOW STATUS显示这个语句做了1000次读和1000次插入操作。换句话说，它拷贝了整张表到一张新表，甚至列的类型、大小和可否为NULL属性都没改变。

理论上，MySQL可以跳过创建新表的步骤。列的默认值实际上存在表的*.frm*文件中，所以可以直接修改这个文件而不需要改动表本身。然而MySQL还没有采用这种优化的方法，所有的MODIFY COLUMN操作都将导致表重建。

另外一种方法是通过ALTER COLUMN操作来改变列的默认值：

```sql
mysql> ALTER TABLE sakila.film
        -> ALTER COLUMN rental_duration SET DEFAULT 5;
```

这个语句会直接修改*.frm*文件而不涉及表数据。所以，这个操作是非常快的。

## 只修改.frm文件

从上面的例子我们看到修改表的*.frm*文件是很快的，但MySQL有时候会在没有必要的时候也重建表。如果愿意冒一些风险，可以让MySQL做一些其他类型的修改而不用重建表。

> 我们下面要演示的技巧是不受官方支持的，也没有文档记录，并且也可能不能正常工作，采用这些技术需要自己承担风险。建议在执行之前首先备份数据！

下面这些操作是有可能不需要重建表的：

- 移除（不是增加）一个列的AUTO_INCREMENT属性。
- 增加、移除，或更改ENUM和SET常量。如果移除的是已经有行数据用到其值的常量，查询将会返回一个空字串值。

基本的技术是为想要的表结构创建一个新的*.frm*文件，然后用它替换掉已经存在的那张表的*.frm*文件，像下面这样：

1. 创建一张有相同结构的空表，并进行所需要的修改（例如增加ENUM常量）。
2. 执行FLUSH TABLES WITH READ LOCK。这将会关闭所有正在使用的表，并且禁止任何表被打开。
3. 交换*.frm*文件.
4. 执行UNLOCK TABLES来释放第2步的读锁。

下面以给sakila.flm表的rating列增加一个常量为例来说明。当前列看起来如下：

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/20230512154547.png)

假设我们需要为那些对电影更加谨慎的父母们增加一个PG-14的电影分级：

```sql
mysql> CREATE TABLE sakila.film_new LIKE sakila.film;
mysql> ALTER TABLE sakila.film_new
  -> MODIFY COLUMN rating ENUM('G','PG','PG-13','R','NC-17', 'PG-14')
        -> DEFAULT 'G';
    mysql> FLUSH TABLES WITH READ LOCK;
```

注意，我们是在常量列表的末尾增加一个新的值。如果把新增的值放在中间，例如PG-13之后，则会导致已经存在的数据的含义被改变：已经存在的R值将变成PG-14，而已经存在的NC-17将成为R，等等。

接下来用操作系统的命令交换*.frm*文件：

```sql
    /var/lib/mysql/sakila# mv film.frm film_tmp.frm
    /var/lib/mysql/sakila# mv film_new.frm film.frm
    /var/lib/mysql/sakila# mv film_tmp.frm film_new.frm
```

再回到MySQL命令行，现在可以解锁表并且看到变更后的效果了：

```sql
    mysql> UNLOCK TABLES;
    mysql> SHOW COLUMNS FROM sakila.film LIKE 'rating'\G
    *************************** 1. row ***************************
    Field: rating
    Type: enum('G','PG','PG-13','R','NC-17','PG-14')
```

最后需要做的是删除为完成这个操作而创建的辅助表：


```sql
mysql> DROP TABLE sakila.film_new;
```

## 快速创建MyISAM索引

为了高效地载入数据到MyISAM表中，有一个常用的技巧是先禁用索引、载入数据，然后重新启用索引：

```sql
mysql> ALTER TABLE test.load_data ENABLE KEYS;
    -- load the data
mysql> ALTER TABLE test.load_data ENABLE KEYS;
```

这个技巧能够发挥作用，是因为构建索引的工作被延迟到数据完全载入以后，这个时候已经可以通过排序来构建索引了。这样做会快很多，并且使得索引树[(17)](javascript:void(0))的碎片更少、更紧凑。

不幸的是，这个办法对唯一索引无效，因为DISABLE KEYS只对非唯一索引有效。MyISAM会在内存中构造唯一索引，并且为载入的每一行检查唯一性。一旦索引的大小超过了有效内存大小，载入操作就会变得越来越慢。

在现代版本的InnoDB版本中，有一个类似的技巧，这依赖于InnoDB的快速在线索引创建功能。这个技巧是，先删除所有的非唯一索引，然后增加新的列，最后重新创建删除掉的索引。Percona Server可以自动完成这些操作步骤。

也可以使用像前面说的ALTER TABLE的骇客方法来加速这个操作，但需要多做一些工作并且承担一定的风险。这对从备份中载入数据是很有用的，例如，当已经知道所有数据都是有效的并且没有必要做唯一性检查时就可以这么来操作。

> 再次说明，这是没有文档说明并且不受官方支持的技巧。若使用的话，需要自己承担风险，并且操作之前一定要先备份数据。

下面是操作步骤：

1. 用需要的表结构创建一张表，但是不包括索引。
2. 载入数据到表中以构建*.MYD*文件。
3. 按照需要的结构创建另外一张空表，这次要包含索引。这会创建需要的*.frm*和*.MYI*文件。
4. 获取读锁并刷新表。
5. 重命名第二张表的*.frm*和*.MYI*文件，让MySQL认为是第一张表的文件。
6. 释放读锁。
7. 使用REPAIR TABLE来重建表的索引。该操作会通过排序来构建所有索引，包括唯一索引。

这个操作步骤对大表来说会快很多。