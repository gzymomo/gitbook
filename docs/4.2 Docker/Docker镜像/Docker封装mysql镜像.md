- [docker封装mysql镜像](https://www.cnblogs.com/xiao987334176/p/11984692.html)



# 一、概述

直接使用官方的镜像

```
docker pull mysql:5.7
```

但是mysqld.cnf并没有优化，还是默认的。

 

# 二、封装镜像

创建目录

```
# dockerfile目录
mkdir -p /opt/dockerfile/mysql
# 持久化目录
mkdir -p /data/mysql/data
```

 

/opt/dockerfile/mysql 目录结构如下：

```
./
├── dockerfile
├── mysqld.cnf
└── run.sh
```

 

## dockerfile

```
FROM mysql:5.7
ADD mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf
```

 

## mysqld.cnf

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```yaml
[client]
port=3306
socket = /var/lib/mysql/mysqld.sock
default-character-set = utf8mb4
[mysql]
no-auto-rehash
auto-rehash
default-character-set = utf8mb4
[mysqld]
###basic settings
server-id = 2
pid-file    = /var/lib/mysql/mysqld.pid
socket        = /var/lib/mysql/mysqld.sock
datadir        = /var/lib/mysql
#log-error    = /var/lib/mysql/error.log
# By default we only accept connections from localhost
#bind-address    = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character_set_server = utf8mb4
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
general_log = OFF
slow_query_log = ON
slow_query_log_file=/var/lib/mysql/slowquery.log
long_query_time=1
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

 

## run.sh

```bash
#!/bin/bash
docker run -d --name mysqld_prod --restart=always -e MYSQL_ROOT_PASSWORD=123456  -p 3306:3306 -v /data/mysql/data:/var/lib/mysql mysqld_prod:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

 

## 生成镜像

```
cd /opt/dockerfile/mysql
docker build -t mysqld_prod:5.7 .
```

 

## 启动镜像

```
bash run.sh
```