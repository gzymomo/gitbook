[SpringCloud(4)---Ribbon服务调用，源码分析](https://www.cnblogs.com/qdhxhz/p/9568481.html)

本篇模拟订单服务调用商品服务，同时商品服务采用集群部署。

注册中心服务端口号7001，订单服务端口号9001,商品集群端口号：8001、8002、8003。

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831214647444-512867267.png)

各服务的配置文件这里我这边不在显示了，和上篇博客配置一样。博客地址：[SpringCloud(3)---Eureka服务注册与发现](https://www.cnblogs.com/qdhxhz/p/9357502.html)

 

## 一、商品中心服务端

####   1、pom.xml

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

####  2、Product商品实体类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product implements Serializable {

    private int id;
    //商品名称
    private String name;
    //价格,分为单位
    private int price;
    //库存
    private int store;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   3、ProductService商品接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface ProductService {

    //查找所有商品
    List<Product> listProduct();

    //根据商品ID查找商品
    Product findById(int id);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   4、ProductServiceImpl商品实现类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Service
public class ProductServiceImpl implements ProductService {

    private static final Map<Integer, Product> daoMap = new HashMap<>();

    //模拟数据库商品数据
    static {
        Product p1 = new Product(1, "苹果X", 9999, 10);
        Product p2 = new Product(2, "冰箱", 5342, 19);
        Product p3 = new Product(3, "洗衣机", 523, 90);
        Product p4 = new Product(4, "电话", 64345, 150);

        daoMap.put(p1.getId(), p1);
        daoMap.put(p2.getId(), p2);
        daoMap.put(p3.getId(), p3);
        daoMap.put(p4.getId(), p4);
    }

    @Override
    public List<Product> listProduct() {
        Collection<Product> collection = daoMap.values();
        List<Product> list = new ArrayList<>(collection);
        return list;
    }
    @Override
    public Product findById(int id) {
        return daoMap.get(id);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    5、ProductController

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
@RequestMapping("/api/v1/product")
public class ProductController {

    //集群情况下，用于订单服务查看到底调用的是哪个商品微服务节点
    @Value("${server.port}")
    private String port;

    @Autowired
    private ProductService productService;

     //获取所有商品列表
    @RequestMapping("list")
    public Object list(){
        return productService.listProduct();
    }

    //根据id查找商品详情
    @RequestMapping("find")
    public Object findById(int id){
        Product product = productService.findById(id);
        Product result = new Product();
        BeanUtils.copyProperties(product,result);
        result.setName( result.getName() + " data from port="+port );
        return result;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    6、测下该服务接口是否成功

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831215738536-922460920.png)

 

## 二、订单中心服务端

####   1、pom.xml

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

####   2、ProductOrder商品订单实体

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProductOrder implements Serializable {  
    //订单ID
    private int id;
    // 商品名称
    private String productName;
    //订单号
    private String tradeNo;
    // 价格,分
    private int price;
    //订单创建时间
    private Date createTime;
    //用户id
    private int userId;
    //用户名
    private String userName;
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    3、ProductOrderService订单接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 订单业务类
 */
public interface ProductOrderService {
     //下单接口
     ProductOrder save(int userId, int productId);
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    4、ProductOrderServiceImpl订单实现类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Service
public class ProductOrderServiceImpl implements ProductOrderService {

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public ProductOrder save(int userId, int productId) {
        //product-service是微服务名称（这里指向的商品微服务名称）,api/v1/product/find?id=? 就是商品微服务对外的接口
        Map<String, Object> productMap = restTemplate.getForObject("http://product-service/api/v1/product/find?id=" + productId, Map.class);

        ProductOrder productOrder = new ProductOrder();
        productOrder.setCreateTime(new Date());
        productOrder.setUserId(userId);
        productOrder.setTradeNo(UUID.randomUUID().toString());
        //获取商品名称和商品价格
        productOrder.setProductName(productMap.get("name").toString());
        productOrder.setPrice(Integer.parseInt(productMap.get("price").toString()));
        
        //因为在商品微服务配置了集群，所以这里打印看下调用了是哪个集群节点，输出端口号。
        System.out.println(productMap.get("name").toString());
        return productOrder;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    5、OrderController类

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

####   6、SpringBoot启动类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

    //当添加@LoadBalanced注解，就代表启动Ribbon,进行负载均衡
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####   7、接口测试

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831220657511-1850638471.png)

多调几次接口，看后台打印

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831220750632-41476104.png)

发现订单服务去掉商品服务的时候，不是固定节点，而且集群的每个节点都有可能。所以通过Ribbon实现了负载均衡。

 

## 三、Ribbon源码分析

####   1、@LoadBalanced注解作用

在springcloud中，引入Ribbon来作为客户端时，负载均衡使用的是被`@LoadBalanced`修饰的`RestTemplate`对象。

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831221407209-1808299223.png)

   RestTemplate 是Spring自己封装的http请求的客户端，也就是说它只能发送一个正常的Http请求,这跟我们要求的负载均衡是有出入的，还有就是这个请求的链接上的域名

是我们微服的一个服务名，而不是一个真正的域名，那它是怎么实现负载均衡功能的呢？

我们来看看RestTemplate的父类InterceptingHttpAccessor。

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831221647049-158144836.png)

   从源码我们可以知道InterceptingHttpAccessor中有一个拦截器列表List<ClientHttpRequestInterceptor>，如果这个列表为空，则走正常请求流程，如果不为空则走

拦截器，所以只要给RestTemplate添加拦截器，而这个拦截器中的逻辑就是Ribbon的负载均衡的逻辑。通过`@LoadBalanced注解`为RestTemplate配置添加拦截器。

具体的拦截器的生成在LoadBalancerAutoConfiguration这个配置类中，所有的RestTemplate的请求都会转到Ribbon的负载均衡器上

(当然这个时候如果你用RestTemplate发起一个正常的Http请求时走不通，因为它找不到对应的服务。)这样就实现了Ribbon的请求的触发。

####    2.拦截器都做了什么？

上面提到过，发起http后请求后，请求会到达到达拦截器中，在拦截其中实现负载均衡，先看看代码：

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831222225985-1013783158.png)

   我们可以看到在intercept()方法中实现拦截的具体逻辑，首先会根据传进来的请求链接，获取微服的名字serviceName,然后调用LoadBalancerClient的

execute(String serviceId, LoadBalancerRequest<T> request)方法，这个方法直接返回了请求结果，所以正真的路由逻辑在LoadBalancerClient的实现类中，

而这个实现类就是RibbonLoadBalancerClient，看看execute()的源码：

![img](https://images2018.cnblogs.com/blog/1090617/201808/1090617-20180831222412387-663724253.png)

​    首先是获得均衡器ILoadBalancer这个类上面讲到过这是Netflix Ribbon中的均衡器，这是一个抽象类，具体的实现类是ZoneAwareLoadBalancer上面也讲到过，

每一个微服名对应一个均衡器，均衡器中维护者微服名下所有的服务清单。getLoadBalancer()方法通过serviceId获得对应的均衡器，getServer()方法通过对应的均衡器

在对应的路由的算法下计算得到需要路由到Server，Server中有该服务的具体域名等相关信息。得到了具体的Server后执行正常的Http请求，整个请求的负载均衡逻辑就完成了。

在微服中Ribbon和 Hystrix通常是一起使用的，其实直接使用Ribbon和Hystrix实现服务间的调用并不是很方便，通常在Spring Cloud中我们使用Feign完成服务间的调用，

而Feign是对Ribbon和Hystrix做了进一步的封装方便大家使用，对Ribbon的学习能帮你更好的完成Spring Cloud中服务间的调用。