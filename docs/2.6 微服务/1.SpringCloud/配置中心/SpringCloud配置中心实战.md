[TOC]

部分优质内容来源：
[简书：CD826：SpringCloud文集](https://www.jianshu.com/nb/14932141)

# 1、统一配置中心(Config)
Spring Cloud Config具有中心化、版本控制、支持动态更新和语言独立等特性。其特点是:

- 提供服务端和客户端支持(Spring Cloud Config Server和Spring Cloud Config Client);
- 集中式管理分布式环境下的应用配置;
- 基于Spring环境，实现了与Spring应用无缝集成;
- 可用于任何语言开发的程序;
- 默认实现基于Git仓库(也支持SVN)，从而可以进行配置的版本管理;

Spring Cloud Config的结构图如下:

![](https://upload-images.jianshu.io/upload_images/1488771-197762860d5c607f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1064/format/webp)

从图中可以看出Spring Cloud Config有两个角色(类似Eureka): Server和Client。Spring Cloud Config Server作为配置中心的服务端承担如下作用:

1. 拉取配置时更新Git仓库副本，保证是配置为最新;
2. 支持从yml、json、properties等文件加载配置;
3. 配合Eureke可实现服务发现，配合Cloud Bus(这个后面我们在详细说明)可实现配置推送更新;
4. 默认配置存储基于Git仓库(可以切换为SVN)，从而支持配置的版本管理.
而对于，Spring Cloud Config Client则非常方便，只需要在启动配置文件中增加使用Config Server上哪个配置文件即可。

## 1.1 Spring项目配置加载顺序

1. 这里是列表文本命令行参数
2. SPRING_APPLICATION_JSON 参数
3. 从java:comp/env 加载 JNDI 属性
4. Java系统属性 （System.getProperties()）
5. 操作系统环境变量
6. 如果有使用 random.* 属性配置，则使用 RandomValuePropertySource 产生
7. 外部特定应用配置文件 例如：application-{profile}.properties 或者 YAML variants
8. 内部特定应用配置文件 例如：application-{profile}.properties 或者 YAML variants
9. 外部应用配置文件 例如：application.properties 或者 YAML variants
10. 内部应用配置文件 例如：application.properties 或者 YAML variants
11. 加载@Configuration类的 @PropertySource 或者 @ConfigurationProperties 指向的配置文件
12. 默认配置，通过SpringApplication.setDefaultProperties 设置

## 1.2 配置规则详解
Config Client从Config Server中获取配置数据的流程:

1. Config Client启动时，根据bootstrap.properties中配置的应用名称(application)、环境名(profile)和分支名(label)，向Config Server请求获取配置数据;
2. Config Server根据Config Client的请求及配置，从Git仓库(这里以Git为例)中查找符合的配置文件;
3. Config Server将匹配到的Git仓库拉取到本地，并建立本地缓存;
4. Config Server创建Spring的ApplicationContext实例，并根据拉取的配置文件，填充配置信息，然后将该配置信息返回给Config Client;
5. Config Client获取到Config Server返回的配置数据后，将这些配置数据加载到自己的上下文中。同时，因为这些配置数据的优先级高于本地Jar包中的配置，因此将不再加载本地的配置。

Config Server又是如何与Git仓库中的配置文件进行匹配的呢？通常，我们会为一个项目建立类似如下的配置文件:

- mallweb.properties: 基础配置文件;
- mallweb-dev.properties: 开发使用的配置文件;
- mallweb-test.properties: 测试使用的配置文件;
- mallweb-prod.properties: 生产环境使用的配置文件;

当我们访问Config Server的端点时，就会按照如下映射关系来匹配相应的配置文件：

1. /{application}/{profile}[/{label}]
2. /{application}-{profile}.yml
3. /{label}/{application}-{profile}.yml
4. /{application}-{profile}.properties
5. /{label}/{application}-{profile}.properties
上面的Url将会映射为格式为:{application}-{profile}.properties(yml)的配置文件。另外，label则对应Git上分支名称，是一个可选参数，如果没有则为默认的master分支。

而Config-Client的bootstrap.properties配置对应如下:

- spring.application.name <==> application;
- spring.cloud.config.profile <==> profile;
- spring.cloud.config.label <==> label.

## 1.3 Git仓库配置
Config Server默认使用的就是Git，所以配置也非常简单，如上面的配置(application.properties):
```yml
spring.cloud.config.server.git.uri=http://
spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
```
那么客户端在请求时服务端就会到该仓库中进行查找。

### 1.3.1 使用占位符
在服务端配置中我们也可以使用{application}、{profile} 和 {label}占位符，如下:
```yml
spring.cloud.config.server.git.uri=http://github.com/cd826/{application}
spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
```
这样，我们就可以为每一个应用客户端创建一个单独的仓库。

> 这里需要注意的是，如果Git的分支或标签中包含"/"时，在{label}参数中需要使用"(_)"替代，这个主要是避免与Http URL转义符处理的冲突。

### 1.3.2 模式匹配
也可以使用{application}/{profile}进行模式匹配，以便获取到相应的配置文件。配置示例如下:
```yml
spring.cloud.config.server.git.uri=https://github.com/spring-cloud-samples/config-repo

spring.cloud.config.server.git.repos.simple=https://github.com/simple/config-repo

spring.cloud.config.server.git.repos.special.pattern=special*/dev*,*special*/dev*
spring.cloud.config.server.git.repos.special.uri=https://github.com/special/config-repo

spring.cloud.config.server.git.repos.local.pattern=local*
spring.cloud.config.server.git.repos.local.uri=file:/home/configsvc/config-repo
```
如果模式中需要配置多个值，那么可以使用逗号分隔。

如果{application}/{profile}没有匹配到任何资源，则使用spring.cloud.config.server.git.uri配置的默认URI。

当我们使用yml类型的文件进行配置时，如果模式属性是一个YAML数组，也可以使用YAML数组格式来定义。这样可以设置成多个配个配置文件，如:

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - */development
                - */staging
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - */qa
                - */production
              uri: https://github.com/staging/config-repo
```

### 1.3.3 搜索目录
当我们把配置文件存放在Git仓库中子目录中时，可以通过设置serch-path来指定该目录。同样，serch-path也支持上面的占位符。示例如下:
```yml
spring.cloud.config.server.git.uri=https://github.com/spring-cloud-samples/config-repo
spring.cloud.config.server.git.searchPaths=foo,bar*
```

这样系统就会自动搜索foo的子目录，以及以bar开头的文件夹中的子目录。

# 2、配置中心服务端
## 2.1 pom.xml
```xml
<parent>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-parent</artifactId>
   <version>Dalston.SR5</version>
   <relativePath/>
</parent>

·······
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

踩过的坑：

1. 第一次添加依赖时，还加入了springboot starter web的依赖，导致项目无法启动。后来发现，注入spring-cloud-starter-parent的父类依赖，然后引入spring-cloud-config-server依赖后，就可以导入对应的包。
2. 需要找到对应的spring-cloud的父版本依赖库。

## 2.2 启动类Application
```java
/**
 * @author aishiyushijiepingxing
 * @description：添加@EnableConfigServer注解
 * @date 2020/6/28
 */
@EnableConfigServer
@SpringBootApplication
public class ConfigCenterApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterApplication.class, args);
    }
}
```

## 2.3 配置application.yml
```yml
server:
  port: 7900
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          # Git仓库地址
          uri: xxxx
          # Git仓库账号
          username: xxxx
          # Git仓库密码
          password: xxxx
          #search-paths: xxxx
      # 读取分支
      label: master
```



# 3、配置中心客户端
config-client可以是任何一个基于Spring boot的应用。

## 3.1 pom.xml
添加cloud客户端依赖以及cloud-context上下文环境依赖。
```xml
<dependency>
      <!--Spring Cloud 上下文依赖，为了使bootstrap配置文件生效-->
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-context</artifactId>
      <version>2.1.0.RC2</version>
</dependency>
       <!--Spring Cloud Config 客户端依赖-->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-config</artifactId>
     <version>2.1.0.RC2</version>
</dependency>
```

## 3.2 bootstrap.yml 多环境
```yml
server:
  port: 8081
management:
  security:
    enabled: false     #SpringBoot 1.5.X 以上默认开通了安全认证，如果不关闭会要求权限
spring:
  application:
    name: config_client
---
spring:
  profiles: dev
  cloud:
    config:
      uri: xxx        #Config server的uri
      profile: dev                       #指定的环境
      label: master                       #指定分支
---
spring:
  profiles: test
  cloud:
    config:
      uri: xxx        #Config server的uri
      profile: test                       #指定的环境
      label: master                       #指定分支
---
spring:
  profiles: prod
  cloud:
    config:
      uri: xxx        #Config server的uri
      profile: prod                       #指定的环境
      label: master                       #指定分支
```

# 4、动态刷新配置
Config-Client中提供了一个refresh端点来实现配置文件的刷新。要想使用该功能，我们需要在Config-Client的pom.xml文件中增加以下依赖:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
这样，当修改配置文件并提交到Git仓库后，就可以使用:http://localhost:8080/refresh刷新本地的配置数据。

> 最好的方式还是和Spring Cloud Bus进行整合，这样才能实现配置的自动分发，而不是需要手工去刷新配置。
