[Redis高级项目实战，都0202年了，还不会Redis？](https://www.cnblogs.com/chenyanbin/p/13506946.html)



## 面试专题[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#面试专题)

### **什么是分布式锁？**[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#什么是分布式锁？)

　　首先，为了确保分布式锁可用，至少要满足以下三个条件

1. **互斥性**。在任意时刻，只有一个客户端能持有锁
2. **不会发生死锁**。即便有一个客户端在持有锁的期间奔溃而没有主动解锁，也能保证后续其他客户端能加锁
3. 解铃还须系铃人。**加锁**和**解锁必须是同一个客户端**，**客户端**自己**不能把别人加的锁给解了**

### **实现分布式锁方式？**[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#实现分布式锁方式？)

　　两种实现，下面都会有讲到

1. 采用**lua脚本**操作分布式锁
2. 采用**setnx、setex命令连用**的方式实现分布式锁

# 分布式锁的场景[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#分布式锁的场景)

## 什么是分布式锁？[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#什么是分布式锁？)

- **分布式锁**是**控制分布式系统**或**不同系统**之间**共同访问共享资源的**一种**锁**实现
- 如果不同的系统或同一个系统的不同主机之间共享了某个资源时，往往通过互斥来防止彼此干扰

## 为什么要有分布式锁？[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#为什么要有分布式锁？)

　　可以**保证**在**分布式**部署的**应用集群**中，**同**一个**方法**在同一**操作**只能**被****一台机器**上的**一个线程执行**。

### 设计要求[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#设计要求)

1. 可**重入**锁(避免死锁)
2. **获取**锁和**释放**锁**高可用**
3. **获取**锁和**释放**锁**高性能**

## 实现方案[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#实现方案)

1. **获取锁**，使用**setnx()**：SETNX key val：当且仅当key不存在时，set一个key为val的字符串，返回1
2. **若key存在，则什么都不做**，返回【0】加锁，锁的value值为当前占有锁服务器内网IP编号拼接任务标识
3. 在**释放锁**的**时**候进行判断。并**使用expire**命令**为锁添加一个超时时间**，超过该时间则**自动释放锁**
4. **返回1**则**成功获取锁**。**还设置**一个**获取**的**超时时间**，若**超过**这个**时间**则**放弃获取锁**，**setex**(key,value,expire)过期**以秒为单位**
5. **释放锁**的时候，**判断**是不是该**锁**(即value为当前服务器内网IP编号拼接任务标识)，若是该锁，则**执行delete**进行**锁释放**



# Redis分布式锁的实现[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#redis分布式锁的实现)

## 创建一个SpringBoot工程[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#创建一个springboot工程)

网址：https://start.spring.io/

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814222311751-1772794455.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814222311751-1772794455.png)

### 步骤[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#步骤)

　　1、启动类上加上注解@EnableScheduling

　　2、执行方法上加上注解@Scheduled

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814230859253-524877502.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814230859253-524877502.png)

## 打包并上传至Linux服务器中启动[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#打包并上传至linux服务器中启动)

　　准备3台Linux服务器，并将打好的jar包，上传至3台服务器中，然后启动

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814231810279-2069054679.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200814231810279-2069054679.png)

### nohub之持久化启动方式 [#](https://www.cnblogs.com/chenyanbin/p/13506946.html#nohub之持久化启动方式 )

```
nohup java -jar jar名称 &
```

#### 查看集群里面所有集群是否启动成功

```
1、先安装lsof：yum install lsof
2、验证：lsof -i:8080
```

# TCP三次握手[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#tcp三次握手)

## 查看本机TCP连接状态[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#查看本机tcp连接状态)

