# 2. 性能指标

在 Dubbo 官方团队提供的 [《Dubbo 性能测试报告》](http://dubbo.apache.org/zh-cn/docs/user/perf-test.html) 的文章里，我们比较明确的可以看到希望的性能指标：

> | 场景名称 | 对应指标名称 | 期望值范围 | 实际值 | 是否满足期望(是/否) |
> | :------- | :----------- | :--------- | :----- | :------------------ |
> | 1k数据   | 响应时间     | 0.9ms      | 0.79ms | 是                  |
> | 1k数据   | TPS          | 10000      | 11994  | 是                  |

# 3. 测试工具

目前可用于 Dubbo 测试的工具如下：

- [dubbo-benchmark](https://github.com/apache/dubbo-benchmark) ：Dubbo 官方，基于 [JMH](https://openjdk.java.net/projects/code-tools/jmh/) 实现的 Dubbo 性能基准测试工具。

  > 对 JMH 不了解的胖友，可以看看 forever alone 的基友写的 [《JAVA 拾遗 — JMH 与 8 个测试陷阱》](https://www.cnkirito.moe/java-jmh/)

- [jmeter-plugins-for-apache-dubbo](https://github.com/dubbo/jmeter-plugins-for-apache-dubbo) ：社区贡献，压力测试工具 [Jmeter](https://jmeter.apache.org/) 对 Dubbo 的插件拓展。

考虑到测试的简便性，以及学习成本（大多数人不会使用 JMeter），所以我们采用 dubbo-benchmark ，虽然说 JMH 也好多人不会。但是，因为 dubbo-benchmark 提供了开箱即用的脚本，即使不了解 JMH ，也能很方便的快速上手。当然，还是希望胖友能去了解下 JMH ，毕竟是 Java 微基准测试框架，可以用来测试我们编写的很多代码的性能。

# 4. dubbo-benchmark

## 4.1 项目结构

在开始正式测试之前，我们先来了解下 dubbo-benchmark 项目的大体结构。![项目结构](http://www.iocoder.cn/images/Performance-Testing/2019_03_01/01.png)

分了比较多的 Maven 模块，我们将它们的关系，重新梳理如下图：![项目层级](http://www.iocoder.cn/images/Performance-Testing/2019_03_01/02.png)

**第一层 benchmark-base**

提供 Dubbo Service 的实现，如下图：![benchmark-base](http://www.iocoder.cn/images/Performance-Testing/2019_03_01/03.png)

- UserService 类中，定义了我们业务场景中常用的四种方法：

  ```
  public interface UserService {
  
      public boolean existUser(String email);
  
      public boolean createUser(User user);
  
      public User getUser(long id);
  
      public Page<User> listUser(int pageNo);
  
  }
  ```

  

  - [UserServiceImpl](https://github.com/apache/dubbo-benchmark/blob/master/benchmark-base/src/main/java/org/apache/dubbo/benchmark/service/UserServiceServerImpl.java) 的实现，胖友自己看下，比较简单。

- AbstractClient，理论来说，应该放到 client-base 中，可能迷路了。

**第二层 client-base**

实现 Dubbo 消费端的，基于 JMH ，实现 Benchmark 基类。重点在 `benchmark.Client` 类，代码如下：



```
private static final int CONCURRENCY = 32;

public static void main(String[] args) throws Exception {
    Options opt;
    ChainedOptionsBuilder optBuilder = new OptionsBuilder()
            // benchmark 所在的类名，此处就是 Client
            .include(Client.class.getSimpleName())
            // 预热 3 轮，每轮 10 秒
            .warmupIterations(3)
            .warmupTime(TimeValue.seconds(10))
            // 测量（测试）3 轮，每轮 10 秒
            .measurementIterations(3)
            .measurementTime(TimeValue.seconds(10))
            // 并发线程数为 32
            .threads(CONCURRENCY)
            // 进行 fork 的次数。如果 fork 数是 2 的话，则 JMH 会 fork 出两个进程来进行测试。
            .forks(1);

    // 设置报告结果
    opt = doOptions(optBuilder).build();

    new Runner(opt).run();
}

private static ChainedOptionsBuilder doOptions(ChainedOptionsBuilder optBuilder) {
    String output = System.getProperty("benchmark.output");
    if (output != null && !output.trim().isEmpty()) {
        optBuilder.output(output);
    }
    return optBuilder;
}
```



- 胖友自己看下注释。
- 如果对 JMH 还是不了解的胖友，可以再看看如下两篇文章：
  - [《Java 微基准测试框架 JMH》](https://www.xncoding.com/2018/01/07/java/jmh.html)
  - [《Java 并发编程笔记：JMH 性能测试框架》](http://blog.dyngr.com/blog/2016/10/29/introduction-of-jmh/)

在 Client 类中，定义了对 UserService 调用的四个 Benchmark 方法，代码如下：



```
private final ClassPathXmlApplicationContext context;
private final UserService userService;

public Client() {
    // 读取 consumer.xml 配置文件，并启动 Spring 容器。这个配置文件，由子项目配置
    context = new ClassPathXmlApplicationContext("consumer.xml");
    context.start();
    // 获得 UserService Bean
    userService = (UserService) context.getBean("userService");
}

@Override
protected UserService getUserService() {
    return userService;
}

@TearDown
public void close() throws IOException {
    ProtocolConfig.destroyAll();
    context.close();
}

@Benchmark
@BenchmarkMode({Mode.Throughput, Mode.AverageTime, Mode.SampleTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Override
public boolean existUser() throws Exception {
    return super.existUser();
}

@Benchmark
@BenchmarkMode({Mode.Throughput, Mode.AverageTime, Mode.SampleTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Override
public boolean createUser() throws Exception {
    return super.createUser();
}

@Benchmark
@BenchmarkMode({Mode.Throughput, Mode.AverageTime, Mode.SampleTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Override
public User getUser() throws Exception {
    return super.getUser();
}

@Benchmark
@BenchmarkMode({Mode.Throughput, Mode.AverageTime, Mode.SampleTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Override
public Page<User> listUser() throws Exception {
    return super.listUser();
}
```



**第二层 server-base**

实现 Dubbo 消费端的，启动 Dubbo 服务。重点在 `benchmark.Server` 类，代码如下：



```
public class Server {

    public static void main(String[] args) throws InterruptedException {
        // 读取 provider.xml 配置文件，并启动 Spring 容器。这个配置文件，由子项目配置
        try (ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml")) {
            context.start();
            // sleep ，防止进程结束
            Thread.sleep(Integer.MAX_VALUE);
        }
    }

}
```



- 因为是被测方，所以无需集成到 JMH 中。

**第三层 {protocol}-{serialize}-client**

具体协议( Protocol )，使用具体序列化( Serialize ) 方式的消费者。

**第四层 {protocol}-{serialize}-server**

具体协议( Protocol )，使用具体序列化( Serialize ) 方式的提供者。

## 4.2 测试环境

- 型号 ：ecs.c5.xlarge

  > 艿艿：和我一样抠门（穷）的胖友，可以买竞价类型服务器，使用完后，做成镜像。等下次需要使用的时候，恢复一下。HOHO 。

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：40 GB ESSD 云盘

- Java ：OpenJDK Runtime Environment (build 1.8.0_212-b04)

- Dubbo ：2.6.1

  > 虽然 Dubbo 项目本身已经完成孵化，但是 dubbo-benchmark 并未更新到最新版本的 2.7.2 。所以，本文还是测试 Dubbo 2.6.1 版本。当然，这个对测试结果影响不大，妥妥的。

## 4.3 安装 dubbo-benchmark

**第一步，克隆项目**



```
git clone https://github.com/apache/dubbo-benchmark.git
cd dubbo-benchmark
```



**第二步，启动服务提供者**



```
sh benchmark.sh dubbo-kryo-server
```



会有一个编译的过程，耐心等待。

**第三步，启动服务消费者**

> 需要新启一个终端



```
sh benchmark.sh dubbo-kryo-client
```



开始 JMH 测试...整个测试过程，持续 15 分钟左右。

## 4.4 dubbo-hessianlite

本小节，我们来测试 dubbo-hessianlite-client 和 dubbo-hessianlite-server 。

> 这个组合，是我们使用 Dubbo 最主流的方式。

- 协议：[Dubbo](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/dubbo.html)
- 序列化：[hessian-lite](https://github.com/dubbo/hessian-lite/) ，Dubbo 对 [Hessian](https://www.oschina.net/p/hessian) 提供的序列化方式的性能优化和 Bug 修复。
- 通信：Netty4

**第一步，启动服务提供者**



```
sh benchmark.sh dubbo-hessianlite-server
```



**第二步，启动服务消费者**

> 需要新启一个终端



```
sh benchmark.sh dubbo-hessianlite-client
```



🔥 **测试结果**



```
Benchmark                               Mode      Cnt   Score   Error   Units
Client.createUser                      thrpt        3  16.887 ? 1.729  ops/ms
Client.existUser                       thrpt        3  47.293 ? 4.993  ops/ms
Client.getUser                         thrpt        3  19.698 ? 8.588  ops/ms
Client.listUser                        thrpt        3   3.457 ? 0.180  ops/ms
Client.createUser                       avgt        3   1.416 ? 0.308   ms/op
Client.existUser                        avgt        3   0.678 ? 0.038   ms/op
Client.getUser                          avgt        3   1.657 ? 0.359   ms/op
Client.listUser                         avgt        3   9.299 ? 0.872   ms/op
Client.createUser                     sample   499898   1.918 ? 0.007   ms/op
Client.createUser:createUser?p0.00    sample            0.279           ms/op
Client.createUser:createUser?p0.50    sample            1.448           ms/op
Client.createUser:createUser?p0.90    sample            2.613           ms/op
Client.createUser:createUser?p0.95    sample            3.027           ms/op
Client.createUser:createUser?p0.99    sample            9.732           ms/op
Client.createUser:createUser?p0.999   sample           16.876           ms/op
Client.createUser:createUser?p0.9999  sample           28.280           ms/op
Client.createUser:createUser?p1.00    sample           39.453           ms/op
Client.existUser                      sample  1376160   0.697 ? 0.002   ms/op
Client.existUser:existUser?p0.00      sample            0.094           ms/op
Client.existUser:existUser?p0.50      sample            0.647           ms/op
Client.existUser:existUser?p0.90      sample            0.842           ms/op
Client.existUser:existUser?p0.95      sample            0.921           ms/op
Client.existUser:existUser?p0.99      sample            1.425           ms/op
Client.existUser:existUser?p0.999     sample           10.355           ms/op
Client.existUser:existUser?p0.9999    sample           16.145           ms/op
Client.existUser:existUser?p1.00      sample           24.773           ms/op
Client.getUser                        sample   568869   1.686 ? 0.006   ms/op
Client.getUser:getUser?p0.00          sample            0.262           ms/op
Client.getUser:getUser?p0.50          sample            1.436           ms/op
Client.getUser:getUser?p0.90          sample            1.954           ms/op
Client.getUser:getUser?p0.95          sample            2.609           ms/op
Client.getUser:getUser?p0.99          sample            9.634           ms/op
Client.getUser:getUser?p0.999         sample           15.862           ms/op
Client.getUser:getUser?p0.9999        sample           31.217           ms/op
Client.getUser:getUser?p1.00          sample           44.302           ms/op
Client.listUser                       sample   103394   9.272 ? 0.038   ms/op
Client.listUser:listUser?p0.00        sample            1.792           ms/op
Client.listUser:listUser?p0.50        sample            9.060           ms/op
Client.listUser:listUser?p0.90        sample           14.287           ms/op
Client.listUser:listUser?p0.95        sample           15.679           ms/op
Client.listUser:listUser?p0.99        sample           17.336           ms/op
Client.listUser:listUser?p0.999       sample           30.966           ms/op
Client.listUser:listUser?p0.9999      sample           38.161           ms/op
Client.listUser:listUser?p1.00        sample           45.351           ms/op
```



## 4.5 dubbo-fst

本小节，我们来测试 dubbo-fst-client 和 dubbo-fst-server 。

> 这个组合，是我们使用 Dubbo 最主流的方式。

- 协议：[Dubbo](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/dubbo.html)
- 序列化：[FST](https://www.oschina.net/p/fst)
- 通信：Netty4

**第一步，启动服务提供者**



```
sh benchmark.sh dubbo-fst-server
```



**第二步，启动服务消费者**

> 需要新启一个终端



```
sh benchmark.sh dubbo-fst-client
```



🔥 **测试结果**



```
Benchmark                               Mode      Cnt   Score    Error   Units
Client.createUser                      thrpt        3  44.810 ?  5.152  ops/ms
Client.existUser                       thrpt        3  53.153 ? 49.787  ops/ms
Client.getUser                         thrpt        3  41.754 ?  8.210  ops/ms
Client.listUser                        thrpt        3  14.791 ?  3.458  ops/ms
Client.createUser                       avgt        3   0.719 ?  0.080   ms/op
Client.existUser                        avgt        3   0.574 ?  0.434   ms/op
Client.getUser                          avgt        3   0.703 ?  0.045   ms/op
Client.listUser                         avgt        3   2.189 ?  0.353   ms/op
Client.createUser                     sample  1267915   0.756 ?  0.002   ms/op
Client.createUser:createUser?p0.00    sample            0.099            ms/op
Client.createUser:createUser?p0.50    sample            0.678            ms/op
Client.createUser:createUser?p0.90    sample            0.888            ms/op
Client.createUser:createUser?p0.95    sample            1.034            ms/op
Client.createUser:createUser?p0.99    sample            3.383            ms/op
Client.createUser:createUser?p0.999   sample           10.502            ms/op
Client.createUser:createUser?p0.9999  sample           22.650            ms/op
Client.createUser:createUser?p1.00    sample           35.979            ms/op
Client.existUser                      sample  1461428   0.656 ?  0.002   ms/op
Client.existUser:existUser?p0.00      sample            0.077            ms/op
Client.existUser:existUser?p0.50      sample            0.504            ms/op
Client.existUser:existUser?p0.90      sample            1.128            ms/op
Client.existUser:existUser?p0.95      sample            1.516            ms/op
Client.existUser:existUser?p0.99      sample            2.802            ms/op
Client.existUser:existUser?p0.999     sample            6.452            ms/op
Client.existUser:existUser?p0.9999    sample           33.358            ms/op
Client.existUser:existUser?p1.00      sample           58.262            ms/op
Client.getUser                        sample  1270938   0.755 ?  0.003   ms/op
Client.getUser:getUser?p0.00          sample            0.084            ms/op
Client.getUser:getUser?p0.50          sample            0.588            ms/op
Client.getUser:getUser?p0.90          sample            1.034            ms/op
Client.getUser:getUser?p0.95          sample            1.626            ms/op
Client.getUser:getUser?p0.99          sample            4.473            ms/op
Client.getUser:getUser?p0.999         sample           10.830            ms/op
Client.getUser:getUser?p0.9999        sample           27.719            ms/op
Client.getUser:getUser?p1.00          sample           45.875            ms/op
Client.listUser                       sample   442763   2.166 ?  0.009   ms/op
Client.listUser:listUser?p0.00        sample            0.306            ms/op
Client.listUser:listUser?p0.50        sample            1.767            ms/op
Client.listUser:listUser?p0.90        sample            3.039            ms/op
Client.listUser:listUser?p0.95        sample            4.415            ms/op
Client.listUser:listUser?p0.99        sample           11.551            ms/op
Client.listUser:listUser?p0.999       sample           21.045            ms/op
Client.listUser:listUser?p0.9999      sample           32.702            ms/op
Client.listUser:listUser?p1.00        sample           45.089            ms/op
```



## 4.6 dubbo-kryo

本小节，我们来测试 dubbo-kryo-client 和 dubbo-kryo-server 。

- 协议：[Dubbo](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/dubbo.html)
- 序列化：[Kryo](https://www.oschina.net/p/kryo)
- 通信：Netty4

**第一步，启动服务提供者**



```
sh benchmark.sh dubbo-kryo-server
```



**第二步，启动服务消费者**

> 需要新启一个终端



```
sh benchmark.sh dubbo-kryo-client
```



🔥 **测试结果**



```
Benchmark                               Mode      Cnt   Score   Error   Units
Client.createUser                      thrpt        3  33.678 ? 2.656  ops/ms
Client.existUser                       thrpt        3  50.030 ? 3.509  ops/ms
Client.getUser                         thrpt        3  34.125 ? 4.886  ops/ms
Client.listUser                        thrpt        3  11.929 ? 1.746  ops/ms
Client.createUser                       avgt        3   0.955 ? 0.164   ms/op
Client.existUser                        avgt        3   0.642 ? 0.051   ms/op
Client.getUser                          avgt        3   0.940 ? 0.071   ms/op
Client.listUser                         avgt        3   2.603 ? 0.748   ms/op
Client.createUser                     sample   985106   0.973 ? 0.003   ms/op
Client.createUser:createUser?p0.00    sample            0.148           ms/op
Client.createUser:createUser?p0.50    sample            0.855           ms/op
Client.createUser:createUser?p0.90    sample            1.147           ms/op
Client.createUser:createUser?p0.95    sample            1.315           ms/op
Client.createUser:createUser?p0.99    sample            5.300           ms/op
Client.createUser:createUser?p0.999   sample           12.517           ms/op
Client.createUser:createUser?p0.9999  sample           21.037           ms/op
Client.createUser:createUser?p1.00    sample           31.850           ms/op
Client.existUser                      sample  1470527   0.652 ? 0.001   ms/op
Client.existUser:existUser?p0.00      sample            0.092           ms/op
Client.existUser:existUser?p0.50      sample            0.601           ms/op
Client.existUser:existUser?p0.90      sample            0.800           ms/op
Client.existUser:existUser?p0.95      sample            0.876           ms/op
Client.existUser:existUser?p0.99      sample            1.550           ms/op
Client.existUser:existUser?p0.999     sample            9.650           ms/op
Client.existUser:existUser?p0.9999    sample           14.844           ms/op
Client.existUser:existUser?p1.00      sample           30.573           ms/op
Client.getUser                        sample  1001893   0.957 ? 0.004   ms/op
Client.getUser:getUser?p0.00          sample            0.127           ms/op
Client.getUser:getUser?p0.50          sample            0.741           ms/op
Client.getUser:getUser?p0.90          sample            1.401           ms/op
Client.getUser:getUser?p0.95          sample            2.191           ms/op
Client.getUser:getUser?p0.99          sample            5.546           ms/op
Client.getUser:getUser?p0.999         sample           12.059           ms/op
Client.getUser:getUser?p0.9999        sample           24.518           ms/op
Client.getUser:getUser?p1.00          sample           49.873           ms/op
Client.listUser                       sample   363958   2.636 ? 0.013   ms/op
Client.listUser:listUser?p0.00        sample            0.390           ms/op
Client.listUser:listUser?p0.50        sample            2.071           ms/op
Client.listUser:listUser?p0.90        sample            4.084           ms/op
Client.listUser:listUser?p0.95        sample            6.947           ms/op
Client.listUser:listUser?p0.99        sample           12.403           ms/op
Client.listUser:listUser?p0.999       sample           22.940           ms/op
Client.listUser:listUser?p0.9999      sample           40.935           ms/op
Client.listUser:listUser?p1.00        sample           60.097           ms/op
```



## 4.7 小结

可能有胖友，对 JMH 的结果报告不熟悉，这里简单介绍下：

- Mode ：thrpt ，throughput 的缩写，吞吐量，单位为：ops/ms ，所以需要乘以 1000 ，换算成 QPS 。
- Mode ：avgt ，AverageTime 的缩写，每个操作🎺的时间，单位为 ms 毫秒。
- Mode ：sample ，采样分布。每一行代表，代表百分之多少(`?p`)的请求，在多少毫秒内。

整理结果如下表：![性能结果](http://www.iocoder.cn/images/Performance-Testing/2019_03_01/04.png)

- 三个测试用例，差别是序列化使用的库，所以序列化的性能，决定了整体的性能结果。
- 方法的性能排行是：existUser > getUser > createUser > listUser
- 项目的性能排行是：dubbo-fst > dubbo-kryo > dubbo-hessianlite

当然，得到这样一个测试结果，我们很自然的会有一个疑惑，既然 FST 和 Kryo 性能比 Hessian Lite 好，为什么默认选择的还是 Hessian Lite 呢？因为在 RPC 场景下，我们有跨语言的诉求，而 FST 和 Kryo 是 Java 序列化的库，不支持跨语言，而 Hessian Lite 支持。