[TOC]

[博客园：东北小狐狸：Config 配置中心与客户端的使用与详细](https://www.cnblogs.com/hellxz/p/9306507.html)

# 1、URI指定配置中心
Spring Clout Config的客户端在启动的时候，默认会从工程的classpath下加载配置信息并启动应用，只有配置了spring.cloud.config.uri的时候，客户端才会尝试连接Config Server拉取配置信息并初始化Spring环境配置。我们必须将uri这个属性参数配置到bootstrap.yml(或.properties)中，这个配置文件的优先级大于application.yml和其它文件，这样才能保证能正确加载远程配置。启动的时候客户端会去连接uri属性的值。

举例：
```yml
spring:
  application:
    name: hellxztest
  cloud:
    config:
      uri: http://localhost:7001/
      profile: dev
```
例子中使用application.name和profile定位配置信息。

# 2、服务化配置中心
服务端详解那一块说过可以把配置中心注册到注册中心，通过服务发现来访问Config Server拉取Git仓库中的配置信息。

下面的内容需要改造ConfigServer和ConfigClient

为ConfigServer和ConfigClient的pom.xml中分别加入Eureka的依赖
```xml
        <!-- Spring Cloud Eureka的依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
```
在上边说过，bootstrap.yml的优化级大于application.yml，这里我们需在将这两个服务都注册到注册中心，所以我选择在ConfigServer的resources下创建一个application.yml，在其中配置连接注册中心的通用信息

applicaiton.yml
```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:1111/eureka/
```
将这个applicaiton.yml 的内容复制到ConfigClient的resources下的bootstrap.yml中。

依赖有了，注册中心的地址也有了，别忘了将这两个模块的主类加上@EnableDiscoveryClient。

完成这一步我们可以先启动注册中心、ConfigServer、ConfigClient 看一看是否注册成功，如图，
![](https://images2018.cnblogs.com/blog/1149398/201807/1149398-20180713180304549-749891655.png)

ConfigServer至此配置完毕。接下来我们为客户端使用服务发现的方式去调用ConfigServer

只需要在ConfigClient的bootstrap.yml中注掉spring.cloud.config.uri那一项，spring.cloud.discovery.enabled=true和spring.cloud.discovery.serviceId=config-server，为了更直观，下面把整个ConfigClient的bootstrap.yml粘出来

```yml
spring:
  application:
    name: hellxztest                     #指定了配置文件的应用名
  cloud:
    config:
#      uri: http://localhost:7001/    #Config server的uri
      profile: dev                          #指定的环境
      label: master                        #指定分支
      discovery:
        enabled: true                     #开启配置服务发现
        serviceId: config-server        #配置中心服务名
server:
  port: 7002
management:
  security:
    enabled: false     #SpringBoot 1.5.X 以上默认开通了安全认证，如果不关闭会要求权限
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:1111/eureka/
```
重启ConfigClient项目，我们之前提供了一个接口/api/from，这个接口会返回git上hellxztest-dev.yml中的from参数的值，现在我们只要调通这个接口就说明我们使用服务发现功能连接配置中心成功。

![](https://images2018.cnblogs.com/blog/1149398/201807/1149398-20180713180330900-344101495.png)
测试成功，服务化配置中心配置完成。

# 3、失败快速响应与重试
Spring Cloud Config的客户端会预先加载很多配置信息，然后才开始连接Config Server进行属性的注入，当应用复杂的时候，连接ConfigServer时间过长会直接影响项目的启动速度，我们还希望能知道Config Client 是否能从Config Server中取到信息。此时，只需在bootstrap.yml（.properties）中配置属性spring.cloud.config.failfast=true即可。

关闭Config Server，重启Config Client，项目直接报错无法启动

java.lang.IllegalStateException: Could not locate PropertySource and the fail fast property is set, failing
	…… 省略其他错误信息 ……
直接失败似乎代价有些高，所以Config客户端还提供了自动重试的功能，开启前请确认spring.cloud.config.failfast=true参数已经配置，然后为ConfigClient的pom.xml增加spring-retry和spring-boot-starter-aop的依赖，如下：
```xml
        <!-- 连接配置中心重试的依赖 -->
        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```
只需加入如上两个依赖就能实现自动重试，当第6次重试失败之后，那么就会上边那个快速失败报的错误

![](https://images2018.cnblogs.com/blog/1149398/201807/1149398-20180713180352310-1929804609.png)

如果对最大重试次数和重试间隔等设置不满意，可以通过下面有参数进行调整。
```yml
spring:
  cloud:
    fail-fast: true                 #快速失败
    retry:
      initial-interval: 1100        #首次重试间隔时间，默认1000毫秒
      multiplier: 1.1D              #下一次重试间隔时间的乘数，比如开始1000，下一次就是1000*1.1=1100
      max-interval: 2000            #最大重试时间，默认2000
      max-attempts: 3               #最大重试次数，默认6次
```