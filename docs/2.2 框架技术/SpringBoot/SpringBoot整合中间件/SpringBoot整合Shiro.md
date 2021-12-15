[TOC]

[江南一点雨-SpringBoot2 教程合集](http://springboot.javaboy.org/2019/0611/springboot-shiro)

在 Spring Boot 中整合 Shiro ，有两种不同的方案：

1. 第一种就是原封不动的，将 SSM 整合 Shiro 的配置用 Java 重写一遍。
2. 第二种就是使用 Shiro 官方提供的一个 Starter 来配置，但是，这个 Starter 并没有简化多少配置。

# 1、原生的整个
## 1.1 创建项目
创建一个 Spring Boot 项目，只需要添加 Web 依赖即可。

项目创建成功后，加入 Shiro 相关的依赖，完整的 pom.xml 文件中的依赖如下：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.4.0</version>
    </dependency>
</dependencies>
```
## 1.2 创建Realm
自定义核心组件 Realm：
```java
public class MyRealm extends AuthorizingRealm {
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = (String) token.getPrincipal();
        if (!"javaboy".equals(username)) {
            throw new UnknownAccountException("账户不存在!");
        }
        return new SimpleAuthenticationInfo(username, "123", getName());
    }
}
```

在 Realm 中实现简单的认证操作即可，不做授权，授权的具体写法和 SSM 中的 Shiro 一样，不赘述。这里的认证表示用户名必须是 javaboy ，用户密码必须是 123 ，满足这样的条件，就能登录成功！

## 1.3 配置shiro

```java
@Configuration
public class ShiroConfig {
    @Bean
    MyRealm myRealm() {
        return new MyRealm();
    }
    
    @Bean
    SecurityManager securityManager() {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm());
        return manager;
    }
    
    @Bean
    ShiroFilterFactoryBean shiroFilterFactoryBean() {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(securityManager());
        bean.setLoginUrl("/login");
        bean.setSuccessUrl("/index");
        bean.setUnauthorizedUrl("/unauthorizedurl");
        Map<String, String> map = new LinkedHashMap<>();
        map.put("/doLogin", "anon");
        map.put("/**", "authc");
        bean.setFilterChainDefinitionMap(map);
        return bean;
    }
}
```
在这里进行 Shiro 的配置主要配置 3 个 Bean ：

1. 首先需要提供一个 Realm 的实例。
2. 需要配置一个 SecurityManager，在 SecurityManager 中配置 Realm。
3. 配置一个 ShiroFilterFactoryBean ，在 ShiroFilterFactoryBean 中指定路径拦截规则等。
4. 配置登录和测试接口。

其中，ShiroFilterFactoryBean 的配置稍微多一些，配置含义如下：

1. setSecurityManager 表示指定 SecurityManager。
2. setLoginUrl 表示指定登录页面。
3. setSuccessUrl 表示指定登录成功页面。
4. 接下来的 Map 中配置了路径拦截规则，注意，要有序。

这些东西都配置完成后，接下来配置登录 Controller:

```java
@RestController
public class LoginController {
    @PostMapping("/doLogin")
    public void doLogin(String username, String password) {
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(new UsernamePasswordToken(username, password));
            System.out.println("登录成功!");
        } catch (AuthenticationException e) {
            e.printStackTrace();
            System.out.println("登录失败!");
        }
    }
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
    @GetMapping("/login")
    public String  login() {
        return "please login!";
    }
}
```

测试时，首先访问 /hello 接口，由于未登录，所以会自动跳转到 /login 接口：
![](http://www.javaboy.org/images/boot/8-2.png)

然后调用 /doLogin 接口完成登录：
![](http://www.javaboy.org/images/boot/8-3.png)

再次访问 /hello 接口，就可以成功访问了.

# 2、使用Shiro Starter
## 2.1 项目创建
创建成功后，添加 shiro-spring-boot-web-starter ，这个依赖可以代替之前的 shiro-web 和 shiro-spring 两个依赖，pom.xml 文件如下：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring-boot-web-starter</artifactId>
        <version>1.4.0</version>
    </dependency>
</dependencies>
```

## 2.2 创建Realm
同上！

## 2.3 配置Shiro
在 application.properties 中配置 Shiro 的基本信息：
```
shiro.sessionManager.sessionIdCookieEnabled=true
shiro.sessionManager.sessionIdUrlRewritingEnabled=true
shiro.unauthorizedUrl=/unauthorizedurl
shiro.web.enabled=true
shiro.successUrl=/index
shiro.loginUrl=/login
```

配置解释：

1. 第一行表示是否允许将sessionId 放到 cookie 中
2. 第二行表示是否允许将 sessionId 放到 Url 地址拦中
3. 第三行表示访问未获授权的页面时，默认的跳转路径
4. 第四行表示开启 shiro
5. 第五行表示登录成功的跳转页面
6. 第六行表示登录页面

## 2.4 配置 ShiroConfig
```java
@Configuration
public class ShiroConfig {
    @Bean
    MyRealm myRealm() {
        return new MyRealm();
    }
    @Bean
    DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm());
        return manager;
    }
    @Bean
    ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition definition = new DefaultShiroFilterChainDefinition();
        definition.addPathDefinition("/doLogin", "anon");
        definition.addPathDefinition("/**", "authc");
        return definition;
    }
}
```