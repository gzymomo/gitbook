# 消息队列RabbitMQ随笔

## 概念

- 由erlang语言开发，基于AMQP（Advanced Message Queue 高级消息队列协议）协议实现的消息队列。

### 使用场景

- 任务异步处理：将不需要同步处理的并且消耗时间长的操作由消息队列通知消息接收方进行异步处理，提高应用程序的响应时间
- 应用程序解耦合：MQ相当于一个中介，生产方通过MQ与消费方交互，将应用程序解耦
- 市场上还有哪些消息队列：ActiveMQ、ZeroMQ、Kafka、MetaMQ、RocketMQ、Redis

### 为什么使用RabbitMQ

- 使用简单、功能强大
- 基于AMQP协议
- 社区活跃，文档完善
- 高并发性能好，这主要得益于Erlang语音
- Spring Boot默认已经集成RabbitMQ

### JMS是什么

- java消息服务（Java Message Service，JMS）应用程序接口是一个Java平台中关于面向消息中间件（MOM）的API。
- JMS是java提供的一套消息服务API标准，其目的是为所有的java应用程序提供统一的消息通信的标准，类似java的jdbc，只要遵循jms标准的应用程序之间都可以进行消息通信。

## 快速入门

### 组成部分

- Broker: 消息队列服务进程，此进程包括两部分：Exchage和Queue
- Exchange：消息队列交换机，按一定规则将消息路由转发到某个队列，对消息进行过滤
- Queue：队列，存储消息的队列，消息到达队列并转发给指定消费者
- Producer：生产者，生成客户端，生产方客户端将消费发送MQ
- Consumer：消费者，即消费方客户端，接受MQ转发的消息

### 消息发布接受流程

- 生产者和Broker建立TCP连接
- 生产者和Broker建立通道
- 生产者通过通道将消息发送到Broker，有Exchange将消息进行转发
- Exchange将消息转发到指定的Queue（队列）

### 接受消息

- 消费者和Broker建立TCP连接
- 消费者和Broker建立通道
- 消费者监听指定Queue（队列）
- 当有消息到达Queue是Broker默认将消息推送给消费者
- 消费者接收消息

### 理解通道（channel）

建立TCP连接：connection 后，再建立通道（Channel），大连接内会有多个**会话通道**，这也可以保证通道之间互相隔离，这也是rabbitMQ 可以支持多个消费者、生产者通信的机制。

### AMQP与TCP的关系

- AMQP 连接通常是长连接。**AMQP 是一个使用 TCP 提供可靠投递的应用层协议**。AMQP 使用认证机制并且提供 TLS（SSL）保护。当一个应用不再需要连接到 AMQP 代理的时候，需要优雅的释放掉 AMQP 连接，而不是直接将 TCP 连接关闭。
- 通道 有些应用需要与 AMQP 代理建立多个连接。无论怎样，同时开启多个 TCP 连接都是不合适的，因为这样做会消耗掉过多的系统资源并且使得防火墙的配置更加困难。AMQP 0-9-1 提供了通道（channels）来处理多连接，可以把通道理解成共享一个 TCP 连接的多个轻量化连接。
- 在涉及多线程 / 进程的应用中，为每个线程 / 进程开启一个通道（channel）是很常见的，并且这些通道不能被线程 / 进程共享。
- 一个特定通道上的通讯与其他通道上的通讯是完全隔离的，因此每个 AMQP 方法都需要携带一个通道号，这样客户端就可以指定此方法是为哪个通道准备的。

### virtualHost

- 设置虚拟机，一个MQ服务可以设置多个虚拟机，每个虚拟机就相当于一个独立的MQ，可以用来模拟多个MQ 的使用

## Producer、Consumer

### declare queue（声明队列）

```
1. 参数说明
 * string queue ：队列名称
 * durable：是否持久化，如果持久化，MQ重启队列还在
 * exclusive：是否独占连接，队列只允许在该连接中访问。特色：如果连接关闭，该队列会自动删除。如果设置为true可用于创建临时队列
 * autoDelete：自动删除，队列不再使用时自动删除此队列，如果将此参数和exclusive参数设置为true就可以实现临时队列（队列不用了自动删除）
 * arguments 参数，可以设置一个队列的扩展参数，比如：可设置存活时间
复制代码
```

### publish message （发送消息 ）

1. 参数说明
   - exchange：交换机，如果不指定将使用MQ的默认交换机
   - routing key：路由key，交换机根据路由key来将消息转发到指定的队列，**如果使用默认的交换机，routing key要设置为队列名称**
   - props ：消息的属性
   - body: 消息内容

### consumer(监听队列)

channel.basicconsumer（...）

1. 参数说明

   - queue :队列名称

   - autoAck：自动回复，当消费者接收到消息后要告诉MQ消息已接收，如果将此参数设置为true，表示会自动回复MQ，如果设置为false要通过编程实现回复

   - callback：消费方法，当消费者接收到消息消息要执行的方法

     ​    DefaultConsumer（实现消费方法）：当接收到消息后此方法将被调用

     1. consumerTag：消费者标签，用来标志消费者
     2. envelope： 信封 ，通过envelope可以拿到**交换机**、**deliveryTag**：消息id，MQ在channel用来标志消息的id，**可用来确认消息已接收**（ack）
     3. properties：消息属性
     4. body: 消息内容

## 关闭连接

