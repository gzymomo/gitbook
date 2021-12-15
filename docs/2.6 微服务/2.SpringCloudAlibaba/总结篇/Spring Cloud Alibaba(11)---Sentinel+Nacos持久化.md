[Spring Cloud Alibaba(11)---Sentinel+Nacos持久化](https://www.cnblogs.com/qdhxhz/p/14758708.html)

##  一、Sentinel+持久化原理

#### 1、为什么需要持久化

前面我们搭建过Nacos + Mysql持久化,因为Nacos默认是将配置数据写在内存中的,所以当Nacos一重启,所有配置信息都会丢失。同样的原因。Sentinel默认也是将规则推送至

客户端并直接更新到内存中，所以客户端一重启，规则即消失。在生产环境肯定是需要持久化配置它。

#### 2、官方配置持久化的方式

官方文档 [在生产环境中使用 Sentinel](https://github.com/alibaba/Sentinel/wiki/在生产环境中使用-Sentinel)

官方给出了两种方式: `Pull模式` 和 `Push模式`

**1)、Pull模式**

```
说明
```

pull 模式的数据源（如本地文件、RDBMS 等）一般是可写入的。 客户端主动向某个规则管理中心(如本地文件、RDBMS 等)**定期轮询拉取规则**。我们既可以在应用本地直接

修改文件来更新规则，也可以通过 Sentinel 控制台推送规则。以本地文件数据源为例，推送过程如下图所示：

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210512101759563-1228942516.jpg)

首先 Sentinel 控制台通过 API 将规则推送至客户端并更新到内存中，接着注册的写数据源会将新的规则保存到本地的文件中。使用 pull 模式的数据源时一般不需要对 Sentinel

控制台进行改造。

**优点**：简单，无任何依赖；规则持久化

**缺点**：不保证一致性(无法保证同步)；实时性不保证(毕竟是轮询)，拉取过于频繁也可能会有性能问题。

**2)、Push模式**

规则中心统一推送，客户端通过注册监听器的方式时刻监听变化，比如使用 Nacos、Zookeeper 等配置中心。这种方式有更好的实时性和一致性保证。

**生产环境下一般采用 push 模式的数据源**。

同时官方也建议，推送的操作不应由 Sentinel 客户端进行，而应该经控制台统一进行管理，直接进行推送，数据源仅负责获取配置中心推送的配置并更新到本地。因此推送规则正确

做法应该是 配置中心控制台/Sentinel 控制台 → 配置中心 → Sentinel 数据源 → Sentinel，而不是经 Sentinel 数据源推送至配置中心。这样的流程就非常清晰了：

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210512101832186-1975585027.jpg)

**优点**：规则持久化；一致性；快速

**缺点**：引入第三方依赖



## 二、Sentinel+Nacos持久化配置

#### 1、pom.xml

之前有关Sentinel和Nacos相关jar包已经添加过 ，所以只添加需要Sentinel和Nacos整合的包。

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

#### 2、bootstrap.yml

```yaml
# Spring
spring:
  application: # 应用名称
    name: mall-goods

  profiles: # 环境配置
    active: dev

  cloud:
    nacos:
      discovery:
        # 服务注册地址
        server-addr: 127.0.0.1:8848
      config:
        # 配置中心地址
        server-addr: 127.0.0.1:8848
        # 配置文件格式
        file-extension: yml
    sentinel:
      # 取消控制台懒加载
      eager: true
      transport:
        # 控制台地址
        dashboard: 127.0.0.1:8282
      # nacos配置持久化
      datasource:
        ds1:
          nacos:
            server-addr: 127.0.0.1:8848
            dataId: ${spring.application.name}-SENTINEL.json
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
```

#### 3、Naocs控制台添加配置

因为上面配置的  dataId: ${spring.application.name}-SENTINEL.json，所以这里在Nacos创建该 **mall-goods-SENTINEL.json** 配置集

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210512101913151-880040937.jpg)

然后在配置集中添加配置

```yaml
[
    {
        "resource": "/api/v1/sentinel/test-sentinel",
        "limitApp": "default",
        "grade": "1",
        "count": "5",
        "strategy": "0",
        "controlBehavior": "0",
        "clusterMode": false
    }
]
```

相关属性说明

```
resource：资源名称
limitApp：来源应用
grade：阀值类型，0：线程数，1：QPS
count：单机阀值
strategy：流控模式，0：直接，1：关联，2：链路
controlBehavior：流控效果，0：快速失败，1：warmUp，2：排队等待
clusterMode：是否集群
```

#### 4、查看Sentinel控制台

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210512102042071-929735944.jpg)

以上都配置好后，我们启动Sentinel就可以看到 ，上面的这个限流规则已经在Sentinel控制台了，所以我们在Nacos配置的限流规则，已经推送到了Sentinel控制台。因为我们

Naocs已经通过Mysql进行持久化所以配置的这个限流规则，会永远存在，不会因为重启而丢失,这样就保证了规则的持久化。

#### 5、补充

1)、其实如果以上都配置好后，我们发现如果我们在Sentinel控制台配置一条限流规则，这个限流规则不会主动推送到Nacos。所以我们每次需要在Nacos配置规则然后会推送到

Sentinel。但是Nacos中的规则需要我们手动添加，这样很不方便。我们希望做到当然是我们是在Sentinel控制台添加熔断规则,自动将熔断规则推送到Nacos数据源。这样当然也是

可以的。这样的话改动的会多点，这里就不做演示了 具体可以网上找找。

2)、这里只演示了一个限流规则，如果你要添加熔断规则等等其它规则，一样也是可以的。



`github地址`  [nacos-feign-sentinel](https://github.com/yudiandemingzi/spring-cloud-alibaba-study/tree/nacos-feign-sentinel)