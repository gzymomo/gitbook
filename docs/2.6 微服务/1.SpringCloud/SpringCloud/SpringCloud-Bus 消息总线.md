[TOC]

# 1、概述
## 1.1 基本介绍
Spring Cloud Bus 目前支持两种消息代理：RabbitMQ、Kafka
Spring Cloud Config 配合 Spring Cloud Bus 使用可以实现配置的动态刷新
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426154933.png)

Spring Cloud Bus 用来将分布式系统的结点与轻量级系统链接起来的框架，它整合了 Java 的事件处理机制和消息中间件的功能。

## 1.2 什么是总线？
在微服务架构的系统中，通常会使用轻量级的消息代理来构建一个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在该主题上的实例都知道的消息。

## 1.3 基本原理
ConfigClient 实例都监听 MQ 中同一个 topic（默认叫 springCloudBus）。当一个服务刷新数据时，会把这个信息放到 Topic 中，这样其它监听同一 Topic 的服务就能得到通知，然后去更新自身的配置。

# 2、设计思想
bus 动态刷新全局广播有两种设计思想
1. 利用消息总线触发一个客户端的 /bus/refresh，进而刷新所有客户端的配置
2. 利用消息总线触发一个服务端 ConfigServer 的 /bus/refresh，进而刷新所有客户端的配置

我们采用第二种，第一种方式不适合的原因有三：
1. 打破了微服务的职责单一性。负责业务模块的微服务不应该承担配置刷新的职责
2. 破坏了微服务各节点的对等性
3. 有一定的局限性。例如微服务迁移时，它的网络地址常常发生变化，如果想要做到自动刷新，还需要增加更多的配置
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426171334.png)

# 3、动态刷新全局广播配置
1. 在配置中心微服务以及所有需要接收消息的客户端中导入 Maven 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```
2. 添加对应配置，使其支持消息总线
添加在配置中心、所有需要接收消息的客户端中
rabbitmq相关配置，15672是web管理界面的端口，5672是mq访问的接口
```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```
添加在配置中心微服务中
rabbitmq相关配置，暴露bus刷新配置的端点
```yml
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```
3. 测试，修改 github 中的配置，使用 curl 发送请求 curl -X POST http://localhost:3344/actuator/bus-refresh 后，刷新每个微服务的页面发现都已被修改。实现了一次修改，广播通知，处处生效！

# 4、动态刷新定点广播配置
如果不想全部通知，只想定点通知，上述的配置都不用变，只需要发送请求的时候在后面指定「微服名:端口」
> curl -X POST http://localhost:3344/actuator/bus-refresh/{destination}

以我的为例就是：
> curl -X POST http://localhost:3344/actuator/bus-refresh/cloud-config-client:3355

# 5、流程图
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426181515.png)

1. 配置中心微服务通过远程库获取配置信息，同时订阅 RabbitMQ 主题
2. 客户端通过配置中心获取配置信息，同时订阅 RabbitMQ 主题
3. 当我们修改远程库的配置后
4. 发送 POST 请求
5. 配置中心向 RabbitMQ 发送刷新事件
6. 客户端监听到刷新事件
7. 从配置中心拉取新的配置