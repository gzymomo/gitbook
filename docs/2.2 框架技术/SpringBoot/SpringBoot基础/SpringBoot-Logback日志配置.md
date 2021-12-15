## SpringBoot中的日志Logback

在项目中，我们使用Logback，其实只需增加一个配置文件（自定义你的配置）即可。

### **配置文件详解**

　　配置文件精简结构如下所示：

```yaml
　　<configuration scan="true" scanPeriod="60 seconds" debug="false">  
　　         <!-- 属性文件:在properties/yml文件中找到对应的配置项 -->
　　    <springProperty scope="context" name="logging.path" source="logging.path"/>
　　    <contextName>程序员小明</contextName> 
　　    
　　    <appender>
　　        //xxxx
　　    </appender>   
　　    
　　    <logger>
　　        //xxxx
　　    </logger>
　　    
　　    <root>             
　　       //xxxx
　　    </root>  
　　</configuration> 
```

这个文件在springboot中默认叫做logback-spring.xml，我们只要新建一个同名文件放在resources下面， 配置即可生效。

　　每个配置的解释如下所示：

　　**contextName**

　　每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用contextName标签设置成其他名字，用于区分不同应用程序的记录。

　　**property**

　　用来定义变量值的标签，property标签有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过property定义的值会被插入到logger上下文中。定义变量后，可以使“${name}”来使用变量。如上面的xml所示。

　　**logger**

　　用来设置某一个包或者具体的某一个类的日志打印级别以及指定appender。

　　**root**

　　根logger，也是一种logger，且只有一个level属性。

　　**appender**

　　负责写日志的组件

　　**appender 的种类**

　　**·**ConsoleAppender：把日志添加到控制台

　　**·**FileAppender：把日志添加到文件

　　**·**RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。它是FileAppender的子类

　　**filter**

　　filter其实是appender里面的子元素。它作为过滤器存在，执行一个过滤器会有返回DENY，NEUTRAL，ACCEPT三个枚举值中的一个。

　　**·**DENY：日志将立即被抛弃不再经过其他过滤器

　　**·**NEUTRAL：有序列表里的下个过滤器过接着处理日志

　　**·**ACCEPT：日志会被立即处理，不再经过剩余过滤器

### **有以下几种过滤器：**

　　**ThresholdFilter**

　　临界值过滤器，过滤掉低于指定临界值的日志。当日志级别等于或高于临界值时，过滤器返回NEUTRAL；当日志级别低于临界值时，日志会被拒绝。

```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>INFO</level>
</filter>
```

**LevelFilter**

　　级别过滤器，根据日志级别进行过滤。如果日志级别等于配置级别，过滤器会根据onMath(用于配置符合过滤条件的操作) 和 onMismatch(用于配置不符合过滤条件的操作)接收或拒绝日志。

```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">   
    <level>INFO</level>   
    <onMatch>ACCEPT</onMatch>   
    <onMismatch>DENY</onMismatch>   
</filter>
```

## 项目实例

### 　　**准备**

　　一个简单正常的Springboot项目

### 　　**配置文件**

　　**application.yml**

　　有关日志的简单配置，我们可以直接在application.yml中进行简单的配置，比如指明日志的打印级别和日志的输出位置：

```yaml
　　logging:
　　  level:
　　    root: info
　　  path: ./logs
```

也可以根据分环境配置指明使用的配置文件，缺省为logback-spring.xml

```yaml
　　logging:
　　  level:
　　    root: info
　　  path: ./logs
　　  config: classpath:/logback-dev.xml
```

**logback-spring.xml**

　　在resources目录下新建logback-spring.xml文件，举例一个简单的需求，如果在项目中我们如果需要指定日志的输出格式以及根据日志级别输出到不同的文件，可以配置如下：

```xml
　　<?xml version="1.0" encoding="UTF-8" ?>
　　<configuration>
　　    <!-- 属性文件:在properties文件中找到对应的配置项 -->
　　    <springProperty scope="context" name="logging.path" source="logging.path"/>
　　    <contextName>xiaoming</contextName>
　　    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
　　        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
　　            <!--格式化输出（配色）：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
　　            <pattern>%yellow(%d{yyyy-MM-dd HH:mm:ss}) %red([%thread]) %highlight(%-5level) %cyan(%logger{50}) - %magenta(%msg) %n
　　            </pattern>
　　            <charset>UTF-8</charset>
　　        </encoder>
　　    </appender>
　　    <!--根据日志级别分离日志，分别输出到不同的文件-->
　　    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
　　        <filter class="ch.qos.logback.classic.filter.LevelFilter">
　　            <level>ERROR</level>
　　            <onMatch>DENY</onMatch>
　　            <onMismatch>ACCEPT</onMismatch>
　　        </filter>
　　        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
　　            <pattern>
　　                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
　　            </pattern>
　　            <charset>UTF-8</charset>
　　        </encoder>
　　        <!--滚动策略-->
　　        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
　　            <!--按时间保存日志 修改格式可以按小时、按天、月来保存-->
　　            <fileNamePattern>${logging.path}/xiaoming.info.%d{yyyy-MM-dd}.log</fileNamePattern>
　　            <!--保存时长-->
　　            <MaxHistory>90</MaxHistory>
　　            <!--文件大小-->
　　            <totalSizeCap>1GB</totalSizeCap>
　　        </rollingPolicy>
　　    </appender>
　　    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
　　        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
　　            <level>ERROR</level>
　　        </filter>
　　        <encoder>
　　            <pattern>
　　                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
　　            </pattern>
　　        </encoder>
　　        <!--滚动策略-->
　　        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
　　            <!--路径-->
　　            <fileNamePattern>${logging.path}/xiaoming.error.%d{yyyy-MM-dd}.log</fileNamePattern>
　　            <MaxHistory>90</MaxHistory>
　　        </rollingPolicy>
　　    </appender>
　　    <root level="info">
　　        <appender-ref ref="consoleLog"/>
　　        <appender-ref ref="fileInfoLog"/>
　　        <appender-ref ref="fileErrorLog"/>
　　    </root>
　　</configuration>
```

再比如如果粒度再细一些，根据不同的模块，输出到不同的文件，可以如下配置：

```xml
　　 <!--特殊功能单独appender 例如调度类的日志-->
　　    <appender name="CLASS-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
　　        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
　　            <level>INFO</level>
　　        </filter>
　　        <encoder>
　　            <pattern>
　　                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{50} - %msg%n
　　            </pattern>
　　        </encoder>
　　        <!--滚动策略-->
　　        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
　　            <!--路径-->
　　            <fileNamePattern>${logging.path}/mkc.class.%d{yyyy-MM-dd}.log</fileNamePattern>
　　            <MaxHistory>90</MaxHistory>
　　        </rollingPolicy>
　　    </appender>
　　    <!--这里的name和业务类中的getLogger中的字符串是一样的-->
　　    <logger name="xiaoming" level="INFO" additivity="true">
　　        <appender-ref ref="CLASS-APPENDER" />
　　    </logger>
```

正常情况下xiaoming是指的：

```java
private Logger xiaoming = LoggerFactory.getLogger("xiaoming");
```

如果我们使用的是lomok插件，则xiaoming指的是topic

```java
@Slf4j(topic = "xiaoming")
public class XiaoMingTest {
}
```



