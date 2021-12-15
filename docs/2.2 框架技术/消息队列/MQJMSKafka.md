# 1 消息队列介绍

首先举个收快递的栗子，传统的收快递，快递小哥把我们的快递送到我们的手里。他需要什么条件嗯？

- 快递小哥有时间送，
- 我们有时间取，
- 快递小哥和我们约定一个时间地点。

但是嗯。快递小哥有那么多的快递需要送，可能送我快递的时候，我不在家，可能我在家的时候，快递小哥送其他的地方的快递。所以嗯，这个时候，要么就是坐在家里等快递，要么就只能从新约个时间点在送。那怎么办去避免这个情况嗯？

于是嗯快递柜出现了。快递小哥不用关心我什么时候在家，因为快递小哥有时间了，就把快递放快递柜，而我有时间了，我就去快递柜取我的快递。

那么快递柜所起到的作用就是我们今天要收的消息队列。我们可以把消息队列比作是一个存放快递的的快递柜，当我们需要获取我们快递的时候就可以从快递柜里面拿到属于我们的快递。

## 1.1 什么是消息队列

我们可以把消息队列比作是一个存放消息的容器，当我们需要使用消息的时候可以取出消息供自己使用。我们看看维基百科上的描述：在计算机科学中，消息队列（Message queue）是一种进程间通信或同一进程的不同线程间的通信方式，软件的贮列用来处理一系列的输入，通常是来自用户。

是不是很难理解，我们换个说法来理解

> 我们可以把消息队列比作是一个存放消息的容器，当我们需要使用消息的时候可以取出消息供自己使用。

## 1.2 消息队列（Message queue）有什么用？

消息队列是分布式系统中重要的组件，使用消息队列主要是为了通过异步处理提高系统性能和削峰、降低系统耦合性。

通过异步处理提高系统性能（削峰、减少响应所需时间）

举个例子：我们在某个网站进行注册账号，我们需要做如下四件事：

- 填写我们的注册信息；
- 提交我们的注册信息；
- 我们的邮箱收到注册信息；
- 我们的短信收到注册信息。

如果采用同步串行，所需要的时间是：a+b+c+d

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavm1mzOpHrib73zPoQbEvg1TAFpuy0j2RkWa9kEmWCdKvlHVrgtcelvRg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

如果采用同步并行，所需要的时间是：a+b+max(c,d)

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavXKAIIyBkJoWNFCV5mjpqcFNeV2T0K473T8XKPtAs14EMe3ibiao913rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

如果采用消息队列，所需要的时间是：a+b+消息队列

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavxYRFhSX4sbefjXyb4aDWPbVllQicAPAJiaDQyP9wPgg2275IZibkbdeRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

### 降低系统耦合性

举个例子，A公司做了某个系统，B公司觉得A公司的某个功能很好，于是B公司和A公司的系统进行了集成。这时C公司也觉得A公司的这个功能很好，于是，C公司也和A公司的系统进行了集成。以后还有D公司…。

介于这种情况，A公司的系统和其他公司的耦合度都很高，每集成一个公司的系统，A公司都需要修改自己的系统。如果采用消息队列，则变成了如下：

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviav6HCQ6A7w9x0a3KxVl2YF2MX7sBH8iaUoicU0h4HkuPeb6JXy759DMekA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

不管以后还有多少公司的应用程序想要用A公司的程序，都不需要和A公司进行集成，谁需要这个功能，谁就去消息队列里面获取。

## 1.3 消息队列的两种模式

### 点对点模式

应用程序由：消息队列，发送方，接收方组成。

每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，直到他们被消费或超时。

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavwHefRpOvw6FC3an88Iiarf8GCVS4VSR0Fz6iaTKH0ia9t88OyVYqBjWjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

### 发布订阅模式

用用程序有由：角色主题（Topic）、发布者(Publisher)、订阅者(Subscriber)构成。

发布者发布一个消息，该消息通过topic传递给所有的客户端。该模式下，发布者与订阅者都是匿名的，即发布者与订阅者都不知道对方是谁。并且可以动态的发布与订阅Topic。Topic主要用于保存和传递消息，且会一直保存消息直到消息被传递给客户端。

