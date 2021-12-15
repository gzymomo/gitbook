[SpringBoot整合Apollo](https://www.cnblogs.com/qdhxhz/p/13449285.html)

这篇文章分为两部分：

1、**跟着官网步骤，快速搭建apollo环境**。

2、**SpringBoot整合apollo,实现配置中心**。

##  一、Apollo快速搭建 

apollo环境的搭建主要参考 [官方文档](https://github.com/nobodyiam/apollo-build-scripts)  ,我们就直接一步一步跟着官方文档来

#### 1、下载Quick Start安装包

下载[apollo-build-scripts](https://github.com/nobodyiam/apollo-build-scripts)项目

#### 2、创建数据库

之前有说过，apollo会有两个数据库: `ApolloPortalDB` 和 `ApolloConfigDB`

创建[ApolloPortalDB](https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloportaldb.sql)

创建[ApolloConfigDB](https://github.com/nobodyiam/apollo-build-scripts/blob/master/sql/apolloconfigdb.sql)

#### 3、更新数据库连接信息

在下载下来的Quick Start项目中，有个叫`demo.sh`的文件，我们需要 **修改数据库连接信息**，修改里头数据库连接地址/用户名/密码

#### 4、 启动Apollo配置中心

上面三部就已经将环境搭建好，现在我们开始启动Apollo配置中心，来看具体效果。

`注意` 这里默认暂用三个端口: `8070`,`8080`,`8090`,所以要先看下这三个端口有木有被暂用。

**启动项目**

```
./demo.sh start
```

#### 5、登录Apollo

`网址`:http://localhost:8070/

**登录界面**

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212210988-465753948.jpg)

`初始账号密码` : **apollo/admin**

进入首页之后，我们可以看到: 默认的环境只有**dev**,m默认有一个**SampleApp**项目

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212448642-1729276800.jpg)

我们在这里添加两个配置

#### 6. 测试客户代码

上面已经设置了两个配置，这个时候我们就需要通过客户端去获取，我们先不通过SpringBoot去获取，而是官方提供的客户端去获取配置

```
运行客户端
./demo.sh client
```

启动后去查询配置

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212544234-1737767734.jpg)

可以看到客户端都能够拿到配置中心的配置。

`注意` 这里有一点在上面我设了 gougou:4岁。而实际客户端获得的是4？，所以中文没有转义成功，在实际开发中，我们也尽量去避免有中文汉字来当配置。

这样下来，官方给的快速启动apollo是成功了，下面我们把客户端换成SpringBoot,来实现获取配置中心的数据。



##  二、SpringBoot整合Apollo  

从上面我们可以知道在apollo官方给了我们一个默认项目叫：**SampleApp**。默认的用户名称：**apollo**。所以这里灵活一点，不用它自己的，而是我们手动新增。

#### 1、Apollo创建新新用户和项目

1）`新增新用户`

```json
http://{portal地址}/user-manage.html  //我们这里设置的是http://localhost:8070/user-manage.html
```

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212631433-995089617.jpg)

这里新增一个用户名叫：`liubei`

2）`新增项目`

访问http://localhost:8070 登录后，选择创建项目。

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212714665-1375105632.jpg)

这里新创建了项目,一个AppId名称叫: `springboot-test-apollo`。

`配置说明`：

**部门**：选择应用所在的部门。（想自定义部门，参照官方文档，这里就选择样例）

**应用AppId**：用来标识应用身份的唯一id，格式为string，需要和客户端。application.properties中配置的app.id对应。

**应用名称**：应用名，仅用于界面展示。

**应用负责人**：选择的人默认会成为该项目的管理员，具备项目权限管理、集群创建、Namespace创建等权限。

提交配置后会出现如下项目配置的管理页面。

3）`添加配置`

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212752731-941570644.jpg)

这里先添加一个新配置: **date.value = 2020.08.04** 。

#### 2、springboot项目搭建

上面把apollo环境搭建好，也创建了新项目，那么这里就去读取新项目的配置信息。

1）`pom.xml`

添加 Apollo 客户端的依赖,其它依赖这里就不展示了，完整项目会放在github上。

```xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.3.0</version>
</dependency>
```

2）`application.yml`

```
server:
  port: 8123

app:
  id: springboot-test-apollo
apollo:
  meta: http://127.0.0.1:8080
  bootstrap:
    enabled: true
    eagerLoad:
      enabled: true

#这里说明在将该项目 com目录下的日志，都采用info模式输出
logging:
  level:
    com: info
配置说明
```

**app.id** ：AppId是应用的身份信息，是配置中心获取配置的一个重要信息。

**apollo.bootstrap.enabled**：在应用启动阶段，向Spring容器注入被托管的application.properties文件的配置信息。

**apollo.bootstrap.eagerLoad.enabled**：将Apollo配置加载提到初始化日志系统之前。

**logging.level.com** ：调整 com 包的 log 级别，为了后面演示在配置中心动态配置日志级别。

3） `TestController`

```java
@Slf4j
@RestController
public class TestController {

    @Value( "${date.value}" )
    String dateValue;

    @GetMapping("test")
    public String test() {
        return "打印配置中心的 dateValue 值: "+ dateValue;
    }

}
```

4）`SpringBoot启动类`

```java
@SpringBootApplication
@EnableApolloConfig
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```

添加 **EnableApolloConfig** 配置注解

#### 3、测试

我在apollo配置了一个属性: date.value = 2020.08.04，如果从springBoot能够获取到该配置，那么就说明成功了。

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212842556-596536111.jpg)

很明显，我们通过接口已经成功获取配置中心的配置。这样一个最简单的整合已经完成了，下面我们在来验证其它东西。



## 三、验证

下面我们来验证一些有意思的东西：

