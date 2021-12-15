[TOC]

Zuul作为网关的其中一个重要功能，就是实现请求的鉴权,通过Zuul提供的过滤器来实现的。

Zuul 也是 Netflix OSS 中的一员，是一个基于 JVM 路由和服务端的负载均衡器，支持动态路由、监控、弹性和安全等特性。

在 1.x 版本中使用的是 Zuul 网关，但是到了 2.x，由于Zuul的升级不断跳票，Spring Cloud 自己研发了一套网关组件：Spring Cloud Gateway。



Spring Cloud 会创建一个嵌入式 Zuul 代理来简化一个常见用例的开发，比如用户程序可能会对一个或多个后端服务进行调用，引入 Zuul 网关能有效避免为所有后端独立管理CORS和身份验证问题的需求

Zuul的使用了一系列的过滤器，这些过滤器可以完成以下功能：

- 身份验证和安全性
  识别每个资源的身份验证要求，并拒绝不满足这些要求的请求。
- 审查与监控
  跟踪有意义的数据和统计数据，以便给我们一个准确的生产视图。
- 动态路由
  根据需要将请求动态路由到不同的后端集群。
- 压力测试
  逐渐增加集群的流量，以评估性能。
- 负载消减
  为每种类型的请求分配容量，并丢弃超出限制的请求。
- 静态响应处理
  直接在边缘构建一些响应，而不是将它们转发到内部集群Zuul 的使用

# 1、过滤器方法的作用
想要使用Zuul实现过滤功能，我们需要自定义一个类继承ZuulFilter类，并实现其中的四个方法，我们先看一下这四个方法的作用是什么。
```java
public class MyFilter extends ZuulFilter {
    /**
     * filterType：返回字符串，代表过滤器的类型。包含以下4种：
     * -- pre：请求在被路由之前执行
     * -- route：在路由请求时调用
     * -- post：在route和errror过滤器之后调用
     * -- error：处理请求时发生错误调用
     * @return 返回以上四个类型的名称
     */
    @Override
    public String filterType() {
        return null;
    }

    /**
     * filterOrder：通过返回的int值来定义过滤器的执行顺序，数字越小优先级越高。
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * shouldFilter：返回一个Boolean值，判断该过滤器是否需要执行。返回true执行，返回false不执行。
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return false;
    }

    /**
     * run：编写过滤器的具体业务逻辑。
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        return null;
    }
}
```

