[TOC]

# 1、Redis集群简介
## 1.1 RedisCluster概念
Redis的分布式解决方案，在3.0版本后推出的方案，有效地解决了Redis分布式的需求，当一个服务宕机可以快速的切换到另外一个服务。redis cluster主要是针对海量数据+高并发+高可用的场景。

# 2、SpringBoot整合Redis集群
## 2.1 核心依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>${spring-boot.version}</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>${redis-client.version}</version>
</dependency>
```
## 2.2 核心配置
```yml
spring:
  # Redis 集群
  redis:
    sentinel:
      # sentinel 配置
      master: mymaster
      nodes: 192.168.0.127:26379
      maxTotal: 60
      minIdle: 10
      maxWaitMillis: 10000
      testWhileIdle: true
      testOnBorrow: true
      testOnReturn: false
      timeBetweenEvictionRunsMillis: 10000
```
## 2.3 参数渲染类
```java
@ConfigurationProperties(prefix = "spring.redis.sentinel")
public class RedisParam {
    private String nodes ;
    private String master ;
    private Integer maxTotal ;
    private Integer minIdle ;
    private Integer maxWaitMillis ;
    private Integer timeBetweenEvictionRunsMillis ;
    private boolean testWhileIdle ;
    private boolean testOnBorrow ;
    private boolean testOnReturn ;
    // 省略GET和SET方法
}
```
## 2.4 集群配置文件
```java
@Configuration
@EnableConfigurationProperties(RedisParam.class)
public class RedisPool {
    @Resource
    private RedisParam redisParam ;
    @Bean("jedisSentinelPool")
    public JedisSentinelPool getRedisPool (){
        Set<String> sentinels = new HashSet<>();
        sentinels.addAll(Arrays.asList(redisParam.getNodes().split(",")));
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMaxTotal(redisParam.getMaxTotal());
        poolConfig.setMinIdle(redisParam.getMinIdle());
        poolConfig.setMaxWaitMillis(redisParam.getMaxWaitMillis());
        poolConfig.setTestWhileIdle(redisParam.isTestWhileIdle());
        poolConfig.setTestOnBorrow(redisParam.isTestOnBorrow());
        poolConfig.setTestOnReturn(redisParam.isTestOnReturn());
        poolConfig.setTimeBetweenEvictionRunsMillis(redisParam.getTimeBetweenEvictionRunsMillis());
        JedisSentinelPool redisPool = new JedisSentinelPool(redisParam.getMaster(), sentinels, poolConfig);
        return redisPool;
    }
    @Bean
    SpringUtil springUtil() {
        return new SpringUtil();
    }
    @Bean
    RedisListener redisListener() {
        return new RedisListener();
    }
}
```
## 2.5 配置Redis模板类
```java
@Configuration
public class RedisConfig {
    @Bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(factory);
        return stringRedisTemplate;
    }
}
```

# 3、模拟队列场景案例
生产者消费者模式：客户端监听消息队列，消息达到，消费者马上消费，如果消息队列里面没有消息，那么消费者就继续监听。基于Redis的LPUSH（BLPUSH）把消息入队，用 RPOP（BRPOP）获取消息的模式。

## 3.1 加锁解锁工具
```java
@Component
public class RedisLock {
    private static String keyPrefix = "RedisLock:";
    @Resource
    private JedisSentinelPool jedisSentinelPool;
    public boolean addLock(String key, long expire) {
        Jedis jedis = null;
        try {
            jedis = jedisSentinelPool.getResource();
            /*
             * nxxx的值只能取NX或者XX，如果取NX，则只有当key不存在是才进行set，如果取XX，则只有当key已经存在时才进行set
             * expx的值只能取EX或者PX，代表数据过期时间的单位，EX代表秒，PX代表毫秒。
             */
            String value = jedis.set(keyPrefix + key, "1", "nx", "ex", expire);
            return value != null;
        } catch (Exception e){
            e.printStackTrace();
        }finally {
            if (jedis != null) jedis.close();
        }
        return false;
    }
    public void removeLock(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisSentinelPool.getResource();
            jedis.del(keyPrefix + key);
        } finally {
            if (jedis != null) jedis.close();
        }
    }
}
```
## 3.2 消息消费
1）封装接口
```java
public interface RedisHandler  {
    /**
     * 队列名称
     */
    String queueName();

    /**
     * 队列消息内容
     */
    String consume (String msgBody);
}
```
2）接口实现
```java
@Component
public class LogAListen implements RedisHandler {
    private static final Logger LOG = LoggerFactory.getLogger(LogAListen.class) ;
    @Resource
    private RedisLock redisLock;
    @Override
    public String queueName() {
        return "LogA-key";
    }
    @Override
    public String consume(String msgBody) {
        // 加锁，防止消息重复投递
        String lockKey = "lock-order-uuid-A";
        boolean lock = false;
        try {
            lock = redisLock.addLock(lockKey, 60);
            if (!lock) {
                return "success";
            }
            LOG.info("LogA-key == >>" + msgBody);
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            if (lock) {
                redisLock.removeLock(lockKey);
            }
        }
        return "success";
    }
}
```
## 3.3 消息监听器
```java
public class RedisListener implements InitializingBean {
    /**
     * Redis 集群
     */
    @Resource
    private JedisSentinelPool jedisSentinelPool;
    private List<RedisHandler> handlers = null;
    private ExecutorService product = null;
    private ExecutorService consumer = null;
    /**
     * 初始化配置
     */
    @Override
    public void afterPropertiesSet() {
        handlers = SpringUtil.getBeans(RedisHandler.class) ;
        product = new ThreadPoolExecutor(10,15,60 * 3,
                TimeUnit.SECONDS,new SynchronousQueue<>());
        consumer = new ThreadPoolExecutor(10,15,60 * 3,
                TimeUnit.SECONDS,new SynchronousQueue<>());
        for (RedisHandler redisHandler : handlers){
            product.execute(() -> {
                redisTask(redisHandler);
            });
        }
    }
    /**
     * 队列监听
     */
    public void redisTask (RedisHandler redisHandler){
        Jedis jedis = null ;
        while (true){
            try {
                jedis = jedisSentinelPool.getResource() ;
                List<String> msgBodyList = jedis.brpop(0, redisHandler.queueName());
                if (msgBodyList != null && msgBodyList.size()>0){
                    consumer.execute(() -> {
                        redisHandler.consume(msgBodyList.get(1)) ;
                    });
                }
            } catch (Exception e){
                e.printStackTrace();
            } finally {
                if (jedis != null) jedis.close();
            }
        }
    }
}
```
## 3.4 消息生产者
```java
@Service
public class RedisServiceImpl implements RedisService {
    @Resource
    private JedisSentinelPool jedisSentinelPool;
    @Override
    public void saveQueue(String queueKey, String msgBody) {
        Jedis jedis = null;
        try {
            jedis = jedisSentinelPool.getResource();
            jedis.lpush(queueKey,msgBody) ;
        } catch (Exception e){
          e.printStackTrace();
        } finally {
            if (jedis != null) jedis.close();
        }
    }
}
```

## 3.5 场景测试接口
```java
@RestController
public class RedisController {
    @Resource
    private RedisService redisService ;
    /**
     * 队列推消息
     */
    @RequestMapping("/saveQueue")
    public String saveQueue (){
        MsgBody msgBody = new MsgBody() ;
        msgBody.setName("LogAModel");
        msgBody.setDesc("描述");
        msgBody.setCreateTime(new Date());
        redisService.saveQueue("LogA-key", JSONObject.toJSONString(msgBody));
        return "success" ;
    }
}
```