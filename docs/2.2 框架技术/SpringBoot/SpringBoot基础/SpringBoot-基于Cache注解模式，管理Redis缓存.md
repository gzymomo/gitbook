[TOC]

# 1、Cache缓存简介 
1. 从Spring3开始定义Cache和CacheManager接口来统一不同的缓存技术；
2. Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
3. Cache接口下Spring提供了各种缓存的实现；
4. 如RedisCache，EhCacheCache ,ConcurrentMapCache等；

# 2、核心API说明
1. Cache缓存接口
定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等

2. CacheManager
缓存管理器，管理各种缓存（cache）组件

3. @Cacheable
主要针对方法配置，能够根据方法的请求参数对其进行缓存

```java
Cacheable 执行流程
1）方法运行之前，按照cacheNames指定的名字先去查询Cache 缓存组件
2）第一次获取缓存如果没有Cache组件会自动创建
3）Cache中查找缓存的内容，使用一个key，默认就是方法的参数
4）key是按照某种策略生成的；默认是使用keyGenerator生成的，这里使用自定义配置
5）没有查到缓存就调用目标方法；
6）将目标方法返回的结果，放进缓存中

Cacheable 注解属性
cacheNames/value：指定方法返回结果使用的缓存组件的名字，可以指定多个缓存
key：缓存数据使用的key
key/keyGenerator：key的生成器，可以自定义
cacheManager：指定缓存管理器
cacheResolver：指定缓存解析器
condition：指定符合条件的数据才缓存
unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存
sync：是否使用异步模式
```

4. @CacheEvict
清除缓存
```java
CacheEvict：缓存清除
key：指定要清除的数据
allEntries = true：指定清除这个缓存中所有的数据
beforeInvocation = false：方法之前执行清除缓存,出现异常不执行
beforeInvocation = true：代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
```

5. @CachePut
保证方法被调用，又希望结果被缓存。
与@Cacheable区别在于是否每次都调用方法，常用于更新，写入
```java
CachePut：执行方法且缓存方法执行的结果
修改了数据库的某个数据，同时更新缓存；
执行流程
 1)先调用目标方法
 2)然后将目标方法的结果缓存起来
```

6. @EnableCaching
开启基于注解的缓存

7. keyGenerator
缓存数据时key生成策略

8. @CacheConfig
统一配置本类的缓存注解的属性

# 3、SpringBoot整合Cache
## 3.1 核心依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
## 3.2 Cache缓存配置
```java
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.lang.reflect.Method;
@Configuration
public class CacheConfig {
    /**
     * 自定义 Cache 的 key 生成器
     */
    @Bean("oneKeyGenerator")
    public KeyGenerator getKeyGenerator (){
        return new KeyGenerator() {
            @Override
            public Object generate(Object obj, Method method, Object... objects) {
                return "KeyGenerator:"+method.getName();
            }
        } ;
    }
}
```
## 3.3 启动类注解开启Cache
```java
@EnableCaching            // 开启Cache 缓存注解
@SpringBootApplication
public class CacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheApplication.class,args) ;
    }
}
```
## 3.4 Cache注解使用代码
1）封装增删改查接口
```java
import com.boot.cache.entity.User;
public interface UserService {
    // 增、改、查、删
    User addUser (User user) ;
    User updateUser (Integer id) ;
    User selectUser (Integer id) ;
    void deleteUser (Integer id);
}
```

2）Cache注解使用案例
```java
import com.boot.cache.entity.User;
import com.boot.cache.service.UserService;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
@Service
public class UserServiceImpl implements UserService {
    // 使用自定义的key生成策略
    // 缓存结果key：addUser::KeyGenerator:addUser
    @CachePut(value = "addUser",keyGenerator="oneKeyGenerator")
    @Override
    public User addUser(User user) {
        return user ;
    }
    // 缓存结果key：updateUser::2
    @CachePut(value = "updateUser",key = "#result.id")
    @Override
    public User updateUser(Integer id) {
        User user = new User() ;
        user.setId(id);
        user.setName("smile");
        return user;
    }
    // 缓存结果key: selectUser::3
    @Cacheable(cacheNames = "selectUser",key = "#id")
    @Override
    public User selectUser(Integer id) {
        User user = new User() ;
        user.setId(id);
        user.setName("cicadaSmile");
        return user;
    }
    // 删除指定key: selectUser::3
    @CacheEvict(value = "selectUser",key = "#id",beforeInvocation = true)
    @Override
    public void deleteUser(Integer id) {

    }
}
```
## 3.5 测试代码块
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = CacheApplication.class)
public class CacheTest {
    @Resource
    private UserService userService ;
    // 分别测试：增、改、查、删,四个方法
    @Test
    public void testAdd (){
        User user = new User() ;
        user.setId(1);
        user.setName("cicada");
        userService.addUser(user) ;
    }
    @Test
    public void testUpdate (){
        userService.updateUser(2) ;
    }
    @Test
    public void testSelect (){
        userService.selectUser(3) ;
    }
    @Test
    public void testDelete (){
        userService.deleteUser(3) ;
    }
}
```