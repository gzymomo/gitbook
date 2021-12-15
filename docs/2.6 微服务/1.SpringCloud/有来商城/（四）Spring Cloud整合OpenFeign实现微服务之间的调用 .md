[Spring Cloud整合OpenFeign实现微服务之间的调用](https://www.cnblogs.com/haoxianrui/p/13615592.html)



## **一. 前言**

微服务实战系列是基于开源微服务项目有来商城微服务框架升级为背景来开展的,本篇则是讲述SpringCloud整合OpenFeign实现微服务之间的相互调用，有兴趣的朋友可以给[youlai-mall](https://github.com/hxrui/youlai-mall) 个star,非常感谢。

## **二. 什么是OpenFeign？**

**想知道什么是OpenFeign,首先要知道何为Feign？**

Feign是SpringCloud组件中一个轻量级RESTFul的HTTP客户端。

Feign内置了Ribbon实现客户端请求的负载均衡。但是Feign是不支持Spring MVC注解的，所以便有了OpenFeign，OpenFeign在Feign的基础上支持Spring MVC注解比如 @RequestMapping等。

OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，通过动态代理生成实现类，实现类做负载均衡并调用其他服务。

## **三. 项目信息**

**[有来商城youlai-mall](https://github.com/hxrui/youlai-mall) 项目结构图：**

[![img](https://i.loli.net/2020/09/04/QlwEB6yNYKMimta.png)](https://i.loli.net/2020/09/04/QlwEB6yNYKMimta.png)

现在要实现这么个需求，**认证中心youlai-auth登录认证时需要调用youlai-admin接口**，这个接口在youlai-admin的请求路径是/users/loadUserByUsername。因为牵涉到微服务之间的调用，所以需要引入HTTP客户端，也就是本篇所说的OpenFeign。

其中youlai-admin-api模块作为youlai-admin模块对外提供FeignClient给其他微服务引用，比如此次的youlai-auth,这样做的好处是无需在youlai-auth去写有关于youlai-admin的FeignClient，直接引入youlai-admin-api即可。而把youlai-admin的FeignClient编写交给负责youlai-admin模块的开发人员，就是让更熟悉此模块的人编写其对外开放的FeignClient。

**本篇设计的项目模块如下：**

| 工程名       | 端口 | 描述               |
| ------------ | ---- | ------------------ |
| nacos-server | 8848 | 注册中心和配置中心 |
| youlai-auth  | 8000 | 认证中心           |
| youlai-admin | 8080 | 平台服务           |

**版本声明：**



```
Nacos Server: 1.3.2

SpringBoot: 2.3.0.RELEASE

SpringCloud: Hoxton.SR5

SpringCloud Alibaba: 2.2.1.RELEASE 
```

## **四. 项目实战**

**1.youlai-admin**

提供接口/users/loadUserByUsername，完整代码下载地址[有来商城youlai-mall](https://github.com/hxrui/youlai-mall)

[![img](https://i.loli.net/2020/09/03/Q6AhpcumiNUOxGK.png)](https://i.loli.net/2020/09/03/Q6AhpcumiNUOxGK.png)

**2.youlai-admin-api**

添加OpenFeign、OkHttp依赖



```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

接口FeignClient代码



```
@FeignClient("youlai-admin")
public interface UmsAdminService {
    @GetMapping("/users/loadUserByUsername")
    UserDTO loadUserByUsername(@RequestParam String username);
}
```

**3.youlai-auth**

添加youlai-admin-api依赖



```
<dependency>
    <groupId>com.youlai</groupId>
    <artifactId>youlai-admin-api</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

配置文件开启OpenFeign使用OkHttp作为底层的client



```
feign:
  okhttp:
    enabled: true
```

远程调用代码

[![img](https://i.loli.net/2020/09/04/bUyn5aBf1j96mX4.png)](https://i.loli.net/2020/09/04/bUyn5aBf1j96mX4.png)



```
@Autowired
private UmsAdminService umsAdminService;

@GetMapping("/loadUserByUsername")
public Result loadUserByUsername(){
    UserDTO userDTO = umsAdminService.loadUserByUsername("admin");
    return Result.success(userDTO);
}
```

**4.微服务调用测试**

依次启动项目nacos-server,youlai-auth,youlai-admin，使用接口测试工具测试接口http://localhost:8000/oauth/loadUserByUsername

**5.OpenFeign底层httpclient选择：HttpURLConnection、feign-httpclient、feign-okhttp？**

HttpURLConnection是JDK默认的，出于性能考虑一般是不可取的。至于其他支持的HC选择，来一波测试数据吧

添加依赖，公平起见引入都是最新版本feign-okhttp和feign-httpclient



```
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>11.0</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>11.0</version>
</dependency>
```

配置信息，两个都为false则默认使用的是HttpURLConnection，开启一方另一方选择关闭。



```
feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: false
```

修改配置后重启youlai-auth进行测试，我这里是单次单次的请求测试，没有模拟高并发的环境去测试。（单位：ms）

| 次数 | HttpURLConnection | feign-httpclient | feign-okhttp |
| ---- | ----------------- | ---------------- | ------------ |
| 1    | 17.79             | 18.97            | 16.39        |
| 2    | 18.02             | 17.45            | 16.96        |
| 3    | 16.67             | 16.25            | 16.27        |
| 4    | 16.65             | 17.28            | 14.79        |
| 5    | 23.03             | 17.62            | 15.06        |
| 6    | 16.37             | 16.80            | 15.14        |
| 7    | 17.01             | 18.51            | 15.71        |
| 8    | 16.15             | 17.12            | 14.93        |
| 9    | 16.86             | 16.79            | 15.76        |
| 10   | 16.28             | 17.26            | 15.05        |

由数据可大概了解到在单次请求测试下，HttpURLConnection和feign-httpclient相差无几，但是feign-okhttp却有着相较于其他两者有着些许的性能优势。所以我最后选择了feign-okhttp，这里只是给大家做个参照。

## **五. 结语**

至此SpringCloud整合OpenFeign实现微服务之间的相互调用已经完成。还有至于OpenFeign为什么选择使用OkHttp作为底层的client给大家做个测试参考。熟悉如何使用OpenFeign去完成微服务之间的调用在后续的工作中是必要的。

源码地址：[youlai-mall](https://github.com/hxrui/youlai-mall)