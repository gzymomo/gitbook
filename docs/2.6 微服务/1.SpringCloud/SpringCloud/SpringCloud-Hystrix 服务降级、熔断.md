[TOC]

# 1、HyStrix概述
Hystrix 是一个<font color='red'>用于处理分布式系统的延迟和容错</font>的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix 能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

「断路器」本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控(类似熔断保险丝)，向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

什么是 熔断和降级 呢？再举个例子，此时我们整个微服务系统是这样的。服务A调用了服务B，服务B再调用了服务C，但是因为某些原因，服务C顶不住了，这个时候大量请求会在服务C阻塞。
![](https://segmentfault.com/img/remote/1460000022470037)

服务C阻塞了还好，毕竟只是一个系统崩溃了。但是请注意这个时候因为服务C不能返回响应，那么服务B调用服务C的的请求就会阻塞，同理服务B阻塞了，那么服务A也会阻塞崩溃。

请注意，为什么阻塞会崩溃。因为这些请求会消耗占用系统的线程、IO 等资源，消耗完你这个系统服务器不就崩了么。
![](https://segmentfault.com/img/remote/1460000022470038)

所谓熔断就是服务雪崩的一种有效解决方案。当指定时间窗内的请求失败率达到设定阈值时，系统将通过断路器直接将此请求链路断开。

也就是我们上面服务B调用服务C在指定时间窗内，调用的失败率到达了一定的值，那么[Hystrix]则会自动将 服务B与C 之间的请求都断了，以免导致服务雪崩现象。

## 服务雪崩

在分布式微服务的架构体系下，一般都会存在多层级服务服务的调用链，当链路中的某个服务发生异常，最后导致整个系统不可用，这种现象称为服务雪崩效应。

[![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095257343-1190177416.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095257343-1190177416.png)

如上图所示，从最开始的整个系统正常状态，到单个服务出现异常，再到多个服务出现异常，到最后整个系统可不用，整个过程就是服务雪崩效应。如果在单个服务出现异常的时候，我们能及时发现、预防、处理，也就不会出现级联效果导致整个系统不可用。Hystrix 就是来保证上面的情况发生时能停止级联故障，保证系统稳定运行的。



其实这里所讲的熔断就是指的[Hystrix]中的断路器模式，你可以使用简单的@[Hystrix]Command注解来标注某个方法，这样[Hystrix]就会使用断路器来“包装”这个方法，每当调用时间超过指定时间时(默认为1000ms)，断路器将会中断对这个方法的调用。



## Hystrix遵循的设计原则

1. 避免线程耗尽
   由于被调用方出现问题，调用方无法及时获取响应结果，而一直在发送请求，最终会耗尽所有线程的资源。
2. 快速失败
   当被调用方出现问题后，调用方发起的请求可以快速失败并返回，这样就不用一直阻塞住，同时也释放了线程资源。
3. 支持回退
   发起的请求在返回失败后，我们可以让用户有回退的逻辑，比如获取备用数据，从缓存中获取数据，记录日志等操作。
4. 资源隔离
   当你的服务依赖了 A、B、C 三个服务，当只有 C 服务出问题的时候，如果没做隔离，最终也会发生雪崩效应，导致整个服务不可用，如果我们进行了资源隔离，A、B、C 三个服务都是相互隔离的，即使 C 服务出问题了，那也不影响 A 和 B。这其实就跟不要把所有的鸡蛋放进一个篮子里是一样的道理。
5. 近实时监控
   它能帮助我们了解整个系统目前的状态，有哪些服务有问题，当前流量有多大，出问题后及时告警等。



## **Hystrix 两种隔离方式**

Hystrix 支持线程池和信号量两种隔离方式，默认使用的线程池隔离。

线程池隔离是当用户请求到 A 服务后，A 服务需要调用其他服务，这个时候可以为不同的服务创建独立的线程池，假如 A 需要调用 B 和 C，那么可以创建 2 个独立的线程池，将调用 B 服务的线程丢入到一个线程池，将调用 C 服务的线程丢入到另一个线程池，这样就起到隔离效果，就算其中某个线程池请求满了，无法处理请求了，对另一个线程池也没有影响。

信号量隔离就比较简单了，信号量就是一个计数器，比如初始化值是 100，那么每次请求过来的时候就会减 1，当信号量计数为 0 的时候，请求就会被拒绝，等之前的请求处理完成后，信号量会加 1，同时也起到了限流的作用，这就是信号量隔离，信号量隔离是在请求主线程中执行的。

线程池隔离的特点是 Command 运行在独立的线程池中，可以支持超时，是单独的线程，支持异步。信号量隔离运行在调用的主线程中，不支持超时，只能同步调用。



# 2、HyStrix几个概念

## 2.1 服务降级
不让客户端等待，并立即返回一个友好的提示（服务器忙，请稍后再试）

🎃 哪些情况会发生服务降级：
- 程序运行异常
- 超时
- 服务熔断引起服务降级
- 线程池/信号量打满也会导致服务降级

## 2.2 服务熔断
类似保险丝，电流过大时，直接熔断断电。

熔断机制是应对雪崩效应的一种微服务链路保护机制，当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息，当检测到该节点微服务调用响应正常后，恢复调用链路。

服务降级 → 服务熔断 → 恢复调用链路。

## 2,3 服务限流
对于高并发的操作，限制单次访问数量。

# 3、服务降级的用法与分析
超时导致服务器变慢：超时不再等待； 出错（宕机或程序运行出错）：要有备选方案
- 服务提供者超时了，调用者不能一直卡死等待，必须要服务降级
- 服务提供者宕机了，调用者不能一直卡死等待，必须要服务降级
- 服务提供者没问题，调用者自己出现故障或者有自我要求（自己的等待时间必须小于服务提供者）

## 3.1 给服务提供方设置服务降级
1. 在需要服务降级的方法上标注注解，fallbackMethod 代表回退方法，需要自己定义，@HystrixProperty 中设置的是该方法的超时时间，如果超过该事件则自动降级
当运行超时或服务内部出错都会调用回退方法：
```java
@HystrixCommand(
    fallbackMethod = "timeoutHandler", 
    commandProperties = {
    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})
public String timeout(Long id) {
    int time = 3000;
    try {
        TimeUnit.MILLISECONDS.sleep(time);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    //模拟异常
    //int i = 10 / 0;
    return "线程：" + Thread.currentThread().getName();
}
```
2. 在启动类上添加注解，开启降级
` @EnableCircuitBreaker `

## 3.2 给服务消费方设置服务降级
1. 添加配置
```yml
# 在feign中开启hystrix
feign:
  hystrix:
    enabled: true
```
```java
@HystrixCommand(
    fallbackMethod = "timeoutHandler", 
    commandProperties = {
    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
})
public String timeout(@PathVariable("id") Long id) {
    int i = 1/0;
    return hystrixService.timeout(id);
}
```

2. 在启动类上添加注解
` @EnableHystrix `

## 3.3 问题
以上配置方式存在的问题：

每个业务方法对应一个回退方法，代码膨胀
每个业务方法上都配置相同的处理，代码冗余
🎉 解决方式1：在类上配置一个全局回退方法，相当于是一个通用处理，当此回退方法能满足你的需求，就无需在方法上指定其它回退方法，如果需要使用特定的处理方法可以再在业务方法上定义
`  @DefaultProperties(defaultFallback = "globalFallbackMethod")   `
🎉 解决方式2：但此时处理代码和依然和业务代码混合在一起，我们还可以使用另一种方式：编写一个类实现 Feign 的调用接口，并重写其方法作为回退方法，然后在 @FeignClient 注解上添加 fallback 属性，值为前面的类。

# 4、服务熔断的用法与分析
在SpringCloud中，熔断机制通过 Hystrix 实现。Hystrix 监控微服务间的调用状况，当失败的调用到一定阈值，默认 5 秒内 20 次调用失败就会启动熔断机制。熔断机制的注解是 @HystrixCommand。
```java
@HystrixCommand(
    fallbackMethod = "paymentCircuitBreakerFallback", 
    commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), //是否开启断路器
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), //请求次数
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), //时间窗口期
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60") //失败率达到多少后跳闸
})
public String circuitBreaker(Long id) {
    if (id < 0) {
        throw new RuntimeException("id 不能为负数");
    }
    return Thread.currentThread().getName() + "\t" + "调用成功，流水号：" + IdUtil.simpleUUID();
}

public String circuitBreakerFallback(Long id) {
    return "id 不能为负数，你的id = " + id;
}
```

@HystrixProperty 中的配置可以参考 com.netflix.hystrix.HystrixCommandProperties 类
详见官方文档：https://github.com/Netflix/Hystrix/wiki/Configuration
也有雷锋同志做了翻译：https://www.jianshu.com/p/39763a0bd9b8

🎨 **熔断类型**

熔断打开：请求不再调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态。
熔断半开：部分请求根据规则调用服务，如果请求成功且符合规则，则关闭熔断。
熔断关闭：不会对服务进行熔断。
🎨 **断路器什么时候起作用**？

根据上面配置的参数，有三个重要的影响断路器的参数

快照时间窗：回路被打开、拒绝请求到再尝试请求并决定回路是否继续打开的时间范围，默认是 5 秒
请求总数阈值：在一个滚动窗口中，打开断路器需要的最少请求数，默认是 20 次（就算前 19 次都失败了，断路器也不会被打开）
错误百分比阈值：错误请求数在总请求数所占的比例，达到设定值才会触发，默认是 50%
🎨 **断路器开启或关闭的条件**

当请求达到一定阈值时（默认 20 次）
当错误率达到一定阈值时（默认 50%）
达到以上条件断路器开启
当开启的时候，所有请求都不会转发
当断路器开启一段时间后（默认 5 秒）进入半开状态，并让其中一个请求进行转发，如果成功断路器关闭，如果失败继续开启，重复第 4 和 5 步
🎨 **断路器开启之后会发生什么**？

再有请求调用时，不再调用主逻辑，而是调用降级 fallback。
断路器开启之后，Hytrix 会启动一个休眠时间窗，在此时间内，fallback 会临时称为主逻辑，当休眠期到了之后，断路器进入半开状态，释放一个请求到原来的主逻辑上，如果请求成功返回，则断路器关闭，如果请求失败，则继续进入打开状态，休眠时间窗重新计时。

# 5、Hystrix服务熔断的工作流程
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200424225855.png)

# 6、Hystrix DashBoard上手
## 6.1 搭建
1. maven依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-hystrix-dashboard</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
2. 添加配置
```yml
server:
  port: 9001
```
3. 开启Hystrix DashBoard
```java
@SpringBootApplication
@EnableHystrixDashboard
public class ConsumerHystrixDashBoard9001 {
    public static void main(String[] args){
        SpringApplication.run(ConsumerHystrixDashBoard9001.class, args);
    }
}
```
浏览器输入 http://localhost:9001/hystrix，出现以下界面即启动成功:
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200424231413.png)

# 7、使用
注意：想要被 Hystrix DashBoard 监控的服务必须导入此依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
在被监控服务的主启动类里添加如下代码，否则某些旧版本可能报错 Unable to connect to Command Metric Stream.
```java
/**
 * 此配置是为了服务监控而配置，与服务容错本身无关,SpringCloud升级后的坑
 * ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
 * 只要在自己的项目里配置上下面的servlet就可以了
 */
@Bean
public ServletRegistrationBean getServlet(){
    HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
    registrationBean.setLoadOnStartup(1);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    return registrationBean;
}
```
在 Hystrix DashBoard 页面输入基本信息，进入仪表盘界面。

![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200424234247.png)

大致情况如下所示：
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200424234642.png)

