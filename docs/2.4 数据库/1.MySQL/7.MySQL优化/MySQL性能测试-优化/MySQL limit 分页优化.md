MySQL limit 子句可以实现分页查询，比如limit 100,10 取偏移100条记录之后的10条记录。limit 子句在提供便捷分页功能的同时，也带来了性能问题，当表数据量非常大，分页数非常多，查询比较靠后的页时，SQL执行性能非常差。

##### 优化方法一，改写SQL

看一个例子，一张表，有64w多条记录。

```
mysql> select count(*) from sbtest2;
+----------+
| count(*) |
+----------+
|   640133 |
+----------+
1 row in set (2.67 sec)
```

查询50w之后的10条记录，耗时5.06秒，如下：

```
mysql> select * from sbtest2 order by id limit 500000,10;
...
10 rows in set (5.06 sec)
```

换一种写法，耗时3.81秒，明显优于前一种写法，如下：

```
mysql> select * from sbtest2 t1, (select id from sbtest2 order by id limit 500000,10) t2 where t1.id=t2.id;
...
10 rows in set (3.81 sec)
```

换一种写法，使用了子查询，子查询中通过id所在索引（主键或辅助索引）扫描，将需要的10条记录的id检索出来，然后再进行表关联，查出10条记录的完整信息，避免了50w条数据由InnoDB存储引擎传递给MySQL Server层的IO消耗，从而提升性能。

offset越大，检索字段越多，这种写法的性能提升越明显。

##### 优化方法二，记录最大最小id

程序记录每个分页的最大和最小id，翻页时根据上一页的最大，最小id来检索记录，可以避免大量无效的数据检索消耗，如下所示：

select * from sbtest2 where id > $max_id order by id limit 10;

select * from sbtest2 where id < $mim_id order by id desc limit 10;

这种方式，由于能够按id走索引，SQL执行性能非常快。