[![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

介绍完了消息队列，接着我们介绍JMS

#  2 JMS介绍

JMS即Java消息服务（Java Message  Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，类似于JDBC。用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。它提供创建、发送、接收、读取消息的服务。由Sun公司和它的合作伙伴设计的应用程序接口和相应语法，使得Java程序能够和其他消息组件进行通信。

JMS是一个消息服务的标准或者说是规范，允许应用程序组件基于JavaEE平台创建、发送、接收和读取消息。它使分布式通信耦合度更低，消息服务更加可靠以及异步性。

介绍到这里，应该明白了消息队列和JMS的区别了吧？

- 消息队列：计算机科学中，A和B进行通信的一种方式。
- JMS：java平台之间分布式通信的一种标准或者规范。

换句话说，JMS就是java对于消息队列的一种实现方式。

## 2.1 JSM消息模型

点对点，发布订阅，消息队列中已经说的很清楚了，这里就不重复说了。

## 2.2 JMS消费

- 同步（Synchronous）

订阅者/接收方通过调用 receive()来接收消息。在receive（）方法中，线程会阻塞直到消息到达或者到指定时间后消息仍未到达。

- 异步（Asynchronous）

消息订阅者需注册一个消息监听者，类似于事件监听器，只要消息到达，JMS服务提供者会通过调用监听器的onMessage()递送消息。

## 2.3 JMS编程模型

JMS编程模型非常类似于JDBC。回忆一下，我们之前讲到的MyBatis。

- SqlSessionFactoryBuilder去构造SqlSessionFactory会话工厂；
- SqlSessionFactory会话工厂给我们打开SqlSession会话；
- SqlSession帮我们去连接数据库，接着我们就可以执行增删查改。

```
sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession openSession = sqlSessionFactory.openSession(true);
ProjectMapper mapper = openSession.getMapper(ProjectMapper.class);
mapper.queryAllTaskByInit("init");
```

JMS模型如下

- Connection Factory给我创建Connection连接;
- Connection连接给我们创建了Session会话；
- Session会话给我们创建消费者和生产者；
- 生产者生成消息；
- 消费者消费消息；

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviav5JibhRRVbWz11LBaoWzNrZRtrJlFGtVJULk6ic8h3APSiaAEYdjOQLYLA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

# 3 MQ介绍

上文中，我们说到了，JMS他并不是一种真正意义的技术，而是一种接口，一种规范。就想JDBC一样，无论是mybatis、hibernate，还是springJPA，不管你是那种技术实现，反正你得遵守JDBC的规范。

在Java中，目前基于JMS实现的消息队列常见技术有ActiveMQ、RabbitMQ、RocketMQ。值得注意的是，RocketMQ并没有完全遵守JMS规范，并且Kafka不是JMS的实现。

## 3.1 AMQP协议

这里我们以RabbitMQ为例介绍MQ，首先介绍下AMQP

AMQP协议（Advanced Message Queuing Protocol，高级消息队列协议）是一个进程间传递异步消息的网络协议。

### AMQP 模型

[![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavkuviagQQ5LKnmYhq0ctb7eFVRRiaWxN7QVY0UUEekgGdMZibYTy5rPibfg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### AMQP 工作过程

发布者（Publisher）发布消息（Message），经由交换机（Exchange）。

交换机根据路由规则将收到的消息分发给与该交换机绑定的队列、（Queue）。

最后 AMQP 代理会将消息投递给订阅了此队列的消费者，或者消费者按照需求自行获取。

RabbitMQ是MQ产品的典型代表，是一款基于AMQP协议可复用的企业消息系统

## 3.2 RabbitMQ模型

RabbitMQ由两部分组成，分别是服务端和应用端；

- 服务端包括：队列和交换机。
- 客户端包括：生产者和消费者。

在rabbitmq server上可以创建多个虚拟的message broker。每一个broker本质上是一个mini-rabbitmq server，分别管理各自的exchange，和bindings。

broker相当于物理的server，可以为不同app提供边界隔离，使得应用安全的运行在不同的broker实例上，相互之间不会干扰。producer和consumer连接rabbit server需要指定一个broker。

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavKI8HUO78O2CkgRv3Kiaaxyibiaoe8UibmqGwScAcANptoxjskYBxj2e72Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavJHe5m0PqIzc0AMOu6Ylqsy4Kh7YeIItjgX7GC174Er8ExCZhNGSgfA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

Exchange有4种类型：direct(默认)，fanout, topic, 和headers

- Direct：直接交换器，工作方式类似于单播，Exchange会将消息发送完全匹配ROUTING_KEY的Queue。
- Fanout：广播是式交换器，不管消息的ROUTING_KEY设置为什么，Exchange都会将消息转发给所有绑定的Queue(所谓绑定就是将一个特定的 Exchange 和一个特定的 Queue 绑定起来。Exchange 和Queue的绑定可以是多对多的关系)。
- Topic:主题交换器，工作方式类似于组播，Exchange会将消息转发和ROUTING_KEY匹配模式相同的所有队列，比如，ROUTING_KEY为user.stock的Message会转发给绑定匹配模式为 * .stock,user.stock， * . * 和#.user.stock.#的队列。（ *  表是匹配一个任意词组，#表示匹配0个或多个词组）。

至于如何在代码中使用RabbitMQ，这里我们先不撸代码，本文目前只介绍理论梳理知识点。

# 4 Kafka

上完中我们提到过，kafka不是JMS的实现，因此在MQ章节中，我们没有提及到它。现在我们开始学习kafka吧。

先来放张kafka的原理图，相信你看到这个图片时，内心是奔溃的。我草，啥玩意。接下来我们就一点一点的消化吧。

## 4.1 kafka原理图

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavyT7ia9fiaamRCt0zm0YziakjGCiaVGSrDIib5sZmWd8rXmLUoMibLk6D0ZWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

先介绍上图中的术语。

- Producer ：消息生产者，就是向kafka broker发消息的客户端。
- Consumer ：消息消费者，向kafka broker取消息的客户端。
- Topic  ：kafka给消息提供的分类方式。broker用来存储不同topic的消息数据。一个Topic可以认为是一类消息，每个topic将被分成多个partition(区),每个partition在存储层面是append  log文件。任何发布到此partition的消息都会被直接追加到log文件的尾部，每条消息在文件中的位置称为offset（偏移量），offset为一个long型数字，它是唯一标记一条消息。它唯一的标记一条消息。kafka并没有提供其他额外的索引机制来存储offset，因为在kafka中几乎不允许对消息进行“随机读写”。
- broker：中间件的kafka cluster，存储消息，是由多个server组成的集群。
- Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序。
- Offset：kafka的存储文件都是按照offset.kafka来命名，例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka。

类似于JMS的特性，但不是JMS规范的实现。kafka对消息保存时根据Topic进行归类，发送消息者成为Producer，消息接受者成为Consumer,此外kafka集群有多个kafka实例组成，每个实例(server)成为broker。无论是kafka集群，还是producer和consumer都依赖于zookeeper来保证系统可用性集群保存信息。

kafka基于文件存储。通过分区,可以将日志内容分散到多个server上，来避免文件尺寸达到单机磁盘的上限，每个partiton都会被当前server(kafka实例)保存；可以将一个topic切分多任意多个partitions，来消息保存/消费的效率.此外越多的partitions意味着可以容纳更多的consumer，有效提升并发消费的能力。

kafka和JMS不同的是:即使消息被消费,消息仍然不会被立即删除。日志文件将会根据broker中的配置要求，保留一定的时间之后删除。

### Kafka高可用机制

- 多个broker组成，每个broker是一个节点；
- 你创建一个topic，这个topic可以划分为多个partition，每个partition可以存在于不同的broker上，每个partition就放一部分数据。
- 采用replica副本机制，每个partition的数据都会同步到其他机器上，形成多个replica副本。
- 所有replica会选举一个leader出来，那么生产和消费都跟这个leader打交道，然后其他replica就是follower。
- 读数据时，从leader读取，写数据时，leader把数据同步到所有follower上去。如果某个broker宕机了，这个broker在其他的broker还保留副本，假设这个broker上面存在leader,那么就重新选一个leader。

内容有点多，需要结合图片一点一点消化

## 4.2 生产者结构图

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfebPS6KVNmkClTwVvOMNviavibJWBMAGvWSFcOcY39yxrtkwCW96bTll0m6ERDbDyvzFEJQnTZ5MqSA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

至此，虽然看的云里雾里，不过相信你们还是能区分了吧？

整理一下：

- 消息队列：指计算机领域中，A和B通信的一种通信方式；
- JMS：Java中对于消息队列的接口规范；
- ActiveMQ/RabbitMQ：JMS接口规范具体实现的一种技术；
- RocketMQ：不完全是JMS接口规范具体实现的一种技术；
- Kafka：非JMS接口规范具体实现的一种技术；