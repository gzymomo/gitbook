# sysbench测试

## **sysbench对数据库进行压力测试的过程**：

1. prepare 阶段 这个阶段是用来做准备的、比较说建立好测试用的表、并向表中填充数据。

2. run    阶段 这个阶段是才是去跑压力测试的SQL
3. cleanup 阶段 这个阶段是去清除数据的、也就是prepare阶段初始化好的表要都drop掉

## **sysbench 中的测试类型大致可以分成内置的，lua脚本自定义的测试：**

　　1、内置：

　　　　fileio 、cpu 、memory 、threads 、 mutex 

　　2、lua脚本自定义型：

　　　　sysbench 自身内涵了一些测试脚本放在了安装目录下的：

```bash
[jianglexing@cstudio sysbench]$ ll share/sysbench
总用量 60
-rwxr-xr-x. 1 root root  1452 10月 17 15:18 bulk_insert.lua
-rw-r--r--. 1 root root 13918 10月 17 15:18 oltp_common.lua
-rwxr-xr-x. 1 root root  1290 10月 17 15:18 oltp_delete.lua
-rwxr-xr-x. 1 root root  2415 10月 17 15:18 oltp_insert.lua
-rwxr-xr-x. 1 root root  1265 10月 17 15:18 oltp_point_select.lua
-rwxr-xr-x. 1 root root  1649 10月 17 15:18 oltp_read_only.lua
-rwxr-xr-x. 1 root root  1824 10月 17 15:18 oltp_read_write.lua
-rwxr-xr-x. 1 root root  1118 10月 17 15:18 oltp_update_index.lua
-rwxr-xr-x. 1 root root  1127 10月 17 15:18 oltp_update_non_index.lua
-rwxr-xr-x. 1 root root  1440 10月 17 15:18 oltp_write_only.lua
-rwxr-xr-x. 1 root root  1919 10月 17 15:18 select_random_points.lua
-rwxr-xr-x. 1 root root  2118 10月 17 15:18 select_random_ranges.lua
drwxr-xr-x. 4 root root    46 10月 17 15:18 tests
```



## IO测试

```bash
[root@test3 ~]# sysbench fileio help               #查看IO测试的文档
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

fileio options:
  --file-num=N                  number of files to create [128]              #文件的数量
  --file-block-size=N           block size to use in all IO operations [16384] #文件块的大小，如果要是针对INNODB的测试，可以设置为innodb_page_size的大小
  --file-total-size=SIZE        total size of files to create [2G]             #文件的总大小
  --file-test-mode=STRING       test mode {seqwr【顺序写】, seqrewr【顺序读写】, seqrd【顺序读】, rndrd【随机读】, rndwr【随机写】, rndrw【随机读写】} #文件测试模式
  --file-io-mode=STRING         file operations mode {sync【同步】,async【异步】,mmap【map映射】} [默认为：sync]          #文件的io模式
  --file-async-backlog=N        number of asynchronous operatons to queue per thread [128] #打开文件时的选项，这是与API相关的参数。
  --file-extra-flags=[LIST,...] #打开文件时的选项，这是与API相关的参数。可选有sync，dsync，direct。--file-fsync-freq=N           #执行fsync函数的频率，fsync主要是同步磁盘文件，因为可能有系统和磁盘缓冲的关系。默认为100，如果为0表示不使用fsync。
  --file-fsync-all[=on|off]     #每执行完一次写操作，就执行一次fsync，默认未off。--file-fsync-end[=on|off]     #在测试结束时，执行fsync，默认为on。--file-fsync-mode=STRING      #文件同步函数的选择，同样是和API相关的参数，由于多个操作对fdatasync支持的不同，因此不建议使用fdatasync。默认为fsync。--file-merged-requests=N      #尽可能合并此数量的io请求(0-不合并)，默认为[0]。
  --file-rw-ratio=N             #测试时的读写比例，默认是2:1。
```

在使用sysbench进行测试的时候，通常分为三个步骤prepare,run,cleanup阶段。

第一步准备数据（prepare阶段）：

