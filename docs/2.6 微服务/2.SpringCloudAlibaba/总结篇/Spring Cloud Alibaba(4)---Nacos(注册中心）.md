[Spring Cloud Alibaba(4)---Nacos(注册中心）](https://www.cnblogs.com/qdhxhz/p/14650407.html)

```
前言
```

有关Nacos客户端的搭建和Nacos的介绍在 **Spring Cloud Alibaba(2)---Nacos概述** 有讲到，所以这里不在陈述。因为是要实现注册中心，所以一定是要有多个微服务，

上一篇博客 **RestTemplate微服务项目**   已经搭建好一个脚手架，这篇是在它的基础上添加Spring Cloud Alibaba框架和Nacos组件。

##  一、概述

#### 1、没有服务注册中心

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210412211050961-310457056.jpg)

正常逻辑如果没有注册服务发现中心的话,订单服务如何去调取商品服务呢？

我们可以将服务调用 **域名** 和 **端口号** 写死到代码或配置文件中,然后通过HTTP请求,这样做是可以,但有很多不足。

```
1、人工维护慢慢会出现瓶颈和问题：新增服务或服务扩容，所有依赖需要新增修改配置；
2、某台服务器挂了还要手动摘流量；服务上下线变更时效慢；
3、人工配置的话容易出现错误或漏配
```

这时你会想如果能让服务自动化完成配置（注册）和查找（发现）就好了，于是乎服务注册发现就应运而生。

#### 2、Nacos注册中心理论

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414122400628-1572160010.jpg)

这里把这个图片做个简单解释

```
1. 订单服务和商品服务启动的时候, 会注册到服务发现中心(Nacos), 告诉它,我的ip和端口号是什么。
2. 订单服务要调商品服务的时候，会先去注册列表获取所以商品服务的注册信息(ip+端口号）。
3. 拿到了商品服务的ip和port, 接下来就可以调用商品服务了.
```

我们可以看出有了注册服务中心， **订单服务** 需要知道 **商品服务** 的地址和端口号不需要通过我们去配置而是可以直接去注册中心拿到。



##  二、项目搭建

#### 1、启动Nacos客户端

有关Nacos客户端的搭建之前有说过，这里不在陈述

```
sh startup.sh -m standalone
```

#### 2、父工程(mall-parent)

对于父工程，只要添加SpringCloudAlibaba相关jar包就可以了

```
pom.xml
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR8</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

#### 3、商品微服务(mall-goods)-消费方

这里需要修改三个地方:**pom.xml**、**SpringBoot启动类**、**bootstrap.yaml配置类**

**1).pom.xml**

添加Nacos客户端jar包

```xml
  <!--添加nacos客户端-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
```

**2).bootstrap.yaml配置类**

注册到指定Nacos客户端

```yaml
spring:
  cloud:
    nacos:
    discovery:
    server-addr: 127.0.0.1:8848
```

**3).SpringBoot启动类**

添加`@EnableDiscoveryClient`注解

```java
@EnableDiscoveryClient
@SpringBootApplication
@MapperScan("com.jincou.goods.dao")
public class GoodsApplication {

    public static void main(String [] args){
        SpringApplication.run(GoodsApplication.class,args);
    }

}
```

#### 4、订单微服务(mall-order)-提供方

这里同样需要修改三个地方:**pom.xml**、**SpringBoot启动类**、**bootstrap.yaml配置类**，代码和上面一样这里就不在贴出。



##  三、测试

```
商品微服务采用集群方式启动，端口号分别为:6001、6002
订单微服务未采用集群方式启动，订单服务端口号:7001
```

上面都启动成功后，我们来访问Nacos客户端。

```
http://127.0.0.1:8848/nacos
```

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210412211119104-1099945129.jpg)

从图中可以看出 在服务列表上

```
 mall-goods 健康实例数有两个。因为商品服务商品是启动了2个。
 mall-order 健康实例数只有一个。
```

我们可以在看下，商品服务列表中的详情，点进去之后，可以看出商品服务集群的端口

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210412211130697-1781059243.jpg)

`总结` 这篇博客也是比较简单了来学习了Nacos作为服务注册中心，下一篇博客开始写Nacos作为配置中心

`GitHub地址`:[spring-cloud-alibaba-study](https://github.com/yudiandemingzi/spring-cloud-alibaba-study/tree/nacos)