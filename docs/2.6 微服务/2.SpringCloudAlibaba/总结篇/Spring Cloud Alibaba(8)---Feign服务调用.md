[Spring Cloud Alibaba(8)---Feign服务调用](https://www.cnblogs.com/qdhxhz/p/14659744.html)

##   一、概念 

一个成熟的微服务集群，内部调用必然依赖一个好的RPC框架，比如：**基于http协议的Feign，基于私有Tcp协议的dubbo**。本文内容介绍Feign。

#### 1、什么是Feign

Feign是由Netflix开发出来的另外一种实现负载均衡的开源框架，它封装了Ribbon和RestTemplate，实现了WebService的 **面向接口编程**，进一步的减低了项目的耦合度，

因为它封装了Riboon和RestTemplate，所以它具有这两种框架的功能，可以 **实现负载均衡和Rest调用**。

#### 2、为什么需要Feign

之前已经创建好了**订单微服务**,**商品微服务**，这两个微服务是互相隔离的，那么微服务和微服务之间如何互相调用呢?

显然两个微服务都可以采用http通信，之前也通过代码来实现restTemplate进行互相访问，但是这种方式对参数传递和使用都不是很方便，我们需要配置请求head、body，

然后才能发起请求。获得响应体后，还需解析等操作，十分繁琐。采用Feign进行服务之间的调用，可以简化调用流程，真正感觉到是在同一个项目中调用另一个类的方法的欢快感。

#### 3、Feign优点

```
 1、Feign旨在使编程java Http客户端变得更容易。
 2、服务调用的时候融合了 Ribbon 技术，所以也支持负载均衡作用。
 3、服务调用的时候融合了 Hystrix 技术，所以也支持熔断机制作用。
```



## 二、项目搭建 

因为我们是通过 **订单服务**(mall-order) 调 **商品服务**(mall-goods),所以商品服务不需要做任何改动，只需在**订单服务**(mall-order)添加相关配置

#### 1、pom.xml

```xml
    <!--引入feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

#### 2、SpringBoot启动类

添加`@EnableDiscoveryClient`注解,开启Feign支持

```java
//开启Feign支持
@EnableFeignClients
@SpringBootApplication
public class OrderApplication {

    public static void main(String [] args){
        SpringApplication.run(OrderApplication.class,args);
    }
}
```

#### 3、GoodsService

这里其实就是通过接口的方式，填写商品服务接口信息

```java
/**
 * mall-good s就是商品微服务的 spring.application.name
 */
@FeignClient(value = "mall-goods")
public interface GoodsService {
   
    /**
     * /api/v1/goods/findByGoodsId就是商品服务提供的接口，参数也是
     */
    @GetMapping("/api/v1/goods/findByGoodsId")
    Goods findById(@RequestParam("goodsId") int goodsId);
}
```

简单来讲这里就三步

```
1、接口上添加@FeignClient 注解
2、value指是你要调的服务应用名称
3、下面提供的接口和参数和你要调的服务提供的接口一致就可以了
```

#### 4、Controller接口

```java
@RestController
@RequestMapping("api/v1/goods_order")
public class OrderController {

    @Autowired
    private  GoodsService goodsService;

    /**
     * 通过Feign请求mall-goods服务
     */
    @RequestMapping("getGoodsByFeign")
    public Object getGoodsByFeign(int goodsId) {
        Goods goods = goodsService.findById(goodsId);
        return goods;
    }

}
```

**这样是不是也可以明显看出，订单服务调商品服务通过Feign就像调内部接口一样，非常方便。**

#### 5、测试

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414202543112-411491232.jpg)

很明显，请求成功。



##  三、负载均衡示例 

前面说了,Feign融合了 Ribbon 技术，所以也支持负载均衡作用。

#### 1、Ribbon⽀持的负载均衡策略

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414202554498-384351902.jpg)

#### 2、默认负载均衡策略(轮询)示例

我们先不配置任何负载均衡策略，看默认是采用什么策略。这里

```
mall-order(订单服务)单机部署端口号:7001
mall-goods(商品服务)集群部署端口号：6001、6002、6003。
mall-goods服务
```

**1)、新增Controller接口**

因为mall-goods是集群部署的，所以想知道到底是那台服务那个端口获取请求

```java
@RestController
@RequestMapping("api/v1/goods")
public class GoodsController {

    @Autowired
    private GoodsService goodsService;

    @RequestMapping("findClusterName")
    public Object findClusterName( HttpServletRequest request) {
        //获取是哪个节点被请求
        String clusterName = "当前服务器名称: " + request.getServerName()+";当前集群节点端口号: "+ request.getServerPort();
        return clusterName;
    }

}
mall-order服务
```

**1)、新增Feign接口**

```java
@FeignClient(value = "mall-goods")
public interface GoodsService {

    /**
     * /api/v1/goods/findByGoodsId就是商品服务提供的接口，参数也是
     */
    @GetMapping("/api/v1/goods/findClusterName")
    String findClusterName();
}
```

**2）新增Controller接口**

```java
@RestController
@RequestMapping("api/v1/goods_order")
public class OrderController {

    @Autowired
    private  GoodsService goodsService;

    /**
     * 通过Feign请求mall-goods服务
     */
    @RequestMapping("getClusterName")
    public Object findClusterName() {
        for (int i = 0; i <10 ; i++) {
            System.out.println(goodsService.findClusterName());
        }
        return "执行结束";
    }

}
```

**这里在for循请求商品服务接口10次**,  看下控制台打印结果

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414202607502-2068138788.jpg)

因为我这边是同一台服务器集群部署多个商品微服务,所以打印出的服务器名称都是一样的。从端口号可以看出 **订单服务是轮询去请求商品微服务集群的**。

#### 3、修改负载均衡策略(随机)示例

我们这里改成 **随机策略**，我们只需添加相关配置

```java
 @Bean
    public IRule loadBalancer(){
        return new RandomRule();
    }
```

再重新请求接口

```
http://localhost:7001/api/v1/goods_order/getClusterName
```

**查看结果**

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414202617098-890514241.jpg)

可以从结果中看出,订单服务已经变成是随机请求商品微服务集群的。

```
策略选择
1、如果每个机器配置⼀样，则建议不修改策略 (也就是轮询策略) 
2、如果部分机器配置强，则可以改为WeightedResponseTimeRule（响应时间加权重策略）
```

`总结` 这篇博客也是比较简单了来学习了Feign作为服务调用,下一篇博客将会讲下 Sentinel。