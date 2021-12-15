- [ SpringCloud （一）-- 从单体架构到微服务架构、代码拆分（maven 聚合）](https://www.cnblogs.com/l-y-h/p/14105682.html)
- [SpringCloud （二）-- 服务注册中心 Eureka、Zookeeper、Consul、Nacos ](https://www.cnblogs.com/l-y-h/p/14193443.html)
- [SpringCloud （三）-- 服务调用、负载均衡 Ribbon、OpenFeign](https://www.cnblogs.com/l-y-h/p/14238203.html)
- [SpringCloud （四）-- 服务降级、熔断 Hystrix、Sentinel](https://www.cnblogs.com/l-y-h/p/14364167.html)
- [SpringCloud （五）-- 配置中心 Config、消息总线 Bus、链路追踪 Sleuth、配置中心 Nacos](https://www.cnblogs.com/l-y-h/p/14447473.html)
- [SpringCloud （六）--  注册中心与配置中心 Nacos、网关 Gateway](https://www.cnblogs.com/l-y-h/p/14604209.html)



# 一、引入 服务降级、熔断

## 1.1 问题 与 解决

```
【问题：】
    通过前面几篇博客介绍，完成了基本项目创建、服务注册中心、服务调用 以及 负载均衡（也即 各个模块 已经能正常通信、共同对外提供服务了）。
    
    对于一个复杂的分布式系统来说，可能存在数十个模块，且模块之间可能会相互调用（嵌套），
    这就带来了一个问题：
        如果某个核心模块突然宕机（或者不能提供服务了），那么所有调用该 核心模块服务 的模块 将会出现问题，
        类似于 病毒感染，一个模块出现问题，将逐步感染其他模块出现问题，最终导致系统崩溃（也即服务雪崩）。

【服务雪崩：】
    服务雪崩 指的是 服务提供者 不可用（不能提供服务） 而导致 服务消费者不可用，并逐级放大的过程。
    比如：
        多个微服务之间形成链式调用，A、B 调用 C，C 调用 D，D 调用其他服务等。。。
        如果 D 因某种原因（宕机、网络延迟等） 不能对外提供服务了，将导致 C 访问出现问题，而 C 出现问题，将可能导致 A、B 出现问题，也即 问题逐级放大（最终可能引起系统崩溃）。

【解决：】
    服务降级、服务熔断 是解决 服务雪崩的 常用手段。
相关技术：
    Hystrix（维护状态，不推荐使用）
    Sentienl（推荐使用）
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202195810777-1058522192.png)

 

## 1.2 服务降级 与 服务熔断

### （1） 服务降级

```
【服务降级：】
    服务降级 指的是 当服务器压力 剧增 时，根据当前 业务、流量 情况 对一些服务（一般为非核心业务）进行有策略的降级，确保核心业务正常执行。
    即 释放非核心服务 占用的服务器资源 确保 核心任务正常执行。
注：
    可以理解为 损失一部分业务能力，保证系统整体正常运行，从而防止 服务雪崩。
    资源是有限的，请求并发高时，若不对服务进行降级处理，系统可能花费大量资源进行非核心业务处理，导致 核心业务 效率降低，进而影响整体服务性能。
    此处的降级可以理解为 不提供服务 或者 延时提供服务（服务执行暂时不正常，给一个默认的返回结果，等一段时间后，正常提供服务）。
    
【服务降级分类：】
手动降级：
    可以通过修改配置中心配置，并根据事先定义好的逻辑，执行降级逻辑。

自动降级：
    超时降级：设置超时时间、超时重试次数，请求超时则服务降级，并使用异步机制检测 进行 服务恢复。
    失败次数降级：当请求失败达到一定次数则服务降级，同样使用异步机制检测 进行服务恢复。
    故障降级：服务宕机了则服务降级。
    限流降级：请求访问量过大则服务降级。
```

### （2）服务熔断

```
【服务熔断：】
    服务熔断 指的是 目标服务不可用 或者 请求响应超时时，为了保证整体服务可用，
    不再调用目标服务，而是直接返回默认处理（释放系统资源），通过某种算法检测到目标服务可用后，则恢复其调用。
注：
    在一定时间内，服务调用失败次数达到一定比例，则认为 当前服务不可用。
    服务熔断 可以理解为 特殊的 服务降级（即 服务不可用 --> 服务降级 --> 服务调用恢复）。 

【martinfowler 相关博客地址：】
    https://martinfowler.com/bliki/CircuitBreaker.html
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200025180-1318552235.png)

### （3）服务降级 和 服务熔断 的区别

```
【相同点：】
    目标相同：均从 可靠性、可用性 触发，避免系统崩溃（服务雪崩）。
    效果相同：均属于 某功能暂不可用。
    
【不同点：】
    服务降级 一般 是从整体考虑，可以手动关闭 非核心业务，确保 核心业务正常执行。
    服务熔断 一般 是某个服务不可用，自动关闭 服务调用，并在一定时间内 重新尝试 恢复该服务调用。
注（个人理解（仅供参考））：
    服务降级 可以作为 预防措施（手动降级），即 服务并没有出错，但是为了提升系统效率，我主动放弃 一部分非核心业务，保证系统资源足够用于 执行 核心业务。
    服务熔断 就是 服务出错的 解决方案（自动降级），即 服务出错后 的一系列处理。
```

 

# 二、服务降级、熔断 -- Hystrix

## 2.1 什么是 Hystrix ？

```
【Hystrix：】
    Hystrix 是一个用于处理分布式系统 延迟 和 容错的 开源库，
    目的是 隔离远程系统、服务和第三方库的访问点，停止级联故障，并在不可避免发生故障的复杂分布式系统中实现恢复能力。
注：
    分布式系统难免出现 阻塞、超时、异常 等问题，Hystrix 可以保证在一个服务出问题时，不影响整个系统使用（避免服务雪崩），提高系统的可用性。
    虽然 Hystrix 已进入维护模式，不再更新，但还是可以学习一下思想、基本使用。

【常用特性：】
    服务降级
    服务熔断
    服务监控

【相关地址：】
    https://github.com/Netflix/Hystrix
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200203323-195598193.png)

 



## 2.2 使用 JMeter 模拟超时故障发生

### （1）什么是 JMeter ？

```
【JMeter】
    Apache 的一款基于 Java 的压力测试工具。
注：
    有兴趣的自行研究，此处不过多赘述。
    
【官网下载地址：】
    http://jmeter.apache.org/download_jmeter.cgi
    
【JMeter 简单使用：】
    https://www.cnblogs.com/stulzq/p/8971531.html
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200237930-1867959064.png)

### （2）说明

```
【说明：】
    此处仅简单演示，不需要启动集群（单机版 Eureka 即可）。
    eureka_server_7000 作为 服务注册中心。
    eureka_client_producer_8001 作为服务提供者。
    eureka_client_consumer_9001 作为服务提供者。 
注：
    单机版 Eureka 可参考：https://www.cnblogs.com/l-y-h/p/14193443.html#_label2_1
    此处使用 RestTemplate 发送请求，使用上一篇 讲到的 OpenFeign 技术亦可。

【演示说明：】
    在 eureka_client_producer_8001 新定义一个接口 testTimeout()，内部暂停 2 秒模拟业务处理所需时间。
    一般情况下，访问 eureka_client_producer_8001 提供的 getUser() 接口时，会立即响应。
    
    但是如果大量请求访问 testTimeout()，而将系统资源（线程）耗尽时，
    此时若有请求访问 getUser() 就需要等待 前面请求执行完成后，才能继续处理。
    而此时就可能造成 超时等待 的情况，从而引起一系列问题。

即：   
    并发度低时：
        先访问 /consumer/user/testTimeout，再访问 /consumer/user/get/{id} 可以瞬间返回结果。

    并发度高时：
        若有大量请求访问 /consumer/user/testTimeout，导致系统资源（线程）暂时耗尽，
        此时再访问  /consumer/user/get/{id} 就需要等待一些时间才能返回结果。
        严重时请求会出现超时故障，从而引起系统异常。
```

### （3）定义接口

　　在 eureka_client_producer_8001  中定义一个新接口 testTimeout()。
　　在 eureka_client_consumer_9001 中定义一个新接口 调用 testTimeout()。

```java
【eureka_client_producer_8001：】
@GetMapping("/testTimeout")
public Result testTimeout() {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return Result.ok();
}

【eureka_client_consumer_9001：】
@GetMapping("/testTimeout")
public Result testTimeout() {
    return restTemplate.getForObject(PRODUCER_URL + "/producer/user/testTimeout", Result.class);
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200402647-1686369347.png)



 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200417461-1070456679.png)

### （4）启动服务，并使用 JMeter 测试

并发度低时：
 　　先访问 /consumer/user/testTimeout，再访问 /consumer/user/get/{id} 可以瞬间返回结果。
注：
　　看页面的刷新按钮。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200449404-1857894584.gif)

 

并发度高时：
　　使用 JMeter 模拟 200 个线程，循环 100 次，访问 /consumer/user/testTimeout。
　　此时再访问 /consumer/user/get/{id} 时，不能瞬间返回结果（等待一段时间）。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200522980-453567661.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200537771-1929932183.gif)

### （5）超时故障

　　前面已经演示了高并发情况下可能出现超时等待情况，而若 业务执行时间过长 或者 服务调用设置了超时时间，那么当访问被阻塞时，将有可能引起故障。

```java
【在声明 RestTemplate 时，定义超时时间】
@Bean
@LoadBalanced // 使用 @LoadBalanced 注解赋予 RestTemplate 负载均衡的能力
public RestTemplate getRestTemplate() {
    SimpleClientHttpRequestFactory httpRequestFactory = new SimpleClientHttpRequestFactory();
    httpRequestFactory.setConnectTimeout(2000);
    httpRequestFactory.setReadTimeout(2000);
    return new RestTemplate(httpRequestFactory);
}
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200707768-768295360.png)

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202200718545-1827079270.png)

 

 

## 2.3 Hystrix 实现服务降级

### （1）服务降级使用场景

　　服务降级 目的是 防止 服务雪崩，本质也就是在 服务调用 出问题时，应该如何处理。

```
【服务降级使用场景：】
    服务器资源耗尽，请求响应慢，导致请求超时。
    服务器宕机 或者 程序执行出错，导致请求出错。
即：
    服务提供者 响应请求超时了，服务消费者 不能一直等待，需要 服务提供者进行 服务降级，保证 请求在一定的时间内被处理。
    服务提供者 宕机了，服务消费者 不能一直等待，需要 服务消费者进行 服务降级，保证 请求在一定的时间内被处理。
    服务提供者正常，但 服务消费者 出现问题了，需要服务消费者 自行 服务降级。
    
注：
    服务降级一般在 服务消费者 中处理，服务提供者 也可以 进行处理。
```

### （2）在服务提供者 上实现服务降级（超时自动降级）

　　在 eureka_client_producer_8001 代码基础上进行补充。
Step1：
　　引入 hystrix 依赖。

```xml
【引入依赖：】
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201041174-977946896.png)

 

Step2：
　　通过 @HystrixCommand 注解 编写 服务降级策略。

```java
【简单说明：】
    @HystrixCommand 表示指定 服务降级 或者 服务熔断的策略。
    fallbackMethod 表示服务调用失败(请求超时 或者 程序执行异常)后执行的方法（方法参数要与 原方法一致）。
    commandProperties 表示配置参数。
    @HystrixProperty 设置具体参数。
注：
    详细参数情况可以参考 HystrixCommandProperties 类。
    com.netflix.hystrix.HystrixCommandProperties 


【定义服务降级策略:】
public Result testTimeoutReserveCase() {
    return Result.ok().message("当前服务器繁忙，请稍后再试！！！");
}

// 定义服务降级策略
@HystrixCommand(
        // 当请求超时 或者 接口异常时，会调用 fallbackMethod 声明的方法（方法参数要一致）
        fallbackMethod = "testTimeoutReserveCase",
        commandProperties = {
                @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds", value="1500")
        }
)
@GetMapping("/testTimeout")
public Result testTimeout() {
    try {
        Thread.sleep(500);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return Result.ok();
}
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201134432-815777463.png)

 

Step3：
　　在启动类上添加 @EnableCircuitBreaker 注解，开启服务降级、熔断。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201155684-1670558926.png)

 

Step4：
　　运行测试（此处演示的是 超时自动降级）。
　　此处定义接口超时时间为 1.5 秒，模拟 0.5 秒业务处理时间，使用 JMeter 压测该接口时，与上面演示的类似，会出现请求超时的情况，而一旦请求超时，则会触发 fallbackMethod 方法，直接返回数据，而不会持续等待。
如下图所示。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201228998-2000574078.gif)