```
 netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

## 为什么要三次握手？[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#为什么要三次握手？)

　　**主要**是为了**防止已失效**的连接**请求报文段**突然**又传**到了**B**，因而报文错乱问题，假定A发出的第一个连接请求报文段并没有丢失，而是在某些网络节点长时间滞留，一直延迟到连接释放以后的某个时间才到达B，本来这是一个早已经失效的报文段。但B收到失效的连接请求报文段后，就误认为是A又发出一个新的连接请求，于是就向A发出确认报文段，同意建立连接。

　　假定**不采用三次握手**，那么只要**B发出确认**，**新的连接就建立**，这样**一直等待A**发来数据，**B的许多资源**就这样**白白浪费**了。

### 图解[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#图解)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200815093340858-1369690611.jpg)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200815093340858-1369690611.jpg)

## 有3次握手了，为啥还有4次挥手？[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#有3次握手了，为啥还有4次挥手？)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816164445553-367496821.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816164445553-367496821.png)

第一次挥手：主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，也就是主动关闭方告诉被动关闭方：我已经不 会再给你发数据了(当然，在fin包之前发送出去的数据，如果没有收到对应的ack确认报文，主动关闭方依然会重发这些数据)，但是，此时主动关闭方还可 以接受数据。
第二次挥手：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号）。
第三次挥手：被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，也就是告诉主动关闭方，我的数据也发送完了，不会再给你发数据了。
第四次挥手：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，至此，完成四次挥手。

### 作用[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#作用)

　　确保数据能够完整传输

# Redis分布式锁实现源码讲解[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#redis分布式锁实现源码讲解)

## 图文讲解[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#图文讲解)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816170658270-1232710320.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816170658270-1232710320.png)

## 步骤[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#步骤)

1. 分布式锁满足**两个条件**，一个是**加有效时间**的**锁**，一个是**高性能解锁**
2. 采用redis命令**setnx** (set if not exist)、**setex**（set expire value）实现
3. **解锁流程不能遗漏**，**否则导致**任务执行一次就**永不过期**
4. 将**加锁代码**和**任务逻辑放到try catch代码块**，**解锁**流程**放**到**finally代码块**

## 项目结构[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#项目结构)

### pom.xml[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#pom.xml)

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cyb</groupId>
    <artifactId>yb-mobile-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>yb-mobile-redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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

### application.properties[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#application.properties)

```
spring.redis.database=0
spring.redis.host=192.168.199.142
spring.redis.port=6379
spring.redis.password=12345678
server.port=9001
```

### RedisService.java[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#redisservice.java)

```
package com.cyb.ybmobileredis.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.*;
import org.springframework.stereotype.Service;

import java.io.Serializable;
import java.util.List;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * @ClassName：RedisService
 * @Description：TODO
 * @Author：chenyb
 * @Date：2020/8/16 5:39 下午
 * @Versiion：1.0
 */
@Service
public class RedisService {
    @Autowired
    private RedisTemplate redisTemplate;

    private static double size = Math.pow(2, 32);


