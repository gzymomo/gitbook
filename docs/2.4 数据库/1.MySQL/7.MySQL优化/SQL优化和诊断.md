[TOC]

# 1、Explain诊断
![](https://www.showdoc.cc/server/api/common/visitfile/sign/7463cf2604d2b0bfe711e9b5c23bc3d4?showdoc=.jpg)

## 1.1 select_type 常见类型及其含义
- SIMPLE：不包含子查询或者 UNION 操作的查询
- PRIMARY：查询中如果包含任何子查询，那么最外层的查询则被标记为 PRIMARY
- SUBQUERY：子查询中第一个 SELECT
- DEPENDENT SUBQUERY：子查询中的第一个 SELECT，取决于外部查询
- UNION：UNION 操作的第二个或者之后的查询
- DEPENDENT UNION：UNION 操作的第二个或者之后的查询,取决于外部查询
- UNION RESULT：UNION 产生的结果集
- DERIVED：出现在 FROM 字句中的子查询

## 1.2 type常见类型及其含义
- system：这是 const 类型的一个特例，只会出现在待查询的表只有一行数据的情况下
- consts：常出现在主键或唯一索引与常量值进行比较的场景下，此时查询性能是最优的
- eq_ref：当连接使用的是完整的索引并且是 PRIMARY KEY 或 UNIQUE NOT NULL INDEX 时使用它
- ref：当连接使用的是前缀索引或连接条件不是 PRIMARY KEY 或 UNIQUE INDEX 时则使用它
- ref_or_null：类似于 ref 类型的查询，但是附加了对 NULL 值列的查询
- index_merge：该联接类型表示使用了索引进行合并优化
- range：使用索引进行范围扫描，常见于 between、> 、< 这样的查询条件
- index：索引连接类型与 ALL 相同，只是扫描的是索引树，通常出现在索引是该查询的覆盖索引的情况
- ALL：全表扫描，效率最差的查找方式

> 阿里编码规范要求：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好

## 1.3 key列
实际在查询中是否使用到索引的标志字段

## 1.4 如何查看Mysql优化器优化之后的SQL
```sql
# 仅在服务器环境下或通过Navicat进入命令列界面
explain extended  SELECT * FROM `student` where `name` = 1 and `age` = 1;

# 再执行
show warnings;

# 结果如下：
/* select#1 */ select `mytest`.`student`.`age` AS `age`,`mytest`.`student`.`name` AS `name`,`mytest`.`student`.`year` AS `year` from `mytest`.`student` where ((`mytest`.`student`.`age` = 1) and (`mytest`.`student`.`name` = 1))
```

# 2、SQL优化
## 2.1 超大分页场景解决方案
如表中数据需要进行深度分页，如何提高效率？
> 利用延迟关联或者子查询优化超多分页场景

说明：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回 N 行，那当 offset 特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 SQL 改写。

```sql
# 反例（耗时129.570s）
select * from task_result LIMIT 20000000, 10;

# 正例（耗时5.114s）
SELECT a.* FROM task_result a, (select id from task_result LIMIT 20000000, 10) b where a.id = b.id;

# 说明
task_result表为生产环境的一个表，总数据量为3400万，id为主键，偏移量达到2000万
```

## 2.2 获取一条数据时的Limit 1
如果数据表的情况已知，某个业务需要获取符合某个Where条件下的一条数据，注意使用Limit

说明：在很多情况下我们已知数据仅存在一条，此时我们应该告知数据库只用查一条，否则将会转化为全表扫描。
```sql
# 反例（耗时2424.612s）
select * from task_result where unique_key = 'ebbf420b65d95573db7669f21fa3be3e_861414030800727_48';

# 正例（耗时1.036s）
select * from task_result where unique_key = 'ebbf420b65d95573db7669f21fa3be3e_861414030800727_48' LIMIT 1;

# 说明
task_result表为生产环境的一个表，总数据量为3400万，where条件非索引字段，数据所在行为第19486条记录
```
## 2.3 批量插入
```sql
# 反例
INSERT into person(name,age) values('A',24)
INSERT into person(name,age) values('B',24)
INSERT into person(name,age) values('C',24)

# 正例
INSERT into person(name,age) values('A',24),('B',24),('C',24);
```

## 2.4 like语句的优化
like语句一般业务要求都是 '%关键字%'这种形式，但是依然要思考能否考虑使用右模糊的方式去替代产品的要求，其中阿里的编码规范提到：
> 页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决

```sql
# 反例（耗时78.843s）
EXPLAIN select * from task_result where taskid LIKE '%tt600e6b601677b5cbfe516a013b8e46%' LIMIT 1;

# 正例（耗时0.986s）
select * from task_result where taskid LIKE 'tt600e6b601677b5cbfe516a013b8e46%' LIMIT 1

##########################################################################
# 对正例的Explain
1	SIMPLE	task_result		range	adapt_id	adapt_id	98		99	100.00	Using index condition

# 对反例的Explain
1	SIMPLE	task_result		ALL					                    33628554	11.11	Using where

# 说明
task_result表为生产环境的一个表，总数据量为3400万，taskid是一个普通索引列，可见%%这种匹配方式完全无法使用索引，从而进行全表扫描导致效率极低，而正例通过索引查找数据只需要扫描99条数据即可
```

## 2.5 使用 ISNULL()来判断是否为 NULL 值
说明：NULL 与任何值的直接比较都为 NULL
```sql
# 1） NULL<>NULL 的返回结果是 NULL，而不是 false。 
# 2） NULL=NULL 的返回结果是 NULL，而不是 true。 
# 3） NULL<>1 的返回结果是 NULL，而不是 true。
```

## 2.6 多表查询
超过三个表禁止 join。需要 join 的字段，数据类型必须绝对一致；多表关联查询时，保证被关联的字段需要有索引。

## 2.7 count(*) 还是 count(id)
阿里的Java编码规范中有以下内容：
【强制】不要使用 count(列名) 或 count(常量) 来替代 count(*)
count(*) 是 SQL92 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关。
说明：count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行

## 2.8 字段类型不同导致索引失效
阿里的Java编码规范中有以下内容：

【推荐】防止因字段类型不同造成的隐式转换，导致索引失效

实际上数据库在查询的时候会作一层隐式的转换，比如 varchar 类型字段通过 数字去查询。
```sql
# 正例
EXPLAIN SELECT * FROM `user_coll` where pid = '1';
type：ref
ref：const
rows:1
Extra:Using index condition

# 反例
EXPLAIN SELECT * FROM `user_coll` where pid = 1;
type：index
ref：NULL
rows:3(总记录数)
Extra:Using where; Using index

# 说明
pid字段有相应索引，且格式为varchar 
```