1)、**配置修改后能否立即生效**。

2)、**通过配置日志打印级别,我们来验证能够动态的修改日志级别**

3)、**能够通过修改端口号，能够修改项目运行的端口号**。

下面我们一个一个来验证

#### 1、配置修改后能否立即生效？

新增一个接口

```java
@Slf4j
@RestController
public class TestController {

    @Value( "${date.value}" )
    String dateValue;

    @GetMapping("test1")
    public void test1() {
        log.info("当前配置中心的 dateValue 值 = {}",dateValue);
    }
}
```

`步骤` :1)先请求接口一次。2)然后修改apollo配置中心。3)然后在请求一次该接口。

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806212953674-2083008839.jpg)

`结论` 从图中很明显看出第一次和第二次请求的接口，返回的配置信息已经不一样了。说明配置是会时时更新的。而且当我在apollo界面修改配置的时候，

控制台日志也会输出相关信息。

#### 2、动态修改日志级别

我们来思考有没有这么一种需求，我们在线上的日志级别一般是INFO级别的,主要记录业务操作或者错误的日志。那么这个时候当线上环境出现问题希望输出DEBUG

日志信息辅助排查的时候怎么办呢？

如果以前，我们可以会修改配置文件，重新打包然后上传重启线上环境，以前确实是这么做的。通过Apollo配置中心我们可以实现不用重启项目，就可以实现让日志

基本从INFO变成DEBUG。下面我们来演示一番。

在项目启动之前，我在`application.yml`配置了日志级别是**info模式**。我们来请求下面接口验证下。

```java
 @GetMapping("test2")
    public void test2() {
      log.debug("我是 debug 打印出的日志");
      log.info("我是 info 打印出的日志");
      log.error("我是 error 打印出的日志");
    }
    
```

请求下接口看下控制台

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806213045076-333419926.jpg)

可以很明显的看出，控制台只打印info级别以上的日志，并没有打印debug日志。

现在我们要做的是将打印日志级别不在交给`application.yml`而是交给`apollo`来配置。这里其实要做两步:

1)、**在apollo配置一条日志数据**。

2)、**通过监听配置的变化，来达到热更新的效果**。

```
java配置类
@Configuration
public class LoggerConfig {

    private static final Logger logger = LoggerFactory.getLogger(LoggerConfig.class);
    private static final String LOGGER_TAG = "logging.level.";

    @Autowired
    private LoggingSystem loggingSystem;

    @ApolloConfig
    private Config config;

    @ApolloConfigChangeListener
    private void configChangeListter(ConfigChangeEvent changeEvent) {
        refreshLoggingLevels();
    }

    @PostConstruct
    private void refreshLoggingLevels() {
        Set<String> keyNames = config.getPropertyNames();
        for (String key : keyNames) {
            if (StringUtils.containsIgnoreCase(key, LOGGER_TAG)) {
                String strLevel = config.getProperty(key, "info");
                LogLevel level = LogLevel.valueOf(strLevel.toUpperCase());
                loggingSystem.setLogLevel(key.replace(LOGGER_TAG, ""), level);
                logger.info("{}:{}", key, strLevel);
            }
        }
    }

}
关键点讲解
```

**@ApolloConfig注解**：将Apollo服务端的中的配置注入这个类中。

**@ApolloConfigChangeListener注解**：监听配置中心配置的更新事件，若该事件发生，则调用refreshLoggingLevels方法，处理该事件。

**ConfigChangeEvent参数**：可以获取被修改配置项的key集合，以及被修改配置项的新值、旧值和修改类型等信息。

从上面可以看出,通过`@PostConstruct` 项目启动的时候就去获取apollo的日志级别去覆盖`application.yml`日志级别。我在apollo配置中心，新增加一条日志配置。

把日志级别设置**error**,那么在项目启动的时候,因为**PostConstruct**注解的原因，所以会去执行一次refreshLoggingLevels方法，把当前日志级别改成**error**。

```
apollo配置
```

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806213146988-383237169.jpg)

我们重启启动项目再来看下，请求上面接口结果。

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806213238281-1222573864.jpg)

从上图我们就可以看出，我们同时在`application.yml` 和`apollo`都配置了日志打印级别，但实际上打印出来的结果来看，最终是以apollo为准。

现在我们再来尝试下，我们修改apollo配置，把日志级别修改成debug级别。

```
apollo配置
```

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806213311509-605430673.jpg)

因为我们在LoggerConfig中，有个 `@ApolloConfigChangeListener`注解，我们在修改配置的时候都会被监听到，监听到之后就会把当前的日志级别，换成最新的。

我们再来请求接口，看下控制台。

![img](https://img2020.cnblogs.com/blog/1090617/202008/1090617-20200806213343302-300223462.jpg)

`总结` 通过上面的验证，可以得出：我在没有重启服务器的情况下，实现了日志级别动态切换的功能。

#### 3、动态修改端口号

这里我就不把验证的过程发出来了，结论就是不可以。如果要验证，在上面的refreshLoggingLevels方法中，加入修改端口的逻辑。

其实理由很简单，因为端口跟进程绑定在一起的，你的进程起来了，端口就无法改变。如果一定要换端口号，那么就需要有一个独立进程，去杀掉你的项目进程，

再起一个新的不同端口的进程。显然apollo没有这样的功能，而且个人觉得也没啥意义。。

好了，整篇文章到这里就结束了，下面把该项目的具体代码放到github上。

`GitHub地址` :[SpringBoot整合Apollo](https://github.com/yudiandemingzi/spring-boot-study)



### 参考

1、[官方文档](https://github.com/nobodyiam/apollo-build-scripts)

2、[SpringBoot 整合 apollo](https://www.jianshu.com/p/3c21c18afdc1)