```bash
[root@test3 systext]# sysbench fileio --file-num=10 --file-total-size=50G prepare
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

10 files, 5242880Kb each, 51200Mb total
Creating files for the test...
Extra file open flags: (none)
Creating file test_file.0
Creating file test_file.1
Creating file test_file.2
Creating file test_file.3
Creating file test_file.4
Creating file test_file.5
Creating file test_file.6
Creating file test_file.7
Creating file test_file.8
Creating file test_file.9
53687091200 bytes written in 489.55 seconds (104.59 MiB/sec).
#这里给出一个每秒写入的数据量104.59MB/s, 这里的写入是顺序写入的，表示磁盘的吞吐量为104.59MB/s。
【一般对顺序的读写称为吞吐量，对随机的IO使用IOPS来表示】
[root@test3 systext]# ll -h      #文件大小为5个G  
total 50G
-rw------- 1 root root 5.0G Nov 27 09:30 test_file.0
-rw------- 1 root root 5.0G Nov 27 09:31 test_file.1
-rw------- 1 root root 5.0G Nov 27 09:32 test_file.2
-rw------- 1 root root 5.0G Nov 27 09:32 test_file.3
-rw------- 1 root root 5.0G Nov 27 09:33 test_file.4
-rw------- 1 root root 5.0G Nov 27 09:34 test_file.5
-rw------- 1 root root 5.0G Nov 27 09:35 test_file.6
-rw------- 1 root root 5.0G Nov 27 09:36 test_file.7
-rw------- 1 root root 5.0G Nov 27 09:36 test_file.8
-rw------- 1 root root 5.0G Nov 27 09:37 test_file.9
```

数据准备好之后，进行测试：

```bash
#这里进行随机读写测试[root@test3 systext]# sysbench fileio --file-num=10 --file-total-size=50G --file-block-size=16384 --file-test-mode=rndrw --file-io-mode=sync --file-extra-flags=direct --time=100  --threads=16 --report-interval=10 run
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Running the test with following options:     #设定的一些参数数值
Number of threads: 16
Report intermediate results every 10 second(s)
Initializing random number generator from current time


Extra file open flags: directio
10 files, 5GiB each
50GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!

[ 10s ] reads: 3.24 MiB/s writes: 2.16 MiB/s fsyncs: 34.08/s latency (ms,95%): 80.025       #每隔10s输出一次报告
[ 20s ] reads: 3.49 MiB/s writes: 2.32 MiB/s fsyncs: 36.70/s latency (ms,95%): 73.135
[ 30s ] reads: 3.45 MiB/s writes: 2.29 MiB/s fsyncs: 37.00/s latency (ms,95%): 75.817
[ 40s ] reads: 3.43 MiB/s writes: 2.29 MiB/s fsyncs: 36.00/s latency (ms,95%): 75.817
[ 50s ] reads: 3.57 MiB/s writes: 2.38 MiB/s fsyncs: 37.40/s latency (ms,95%): 73.135
[ 60s ] reads: 3.08 MiB/s writes: 2.06 MiB/s fsyncs: 32.30/s latency (ms,95%): 86.002
[ 70s ] reads: 3.41 MiB/s writes: 2.27 MiB/s fsyncs: 36.40/s latency (ms,95%): 75.817
[ 80s ] reads: 3.47 MiB/s writes: 2.31 MiB/s fsyncs: 36.20/s latency (ms,95%): 73.135
[ 90s ] reads: 3.46 MiB/s writes: 2.31 MiB/s fsyncs: 36.20/s latency (ms,95%): 77.194
[ 100s ] reads: 3.10 MiB/s writes: 2.07 MiB/s fsyncs: 33.50/s latency (ms,95%): 75.817

Throughput:
         read:  IOPS=215.57 3.37 MiB/s (3.53 MB/s)    #通常的机械磁盘随机IOPS也就是200多一点。
         write: IOPS=143.72 2.25 MiB/s (2.35 MB/s)    #随机写入的速度明显要低很多。
         fsync: IOPS=37.13

Latency (ms):
         min:                                  0.08
         avg:                                 40.51
         max:                               1000.31
         95th percentile:                     77.19
         sum:                            1601329.71#随机读大概是2.10M/s,文件块的大小为16KB,可以大概估计磁盘转速： 2.10*1024KB*60s/16KB=7560n/m, 大概就是7500转每分
```

