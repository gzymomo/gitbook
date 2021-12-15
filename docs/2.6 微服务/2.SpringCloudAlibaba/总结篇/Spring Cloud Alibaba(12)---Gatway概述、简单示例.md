[Spring Cloud Alibaba(12)---Gatway概述、简单示例](https://www.cnblogs.com/qdhxhz/p/14766527.html)

## 一、Gatway概念

#### 1.Gatway是什么？

Gatway是在**Spring生态系统之上**构建的API网关服务，基于Spring 5，Spring Boot2和Project Reactor等技术。Gateway旨在提供一种简单而有效的方式来对API进行路由，

以及提供一些强大的过滤器功能，例如：**反向代理**、**熔断**、**限流**、**重试**等。

微服务架构中网关所处位置(盗图)：

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513222636745-843684540.jpg)

#### 2、Gatway有哪些特性？

1)、为了提升网关的性能，Spring Cloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。
 2)、基于SpringFramework5，ProjectReactor和SpringBoot2.0进行构建
 3)、能够匹配任何任何请求属性
 4)、可以对路由指定Predicates和Filters,易于编写的Predicates和Filters
 5)、集成断路器
 6)、集成Spring Cloud服务发现
 7)、支持请求限流
 8)、支持路径重写

#### 3、Gatway三大重要概念

Gatway有着很重要的3个概念 `路由`、`断言`、`过滤`。

`路由`：**路由是构建网关的基本模块，它由ID，目标URI，一系列的断言Predicates和过滤器Filters组成，如果断言为true，则匹配该路由。**

`断言`：**参考Java8的java.util.function.Predicate，开发人员可以匹配HTTP请求中的所有内容，例如请求头或请求参数，如果请求与断言相匹配则进行路由。**

之前有写过Predicate的文章 [java代码之美（13）--- Predicate详解](https://www.cnblogs.com/qdhxhz/p/11323595.html)

`过滤`：**Spring框架中GatewayFilter的实例，使用过滤器，可以载请求被路由前或者后对请求进行修改。**

**总结下就是**: 一个web请求，通过一些匹配条件，定位到真正的服务节点，在这个转发的过程中，进行一些精细化的控制。predicate就是匹配条件,filter就是拦截器。

predicate + filter + 目标uri实现路由route，所以一般都是组合使用，`一个请求可以经过多个predicate或者多个filter`。

#### 4、Gatway工作流程

下面是官网给的一张图,它总体上描述了Spring Cloud Gateway的工作流程

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513222851270-281837570.jpg)

1）客户端向Spring Cloud Gateway发出请求。

2）如果Gateway Handler Mapping确定请求与路由匹配，则将其发送到Gateway Web Handler。

3）Handler通过指定的过滤器链将请求发送到我们实际的服务执行业务逻辑，然后返回。

4）过滤器由虚线分隔的原因是，过滤器可以在发送代理请求之前或之后执行逻辑。

`核心`：**路由转发 + 过滤器链**



## 二、Gatway与Zuul区别 

Zuul有两个大版本分别为: Zuul1和Zuul2。所以要比较的话有个跟这两个版本进行比较。

#### 1、Gatway与Zuul1比较

1）Zuul1.x是一个基于阻塞I/O的API 网关

2）Zuul1.x基于Servlet2.5使用`阻塞架构`，它不支持任何长连接（如WebSocket）Zuul的设计模式和Nginx比较相似，每次I/O操作都是从工作线程中选择一个执行，请求线程

被阻塞到工作线程完成，但是差别是Nginx用C++实现，Zuul用Java实现。

3） Spring Cloud Gateway建立在Spring Framewor5、Project Reactor和Spring Boot2之上，`使用非阻塞API`。

#### 2、Gatway与Zuul2比较

1）首先Zuul2从产品角度来看 因为Zuul2的升级一直跳票，一下开源一下闭源。所以SpringCloud目前还没有整合Zuul2。这就带来很大的不方便。也是因此SpringCloud最后自己

研发了一个网关代替Zuul，就是SpringCloud Gateway。它才是Spring的亲儿子。

2）跟Zuul1相比Zuul2.x理念更加先进，想基于Netty非阻塞和支持长连接。Zuul2.x的性能较Zuul1.x有很大提升。在性能方面，根据官网提供的基准测试，

SpringCloud Gateway 的RPS（每秒请求数）是Zuul的1.6倍。



##  三、编码实现 