    /**
     * 写入缓存
     *
     * @param key
     * @param offset   位 8Bit=1Byte
     * @return
     */
    public boolean setBit(String key, long offset, boolean isShow) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.setBit(key, offset, isShow);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 写入缓存
     *
     * @param key
     * @param offset
     * @return
     */
    public boolean getBit(String key, long offset) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            result = operations.getBit(key, offset);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }


    /**
     * 写入缓存
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 写入缓存设置时效时间
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value, Long expireTime) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            redisTemplate.expire(key, expireTime, TimeUnit.SECONDS);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 批量删除对应的value
     *
     * @param keys
     */
    public void remove(final String... keys) {
        for (String key : keys) {
            remove(key);
        }
    }


    /**
     * 删除对应的value
     *
     * @param key
     */
    public void remove(final String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }

    /**
     * 判断缓存中是否有对应的value
     *
     * @param key
     * @return
     */
    public boolean exists(final String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 读取缓存
     *
     * @param key
     * @return
     */
    public Object get(final String key) {
        Object result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = operations.get(key);
        return result;
    }

    /**
     * 哈希 添加
     *
     * @param key
     * @param hashKey
     * @param value
     */
    public void hmSet(String key, Object hashKey, Object value) {
        HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
        hash.put(key, hashKey, value);
    }

    /**
     * 哈希获取数据
     *
     * @param key
     * @param hashKey
     * @return
     */
    public Object hmGet(String key, Object hashKey) {
        HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
        return hash.get(key, hashKey);
    }

    /**
     * 列表添加
     *
     * @param k
     * @param v
     */
    public void lPush(String k, Object v) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        list.rightPush(k, v);
    }

    /**
     * 列表获取
     *
     * @param k
     * @param l
     * @param l1
     * @return
     */
    public List<Object> lRange(String k, long l, long l1) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        return list.range(k, l, l1);
    }

    /**
     * 集合添加
     *
     * @param key
     * @param value
     */
    public void add(String key, Object value) {
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        set.add(key, value);
    }

    /**
     * 集合获取
     *
     * @param key
     * @return
     */
    public Set<Object> setMembers(String key) {
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        return set.members(key);
    }

    /**
     * 有序集合添加
     *
     * @param key
     * @param value
     * @param scoure
     */
    public void zAdd(String key, Object value, double scoure) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        zset.add(key, value, scoure);
    }

    /**
     * 有序集合获取
     *
     * @param key
     * @param scoure
     * @param scoure1
     * @return
     */
    public Set<Object> rangeByScore(String key, double scoure, double scoure1) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        redisTemplate.opsForValue();
        return zset.rangeByScore(key, scoure, scoure1);
    }


    //第一次加载的时候将数据加载到redis中
    public void saveDataToRedis(String name) {
        double index = Math.abs(name.hashCode() % size);
        long indexLong = new Double(index).longValue();
        boolean availableUsers = setBit("availableUsers", indexLong, true);
    }

    //第一次加载的时候将数据加载到redis中
    public boolean getDataToRedis(String name) {

        double index = Math.abs(name.hashCode() % size);
        long indexLong = new Double(index).longValue();
        return getBit("availableUsers", indexLong);
    }

    /**
     * 有序集合获取排名
     *
     * @param key 集合名称
     * @param value 值
     */
    public Long zRank(String key, Object value) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        return zset.rank(key,value);
    }


    /**
     * 有序集合获取排名
     *
     * @param key
     */
    public Set<ZSetOperations.TypedTuple<Object>> zRankWithScore(String key, long start,long end) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        Set<ZSetOperations.TypedTuple<Object>> ret = zset.rangeWithScores(key,start,end);
        return ret;
    }

    /**
     * 有序集合添加
     *
     * @param key
     * @param value
     */
    public Double zSetScore(String key, Object value) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        return zset.score(key,value);
    }


    /**
     * 有序集合添加分数
     *
     * @param key
     * @param value
     * @param scoure
     */
    public void incrementScore(String key, Object value, double scoure) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        zset.incrementScore(key, value, scoure);
    }


    /**
     * 有序集合获取排名
     *
     * @param key
     */
    public Set<ZSetOperations.TypedTuple<Object>> reverseZRankWithScore(String key, long start,long end) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        Set<ZSetOperations.TypedTuple<Object>> ret = zset.reverseRangeByScoreWithScores(key,start,end);
        return ret;
    }

    /**
     * 有序集合获取排名
     *
     * @param key
     */
    public Set<ZSetOperations.TypedTuple<Object>> reverseZRankWithRank(String key, long start, long end) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        Set<ZSetOperations.TypedTuple<Object>> ret = zset.reverseRangeWithScores(key, start, end);
        return ret;
    }
}
```

### LockNxExJob.java[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#locknxexjob.java)

```
package com.cyb.ybmobileredis.schedule;

import com.cyb.ybmobileredis.service.RedisService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.net.Inet4Address;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.Enumeration;

/**
 * @ClassName：LockNxExJob
 * @Description：分布式获取锁和释放锁
 * @Author：chenyb
 * @Date：2020/8/16 5:44 下午
 * @Versiion：1.0
 */