```bash
[root@test3 systext]# sysbench fileio --file-num=10 --file-total-size=50G --file-block-size=16384 --file-test-mode=seqrd --file-io-mode=sync --file-extra-flags=direct --time=100  --threads=16 --report-interval=10 run
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 16
Report intermediate results every 10 second(s)
Initializing random number generator from current time


Extra file open flags: directio
10 files, 5GiB each
50GiB total file size
Block size 16KiB
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing sequential read test
Initializing worker threads...

Threads started!

[ 10s ] reads: 98.88 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.020
[ 20s ] reads: 98.64 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.681
[ 30s ] reads: 93.24 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 2.913
[ 40s ] reads: 89.12 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 4.028
[ 50s ] reads: 93.17 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 4.487
[ 60s ] reads: 91.98 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 4.652
[ 70s ] reads: 97.08 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.425
[ 80s ] reads: 93.71 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.020
[ 90s ] reads: 94.63 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.304
[ 100s ] reads: 89.57 MiB/s writes: 0.00 MiB/s fsyncs: 0.00/s latency (ms,95%): 3.364

Throughput:
         read:  IOPS=6016.01 94.00 MiB/s (98.57 MB/s)
         write: IOPS=0.00 0.00 MiB/s (0.00 MB/s)
         fsync: IOPS=0.00

Latency (ms):
         min:                                  0.40
         avg:                                  2.66
         max:                                687.00
         95th percentile:                      3.62
         sum:                            1599247.42

#测试结果可以看到顺序的读和随机读的差距还是超大的

顺序读的测试
```

可以更改--file-test-mode的模式，改变测试的模式。

测试阶段完成之后，需要进行最后的cleanup阶段，

```bash
[root@test3 systext]# sysbench fileio --file-num=10 --file-total-size=50 cleanup
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Removing test files...
[root@test3 systext]# ls
[root@test3 systext]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda3        29G  8.4G   20G  31% /
tmpfs           3.9G   44K  3.9G   1% /dev/shm
/dev/vda1       190M   30M  151M  17% /boot
/dev/vdb        100G   25G   76G  25% /data
cgroup_root     3.9G     0  3.9G   0% /cgroup
#看到磁盘空间已经释放
```

## 测试MySQL的OLTP

测试数据库时的一些参数：

```bash
General database options:

  --db-driver=STRING  specifies database driver to use ('help' to get list of available drivers) [mysql] #指定数据库驱动，默认是mysql
  --db-ps-mode=STRING prepared statements usage mode {auto, disable} [auto]                              #
  --db-debug[=on|off] print database-specific debug information [off]                                    #dubug模式


Compiled-in database drivers:
  mysql - MySQL driver

mysql options:
  --mysql-host=[LIST,...]          MySQL server host [localhost]
  --mysql-port=[LIST,...]          MySQL server port [3306]
  --mysql-socket=[LIST,...]        MySQL socket
  --mysql-user=STRING              MySQL user [sbtest]
  --mysql-password=STRING          MySQL password []
  --mysql-db=STRING                MySQL database name [sbtest]              #数据库名字，默认是sbtest
  --mysql-ssl[=on|off]             use SSL connections, if available in the client library [off]  #以下是ssl的连接测试
  --mysql-ssl-key=STRING           path name of the client private key file
  --mysql-ssl-ca=STRING            path name of the CA file
  --mysql-ssl-cert=STRING          path name of the client public key certificate file
  --mysql-ssl-cipher=STRING        use specific cipher for SSL connections []
  --mysql-compression[=on|off]     use compression, if available in the client library [off]      #压缩测试
  --mysql-debug[=on|off]           trace all client library calls [off]
  --mysql-ignore-errors=[LIST,...] list of errors to ignore, or "all" [1213,1020,1205]            #忽略的错误
  --mysql-dry-run[=on|off]         Dry run, pretend that all MySQL client API calls are successful without executing them [off]
```

MySQL测试的lua脚本：

