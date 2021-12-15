- [Spring Cloud整合Gateway实现API网关](https://www.cnblogs.com/haoxianrui/p/13608650.html)



## **一. 前言**

微服务实战系列是基于开源微服务项目 [有来商城youlai-mall](https://github.com/hxrui/youlai-mall) 版本升级为背景来开展的,本篇则是讲述API网关使用Gateway替代Zuul，有兴趣的朋友可以进去给个star,非常感谢。

## **二. 什么是微服务网关？**

微服务网关是位于服务之前或者应用程序之前的一个层面，用于保护、增强和控制微服务的访问。

其常见的作用有：

1. 鉴权校验：验证是否认证和授权
2. 统一入口：提供所有微服务的入口点，起到隔离作用，保障服务的安全性
3. 限流熔断
4. 路由转发
5. 负载均衡
6. 链路追踪

## **三. 网关如何选型？**

至于为什么使用Gateway而放弃Zuul？

SpringCloud 生态提供了两种API网关产品，分别是Netflix开源的Zuu1和Spring自己开发的SpringCloud  Gateway,SpringCloud以Finchely版本为分界线，之前版本使用Zuul作为API网关，之后更推荐使用Gateway。

Netflix已经在2018年开源了Zuul2，但是SpringCloud已经推出了Gateway,并且在github标识没有集成Zuul2的计划。

[SpringCloud Gateway和Zuul对比及技术选型？](https://www.zhihu.com/question/280850489)

## **四. 项目信息**

[有来商城youlai-mall](https://github.com/hxrui/youlai-mall) 完整项目结构图

[![img](https://i.loli.net/2020/09/03/kSWG1JCBrxucaUe.png)](https://i.loli.net/2020/09/03/kSWG1JCBrxucaUe.png)

本篇文章涉及项目模块

| 工程名         | 端口 | 描述               |
| -------------- | ---- | ------------------ |
| nacos-server   | 8848 | 注册中心和配置中心 |
| youlai-gateway | 9999 | API网关            |
| youlai-admin   | 8080 | 管理平台           |

版本声明



```
Nacos Server: 1.3.2

SpringBoot: 2.3.0.RELEASE

SpringCloud: Hoxton.SR5

SpringCloud Alibaba: 2.2.1.RELEASE 
```

## **五. 项目实战**

**1.添加SpringCloud Gateway依赖**



```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

**2.bootstrap.yml配置信息**

[![img](https://i.loli.net/2020/08/30/R6WE8DSk4sguUIh.png)](https://i.loli.net/2020/08/30/R6WE8DSk4sguUIh.png)



```
server:
  port: 9999

spring:
  application:
    name: youlai-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 启用自动根据服务ID生成路由
          lower-case-service-id: true # 设置路由的路径为小写的服务ID
      routes:
        - id: youlai-auth
          uri: lb://youlai-auth
          predicates:
            - Path=/youlai-auth/**
          filters:
            - StripPrefix=1 # 移除前缀 youlai-auth
        - id: youlai-admin
          uri: lb://youlai-admin
          predicates:
            - Path=/youlai-admin/**
          filters:
            - StripPrefix=1
```

**3.微服务接口**

youlai-admin添加一个接口方法用来测试网关转发能力
 [![img](https://i.loli.net/2020/09/03/Q6AhpcumiNUOxGK.png)](https://i.loli.net/2020/09/03/Q6AhpcumiNUOxGK.png)

**4.网关测试**

依次启动项目nacos-server,youlai-admin,youlai-gateway

[![img](https://i.loli.net/2020/09/03/NprkJCjH5sTxIzR.png)](https://i.loli.net/2020/09/03/NprkJCjH5sTxIzR.png)

可以看到当我们请求网关的服务路径http://localhost:9999/youlai-admin/users的时候，路由根据匹配规则
 [![img](https://i.loli.net/2020/09/03/A7GK4EBu8zWtPjw.png)](https://i.loli.net/2020/09/03/A7GK4EBu8zWtPjw.png)
 将以/youlai-admin为前缀的请求路径转发到服务youlai-admin实例上去了。

## **六. 结语**

至此SpringCloud整合Gateaway就成功了，当然这里只是验证了API网关的路由转发功能。后面会写一篇关于SpringCloud Gateaway整合Oauth2实现网关鉴权功能。