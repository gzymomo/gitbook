[Spring Boot 第三弹，一文带你了解日志如何配置？](https://www.cnblogs.com/Chenjiabing/p/13749073.html)



## 1、日志框架

故事：有一个开发人员，开发一个大型系统；

> 遇到重要数据，喜欢`System.out.println("")`，将关键数据打印在控制台
>
> 去掉？写在一个文件？方便？
>
> 框架来记录系统的一些运行时信息，日志框架：first.jar
>
> 高大上的几个功能？异步模式？自动规定？等等？：second.jar
>
> 将以前的框架卸下来？换上新的框架，更新修改之前相关API：third.jar
>
> JDBC---数据库驱动：
>
> - 写了一个统一的接口层：暂时叫做日志门面（日志的一个抽象层）：fourth.jar
> - 给项目中导入具体的日志实现就行了，我们之前的日志框架都是实现的抽象层

### 日志级别

几种常见的日志级别由低到高分为：`TRACE < DEBUG < INFO < WARN < ERROR < FATAL`。

如何理解这个日志级别呢？很简单，如果项目中的日志级别设置为`INFO`，那么比它更低级别的日志信息就看不到了，即是`TRACE`、`DEBUG`日志将会不显示。



## 2、市面上的日志框架

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j...

### 2.1 下表行间无任何对应关系

| 日志门面                               | 日志实现                 |
| -------------------------------------- | ------------------------ |
| JCL( Jakarta Commons Logging)          | Log4j                    |
| SLF4j( Simple Logging Facade for Java) | JUL( java. util.logging) |
| jboss-logging                          | Log4j2                   |
|                                        | Logback                  |

左边选一个门面（抽象层）、右边选一个实现

选哪个呢？排除法

### 2.2 日志门面：slf4j

Jboss-logging：普通程序员用不了

JCL：最后一次更新是在2014年，廉颇老矣，尚能饭否？

剩下slf4j理所应当

### 2.3 日志实现：logback

log4j、logback和slf4j都是一个人写的，适配性好，log4j不错但有性能问题，但升级消耗太大，就重写了logback

所有log4j没有logback先进，JUL是Java自带的，怕日志市场被占，比较简略

log4j2是借log4j之名，由Apache公司重新做的框架，设计地非常好，由于太好还没适配

### 2.4 Spring Boot怎么做的呢？

Spring框架默认是用JCL日志框架

Spring Boot选用slf4j和logback

## 3、slf4j的使用

如何在系统中使用slf4j？[官方文档](http://www.slf4j.org/)

- 以后开发的时候，日志记录方法得调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法
- 参见用户手册[SLF4J user manual](http://www.slf4j.org/manual.html)给系统中导入slf4j的jar和logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

> 虽然默认用的是logback的实现，如果想要其他实现也可以，毕竟slf4j是抽象层，实现用什么都行

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/SLF4J-user-manual.png)

### 图解

- 如果系统中只导入了slf4j，我们要进行日志记录，就会返回空值，因为没有任何实现
- 正确用法：我的应用程序面向slf4j编程，调用它的方法进行日志记录，在程序中也导入日志实现，虽然调用slf4j接口，但logback会实现，记录到文件或控制台
- 如果slf4j要绑定log4j，log4j出现比较早，没想到要适配slf4j，所以两者绑定要有一个适配层（slf4j实现的），适配层相当于上面实现了slf4j的具体方法，而在方法里面要进行真正日志记录的时候，又调了log4j的API，要用log4j还要导入适配层即可
  JUL同理
- slf4j也有简单日志实现也能用，或者slf4j没有什么操作的实现包，也是输出空值
- 每一个日志的实现框架都有自己的配置文件，使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件
- slf4j只提供抽象层，用哪个实现就写那个的配置文件

## 4、遗留问题

开发某个系统时：使用{slf4j+logback}，依赖Spring框架（commons-logging），依赖Hibernate框架（Jboss-logging），依赖MyBatis框架等等可能一大堆

出现什么问题，系统中日志杂交？

现在就要做同一日志记录，即使是别的框架和我一起使用slf4j进行输出？

进入[slf4j官方文档](http://www.slf4j.org/)的[legacy APIs](http://www.slf4j.org/legacy.html)

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/legacy.png)

统一slf4j，使用其他包替换原有日志框架，替换的意思就是，例如要把原框架里面对Commons-logging的依赖排除掉

但如果我现在用的Spring框架缺少Commons-logging就运行不起来了，Spring底层记录日志就需要Commons-logging，那怎么办呢？就用jcl-over-slf4j.jar替换这个包，Spring要用的类这个替换包例还是有的，就不会报错了

但新的包实现怎么办呢？新的包调入slf4j，二slf4j又调到真正的实现中，其他框架不同日志框架同理替换

其他组合方式也是如此

如何让系统中所有的日志都同一到slf4j：

- 将系统中其他日志框架先排除去；
- 用中间包来替换原有的日志框架
- 我们导入slf4j其他的实现

## 5、Spring Boot日志关系

每个启动器（场景）都要依赖的

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
```

Spring Boot使用它来做日志功能

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

依赖图示：

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/slf4j-uml.png)

总结：

- Spring Boot底层也是使用slf4j+logback的方式进行日志记录

- Spring Boot也是把其他的日志都替换成了slf4j

- 中间替换包，以`jcl-over-slf4j.jar`为例：

  在项目的依赖包中找到其对应jar包：（中间转换包）

  ![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/jcl-over-slf4j.PNG)

- 从图中看出，虽然包名用的Apache的，但实现却是使用的`SLF4JLogFactory()`的日志工厂

```java
@SuppressWarnings("rawtypes")
public abstract class LogFactory {

  static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";

  static LogFactory logFactory = new SLF4JLogFactory();
  ...
}
```

- 如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉！

  Spring框架用的是commons-logging：

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <exclusions>
    <exclusion>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

Spring Boot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，我们唯一需要做的是，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉。



Spring Boot中默认的日志级别是`INFO`，启动项目日志打印如下：![img](https://gitee.com/chenjiabing666/Blog-file/raw/master/Spring%20Boot%20%E7%AC%AC%E4%B8%89%E5%BC%B9%EF%BC%8C%E6%97%A5%E5%BF%97%E6%A1%86%E6%9E%B6/1.png)

从上图可以看出，输出的日志的默认元素如下：

1. 时间日期：精确到毫秒
2. 日志级别：ERROR, WARN, INFO, DEBUG , TRACE
3. 进程ID
4. 分隔符：— 标识实际日志的开始
5. 线程名：方括号括起来（可能会截断控制台输出）
6. Logger名：通常使用源代码的类名
7. 日志内容



## 6、日志使用

### 6.1 默认配置

当我们初始化项目运行后，自己没有配置日志，但控制台是由输出信息的

Spring Boot默认帮我们配置好了日志，直接使用就可以了

```java
// LoggerFactory是记录器工厂，记录器
Logger logger = LoggerFactory.getLogger(getClass());

@Test
public void testLog() {
  /**
    * 日志的级别
    * 由低到高：trace<debug<info<waring<error
    * 可以调整输出的日志级别
    * 只打印高级别即以后（更高级别）的信息
    */
  logger.trace("这是跟踪轨迹日志...trace...");
  logger.debug("这是调试日志...debug");
  /**
    * Spring Boot默认使用的是info级别的，输出info级别即以后的内容
    * 没有指定级别的就用Spring Boot默认规定的级别（root级别）
    */
  logger.info("这是信息日志...info...");
  logger.warn("这是警告信息...warning...");
  logger.error("这是错误信息日志，异常捕获...error...");
  // 也可以通过配置文件修改级别
}
```

日志的输出格式

- %d：表示时间
- %thread：表示线程名
- %-5level：级别从左显示5个字符宽度
- %logger{50}：表示logger名字最长50个字符，否则按照句点分割
- %msg：日志消息
- %n：换行

```properties
# trace及以后的级别生效
logging.level.com.initializr=trace

# 生成springboot日志文件
# 可以用决定路径
# 如果不指定路径，就在当前项目下生成
logging.file=springboot.log

# 使用logging.path可以不使用logging.file，使用spring.log默认文件
# 在当前磁盘的根路径下创建文件夹，并生成日志文件
#logging.path=/spring/log

# 在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中输出的日志的格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} === [%thread] === %-5level %logger{50} === %msg%n
```

### 6.2 自定义日志、指定配置

可从[官方文档-日志](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/boot-features-logging.html)参见第26.5条Custom log configuration

给类路径下放上每个日志框架自己的配置文件即可，Spring Boot就不使用默认配置了

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

`logback.xml`：直接就被日志框架识别了

`logback-spring.xml`：日志框架就不直接加载日志的配置项，由Spring Boot解析日志配置，可以使用Spring Boot的高级Profile功能

```xml
<springProfile name="staging">
  <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev, staging">
  <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
  <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

可以指定某段配置只在某个环境下生效

否则就会报错

```powershell
java.lang.IllegalStateException: Logback configuration error detected: 
ERROR in ch.qos.logback.core.joran.spi.Interpreter@23:39 - 
	no applicable action for [springProfile]...
```

使用`logback-spring.xml`配置文件时：

```xml
<!-- 控制台打印 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
  <encoder charset="utf-8">
    <springProfile name="dev">
      <pattern>${CONSOLE_LOG_PATTERN}</pattern>
    </springProfile>
    <springProfile name="!dev">
      <pattern>${FILE_LOG_PATTERN}</pattern>
    </springProfile>
  </encoder>
</appender>
```



### 6.3 **代码中如何使用日志？**

第一种其实也是很早之前常用的一种方式，只需要在代码添加如下：

```java
private final Logger logger= LoggerFactory.getLogger(DemoApplicationTests.class);
```

这种方式显然比较鸡肋，如果每个类中都添加一下岂不是很low。别着急，lombok为我们解决了这个难题。

要想使用lombok，需要添加如下依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

使用也是很简单，只需要在类上标注一个注解`@Slf4j`即可，如下：

```java
@Slf4j
class DemoApplicationTests {
  @Test
  public void test(){
    log.debug("输出DEBUG日志.......");
  }
}
```

## 7、切换日志框架

### 7.1 现在我们想用log4j实现

根据之前的原理，就要除去有关log4j的转换包，用原始包；还要导出logback的jar包

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/exclude-logback.png)

IDEA->在`pom.xml`文件中鼠标右键->Diagrams->Show Dependencies->选择要除去的jar包->鼠标右键->Exclude

其他的JCL和JCL有的框架还要用，所有转换包要留着

但log4j-over-slf4j.jar要去掉，这是个替换包，里面的log4j都用处slf4j了，但现在我们要用log4j了，而不是替换，所以也将其排除

最终：

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/excluded.PNG)