```lua
#因为是源码安装，索引目录在这里
[root@test3 lua]# pwd
/data/sysbench-master/src/lua
[root@test3 lua]# ls
bulk_insert.lua  Makefile     oltp_common.lua  oltp_point_select.lua  oltp_update_index.lua      prime-test.lua
empty-test.lua   Makefile.am  oltp_delete.lua  oltp_read_only.lua     oltp_update_non_index.lua  select_random_points.lua
internal         Makefile.in  oltp_insert.lua  oltp_read_write.lua    oltp_write_only.lua        select_random_ranges.lua
#根据脚本的名字可以选择对应的基本#查看某个lua脚本的用法
[root@test3 lua]# sysbench oltp_common.lua help  sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)
oltp_common.lua options:  
--auto_inc[=on|off]           Use AUTO_INCREMENT column as Primary Key (for MySQL), or its alternatives in other DBMS. When disabled, use client-generated IDs [on]  
--create_secondary[=on|off]   Create a secondary index in addition to the PRIMARY KEY [on]  
--create_table_options=STRING Extra CREATE TABLE options []  
--delete_inserts=N            Number of DELETE/INSERT combinations per transaction [1]  
--distinct_ranges=N           Number of SELECT DISTINCT queries per transaction [1]  
--index_updates=N             Number of UPDATE index queries per transaction [1]  
--mysql_storage_engine=STRING Storage engine, if MySQL is used [innodb]  
--non_index_updates=N         Number of UPDATE non-index queries per transaction [1]  
--order_ranges=N              Number of SELECT ORDER BY queries per transaction [1]  
--pgsql_variant=STRING        Use this PostgreSQL variant when running with the PostgreSQL driver. The only currently supported variant is 'redshift'. When enabled, create_secondary is automatically disabled, and delete_inserts is set to 0  --point_selects=N             Number of point SELECT queries per transaction [10]  
--range_selects[=on|off]      Enable/disable all range SELECT queries [on]  
--range_size=N                Range size for range SELECT queries [100]  
--secondary[=on|off]          Use a secondary index in place of the PRIMARY KEY [off]  
--simple_ranges=N             Number of simple range SELECT queries per transaction [1]  
--skip_trx[=on|off]           Don't start explicit transactions and execute all queries in the AUTOCOMMIT mode [off]  
--sum_ranges=N                Number of SELECT SUM() queries per transaction [1]  
--table_size=N                Number of rows per table [10000]  --tables=N                    Number of tables [1]
```

### prepare阶段：

创建默认的测试库：

```mysql
mysql> create database sbtest;      #创建数据库
Query OK, 1 row affected (0.11 sec)

#准备数据，时间比较长，可以把table_size设置的小一点
[root@test3 lua]# sysbench /data/sysbench-master/src/lua/oltp_read_write.lua --tables=3 --table_size=10000000 --mysql-user=root --mysql-password=123456 --mysql-host=10.0.102.214 --mysql-port=3306 --mysql-db=sbtest prepare
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Creating table 'sbtest1'...
Inserting 10000000 records into 'sbtest1'
Creating a secondary index on 'sbtest1'...
Creating table 'sbtest2'...
Inserting 10000000 records into 'sbtest2'
Creating a secondary index on 'sbtest2'...
Creating table 'sbtest3'...
Inserting 10000000 records into 'sbtest3'
Creating a secondary index on 'sbtest3'...
#在MySQL  shel1中查看数据
mysql> select count(*) from sbtest1;+----------+| count(*) |+----------+| 10000000 |+----------+1 row in set (1.89 sec)mysql> show tables;+------------------+| Tables_in_sbtest |+------------------+| sbtest1          || sbtest2          || sbtest3          |+------------------+3 rows in set (0.00 sec)
```

### run阶段

选择一个合适的lua脚本进行测试：

