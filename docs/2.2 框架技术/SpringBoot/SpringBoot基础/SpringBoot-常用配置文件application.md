[TOC]

部分内容原文地址：博客园-JackYang[SpringCloud入门之常用的配置文件 application.yml和 bootstrap.yml区别](https://www.cnblogs.com/BlogNetSpace/p/8469033.html)
______

Spring Boot 中有以下两种配置文件

- bootstrap (.yml 或者 .properties)
- application (.yml 或者 .properties)

# 1、SpringBoot bootstrap配置文件不生效问题
单独使用SpringBoot，发现其中的bootstrap.properties文件无法生效，改成yaml格式也无济于事。

最后调查发现原来是因为SpringBoot本身并不支持，需要和Spring Cloud 的组件结合——只有加上Spring Cloud Context依赖才能生效。

即在pom中引入：
```xml
<!--需要引入该jar才能使bootstrap配置文件生效-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-context</artifactId>
</dependency>
```

# 2、bootstrap/ application 的区别
> 在 Spring Boot 中有两种上下文，一种是 bootstrap, 另外一种是 application, bootstrap 是应用程序的父上下文，也就是说 bootstrap 加载优先于 applicaton。bootstrap 主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。这两个上下文共用一个环境，它是任何Spring应用程序的外部属性的来源。bootstrap 里面的属性会优先加载，它们默认也不能被本地相同配置覆盖。

因此，对比 application 配置文件，bootstrap 配置文件具有以下几个特性。

- boostrap 由父 ApplicationContext 加载，比 applicaton 优先加载
- boostrap 里面的属性不能被覆盖

# 3、bootstrap/ application 的应用场景
application 配置文件这个容易理解，主要用于 Spring Boot 项目的自动化配置。

bootstrap 配置文件有以下几个应用场景。

- 使用 Spring Cloud Config 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
- 一些固定的不能被覆盖的属性
- 一些加密/解密的场景；

技术上，bootstrap.yml 是被一个父级的 Spring ApplicationContext 加载的。这个父级的 Spring ApplicationContext是先加载的，在加载application.yml 的 ApplicationContext之前。

**为何需要把 config server 的信息放在 bootstrap.yml 里？**

当使用 Spring Cloud 的时候，配置信息一般是从 config server 加载的，为了取得配置信息（比如密码等），你需要一些提早的引导配置。因此，把 config server 信息放在 bootstrap.yml，用来加载在这个时期真正需要的配置信息。

# 4、高级使用场景
## 4.1 启动上下文
Spring Cloud会创建一个`Bootstrap Context`，作为Spring应用的`Application Context`的父上下文。初始化的时候，`Bootstrap Context`负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的`Environment`。`Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 `Bootstrap context`和`Application Context`有着不同的约定，所以新增了一个`bootstrap.yml`文件，而不是使用`application.yml` (或者`application.properties`)。保证`Bootstrap Context`和`Application Context`配置的分离。下面是一个例子： **bootstrap.yml**

```yml
spring:
  application:
    name: foo
  cloud:
    config:
      uri: ${SPRING_CONFIG_URI:http://localhost:8888}
```
**推荐在`bootstrap.yml` or `application.yml`里面配置`spring.application.name`. 你可以通过设置`spring.cloud.bootstrap.enabled=false`来禁用`bootstrap`。**

## 4.2 应用上下文层次结构
如果你通过`SpringApplication`或者`SpringApplicationBuilder`创建一个`Application Context`,那么会为spring应用的`Application Context`创建父上下文`Bootstrap Context`。在Spring里有个特性，子上下文会继承父类的`property sources` and `profiles` ，所以`main application context` 相对于没有使用Spring Cloud Config，会新增额外的`property sources`。额外的`property sources`有：

- “bootstrap” : 如果在Bootstrap Context扫描到PropertySourceLocator并且有属性，则会添加到CompositePropertySource。Spirng Cloud Config就是通过这种方式来添加的属性的，详细看源码ConfigServicePropertySourceLocator`。下面也也有一个例子自定义的例子。
- “applicationConfig: [classpath:bootstrap.yml]” ，（如果有spring.profiles.active=production则例如 applicationConfig: [classpath:/bootstrap.yml]#production）: 如果你使用bootstrap.yml来配置Bootstrap Context，他比application.yml优先级要低。它将添加到子上下文，作为Spring Boot应用程序的一部分。下文有介绍。

由于优先级规则，Bootstrap Context不包含从bootstrap.yml来的数据，但是可以用它作为默认设置。

你可以很容易的**扩展任何你建立的上下文层次**，可以使用它提供的接口，或者使用SpringApplicationBuilder包含的方法（parent()，child()，sibling()）。Bootstrap Context将是最高级别的父类。扩展的每一个Context都有有自己的bootstrap property source（有可能是空的）。扩展的每一个Context都有不同spring.application.name。同一层层次的父子上下文原则上也有一有不同的名称，因此，也会有不同的Config Server配置。子上下文的属性在相同名字的情况下将覆盖父上下文的属性。

**注意SpringApplicationBuilder允许共享Environment到所有层次，但是不是默认的。**因此，同级的兄弟上下文不在和父类共享一些东西的时候不一定有相同的profiles或者property sources。

## 4.3 修改bootstrap属性配置
源码位置BootstrapApplicationListener。
```java
String configName = environment.resolvePlaceholders("${spring.cloud.bootstrap.name:bootstrap}");

    String configLocation = environment.resolvePlaceholders("${spring.cloud.bootstrap.location:}");

    Map<String, Object> bootstrapMap = new HashMap<>();bootstrapMap.put("spring.config.name",configName);
    if(StringUtils.hasText(configLocation)){
        bootstrapMap.put("spring.config.location", configLocation);
    }
```
 bootstrap.yml是由spring.cloud.bootstrap.name（默认:”bootstrap”）或者spring.cloud.bootstrap.location（默认空）。这些属性行为与spring.config.*类似，通过它的Environment来配置引导ApplicationContext。如果有一个激活的profile（来源于spring.profiles.active或者Environment的Api构建），例如bootstrap-development.properties 就是配置了profile为development的配置文件.

### 4.3.1 覆盖远程属性
property sources被bootstrap context 添加到应用通常通过远程的方式，比如”Config Server”。默认情况下，本地的配置文件不能覆盖远程配置，但是可以通过启动命令行参数来覆盖远程配置。如果需要本地文件覆盖远程文件，需要在远程配置文件里

### 4.3.2 设置授权
spring.cloud.config.allowOverride=true（这个配置不能在本地被设置）。一旦设置了这个权限，你可以配置更加细粒度的配置来配置覆盖的方式，

比如：

- spring.cloud.config.overrideNone=true 覆盖任何本地属性 
- spring.cloud.config.overrideSystemProperties=false 仅仅系统属性和环境变量 
源文件见PropertySourceBootstrapProperties.

## 4.4 自定义启动配置
bootstrap context是依赖/META-INF/spring.factories文件里面的org.springframework.cloud.bootstrap.BootstrapConfiguration条目下面，通过逗号分隔的Spring  @Configuration类来建立的配置。任何main application context需要的自动注入的Bean可以在这里通过这种方式来获取。这也是ApplicationContextInitializer建立@Bean的方式。可以通过@Order来更改初始化序列，默认是”last”。

```yml
# spring-cloud-context-1.1.1.RELEASE.jar
# spring.factories
# AutoConfiguration
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\
org.springframework.cloud.autoconfigure.RefreshAutoConfiguration,\
org.springframework.cloud.autoconfigure.RefreshEndpointAutoConfiguration,\
org.springframework.cloud.autoconfigure.LifecycleMvcEndpointAutoConfiguration

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.cloud.bootstrap.BootstrapApplicationListener,\
org.springframework.cloud.context.restart.RestartListener

# Bootstrap components
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration,\
org.springframework.cloud.bootstrap.encrypt.EncryptionBootstrapConfiguration,\
org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
```

> 添加的自定义BootstrapConfiguration类没有错误的@ComponentScanned到你的主应用上下文，他们可能是不需要的。使用一个另外的包不被@ComponentScan或者@SpringBootApplication注解覆盖到。

bootstrap context通过spring.factories配置的类初始化的所有的Bean都会在SpingApplicatin启动前加入到它的上下文里去。

## 4.5 自定义引导配置来源：Bootstrap Property Sources
默认的`property source`添加额外的配置是通过配置服务（Config Server），你也可以自定义添加`property source`通过实现`PropertySourceLocator`接口来添加。你可以使用它加配置属性从不同的服务、数据库、或者其他。

下面是一个自定义的例子:
```java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {

    @Override
    public PropertySource<?> locate(Environment environment) {
        return new MapPropertySource("customProperty",
                Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
    }
}
```
Environment被ApplicationContext建立，并传入property sources（可能不同个profile有不同的属性），所以，你可以从Environment寻找找一些特别的属性。比如spring.application.name，它是默认的Config Server property source。

如果你建立了一个jar包，里面添加了一个META-INF/spring.factories文件：

org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
那么，”customProperty“的PropertySource将会被包含到应用。