操作界面分析：
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200424233620.png)

# 8、Hystrix实战

### 引入Hystrix 依赖

1. 引入相关依赖

   

   ```
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

2. 在`ribbon-client`服务的启动类上添加注解`@EnableHystrix`
   这时候服务已经可以启动成功了

### Hystrix 的三种使用方式

### 1.@HystrixCommand 注解方式

HystrixCommand 注解作用于方法上，哪个方法想要使用 Hystrix 来进行保护，就在这个方法上增加 HystrixCommand 注解。

比如在我们的queryPort方法上添加@HystrixCommand注解：



```
@HystrixCommand(commandKey = "queryPort")
@GetMapping("queryPort")
public String queryPort(){
    return providerFeign.queryPort();
}
```

其中commandKey不指定的话，会默认使用方法名，这里也是queryPort；

@HystrixCommand 有很多默认的配置，比如超时时间，隔离方式等；我们可以手动指定配置信息有比如 commandKey、groupKey、fallbackMethod 等。

**配置回退方法fallbackMethod**

使用@HystrixCommand 注解方式配置回退方法，需要将回退方法定义在HystrixCommand所在的类中，且回退方法的签名与调用的方法签名（入参，返回值）应该保持一致，比如：



```
private String queryPortFallBack(){
    return "sorry queryPort,jinglingwang.cn no back!";
}

