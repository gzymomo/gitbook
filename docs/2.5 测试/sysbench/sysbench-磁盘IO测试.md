# [如何用sysbench做好IO性能测试](https://blog.csdn.net/xstardust/article/details/85163513)



```bash
[root@centos01 sysbench-0.5]# cd sysbench/
[root@centos01 sysbench-0.5]# ./sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw prepare
[root@centos01 sysbench-0.5]# ./sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw run
```

![img](https://www.pianshen.com/images/252/0da56b8080592926ef6cf94fe5e3d75c.png)
**可以看到随机写性能 84.92Mb/sec，随机读性能5434.86 Requests/sec**

-- 清理数据

```bash
[root@centos01 sysbench-0.5]# ./sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw cleanup
```

# sysbench fileio测试

言归正传，sysbench怎么做IO的性能测试呢，`sysbench fileio help`,参数如下：

```bash
#/usr/local/sysbench_1/bin/sysbench fileio help
sysbench 1.0.9 (using bundled LuaJIT 2.1.0-beta2)
 
fileio options:
  --file-num=N              number of files to create [128]
  --file-block-size=N       block size to use in all IO operations [16384]
  --file-total-size=SIZE    total size of files to create [2G]
  --file-test-mode=STRING   test mode {seqwr, seqrewr, seqrd, rndrd, rndwr, rndrw}
  --file-io-mode=STRING     file operations mode {sync,async,mmap} [sync]
  --file-async-backlog=N    number of asynchronous operatons to queue per thread [128]
  --file-extra-flags=STRING additional flags to use on opening files {sync,dsync,direct} []
  --file-fsync-freq=N       do fsync() after this number of requests (0 - don't use fsync()) [100]
  --file-fsync-all[=on|off] do fsync() after each write operation [off]
  --file-fsync-end[=on|off] do fsync() at the end of test [on]
  --file-fsync-mode=STRING  which method to use for synchronization {fsync, fdatasync} [fsync]
  --file-merged-requests=N  merge at most this number of IO requests if possible (0 - don't merge) [0]
  --file-rw-ratio=N         reads/writes ratio for combined test [1.5]
```

sysbench的性能测试都需要做`prepare`,`run`,`cleanup`这三步，准备数据，跑测试，删除数据。那下面就开始实战：
客户用2C4G的vm,挂载120G的SSD云盘做了性能测试，测试命令如下：

```
cd /mnt/vdb  #一定要到你测试的磁盘目录下执行，否则可能测试系统盘了
sysbench fileio --file-total-size=15G --file-test-mode=rndrw --time=300 --max-requests=0 prepare
sysbench fileio --file-total-size=15G --file-test-mode=rndrw --time=300 --max-requests=0 run
sysbench fileio --file-total-size=15G --file-test-mode=rndrw --time=300 --max-requests=0 cleanup
```

结果如下：

```bash
File operations:
    reads/s:                      2183.76
    writes/s:                     1455.84
    fsyncs/s:                     4658.67
 
Throughput:
    read, MiB/s:                  34.12
    written, MiB/s:               22.75
 
General statistics:
    total time:                          300.0030s
    total number of events:              2489528
 
Latency (ms):
         min:                                  0.00
         avg:                                  0.12
         max:                                204.04
         95th percentile:                      0.35
         sum:                             298857.30
 
Threads fairness:
    events (avg/stddev):           2489528.0000/0.00
    execution time (avg/stddev):   298.8573/0.00
```

随机读写性能好像不咋地，换算IOPS为(34.12+22.75)*1024/16.384=3554.375，与宣称的5400IOPS有很大差距。眼尖的人肯定发现只有2个核，去遍历128个文件，好像会降低效率，于是定制file-num去做了系列测试，测试结果如下：

| file-num    | 1     | 2    | 4     | 8     | 16    | 32    | 64    | 128   |
| :---------- | :---- | :--- | :---- | :---- | :---- | :---- | :---- | :---- |
| read(MB/s)  | 57.51 | 57.3 | 57.36 | 57.33 | 55.12 | 47.72 | 41.11 | 34.12 |
| write(MB/s) | 38.34 | 38.2 | 38.24 | 38.22 | 36.75 | 31.81 | 27.4  | 22.75 |

明显可以看到，默认测试方法会导致性能下降，文件数设置为1达到最大性能。
那file-num=128与file-num=1的区别是测试文件从128个变成1个，但是总文件大小都是15G，都是随机读写，按理性能应该是一致的，区别是会在多个文件之间切换读写，那么可能会导致中断增加和上下文切换开销增大。通过vmstat命令得到了验证：
file-num=128的vmstat输出是这样的：
![sysbench_rw_1_128](https://img-blog.csdnimg.cn/20181221162240420)
file-num=1的vmstat输出是这样的：
![sysbench_rw_1_1](https://img-blog.csdnimg.cn/20181221162240439)
从上面两个图可以看出file-num=1的时候上下文切换只有8500左右比file-num=128的时候24800小多了，in（中断）也少太多了。减少了中断和上下文切换开销，吞吐能力显著提升了。
再做了一个实验，同样磁盘大小，改成挂载到8C的vm下，改成8线程进行测试，得到如下数据：

| file-num    | 1      | 2      | 4      | 8      | 16     | 32    | 64    | 128   |
| :---------- | :----- | :----- | :----- | :----- | :----- | :---- | :---- | :---- |
| read(MB/s)  | 253.08 | 209.86 | 193.38 | 159.73 | 117.98 | 86.78 | 67.39 | 51.98 |
| write(MB/s) | 168.72 | 139.9  | 128.92 | 106.49 | 78.66  | 57.85 | 44.93 | 34.65 |

可以得出同样的结论，file-num=1可以得到最好的性能，理由如上。

# 与fio测试的比较

单进程下，file-num=1换算到IOPS为(57.51+38.34)*1024/16.384=5990.625，这好像超过我们的IOPS设置限定了。通过fio是怎么测得这个IOPS的呢：

```
fio -direct=1 -iodepth=128 -rw=randrw -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=randrw_test
```

通过阅读源代码，发现很多不同：

1. 一个是通过libaio，一个是通过pwrite/pread。libaio的性能是非常强劲的，详情可以参考[文章](http://blog.yufeng.info/archives/741)。
   即使ioengine=psync，这个engine的读写方法是pread和pwrite，但是整个实现也是不一致的。
2. fio测试的时候direct=1，就是每次都写入磁盘，而sysbench默认file-fsync-freq=100，也就是完成100次操作才会有一个fsync操作，这种操作涉及系统缓存。



# 使用sysbench对磁盘进行性能测试

## 准备IO测试文件：执行如下命令，生成16个文件，用于本次测试。

```
sysbench --test=fileio --file-num=16 --file-total-size=2G prepare
```

参数含义：
–test=fileio 测试的名称叫做 fileio
–file-num=16 文件的数量是 16 个
–file-total-size=2G 文件的总体大小是 2GB
说明：每个测试文件的大小是 130MB，总计花费 32 秒。
查看测试文件：

```bash
$ ll
总用量 2097156
drwxr-xr-x 13 root root      4096 8月  24 15:50 sysbench
-rw-------  1 root root 134217728 8月  24 15:57 test_file.0
-rw-------  1 root root 134217728 8月  24 15:57 test_file.1
-rw-------  1 root root 134217728 8月  24 15:58 test_file.10
-rw-------  1 root root 134217728 8月  24 15:58 test_file.11
-rw-------  1 root root 134217728 8月  24 15:58 test_file.12
-rw-------  1 root root 134217728 8月  24 15:58 test_file.13
-rw-------  1 root root 134217728 8月  24 15:58 test_file.14
-rw-------  1 root root 134217728 8月  24 15:58 test_file.15
-rw-------  1 root root 134217728 8月  24 15:57 test_file.2
-rw-------  1 root root 134217728 8月  24 15:57 test_file.3
-rw-------  1 root root 134217728 8月  24 15:57 test_file.4
-rw-------  1 root root 134217728 8月  24 15:57 test_file.5
-rw-------  1 root root 134217728 8月  24 15:57 test_file.6
-rw-------  1 root root 134217728 8月  24 15:58 test_file.7
-rw-------  1 root root 134217728 8月  24 15:58 test_file.8
-rw-------  1 root root 134217728 8月  24 15:58 test_file.9
```

## 测试多线程下小IO的随机只读性能

```bash
$ sysbench --test=fileio --file-num=16 --file-total-size=2G --file-test-mode=rndrd  --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.6 (using system LuaJIT 2.0.4)
 
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time
 
 
Extra file open flags: 3
16 files, 128MiB each
2GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random read test
Initializing worker threads...
 
Threads started!
  
File operations:
    reads/s:                      1487.34
    writes/s:                     0.00
    fsyncs/s:                     0.00
 
Throughput:
    read, MiB/s:                  23.24
    written, MiB/s:               0.00
 
General statistics:
    total time:                          10.0002s
    total number of events:              14877
 
Latency (ms):
         min:                                  0.15
         avg:                                  0.67
         max:                                168.34
         95th percentile:                      0.84
         sum:                               9978.66
 
Threads fairness:
    events (avg/stddev):           14877.0000/0.00
    execution time (avg/stddev):   9.9787/0.00
```

参数含义：
–test=fileio 测试的名称叫做 fileio
–file-num=16 文件的数量是 16 个
–file-total-size=2G 文件的总体大小是 2GB
–file-test-mode=rndrd 测试模式是随机读取
–file-extra-flags=direct 使用额外的标志来打开文件{sync,dsync,direct}
–file-fsync-freq=0 执行 fsync() 的频率
–file-block-size=16384 测试时文件块的大小位 16384 (16K)

测试结果：
随机读取的数据吞吐量：23.24 MB/s
随机读的IOPS： 1487.34
平均延迟：0.67毫秒

## 测试多线程下小IO的随机写入性能

```bash
$ sysbench --test=fileio --file-num=16 --file-total-size=2G --file-test-mode=rndwr --max-time=180  --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
WARNING: --max-time is deprecated, use --time instead
sysbench 1.0.6 (using system LuaJIT 2.0.4)
 
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time
 
 
Extra file open flags: 3
16 files, 128MiB each
2GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random write test
Initializing worker threads...
 
Threads started!
 
 
File operations:
    reads/s:                      0.00
    writes/s:                     1536.37
    fsyncs/s:                     0.00
 
Throughput:
    read, MiB/s:                  0.00
    written, MiB/s:               24.01
 
General statistics:
    total time:                          180.0041s
    total number of events:              276556
 
Latency (ms):
         min:                                  0.22
         avg:                                  0.65
         max:                                279.43
         95th percentile:                      1.01
         sum:                             179649.14
 
Threads fairness:
    events (avg/stddev):           276556.0000/0.00
    execution time (avg/stddev):   179.6491/0.00
```

参数含义：
–file-test-mode=rndwr 测试模式是随机写

测试结果：
随机写入的数据吞吐量：24.01 MB/s
随机写的IOPS： 1536.37
平均延迟：0.65毫秒

## 测试多线程下小IO的随机读写性能

```bash
$ sysbench --test=fileio --file-num=16 --file-total-size=2G --file-test-mode=rndrw --file-extra-flags=direct --file-fsync-freq=0 --file-block-size=16384 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.6 (using system LuaJIT 2.0.4)
 
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time
 
 
Extra file open flags: 3
16 files, 128MiB each
2GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...
 
Threads started!
 
 
File operations:
    reads/s:                      1516.94
    writes/s:                     1011.33
    fsyncs/s:                     0.00
 
Throughput:
    read, MiB/s:                  23.70
    written, MiB/s:               15.80
 
General statistics:
    total time:                          10.0073s
    total number of events:              25307
 
Latency (ms):
         min:                                  0.12
         avg:                                  0.39
         max:                                 66.01
         95th percentile:                      0.77
         sum:                               9981.29
 
Threads fairness:
    events (avg/stddev):           25307.0000/0.00
    execution time (avg/stddev):   9.9813/0.00
```

参数含义：
–file-test-mode=rndrw  测试模式是随机读写
随机读取的数据吞吐量：23.70 MB/s
随机写入的数据吞吐量：15.80 MB/s
随机读IOPS：1516.94
随机写IOPS：1011.33
平均延迟：0.39毫秒



## 附录：

Sysbench 的文件测试模式，seqwr 顺序写，seqrew r顺序读写，seqrd 顺序读，rndrd 随机读，rndwr 随机写，rndrw 随机读写。

–file-rw-ratio=4 读写比例调整，为4表示读写比例4:1