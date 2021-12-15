# 1. 概述

当我们希望提高性能的使用，往往想到的是异步、缓存这个两种手段。

- 前者，例如说内存队列（例如说 JDK Queue、Disruptor 等）、分布式队列（例如说 RabbitMQ、RocketMQ、Kafka），更多适合写操作。
- 后者，例如说内存缓存（例如说 JDK Map、EhCache 等）、分布式缓存（例如说 Redis、Memcached 等），更适合读操作。

不过，本文我们不会去聊上述所有的手段或是框架、中间件，而是聚焦本文的主角 Redis 。

日常中，我们经常能在公司、论坛、技术群里看到如下一段对话：

> 甲：我们的 MySQL 读取很慢啊，有什么办法解决啊？
> 乙：上缓存啊，Redis 额。

那么，为什么上 Redis 就一般能解决读的问题呢？为了避免将问题复杂化，我们直接看 Redis 和 MySQL 的性能对比。还是老样子，我们来对比阿里云的 [MySQL 性能规格](https://help.aliyun.com/document_detail/109376.htm) 和 [Redis 性能规格](https://help.aliyun.com/document_detail/26350.html) ：

- 1C 1GB 配置
  - Redis 1C 1GB 主从版，提供 80000 QPS
  - MySQL 1C 1GB 通用型，提供 465 QPS
  - 相差 172 倍左右的性能
- 16C 128G 配置
  - Redis 16C 128G 集群版（单副本），提供 1280000 QPS
  - MySQL 16C 128G 独享版，提供 48102 QPS
  - 相差 26 倍左右的性能

> 当然，两者测试的方式，有一定差异，这里仅仅作为一个量级上的对比。

在开始基准测试之前，我们再来看看 Redis 大体的性能规格，从各大云厂商提供的 Redis 云服务。

- 阿里云 Redis ：https://help.aliyun.com/document_detail/26350.html
- 华为云 Redis ：暂未找到性能规格
- 腾讯云 Redis ： https://cloud.tencent.com/document/product/239/17952
- 百度云 Redis ：暂未找到性能规格
- UCloud Redis ： https://docs.ucloud.cn/database/uredis/test 只提供测试方法，不提供性能规格
- 美团云 Redis ：未提供性能规格文档

# 2. 性能指标

通过我们看各大厂商提供的指标，我们不难发现，主要是 **QPS** 。

# 3. 测试工具

Redis 的性能测试工具，目前主流使用的是 [redis-benchmark](https://redis.io/topics/benchmarks) 。为什么这么说呢？

- 在我们 Google 搜索 “Redis 性能测试”时，清一色的文章选择的工具，清一色的都是 redis-benchmark 。
- 我翻看了云厂商（腾讯云、UCloud 等），提供的测试方法，都是基于 redis-benchmark 。

当然，也是有其它工具：

- [memtier_benchmark](https://github.com/RedisLabs/memtier_benchmark) ：目前阿里云提供的测试方法，是基于它来实现，具体可以看看 [《阿里云 Redis —— 测试工具》](https://help.aliyun.com/document_detail/100347.html) 。
- [YCSB](https://github.com/brianfrankcooper/YCSB/tree/master/redis) ：YCSB 能够测试的服务特别多，[上一节](http://www.iocoder.cn/Performance-Testing/MongoDB-benchmark/?self) 我们就介绍了对 MongoDB 的性能测试。

考虑到主流，本文使用 redis-benchmark 作为性能测试工具。

# 4. redis-benchmark

> FROM [《Redis 有多快?》](http://redis.cn/topics/benchmarks.html)
>
> Redis 自带了一个叫 redis-benchmark 的工具来模拟 N 个客户端同时发出 M 个请求。（类似于 Apache ab 程序）。

## 4.1 测试环境

- 型号 ：ecs.c5.xlarge

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：40 GB ESSD 云盘

- Redis ：5.0.5

  > 不想编译安装的朋友，可以看看 [《How to Install Latest Redis on CentOS 7》](https://computingforgeeks.com/how-to-install-latest-redis-on-centos-7/) 文章。

## 4.2 安装工具

因为 redis-benchmark 是 Redis 自带的，所以不需要专门去安装，舒服~。

## 4.3 使用指南

redis-benchmark 的使用非常简单，只要了解它每个参数的作用，就可以非常方便的执行一次性能测试。我们来一起看看有哪些参数。执行 `redis-benchmark -h` 命令，返回参数列表：

```
Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests>] [-k <boolean>]

 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 3)
 --dbnum <db>       SELECT the specified db number (default 0)
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -e                 If server replies with errors, show them on stdout.
                    (no more than 1 error per second is displayed)
 -q                 Quiet. Just show query/sec values
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
```

- 😈 实际 `redis-benchmark -h` 命令并不是类似很多命令 `--help` 返回参数列表，仅仅是因为 `redis-benchmark -h` 命令是一条错误的命令，所以返回参数列表，提示我们应该怎么做。

- 连接 Redis 服务相关

  - `-h` ：Redis 服务主机地址，默认为 127.0.0.1 。
  - `-p` ：Redis 服务端口，默认为 6379 。
  - `-s` ：指定连接的 Redis 服务地址，用于覆盖 `-h` 和 `-p` 参数。一般情况下，我们并不会使用。
  - `-a` ：Redis 认证密码。
  - `--dbnum` ：选择 Redis 数据库编号。
  - `k` ：是否保持连接。默认会持续保持连接。

- 请求相关参数

  - 🔥 重要：一般情况下，我们会自动如下参数，以达到不同场景下的性能测试。

  - `-c` ：并发的客户端数（每个客户端，等于一个并发）。

  - `-n` ：总共发起的操作（请求）数。例如说，一次 GET 命令，算作一次操作。

  - `-d` ：指定 SET/GET 操作的数据大小，单位：字节。

  - ```
    -r
    ```

     

    ：SET/GET/INCR 使用随机 KEY ，SADD 使用随机值。

    - 默认情况下，使用 `__rand_int__` 作为 KEY 。
    - 通过设置 `-r` 参数，可以设置 KEY 的随机范围。例如说，`-r 10` 生成的 KEY 范围是 `[0, 9)` 。

  - `-P` ：默认情况下，Redis 客户端一次请求只发起一个命令。通过 `-P` 参数，可以设置使用 [pipelining](http://www.iocoder.cn/Performance-Testing/Redis-benchmark/pipelining) 功能，一次发起指定个请求，从而提升 QPS 。

  - `-l` ：循环，一直执行基准测试。

  - `-t` ：指定需要测试的 Redis 命令，多个命令通过逗号分隔。默认情况下，测试 PING_INLINE/PING_BULK/SET/GET 等等命令。如果胖友只想测试 SET/GET 命令，则可以 `-t SET,GET` 来指定。

  - `-I` ：Idle 模式。仅仅打开 N 个 Redis Idle 个连接，然后等待，啥也不做。不是很理解这个参数的目的，目前猜测，仅仅用于占用 Redis 连接。

- 输出相关：

  - `-e` ：如果 Redis Server 返回错误，是否将错误打印出来。默认情况下不打印，通过该参数开启。
  - `-q` ：精简输出结果。即只展示每个命令的 QPS 测试结果。如果不理解的胖友，跑下这个参数就可以很好的明白了。
  - `--csv` ：按照 CSV 的格式，输出结果。

## 4.4 快速测试

```
redis-benchmark
```

在安装 Redis 的服务器上，直接执行，不带任何参数，即可进行测试。测试结果如下：

```bash
====== PING_INLINE ======
  100000 requests completed in 1.18 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
84388.19 requests per second

====== PING_BULK ======
  100000 requests completed in 1.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
85106.38 requests per second

====== SET ======
  100000 requests completed in 1.18 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 1 milliseconds
99.95% <= 2 milliseconds
99.95% <= 3 milliseconds
100.00% <= 3 milliseconds
85034.02 requests per second

====== GET ======
  100000 requests completed in 1.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 1 milliseconds
99.99% <= 2 milliseconds
100.00% <= 2 milliseconds
85106.38 requests per second

====== INCR ======
  100000 requests completed in 1.19 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 2 milliseconds
99.96% <= 3 milliseconds
100.00% <= 3 milliseconds
84317.03 requests per second

====== LPUSH ======
  100000 requests completed in 1.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
85763.29 requests per second

====== RPUSH ======
  100000 requests completed in 1.15 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
87260.03 requests per second

====== LPOP ======
  100000 requests completed in 1.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
85689.80 requests per second

====== RPOP ======
  100000 requests completed in 1.16 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
86281.27 requests per second

====== SADD ======
  100000 requests completed in 1.17 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 2 milliseconds
99.96% <= 3 milliseconds
100.00% <= 3 milliseconds
85106.38 requests per second

====== HSET ======
  100000 requests completed in 1.14 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
87719.30 requests per second

====== SPOP ======
  100000 requests completed in 1.16 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
85836.91 requests per second

====== LPUSH (needed to benchmark LRANGE) ======
  100000 requests completed in 1.15 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.92% <= 1 milliseconds
100.00% <= 1 milliseconds
86805.56 requests per second

====== LRANGE_100 (first 100 elements) ======
  100000 requests completed in 2.03 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 1 milliseconds
99.95% <= 2 milliseconds
99.96% <= 3 milliseconds
99.99% <= 4 milliseconds
100.00% <= 4 milliseconds
49261.09 requests per second

====== LRANGE_300 (first 300 elements) ======
  100000 requests completed in 4.58 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

6.06% <= 1 milliseconds
99.78% <= 2 milliseconds
99.94% <= 3 milliseconds
99.98% <= 4 milliseconds
100.00% <= 5 milliseconds
100.00% <= 5 milliseconds
21815.01 requests per second

====== LRANGE_500 (first 450 elements) ======
  100000 requests completed in 6.51 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

0.04% <= 1 milliseconds
83.91% <= 2 milliseconds
99.93% <= 3 milliseconds
99.97% <= 4 milliseconds
99.98% <= 5 milliseconds
99.99% <= 6 milliseconds
100.00% <= 7 milliseconds
100.00% <= 7 milliseconds
15372.79 requests per second

====== LRANGE_600 (first 600 elements) ======
  100000 requests completed in 8.66 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

0.03% <= 1 milliseconds
62.47% <= 2 milliseconds
98.11% <= 3 milliseconds
99.86% <= 4 milliseconds
99.94% <= 5 milliseconds
99.97% <= 6 milliseconds
99.98% <= 7 milliseconds
100.00% <= 8 milliseconds
100.00% <= 8 milliseconds
11551.35 requests per second

====== MSET (10 keys) ======
  100000 requests completed in 1.11 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.95% <= 2 milliseconds
99.96% <= 3 milliseconds
100.00% <= 3 milliseconds
90009.01 requests per second
```

基本可以看到，常用的 GET/SET/INCR 等命令，都在 8W+ QPS 以上，美滋滋。

## 4.5 精简测试

```bash
redis-benchmark -t set,get,incr -n 1000000 -q
```

- 通过 `-t` 参数，设置仅仅测试 SET/GET/INCR 命令
- 通过 `-n` 参数，设置每个测试执行 1000000 次操作。
- 通过 `-q` 参数，设置精简输出结果。

执行结果如下：

```bash
[root@iZuf6hci646px19gg3hpuwZ ~]# redis-benchmark -t set,get,incr -n 1000000 -q
SET: 85888.52 requests per second
GET: 85881.14 requests per second
INCR: 86722.75 requests per second
```

是不是一下子精简很多？！

## 4.6 pipeline 测试

在一些业务场景，我们希望通过 Redis pipeline 功能，批量提交命令给 Redis Server ，从而提升性能。那么，我们就来测试下

```
redis-benchmark -t set,get,incr -n 1000000 -q -P 10
```

- 通过 `-P` 参数，设置每个 pipeline 执行 10 次 Redis 命令。

执行结果如下：

```
SET: 625782.19 requests per second
GET: 827814.62 requests per second
INCR: 745712.19 requests per second
```

相比 [「4.5 精简测试」](http://www.iocoder.cn/Performance-Testing/Redis-benchmark/#) 来说，性能有了 8-10 倍左右的提升，无敌！

## 4.7 随机 KEY 测试

本小节，我们主要来看看 `-r` 参数的使用。为了更好的对比，我们来先看看未使用的 `-r` 的情况，然后再测试使用 `-r` 的情况。

**未使用**

```
redis-benchmark -t set -n 1000 -q
```

我们来查看 Redis 中，有哪些 KEY ：

```
$ redis-cli flushdb # 用于清空 Redis 中的数据

$ redis-cli keys \*
1) "key:__rand_int__"
```

- 只有一个以 `key:` 开头，结尾是 `__rand_int__"` 的 KEY 。这说明，整个测试过程，使用的都是这个 KEY 。

**使用**

```
$ redis-cli flushdb # 用于清空 Redis 中的数据

$ redis-benchmark -t set -n 1000 -q -r 10
```

- 通过 `-r 10` 参数，设置 KEY 的随机范围为 `-r 10` 。

我们来查看 Redis 中，有哪些 KEY ：

```
$ redis-cli keys \*
 1) "key:000000000001"
 2) "key:000000000009"
 3) "key:000000000004"
 4) "key:000000000005"
 5) "key:000000000002"
 6) "key:000000000008"
 7) "key:000000000003"
 8) "key:000000000007"
 9) "key:000000000006"
10) "key:000000000000"
```

- 可以看到以 `key:` 开头，结果是 `[0, 9)` 范围内的KEY 。

这样，是不是对 `-r` 参数，有了理解落。通过 `-r` 参数，我们可以测试随机 KEY 的情况下的性能。