```bash
[root@test3 lua]# sysbench /data/sysbench-master/src/lua/oltp_point_select.lua --tables=3 --table_size=10000000 --mysql-user=root --mysql-password=123456 --mysql-host=10.0.102.214 --mysql-port=3306 --mysql-db=sbtest --threads=128 --time=100 --report-interval=5 run
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 128
Report intermediate results every 5 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 5s ] thds: 128 tps: 15037.47 qps: 15037.47 (r/w/o: 15037.47/0.00/0.00) lat (ms,95%): 41.10 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 128 tps: 18767.43 qps: 18767.43 (r/w/o: 18767.43/0.00/0.00) lat (ms,95%): 46.63 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 128 tps: 22463.68 qps: 22463.68 (r/w/o: 22463.68/0.00/0.00) lat (ms,95%): 40.37 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 128 tps: 26848.42 qps: 26848.42 (r/w/o: 26848.42/0.00/0.00) lat (ms,95%): 28.67 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 128 tps: 27005.57 qps: 27005.57 (r/w/o: 27005.57/0.00/0.00) lat (ms,95%): 15.00 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 128 tps: 26965.62 qps: 26965.62 (r/w/o: 26965.62/0.00/0.00) lat (ms,95%): 1.82 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 128 tps: 27626.74 qps: 27626.74 (r/w/o: 27626.74/0.00/0.00) lat (ms,95%): 0.42 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 128 tps: 27244.27 qps: 27244.27 (r/w/o: 27244.27/0.00/0.00) lat (ms,95%): 0.33 err/s: 0.00 reconn/s: 0.00
[ 45s ] thds: 128 tps: 26522.56 qps: 26522.56 (r/w/o: 26522.56/0.00/0.00) lat (ms,95%): 1.42 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 128 tps: 26791.43 qps: 26791.43 (r/w/o: 26791.43/0.00/0.00) lat (ms,95%): 5.57 err/s: 0.00 reconn/s: 0.00
[ 55s ] thds: 128 tps: 27088.42 qps: 27088.42 (r/w/o: 27088.42/0.00/0.00) lat (ms,95%): 1.42 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 128 tps: 28056.06 qps: 28056.06 (r/w/o: 28056.06/0.00/0.00) lat (ms,95%): 0.22 err/s: 0.00 reconn/s: 0.00
[ 65s ] thds: 128 tps: 27296.11 qps: 27296.11 (r/w/o: 27296.11/0.00/0.00) lat (ms,95%): 0.73 err/s: 0.00 reconn/s: 0.00
[ 70s ] thds: 128 tps: 28621.60 qps: 28621.60 (r/w/o: 28621.60/0.00/0.00) lat (ms,95%): 0.19 err/s: 0.00 reconn/s: 0.00
[ 75s ] thds: 128 tps: 28992.29 qps: 28992.29 (r/w/o: 28992.29/0.00/0.00) lat (ms,95%): 0.19 err/s: 0.00 reconn/s: 0.00
[ 80s ] thds: 128 tps: 28279.88 qps: 28279.88 (r/w/o: 28279.88/0.00/0.00) lat (ms,95%): 0.20 err/s: 0.00 reconn/s: 0.00
[ 85s ] thds: 128 tps: 28612.84 qps: 28612.84 (r/w/o: 28612.84/0.00/0.00) lat (ms,95%): 0.20 err/s: 0.00 reconn/s: 0.00
[ 90s ] thds: 128 tps: 28031.47 qps: 28031.47 (r/w/o: 28031.47/0.00/0.00) lat (ms,95%): 0.20 err/s: 0.00 reconn/s: 0.00
[ 95s ] thds: 128 tps: 28734.66 qps: 28734.66 (r/w/o: 28734.66/0.00/0.00) lat (ms,95%): 0.20 err/s: 0.00 reconn/s: 0.00
[ 100s ] thds: 128 tps: 28767.20 qps: 28767.20 (r/w/o: 28767.20/0.00/0.00) lat (ms,95%): 2.39 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            2638920   #总的select数量
        write:                           0
        other:                           0
        total:                           2638920
    transactions:                        2638920 (26382.71 per sec.)   #TPS
    queries:                             2638920 (26382.71 per sec.)   #QPS
    ignored errors:                      0      (0.00 per sec.)        #忽略的错误
    reconnects:                          0      (0.00 per sec.)        #重新连接

Throughput:
    events/s (eps):                      26382.7081                   #每秒的事件数，一般和TPS一样
    time elapsed:                        100.0246s                    #测试的总时间
    total number of events:              2638920                      #总的事件数，一般和TPS一样

Latency (ms):
         min:                                    0.11          #最小响应时间
         avg:                                    4.85          #平均响应时间
         max:                                  649.29          #最大响应时间
         95th percentile:                       25.74          #95%的响应时间是这个数据  
         sum:                             12796148.28

Threads fairness:
    events (avg/stddev):           20616.5625/196.08
    execution time (avg/stddev):   99.9699/0.00#在这个测试中，可以看到TPS与QPS的大小基本一致，说明这个lua脚本中的一个查询一般就是一个事务！
```

我们一般关注的指标主要有:

- response time avg：平均响应时间（后面的95%的大小可以通过–percentile=98的方式去更改）。
- transactions：精确的说是这一项后面的TPS，但如果使用了–skip-trx=on，这项事务数为0，需要用total number of events去除以总时间，得到tps（其实还可以分为读tps和写tps）。
- queries：用它除以总时间，得到吞吐量QPS。

因为上面的TPS与QPS是一样的，因此只绘了TPS的图，如下：

