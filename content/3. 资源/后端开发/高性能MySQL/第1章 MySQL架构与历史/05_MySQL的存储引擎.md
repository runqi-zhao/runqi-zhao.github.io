---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
在文件系统中，MySQL将每个数据库（也可以称之为schema）保存为数据目录下的一个子目录。创建表时，MySQL会在数据库子目录下创建一个和表同名的.frm文件保存表的定义。例如创建一个名为MyTable的表，MySQL会在MyTable.frm文件中保存该表的定义。因为MySQL使用文件系统的目录和文件来保存数据库和表的定义，大小写敏感性和具体的平台密切相关。

可以使用SHOW TABLE STATUS命令（在MySQL 5.0以后的版本中，也可以查询INFORMATION_SCHEMA中对应的表）显示表的相关信息。例如，对于mysql数据库中的user表：

```sql
   mysql> SHOW TABLE STATUS LIKE 'user' \G
    *************************** 1. row ***************************
               Name: user
          Engine: MyISAM
         Row_format: Dynamic
               Rows: 6
     Avg_row_length: 59
        Data_length: 356
    Max_data_length: 4294967295
       Index_length: 2048
          Data_free: 0
     Auto_increment: NULL
        Create_time: 2002-01-24 18:07:17
        Update_time: 2002-01-24 21:56:29
         Check_time: NULL
          Collation: utf8_bin
           Checksum: NULL
     Create_options:
            Comment: Users and global privileges
    1 row in set (0.00 sec)
```

Name：表名

Engine：表的存储引擎类型。

Row_format：行的格式。

Rows：表中的行数。

Avg_row_length：平均每行包含的字节数。

Data_length：表数据的大小（以字节为单位）。

Max_data_length：表数据的最大容量，该值和存储引擎有关。

Index_length：索引的大小（以字节为单位）。

Data_free：对于MyISAM表，表示已分配但目前没有使用的空间。这部分空间包括了之前删除的行，以及后续可以被INSERT利用到的空间。

Auto_increment：下一个AUTO_INCREMENT的值。

Create_time：表的创建时间。

Update_time：表数据的最后修改时间。

Check_time：使用CKECK TABLE命令或者myisamchk工具最后一次检查表的时间。

Collation：表的默认字符集和字符列排序规则。

Checksum：如果启用，保存的是整个表的实时校验和。

Create_options：创建表时指定的其他选项。

Comment：该列包含了一些其他的额外信息。对于MyISAM表，保存的是表在创建时带的注释。对于InnoDB表，则保存的是InnoDB表空间的剩余空间信息。如果是一个视图，则该列包含“VIEW”的文本字样。

## InnoDB存储引擎

设计成用来处理大量的短期（short-lived）事务，短期事务大部分情况都是正常提交的，很少会被回滚。

### InnoDB概况

InnoDB的数据库存储在表空间（tablespace）中，表空间是由InnoDB管理的黑盒子，有一系列的数据文件组成。

InnoDB采用MVCC来支持高并发，并且实现了四个标准的隔离级别。

默认级别是REPEATABLE READ（可重复读），并且通过间隙锁（next-key locking）策略防止幻读的出现。

InnoDB表是基于聚簇索引建立的。聚簇索引的二级索引（secondary index，非主键索引）中必须包含主键列，所以如果主键列很大的话，其他的所有索引都会很大。

InnoDB的存储个格式是平台独立的。

InnoDB通过一些机制和工具支持真正的热备份。

## MyISAM存储引擎

MyISAM不支持事务和行级锁，而且有一个毫无疑问的缺陷就是崩溃后无法安全恢复。

对于只读的数据，或者表比较小、可以忍受修复（repair）操作，则依然可以继续使用MyISAM（但请不要默认使用MyISAM，而是应当默认使用InnoDB）。

### 存储

MyISAM会将表存储在两个文件中：数据文件和索引文件，分别以.MYD和MYI为拓展名。

MyISAM表可以包含动态或者静态（长度固定）行。

