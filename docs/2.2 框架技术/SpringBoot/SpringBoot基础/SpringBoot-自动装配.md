[TOC]

整理自：
博客园：龙四丶：[SpringBoot系列](https://www.cnblogs.com/loongk/tag/SpringBoot/)

# 1、SpringBoot自动装配
SpringBoot 中运用了大量的 Spring 注解，其注解大致分为这几类：
1. 配置注解：@Configuration、@ComponentScan、@Import、@Conditional、Bean
2. 模式注解：@Componnt、@Repository、@Service、@Controller
3. @Enable 模块注解：@EnableWebMvc、@EnableTransactionManagement、@EnableWebFlux

模式注解都标注了 @Component 注解，属于 @Component 的派生注解，@ComponentScan 会扫描标注 @Component 及其派生注解的类，并将这些类加入到 Spring 容器中；@Enable 模块注解中通过 @Import 导入配置类，在这些配置类中加载 @Enable 模块需要的组件。

模式注解是一种用于声明在应用中扮演“组件”角色的注解。如 Spring 中的 @Repository 是用于扮演仓储角色的模式注解，用来管理和存储某种领域对象。还有如@Component 是通用组件模式、@Service 是服务模式、@Configuration 是配置模式等。其中@Component 作为一种由 Spring 容器托管的通用模式组件，任何被 @Component 标注的组件均为组件扫描的候选对象。类似地，凡是被 @Component 标注的注解，如@Service ，当任何组件标注它时，也被视作组件扫描的候选对象。

## 1.1 Spring装配方式
- @ComponentScan 方式

```java
@ComponentScan(basePackages = "com.loong.spring.boot")
public class SpringConfiguration {

}
```
注解的形式，依靠 basePackages 属性指定扫描范围。Spring 在启动时，会在某个生命周期内创建所有的配置类注解解析器，而 @ComponentScan 的处理器为 ComponentScanAnnotationParser。

## 1.2 Spring @Enable 模块驱动
所谓“模块”是指具备相同领域的功能组件集合，组合所形成的一个独立的单元，比如 Web MVC 模块、AspectJ代理模块、Caching(缓存)模块、JMX(Java 管理扩展)模块、Async(异步处理)模块等。

Spring Boot和Spring Cloud版本中都一直被使用，这种模块化的注解均以 @Enable 作为前缀，如下所示：

| 框架实现  | @Enable注解模块  | 激活模块  |
| ------------ | ------------ | ------------ |
| Spring Framework  | @EnableWebMvc  | Web Mvc 模块  |
| /  | @EnableTransactionManagement  | 事物管理模块  |
| / | @EnableWebFlux  | Web Flux 模块  |
| Spring Boot  | @EnableAutoConfiguration  | 自动装配模块  |
| /  | @EnableConfigurationProperties  | 配置属性绑定模块  |
| / | @EnableOAuth2Sso  | OAuth2 单点登陆模块  |
| Spring Cloud | @EnableEurekaServer  | Eureka 服务器模块  |
| / | @EnableFeignClients  | Feign 客户端模块  |
| / | @EnableZuulProxy  | 服务网关 Zuul 模块  |
| / | @EnableCircuitBreaker  | 服务熔断模块  |

## 1.3 Spring 条件装配
条件装配指的是通过一些列操作判断是否装配 Bean ，也就是 Bean 装配的前置判断。实现方式主要有两种：@Profile 和 @Conditional。
```java
@Conditional(HelloWorldCondition.class)
@Component
public class HelloWorldConfiguration {
    public HelloWorldConditionConfiguration (){
        System.out.println("HelloWorldConfiguration初始化。。。");
    }
}

public class HelloWorldCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        // ...
        return true;
    }
}
```
通过自定义一个 HelloWorldConfiguration 配置类，再标注 @Conditional 注解导入 HelloWorldCondition类，该类必须实现 Condition 接口，然后重写 matches 方法，在方法中可以通过两个入参来获取一系列的上下文数据和元数据，最终返回ture或false来判定该类是否初始化，

# 2、自动装配正文
在 SpringBoot 时代，通过一个main方法就可以启动一个应用，其底层依赖的就是 Spring 几个注解。从 @SpringBootApplication 注解中的 @EnableAutoConfiguration 注解开始，@EnableAutoConfiguration 属于 Spring 的 @Enable 模块注解，在该注解中通过 @Import 导入 AutoConfigurationImportSelector 类，在该类中加载所有以 AutoConfiguration 为后缀且标注 @Configuration 注解的自动配置类，每个自动配置类可以装配一个外部模块，如 Web MVC 模块对应的配置类是 WebMvcAutoConfiguration 。在自动配置类中又有众多 @Conditional 条件注解，可达到灵活装配的目的。

## 2.1 Spring Boot 自动装配实现
Spring Boot 的启动过程非常简单，只需要启动一个 main 方法，项目就可以运行，就算依赖了诸多外部模块如：MVC、Redis等，也不需要我们进行过多的配置。

```java
@SpringBootApplication
public class LoongSpringBootApplication {
	public static void main(String[] args) {
		SpringApplication.run(LoongSpringBootApplication.class, args);
	}
}

//关注的是 @SpringBootApplication 这个注解：
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

}
```
- @SpringBootConfiguration：它里面标注了 @Configuration 注解，表明这是个配置类，功能与 @Configuration 无异。
- @EnableAutoConfiguration：这个就是实现自动装配的核心注解，是用来激活自动装配的，其中默认路径扫描以及组件装配、排除等都通过它来实现。
- @ComponentScan：这是用来扫描被 @Component标注的类 ，只不过这里是用来过滤 Bean 的，指定哪些类不进行扫描，而且用的是自定义规则。
- Class<?>[] exclude()：根据class来排除，排除指定的类加入spring容器，传入的类型是class类型。且继承自 @EnableAutoConfiguration 中的属性。
- String[] excludeName()：根据class name来排除，排除特定的类加入spring容器，参数类型是class的全类名字符串数组。同样继承自 @EnableAutoConfiguration。
- String[] scanBasePackages()：可以指定多个包名进行扫描。继承自 @ComponentScan 。
- Class<?>[] scanBasePackageClasses()：可以指定多个类或接口的class，然后扫描 class 所在包下的所有组件。同样继承自 @ComponentScan 。

## 2.2 @EnableAutoConfiguration 实现
@EnableAutoConfiguration 是实现自动装配的核心注解，是用来激活自动装配的，看注解前缀我们应该知道是Spring @Enable 模块驱动的设计模式，所以它必然会有 @Import 导入的被 @Configuration 标注的类或实现 ImportSelector 或 ImportBeanDefinitionRegistrar 接口的类。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	...
}
```
- @AutoConfigurationPackage：这是用来将启动类所在包，以及下面所有子包里面的所有组件扫描到Spring容器中，这里的组件是指被 @Component或其派生注解标注的类。这也就是为什么不用标注@ComponentScan的原因。
- @Import(AutoConfigurationImportSelector.class)：这里导入的是实现了 ImportSelector 接口的类，组件自动装配的逻辑均在重写的 selectImports 方法中实现。
### 2.2.1 获取默认包扫描路径
Spring Boot通过 @AutoConfigurationPackage 注解获取默认包扫描路径:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
}
```
它是通过 @Import 导入了 AutoConfigurationPackages.Registrar 类，该类实现了 ImportBeanDefinitionRegistrar 接口。