//调用方法改造
@HystrixCommand(commandKey = "queryPort",fallbackMethod = "queryPortFallBack")
```

然后我们把eureka-provider服务停掉或者故意超时，访问接口会出现如下图所示的结果：

[![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095333921-437918536.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095333921-437918536.png)

**我们也可以结合`@HystrixProperty`注解来丰富我们的配置**



```
@HystrixCommand(commandKey = "queryPort",commandProperties ={
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000"),//超时时间，默认1000，即1秒
        @HystrixProperty(name = "execution.isolation.strategy",value = "SEMAPHORE"),//信号量隔离级别
        @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests",value = "50") //信号量模式下，最大请求并发数，默认10
    },fallbackMethod = "queryPortFallBack")
@GetMapping("queryPort")
public String queryPort(){
    return providerFeign.queryPort();
}
```

上面的一些配置信息我们还可以配置到配置文件中，效果是一样的：



```
# queryPort 是@HystrixCommand注解里面的commandKey
# 隔离方式，SEMAPHORE：信号量隔离，THREAD：线程隔离（默认值）
hystrix.command.queryPort.execution.isolation.strategy = SEMAPHORE
# 信号量模式下，最大请求并发数，默认10
hystrix.command.queryPort.execution.isolation.semaphore.maxConcurrentRequests = 50
# 超时时间，默认值是1000，也就是1秒；在HystrixCommandProperties类可以看到
hystrix.command.queryPort.execution.isolation.thread.timeoutInMilliseconds = 3000
```

**下面的代码展示了线程隔离级别下的配置示例：**



```
@HystrixCommand(commandKey = "queryTempPort",
        threadPoolProperties = {
            @HystrixProperty(name = "coreSize", value = "30"),
            @HystrixProperty(name = "maxQueueSize", value = "101"),
            @HystrixProperty(name = "keepAliveTimeMinutes", value = "2"),
            @HystrixProperty(name = "queueSizeRejectionThreshold", value = "15"),
            @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "12"),
            @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "1440")
        }
        ,fallbackMethod = "queryTempPortFallBack")
