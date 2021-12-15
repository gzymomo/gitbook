



# 一、背景 

对很多人来说，未知、不确定、不在掌控的东西，会有潜意识的逃避。当我第一次接触 Prometheus 的时候也有类似的感觉。对初学者来说， Prometheus 包含的概念太多了，门槛也太高了。

> 概念：Instance、Job、Metric、Metric Name、Metric Label、Metric Value、Metric Type（Counter、Gauge、Histogram、Summary）、DataType（Instant Vector、Range Vector、Scalar、String）、Operator、Function

马云说：“虽然阿里巴巴是全球最大的零售平台，但阿里不是零售公司，是一家数据公司”。

Prometheus 也是一样，本质来说是一个基于数据的监控系统。



## 1.1 Promethues介绍

> Prometheus 是一套开源的系统监控报警框架。它启发于 Google 的 borgmon 监控系统，由工作在 SoundCloud 的 google 前员工在 2012 年创建，作为社区开源项目进行开发，并于 2015 年正式发布。2016 年，Prometheus 正式加入 Cloud Native Computing Foundation，成为受欢迎度仅次于 Kubernetes 的项目。



作为新一代的监控框架，Prometheus 具有以下特点：

- 强大的多维度数据模型： 时间序列数据通过 metric 名和键值对来区分。 所有的 metrics 都可以设置任意的多维标签。
- 数据模型更随意，不需要刻意设置为以点分隔的字符串。 可以对数据模型进行聚合，切割和切片操作。
- 支持双精度浮点类型，标签可以设为全unicode。 灵活而强大的查询语句（PromQL）：在同一个查询语句，可以对多个 metrics进行乘法、加法、连接、取分数位等操作。
- 易于管理： Prometheus server是一个单独的二进制文件，可直接在本地工作，不依赖于分布式存储。 高效：平均每个采样点仅占 3.5 bytes，且一个 Prometheus server 可以处理数百万的 metrics。 使用 pull模式采集时间序列数据，这样不仅有利于本机测试而且可以避免有问题的服务器推送坏的 metrics。 可以采用 push gateway 的方式把时间序列数据推送至 Prometheus server 端。 可以通过服务发现或者静态配置去获取监控的 targets。
- 有多种可视化图形界面。 易于伸缩。 需要指出的是，由于数据采集可能会有丢失，所以 Prometheus 不适用对采集数据要 100%
- 准确的情形。但如果用于记录时间序列数据，Prometheus 具有很大的查询优势，此外，Prometheus 适用于微服务的体系架构。

**示例图:**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130161151142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FhendzeHBjbQ==,size_16,color_FFFFFF,t_70)



## 1.2 Prometheus的适用场景

- 在选择Prometheus作为监控工具前，要明确它的适用范围，以及不适用的场景。
- Prometheus在记录纯数值时间序列方面表现非常好。它既适用于以服务器为中心的监控，也适用于高动态的面向服务架构的监控。
- 在微服务的监控上，Prometheus对多维度数据采集及查询的支持也是特殊的优势。
- Prometheus更强调可靠性，即使在故障的情况下也能查看系统的统计信息。权衡利弊，以可能丢失少量数据为代价确保整个系统的可用性。因此，它不适用于对数据准确率要求100%的系统，比如实时计费系统（涉及到钱）。



## 1.3 Prometheus核心组件介绍

### Prometheus Server:

> Prometheus Server是Prometheus组件中的核心部分，负责实现对监控数据的获取，存储以及查询。 Prometheus Server可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。其次Prometheus Server需要对采集到的监控数据进行存储，Prometheus Server本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。最后Prometheus Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。 Prometheus Server内置的Express Browser UI，通过这个UI可以直接通过PromQL实现数据的查询以及可视化。 Prometheus Server的联邦集群能力可以使其从其他的Prometheus Server实例中获取数据，因此在大规模监控的情况下，可以通过联邦集群以及功能分区的方式对Prometheus Server进行扩展。

### Exporters:

> Exporter将监控数据采集的端点通过HTTP服务的形式暴露给Prometheus Server，Prometheus
> Server通过访问该Exporter提供的Endpoint端点，即可获取到需要采集的监控数据。 一般来说可以将Exporter分为2类：
> 直接采集：这一类Exporter直接内置了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
> 间接采集：间接采集，原有监控目标并不直接支持Prometheus，因此我们需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