## 2.3 自动装配的组件内部实现
```java
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
        ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
    ...
    @Configuration
    @Import(EnableWebMvcConfiguration.class)
    @EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class})
    @Order(0)
    public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ResourceLoaderAware {
        ...
        @Bean
        @ConditionalOnBean(View.class)
        @ConditionalOnMissingBean
        public BeanNameViewResolver beanNameViewResolver() {
            ...
        }
        ...
    }

    @Configuration
    public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {

        @Bean
        @Override
        public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
            ...
        }

        @Bean
        @Primary
        @Override
        public RequestMappingHandlerMapping requestMappingHandlerMapping() {
            ...
        }
    }
    ...
}
```
注解部分：
- @Configuration：标识该类是一个配置类
- @ConditionalXXX：Spring 条件装配，只不过经由 Spring Boot 扩展形成了自己的条件化自动装配，且都是@Conditional 的派生注解。
  - @ConditionalOnWebApplication：参数值是 Type 类型的枚举，当前项目类型是任意、Web、Reactive其中之一则实例化该 Bean。这里指定如果为 Web 项目才满足条件。
  - @ConditionalOnClass：参数是 Class 数组，当给定的类名在类路径上存在，则实例化当前Bean。这里当Servlet.class、 DispatcherServlet.class、 WebMvcConfigurer.class存在才满足条件。
  - @ConditionalOnMissingBean：参数是也是 Class 数组，当给定的类没有实例化时，则实例化当前Bean。这里指定当 WebMvcConfigurationSupport 该类没有实例化时，才满足条件。