生产者 关闭连接前，先关闭通道（channel）

## 工作模式

### 几种工作模式

```
1. Work queues 工作队列
2. Publish/Subscribe 发布订阅
3. Routing 路由
4. Topics 通配符
5. Header Header转发器
6. RPC 远程过程调用
复制代码
```

### Work queue模式（工作队列）



![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe726dc691de5?imageView2/0/w/1280/h/960/ignore-error/1)



- work queue与入门程序相比，多了一个消费端，两个消费端共同消费同一个队列中的消息

  特点：1. 一个生产者将消息发送给一个队列，**多个消费者**同时监听同一个队列的消息

  ```
       2. 消息不能被重复消费
    	   3. rabbitMQ采用轮询的方式将消息**平均发送**给消费者
  复制代码
  ```

### Publish/Subscribe(发布订阅)



![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe7270414f78c?imageView2/0/w/1280/h/960/ignore-error/1)



场景：邮件短信异步通知

- 一个生产者将消息发给交换机
- 与交换机绑定的有多个队列，每个消费者监听自己的队列
- 生产者将消息发给交换机，由交换机将将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列将收到消息
- 如果消息发给没有绑定队列的交换机上，消息将丢失
- declare exchange(声明交换机) ：
  1. exchange：交换机的名称
  2. type:交换机的类型
     - fanout：对应的rabbitmq的工作模式是publish/subscribe
     - direct：对应的是Routing的工作模式
     - topic：对应的Topics的工作模式
     - headers: 对应的是headers工作模式
- queueBind （进行交换机和队列绑定）
  1. queue 队列名称
  2. exchange 交换机的名称
  3. routingKey 路由key，作用是交换机根据路由key值将消息转发到指定的队列当中，在发布订阅模式中协调为空字符串
- 

### Publish/Subscribe与Work queue的区别

- publish/subscribe可以定义一个交换机绑定多个队列，一个消息可以发送给多个队列
- work queue无需自定义交换机，一个消息一次只能发送给一个队列
- publish/subscribe比work queue的功能更强大，publish/subscribe也可以将多个消费者监听同一个队列实现work queue的功能

### Routing(路由模式)



![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe7272c88ca54?imageView2/0/w/1280/h/960/ignore-error/1)







- 一个交换机绑定多个队列，每个队列设置routingKey,并且一个队列可以设置多个routingKey
- 每个消费者监听自己的队列
- 生产者将消息发给交换机，发送消息是需要知道routingKey的值，交换机来判断该routingKey的值和那个队列的RoutingKey绑定，如果相等则将消息转发给该队列

### routing与publish/subscribe的区别

- Publish/Subscribe模式在绑定交换机是不需要指定routingKey，消息会发送到每个绑定交换机的队列

- Routing模式要求队列在绑定交换机时要指定routingKey（这就是队列的routingKey）,发送消息将消息发送到和routingKey的值相等的队列中，正如上图所示，每个队列可以指定多个routingKey,如果发送消息是指定的routingKey为“error”，由于C1和C2的routingKey都是error，所以消息发送给了C1和C2

  如果发送消息时指定routingKey为“info”，则只有C2可以接收到消息

  所以，Routing模式更加强大，它可以实现Publish/Subscribe的功能

### Topics（通配符工作模式）

使用案例：根据用户的通知设置去通知用户，设置接收Email的用户只接收Email，设置接收sms的用户只接收sms，设置两种通知类型都接收的则两种通知都有效。



![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe7275c09a4e4?imageView2/0/w/1280/h/960/ignore-error/1)







- 一个交换机可以绑定多个队列，每个队列可以设置一个或多个带通配符的routingKey
- 生产者将消息发给交换机，交换机根据routingKey的值来匹配队列，匹配时采用通配符的方式，匹配成功的将消息转发到指定的队列

### Topics与Routing的区别

- Topics和Routing的基本原理相同，即：生产者将消息发送给交换机，交换机根据routingKey将消息转发给与routingKey**匹配**的队列
- 不同之处是：routingKey的匹配方式，Routing模式是相等匹配，topics模式是通配符匹配
- 符号#：匹配一个或多个词，比如inform.#可以匹配inform.sms、inform.email、inform.email.sms
- 符号*：只能匹配一个词，比如inform.* 可以匹配到inform.sms、inform.email

### Header和RPC

- hender模式与routing不同的地方在于header模式取消routingKey，使用header中的key/value(键值对)匹配队列

  案例：根据用户的通知设置去通知用户，设置接收email的只接收email，设置接收sms的用户只接收sms，设置两种通知类型都接收的则两种通知都有效

  

  ![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe7278604274d?imageView2/0/w/1280/h/960/ignore-error/1)

  

  

  ![img](https://user-gold-cdn.xitu.io/2020/6/29/172fe727b551ef6c?imageView2/0/w/1280/h/960/ignore-error/1)

  

- RPC即客户端远程调用服务端的方法，使用MQ可以实现RPC的异步调用，基于Direct交换机实现，流程如下：

  1. 客户端即生产者就是消费者，想RPC请求队列发送RPC调用消息，同时监听RPC响应对列
  2. 服务端监听RPC请求队列信息，收到消息后执行服务端的方法，得到方法返回的结果
  3. 服务端将RPC方法的结果发送到RPC响应队列


作者：几个你_
链接：https://juejin.cn/post/6844904201768681486