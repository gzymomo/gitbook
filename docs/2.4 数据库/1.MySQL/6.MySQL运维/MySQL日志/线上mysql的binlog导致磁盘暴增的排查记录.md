- [线上mysql的binlog导致磁盘暴增的排查记录](https://blog.51cto.com/mapengfei/2675813)



# 事情由来：

一大早突然收到zabbix告警，说磁盘就剩不到15了，赶紧上去瞅瞅什么情况

![线上mysql的binlog导致磁盘暴增的排查记录](https://s4.51cto.com/images/blog/202103/29/da326741288465234cf33317aa873d28.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

# 排查过程

1、df -ah 看看确认是/data目录占用91%了已经
![线上mysql的binlog导致磁盘暴增的排查记录](https://s4.51cto.com/images/blog/202103/29/662b71cbb0fb8b4700ae48e8508fe74d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

2、找到具体/data下的哪个目录用的，最终定位到是mysql的binlog占用，这很明显是binlog写的太多导致的了，
ps， 这里排查过程那个图忘了截了，可以用du -ah -d 1  看具体某个目录下的所有子目录的占用大小 -d 是代表层级关系，一级一级敲下来就找到哪个目录用的了
![线上mysql的binlog导致磁盘暴增的排查记录](https://s4.51cto.com/images/blog/202103/29/bed2bb6983bcb4bf372ddfd9c3e786bb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

3、开始把binlog从raw改为STATEMENT模式；

```
mysql> show global variables like "%binlog_format%";
+---------------+-----------+
| Variable_name | Value     |
+---------------+-----------+
| binlog_format | STATEMENT |
+---------------+-----------+
1 row in set (0.01 sec)

mysql> SET global binlog_format='STATEMENT';
Query OK, 1 rows affected (0.00 sec)
mysql> reset master;
Query OK, 0 rows affected (1.49 sec)
```

改完mysql配置之后，再去binlog目录下去看，就会发现一堆binlog没啦，磁盘使用也降到正常了

4、改配置方式也可以：
去mysql的my.cnf中修改对应的值：

```
# 1天过期清理binlog日志
expire_logs_days =1
# 修改记录模式为statement
binlog_format = statement
```

# 原因分析

```
mysql是默认开启binlog日志的并且对于已经存储的binlog日志没有设置自动清理时间，
mysql在5.7.7版本之后默认binlog存储格式改为了ROW（也就是最详细最消耗磁盘的一种）
所以会出现binlog日志50多G的情况
```

### binlog基本概念

```
binlog是Mysql sever层维护的一种二进制日志，与innodb引擎中的redo/undo log是完全不同的日志；
其主要是用来记录对mysql数据更新或潜在发生更新的SQL语句，
记录了所有的DDL和DML(除了数据查询语句)语句，
并以事务的形式保存在磁盘中，还包含语句所执行的消耗的时间，
MySQL的二进制日志是事务安全型的。
```

### binlog的三种存储模式

### Row

```
优点：在row level模式下，bin-log中可以不记录执行的sql语句的上下文相关的信息，
仅仅只需要记录那一条被修改。
缺点：
row level，所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，会产生大量的日志内容。
```

### Statement

```
优点：statement level下的优点首先就是解决了row level下的缺点，不需要记录每一行数据的变化，减少bin-log日志量，节约IO，提高性能，
因为它只需要在Master上锁执行的语句的细节，以及执行语句的上下文的信息。
缺点：由于只记录语句，所以，在statement level下 已经发现了有不少情况会造成MySQL的复制出现问题，
主要是修改数据的时候使用了某些定的函数或者功能的时候会出现。
```

### Mixed

```
在Mixed模式下，MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志格式，
也就是在Statement和Row之间选择一种。
如果sql语句确实就是update或者delete等修改数据的语句，那么还是会记录所有行的变更。
```