面向slf4j编程，用log4j实现

导入一个适配层的包

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

查看其源码，自动导入了log4j的框架

```xml
...
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
</dependency>
...
```

可以按照slf4j的日志适配图，进行相关的切换

得到最终的pom.xml文件配置为：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

### 7.2 切换至log4j2

参阅[官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/html/using-boot-build-systems.html#using-boot-starter)表13.3. Spring Boot technical starters

![Spring Boot](https://gitee.com/mysteryguest/ObjectStorage/raw/master/Spring/exclude-starter-logging.png)

按照之前的操作将`spring-boot-start-logging.jar`除去

在pom.xml文件中导入：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>spring-boot-starter-logging</artifactId>
      <groupId>org.springframework.boot</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

## 8、logback-spring.xml

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置。因此只需要在`src/resources`文件夹下创建`logback-spring.xml`即可，配置文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志存放目录 -->
    <property name="logPath" value="logs"/>
    <!--    日志输出的格式-->
    <property name="PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t-%L] %-5level %logger{36} %L %M - %msg%xEx%n"/>
    <contextName>logback</contextName>

    <!--输出到控制台 ConsoleAppender-->
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <!--展示格式 layout-->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>${PATTERN}</pattern>
        </layout>
            <!--过滤器，只有过滤到指定级别的日志信息才会输出，如果level为ERROR，那么控制台只会输出ERROR日志-->
<!--        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">-->
<!--            <level>ERROR</level>-->
<!--        </filter>-->
    </appender>

    <!--正常的日志文件，输出到文件中-->
    <appender name="fileDEBUGLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--如果只是想要 Info 级别的日志，只是过滤 info 还是会输出 Error 日志，因为 Error 的级别高，
        所以我们使用下面的策略，可以避免输出 Error 的日志-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--过滤 Error-->
            <level>Error</level>
            <!--匹配到就禁止-->
            <onMatch>DENY</onMatch>
            <!--没有匹配到就允许-->
            <onMismatch>ACCEPT</onMismatch>
        </filter>

        <!--日志名称，如果没有File 属性，那么只会使用FileNamePattern的文件路径规则
            如果同时有<File>和<FileNamePattern>，那么当天日志是<File>，明天会自动把今天
            的日志改名为今天的日期。即，<File> 的日志都是当天的。
        -->
        <File>${logPath}/log_demo.log</File>
        <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--文件路径,定义了日志的切分方式——把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间-->
            <FileNamePattern>${logPath}/log_demo_%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--只保留最近90天的日志-->
            <maxHistory>90</maxHistory>
            <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
            <!--<totalSizeCap>1GB</totalSizeCap>-->
        </rollingPolicy>
        <!--日志输出编码格式化-->
        <encoder>
            <charset>UTF-8</charset>
            <pattern>${PATTERN}</pattern>
        </encoder>
    </appender>

    <!--输出ERROR日志到指定的文件中-->
    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--如果只是想要 Error 级别的日志，那么需要过滤一下，默认是 info 级别的，ThresholdFilter-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>Error</level>
        </filter>
        <!--日志名称，如果没有File 属性，那么只会使用FileNamePattern的文件路径规则
            如果同时有<File>和<FileNamePattern>，那么当天日志是<File>，明天会自动把今天
            的日志改名为今天的日期。即，<File> 的日志都是当天的。
        -->
        <File>${logPath}/error.log</File>
        <!--滚动策略，按照时间滚动 TimeBasedRollingPolicy-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--文件路径,定义了日志的切分方式——把每一天的日志归档到一个文件中,以防止日志填满整个磁盘空间-->
            <FileNamePattern>${logPath}/error_%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--只保留最近90天的日志-->
            <maxHistory>90</maxHistory>
            <!--用来指定日志文件的上限大小，那么到了这个值，就会删除旧的日志-->
            <!--<totalSizeCap>1GB</totalSizeCap>-->
        </rollingPolicy>
        <!--日志输出编码格式化-->
        <encoder>
            <charset>UTF-8</charset>
            <pattern>${PATTERN}</pattern>
        </encoder>
    </appender>


    <!--指定最基础的日志输出级别-->
    <root level="DEBUG">
        <!--appender将会添加到这个loger-->
        <appender-ref ref="consoleLog"/>
        <appender-ref ref="fileDEBUGLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>

    <!--    定义指定package的日志级别-->
    <logger name="org.springframework" level="DEBUG"></logger>
    <logger name="org.mybatis" level="DEBUG"></logger>
    <logger name="java.sql.Connection" level="DEBUG"></logger>
    <logger name="java.sql.Statement" level="DEBUG"></logger>
    <logger name="java.sql.PreparedStatement" level="DEBUG"></logger>
    <logger name="io.lettuce.*" level="INFO"></logger>
    <logger name="io.netty.*" level="ERROR"></logger>
    <logger name="com.rabbitmq.*" level="DEBUG"></logger>
    <logger name="org.springframework.amqp.*" level="DEBUG"></logger>
    <logger name="org.springframework.scheduling.*" level="DEBUG"></logger>
    <!--定义com.xxx..xx..xx包下的日志信息不上传，直接输出到fileDEBUGLog和fileErrorLog这个两个appender中，日志级别为DEBUG-->
    <logger name="com.xxx.xxx.xx"  additivity="false" level="DEBUG">
        <appender-ref ref="fileDEBUGLog"/>
        <appender-ref ref="fileErrorLog"/>
    </logger>

</configuration>
```

然，如果就不想用Spring Boot推荐的名字，想自己定制也行，只需要在配置文件中指定配置文件名即可，如下：

```
logging.config=classpath:logging-config.xml
```

