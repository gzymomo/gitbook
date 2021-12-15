[TOC]

# 什么是Open Feign
有了 Eureka，RestTemplate，Ribbon我们就可以愉快地进行服务间的调用了，但是使用RestTemplate还是不方便，我们每次都要进行这样的调用。
```java
@Autowired
private RestTemplate restTemplate;
// 这里是提供者A的ip地址，但是如果使用了 Eureka 那么就应该是提供者A的名称
private static final String SERVICE_PROVIDER_A = "http://localhost:8081";

@PostMapping("/judge")
public boolean judge(@RequestBody Request request) {
    String url = SERVICE_PROVIDER_A + "/service1";
    // 是不是太麻烦了？？？每次都要 url、请求、返回类型的
    return restTemplate.postForObject(url, request, Boolean.class);
}
```
这样每次都调用RestRemplate的API是否太麻烦，能不能像调用原来代码一样进行各个服务间的调用呢？

想到用映射，就像域名和IP地址的映射。将被调用的服务代码映射到消费者端，这样就可以“无缝开发”。

OpenFeign 也是运行在消费者端的，使用 Ribbon 进行负载均衡，所以 OpenFeign 直接内置了 Ribbon。
在导入了Open Feign之后就可以进行编写 Consumer端代码了。
```java
// 使用 @FeignClient 注解来指定提供者的名字
@FeignClient(value = "eureka-client-provider")
public interface TestClient {
    // 这里一定要注意需要使用的是提供者那端的请求相对路径，这里就相当于映射了
    @RequestMapping(value = "/provider/xxx",
    method = RequestMethod.POST)
    CommonResponse<List<Plan>> getPlans(@RequestBody planGetRequest request);
}
```
然后在Controller就可以像原来调用Service层代码一样调用它了。
```java
@RestController
public class TestController {
    // 这里就相当于原来自动注入的 Service
    @Autowired
    private TestClient testClient;
    // controller 调用 service 层代码
    @RequestMapping(value = "/test", method = RequestMethod.POST)
    public CommonResponse<List<Plan>> get(@RequestBody planGetRequest request) {
        return testClient.getPlans(request);
    }
}
```

# 1、OpenFeign 超时控制
feign客户端调用服务时默认等待1秒钟，如果获取不到服务就会报错。

如果需要增加超时时间，需要配置如下信息：
```yml
# 设置feign客户端超时时间（OpenFeign默认支持Ribbon）
ribbon:
  # 建立连接所用时间，适用于网络正常的情况下，两端连接所用的时间
  ConnectTimeout: 5000
  # 建立连接后从服务器读取到可用资源所用的时间
  ReadTimeout: 5000
```

# 2、OpenFeign日志增强
Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。说白了就是对Feign接口的调用情况进行监控和输出。

📈 日志级别：

- NONE：默认的，不显示任何日志
- BASIC：仅记录请求方法、URL、 响应状态码及执行时间
- HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息
- FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据
🔧 配置方式：

## 2.1 编写一个配置类，设置日志级别
```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```
## 2.2 在配置文件配置如下内容
```yml
logging:
  level:
    # feign以什么级别监控哪个接口
    com.sjl.springcloud.feign.PaymentFeignService: debug
```
再次调用接口，查看控制台即可看见详细的调用信息
![](https://gitee.com/songjilong/FigureBed/raw/master/img/20200423121607.png)