有些人可能会问，为什么我看到很多公司的api接口文档里面，都有/api/v1/ 这样的地址呢？其实，/api 就是为了和一般的业务地址区分，标明这个地址是api 的接口。v1 则代表版本号。

可能很多人又会问了，为什么要版本号呢？那么，接下来就聊一聊Restful 接口为什么要加版本号？ 如何优雅的设计 Restful API 接口版本号？

 

# 一、为什么加版本号

一般来说，api 接口是提供给其他系统或是其他公司使用，不能随意频繁的变更。然而，需求和业务不断变化，接口和参数也会发生相应的变化。如果直接对原来的接口进行修改，势必会影响线其他系统的正常运行。这就必须对api 接口进行有效的版本控制。

例如，添加用户的接口，由于业务需求变化，接口的字段属性也发生了变化而且可能和之前的功能不兼容。为了保证原有的接口调用方不受影响，只能重新定义一个新的接口。

- [http://localhost:8080/api/v1/user](http://localhost:8080/v1/my/test)
- [http://localhost:8080/api/v2/user](http://localhost:8080/v2/my/test)

 

Api 版本控制的方式：

　　1、域名区分管理，即不同的版本使用不同的域名，v1.api.test.com，v2.api.test.com

　　2、请求url 路径区分，在同一个域名下使用不同的url路径，test.com/api/v1/，test.com/api/v2

　　3、请求参数区分，在同一url路径下，增加version=v1或v2 等，然后根据不同的版本，选择执行不同的方法。

实际项目中，一般选择第二种：请求url路径区分。因为第二种既能保证水平扩展，有不影响以前的老版本。

# 二、Spring Boot如何实现

实现方案：

1、首先创建自定义的@APIVersion 注解和自定义URL匹配规则ApiVersionCondition。

2、然后创建自定义的 RequestMappingHandlerMapping 匹配对应的request，选择符合条件的method handler。

 

## 2.1 创建自定义注解

首先，在com.weiz.config 包下，创建一个自定义版本号标记注解 @ApiVersion。

```java
package com.weiz.config;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * API版本控制注解
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiVersion {
    /**
     * @return 版本号
     */
    int value() default 1;
}
```

说明：

```
　ApiVersion 为自定义的注解，API版本控制，返回对应的版本号。
```

 

## 2.2 自定义url匹配逻辑

创建 ApiVersionCondition 类，并继承RequestCondition 接口，作用是：版本号筛选，将提取请求URL中版本号，与注解上定义的版本号进行比对，以此来判断某个请求应落在哪个controller上。

在com.weiz.config 包下创建ApiVersionCondition 类，重写 RequestCondition，创建自定义的url匹配逻辑。

```java
package com.weiz.config;

import org.springframework.web.servlet.mvc.condition.RequestCondition;

import javax.servlet.http.HttpServletRequest;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class ApiVersionCondition implements RequestCondition<ApiVersionCondition> {
    private final static Pattern VERSION_PREFIX_PATTERN = Pattern.compile(".*v(\\d+).*");

    private int apiVersion;

    ApiVersionCondition(int apiVersion) {
        this.apiVersion = apiVersion;
    }

    private int getApiVersion() {
        return apiVersion;
    }


    @Override
    public ApiVersionCondition combine(ApiVersionCondition apiVersionCondition) {
        return new ApiVersionCondition(apiVersionCondition.getApiVersion());
    }

    @Override
    public ApiVersionCondition getMatchingCondition(HttpServletRequest httpServletRequest) {
        Matcher m = VERSION_PREFIX_PATTERN.matcher(httpServletRequest.getRequestURI());
        if (m.find()) {
            Integer version = Integer.valueOf(m.group(1));
            if (version >= this.apiVersion) {
                return this;
            }
        }
        return null;
    }

    @Override
    public int compareTo(ApiVersionCondition apiVersionCondition, HttpServletRequest httpServletRequest) {
        return apiVersionCondition.getApiVersion() - this.apiVersion;
    }
}
```

当方法级别和类级别都有ApiVersion注解时，二者将进行合并（ApiVersionRequestCondition.combine）。最终将提取请求URL中版本号，与注解上定义的版本号进行比对，判断url是否符合版本要求。

 

## 2.3 自定义匹配的处理器

在com.weiz.config 包下创建 ApiRequestMappingHandlerMapping 类，重写部分 RequestMappingHandlerMapping 的方法。

```java
package com.weiz.config;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.mvc.condition.RequestCondition;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import java.lang.reflect.Method;

public class ApiRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
    private static final String VERSION_FLAG = "{version}";

    private static RequestCondition<ApiVersionCondition> createCondition(Class<?> clazz) {
        RequestMapping classRequestMapping = clazz.getAnnotation(RequestMapping.class);
        if (classRequestMapping == null) {
            return null;
        }
        StringBuilder mappingUrlBuilder = new StringBuilder();
        if (classRequestMapping.value().length > 0) {
            mappingUrlBuilder.append(classRequestMapping.value()[0]);
        }
        String mappingUrl = mappingUrlBuilder.toString();
        if (!mappingUrl.contains(VERSION_FLAG)) {
            return null;
        }
        ApiVersion apiVersion = clazz.getAnnotation(ApiVersion.class);
        return apiVersion == null ? new ApiVersionCondition(1) : new ApiVersionCondition(apiVersion.value());
    }

    @Override
    protected RequestCondition<?> getCustomMethodCondition(Method method) {
        return createCondition(method.getClass());
    }

    @Override
    protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
        return createCondition(handlerType);
    }
}
```

 

## 2.4 配置注册自定义的RequestMappingHandlerMapping

重写请求过处理的方法，将之前创建的 ApiRequestMappingHandlerMapping 注册到系统中。

```java
package com.weiz.config;

import org.springframework.boot.autoconfigure.web.servlet.WebMvcRegistrations;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

@Configuration
public class WebMvcRegistrationsConfig implements WebMvcRegistrations {
    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new ApiRequestMappingHandlerMapping();
    }
}
```

上面四步，把api 版本控制配置完了。代码看着复杂，其实都是重写spring boot 内部的处理流程。

 

# 三、测试

配置完成之后，接下来编写测试的控制器进行测试。

## 3.1 在Controller/api 目录下，分别创建UserV1Controller 和 UserV2Controller

UserV1Controller

```java
@RequestMapping("api/{version}/user")
@RestController
public class UserV1Controller {

    @GetMapping("/test")
    public String test() {
        return "version1";
    }
    @GetMapping("/extend")
    public String extendTest() {
        return "user v1 extend";
    }
}
```

 

UserV2Controller

```java
@RequestMapping("api/{version}/user")
@RestController
@ApiVersion(2)
public class UserV2Controller {
    @GetMapping("/test")
    public String test() {
        return "user v2 test";
    }
}
```

 

## 3.2 启动项目后，输入相关地址，查看版本控制是否生效

测试结果：

正确的接口地址

![img](https://img2020.cnblogs.com/blog/306976/202006/306976-20200623192820754-249290296.png) *![img](https://img2020.cnblogs.com/blog/306976/202006/306976-20200623192738062-499033746.png)*

 

继承的接口地址

![img](https://img2020.cnblogs.com/blog/306976/202006/306976-20200623193256023-911402799.png)

说明：

　　上图的前两个截图说明，请求正确的版本地址，会自动匹配版本的对应接口。当请求的版本大于当前版本时，默认匹配当前版本。

　　第三个截图说明，当请求对应的版本不存在接口时，会匹配之前版本的接口，即请求/v2/user/extend 接口时，由于v2 控制器未实现该接口，所以自动匹配v1 版本中的接口。这就是所谓的版本继承。