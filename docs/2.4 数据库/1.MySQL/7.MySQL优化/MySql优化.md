[TOC]

# 1、优化思路
 1. 设计数据库时：数据库表、字段的设计，存储引擎。
 2. 利用好MySQL自身提供的功能，如索引等。
 3. 横向扩展：MySQL集群、负载均衡、读写分离。
 4. SQL语句的优化（收效较低）

# 2、字段设计
## 2.1 原则：定长和非定长数据类型的选择
decimal不会损失精度，存储空间会随数据的增大而增大。double占用固定空间，较大数的存储会损失精度。非定长的还有varchar、text。
 - 金额
对数据精度要求较高，小数的运算和存储存在精度问题（不能将所有小数转换成二进制）。
 - 定点数decimal
price decimal(8,3)有2位小数的定点数，定点数支持很大的数（甚至是超过int，bigint存储范围的数）
 - 字符串存储
定长char，非定长varchar、text（上限65535,其中varchar还会消耗1-3字节记录长度，而text使用额外空间记录长度）
 - 小单位大数额避免出现小数。

## 2.2 原则：尽可能使用小的数据类型和指定短的长度
 - 尽可能使用not null
  - 非null字段的处理要比null字段处理的高效。切不需要判断是否为null。
  - null在mysql中，存储需要额外空间，运算也需要特殊的运算符。如select null = null和select null <> null，有相同结果，只能通过is null 和 is not null来判断字段是否为null。
  - mysql中每条记录都需要额外的存储空间，表示每个字段是否为null。因此通常使用特殊的数据进行占位，比如int not null default 0，string not null default ''。

## 2.3 单表字段不宜过多
 - <30为合格， <20为佳

# 3、关联表的设计
外键foreign key只能实现一对一或一对多的映射。
 - 一对多：使用外键
 - 多对多：单独新建一张表将多对多拆分成两个一对多。
 - 一对一：使用相同的主键或者增加一个外键字段。