# 2、自定义过滤器
```java
@Component
public class LoginFilter extends ZuulFilter {

    //过滤类型 pre route post error
    @Override
    public String filterType() {
        return "pre";
    }

    //过滤优先级，数字越小优先级越高
    @Override
    public int filterOrder() {
        return 10;
    }

    //是否执行run方法
    @Override
    public boolean shouldFilter() {
        return true;
    }

    //过滤逻辑代码
    @Override
    public Object run() throws ZuulException {
        //获取zuul提供的上下文对象
        RequestContext context = RequestContext.getCurrentContext();
        //获取request对象
        HttpServletRequest request = context.getRequest();
        //获取请求参数
        String token = request.getParameter("username");
        //判断
        if (StringUtils.isBlank(username)){
            //过滤该请求，不对其进行路由
            context.setSendZuulResponse(false);
            //设置响应码401
            context.setResponseStatusCode(HttpStatus.SC_UNAUTHORIZED);
            //设置响应体
            context.setResponseBody("request error....");
        }
        // 校验通过，把登陆信息放入上下文信息，继续向后执行
        context.set("username",username);
        return null;
    }
}
```
没添加过滤功能之前是这样的 ↓，无论加不加username都可以得到数据:
![](https://oscimg.oschina.net/oscnet/up-1a5651610b01b9c7575679627b5d0536676.png)

![](https://oscimg.oschina.net/oscnet/up-0af5e106640953ef97453a51af59f8a751b.png)

添加了过滤功能之后是这样的 ↓，只有加了username才能访问
![](https://oscimg.oschina.net/oscnet/up-2ff2102e6e05b3e0ed284427a03efb51d15.png)

![](https://oscimg.oschina.net/oscnet/up-babab84f1ea7f7ec9207f5c4332e6071831.png)

# 3、过滤器执行的声明周期
![](https://oscimg.oschina.net/oscnet/up-61f6470431d6b3957068fce575e77ea39ac.png)

正常流程：

- 请求到达首先会经过pre类型过滤器，而后到达route类型，进行路由，请求就到达真正的服务提供者，执行请求，返回结果后，会到达post过滤器。而后返回响应。

异常流程：

- 整个过程中，pre或者route过滤器出现异常，都会直接进入error过滤器，在error处理完毕后，会将请求交给POST过滤器，最后返回给用户。
- 如果是error过滤器自己出现异常，最终也会进入POST过滤器，将最终结果返回给请求客户端。
- 如果是POST过滤器出现异常，会跳转到error过滤器，但是与pre和route不同的是，请求不会再到达POST过滤器了。

# 4、实战Zuul

- 引入zuul依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

- 启动类上加入注解`@EnableZuulProxy`
- 引入Eureka注册中心，并注册上去
- 现在不需要额外配置就可以启动了，启动之后你会看到默认的服务映射相关日志：

```java
Mapped URL path [/eureka-provider-temp/**] onto handler of type [class org.springframework.cloud.netflix.zuul.web.ZuulController]
Mapped URL path [/eureka-server/**] onto handler of type [class org.springframework.cloud.netflix.zuul.web.ZuulController]
Mapped URL path [/eureka-provider/**] onto handler of type [class org.springframework.cloud.netflix.zuul.web.ZuulController]
Mapped URL path [/ribbon-client/**] onto handler of type [class org.springframework.cloud.netflix.zuul.web.ZuulController]
Mapped URL path [/eureka-client/**] onto handler of type [class org.springframework.cloud.netflix.zuul.web.ZuulController]
```

- 然后就可以通过zuul网关来访问后端服务了



## 4.1 管理站点

Zuul 默认依赖了 actuator，并且会暴露`/actuator/routes`和`/actuator/filters` 两个端点，访问这两个断点，可以很直观的查看到路由信息，在查看之前需要添加以下配置：



```
# 应该包含的端点ID，全部:*
management.endpoints.web.exposure.include: *
```

访问http://127.0.0.1:8000/actuator可以查看所有端点信息，访问http://127.0.0.1:8000/actuator/routes 可查看到路由信息：

[![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208101840979-2146211634.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208101840979-2146211634.png)

## 4.2 路由配置

### 路由配置

**为网关添加前缀**



```
# 访问网关的时候必须要加的路径前缀
zuul.prefix = /api
```

添加以上配置后，访问网关时路径必须是/api/**，然后才会正确的路由到后端对应的服务

如果在转发请求到服务的时候要去掉这个前缀，可以设置strip-prefix= false来忽略



```
# 请求转发前是否要删除 zuul.prefix 设置的前缀 ，true:转发前要带上前缀(默认值)，fasle:不带上前缀
zuul.routes.ecs.strip-prefix = true
```

**配置路由**



```
# 忽略注册中心 eureka-server，*：会忽略所有的服务
zuul.ignored-services = eureka-server,eureka-client
# eureka-client 服务映射规则，http://127.0.0.1:8000/ec/sayHello
zuul.routes.eureka-client = /ec/**
```

上面的配置会忽略eureka-server和eureka-client，访问`http://127.0.0.1:8000/api/ec/**`的请求的都会被路由到eureka-client，如果没有忽略eureka-client，则访问`/eureka-client/**` 和`/ec/**` 都会路由到eureka-client服务。

注意`/ec/*`只会匹配一个层级，`/ec/**` 会匹配多个层级。

[![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208102025066-219458671.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208102025066-219458671.png)

**指定服务id和path**



```
# 指定service-id和path
zuul.routes.rcs.service-id = ribbon-client
zuul.routes.rcs.path = /rc/**
```

然后访问[http://127.0.0.1:8000/api/rc/queryPort](http://127.0.0.1:8000/rc/queryPort)接口就会被路由到`ribbon-client`服务

**路由配置顺序**

如果想按照配置的顺序进行路由规则控制，则需要使用YAML，如果是使用propeties文件，则会丢失顺序。例如:



```
zuul:
  routes:
    users:
      path: /myusers/**
    legacy:
      path: /**
```

使用propeties文件，旧的路径可能出现在用户路径的前面，从而导致用户路径无法访问。

**关闭重试**

可以通过将zuul.retryable设置为false来关闭Zuul的重试功能，默认值也是false。



```
zuul.retryable=false
```

还可以通过将zuul.routes.routename.retryable设置为false来禁用逐个路由的重试功能



```
# 关闭指定路由的重试
zuul.routes.ecs.retryable = false
```

### 忽略服务路由

添加以下配置会忽略指定的服务，很明显注册中心一般是不需要通过网关来访问的，所以需要忽略它



```
# 忽略注册中心 eureka-server，*：会忽略所有的服务
zuul.ignored-services = eureka-server
```

也可以通过`zuul.ignoredPatterns` 来配置你不想暴露出去的API

## 4.3 隔离策略

### 设置信号量



```
# 改为信号量隔离
zuul.ribbon-isolation-strategy=semaphore
# Hystrix的最大总信号量
zuul.semaphore.max-semaphores=1000
# 单个路由可以使用的最大连接数
zuul.host.max-per-route-connections=500
```

最大总信号量默认是100，单个路由最大的连接数默认是20，有时候并发量上不去可能就是使用的默认配置。

### 设置独立的线程池

Zuul 中默认采用信号量隔离机制，如果想要换成线程，需要配置 `zuul.ribbon-isolation-strategy=THREAD`，配置后所有的路由对应的 Command 都在一个线程池中执行，这样其实达不到隔离的效果，所以我们需要增加一个 zuul.thread-pool.use-separate-thread-pools 的配置，让每个路由都使用独立的线程池，zuul.thread-pool.thread-pool-key-prefix 可以为线程池配置对应的前缀，方便调试。



```
## 线程隔离
#zuul.ribbon-isolation-strategy=THREAD
## 每个路由使用独立的线程池
#zuul.thread-pool.use-separate-thread-pools=true
## 线程池前缀
#zuul.thread-pool.thread-pool-key-prefix=zuul-pool-
```

## 4.4 其他配置

### 更换Http客户端

Zuul默认使用的是 Apache HTTP Client，需要更换的话只需要设置对应的属性即可



```
# Ribbon RestClient
ribbon.restclient.enabled=true
#  or okhttp
ribbon.okhttp.enabled=true
```

### Cookie和请求头

Zuul 提供了一个敏感头属性配置，设置了该属性后，Zuul 就不会把相关的请求头转发到下游的服务，比如：



```
# 请求头里面的字段不会带到eureka-client服务
zuul.routes.ecs.sensitive-headers = jinglingwang
```

sensitiveHeaders 的默认值是Cookie、Set-Cookie、Authorization，如果把该值配置成空值，则会把所有的头都传递到下游服务。

还可以通过设置zuul.sensitiveHeaders来设置全局的敏感标头。 如果在路由上设置了sensitiveHeaders，它将覆盖全局的sensitiveHeaders设置

### 忽略请求头

除了对路由敏感的标头单独设置之外，还可以设置一个名为zuul.ignoredHeaders的全局值，比如：



```
# 该配置的Header也不会转发到下游服务
zuul.ignored-headers=jinglingwang
```

在默认情况下是没有这个配置的，如果项目中引入了Spring Security，那么Spring Security会自动加上这个配置，默认值为: Pragma,Cache-Control,X-Frame-Options,X-Content-Type-Options,X-XSS-Protection,Expries。

下游服务需要使用Spring Security的Header时，可以增加`zuul.ignoreSecurityHeaders=false`的配置

### 文件上传

通过Zuul网关上传文件时，只要文件不大，都可以正常的上传，对于大文件，Zuul有一个替代路径（`/zuul/*`）可以绕过Spring DispatcherServlet，比如你的文件服务（file-service）路由配置是`zuul.routes.file-service=/file/**`，然后你post提交文件到`/zuul/file/**` 即可。

还有一种办法就是直接修改可上传文件大小的配置：



```
# 文件最大值。值可以使用后缀“ MB”或“ KB”分别表示兆字节或千字节
spring.servlet.multipart.max-file-size=10MB
# 最大请求大小
spring.servlet.multipart.max-request-size=30MB
```

两种办法都需要在文件服务里面添加以上的配置

在上传大文件时也需要设置合理的超时时间：



```
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 60000
ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
```

### 自定义过滤器

过滤器是 Zuul 中的核心内容，很多高级的扩展都需要自定义过滤器来实现，在 Zuul 中自定义一个过滤器只需要继承 ZuulFilter，然后重写 ZuulFilter 的四个方法即可：



```
@Component
public class LogFilter extends ZuulFilter{
    /**
     * 返回过滤器的类型，可选值有 pre、route、post、error 四种类型
     * @return
     */
    @Override
    public String filterType(){
        return "pre";
    }

    /**
     * 指定过滤器的执行顺序，数字越小，优先级越高
     * 默认的filter的顺序可以在FilterConstants类中查看。
     * @return
     */
    @Override
    public int filterOrder(){
        // pre filter
        return PRE_DECORATION_FILTER_ORDER - 1 ;
        // ROUTE filter
        //return SIMPLE_HOST_ROUTING_FILTER_ORDER - 1 ;
        // POST filter
        //return SEND_RESPONSE_FILTER_ORDER - 1 ;
    }

    /**
     * 决定了是否执行该过滤器，true 为执行，false 为不执行
     * @return
     */
    @Override
    public boolean shouldFilter(){
        return true;
    }

    /**
     * 如果shouldFilter（）为true，则将调用此方法。该方法是ZuulFilter的核心方法
     * @return 返回值会被忽略
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException{
        HttpServletRequest req = (HttpServletRequest) RequestContext.getCurrentContext().getRequest();
        System.out.println("ZUUL REQUEST:: " + req.getScheme() + " " + req.getRemoteAddr() + ":" + req.getRemotePort() + " uri::"+ req.getRequestURI()) ;
        return null;
    }
}
```

### 禁用过滤器

Zuul 默认提供了很多过滤器（ZuulFilter），有关可启用的过滤器列表，可以参考Zuul 过滤器的包（netflix.zuul.filters）。如果要禁用一个过滤器，可以按照`zuul.<SimpleClassName>.<filterType>.disable=true` 格式来进行设置，比如：



```
zuul.SendResponseFilter.post.disable=true
```

### 跨域支持

如果是外部网页应用需要调用网关的 API，不在同一个域名下则会存在跨域的问题，想让Zuul处理这些跨域的请求，可以通过提供自定义WebMvcConfigurer bean来完成：



```
@Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            /**
             * 配置跨源请求处理
             * @param registry
             */
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/path/**")
                        .allowedOrigins("https://jinglingwang.cn")
                        .allowedMethods("GET", "POST");
            }
        };
    }
```

上面的示例中，允许`jinglingwang.cn`的`GET`和`POST`方法将跨域请求发送到 /path/**开头的端点

### Zuul 超时

有两种情况：

1. 如果Zuul使用服务发现，则需要配置Ribbon的属性配置超时

   

   ```
   ribbon.ReadTimeout
   ribbon.SocketTimeout
   ```

2. 如果通过指定URL配置了Zuul路由

   

   ```
   # 套接字超时（以毫秒为单位）。默认为10000
   zuul.host.socket-timeout-millis=15000
   # 连接超时（以毫秒为单位）。默认为2000
   zuul.host.connect-timeout-millis=3000
   ```

### 服务容错与回退

Spring Cloud 中，Zuul 默认整合了 Hystrix，当Zuul中给定路由的电路跳闸时，可以通过创建FallbackProvider类型的bean提供回退响应。配置示例代码如下：



```
@Component
public class EurekaClientFallbackProvider implements FallbackProvider{
    @Override
    public String getRoute(){
        // 路由的server-id，* or null：为所有的路由都配置回退
        return "eureka-client";
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route,Throwable cause){
        if (cause instanceof HystrixTimeoutException) {
            return response(HttpStatus.GATEWAY_TIMEOUT);
        } else {
            return response(HttpStatus.INTERNAL_SERVER_ERROR);
        }

    }

    private ClientHttpResponse response(HttpStatus status){
        return new ClientHttpResponse(){
            @Override
            public HttpStatus getStatusCode() throws IOException{
                return status;
            }

            @Override
            public int getRawStatusCode() throws IOException{
                return status.value();
            }

            @Override
            public String getStatusText() throws IOException{
                return status.getReasonPhrase();
            }

            @Override
            public void close(){
            }

            @Override
            public InputStream getBody() throws IOException{
                return new ByteArrayInputStream("eureka-client 服务暂不可用，jinglingwang请你稍后重试!".getBytes());
            }

            @Override
            public HttpHeaders getHeaders(){
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
                return headers;
            }
        };
    }
}
```

重启后，运行效果如下：
[![img](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208102041092-674465564.png)](https://img2020.cnblogs.com/blog/709068/202012/709068-20201208102041092-674465564.png)

如果要为所有路由提供默认回退，getRoute方法返回*或null即可。

### Ribbon client延迟加载

Zuul内部使用Ribbon来调用远程URL。 默认情况下，Ribbon 客户端在第一次调用时由Spring Cloud进行延迟加载。可以通过以下配置来开启启动时立即加载：



```
zuul.ribbon.eager-load.enabled=true
```

### @EnableZuulProxy vs @EnableZuulServer

Spring Cloud Netflix安装了很多过滤器，具体取决于用于启用Zuul的注解。 @EnableZuulProxy是@EnableZuulServer的超集。换句话说，@ EnableZuulProxy包含@EnableZuulServer安装的所有过滤器。 “proxy”中的其他过滤器启用路由功能。 如果需要一个“空白”的 Zuul，则应使用@EnableZuulServer。