### （3）配置默认服务降级方法

　　通过上面简单演示可以完成 服务降级，但是存在一个问题，如果为每一个接口都绑定一个 fallbackMethod，那么代码将非常冗余。
　　通过 @DefaultProperties 注解 定义一个默认的 defaultFallback 方法，接口异常时调用默认的方法，并仅对特殊的接口进行单独处理，从而减少代码冗余。

如下，新增一个 运行时异常，访问接口时，将会调用 globalFallBackMethod() 方法。
而前面特殊定义的 testTimeout 超时后，仍调用 testTimeout_reserve_case() 方法。

```java
@DefaultProperties(defaultFallback = "globalFallBackMethod")
public class UserController {
    public Result globalFallBackMethod() {
        return Result.ok().message("系统异常，请稍后再试！！！");
    }

    @GetMapping("/testRuntimeError")
    @HystrixCommand
    public Result testRuntimeError() {
        int temp = 10 / 0;
        return Result.ok();
    }
}
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201332023-882206286.png)

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201351993-1293548902.gif)

 

## 2.4 OpenFeign 实现服务降级

### （1）说明

```
【说明：】
    上面使用 Hystrix 简单演示了 服务提供者 的服务降级。
    这里使用 OpenFeign 演示 服务消费者 的服务降级。