这里先附上一张图，到目前为止，**Nacos注册中心**、**Nacos配置中心**、**Feign服务调用**、**Sentinel流量控制** 在之前的博客都有写,项目中都有实现，这里也在之前

项目的基础上开始实现Gateway功能。

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513223052848-280211947.jpg)

#### 1、pom.xml

```xml
     <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
     </dependency>
```

#### 2、application.yml

```yaml
server:
  port: 8001

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes: #数组形式
        - id: mall-goods  #路由唯一标识
          uri: http://127.0.0.1:6001  #想要转发到的地址
          order: 1 #优先级，数字越小优先级越高
          predicates: #断言 配置哪个路径才转发
            - Path=/mall-goods/**
            #- Before=2020-09-11T01:01:01.000+08:00  # 在这个时间点之后不能访问
            #- Query=source  #一定携带这个参数
          filters: #过滤器，请求在传递过程中通过过滤器修改
            - StripPrefix=1  #去掉第一层前缀
```

#### 3、启动类

```java
@SpringBootApplication
public class GatWayApplication {

    public static void main(String [] args){
        SpringApplication.run(GatWayApplication.class,args);
    }

}
```

`注意`  pom文件中不能有 **spring-boot-starter-web** jar包。否则启动会直接报错。

#### 4、测试

1）**直接访问mall-goods地址**

```
http://localhost:6001/api/v1/sentinel/test-sentinel
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513223135512-613453154.jpg)

2）**通过网关服务上面这个接口**

```
http://localhost:8001/mall-goods/api/v1/sentinel/test-sentinel
```

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513223147210-15412357.jpg)

发现通过服务网关地址后成功转发到商品微服务的接口。



##   四、配置详解

上面有讲过Gatway有着很重的3个概念 **路由**、**断言**、**过滤**。

```yaml
id：路由的ID
uri：匹配路由的转发地址
predicates：配置该路由的断言，通过PredicateDefinition类进行接收配置。
order：路由的优先级，数字越小，优先级越高。
filters: 过滤器
```

一个routes下可以有多个上面的配置，上面一个配置可以理解成一个微服务节点。如果有多个微服务需要转发那么就在routes下多配置几个，示例

![img](https://img2020.cnblogs.com/blog/1090617/202105/1090617-20210513223158027-1389509313.jpg)

```
id
```

路由的ID。可以任意字符串。但如果有几十个微服务都随意取名,到时候不好找。这里用微服务的名称当成路由的ID，这样一看就知道对于那个微服务做路由转发。

```
uri
```

匹配路由的转发地址。这里好理解就是将请求网关的地址，转到具体微服务节点的地址。

```
predicates
```

配置该路由的断言。主要请求满足该断言(true)才会进入该路由。这个可以同时配置多个。多个的情况下代表需要 **同时满足**。

```
order
```

路由的优先级，数字越小，优先级越高。可能我们一个请求经来，同时满足匹配多个路由。那这里order谁小就进那个路由。

```
filters
```

过滤器。跟predicates一样。同时可以配置多个。多个的意思就是一个一个过滤。

上面这样解释或许有点枯燥，所以这里根据上面的实际例子来做解释。

```yaml
#为什么通过配置后请求第一个url会转发到第二个url
http://localhost:8001/mall-goods/api/v1/sentinel/test-sentinel
http://localhost:6001/api/v1/sentinel/test-sentinel
```

步骤如下

```
1）首先是断言predicates 上面只配置了一个  - Path=/mall-goods/**。上面的请求中是有 /mall-goods/ 所以命中当前路由。
2）那么根据 uri: http://127.0.0.1:6001 属性，那么请求的IP+端口号 由 localhost:8001 转为 127.0.0.1:6001。
3）同时上面配置了一个过滤器   - StripPrefix=1  去掉第一层前缀，也就是去掉 /mall-goods/ 这一层。所以最终的路由转发就变成了http://127.0.0.1:6001/api/v1/sentinel/test-sentinel。 也就是商品微服务对应的请求接口。
```

`总结`:有关 **predicates** 和 **filters** 官方都提供了很多种规则实现类，我们要根据特定的需求去使用他们，如果不满足那就需要我们自定义predicates 或 filters。

`github地址`  [nacos-feign-sentinel-gatway](https://github.com/yudiandemingzi/spring-cloud-alibaba-study/tree/nacos-feign-sentinel-gatway)



###   参考

[Gateway--概述](https://blog.csdn.net/cold___play/article/details/104951061)

[如何评价 spring cloud gateway? 对比 zuul2.0 主要的优势是什么?](https://www.zhihu.com/question/280850489)