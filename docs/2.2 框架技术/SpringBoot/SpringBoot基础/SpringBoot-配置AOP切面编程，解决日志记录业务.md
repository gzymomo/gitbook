[TOC]

# 1、AOP切面编程
在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP（面向对象编程）的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

## 1.1 AOP编程特点
1. AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码
2. 经典应用：事务管理、性能监视、安全检查、缓存 、日志等
3. aop底层将采用代理机制进行实现
4. 接口 + 实现类 ：spring采用 jdk 的动态代理Proxy
5. 实现类：spring 采用 cglib字节码增强

## 1.2 AOP中术语和图解
1. target：目标类
   需要被代理的类。例如：UserService
2. Joinpoint：连接点
   所谓连接点是指那些可能被拦截到的方法。例如：所有的方法
3. PointCut：切入点
   已经被增强的连接点。例如：addUser()
4. advice：通知/增强
   增强代码。例如：after、before
5. Weaving：织入
   指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.
6. proxy 代理类
7. Aspect(切面): 是切入点pointcut和通知advice的结合
    一个线是一个特殊的面。
    一个切入点和一个通知，组成成一个特殊的面。

![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvDU0tpQfB5hiaTibhwzF7hVDg5Z0cDr0sFrzVFFmjqIJmKaK09Th7fB7l3YrGvWh4puYo8XU5G2XzsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 2、SpringBoot整合AOP
## 2.1 核心依赖
```xml
<!-- AOP依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
## 2.2 编写日志记录注解
```java
package com.boot.aop.config;
import java.lang.annotation.*;
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LogFilter {
    String value() default "" ;
}
```
## 2.3 编写日志记录的切面代码
这里分为两种情况处理，一种正常的请求日志，和系统异常的错误日志。
核心注解两个。@Aspect和@Component。
```java
package com.boot.aop.config;
import com.alibaba.fastjson.JSONObject;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;

@Aspect
@Component
public class LogAspect {

    private static final Logger LOGGER = LoggerFactory.getLogger(LogAspect.class) ;

    @Pointcut("@annotation(com.boot.aop.config.LogFilter)")
    public void logPointCut (){

    }
    @Around("logPointCut()")
    public Object around (ProceedingJoinPoint point) throws Throwable {
        Object result = null ;
        try{
            // 执行方法
            result = point.proceed();
            // 保存请求日志
            saveRequestLog(point);
        } catch (Exception e){
            // 保存异常日志
            saveExceptionLog(point,e.getMessage());
        }
        return result;
    }
    private void saveExceptionLog (ProceedingJoinPoint point,String exeMsg){
        LOGGER.info("捕获异常:"+exeMsg);
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        LOGGER.info("请求路径:"+request.getRequestURL());
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        LOGGER.info("请求方法:"+method.getName());
        // 获取方法上LogFilter注解
        LogFilter logFilter = method.getAnnotation(LogFilter.class);
        String value = logFilter.value() ;
        LOGGER.info("模块描述:"+value);
        Object[] args = point.getArgs();
        LOGGER.info("请求参数:"+ JSONObject.toJSONString(args));
    }
    private void saveRequestLog (ProceedingJoinPoint point){
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        LOGGER.info("请求路径:"+request.getRequestURL());
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        LOGGER.info("请求方法:"+method.getName());
        // 获取方法上LogFilter注解
        LogFilter logFilter = method.getAnnotation(LogFilter.class);
        String value = logFilter.value() ;
        LOGGER.info("模块描述:"+value);
        Object[] args = point.getArgs();
        LOGGER.info("请求参数:"+ JSONObject.toJSONString(args));
    }
}
```
## 2.4 请求日志测试
```java
@LogFilter("保存请求日志")
@RequestMapping("/saveRequestLog")
public String saveRequestLog (@RequestParam("name") String name){
    return "success："+name ;
}
```
切面类信息打印
```java
/**
 * 请求路径:http://localhost:8011/saveRequestLog
 * 请求方法:saveRequestLog
 * 模块描述:保存请求日志
 * 请求参数:["cicada"]
 */
```
## 2.5 异常日志测试
```java
@LogFilter("保存异常日志")
@RequestMapping("/saveExceptionLog")
public String saveExceptionLog (@RequestParam("name") String name){
    int error = 100 / 0 ;
    System.out.println(error);
    return "success："+name ;
}
```
切面类信息打印
```java
/**
 * 捕获异常:/ by zero
 * 请求路径:http://localhost:8011/saveExceptionLog
 * 请求方法:saveExceptionLog
 * 模块描述:保存异常日志
 * 请求参数:["cicada"]
 */
```