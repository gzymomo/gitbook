# MySQL工具Atlas的安装使用

## 1、介绍

`Atlas`是由`Qihoo 360`, `Web`平台部基础架构团队开发维护的一个基于`MySQL`协议的数据中间层项目 它是在`mysql-proxy 0.8.2`版本的基础上，对其进行了优化，增加了一些新的功能特性`360`内部使用`Atlas`运行的`mysql`业务，每天承载的读写请求数达几十亿条 下载地址 ：https://github.com/Qihoo360/Atlas/releases

注意：1、`Atlas`只能安装运行在 64 位的系统上 2、`Centos 5.X`安装`Atlas-XX.el5.x86_64.rpm`，`Centos 6.X/7.X`安装`Atlas-XX.el6.x86_64.rpm`3、后端`mysql`版本应大于`5.1`，建议使用`Mysql 5.6`以上

## 2、安装配置

### 2.1 环境准备

两台服务器

192.168.10.54：MySQL 5.7.30 一主三从，3307-3310 端口

192.168.10.55：Atlas

### 2.2 下载安装 Altas

```
wget https://github.com/Qihoo360/Atlas/releases/download/2.2.1/Atlas-2.2.1.el6.x86_64.rpm
[root@db3 ~]# rpm -ivh Atlas-2.2.1.el6.x86_64.rpm
Preparing...        ####################### [100%]
Updating / installing...
   1:Atlas-2.2.1-1  ####################### [100%]
```

### 2.3 处理配置文件

```
[root@db3 ~]# cd /usr/local/mysql-proxy/conf/
[root@db3 conf]# mv test.cnf test.cnf.bak

[root@db3 conf]# vim test.cnf
[mysql-proxy]
admin-username = user    # 管理用户
admin-password = pwd     # 管理密码
proxy-backend-addresses = 192.168.10.54:3307    # 写节点（主库）
proxy-read-only-backend-addresses = 192.168.10.54:3308,192.168.10.54:3309,192.168.10.54:3310   # 只读节点（从库）
pwds = test:3yb5jEku5h4=,repl:3yb5jEku5h4=      # 业务连接用户密码（逗号分隔），密码为Atlas/bin目录下encrypt命令生成
daemon = true       # 后台运行
keepalive = true    # 监测节点心跳
event-threads = 4   # 并发数量，设置cpu核数一半
log-level = message   # 日志级别
log-path = /usr/local/mysql-proxy/log  # 日志目录
sql-log = On   # sql记录（可做审计）
proxy-address = 0.0.0.0:3306    # 业务连接端口
admin-address = 0.0.0.0:2345    # 管理连接端口
charset=utf8   # 字符集
```

### 2.4 启动服务

```
[root@db3 mysql-proxy]# /usr/local/mysql-proxy/bin/mysql-proxyd test start
OK: MySQL-Proxy of test is started
[root@db3 mysql-proxy]# ps -ef |grep proxy
root     105981      1  0 17:19 ?        00:00:00 /usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/usr/local/mysql-proxy/conf/test.cnf
root     105982 105981  0 17:19 ?        00:00:00 /usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/usr/local/mysql-proxy/conf/test.cnf
root     105995   1249  0 17:19 pts/0    00:00:00 grep --color=auto proxy
```

## 3、读写分离功能测试

测试读操作：

```
mysql -umha -pmha  -h 10.0.0.53 -P 33060
db03 [(none)]>select @@server_id;
```

测试写操作：

```
mysql> begin;select @@server_id;commit;
```

**注意:**`DDL`建议不要在`Atlas`触发，最好是到主库触发`Online DDL`或者`PT-OSC``DML`建议`begin`; `DML`; `commit`;

### 3.1 连接服务

```
[root@db5 conf]# mysql -urepl -p123 -h192.168.10.55 -P3306
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.0.81-log MySQL Community Server (GPL)   # 5.0.81版本
```

### 3.2 只读测试

```
mysql> select @@server_id;
+-------------+
| @@server_id |
+-------------+
|           9 |
+-------------+
1 row in set (0.00 sec)

mysql> select @@server_id;
+-------------+
| @@server_id |
+-------------+
|          10 |
+-------------+
1 row in set (0.00 sec)

mysql> select @@server_id;
+-------------+
| @@server_id |
+-------------+
|           8 |
+-------------+
1 row in set (0.00 sec)
```

### 3.3 写入测试

