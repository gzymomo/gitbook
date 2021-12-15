[SpringCloud(6)---熔断降级理解、Hystrix实战](https://www.cnblogs.com/qdhxhz/p/9581440.html)

## 一、概念

####   1、为什么需要熔断降级

（**1）需求背景**

  它是系统负载过高，突发流量或者网络等各种异常情况介绍，常用的解决方案。

  在一个分布式系统里，一个服务依赖多个服务，可能存在某个服务调用失败，比如超时、异常等，如何能够保证在一个依赖出问题的情况下，不会导致整体服务失败。

  比如：某微服务业务逻辑复杂，在高负载情况下出现超时情况。

 内部条件：程序bug导致死循环、存在慢查询、程序逻辑不对导致耗尽内存

 外部条件：黑客攻击、促销、第三方系统响应缓慢。

**（2）解决思路**

  解决接口级故障的核心思想是优先保障核心业务和优先保障绝大部分用户。比如登录功能很重要，当访问量过高时，停掉注册功能，为登录腾出资源。

**（3）解决策略**

 熔断，降级，限流，排队。

####   2、什么是熔断

   一般是某个服务故障或者是异常引起的，类似现实世界中的‘保险丝’，当某个异常条件被触发，直接熔断整个服务，而不是一直等到此服务超时，为了防止防止整个系统的故障，

而采用了一些保护措施。过载保护。比如A服务的X功能依赖B服务的某个接口，当B服务接口响应很慢时，A服务X功能的响应也会被拖慢，进一步导致了A服务的线程都卡在了X功能

上，A服务的其它功能也会卡主或拖慢。此时就需要熔断机制，即A服务不在请求B这个接口，而可以直接进行降级处理。

####   3、什么是降级

   服务器当压力剧增的时候，根据当前业务情况及流量，对一些服务和页面进行有策略的降级。以此缓解服务器资源的的压力，以保证核心业务的正常运行，同时也保持了客户和

大部分客户的得到正确的相应。

**自动降级**：超时、失败次数、故障、限流

 （1）配置好超时时间(异步机制探测回复情况)；

 （2）不稳的的api调用次数达到一定数量进行降级(异步机制探测回复情况)；

 （3）调用的远程服务出现故障(dns、http服务错误状态码、网络故障、Rpc服务异常)，直接进行降级。

**人工降级**：秒杀、双十一大促降级非重要的服务。

####  4、熔断和降级异同

 **相同点**：

  1）从可用性和可靠性触发，为了防止系统崩溃

  2）最终让用户体验到的是某些功能暂时不能用

 **不同点：**

  1）服务熔断一般是下游服务故障导致的，而服务降级一般是从整体系统负荷考虑，由调用方控制

  2）触发原因不同，上面颜色字体已解释

####   5、熔断到降级的流程讲解

Hystrix提供了如下的几个关键参数，来对一个熔断器进行配置：

```
circuitBreaker.requestVolumeThreshold    //滑动窗口的大小，默认为20 
circuitBreaker.sleepWindowInMilliseconds //过多长时间，熔断器再次检测是否开启，默认为5000，即5s钟 
circuitBreaker.errorThresholdPercentage  //错误率，默认50%
```

3个参数放在一起，所表达的意思就是：

  每当20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开。

这里面有个很关键点，达到熔断之后，那么后面它就直接不去调该微服务。那么既然不去调该微服务或者调的时候出现异常，出现这种情况首先不可能直接把错误信息传给用户，所以针对熔断

我们可以考虑采取降级策略。所谓降级，就是当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的fallback回调，返回一个缺省值。 

这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强，当然这也要看适合的业务场景。

 

## 二、Hystrix实战

 

   使用到的组件包括：Eureka、Feign包括以下三个项目：

 

  （1）Eureka-server：  7001  注册中心

 

  （2）product-server ：8001  商品微服务

 

  （3）order-server ：  9001  订单微服务

 

注册中心、商品微服务、在之前博客都已搭建，这里就不重复写。这里只写order-server微服务。具体可以看上篇博客：[SpringCloud(5)---Feign服务调用](https://www.cnblogs.com/qdhxhz/p/9571600.html)

####   1、pom.xml

```
        <!--hystrix依赖，主要是用  @HystrixCommand -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

####   2、application.yml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
server:
  port: 9001

#指定注册中心地址
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7001/eureka/

#服务的名称
spring:
  application:
    name: order-service
    
#开启feign支持hystrix  (注意，一定要开启，旧版本默认支持，新版本默认关闭)
# #修改调用超时时间（默认是1秒就算超时）
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connectTimeout: 2000
        readTimeout: 2000
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####  3、SpringBoot启动类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SpringBootApplication
@EnableFeignClients
//添加熔断降级注解
@EnableCircuitBreaker
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   4、ProductClient

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 商品服务客户端
 * name = "product-service"是你调用服务端名称
 * fallback = ProductClientFallback.class，后面是你自定义的降级处理类，降级类一定要实现ProductClient
 */
@FeignClient(name = "product-service",fallback = ProductClientFallback.class)
public interface ProductClient {

    //这样组合就相当于http://product-service/api/v1/product/find
    @GetMapping("/api/v1/product/find")
    String findById(@RequestParam(value = "id") int id);

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   5、ProductClientFallback降级处理类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 针对商品服务，错降级处理
 */
@Component
public class ProductClientFallback implements ProductClient {

    @Override
    public String findById(int id) {

        System.out.println("ProductClientFallback中的降级方法");

        //这对gai该接口进行一些逻辑降级处理........
        return null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    6、OrderController类

  **注意**：fallbackMethod = "saveOrderFail"中的saveOrderFail方法中的参数类型，个数，顺序要和save一模一样，否则会报找不到saveOrderFail方法。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
@RequestMapping("api/v1/order")
public class OrderController {

    @Autowired
    private ProductOrderService productOrderService;

    @RequestMapping("save")
    //当调用微服务出现异常会降级到saveOrderFail方法中
    @HystrixCommand(fallbackMethod = "saveOrderFail")
    public Object save(@RequestParam("user_id")int userId, @RequestParam("product_id") int productId){

        return productOrderService.save(userId, productId);
    }

    //注意，方法签名一定要要和api方法一致
    private Object saveOrderFail(int userId, int productId){

        System.out.println("controller中的降级方法");

        Map<String, Object> msg = new HashMap<>();
        msg.put("code", -1);
        msg.put("msg", "抢购人数太多，您被挤出来了，稍等重试");
        return msg;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    7、测试

**（1）正常情况**

先将订单服务（order）和商品服务（product）同时启动，如图：

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180903222258382-1416578229.png)

订单服务调用商品服务正常

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180903222535207-475250943.png)

**（2）异常情况**

此刻我将商品微服务停掉：只启动订单微服务，这时去调用商品服务当然会出现超时异常情况。

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180903222724254-1039113267.png)

在调接口，发现已经成功到降级方法里

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180903223730795-1637233534.png)

在看controller中的降级方法和ProductClientFallback降级方法的实现先后顺序，它们的顺序是不固定的，有可能controller中降级方法先执行，也可能ProductClientFallback降级方法先执行。

具体要看哪个线程先获得cpu执行权。

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180903225927659-1273582782.png)

 

## 三、结合redis模拟熔断降级服务异常报警通知实战

主要是完善完善服务熔断处理，报警机制完善结合redis进行模拟短信通知用户下单失败。

####   1、pom.xml

```
    <!--springboot整合redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

####   2、application.yml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#服务的名称
#redis
spring:
  application:
    name: order-service
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    timeout: 2000
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   3、OrderController类

主要看降级方法的不同

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
@RequestMapping("api/v1/order")
public class OrderController {

    @Autowired
    private ProductOrderService productOrderService;

    //添加bean
    @Autowired
    private StringRedisTemplate redisTemplate;

    @RequestMapping("save")
    //当调用微服务出现异常会降级到saveOrderFail方法中
    @HystrixCommand(fallbackMethod = "saveOrderFail")
    public Object save(@RequestParam("user_id")int userId, @RequestParam("product_id") int productId,HttpServletRequest request){

        return productOrderService.save(userId, productId);
    }

    //注意，方法签名一定要要和api方法一致
    private Object saveOrderFail(int userId, int productId, HttpServletRequest request){
        
        //监控报警
        String saveOrderKye = "save-order";
        //有数据代表20秒内已经发过
        String sendValue = redisTemplate.opsForValue().get(saveOrderKye);
        final String ip = request.getRemoteAddr();

        //新启动一个线程进行业务逻辑处理
        new Thread( ()->{
            if (StringUtils.isBlank(sendValue)) {
                System.out.println("紧急短信，用户下单失败，请离开查找原因,ip地址是="+ip);
                //发送一个http请求，调用短信服务 TODO
                redisTemplate.opsForValue().set(saveOrderKye, "save-order-fail", 20, TimeUnit.SECONDS);

            }else{
                System.out.println("已经发送过短信，20秒内不重复发送");
            }
        }).start();

        Map<String, Object> msg = new HashMap<>();
        msg.put("code", -1);
        msg.put("msg", "抢购人数太多，您被挤出来了，稍等重试");
        return msg;
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    4、测试

  当20秒内连续发请求会提醒已发短信。

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180904133317445-1330239038.png)

 官方文档：https://github.com/Netflix/Hystrix/wiki/Configuration#execution.isolation.strategy