注：
    重新新建一个模块 eureka_openfeign_client_consumer_9007 作为服务消费者用于演示。
    可参考上一篇 OpenFeign 的使用：https://www.cnblogs.com/l-y-h/p/14238203.html#_label3_2
    服务提供者仍然是 eureka_client_producer_8001。
```

### （2）配置 OpenFeign 基本代码环境

Step1：
　　创建模块 eureka_openfeign_client_consumer_9007。
　　修改父工程 与 当前工程 pom.xml 文件。
　　修改配置类。
　　在启动类上添加 @EnableFeignClients 注解。

```yaml
【依赖：】
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

【application.yml】
server:
  port: 9007
spring:
  application:
    name: eureka-openfeign-client-consumer

eureka:
  instance:
    appname: eureka-openfeign-client-consumer-9007 # 优先级比 spring.application.name 高
    instance-id: ${eureka.instance.appname} # 设置当前实例 ID
  client:
    register-with-eureka: true # 默认为 true，注册到 注册中心
    fetch-registry: true # 默认为 true，从注册中心 获取 注册信息
    service-url:
      # 指向 注册中心 地址，也即 eureka_server_7000 的地址。
      defaultZone: http://localhost:7000/eureka

# 设置 OpenFeign 超时时间（OpenFeign 默认支持 Ribbon）
ribbon:
  # 指的是建立连接所用的超时时间
  ConnectTimeout: 2000
  # 指的是建立连接后从服务器获取资源的超时时间（即请求处理的超时时间）
  ReadTimeout: 2000
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201600732-570518751.png)

 

Step2：
　　使用 @FeignClient 编写服务调用。



```java
【ProducerFeignService：】
package com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service;

import com.lyh.springcloud.common.tools.Result;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "EUREKA-CLIENT-PRODUCER-8001")
@Component
public interface ProducerFeignService {
    @GetMapping("/producer/user/get/{id}")
    Result getUser(@PathVariable Integer id);

    @GetMapping("/producer/user/testTimeout")
    Result testFeignTimeout();

    @GetMapping("/producer/user/testRuntimeError")
    Result testRuntimeError();
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201637649-211192524.png)

 

Step3：
　　编写 controller，并进行测试 openfeign 是否能成功访问服务。



```java
【ConsumerController】
package com.lyh.springcloud.eureka_openfeign_client_consumer_9007.controller;

import com.lyh.springcloud.common.tools.Result;
import com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.ProducerFeignService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/consumer/user")
public class ConsumerController {
    @Autowired
    private ProducerFeignService producerFeignService;

    @GetMapping("/get/{id}")
    public Result getUser(@PathVariable Integer id) {
        return producerFeignService.getUser(id);
    }

    @GetMapping("/testTimeout")
    public Result testFeignTimeout() {
        return producerFeignService.testFeignTimeout();
    }

    @GetMapping("/testRuntimeError")
    public Result testFeignRuntimeError() {
        return producerFeignService.testRuntimeError();
    }
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201711826-1330278011.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201743361-1908684054.png)

 

### （3）OpenFeign 实现服务降级

```
【步骤：】
Step1：在配置文件中，配置 feign.feign.enabled=true，开启服务降级。
Step2：定义一个 实现类，实现 服务调用的 接口，并为每个方法重写 调用失败的逻辑。
Step3：在 @FeignClient 注解中，通过 fallback 参数指定 该实现类。
```

Step1：
　　在配置文件中，开启服务降级。

```yaml
【application.yml】
# 开启服务降级
feign:
  hystrix:
    enabled: true
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201921749-831969968.png)

Step2：
　　定义一个实现类，实现 服务调用的接口。
　　@Component 注解不要忘了，启动时可能会报错。

```java
【ProducerFeignServiceImpl：】
package com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.impl;

import com.lyh.springcloud.common.tools.Result;
import com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.ProducerFeignService;
import org.springframework.stereotype.Component;

@Component
public class ProducerFeignServiceImpl implements ProducerFeignService {
    @Override
    public Result getUser(Integer id) {
        return Result.ok().message("系统异常，请稍后再试 -- 11111111111");
    }

    @Override
    public Result testFeignTimeout() {
        return Result.ok().message("系统异常，请稍后再试 -- 222222222222");
    }

    @Override
    public Result testRuntimeError() {
        return Result.ok().message("系统异常，请稍后再试 -- 333333333333");
    }
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202201953771-1704592202.png)



注：
　　未添加 @Component 注解，启动会报下面的错误。

