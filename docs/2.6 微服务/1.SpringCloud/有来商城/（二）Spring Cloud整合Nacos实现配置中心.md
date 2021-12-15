- [Spring Cloud整合Nacos实现配置中心](https://www.cnblogs.com/haoxianrui/p/13585125.html)



## **前言**

随着eureka的停止更新，如果同时实现注册中心和配置中心需要SpringCloud Eureka和SpringCloud  Config两个组件;配置修改刷新时需要SpringCloud  Bus消息总线发出消息通知(Kafka、RabbitMQ等)到各个服务完成配置动态更新，否者只有重启各个微服务实例,但是nacos可以同时实现注册和配置中心，以及配置的动态更新。

## **版本声明**

Nacos Server: 1.3.2

SpringBoot: 2.3.0.RELEASE

SpringCloud: Hoxton.SR5

SpringCloud Alibaba: 2.2.1.RELEASE

## **项目实战**

**1.youlai-auth添加nacos-config依赖**



```
<!-- nacos-config 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**2.项目配置文件bootstrap.yml指定nacos配置文件名**

[![img](https://i.loli.net/2020/08/30/R6WE8DSk4sguUIh.png)](https://i.loli.net/2020/08/30/R6WE8DSk4sguUIh.png)



```
spring:
  application:
    name: youlai-auth
  cloud:
    nacos:
      discovery:
        server-addr: http://localhost:8848
      config:
        file-extension: yaml  # 必须修改成yaml
        group: DEFAULT_GROUP  # 缺省即可
        prefix: ${spring.application.name} # 缺省即可
rsa:
  publicKey: 123456
```

注意这里使用bootstrap.yml而非application.yml，避免applicaton.yml后加载于nacos配置并覆盖

SpringBoot读取配置文件顺序：bootstrap.yml>bootstrap.yaml>bootstrap.properties>nacos的配置>application.yml>application.yaml>application.properties

**3.添加接口读取配置信息并添加动态刷新配置的注解@RefreshScope**



```
@RefreshScope
@RestController
@RequestMapping("/oauth")
public class AuthController {

    @Value("${rsa.publicKey}")
    public String publicKey;

    @GetMapping("/publicKey")
    public Result getPublicKey(){
        return Result.success(this.publicKey);
    }
}
```

**4.打开nacos管理控制台添加配置**



```
DATA-ID :  ${prefix}-${spring.profiles.active}.${file-extension} 
```

[![img](https://i.loli.net/2020/08/30/6AM4xtFf7N8VBXC.png)](https://i.loli.net/2020/08/30/6AM4xtFf7N8VBXC.png)



```
a). prefix 默认spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置

b). file-extension默认properties，比如我这里使用的是yaml，那么更改spring.cloud.nacos.config.file-extension= yaml

c). Group默认DEFAULT_GROUP，也可以通过配置项 spring.cloud.nacos.config.group来配置
```

**5.启动服务后第一次读取配置信息**

[![img](https://i.loli.net/2020/08/30/cvAITCy5G9ewpna.png)](https://i.loli.net/2020/08/30/cvAITCy5G9ewpna.png)

**6.再次请求接口获取配置信息**

[![img](https://i.loli.net/2020/08/30/WcAey9w8ido5jlv.png)](https://i.loli.net/2020/08/30/WcAey9w8ido5jlv.png)

可以看到通过接口第二次获取配置信息已变更，完成配置信息的动态刷新