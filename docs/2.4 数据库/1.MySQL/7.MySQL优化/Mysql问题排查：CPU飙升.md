[TOC]

# 1、问题排查-show full processlist
一般用到 show processlist 或 show full processlist 都是为了查看当前 mysql 是否有压力，都在跑什么语句，当前语句耗时多久了，有没有什么慢 SQL 正在执行之类的。

可以看到总共有多少链接数，哪些线程有问题(time是执行秒数，时间长的就应该多注意了)，然后可以把有问题的线程 kill 掉，这样可以临时解决一些突发性的问题。

show full processlist 可以看到所有链接的情况，但是大多链接的 state 其实是 Sleep 的，这种的其实是空闲状态，没有太多查看价值。
我们要观察的是有问题的，所以可以进行过滤：
```sql
-- 查询非 Sleep 状态的链接，按消耗时间倒序展示，自己加条件过滤
select id, db, user, host, command, time, state, info
from information_schema.processlist
where command != 'Sleep'
order by time desc;
```
过滤出来哪些是正在干活的，然后按照消耗时间倒叙展示，排在最前面的，极大可能就是有问题的链接了，然后查看 info 一列，就能看到具体执行的什么 SQL 语句了，针对分析：
![](http://on6gnkbff.bkt.clouddn.com/20170708035641_mysql-show-full-processlist-nonsleep.png)

展示列解释：
- id - 线程ID，可以用：kill id; 杀死一个线程，很有用
- db - 数据库
- user - 用户
- host - 连库的主机IP
- command - 当前执行的命令，比如最常见的：Sleep，Query，Connect 等
- time - 消耗时间，单位秒，很有用
- state - 执行状态，比如：Sending data，Sorting for group，Creating tmp table，Locked等等
- info - 执行的SQL语句，很有用

state列，mysql列出的状态主要有以下几种：
## 1.1 state列状态
**Checking table**
　正在检查数据表（这是自动的）。
**Closing tables**
　正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
**Connect Out**
　复制从服务器正在连接主服务器。
**Copying to tmp table on disk**
　由于临时结果集大于tmp_table_size，正在将临时表从内存存储转为磁盘存储以此节省内存。
**Creating tmp table**
　正在创建临时表以存放部分查询结果。
**deleting from main table**
　服务器正在执行多表删除中的第一部分，刚删除第一个表。
**deleting from reference tables**
　服务器正在执行多表删除中的第二部分，正在删除其他表的记录。
**Flushing tables**
　正在执行FLUSH TABLES，等待其他线程关闭数据表。
**Killed**
　发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效。
**Locked**
　被其他查询锁住了。
**Sending data**
　正在处理SELECT查询的记录，同时正在把结果发送给客户端。
**Sorting for group**
　正在为GROUP BY做排序。
　Sorting for order
　正在为ORDER BY做排序。
**Opening tables**
　这个过程应该会很快，除非受到其他因素的干扰。例如，在执ALTER TABLE或LOCK TABLE语句行完以前，数据表无法被其他线程打开。正尝试打开一个表。
**Removing duplicates**
　正在执行一个SELECT DISTINCT方式的查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。
**Reopen table**
　获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。已经释放锁，关闭数据表，正尝试重新打开数据表。
**Repair by sorting**
　修复指令正在排序以创建索引。
**Repair with keycache**
　修复指令正在利用索引缓存一个一个地创建新索引。它会比Repair by sorting慢些。
**Searching rows for update**
　正在讲符合条件的记录找出来以备更新。它必须在UPDATE要修改相关的记录之前就完成了。
**Sleeping**
　正在等待客户端发送新请求.
**System lock**
　正在等待取得一个外部的系统锁。如果当前没有运行多个mysqld服务器同时请求同一个表，那么可以通过增加--skip-external-locking参数来禁止外部系统锁。
**Upgrading lock**
　INSERT DELAYED正在尝试取得一个锁表以插入新记录。
**Updating**
　正在搜索匹配的记录，并且修改它们。
**User Lock**
　正在等待GET_LOCK()。
**Waiting for tables**
　该线程得到通知，数据表结构已经被修改了，需要重新打开数据表以取得新的结构。然后，为了能的重新打开数据表，必须等到所有其他线程关闭这个表。以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE,或OPTIMIZE TABLE。
**waiting for handler insert**
　INSERT DELAYED已经处理完了所有待处理的插入操作，正在等待新的请求。
　大部分状态对应很快的操作，只要有一个线程保持同一个状态好几秒钟，那么可能是有问题发生了，需要检查一下。
　还有其他的状态没在上面中列出来，不过它们大部分只是在查看服务器是否有存在错误是才用得着。

# 2、kill 使用
上面提到的 线程ID 是可以通过 kill 杀死的；所以上面基本上可以把有问题的执行语句找出来，然后就可以 kill 掉了。
```sql
-- 查询执行时间超过2分钟的线程，然后拼接成 kill 语句
select concat('kill ', id, ';')
from information_schema.processlist
where command != 'Sleep'
and time > 2*60
order by time desc;
```

# 3、mysql 查看当前连接数
```sql
show processlist;
```
如果是root帐号，你能看到所有用户的当前连接。如果是其它普通帐号，只能看到自己占用的连接。
show processlist;只列出前100条，如果想全列出请使用show full processlist;

![](https://img-blog.csdnimg.cn/20200403174909193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4dzE4NDQ5MTI1MTQ=,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200403174931106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4dzE4NDQ5MTI1MTQ=,size_16,color_FFFFFF,t_70)


```sql
show status;
```
Aborted_clients 由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。
Aborted_connects 尝试已经失败的MySQL服务器的连接的次数。
Connections 试图连接MySQL服务器的次数。
Created_tmp_tables 当执行语句时，已经被创造了的隐含临时表的数量。
Delayed_insert_threads 正在使用的延迟插入处理器线程的数量。
Delayed_writes 用INSERT DELAYED写入的行数。
Delayed_errors 用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。
Flush_commands 执行FLUSH命令的次数。
Handler_delete 请求从一张表中删除行的次数。
Handler_read_first 请求读入表中第一行的次数。
Handler_read_key 请求数字基于键读行。
Handler_read_next 请求读入基于一个键的一行的次数。
Handler_read_rnd 请求读入基于一个固定位置的一行的次数。
Handler_update 请求更新表中一行的次数。
Handler_write 请求向表中插入一行的次数。
Key_blocks_used 用于关键字缓存的块的数量。
Key_read_requests 请求从缓存读入一个键值的次数。
Key_reads 从磁盘物理读入一个键值的次数。
Key_write_requests 请求将一个关键字块写入缓存次数。
Key_writes 将一个键值块物理写入磁盘的次数。
Max_used_connections 同时使用的连接的最大数目。
Not_flushed_key_blocks 在键缓存中已经改变但是还没被清空到磁盘上的键块。
Not_flushed_delayed_rows 在INSERT DELAY队列中等待写入的行的数量。
Open_tables 打开表的数量。
Open_files 打开文件的数量。
Open_streams 打开流的数量(主要用于日志记载）
Opened_tables 已经打开的表的数量。
Questions 发往服务器的查询的数量。
Slow_queries 要花超过long_query_time时间的查询数量。
Threads_connected 当前打开的连接的数量。
Threads_running 不在睡眠的线程数量。
Uptime 服务器工作了多少秒。

# 4、explain分析语句
![](https://img-blog.csdnimg.cn/2020040317533937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4dzE4NDQ5MTI1MTQ=,size_16,color_FFFFFF,t_70)