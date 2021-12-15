### 简介

Netflix Eureka 是一款由 Netflix 开源的基于 REST 服务的注册中心，用于提供服务发现功能。Spring Cloud Eureka 是 Spring Cloud Netflix 微服务套件的一部分，基于 Netflix Eureka 进行了二次封装，主要负责完成微服务架构中的服务治理功能。

Spring Cloud Eureka 是一个基于 REST 的服务，并提供了基于 Java 的客户端组件，能够非常方便的将服务注册到 Spring Cloud Eureka 中进行统一管理。

### 部署 Eureka Server

1. 创建一个名为 eureka-server 的 Spring Cloud 的项目（略）

2. 引入 eureka-server 依赖（maven）

   

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```

3. 开启 EurekaServer
   在启动类上添加 `@EnableEurekaServer`注解，开启 EurekaServer 的自动装配功能。

4. 修改服务端口为8761

5. 修改 register-with-eureka 配置
   添加一个`eureka.client.register-with-eureka=false`的配置，作为EurekaServer可以不将自己的实例注册到 Eureka Server 中，如果是集群部署设置为true（不配置默认值也是true）。

6. 修改 `fetch-registry` 配置
   添加一个 eureka.client.fetch-registry=false 的配置，表示不从 Eureka Server 中获取 Eureka 的注册表信息，如果是集群部署设置为true（不配置默认值也是true）。

7. 添加defaultZone配置
   添加一条配置`eureka.client.service-url.defaultZone=http://localhost:8761/eureka/`（如果不加这个的话又自定义了端口，可能会报错Connect to localhost:8761 timed out）

