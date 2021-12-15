# Spring Cloud整合Nacos实现注册中心 

- [Spring Cloud整合Nacos实现注册中心](https://www.cnblogs.com/haoxianrui/p/13584204.html)



## **前言**

随着eureka的停止更新，如果同时实现注册中心和配置中心需要SpringCloud Eureka和SpringCloud  Config两个组件;配置修改刷新时需要SpringCloud  Bus消息总线发出消息通知(Kafka、RabbitMQ等)到各个服务完成配置动态更新，否者只有重启各个微服务实例,但是nacos可以同时实现注册和配置中心，以及配置的动态更新。

## **版本声明**

Nacos Server: 1.3.2

SpringBoot: 2.3.0.RELEASE

SpringCloud: Hoxton.SR5

SpringCloud Alibaba: 2.2.1.RELEASE

## **项目实战**

### **1.项目背景**

本篇项目实战是基于开源 [有来商城youlai-mall](https://github.com/hxrui/youlai-mall) 使用的微服务基础脚手架 [youlai](https://github.com/hxrui/youlai) 这个项目，原先使用的是eureka，现在在此基础上创建个nacos分支，并在这个分支上完成eureka到nacos的升级，话不多说，一张图说明。

[![img](https://i.loli.net/2020/08/29/DuBJe9521ZdkTpt.png)](https://i.loli.net/2020/08/29/DuBJe9521ZdkTpt.png)

下文就认证中心youlai-auth模块升级为nacos举例说明

### **2.工程整合nacos**

父工程添加spring-cloud-alibaba依赖



```xml
<properties>
    <spring-cloud-alibaba.version>2.2.0.RELEASE</spring-cloud-alibaba.version>
</properties>

<dependencyManagement>
    <dependencies>
        ...
        <!--Spring Cloud Alibaba 相关依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring-cloud-alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        ...
    </dependencies>
</dependencyManagement>
```

子模块youlai-auth添加nacos依赖



```xml
<!-- nacos 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

应用启动类入口类添加注解开启服务的注册和发现



```java
@EnableDiscoveryClient
@SpringBootApplication
public class AuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class);
    }
}
```

服务注册中心配置



```yaml
spring:
  application:
    name: youlai-auth
  cloud:
    nacos:
      discovery:
        server-addr: http://localhost:8848
```

打开nacos控制台进入服务列表查看

[![img](https://i.loli.net/2020/08/30/CTgk6BqEOwD1Hdx.png)](https://i.loli.net/2020/08/30/CTgk6BqEOwD1Hdx.png)

## 