```java
【报错信息：】
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'consumerController': 
Unsatisfied dependency expressed through field 'producerFeignService'; nested exception is org.springframework.beans.factory.BeanCreationException: 
Error creating bean with name 'com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.ProducerFeignService': 
FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: 
No fallback instance of type class com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.impl.ProducerFeignServiceImpl found for feign client EUREKA-CLIENT-PRODUCER-8001
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202028461-938864852.png)

 

Step3：
　　在 @FeignClient 注解上，通过 fallback  参数指定上面定义的实现类。

```java
package com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service;

import com.lyh.springcloud.common.tools.Result;
import com.lyh.springcloud.eureka_openfeign_client_consumer_9007.service.impl.ProducerFeignServiceImpl;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "EUREKA-CLIENT-PRODUCER-8001", fallback = ProducerFeignServiceImpl.class)
@Component
public interface ProducerFeignService {
    @GetMapping("/producer/user/get/{id}")
    Result getUser(@PathVariable Integer id);

    @GetMapping("/producer/user/testTimeout")
    Result testFeignTimeout();

    @GetMapping("/producer/user/testRuntimeError")
    Result testRuntimeError();
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202102983-1836770131.png)

 

Step4：
　　简单测试一下。
　　当服务提供者 宕机时，此时服务调用失败，将会执行 实现类中的逻辑。
　　而服务提供者正常提供服务时，由于上面已经在 服务提供者 处配置了 服务降级，则执行 服务提供者的服务降级策略。

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202139596-86309045.gif)

 

## 2.5 Hystrix 实现服务熔断

### （1）说明

```
【服务熔断：】
    服务熔断可以理解为特殊的服务降级，当某个服务出错或者响应时间长时，将会进行服务降级处理，
    从而熔断该服务的调用，但其会检测服务是否正常，若正常，则恢复服务调用。
注：
    代码方面 与 上面配置 超时服务自动降级 类似（在其基础上进行代码扩充）。

【断路器开启、关闭条件：】
开启条件：
    满足一定的请求阈值（默认 10 秒内请求数超过 20），且失败率达到阈值（默认 10 秒内 50% 的请求失败）。此时将会开启断路器。

关闭条件：
    断路器开启一段时间后（默认 5 秒），此时断路器处于半开状态，会对其中一部分请求进行转发，如果成功访问，则断路器关闭。
    若请求仍然失败，则再次进入开启状态。  
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202246002-55388469.png)

 

### （2）添加接口配置服务熔断

　　在 eureka_client_producer_8001 中新增一个接口，并配置服务熔断逻辑。

```java
【服务熔断：】
public Result testCircuitBreakerFallBack(@PathVariable Integer id) {
    return Result.ok().message("调用失败， ID 不能为负数");
}

@GetMapping("/testCircuitBreaker/{id}")
@HystrixCommand(fallbackMethod = "testCircuitBreakerFallBack", commandProperties = {
        // 是否开启断路器。默认为 true。
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
        // 在一定时间内，请求总数达到了阈值，才有资格进行熔断。默认 20 个请求。
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
        // 熔断之后，重新尝试恢复服务调用的时间，在此期间，会执行 fallbackMethod 定义的逻辑。默认 5 秒。
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),
        // 出错阈值，请求总数超过了阈值，并且调用失败率达到一定比率，会熔断。默认 50%。
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")
})
public Result testCircuitBreaker(@PathVariable Integer id) {
    if (id < 0) {
        throw new RuntimeException("ID 不能为负数");
    }
    return Result.ok().message("调用成功， ID = " + id);
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202318676-157805061.png) 

### （3）直接访问该服务测试一下（使用 JMeter 测试亦可）。

　　如下图所示，当请求失败达到一定比率，将会开启断路器。
　　一段时间后，尝试恢复服务调用，关闭断路器。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202345114-2035696902.gif)

 

 

## 2.6 Hystrix Dashboard

### （1）Dashboard

　　Hystrix 提供了图形化的监控工具（Hystrix Dashboard）进行准实时的调用监控，其可以持续的记录通过 Hystrix 发送的请求执行信息，并以图形、统计报表的形式呈现给用户。
　　SpringCloud 对其进行了整合，导入相关依赖即可。

 

### （2）使用

Step1：
　　引入依赖（hystrix-dashboard 以及 actuator）。

```xml
【依赖：】
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202518062-670419226.png)

 

Step2：
　　在启动类上添加 @EnableHystrixDashboard 注解，开启 Dashboard。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202548048-165549189.png)

 

Step3：
　　启动服务后，访问 http://localhost:8001/hystrix，填写需要监控的地址即可。
　　开启监控后，访问配置了 @HystrixCommand 注解的服务时，将会触发监控。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202613508-1319333701.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202624877-1089300585.png)

 

## 2.7 Dashboard 错误解决（Unable to connect to Command Metric Stream.）

### （1）错误

　　使用 Dashboard 最常见的错误就是 Unable to connect to Command Metric Stream。
　　根据控制台打印的 日志进行相关配置即可解决此错误。
　　错误效果如下图所示，

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202657141-1913996542.png)

 

### （2）错误一与解决：

```java
【错误一：】
控制台打印错误如下：
    Proxy opening connection to: http://localhost:8001/hystrix.stream

【解决：】
    在配置类中添加如下配置。
/**
 * 错误 Unable to connect to Command Metric Stream. 本质是无法解析  "/hystrix.stream"。
 * 自行配置一下即可。
 */
@Bean
public ServletRegistrationBean getServlet() {
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean<HystrixMetricsStreamServlet> registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202725531-512887175.png)

 

### （3）错误二与解决：



```yaml
【错误二：】
    上面解决了连接错误，但是仍然报错。
控制台打印错误如下：
    Origin parameter: http://localhost:8001/hystrix.stream is not in the allowed list of proxy host names.  
    If it should be allowed add it to hystrix.dashboard.proxyStreamAllowList.
    
【解决：】
    在配置文件中配置 proxyStreamAllowList 放行 host 即可。
hystrix:
  dashboard:
    proxy-stream-allow-list: "*"
#    proxy-stream-allow-list: "localhost"
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202803495-925572500.png)

 

# 三、服务降级、熔断 -- Sentinel

## 3.1 什么是 Sentinel？

### （1）什么是 Sentinel?

```
【Sentinel：】
    Sentinel 是阿里开源的项目，面向云原生微服务的高可用流控防护组件。
    从流量控制、熔断降级、系统负载保护等多个维度来保障服务之间的稳定性。
