# Redis知识体系总结

原文地址：https://blog.csdn.net/guorui_java/article/details/116850879



# 一、百度百科

## 1、简介

（1）Redis（Remote Dictionary Server 远程字段服务）是一个开源的使用ANSI C语言编写、支持网络、科技与内存亦可持久化的日志型、key-value数据库，并提供多种语言的API。

（2）Redis是一个key-value存储系统，它支持存储的value类型相对更多，包括string、list、set、zset（sorted set --有序集合）和hash。这些数据结构都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，Redis支持各种不同方式的排序。为了保证效率，数据都是缓存在内存中，Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave（主从）同步。

（3）Redis提供了java、C/C++、PHP、JavaScript、Perl、Object-C、Python、Ruby、Erlang等客户端，使用很方便。

（4）Reids支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他服务器的主服务器。这使得Redis可执行单层数复制。存盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。

（5）在我们日常的Java Web开发中，无不都是使用数据库来进行数据的存储，由于一般的系统任务中通常不会存在高并发的情况，所以这样看起来并没有什么问题，可是一旦涉及大数据量的需求，比如一些商品抢购的情景，或者是主页访问量瞬间较大的时候，单一使用数据库来保存数据的系统会因为面向磁盘，磁盘读/写速度比较慢的问题而存在严重的性能弊端，一瞬间成千上万的请求到来，需要系统在极短的时间内完成成千上万次的读/写操作，这个时候往往不是数据库能够承受的，极其容易造成数据库系统瘫痪，最终导致服务宕机的严重生产问题。

## 2、什么是nosql技术

为了克服上述问题，java web项目通常会引入NoSQL技术，这是一种基于内存的数据库，并且提供一定的持久化功能。

Redis和MongoDB是当前使用最广泛的NoSQL， 而就Redis技术而言，它的性能十分优越，可以支持每秒十几万的读写操作，其性能远超数据库，并且还支持集群、。分布式、主从同步等配置，原则上可以无限扩展，让更多的数据存储在内存中，更让人欣慰的是它还支持一定的事务能力，这保证了高并发的场景下数据的安全和一致性。

## 3、Redis为何能解决高并发问题

    Redis是基于内存的，内存的读写速度非常快；
    Redis是单线程的，省去了很多上下文切换线程的时间；
    Redis使用多路复用技术，可以处理并发的连接。非IO内部实现采用epoll，采用了epoll自己实现的简单的事件框架。epoll的读写、关闭、连接都转化为事件，然后利用epoll的多路复用特性，绝不在IO上浪费一点时间。

### Redis高并发总结

    Redis是纯内存数据库，一般都是简单存取操作，线程占用的时间很多，时间的花费主要集中在IO上，所以读取速度快；
    Redis使用的是非阻塞IO，IO多路复用，使用了单线程来轮询描述符，将数据库的开、关、读、写都转换成事件，减少了线程切换时上下文切换和竞争。
    Redis采用了单线程的模型，保证了每个操作的原子性，也减少了线程的上下文切换和竞争。
    Redis全程使用hash结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如跳表，使用有序的数据结构加快读写的速度。
    Redis采用自己实现的事件分离器，效率比较高，内部采用非阻塞的执行方式，吞吐能力比较大。

## 4、Redis的优劣势

### （1）优势

    代码更清晰，处理逻辑更简单
    不用考虑各种锁的问题，不存在加锁和释放锁的操作，没有因为可能出现死锁而导致的性能消耗
    不存在多线程切换而消耗CPU

### （2）劣势

无法发挥多核CPU性能优势，不过可以通过单击开多个Redis实例来完善。

# 二、Redis为什么是单线程的

## 1、官方答案

Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络宽带。既然单线程容易实现，而且CPU不会成为瓶颈，那么顺理成章的采用单线程的方案。

## 2、我的理解

### （1）不需要各种锁的性能消耗

