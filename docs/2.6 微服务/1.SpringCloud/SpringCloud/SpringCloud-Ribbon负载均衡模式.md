[TOC]

Spring Cloud Ribbon是一个<font color='blue'>基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现</font>。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。

因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的，包括后续我们将要介绍的Feign，它也是基于Ribbon实现的工具。

Ribbon为我们提供了很多负载均衡算法，例如轮询、随机等等，也可以自己定义算法。

<font color='blue'>Ribbon默认使用轮询的负载均衡模式。</font>

# 1、Ribbon内置的负载均衡规则
在 com.netflix.loadbalancer 包下有一个接口 IRule，它可以根据特定的算法从服务列表中选取一个要访问的服务，默认使用的是「轮询机制」
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200422125835.png)

- RoundRobinRule：轮询
- RandomRule：随机
- RetryRule：先按照 RoundRobinRule 的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
- WeightedResponseTimeRule：对 RoundRobinRule 的扩展，响应速度越快的实例选择权重越大，越容易被选择
- BestAvailableRule：会过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- AvailabilityFilteringRule：先过滤掉故障实例，再选择并发较小的实例
- ZoneAvoidanceRule：默认规则，复合判断 server 所在区域的性能和 server 的可用性选择服务器

# 2、负载均衡的替换
如果不想使用 Ribbon 默认使用的规则，我们可以通过自定义配置类的方式，手动指定使用哪一种。

需要注意的是，自定义配置类不能放在 @ComponentScan 所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的 Ribbon 客户端所共享，达不到特殊化定制的目的了。

因此我们需要在 Spring Boot 启动类所在包的外面新建一个包存放自定义配置类
```java
@Configuration
public class MyselfRule {
    @Bean
    public IRule rule(){
        //随机
        return new RandomRule();
    }
}
```
然后在启动类上添加如下注解，指定服务名及自定义配置类
```java
@RibbonClient(value = "CLOUD-PAYMENT-SERVICE", configuration = MyselfRule.class)
```

# 3、Ribbon默认负载轮询算法的原理
## 3.1 算法概述
rest 接口第几次请求数 % 服务器集群总个数 = 实际调用服务器位置下标，服务每次重启后 rest 请求数变为1。

## 3.2 源码
```java
public Server choose(ILoadBalancer lb, Object key) {
    if (lb == null) {
        log.warn("no load balancer");
        return null;
    }

    Server server = null;
    int count = 0;
    //循环获取服务，最多获取10次
    while (server == null && count++ < 10) {
        List<Server> reachableServers = lb.getReachableServers();
        List<Server> allServers = lb.getAllServers();
        //开启的服务个数
        int upCount = reachableServers.size();
        int serverCount = allServers.size();

        if ((upCount == 0) || (serverCount == 0)) {
            log.warn("No up servers available from load balancer: " + lb);
            return null;
        }

        //计算下一个服务的下标
        int nextServerIndex = incrementAndGetModulo(serverCount);
        server = allServers.get(nextServerIndex);

        if (server == null) {
            /* Transient. */
            Thread.yield();
            continue;
        }

        if (server.isAlive() && (server.isReadyToServe())) {
            return (server);
        }

        // Next.
        server = null;
    }

    if (count >= 10) {
        log.warn("No available alive servers after 10 tries from load balancer: "
                + lb);
    }
    return server;
}

//通过此方法获取服务的下标，使用了 CAS 和自旋锁
private int incrementAndGetModulo(int modulo) {
    for (;;) {
        int current = nextServerCyclicCounter.get();
        int next = (current + 1) % modulo;
        if (nextServerCyclicCounter.compareAndSet(current, next))
            return next;
    }
}
```
# 4、手写轮询算法
在服务提供者写一个方法，返回端口号看效果就行
```java
@GetMapping("/payment/lb")
public String roundLb(){
    return this.serverPort;
}
```
负载均衡接口
```java
public interface LoadBalancer {
    /**
     * 获取服务实例
     */
    ServiceInstance getInstance(List<ServiceInstance>serviceInstances);
}
```
算法实现类
```java
@Component
public class MyLb implements LoadBalancer {

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    /**
     * 使用「自旋锁」和「CAS」增加请求次数
     */
    public final int incrementAndGet() {
        int current;
        int next;
        do {
            current = atomicInteger.get();
            //防溢出
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        } while (!atomicInteger.compareAndSet(current, next));
        return next;
    }

    @Override
    public ServiceInstance getInstance(List<ServiceInstance> serviceInstances) {
        // 实际调用服务器位置下标 = rest 接口第几次请求数 % 服务器集群总个数
        int index = incrementAndGet() % serviceInstances.size();
        return serviceInstances.get(index);
    }
}
```
编写服务消费者方法，记得注释 @LoadBalanced 注解，否则不生效
```java
@GetMapping("/consumer/payment/lb")
public String roundLb(){
    List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
    if (instances == null || instances.size() <= 0){
        return null;
    }
    ServiceInstance instance = loadBalancer.getInstance(instances);
    URI uri = instance.getUri();
    return restTemplate.getForObject(uri + "/payment/lb", String.class);
}
```

# 5、Nginx 和 Ribbon 的对比
Nignx是一种集中式的负载均衡器。
何为集中式呢？简单理解就是将所有请求都集中起来，然后再进行负载均衡。如下图。
![](https://segmentfault.com/img/remote/1460000022470035)

Nginx是接收了所有的请求进行负载均衡的，而对于Ribbon来说它是在消费者端进行的负载均衡。如下图。
![](https://segmentfault.com/img/remote/1460000022470036)

> 请注意Request的位置，在Nginx中请求是先进入负载均衡器，而在Ribbon中是先在客户端进行负载均衡才进行请求的。

# 6、Ribbon总结

1. Ribbon 没有类似@EnableRibbon这样的注解
2. 新版的SpringCloud已经不使用Ribbon作为默认的负载均衡器了
3. 可以使用`@RibbonClients` 或`@RibbonClient` 注解来负载均衡相关策略的配置
4. 实现对应的接口就可以完成自定义负载均衡策略
5. Ribbon 配置的所有key都可以在`CommonClientConfigKey`类中查看