@GetMapping("queryTempPort")
public String queryTempPort(){
    return providerTempFeign.queryPort();
}
```

我们也可以使用`@DefaultProperties`注解来配置默认属性；

@DefaultProperties是作用在类上面的，可以配置一些比如groupKey、threadPoolKey、commandProperties、threadPoolProperties、ignoreExceptions和raiseHystrixExceptions等属性。方法级别的@HystrixCommand命令中单独指定了的属性会覆盖默认的属性，比如：



```
@RestController
@DefaultProperties(groupKey = "DefaultGroupKey")
public class RibbonController{
   ...

    @HystrixCommand(commandKey = "queryTempPort",groupKey="eureka-provider-temp",
            threadPoolProperties = {
                @HystrixProperty(name = "coreSize", value = "30"),
                @HystrixProperty(name = "maxQueueSize", value = "101"),
                @HystrixProperty(name = "keepAliveTimeMinutes", value = "2"),
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "15"),
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "12"),
                @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "1440")
            }
            ,fallbackMethod = "queryTempPortFallBack")
    @GetMapping("queryTempPort")
    public String queryTempPort(){
        return providerTempFeign.queryPort();
    }
}
```

### 2.Feign 整合 Hystrix

**开启Feign对Hystrix的支持**

在配置文件添加如下配置



```
# 如果为true，则将使用Hystrix断路器包装OpenFeign客户端，默认是false
feign.hystrix.enabled=true
```

**配置fallback**

1. 为Feign配置回退方法，将fallback属性设置成回退的类名，例如：

   

   ```
   @Component
   public class ProviderTempFeignFallback implements ProviderTempFeign{
   
       @Override
       public String queryPort(){
           return "sorry ProviderTempFeign, jinglingwang.cn no back!";
       }
   }
   
   @FeignClient(value = "eureka-provider-temp",fallback = ProviderTempFeignFallback.class)
   public interface ProviderTempFeign{
   
       @RequestMapping("/queryPort")
       String queryPort();
   }
   ```

2. 我们保留上面的@HystrixCommand注解，然后启动项目，把eureka-provider项目的接口加一个断点，保证接口会超时。同时配置有两个fallback时，发现最后生效的是@HystrixCommand注解配置的fallback，说明@HystrixCommand注解的优先级要高一些，返回结果如图：

   [![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095352875-46131937.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095352875-46131937.png)
   然后我们把@HystrixCommand注解注释掉，再重启，成功执行了Feign配置的fallback，效果如图：

   [![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095403605-1020875720.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201203095403605-1020875720.png)

**fallback返回失败的原因**

如果需要访问导致失败回退的原因，可以使用@FeignClient内的fallbackFactory属性。



```
@Component
public class ProviderFeignFallbackFactory implements FallbackFactory<ProviderFeign>{