Redis的数据结构并不全是key-value形式的，还有list，hash等复杂的结构，这些结构有可能会进行很细粒度的操作，比如在很长的列表后面添加一个元素，在hash中添加或删除一个对象，这些操作可能就需要加非常多的锁，导致的结果是同步开销大大增加。

总之，在单线程的情况下，就不用去考虑各种锁的问题，不存在加锁和释放锁的操作，没有因为可能出现的死锁而导致的性能消耗。

### （2）单线程多进程集群方案

单线程的威力实际上非常强大，每核心效率也非常高，多线程自然是可以比单线程有更高的性能上限，但是在今天的计算环境中，即使是单机多线程的上限也往往不能满足需要了，需要进一步摸索的是多服务器集群化的方案，这些方案中多线程的技术照样是用不上的。

所以单线程、多进程的集群不失为一个时髦的解决方案。

### （3）CPU消耗

采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗CPU。

但是如果CPU称为Redis的瓶颈，或者不想让服务器其它CPU核闲置，那怎么办？

可以考虑多起几个Redis进程，Redis是key-value数据库，不是关系型数据库，数据之间没有约束。只要客户端分清哪些key放在哪个Redis进程中就可以了。

# 三、Linux中安装Redis

## 1、Redis下载

![img](https://img-blog.csdnimg.cn/2021051515045774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## 2、上传、解压

Redis一般安装在Linux环境下，开启虚拟机，通过xftp将redis压缩包上传到Linux服务器，并进行解压。

![img](https://img-blog.csdnimg.cn/20210515150618746.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

修改redis.conf配置文件，使其在后台启动

![img](https://img-blog.csdnimg.cn/20210515150658400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

# 四、Redis在Java Web中的应用

Redis 在 Java Web 主要有两个应用场景：

    存储缓存用的数据
    需要高速读写的场合

## 1、存储缓存用的数据 

在日常对数据库的访问中，读操作的次数远超写操作，比例大概在 1:9 到 3:7，所以需要读的可能性是比写的可能大得多的。当我们使用SQL语句去数据库进行读写操作时，数据库就会去磁盘把对应的数据索引取回来，这是一个相对较慢的过程。 

如果放在Redis中，也就是放在内存中，让服务器直接读取内存中的数据，那么速度就会快很多，并且会极大减少数据库的压力，但是使用内存进行数据存储开销也是比较大的，限于成本的原因，一般我们只是使用Redis存储一些常用的和主要的数据，比如用户登录信息等。

一般而言在使用 Redis 进行存储的时候，我们需要从以下几个方面来考虑:

（1）业务数据常用吗？使用率如何？

如果使用率较低，就没必要写入缓存。

（2）该业务是读操作多，还是写操作多？

如果写操作多，频繁需要写入数据库，也没必要使用缓存。

（3）业务数据大小如何？

如果要存储几百兆字节的文件，会给缓存带来很大的压力，这样也没必要。

在考虑了这些问题之后，如果觉得有必要使用缓存，那么就使用它！

从上图我们可以知道以下两点：

（1）当第一次读取数据的时候，读取Redis的数据就会失败，此时就会触发程序读取数据库，把数据读取出来，并且写入Redis中

（2）当第二次以及以后需要读取数据时，就会直接读取Redis，读取数据后就结束了流程，这样速度大大提高了。

从上面的分析可以知道，读操作的可能性是远大于写操作的，所以使用 Redis 来处理日常中需要经常读取的数据，速度提升是显而易见的，同时也降低了对数据库的依赖，使得数据库的压力大大减少。

分析了读操作的逻辑，下面我们来看看写操作流程：

![img](https://img-blog.csdnimg.cn/20210515153337562.png)

从流程可以看出，更新或者写入的操作，需要多个 Redis 的操作，如果业务数据写次数远大于读次数那么就没有必要使用 Redis。

## 2、高速读写场合

在如今的互联网中，越来越多的存在高并发的情况，比如天猫双11、抢红包、抢演唱会门票等，这些场合都是在某一个瞬间或者是某一个短暂的时刻有成千上万的请求到达服务器，如果单纯的使用数据库来进行处理，就算不崩，也会很慢的，轻则造成用户体验极差用户量流水，重则数据库瘫痪，服务宕机，而这样的场合都是不允许的！

所以我们需要使用 Redis 来应对这样的高并发需求的场合，我们先来看看一次请求操作的流程：

![img](https://img-blog.csdnimg.cn/20210515153405496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

我们来进一步阐述这个过程：

（1）当一个请求到达服务器时，只是把业务数据在Redis上进行读写，而没有对数据库进行任何的操作，这样就能大大提高读写的速度，从而满足高速相应的需求。

（2）但是这些缓存的数据仍然需要持久化，也就是存入数据库之中，所以在一个请求操作完Redis的读写之后，会去判断该高速读写的业务是否结束，这个判断通常会在秒杀商品为0，红包金额为0时成立，如果不成立，则不会操作数据库；如果成立，则触发事件将Redis的缓存的数据以批量的形式一次性写入数据库，从而完成持久化的工作。

# 五、Redis代码实例

## 1、Java整合Redis

（1）导入pom

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

（2）编写Java主方法

调用Redis中的ping方法，惊现异常：

![img](https://img-blog.csdnimg.cn/20210515150813790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

开始的时候以为是防火墙的问题，后来通过查看redis状态发现IP地址不对，不应该是127.0.0.1

![img](https://img-blog.csdnimg.cn/20210515150845526.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

修改redis.conf

![img](https://img-blog.csdnimg.cn/20210515150931918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

注意：需要注意的是在修改redis.conf时，①注掉bind 127.0.0.1；②需要将本机访问保护模式设置为no；③此时可以配置多个ip

（3）再次执行主方法，执行成功！

![img](https://img-blog.csdnimg.cn/2021051515101838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## 2、五大数据类型代码实例

```java
package com.guor.redis;
 
import redis.clients.jedis.Jedis;
 
import java.util.List;
import java.util.Set;
 
public class JedisTest01 {
    public static void main(String[] args) {
        test05();
    }
 
    private static void test01(){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        String value = jedis.ping();
        System.out.println(value);
        //添加
        jedis.set("name","GooReey");
        //获取
        String name = jedis.get("name");
        System.out.println(name);
 
        jedis.set("age","30");
        jedis.set("city","dalian");
        //获取全部的key
        Set<String> keys = jedis.keys("*");
        for(String key : keys){
            System.out.println(key+" --> "+jedis.get(key));
        }
 
        //加入多个key和value
        jedis.mset("name1","zs","name2","ls","name3","ww");
        List<String> mget = jedis.mget("name1", "name2");
        System.out.println(mget);//[zs, ls]
    }
 
    //list
    private static void test02(){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        jedis.lpush("key1","01","02","03");
        List<String> values = jedis.lrange("key1",0,-1);
        System.out.println(values);//[03, 02, 01]
    }
 
    //set
    private static void test03(){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        jedis.sadd("username","zs","ls","ww");
        Set<String> names = jedis.smembers("username");
        System.out.println(names);//[ww, zs, ls]
    }
 
    //hash
    private static void test04(){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        jedis.hset("users","age", "20");
        String hget = jedis.hget("users","age");
        System.out.println(hget);
    }
 
    //zset
    private static void test05(){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        jedis.zadd("china",100d,"shanghai");
        Set<String> names = jedis.zrange("china",0,-1);
        System.out.println(names);//[shanghai]
    }
}
```

## 3、手机验证码功能代码实例

```java
package com.guor.redis;
 
import redis.clients.jedis.Jedis;
 
import java.util.Random;
 
public class PhoneCode {
    public static void main(String[] args) {
        verifyCode("10086");//795258
        getRedisCode("10086","795258");//success.
    }
 
    //1、生成6位数字验证码
    public static String getCode(){
        Random random = new Random();
        String code = "";
        for (int i = 0; i < 6; i++) {
            int rand = random.nextInt(10);
            code += rand;
        }
        return code;//849130
    }
 
    //2、每个手机每天只能发送三次，验证码放到redis中，设置过期时间
    public static void verifyCode(String phone){
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        //拼接key
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";
        //每个手机每天只能发送三次
        String count = jedis.get(countKey);
        if(count == null){
            //设置过期时间
            jedis.setex(countKey,24*60*60,"1");
        }else if(Integer.parseInt(count)<=2){
            //发送次数+1
            jedis.incr(countKey);
        }else if(Integer.parseInt(count)>2){
            System.out.println("今天的发送次数已经超过三次");
            jedis.close();
        }
 
        String vCode = getCode();
        jedis.setex(codeKey,120,vCode);
        jedis.close();
    }
 
    //3、验证码校验
    public static void getRedisCode(String phone, String code){
        //从redis中获取验证码
        Jedis jedis = new Jedis("192.168.194.131", 6379);
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);
        if(redisCode.equals(code)){
            System.out.println("success.");
        }else{
            System.out.println("error");
        }
        jedis.close();
    }
}
```

![img](https://img-blog.csdnimg.cn/20210515151106101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

当超过三次时： 

![img](https://img-blog.csdnimg.cn/20210515151123946.png)

## 4、SpringBoot整合Redis

（1）建工程，引入pom

![img](https://img-blog.csdnimg.cn/20210515151143420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.guor</groupId>
    <artifactId>redisspringboot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redisspringboot</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
 
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>2.4.5</version>
        </dependency>
 
        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.9.0</version>
        </dependency>
 
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
 
</project>
```

（2）配置类

application.properties

```yaml
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=192.168.194.131
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=20
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=10
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=1000
```

RedisConfig

```java
package com.guor.redisspringboot.config;
 
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.*;
 
import java.time.Duration;
 
@EnableCaching
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);
 
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jackson2JsonRedisSerializer.setObjectMapper(om);
 
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
 
    /**
     * 基于SpringBoot2 对 RedisCacheManager 的自定义配置
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        //初始化一个RedisCacheWriter
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);
        //设置CacheManager的值序列化方式为json序列化
        RedisSerializer<Object> jsonSerializer = new GenericJackson2JsonRedisSerializer();
        RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair.fromSerializer(jsonSerializer);
        RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(pair);
 
        //设置默认超过时期是1天
        defaultCacheConfig.entryTtl(Duration.ofDays(1));
        //初始化RedisCacheManager
        return new RedisCacheManager(redisCacheWriter, defaultCacheConfig);
    }
}
```

（3）控制类测试

```java
package com.guor.redisspringboot.controller;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
@RequestMapping("/redisTest")
public class RedisTestController {
 
    @Autowired
    private RedisTemplate redisTemplate;
 
    @GetMapping
    public String getRedis(){
        redisTemplate.opsForValue().set("name","zs");
        String name = (String) redisTemplate.opsForValue().get("name");
        return name;
    }
}
```

（4）测试

![img](https://img-blog.csdnimg.cn/20210515151256538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

# 六、Redis事务

众所周知，事务是指“一个完整的动作，要么全部执行，要么什么也没有做”。

在聊redis事务处理之前，要先和大家介绍四个redis指令，即multi、exec、discard、watch。这四个指令构成了redis事务处理的基础。

1.multi用来组装一个事务；
2.exec用来执行一个事务；
3.discard用来取消一个事务；
4.watch用来监视一些key，一旦这些key在事务执行之前被改变，则取消事务的执行。

```bash
redis> multi 
OK
redis> INCR id
QUEUED
redis> INCR id
QUEUED
redis> INCR id
QUEUED
redis> exec
1) (integer) 1
2) (integer) 2
3) (integer) 3
```

我们在用multi组装事务时，每一个命令都会进入内存中缓存起来，QUEUED表示缓存成功，在exec时，这些被QUEUED的命令都会被组装成一个事务来执行。

对于事务的执行来说，如果redis开启了AOF持久化的话，那么一旦事务被成功执行，事务中的命令就会通过write命令一次性写到磁盘中去，如果在向磁盘中写的过程中恰好出现断电、硬件故障等问题，那么就可能出现只有部分命令进行了AOF持久化，这时AOF文件就会出现不完整的情况，这时，我们可以使用redis-check-aof工具来修复这一问题，这个工具会将AOF文件中不完整的信息移除，确保AOF文件完整可用。

有关事务，大家经常会遇到的是两类错误：

1.调用EXEC之前的错误
2.调用EXEC之后的错误

“调用EXEC之前的错误”，有可能是由于语法有误导致的，也可能时由于内存不足导致的。只要出现某个命令无法成功写入缓冲队列的情况，redis都会进行记录，在客户端调用EXEC时，redis会拒绝执行这一事务。（这时2.6.5版本之后的策略。在2.6.5之前的版本中，redis会忽略那些入队失败的命令，只执行那些入队成功的命令）。

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> hello world //错误指令
    (error) ERR unknown command 'hello world'
    127.0.0.1:6379> ping
    QUEUED
    127.0.0.1:6379> exec
    (error) EXECABORT Transaction discarded because of previous errors.

而对于“调用EXEC之后的错误”，redis则采取了完全不同的策略，即redis不会理睬这些错误，而是继续向下执行事务中的其他命令。这是因为，对于应用层面的错误，并不是redis自身需要考虑和处理的问题，所以一个事务中如果某一条命令执行失败，并不会影响接下来的其他命令的执行。我们也来看一个例子：

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set age 23
    QUEUED
    //age不是集合，所以如下是一条明显错误的指令
    127.0.0.1:6379> sadd age 15 
    QUEUED
    127.0.0.1:6379> set age 29
    QUEUED
    127.0.0.1:6379> exec //执行事务时，redis不会理睬第2条指令执行错误
    1) OK
    2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
    3) OK
    127.0.0.1:6379> get age
    "29" //可以看出第3条指令被成功执行了

好了，我们来说说最后一个指令“watch”，这是一个很好用的指令，它可以帮我们实现类似于“乐观锁”的效果，即CAS（check and set）。

watch本身的作用是“监视key是否被改动过”，而且支持同时监视多个key，只要还没真正触发事务，watch都会尽职尽责的监视，一旦发现某个key被修改了，在执行exec时就会返回nil，表示事务无法触发。

    127.0.0.1:6379> set age 23
    OK
    127.0.0.1:6379> watch age //开始监视age
    OK
    127.0.0.1:6379> set age 24 //在EXEC之前，age的值被修改了
    OK
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set age 25
    QUEUED
    127.0.0.1:6379> get age
    QUEUED
    127.0.0.1:6379> exec //触发EXEC
    (nil) //事务无法被执行

# 七、Redis持久化的两种方式

redis提供了两种持久化的方式，分别是RDB（Redis DataBase）和AOF（Append Only File）。

RDB，简而言之，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上；

AOF，则是换了一个角度来实现持久化，那就是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。

其实RDB和AOF两种方式也可以同时使用，在这种情况下，如果redis重启的话，则会优先采用AOF方式来进行数据恢复，这是因为AOF方式的数据恢复完整度更高。

如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库，就像memcache一样。

## 1、RDB

RDB方式，是将redis某一时刻的数据持久化到磁盘中，是一种快照式的持久化方法。

redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。

对于RDB方式，redis会单独创建（fork）一个子进程来进行持久化，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。

如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

虽然RDB有不少优点，但它的缺点也是不容忽视的。如果你对数据的完整性非常敏感，那么RDB方式就不太适合你，因为即使你每5分钟都持久化一次，当redis故障时，仍然会有近5分钟的数据丢失。所以，redis还提供了另一种持久化方式，那就是AOF。

![img](https://img-blog.csdnimg.cn/20210515154124143.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## 2、AOF

AOF，英文是Append Only File，即只允许追加不允许改写的文件。

如前面介绍的，AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍，就这么简单。

我们通过配置redis.conf中的appendonly yes就可以打开AOF功能。如果有写操作（如SET等），redis就会被追加到AOF文件的末尾。

默认的AOF持久化策略是每秒钟fsync一次（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据。

如果在追加日志时，恰好遇到磁盘空间满、inode满或断电等情况导致日志写入不完整，也没有关系，redis提供了redis-check-aof工具，可以用来进行日志修复。

因为采用了追加方式，如果不做任何处理的话，AOF文件会变得越来越大，为此，redis提供了AOF文件重写（rewrite）机制，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令，这就是重写机制的原理。

在进行AOF重写时，仍然是采用先写临时文件，全部完成后再替换的流程，所以断电、磁盘满等问题都不会影响AOF文件的可用性，这点大家可以放心。

AOF方式的另一个好处，我们通过一个“场景再现”来说明。某同学在操作redis时，不小心执行了FLUSHALL，导致redis内存中的数据全部被清空了，这是很悲剧的事情。不过这也不是世界末日，只要redis配置了AOF持久化方式，且AOF文件还没有被重写（rewrite），我们就可以用最快的速度暂停redis并编辑AOF文件，将最后一行的FLUSHALL命令删除，然后重启redis，就可以恢复redis的所有数据到FLUSHALL之前的状态了。是不是很神奇，这就是AOF持久化方式的好处之一。但是如果AOF文件已经被重写了，那就无法通过这种方法来恢复数据了。

虽然优点多多，但AOF方式也同样存在缺陷，比如在同样数据规模的情况下，AOF文件要比RDB文件的体积大。而且，AOF方式的恢复速度也要慢于RDB方式。

如果你直接执行BGREWRITEAOF命令，那么redis会生成一个全新的AOF文件，其中便包括了可以恢复现有数据的最少的命令集。

如果运气比较差，AOF文件出现了被写坏的情况，也不必过分担忧，redis并不会贸然加载这个有问题的AOF文件，而是报错退出。这时可以通过以下步骤来修复出错的文件：

1.备份被写坏的AOF文件
2.运行redis-check-aof –fix进行修复
3.用diff -u来看下两个文件的差异，确认问题点
4.重启redis，加载修复后的AOF文件

![img](https://img-blog.csdnimg.cn/2021051515414762.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## 3、AOF重写

AOF重写的内部运行原理，我们有必要了解一下。

在重写即将开始之际，redis会创建（fork）一个“重写子进程”，这个子进程会首先读取现有的AOF文件，并将其包含的指令进行分析压缩并写入到一个临时文件中。

与此同时，主工作进程会将新接收到的写指令一边累积到内存缓冲区中，一边继续写入到原有的AOF文件中，这样做是保证原有的AOF文件的可用性，避免在重写过程中出现意外。

当“重写子进程”完成重写工作后，它会给父进程发一个信号，父进程收到信号后就会将内存中缓存的写指令追加到新AOF文件中。

当追加结束后，redis就会用新AOF文件来代替旧AOF文件，之后再有新的写指令，就都会追加到新的AOF文件中了。
4、如何选择RDB和AOF

对于我们应该选择RDB还是AOF，官方的建议是两个同时使用。这样可以提供更可靠的持久化方案。

# 八、Redis集群

![img](https://img-blog.csdnimg.cn/20210515154200480.png)

## 1、主从同步简介

像MySQL一样，redis是支持主从同步的，而且也支持一主多从以及多级从结构。

主从结构，一是为了纯粹的冗余备份，二是为了提升读性能，比如很消耗性能的SORT就可以由从服务器来承担。

redis的主从同步是异步进行的，这意味着主从同步不会影响主逻辑，也不会降低redis的处理性能。

主从架构中，可以考虑关闭主服务器的数据持久化功能，只让从服务器进行持久化，这样可以提高主服务器的处理性能。

在主从架构中，从服务器通常被设置为只读模式，这样可以避免从服务器的数据被误修改。但是从服务器仍然可以接受CONFIG等指令，所以还是不应该将从服务器直接暴露到不安全的网络环境中。如果必须如此，那可以考虑给重要指令进行重命名，来避免命令被外人误执行。

## 2、主从同步原理

从服务器会向主服务器发出SYNC指令，当主服务器接到此命令后，就会调用BGSAVE指令来创建一个子进程专门进行数据持久化工作，也就是将主服务器的数据写入RDB文件中。在数据持久化期间，主服务器将执行的写指令都缓存在内存中。

在BGSAVE指令执行完成后，主服务器会将持久化好的RDB文件发送给从服务器，从服务器接到此文件后会将其存储到磁盘上，然后再将其读取到内存中。这个动作完成后，主服务器会将这段时间缓存的写指令再以redis协议的格式发送给从服务器。

另外，要说的一点是，即使有多个从服务器同时发来SYNC指令，主服务器也只会执行一次BGSAVE，然后把持久化好的RDB文件发给多个下游。在redis2.8版本之前，如果从服务器与主服务器因某些原因断开连接的话，都会进行一次主从之间的全量的数据同步；而在2.8版本之后，redis支持了效率更高的增量同步策略，这大大降低了连接断开的恢复成本。

主服务器会在内存中维护一个缓冲区，缓冲区中存储着将要发给从服务器的内容。从服务器在与主服务器出现网络瞬断之后，从服务器会尝试再次与主服务器连接，一旦连接成功，从服务器就会把“希望同步的主服务器ID”和“希望请求的数据的偏移位置（replication offset）”发送出去。主服务器接收到这样的同步请求后，首先会验证主服务器ID是否和自己的ID匹配，其次会检查“请求的偏移位置”是否存在于自己的缓冲区中，如果两者都满足的话，主服务器就会向从服务器发送增量内容。

增量同步功能，需要服务器端支持全新的PSYNC指令。这个指令，只有在redis-2.8之后才具有。

![img](https://img-blog.csdnimg.cn/20210515154237798.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

# 九、redis.conf配置文件简介

[【Redis 4】配置文件redis.conf简介](https://blog.csdn.net/guorui_java/article/details/116767168)

# 十、Redis常见问题

## 1、缓存穿透

每次针对某key的请求在缓存中获取不到，请求都会压到数据库，从而可能压垮数据库。

## 2、缓存击穿

某key对应的数据存在，但在Redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期，一般都会从数据库中加载数据并设置到缓存中，这个时候大并发的请求可能会瞬间把数据库压垮。

## 3、缓存雪崩

某key对应的数据存在，但在Redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期，一般都会从数据库中加载数据并设置到缓存中，这个时候大并发的请求可能会瞬间把数据库压垮。

缓存雪崩和缓存击穿的区别就在于是多个key还是某一个key。

## 4、分布式锁

使用Redis实现分布式锁

redis命令：set users 10 nx ex 12   原子性命令

```java
//使用uuid，解决锁释放的问题
@GetMapping
public void testLock() throws InterruptedException {
    String uuid = UUID.randomUUID().toString();
    Boolean b_lock = redisTemplate.opsForValue().setIfAbsent("lock", uuid, 10, TimeUnit.SECONDS);
    if(b_lock){
        Object value = redisTemplate.opsForValue().get("num");
        if(StringUtils.isEmpty(value)){
            return;
        }
        int num = Integer.parseInt(value + "");
        redisTemplate.opsForValue().set("num",++num);
        Object lockUUID = redisTemplate.opsForValue().get("lock");
        if(uuid.equals(lockUUID.toString())){
            redisTemplate.delete("lock");
        }
    }else{
        Thread.sleep(100);
        testLock();
    }
}
```

备注：可以通过lua脚本，保证分布式锁的原子性。

