在日常mysql运维中，经常要查询当前mysql下正在执行的sql语句及其他在跑的mysql相关线程，这就用到mysql processlist这个命令了。

```mysql
mysql> show processlist;      //查询正在执行的sql语句
mysql> show full processlist;    //查询正在执行的完整sql语句
mysql> **kill connection id**      //停掉processlist查询出的某个线程，id是对应的id号
mysql> show processlist;
```



```mysql
+------+-------+---------------------+--------+-------------+------+-----------------------------------------------------------------------+------------------+
| Id   | User  | Host                | db     | Command     | Time | State                                                                 | Info             |
+------+-------+---------------------+--------+-------------+------+-----------------------------------------------------------------------+------------------+
| 1574 | root  | localhost           | huanpc | Sleep       |  702 |                                                                       | NULL             |
| 1955 | root  | localhost           | NULL   | Query       |    0 | init                                                                  | show processlist |
| 1958 | slave | 192.168.1.102:37399 | NULL   | Binlog Dump |   10 | Master has sent all binlog to slave; waiting for binlog to be updated | NULL             |
+------+-------+---------------------+--------+-------------+------+-----------------------------------------------------------------------+------------------+
3 rows in set (0.00 sec)
 
mysql> kill connection 1574;
Query OK, 0 rows affected (0.00 sec)
 
mysql> show processlist;
+------+-------+---------------------+------+-------------+------+-----------------------------------------------------------------------+------------------+
| Id   | User  | Host                | db   | Command     | Time | State                                                                 | Info             |
+------+-------+---------------------+------+-------------+------+-----------------------------------------------------------------------+------------------+
| 1955 | root  | localhost           | NULL | Query       |    0 | init                                                                  | show processlist |
| 1958 | slave | 192.168.1.102:37399 | NULL | Binlog Dump |   18 | Master has sent all binlog to slave; waiting for binlog to be updated | NULL             |
+------+-------+---------------------+------+-------------+------+-----------------------------------------------------------------------+------------------+
2 rows in set (0.00 sec)
```

除此之外，**show processlist还能查看当前mysql连接数。**
如果是root帐号，能看到所有用户的当前连接。
如果是其它普通帐号，只能看到自己占用的连接。
注意：
show processlist;只列出前100条
如果想全列出要使用show full processlist;

 

使用show status;可以比较全面地查看到mysql状态
mysql> show status;

参数解释：
Aborted_clients        由于客户没有正确关闭连接已经死掉，已经放弃的连接数量。
Aborted_connects      尝试已经失败的MySQL服务器的连接的次数。
Connections          试图连接MySQL服务器的次数。
Created_tmp_tables    当执行语句时，已经被创造了的隐含临时表的数量。
Delayed_insert_threads  正在使用的延迟插入处理器线程的数量。
Delayed_writes        用INSERT DELAYED写入的行数。
Delayed_errors        用INSERT DELAYED写入的发生某些错误(可能重复键值)的行数。
Flush_commands       执行FLUSH命令的次数。
Handler_delete         请求从一张表中删除行的次数。
Handler_read_first      请求读入表中第一行的次数。
Handler_read_key       请求数字基于键读行。
Handler_read_next      请求读入基于一个键的一行的次数。
Handler_read_rnd       请求读入基于一个固定位置的一行的次数。
Handler_update         请求更新表中一行的次数。
Handler_write          请求向表中插入一行的次数。
Key_blocks_used        用于关键字缓存的块的数量。
Key_read_requests      请求从缓存读入一个键值的次数。
Key_reads             从磁盘物理读入一个键值的次数。
Key_write_requests      请求将一个关键字块写入缓存次数。
Key_writes             将一个键值块物理写入磁盘的次数。
Max_used_connections    同时使用的连接的最大数目。
Not_flushed_key_blocks   在键缓存中已经改变但是还没被清空到磁盘上的键块。
Not_flushed_delayed_rows 在INSERT DELAY队列中等待写入的行的数量。
Open_tables            打开表的数量。
Open_files              打开文件的数量。
Open_streams           打开流的数量(主要用于日志记载）
Opened_tables           已经打开的表的数量。
Questions              发往服务器的查询的数量。
Slow_queries            要花超过long_query_time时间的查询数量。
Threads_connected        当前打开的连接的数量。
Threads_running          不在睡眠的线程数量。
Uptime                 服务器工作了多少秒。