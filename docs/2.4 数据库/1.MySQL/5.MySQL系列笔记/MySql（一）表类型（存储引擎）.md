[TOC]

MySql（一）表类型（存储引擎）
# MYSQL存储引擎概述
MYSQL支持的存储引擎包括：MyISAM、InnoDB、BDB、MERGE、EXAMPLE、NDB Cluster、CSV等。
其中InnoDB和BDB提供**事务安全表**，其他存储引擎都是非事务安全表。

查看当前的默认存储引擎：
```sql
mysql> show variables like 'table_type';
```
查询当前数据库支持的存储引擎：
```sql
mysql> show varuables like 'hava%';
```
在创建新表的时候，可以通过增加ENGINE关键字设置新建表的存储引擎，例如：
```sql
create table ai (
	i bigint(20) not null AUTO_INCREMENT,
	PRIMART kEY(i)
)ENGINE=MyISAM DEAFULT CHARSET=gbk;
```
```sql
create table country(
	country_id smallint unsigned not null auto_increment,
	country varchar(50) not null,
	last_update timestamp not null default current_timestamp on update current_timestamp,
	primary key (country id)
)engine=InnoDB deafult charset=gbk;
```
也可以使用Alter table，将一个已经存在的表修改成其他的存储引擎，例如：
```sql
mysql> alter table ai engine = innodb;
```
# 存储引擎的特性对比
| 特点  | MyISAM  | InnoDB  |
| ------------ | ------------ | ------------ |
| 存储限制  |  有 | 64TB  |
| 事务安全  |   | 支持  |
| 锁机制  | 表锁  | 行锁  |
| B树索引  | 支持  | 支持  |
| 哈希索引  |   |   |
| 全文索引  | 支持  |   |
| 集群索引  |   | 支持  |
| 数据缓存  |   | 支持  |
| 索引缓存  | 支持  | 支持  |
| 数据可压缩  | 支持  |   |
| 空间使用  | 低  | 高  |
| 内存使用  | 低  |  高 |
| 批量插入的速度  |  高 | 低  |
| 支持外键  |   |  支持 |

## MyISAM
MyISAM不支持事务，也不支持外键，优势是访问速度快，对事务完整性没有要求或者以select、insert为主的应用基本都可以使用这个引擎来建表。
每个MyISAM在磁盘上存储成3个文件，其文件名和表明相同，扩展名分别是：
- .frm(存储表定义);
- .MYD(MYData,存储数据)；
- .MYI(MYIndex，存储索引)；
数据文件和索引文件可放在不同的目录，平均分布IO，获得更快的速度。

## InnoDB
InnoDB存储引擎提供了具有体积、回滚和崩溃恢复能力的事务安全，但对比MyISAM的存储引擎，其写的处理效率差一些，并且会占用更多的磁盘空间以保留数据和索引。
### 自动增长列
InnoDB表的字段增长列可以手工插入，若插入的值是空或者0，则实际插入的是自动增长后的值。

```sql
mysql> create table autoincre_demo
	->(i smallint not null auto_increment,
	name varchar(10),primary key(i)
	)engine=innodb;

mysql> insert into autoincre_dem values(1,'1'),(0,'2').(null,'3');

mysql> select * from autoincre_demo;
```

| i  | name  |
| ------------ | ------------ |
| 1  | 1  |
|  2 | 2  |
| 3  | 3  |

对于InnoDB表，自动增长列必须是索引。如果是组合索引，也必须是组合索引的第一列。

### 外键约束
MySQL支持外键的存储引擎只有InnoDB，在创建外键的时候，要求父表必须由对应的索引，子表在创建外键的时候也会自动创建对应的索引。
例如：country表是父表，country_id为主键索引，city表是子表，country_id字段是外键，对应于country表的主键country_id。
```sql
create table country(
	country_id smallint unsigned not null auto_increment,
	country varchar(50) not null,
	last_update timestamp not null default current_timestamp on update current_timestamp,
	primary key (country_id)
)engine=InnoDB default charset=utf8;

create table city(
	city_id smallint unsigned not null auto_increment,
	city varchar(50) not null,
	country_id smallint unsigned not null,
	last_update timestamp not null default current_timestamp on update current_timestamp,
	primary key (city_id),
	key idx_fx_country_id(country_id),
	constraint 'fk_city_country' foreign key (country_id) regerences country (country_id) on delete  restrict on update cascade
)engine=InnoDB default charset=utf8;
```
在创建索引时，可以指定在删除、更新父表时，对子表进行的相应操作，包括Restrict、cascade、set null和No action。
其中Restrict和no_action相同，是指限制在子表有关联记录的情况下父表不能更新；
CASCADE表示父表在更新或删除时，更新或者删除子表对应记录；
SET NULL则表示父级在更新或删除的时候，子表的对应字段被SET NULL。
例如，对于上面创建的两个表，子表的外键指定的是ON DELETE RESTRICT ON UPDATE CASCADE方式的，那么是在主表删除记录的时候，如果子表有对应记录，则不允许删除，主表在更新记录的时候，如果子表有对应记录，则子表对应更新；

当某个表被其他表创建了外键参照，那么该表的对应索引或者主键禁止被删除。

在导入多个表的数据时，如果需要忽略表之前的导入顺序，可以暂时关闭外键的检查；同样，在执行LOAD DATA和ALTER TABLE操作的时候，可以通过暂时关闭外键约束来加快处理的速度，关闭的命令是
```sql
set foreign_key_checks=0;
```
执行完成后，通过执行：
```sql
set foreign_key_checks=1;
```
改回原来的状态。

对于InnoDB类型的表，外键的信息通过使用show create table或者show table status命令来显示。
```sql
mysql> show table status like 'city' \G
```

### 存储方式
InnoDB存储表和索引有以下两种方式：
 - 使用共享表空间存储，这种方式创建的表的表结构保存在.frm文件中，数据和索引保存在innodb_data_home_dir和innodb_data_file_path定义的表空间中，可以是多个文件。
 - 使用多表空间存储，这种方式创建的表的表结构仍然保存在.frm文件中，但是每个表的数据和索引单独保存在.ibd中。若果是个分区表，则每个分区对应单独的.ibd文件，文件名是“表名+分区名”，可以在创建分区的时候制定每个分区的数据文件的位置，以此来将表的IO均匀分布在多个磁盘上。

# 如何选择合适的存储引擎
 - MyISAM：若应用以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，则选择此存储引擎。MyISAM是在Web、数据仓储和其他应用环境下常用的存储引擎之一。
 - InnODB：用于事务处理应用程序，支持外键。若应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询以外，还包括很多的更新、删除操作，那么InnoDB存储引擎应该是比较合适的选择。对于计费系统或者财务系统等数据准确性比较高的系统，推荐使用。
