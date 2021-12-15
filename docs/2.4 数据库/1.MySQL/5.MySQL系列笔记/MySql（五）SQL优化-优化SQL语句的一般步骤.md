[TOC]

# 一、优化SQL语句的一般步骤
## 1.1 通过show status命令了解各种SQL的执行频率
Mysql客户端连接成功后，通过
```sql
show [session|global] status
```
命令可以提供服务器状态信息。`show [session|global] status`可以根据需要加上参数"session"或者"global"来显示seesion级（当前连接）的统计结果和global级（自数据库上次启动至今）的统计结果。如果不写，默认使用的参数时"session"。
例如：显示当前session中所有统计参数的值：
```sql
mysql> show status like 'Com_%';
```
Com_xxx表示每个xxx语句执行的次数。
 - Com_select：执行select操作的次数，一次查询之累加1.
 - Com_insert：执行insert操作的次数，对于批量插入insert操作，只累加一次。
 - Com_update：执行update操作的次数。
 - Com_delete：执行delete操作的次数。
上面这些参数对于所有存储引擎的表操作都会进行累计。下面这几个参数只是针对InnoDB存储引擎的，累加的算法也略有不同。
 - Innodb_rows_read：select查询返回的行数。
 - Innodb_rows_inserted：执行insert操作插入的行数。
 - Innodb_rows_updated：执行update操作更新的行数。
 - Innodb_rows_deleted：执行delete操作删除的行数。
通过以上参数，可以很容易了解到当前数据库的应用是以插入更新为主还是以查询操作为主，以及各种类型的SQK大致的执行比例是多少。
对于更新操作的计数，是对执行次数的计数，不论提交还是回滚都会进行累加。

对于事务型的应用，通过Com_commit和Com_rollback可以了解事物提交和回滚的情况，对于回滚操作频繁的数据库，可能意味着应用编写存在问题。

此外，以下几个参数便于用户了解数据库的基本情况：
 - Connections：试图连接Mysql服务器的次数。
 - Uptime：服务器工作时间。
 - Slow_queries：慢查询的次数。
## 1.2 定位执行效率较低的SQL语句
 - 通过慢查询日志定位那些执行效率较低的SQL语句,用-log-slow-queries[=file_name]选项启动时, mysqld写一个包含所有执行时间超过long_ query_ _time秒的SQL语句的日志文件。
 - 慢查询日志在查询结束以后才记录,所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看SQL的执行情况，同时对一些锁表操作进行优化。

## 1.3 通过explain分析低效sql的执行计划
通过以上步骤查询到效率低的SQL语句后，可以通过explain或者desc命令获取mysql如何执行select 语句的信息，包括在select语句执行过程中表如何连接和连接的顺序。
比如想统计某个email为租赁电影拷贝所支付的总金额，需要关联客户表customer和付款表payment，并且对付款金额amount字段做求和sum操作：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/c2278f4a69b76ae5c6ad558c3c95f018?showdoc=.jpg)

对每个列简单地进行一下说明。
 - select_type：表示select的类型，常见的取值有simple（简单表，即不使用表连接或者子查询）、Primary（主查询，即外层的查询）、UNION（UNION中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个SELECT）等。
 - table：输出结果集的表。
 - type：表示mysql在表中找到所需行的方式，或者叫访问类型。常见类型有：` all、index、range、ref、eq_ref、const,system、null`
 从左到右，性能由最差到最好。
