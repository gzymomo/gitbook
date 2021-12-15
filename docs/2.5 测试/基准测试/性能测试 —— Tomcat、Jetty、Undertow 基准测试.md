# 1. 概述

在 Spring Boot 中，内置了三种 Servlet 容器：Tomcat、Jetty、Undertow ，也就是本文的三位主角。实际生产中，我们绝大多数情况，使用的都是 Tomcat ，因为我们很多人，初始学习的都是 Tomcat ，比较偶尔听到 Jetty ，非常小众知道有 Undertow 这个容器。那么，抛出他们的具体实现不说，我们来一起测试一下，它们的性能差别有多少。

在阅读本文之前，希望胖友已经阅读过 [《性能测试 —— Nginx 基准测试》](http://www.iocoder.cn/Performance-Testing/Nginx-benchmark/self) 文章，一方面我们会继续使用 **wrk** 这个 HTTP 压力测试工具，另一方面 **Nginx** 本身是个性能非常强劲的 Web 服务器，可以作为性能的比较参考。

# 2. 性能指标

和 [《性能测试 —— Nginx 基准测试》](http://www.iocoder.cn/Performance-Testing/Nginx-benchmark/self) 保持一致，我们还是以 **QPS** 作为性能的指标。

在 https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-05-benchmark-tomcat-jetty-undertow 中，我们提供了四个示例，分别是：

- lab-05-tomcat ：Tomcat 9.0.16 NIO 模式。
- lab-05-tomcat-apr ：Tomcat 9.0.16 APR 模式。
- lab-05-jetty ：Jetty 9.4.14.v20181114 。
- lab-05-undertow ：Undertow 2.0.17.Final 。

胖友可以使用 `mvn package` 命令，打包出不同的示例，进行压力测试。

另外，上述示例，我们在 Controller 提供了简单的 HTTP Restful API 接口，返回简单的字符串。如下：

```
@RestController
public class Controller {

    @GetMapping("/hello")
    public String hello() {
        return "world";
    }

}
```

# 3. 测试环境

- 型号 ：ecs.c5.xlarge

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：40 GB ESSD 云盘

- JDK ：openjdk version “1.8.0_212”

- JVM 参数 ：`-Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k`

因为我们在跑的过程，发现 wrk 占用 CPU 不是很高，所以直接本机运行。

有一点要注意，JVM 本身有[预热](https://codeday.me/bug/20180203/128666.html)的过程，Tomcat、Jetty、Undertow 本也有预热的过程（例如说，线程的初始化），所以需要多次测试，取平均值。

本文，我们使用 wrk 如下命令进行测试：

```
./wrk -t50 -c400 -d30s http://127.0.0.1:8080
```

- `-t50` 参数，设置 50 并发线程。
- `-c400` 参数，设置 400 连接。
- `-d30s` 参数，设置执行 30s 的时长的 HTTP 请求。
- `http://127.0.0.1:8080` 参数，请求本地的 Web 服务。
- 和 [《性能测试 —— Nginx 基准测试》](http://www.iocoder.cn/Performance-Testing/Nginx-benchmark/self) 基本保持一致。
- 因为偷懒，所以没有跑多种并发( `-c` 设置成不同)的情况。严格来说，胖友可以去尝试下，不同的并发量的情况下，各个 Web 服务的 QPS 上升还是下滑。

# 3. Tomcat NIO

> FROM [《Java 应用服务器 Tomcat》](https://www.oschina.net/p/tomcat)
>
> Tomcat 是一个小型的轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache 服务器，可利用它响应对HTML 页面的访问请求。实际上Tomcat 部分是Apache 服务器的扩展，但它是独立运行的，所以当你运行tomcat 时，它实际上作为一个与Apache 独立的进程单独运行的。

可能会有胖友有疑惑，Tomcat 为什么还分不同的模式。实际上，Tomcat 一共分成三种模式：

> 参考资料 [《tomcat bio nio apr 模式性能测试与个人看法》](http://www.iocoder.cn/Performance-Testing/Tomcat-Jetty-Undertow-benchmark/tomcat bio nio apr 模式性能测试与个人看法) 文章。

- BIO (blocking I/O)模式：阻塞式 I/O 操作，表示 Tomcat 使用的是传统的 Java I/O 操作（即 `java.io` 包）。在 Tomcat 7 以及以前版本，默认使用 BIO 模式运行。一般情况下，BIO 模式是三种运行模式中性能最低的。

  > 也因为 BIO 模式，基本是性能最低的，所以本文我们也不进行测试，节约时间。

- NIO (Non-blocking I/O)模式：非阻塞时 I/O 操作，表示 Tomcat 使用的是 JDK 1.4 后新提供的 I/O 操作（即 `java.nio` 包）。在 Tomcat 8 以及到目前最新版本的 Tomcat 9 ，默认使用的都是 NIO 模式运行。相比 BIO 模式来说，它能提供更好的并发性能。

- APR (Apache Portable Runtime/Apache 可移植运行库) 模式：APR 是 [Apache](https://httpd.apache.org/) HTTP 服务器的支持库。我们可以理解成，Tomcat 将以 [JNI](https://zh.wikipedia.org/zh-hans/Java本地接口) 的方式，调用 Apache HTTP 服务器的核心动态链接库来处理文件或网络传输，从而大大的提升 Tomcat 对静态文件的处理性能。Tomcat APR 模式，也是 Tomcat 上运行高并发应用的首选模式。

  > 按照这个说法，现在前后端分离后，我们使用 Tomcat + SpringMVC 提供 Restful API ，所以 APR 模式，处理静态文件的能力，未必能带来性能上的提升。当然，还是测试出真知。

下面，我们开始正式的测试。启动 Tomcat 服务比较简单。如下：

```
java -jar lab-05-tomcat-1.0-SNAPSHOT.jar -Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k
```

然后，执行 wrk 进行性能测试，结果如下：

```
$ ./wrk -t50 -c400 -d30s http://127.0.0.1:8080/hello

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    13.81ms    7.09ms 159.52ms   86.27%
    Req/Sec   594.55     62.11     1.98k    78.31%
  890078 requests in 30.09s, 100.32MB read
Requests/sec:  29575.99
Transfer/sec:      3.33MB
```

- QPS 为 29575.99 。
- 平均延迟为 13.81ms。

# 4. Tomcat APR

相比 Tomcat NIO 来说，Tomcat APR 会相对麻烦一些，需要多三个步骤：

> 参考自 [《Apache Tomcat Native Library》](http://tomcat.apache.org/native-doc/) 文章。

- 1、安装 APR 。

  > 这个就是我们说到的 Apache Portable Runtime/Apache 可移植运行库 。

  ```
  yum install apr-devel -f
  ```

  - 安装的版本是，1.4.8-3.el7_4.1 。

- 2、安装 Tomcat Native 。

  > 这个就是我们说到的 Tomcat 将以 JNI 的方式调用 APR 的实现库。

  ```
  yum install tomcat-native
  ```

  - 安装的版本是，1.2.17-1.el7 。

- 3、设置 Tomcat 使用 APR 。如果在 Spring Boot 使用 Tomcat 内嵌服务器，可以参考 [TomcatAprApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-05-benchmark-tomcat-jetty-undertow/lab-05-tomcat-apr/src/main/java/cn/iocoder/springboot/labs/lab05/tomcat/TomcatAprApplication.java) 。

下面，我们开始正式的测试。启动 Tomcat 服务比较简单。如下：

```
java -jar lab-05-tomcat-apr-1.0-SNAPSHOT.jar -Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k
```

然后，执行 wrk 进行性能测试，结果如下：

```
$ ./wrk -t50 -c400 -d30s http://127.0.0.1:8080

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    12.15ms    5.12ms 229.46ms   87.59%
    Req/Sec   667.77     62.65     2.01k    80.34%
  999673 requests in 30.10s, 112.67MB read
Requests/sec:  33213.47
Transfer/sec:      3.74MB
```

- QPS 为 33213.47 。
- 平均延迟为 12.15ms。

相比 Tomcat NIO 模式，大概有 3500 QPS 左右的提升，12% 左右的性能提升。

当然，生产环境，我们实际使用 Tomcat NIO 模式，已经能够满足我们的诉求，不一定需要修改成 Tomcat APR 模式，因为相对麻烦一些。

# 5. Jetty

> FROM [《Servlet 容器 Jetty》](https://www.oschina.net/p/jetty)
>
> Jetty 是一个开源的 servlet 容器，它为基于 Java 的 web 内容，例如 JSP 和 servlet 提供运行环境。Jetty 是使用 Java 语言编写的，它的 API 以一组 JAR 包的形式发布。开发人员可以将 Jetty 容器实例化成一个对象，可以迅速为一些独立运行（stand-alone）的 Java 应用提供网络和 web 连接。

下面，我们开始正式的测试。启动 Jetty 服务比较简单。如下：

```
java -jar lab-05-jetty-apr-1.0-SNAPSHOT.jar -Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k
```

然后，执行 wrk 进行性能测试，结果如下：

```
$ ./wrk -t50 -c400 -d30s http://127.0.0.1:8080

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     9.61ms    4.82ms 110.04ms   73.24%
    Req/Sec   849.00    167.82     5.16k    87.43%
  1270290 requests in 30.11s, 145.37MB read
Requests/sec:  42193.37
Transfer/sec:      4.83MB
```

- QPS 为 42193.37 。
- 平均延迟为 9.61ms 毫秒。

相比 Tomcat NIO 模式，大概有 12617.38 QPS 提升，43% 左右的性能提升。牛逼闪电啊！！！

# 6. Undertow

> FROM [《嵌入式 Web 服务器 Undertow》](https://www.oschina.net/p/undertow)
>
> Undertow 是一个采用 Java 开发的灵活的高性能 Web 服务器，提供包括阻塞和基于 NIO 的非堵塞机制。Undertow 是红帽公司的开源产品，是 Wildfly 默认的 Web 服务器。
>
> Undertow 提供一个基础的架构用来构建 Web 服务器，这是一个完全为嵌入式设计的项目，提供易用的构建器 API，完全兼容 Java EE Servlet 3.1 和低级非堵塞的处理器。

下面，我们开始正式的测试。启动 Undertow 服务比较简单。如下：

```
java -jar lab-05-undertow-1.0.0.jar -Xms2g -Xmx2g -Xmn1g -XX:MaxMetaspaceSize=256m -Xss256k
```

然后，执行 wrk 进行性能测试，结果如下：

```
$ ./wrk -t50 -c400 -d30s http://127.0.0.1:8080

Running 30s test @ http://127.0.0.1:8080/hello
  50 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    10.09ms    4.36ms 207.32ms   74.34%
    Req/Sec   802.76     99.60     2.16k    73.54%
  1198795 requests in 30.10s, 164.63MB read
Requests/sec:  39822.02
Transfer/sec:      5.47MB
```

- QPS 为 39822.02 。
- 平均延迟为 18.46 毫秒。

相比 Tomcat NIO 模式，大概有 10246.03 QPS 提升，34% 左右的性能提升。也牛逼闪电啊！！！