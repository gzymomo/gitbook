[TOC]

- [Spring Cloud Gateway的功能及综合使用](https://www.cnblogs.com/xiaowan7/p/14197270.html)

# 1、GateWay简介

Spring Cloud 全家桶中有个很重要的组件：网关。在 1.x 版本中使用的是 Zuul 网关，但是到了 2.x，由于Zuul的升级不断跳票，Spring Cloud 自己研发了一套网关组件：Spring Cloud Gateway。

Spring Cloud Gateway基于 Spring Boot 2.x，Spring WebFlux 和 Project Reactor 构建，使用了 Webflux 中的 reactor-netty 响应式编程组件，底层使用了 Netty 通讯框架。

## 1.1 GateWay作用
- 反向代理
- 鉴权
- 流量控制
- 熔断
- 日志监控

## 1.2 网关在微服务架构中的位置
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425105729.png)

## 1.3 GateWay的三大概念
- Route（路由）：路由是构建网关的基本模块，它由 ID、目标 URI、一系列的断言和过滤器组成，如果断言为 true 则匹配该路由
- Predicate（断言）：参考的是 Java8 中的 java.util.function.Predicate。开发人员可以匹配 HTTP 请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由
- Filter（过滤）：指的是 Spring 框架中 GatewayFilter 的实例，使用过滤器，可以在请求被路由之前或之后对请求进行修改
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425115208.png)

## 1.4 工作流程
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425115400.png)

客户端向 Spring Cloud Gateway 发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关 Web 处理程序。该处理程序通过特定于请求的过滤器链来运行请求。 筛选器由虚线分隔的原因是，筛选器可以在发送代理请求之前和之后运行逻辑。所有 “前置“ 过滤器逻辑均被执行，然后发出代理请求，发出代理请求后，将运行“ 后置 ”过滤器逻辑。
**总结：路由转发 + 执行过滤器链。**

# 2、两种配置方式
## 2.1 配置文件方式
以访问「百度新闻网」为例，添加如下配置：
```yml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway9527
  cloud:
    gateway:
      routes:
        - id: news						# 路由id
          uri: http://news.baidu.com	# 真实调用地址
          predicates:
            - Path=/guonei				# 断言，符合规则进行路由
```
浏览器虽然输入 localhost:9527/guonei，却会转发到指定的地址
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425145640.png)

## 2.2 编码方式
新增配置文件
```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("news2", r -> r.path("/guoji").uri("http://news.baidu.com"))
                .build();
    }
}
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425151001.png)

# 3、动态路由
开启后，默认情况下 Gateway 会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。
```yml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh1
          #uri: http://localhost:8001     #静态，写死了地址，只能调用一个服务
          uri: lb://CLOUD-PAYMENT-SERVICE #动态，lb://微服务名
          predicates:
            - Path=/payment/get/**
        - id: payment_routh2
          #uri: http://localhost:8001
          uri: lb://CLOUD-PAYMENT-SERVICE
          predicates:
            - Path=/payment/lb/**
```

# 4、Predicate的使用
## 4.1 时间相关配置
- After：在指定时间之后进行路由
- Before：在指定时间之前进行路由
- Between：在指定时间之间进行路由
```yml
predicates:
    - Path=/payment/lb/**
    #- After=2020-04-25T16:30:58.215+08:00[Asia/Shanghai]
    #- Before=2020-04-25T16:40:58.215+08:00[Asia/Shanghai]
    - Between=2020-04-25T16:35:58.215+08:00[Asia/Shanghai],2020-04-25T16:40:58.215+08:00[Asia/Shanghai]
```
上述配置的时间格式可以通过以下代码得到。
```java
@Test
public void test(){
    ZonedDateTime now = ZonedDateTime.now();
    System.out.println(now);
}
```
## 4.2 请求相关配置
**Cookie**
配置说明：【Cookie=cookie名, cookie值的正则表达式规则】
```yml
predicates:
  - Path=/payment/lb/**
  - Cookie=id, [0-9]
```
使用 curl 工具模拟携带 cookie 发送请求:
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425165551.png)

**Header**
配置说明：【Header=header名, header值的正则表达式规则】
```yml
predicates:
  - Path=/payment/lb/**
  - Header=h, [a-h]
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425170924.png)

**Method**
配置说明：【Method=请求类型】
```yml
predicates:
  - Path=/payment/lb/**
  - Method=GET
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425171953.png)

**Path**
配置说明：【Path=请求路径】
```yml
predicates:
  - Path=/payment/lb/**
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425171953.png)

**Query**
配置说明：【Query=参数名，参数值】
```yml
predicates:
  - Path=/payment/lb/**
  - Query=name, zhangsan
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425172720.png)

# 5、Filter的使用
生命周期：pre、post
种类：GatewayFilter、GlobalFilter。

**自定义全局过滤器：**
```java
@Component
@Slf4j
public class MyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String username = exchange.getRequest().getQueryParams().getFirst("username");
        //用户名为空时，给出错误响应
        if (username == null) {
            log.info("用户名为空，非法登录");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425175644.png)

![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200425175544.png)