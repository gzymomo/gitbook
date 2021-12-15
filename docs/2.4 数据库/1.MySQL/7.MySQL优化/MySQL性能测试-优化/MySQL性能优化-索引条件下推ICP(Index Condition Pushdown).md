> 索引条件下推，Index Condition Pushdown，简称ICP，是MySQL内部通过索引查询数据的一种优化方法，简单来说就是将原本需要在Server层对数据进行过滤的条件下推到了引擎层去做，在引擎层过滤更多的数据，这样从引擎层发送到Server层的数据就会显著减少，从而优化性能。

原文地址：

https://mytecdb.com/blogDetail.php?id=97

#### 1. ICP索引下推原理

举一个例子，有一个索引如下：
idx_all(a,b,c)

查询语句：
select d from t where a='xx' and b like '%xx%' and c like '%xx%'

查询走索引idx_all，但是只能使用前缀a的部分。

这样一个查询，在没有使用ICP时，存储引擎根据索引条件a='xx'，获取记录，将这些记录返回给Server层，Server层再根据条件 b like '%xx%' 和 c like '%xx%' 来进一步过滤记录，最后回表拿到d字段数据返回给用户。

使用ICP时，存储引擎根据索引条件a='xx'，获取记录，并在引擎层根据条件 b like '%xx%' 和 c like '%xx%' 来过滤数据，然后引擎返回记录到Server层，显然，使用ICP时，返回给Server层的记录数量会显著减少，Server层不需要再过滤，直接回表查询，整体效率将会提高很多。

总的来说，ICP主要在引擎层增加了条件过滤能力，减少了引擎层向Server层传输的数据量。

#### 2. ICP索引下推的适用条件

1. 查询走索引，explain中的访问方式为range，ref，eq_ref，ref_or_null，并且需要回表查询。
2. ICP可用于InnoDB、MyISAM及分区表。
3. 对于InnoDB表，ICP只适用于走二级索引的查询。
4. ICP不支持在虚拟列上创建的二级索引。
5. 如果where条件涉及子查询，则不能使用ICP。
6. 如果where条件使用函数，不能使用ICP。
7. 触发器条件也不能使用ICP。

如果一个SQL使用了ICP优化，那么在Explain的输出中，其Extra列会显示 Using index condition。

#### 3. ICP索引下推举例

表结构：

```
CREATE TABLE `people` (
  `zipcode` varchar(50) DEFAULT NULL,
  `lastname` varchar(50) DEFAULT NULL,
  `address` varchar(50) DEFAULT NULL,
  `age` int(11) DEFAULT '0',
  KEY `idx_all` (`zipcode`,`lastname`,`address`)
);
```

select age from people where zipcode='0000005' and lastname like '%1%' and address like '%1%';

这个SQL走了二级索引 idx_all，并且需要回表查询，所以满足ICP优化的条件，从explain的结果来看，Extra字段也确实是Using index condition，如下：

```
mysql> explain select age from people where zipcode='0000005' and lastname like '%xx%' and address like '%xx%';
+----+------+---------------+---------+---------+-------+------+----------+-----------------------+
| id | type | possible_keys | key     | key_len | ref   | rows | filtered | Extra                 |
+----+------+---------------+---------+---------+-------+------+----------+-----------------------+
|  1 | ref  | idx_all       | idx_all | 153     | const |    1 |    50.00 | Using index condition |
+----+------+---------------+---------+---------+-------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```

**ICP优化效果：**

- 表中记录总数：393216
- 满足条件【zipcode='0000005'】的记录数：32768
- 满足条件【zipcode='0000005' and lastname like '%1%' and address like '%1%'】的记录数：15000

执行耗时：

- 未使用ICP：1.07秒
- 使用ICP：0.62秒

从以上执行耗时来看，ICP优化效果还是不错的，当然不同的数据分布对优化效果也会有一定的影响，有兴趣的可以自己测试一下。