### PushGateway:

> 在Prometheus Server中支持基于PromQL创建告警规则，如果满足PromQL定义的规则，则会产生一条告警，而告警的后续处理流程则由AlertManager进行管理。在AlertManager中我们可以与邮件，Slack等等内置的通知方式进行集成，也可以通过Webhook自定义告警处理方式。

### Service Discovery:

> 服务发现在Prometheus中是特别重要的一个部分，基于Pull模型的抓取方式，需要在Prometheus中配置大量的抓取节点信息才可以进行数据收集。有了服务发现后，用户通过服务发现和注册的工具对成百上千的节点进行服务注册，并最终将注册中心的地址配置在Prometheus的配置文件中，大大简化了配置文件的复杂程度，
> 也可以更好的管理各种服务。 在众多云平台中（AWS,OpenStack），Prometheus可以
> 通过平台自身的API直接自动发现运行于平台上的各种服务，并抓取他们的信息Kubernetes掌握并管理着所有的容器以及服务信息，那此时Prometheus只需要与Kubernetes打交道就可以找到所有需要监控的容器以及服务对象.
>
> - Consul（官方推荐）等服务发现注册软件
> - 通过DNS进行服务发现
> - 通过静态配置文件（在服务节点规模不大的情况下）

## 1.4 Prometheus UI

Prometheus UI是Prometheus内置的一个可视化管理界面，通过Prometheus UI用户能够轻松的了解Prometheus当前的配置，监控任务运行状态等。 通过Graph面板，用户还能直接使用**PromQL**实时查询监控数据。访问ServerIP:9090/graph打开WEB页面，通过PromQL可以查询数据，可以进行基础的数据展示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130161210554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FhendzeHBjbQ==,size_16,color_FFFFFF,t_70)

如下所示，查询主机负载变化情况，可以使用关键字node_load1可以查询出Prometheus采集到的主机负载的样本数据，这些样本数据按照时间先后顺序展示，形成了主机负载随时间变化的趋势图表：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201130161230711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FhendzeHBjbQ==,size_16,color_FFFFFF,t_70)



# 二、日常监控

假设需要监控 WebServerA 每个API的请求量为例，需要监控的维度包括：服务名（job）、实例IP（instance）、API名（handler）、方法（method）、返回码(code)、请求量（value）。

![img](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZff6g91Tpu8esnrLn2SMCkyHenx6P0VrpC6xRr913YWghX2X5SOsyWXziakWEE0tpmbM3xk5ABeiabhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)promql

如果以SQL为例，演示常见的查询操作：

1. 查询 method=put 且 code=200 的请求量(红框)

   > SELECT * from http_requests_total WHERE code=”200” AND method=”put” AND created_at BETWEEN 1495435700 AND 1495435710;

2. 查询 handler=prometheus 且 method=post 的请求量(绿框)

   > SELECT * from http_requests_total WHERE handler=”prometheus” AND method=”post” AND created_at BETWEEN 1495435700 AND 1495435710;

3. 查询 instance=10.59.8.110 且 handler 以 query 开头 的请求量(绿框)

   > SELECT * from http_requests_total WHERE handler=”query” AND instance=”10.59.8.110” AND created_at BETWEEN 1495435700 AND 1495435710;

通过以上示例可以看出，在常用查询和统计方面，日常监控多用于根据监控的维度进行查询与时间进行组合查询。**如果监控100个服务，平均每个服务部署10个实例，每个服务有20个API，4个方法，30秒收集一次数据，保留60天。那么总数据条数为：100(服务)\* 10（实例）\* 20（API）\* 4（方法）\* 86400（1天秒数）\* 60(天) / 30（秒）= 138.24 亿条数据，写入、存储、查询如此量级的数据是不可能在Mysql类的关系数据库上完成的**。因此 Prometheus 使用 TSDB 作为 存储引擎

# 三、存储引擎

TSDB 作为 Prometheus 的存储引擎完美契合了监控数据的应用场景

