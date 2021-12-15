[TOC]

pom.xml中添加Nacos的spring-cloud-alibaba-nacos-discovery依赖，同时去掉Eureka的spring-cloud-starter-netflix-eureka-client依赖。
```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-alibaba-nacos-discovery -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
    <version>0.2.1.RELEASE</version>
</dependency>
```

# application配置。
修改application配置application.properties。
```xml
nacos.discovery.server-addr=127.0.0.1:8848
#有人说不生效，就试下下面的
#spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

如果是application.yml，请试下下面的配置：
```yml
nacos:
    discovery:
      server-addr: 127.0.0.1:8848
#建议先用上面的
spring:
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
```

# @EnableDiscoveryClient注解
更换EnableEurekaClient 注解。如果在你的应用启动程序启动类加了@EnableEurekaClient ，请修改为@EnableDiscoveryClient 。

# @PostConstruct服务注册
被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。也就是运行程序之后就注册到nacos上去。

```java
@SpringBootApplication
@RestController
public class Springboot2NacosDiscoveryApplication {

	@NacosInjected
	private NamingService namingService;

	@Value("${server.port}")
	private int serverPort;

	@Value("${spring.application.name}")
	private String applicationName;

	@PostConstruct
	public void registerInstance() throws NacosException{
		namingService.registerInstance(applicationName,"127.0.0.1",serverPort);
	}

	@GetMapping( "/getInstance")
	public List<Instance> getInstance(@RequestParam String serviceName) throws NacosException {
		return namingService.getAllInstances(serviceName);
	}

	public static void main(String[] args) {
		SpringApplication.run(Springboot2NacosDiscoveryApplication.class, args);
	}
}
```

- registerInstance：注册实例，有多个方法，本文使用的方法需要传入三个参数，分别是：服务名，ip和端口号。
- getAllInstances：获取实例，传入服务名。