部分内容原文地址：
博客园：鸡员外：[五年Java经验，面试还是说不出日志该怎么写更好？——日志规范与最佳实践篇](https://www.cnblogs.com/xuningfans/p/12164734.html)



# 日志框架介绍及使用




 - **运维人员**   整个系统大部分时间都是运维人员来维护，日志可以帮助运维人员来了解系统状态（很多运维系统接入的也是日志），运维人员发现日志有异常信息也可以及时通知开发来排查
 - **运营人员**  运营人员，比如电商的转化率、视频网站的完播率、普通PV数据等都可以通过日志进行统计，随着大数据技术的普及，这部分日志占比也越来越高

日志有几种？
 - **调试日志** 用于开发人员开发或者线上回溯问题。
 - **诊断日志** 一般用于运维人员监控系统与安全人员分析预警。
 - **埋点日志** 一般用于运营决策分析，也有用作微服务调用链路追踪的（运维、调试）。
 - **审计日志** 与诊断日志类似，诊断日志偏向运维，审计日志偏向安全。

日志都需要输出什么？

 - **调试日志**
   - DEBUG 或者 TRACE 级别，比如方法调用参数，网络连接具体信息，一般是开发者调试程序使用，线上非特殊情况关闭这些日志
   - INFO 级别，一般是比较重要却没有风险的信息，如初始化环境、参数，清理环境，定时任务执行，远程调用第一次连接成功
   - WARN   级别，有可能有风险又不影响系统继续执行的错误，比如系统参数配置不正确，用户请求的参数不正确（要输出具体参数方便排查），或者某些耗性能的场景，比如一次请求执行太久、一条sql执行超过两秒，某些第三方调用失败，不太可能被运行的if分支等
   - ERROR   级别，用于程序出错打印堆栈信息，不应该用于输出程序问题之外的其他信息，需要注意打印了日志异常（Exception）就不应该抛（throw）了
 - **诊断日志** 一般输出 INFO 级别，请求响应时间，内存占用等等，线上接入监控系统时打开，建议输出到独立的文件，可以考虑 JSON   格式方便外部工具分析
 - **埋点日志** 业务按需定制，比如上文提到的转化率可以在用户付款时输出日志，完播率可以在用户播放完成后请求一次后台输出日志，一般可输出 INFO   级别，建议输出到独立的文件，可以考虑JSON格式方便外部工具分析
 - **审计日志** 大多 WARN 级别或者 INFO 级别，一般是敏感操作即可输出，登陆、转账付款、授权消权、删除等等，建议输出到独立的文件，可以考虑JSON格式方便外部工具分析

一般调试日志由开发者自定义输出，其他三种应该根据实际业务需求来定制。

日志的其他注意点

 1. 线上日志应尽量谨慎，要思考：这个位置输出日志能帮助排除问题吗？输出的信息与排查问题相关吗？输出的信息足够排除问题吗？做到不少输出必要信息，不多输出无用信息（拖慢系统，淹没有用信息）
 2. 超级 SessionId 与 RequestId，无论是单体应用还是微服务架构，应该为每个用户每次登陆生成一个超级SessionId，方便跟踪区分一个用户；RequestId，每次请求生成一个RequestId，用于跟踪一次请求，微服务也可以用于链路追踪
 3. 日志要尽量单行输出，一条日志输出一行，否则不方便阅读以及其他第三方系统或者工具分析
 4. 公司内部应该制定一套通用的日志规范，包括日志的格式，变量名（驼峰、下划线），分隔符（“=”或“:”等），何时输出（比如规定调用第三方前后输出INFO日志），公司的日志规范应该不断优化、调整，找到适合公司业务的最佳规范

# 一、Log4j

## 1.1新建一个Java工程，导入Log4j包，pom文件中对应的配置代码如下：

```xml
<!-- log4j support -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

## 1.2resources目录下创建log4j.properties文件。

```yml
### 设置###
log4j.rootLogger = debug,stdout,D,E

### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

### 输出DEBUG 级别以上的日志到=/home/duqi/logs/debug.log ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = /home/duqi/logs/debug.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 输出ERROR 级别以上的日志到=/home/admin/logs/error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =/home/admin/logs/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

```

## 1.3输出日志

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4JTest {
    private static final Logger logger = LoggerFactory.getLogger(Log4JTest.class);

    public static void main(String[] args) {
        // 记录debug级别的信息
        logger.debug("This is debug message.");
        // 记录info级别的信息
        logger.info("This is info message.");
        // 记录error级别的信息
        logger.error("This is error message.");
    }
}

```
## 1.4控制台查看输出结果。

然后可以再/Users/duqi/logs目录下的debug.log和error.log文件中查看信息。

# 二、Log4J基本使用方法

Log4j由三个重要的组件构成：日志信息的优先级，日志信息的输出目的地，日志信息的输出格式。日志信息的优先级从高到低有ERROR、WARN、 INFO、DEBUG，分别用来指定这条日志信息的重要程度；日志信息的输出目的地指定了日志将打印到控制台还是文件中；而输出格式则控制了日志信息的显 示内容。

## 2.1	定义配置文件

也可以完全不使用配置文件，而是在代码中配置Log4j环境。但是，使用配置文件将使您的应用程序更加灵活。Log4j支持两种配置文件格式，一种是XML格式的文件，一种是Java特性文件（键=值）。下面我们介绍使用Java特性文件做为配置文件的方法：

1：配置根Logger，其语法为：

```yml
log4j.rootLogger = [ level ] , appenderName, appenderName, …
```

其中，level 是日志记录的优先级，分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。Log4j建议只使用四个级别，优 先级从高到低分别是ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定 义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来。 appenderName就是指把日志信息输出到哪个地方。您可以同时指定多个输出目的地，例如上述例子我们制定了stdout、D和E这三个地方。

2：配置文件的输出目的地Appender，一般，配置代码的格式如下。

```yml
log4j.appender.appenderName = fully.qualified.name.of.appender.class  
log4j.appender.appenderName.option1 = value1  
…  
log4j.appender.appenderName.option = valueN

```

其中，Log4j提供的appender有以下几种：

 - org.apache.log4j.ConsoleAppender（控制台），
 - org.apache.log4j.FileAppender（文件），
 - org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），
 - org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件），
 - org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

3：配置日志信息的格式（布局），其语法为：

```yml
log4j.appender.appenderName.layout = fully.qualified.name.of.layout.class  
log4j.appender.appenderName.layout.option1 = value1  
…  
log4j.appender.appenderName.layout.option = valueN

```

其中，Log4j提供的layout有以下几种：

 - org.apache.log4j.HTMLLayout（以HTML表格形式布局），
 - org.apache.log4j.PatternLayout（可以灵活地指定布局模式），
 - rg.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），
 - org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下：

 - %m 输出代码中指定的消息
 - %p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL
 - %r 输出自应用启动到输出该log信息耗费的毫秒数
 - %c 输出所属的类目，通常就是所在类的全名
 - %t 输出产生该日志事件的线程名
 - %n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”
 - %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921
 - %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)

