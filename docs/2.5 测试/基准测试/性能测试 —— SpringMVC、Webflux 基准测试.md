# 1. 概述

自 Spring 5.x 发布后，新增了 Spring WebFlux ，一个基于 [Reactor](https://github.com/reactor/reactor-core) 实现的[响应式](http://www.iocoder.cn/Performance-Testing/SpringMVC-Webflux-benchmark/响应式编程) Web 框架。其功能最大的特点，在于**非阻塞异步**，所以其需要运行如下环境下：

- 1、支持 [Servlet 3.1](https://www.ibm.com/support/knowledgecenter/zh/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/cwlp_servlet31.html) 的 Web 容器中，例如说 Tomcat、Jetty、Undertow 。
- 2、Netty 网络通信框架，在 Webflux 使用的是 [Reactor Netty](https://github.com/reactor/reactor-netty) 框架进行实现。并且，在使用 `spring-boot-starter-webflux` 时，默认使用的就是这种环境。

得益于 Webflux 框架，我们可以非常的将相对耗时的 IO 操作，提交到线程池中，从而达到异步非阻塞的功能，进一步提升并发性能。当然，实际上，我们使用 Servlet 3.1 + 线程池，也能实现非阻塞的效果，但是相比来说会麻烦一些。

> 艿艿：上面的这段话，可能写的有点绕口，或者不好理解，欢迎一起交流。

不过，Spring 在推出 WebFlux 后，带上响应式的概念，在目前这个时候，已经有些被魔化的带上“高并发”的说法？！所以，带着这样的好奇与疑惑，我们一起来做下 SpringMVC 和 Webflux 的性能基准测试。

在继续往下阅读本文之前，希望胖友已经阅读过如下两篇文章，因为有一些涉及到的关联知识，本文不会赘述：

- [《性能测试 —— Nginx 基准测试》](http://www.iocoder.cn/Performance-Testing/Nginx-benchmark/self)
- [《性能测试 —— Tomcat、Jetty、Undertow 基准测试》](http://www.iocoder.cn/Performance-Testing/Tomcat-Jetty-Undertow-benchmark/?self)

# 2. 性能指标

和 [《性能测试 —— Nginx 基准测试》](http://www.iocoder.cn/Performance-Testing/Nginx-benchmark/self) 保持一致，我们还是以 **QPS** 作为性能的指标。

在 https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-06 中，我们提供了三个示例，分别是：

- lab-06-springmvc-tomcat ：Tomcat 9.0.16 NIO 模式 + Spring MVC

- lab-06-webflux-tomcat ：Tomcat 9.0.16 NIO 模式 + Webflux

  > 相比上个示例，保持使用的 Tomcat 的前提下，将 SpringMVC 替换成 Webflux 。

- lab-06-webflux-netty ：Netty 4.1.33.Final + Webflux

  > 相比上个示例，保持使用的 Weblufx 的前提下，将 Tomcat 替换成 Netty 。

胖友可以使用 `mvn package` 命令，打包出不同的示例，进行压力测试。

# 3. 测试环境

- 型号 ：ecs.c5.xlarge

  > 艿艿：和我一样抠门（穷）的胖友，可以买竞价类型服务器，使用完后，做成镜像。等下次需要使用的时候，恢复一下。HOHO 。

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：40 GB ESSD 云盘

- JDK ：openjdk version "1.8.0_212"

- JVM 参数 ：`-Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k`

因为我们在跑的过程，发现 wrk 占用 CPU 不是很高，所以直接本机运行。

有一点要注意，JVM 本身有[预热](https://codeday.me/bug/20180203/128666.html)的过程，Tomcat、Jetty、Undertow 本也有预热的过程（例如说，线程的初始化），所以需要多次测试，取平均值。

本文，我们使用 wrk 如下命令进行测试：



```
./wrk -t50 -c并发 -d30s http://127.0.0.1:8080
```



- `-t50` 参数，设置 50 并发线程。
- `-c并发` 参数，设置并发连接，目前会按照 300、1000、3000、5000 的维度，进行测试。
- `-d30s` 参数，设置执行 30s 的时长的 HTTP 请求。
- `http://127.0.0.1:8080` 参数，请求本地的 Web 服务。

下面，让我们进入正式的测试。在每一轮中，我们将测试相同场景下，SpringMVC 和 Webflux 的表现。

# 4. 第一轮

在这轮中，我们想先来测试下，在逻辑中完全无 IO 的情况下，三者的性能情况。请求的示例接口如下：

- Spring MVC ，直接返回 `"world"` 字符串。

  ```
  @GetMapping("/hello")
  public String hello() {
      return "world";
  }
  ```

  

- Spring Webflux ，直接返回 `"world"` 字符串的 Mono 对象。

  ```
  @GetMapping("/hello")
  public Mono<String> hello() {
      return Mono.just("world");
  }
  ```

  

## 4.1 SpringMVC + Tomcat

🚚 **300 并发**



```
$ ./wrk -t50 -c300 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    10.82ms    8.36ms 180.77ms   92.82%
    Req/Sec   585.78     64.82     2.22k    77.93%
  876355 requests in 30.10s, 98.78MB read
Requests/sec:  29115.47
Transfer/sec:      3.28MB
```



- 29115 QPS
- 10.82ms Avg Latency

🚚 **1000 并发**



```
$./wrk -t50 -c1000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    34.91ms    6.73ms 101.04ms   82.96%
    Req/Sec   574.99     64.19     2.67k    83.73%
  863098 requests in 30.10s, 97.27MB read
Requests/sec:  28678.48
Transfer/sec:      3.23MB
```



- 28678 QPS
- 34.91ms Avg Latency

🚚 **3000 并发**



```
Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   103.25ms   15.48ms 303.58ms   92.31%
    Req/Sec   579.84     85.08     4.05k    88.02%
  870323 requests in 30.10s, 98.05MB read
Requests/sec:  28911.19
Transfer/sec:      3.26MB
```



- 28911 QPS
- 103.25ms Avg Latency

🚚 **5000 并发**



```
Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 5000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   169.61ms   27.88ms 558.41ms   93.53%
    Req/Sec   585.99    185.74     7.03k    80.49%
  876680 requests in 30.10s, 98.75MB read
Requests/sec:  29126.46
Transfer/sec:      3.28MB
```



- 29126 QPS
- 169.61ms Avg Latency

**小结**

总的来说，QPS 比较稳定在 29000 左右，而延迟逐步提升。

## 4.2 Webflux + Tomcat

🚚 **300 并发**



```
$ ./wrk -t50 -c300 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    13.51ms    5.53ms 146.89ms   83.08%
    Req/Sec   450.70     42.24     1.53k    77.86%
  674480 requests in 30.09s, 76.02MB read
Requests/sec:  22413.32
Transfer/sec:      2.53MB
```



- 22413 QPS
- 13.51ms Avg Latency

🚚 **1000 并发**



```
$ ./wrk -t50 -c1000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    45.58ms    7.52ms 242.98ms   85.81%
    Req/Sec   439.07     49.56     1.76k    85.46%
  656334 requests in 30.10s, 73.97MB read
Requests/sec:  21805.86
Transfer/sec:      2.46MB
```



- 21805.86 QPS
- 45.58ms Avg Latency

🚚 **3000 并发**



```
$ ./wrk -t50 -c3000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   136.81ms   21.59ms 479.21ms   94.26%
    Req/Sec   437.68    103.92     4.55k    82.03%
  655949 requests in 30.10s, 73.92MB read
Requests/sec:  21791.89
Transfer/sec:      2.46MB
```



- 21791.89 QPS
- 136.81ms Avg Latency

🚚 **5000 并发**



```
$ ./wrk -t50 -c5000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 5000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   220.63ms   37.76ms 432.82ms   92.59%
    Req/Sec   448.51    199.83     4.37k    74.60%
  660039 requests in 30.10s, 74.37MB read
Requests/sec:  21925.27
Transfer/sec:      2.47MB
```



- 21925.27 QPS
- 220.63ms Avg Latency

**小结**

- 从自身角度：总的来说，QPS 比较稳定在 21000 左右，而延迟逐步提升。
- 对比 SpringMVC + Tomcat 角度：因为 Webflux 使用的是 Tomcat 对 Servlet 3.1 的实现，而这个场景的逻辑中，是完全没有 IO 的，所以“多余”了异步的过程，反倒带来了性能的下降。

## 4.3 Webflux + Netty

🚚 **300 并发**



```
$./wrk -t50 -c300 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     8.07ms    5.53ms  72.89ms   76.78%
    Req/Sec   804.91    143.55     4.62k    78.17%
  1203712 requests in 30.10s, 95.28MB read
Requests/sec:  39986.10
Transfer/sec:      3.17MB
```



- 39986.10 QPS
- 8.07ms Avg Latency

🚚 **1000 并发**



```
$./wrk -t50 -c1000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    22.00ms    7.60ms 261.83ms   84.69%
    Req/Sec     0.90k   210.39    13.14k    89.39%
  1342217 requests in 30.10s, 106.24MB read
Requests/sec:  44593.68
Transfer/sec:      3.53MB
```



- 44593.68 QPS
- 22.00ms Avg Latency

🚚 **3000 并发**



```
$./wrk -t50 -c3000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    72.83ms   24.03ms   1.02s    82.76%
    Req/Sec   803.62    247.83     9.80k    87.97%
  1208192 requests in 30.11s, 95.63MB read
Requests/sec:  40123.56
Transfer/sec:      3.18MB
```



- 40123.56 QPS
- 72.83ms Avg Latency

🚚 **5000 并发**



```
$./wrk -t50 -c5000 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 5000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   126.80ms   68.82ms   1.98s    90.25%
    Req/Sec   764.53    312.29    17.26k    84.27%
  1141219 requests in 30.17s, 90.33MB read
  Socket errors: connect 0, read 0, write 0, timeout 37
Requests/sec:  37823.63
Transfer/sec:      2.99MB
```



- 37823.63 QPS
- 126.80ms Avg Latency

**小结**

- 从自身角度：随着并发请求的上升，QPS 先从 4w 左右，涨到 4.4w ，然后下滑到 3.8W 左右。

- 对比 SpringMVC + Tomcat 角度：Netty 相比 Tomcat 的线程模型，更加简洁。主要体现在 Netty worker 线程解析好请求，直接进行了 Webflux 的逻辑执行，而 Tomcat Poller 线程拿到请求后，具体的请求需要丢给 Tomcat Worker 线程来解析，再进行了 SpringMVC 的逻辑执行。这样，就导致 Tomcat 多了一次线程的切换，而这个场景的逻辑中，是完全没有 IO 的，所以性能会相对低。

  > 不了解 Tomcat NIO 线程模型的胖友，可以看看如下两篇文章：
  >
  > - [《Tomcat NIO 线程模型深入分析》](https://www.jianshu.com/p/f91f99610b9e)
  > - [《Tomcat NIO 线程模型分析》](https://www.jianshu.com/p/4e239e217ada)
  >
  > 不了解 Netty NIO 线程模型的胖友，可以看看如下文章：
  >
  > - [《netty学习系列二：NIO Reactor模型 & Netty线程模型》](https://www.jianshu.com/p/38b56531565d)
  >
  > 当然，胖友可以思考下，为什么 Tomcat 会比 Netty 多了一个线程池呢？其实是殊途同归的。然后，如果有使用过 Dubbo 的胖友，再去看看 Dubbo 的[线程模型](http://dubbo.apache.org/zh-cn/docs/user/demos/thread-model.html)，在体会一波。
  >
  > 🔥 其实，Tomcat 的 Worker 线程池，是为了并发执行业务逻辑，往往来说，业务逻辑都是有 IO 操作的，所以进行了这样的设计。而 Netty 的 Worker 线程，和 Tomcat 的 Poller 线程，是基本等价的。实际场景下，Netty Worker 线程，解码完消息包后，我们也会自己创建一个业务线程池，将解析的消息丢入其中，进行执行逻辑。 也就是说，本质上，是将具有 IO 操作的逻辑，丢到线程池中，避免**有限**的 Netty Worker 线程，或者 Tomcat Poller 线程被阻塞。 然后，我们把这个设计思路，带到 Webflux 上，是不是也是通的，嘻嘻。

- 对比 Webflux + Tomcat 角度：相比来说，Netty 会比 Tomcat 更加适合作为 Webflux 的运行环境。

# 5. 第二轮

在这轮中，我们要来测试下，在逻辑中有 IO 的情况下，三者的性能情况。考虑到让测试更加简单，我们采用 `Thread.sleep(100L)` 暂停线程 100ms 的方式，来模拟 IO 阻塞的行为。请求的示例接口如下：

- Spring MVC ，先 sleep 100ms ，再返回 `"world"` 字符串。

  ```
  @GetMapping("/sleep")
  public String sleep() throws InterruptedException {
      Thread.sleep(100L);
      return "world";
  }
  ```

  

- Webflux ，先 sleep 100ms ，再返回 `"world"` 字符串的 Mono 对象。

  ```
  @GetMapping("/sleep")
  public Mono<String> sleep() {
      return Mono.defer(() -> {
          try {
              Thread.sleep(100L);
          } catch (InterruptedException ignored) {
          }
          return Mono.just("world");
      }).subscribeOn(Schedulers.parallel());
  }
  ```

  

  - 因为 sleep 会阻塞 100ms ，所以我们将逻辑的执行，通过 `subscribeOn(Schedulers.parallel())` 代码块，调度到 Reactor 内置的用于并发执行的线程池，它的大小是 CPU 的**线程**数。例如说，本文使用的阿里云服务器，创建的线程池大小就是 4 。😈 为什么提这个呢？下文我们就会明白了。

## 5.1 SpringMVC + Tomcat

🚚 **300 并发**



```
$./wrk -t50 -c300 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   150.48ms   44.14ms 216.52ms   48.29%
    Req/Sec    39.86     11.50    70.00     81.41%
  59735 requests in 30.10s, 6.73MB read
Requests/sec:   1984.58
Transfer/sec:    228.96KB
```



- 1984 QPS
- 150.48ms Avg Latency

🚚 **1000 并发**



```
$./wrk -t50 -c1000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   497.76ms  118.19ms 955.99ms   89.35%
    Req/Sec    40.54     19.53   190.00     68.86%
  59815 requests in 30.10s, 6.73MB read
Requests/sec:   1987.28
Transfer/sec:    229.00KB
```



- 1987 QPS
- 497.76ms Avg Latency

🚚 **3000 并发**



```
$./wrk -t50 -c3000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.46s   286.08ms   1.65s    93.85%
    Req/Sec    97.48     97.90   590.00     82.99%
  59881 requests in 30.10s, 6.74MB read
Requests/sec:   1989.43
Transfer/sec:    229.25KB
```



- 1989 QPS
- 1.46s Avg Latency

🚚 **5000 并发**



```
$./wrk -t50 -c5000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 5000 connections

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   665.93ms  651.42ms   1.95s    76.81%
    Req/Sec   110.86    154.20     0.97k    86.41%
  59897 requests in 30.10s, 6.74MB read
  Socket errors: connect 0, read 0, write 0, timeout 54494
Requests/sec:   1989.93
Transfer/sec:    229.31KB
```



- 1989 QPS
- 665.93ms Avg Latency

**小结**

- 从自身角度：QPS 稳定在 2000 QPS 不到，这个是为什么呢？默认情况下，Spring Boot 设置内嵌的 Tomcat 的 Worker 线程池大小为 200 ，加上每个逻辑需要 sleep 100 ms ，所以每个线程能处理 10 个情况，所以 QPS 最大就在 10 * 200 = 2000 ，所以测试结果就基本在 2000 QPS 。

## 5.2 Webflux + Tomcat

🚚 **300 并发**



```
$./wrk -t50 -c300 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.00us    0.00us   0.00us    -nan%
    Req/Sec     2.96      3.61    20.00     83.63%
  1065 requests in 30.06s, 122.72KB read
  Socket errors: connect 0, read 0, write 0, timeout 1065
Requests/sec:     35.43
Transfer/sec:      4.08KB
```



- 35.43 QPS
- 0.00us Avg Latency 【计算错误】

是不是看到这样的性能，一脸懵逼？淡定，我们下面会解释。

🚚 **1000 并发**



```
$./wrk -t50 -c1000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.01s   552.42ms   1.91s    57.89%
    Req/Sec     7.56      7.71    50.00     85.68%
  1192 requests in 30.10s, 137.36KB read
  Socket errors: connect 0, read 0, write 0, timeout 1116
Requests/sec:     39.61
Transfer/sec:      4.56KB
```



- 35.43 QPS
- 0.00us Avg Latency 【计算错误】

🚚 **3000 并发**



```
$./wrk -t50 -c3000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   983.66ms  543.02ms   1.96s    60.00%
    Req/Sec    11.37      9.50    40.00     76.67%
  1184 requests in 30.10s, 136.44KB read
  Socket errors: connect 0, read 0, write 0, timeout 1124
Requests/sec:     39.34
Transfer/sec:      4.53KB
```



- 39.34 QPS
- 983.66ms Avg Latency

🚚 **5000 并发**



```
$./wrk -t50 -c5000 -d30s http://127.0.0.1:8080/sleep

Running 30s test @ http://127.0.0.1:8080/sleep
  50 threads and 5000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.25s   480.74ms   1.94s    65.45%
    Req/Sec    12.07     10.54    40.00     73.63%
  1187 requests in 30.10s, 136.78KB read
  Socket errors: connect 0, read 0, write 0, timeout 1132
Requests/sec:     39.44
Transfer/sec:      4.54KB
```



- 39.44 QPS
- 1.25s Avg Latency

**小结**

- 从自身角度：QPS 稳定在 40 QPS 不到，这个是为什么呢？在上文中，我们也提到，Reactor 内置的 parallel 线程，线程池大小是 4 ，因为逻辑里 sleep 了 100ms ，所以每个线程每秒能处理 10 个请求，这个就导致最大 QPS 最大就在 10 * 4 = 40 。

## 5.3 Webflux + Netty

Webflux + Netty 的组合，效果和 Webflux + Tomcat 是一致的，就不重复测试了。

# 6. 第三轮

看到此处，可能有胖友就懵逼了，“哎呀，Webflux 什么情况，性能这么差，是不是你测试错了呀？”。我们来使用 Reactor elastic 调度器，看看性能情况。示例代码如下：



```
@GetMapping("/sleep2")
public Mono<String> sleep2() {
    return Mono.defer(() -> {
        try {
            Thread.sleep(100L);
        } catch (InterruptedException ignored) {
        }
        return Mono.just("world");
    }).subscribeOn(Schedulers.elastic());
}
```



- 怎么理解 Reactor 内置的 elastic 调度器呢？胖友可以先简单理解成 ExecutorService 的 newCachedThreadPool 线程池，一个无限大的线程池。我们来想想下，如果使用了它，那么每个请求会进入线程池中，进行 sleep 100ms ，然后完成返回，并且**不限量**，这个非常关键！

下面，开始我们的表演。这里，我们依然使用 Webflux + Tomcat 的组合。

🚚 **300 并发**



```
$wrk -t50 -c300 -d30s http://127.0.0.1:8080/sleep2

Running 30s test @ http://127.0.0.1:8080/sleep2
  50 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   100.72ms    1.94ms 300.90ms   97.04%
    Req/Sec    59.47      3.25   102.00     95.77%
  89122 requests in 30.10s, 10.04MB read
Requests/sec:   2961.05
Transfer/sec:    341.58KB
```



- 2961.05 QPS
- 100.72ms Avg Latency

🚚 **1000 并发**



```
$wrk -t50 -c1000 -d30s http://127.0.0.1:8080/sleep2

Running 30s test @ http://127.0.0.1:8080/sleep2
  50 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   104.23ms   14.70ms 353.66ms   95.03%
    Req/Sec   193.77     20.42   252.00     91.85%
  288903 requests in 30.10s, 32.55MB read
Requests/sec:   9598.51
Transfer/sec:      1.08MB
```



- 9598.51 QPS
- 104.23ms Avg Latency

🚚 **3000 并发**



```
$wrk -t50 -c3000 -d30s http://127.0.0.1:8080/sleep2

./wrk -t50 -c3000 -d30s http://127.0.0.1:8080/sleep2
Running 30s test @ http://127.0.0.1:8080/sleep2
  50 threads and 3000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   185.33ms  103.05ms   1.30s    89.87%
    Req/Sec   345.33    160.48   669.00     61.51%
  468276 requests in 30.10s, 52.75MB read
Requests/sec:  15555.70
Transfer/sec:      1.75MB
```



- 15555.70 QPS
- 185.33ms Avg Latency

🚚 **5000 并发**



```
$wrk -t50 -c5000 -d30s http://127.0.0.1:8080/sleep2

[root@iZuf6hci646px19gg3hpuwZ wrk]# ./wrk -t50 -c5000 -d30s http://127.0.0.1:8080/sleep2
Running 30s test @ http://127.0.0.1:8080/sleep2
  50 threads and 5000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   269.61ms  157.51ms   1.90s    86.07%
    Req/Sec   349.51    232.46     1.01k    62.82%
  455185 requests in 30.10s, 51.28MB read
Requests/sec:  15122.01
Transfer/sec:      1.70MB
```



- 15122.01 QPS
- 269.61ms Avg Latency

**小结**

- 从自身角度：
  - 300 和 1000 并发的时候，并发和 QPS 基本是 1:10 ，这是因为每个请求 sleep 100ms ，那么在线程池足够大小的时候，最大 QPS 是并发 * 10 ，所以 300 并发时，是 3000 QPS 不到，1000 并发时，是 10000 QPS 不到。
  - 3000 和 5000 并发的时候，并发量在 15000 QPS 左右，趋于稳定。不过按照上面并发和 QPS 是 1:10 的说法，应该要能达到 30000 或 50000 QPS ，但是从实际自己打的日志，到不了这么多线程数。再具体的原因，就暂时没去分析。
- 对比 SpringMVC + Tomcat 角度：Webflux 比 SpringMVC 的 QPS 提升很多。我们从示例中，也可以看到，得益于可以将请求提交到 Reactor **无限大**的线程池中，从而提高并发性能。不过，我们也一定要清楚这一点，为什么 Webflux 性能得到了提升。当然，我们也可以使用 SpringMVC + Servlet 3.1 的特性，也能实现类似的效果，只是说会麻烦一些。