8. 启动 Eureka Server，访问 [http://localhost:8761/](https://jinglingwang.cn/)，如果顺利的话可以看到如下成功页面
   [![img](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141416546-30569584.png)](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141416546-30569584.png)

至此，一个简单的Eureka注册中心就完成了，后面实战中的 Eureka Client 都会注册到这个注册中心。上面的demo只是一个单机部署，接下里我们看看我们要部署多个Eureka节点时怎么做。

### **Eureka Server 集群部署**

集群部署一般有两种情况，一是伪集群部署，二是真正的集群部署。

集群部署，我们可以在多台物理机上部署，这样多个实例可以用同一个端口，不会出现伪集群端口冲突的问题，更推荐这种方式，性能更高，稳定性也更好。

伪集群部署一般说的是在同一台物理机器上部署多个节点，这时候端口就必须不一样，否则启动的时候会出现端口冲突；

**伪集群部署示例：**

假设要部署3个节点：master/slave1/slave2

1. 在application.yml配置定义三个节点的端口：

   

   ```
   port:
     master: 8761
     slave1: 8762
     slave2: 8763
   ```

2. 我们可以分别创建三个配置文件application-master.yml、application-slave1.yml、application-slave2.yml，三个配置文件除了有冲突的地方端口不一样，其他配置完全一样

   

   ```
   # application-master.yml
   server:
     port: ${port.slave1} # 服务端口
   
   # application-master.yml
   server:
     port: ${port.slave1} # 服务端口
   
   # application-slave2slave2.yml
   server:
     port: ${port.slave2} # 服务端口
   
   # 以下配置三个配置文件都一样
   eureka:
     client:
       register-with-eureka: true #不将自己的实例注册到 Eureka Server
       fetch-registry: true #不从 Eureka Server 中获取 Eureka 的注册表信息
       service-url:
         defaultZone: http://127.0.0.1:${port.master}/eureka/,http://127.0.0.1:${port.slave1}/eureka/,http://127.0.0.1:${port.slave2}/eureka/
     instance:
       hostname: eureka-server
     server:
       enable-self-preservation: true # 开启自我保护机制，默认也是开启的
   ```

3. IDEA 分别以三个不同的profiles启动
   [![img](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141459758-1401235544.png)](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141459758-1401235544.png)

4. 访问 [http://localhost:8761/](https://jinglingwang.cn/) 或者 [http://localhost:8762/](https://jinglingwang.cn/) 或者 [http://localhost:8761/](https://jinglingwang.cn/)，出现以下类似页面则代表成功
   [![img](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141513610-788851847.png)](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141513610-788851847.png)

观察上面的页面，发现 Eureka Server 节点均出现在 unavailable-replicas 下，说明集群搭建还是失败了，那这个问题怎么解决呢？

1. 在`host`添加以下配置

   

   ```
   127.0.0.1       eureka-server-master
   127.0.0.1       eureka-server-slave1
   127.0.0.1       eureka-server-slave2
   ```

2. 修改三个配置文件的`defaultZone`信息

   

   ```
   eureka:
     client:
       service-url:
         defaultZone: http://eureka-server-master:${port.master}/eureka/,http://eureka-server-slave1:${port.slave1}/eureka/,http://eureka-server-slave2:${port.slave2}/eureka/
   ```

3. 配置`eureka.instance.hostname`信息（尤其是在同一台物理机上配置三个节点时，需要修改为不同的host）

   

   ```
   eureka:
   	instance:
       hostname: eureka-server-master
   
   eureka:
   	instance:
       hostname: eureka-server-slave1
   
   **eureka:
   	instance:
       hostname: eureka-server-slave2
   ```

4. 重新启动，访问[http://localhost:8761/](https://jinglingwang.cn/) ，其他两个节点君出现在 available-replicas 选项
   [![img](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141534356-1823256569.png)](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141534356-1823256569.png)

   注意：如果执行完上面还是出现在，请检查是否配置了 prefer-ip-address = true，true #以IP地址注册到服务中心，相互注册使用IP地址，如果是在一台物理机上，IP都是一个，所以建议设置成false，或者不配置再试试。

### 部署 Eureka Client

1. 创建一个名为eureka-client 的SprintBoot的项目（略）

2. 引入eureka-client依赖（maven）

   

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

3. 引入spring-boot-starter-web依赖，如果没有加上spring-boot-starter-web，服务无法正常启动

   

   ```
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

4. 开启 EurekaClient
   在启动类上加入注解`@EnableEurekaClient`，用于启用Eureka发现配置

5. 配置端口为8081

   

   ```
   server.port = 8081
   port.master = 8761
   port.slave1 = 8762
   port.slave2 = 8763
   ```

6. 配置注册中心地址
   添加配置 `eureka.client.serviceUrl.defaultZone=http://eureka-server-master:${port.master}/eureka/,http://eureka-server-slave1:${port.slave1}/eureka/,http://eureka-server-slave2:${port.slave2}/eureka/`

7. 启动服务，刷新 [http://localhost:8761/](https://jinglingwang.cn//) 页面，如果看到了EUREKA-CLIENT应用则表示注册成功
   [![img](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141553558-1061251503.png)](https://img2020.cnblogs.com/blog/709068/202011/709068-20201124141553558-1061251503.png)

### **Eureka自我保护机制**

自我保护机制是为了避免因网络分区故障而导致服务不可用的问题。具体现象为当网络故障后，所有的服务与 Eureka Server 之间无法进行正常通信，一定时间后，Eureka Server 没有收到续约的信息，将会移除没有续约的实例。这个时候正常的服务也会被移除掉，所以需要引入自我保护机制来解决这种问题。

当服务提供者出现网络故障，无法与 Eureka Server 进行续约，Eureka Server 会将该实例移除，此时服务消费者从 Eureka Server 拉取不到对应的信息，实际上服务提供者处于可用的状态，问题就是这样产生的。

**开启自我保护机制**



```
eureka.server.enable-self-preservation=true # 开启自我保护机制，默认也是开启的
```

当服务提供者出现网络故障，无法与 Eureka Server 进行续约时，虽然 Eureka Server 开启了自我保护模式，但没有将该实例移除，服务消费者还是可以正常拉取服务提供者的信息，正常发起调用。

但是自我保护机制也有不好的地方，如果服务提供者真的下线了，由于 Eureka Server 自我保护还处于打开状态，不会移除任务信息，当服务消费者对服务提供者 B 进行调用时，就会出错。

自我保护模式有利也有弊，但我们建议在生产环境中还是开启该功能，默认配置也是开启的。

完整代码实例：

1. [Eureka Server 代码实例](https://github.com/Admol/learning_examples/tree/master/SpringCloud-Demo/eureka-server)
2. [Eureka Client 代码示例](https://github.com/Admol/learning_examples/tree/master/SpringCloud-Demo/eurela-client)

### 总结

1. 使用`@EnableEurekaServer` 注解实现注册中心
2. 使用`@EnableEurekaClient` 注册到注册中心
3. Eureka Server 集群部署的时候需要保证`register-with-eureka`和 `fetch-registry` 为true，单机部署可以为false
4. 生产环境建议开启自我保护机制