## 2.2	在代码中使用Log4j

1：获取记录器。

> 使用Log4j，第一步就是获取日志记录器，这个记录器将负责控制日志信息。其语法为：public static Logger getLogger( String name)；通过指定的名字获得记录器，如果必要的话，则为这个名字创建一个新的记录器。Name一般取本类的名字，比如：static Logger logger = Logger.getLogger ( ServerWithLog4j.class.getName () )。

2：读取配置文件
当获得了日志记录器之后，第二步将配置Log4j环境，其语法为：

```
BasicConfigurator.configure ()： 自动快速地使用缺省Log4j环境。  
PropertyConfigurator.configure ( String configFilename) ：读取使用Java的特性文件编写的配置文件。  
DOMConfigurator.configure ( String filename ) ：读取XML形式的配置文件。

```

3：插入记录信息（格式化日志信息）

当上两个必要步骤执行完毕，您就可以轻松地使用不同优先级别的日志记录语句插入到您想记录日志的任何地方，其语法如下：

```java
Logger.debug ( Object message ) ;  
Logger.info ( Object message ) ;  
Logger.warn ( Object message ) ;  
Logger.error ( Object message ) ;
```

## 2.3	日志级别

每个Logger都被了一个日志级别（log level），用来控制日志信息的输出。日志级别从高到低分为：

 - A：off 最高等级，用于关闭所有日志记录。
 - B：fatal 指出每个严重的错误事件将会导致应用程序的退出。
 - C：error 指出虽然发生错误事件，但仍然不影响系统的继续运行。
 - D：warm 表明会出现潜在的错误情形。
 - E：info 一般和在粗粒度级别上，强调应用程序的运行全程。
 - F：debug 一般用于细粒度级别上，对调试应用程序非常有帮助。
 - G：all 最低等级，用于打开所有日志记录。

