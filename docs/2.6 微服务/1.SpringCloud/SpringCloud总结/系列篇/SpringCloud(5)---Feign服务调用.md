[SpringCloud(5)---Feign服务调用](https://www.cnblogs.com/qdhxhz/p/9571600.html)

上一篇写了通过Ribbon进行服务调用，这篇其它都一样，唯一不一样的就是通过Feign进行服务调用。

注册中心和商品微服务不变，和上篇博客一样，具体参考：[SpringCloud(4)---Ribbon服务调用，源码分析](https://www.cnblogs.com/qdhxhz/p/9568481.html)

这边只重写订单微服务。

 **项目代码GitHub地址**：https://github.com/yudiandemingzi/spring-cloud-study

## 一、OrderService 订单微服务

####   1、pom.xml

这里相对于上一篇的订单微服务只要新添加一个jar包

```
    <!--feign依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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

#自定义负载均衡策略（一般不用配用默认的）
product-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    3、SpringBoot启动类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SpringBootApplication
//添加@EnableFeignClients注解
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    4、ProductOrderServiceImpl订单接口实现类

订单实体类和订单接口这里就不写类，和上篇一样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Service
public class ProductOrderServiceImpl implements ProductOrderService {

    @Autowired
    private ProductClient productClient;

    @Override
    public ProductOrder save(int userId, int productId) {

        //获取json格式的字符串数据
        String response = productClient.findById(productId);
        //Json字符串转换成JsonNode对象
        JsonNode jsonNode = JsonUtils.str2JsonNode(response);

        //将数据封装到订单实体中
        ProductOrder productOrder = new ProductOrder();
        productOrder.setCreateTime(new Date());
        productOrder.setUserId(userId);
        productOrder.setTradeNo(UUID.randomUUID().toString());
        //获取商品名称和商品价格
        productOrder.setProductName(jsonNode.get("name").toString());
        productOrder.setPrice(Integer.parseInt(jsonNode.get("price").toString()));

        //因为在商品微服务配置了集群，所以这里打印看下调用了是哪个集群节点，输出端口号。
        System.out.println(jsonNode.get("name").toString());
        return productOrder;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    5、ProductClient类

可以把这里类理解成，就是你需要调用的微服务的controller层（这里指商品微服务），这样相对于Ribbon来讲代码的可读性就高多了。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 商品服务客户端
 * name = "product-service"是服务端名称
 */
@FeignClient(name = "product-service")
public interface ProductClient {

    //这样组合就相当于http://product-service/api/v1/product/find
    @GetMapping("/api/v1/product/find")
    String findById(@RequestParam(value = "id") int id);

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    6、JsonUtils工具类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * json工具类
 */
public class JsonUtils {

    private static final ObjectMapper objectMappper = new ObjectMapper();
     //json字符串转JsonNode对象的方法
    public static JsonNode str2JsonNode(String str){
        try {
            return  objectMappper.readTree(str);
        } catch (IOException e) {
            return null;
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   7、OrderController类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
@RequestMapping("api/v1/order")
public class OrderController {

    @Autowired
    private ProductOrderService productOrderService;

    @RequestMapping("save")
    public Object save(@RequestParam("user_id")int userId, @RequestParam("product_id") int productId){
        return productOrderService.save(userId, productId);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   8、查看运行结果

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180901210241436-345984843.png)

同时同样它可以达到负载均衡的效果。

 

## 二、概念讲解

在使用Feign的时候，要注意使用requestBody，应该使用@PostMapping

####   1、执行流程

  总到来说，Feign的源码实现的过程如下：

  （1）首先通过@EnableFeignCleints注解开启FeignCleint

  （2）根据Feign的规则实现接口，并加@FeignCleint注解

  （3）程序启动后，会进行包扫描，扫描所有的@ FeignCleint的注解的类，并将这些信息注入到ioc容器中。

  （4）当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate

  （5）RequesTemplate在生成Request

  （6）Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp

  （7）最后Client被封装到LoadBalanceClient类，这个类结合类Ribbon做到了负载均衡。

#### 2、Feign和Ribbon比较优点

 （1） feign本身里面就包含有了ribbon，只是对于ribbon进行进一步封装

 （2） feign自身是一个声明式的伪http客户端，写起来更加思路清晰和方便

 （3） fegin是一个采用基于接口的注解的编程方式，更加简便

最后推荐一篇对源码解析不错的博客：[深入理解Feign之源码解析](https://blog.csdn.net/forezp/article/details/73480304)。