![img](https://img2018.cnblogs.com/blog/1375201/201811/1375201-20181127151233197-873146394.jpg)

刚开始的时候有一个明显的上升，这时候是因为在bp中没有缓存数据，需要从磁盘中读数据，也就是预热阶段！



### 清理数据

```bash
[root@test3 lua]# sysbench /data/sysbench-master/src/lua/oltp_read_write.lua --tables=3 --table_size=10000000 --mysql-user=root --mysql-password=123456 --mysql-host=10.0.102.214 --mysql-port=3306 --mysql-db=sbtest cleanup
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Dropping table 'sbtest1'...
Dropping table 'sbtest2'...
Dropping table 'sbtest3'...
[root@test3 lua]#
```

## 通过sysbench自带的lua脚本对mysql进行测试：

1. 第一步 prepare  

```bash
sysbench --mysql-host=localhost --mysql-port=3306 --mysql-user=sbtest \
    --mysql-password=123456 --mysql-db=tempdb oltp_insert prepare
```

  　　2. 第二步 run

```bash
sysbench --mysql-host=localhost --mysql-port=3306 --mysql-user=sbtest     --mysql-password=123456 --mysql-db=tempdb oltp_insert run                                                              
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Initializing worker threads...

Threads started!

SQL statistics:
    queries performed:
        read:                            0
        write:                           22545
        other:                           0
        total:                           22545
    transactions:                        22545  (2254.37 per sec.)
    queries:                             22545  (2254.37 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

Throughput:
    events/s (eps):                      2254.3691
    time elapsed:                        10.0006s
    total number of events:              22545

Latency (ms):
         min:                                  0.31
         avg:                                  0.44
         max:                                 10.47
         95th percentile:                      0.67
         sum:                               9918.59

Threads fairness:
    events (avg/stddev):           22545.0000/0.00
    execution time (avg/stddev):   9.9186/0.00
```

3. 第三步 cleanup

```bash
sysbench --mysql-host=localhost --mysql-port=3306 --mysql-user=sbtest     --mysql-password=123456 --mysql-db=tempdb oltp_insert cleanup                                  
sysbench 1.1.0 (using bundled LuaJIT 2.1.0-beta3)
Dropping table 'sbtest1'...
```

# 使用sysbench对mysql压测

## 只读示例

```bash
./bin/sysbench --test=share/sysbench/oltp.lua \
--mysql-host=10.229.153.175 --mysql-port=7001 --mysql-user=kp --mysql-password=kp123456 \
--mysql-db=conanwang --oltp-tables-count=10 --oltp-table-size=10000000 \
--report-interval=10 --oltp-dist-type=uniform --rand-init=on --max-requests=0 \
--oltp-test-mode=nontrx --oltp-nontrx-mode=select \
--oltp-read-only=on --oltp-skip-trx=on \
--max-time=120 --num-threads=12 \
```



[prepare|run|cleanup]

 

注意最后一行，一项测试开始前需要用`prepare`来准备好表和数据，`run`执行真正的压测，`cleanup`用来清除数据和表。实际prepare的表结构：

 ```mysql
mysql> desc dbtest1a.sbtest1;
+-------+------------------+------+-----+---------+----------------+
| Field | Type | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+----------------+
| id | int(10) unsigned | NO | PRI | NULL | auto_increment |
| k | int(10) unsigned | NO | MUL | 0 | |
| c | char(120) | NO | | | |
| pad | char(60) | NO | | | |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
 ```

上面的测试命令代表的是：对mysql进行oltp基准测试，表数量10，每表行数约1000w（几乎delete多少就会insert的多少），并且是非事务的只读测试，持续60s，并发线程数12。

**需要说明的选项：**

- `mysql-db=dbtest1a`：测试使用的目标数据库，这个库名要事先创建

- `--oltp-tables-count=10`：产生表的数量

- `--oltp-table-size=10000000`：每个表产生的记录行数

- `--oltp-dist-type=uniform`：指定随机取样类型，可选值有 uniform(均匀分布), Gaussian(高斯分布), special(空间分布)。默认是special

- `--oltp-read-only=off`：表示不止产生只读SQL，也就是使用oltp.lua时会采用读写混合模式。默认 off，如果设置为on，则不会产生update,delete,insert的sql。

- ```
  --oltp-test-mode=nontrx
  ```

  ：执行模式，这里是非事务式的。可选值有simple,complex,nontrx。默认是complex

  - simple：简单查询，SELECT c FROM sbtest WHERE id=N
  - complex (advanced transactional)：事务模式在开始和结束事务之前加上begin和commit， 一个事务里可以有多个语句，如点查询、范围查询、排序查询、更新、删除、插入等，并且为了不破坏测试表的数据，该模式下一条记录删除后会在同一个事务里添加一条相同的记录。
  - nontrx (non-transactional)：与simple相似，但是可以进行update/insert等操作，所以如果做连续的对比压测，你可能需要重新cleanup,prepare。

- `--oltp-skip-trx=[on|off]`：省略begin/commit语句。默认是off

- `--rand-init=on`：是否随机初始化数据，如果不随机化那么初始好的数据每行内容除了主键不同外其他完全相同

- `--num-threads=12`： 并发线程数，可以理解为模拟的客户端并发连接数

- `--report-interval=10`：表示每10s输出一次测试进度报告

- `--max-requests=0`：压力测试产生请求的总数，如果以下面的`max-time`来记，这个值设为0

- `--max-time=120`：压力测试的持续时间，这里是2分钟。

注意，针对不同的选项取值就会有不同的子选项。比如`oltp-dist-type=special`，就有比如`oltp-dist-pct=1`、`oltp-dist-res=50`两个子选项，代表有50%的查询落在1%的行（即热点数据）上，另外50%均匀的(sample uniformly)落在另外99%的记录行上。

再比如`oltp-test-mode=nontrx`时, 就可以有`oltp-nontrx-mode`，可选值有select（默认）, update_key, update_nokey, insert, delete，代表非事务式模式下使用的测试sql类型。

以上代表的是一个只读的例子，可以把`num-threads`依次递增（16,36,72,128,256,512），或者调整my.cnf参数，比较效果。另外需要注意的是，大部分mysql中间件对事务的处理，默认都是把sql发到主库执行，所以只读测试需要加上`oltp-skip-trx=on`来跳过测试中的显式事务。

ps1: 只读测试也可以使用`share/tests/db/select.lua`进行，但只是简单的point select。
ps2: 我在用sysbench压的时候，在mysql后端会话里有时看到大量的query cache lock，如果使用的是uniform取样，最好把查询缓存关掉。当然如果是做两组性能对比压测，因为都受这个因素影响，关心也不大。

## 混合读写

读写测试还是用oltp.lua，只需把`--oltp-read-only`等于`off`。

```bash
./bin/sysbench --test=./share/tests/db/oltp.lua --mysql-host=10.0.201.36 --mysql-port=8066 --mysql-user=ecuser --mysql-password=ecuser --mysql-db=dbtest1a --oltp-tables-count=10 --oltp-table-size=500000 --report-interval=10 --rand-init=on --max-requests=0 --oltp-test-mode=nontrx --oltp-nontrx-mode=select --oltp-read-only=off --max-time=120 --num-threads=128 prepare

./bin/sysbench --test=./share/tests/db/oltp.lua --mysql-host=10.0.201.36 --mysql-port=8066 --mysql-user=ecuser --mysql-password=ecuser --mysql-db=dbtest1a --oltp-tables-count=10 --oltp-table-size=500000 --report-interval=10 --rand-init=on --max-requests=0 --oltp-test-mode=nontrx --oltp-nontrx-mode=select --oltp-read-only=off --max-time=120 --num-threads=128 run

./bin/sysbench --test=./share/tests/db/oltp.lua --mysql-host=10.0.201.36 --mysql-port=8066 --mysql-user=ecuser --mysql-password=ecuser --mysql-db=dbtest1a --oltp-tables-count=10 --oltp-table-size=500000 --report-interval=10 --rand-init=on --max-requests=0 --oltp-test-mode=nontrx --oltp-nontrx-mode=select --oltp-read-only=off --max-time=120 --num-threads=128 cleanup
```



然而`oltp-test-mode=nontrx`一直没有跟着我预期的去走，在mysql general log里面看到的sql记录与`complex`模式相同。所以上面示例中的`--oltp-test-mode=nontrx --oltp-nontrx-mode=select`可以删掉。

 

**update:** 
sysbench作者 akopytov 对我这个疑问有了回复：https://github.com/akopytov/sysbench/issues/34 ，原来sysbench 0.5版本去掉了这个选项，因为作者正在准备1.0版本，所以也就没有更新0.5版本的doc。网上的博客漫天飞，就没有一个提出来的，也是没谁了。

分析一下oltp.lua脚本内容，可以清楚单个事务各操作的默认比例：select:update_key:update_non_key:delete:insert = 14:1:1:1:1，可通过`oltp-point-selects`、`oltp-simple-ranges`、`oltp-sum-ranges`、`oltp-order-ranges`、`oltp-distinct-ranges`，`oltp-index-updates`、`oltp-non-index-updates`这些选项去调整读写权重。

同只读测试一样，在atlas,mycat这类中间件测试中如果不加`oltp-skip-trx=on`，那么所有查询都会发往主库，但如果在有写入的情况下使用`--oltp-skip-trx=on`跳过BEGIN和COMMIT，会出现问题：

> ALERT: failed to execute MySQL query: `INSERT INTO sbtest4 (id, k, c, pad) VALUES (48228, 47329, '82773802508-44916890724-85859319254-67627358653-96425730419-64102446666-75789993135-91202056934-68463872307-28147315305', '13146850449-23153169696-47584324044-14749610547-34267941374')`:
> ALERT: Error 1062 Duplicate entry ‘48228’ for key ‘PRIMARY’
> FATAL: failed to execute function `event’: (null)

原因也很容易理解，每个线程将选择一个随机的表，不加事务的情况下高并发更新（插入）出现重复key的概率很大，但我们压测不在乎这些数据，所以需要跳过这个错误`--mysql-ignore-errors=1062`，这个问题老外有出过打补丁的方案允许`--mysql-ignore-duplicates=on`，但作者新加入的忽略错误码这个功能已经取代了它。`mysql-ignore-errors`选项是0.5版本加入的，但目前没有文档标明，也是我在github上提的 [issue](https://github.com/akopytov/sysbench/issues/23) 作者回复的。

这里不得不佩服老外的办事效率和责任心，提个疑惑能立马得到回复，反观国内，比如在atlas,mycat项目里提到问题到现在都没人搭理。。。

 

## 只更新

如果基准测试的时候，你只想比较两个项目的update（或insert）效率，那可以不使用oltp脚本，而直接改用`update_index.lua`：

```bash
./bin/sysbench --test=./share/tests/db/update_index.lua \

--mysql-host=10.0.201.36 --mysql-port=8066 --mysql-user=ecuser --mysql-password=ecuser \

--mysql-db=dbtest1a --oltp-tables-count=10 --oltp-table-size=500000 \

--report-interval=10 --rand-init=on --max-requests=0 \

--oltp-read-only=off --max-time=120 --num-threads=128 \
```

[ prepare | run | cleanup ]

  

此时像`oltp-read-only=off`许多参数都失效了。需要说明的是这里 (非)索引更新，不是where条件根据索引去查找更新，而是更新索引列上的值。

# sysbench使用举例

下面是sysbench使用的一个例子：

### （1）准备数据

```
sysbench ./tests/include/oltp_legacy/oltp.lua ``--mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --oltp-test-mode=complex --oltp-tables-count=10 --oltp-table-size=100000 --threads=10 --time=120 --report-interval=10 prepare
```

其中，执行模式为complex，使用了10个表，每个表有10万条数据，客户端的并发线程数为10，执行时间为120秒，每10秒生成一次报告。

 ![img](https://images2017.cnblogs.com/blog/1174710/201709/1174710-20170930175906606-700759255.png)

### （2）执行测试

将测试结果导出到文件中，便于后续分析。

```
sysbench ./tests/include/oltp_legacy/oltp.lua ``--mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 --oltp-test-mode=complex --oltp-tables-count=10 --oltp-table-size=100000 --threads=10 --time=120 --report-interval=10 run >> /home/test/mysysbench.log
```

### （3）清理数据

执行完测试后，清理数据，否则后面的测试会受到影响。

```
sysbench ./tests/include/oltp_legacy/oltp.lua ``--mysql-host=192.168.10.10 --mysql-port=3306 --mysql-user=root --mysql-password=123456 cleanup
```

## 5、测试结果

测试结束后，查看输出文件，如下所示：

![img](https://images2017.cnblogs.com/blog/1174710/201709/1174710-20170930175919700-791017735.png)

其中，对于我们比较重要的信息包括：

queries：查询总数及qps

transactions：事务总数及tps

Latency-95th percentile：前95%的请求的最大响应时间，本例中是344毫秒，这个延迟非常大，是因为我用的MySQL服务器性能很差；在正式环境中这个数值是绝对不能接受的。



