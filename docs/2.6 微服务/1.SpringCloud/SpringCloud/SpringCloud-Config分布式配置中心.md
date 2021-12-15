[TOC]

# 1、Spring Cloud Config概述
Spring Cloud Config 为微服务提供了集中化的外部配置支持，配置服务器为不同微服务应用的所有环境提供了一个中心化的外部配置。

**Spring Cloud Config 分为服务端和客户端两部分。**

- 服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器，并为客户端提供获取配置信息、加密解密信息灯访问接口。
- 客户端则是通过指定的配置中心来管理应用资源以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息，配置服务器默认使用 git 来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过 git 客户端工具来方便的管理和访问配置内容。

## 1.1 Spring Cloud Config作用
- 集中管理配置文件
- 不同环境不同配置，动态化的配置更新，分环境部署，比如dev/prod/test/beta/release
- 运行期间动态调整配置，不再需要在每个服务上编写配置文件，服务会向配置中心统一拉取自己的配置
- 当配置发生变动时，服务无需重启，可以动态的应用新配置
- 将配置信息以 REST 接口的形式暴露给微服务

SpringCloud涉及到三个角色：

- 配置中心服务端：为配置客户端提供对应的配置信息，配置信息的来源是配置仓库。应用启动时，会从配置仓库拉取配置信息缓存到本地仓库中。
- 配置中心客户端：应用启动时从配置服务端拉取配置信息。
- 配置仓库：为配置中心服务端提供配置信息存储，Spring Cloud Config 默认是使用git作为仓库的。

![](http://image.winrains.cn/2020/03/b7c0f-3586089d566730b5fc6f5df08e96e5bb0b1.jpg)

整体过程：

- 环境部署之前，将所需的配置信息推送到配置仓库
- 启动配置中心服务端，将配置仓库的配置信息拉取到服务端，配置服务端对外提供REST接口
- 启动配置客户端，客户端根据 spring.cloud.config 配置的信息去服务器拉取相应的配置

# 2、与Git整合
Spring Cloud Config 默认使用 Git 来存储配置文件（也有其他方式，比如SVN、本地文件，但最推荐的还是 Git），而且使用的是 http/https 访问的形式。
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426144813.png)

# 3、基本使用
## 3.1 服务端
1. 使用 GitHub 或其它代码库创建一个仓库 springcloud-config，添加几个文件，创建一个 dev 分支：
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426132358.png)

2. 新建一个项目当作配置中心，添加 maven 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

3. 在application.yml添加如下配置，配置自己的远程仓库地址，如果 ssh 无法连接可以尝试使用 https
```yml
server:
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          # 远程库地址
          uri: @*&%$%#$%
          # 搜索目录
          search-paths:
            - springcloud-config
      # 读取分支
      label: master
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

4. 在主启动类上开启配置服务
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args){
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```
5. 在浏览器输入如下地址可以访问到配置文件的信息
> locahost:3344/master/config-test.yml

**几种访问方式：**
```xml
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
第一种方式返回的是 json 数据（如下图所示），其它方式返回的都是文件真正的内容
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426133718.png)

## 3.2 配置规则详解
- hellxztest.yml
- hellxztest-dev.yml
- hellxztest-stable.yml
- hellxztest-prod.yml

这里的application可以自定义为其它的名称，这里可以用应用的名称，即应用名，后边的dev、stable、prod这些都可以视为一个应用下多个不同的配置文件，可以当做环境名，以下均用环境名代称。

Config支持我们使用的请求的参数规则为：

- / { 应用名 } / { 环境名 } [ / { 分支名 } ]
- / { 应用名 } - { 环境名 }.yml
- / { 应用名 } - { 环境名 }.properties
- / { 分支名 } / { 应用名 } - { 环境名 }.yml
- / { 分支名 } / { 应用名 } - { 环境名 }.properties

**注意**：

1. 第一个规则的分支名是可以省略的，默认是master分支
2. 无论你的配置文件是properties，还是yml，只要是应用名+环境名能匹配到这个配置文件，那么就能取到
3. 如果是想直接定位到没有写环境名的默认配置，那么就可以使用default去匹配没有环境名的配置文件
4. 使用第一个规则会匹配到默认配置
5. 如果直接使用应用名来匹配，会出现404错误，此时可以加上分支名匹配到默认配置文件
6. 如果配置文件的命名很由多个-分隔，此时直接使用这个文件名去匹配的话，会出现直接将内容以源配置文件内容直接返回，内容前可能会有默认配置文件的内容


# 4、客户端准备
使用 bootstrap.yml 最为配置文件

- application.yml 是用户级的资源配置项
- bootstrap.yml 是系统级的，优先级更高

Spring Cloud 会创建一个 Bootstrap Context，作为 Spring 应用的 Application Context 的父上下文。初始化的时候，Bootstrap Context 负责从外部源加载配置属性，并解析配置。这两个上下文共享一个从外部获取的 Environment。

Bootstrap 属性有高优先级，默认情况下，它们不会被本地配置覆盖，Bootstrap Context 和 Application Context 有着不同的约定，所以新加一个 bootstrap.yml 文件，保证 Bootstrap Context 和 Application Context 配置的分离

## 4.1 添加 Maven 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
## 4.2 添加配置文件 bootstrap.yml
```yml
server:
  port: 3355
spring:
  application:
    name: cloud-config-client
  cloud:
    config:
      label: master #分支名
      name: config #配置文件名
      profile: test #配置文件后缀
      uri: http://localhost:3344
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

## 4.3 编写 controller，获取配置中心中的文件属性
```java
@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/info")
    public String getConfigInfo(){
        return configInfo;
    }
}
```
## 4.4 浏览器输入地址访问
> localhost:3355/info
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200426143144.png)

如果需要获取其它配置文件内容，只需要修改 bootstrap.yml 中的 label、name、profile 即可。

# 5、Config动态刷新
对客户端进行修改
## 5.1 需要引入 actuator 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
## 5.2 添加如下配置
- 暴露监控端点
```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
在 Controller 上添加注解 @RefreshScope。

刷新服务端后，发送 Post 请求，curl -X POST http://localhost:3355/actuator/refresh
客户端刷新即可获取最新内容，避免了服务重启。