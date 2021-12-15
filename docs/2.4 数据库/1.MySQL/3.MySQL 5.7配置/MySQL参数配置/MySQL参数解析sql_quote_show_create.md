> MySQL执行show create table和show create database命令，显示建表或者建库语句时，会在表名、库名、字段名的两边加上引号，比如 `id`，参数 sql_quote_show_create 设置为OFF时，可以将库名、表名、字段名两侧的引号去除。

**sql_quote_show_create：**

- 作用范围：全局，会话级
- 动态修改：是
- 取值范围：ON，OFF
- 默认值：ON

**示例：**
（1）sql_quote_show_create=ON，有引号

```
mysql> show create table tb\G
*************************** 1. row ***************************
       Table: tb
Create Table: CREATE TABLE `tb` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```

（2）sql_quote_show_create=OFF，无引号

```
mysql> show create table tb\G
*************************** 1. row ***************************
       Table: tb
Create Table: CREATE TABLE tb (
  id int(11) NOT NULL AUTO_INCREMENT,
  username varchar(50) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY idx_username (username)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```

在sql_quote_show_create=OFF时，有时候也有可能出现引号，比如表名，库名或者字段名使用了MySQL的关键字时，也会加上引号，如下所示，字段create，由于是MySQL关键字，两边加上了引号，而age没有加引号。

```
mysql> show create table tb1\G
*************************** 1. row ***************************
       Table: tb1
Create Table: CREATE TABLE tb1 (
  `create` varchar(50) DEFAULT NULL,
  age int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```