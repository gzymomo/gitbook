#### 一、硬件相关的参数调优

根据MySQL实例运行的硬件环境，一些硬件相关的参数需要根据实际情况进行设置，主要如下：

##### 1.1 innodb_buffer_pool_size

- 该参数通常设置为总内存大小的50%~70%，充分利用内存缓存，减少换页带来的磁盘性能损耗。
- 该参数设置的大小尽量不要大于数据库的总大小，否则造成内存浪费。
- 在MySQL运行过程中，监控buffer pool的使用率，根据实际情况进行调整。

##### 1.2 innodb_log_file_size

- 该参数通常设置为128M ~ 2G之间。
- 该参数大小应当能够支持最少一个小时的日志量，保证在刷脏页和做检查点操作期间，有足够的空间顺序写日志。

##### 1.3 innodb_flush_log_at_trx_commit

- 该参数设置为1，事务实时落盘，保证数据的持久性，但是影响性能。
- 该参数设置为0或2，性能较高，但是事务无法实时落盘，存在丢失数据的风险。

##### 1.4 sync_binlog

- 该参数设置为1，binlog日志实时落盘，保证数据的持久性，但是影响性能。
- 该参数设置为0，性能较高，但是binlog日志无法实时落盘，存在丢失数据的风险。

##### 1.5 innodb_flush_method

将该参数设置为O_DIRECT将避免双缓冲带来的性能损失，从buffer pool直接往磁盘上写，避免经过操作系统的缓冲带来的性能损耗。

#### 二、参数优化最佳实践

##### 2.1 innodb_file_per_table

innodb_file_per_table设置为ON，为每一个表生成一个独立的表空间。

##### 2.2 innodb_stats_on_metadata

innodb_stats_on_metadata设置为OFF，避免不必要的InnoDB统计信息更新，可大大提高读取速度。

##### 2.3 innodb_buffer_pool_instances

innodb_buffer_pool_instances参数，一个比较好的设置值为8，如果buffer pool size 小于 1G，将该参数设置为1。

##### 2.4 query_cache_type & query_cache_size

这两个参数应当设置为0，禁用查询缓存。在MySQL 8.0 版本，查询缓存功能及参数被整体移除，而5.7及以下版本，应当禁用查询缓存。

##### 2.5 innodb_autoinc_lock_mode

该参数设置为2（交错模式）能够避免auto-inc锁（表级锁）,显著提高性能，前提binlog格式为ROW或者MIXED。

##### 2.6 innodb_io_capacity & innodb_io_capacity_max

在写入很重的场景下，这两个参数将会影响MySQL的写入性能。首先需要了解磁盘IO的性能，即IOPS，可以先使用sysbench测试磁盘IO性能，然后再调整innodb_io_capacity和innodb_io_capacity_max，以便最大程度利用磁盘IO的能力。