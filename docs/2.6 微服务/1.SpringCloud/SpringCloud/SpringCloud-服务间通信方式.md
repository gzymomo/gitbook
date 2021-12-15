[TOC]

# SpringCloud应用间通信
SpringCloud中服务间两种restful调用方式

- RestTemplate
- Feign

>  在springcloud中服务间调用方式主要是使用 http restful方式进行服务间调用





# 1.RestTemplate的三种使用方式
```java
@RestController
@Slf4j
public class clientController{

   @GetMapping("/getProductMsg")
   public String getProductMsg(){
     //1：第一种方式
	 RestTemplate restTemplage = new RestTemplate();
	 String response = restTemplate.getForObject("http://localhost:8080/msg",String.class);
	 log.info("response={}",response);
	 return response;
   }


   @Autowird
   private LoadBalancerClient loadBalancerClient;

   @GetMapping("/getProductMsg2")
   public String getProductMsg2(){
     //2：第二种方式：注入LoadBalancerClient;
	 ServiceInstance serviceInstance = loadBalancerClient.choose("PRODUCT");
	 String url = String.format("http://%s:%s",serviceInstance.getHost,serviceInstance/getPort());
	 RestTemplate restTemplage = new RestTemplate();
	 String response = restTemplate.getForObject(url,String.class);
	 log.info("response={}",response);
	 return response;
   }

    @Autowired
	private RestTemplate restTemplate;

   @GetMapping("/getProductMsg3")
   public String getProductMsg3(){
     //3：第三种方式：RestTemplateConfig;
	 String response = restTemplate.getForObject("http://PRODUCT/msg",String.class);
	 log.info("response={}",response);
	 return response;
   }

}
```

```java
@Component
public class RestTemplateConfig{

   @Bean
   @LoadBalanced
   public RestTemplate restTemplate(){
      return new RestTemplate();
   }

}
```

# 2.基于RestTemplate的服务调用

使用的是consul注册，pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.md</groupId>
    <artifactId>04-products9998</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>04-products9998</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--引入consul依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>

        <!-- 这个包是用做健康度监控的-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>



    </dependencies>

    <!--全局管理springcloud版本,并不会引入具体依赖-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

## 2.1 说明

- Spring框架提供的RestTemplate类可用于在应用中调用rest服务，它简化了与http服务的通信方式，统一了RESTful的标准，封装了http链接， 我们只需要传入url及返回值类型即可。相较于之前常用的HttpClient，RestTemplate是一种更优雅的调用RESTful服务的方式。



## 2.2 RestTmplate服务调用

1. 创建两个服务并注册到consul注册中心中

- users 代表用户服务 端口为 9999
- products 代表商品服务 端口为 9998
  `注意:这里服务仅仅用来测试,没有实际业务意义

[![img](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208215001707-736500113.png)](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208215001707-736500113.png)
[![img](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208215944397-758680704.png)](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208215944397-758680704.png)



## 2.3 在商品服务中提供服务方法

```java
@RestController
@Slf4j
public class ProductController {
    @Value("${server.port}")
    private int port;
    @GetMapping("/product/findAll")
    public Map<String,Object> findAll(){
        log.info("商品服务查询所有调用成功,当前服务端口:[{}]",port);
        Map<String, Object> map = new HashMap<String,Object>();
        map.put("msg","服务调用成功,服务提供端口为: "+port);
        map.put("status",true);
        return map;
    }
}
```

## 2.4 在用户服务中使用restTemplate进行调用

```java
@RestController
@Slf4j
public class UserController {
    @GetMapping("/user/findAll")
    public String findAll(){
        log.info("调用用户服务...");
        //1. 使用restTemplate调用商品服务
        RestTemplate restTemplate = new RestTemplate();
        // get请求getxxx，post请求postxxx
        // 参数1：请求路径，参数2：返回的类型是String的
        String forObject = restTemplate.getForObject("http://localhost:9998/product/findAll", 
                                                     String.class);
        return forObject;
    }
}
```



启动服务



## 2.5 测试服务调用

- 浏览器访问用户服务 http://localhost:9999/user/findAll



## 2.6 总结

- rest Template是直接基于服务地址调用没有在服务注册中心获取服务,也没有办法完成服务的负载均衡如果需要实现服务的负载均衡需要自己书写服务负载均衡策略。
  **restTemplate直接调用存在问题**
- 1.直接使用restTemplate方式调用没有经过服务注册中心获取服务地址,代码写死不利于维护,当服务宕机时不能高效剔除
- 2.调用服务时没有负载均衡需要自己实现负载均衡策略

# 3、基于Ribbon的服务调用

## 3.1 说明

- 官方网址: https://github.com/Netflix/ribbon
- Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。

再创建一个服务类，和上面的服务类一样，只有端口不同

```yaml
server.port=9997
spring.application.name=products
spring.cloud.consul.port=8500
spring.cloud.consul.host=localhost
```

其他一样

```java
@RestController
@Slf4j
public class ProductController {
    @Value("${server.port}")
    private int port;
    @GetMapping("/product/findAll")
    public Map<String,Object> findAll(){
        log.info("商品服务查询所有调用成功,当前服务端口:[{}]",port);
        Map<String, Object> map = new HashMap<String,Object>();
        map.put("msg","服务调用成功,服务提供端口为: "+port);
        map.put("status",true);
        return map;
    }
}
```

## 3.2 Ribbon服务调用

```
# 1.项目中引入依赖
- 说明: 
	1.如果使用的是eureka client 和 consul client,无须引入依赖,因为在eureka,consul中默认集成了ribbon组件
	2.如果使用的client中没有ribbon依赖需要显式引入如下依赖
