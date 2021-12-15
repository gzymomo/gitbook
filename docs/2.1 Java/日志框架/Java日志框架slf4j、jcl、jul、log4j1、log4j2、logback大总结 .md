- [Java日志框架slf4j、jcl、jul、log4j1、log4j2、logback大总结](https://juejin.cn/post/6942481722856603655)



# 1 系列目录

- [jdk-logging、log4j、logback日志介绍及原理](http://my.oschina.net/pingpangkuangmo/blog/406618)
- [commons-logging与jdk-logging、log4j1、log4j2、logback的集成原理](http://my.oschina.net/pingpangkuangmo/blog/407895)
- [slf4j与jdk-logging、log4j1、log4j2、logback的集成原理](http://my.oschina.net/pingpangkuangmo/blog/408382)
- [slf4j、jcl、jul、log4j1、log4j2、logback大总结](http://my.oschina.net/pingpangkuangmo/blog/410224)

# 2各种jar包总结

- log4j1:

  - log4j：log4j1的全部内容

- log4j2:

  - log4j-api:log4j2定义的API
  - log4j-core:log4j2上述API的实现

- logback:

  - logback-core:logback的核心包
  - logback-classic：logback实现了slf4j的API

- commons-logging:

  - commons-logging:commons-logging的原生全部内容
  - log4j-jcl:commons-logging到log4j2的桥梁
  - jcl-over-slf4j：commons-logging到slf4j的桥梁

- slf4j转向某个实际的日志框架:

  场景介绍：如 使用slf4j的API进行编程，底层想使用log4j1来进行实际的日志输出，这就是slf4j-log4j12干的事。

  - slf4j-jdk14：slf4j到jdk-logging的桥梁
  - slf4j-log4j12：slf4j到log4j1的桥梁
  - log4j-slf4j-impl：slf4j到log4j2的桥梁
  - logback-classic：slf4j到logback的桥梁
  - slf4j-jcl：slf4j到commons-logging的桥梁

- 某个实际的日志框架转向slf4j：

  场景介绍：如 使用log4j1的API进行编程，但是想最终通过logback来进行输出，所以就需要先将log4j1的日志输出转交给slf4j来输出，slf4j 再交给logback来输出。将log4j1的输出转给slf4j，这就是log4j-over-slf4j做的事

  这一部分主要用来进行实际的日志框架之间的切换（下文会详细讲解）

  - jul-to-slf4j：jdk-logging到slf4j的桥梁
  - log4j-over-slf4j：log4j1到slf4j的桥梁
  - jcl-over-slf4j：commons-logging到slf4j的桥梁

# 3集成总结

## 3.1 commons-logging与其他日志框架集成

- 1 commons-logging与jdk-logging集成：

  需要的jar包：

  - commons-logging

- 2 commons-logging与log4j1集成：

  需要的jar包：

  - commons-logging
  - log4j

- 3 commons-logging与log4j2集成：

  需要的jar包：

  - commons-logging
  - log4j-api
  - log4j-core
  - log4j-jcl(集成包)

- 4 commons-logging与logback集成：

  需要的jar包：

  - logback-core
  - logback-classic
  - slf4j-api、jcl-over-slf4j(2个集成包,可以不再需要commons-logging)

- 5 commons-logging与slf4j集成：

  需要的jar包：

  - jcl-over-slf4j(集成包，不再需要commons-logging)
  - slf4j-api

## 3.2 slf4j与其他日志框架集成

- slf4j与jdk-logging集成：

  需要的jar包：

  - slf4j-api
  - slf4j-jdk14(集成包)

- slf4j与log4j1集成：

  需要的jar包：

  - slf4j-api
  - log4j
  - slf4j-log4j12(集成包)

- slf4j与log4j2集成：

  需要的jar包：

  - slf4j-api
  - log4j-api
  - log4j-core
  - log4j-slf4j-impl(集成包)

- slf4j与logback集成：

  需要的jar包：

  - slf4j-api
  - logback-core
  - logback-classic(集成包)

- slf4j与commons-logging集成：

  需要的jar包：

  - slf4j-api
  - commons-logging
  - slf4j-jcl(集成包)

# 4 日志系统之间的切换

## 4.1 log4j无缝切换到logback

### 4.1.1 案例

我们已经在代码中使用了log4j1的API来进行日志的输出，现在想不更改已有代码的前提下，使之通过logback来进行实际的日志输出。

已使用的jar包：

- log4j

使用案例：

```
private static final Logger logger=Logger.getLogger(Log4jTest.class);

public static void main(String[] args){
    if(logger.isInfoEnabled()){
        logger.info("log4j info message");
    }
}
复制代码
```

上述的Logger是log4j1自己的org.apache.log4j.Logger，在上述代码中，我们在使用log4j1的API进行编程

现在如何能让上述的日志输出通过logback来进行输出呢？

只需要更换一下jar包就可以：

- 第一步：去掉log4j jar包
- 第二步：加入以下jar包
  - log4j-over-slf4j（实现log4j1切换到slf4j）
  - slf4j-api
  - logback-core
  - logback-classic
- 第三步：在类路径下加入logback的配置文件

原理是什么呢？

### 4.1.2 切换原理

看下log4j-over-slf4j就一目了然了：

[![Java日志框架slf4j、jcl、jul、log4j1、log4j2、logback大总结](data:image/svg+xml;utf8,)](http://static.oschina.net/uploads/space/2015/0429/202452_8hEO_2287728.png)

我们可以看到，这里面其实是简化更改版的log4j。去掉log4j1的原生jar包，换成该简化更改版的jar包（可以实现无缝迁移）。

但是简化更改版中的Logger和原生版中的实现就不同了，简化版中的Logger实现如下（继承了Category）：

```
public class Category {
    private String name;
    protected org.slf4j.Logger slf4jLogger;
    private org.slf4j.spi.LocationAwareLogger locationAwareLogger;

    Category(String name) {
        this.name = name;
        slf4jLogger = LoggerFactory.getLogger(name);
        if (slf4jLogger instanceof LocationAwareLogger) {
            locationAwareLogger = (LocationAwareLogger) slf4jLogger;
        }
    }
}
复制代码
```

从上面可以看到简化版中的Logger内部是使用slf4j的API来生成的，所以我们使用的简化版的Logger会委托给slf4j来进行输出， 由于当前类路径下有logback-classic，所以slf4j会选择logback进行输出。从而实现了log4j到logback的日志切换。

下面的内容就只讲解日志系统到slf4j的切换，不再讲解slf4j选择何种日志来输出

## 4.2 jdk-logging无缝切换到logback

### 4.2.1 案例

```
private static final Logger logger=Logger.getLogger(JulSlf4jLog4jTest.class.getName());

public static void main(String[] args){
    logger.log(Level.INFO,"jul info a msg");
    logger.log(Level.WARNING,"jul waring a msg");
}
复制代码
```

可以看到上述是使用jdk-logging自带的API来进行编程的，现在我们想这些日志交给logback来输出

解决办法如下：

- 第一步：加入以下jar包：

  - jul-to-slf4j （实现jdk-logging切换到slf4j）
  - slf4j-api
  - logback-core
  - logback-classic

- 第二步：在类路径下加入logback的配置文件

- 第三步：在代码中加入如下代码：

  ```
  static{
      SLF4JBridgeHandler.install();
  }
  复制代码
  ```

### 4.2.2 切换原理

先来看下jul-to-slf4j jar包中的内容：

[![Java日志框架slf4j、jcl、jul、log4j1、log4j2、logback大总结](data:image/svg+xml;utf8,)](http://static.oschina.net/uploads/space/2015/0429/205834_VT4Z_2287728.png)

我们看到只有一个类：SLF4JBridgeHandler

它继承了jdk-logging中定义的java.util.logging.Handler，Handler是jdk-logging处理日志过 程中的一个处理器（具体我也没仔细研究过），在使用之前，必须要提前注册这个处理器，即上述的SLF4JBridgeHandler.install() 操作，install后我们就可以通过这个handler实现日志的切换工作，如下：

```
protected Logger getSLF4JLogger(LogRecord record) {
    String name = record.getLoggerName();
    if (name == null) {
        name = UNKNOWN_LOGGER_NAME;
    }
    return LoggerFactory.getLogger(name);
}
复制代码
```

在处理日志的过程中，使用了slf4j的原生方式LoggerFactory来获取一个slf4j定义的Logger来进行日志的输出

而slf4j则又会选择logback来进行实际的日志输出

## 4.3 commons-logging切换到logback

### 4.3.1 使用案例

使用的jar包

- commons-logging

案例如下：

```
private static Log logger=LogFactory.getLog(JulJclTest.class);

public static void main(String[] args){
    if(logger.isTraceEnabled()){
        logger.trace("commons-logging-jcl trace message");
    }
}
复制代码
```

可以看到我们使用commons-logging的API来进行日志的编程操作，现在想切换成logback来进行日志的输出（这其实就是commons-logging与logback的集成）

解决办法如下：

- 第一步：去掉commons-logging jar包（其实去不去都无所谓）
- 第二步：加入以下jar包：
  - jcl-over-slf4j（实现commons-logging切换到slf4j）
  - slf4j-api
  - logback-core
  - logback-classic
- 第三步：在类路径下加入logback的配置文件

### 4.3.2 切换原理

这个原理之前都已经说过了，可以看下[commons-logging与logback的集成](http://my.oschina.net/pingpangkuangmo/blog/407895#OSC_h1_17)

就是commons-logging通过jcl-over-slf4j 来选择slf4j作为底层的日志输出对象，而slf4j又选择logback来作为底层的日志输出对象。

## 4.4 常用的日志场景切换解释

上面把日志的切换原理说清楚了，下面就针对具体的例子来进行应用

先来看下slf4j官方的一张图：

![Java日志框架slf4j、jcl、jul、log4j1、log4j2、logback大总结](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="800" height="600"></svg>)

下面分别详细说明这三个案例

### 4.4.1 左上图

- 现状：

  目前的应用程序中已经使用了如下混杂方式的API来进行日志的编程：

  - commons-logging
  - log4j1
  - jdk-logging

  现在想统一将日志的输出交给logback

- 解决办法：

  - 第一步：将上述日志系统全部无缝先切换到slf4j
    - 去掉commons-logging（其实去不去都可以），使用jcl-over-slf4j将commons-logging的底层日志输出切换到slf4j
    - 去掉log4j1(必须去掉),使用log4j-over-slf4j,将log4j1的日志输出切换到slf4j
    - 使用jul-to-slf4j，将jul的日志输出切换到slf4j
  - 第二步：使slf4j选择logback来作为底层日志输出

  加入以下jar包：

  - slf4j-api
  - logback-core
  - logback-classic

下面的2张图和上面就很类似

### 4.4.2 右上图

- 现状：

  目前的应用程序中已经使用了如下混杂方式的API来进行日志的编程：

  - commons-logging
  - jdk-logging

  现在想统一将日志的输出交给log4j1

- 解决办法：

  - 第一步：将上述日志系统全部无缝先切换到slf4j
    - 去掉commons-logging（其实去不去都可以），使用jcl-over-slf4j将commons-logging的底层日志输出切换到slf4j
    - 使用jul-to-slf4j，将jul的日志输出切换到slf4j
  - 第二步：使slf4j选择log4j1来作为底层日志输出

  加入以下jar包：

  - slf4j-api
  - log4j
  - slf4j-log4j12(集成包)

### 4.4.3 左下图

- 现状：

  目前的应用程序中已经使用了如下混杂方式的API来进行日志的编程：

  - commons-logging
  - log4j

  现在想统一将日志的输出交给jdk-logging

- 解决办法：

  - 第一步：将上述日志系统全部无缝先切换到slf4j
    - 去掉commons-logging（其实去不去都可以），使用jcl-over-slf4j将commons-logging的底层日志输出切换到slf4j
    - 去掉log4j1(必须去掉),使用log4j-over-slf4j,将log4j1的日志输出切换到slf4j
  - 第二步：使slf4j选择jdk-logging来作为底层日志输出

  加入以下jar包：

  - slf4j-api
  - slf4j-jdk14(集成包)

# 5 冲突说明

仍然是这里的内容[slf4j官网的冲突说明](http://www.slf4j.org/legacy.html)

其实明白上面介绍的各jar包的作用，就很容易理解

## 5.1 jcl-over-slf4j 与 slf4j-jcl 冲突

- jcl-over-slf4j： commons-logging切换到slf4j
- slf4j-jcl : slf4j切换到commons-logging

如果这两者共存的话，必然造成相互委托，造成内存溢出

## 5.2 log4j-over-slf4j 与 slf4j-log4j12 冲突

- log4j-over-slf4j ： log4j1切换到slf4j
- slf4j-log4j12 : slf4j切换到log4j1

如果这两者共存的话，必然造成相互委托，造成内存溢出。但是log4j-over-slf4内部做了一个判断，可以防止造成内存溢出：

即判断slf4j-log4j12 jar包中的org.slf4j.impl.Log4jLoggerFactory是否存在，如果存在则表示冲突了，抛出异常提示用户要去掉对应的jar 包，代码如下，在slf4j-log4j12 jar包的org.apache.log4j.Log4jLoggerFactory中：

![Java日志框架slf4j、jcl、jul、log4j1、log4j2、logback大总结](http://static.open-open.com/lib/uploadImg/20150504/20150504085433_820.png)

## 5.3 jul-to-slf4j 与 slf4j-jdk14 冲突

- jul-to-slf4j ： jdk-logging切换到slf4j
- slf4j-jdk14 : slf4j切换到jdk-logging

如果这两者共存的话，必然造成相互委托，造成内存溢出