注：
    官方文档上写的还是很详细的，并提供了相应的中文文档。
    此处不过多描述，仅介绍使用，相关概念、原理 请自行翻阅文档。

【官网地址：】
    https://github.com/alibaba/Sentinel
    https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D
```



Sentinel 主要特性如下（图片来源于网络：）

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202202954398-796396125.png)

### （2）Sentinel 组成

```
【Sentinel 组成：】
    Sentinel 可以分为两部分：Java 客户端  、Dashboard 控制台。

Java 客户端（核心库）：    
    核心库 不依赖于 任何框架、库，能够运行于 Java7 及以上版本的 Java 运行时环境，
    同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
    
Dashboard 控制台：
    基于 SpringBoot 开发，打包后可直接运行，无需额外的 Tomcat 容器部署。
    控制台主要负责管理推送规则、监控、集群限流分配管理、机器发现等。
注：
    控制台使用参考文档：
        https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0
    控制台 jar 包下载：
        https://github.com/alibaba/Sentinel/releases

注：
    通过 Dashboard 控制台，可以很轻松的通过 web 页面设置 服务降级、熔断等规则。也可以通过代码的方式进行配置。
    个人比较倾向于 web 页面操作，省去了编写代码的工作（视工作环境而定）。
     后面介绍的 Dashboard 操作均以 web 界面进行操作，若想通过代码进行配置，请自行翻阅官方文档。
     web 页面编辑的规则 在 重启服务后 会丢失，需要将其进行持久化处理，一般都是持久化到 nacos 中（后续介绍）。
```

### （3）Hystrix 与 Sentinel 区别

```
【官网地址：】
    https://github.com/alibaba/Sentinel/wiki/Guideline:-%E4%BB%8E-Hystrix-%E8%BF%81%E7%A7%BB%E5%88%B0-Sentinel
注：
    详情请自行查阅官网文档。
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203143221-739543303.png)

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203209661-427290677.png)

 

## 3.2 基本模块构建

### （1）说明

```
【说明：】
    Sentinel 一般与 Nacos 一起使用，Nacos 使用后续介绍，此处仍使用 Eureka 进行整合。
    新建一个 eureka_client_sentinel_producer_8010 模块（引入核心库依赖）进行演示。
    从官网下载 sentinel-dashboard，用于启动 Dashboard 界面。
注：
    下载地址：https://github.com/alibaba/Sentinel/releases    
```

 

### （2）下载并启动 sentinel-dashboard。

Step1：下载 sentinel-dashboard。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203259006-1830148292.png)

 

 

Step2：命令行启动 jar 包。

```
【命令行启动 jar 包：】
    java -jar sentinel-dashboard-1.8.0.jar
注：
    启动后，默认访问端口为 8080，可以通过 -Dserver.port 参数进行修改。
    比如： java -jar -Dserver.port=8888 sentinel-dashboard-1.8.0.jar
    
【访问地址：】
    通过 IP + 端口 即可进入登录界面，登录用户、密码 都默认为 sentinel。
    比如：http:localhost:8888
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203333173-1988708434.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203347175-479054552.png)

 

 

### （3）基本模块 eureka_client_sentinel_producer_8010 创建

Step1：
　　修改 父工程、当前工程 pom.xml 文件，并引入相关依赖。

```xml
【直接引入依赖（带上版本号）：】
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>2.1.0.RELEASE</version>
</dependency>

【或者在父工程中统一进行版本管理：】
<properties>
  <spring.cloud.alibaba.version>2.1.0.RELEASE</spring.cloud.alibaba.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-alibaba-dependencies</artifactId>
          <version>${spring.cloud.alibaba.version}</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

注：
    spring-cloud-alibaba 各版本地址：
    https://github.com/alibaba/spring-cloud-alibaba/releases
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203419662-164755379.png)

 

Step2：
　　修改配置文件，配置 dashboard 信息。

```yaml
【application.yml】
server:
  port: 8010

spring:
  application:
    name: eureka_client_sentinel_producer_8010
  # 配置 sentinel
  cloud:
    sentinel:
      transport:
        # 配置 sentinel-dashboard 地址，此处在本地启动，所以 host 为 localhost
        dashboard: localhost:8888
        # 应用与 dashboard 交互的端口，默认为 8719 端口
        port: 8719

eureka:
  instance:
    appname: eureka_client_sentinel_producer_8010 # 优先级比 spring.application.name 高
    instance-id: ${eureka.instance.appname} # 设置当前实例 ID
  client:
    register-with-eureka: true # 默认为 true，注册到 注册中心
    fetch-registry: true # 默认为 true，从注册中心 获取 注册信息
    service-url:
      # 指向 注册中心 地址，也即 eureka_server_7000 的地址。
      defaultZone: http://localhost:7000/eureka
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203450432-1145103790.png)

 

Step3：
　　编写测试 controller，进行测试。

```java
【TestController】
package com.spring.cloud.eureka_client_sentinel_producer_8010.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/testSentinel")
public class TestController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }

    @GetMapping("/relation")
    public String relation() {
        return "relation";
    }
}
```

 

Step4：
　　启动 eureka_server_7000、以及当前服务 ，测试一下。
注：
　　即使服务启动，Sentinel Dashboard 也是空白的，需要调用一下当前服务的接口，其相关信息才会出现在 Sentinel 中。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203536542-1241366326.gif)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203546576-469505710.png)

 

## 3.3 Sentinel Dashboard 使用 -- 流控规则

### （1）什么是流量控制（flow control）？

```
【流量控制：】
    流量控制 本质上是 监控 应用流量的 QPS 或者 并发线程数等指标，
    当监控的指标达到 阈值 后，将会对访问进行限制，减少请求的正常响应。
    从而避免应用 被瞬间的高并发请求击垮而崩溃，进而保障应用的高可用性。
注：
    QPS（Query Per Second）：每秒查询率，即服务每秒能响应的查询（请求）次数。

【文档地址：】
    https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6
```

## （2）流控规则基本参数

```
【相关类：】
    com.alibaba.csp.sentinel.slots.block.flow.FlowRule
