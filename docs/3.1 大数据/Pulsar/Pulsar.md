下图是几款消息中间件的历史：

![图片](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUOCSEY4HAuOdgGwYmRuO5icX3NQzYicQ0r3BRmCe4WwAenL0xJNXGp5nw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2012年pulsar在Yahoo内部开发，2016年开源并捐献给Apache，2018成为Apache顶级项目。

## 1架构 

pulsar的架构图如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUA1b0sC38kGraH6mPqicd7gs0ic8DHexKldAtaZIicrW80DA2SR8dHSOrw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结一下，pulsar有下面的几个特性。

### 1.1 计算存储分离

pulsar采用计算和存储相分离的架构，Broker集群负责把producer发出的消息发送给consumer，同时承担负载均衡的作用。

Pulsar用 Apache BookKeeper作为持久化存储，Broker持有BookKeeper client，把未确认的消息发送到BookKeeper进行保存。

BookKeeper是一个分布式的WAL(Write Ahead Log)系统，pulsar使用BookKeeper有下面几个便利：

- 可以为Topic创建多个ledgers

> Ledger是一个只追加的数据结构，并且只有一个writer，这个writer负责多个BookKeeper存储节点(就是Bookies)的写入。Ledger的条目会被复制到多个bookies。

- Broker可以创建、关闭和删除Ledger，也可以追加内容到Ledger。
- Ledger被关闭后，只能以只读状态打开，除非要明确地写数据或者是因为writer挂掉导致的关闭。
- Ledger只能有writer这一个进程写入，这样写入不会有冲突，所以写入效率很高。如果writer挂了，Ledger会启动恢复进程来确定Ledger最终状态和最后提交的日志，保证之后所有Ledger进程读取到相同的内容。
- 除了保存消息数据外，还会保存cursors，也就是消费端订阅消费的位置。这样所有cursors消费完一个Ledger的消息后这个Ledger就可以被删除，这样可以实现ledgers的定期翻滚从头写。

### 1.2 节点对等

从架构图可以看出，broker节点不保存数据，所有broker节点都是对等的。如果一个broker宕机了，不会丢失任何数据，只需要把它服务的topic迁移到一个新的broker上就行。

Broker的topic拥有多个逻辑分区，同时每个分区又有多个segment，writer写数据时，首先会选择Bookies，比如图中的segment1，选择了Bookie1、Bookie2、Bookie4，然后并发地写下去。这样这三个节点并没有主从关系，协调完全依赖于writer，因此它们也是对等的。

### 1.3 扩展和扩容

在遇到双十一等大流量的场景时，必须增加consumer，这时因为Broker不存储任何数据，可以方便的增加broker。broker集群会有一个或多个broker做消息负载均衡，当新的broker加入后，流量会自动从压力大的broker上迁移过来。

对于BookKeeper，如果对存储要求变高，比如之前存储2个副本，现在需要存储4个副本，这时可以单独扩展bookies而不用考虑broker。因为节点对等，之前节点的segment又堆放整齐，加入新节点并不用搬移数据。writer会感知新的节点并优先选择使用。

### 1.4 容错机制

对于broker，因为不保存任何数据，如果节点宕机了，就相当于客户端断开，重新连接其他的broker就可以了。

对于BookKeeper，因为保存了多份副本，并且这些副本都是对等的，没有主从关系，所以当一个节点宕机后，不用立即恢复，后台有一个线程会检查宕机节点的数据备份进行恢复。

## 2 BookKeeper简介 

从上一节的讲解看出，Apache Bookkeeper是一个易扩展、高可用、运维简单的分布式存储系统。这节再看一下Bookkeeper的其他三个特性。

### 2.1 客户端数量

我们知道，在Kafka中，客户端只能从leader节点读取数据。但在BookKeeper中，**客户端可以从任何一个bookie副本读取数据**，这有三个好处：

- 增加了读高可用
- 把客户端流量平均分配到了不同的bookie
- 可以通过增加客户端数量来提高读取效率

客户端和服务器通信采用Netty实现异步I/O。网络I/O使用单个TCP连接进行多路复用，这就以很少的资源消耗实现了非常高的吞吐量。

### 2.3 I/O隔离

为什么要做I/O隔离？在大多数消息系统中，如果consumer处理慢，可能会导致消息积压。这迫使存储系统从持久存储介质中读取数据。当存储系统I/O组件共享写入、追尾读、追赶读的单一路径时，就会出现I/O抖动及页面缓存的换入换出。

写入和追尾读对可预测的低延迟有较高要求，而追赶读则对吞吐量的要求比较高，分离这三个路径很重要。

在BookKeeper中，bookie使用三条独立的I/O路径，分别用于写入、追尾读、追赶读。如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yU1JlomMXBcvHuKicnu8ADHFmsibpUBdg2JsraFz8picOR9ibn0rqKibzZuYw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**参考**[1]

## 3 多租户 

Pulsar可以使用多租户来管理大集群。Pulsar的租户可以跨集群分布，每个租户都可以有单独的认证和授权机制。租户也是存储配额、消息TTL和隔离策略的管理单元。

Pulsar的多租户性质主要体现在topic的URL中，其结构如下：

```
persistent://tenant/namespace/topic
```

可以看到，租户是topic的最基本单位。

假如一个公司有三个部门，tenant1、tenant2、tenant3，可以分配三个租户，这三个租户互不干扰，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUNOwrNkHLwCPBEbNsVrduuePicPF97SJlO8v5umSZZYeJABEqwWwswKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果消息平台不支持租户，那部门之间想要隔离，就要给每个部门部署一套集群，运维成本非常高。

## 4 消息模型 

### 4.1 消息结构

首先看一下Pulsar的消息结构，如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUiash91U6Z7kpk2oxJYeMbs37Udqia0QBJJwgdmdqr7knib989Z6iaRFMibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

消息流由多个独立的segment组成，(这里的segment就是上面讲的ledger),segment又包含独立的entry，entry又由独立的message组成。这里的message就是consumer发来的消息。

可以看到，一个message的id组成包括Ledger-id，entry-id，batch-index，partition-index。

> 需要注意两点：
>
> - segment和entry都是BookKeeper里面的概念。
> - pulsar作为消息平台时，一个message就是一个entry。当pulsar作为流平台时，为了提高吞吐量，会开启batch，这样多个message组成一个entry。

### 4.2 创建过程

消息的创建过程如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUDIr9bGvoGp7iaiciblZqe346rBvHmTrCtXvS2eItjbOCaUqCOavgsqtWQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

消息创建后主要经历下面几步：

1. 选择一个partition
2. 发送到管理这个partition的broker
3. broker将消息并发的发送给N个bookie，这个N是可以配置的。broker持有BookKeeper的客户端，也就是writer，writer收到写请求后，会并发的写入N个bookie。上图中N=3。
4. bookie写完消息后会给broker一个回复，broker收到指定数量的确认消息后就会认为写BookKeeper成功。这个数量是这个配置的，比如M，M越大，写BookKeeper延迟越大，数据一致性越高。因此这个配置要对一致性和延迟到进行。

## 5 消费模型 

### 5.1 概要

Pulsar的消费模型如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUbIHssR1RrJwicVicxbwz6nbXqPPzcJY2aMZeAJYPpu8ajbKMzgiaeF7zw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

producer将消息发送给topic，topic下有多个partition，partition下面又有多个broker。

broker负责接收消息并把消息分配给给consumer，并把消息写到BookKeeper。

> broker还具有限流功能，可以根据限流阈值对producer的消息进行限流。

consumer并不能直接从broker中获取消息，consumer和broker之间有一个Subscription，Consumer通过Subscription获取消息。

### 5.2 subscription

subscription有四种类型：

- 独占模式(Exclusive):同一个topic只能有一个消费者，如果多个消费者，就会出错。
- 灾备模式(Failover)：同一个topic可以有多个消费者，但是只能有一个消费者消费，其他消费者作为故障转移备用，如果当前消费者出了故障，就从备用消费者中选择一个进行消费。如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yU4kRukLL3y9z7Yoq6EJFfoibHbPPaia51cOIkQiajYiaEus2hibqPJcvWQ6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

- 共享模式(Shared):同一个topic可以由多个消费者订阅和消费。消息通过round robin轮询机制分发给不同的消费者，并且每个消息仅会被分发给一个消费者。当消费者断开，发送给它的没有被消费的消息还会被重新分发给其它存活的消费者。如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yURr1OnicAdLNqv3gfJTMbObhWd8V8FYGAyL37DNo0eoB0Vl1hYLS33iaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

- Key_Shared：消息和消费者都会绑定一个key，消息只会发送给绑定同一个key的消费者。如果有新消费者建立连接或者有消费者断开连接，就需要更新一些消息的key。如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUZykias6YXH3RpibU4a5jHRrWatZHS7swSAS1PKTR5BibbW7B5icNQKXGeg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

> 跟Shared模式相比，Key_Shared的好处是既可以让消费者并发地消费消息，又能保证同一Key下的消息顺序。

### 5.3 Cursor

当多个consumer订阅同一个topic时，Subscription为每一个consumer分配一个Cursor，这样多个Consumer之间就不会相互影响了。如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUw3CQpafcXfZhNZ8uwfPWtpibR2B87IbYx1z4jrfbsyxmibM2qvOYOeIA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

Subscription会维护一个消息的ACK状态，consumer处理完消息后，会给broker返回ACK，表示消息已经处理完成。如果broker一直没有收到ACK，就会把消息发送到其他consumer。

如果客户端想要重新消费Cursor以前的消息，Cursor是支持reset的，reset之后，Cursor就回退回去了，这时consumer可以从新的Cursor位置进行消费。

Cursor的位置是会实时写入BookKeeper的，这必定会有一定的性能损耗。因此，Pulsar提供了一种非持久化的Subscription(Non-durable Exclusive)。Pulsar的Reader接口内嵌了Non—durable Exclusive Cursor，它读取消息不会返回ACK。

## 6 broker代理 

通过前面的讲解可以看到，consumer和producer只需要跟broker进行交互，而不用跟底层的BookKeeper交互，事实上，broker还有一层代理，consumer和producer直接跟代理进行交互。如下图：

![图片](https://mmbiz.qlogo.cn/mmbiz_png/a1gicTYmvicdibLSib4VZ0Zvkrc9ettsS3yUWUaIoOXfGcQfYOSTib65KG1AMUxAKPibib0iaydYAaB6VvLTWBRMU0kKeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

## 7 Zookeeper 

Pulsar提供了System topic用来保存策略之类的元数据，尽量减少对Zookeeper的依赖。

Zookeeper也保存一些策略相关的元数据，还保存了broker和BookKeeper集群相关的配置元数据，比如服务发现相关的元数据。

## 8 总结 

Pulsar是一款非常优秀的中间件，实现了计算和存储相分离，支持多租户，扩展和扩容、容错都是非常容易的。