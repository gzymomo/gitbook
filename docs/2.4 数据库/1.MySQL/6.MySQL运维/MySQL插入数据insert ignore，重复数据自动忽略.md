> MySQL中插入数据，如果插入的数据在表中已经存在（主键或者唯一键已存在），使用insert ignore 语法可以忽略插入重复的数据。

#### 1、insert ignore 语法

insert ignore into table_name values...

使用insert ignore语法插入数据时，如果发生主键或者唯一键冲突，则忽略这条插入的数据。

满足以下条件之一：

- 主键重复
- 唯一键重复

#### 2、insert ignore 案例

先看一张表，表名table_name，主键id，唯一键name，具体表结构及表中数据如下：

```
 CREATE TABLE table_name(
  id int(11) NOT NULL,
  name varchar(50) DEFAULT NULL,
  age int(11) DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE KEY uk_name (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
mysql> select * from table_name;
+----+------+------+
| id | name | age  |
+----+------+------+
|  1 | Tom  |   20 |
+----+------+------+
```

##### 2.1 主键冲突

插入一条记录，id为1，如果不加 ignore ，报主键冲突的错误，如下：
mysql> **insert into table_name values(1,'Bill', 21);**
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

加上ignore之后，不会报错，但有一个warning警告，如下：
mysql> **insert ignore into table_name values(1,'Bill', 21);**
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> show warnings;
+---------+------+---------------------------------------+
| Level  | Code | Message                |
+---------+------+---------------------------------------+
| Warning | 1062 | Duplicate entry '1' for key 'PRIMARY' |
+---------+------+---------------------------------------+

查询表，发现插入的数据被忽略了。
mysql> select * from table_name;
+----+------+------+
| id | name | age |
+----+------+------+
| 1 | Tom |  20 |
+----+------+------+

##### 2.2 唯一键冲突

同样，插入唯一键冲突的数据也会忽略，如下所示：
mysql> **insert into table_name values(2,'Tom',21);**
ERROR 1062 (23000): Duplicate entry 'Tom' for key 'uk_name'

mysql> **insert ignore into table_name values(2,'Tom',21);**
Query OK, 0 rows affected, 1 warning (0.00 sec)

如果业务逻辑需要插入重复数据时自动忽略，不妨试试MySQL 的 insert ignore 功能。