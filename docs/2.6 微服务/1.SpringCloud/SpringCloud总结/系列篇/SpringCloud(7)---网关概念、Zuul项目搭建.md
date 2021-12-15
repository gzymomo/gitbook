[SpringCloud(7)---网关概念、Zuul项目搭建](https://www.cnblogs.com/qdhxhz/p/9594521.html)

**项目代码GitHub地址**：https://github.com/yudiandemingzi/spring-cloud-study

## 一、网关概念

####   1、什么是路由网关

网关是系统的唯一对外的入口，介于客户端和服务器端之间的中间层，处理非业务功能 提供路由请求、鉴权、监控、缓存、限流等功能。它将"1对N"问题转换成了"1对1”问题。

通过服务路由的功能，可以在对外提供服务时，只暴露 网关中配置的调用地址，而调用方就不需要了解后端具体的微服务主机。

####  2、为什么要使用微服务网关

  不同的微服务一般会有不同的网络地址，而客户端可能需要调用多个服务接口才能完成一个业务需求，若让客户端直接与各个微服务通信，会有以下问题：

（1）客户端会多次请求不同微服务，增加了客户端复杂性

（2）存在跨域请求，处理相对复杂

（3）认证复杂，每个服务都需要独立认证

（4）难以重构，多个服务可能将会合并成一个或拆分成多个

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905210245220-512410227.png)

####   3、网关的优点

  微服务网关介于服务端与客户端的中间层，所有外部服务请求都会先经过微服务网关客户只能跟微服务网关进行交互，无需调用特定微服务接口，使得开发得到简化

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905210823038-708485043.png)

总的理解网关优点

服务网关 = 路由转发 + 过滤器

（1）路由转发：接收一切外界请求，转发到后端的微服务上去。

（2）过滤器：在服务网关中可以完成一系列的横切功能，例如权限校验、限流以及监控等，这些都可以通过过滤器完成（其实路由转发也是通过过滤器实现的）。

####   4、服务网关技术选型

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905211133067-1494346492.png)

引入服务网关后的微服务架构如上，总体包含三部分：服务网关、open-service和service。

**（1）总体流程**

​    服务网关、open-service和service启动时注册到注册中心上去；

用户请求时直接请求网关，网关做智能路由转发（包括服务发现，负载均衡）到open-service，这其中包含权限校验、监控、限流等操作

open-service聚合内部service响应，返回给网关，网关再返回给用户

**（2）引入网关的注意点**

  增加了网关，多了一层转发（原本用户请求直接访问open-service即可），性能会下降一些（但是下降不大，通常，网关机器性能会很好，而且网关与open-service的访问通常

是内网访问，速度很快）；

**（3）服务网关基本功能**

智能路由：接收外部一切请求，并转发到后端的对外服务open-service上去；

   注意：我们只转发外部请求，服务之间的请求不走网关，这就表示全链路追踪、内部服务API监控、内部服务之间调用的容错、智能路由不能在网关完成；

​       当然，也可以将所有的服务调用都走网关，那么几乎所有的功能都可以集成到网关中，但是这样的话，网关的压力会很大，不堪重负。

权限校验：可在微服务网关上进行认证，然后在将请求转发给微服务，无须每个微服务都进行认证，不校验服务内部的请求。服务内部的请求有必要校验吗？

 API监控：只监控经过网关的请求，以及网关本身的一些性能指标（例如，gc等）；

   限流：与监控配合，进行限流操作；

API日志统一收集：类似于一个aspect切面，记录接口的进入和出去时的相关日志。

 

## 二、Zuul项目搭建

 ![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905212511443-1154200461.png)

   使用到的组件包括：Eureka、Feign、Zuul，包括以下四个项目：

  （1）Eureka-server：  7001  注册中心

  （2）product-server ：8001  商品微服务

  （3）order-server ：  9001  订单微服务

  （4）zuul-gateway ：  6001  Zuul网关

注册中心、商品微服务、order在之前博客都已搭建，这里就不重复写。这里只写zuul-gateway微服务。

####    1、pom.xml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
        <!--客户端jar包，这个在订单微服务，商品微服务都要添加-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!--zuuljar包-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   2、application.yml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
server:
  port: 6001

#服务的名称
spring:
  application:
    name: zuul-gateway

#指定注册中心地址
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7001/eureka/

#自定义路由映射
zuul:
  routes:
    order-service: /apigateway/order/**
    product-service: /apigateway/product/**
  #统一入口为上面的配置，其他入口忽略
  ignored-patterns: /*-service/**
  #忽略整个服务，对外提供接口
  ignored-services: order-service
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    3、SpringBoot启动类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SpringBootApplication
//加上网关注解
@EnableZuulProxy
public class ZuulServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulServerApplication.class, args);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   4、测试

**（1）直接订单服务调商品服务**

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905214914776-109308983.png)

 **(2)通过Zuul网关实现订单接口调商品服务**

 ![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905215908270-736215453.png)

####    5、注意些细节

   （1）url不能重复，否则会覆盖

```
    order-service:    /apigateway/order/**
    product-service:  /apigateway/product/**
```

  （2）通过zuul后，request中的cookie值获取不到，那是因为网关给过滤掉了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    @RequestMapping("list")
    public void list(@RequestParam("user_id")int userId, @RequestParam("product_id") int productId, HttpServletRequest request){
        String token = request.getHeader("token");
        String cookie = request.getHeader("cookie");
       //会发现token值能够获取，cookie无法获取，原因是因为网关会过滤掉敏感词
        System.out.println("token="+token);
        System.out.println("cookie="+cookie);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

  想要不过滤掉cookie值那么在配置里配置

```
zuul:
  #处理http请求头为空的问题
  sensitive-headers:
```

####    6、问题

 自己遇到两个问题记录下，后期再来思考解决。

1、现在通过订单服务地址可以直接访问订单微服务，如何配置成订单微服务不能直接服务，只能通过网关访问。

 思考，是不是以后订单微服务配置到内网就不会有这个问题了。

2、当我的订单服务调商品服务异常时，直接访问订单微服务熔断降级能够完成，通过网关竟然直接报异常了。

我在商品微服务相关接口添加：

```
//睡眠两秒，微服务默认一秒就超时，所以会到降级方法
TimeUnit.SECONDS.sleep(2);
```

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905220643366-675214280.png)

直接调订单服务，降级信息返回正常。如果通过网关访问。

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180905220801706-1974286813.png)

返回的是异常，这不是很蛋疼吗，总是有解决办法让降级信息返回来的，以后解决来再来写。

 

### 参考

[Spring Cloud：服务网关 Zuul](https://www.jianshu.com/p/29e9c91e3f3e)