装配顺序：
- @AutoConfigureOrder：参数是int类型的数值，数越小越先初始化。
- @AutoConfigureAfter：参数是 Class 数组，在指定的配置类初始化后再加载。
- @AutoConfigureBefore：参数同样是 Class 数组，在指定的配置类初始化前加载。

代码部分：
这部分就比较直接了，实例化了和 Web MVC 相关的Bean，如 HandlerAdapter、HandlerMapping、ViewResolver等。其中，出现了 DelegatingWebMvcConfiguration 类，这是 @EnableWebMvc 所 @Import导入的配置类。
可以看到，在Spring Boot 自动装配的类中，经过了一系列的 @Conditional 条件判断，然后实例化某个模块需要的Bean，且无需我们配置任何东西。

# 3、SpringApplication启动类准备阶段
在构造 SpringApplication 启动类时，初始化了几个重要的类，如 WebApplicationType 、ApplicationContextInitializer、ApplicationListener。其中 WebApplicationType 存储的是当前应用类型，如 Servlet Web 、Reactive Web； ApplicationContextInitializer 和 ApplicationListener 则是 SpringBoot 通过扩展 Spring 特性创建的初始化器及监听器。

## 3.1 SpringApplication 配置
SpringApplication 准备阶段结束后，按道理应该进入运行阶段，但运行阶段之前还有一个操作，就是可以修改 SpringApplication 默认配置。开头的代码示例可以看到，应用程序主类中的main方法中写的都是SpringApplication.run(xx.class)，可能这种写法不满足我们的需求，我们可以对SpringApplication进行一些配置，例如关闭Banner，设置一些默认的属性等。下面则是利用 SpringApplicationBuilder 的方式来添加配置：
```java
@SpringBootApplication
public class DiveInSpringBootApplication {
	public static void main(String[] args) {
		new SpringApplicationBuilder(DiveInSpringBootApplication.class)
				// 设置当前应用类型
				.web(WebApplicationType.SERVLET)
				// 设置 banner 横幅打印方式、有关闭、日志、控制台
				.bannerMode(Banner.Mode.OFF)
				// 设置自定义的 banner
				.banner()
				// 追加自定义的 initializer 到集合中
				.initializers()
				// 追加自定义的 listeners 到集合中
				.listeners()
				.run(args);
	}
}
```
使用该方式实现的SpringApplication可以对其添加自定义的配置。

# 4、SpringApplication启动类运行阶段
在 SpringApplication 运行阶段中，先是通过扩展 Spring 监听机制，在 SpringBoot 各个阶段发布不同事件，执行多个事件监听器；然后创建 Environment 类，这是外部化配置的核心类；最后启动 Spring 容器，通过 WebApplicationType 判定当前应用类型，创建应用对应 ApplicationContext 应用上下文，再调用 ApplicationContext#refresh 方法启动容器。

# 5、外部化配置之Environment
外部化配置的几种资源类型，如 properties、YAML、环境变量、系统属性、启动参数等。还详细介绍了 Environment 类，该类是外部化配置核心类，所有外部化配置数据，都保存在该类中，并和大家讨论了整个存储流程。

