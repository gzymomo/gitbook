[TOC]

# 1、什么是微服务

## 1.1、架构演进

架构的发展历程是从单体式架构，到分布式架构，到SOA架构，再到微服务架构。

图1：架构演进

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810104800618.png?#pic_center)



- 单体架构：未做任何拆分的Java Web程序

图2：单体架构示意图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810095010767.png?#pic_center)

- 分布式架构:按照业务垂直划分，每个业务都是单体架构，通过API互相调用。

图3：分布式架构示意图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810100445816.png?#pic_center)



- SOA架构：SOA是一种面向服务的架构。其应用程序的不同组件通过网络上的通信协议向其它组件提供服务或消费服务，所以也是分布式架构的一种。

图4：SOA架构示意图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810101312890.png?#pic_center)



## 1.2、微服务架构

微服务架构在某种程度上是SOA架构的进一步的发展。

微服务目前并没有比较官方的定义。微服务 Microservices 之父，马丁.福勒，对微服务大概的概述如下：

> 就目前而言，对于微服务业界并没有一个统一的、标准的定义（While there is no precise definition of this architectural style ) 。
> 但通常在其而言，微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。
> 服务之间采用轻量级的通信机制互相沟通（通常是基于 HTTP 的 RESTful API ) 。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。
> 另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务。可以使用不同的语言来编写服务，也可以使用不同的数据存储。

图5：微服务定义思维导图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810102436348.png?#pic_center)

图6：微服务架构示意图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810103042537.png?#pic_center)





## 1.3、微服务解决方案

目前最流行的两种微服务解决方案是SpringCloud和Dubbo。



# 2、SpringCloud概览

## 2.1、什么是SpringCloud

Spring Cloud 作为 Java 言的微服务框架，它依赖于 Spring Boot ，有快速开发、持续交付和容易部署等特点。 Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区 Spring、Netflix Pivotal 两大公司的推动下越来越完善。

SpringCloud是一系列组件的有机集合。

图7：SpringCloud技术体系

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810105241576.png?#pic_center)

图8：SpringCloud技术体系思维导图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810112743519.png?#pic_center)



## 2.1、SpringCloud主要组件

### 2.1.1、Eureka

Netflix Eureka 是由 Netflix 开源的一款基于 REST 的服务发现组件，包括 Eureka Server 及 Eureka Client。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810120101878.png?#pic_center)



### 2.1.2、Ribbon

Ribbon Netflix 公司开源的一个负载均衡的组件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810174308572.png?#pic_center)



### 2.1.3、Feign

Feign是是一个声明式的Web Service客户端。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810182853299.png?0#pic_center)



### 2.1.4、Hystrix

Hystrix是Netstflix 公司开源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810205126528.png?#pic_center)



### 2.1.5、Zuul

Zuul 是由 Netflix 孵化的一个致力于“网关 “解决方案的开源组件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810211514805.png?#pic_center)



### 2.1.6、Gateway

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0、 Spring Boot 2.0 和 Project Reactor 等
技术开发的网关， Spring Cloud Gateway 旨在为微服务架构提供简单、 有效且统一的 API 路由
管理方式。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810212808923.png?#pic_center)



### 2.1.7、Config

Spring Cloud 中提供了分布式配置中 Spring Cloud Config ，为外部配置提供了客户端和服务器端的支持。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810214123442.png?#pic_center)



### 2.1.8、 Bus

使用 Spring Cloud Bus, 可以非常容易地搭建起消息总线。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810223142844.png?0#pic_center)



### 2.1.9、OAuth2

Sprin Cloud 构建的微服务系统中可以使用 Spring Cloud OAuth2 来保护微服务系统。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811205313656.png?#pic_center)



### 2.1.10、Sleuth

Spring Cloud Sleuth是Spring Cloud 个组件，它的主要功能是在分布式系统中提供服务链路追踪的解决方案。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811212137615.png?#pic_center)

# 3、总结

本文中对架构的演进及Spring Cloud 构建微服务的基本组件进行了概览。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200811213243542.png?#pic_center)