注：
    通过代码设置流控规则可以参考如下代码：
    https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-basic/src/main/java/com/alibaba/csp/sentinel/demo/flow/FlowQpsDemo.java

【流控规则页面参数：】
资源名：
    默认为 请求的资源路径，唯一。
    
针对来源：
    默认为 default，表示不区分来源。
注：
    网上查阅的资料都说可以根据 微服务名 进行限流，有待确认。
    此处未研究原理，留个坑，后续有时间再去研究。
    
阈值类型：
    QPS：
        当调用该资源接口的 QPS 达到阈值后，将会限流。
    线程数：
        当调用该资源接口的 线程数 达到阈值后，将会限流。
        
单机阈值：
    设置 阈值类型（QPS、线程数）的 阈值。

流控模式：
    直接：
        当资源接口达到限流条件时，会当前资源直接限流。
    关联：
        当某个资源关联的资源达到限流条件时，则 当前资源 被限流。
    链路：
        资源之间的调用形成调用链路，而 链路 模式仅记录 指定入口的流量，如果达到限流条件，则限流。
        
流控效果：
    快速失败：
        一旦达到限流条件，则直接抛异常（FlowException），然后走失败的处理逻辑。
    Warm Up：
        根据冷加载因子（coldFactor，默认 3），刚开始阈值请求数为 原阈值/coldFactor，经过一段时间后，阈值才会变为 原阈值。
    排队等待：
        请求会排队等待执行，匀速通过，此时的 阈值类型必须为 QPS。 
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203657239-1577170723.png)

### （3）演示 -- 直接、快速失败。

```
【说明：】
配置 /testSentinel/hello 的流控规则，不区分请求来源，
当 QPS 超过 1（即 1 秒钟超过 1 次查询）时，将会执行 直接限流，效果为 快速失败（会显示默认错误）。

【设置流控规则如下：】
资源名： /testSentinel/hello
针对来源： default
阈值类型： QPS
单机阈值： 1
流控模式： 直接
流控效果： 快速失败
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203732832-1663074521.png)

 

 

如下图所示，1 秒刷新一次是正常返回的结果，而 1 秒刷新多次后，将会输出默认的错误信息。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203754025-736651807.gif)

 

 

### （4）演示 -- 关联、快速失败。



```
【说明：】
现有资源 A、B，A 为 /testSentinel/hello，B 为 /testSentinel/relation。
配置 A 的流控规则，不区分请求来源，将 A 关联 B。
当 B 的 QPS 超过 1（即 1 秒钟超过 1 次查询）时，A 将会被限流，效果为 快速失败（会显示默认错误）。

【设置流控规则如下：】
资源名： /testSentinel/hello
针对来源： default
阈值类型： QPS
单机阈值： 1
流控模式： 关联
关联资源： /testSentinel/relation
流控效果： 快速失败

【实际使用场景举例：】
    两个资源之间具有依赖关系或者竞争资源时，可以使用关联。
比如：
    对数据库 读操作、写操作 进行限制，可以设置写操作优先，当 写操作 过于频繁时，读操作将被限流。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203828102-143771634.png)

 

 如下图所示，正常访问 A 是没问题的，但是 B 在 1 秒内多次刷新后，A 将会输出默认出错信息。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203853851-1469931751.gif)

 

 

### （5）演示 -- 直接、Wram Up。

```
【说明：】
配置 /testSentinel/hello 的流控规则，不区分请求来源，
设置 QPS 为 6，预热时间为 3 秒，则开始 QPS 阈值将为 2，预热时间结束后，QPS 会恢复到 6。

【设置流控规则如下：】
资源名： /testSentinel/hello
针对来源： default
阈值类型： QPS
单机阈值： 6
流控模式： 直接
流控效果： Warm Up
预热时长： 3

【实际场景举例：】
秒杀系统开启瞬间会有很多请求进行访问，如果不做限制，可能一下子系统直接崩溃了。
采用 Warm Up 方式，给系统一个缓冲时间，慢慢的增大 QPS。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203939829-1934498286.png)

 

 如下图所示，开始 QPS 较小，刷新容易报错，3 秒后，QPS 恢复原值，刷新不容易报错。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202203957022-253391046.gif)

 

 

### （6）演示 -- 直接、排队等待



```
【说明：】
配置 /testSentinel/hello 的流控规则，不区分请求来源，
QPS 超过 1 时，请求将会排队等待，超时时间为 2 秒，超时后将会输出错误信息。

【设置流控规则如下：】
资源名： /testSentinel/hello
针对来源： default
阈值类型： QPS
单机阈值： 1
流控模式： 直接
流控效果： 排队等待
超时时间： 2000

【实际场景举例：】
    通常用于处理 间隔性请求。
比如：
    消息队列，某瞬间的请求很多，但之后却没有请求，此时可以使用排队等待，将请求延迟执行（而不是直接拒绝）。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204031221-393040287.png)

 

 如下图所示，快速刷新页面时，请求将会排队等待执行，超时后将会报错。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204049816-877205202.gif)

 

## 3.4 @SentinelResource 注解

### （1）@SentinelResource 注解

　　@SentinelResource 可以等同于 Hystrix 中的 @HystrixCommand 注解进行理解。

