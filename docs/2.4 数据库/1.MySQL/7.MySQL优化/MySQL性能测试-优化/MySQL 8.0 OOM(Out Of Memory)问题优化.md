##### 个人网站的软硬件配置：

- CentOS 7 1核1G
- MySQL 8.0.17
- PHP 7

一般来说进程 OOM，主要原因就是内存不足，MySQL 从 5.1 到 5.5， 5.6， 5.7 以及目前的 8.0版本，功能越来越强大，性能也越来越高，但是占用的内存也越来越大，因此优化的关键点在于减少 MySQL 的内存占用。

##### 优化一：innodb_buffer_pool_size 设置为最小128M

innodb_buffer_pool_size 是MySQL内存占用的大户，首先拿它开刀，设置为最小的128M。在我的 1G 内存的云主机上，设置为128M之后，MySQL 8.0 仍然会小概率OOM，没办法，继续优化。

##### 优化二：关闭performance_schema

performance_schema 设置为 OFF，即关闭performance_schema 中的各项性能监控。关闭之后，MySQL 8.0 运行一周多时间，没有发生 OOM。performance_schema 功能在带给我们更多性能监控手段的同时，也占用了太多的内存，看来它是引起 MySQL OOM 的主要原因。MySQL Bug 列表中有一个关于 performance_schema 的 Bug，它会占用大量内存，导致 MySQL 进程 OOM，所以如果内存不大，可以考虑将其关闭，或者升级到最新的 MySQL 版本。

##### 优化三：调低各种buffer,cache参数

各种buffer, cache 参数，尽量设置的小一点，反正个人网站，访问量也不高，并不需要太高的MySQL性能，只要不经常OOM，能稳定运行就好。

比如：

- innodb_log_buffer_size
- innodb_sort_buffer_size
- join_buffer_size
- key_buffer_size
- myisam_sort_buffer_size
- sort_buffer_size
- binlog_cache_size

等等

##### 优化四：关闭不需要的功能

比如我的个人网站，MySQL 单实例，不需要主从复制，可以关闭 mysql 的binlog功能，从而降低内存使用，提高性能。在 my.cnf 配置文件里加上skip-log-bin 参数即可。其他用不到的功能，都可以关闭，以降低内存使用，提高性能。

##### 优化五：优化SQL

低效的 SQL ，尤其是执行计划中带 filesort 的 SQL，在高并发下，会占用大量的内存。可以通过添加索引，优化表结构，优化 SQL 语句，优化业务逻辑等等，减少慢 SQL 的产生，进而减少内存的占用。