# 6、外部化配置之@ConfigurationProperties
@ConfigurationProperties 是 SpringBoot 实现外部化配置的重要注解，配合 SprinBoot 自动装配特性来达到快速开发的目的。主要将 properties 配置文件和 Properties 配置类中的属性进行映射，同样也和大家讨论了整个映射流程。

## 6.1 @ConfigurationProperties
当 Spring Boot 集成外部组件后，就可在 properties 或 YAML 配置文件中定义组件需要的属性，如 Redis 组件：
```xml
spring.redis.url=redis://user:password@example.com:6379
spring.redis.host=localhost
spring.redis.password=123456
spring.redis.port=6379
```

其中都是以 spring.redis 为前缀。这其实是 Spring Boot 为每个组件提供了对应的 Properties 配置类，并将配置文件中的属性值給映射到配置类中，而且它们有个特点，都是以 Properties 结尾，如 Redis 对应的配置类是 RedisProperties：
```java
@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {
    private String url;

	private String host = "localhost";

	private String password;

	private int port = 6379;
	...
}
```
@ConfigurationProperties 的注解，它的 prefix 参数就是约定好的前缀。该注解的功能就是将配置文件中的属性和 Properties 配置类中的属性进行映射，来达到自动配置的目的。这个过程分为两步，第一步是注册 Properties 配置类，第二步是绑定配置属性，过程中还涉及到一个注解，它就是 @EnableConfigurationProperties ，该注解是用来触发那两步操作的。

Spring Boot 外部化配置做一个整体的总结：
1. 首先，外部化配置是 Spring Boot 的一个特性，主要是通过外部的配置资源实现与代码的相互配合，来避免硬编码，提供应用数据或行为变化的灵活性。
2. 然后介绍了几种外部化配置的资源类型，如 properties 和 YAML 配置文件类型，并介绍了获取外部化配置资源的几种方式。
3. 其次，介绍了 Environment 类的加载流程，以及所有外部化配置加载到 Environment 中的底层是实现。Environment 是 Spring Boot 外部化配置的核心类，该类存储了所有的外部化配置资源，且其它获取外部化配置资源的方式也都依赖于该类。
4. 最后，介绍了 Spring Boot 框架中核心的 @ConfigurationProperties 注解，该注解是将 application 配置文件中的属性值和 Properties 配置类中的属性进行映射，来达到自动配置的目的，并带大家探讨了这一过程的底层实现。

# 7、嵌入式Web容器
传统 Spring 应用需手动创建和启动 Web 容器，在 SpringBoot 中，则是嵌入式的方式自动创建和启动。SpringBoot 支持的 Web 容器类型有 Servlet Web 容器和 Reactive Web 容器，它们都有具体容器实现，Sevlet Web 对应的是 Tomcat、Jetty、Undertow，默认实现是 Tomcat；Reactive Web 对应的是 Netty 。

# 8、Starter机制之自定义Starter
在 Spring 时代，搭建一个 Web 应用通常需要在 pom 文件中引入多个 Web 模块相关的 Maven 依赖，如 SpringMvc、Tomcat 等依赖，而 SpringBoot 则只需引入 spring-boot-starter-web 依赖即可。这就是 SpringBoot 的 Starter 特性，用来简化项目初始搭建以及开发过程，它是一个功能模块的所有 Maven 依赖集合体。

## 8.1 SpringBoot Starter 原理
SpringBoot 提供了非常多的 Starter，下面列出常用的几个：

| 名称  | 功能  |
| ------------ | ------------ |
| spring-boot-starter-web | 支持 Web 开发，包括 Tomcat 和 spring-webmvc  |
| spring-boot-starter-redis  | 支持 Redis 键值存储数据库，包括 spring-redis  |
| spring-boot-starter-test  | 支持常规的测试依赖，包括 JUnit、Hamcrest、Mockito 以及 spring-test 模块  |
| spring-boot-starter-aop  | 支持面向切面的编程即 AOP，包括 spring-aop 和 AspectJ  |
| spring-boot-starter-jdbc  |  支持JDBC数据库 |
| spring-boot-starter-data-jpa | 支持 JPA ，包括 spring-data-jpa、spring-orm、Hibernate  |