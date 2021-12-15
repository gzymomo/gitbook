

博客园：

- [陈彦斌](https://www.cnblogs.com/chenyanbin/)：[Mysql数据安全备份](https://www.cnblogs.com/chenyanbin/p/14020293.html)

- [Grey Zeng](https://home.cnblogs.com/u/greyzeng/):[Linux下MySQL数据库的备份与恢复](https://www.cnblogs.com/greyzeng/p/14139141.html)





# 数据安全备份的意义

1. 在出现意外的时候(硬盘损坏、断点、黑客攻击)，以便数据的恢复
2. 导出生产的数据以便研发人员或者测试人员测试学习
3. 高权限的人员那操作失误导致数据丢失，以便恢复

## 备份类型

- 完全备份：对整个数据库的备份
- 部分备份：对数据进行部分备份(一张或多张表)
  - 增量备份：是以上一次备份为基础来备份变更数据
  - 差异备份：是以第一次完全备份为基础来备份变更数据

## 备份方式

- 逻辑备份：直接生成sql语句，在恢复数据的时候执行sql语句
- 物理备份：复制相关库文件，进行数据备份(**my.cnf指向的数据存放目录**)

### 区别

1. 逻辑备份效率低，恢复数据效率低，节约空间
2. 物理备份浪费空间，备份数据效率快

## 备份场景

- 热备份：备份时，不影响数据库的读写操作
- 温备份：备份时，可以读，不能写
- 冷备份：备份时，关闭mysql服务，不能进行任何读写操作

# Mysqldump备份(跨机器)

## 单库语法

```mysql
备份基础语法：
mysqldump -u用户 -hip -p密码 数据库名 表名 | 压缩方式 > 绝对路径+文件名
```

## 跨机器备份

```
跨机器备份：

备份描述：mac本上安装了mysql数据库(172.20.10.2)，使用自搭Linux(172.20.10.4)机器上的mysql备份mac本上的nba库，并使用压缩文件方式，备份至：/mysql_data_back下
来到linux的mysql安装目录(172.20.10.4)：
创建目录：mkdir /mysql_data_back
切换：cd /usr/local/mysql/bin
备份：./mysqldump -uroot -proot -h172.20.10.2 nba | gzip > /mysql_data_back/nba.sql.gz
```

## 本机备份

```
本机备份：

备份描述：linux(自搭)，备份本就上的db1库，并使用压缩方式，备份至：/mysql_data_back下
备份：./mysqldump -uroot -proot db1 | gzip > /mysql_data_back/db1.sql.gz
```

## 备份库中的某张表

```
语法：./mysqldump -uroot -proot -h172.20.10.2 nba | gzip > /mysql_data_back/nba.sql.gz

备份描述：在Linux(自搭，172.20.10.4)上，远程备份mac本(172.20.10.2)nba(库)的nba_player(表)
在原来的基础上，nba(库名) 在追加表名即可
./mysqldump -uroot -proot -h172.20.10.2 nba nba_player | gzip > /mysql_data_back/nba-nba_player.sql.gz
```

## 备份多库

```
语法：./mysqldump -u用户 -p密码 --databases 库1 库2 | gzip > 绝对路径+文件名
备份描述：备份本机的：db1、db2两个库
备份：./mysqldump -uroot -proot --databases db1 db2 | gzip > /mysql_data_back/db1-db2.sql.gz
```

### 注意

　　只备份表结构，数据没备份！

## 备份全库

　　描述：如果远程服务器上数据库较多的话，可以使用全库备份

```
语法：
./mysqldump -uroot -proot -all --databases | gzip > /mysql_data_back/all.sql.gz
```



# Mysql数据的恢复

备份的数据，不加--databases是没有创建库语句的！

```
先备份数据：
./mysqldump -uroot -proot --databases db1 | gzip > /mysql_data_back/db1.sql.gz

删除库：
drop database db1;

还原数据：
2、解压gz文件：gunzip -d db1.sql.gz
1、登录数据库：mysql -uroot -proot -h 127.0.0.1 < /mysql_data_back/db1.sql
```

也可以指定数据库后，在恢复数据

```
语法：
    mysql -u用户 -p密码 -h ip地址 数据库 < 绝对路径+文件名
示例：mysql -uroot -proot -h 127.0.0.1 issdb_1 < /mysql_data_back/issdb_1.sql
```

# 查看Mysql数据库源文件

## 方式一

```
登录mysql：mysql -uroot -proot
查看：show variables like 'datadir%';
===========================
mysql> show variables like 'datadir%';
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| datadir       | /usr/local/mysql/data/ |
+---------------+------------------------+
1 row in set (0.01 sec)
```

## 方式二

　　直接查看my.cnf文件即可

[![img](https://img2020.cnblogs.com/blog/1504448/202011/1504448-20201122214031000-1184156530.png)](https://img2020.cnblogs.com/blog/1504448/202011/1504448-20201122214031000-1184156530.png) 