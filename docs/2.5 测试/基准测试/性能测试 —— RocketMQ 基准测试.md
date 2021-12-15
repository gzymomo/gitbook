# 1. 概述

> FROM [《消息中间件 Apache RocketMQ》](https://www.oschina.net/p/rocketmq)
>
> RocketMQ 是一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。同时，广泛应用于多个领域，包括异步通信解耦、企业解决方案、金融支付、电信、电子商务、快递物流、广告营销、社交、即时通信、移动应用、手游、视频、物联网、车联网等。
>
> 具有以下特点：
>
> - 能够保证严格的消息顺序
> - 提供丰富的消息拉取模式
> - 高效的订阅者水平扩展能力
> - 实时的消息订阅机制
> - 亿级消息堆积能力
> - Metaq 3.0 版本改名，产品名称改为 RocketMQ

2013 年才开始使用 RocketMQ ，主要选型的原因是，Java 语言实现的高性能的消息队列中间件，可能比 RabbitMQ 自己会更容易把控。正好当时官方也提供了三个很牛逼的文档，让自己心里也更有底了：

- [《RocketMQ 用户指南》](http://gd-rus-public.cn-hangzhou.oss-pub.aliyun-inc.com/attachment/201604/08/20160408164726/RocketMQ_userguide.pdf) 基于 RocketMQ 3 的版本。
- [《RocketMQ 原理简介》](http://gd-rus-public.cn-hangzhou.oss-pub.aliyun-inc.com/attachment/201604/08/20160408165024/RocketMQ_design.pdf) 基于 RocketMQ 3 的版本。
- [《RocketMQ 最佳实践》](http://gd-rus-public.cn-hangzhou.oss-pub.aliyun-inc.com/attachment/201604/08/20160408164929/RocketMQ_experience.pdf) 基于 RocketMQ 3 的版本。

2017 年的时候，因为想要进一步提（打）升（发）技（时）术（间），就学习了 RocketMQ 源码，并且写了第一套系列博客 [《RocketMQ 源码解析》](http://www.iocoder.cn/categories/RocketMQ/#) 。当然，一直以来比较遗憾的是，线上每天消息量都仅千万不到，没有机会体验 RocketMQ 大并发场景的使用。所以，我们今儿就来进行 RocketMQ 的性能基准测试，看看 RocketMQ 到底能跑多 666 。

# 2. 性能指标

在 [《消息队列 RocketMQ 企业铂金版》](https://common-buy.aliyun.com/?commodityCode=onspre#/buy) 中，我们看到阿里云按照 **TPS 的峰值**进行定价，分别是（单位：月）：

- 5 千条/秒 ：29480 元
- 1 万条/秒 ：34176 元
- 2 万条/秒 ：41220 元
- 5 万条/秒 ：52960 元
- 10 万条/秒 ：76440 元
- 20 万条/秒 ：170360 元
- 30 万条/秒 ：329232 元
- 50 万元/秒 ：571657 元
- 80 万条/秒 ：814081 元
- 100 万条/秒 ：1056505 元

> 艿艿：看到报价，瑟瑟发抖。
>
> 并且，Producer 发送一条消息，Consumer 消费一条消息，单独计算一条，也就是 2 条！

所以，本文我们，以 TPS 作为我们测试的重点。当然，我们也和阿里云相同的计算方式，按照消息大小基数为 **1KB** 。

# 3. 测试工具

目前可用于 RocketMQ 测试的工具，暂时没有。所幸，RocketMQ 的 [benchmark](https://github.com/apache/rocketmq/tree/master/example/src/main/java/org/apache/rocketmq/example/benchmark) 包下，提供了性能基准测试的工具。

- [Producer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Producer.java) ：测试普通 MQ 生产者的性能。
- [Consumer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Consumer.java) ：测试 MQ 消费者的性能。
- [TransactionProducer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/TransactionProducer.java) ：测试事务 MQ 生产者的性能。

事务消息，我们暂时不测试，所以我们需要使用的就是 Producer 和 Consumer 类。

# 4. 测试环境

- 型号 ：ecs.c5.xlarge

  > 艿艿：和我一样抠门（穷）的胖友，可以买竞价类型服务器，使用完后，做成镜像。等下次需要使用的时候，恢复一下。HOHO 。

- 系统 ：CentOS 7.6 64位

- CPU ：4 核

- 内存 ：8 GB

- 磁盘 ：100 GB ESSD 云盘，能够提供 6800 IOPS 。

- Java ：OpenJDK Runtime Environment (build 1.8.0_212-b04)

- RocketMQ ：4.5.1

我们买了两台阿里云 ECS ，用于搭建 RocketMQ 的一个**主从**集群。

我们会测试三轮，每一轮的目的分别是：

- 1、Producer 在不同并发下的发送 TPS 。此时，我们会使用异步复制、异步刷盘的 RocketMQ 集群。
- 2、Producer 在不同集群模式下的 RocketMQ 的发送 TPS 。此时，我们会使用相同的并发数。
- 3、Consumer 的消费 TPS 。

> 注意，用于 RocketMQ 使用同一的 CommitLog 存储，所以 Topic 数量或是 Topic 的队列数，不影响 Producer 的发送 TPS 。
>
> 当然，更多的 Topic 的队列数，可以更多的 Consumer 消费，考虑到我们测试纸使用单个 Consumer ，所以还是默认队列大小为 4 ，不进行调整。

# 5. 搭建集群

本小节，我们来简述下，如何搭建一个 RocketMQ 集群。

**1、编译 RocketMQ**

> 可以参考 [《Apache RocketMQ —— Quick Start》](https://rocketmq.apache.org/docs/quick-start/) 文章。
>
> 作为一个菜鸡，还是希望 RocketMQ 官网，能够提供友好的文档，可以参考 [Dubbo 官网](http://dubbo.apache.org/) 。
>
> 不过，无意中发现，原来 [Github RocketMQ - /docs/cn](https://github.com/apache/rocketmq/tree/master/docs/cn) 目录下，提供了中文文档，Amazing !



```
git clone https://github.com/apache/rocketmq
cd rocketmq
mvn -Prelease-all -DskipTests clean install -U
```



编译完成，在我们进入 distribution 目录下，就可以看到 RocketMQ 的发布包了。



```
$ cd distribution/target/rocketmq-4.5.1/rocketmq-4.5.1
$ ls

LICENSE  NOTICE  README.md  benchmark  bin  conf  lib
```



**2、启动 Namesrv**

在准备搭建 RocketMQ Master 主节点的机子上执行。



```
nohup sh bin/mqnamesrv &
```



启动完成后，查看日志。



```
tail -f ~/logs/rocketmqlogs/namesrv.log

2019-06-02 21:05:52 INFO main - The Name Server boot success. serializeType=JSON
```



**3、启动 Broker Master 主节点**

在 conf 目录下，提供了 RocketMQ 集群的配置：

- `2m-2s-async` ：两主两从，异步复制，异步刷盘。
- `2m-2s-sync` ：两主两从，同步复制，异步刷盘。

因为我们测试 RocketMQ 单集群，所以只需要使用上述配置的一主一从就可以了。不过，我们需要测试**同步刷盘**的情况，所以我们分别复制出 `2m-2s-async-sync` 和 `2m-2s-sync-sync` 目录，修改成同步刷盘的配置。修改方式是，修改 `broker-a.properties` 和 `broker-a-s.properties` 的 `flushDiskType=SYNC_FLUSH` 属性。

☺ OK，我们先使用 `2m-2s-async` 配置，启动一主一从 RocketMQ 集群，异步复制，异步刷盘。

因为我们的服务器是 4C8G ，内存相对小，所以我们修改下 `runbroker.sh` 脚本，将 Broker JVM 内存调小。如下：



```
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g"
```



然后，我们来启动 RocketMQ Broker Master 主节点。



```
nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a.properties  -n localhost:9876 &
```



- 通过 `-c` 参数，配置读取的主 Broker 配置。
- 通过 `-n` 参数，设置 RocketMQ Namesrv 地址。

启动完成后，查看日志。



```
tail -f ~/logs/rocketmqlogs/broker.log

2019-06-02 21:45:45 INFO main - The broker[broker-a, 172.19.83.161:10911] boot success. serializeType=JSON and name server is localhost:9876
```



**4、启动 Broker Slave 子节点**

在第二台阿里云 ECS 上，启动 Broker Slave 子节点。



```
nohup sh bin/mqbroker -c conf/2m-2s-async/broker-a-s.properties  -n 172.19.83.161:9876 &
```



- 通过 `-c` 参数，配置读取的从 Broker 配置。
- 通过 `-n` 参数，设置 RocketMQ Namesrv 地址。这里，一定要改成自己的 Namesrv 地址噢。

启动完成后，查看日志。



```
2019-06-02 22:01:46 INFO main - The broker[broker-a, 172.19.83.162:10911] boot success. serializeType=JSON and name server is 172.19.83.161:9876
2019-06-02 22:01:49 INFO BrokerControllerScheduledThread1 - Update slave topic config from master, 172.19.83.161:10911
2019-06-02 22:01:49 INFO BrokerControllerScheduledThread1 - Update slave consumer offset from master, 172.19.83.161:10911
```



**5、测试消息发送**

在第一台阿里云 ECS 上，测试消息发送。



```
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```



如果发送成功，我们会看到大量成功的发送日志。



```
BE2, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=0], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=AC1353A112006F94FA3E09EEA17003E5, offsetMsgId=AC1353A100002A9F000000000002BC96, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=1], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=AC1353A112006F94FA3E09EEA17103E6, offsetMsgId=AC1353A100002A9F000000000002BD4A, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=2], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=AC1353A112006F94FA3E09EEA17103E7, offsetMsgId=AC1353A100002A9F000000000002BDFE, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=3], queueOffset=249]
```



至此，我们的 RocketMQ 一主一从集群，就成功的搭建完成。下面，我们开始正片流程，开始基准测试流程。

# 6. 第一轮

这一轮，我们来测试 **Producer 在不同并发下的发送 TPS 。此时，我们会使用异步复制、异步刷盘的 RocketMQ 集群**。对应的 RocketMQ Broker 配置文件是，`2m-2s-async` 文件。

这里，我们将测试的并发数，分别是 1、8、16、32、64、128 。

在 benchmark 目录下，已经提供了对 [Producer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Producer.java) 和 [Consumer.java](https://github.com/apache/rocketmq/blob/master/example/src/main/java/org/apache/rocketmq/example/benchmark/Consumer.java) 的封装，可以直接使用。如下：



```
$ ls
LICENSE  NOTICE  README.md  benchmark  bin  conf  hs_err_pid4469.log  lib  nohup.out

$ cd benchmark
$ ls
consumer.sh  producer.sh  runclass.sh  tproducer.sh
```



要注意，benchmark 下的 shell 脚本，需要设置正确 JAVA_HOME 地址。😈 例如说，我是使用 yum 安装的 OpenJDK ，如下：



```
export JAVA_HOME=/usr/lib/jvm/java/
```



## 6.1 并发量为 1



```
sh producer.sh --w=1 --s=1024
```



- 通过 `--w` 参数，设置并发线程数。
- 通过 `--s` 参数，设置 Topic 消息大小。

执行结果如下：



```
Send TPS: 3770 Max RT: 198 Average RT:   0.259 Send Failed: 0 Response Failed: 0
Send TPS: 3803 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3817 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3819 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
Send TPS: 3846 Max RT: 198 Average RT:   0.255 Send Failed: 0 Response Failed: 0
Send TPS: 3813 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3784 Max RT: 198 Average RT:   0.259 Send Failed: 0 Response Failed: 0
Send TPS: 3833 Max RT: 198 Average RT:   0.255 Send Failed: 0 Response Failed: 0
Send TPS: 3797 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3828 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
Send TPS: 3848 Max RT: 198 Average RT:   0.255 Send Failed: 0 Response Failed: 0
Send TPS: 3836 Max RT: 198 Average RT:   0.255 Send Failed: 0 Response Failed: 0
Send TPS: 3819 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
Send TPS: 3841 Max RT: 198 Average RT:   0.254 Send Failed: 0 Response Failed: 0
Send TPS: 3821 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
Send TPS: 3813 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3813 Max RT: 198 Average RT:   0.257 Send Failed: 0 Response Failed: 0
Send TPS: 3818 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
Send TPS: 3816 Max RT: 198 Average RT:   0.256 Send Failed: 0 Response Failed: 0
```



- TPS ：3800 左右。
- Average RT ：0.256ms 左右。
- Max RT ：198ms 左右。

因为 `producer.sh` 是后台执行的，所以每次测试完，需要 kill 掉。例如：



```
$ ￥ps -ef | grep Producer
root      7197     1  0 10:31 pts/0    00:00:00 sh ./runclass.sh -Dorg.apache.rocketmq.client.sendSmartMsg=true org.apache.rocketmq.example.benchmark.Producer --w=8 --s=1024

$ kill -9 7197 # 直接强制 kill 哈。
```



## 6.2 并发量为 8



```
sh producer.sh --w=8 --s=1024
```



执行结果如下：



```
Send TPS: 18065 Max RT: 984 Average RT:   0.438 Send Failed: 0 Response Failed: 7
Send TPS: 17057 Max RT: 984 Average RT:   0.444 Send Failed: 0 Response Failed: 14
Send TPS: 15182 Max RT: 984 Average RT:   0.432 Send Failed: 0 Response Failed: 14
Send TPS: 18025 Max RT: 984 Average RT:   0.439 Send Failed: 0 Response Failed: 14
Send TPS: 18134 Max RT: 984 Average RT:   0.437 Send Failed: 0 Response Failed: 14
```



- TPS ：18000 左右。
- Average RT ：0.439ms 左右。
- Max RT ：984ms 左右。
- 存在极少量发送失败的消息。

## 6.3 并发量为 16



```
sh producer.sh --w=16 --s=1024
```



执行结果如下：



```
Send TPS: 22511 Max RT: 705 Average RT:   0.707 Send Failed: 0 Response Failed: 30
Send TPS: 22615 Max RT: 705 Average RT:   0.703 Send Failed: 0 Response Failed: 30
Send TPS: 22078 Max RT: 705 Average RT:   0.721 Send Failed: 0 Response Failed: 30
Send TPS: 21962 Max RT: 705 Average RT:   0.724 Send Failed: 0 Response Failed: 30
Send TPS: 22067 Max RT: 705 Average RT:   0.721 Send Failed: 0 Response Failed: 30
Send TPS: 21932 Max RT: 705 Average RT:   0.726 Send Failed: 0 Response Failed: 30
Send TPS: 22459 Max RT: 705 Average RT:   0.709 Send Failed: 0 Response Failed: 30
Send TPS: 15611 Max RT: 705 Average RT:   0.678 Send Failed: 0 Response Failed: 45
Send TPS: 22235 Max RT: 705 Average RT:   0.716 Send Failed: 0 Response Failed: 45
Send TPS: 22391 Max RT: 705 Average RT:   0.711 Send Failed: 0 Response Failed: 45
Send TPS: 21600 Max RT: 705 Average RT:   0.737 Send Failed: 0 Response Failed: 45
Send TPS: 22233 Max RT: 705 Average RT:   0.715 Send Failed: 0 Response Failed: 45
```



- TPS ：22000 左右。
- Average RT ：0.715ms 左右。
- Max RT ：705ms 左右。
- 存在极少量发送失败的消息。

## 6.4 并发量为 32



```
sh producer.sh --w=32 --s=1024
```



执行结果如下：



```
Send TPS: 17980 Max RT: 1102 Average RT:   1.159 Send Failed: 0 Response Failed: 93
Send TPS: 26759 Max RT: 1102 Average RT:   1.189 Send Failed: 0 Response Failed: 93
Send TPS: 22599 Max RT: 1102 Average RT:   1.183 Send Failed: 0 Response Failed: 124
Send TPS: 20627 Max RT: 1102 Average RT:   1.168 Send Failed: 0 Response Failed: 155
Send TPS: 24675 Max RT: 1102 Average RT:   1.196 Send Failed: 0 Response Failed: 155
Send TPS: 26706 Max RT: 1102 Average RT:   1.192 Send Failed: 0 Response Failed: 155
Send TPS: 18342 Max RT: 1102 Average RT:   1.132 Send Failed: 0 Response Failed: 186
Send TPS: 26663 Max RT: 1102 Average RT:   1.194 Send Failed: 0 Response Failed: 186
Send TPS: 19870 Max RT: 1102 Average RT:   1.166 Send Failed: 0 Response Failed: 217
Send TPS: 26842 Max RT: 1102 Average RT:   1.186 Send Failed: 0 Response Failed: 217
Send TPS: 27095 Max RT: 1102 Average RT:   1.175 Send Failed: 0 Response Failed: 217
Send TPS: 19093 Max RT: 1102 Average RT:   1.150 Send Failed: 0 Response Failed: 248
Send TPS: 26416 Max RT: 1102 Average RT:   1.205 Send Failed: 0 Response Failed: 248
Send TPS: 18032 Max RT: 1102 Average RT:   1.156 Send Failed: 0 Response Failed: 279
Send TPS: 26771 Max RT: 1102 Average RT:   1.189 Send Failed: 0 Response Failed: 279
Send TPS: 26543 Max RT: 1102 Average RT:   1.199 Send Failed: 0 Response Failed: 279
Send TPS: 20932 Max RT: 1102 Average RT:   1.148 Send Failed: 0 Response Failed: 310
Send TPS: 24804 Max RT: 1102 Average RT:   1.195 Send Failed: 0 Response Failed: 341
Send TPS: 22384 Max RT: 1102 Average RT:   1.177 Send Failed: 0 Response Failed: 341
Send TPS: 26247 Max RT: 1102 Average RT:   1.213 Send Failed: 0 Response Failed: 341
```



- TPS ：26000 左右。波动较大。
- Average RT ：1.213ms 左右。
- Max RT ：1102ms 左右。
- 存在极少量发送失败的消息。

## 6.5 并发量为 64



```
sh producer.sh --w=64 --s=1024
```



执行结果如下：



```
Send TPS: 25256 Max RT: 579 Average RT:   2.527 Send Failed: 0 Response Failed: 0
Send TPS: 17181 Max RT: 832 Average RT:   2.157 Send Failed: 0 Response Failed: 63
Send TPS: 29850 Max RT: 832 Average RT:   2.137 Send Failed: 0 Response Failed: 63
Send TPS: 29571 Max RT: 832 Average RT:   2.157 Send Failed: 0 Response Failed: 63
Send TPS: 29734 Max RT: 832 Average RT:   2.146 Send Failed: 0 Response Failed: 63
Send TPS: 20329 Max RT: 1108 Average RT:   2.036 Send Failed: 0 Response Failed: 126
Send TPS: 30445 Max RT: 1108 Average RT:   2.096 Send Failed: 0 Response Failed: 126
Send TPS: 20297 Max RT: 1108 Average RT:   2.038 Send Failed: 0 Response Failed: 189
Send TPS: 30163 Max RT: 1108 Average RT:   2.115 Send Failed: 0 Response Failed: 189
```



- TPS ：29000 左右。波动较大。
- Average RT ：2.115ms 左右。
- Max RT ：1108ms 左右。
- 存在极少量发送失败的消息。

## 6.6 并发量为 128



```
sh producer.sh --w=128 --s=1024
```



执行结果如下：



```
Send TPS: 22229 Max RT: 884 Average RT:   3.651 Send Failed: 0 Response Failed: 254
Send TPS: 33189 Max RT: 884 Average RT:   3.847 Send Failed: 0 Response Failed: 254
Send TPS: 32674 Max RT: 884 Average RT:   3.906 Send Failed: 0 Response Failed: 254
Send TPS: 20664 Max RT: 884 Average RT:   3.977 Send Failed: 0 Response Failed: 381
Send TPS: 32697 Max RT: 884 Average RT:   3.905 Send Failed: 0 Response Failed: 381
Send TPS: 22260 Max RT: 884 Average RT:   3.704 Send Failed: 0 Response Failed: 508
Send TPS: 23163 Max RT: 884 Average RT:   3.822 Send Failed: 0 Response Failed: 635
Send TPS: 22214 Max RT: 884 Average RT:   3.717 Send Failed: 0 Response Failed: 762
Send TPS: 33026 Max RT: 884 Average RT:   3.878 Send Failed: 0 Response Failed: 762
Send TPS: 32407 Max RT: 884 Average RT:   3.939 Send Failed: 0 Response Failed: 762
Send TPS: 22484 Max RT: 884 Average RT:   3.670 Send Failed: 0 Response Failed: 889
Send TPS: 24857 Max RT: 884 Average RT:   3.729 Send Failed: 0 Response Failed: 1016
Send TPS: 22179 Max RT: 884 Average RT:   3.718 Send Failed: 0 Response Failed: 1143
Send TPS: 22307 Max RT: 884 Average RT:   3.700 Send Failed: 0 Response Failed: 1270
Send TPS: 12191 Max RT: 884 Average RT:   3.232 Send Failed: 0 Response Failed: 1524
Send TPS: 32359 Max RT: 884 Average RT:   3.781 Send Failed: 0 Response Failed: 1539
Send TPS: 23205 Max RT: 884 Average RT:   3.720 Send Failed: 0 Response Failed: 1666
Send TPS: 32798 Max RT: 884 Average RT:   3.892 Send Failed: 0 Response Failed: 1666
Send TPS: 21936 Max RT: 884 Average RT:   3.762 Send Failed: 0 Response Failed: 1793
Send TPS: 33008 Max RT: 884 Average RT:   3.868 Send Failed: 0 Response Failed: 1793
Send TPS: 22432 Max RT: 884 Average RT:   3.674 Send Failed: 0 Response Failed: 1920
Send TPS: 25763 Max RT: 884 Average RT:   3.835 Send Failed: 0 Response Failed: 2047
```



- TPS ：波动比较大，很难统计。
- Average RT ：3.822ms 左右。
- Max RT ：884ms 左右。
- 存在极少量发送失败的消息。

# 7. 第二轮

通过第一轮的测试，考虑到测试结果的稳定性，我们选用 Producer 并发量为 20 的大小，进行测试。

## 7.1 异步复制 + 异步刷盘

> 使用配置文件 2m-2s-async 目录。



```
sh producer.sh --w=16 --s=1024
```



执行结果如下：



```
Send TPS: 22511 Max RT: 705 Average RT:   0.707 Send Failed: 0 Response Failed: 30
Send TPS: 22615 Max RT: 705 Average RT:   0.703 Send Failed: 0 Response Failed: 30
Send TPS: 22078 Max RT: 705 Average RT:   0.721 Send Failed: 0 Response Failed: 30
Send TPS: 21962 Max RT: 705 Average RT:   0.724 Send Failed: 0 Response Failed: 30
Send TPS: 22067 Max RT: 705 Average RT:   0.721 Send Failed: 0 Response Failed: 30
Send TPS: 21932 Max RT: 705 Average RT:   0.726 Send Failed: 0 Response Failed: 30
Send TPS: 22459 Max RT: 705 Average RT:   0.709 Send Failed: 0 Response Failed: 30
Send TPS: 15611 Max RT: 705 Average RT:   0.678 Send Failed: 0 Response Failed: 45
Send TPS: 22235 Max RT: 705 Average RT:   0.716 Send Failed: 0 Response Failed: 45
Send TPS: 22391 Max RT: 705 Average RT:   0.711 Send Failed: 0 Response Failed: 45
Send TPS: 21600 Max RT: 705 Average RT:   0.737 Send Failed: 0 Response Failed: 45
Send TPS: 22233 Max RT: 705 Average RT:   0.715 Send Failed: 0 Response Failed: 45
```



- TPS ：22000 左右。
- Average RT ：0.715ms 左右。
- Max RT ：705ms 左右。
- 存在极少量发送失败的消息。

## 7.2 同步复制 + 异步刷盘

> 使用配置文件 2m-2s-sync 目录。
>
> - 主 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a.properties -n localhost:9876 &` 。
> - 从 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a-s.properties -n 172.19.83.161:9876 &` 。
>
> 另外，参考 [《官方文档 —— 运维管理》](https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md) 的 「3.5 性能调优问题」中，描述 `sendMessageThreadPoolNums` 参数，对同步复制和同步刷盘，是有性能影响的，所以我们修改成 `sendMessageThreadPoolNums = 16 + Runtime.getRuntime().availableProcessors() * 4 = 16 + 4 * 4 = 32` 。这个公式，可以在 `BrokerConfig.sendMessageThreadPoolNums` 类中看到。



```
sh producer.sh --w=16 --s=1024
```



执行结果如下：



```
Send TPS: 7678 Max RT: 312 Average RT:   2.078 Send Failed: 0 Response Failed: 0
Send TPS: 6468 Max RT: 312 Average RT:   2.470 Send Failed: 0 Response Failed: 0
Send TPS: 3552 Max RT: 1942 Average RT:   4.229 Send Failed: 0 Response Failed: 0
Send TPS: 6014 Max RT: 2187 Average RT:   3.105 Send Failed: 0 Response Failed: 0
Send TPS: 7261 Max RT: 2187 Average RT:   2.200 Send Failed: 0 Response Failed: 0
Send TPS: 6873 Max RT: 2187 Average RT:   2.320 Send Failed: 0 Response Failed: 0
Send TPS: 7070 Max RT: 2187 Average RT:   2.267 Send Failed: 0 Response Failed: 0
Send TPS: 7118 Max RT: 2187 Average RT:   2.245 Send Failed: 0 Response Failed: 0
Send TPS: 5528 Max RT: 2187 Average RT:   2.889 Send Failed: 0 Response Failed: 0
Send TPS: 6819 Max RT: 2187 Average RT:   2.341 Send Failed: 0 Response Failed: 0
Send TPS: 6734 Max RT: 2187 Average RT:   2.373 Send Failed: 0 Response Failed: 0
Send TPS: 7058 Max RT: 2187 Average RT:   2.260 Send Failed: 0 Response Failed: 0
Send TPS: 6237 Max RT: 2187 Average RT:   2.560 Send Failed: 0 Response Failed: 0
Send TPS: 6708 Max RT: 2187 Average RT:   2.380 Send Failed: 0 Response Failed: 0
Send TPS: 6665 Max RT: 2187 Average RT:   2.388 Send Failed: 0 Response Failed: 0
Send TPS: 7202 Max RT: 2187 Average RT:   2.237 Send Failed: 0 Response Failed: 0
Send TPS: 7486 Max RT: 2187 Average RT:   2.129 Send Failed: 0 Response Failed: 0
```



- TPS ：7000 左右。
- Average RT ：2.129ms 左右。
- Max RT ：2187ms 左右。

相比“异步复制 + 异步刷盘”来说，性能有比较大的下滑。不过在看 [《官方文档 —— 运维管理》](https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md) 的「1.4 多Master多Slave模式-同步双写」中，描述说，“性能比异步复制模式略低（大约低10%左右）”。这点，让我有点懵逼。感兴趣的胖友，可以也测试下，分享下结果。

## 7.3 异步复制 + 同步刷盘

> 使用配置文件 2m-2s-async-sync 目录。
>
> - 主 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-async-sync/broker-a.properties -n localhost:9876 &` 。
> - 从 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-async-sync/broker-a-s.properties -n 172.19.83.161:9876 &` 。



```
sh producer.sh --w=16 --s=1024
```



执行结果如下：



```
Send TPS: 2645 Max RT: 267 Average RT:   6.032 Send Failed: 0 Response Failed: 0
Send TPS: 2513 Max RT: 267 Average RT:   6.355 Send Failed: 0 Response Failed: 0
Send TPS: 2527 Max RT: 267 Average RT:   6.318 Send Failed: 0 Response Failed: 0
Send TPS: 2607 Max RT: 267 Average RT:   6.113 Send Failed: 0 Response Failed: 0
Send TPS: 2790 Max RT: 267 Average RT:   5.731 Send Failed: 0 Response Failed: 0
Send TPS: 2558 Max RT: 267 Average RT:   6.257 Send Failed: 0 Response Failed: 0
Send TPS: 2681 Max RT: 267 Average RT:   5.959 Send Failed: 0 Response Failed: 0
Send TPS: 2829 Max RT: 267 Average RT:   5.648 Send Failed: 0 Response Failed: 0
```



- TPS ：2600 左右。
- Average RT ：5.648ms 左右。
- Max RT ：267ms 左右。

## 7.4 同步复制 + 同步刷盘

> 使用配置文件 2m-2s-sync-sync 目录。
>
> - 主 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-sync-sync/broker-a.properties -n localhost:9876 &` 。
> - 从 Broker ：`nohup sh bin/mqbroker -c conf/2m-2s-sync-sync/broker-a-s.properties -n 172.19.83.161:9876 &` 。



```
sh producer.sh --w=16 --s=1024
```



执行结果如下：



```
Send TPS: 3567 Max RT: 308 Average RT:   4.481 Send Failed: 0 Response Failed: 0
Send TPS: 3391 Max RT: 308 Average RT:   4.705 Send Failed: 0 Response Failed: 0
Send TPS: 3392 Max RT: 308 Average RT:   4.709 Send Failed: 0 Response Failed: 0
Send TPS: 3147 Max RT: 308 Average RT:   5.072 Send Failed: 0 Response Failed: 0
Send TPS: 3276 Max RT: 308 Average RT:   4.886 Send Failed: 0 Response Failed: 0
```



- TPS ：3200 左右。
- Average RT ：5.072ms 左右。
- Max RT ：308ms 左右。

## 7.5 小结

从性能上来看，**异步复制 + 异步刷盘**的方式，性能是最优秀的，甩其它方式好几条街。所以，生产上，最多使用的也是这种方式。另外，异步刷盘，可以配置 `transientStorePoolEnable = true` ，进一步提升性能。

当然，比较奇怪但是，**异步复制 + 同步刷盘** TPS 低于**同步复制 + 同步刷盘**，有点神奇？！具体的原因，还没去深究。

另外，RocketMQ 提供了基于 Raft 协议的 CommitLog 存储库 [dledger](https://www.jianshu.com/p/99e5df8e2657) 的方式，暂时并未去测试。

# 8. 第三轮

因为 RocketMQ benchmark 提供的 Consumer 测试，每次都启动一个新的消费集群，并且从 `CONSUME_FROM_LAST_OFFSET` 进行开始测试消费。所以，我们需要启动一个 Producer 不断发新消息，提供给消费者消费。

本轮，我们将启动**异步复制 + 异步刷盘** RocketMQ 集群，并且使用 Producer 按照并**发量为 16**进行发送消息。

> 艿艿：此处，我们假装胖友已经启动好 Broker 和 Producer 。



```
sh consumer.sh
```



执行结果如下：



```
Consume TPS: 13955 Average(B2C) RT:   3.693 Average(S2C) RT:   3.059 MAX(B2C) RT: 8602 MAX(S2C) RT: 8602
Consume TPS: 14920 Average(B2C) RT:   2.227 Average(S2C) RT:   1.643 MAX(B2C) RT: 8602 MAX(S2C) RT: 8602
Consume TPS: 15070 Average(B2C) RT:   2.223 Average(S2C) RT:   1.642 MAX(B2C) RT: 8602 MAX(S2C) RT: 8602
Consume TPS: 15142 Average(B2C) RT:   2.371 Average(S2C) RT:   1.806 MAX(B2C) RT: 8602 MAX(S2C) RT: 8602
Consume TPS: 10703 Average(B2C) RT:   2.326 Average(S2C) RT:   1.792 MAX(B2C) RT: 8602 MAX(S2C) RT: 8602
```



- 消费的 TPS 是 15000 左右。此时 Producer TPS 如下：

  ```
  Send TPS: 14901 Max RT: 2004 Average RT:   1.068 Send Failed: 0 Response Failed: 15
  Send TPS: 14982 Max RT: 2004 Average RT:   1.062 Send Failed: 0 Response Failed: 15
  Send TPS: 15123 Max RT: 2004 Average RT:   1.052 Send Failed: 0 Response Failed: 15
  Send TPS: 11460 Max RT: 2004 Average RT:   0.989 Send Failed: 0 Response Failed: 30
  Send TPS: 15466 Max RT: 2004 Average RT:   1.029 Send Failed: 0 Response Failed: 30
  Send TPS: 15110 Max RT: 2004 Average RT:   1.053 Send Failed: 0 Response Failed: 30
  Send TPS: 15132 Max RT: 2004 Average RT:   1.052 Send Failed: 0 Response Failed: 30
  Send TPS: 15265 Max RT: 2004 Average RT:   1.042 Send Failed: 0 Response Failed: 30
  Send TPS: 11945 Max RT: 2004 Average RT:   1.033 Send Failed: 0 Response Failed: 45
  Send TPS: 15027 Max RT: 2004 Average RT:   1.052 Send Failed: 0 Response Failed: 45
  Send TPS: 15233 Max RT: 2004 Average RT:   1.045 Send Failed: 0 Response Failed: 45
  Send TPS: 15237 Max RT: 2004 Average RT:   1.044 Send Failed: 0 Response Failed: 45
  Send TPS: 15182 Max RT: 2004 Average RT:   1.047 Send Failed: 0 Response Failed: 45
  ```

  

  - 发送的 Producer 的 TPS 也是 15000 左右。

- 因为测试的 Consumer ，是直接启动在 Broker 主节点，而测试的 Producer 也是直接启动在 Broker 主节点。所以测试出来的 TPS 并没有跑到 22000+ 。当然，从这个数据，我们也可以看出，Consumer 可以追上 Producer 的发送速度。

😈 Consumer 测试的比较偷懒，感兴趣的胖友，可以把 Consumer 启动在 Broker 从节点，进行基准测试。