上面这些级别是定义在org.apache.log4j.Level类中。Log4j只建议使用4个级别，优先级从高到低分别是error,warn,info和debug。通过使用日志级别，可以控制应用程序中相应级别日志信息的输出。例如，如果使用b了info级别，则应用程序中所有低于info级别的日志信息(如debug)将不会被打印出来。

# 三、Spring中使用Log4j


一般是在web.xml配置文件中配置Log4j监听器和log4j.properties文件，代码如下：

```xml
<context-param>
   <param-name>log4jConfigLocation</param-name>
   <param-value>classpath:/config/log4j.properties</param-value>
</context-param>
<context-param>
   <param-name>log4jRefreshInterval</param-name>
   <param-value>60000</param-value>
</context-param>
<listener>
   <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
</listener>
```

# 四、Commons Logging
common-logging是apache提供的一个通用的日志接口，
在common-logging中，有一个Simple logger的简单实现，但是它功能很弱，所以使用common-logging，通常都是配合着log4j来使用；

Commons Logging定义了一个自己的接口 org.apache.commons.logging.Log，以屏蔽不同日志框架的API差异，这里用到了Adapter Pattern（适配器模式）。

# 五、SLF4J
Simple Logging Facade for Java（SLF4J）用作各种日志框架（例如java.util.logging，logback，log4j）的简单外观或抽象，允许最终用户在部署时插入所需的日志框架。

要切换日志框架，只需替换类路径上的slf4j绑定。 例如，要从java.util.logging切换到log4j，只需将slf4j-jdk14-1.8.0-beta2.jar替换为slf4j-log4j12-1.8.0-beta2.jar

SLF4J不依赖于任何特殊的类装载机制。 实际上，每个SLF4J绑定在编译时都是硬连线的，以使用一个且只有一个特定的日志记录框架。 例如，slf4j-log4j12-1.8.0-beta2.jar绑定在编译时绑定以使用log4j。 在您的代码中，除了slf4j-api-1.8.0-beta2.jar之外，您只需将您选择的一个且只有一个绑定放到相应的类路径位置。 不要在类路径上放置多个绑定。