- 存储的数据量级十分庞大
- 大部分时间都是写入操作
- 写入操作几乎是顺序添加，大多数时候数据到达后都以时间排序
- 写操作很少写入很久之前的数据，也很少更新数据。大多数情况在数据被采集到数秒或者数分钟后就会被写入数据库
- 删除操作一般为区块删除，选定开始的历史时间并指定后续的区块。很少单独删除某个时间或者分开的随机时间的数据
- 基本数据大，一般超过内存大小。一般选取的只是其一小部分且没有规律，缓存几乎不起任何作用
- 读操作是十分典型的升序或者降序的顺序读
- 高并发的读操作十分常见

那么 TSDB 是怎么实现以上功能的呢？

```json
"labels": [{
 "latency":        "500"
}]
"samples":[{
 "timestamp": 1473305798,
 "value": 0.9
}]
```

原始数据分为两部分 label, samples。前者记录监控的维度（标签:标签值），指标名称和标签的可选键值对唯一确定一条时间序列（使用 series_id 代表）；后者包含包含了时间戳（timestamp）和指标值（value）。

```
series
^
│. . . . . . . . . . . .   server{latency="500"}
│. . . . . . . . . . . .   server{latency="300"}
│. . . . . . . . . .   .   server{}
│. . . . . . . . . . . .
v
<-------- time ---------->
```

TSDB 使用 timeseries:doc:: 为 key 存储 value。为了加速常见查询查询操作：label 和 时间范围结合。TSDB 额外构建了三种索引：`Series`, `Label Index` 和 `Time Index`。

以标签 `latency` 为例：

- Series

  > 存储两部分数据。一部分是按照字典序的排列的所有标签键值对序列（series）；另外一部分是时间线到数据文件的索引，按照时间窗口切割存储数据块记录的具体位置信息，因此在查询时可以快速跳过大量非查询窗口的记录数据

- Label Index

  > 每对 label 为会以 index:label: 为 key，存储该标签所有值的列表，并通过引用指向 `Series` 该值的起始位置。

- Time Index

  > 数据会以 index:timeseries:: 为 key，指向对应时间段的数据文件

# 四、数据计算

强大的存储引擎为数据计算提供了完美的助力，使得 Prometheus 与其他监控服务完全不同。Prometheus 可以查询出不同的数据序列，然后再加上基础的运算符，以及强大的函数，就可以执行 `metric series` 的矩阵运算（见下图）。

![img](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZff6g91Tpu8esnrLn2SMCkyHHxpXibG4ib6S7EwIJoOPwAGXImUCsib0yfxv9DQhXnOdmPSxR8qydPywQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)time series matrix

如此，Promtheus体系的能力不弱于监控界的“数据仓库”+“计算平台”。因此，在大数据的开始在业界得到应用，就能明白，这就是监控未来的方向。

# 五、一次计算，处处查询

当然，如此强大的计算能力，消耗的资源也是挺恐怖的。因此，查询预计算结果通常比每次需要原始表达式都要快得多，尤其是在仪表盘和告警规则的适用场景中，仪表盘每次刷新都需要重复查询相同的表达式，告警规则每次运算也是如此。因此，Prometheus提供了 Recoding rules，可以预先计算经常需要或者计算量大的表达式，并将其结果保存为一组新的时间序列， 达到 一次计算，多次查询 的目的

# 六、其他配置

## 6.1 prometheus动态加载配置

Prometheus数据源的配置主要分为静态配置和动态发现, 常用的为以下几类:

- static_configs: 静态服务发现
- file_sd_configs: 文件服务发现
- dns_sd_configs: DNS 服务发现
- kubernetes_sd_configs: Kubernetes 服务发现
- consul_sd_configs:Consul 服务发现（推荐使用）

file_sd_configs的方式提供简单的接口，可以实现在单独的配置文件中配置拉取对象，并监视这些文件的变化并自动加载变化。基于这个机制，我们可以自行开发程序，监控监控对象的变化自动生成配置文件，实现监控对象的自动发现。

在prometheus文件夹目录下创建targets.json文件
配置如下:

```json
[
    {
      "targets": [ "192.168.214.129:9100"],
      "labels": {
        "instance": "node",
        "job": "server-129"
      }
    },
    {
        "targets": [ "192.168.214.134:9100"],
        "labels": {
          "instance": "node",
          "job": "server-134"
        }
      }
  ]
```

然后在prometheus目录下新增如下配置:

```yaml
 - job_name: 'file_sd'   
    file_sd_configs:
      - files:
        - targets.json  
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020113023501971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FhendzeHBjbQ==,size_16,color_FFFFFF,t_70)