MySQL会根据表的定义来决定采用何种行格式。MyISAM表可以存储的行记录数，一般受限于可用的磁盘空间，或者操作系统中单个文件的最大尺寸。

### MyISAM特性

#### 加锁与并发

对整张表进行加锁，而不是对某一行。

#### 修复

对于MyISAM表，MySQL可以手工或者自动执行检查和修复操作，但是这里的修复和事务修复以及崩溃修复是不同的概念。

#### 索引特征

对于MyISAM表，即使是BLOB和TEXT等长字段，也可以基于其前500个字符创建索引。MyISAM也支持全文索引，这是一种基于分词创建的索引，可以支持复杂的查询。

#### 延迟更新索引键（Delayed Key Write）

创建MyISAM表的时候，如果指定了DELAY_KEY_WRITE选项，在每次修改执行完成时，不会立刻将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区（in-memory key buffer），只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入到磁盘。

### MyISAM压缩表

如果表在创建并导入数据以后，不会再进行修改操作，那么这样的表或许适合采用MyISAM压缩表。

可以使用*myisampack*对MyISAM表进行压缩（也叫打包pack）。

### MyISAM性能

MyISAM最典型的性能问题还是表锁的问题，如果你发现所有的查询都长期处于“Locked”状态，那么毫无疑问表锁就是罪魁祸首。

## MySQL内奸的其他存储引擎

MySQL还有一些有特殊用途的存储引擎。在新版本中，有些可能因为一些原因已经不再支持；另外还有些会继续支持，但是需要明确地启用后才能使用。

### Archive引擎

Archive存储引擎只支持INSERT和SELECT操作。

Archive表适合日志和数据采集类应用。

Archive引擎支持行级锁和专用的缓冲区，所以可以实现高并发的插入。

### Blackhole引擎

Blackhole引擎没有实现任何的存储机制，它会丢弃所有插入的数据，不做任何保存。

可以用于复制数据到备库，或者只是简单地记录到日志。

### CSV引擎

CSV引擎可以将普通的CSV文件（逗号分割值的文件）作为MySQL的表来处理，但这种表不支持索引。

将Excel等电子表格软件中的数据存储为CSV文件，然后复制到MySQL数据目录下，就能在MySQL中打开使用。

### Federated引擎

Federated引擎是访问其他MySQL服务器的一个代理，它会创建一个到远程MySQL服务器的客户端连接，并将查询传输到远程服务器执行，然后提取或者发送需要的数据。

### Memory引擎

如果需要快速地访问数据，并且这些数据不会被修改，重启以后丢失也没有关系，那么使用Memory表（以前也叫做HEAP表）是非常有用的。

Memory表在很多场景可以发挥好的作用：

- 用于查找（lookup）或者映射（mapping）表，例如将邮编和州名映射的表。

- 用于缓存周期性聚合数据（periodically aggregated data）的结果。
- 用于保存数据分析中产生的中间数据。

Memory表支持Hash索引，因此查找操作非常快。虽然Memory表的速度非常快，但还是无法取代传统的基于磁盘的表。Memory表是表级锁，因此并发写入的性能较低。它不支持BLOB或TEXT类型的列，并且每行的长度是固定的，所以即使指定了VARCHAR列，实际存储时也会转换成CHAR，这可能导致部分内存的浪费（其中一些限制在Percona版本已经解决）。

如果MySQL在执行查询的过程中需要使用临时表来保存中间结果，内部使用的临时表就是Memory表。

如果中间结果太大超出了Memory表的限制，或者含有BLOB或TEXT字段，则临时表会转换成MyISAM表。

> 人们经常混淆Memory表和临时表。临时表是指使用CREATE TEMPORARY TABLE语句创建的表，它可以使用任何存储引擎，因此和Memory表不是一回事。临时表只在单个连接中可见，当连接断开时，临时表也将不复存在。

### Merge引擎

Merge引擎是MyISAM引擎的一个变种。Merge表是由多个MyISAM表合并而来的虚拟表。如果将MySQL用于日志或者数据仓库类应用，该引擎可以发挥作用。

引入分区之后，被放弃。

### NDB引擎

