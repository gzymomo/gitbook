> MySQL一直以来提供show profile命令来获取某一条SQL执行过程中的资源使用与耗时情况，这个命令对于分析具体SQL的性能瓶颈有非常大的帮助，但是这个功能在MySQL新的版本里将会被废弃，取而代之的是使用Performance Schema来提供同样的功能。本文将介绍如何使用Performance Schema来实现show profile SQL性能分析的功能。

#### 1. 测试环境配置

- 测试版本：MySQL 5.7.19
- 配置参数：performance_schema=ON，该参数配置在my.cnf文件中，生效需要重启MySQL。

#### 2. 配置performance_schema

##### 2.1 配置表setup_actors

默认情况下，performance_schema功能打开后，将会收集所有用户的SQL执行历史事件，因为收集的信息太多，对数据库整体性能有一定影响，而且也不利于排查指定SQL的性能问题，因此需要修改setup_actors表的配置，只收集特定用户的历史事件信息。setup_actors表配置如下：

```
mysql> select * from performance_schema.setup_actors;
+-----------+------+------+---------+---------+
| HOST      | USER | ROLE | ENABLED | HISTORY |
+-----------+------+------+---------+---------+
| %         | %    | %    | NO      | NO      |
| localhost | root | %    | YES     | YES     |
+-----------+------+------+---------+---------+
2 rows in set (0.00 sec)
```

只收集本地root用户的SQL执行历史事件。

##### 2.2 配置表setup_instruments

启用statement和stage监视器。

```
UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES' WHERE NAME LIKE '%statement/%';

UPDATE performance_schema.setup_instruments SET ENABLED = 'YES', TIMED = 'YES' WHERE NAME LIKE '%stage/%';
```

##### 2.3 配置表setup_consumers

启用events_statements_*，events_stages_* 开头的事件类型消费。

```
UPDATE performance_schema.setup_consumers SET ENABLED = 'YES' WHERE NAME LIKE '%events_statements_%';

UPDATE performance_schema.setup_consumers SET ENABLED = 'YES' WHERE NAME LIKE '%events_stages_%';
```

#### 3. 收集具体SQL的性能分析

##### 3.1 执行业务SQL

在上述配置完成之后，执行一个需要分析的业务SQL，比如：
select * from blog;

##### 3.2 获取业务SQL的事件ID

通过以下SQL先查询事件ID。

```
SELECT EVENT_ID, TRUNCATE(TIMER_WAIT/1000000000000,6) as Duration, SQL_TEXT FROM performance_schema.events_statements_history_long WHERE SQL_TEXT like '%blog%';
+----------+----------+-------------------------+
| EVENT_ID | Duration | SQL_TEXT                |
+----------+----------+-------------------------+
|      243 | 0.002698 | select * from blog.blog |
+----------+----------+-------------------------+
1 row in set (0.00 sec)
```

##### 3.3 根据事件ID，获取各阶段执行耗时

根据上一步获取的事件ID(EVENT_ID)，查询该SQL各个阶段的耗时情况。如下：

```
SELECT event_name AS Stage, TRUNCATE(TIMER_WAIT/1000000000000,6) AS Duration FROM performance_schema.events_stages_history_long WHERE NESTING_EVENT_ID=243;
+--------------------------------+----------+
| Stage                          | Duration |
+--------------------------------+----------+
| stage/sql/starting             | 0.000117 |
| stage/sql/checking permissions | 0.000010 |
| stage/sql/Opening tables       | 0.000031 |
| stage/sql/init                 | 0.000055 |
| stage/sql/System lock          | 0.000024 |
| stage/sql/optimizing           | 0.000003 |
| stage/sql/statistics           | 0.000020 |
| stage/sql/preparing            | 0.000015 |
| stage/sql/executing            | 0.000001 |
| stage/sql/Sending data         | 0.002321 |
| stage/sql/end                  | 0.000004 |
| stage/sql/query end            | 0.000019 |
| stage/sql/closing tables       | 0.000018 |
| stage/sql/freeing items        | 0.000048 |
| stage/sql/cleaning up          | 0.000001 |
+--------------------------------+----------+
15 rows in set (0.00 sec)
```

上述结果与show profile的输出结果类似，能够看到每个阶段的耗时情况。相对于show profile来说，似乎更加繁琐，不过performance schema是MySQL未来性能分析的趋势，提供了非常丰富的性能诊断工具，熟悉performance schema的使用将有助于更好的优化MySQL。