来自：开源中国，作者：哈库纳 	链接：https://my.oschina.net/ta8210/blog/3234639



# 一、Dataway介绍

> Dataway 是基于 DataQL 服务聚合能力，为应用提供的一个接口配置工具。使得使用者无需开发任何代码就配置一个满足需求的接口。整个接口配置、测试、冒烟、发布。一站式都通过 Dataway 提供的 UI 界面完成。UI 会以 Jar 包方式提供并集成到应用中并和应用共享同一个 http 端口，应用无需单独为 Dataway 开辟新的管理端口。
>
> 
>
> 这种内嵌集成方式模式的优点是，可以使得大部分老项目都可以在无侵入的情况下直接应用 Dataway。进而改进老项目的迭代效率，大大减少企业项目研发成本。
>
> 
>
> Dataway 工具化的提供 DataQL 配置能力。这种研发模式的变革使得，相当多的需求开发场景只需要配置即可完成交付。从而避免了从数据存取到前端接口之间的一系列开发任务，例如：Mapper、BO、VO、DO、DAO、Service、Controller 统统不在需要。



Dataway 是 Hasor 生态中的一员，因此在 Spring 中使用 Dataway 首先要做的就是打通两个生态。根据官方文档中推荐的方式我们将 Hasor 和 Spring Boot 整合起来。



# 二、SpringBoot集成Dataway

## 2.1 第一步：引入相关依赖

```xml
<dependency>
    <groupId>net.hasor</groupId>   
    <artifactId>hasor-spring</artifactId>   
    <version>4.1.6</version>
</dependency>

<dependency>    
    <groupId>net.hasor</groupId>    
    <artifactId>hasor-dataway</artifactId>    
    <version>4.1.6</version>
</dependency>
```

hasor-spring 负责 Spring 和 Hasor 框架之间的整合。hasor-dataway 是工作在 Hasor 之上，利用 hasor-spring 我们就可以使用 dataway了。

## 2.2 第二步：配置 Dataway，并初始化数据表

dataway 会提供一个界面让我们配置接口，这一点类似 Swagger 只要jar包集成就可以实现接口配置。找到我们 springboot 项目的配置文件 application.properties。

```yaml
# 是否启用 Dataway 功能（必选：默认false）
HASOR_DATAQL_DATAWAY=true
# 是否开启 Dataway 后台管理界面（必选：默认false）
HASOR_DATAQL_DATAWAY_ADMIN=true
# dataway  API工作路径（可选，默认：/api/）
HASOR_DATAQL_DATAWAY_API_URL=/api/
# dataway-ui 的工作路径（可选，默认：/interface-ui/）
HASOR_DATAQL_DATAWAY_UI_URL=/interface-ui/
# SQL执行器方言设置（可选，建议设置）
HASOR_DATAQL_FX_PAGE_DIALECT=mysql
```

Dataway 一共涉及到 5个可以配置的配置项，但不是所有配置都是必须的。

其中 HASOR_DATAQL_DATAWAY**、**HASOR_DATAQL_DATAWAY_ADMIN 两个配置是必须要打开的，默认情况下 Datawaty 是不启用的。

Dataway 需要两个数据表才能工作，下面是这两个数据表的简表语句。下面这个 SQL 可以在 dataway的依赖 jar 包中 “META-INF/hasor-framework/mysql” 目录下面找到，建表语句是用 mysql 语法写的。

其它数据库的建表语句请参看官方说明手册：

> https://www.hasor.net/web/dataway/for_boot.html#mysql

## 2.3 第三步：配置数据源

作为 Spring Boot 项目有着自己完善的数据库方面工具支持。我们这次采用 druid + mysql + spring-boot-starter-jdbc 的方式。

首先引入依赖

```xml
<dependency>   
    <groupId>mysql</groupId>   
    <artifactId>mysql-connector-java</artifactId>    
    <version>5.1.30</version>
</dependency>

<dependency>    
    <groupId>com.alibaba</groupId>    
    <artifactId>druid</artifactId>    
    <version>1.1.21</version>
</dependency>

<dependency>   
    <groupId>org.springframework.boot</groupId>   
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>    
    <groupId>com.alibaba</groupId>    
    <artifactId>druid-spring-boot-starter</artifactId>   
    <version>1.1.10</version>
</dependency>
```

