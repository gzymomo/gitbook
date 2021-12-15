[TOC]

# 1、模板热部署
在 Spring Boot 中，模板引擎的页面默认是开启缓存的，如果修改了页面的内容，则刷新页面是得不到修改后的页面的，因此我们可以在application.properties中关闭模版引擎的缓存，如下：

1. Thymeleaf的配置：
  -- spring.thymeleaf.cache=false
2. FreeMarker的配置：
  -- spring.freemarker.cache=false
3. Groovy的配置：
  -- spring.groovy.template.cache=false
4. Velocity的配置：
  -- spring.velocity.cache=false

# 2、spring-boot-devtools
在 Spring Boot 项目中添加 spring-boot-devtools依赖即可实现页面和代码的热部署。
如下：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
</dependency>
```
此种方式的特点是作用范围广，系统的任何变动包括配置文件修改、方法名称变化都能覆盖，但是后遗症也非常明显，它是采用文件变化后重启的策略来实现了，主要是节省了我们手动点击重启的时间，提高了实效性，在体验上会稍差。

spring-boot-devtools 默认关闭了模版缓存，如果使用这种方式不用单独配置关闭模版缓存。