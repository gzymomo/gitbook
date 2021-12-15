[TOC]

# 1、拦截器简介
拦截器，请求的接口被访问之前，进行拦截然后在之前或之后加入某些操作。拦截是AOP的一种实现策略。 拦截器主要用来按照指定规则拒绝请求。

## 1.1 拦截器中应用
- Token令牌验证
- 请求数据校验
- 用户权限校验
- 放行指定接口

# 2、拦截器用法
## 2.1 编写两个拦截器
自定义类实现HandlerInterceptor接口
### 2.1.1 OneInterceptor 拦截器
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 拦截器一
 */
public class OneInterceptor implements HandlerInterceptor {
   private static final Logger LOGGER = LoggerFactory.getLogger(OneInterceptor.class.getName());
   @Override
   public boolean preHandle(HttpServletRequest request,HttpServletResponse response, Object o) throws Exception {
     String url =String.valueOf(request.getRequestURL()) ;
     LOGGER.info("1、url=="+url);
     // 放开拦截
     return true;
 }

 @Override
 public void postHandle(HttpServletRequest httpServletRequest,
   HttpServletResponse httpServletResponse,
   Object o, ModelAndView modelAndView) throws Exception {
   LOGGER.info("1、postHandle");
 }
 @Override
 public void afterCompletion(HttpServletRequest httpServletRequest,
   HttpServletResponse httpServletResponse,
   Object o, Exception e) throws Exception {
   LOGGER.info("1、afterCompletion");
 }
}
```
### 2.1.2 TwoInterceptor 拦截器
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 拦截器二
 */
public class TwoInterceptor implements HandlerInterceptor {
   private static final Logger LOGGER = LoggerFactory.getLogger(TwoInterceptor.class.getName());
   @Override
   public boolean preHandle(HttpServletRequest request,HttpServletResponse response, Object o) throws Exception {
    String url =String.valueOf(request.getRequestURL()) ;
    LOGGER.info("2、url=="+url);
    // 放开拦截
    return true;
 }
 @Override
 public void postHandle(HttpServletRequest httpServletRequest,HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
     LOGGER.info("2、postHandle");
 }

 @Override
 public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    LOGGER.info("2、afterCompletion");
 }
}
```
## 2.2 Web配置文件中注入拦截器
```java
import com.boot.intercept.intercept.OneInterceptor;
import com.boot.intercept.intercept.TwoInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
/**
 * Web配置文件
 */
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
 public void addInterceptors(InterceptorRegistry registry) {
 // 拦截所有路径
 // 注册自定义两个拦截器
 registry.addInterceptor(new OneInterceptor()).addPathPatterns("/**");
 registry.addInterceptor(new TwoInterceptor()).addPathPatterns("/**");
 }
}
```

## 2.3 编写测试接口
```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class InterceptController {
 @RequestMapping("/reqUrl")
 public String reqUrl (){
 return "success" ;
 }
}
```

## 2.4 访问测试接口
日志输出内容如下
```java
intercept.OneInterceptor : 1、url==http://127.0.0.1:8005/reqUrl
intercept.TwoInterceptor : 2、url==http://127.0.0.1:8005/reqUrl
intercept.TwoInterceptor : 2、postHandle
intercept.OneInterceptor : 1、postHandle
intercept.TwoInterceptor : 2、afterCompletion
intercept.OneInterceptor : 1、afterCompletionla
```
拦截器的拦截顺序，是按照Web配置文件中注入拦截器的顺序执行的。