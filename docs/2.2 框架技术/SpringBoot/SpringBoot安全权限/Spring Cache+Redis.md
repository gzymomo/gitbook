[TOC]

Spring3.1中开始引入了令人激动的Cache，在Spring Boot中，可以非常方便的使用Redis来作为Cache的实现，进而实现数据的缓存。

# 1、创建工程
创建一个Spring Boot工程，注意创建的时候需要引入三个依赖，web、cache以及redis：
对应的依赖内容如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

# 2、基本配置
工程创建好之后，首先需要简单配置一下Redis，Redis的基本信息，另外，这里要用到Cache，因此还需要稍微配置一下Cache，如下：
```yml
spring.redis.port=6380
spring.redis.host=192.168.66.128

spring.cache.cache-names=c1
```
简单起见，这里我只是配置了Redis的端口和地址，然后给缓存取了一个名字，这个名字在后文会用到。

另外，还需要在配置类上添加如下代码，表示开启缓存：
```java
@SpringBootApplication
@EnableCaching
public class RediscacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(RediscacheApplication.class, args);
    }

}
```
完成了这些配置之后，Spring Boot就会自动帮我们在后台配置一个RedisCacheManager，相关的配置是在org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration类中完成的。部分源码如下：

```java
@Configuration
@ConditionalOnClass(RedisConnectionFactory.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
@ConditionalOnBean(RedisConnectionFactory.class)
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {

	@Bean
	public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory,
			ResourceLoader resourceLoader) {
		RedisCacheManagerBuilder builder = RedisCacheManager
				.builder(redisConnectionFactory)
				.cacheDefaults(determineConfiguration(resourceLoader.getClassLoader()));
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			builder.initialCacheNames(new LinkedHashSet<>(cacheNames));
		}
		return this.customizerInvoker.customize(builder.build());
	}
}
```

看类上的注解，发现在万事俱备的情况下，系统会自动提供一个RedisCacheManager的Bean，这个RedisCacheManager间接实现了Spring中的Cache接口，有了这个Bean，我们就可以直接使用Spring中的缓存注解和接口了，而缓存数据则会被自动存储到Redis上。在单机的Redis中，这个Bean系统会自动提供，如果是Redis集群，这个Bean需要开发者来提供

# 3、缓存使用
## 3.1 @CacheConfig
这个注解在类上使用，用来描述该类中所有方法使用的缓存名称，当然也可以不使用该注解，直接在具体的缓存注解上配置名称，示例代码如下：
```java
@Service
@CacheConfig(cacheNames = "c1")
public class UserService {
}
```

## 3.2 @Cacheable
这个注解一般加在查询方法上，表示将一个方法的返回值缓存起来，默认情况下，缓存的key就是方法的参数，缓存的value就是方法的返回值。示例代码如下：
```java
@Cacheable(key = "#id")
public User getUserById(Integer id,String username) {
    System.out.println("getUserById");
    return getUserFromDBById(id);
}
```

当有多个参数时，默认就使用多个参数来做key，如果只需要其中某一个参数做key，则可以在@Cacheable注解中，通过key属性来指定key，如上代码就表示只使用id作为缓存的key，如果对key有复杂的要求，可以自定义keyGenerator。当然，Spring Cache中提供了root对象，可以在不定义keyGenerator的情况下实现一些复杂的效果：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/550c998a1edc357d31f10b81838b6e5a?showdoc=.jpg)

## 3.3 @CachePut
这个注解一般加在更新方法上，当数据库中的数据更新后，缓存中的数据也要跟着更新，使用该注解，可以将方法的返回值自动更新到已经存在的key上，示例代码如下：
```java
@CachePut(key = "#user.id")
public User updateUserById(User user) {
    return user;
}
```

## 3.4 @CacheEvict
这个注解一般加在删除方法上，当数据库中的数据删除后，相关的缓存数据也要自动清除，该注解在使用的时候也可以配置按照某种条件删除（condition属性）或者或者配置清除所有缓存（allEntries属性），示例代码如下：
```java
@CacheEvict()
public void deleteUserById(Integer id) {
    //在这里执行删除操作， 删除是去数据库中删除
}
```

# 4、总结
在Spring Boot中，使用Redis缓存，既可以使用RedisTemplate自己来实现，也可以使用使用这种方式，这种方式是Spring Cache提供的统一接口，实现既可以是Redis，也可以是Ehcache或者其他支持这种规范的缓存框架。从这个角度来说，Spring Cache和Redis、Ehcache的关系就像JDBC与各种数据库驱动的关系。