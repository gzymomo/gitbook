- [docker mysql初始化多个sql脚本](https://www.cnblogs.com/xiao987334176/p/12721498.html)



# 一、概述

现有一台服务器，需要部署mysql。其中mysql容器，需要在第一次启动时，执行多个sql文件。

| 文件名    | 说明               | 执行顺序 |
| --------- | ------------------ | -------- |
| init.sql  | 创建数据库以及用户 | 1        |
| users.sql | 用户表             | 2        |
| role.sql  | 用户角色           | 3        |

**注意：必须严格按照执行顺序来执行，不能错乱。**

有些人可能会问：为啥不把这3个sql文件合并成1个sql？答案是可以的。假设有上万个用户，那么这个sql文件就会很大，后期维护不方便。

那么可不可以让一个sql文件，执行另外3个sql文件呢？答案是可以的。这样就可以控制sql文件的执行顺序。比如：

```mysql
source /opt/sql/init.sql;
use test;
source /opt/sql/users.sql;
source /opt/sql/role.sql;
```

# 二、容器演示

## 2.1 环境说明

操作系统：centos 7.6

docker版本： 19.03.8

docker-compose版本： 1.24.1

## 2.2 目录结构

/opt/mysql_test 目录结构如下：

```
./
├── docker-compose.yml
└── mysql
    ├── dockerfile
    ├── init
    │   └── init.sql
    ├── mysqld.cnf
    └── sql
        ├── init.sql
        ├── role.sql
        └── users.sql 
```

docker-compose.yml

```yaml
version: '3'
services:
  mysql:
    image: mysql:1
    container_name: mysql
    build: ./mysql
    volumes:
      - /data/mysql/data:/var/lib/mysql
      - ./mysql/init:/docker-entrypoint-initdb.d/
      - ./mysql/sql:/opt/sql
    environment:
      - MYSQL_ROOT_PASSWORD=abcd1234
    ports:
      - "3306:3306"
    restart: always
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

说明：将sql文件放到容器中的 /docker-entrypoint-initdb.d/ 目录，就会在mysql第一次启动时执行。之后重启容器不会重复执行！

如果此目录放置多个sql文件，它执行时是没有顺序的。因此，这个目录只放一个init.sql，专门用来控制执行sql顺序的。 

mysql/dockerfile

```
FROM mysql:5.7
ADD mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
```

mysql/init/init.sql

```
source /opt/sql/init.sql;
use test;
source /opt/sql/users.sql;
source /opt/sql/role.sql;
```

mysql/mysqld.cnf

```yaml
[client]
port=3306
socket = /var/run/mysqld/mysqld.sock
[mysql]
no-auto-rehash
auto-rehash
default-character-set=utf8mb4
[mysqld]
###basic settings
server-id = 2
pid-file    = /var/run/mysqld/mysqld.pid
socket        = /var/run/mysqld/mysqld.sock
datadir        = /var/lib/mysql
#log-error    = /var/lib/mysql/error.log
# By default we only accept connections from localhost
#bind-address    = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server = utf8mb4
sql_mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
default-storage-engine=INNODB
transaction_isolation = READ-COMMITTED
auto_increment_offset = 1
connect_timeout = 20
max_connections = 3500
wait_timeout=86400
interactive_timeout=86400
interactive_timeout = 7200
log_bin_trust_function_creators = 1
wait_timeout = 7200
sort_buffer_size = 32M
join_buffer_size = 128M
max_allowed_packet = 1024M
tmp_table_size = 2097152
explicit_defaults_for_timestamp = 1
read_buffer_size = 16M
read_rnd_buffer_size = 32M
query_cache_type = 1
query_cache_size = 2M
table_open_cache = 1500
table_definition_cache = 1000
thread_cache_size = 768
back_log = 3000
open_files_limit = 65536
skip-name-resolve
########log settings########
log-output=FILE
general_log = ON
general_log_file=/var/lib/mysql/general.log
slow_query_log = ON
slow_query_log_file=/var/lib/mysql/slowquery.log
long_query_time=10
#log-error=/var/lib/mysql/error.log
log_queries_not_using_indexes = OFF
log_throttle_queries_not_using_indexes = 0
#expire_logs_days = 120
min_examined_row_limit = 100
########innodb settings########
innodb_io_capacity = 4000
innodb_io_capacity_max = 8000
innodb_buffer_pool_size = 6144M
innodb_file_per_table = on
innodb_buffer_pool_instances = 20
innodb_buffer_pool_load_at_startup = 1
innodb_buffer_pool_dump_at_shutdown = 1
innodb_log_file_size = 300M
innodb_log_files_in_group = 2 
innodb_log_buffer_size = 16M
innodb_undo_logs = 128
#innodb_undo_tablespaces = 3
#innodb_undo_log_truncate = 1
#innodb_max_undo_log_size = 2G
innodb_flush_method = O_DIRECT
innodb_flush_neighbors = 1
innodb_purge_threads = 4
innodb_large_prefix = 1
innodb_thread_concurrency = 64
innodb_print_all_deadlocks = 1
innodb_strict_mode = 1
innodb_sort_buffer_size = 64M
innodb_flush_log_at_trx_commit=1
innodb_autoextend_increment=64
innodb_concurrency_tickets=5000
innodb_old_blocks_time=1000
innodb_open_files=65536
innodb_stats_on_metadata=0
innodb_file_per_table=1
innodb_checksum_algorithm=0
#innodb_data_file_path=ibdata1:60M;ibdata2:60M;autoextend:max:1G
innodb_data_file_path = ibdata1:12M:autoextend
#innodb_temp_data_file_path = ibtmp1:500M:autoextend:max:20G
#innodb_buffer_pool_dump_pct = 40
#innodb_page_cleaners = 4
#innodb_purge_rseg_truncate_frequency = 128
binlog_gtid_simple_recovery=1
#log_timestamps=system
##############
delayed_insert_limit = 100
delayed_insert_timeout = 300
delayed_queue_size = 1000
delay_key_write = ON
disconnect_on_expired_password = ON
div_precision_increment = 4
end_markers_in_json = OFF
eq_range_index_dive_limit = 10
innodb_adaptive_flushing = ON
innodb_adaptive_hash_index = ON
innodb_adaptive_max_sleep_delay = 150000
#innodb_additional_mem_pool_size = 2097152
innodb_autoextend_increment = 64
innodb_autoinc_lock_mode = 1
```

mysql/sql/init.sql 

```mysql
-- 创建数据库
CREATE DATABASE  `test` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
-- 创建普通用户
grant all PRIVILEGES on test.* to test@'%' identified by '123456';
flush privileges;
use test;
```

mysql/sql/users.sql

 ```mysql
CREATE TABLE `users` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) COLLATE utf8mb4_bin NOT NULL COMMENT '用户名',
  `password` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '密码',
  `phone` varchar(20) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '手机号',
  `email` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '邮箱',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