```
mysql> begin;select @@server_id;commit;
Query OK, 0 rows affected (0.00 sec)

+-------------+
| @@server_id |
+-------------+
|           7 |
+-------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> begin;select @@server_id;commit;
Query OK, 0 rows affected (0.00 sec)

+-------------+
| @@server_id |
+-------------+
|           7 |
+-------------+
1 row in set (0.00 sec)

Query OK, 0 rows affected (0.00 sec)
```

## 4、管理功能简介

### 4.1 持久化配置文件

```
mysql> save config;
Empty set (0.00 sec)
```

### 4.2 连接管理服务

```
[root@db5 bin]# mysql -uuser -ppwd -h192.168.10.55 -P2345
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.0.99-agent-admin
```

### 4.3 节点管理

#### 4.3.1 查看所有节点

```
mysql> SELECT * FROM backends;
+-------------+--------------------+-------+------+
| backend_ndx | address            | state | type |
+-------------+--------------------+-------+------+
|           1 | 192.168.10.54:3307 | up    | rw   |
|           2 | 192.168.10.54:3308 | up    | ro   |
|           3 | 192.168.10.54:3309 | up    | ro   |
|           4 | 192.168.10.54:3310 | up    | ro   |
+-------------+--------------------+-------+------+
4 rows in set (0.00 sec)
```

#### 4.3.2 节点的上线和下线

下线

```
mysql> set offline 1;
+-------------+--------------------+---------+------+
| backend_ndx | address            | state   | type |
+-------------+--------------------+---------+------+
|           1 | 192.168.10.54:3307 | offline | rw   |
+-------------+--------------------+---------+------+
1 row in set (0.00 sec)
mysql> SELECT * FROM backends;
+-------------+--------------------+---------+------+
| backend_ndx | address            | state   | type |
+-------------+--------------------+---------+------+
|           1 | 192.168.10.54:3307 | offline | rw   |
|           2 | 192.168.10.54:3308 | up      | ro   |
|           3 | 192.168.10.54:3309 | up      | ro   |
|           4 | 192.168.10.54:3310 | up      | ro   |
+-------------+--------------------+---------+------+
4 rows in set (0.00 sec)
```

上线

```
mysql> set online 1;
+-------------+--------------------+---------+------+
| backend_ndx | address            | state   | type |
+-------------+--------------------+---------+------+
|           1 | 192.168.10.54:3307 | unknown | rw   |
+-------------+--------------------+---------+------+
1 row in set (0.00 sec)
```

#### 4.3.3 添加删除节点

删除

```
mysql> remove backend 4;
Empty set (0.00 sec)

mysql> SELECT * FROM backends;
+-------------+--------------------+-------+------+
| backend_ndx | address            | state | type |
+-------------+--------------------+-------+------+
|           1 | 192.168.10.54:3307 | up    | rw   |
|           2 | 192.168.10.54:3308 | up    | ro   |
|           3 | 192.168.10.54:3309 | up    | ro   |
+-------------+--------------------+-------+------+
3 rows in set (0.00 sec)
```

添加

```
mysql> add slave 192.168.10.54:3310;
Empty set (0.00 sec)

mysql> SELECT * FROM backends;
+-------------+--------------------+-------+------+
| backend_ndx | address            | state | type |
+-------------+--------------------+-------+------+
|           1 | 192.168.10.54:3307 | up    | rw   |
|           2 | 192.168.10.54:3308 | up    | ro   |
|           3 | 192.168.10.54:3309 | up    | ro   |
|           4 | 192.168.10.54:3310 | up    | ro   |
+-------------+--------------------+-------+------+
4 rows in set (0.00 sec)
```

### 4.4 用户管理

#### 4.4.1 在主库增加数据库用户

```
mysql> grant all on *.* to user1@'192.168.10.%' identified by '123';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

#### 4.4.2 查看当前用

```
mysql> select * from pwds;
+----------+--------------+
| username | password     |
+----------+--------------+
| test     | 3yb5jEku5h4= |
| repl     | 3yb5jEku5h4= |
+----------+--------------+
2 rows in set (0.00 sec)
```

#### 4.4.3 增加 Atlas 用户

```
mysql> add pwd user1:123;
Empty set (0.00 sec)

mysql> select * from pwds;
+----------+--------------+
| username | password     |
+----------+--------------+
| test     | 3yb5jEku5h4= |
| repl     | 3yb5jEku5h4= |
| user1    | 3yb5jEku5h4= |
+----------+--------------+
3 rows in set (0.00 sec)
```