[TOC]

[博客园：东北小狐狸:Spring Cloud（十四）Config 配置中心与客户端的使用与详细](https://www.cnblogs.com/hellxz/p/9306507.html#top)

# 1、创建Config Client
## 1.1 pom.xml

```xml
    <parent>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-parent</artifactId>
        <version>Dalston.SR5</version>
        <relativePath/>
    </parent>

    <dependencies>
        <!--Spring Cloud Config 客户端依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <!-- web的依赖，必须加 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--Spring Boot Actuator，感应服务端变化-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
```

## 1.2 TestController
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * 当有请求/fresh节点的时候，会重新请求一次ConfigServer去拉取最新的配置文件
 * 请求/fresh需要有几点要求：1.加actuator的依赖 2.SpringCloud1.5以上需要设置 management.security.enabled=false
 * 这个Controller的作用是查看from这个key的值
 */
@RestController
@RefreshScope //开启更新功能
@RequestMapping("api")
public class TestController {

    @Value("${from}")
    private String fromValue;

    /**
     * 返回配置文件中的值
     */
    @GetMapping("/from")
    @ResponseBody
    public String returnFormValue(){
        return fromValue;
    }
}
```
加@RefreshScope是为了可以动态刷新这个Controller的Bean

## 1.3 bootstrap.yml
在resources目录下创建bootstrap.yml
```yml
spring:
  application:
    name: hellxztest                     #指定了配置文件的应用名
  cloud:
    config:
      uri: http://localhost:7001/        #Config server的uri
      profile: dev                       #指定的环境
      label: master                      #指定分支
server:
  port: 7002
management:
  security:
    enabled: false     #SpringBoot 1.5.X 以上默认开通了安全认证，如果不关闭会要求权限
```

**注意**：
参数：

1. spring.application.name：对应文件规则的应用名
2. spring.cloud.config.profile：对应环境名
3. spring.cloud.config.label：对应分支名
4. spring.cloud.config.uri：对应Config Server开放的地址

如果要用到动态刷新，SpringBoot 1.5版本以上需要使用management.security.enabled=false