@Service
public class LockNxExJob {
    private static final Logger logger = LoggerFactory.getLogger(LockNxExJob.class);
    @Autowired
    private RedisService redisService;
    @Autowired
    private RedisTemplate redisTemplate;
    private static String LOCK_PREFIX = "prefix_";
    @Scheduled(fixedRate = 8000)
    public void lockJob() {
        String lock = LOCK_PREFIX + "LockNxExJob";
        boolean nxRet=false;
        try{
            //redistemplate setnx操作
            nxRet = redisTemplate.opsForValue().setIfAbsent(lock,getHostIp());
            Object lockValue = redisService.get(lock);
            System.out.println(lockValue);
            //获取锁失败
            if(!nxRet){
                String value = (String)redisService.get(lock);
                //打印当前占用锁的服务器IP
                logger.info(System.currentTimeMillis()+" get lock fail,lock belong to:{}",value);
                return;
            }else{
                redisTemplate.opsForValue().set(lock,getHostIp(),3600);

                //获取锁成功
                logger.info(System.currentTimeMillis()+" start lock lockNxExJob success");
                Thread.sleep(4000);
            }
        }catch (Exception e){
            logger.error("lock error",e);

        }finally {
            if (nxRet){
                System.out.println("释放锁成功");
                redisService.remove(lock);
            }
        }
    }
    /**
     * 获取本机内网IP地址方法
     * @return
     */
    private static String getHostIp(){
        try{
            Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            while (allNetInterfaces.hasMoreElements()){
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()){
                    InetAddress ip = (InetAddress) addresses.nextElement();
                    if (ip != null
                            && ip instanceof Inet4Address
                            && !ip.isLoopbackAddress() //loopback地址即本机地址，IPv4的loopback范围是127.0.0.0 ~ 127.255.255.255
                            && ip.getHostAddress().indexOf(":")==-1){
                        return ip.getHostAddress();
                    }
                }
            }
        }catch(Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```

## 验证[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#验证)

　　打个jar运行在Linux上，一个在本地运行，**一个**获取锁**成功**，**一个**获取锁**失败**

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816223554492-1313888447.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200816223554492-1313888447.gif)

# Redis分布式锁可能出现的问题[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#redis分布式锁可能出现的问题)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200818223530355-1760608448.jpg)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200818223530355-1760608448.jpg)

　　**上面**我们已经使用**代码实现**了**分布式锁**的功能，**同一时刻****只能一把锁获取成功**。从上图可以看出，**极端情况下**，**第一个Server获取锁成功**后，**服务或者Redis宕机**了，会**导致Redis锁无法释放**的问题，**其他**的**Server**就**一直获取锁失败**。

## 模拟server获取锁宕机[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#模拟server获取锁宕机)

　　先把**项目跑起来**，**获取锁之后**，立马**kill -9 进程id**，**杀掉当前进程**，然后在运行项目，控制台就会一直提示，获取锁失败了。

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819000547581-1754769577.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819000547581-1754769577.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819001314018-1165781808.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819001314018-1165781808.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819001407749-2091216048.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819001407749-2091216048.gif)

## 解决方案(重点)[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#解决方案(重点))

1. 一次性执行一条命令就不会出现该情况发生，采用**Lua脚本**
2. Redis从2.6之后，支持**setnx、setex连用**

### lua脚本[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#lua脚本)

1. 在redource目录下新增一个后缀名为.lua结尾的文件
2. 编写lua脚本
3. 传入lua脚本的key和arg
4. 调用redisTemplate.execute方法执行脚本

#### 编写lua脚本

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819224859707-963451524.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819224859707-963451524.png)

```
local lockKey = KEYS[1]
local lockValue = KEYS[2]
-- setnx info
local result_1 = redis.call('SETNX',lockKey,lockValue)
if result_1 == true
then
local result_2 = redis.call('SETEX',lockKey,3600,lockValue)
return result_1
else
return result_1
end
```

#### 封装调用lua脚本方法

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819225018239-205926746.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819225018239-205926746.png)

```
    @Autowired
    private RedisTemplate redisTemplate;
    private DefaultRedisScript<Boolean> lockScript;

    /**
     * 获取lua结果
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public Boolean luaExpress(String key, String value) {
        lockScript = new DefaultRedisScript<>();
        lockScript.setScriptSource(
                new ResourceScriptSource(new ClassPathResource("add.lua"))
        );
        //设置返回值
        lockScript.setResultType(Boolean.class);
        //封装参数
        List<Object> keyList = new ArrayList<>();
        keyList.add(key);
        keyList.add(value);
        Boolean result = (Boolean) redisTemplate.execute(lockScript, keyList);
        return result;
    }
```

#### 改造之前的分布式锁方法

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819225150156-161111106.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819225150156-161111106.png)

```
package com.cyb.ybmobileredis.schedule;

import com.cyb.ybmobileredis.service.RedisService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;

import java.net.Inet4Address;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

/**
 * @ClassName：LockNxExJob
 * @Description：分布式获取锁和释放锁
 * @Author：chenyb
 * @Date：2020/8/16 5:44 下午
 * @Versiion：1.0
 */
@Service
public class LockNxExJob {
    private static final Logger logger = LoggerFactory.getLogger(LockNxExJob.class);
    @Autowired
    private RedisService redisService;
    @Autowired
    private RedisTemplate redisTemplate;
    private static String LOCK_PREFIX = "prefix_";
    private DefaultRedisScript<Boolean> lockScript;
//一般分布式锁
//    @Scheduled(fixedRate = 8000)
//    public void lockJob() {
//        String lock = LOCK_PREFIX + "LockNxExJob";
//        boolean nxRet = false;
//        try {
//            //redistemplate setnx操作
//            nxRet = redisTemplate.opsForValue().setIfAbsent(lock, getHostIp());
//            Object lockValue = redisService.get(lock);
//            System.out.println(lockValue);
//            //获取锁失败
//            if (!nxRet) {
//                String value = (String) redisService.get(lock);
//                //打印当前占用锁的服务器IP
//                logger.info(System.currentTimeMillis() + " get lock fail,lock belong to:{}", value);
//                return;
//            } else {
//                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);
//
//                //获取锁成功
//                logger.info(System.currentTimeMillis() + " start lock lockNxExJob success");
//                Thread.sleep(4000);
//            }
//        } catch (Exception e) {
//            logger.error("lock error", e);
//
//        } finally {
//            if (nxRet) {
//                System.out.println("释放锁成功");
//                redisService.remove(lock);
//            }
//        }
//    }

    /**
     * lua脚本方式分布式锁
     */
    @Scheduled(fixedRate = 8000)
    public void luaLockJob() {
        String lock = LOCK_PREFIX + "LockNxExJob";
        boolean nxRet = false;
        try {
            //redistemplate setnx操作
            nxRet = luaExpress(lock,getHostIp());
            Object lockValue = redisService.get(lock);
            System.out.println(lockValue);
            //获取锁失败
            if (!nxRet) {
                String value = (String) redisService.get(lock);
                //打印当前占用锁的服务器IP
                logger.info(System.currentTimeMillis() + " lua get lock fail,lock belong to:{}", value);
                return;
            } else {
                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);

                //获取锁成功
                logger.info(System.currentTimeMillis() + " lua start lock lockNxExJob success");
                Thread.sleep(4000);
            }
        } catch (Exception e) {
            logger.error("lua lock error", e);

        } finally {
            if (nxRet) {
                System.out.println("lua 释放锁成功");
                redisService.remove(lock);
            }
        }
    }
    /**
     * 获取lua结果
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public Boolean luaExpress(String key, String value) {
        lockScript = new DefaultRedisScript<>();
        lockScript.setScriptSource(
                new ResourceScriptSource(new ClassPathResource("add.lua"))
        );
        //设置返回值
        lockScript.setResultType(Boolean.class);
        //封装参数
        List<Object> keyList = new ArrayList<>();
        keyList.add(key);
        keyList.add(value);
        Boolean result = (Boolean) redisTemplate.execute(lockScript, keyList);
        return result;
    }

    /**
     * 获取本机内网IP地址方法
     *
     * @return
     */
    private static String getHostIp() {
        try {
            Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress ip = (InetAddress) addresses.nextElement();
                    if (ip != null
                            && ip instanceof Inet4Address
                            && !ip.isLoopbackAddress() //loopback地址即本机地址，IPv4的loopback范围是127.0.0.0 ~ 127.255.255.255
                            && ip.getHostAddress().indexOf(":") == -1) {
                        return ip.getHostAddress();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### 验证[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#验证)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819231306639-1947883497.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819231306639-1947883497.gif)

### 补充解决Redis中的key乱码问题[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#补充解决redis中的key乱码问题)

　　只需要添加RedisConfig.java配置文件即可

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819231856166-1949069750.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200819231856166-1949069750.png)

```
package com.cyb.ybmobileredis.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * @ClassName：RedisConfig
 * @Description：Redis配置类
 * @Author：chenyb
 * @Date：2020/8/16 11:48 下午
 * @Versiion：1.0
 */
@Configuration //当前类为配置类
public class RedisConfig {
    @Bean //redisTemplate注入到Spring容器
    public RedisTemplate<String,String> redisTemplate(RedisConnectionFactory factory){
        RedisTemplate<String,String> redisTemplate=new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        redisTemplate.setConnectionFactory(factory);
        //key序列化
        redisTemplate.setKeySerializer(redisSerializer);
        //value序列化
        redisTemplate.setValueSerializer(redisSerializer);
        //value hashmap序列化
        redisTemplate.setHashKeySerializer(redisSerializer);
        //key hashmap序列化
        redisTemplate.setHashValueSerializer(redisSerializer);
        return redisTemplate;
    }
}
```

### RedisConnection实现分布式锁[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#redisconnection实现分布式锁)

#### 简介

　　RedisConnection实现分布式锁的方式，采用redisTemplate操作redisConnection实现setnx和setex两个命令连用

#### 代码实现

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200820001936699-723207755.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200820001936699-723207755.png)

```
package com.cyb.ybmobileredis.schedule;

import com.cyb.ybmobileredis.service.RedisService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.connection.RedisStringCommands;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.core.types.Expiration;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;

import java.net.Inet4Address;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

/**
 * @ClassName：LockNxExJob
 * @Description：分布式获取锁和释放锁
 * @Author：chenyb
 * @Date：2020/8/16 5:44 下午
 * @Versiion：1.0
 */
@Service
public class LockNxExJob {
    private static final Logger logger = LoggerFactory.getLogger(LockNxExJob.class);
    @Autowired
    private RedisService redisService;
    @Autowired
    private RedisTemplate redisTemplate;
    private static String LOCK_PREFIX = "prefix_";
    private DefaultRedisScript<Boolean> lockScript;
//一般分布式锁
//    @Scheduled(fixedRate = 8000)
//    public void lockJob() {
//        String lock = LOCK_PREFIX + "LockNxExJob";
//        boolean nxRet = false;
//        try {
//            //redistemplate setnx操作
//            nxRet = redisTemplate.opsForValue().setIfAbsent(lock, getHostIp());
//            Object lockValue = redisService.get(lock);
//            System.out.println(lockValue);
//            //获取锁失败
//            if (!nxRet) {
//                String value = (String) redisService.get(lock);
//                //打印当前占用锁的服务器IP
//                logger.info(System.currentTimeMillis() + " get lock fail,lock belong to:{}", value);
//                return;
//            } else {
//                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);
//
//                //获取锁成功
//                logger.info(System.currentTimeMillis() + " start lock lockNxExJob success");
//                Thread.sleep(4000);
//            }
//        } catch (Exception e) {
//            logger.error("lock error", e);
//
//        } finally {
//            if (nxRet) {
//                System.out.println("释放锁成功");
//                redisService.remove(lock);
//            }
//        }
//    }

    /**
     * lua脚本方式分布式锁
     */
//    @Scheduled(fixedRate = 8000)
//    public void luaLockJob() {
//        String lock = LOCK_PREFIX + "LockNxExJob";
//        boolean nxRet = false;
//        try {
//            //redistemplate setnx操作
//            //nxRet = luaExpress(lock,getHostIp());
//            nxRet = setLock(lock,600);
//            Object lockValue = redisService.get(lock);
//            System.out.println(lockValue);
//            //获取锁失败
//            if (!nxRet) {
//                String value = (String) redisService.get(lock);
//                //打印当前占用锁的服务器IP
//                logger.info(System.currentTimeMillis() + " lua get lock fail,lock belong to:{}", value);
//                return;
//            } else {
//                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);
//
//                //获取锁成功
//                logger.info(System.currentTimeMillis() + " lua start lock lockNxExJob success");
//                Thread.sleep(4000);
//            }
//        } catch (Exception e) {
//            logger.error("lua lock error", e);
//
//        } finally {
//            if (nxRet) {
//                System.out.println("lua 释放锁成功");
//                redisService.remove(lock);
//            }
//        }
//    }
    /**
     * setnx和setex连用分布式锁
     */
    @Scheduled(fixedRate = 8000)
    public void setLockJob() {
        String lock = LOCK_PREFIX + "LockNxExJob";
        boolean nxRet = false;
        try {
            //redistemplate setnx操作
            //nxRet = luaExpress(lock,getHostIp());
            nxRet = setLock(lock,getHostIp(),3);
            Object lockValue = redisService.get(lock);
            System.out.println(lockValue);
            //获取锁失败
            if (!nxRet) {
                String value = (String) redisService.get(lock);
                //打印当前占用锁的服务器IP
                logger.info(System.currentTimeMillis() + " setnx and setex get lock fail,lock belong to:{}", value);
                return;
            } else {
                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);

                //获取锁成功
                logger.info(System.currentTimeMillis() + " setnx and setex start lock lockNxExJob success");
                Thread.sleep(4000);
            }
        } catch (Exception e) {
            logger.error(" setnx and setex lock error", e);

        } finally {
            if (nxRet) {
                System.out.println(" setnx and setex 释放锁成功");
                redisService.remove(lock);
            }
        }
    }

    /**
     * setnx和setex连用
     * @param key 键
     * @param value 值
     * @param expire 超时时间
     * @return
     */
    public boolean setLock(String key,String value,long expire){
        try{
            Boolean result=(boolean)redisTemplate.execute(new RedisCallback<Boolean>() {

                @Override
                public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                    return connection.set(key.getBytes(),value.getBytes(),Expiration.seconds(expire),RedisStringCommands.SetOption.ifAbsent());
                }
            });
            return result;
        }catch (Exception e){
            logger.error("set redis occured an exception",e);
        }
        return false;
    }
    /**
     * 获取lua结果
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public Boolean luaExpress(String key, String value) {
        lockScript = new DefaultRedisScript<>();
        lockScript.setScriptSource(
                new ResourceScriptSource(new ClassPathResource("add.lua"))
        );
        //设置返回值
        lockScript.setResultType(Boolean.class);
        //封装参数
        List<Object> keyList = new ArrayList<>();
        keyList.add(key);
        keyList.add(value);
        Boolean result = (Boolean) redisTemplate.execute(lockScript, keyList);
        return result;
    }

    /**
     * 获取本机内网IP地址方法
     *
     * @return
     */
    private static String getHostIp() {
        try {
            Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress ip = (InetAddress) addresses.nextElement();
                    if (ip != null
                            && ip instanceof Inet4Address
                            && !ip.isLoopbackAddress() //loopback地址即本机地址，IPv4的loopback范围是127.0.0.0 ~ 127.255.255.255
                            && ip.getHostAddress().indexOf(":") == -1) {
                        return ip.getHostAddress();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### 测试[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#测试)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200820002558173-1162135335.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200820002558173-1162135335.gif)

## 分布式锁优化细节(重点)[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#分布式锁优化细节(重点))

　　**上面**几个**案例**，已经**实现了分布式锁**的功能，但是**极端情况下**，**ServerA**程序还**没执行完**，**ServerB程序执行完**，**把锁释放掉了**，就会造成A的锁释放掉了，这不是扯嘛，**ServerA还没执行完，锁就被其他人释放了**。解决方案：**释放的时候，使用lua，通过get方法获取value，判断value是否等于本机ip，是自己的才能释放**

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200821234232104-937570704.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200821234232104-937570704.png)

 

```
package com.cyb.ybmobileredis.schedule;

import com.cyb.ybmobileredis.service.RedisService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.connection.RedisStringCommands;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.core.types.Expiration;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Service;

import java.net.Inet4Address;
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

/**
 * @ClassName：LockNxExJob
 * @Description：分布式获取锁和释放锁
 * @Author：chenyb
 * @Date：2020/8/16 5:44 下午
 * @Versiion：1.0
 */
@Service
public class LockNxExJob {
    private static final Logger logger = LoggerFactory.getLogger(LockNxExJob.class);
    @Autowired
    private RedisService redisService;
    @Autowired
    private RedisTemplate redisTemplate;
    private static String LOCK_PREFIX = "prefix_";
    private DefaultRedisScript<Boolean> lockScript;
//一般分布式锁
//    @Scheduled(fixedRate = 8000)
//    public void lockJob() {
//        String lock = LOCK_PREFIX + "LockNxExJob";
//        boolean nxRet = false;
//        try {
//            //redistemplate setnx操作
//            nxRet = redisTemplate.opsForValue().setIfAbsent(lock, getHostIp());
//            Object lockValue = redisService.get(lock);
//            System.out.println(lockValue);
//            //获取锁失败
//            if (!nxRet) {
//                String value = (String) redisService.get(lock);
//                //打印当前占用锁的服务器IP
//                logger.info(System.currentTimeMillis() + " get lock fail,lock belong to:{}", value);
//                return;
//            } else {
//                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);
//
//                //获取锁成功
//                logger.info(System.currentTimeMillis() + " start lock lockNxExJob success");
//                Thread.sleep(4000);
//            }
//        } catch (Exception e) {
//            logger.error("lock error", e);
//
//        } finally {
//            if (nxRet) {
//                System.out.println("释放锁成功");
//                redisService.remove(lock);
//            }
//        }
//    }

    /**
     * lua脚本方式分布式锁
     */
//    @Scheduled(fixedRate = 8000)
//    public void luaLockJob() {
//        String lock = LOCK_PREFIX + "LockNxExJob";
//        boolean nxRet = false;
//        try {
//            //redistemplate setnx操作
//            //nxRet = luaExpress(lock,getHostIp());
//            nxRet = setLock(lock,600);
//            Object lockValue = redisService.get(lock);
//            System.out.println(lockValue);
//            //获取锁失败
//            if (!nxRet) {
//                String value = (String) redisService.get(lock);
//                //打印当前占用锁的服务器IP
//                logger.info(System.currentTimeMillis() + " lua get lock fail,lock belong to:{}", value);
//                return;
//            } else {
//                redisTemplate.opsForValue().set(lock, getHostIp(), 3600000);
//
//                //获取锁成功
//                logger.info(System.currentTimeMillis() + " lua start lock lockNxExJob success");
//                Thread.sleep(4000);
//            }
//        } catch (Exception e) {
//            logger.error("lua lock error", e);
//
//        } finally {
//            if (nxRet) {
//                System.out.println("lua 释放锁成功");
//                redisService.remove(lock);
//            }
//        }
//    }

    /**
     * setnx和setex连用分布式锁
     */
    @Scheduled(fixedRate = 8000)
    public void setLockJob() {
        String lock = LOCK_PREFIX + "LockNxExJob";
        boolean nxRet = false;
        try {
            //redistemplate setnx操作
            //nxRet = luaExpress(lock,getHostIp());
            System.out.println("hostIp1="+getHostIp());
            nxRet = setLock(lock, getHostIp(), 30);
            Object lockValue = redisService.get(lock);
            System.out.println(lockValue);
            //获取锁失败
            if (!nxRet) {
                String value = (String) redisService.get(lock);
                //打印当前占用锁的服务器IP
                logger.info(System.currentTimeMillis() + " setnx and setex get lock fail,lock belong to:{}", value);
                return;
            } else {
                //获取锁成功
                logger.info(System.currentTimeMillis() + " setnx and setex start lock lockNxExJob success");
                Thread.sleep(4000);
            }
        } catch (Exception e) {
            logger.error(" setnx and setex lock error", e);

        } finally {
            if (nxRet) {
                System.out.println(" setnx and setex 释放锁成功");
                //redisService.remove(lock);
                //使用lua脚本释放锁
                System.out.println("hostIp2="+getHostIp());
                Boolean result = releaseLock(lock, getHostIp());
                System.out.println("状态:"+result);
            }
        }
    }

    /**
     * 释放锁操作
     *
     * @param key   键
     * @param value 值
     * @return
     */
    private boolean releaseLock(String key, String value) {
        lockScript = new DefaultRedisScript<Boolean>();
        lockScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("unlock.lua")));
        lockScript.setResultType(Boolean.class);
        //封装参数
        List<Object> keyList = new ArrayList<>();
        keyList.add(key);
        keyList.add(value);
        Boolean result = (Boolean) redisTemplate.execute(lockScript, keyList);
        return result;
    }

    /**
     * setnx和setex连用
     *
     * @param key    键
     * @param value  值
     * @param expire 超时时间
     * @return
     */
    public boolean setLock(String key, String value, long expire) {
        try {
            Boolean result = (boolean) redisTemplate.execute(new RedisCallback<Boolean>() {

                @Override
                public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                    return connection.set(key.getBytes(), value.getBytes(), Expiration.seconds(expire), RedisStringCommands.SetOption.ifAbsent());
                }
            });
            return result;
        } catch (Exception e) {
            logger.error("set redis occured an exception", e);
        }
        return false;
    }