以下是slf4j 绑定其它日志组件的图解说明。
![slf4j](https://img-blog.csdnimg.cn/20190219091904859.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70)

因此，slf4j 就是众多日志接口的集合，他不负责具体的日志实现，只在编译时负责寻找合适的日志系统进行绑定。具体有哪些接口，全部都定义在slf4j-api中。查看slf4j-api源码就可以发现，里面除了public final class LoggerFactory类之外，都是接口定义。因此，slf4j-api本质就是一个接口定义。

总之，Slf4j更好的兼容了各种具体日志实现的框架，如图：
![slf4j](https://img-blog.csdnimg.cn/20190219091945424.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70)

# 六、Log4j2
Apache Log4j 2是对Log4j的升级，它比其前身Log4j 1.x提供了重大改进，并提供了Logback中可用的许多改进，同时修复了Logback架构中的一些固有问题。

与Logback一样，Log4j2提供对SLF4J的支持，自动重新加载日志配置，并支持高级过滤选项。 除了这些功能外，它还允许基于lambda表达式对日志语句进行延迟评估，为低延迟系统提供异步记录器，并提供无垃圾模式以避免由垃圾收集器操作引起的任何延迟。

所有这些功能使Log4j2成为这三个日志框架中最先进和最快的。



Log4j有三个主要的组件：Loggers([**记录**](javascript:;)器)，Appenders (输出源)和Layouts(布局)。这里可简单理解为日志类别，日志要输出的地方和日志以何种形式输出。综合使用这三个组件可以轻松地记录信息的类型和级别，并可以在运行时控制日志输出的样式和位置。



## 6.1 Loggers

　　Loggers组件在此系统中被分为五个级别：DEBUG、INFO、WARN、ERROR和FATAL。这五个级别是有顺序的，DEBUG < INFO < WARN < ERROR < FATAL，分别用来指定这条日志信息的重要程度，明白这一点很重要，Log4j有一个规则：只输出级别不低于设定级别的日志信息，假设Loggers级别设定为INFO，则INFO、WARN、ERROR和FATAL级别的日志信息都会输出，而级别比INFO低的DEBUG则不会输出。



## 6.2 Appenders

　　禁用和使用日志请求只是Log4j的基本功能，Log4j日志系统还提供许多强大的功能，比如允许把日志输出到不同的地方，如控制台（Console）、文件（Files）等，可以根据天数或者文件大小产生新的文件，可以以流的形式发送到其它地方等等。

　　常使用的类如下：

　　**·**org.apache.log4j.ConsoleAppender（控制台）

　　**·**org.apache.log4j.FileAppender（文件）

　　**·**org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）

　　**·**org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）

　　**·**org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

### **配置模式**：

　　**·**log4j.appender.appenderName = className

　　**·**log4j.appender.appenderName.Option1 = value1

　　**·**log4j.appender.appenderName.OptionN = valueN



## 6.3 Layouts

　　有时用户希望根据自己的喜好格式化自己的日志输出，Log4j可以在Appenders的后面附加Layouts来完成这个功能。Layouts提供四种日志输出样式，如根据HTML样式、自由指定样式、包含日志级别与信息的样式和包含日志时间、线程、类别等信息的样式。

　　常使用的类如下：

　　**·**org.apache.log4j.HTMLLayout（以HTML表格形式布局）

　　**·**org.apache.log4j.PatternLayout（可以灵活地指定布局模式）

　　**·**org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）

　　**·**org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等信息）

#### **配置模式：**

　　**·**log4j.appender.appenderName.layout =className

　　**·**log4j.appender.appenderName.layout.Option1 = value1

　　**·**log4j.appender.appenderName.layout.OptionN = valueN



## 6.4 配置详解

　　在实际应用中，要使Log4j在系统中运行须事先设定配置文件。配置文件事实上也就是对Logger、Appender及Layout进行相应设定。Log4j支持两种配置文件格式，一种是XML格式的文件，一种是properties属性文件。下面以properties属性文件为例介绍log4j.properties的配置。

### 1、配置根Logger

　　log4j.rootLogger = [ level ] , appenderName1, appenderName2, … 

　　log4j.additivity.org.apache=false：表示Logger不会在父Logger的appender里输出，默认为true。 

　　level ：设定日志记录的最低级别，可设的值有OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者自定义的级别，Log4j建议只使用中间四个级别。通过在这里设定级别，您可以控制应用程序中相应级别的日志信息的开关，比如在这里设定了INFO级别，则应用程序中所有DEBUG级别的日志信息将不会被打印出来。 

　　appenderName：就是指定日志信息要输出到哪里。可以同时指定多个输出目的地，用逗号隔开。 

　　例如：log4j.rootLogger＝INFO,A1,B2,C3

### **2、配置日志信息输出目的地（appender）**

　　log4j.appender.appenderName = className 

　　appenderName：自定义appderName，在log4j.rootLogger设置中使用； 

　　className：可设值如下：

　　**·**org.apache.log4j.ConsoleAppender（控制台）

　　**·**org.apache.log4j.FileAppender（文件）

　　**·**org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）

　　**·**org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）

　　**·**org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

#### **(1)ConsoleAppender选项：**

　　Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。 

　　ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。 

　　Target=System.err：默认值是System.out。

#### **(2)FileAppender选项：**

　　Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。 

　　ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。 

　　Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。 

　　File=D:/logs/logging.log4j：指定消息输出到logging.log4j文件中。

#### **(3)DailyRollingFileAppender选项：**

　　Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。 

　　ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。 

　　Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。 

　　File=D:/logs/logging.log4j：指定当前消息输出到logging.log4j文件中。 

　　DatePattern=’.’yyyy-MM：每月滚动一次日志文件，即每月产生一个新的日志文件。当前月的日志文件名为logging.log4j，前一个月的日志文件名为logging.log4j.yyyy-MM。

　　另外，也可以指定按周、天、时、分等来滚动日志文件，对应的格式如下：

　　yyyy-MM：每月

　　yyyy-ww：每周

　　yyyy-MM-dd：每天

　　yyyy-MM-dd-a：每天两次

　　yyyy-MM-dd-HH：每小时

　　yyyy-MM-dd-HH-mm：每分钟

#### **(4)RollingFileAppender选项：**

　　Threshold=WARN：指定日志信息的最低输出级别，默认为DEBUG。 

　　ImmediateFlush=true：表示所有消息都会被立即输出，设为false则不输出，默认值是true。 

　　Append=false：true表示消息增加到指定文件中，false则将消息覆盖指定的文件内容，默认值是true。 

　　File=D:/logs/logging.log4j：指定消息输出到logging.log4j文件中。 

　　MaxFileSize=100KB：后缀可以是KB, MB 或者GB。在日志文件到达该大小时，将会自动滚动，即将原来的内容移到logging.log4j.1文件中。 

　　MaxBackupIndex=2：指定可以产生的滚动文件的最大数，例如，设为2则可以产生logging.log4j.1，logging.log4j.2两个滚动文件和一个logging.log4j文件。

### **3、配置日志信息的输出格式（Layout）**

　　log4j.appender.appenderName.layout=className 

　　className：可设值如下：

　　**·**org.apache.log4j.HTMLLayout（以HTML表格形式布局）

　　**·**org.apache.log4j.PatternLayout（可以灵活地指定布局模式）

　　**·**org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）

　　**·**org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

#### **(1)HTMLLayout选项：**

　　LocationInfo=true：输出java文件名称和行号，默认值是false。 

　　Title=My Logging： 默认值是Log4J Log Messages。

#### **(2)PatternLayout选项：**

　　ConversionPattern=%m%n：设定以怎样的格式显示消息。

　　格式化符号说明：

　　%p：输出日志信息的优先级，即DEBUG，INFO，WARN，ERROR，FATAL。 

　　%d：输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，如：%d{yyyy/MM/dd HH:mm:ss,SSS}。 

　　%r：输出自应用程序启动到输出该log信息耗费的毫秒数。 

　　%t：输出产生该日志事件的线程名。 

　　%l：输出日志事件的发生位置，相当于%c.%M(%F:%L)的组合，包括类全名、方法、文件名以及在代码中的行数。例如：test.TestLog4j.main(TestLog4j.java:10)。 

　　%c：输出日志信息所属的类目，通常就是所在类的全名。 

　　%M：输出产生日志信息的方法名。 

　　%F：输出日志消息产生时所在的文件名称。 

　　%L:：输出代码中的行号。 

　　%m:：输出代码中指定的具体日志信息。 

　　%n：输出一个回车换行符，[**Windows**](javascript:;)平台为”rn”，Unix平台为”n”。 

　　%x：输出和当前线程相关联的NDC(嵌套诊断环境)，尤其用到像java servlets这样的多客户多线程的应用中。 

　　%%：输出一个”%”字符。

　　另外，还可以在%与格式字符之间加上修饰符来控制其最小长度、最大长度、和文本的对齐方式。如：

　　**·**c：指定输出category的名称，最小的长度是20，如果category的名称长度小于20的话，默认的情况下右对齐。

　　**·**%-20c：”-“号表示左对齐。

　　**·**%.30c：指定输出category的名称，最大的长度是30，如果category的名称长度大于30的话，就会将左边多出的字符截掉，但小于30的话也不会补空格。

## 6.5 附：Log4j比较全面的配置

Log4j配置文件实现了输出到控制台、文件、回滚文件、发送日志邮件、输出到[**数据库**](javascript:;)日志表、自定义标签等全套功能。

```yaml
log4j.rootLogger=DEBUG,console,dailyFile,im
log4j.additivity.org.apache=true
# 控制台(console)
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.ImmediateFlush=true
log4j.appender.console.Target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 日志文件(logFile)
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.Threshold=DEBUG
log4j.appender.logFile.ImmediateFlush=true
log4j.appender.logFile.Append=true
log4j.appender.logFile.File=D:/logs/log.log4j
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 回滚文件(rollingFile)
log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.Threshold=DEBUG
log4j.appender.rollingFile.ImmediateFlush=true
log4j.appender.rollingFile.Append=true
log4j.appender.rollingFile.File=D:/logs/log.log4j
log4j.appender.rollingFile.MaxFileSize=200KB
log4j.appender.rollingFile.MaxBackupIndex=50
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 定期回滚日志文件(dailyFile)
log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyFile.Threshold=DEBUG
log4j.appender.dailyFile.ImmediateFlush=true
log4j.appender.dailyFile.Append=true
log4j.appender.dailyFile.File=D:/logs/log.log4j
log4j.appender.dailyFile.DatePattern='.'yyyy-MM-dd
log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout
log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于socket
log4j.appender.socket=org.apache.log4j.RollingFileAppender
log4j.appender.socket.RemoteHost=localhost
log4j.appender.socket.Port=5001
log4j.appender.socket.LocationInfo=true
# Set up for Log Factor 5
log4j.appender.socket.layout=org.apache.log4j.PatternLayout
log4j.appender.socket.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# Log Factor 5 Appender
log4j.appender.LF5_APPENDER=org.apache.log4j.lf5.LF5Appender
log4j.appender.LF5_APPENDER.MaxNumberOfRecords=2000
# 发送日志到指定邮件
log4j.appender.mail=org.apache.log4j.net.SMTPAppender
log4j.appender.mail.Threshold=FATAL
log4j.appender.mail.BufferSize=10
log4j.appender.mail.From = xxx@mail.com
log4j.appender.mail.SMTPHost=mail.com
log4j.appender.mail.Subject=Log4J Message
log4j.appender.mail.To= xxx@mail.com
log4j.appender.mail.layout=org.apache.log4j.PatternLayout
log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 应用于数据库
log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.database.URL=jdbc:mysql://localhost:3306/test
log4j.appender.database.driver=com.mysql.jdbc.Driver
log4j.appender.database.user=root
log4j.appender.database.password=
log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) --> [%t] %l: %m %x %n')
log4j.appender.database.layout=org.apache.log4j.PatternLayout
log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
# 自定义Appender
log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender
log4j.appender.im.host = mail.cybercorlin.net
log4j.appender.im.username = username
log4j.appender.im.password = password
log4j.appender.im.recipient = corlin@cybercorlin.net
log4j.appender.im.layout=org.apache.log4j.PatternLayout
log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n
```

log4j的强大功能无可置疑，但实际应用中免不了遇到某个功能需要输出独立的日志文件的情况，怎样才能把所需的内容从原有日志中分离，形成单独的日志文件呢？其实只要在现有的log4j基础上稍加配置即可轻松实现这一功能。

　　先看一个常见的log4j.properties文件，它是在控制台和myWeb.log文件中记录日志：

```yaml
log4j.rootLogger=debug, Console, file
# Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%d [%t] %-5p [%c] - %m%n
# file
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.DatePattern='-'yyyy-MM-dd
log4j.appender.file.File=../myWeb.log
log4j.appender.file.Append=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} [%p] [ %l ] %n %m%n
```

如果想对不同的类输出不同的文件(以cn.com.Test为例)，先要在Test.java中定义:

```java
private static Log logger = LogFactory.getLog(Test.class);
```

　log4j.properties中加入

```yaml
log4j.logger.cn.com.Test= DEBUG, test
log4j.appender.test=org.apache.log4j.FileAppender
log4j.appender.test.File=${myweb.root}/WEB-INF/log/test.log
log4j.appender.test.layout=org.apache.log4j.PatternLayout
log4j.appender.test.layout.ConversionPattern=%d %p [%c] - %m%n
```

也就是让cn.com.Test中的logger使用log4j.appender.test所做的配置。

　　但是，如果在同一类中需要输出多个日志文件呢？其实道理是一样的，先在Test.java中定义：

```java
private static Log logger1 = LogFactory.getLog("myTest1");
private static Log logger2 = LogFactory.getLog("myTest2");
```

然后在log4j.properties中加入

```yaml
log4j.logger.myTest1= DEBUG, test1
log4j.appender.test1=org.apache.log4j.FileAppender
log4j.appender.test1.File=${myweb.root}/WEB-INF/log/test1.log
log4j.appender.test1.layout=org.apache.log4j.PatternLayout
log4j.appender.test1.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.logger.myTest2= DEBUG, test2
log4j.appender.test2=org.apache.log4j.FileAppender
log4j.appender.test2.File=${myweb.root}/WEB-INF/log/test2.log
log4j.appender.test2.layout=org.apache.log4j.PatternLayout
log4j.appender.test2.layout.ConversionPattern=%d %p [%c] - %m%n
```

也就是在用logger时给它一个自定义的名字(如这里的”myTest1”)，然后在log4j.properties中做出相应配置即可。别忘了不同日志要使用不同的logger(如输出到test1.log的要用logger1.info(“abc”))。

　　还有一个问题，就是这些自定义的日志默认是同时输出到log4j.rootLogger所配置的日志中的，如何能只让它们输出到自己指定的日志中呢？别急，这里有个开关：

　　log4j.additivity.myTest1 = false

　　它用来设置是否同时输出到log4j.rootLogger所配置的日志中，设为false就不会输出到其它地方啦！注意这里的”myTest1”是你在程序中给logger起的那个自定义的名字！

　　如果你说，我只是不想同时输出这个日志到log4j.rootLogger所配置的logfile中，stdout里我还想同时输出呢！那也好办，把你的log4j.logger.myTest1 = DEBUG, test1改为下式就OK啦！

　　log4j.logger.myTest1=DEBUG, test1

　　下面是文件上传时记录文件类型的log日志，并输出到指定文件的配置

```yaml
log4j.rootLogger=INFO, stdout
######################### logger ##############################
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.conversionPattern = %d [%t] %-5p %c - %m%n
log4j.logger.extProfile=INFO, extProfile # 日志级别是INFO,标签是extProfile
log4j.additivity.extProfile=false; # 输出到指定文件extProfile.log中
# userProfilelog\uff08\u8bb0\u5f55\u4fee\u6539\u5bc6\u7801\uff0c\u627e\u56de\u5bc6\u7801\uff0c\u4fee\u6539\u90ae\u7bb1\uff0c\u4fee\u6539\u624b\u673a\u53f7\uff09
log4j.appender.extProfile=org.apache.log4j.RollingFileAppender
log4j.appender.extProfile.File=logs/extProfile.log # 输出到resin根目录的logs文件夹,log4j会自动生成目录和文件
log4j.appender.extProfile.MaxFileSize=20480KB # 超过20M就重新创建一个文件
log4j.appender.extProfile.MaxBackupIndex=10
log4j.appender.extProfile.layout=org.apache.log4j.PatternLayout
log4j.appender.extProfile.layout.ConversionPattern=%d [%t] %-5p %c - %m%n
```



# 七、Logback

logback是由log4j创始人设计的又一个开源日志组件，作为流行的log4j项目的后续版本，从而替代log4j。

Springboot默认使用的日志框架是Logback。



　1、更快的执行速度：基于我们先前在Log4j上的工作，Logback 重写了内部的实现，在某些特定的场景上面，甚至可以比之前的速度快上10倍。在保证Logback的组件更加快速的同时，同时所需的内存更加少；

　　2、充分的[**测试**](javascript:;)：Logback 历经了几年，数不清小时数的测试。尽管Log4j也是测试过的，但是Logback的测试更加充分，跟Log4j不在同一个级别。我们认为，这正是人们选择Logback而不是Log4j的最重要的原因。



Logback的体系结构足够通用，以便在不同情况下应用。 目前，logback分为三个模块：logback-core，logback-classic和logback-access。

> logback-core：模块为其他两个模块的基础。
>
> logback-classic：模块可以被看做是log4j的改进版本。此外，logback-classic本身实现了SLF4J
> API，因此可以在logback和其他日志框架（如log4j或java.util.logging（JUL））之间来回切换。
>
> logback-access：模块与Servlet容器（如Tomcat和Jetty）集成，以提供HTTP访问日志功能。



logback-core是其它模块的基础设施，其它模块基于它构建，显然，logback-core提供了一些关键的通用机制。logback-classic的地位和作用等同于 Log4J，它也被认为是 Log4J的一个改进版，并且它实现了简单日志门面 SLF4J；而 logback-access主要作为一个与 Servlet容器交互的模块，比如说tomcat或者 jetty，提供一些与 HTTP访问相关的功能。

## 7.1 SpringBoot中的日志Logback



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

## 7.2 项目实例

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



