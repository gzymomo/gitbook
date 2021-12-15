[Spring Cloud Alibaba(10)---Sentinel控制台搭建+整合SpringCloudAlibaba](https://www.cnblogs.com/qdhxhz/p/14743678.html)

上一篇博客讲了Sentinel一些概念性的东西  [Spring Cloud Alibaba(9)---Sentinel概述](https://www.cnblogs.com/qdhxhz/p/14718138.html)

这篇博客主要讲 **Sentinel控制台搭建**，和 **整合SpringCloudAlibaba来实现流量控制**、**降级控制**。至于其它比如热点配置、系统规则和授权规则等

自己去官网详细看，这里就不叙述了。

##  一、Sentinel控制台搭建 

#### 1、下载地址

官方有提供直接下载地址，我们可以下载自己需要的版本，我这边下载的版本是 **1.8.0**

```
https://github.com/alibaba/Sentinel/releases
```

#### 2、启动控制台

下载之后我们发现就是一个jar包，我们就可以用jar的方式去启动它

```
java -Dserver.port=8282 -Dcsp.sentinel.dashboard.server=localhost:8282 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-1.8.0.jar
```

`说明` 这里我通过 **8282** 端口来启动它。

#### 3、登陆控制台

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508094127945-1487783085.jpg)

```
重点` 启动之后访问 **localhost:8282**; 登录即可用户名和密码默认是`sentinel
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508094152832-1850404499.jpg)

登录之后看到左侧的菜单只有默认的一个；因为现在sentinel还没有发现其他微服务，这样一来Sentinel客户端就搭建成功了，接下来开始整合SpringCloudAlibaba。



## 二、Sentinel整合SpringCloudAlibaba

这篇也是在之前搭建好的基础上添加,这里在**mall-goods微服务**上做演示

#### 1、pom.xml

```xml
        <!--引入sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

#### 2、application.yml

```
spring:
  application:
    name: mall-goods
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8282
        port: 9999
```

#### 3、SentinelTestController

这里新增一个接口，来方便接下来测试限流、熔断等。

```java
@RestController
@RequestMapping("api/v1/sentinel")
public class SentinelTestController {

    private volatile int total = 0;
    
    @RequestMapping("test-sentinel")
    public Object findByGoodsId() {
        return total++;
    }
}
```

#### 4、测试

这个是时候我们重新启动 mall-goods微服务。重启之后我们发现Sentinel还是并没有mall-goods服务，那是因为于`Sentinel是懒加载模式`，所以需要先访问上面这个接口后才会

在控制台出现。所以这里我们访问下

```
http://localhost:6001/api/v1/sentinel/test-sentinel
```

访问之后我们再看控制台

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508094324366-1038161100.jpg)

我们可以看到控制台已经有 mall-goods 服务了。而且我们刚刚请求的接口这里也有了。这说明Sentinel **默认会把接口直接当成一个资源**。既然是这样，接下来就对这个请求

进行流控、降级、授权、热点等配置了；先来介绍如何添加流控吧。



## 三、流量控制规则及示例

`概念`  流量控制(flow control), 其原理是监控应用的**QPS**或**并发线程数**等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

**流量控制官方文档**

```
https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6
```

#### 1、规则说明

点击**+流控**的按钮，出现下面弹窗。

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508094618005-1522022210.jpg)

图中一共有6个名词

```
资源名
```

默认是请求路径，可⾃定义

```
针对来源
```

对哪个微服务进⾏限流，默认是不区分来源，全部限流。这个是针对 区分上游服务进⾏限流, ⽐ 如 商品服务 被 订单服务、⽤户服务调⽤，就可以针对来源进⾏限流。

```
阈值类型
```

其实就是通过哪种方式限流，是通过QPS呢,还是通过线程数。他们的含义我这里不在过多解释了，具体可以看这篇文章。[什么是QPS，TPS，吞吐量](https://www.jianshu.com/p/2fff42a9dfcf)

```
单机阈值
```

很好理解，就是现在每秒QPS或者线程数达到这个数量就会限流。

**举例子**

```
1)上面阈值类型设置QPS，下面单机阈值设置1。那组合的意思就是 每秒的请求数超过1会直接被限流。
2)上面阈值类型设置线程数量，下面单机阈值设置1。那组合的意思就是 当第一个线程未处理完成时，其他新开启请求的线程都将被限流。
```

线程数稍微难理解点，这里再通俗的解释下 我们对一个接口资源 **阈值类型设置线程数量，下面单机阈值设置1**。这个接口方法内有2秒钟的睡眠延迟，那么，当第一个线程未

处理完成时（即2秒内），其他新开启请求的线程都将被限流，只有第一个未限流的线程成功处理，新的请求才会进来。

```
并发数控制⽤于保护业务线程池不被慢调⽤耗尽Sentinel 并发控制不负责创建和管理线程池，⽽是简单统计当前请求上下⽂的线程数⽬（正在执⾏的调⽤数⽬）
如果超出阈值，新的请求会被⽴即拒绝，效果类似于信号量隔离。并发数控制通常在调⽤端进⾏配置
流控模式
```

这里有三种模式 **直接**    **关联**   **链路**,这个这里也不做过度解释，具体看上面官方文档。

```
流控效果
```

这里也有三种:**快速失败**、**Warm Up** 、**排队等待**。具体也看上面官方文档说明。

```
总结
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508101857182-306891270.jpg)

#### 2、测试

我这里配置如下

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508101932676-1950306380.jpg)

```
阈值类型:QPS,单机阈值:1,流控模式:直接,流控效果:快速失败
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508101941644-1789062615.jpg)

然后我们在来请求上面的接口

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508101950560-1798259030.gif)

很明显，如果一秒内有两个请求就会限流。



##  四、降级规则说明及示例

除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。

```
降级规则官方文档
https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7
```

#### 1、规则说明

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508102002408-905825812.jpg)

```
熔断策略
```

Sentinel 提供以下几种熔断策略：**慢调用比例**、**异常比例**、**异常数**

**慢调用比例**：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（**即最大的响应时间**），请求的响应时间大于该值则统计为慢调用。

**异常比例** ：当单位统计时长内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。

**异常数** ：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。

```
熔断降级规则说明
```

熔断降级规则（DegradeRule）包含下面几个重要的属性：

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508102027667-643268492.jpg)

#### 2、测试

这里添加如下规则配置

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508102035966-42810934.jpg)

然后我们新增一个接口

```java
 @RequestMapping("test-sentinel-exception")
    public Object testSentinelException() {
        int i = (int) (Math.random() * 100);
        if(i>10){
            throw new NullPointerException("随机错误");
        }
        return "成功";
    }
测试
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210508102044103-1438672175.gif)

我们可以看出 当请求超过2次异常，那么就会报熔断的异常错误。有关其它的规则我这里不在阐述了，具体的都可以看官网。



### 参考

[1、分布式系统的流量防卫兵](https://github.com/alibaba/Sentinel/wiki/介绍)