```

```xml
<!--引入ribbon依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

```
# 2.查看consul client中依赖的ribbon
```

```
# 3.使用restTemplate + ribbon进行服务调用
- 使用discovery client  进行客户端调用
- 使用loadBalanceClient 进行客户端调用
- 使用@loadBalanced     进行客户端调用
```

```
# 3.1 使用discovery Client形式调用
```

```java
@Autowired
private DiscoveryClient discoveryClient;

//获取服务列表，返回全部的，服务id，上面的products
List<ServiceInstance> products = discoveryClient.getInstances("服务ID");
for (ServiceInstance product : products) {
  log.info("服务主机:[{}]",product.getHost());
  log.info("服务端口:[{}]",product.getPort());
  log.info("服务地址:[{}]",product.getUri());
  log.info("====================================");
}
```

```
# 3.2 使用loadBalance Client形式调用
```

```java
@Autowired
private LoadBalancerClient loadBalancerClient;
//根据负载均衡策略选取某一个服务调用，服务id=products
ServiceInstance product = loadBalancerClient.choose("服务ID");//地址  默认轮询策略
log.info("服务主机:[{}]",product.getHost());
log.info("服务端口:[{}]",product.getPort());
log.info("服务地址:[{}]",product.getUri());
```

```
# 3.3 使用@loadBalanced(常用)
```

```java
//1.整合restTemplate + ribbon
@Configuration
public class RestTemplateConfig {

    // 在工厂中创建一个restTemplate对象
    @Bean
    // 加上这个注解代表当前的restTemplate对象带有ribbon负载均衡
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

//2.调用controller服务位置注入RestTemplate
// 此时的restTemplate就具有了负载均衡的功能
@Autowired
private RestTemplate restTemplate;

//3.调用
@RestController
@Slf4j
public class UserController {
    @Autowired
	private RestTemplate restTemplate;
    
    @GetMapping("/user/findAll")
    public String findAll(){
        // 服务id就是在配置文件中的当前服务名products
        String forObject = restTemplate.getForObject("http://服务ID/product/findAll", 
                                                     String.class);
        return forObject;
    }
}
```

默认的轮训策略，也就是当前访问的9998，之后是9997，循环访问

## 3.3 Ribbon负载均衡模式

**ribbon负载均衡算法**

- RoundRobinRule 轮训策略 按顺序循环选择 Server
- RandomRule 随机策略 随机选择 Server
- AvailabilityFilteringRule 可用过滤策略
  `会先过滤由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问
- WeightedResponseTimeRule 响应时间加权策略
  `根据平均响应的时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高，刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够会切换到
- RetryRule 重试策略
  `先按照RoundRobinRule的策略获取服务，如果获取失败则在制定时间内进行重试，获取可用的服务。
- BestAviableRule 最低并发策略
  `会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

[![img](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208220216377-471842662.png)](https://img2020.cnblogs.com/blog/1212924/202012/1212924-20201208220216377-471842662.png)



## 3.4 修改服务的默认负载均衡策略

```
# 1.修改服务默认随机策略
- 服务id.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
	`下面的products为服务的唯一标识
```

```yaml
products.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
```