然后增加数据源的配置：

```yaml
# db
spring.datasource.url=jdbc:mysql://xxxxxxx:3306/example
spring.datasource.username=xxxxx
spring.datasource.password=xxxxx
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type:com.alibaba.druid.pool.DruidDataSource
# druid
spring.datasource.druid.initial-size=3
spring.datasource.druid.min-idle=3
spring.datasource.druid.max-active=10
spring.datasource.druid.max-wait=60000
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
spring.datasource.druid.filter.stat.log-slow-sql=true
spring.datasource.druid.filter.stat.slow-sql-millis=1
```

如果项目已经集成了自己的数据源，那么可以忽略第三步。

## 2.4 第四步：把数据源设置到 Hasor 容器中

Spring Boot 和 Hasor 本是两个独立的容器框架，我们做整合之后为了使用 Dataway 的能力需要把 Spring 中的数据源设置到 Hasor 中。

首先新建一个 Hasor 的 模块，并且将其交给 Spring 管理。然后把数据源通过 Spring 注入进来。



```java
@DimModule
@Component
publicclass ExampleModule implements SpringModule{   
    @Autowired   
    private DataSource  dataSource = null;
    
    @Override    
    public void loadModule(ApiBinder apiBinder) throws Throwable{        
    // .DataSource form Spring boot into Hasor
            apiBinder.installModule(new JdbcModule(Level.Full, this.dataSource));
    }
}
```

Hasor 启动的时候会调用 loadModule 方法，在这里再把 DataSource 设置到 Hasor 中。



## 2.5 第五步：在SprintBoot 中启用 Hasor

```java
@EnableHasor()
@EnableHasorWeb()
@SpringBootApplication(scanBasePackages = { "net.example.hasor"})
public class ExampleApplication{    
	public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```

这一步非常简单，只需要在 Spring 启动类上增加两个注解即可。

# 第六步：启动应用

应用在启动过程中会看到 Hasor Boot 的欢迎信息及日志。

当看到 “dataway api workAt **/api/**” 、 dataway admin workAt **/interface-ui/** 信息时，就可以确定 Dataway 的配置已经生效了。



# 第七步：访问接口管理页面进行接口配置

在浏览器中输入 “http://127.0.0.1:8080/interface-ui/” 就可以看到期待已久的界面了。



![img](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr6wVv9b3Kbgdn4xcIKP7neOk4JuQsiaadQmBBQuBibrdqPmcZJicxMGQyssj5SlQt5OCvnBZn2v5ApRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 第八步：新建一个接口

Dataway 提供了2中语言模式，我们可以使用强大的 DataQL 查询语言，也可以直接使用 SQL 语言（在 Dataway 内部 SQL 语言也会被转换为 DataQL 的形式执行。）


![img](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr6wVv9b3Kbgdn4xcIKP7neONgoEFrVEhlUzVEgyoia0H1ib1MopZPqbt9CWMibGgyibG0mVdT5sQY4AtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先我们在 SQL 模式下尝试执行一条 select 查询，立刻就可以看到这条 SQL 的查询结果。

![img](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr6wVv9b3Kbgdn4xcIKP7neOKibEruC0ociaedbjiadWAzbbIWhoydU69g3z4CpwAJjd8OdhiaX4R6dBeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同样的方式我们使用 DataQL 的方式需要这样写：

```
var query = @@sql()<%    select* from interface_info%>return query()
```

其中 var query = **@@sql()<% ... %>** 是用来定义SQL外部代码块，并将这个定义存入 query 变量名中。<% %> 中间的就是 SQL 语句。

最后在 DataQL 中调用这个代码块，并返回查询结果。

当接口写好之后就可以保存发布了，为了测试方便，我选用 GET 方式。

![img](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr6wVv9b3Kbgdn4xcIKP7neOFhWuA3vym75qmyjhh1gcFg8KDZtSwaZFxvVQDHe2Z6wlA6STs5gCng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

接口发布之后我们直接请求：http://127.0.0.1:8080/api/demos，就看到期待已久的接口返回值了。

![img](https://mmbiz.qpic.cn/mmbiz_png/tO7NEN7wjr6wVv9b3Kbgdn4xcIKP7neOBZGwIfFd5SmXOTgdL8VkJpAAEWv6UHctOpoR6KDeicqCCEiaWmkeplPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)