INSERT INTO `test`.`users` (`id`, `username`, `password`, `phone`, `email`, `create_time`) VALUES ('1', 'xiao', '123', '12345678910', '123@qq.com', '2020-04-10 01:22:07');
 ```

mysql/sql/role.sql

```mysql
CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL COMMENT '角色名称，显示用',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '更新时间',
  `status` tinyint(1) DEFAULT '1' COMMENT '是否失效，1-有效，0-失效',
  `version` int(11) DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO `test`.`role` (`id`, `name`, `create_time`, `update_time`, `status`, `version`) VALUES ('1', 'admin', '2020-04-17 09:35:48', '2020-04-17 09:35:48', '1', '1');
```

## 2.3 构建镜像

```
cd /opt/mysql_test
docker-compose build
```

## 2.4 运行

```
docker-compose up -d
```

查看日志

```
docker logs -f mysql
```

输出：

```
...
2020-04-21 07:29:05+00:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/init.sql
...

Version: '5.7.29-log'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
2020-04-21T07:29:09.473241Z 0 [Note] InnoDB: Buffer pool(s) load completed at 200421  7:29:09
```

可以发现，执行了初始化脚本。

## 2.5 查看数据

用户表

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200421153111120-124282194.png)

 

角色表

 ![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200421153126197-1671510902.png)

 

