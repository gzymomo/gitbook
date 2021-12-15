[TOC]

# 1、Eureka两大组件
🚩 Eureka Server：提供服务注册服务
- 各个微服务节点通过配置启动后，会在 Eureka Server 中进行注册， 这样 Eureka Server 中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

🚩 Eureka Client：通过服务注册中心访问
- 是一个Java客户端，用于简化Eureka Server的交互,客户端同时也具备一个内置的、 使用轮询（round-robin）负载 算法的负载均衡器在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒）。如果 Eureka Server 在多个心跳周期内没有接收到某个节点的心跳，Eureka Server 将会从服务注册表中把这个服务节点移除（默认90秒）。

# 2、Eureka Server搭建
## 2.1 在注册中心服务 导入maven依赖
```xml
<!--eureka server-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

## 2.2 编写配置文件
```yml
server:
  port: 7001
eureka:
  instance:
    # eureka服务端的实例名称
    hostname: localhost
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    # 设置与 Eureka Server 交互的地址查询服务和注册服务都需要依赖此地址
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

## 2.3 在主启动类开启Eureka服务注册功能
```java
@SpringBootApplication
@EnableEurekaServer //开启服务注册功能
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class, args);
    }
}
```
## 2.4 查看Eureka配置中心
> http://localhost:7001/

![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419141823.png)

# 3、Eureka Client注册
## 3.1 在服务提供者 添加客户端的Maven依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
## 3.2 添加配置
```yml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:7001/eureka # 注册中心的地址
```
## 3.3 在主启动类开启 Eureka 服务端
```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class);
    }
}
```
## 3.4 进入注册面板即可发现服务已注册进来
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419144502.png)

# 4、Eureka高可用
搭建 Eureka 注册中心集群，实现负载均衡 + 故障容错。
Eureka 集群原理说明：
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419154354.png)

## 4.1 搭建Eureka集群
1. 为了模拟集群，我们需要修改 host 文件
```xml
SpringCloud Eureka 集群配置
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
```

2. 新建一个项目当作 Eureka 服务注册中心，配置文件如下：
```yml
server:
  port: 7002
eureka:
  instance:
    # eureka服务端的实例名称
    hostname: eureka7002.com
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    # 设置与 Eureka Server 交互的地址查询服务和注册服务都需要依赖此地址
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

3. 修改配置，完成两个注册中心之间的相互注册
```yml
server:
  port: 7001
eureka:
  instance:
    # eureka服务端的实例名称
    hostname: eureka7001.com
  client:
    # false表示不向注册中心注册自己
    register-with-eureka: false
    # false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    # 设置与 Eureka Server 交互的地址查询服务和注册服务都需要依赖此地址
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
```

4. 访问：http://eureka7001.com:7001/ 和 http://eureka7002.com:7002/
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419164948.png)

5. 将之前的defaultZone换为如下配置，重新启动后可看见两个注册中心中都有了其他服务
` defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka `

6. 新建一个服务提供者，可以直接拷贝前面的代码，启动服务、刷新注册中心，现在就有了两个服务提供者:
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419192448.png)

7. 此时使用服务消费者不断调用提供者，是以 轮询 的负载均衡方式调用
8. 通过如下配置「服务名称」和「访问信息提示IP地址」
```yml
eureka:
  instance:
    instance-id: payment8001
    prefer-ip-address: true   #访问路径可以显示ip地址
```
前提是引入了 actuator
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

# 5、服务发现Discovery
1. 注入DiscoveryClient
```java
@Autowired
private DiscoveryClient discoveryClient;
```

2. 获取服务信息
```java
@GetMapping("/payment/discovery")
public Object discovery() {
    List<String> services = discoveryClient.getServices();
    for (String service : services) {
        log.info("service:" + service);
    }

    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    for (ServiceInstance instance : instances) {
        log.info("serviceId:" + instance.getServiceId() + "\t" + "Host:" + instance.getHost() + "\t" + "port:" + instance.getPort() + "\t" + "uri:" + instance.getUri());
    }

    return this.discoveryClient;
}
```

3. 在主启动类上添加@EnableDiscoveryClient，开启服务发现

# 6、Eureka自我保护
在 Eureka 注册中心的页面会看到这样的提示，说明 Eureka 进入了保护模式：
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419210836.png)

保护模式主要用于一组客户端和 Eureka Server 之间存在网络分区场景下的保护。一旦进入保护模式，**Eureka Server 将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。**

简单来说就是：某时刻某个微服务不可用了，Eureka 不会立即清理，依然会对该微服务的信息进行保存；属于 CAP 理论中的 AP 分支

🎨 **为什么会产生Eureka自我保护机制？**
- 为了防止 EurekaClient 可以正常运行，但是与 EurekaServer 网络不通情况下，EurekaServer 不会立刻 将EurekaClient 服务剔除

🎨 **什么是自我保护模式？**
- 默认情况下，如果 EurekaServer 在一定时间内没有接收到某个微服务实例的心跳，EurekaServer 将会注销该实例(默认90秒)。但是当网络分区故障发生（延时、卡顿、 拥挤）时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了 —— 因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过「自我保护模式」来解决这个问题 —— 当 EurekaServer 节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。

**在自我保护模式中，EurekaServer 会保护服务注册表中的信息，不再注销任何服务实例。**

它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着I

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

🎨 **如何禁用自我保护模式**

在注册中心服务配置中加入以下内容
```yml
eureka:
  server:
    # 关闭自我保护机制,保证不可用服务及时被剔除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```
刷新注册中心页面可以看到，提示内容已经变了，自我保护模式已关闭
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200419215228.png)

在其它服务配置中加入以下内容
```yml
eureka:
  instance:
    # Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    # Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```
正常启动服务后，服务出现在注册中心列表，当我们关闭服务再查看列表，可以看到服务在 2s 内直接被剔除了.