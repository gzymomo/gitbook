- [Docker-compose封装mysql并初始化数据以及redis](https://www.cnblogs.com/xiao987334176/p/12669080.html)



# 一、概述

现有一台服务器，需要部署mysql和redis。其中mysql容器，需要在第一次启动时，执行sql文件。

redis保持空数据即可。

# 二、封装mysql

本文使用的mysql 5.7版本，基础镜像为官方的mysql

## 目录结构

```
./
├── docker-compose.yml
└── mysql
    ├── dockerfile
    ├── init
    │   └── test.sql
    └── mysqld.cnf
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
    environment:
      - MYSQL_ROOT_PASSWORD=abcd1234
    ports:
      - "3306:3306"
    restart: always
    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

参数解释：

image：指定服务的镜像名称或镜像ID。如果镜像在本地不存在，Compose将会尝试拉取镜像。

container_name：容器名称，相当于docker run --name xxx，里面的--name参数。

build：指定Dockerfile所在文件夹的路径。Compose将会利用Dockerfile自动构建镜像，然后使用镜像启动服务容器。

volumes：挂载一个目录或者一个已存在的数据卷容器，相当于docker run -v xxx:xxx里面的-v参数。

environment：环境变量，相当于docker run -e xxx=xxx里面的-e参数。

ports：映射端口，相当于docker run -p xx:xx里面的-p参数。

restart：重启方式，相当于docker run --restart里面的--restart参数。

command：覆盖容器启动后默认执行的命令，相当于docker run xxx /bin/bash里面最后一段命令。

 

其实这个 docker-compose，等于命令：

```
docker run -d --restart=always --name mysql -e MYSQL_ROOT_PASSWORD=abcd1234 -p 3306:3306 -v /data/mysql/data:/var/lib/mysql -v ./mysql/init:/docker-entrypoint-initdb.d/  mysql:1 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

 

dockerfile

```
FROM mysql:5.7
ADD mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
```

 

mysqld.cnf

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



test.sql

```mysql
CREATE DATABASE  `test` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
grant all PRIVILEGES on test.* to test@'%' identified by '123456';
flush privileges;
use test;
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



## 生成镜像

```
docker-compose build --no-cache
```

## 启动docker-compose

```
docker-compose up -d
```

## 测试

使用mysql客户端，查看数据是否存在。

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200410094559902-37615680.png)

 

 

# 三、封装redis

本文使用的redis 4.0.11版本，基础镜像为官方的redis

## 目录结构

```
./
├── docker-compose.yml
└── redis
    ├── dockerfile
    └── redis.conf
```

 

docker-compose.yml

```yaml
version: '3'
services:
  redis:
    image: redis:1
    container_name: redis
    build: ./redis
    volumes:
      - /data/redis1:/data
    ports:
      - "6379:6379"
    restart: always
```

 

dockerfile

```
FROM redis:4.0.10
COPY redis.conf /usr/local/etc/redis/redis.conf
EXPOSE 6379
ENTRYPOINT [ "redis-server", "/usr/local/etc/redis/redis.conf"]
```

 

redis.conf

```
port 6379
dir /data
pidfile /data/redis.pid
logfile "/data/redis.log"
repl-disable-tcp-nodelay yes
no-appendfsync-on-rewrite yes
maxmemory 2048m
maxmemory-policy allkeys-lru
requirepass abcd1234
```

注意：默认端口为6379，密码为abcd1234

 

## 生成镜像

```
docker-compose build --no-cache
```

 

## 启动docker-compose

```
docker-compose up -d
```

 

## 测试

进入容器，使用redis-cli测试。

```
# docker exec -it redis /bin/bash
root@5fa954a74ae2:/data# redis-cli 
127.0.0.1:6379> auth abcd1234
OK
127.0.0.1:6379> info
# Server
redis_version:4.0.10
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:3e68f04515f466a2
redis_mode:standalone
os:Linux 3.10.0-957.el7.x86_64 x86_64
...
```

 

本文参考连接：

https://www.cnblogs.com/mmry/p/8812599.html