```
【相关地址：】
    https://github.com/alibaba/Sentinel/wiki/%E6%B3%A8%E8%A7%A3%E6%94%AF%E6%8C%81
    
【@SentinelResource：】
    @SentinelResource 注解用于定义资源，并提供了可选的异常处理 以及 fallback 配置项。

value：资源名称，必须项（不能为空）。
    
blockHandler：
    指定限流异常（BlockException）发生后，应该执行的方法。
    注意事项：
        方法访问权限修饰符为 public。
        返回值类型 与 原方法返回值类型一致。
        参数类型 与 原方法一致，并追加一个 额外参数，参数类型为 BlockException。
        blockHandler 指定的函数默认需要与 原方法在同一个类中。

blockHandlerClass：
    指定限流异常（BlockException）发生后，应该执行的方法（此方法可以位于 其他类）。
    注意事项：
        方法访问权限修饰符为 public。
        方法必须是 static 函数，否则无法解析。

fallback：
    指定异常（除了 exceptionsToIgnore 指定的异常外的异常）发生后，应该执行的方法。
    注意事项：
        返回值类型 与 原方法返回值类型一致。
        参数类型 与 原方法一致，可以额外增加一个 Throwable 类型的参数用于接收对应的异常。
        fallback 指定的函数默认需要与 原方法在同一个类中。
        
fallbackClass：
    指定异常发生后，应该执行的方法（此方法可以位于 其他类）。
    注意事项：
        方法访问权限修饰符为 public。
        方法必须是 static 函数，否则无法解析。 

defaultFallback:
    指定异常发生后，执行默认的逻辑（即 通用处理逻辑）。
    与 fallback 类似，但 其指定的方法参数为空，可以额外增加一个 Throwable 类型的参数用于接收对应的异常。
   当 fallback 与 defaultFallback 同时存在时，只有 fallback 会生效。

exceptionsToIgnore:
    用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中（直接对外抛出原异常）。

【@SentinelResource 使用注意事项：】
    若 blockHandler 和 fallback 都配置了，当限流降级异常发生时(即 抛出 BlockException)，只会执行 blockHandler 指定的方法。
   若未配置 fallback、blockHandler 时，则限流降级时，则可能直接抛出 BlockException 异常，若方法本身没定义 throws BlockException，则异常将会被 JVM 包装为 UndeclaredThrowableException 异常。

可以简单的理解为： 
    blockHandler 用于指定 限流、降级 等异常(BlockException)发生后应该执行的规则。
    fallback 用于指定 其他异常（比如： RuntimeException）发生后应该执行的规则。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204204533-730519103.png)

### （2）@SentinelResource 使用举例

```java
【说明：】
    在 controller 中新增如下代码，
    fallback() 表示异常发生后的回调函数。
    defaultFallback() 表示异常发生后默认的回调函数。
    blockHandler()、blockHandler2() 表示 流控、降级 等 BlockException 异常发生后的回调函数。
        
    testBlockHandler() 用于测试 blockHandler 参数。
    testFallback() 用于测试 fallback 参数。
    testDefaultFallback() 用于测试 defaultFallback 参数。

【新增代码：】
public String fallback(Integer id, Throwable ex) {
    return ex.getMessage();
}

public String defaultFallback(Throwable ex) {
    return ex.getMessage();
}

public String blockHandler(BlockException ex) {
    return "block Handler";
}

public String blockHandler2(Integer id, BlockException ex) {
    return "block Handler --------- 2";
}

@GetMapping("/testBlockHandler")
@SentinelResource(value = "testBlockHandler", blockHandler = "blockHandler")
public String testBlockHandler() {
    return "ok";
}

@GetMapping("/testFallback/{id}")
@SentinelResource(value = "testFallback", blockHandler = "blockHandler", fallback = "fallback")
public String testFallback(@PathVariable Integer id) {
    if (id > 10) {
        throw new RuntimeException("fallback");
    }
    return "ok";
}

@GetMapping("/testDefaultFallback/{id}")
@SentinelResource(value = "testDefaultFallback", blockHandler = "blockHandler2", defaultFallback = "defaultFallback")
public String testDefaultFallback(@PathVariable Integer id) {
    if (id > 10) {
        throw new RuntimeException("defaultFallback");
    }
    return "ok";
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204244995-1850235777.png)

 

 

Step1：
　　访问 testBlockHandler()，测试 blockHandler 执行回调函数。
　　如下，给 testBlockHandler 添加流控规则，当 QPS 大于 1 时，将进行限流，此时将会执行 blockHandler 指定的回调函数。
注：
　　正常访问 testBlockHandler() 时，sentinel dashboard 会监控到两个资源名，此处应选择 @SentinelResource 注解中 value 定义的资源名，并配置 流控、降级 规则。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204328531-1433525712.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204343334-1173964226.png)

 

 

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204401973-650971044.gif)

 

 

Step2：
　　访问 testFallback()，测试 fallback 执行回调函数。
　　当 id 小于等于 10 时，正常调用。
　　当 id 大于 10 时，抛出异常后被 fallback 接收并执行回调函数。
　　同样，设置 流控规则，QPS 大于 1 时，限流，但由于 blockHandler 参数指定的回调方法参数 与 原方法不同，所以该回调函数不生效（空白）。
注：
　　限流、降级 等异常 执行的是 blockHandler 指定的回调函数。
　　而其他异常 执行的是 fallback 指定的回调函数。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204446180-1282060119.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204502176-1777455597.gif)

 

 

Step3：
　　访问 testDefaultFallback()，测试 defaultFallback 执行回调函数。
　　与上例类似，只是此处 blockHandler 回调函数参数 与 原方法相同，可以调用成功。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204540815-1180679976.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204553458-1071459561.gif)

 

## 3.5 Sentinel Dashboard 使用 -- 降级规则、系统规则（系统自适应限流）

### （1）熔断降级

　　熔断降级相关概念，前面在 Hystrix 已经介绍了。
　　此处仅演示 Sentinel 降级操作。



