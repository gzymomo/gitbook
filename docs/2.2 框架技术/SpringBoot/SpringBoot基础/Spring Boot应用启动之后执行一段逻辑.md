[TOC]

[码农小胖哥:如何在Spring Boot应用启动之后立刻执行一段逻辑](https://www.cnblogs.com/felordcn/p/13029558.html)

项目启动后立马执行一些逻辑。比如简单的缓存预热，或者上线后的广播之类等等。如果你使用 Spring Boot 框架的话就可以借助其提供的接口CommandLineRunner和 ApplicationRunner来实现。

# 1、CommandLineRunner
org.springframework.boot.CommandLineRunner 是Spring Boot提供的一个接口，当你实现该接口并将之注入Spring IoC容器后，Spring Boot应用启动后就会执行其run方法。一个Spring Boot可以存在多个CommandLineRunner的实现，当存在多个时，你可以实现Ordered接口控制这些实现的执行顺序(Order 数值越大优先级越低)。接下来我们来声明两个实现并指定顺序：

优先执行：
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;

/**
 * 优先级最高
 * 该类期望在springboot 启动后第一顺位执行
 **/
@Slf4j
@Component
public class HighOrderCommandLineRunner implements CommandLineRunner, Ordered {
    @Override
    public void run(String... args) throws Exception {
        for (String arg : args) {
            log.info("arg = " + arg);
        }
        log.info("i am highOrderRunner");
    }

    @Override
    public int getOrder() {
        return Integer.MIN_VALUE+1;
    }
}
```
第二顺序执行：
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;

/**
 * 优先级低于{@code HighOrderCommandLineRunner}
 **/
@Slf4j
@Component
public class LowOrderCommandLineRunner implements CommandLineRunner, Ordered {

    @Override
    public void run(String... args) throws Exception {
        log.info("i am lowOrderRunner");
    }

    @Override
    public int getOrder() {
        return Integer.MIN_VALUE+1;
    }
}
```
启动Spring Boot应用后,控制台按照预定的顺序打印出了结果：
```java
2020-05-30 23:11:03.685  INFO 11976 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-05-30 23:11:03.701  INFO 11976 --- [           main] c.f.Application  : Started SpringBootApplication in 4.272 seconds (JVM running for 6.316)
2020-05-30 23:11:03.706  INFO 11976 --- [           main] c.f.HighOrderCommandLineRunner   : i am highOrderRunner
2020-05-30 23:11:03.706  INFO 11976 --- [           main] c.f.LowOrderCommandLineRunner   : i am lowOrderRunner
```

# 2、ApplicationRunner
在Spring Boot 1.3.0又引入了一个和CommandLineRunner功能一样的接口ApplicationRunner。CommandLineRunner接收可变参数String... args，而ApplicationRunner 接收一个封装好的对象参数ApplicationArguments。除此之外它们功能完全一样，甚至连方法名都一样。 声明一个ApplicationRunner并让它优先级最低:
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.List;
import java.util.Set;

/**
 * 优先级最低
 **/
@Slf4j
@Component
public class DefaultApplicationRunner implements ApplicationRunner, Ordered {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("i am applicationRunner");
        Set<String> optionNames = args.getOptionNames();
        log.info("optionNames = " + optionNames);
        String[] sourceArgs = args.getSourceArgs();
        log.info("sourceArgs = " + Arrays.toString(sourceArgs));
        List<String> nonOptionArgs = args.getNonOptionArgs();
        log.info("nonOptionArgs = " + nonOptionArgs);
        List<String> optionValues = args.getOptionValues("foo");
        log.info("optionValues = " + optionValues);
    }

    @Override
    public int getOrder() {
        return Integer.MIN_VALUE+2;
    }
}
```
按照顺序打印了三个类的执行结果：
```java
2020-06-01 13:02:39.420  INFO 19032 --- [           main] c.f.MybatisResultmapApplication  : Started MybatisResultmapApplication in 1.801 seconds (JVM running for 2.266)
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.HighOrderCommandLineRunner   : i am highOrderRunner
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.LowOrderCommandLineRunner    : i am lowOrderRunner
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.DefaultApplicationRunner     : i am applicationRunner
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.DefaultApplicationRunner     : optionNames = []
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.DefaultApplicationRunner     : sourceArgs = []
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.DefaultApplicationRunner     : nonOptionArgs = []
2020-06-01 13:02:39.423  INFO 19032 --- [           main] c.f.DefaultApplicationRunner     : optionValues = null
```
**Ordered接口并不能被 @Order注解所代替。**

# 3、传递参数
Spring Boot应用启动时是可以接受参数的，换句话说也就是Spring Boot的main方法是可以接受参数的。这些参数通过命令行 java -jar yourapp.jar 来传递。CommandLineRunner会原封不动照单全收这些接口，这些参数也可以封装到ApplicationArguments对象中供ApplicationRunner调用。 我们来认识一下ApplicationArguments的相关方法：

- getSourceArgs() 被传递给应用程序的原始参数，返回这些参数的字符串数组。
- getOptionNames() 获取选项名称的Set字符串集合。如 --spring.profiles.active=dev --debug 将返回["spring.profiles.active","debug"] 。
- getOptionValues(String name) 通过名称来获取该名称对应的选项值。如--foo=bar --foo=baz 将返回["bar","baz"]。
- containsOption(String name) 用来判断是否包含某个选项的名称。
- getNonOptionArgs() 用来获取所有的无选项参数。

接下来我们试验一波，你可以通过下面的命令运行一个 Spring Boot应用 Jar
`java -jar yourapp.jar --foo=bar --foo=baz --dev.name=码农小胖哥 java felordcn`
或者在IDEA开发工具中打开Spring Boot应用main方法的配置项，进行如下配置，其他IDE工具同理。

![](https://img2020.cnblogs.com/other/1739473/202006/1739473-20200602100615082-78938241.png)