2003年，当时的MySQL AB公司从索尼爱立信公司收购了NDB数据库，然后开发了NDB集群存储引擎，作为SQL和NDB原生协议之间的接口。MySQL服务器、NDB集群存储引擎，以及分布式的、share-nothing的、容灾的、高可用的NDB数据库的组合，被称为MySQL集群（MySQL Cluster）。

## 第三方存储引擎

### OLTP类存储引擎

面向在线事务处理，沿用 InnoDB 的表格式和接口，可以直接替换使用。

Percona XtraDB 是代表，在高并发和可观测性上做了加强，配套的 XtraBackup 支持在线备份；MariaDB 早期也以 XtraDB 作为默认引擎。

### 面向列的存储引擎

按列存储、面向分析型查询，扫少量列时 I/O 少、压缩比高。写入和点查弱，一般不支持事务。

Infobright、InfiniDB 属于这一类，适合数据仓库和报表，不适合 OLTP。

### 社区存储引擎

社区维护的小众引擎，通常只解决一个特定问题。Aria 是 MyISAM 的改进版，补了崩溃恢复；Groonga 提供全文检索。

选用之前先确认维护状态和备份方案，出问题时没有官方的兜底。

### 选择合适的存储引擎

除非需要用到某些InnoDB不具备的特性，并且没有其他办法可以替代，否则都应该优先选择InnoDB引擎。

## 转换表的引擎

### ALTER TABLE

将表从一个引擎修改为另一个引擎最简单的办法是使用ALTER TABLE语句。下面的语句将mytable的引擎修改为InnoDB：

```sql
   mysql> ALTER TABLE mytable ENGINE=InnoDB;
```

上述语法可以适用任何存储引擎。但有一个问题：需要执行很长时间。MySQL会按行将数据从原表复制到一张新的表中，在复制期间可能会消耗系统所有的I/O能力，同时原表上会加上读锁。所以，在繁忙的表上执行此操作要特别小心。一个替代方案是采用接下来将讨论的导出与导入的方法，手工进行表的复制。

如果转换表的存储引擎，将会失去和原引擎相关的所有特性。例如，如果将一张InnoDB表转换为MyISAM，然后再转换回InnoDB，原InnoDB表上所有的外键将丢失。

### 导入与导出

为了更好地控制转换的过程，可以使用*mysqldump*工具将数据导出到文件，然后修改文件中CREATE TABLE语句的存储引擎选项，注意同时修改表名，因为同一个数据库中不能存在相同的表名，即使它们使用的是不同的存储引擎。同时要注意*mysqldump*默认会自动在CREATE TABLE语句前加上DROP TABLE语句，不注意这一点可能会导致数据丢失。

### 创建与查询（CREATE和SELECT）

第三种转换的技术综合了第一种方法的高效和第二种方法的安全。不需要导出整个表的数据，而是先创建一个新的存储引擎的表，然后利用INSERT…SELECT语法来导数据：

```sql
mysql> CREATE TABLE innodb_table LIKE myisam_table;
mysql> ALTER TABLE innodb_table ENGINE=InnoDB;
mysql> INSERT INTO innodb_table SELECT * FROM myisam_table;
```

数据量不大的话，这样做工作得很好。如果数据量很大，则可以考虑做分批处理，针对每一段数据执行事务提交操作，以避免大事务产生过多的undo。假设有主键字段id，重复运行以下语句（最小值x和最大值y进行相应的替换）将数据导入到新表：

```sql
 mysql> START TRANSACTION;
    mysql> INSERT INTO innodb_table SELECT * FROM myisam_table
        -> WHERE id BETWEEN x AND y;
    mysql> COMMIT;
```

这样操作完成以后，新表是原表的一个全量复制，原表还在，如果需要可以删除原表。如果有必要，可以在执行的过程中对原表加锁，以确保新表和原表的数据一致。

Percona Toolkit提供了一个*pt-online-schema-change*的工具（基于Facebook的在线schema变更技术），可以比较简单、方便地执行上述过程，避免手工操作可能导致的失误和烦琐。