```
【降级策略：】
慢调用比例（SLOW_REQUEST_RATIO）： 
    若选择 慢调用比例 作为阈值，需外同时设置几个参数。
    参数：
        慢调用最大响应时间（最大 RT）。当请求响应时间大于该值时，将被统计为慢调用。
        比例阈值。比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。当慢调用比例大于该值时，将会触发熔断机制。
        最小请求数。单位统计时长内接收请求的最小数。
        熔断时长。熔断执行的时间。
    简单解释：
        当单位统计时长（statIntervalMs）内请求数目 大于 最小请求数，且 慢调用比例（超时请求占总请求数的比例） 大于 比例阈值 时，
        将会在一定的 熔断时长 内熔断请求（执行 熔断的相关代码）。
        熔断时长结束后，会进入探测恢复状态（即 Hystrix 中提到的 HALF-OPEN 状态），若检测到接下来的一个请求正常调用，则结束熔断，若依旧超时，则再次熔断。
注：
    此处的 HALF-OPEN 状态，来源于官网介绍（针对 Sentinel 1.8.0 及以上版本）。
    旧版本可能没有 HALF-OPEN 状态（没实际验证过）。

异常比例（ERROR_RATIO）：
    异常比例 与 慢调用比例 类似，异常比例的参数少了个 最大响应时间。
    简单解释：
        当单位统计时长（statIntervalMs）内请求数目 大于 最小请求数，且 异常比例（异常请求占总请求数的比例） 大于 比例阈值 时，
        将会在一定的 熔断时长 内熔断请求（执行 熔断的相关代码）。
        熔断时长结束后，会进入探测恢复状态（HALF-OPEN 状态），若检测到接下来的一个请求正常调用，则结束熔断，否则会再次被熔断。

异常数（ERROR_COUNT）：
    异常数 与 异常比例 类似，异常数 将参数 异常比例 变为 异常数（直接监控异常数，而非比例）。
    简单解释：
        当单位统计时长（statIntervalMs）内 异常请求数 超过 异常数阈值 后，
        将会在一定的 熔断时长 内熔断请求（执行 熔断的相关代码）。
        熔断时长结束后，会进入探测恢复状态（HALF-OPEN 状态），若检测到接下来的一个请求正常调用，则结束熔断，否则会再次被熔断。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204636597-1608752490.png)

### （2）演示 -- 异常比例

　　在上面 testDefaultFallback() 基础上，添加 降级 规则，演示 异常比例 降级。
注：
　　删除添加的流控规则，并指定 降级规则。
　　当降级发生时，将会触发 blockHandler 指定的回调方法。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204703535-1858332245.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204714065-644392971.gif)

### （3）系统规则

```
【相关文档：】
    https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81

【什么是系统规则（系统自适应限流）：】
    系统自适应限流 是从 整体维度 对 应用程序 入口流量 进行控制，即 在调用应用程序的 方法（接口） 前，将请求拦截下来。
    通过自适应的流控策略，让 系统的入口流量 和 系统的负载 达到一个平衡，即 让系统尽可能 在保证 最大吞吐量的同时 保证 系统整体的稳定性，
常用指标： 
    应用的负载（Load）、
    CPU 使用率、
    总体平均响应时间（RT）、
    入口 QPS、
    并发线程数 等。
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204755140-978105838.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204806497-338651774.gif)

 

## 3.6 OpenFeign 整合 Sentinel（引入 sentinel 依赖需要注意版本问题）

### （1）说明

```
【说明：】
    OpenFeign 一般用于消费端，此处以 eureka_client_consumer_9001 为基础，整合 Sentinel。
注：
    此处为了省事，直接用之前创建好的子模块，亦可自行创建新的模块。
    使用流程 与 Hystrix 类似。
```

### （2）整合 Sentinel

Step1:
　　在 eureka_client_consumer_9001 基础上引入 依赖。

```xml
【依赖：】
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- alibaba-sentinel -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>2.2.1.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.dataformat</groupId>
            <artifactId>jackson-dataformat-xml</artifactId>
        </exclusion>
    </exclusions>
</dependency>

【注意事项：（巨坑）】
    alibaba-sentinel 的版本可能会影响程序的执行。

此处使用的版本：
    springcloud  Hoxton.SR9
    spring.cloud.alibaba 2.1.0.RELEASE
    springboot 2.3.5.RELEASE
引入 alibaba-sentinel 依赖后，服务一直无法启动。
报错：
    nested exception is java.lang.AbstractMethodError: Receiver class com.alibaba.cloud.sentinel.feign.SentinelContractHolder does not define or inherit an implementation of the resolved method 'abstract java.util.List parseAndValidateMetadata(java.lang.Class)' of interface feign.Contract.

具体原因没整明白，但是如上引入依赖后，可以解决问题（有时间再去研究）。
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204912616-504429369.png)

 

 

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204924238-902744124.png)

 

 

Step2：
　　修改配置文件。开启 Sentinel 对 Feign 的支持。

```yaml
【application.yml】
# 开启 sentinel 对 feign 的支持
feign:
  sentinel:
    enabled: true
```

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202204957247-1011789760.png)

 

Step3：
　　使用 @FeignClient 编写服务调用。



```java
【ProducerFeignService】
package com.lyh.springcloud.eureka_client_consumer_9001.service;

import com.lyh.springcloud.eureka_client_consumer_9001.service.impl.ProducerFeignServiceImpl;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "EUREKA-CLIENT-SENTINEL-PRODUCER-8010", fallback = ProducerFeignServiceImpl.class)
@Component
public interface ProducerFeignService {

    @GetMapping("/testSentinel/testDefaultFallback/{id}")
    String testDefaultFallback(@PathVariable Integer id);

    @GetMapping("/testSentinel/hello")
    String hello();
}

【ProducerFeignServiceImpl】
package com.lyh.springcloud.eureka_client_consumer_9001.service.impl;

import com.lyh.springcloud.eureka_client_consumer_9001.service.ProducerFeignService;
import org.springframework.stereotype.Component;

@Component
public class ProducerFeignServiceImpl implements ProducerFeignService {
    @Override
    public String testDefaultFallback(Integer id) {
        return "系统异常，请稍后重试 --------- 1111111111111";
    }

    @Override
    public String hello() {
        return "系统异常，请稍后重试 --------- 2222222222222";
    }
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202205028378-1138507411.png)

 

 

Step4：
　　编写 controller。



```java
【TestController】
package com.lyh.springcloud.eureka_client_consumer_9001.controller;

import com.lyh.springcloud.eureka_client_consumer_9001.service.ProducerFeignService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RequestMapping("/consumer2")
@RestController
public class TestController {

    @Autowired
    private ProducerFeignService producerFeignService;

    @GetMapping("/testDefaultFallback/{id}")
    public String testDefaultFallback(@PathVariable Integer id) {
        return producerFeignService.testDefaultFallback(id);
    }

    @GetMapping("/hello")
    public String hello() {
        return producerFeignService.hello();
    }
}
```



![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202205100012-1368973141.png)

 

Step5：
　　在启动类上添加 @EnableFeignClients 注解，开启 feign 功能。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202205124588-1380726734.png)

 

Step6：
　　测试。
　　给 testDefaultFallback() 添加流控规则，QPS 大于 1 时将限流。
　　QPS 小于等于 1 时，正常访问。
　　若远程服务断开后，访问 testDefaultFallback() 将失败，从而执行本地添加的逻辑。

![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202205201461-1701764484.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202102/1688578-20210202205213798-1913961742.gif)

 