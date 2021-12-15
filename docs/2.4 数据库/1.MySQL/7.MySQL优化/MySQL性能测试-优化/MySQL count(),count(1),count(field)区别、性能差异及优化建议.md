> count函数是用来获取表中满足一定条件的记录数，常见用法有三种，count(*),count(1),count(field)，这三种有什么区别？在性能上有何差异？本文将通过测试案例详细介绍和分析。

原文地址：

[mytecdb.com/blogDetail.php?id=81](https://mytecdb.com/blogDetail.php?id=81)

三者有何区别：

- count(field)不包含字段值为NULL的记录。
- count(*)包含NULL记录。
- select(*)与select(1) 在InnoDB中性能没有任何区别，处理方式相同。官方文档描述如下：
  InnoDB handles SELECT COUNT(*) and SELECT COUNT(1) operations in the same way. There is no performance difference.

##### 1. 性能对比

通过案例来测试一下count(*)，count(1)，count(field)的性能差异，MySQL版本为5.7.19，测试表是一张sysbench生成的表，表名sbtest1，总记录数2411645，如下：

```
CREATE TABLE sbtest1 (
id int(11) NOT NULL AUTO_INCREMENT,
k int(11) DEFAULT NULL,
c char(120) NOT NULL DEFAULT '',
pad char(60) NOT NULL DEFAULT '',
PRIMARY KEY (id),
KEY k_1 (k)
) ENGINE=InnoDB;
```

**测试SQL语句：**

select count(*) from sbtest1;
select count(1) from sbtest1;
select count(id) from sbtest1;
select count(k) from sbtest1;
select count(c) from sbtest1;
select count(pad) from sbtest1;

针对count(*)、count(1)和count(id)，加了强制走主键的测试，如下：
select count(*) from sbtest1 force index(primary);
select count(1) from sbtest1 force index(primary);
select count(id) from sbtest1 force index(primary);

另外对不同的测试SQL，收集了profile，发现主要耗时都在Sending data这个阶段，记录Sending data值。

**汇总测试结果：**

| 类型                | 耗时(s) | 索引        | Sending data耗时(s) |
| ------------------- | ------- | ----------- | ------------------- |
| count(*)            | 0.47    | k_1         | 0.463624            |
| count(1)            | 0.46    | k_1         | 0.463242            |
| count(id)           | 0.52    | k_1         | 0.521618            |
| count(*)强制走主键  | 0.54    | primay key  | 0.538737            |
| count(1)强制走主键  | 0.55    | primary key | 0.545007            |
| count(id)强制走主键 | 0.60    | primary key | 0.598975            |
| count(k)            | 0.53    | k_1         | 0.529366            |
| count(c)            | 0.81    | NULL        | 0.813918            |
| count(pad)          | 0.76    | NULL        | 0.762040            |

**结果分析：**

1. 从以上测试结果来看，count(*)和count(1)性能基本一样，默认走二级索引(k_1)，性能最好，这也验证了count(*)和count(1)在InnoDB内部处理方式一样。
2. count(id) 虽然也走二级索引(k_1)，但是性能明显低于count(*)和count(1)，可能MySQL内部在处理count(*)和count(1)时做了额外的优化。
3. 强制走主键索引时，性能反而没有走更小的二级索引好，InnoDB存储引擎是索引组织表，行数据在主键索引的叶子节点上，走主键索引扫描时，处理的数据量比二级索引更多，所以性能不及二级索引。
4. count(c)和count(pad)没有走索引，性能最差，但是明显count(pad)比count(c)好，因为pad字段类型为char(60)，小于字段c的char(120)，尽管两者性能垫底，但是字段小的性能相对更好些。

##### 2. count(*)延伸

- 在5.7.18版本之前，InnoDB处理select count(*) 是通过扫描聚簇索引，来获取总记录数。
- 从5.7.18版本开始，InnoDB扫描一个最小的可用的二级索引来获取总记录数，或者由SQL hint来告诉优化器使用哪个索引。如果二级索引不存在，InnoDB将会扫描聚簇索引。

执行select count(*)在大部分场景下性能都不会太好，尤其是表记录数特别大的情况下，索引数据不在buffer pool里面，需要频繁的读磁盘，性能将更差。

##### 3. count(*)优化思路

1. 一种优化方法，是使用一个统计表来存储表的记录总数，在执行DML操作时，同时更新该统计表。这种方法适用于更新较少，读较多的场景，而对于高并发写操作，性能有很大影响，因为需要并发更新热点记录。
2. 如果业务对count数量的精度没有太大要求，可使用show table status中的行数作为近似值。