    /**
     * 获取lua结果
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public Boolean luaExpress(String key, String value) {
        lockScript = new DefaultRedisScript<>();
        lockScript.setScriptSource(
                new ResourceScriptSource(new ClassPathResource("add.lua"))
        );
        //设置返回值
        lockScript.setResultType(Boolean.class);
        //封装参数
        List<Object> keyList = new ArrayList<>();
        keyList.add(key);
        keyList.add(value);
        Boolean result = (Boolean) redisTemplate.execute(lockScript, keyList);
        return result;
    }

    /**
     * 获取本机内网IP地址方法
     *
     * @return
     */
    private static String getHostIp() {
        try {
            Enumeration<NetworkInterface> allNetInterfaces = NetworkInterface.getNetworkInterfaces();
            while (allNetInterfaces.hasMoreElements()) {
                NetworkInterface netInterface = (NetworkInterface) allNetInterfaces.nextElement();
                Enumeration<InetAddress> addresses = netInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress ip = (InetAddress) addresses.nextElement();
                    if (ip != null
                            && ip instanceof Inet4Address
                            && !ip.isLoopbackAddress() //loopback地址即本机地址，IPv4的loopback范围是127.0.0.0 ~ 127.255.255.255
                            && ip.getHostAddress().indexOf(":") == -1) {
                        return ip.getHostAddress();
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### unlock.lua脚本[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#unlock.lua脚本)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200821235129478-45459808.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200821235129478-45459808.png)

 

```
local lockKey = KEYS[1]
local lockValue = KEYS[2]

-- get key
local result_1 = redis.call('get', lockKey)
if result_1 == lockValue
then
local result_2= redis.call('del', lockKey)
return result_2
else
return false
end
```

### 演示[#](https://www.cnblogs.com/chenyanbin/p/13506946.html#演示)

　　为了演示方便，我把失效时间设置短一点，8秒

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200822000116676-465132010.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200822000116676-465132010.gif)