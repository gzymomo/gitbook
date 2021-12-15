#### 1. 什么是MySQL派生表？

派生表（Derived Table），是复杂SQL在执行过程中的中间表，也可认为是临时表，存放SQL执行过程中必不可少的中间数据。通常由跟在from子句或者join子句之后的子查询产生，比如下面两个派生表例子，derived_t1和derived_t2都是派生表。

select * from (select * from t1) as derived_t1;

select t1.* from t1 join (select distinct class_id from t2) as derived_t2 where t1.id=derived_t2.class_id;

#### 2. MySQL优化器如何处理派生表？

MySQL优化器处理派生表有两种策略：

1. 将派生生合并(Merging)到外部的查询块
2. 物化(Materialize)派生表到内部的临时表

##### 2.1 合并(Merging )

对于简单的派生表，比如没有使用group by，having，distinct，limit等，这类派生表可直接合并到外部的查询块中。如下面两个例子：

**例子1：**
SELECT * FROM (SELECT * FROM t1) AS derived_t1;

合并后变成：

SELECT * FROM t1;

**例子2：**

SELECT * FROM t1 JOIN (SELECT t2.f1 FROM t2) AS derived_t2 ON t1.f2=derived_t2.f1 WHERE t1.f1 > 0;

合并后变成：

SELECT t1.* FROM t1 JOIN t2 ON t1.f2=t2.f1 WHERE t1.f1 > 0;

##### 2.2 物化(Materialization)

所谓物化，可以理解为在内存中生成一张临时表，将派生表实例化，物化派生表成本较高，尤其当派生表很大，内存不够用时，物化的派生表还要转存到磁盘中，对性能影响进一步增加。

##### 2.3 优化器对派生表做的优化

1. 对于简单的派生表，进行合并处理。派生表合并也有限制，当外层查询块涉及的表数量超过一定阈值(61)时，优化器将选择物化派生表，而不是合并派生表。
2. MySQL优化器在处理派生表时，能合并的先合并，不能合并的，考虑物化派生表，然而物化派生表也是有成本的，优化器也会尽量推迟派生表的物化，甚至不物化派生表。比如join的两个表，A为派生表，另外一个表为B，优化器推迟派生表A的物化，先处理表B，当表B中的数据根据条件过滤后，数据很少，或者完全没有数据时，此时物化派生表A成本就会低很多，甚至不用物化派生表。
3. 查询条件从外层查询下推到派生表中，以便生成更小的派生表，越小的派生表，其性能越好。
4. 优化器将会对物化的派生表增加索引，以加速访问。比如下面这个例子，优化器将会对derived_t2的f1字段添加索引，以提高查询性能。
   SELECT * FROM t1 JOIN (SELECT DISTINCT f1 FROM t2) AS derived_t2 ON t1.f1=derived_t2.f1;

#### 3. 派生表合并开关

派生表合并，通过参数optimizer_switch来控制是否打开，默认为打开。
optimizer_switch='derived_merge=ON';

#### 4. 派生表合并限制

不是所有的派生表都可以被merge，简单的派生表，可以被merge，复杂的带有GROUP，HAVING，DISTINCT，LIMIT，聚集函数等子句的派生表只能被物化，不能被合并。