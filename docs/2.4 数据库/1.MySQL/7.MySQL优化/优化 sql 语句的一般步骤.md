[TOC]

# 一、通过 show status 命令了解各种 sql 的执行频率
mysql 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息，也可以在操作系统上使用 mysqladmin extend-status 命令获取这些消息。 show status 命令中间可以加入选项 session（默认） 或 global：

- session （当前连接）
- global （自数据上次启动至今）

```sql
# Com_xxx 表示每个 xxx 语句执行的次数。
mysql> show status like 'Com_%';
```

**通常比较关心的是以下几个统计参数：**

- Com_select : 执行 select 操作的次数，一次查询只累加 1。
- Com_insert : 执行 insert 操作的次数，对于批量插入的 insert 操作，只累加一次。
- Com_update : 执行 update 操作的次数。
- Com_delete : 执行 delete 操作的次数。

上面这些参数对于所有存储引擎的表操作都会进行累计。下面这几个参数只是针对 innodb 的，累加的算法也略有不同：

- Innodb_rows_read : select 查询返回的行数。
- Innodb_rows_inserted : 执行 insert 操作插入的行数。
- Innodb_rows_updated : 执行 update 操作更新的行数。
- Innodb_rows_deleted : 执行 delete 操作删除的行数。

通过以上几个参数，可以很容易地了解当前数据库的应用是以插入更新为主还是以查询操作为主，以及各种类型的 sql 大致的执行比例是多少。对于更新操作的计数，是对执行次数的计数，不论提交还是回滚都会进行累加。

对于事务型的应用，通过 Com_commit 和 Com_rollback 可以了解事务提交和回滚的情况，对于回滚操作非常频繁的数据库，可能意味着应用编写存在问题。
此外，以下几个参数便于用户了解数据库的基本情况：

- Connections ： 试图连接 mysql 服务器的次数。
- Uptime ： 服务器工作时间。
- Slow_queries : 慢查询次数。

# 二、定义执行效率较低的 sql 语句
1. 通过慢查询日志定位那些执行效率较低的 sql 语句，用 --log-slow-queries[=file_name] 选项启动时，mysqld 写一个包含所有执行时间超过 long_query_time 秒的 sql 语句的日志文件。

2. 慢查询日志在查询结束以后才记录，所以在应用反映执行效率出现问题的时候慢查询日志并不能定位问题，可以使用 show processlist 命令查看当前 mysql 在进行的线程，包括线程的状态、是否锁表等，可以实时的查看 sql 的执行情况，同时对一些锁表操作进行优化。

# 三、通过 explain 分析低效 sql 的执行计划

# 四、通过 performance_schema 分析 sql 性能

# 五、通过 trace 分析优化器如何选择执行计划。
mysql5.6 提供了对 sql 的跟踪 trace，可以进一步了解为什么优化器选择 A 执行计划而不是 B 执行计划，帮助我们更好的理解优化器的行为。

使用方式：首先打开 trace ，设置格式为 json，设置 trace 最大能够使用的内存大小，避免解析过程中因为默认内存过小而不能够完整显示。