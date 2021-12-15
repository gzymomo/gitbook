# 1. 概述

基本所有的服务架构中，Nginx 都是不可或缺的基础组件：

- HTTP 负载均衡，将请求转发到后端 API 服务器。
- 健康检查，将后端无法服务的节点移除。
- 配置 HTTPS ，增强安全性。
- 静态服务器，随着前后端分离，很多前端直接会将 Nginx 作为服务器。
- 基于 Nginx + Lua 结合的解决方案 [OpenResty](https://openresty.org/cn/) ，可以构建出网关等服务，例如说 [Kong](https://konghq.com/kong/) 。
- TCP 支持，可以负载均衡需要长连接的服务，例如说 Netty、Websocket、MySQL 等等。
- ...

基本上，所有后端的服务的接入，入口都是 Nginx 或经过 Nginx ，所以它的性能就会尤其重要。我们都知道，Nginx 性能非常强劲，那么究竟有多强呢？那就让我们来进行一次 Nginx 的性能基准测试。

> FROM [《高性能 Web 服务器 Nginx》](https://www.oschina.net/p/nginx)
>
> Nginx 在官方测试的结果中，能够支持五万个平行连接，而在实际的运作中，可以支持二万至四万个平行链接。

在开始基准测试之前，我们再来看看 Nginx 大体的性能规格，从各大云厂商提供的 Nginx 云服务。

> 艿艿：一般云服务提供的都是 **负载均衡** 服务，所以不一定是 Nginx 服务。这里的整理，更多是为了做一个性能规格上的了解。🙂

- 阿里云 **负载均衡**：https://help.aliyun.com/document_detail/27657.html

- 华为云 **负载均衡**：未提供性能规格文档

- 腾讯云 **负载均衡**： https://cloud.tencent.com/document/product/214/5983

  > 貌似底层是 Nginx 服务。

- 百度云 **负载均衡**：未提供性能规格文档

- UCloud **负载均衡**：https://docs.ucloud.cn/network/ulb/faq

- 美团云 **负载均衡**：未提供性能规格文档

😈 看来，想找个 Nginx 的性能规格作为参考，好难。

# 2. 性能指标

通过我们看各大厂商提供的指标，我们不难发现，主要是 **QPS** 。

# 3. 测试工具

因为我们测试 Nginx ，主要测试 HTTP 接口的处理能力。相对来说，HTTP 接口的测试工具还是比较多的：

- Wrk
- Apache Benchmark （ab）
- Locust
- Jmeter

在研究 Nginx 的过程，看到 [《性能测试工具 wrk,ab,locust,Jmeter 压测结果比较》](https://testerhome.com/topics/17068) 文章，写的非常不错，所以本文会写的比较简短。更多的是，作为记录自己测试 Nginx 的过程。因为只关注 QPS 的性能测试，所以选择了 Wrk 作为测试工具。另外，华为云负载均衡的测试，也是选择 Wrk 嘻嘻。

# 4. Wrk

> FROM [《wrk Github》](https://github.com/wg/wrk)
>
> wrk is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU. It combines a multithreaded design with scalable event notification systems such as epoll and kqueue.

> An optional LuaJIT script can perform HTTP request generation, response processing, and custom reporting. Details are available in SCRIPTING and several examples are located in [scripts/](https://github.com/wg/wrk/tree/master/scripts).

Wrk 是一个比较先进的 HTTP 压力测试工具，并且支持多核运行。

## 4.1 测试环境

- 型号 ：ecs.c5.xlarge

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：40 GB ESSD 云盘

- Nginx ：1.9.9

  > 不会安装 Nginx 的胖友，可以参考 [《CentOS 7 源码编译安装 Nginx》](https://www.cnblogs.com/stulzq/p/9291223.html) 文章，进行编译安装。
  >
  > 在 http://nginx.org/download/ 中，可以下载各个版本的 Nginx 。

## 4.2 安装工具

wrk 的安装，我们将采用编译安装，而源码的来源，我们将从 Github 中获取。

**1、安装 git**

```
yum install git -y
```

**2、克隆 wrk 代码**

```
git clone https://github.com/wg/wrk.git
```

**3、安装 gcc**

```
yum -y install gcc
```

**4、编译安装 wrk**

```
cd wrk
make
```

时间会比较就久，耐心等待。

## 4.3 使用指南

wrk 的使用非常简单，只要了解它每个参数的作用，就可以非常方便的执行一次性能测试。我们来一起看看有哪些参数。执行 `wrk` 命令，返回参数列表：



```
Usage: wrk <options> <url>
  Options:
    -c, --connections <N>  Connections to keep open
    -d, --duration    <T>  Duration of test
    -t, --threads     <N>  Number of threads to use

    -s, --script      <S>  Load Lua script file
    -H, --header      <H>  Add header to request
        --latency          Print latency statistics
        --timeout     <T>  Socket/request timeout
    -v, --version          Print version details

  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```



- `-c` ：需要模拟的连接数。

- `-t` ：并发的线程数。

  > 艿艿：这里要注意下，因为同时有 `-c` 和 `-t` 两个参数，是不是有点费解？这里表示的是，使用 `-t` 个线程，模拟 `-c` 个并发请求。wrk 使用**异步非阻塞**的 io 的方式，并不是用线程去模拟并发连接，因此不需要设置很多的线程，一般根据 CPU 的核心数量设置即可。

- `-d` ：测试的测试时长。

- `-s` ：指定 Lua 脚本的路径。一般情况下，我们不需要这个参数。

- `--header` ：指定请求带的 Header 参数。

- `--latency` ：是否打印请求延迟统计。

- `--timeout` ：设置请求超时时间。

- `-v` ：显示 wrk 版本信息。

## 4.4 快速测试



```
$ ./wrk -t50 -c400 -d30s http://127.0.0.1
```



- `-t50` 参数，设置 50 并发线程。
- `-c400` 参数，设置 400 连接。
- `-d30s` 参数，设置执行 30s 的时长的 HTTP 请求。
- `http://127.0.0.1` 参数，请求本地的 Nginx 服务。

执行结果如下：



```
Running 30s test @ http://127.0.0.1
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    53.49ms  135.94ms   2.00s    87.53%
    Req/Sec     1.26k     0.97k    8.08k    80.95%
  1084727 requests in 30.10s, 0.86GB read
  Socket errors: connect 0, read 85, write 0, timeout 34
Requests/sec:  36038.64
Transfer/sec:     29.18MB
```



- 基本能达到 3W6+ 的 QPS 的性能，美滋滋。

## 4.5 性能优化

胖友可以使用 Google 搜索 Nginx 性能优化相关的文章。艿艿目前看到还不错的文章：

- [《Nginx 配置和性能调优》](https://blog.csdn.net/lamp_yang_3533/article/details/80383039)
- [《Nginx 性能优化》](https://zhuanlan.zhihu.com/p/27288422)

经过优化后，执行结果如下：



```
Running 30s test @ http://127.0.0.1
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     4.44ms    3.96ms 206.30ms   82.96%
    Req/Sec     1.94k   568.42    11.59k    82.31%
  2857342 requests in 30.10s, 2.26GB read
Requests/sec:  94931.16
Transfer/sec:     76.86MB
```



- 基本能达到 9W4+ 的 QPS 的性能，舒服啊~

比较大的提升是，因为机子是 4C ，所以讲 Nginx 的 worker_processes 从 1 改成 4 之后，性能一下子 Double 了一下。

然后，又优化下 worker_cpu_affinity 参数，绑定 Nginx 进程到不同的 CPU 上，又提升了比较大的性能。