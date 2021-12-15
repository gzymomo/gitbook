[Spring Cloud Alibaba(14)---SpringCloudAlibaba整合Sleuth](https://www.cnblogs.com/qdhxhz/p/14785093.html)

这篇我们开始通过示例来演示链路追踪。

## 一、环境准备

既然是演示链路追踪，那么就需要有多个微服务之间进行调用，这里的项目也是在之间已经搭建好的基础上加上Sleuth组件，具体链路是这个的：

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210519145603674-606591731.jpg)

从图中可以看出，这里一个完整的链路是 一个请求通过`网关服务`，然后转发到 `订单微服务`，然后订单微服务中会去调`商品服务`。

所以这里涉及三个微服务

```yaml
mall-gateway: 网关服务。端口号:8001。
mall-goods: 商品服务。 端口号:6001。
mall-order: 订单服务。端口号:7001。
```

这三个服务都已经注册到nacos中，如图

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210519145612962-205066596.jpg)

## 二、SpringCloudAlibaba整合Sleuth

`注意` 这里不把所有代码都复制在这里，完整项目代码，会放到github上，在文章下方会提供地址。

#### 1、pom.xml

在需要进行链路追踪的项目中（**服务网关**、**商品服务**、**订单服务**）添加 spring-cloud-starter-sleuth 依赖。

```xml
     <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
     </dependency>
```

#### 2、测试

**访问地址如下:**

```
#通过网关访问订单服务
http://localhost:8001/mall-order/api/v1/goods_order/getGoodsByFeign?goodsId=1
```

接下来我们来看各个微服务打印的日志

```
网关服务(mall-gatway)
2021-05-19 19:17:46.677  INFO [mall-gateway,4ef9402f9a9500a1,4ef9402f9a9500a1,true] 92553 --- [ctor-http-nio-3] com.jincou.getway.CustomGatewayFilter 
订单服务(mall-order)
2021-05-19 19:17:47.284  INFO [mall-order,4ef9402f9a9500a1,94a660c5c94cffb4,true] 92561 --- [nio-7001-exec-1] c.j.order.controller.OrderController 
商品服务(mall-goods)
2021-05-19 19:17:49.077  INFO [mall-goods,4ef9402f9a9500a1,4c48de8ab2b6377a,true] 92566 --- [nio-6001-exec-1] c.j.goods.controller.GoodsController 
```

解释下含义 **[mall-gateway,4ef9402f9a9500a1,4ef9402f9a9500a1,true]**

**第⼀个值**，spring.application.name的值。

**第⼆个值**，4ef9402f9a9500a1 ，sleuth⽣成的⼀个ID，叫Trace ID，⽤来标识⼀条请求链路，⼀条请求链路中包含⼀个Trace ID，多个Span ID。

**第三个值**，4ef9402f9a9500a1、spanId 基本的⼯作单元，获取元数据，如发送⼀个http。

**第四个值**：true，是否要将该信息输出到zipkin服务中来收集和展示。

我们可以看出这三个微服务的TraceID是一样的，都为4ef9402f9a9500a1。代表是一个请求链路，但是4c48de8ab2b6377a是不一样的，每个请求都是自己的SpanID

`总结` 查看日志文件并不是一个很好的方法，当微服务越来越多日志文件也会越来越多，查询工作会变得越来越麻烦，下一篇我们通过 Zipkin 进行链路跟踪。Zipkin 可以将日志聚合，并进行可视化展示和全文检索。



`github地址` [nacos-feign-sentinel-gatway-sleuth](https://github.com/yudiandemingzi/spring-cloud-alibaba-study/tree/nacos-feign-sentinel-gatway-sleuth)