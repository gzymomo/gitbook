## Redis Sentinel 介绍

Redis Sentinel：哨兵，放哨，看中文名字就知道它是一种 Redis 高可用解决方案，主要是针对 Redis 主从模式实现主从节点监控、故障自动切换。

没有 Redis Sentinel 架构之前，如果主节点挂了，需要运维人员手动进行主从切换，然后更新所有用到的 Redis IP 地址参数再重新启动系统，所有恢复操作都需要人为干预，如果半夜挂了，如果系统很多，如果某个操作搞错了，等等，这对运维人员来说简直就是恶梦。

有了 Redis Sentinel，主从节点故障都是自动化切换，应用程序参数什么也不用改，对于客户端来说都是透明无缝切换的，运维人员再也不用担惊受怕了。

如一个 1 主 3 从的 Redis 架构如下：

![img](https://img2020.cnblogs.com/other/1218593/202009/1218593-20200902135125854-869020623.png)

加入 Redis 哨兵之后的架构如下：

![img](https://img2020.cnblogs.com/other/1218593/202009/1218593-20200902135126568-806604978.png)

为了保证 Redis Sentinel 架构自身的高可用性，自身也不能有单点，一般也要由 3 个或以上 Sentinel 节点组成，一起负责监控主从节点，当大部分 Sentinel 节点认为主节点不可用时，会选一个 Sentinel 节点进行故障切换。

哨后架构的搭建这里不展开了，大家可以移步公众号Java技术栈，关于 Redis 单机、哨后、集群的搭建、以及往期 Redis 和 Spring Boot 集成、分布式锁实战教程等在公众号Java技术栈后台回复redis进行翻阅。

## Spring Boot & Redis Sentinel 实战

搞懂了 Redis 哨兵的用处之后，再来看一下 Spring Boot 如何快速集成 Redis Sentinel。

要知道如何自动配置 Redis Sentinel，除了看官方教程（不一定详细），最好的方式就是看源码了。

看过上篇的都知道 Spring Boot Redis 的默认客户端是：Lettuce，我们再来看下 LettuceConnectionFactory 的自动配置源码：

> org.springframework.boot.autoconfigure.data.redis.LettuceConnectionConfiguration

![img](https://img2020.cnblogs.com/other/1218593/202009/1218593-20200902135127141-342106968.png)

如源码所示，我们可以知道 Redis 连接自动配置的优先顺序是：

> Redis Sentinel（哨兵） > Redis Cluster（集群） > Standalone（单机）

哨兵模式优先极是最高的，再来看下 getSentinelConfig 方法源码：

![img](https://img2020.cnblogs.com/other/1218593/202009/1218593-20200902135127565-650159853.png)

master、sentinels 是必须参数，password、SentinelPassword 是可选的，database 默认是第 0 个数据库。

配置参数源码：

> org.springframework.boot.autoconfigure.data.redis.RedisProperties.Sentinel

![img](https://img2020.cnblogs.com/other/1218593/202009/1218593-20200902135128133-799804036.png)

所以，我们只需要提供 Redis Sentinel 的基本配置参数即可。

application.yml 配置如下：

```
spring:
  profiles:
    active: sentinel

---
spring:
  profiles: standalone
  redis:
    host: 192.168.1.110
    port: 6379
    password: redis2020
    database: 1

---
spring:
  profiles: sentinel
  redis:
    password: redis2020
    sentinel:
      master: mymaster
      nodes:
        - 192.168.1.110:26379
        - 192.168.1.111:26379
        - 192.168.1.112:26379
```

这样就能在单机和哨兵模式下切换，这是 yaml 配置的优势，一个文件搞定多套环境配置，不熟悉的关注公众号Java技术栈阅读我写的 Spring Boot 系列文章，当然这里配置两套只是为了测试，实际项目这样做没有意义。

配置成功后，该怎么使用还是怎么使用了，Redis Sentinel 对于客户端来说是透明的。