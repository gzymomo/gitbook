[扛麻袋的少年](https://blog.csdn.net/lzb348110175)

- [Spring Boot 配置 logback 实现【日志多环境下按指定条件滚动输出】](https://blog.csdn.net/lzb348110175/article/details/10543794- 0)
- [Spring Boot 配置 logback 实现【异常发送邮件提醒】](https://blog.csdn.net/lzb348110175/article/details/105439689)





# 一、日志多环境下按指定条件滚动输出

- 控制 dev、test、prod 等不同环境下，日志输出控制台 或者 写入到文件的配置；
- 可实现自定义返回日志格式；
- 可实现日志大于我们指定大小，滚动输出；
- 指定日志保留天数，超期自动删除；

## 1.1 配置文件配置

​    在**application.properties**配置文件中，配置以下内容，主要是为了下文 **logback-spring.xml** 中使用。**logback.project.name**、**logback.log.dir**这两个名称你可以来自定义，但是要与 **logback-spring.xml** 中名称一致。

```yaml
# 配置logback相关内容
# 项目名
logback.project.name=bi-project
# 日志保存路径
logback.log.dir=E:/logs/tmp/
```

## 1.2 新建 logback-spring.xml 文件

  在 classpath 下创建 **logback-spring.xml** 文件。**Spring Boot 在加载 logback 配置时，默认会读取 classpath 目录下的 logback-spring.xml 文件。**如果文件名需要自定义，则需要在 application.properties 配置文件下通过该**logging.config**属性来指定。

**logback-spring.xml 配置如下：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<configuration>
    <contextName>logback-log</contextName>

    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。 -->
    <springProperty scope="context" name="log.path" source="logback.log.dir" />
    <springProperty scope="context" name="log.project.name" source="logback.project.name" />

    <!--0. 日志格式和颜色渲染 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.sss}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!--1. 输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--2. 输出到文档-->
    <!-- 2.1 level为 DEBUG 日志，时间滚动输出  -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/${log.project.name}_debug.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <fileNamePattern>${log.path}/${log.project.name}-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>1000kb</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录debug级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.2 level为 INFO 日志，时间滚动输出  -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/${log.project.name}_info.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/${log.project.name}-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>1000kb</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.3 level为 WARN 日志，时间滚动输出  -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/${log.project.name}_warn.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/${log.project.name}-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100mb</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 2.4 level为 ERROR 日志，时间滚动输出  -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/${log.project.name}_error.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录(如下按小时记录) -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/${log.project.name}-error-%d{yyyy-MM-dd-HH}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!--单个文件最大值,单位kb,mb,gb-->
                <maxFileSize>100mb</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--
        <logger>用来设置某一个包或者具体的某一个类的日志打印级别、
        以及指定<appender>。<logger>仅有一个name属性，
        一个可选的level和一个可选的addtivity属性。
        name:用来指定受此logger约束的某一个包或者具体的某一个类。
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
              还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。
              如果未设置此属性，那么当前logger将会继承上级的级别。
        addtivity:是否向上级logger传递打印信息。默认是true。
        <logger name="org.springframework.web" level="info"/>
        <logger name="org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor" level="INFO"/>
    -->

    <!--
        使用mybatis的时候，sql语句是debug下才会打印，而这里我们只配置了info，所以想要查看sql语句的话，有以下两种操作：
        第一种把<root level="info">改成<root level="DEBUG">这样就会打印sql，不过这样日志那边会出现很多其他消息
        第二种就是单独给dao下目录配置debug模式，代码如下，这样配置sql语句会打印，其他还是正常info级别：
        【logging.level.org.mybatis=debug logging.level.dao=debug】
     -->

    <!--
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->

    <!-- 4. 最终的策略 -->
    <!--你可以来指定某些包来发邮件,也可以指定某个环境下都发邮件-->
    
    <!--4.0 指定某个包(debug,info级别日志)写入文件-->
	<logger name="包名" level="INFO" additivity="false">
        <appender-ref ref="DEBUG_FILE" />
		<appender-ref ref="INFO_FILE" />
		<!--此处邮件相关介绍，这个 MAIL 未作配置，可参考：https://blog.csdn.net/lzb348110175/article/details/105439689-->
		<!--这样可以来指定，只有这个包才会发邮件-->
		<!--<appender-ref ref="MAIL"/>-->
    </logger>

    <!-- 4.1 开发环境:打印控制台-->
    <springProfile name="dev">
        <root level="info">
            <!--代表开发环境，只打印日志到客户端-->
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

    <!-- 4.2 测试环境:输出到控制台 + 文档 -->
    <springProfile name="test">
        <root level="info">
            <!--控制台-->
            <appender-ref ref="CONSOLE" />
            <!--文档-->
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile>

    <!-- 4.3 生产环境:输出到控制台 + 文档 -->
    <springProfile name="prod">
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile>

</configuration>
```





# 二、异常发送邮件提醒

 项目在出现异常时，能够及时通过邮件方式来发送报警信息。

## 2.1 添加maven依赖

```xml
<dependency>
	<groupId>javax.mail</groupId>
	<artifactId>mail</artifactId>
	<version>1.4.7</version>
</dependency>
```

## 2.2 配置文件配置

在**application.properties**配置文件中，配置一下内容，主要是为了下文 **logback-spring.xml** 中使用。具体配置介绍如下所示：

```yaml
# 配置logback相关内容
logback.project.name=项目名(bi-project)
logback.log.dir=E:/logs/tmp/

# 如果logback配置名称为:logback-spring.xml 则不需要配置 logging.config，会默认读取这个名称文件
# 否则需要配置 logging.config 属性
logging.config=classpath:logback-spring.xml

# 此处设置debug，会将打印的sql，全部保存到 logback-spring.xml 文件下配置的debug文件下
# 此处配置会打印 sql 到控制台
logging.level.com.example.logback.mapper=debug #(com.example.logback.mapper是项目包名)

# logback 邮件相关配置
spring.mail.host=smtp.qq.com   #服务器地址
spring.mail.username=348110xxx@qq.com    #发送邮件用户
spring.mail.password=zkeujotaaxxxxxxx    #密码
spring.mail.default-encoding=UTF-8    #编码
spring.mail.error.subject=[ERROR] in 项目名    #自定义邮件主题(填写项目名用于提示)
spring.mail.error.to=lzb348110xxx@163.com,lzb348110xxx@126.com     #接受者邮件地址
```

 **Tips：**服务器地址，密码等，并不是QQ密码，是一个授权码。详细介绍请参考：[Spring Boot配置邮件发送](https://blog.csdn.net/lzb348110175/article/details/105427148))。**spring.mail.error.subject**、**spring.mail.error.to** 这两个属性名可以自定义，用于 **logback-spring.xml** 配置文件中使用。



## 2.3 新建 logback-spring.xml 文件

​	在 classpath 下创建 **logback-spring.xml** 文件。**Spring Boot 在加载 logback 配置时，默认会读取 classpath 目录下的 logback-spring.xml 文件。**如果文件名需要自定义，则需要在 application.properties 配置文件下通过该**logging.config**属性来指定。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<configuration>
    <contextName>logback-email</contextName>
    
    <!--logback异常邮件发送-->
    <!-- 邮件配置 -->
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。 -->
    <springProperty scope="context" name="smtpHost" source="spring.mail.host" />
    <springProperty scope="context" name="username" source="spring.mail.username" />
    <springProperty scope="context" name="password" source="spring.mail.password" />
    <springProperty scope="context" name="mailSubject" source="spring.mail.error.subject" />
    <springProperty scope="context" name="mailTo" source="spring.mail.error.to" />
    <springProperty scope="context" name="charsetEncoding" source="spring.mail.default-encoding" />

    <appender name="MAIL" class="ch.qos.logback.classic.net.SMTPAppender">
        <!--服务器地址-->
        <smtpHost>${smtpHost}</smtpHost>
        <!--端口(默认为25)-->
        <smtpPort>25</smtpPort>
        <!--用户名-->
        <username>${username}</username>
        <!--密码(授权码)-->
        <password>${password}</password>
        <!--是否开启SSL安全-->
        <SSL>false</SSL>
        <!--是否同步发送-->
        <asynchronousSending>true</asynchronousSending>
        <!--发送者-->
        <from>${username}</from>
        <!--接收者-->
        <to>${mailTo}</to>
        <!--邮件主题-->
        <subject>${mailSubject}: %logger{0} </subject>
        <!--编码-->
        <charsetEncoding>${charsetEncoding}</charsetEncoding>
        <cyclicBufferTracker class="ch.qos.logback.core.spi.CyclicBufferTracker">
            <!-- 每个电子邮件只发送一个日志条目 -->
            <bufferSize>1</bufferSize>
        </cyclicBufferTracker>
        <!--HTML展示-->
        <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <!--错误级别(只会提示大于该级别的错误)-->
            <level>ERROR</level>
        </filter>
    </appender>
    
	
    <!-- 4. 最终的策略 -->
    <!--你可以来指定某些包来发邮件,也可以指定某个环境下都发邮件-->
    
    <!--4.0 指定某个包发送邮件-->
	<logger name="包名" level="INFO" additivity="false">
        <appender-ref ref="MAIL"/>
    </logger>

    <!-- 4.1 开发环境:打印控制台-->
    <springProfile name="dev">
        <root level="info">
            <!--代表开发环境，只打印日志到客户端-->
            <!--<appender-ref ref="CONSOLE" />--> <!--此处介绍邮件相关，这个 CONSOLE 未作配置-->
            <!--代表开发环境，有大于Error级别(级别自己配置)的，会通过邮件提醒-->
            <appender-ref ref="MAIL"/>
        </root>
    </springProfile>
    
	<!-- 4.2 测试环境+生产环境:输出到控制台 + 文档(如下CONSOLE、DEBUG_FILE 等未配置，这部分具体配置可参考：https://blog.csdn.net/lzb348110175/article/details/105437940) -->
    <springProfile name="test,prod">
        <root level="info">
            <!--控制台-->
            <appender-ref ref="CONSOLE" />
            <!--文档-->
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
			<appender-ref ref="MAIL" />
        </root>
    </springProfile>
</configuration>
```

## 2.4 测试

Service 层，我们来手动一个错误。发送请求后，程序便会执行到 catch，然后触发 **log.error**进行日志输出。由于我们配置的日志级别为**error**，此时我们便会收到一条异常提醒邮件。邮件如下图所示

```java
@Service
@Slf4j
public class UserServiceImpl implements UserService {

    @Autowired
    UserMapper userMapper;

    @Override
    public User getUserById(int id) {
        User user = null;
        try {
             userMapper.getUserById(id);
            System.out.println(1 / 0);
        } catch (Exception e) {
            log.error("出错了",e);
        }
        return user;
    }
}
```