1. type=all,全表扫描，mysql遍历全表来找到匹配的行：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/8531f1a04f70def98d9abc57c3e56f12?showdoc=.jpg)
2. type=index，索引全扫描，mysql遍历整个索引来查询匹配的行：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/d8cfebe5a7129b7b213c7df42ee614e6?showdoc=.jpg)
3. type=range，索引范围扫描，常见于<、<=、>、>=、between等操作符：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/07dfc1f457b07074322733f2f4790ea9?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/5cdfb1a0dc6c393d8928367850892591?showdoc=.jpg)
4. type=ref，使用非唯一索引扫描或唯一索引的前缀扫描，返回匹配某个单独值的记录行，例如：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/524ae1dfbaad25f57664430f48358254?showdoc=.jpg)
索引idx_fk_customer_id是非唯一索引，查询条件为等值查询条件customer_id=35，所以扫描索引的类型为ref。ref还经常出现在join操作中：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/60851deac070f774c1b6453cb00b96bc?showdoc=.jpg)
5. type=eq_ref，类似ref，区别就在使用的索引时唯一索引，对于每个索引键值，表中只有一条记录匹配。即多表连接中使用primary key或者unique index作为关联条件。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/b90c8907efc2760537d55efa8b67f0cf?showdoc=.jpg)
6. type=const/system，单表中最多有一个匹配行，查询起来非常迅速，所以这个匹配行中的其他列的值被优化器在当前查询中当作常量来处理，例如，根据主键primary key或唯一索引unique index进行的查询。
构造一个查询：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/5c67476c5ae91a49f120edf0eae4f23b?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/c2285b5839db492c08e0ad55f3c30696?showdoc=.jpg)
通过唯一索引uk_email访问的时候，类型为const，而从我们构造的仅有一条记录的a表中检索时，类型type就为system。
7. type=null,mysql不用访问表或者索引，直接就能得到结果，例如：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/3790eb321fd85439a57d06da02380376?showdoc=.jpg)

类型type还有其他值，如ref_or_null、index_merge（索引合并优化）、unique_subquery（in的后面是一个查询主键字段的子查询）、index_subquery（与unique_subquery类似，区别在于in的后面是查询非唯一索引字段的子查询）等。
  - possible_keys：表示查询时可能使用的索引。
  - key：表示实际使用的索引。
  - key_len：使用到索引字段的长度。
  - rows：扫描行的数量。
  - Extra：执行情况的说明和描述，包含不适合在其他列中显示但是对执行计划非常重要的额外信息。
## 1.4 通过show profile分析sql
通过have_profiling参数，能够看到当前mysql是否支持profile：
```sql
mysql> select @@hava_profiling;
```
默认profiling是关闭的，可通过set语句在Session级别开启profiling：
```sql
mysql> select @@profiling;

mysql> set profiling=1;
```
通过profile，能清楚地了解sql执行的过程。
在一个InnoDB引擎的付款表payment上，执行一个count(*)查询：
```sql
mysql> select count(*) from payment;
```
执行完毕后，通过show profiles语句，查看当前SQL的Query ID：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/eb5dfb37e4e69472194f4b41038bf3dc?showdoc=.jpg)

通过show profile for query语句能够看到执行过程中线程的每个状态和消耗的时间：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/a9a7dae8591d1155fc9a1dd216433535?showdoc=.jpg)

`Sending data状态表示mysql线程开始访问数据行并把结果返回给客户端，而不仅仅是返回结果给客户端。由于在Sending data状态下，mysql线程往往需要做大量的磁盘读取操作，所以经常是整个查询中最耗时的状态。`

在获取到最消耗时间的线程状态后，mysql支持进一步选择all、cpu、block io、context switch、page faults等明细类型来查mysql在使用什么资源上耗费了过高的时间。
例如：选择查询CPU的耗费时间：
```sql
mysql> show profile cpu for query 4;
```
Sending data状态下，时间消耗主要在CPU上了。

## 1.5 通过trace分析优化器如何选择执行计划
通过trace文件能够进一步了解为什么优化器选择A执行计划而不选择B执行计划。

使用方式：首先打开trace，设置格式为JSON，设置trace最大能够使用的内存大小，避免解析过程中因为默认内存过小而不能够完全显示。
```sql
mysql> set optimizer_trace="enabled=on",end_markers_in_json=on;

mysql> set optimizer_trace_max_size=1000000;
```
执行想做trace的SQL语句：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/732180b632b11575b8a419772624ad57?showdoc=.jpg)

最后检查Information_schema.optimizer_trace就可以知道mysql是如何执行SQL的：
```sql
mysql> select * from information_schema.optimizer_trace \G
```
## 1.6 确定问题并采取相应的优化措施