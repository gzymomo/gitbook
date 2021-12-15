[SpringCloud(9)---mysql实现配置中心](https://www.cnblogs.com/qdhxhz/p/9624386.html)

 本公司配置数据的管理是通过mysql进行配置管理，因为已经搭建好了，所以自己动手重新搭建一遍，熟悉整个流程。有关项目源码后期会补上github地址

微服务要实现集中管理微服务配置、不同环境不同配置、运行期间也可动态调整、配置修改后可以自动更新的需求，Spring Cloud Config同时满足了以上要求。

 **项目代码GitHub地址**：https://github.com/yudiandemingzi/spring-cloud-study

## 一、项目搭建

本次主要用三个微服务

（1）Eureka-server： 7001 注册中心

（2）config-server ： 5001 配置中心

（3）product-server ： 8001 商品微服务

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180910232534543-36360615.png)

#### 1、Eureka-server注册中心

注册中心很简单，这里不在重复些，注册中心没有变化。可以看之前写的博客 : SpringCloud(3)---Eureka服务注册与发现

#### 2、配置中心微服务

  **1、pom.xml**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!--服务中心jar包-->
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!--配置中心jar包-->
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-config-server</artifactId>
</dependency>

<!--连接msql数据库相关jar包-->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.21</version>
</dependency>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

  **2、application.yml**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#服务名称
 server:
   port: 5001

#连接配置信息
 spring:
   application:
     name: config-server-jdbc
   profiles:
     active: jdbc
   cloud:
     config:
       server:
         default-label: dev
         jdbc:
           sql: SELECT akey , avalue FROM config_server where APPLICATION=? and APROFILE=? and LABEL=?
 #####################################################################################################
 # mysql 属性配置
   datasource:
     driver-class-name: com.mysql.jdbc.Driver
     url: jdbc:mysql://127.0.0.1:3306/test
     username: root
     password: root
 #####################################################################################################

#指定注册中心地址
 eureka:
   client:
     serviceUrl:
       defaultZone: http://localhost:7001/eureka/
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里主要讲下连接配置信息

（1）spring.profiles.active=jdbc，自动实现JdbcEnvironmentRepository。

（2）sql语句自定义，否则会默认为“SELECT KEY, VALUE from PROPERTIES where APPLICATION=? and PROFILE=? and LABEL=?”，具体可以参考JdbcEnvironmentRepository实现。

（3）本人数据库建表为config_server，由于key，value和profile是mysql关键字，所以我都在最前面加了a。当然表名字段名都可以自定义。

（4） {application} 对应客户端的"spring.application.name"属性;

​     {aprofile} 对应客户端的 "spring.profiles.active"属性(逗号分隔的列表); 和

​     {label} 对应服务端属性,这个属性能标示一组配置文件的版本.

（5）只要select出来是两个字段，框架会自动包装到environment的map<key,value>。

####     **3、mysql数据**

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180910233438186-1244022037.png)

  **4、springboot启动类**

添加@EnableConfigServer注解

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@SpringBootApplication
@EnableConfigServer
public class ConfigserverApplication {

public static void main(String[] args) {
SpringApplication.run(ConfigserverApplication.class, args);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####  3、product-service微服务

   **1、pom.xml** 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
        <!--服务中心jar-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        
        <!--配置中心客户端jar-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   2、bootstrap.yml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#指定注册中心地址
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7001/eureka/

#服务的名称
spring:
  application:
    name: product-service
  #指定从哪个配置中心读取
  cloud:
    config:
      discovery:
        service-id: config-server-jdbc
        enabled: true
      profile: dev
      label: dev

server:
  port: 8001
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这里为什么用bootstrap.yml而不用application.yml，是因为若application.yml 和bootStrap.yml 在同一目录下，

则bootStrap.yml 的加载顺序要高于application.yml,即bootStrap.yml 会优先被加载。

**为何需要把 config server 的信息放在 bootstrap.yml 里？**

当使用 Spring Cloud 的时候，配置信息一般是从 config server 加载的，为了取得配置信息（比如密码等），你需要一些提早的或引导配置。

因此，把 config server 信息放在 bootstrap.yml，用来加载真正需要的配置信息。

   **3、ConfigController类（测试用）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
@RequestMapping("/api/v1/product")
public class ConfigController {

    @Value("${item_url}")
    private String url;

    /**
     * 输出url
     */
    @RequestMapping("url")
    public void list(){

        System.out.println(url);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

​    **4、测试**

通过访问：http://localhost:8001/api/v1/product/url 进入断点。

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180910234524780-731245319.png)

 