    @Override
    public ProviderFeign create(Throwable cause){
        return new ProviderFeign(){
            @Override
            public String queryPort(){
                return "sorry ProviderFeignFallbackFactory, jinglingwang.cn no back! why? ==>" + cause.getCause();
            }
        };
    }
}

@FeignClient(value = "eureka-provider",fallbackFactory = ProviderFeignFallbackFactory.class)
public interface ProviderFeign{
    /**
     * 调用服务提供方，其中会返回服务提供者的端口信息
     * @return jinglingwang.cn
     */
    @RequestMapping("/queryPort")
    String queryPort();

}
```

### 3.网关中使用Hystrix

网关中使用Hystrix等到了整合网关的时候再细讲。

### hystrix配置总结

1. 默认配置是全局有效的

   

   ```
   # 配置 Hystrix 默认的配置
   # To set thread isolation to SEMAPHORE
   hystrix.command.default.execution.isolation.strategy: SEMAPHORE
   hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 3000
   hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests: 40
   ```

2. 单独为Feign Client 来指定超时时间

   

   ```
   # 单独为 ProviderFeign 配置
   hystrix.command.ProviderFeign.execution.isolation.strategy = SEMAPHORE
   # 超时时间
   hystrix.command.ProviderFeign.execution.isolation.thread.timeoutInMilliseconds = 5000
   # 最大请求并发数，默认10
   hystrix.command.ProviderFeign.execution.isolation.semaphore.maxConcurrentRequests: 200
   ```

3. 单独为ProviderTempFeign类的queryPort()方法进行配置

   

   ```
   # 单独为ProviderTempFeign类的queryPort()方法配置
   hystrix.command.ProviderTempFeign#queryPort().execution.isolation.strategy = THREAD
   # 超时时间
   hystrix.command.ProviderTempFeign#queryPort().execution.isolation.thread.timeoutInMilliseconds = 5000
   ```

4. 使用 @HystrixCommand 注解配置
   具体做法可以参考上面的示例代码

Hystrix的配置项有很多，其他属性的配置key可以参考`HystrixCommandProperties`类。

### **如何合理的配置Hystrix和Ribbon超时时间**

Hystrix 的超时时间是和Ribbon有关联的，如果配置的不对，可能会出现莫名其妙的问题。

在Hystrix源码里面是建议`hystrixTimeout`应该大于等于`ribbonTimeout`的时间的，否则会输出一句警告：



```
LOGGER.warn("The Hystrix timeout of " + hystrixTimeout + "ms for the command " + commandKey +
				" is set lower than the combination of the Ribbon read and connect timeout, " + ribbonTimeout + "ms.");
```

而在取`ribbonTimeout`配置值的时候，是有一个计算公式的：
`ribbonTimeout = (ribbonReadTimeout + ribbonConnectTimeout) * (maxAutoRetries + 1) * (maxAutoRetriesNextServer + 1);`

假如我们Ribbon的超时时间配置如下：



```
#读超时
ribbon.ReadTimeout=3000
#连接超时
ribbon.ConnectTimeout=3000
#同一台实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetries=0
#重试负载均衡其他的实例最大重试次数,不包括首次调用
ribbon.MaxAutoRetriesNextServer=1
```

将上面的值代入到公式计算，得到结果：ribbonTimeout=(3000+3000)*(0+1)*(1+1)，结果为12000，也就是说Hystrix 的超时时间建议配置值要大于等于12000，也就是12秒。

# Hystrix总结

1. Hystrix 支持@HystrixCommand 命令和配置文件两种方式进行配置
2. Hystrix 支持两种隔离级别，在网关中建议使用信号量的方式，能起到一定限流的作用
3. Hystrix 的线程池隔离级别可以为每个client分别配置线程池，起到资源隔离的作用
4. Hystrix 的线程池隔离级别中使用 ThreadLocal 时数据可能会丢失，需要单独处理
5. Hystrix 的fallback我们可以用来记录日志或者进行相应的业务告警
6. Hystrix 超时时间的合理计算和ribbon的配置有关系，否则可能出现莫名其妙的问题