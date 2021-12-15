## 开源 MQTT Broker 对比

#### 常见开源 MQTT Broker

- EMQ X - EMQ X 基于 Erlang/OTP 平台开发，是开源社区中最流行的 MQTT 消息服务器。除了 MQTT 协议之外，EMQ X 还支持 MQTT-SN、CoAP、LwM2M、STOMP 等协议。目前，EMQ X 在全球市场已有 5000+ 企业用户，20+ 世界五百强合作伙伴。
- Eclipse Mosquitto - Mosquitto 是开源时间较早的 MQTT Broker，它包含了一个C/C ++的客户端库，以及用于发布和订阅的 `mosquitto_pub`、`mosquitto_sub` 命令行客户端。Mosquitto 比较轻量，适合在从低功耗单板计算机到完整服务器的所有设备上使用。
- VerneMQ - VerneMQ 基于 Erlang/OTP 平台开发，是高性能的分布式 MQTT 消息代理。它可以在硬件上水平和垂直扩展，以支持大量并发客户端，同时保持较低的延迟和容错能力。
- HiveMQ CE - HiveMQ CE 是基于 Java 的开源 MQTT 消息服务器，它完全支持 MQTT 3.x 和 MQTT 5，是 HiveMQ 企业版消息连接平台的基础。



本文选取了几个热门开源的 MQTT Broker，其中部分项目提供商业支持，做简单选型对比。

| 对比项目   | EMQ                                                          | HiveMQ                                                       | VerneMQ                                                      | ActiveMQ                                                     | Mosquitto                                          |
| :--------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------------------------------------------- |
| License    | 开源+商业版                                                  | 开源+商业版                                                  | 开源+商业版                                                  | 开源                                                         | 开源                                               |
| 公司/社区  | EMQ                                                          | HiveMQ                                                       | VerenMQ                                                      | Apache 基金会                                                | Eclipse 基金会                                     |
| 开源协议   | Apache License 2.0                                           | Apache License 2.0                                           | Apache License 2.0                                           | Apache License 2.0                                           | EPL/EDL licensed                                   |
| 开发团队   | 杭州映云科技有限公司                                         | dc-square 股份有限公司，德国                                 | [Octavo Labs AG，瑞士](https://octavolabs.com/)              | Apache 项目维护者                                            | Eclipse 开源社区                                   |
| 开发语言   | Erlang                                                       | Java                                                         | Erlang                                                       | Java                                                         | C                                                  |
| 项目历史   | 2012年开始开源，2016年开始商业化                             | 2013 年成立，一直以闭源方式向客户提供软件，2019 年开源       | 提供基于开源的商业化定制服务                                 | 2004 由 LogicBlaze 创建；原本规划的 ActiveMQ 的下一代开源项目 Apollo 已经不活动（4年没有代码更新） |                                                    |
| 集群架构   | 支持                                                         | 仅企业版                                                     | 支持                                                         | 支持                                                         | 不支持（有伪集群实现）                             |
| 系统部署   | 物理机、虚拟机、K8S                                          | 物理机、虚拟机、K8S                                          | 物理机、虚拟机、K8S                                          | 物理机、虚拟机、容器                                         | 物理机、虚拟机、容器                               |
| 支持协议   | MQTT、CoAP、MQTT-SN、WebSocket、TCP、UDP、LwM2M              | MQTT                                                         | MQTT                                                         | JMS、Openwire、Stomp、AMQP、MQTT、WebSocket XMPP             | MQTT、WebSocket                                    |
| 系统性能   | 单机性能较高，单机支持百万级并发，集群支持千万级并发         | 集群支持千万级并发                                           | 集群支持百万级并发                                           | 支持集群                                                     | 单机10W                                            |
| MQTT       | v3.1，v3.1.1，v5.0                                           | v3.1，v3.1.1，v5.0                                           | v3.1，v3.1.1，v5.0                                           | v3.1                                                         | v3.1，v3.1.1，v5.0                                 |
| 边缘计算   | EMQ X Edge 支持树莓派，ARM 等架构，支持数据同步到云服务 Azure IoT Hub AWS | 不支持                                                       | 不支持                                                       | 不支持                                                       | 支持（自身比较轻量）                               |
| 安全与认证 | TLS/DTLS、X.509证书、JWT、OAuth2.0、应用协议（ID/用户名/密码）、数据库与接口形式的认证与 ACL 功能（LDAP、DB、HTTP） | TLS/DTLS、X.509证书、JWT、OAuth2.0、应用协议（ID/用户名/密码）、配置文件形式的认证与 ACL 功能 | TLS/DTLS、X.509证书、配置文件形式的认证与 ACL 功能、数据库形式的认证与 ACL 功能，但支持数据库较少 | LDAP (JAAS)、Apache Shiro                                    | 等待                                               |
| 运行持久化 | 支持将消息数据持久化至外部数据库如 Redis、MySQL、PostgreSQL、MongoDB、Cassa、Dynamo 等，**需企业版**，开源版宕机则丢失 | 开源企业均支持本地持久化，采用磁盘系统，支持备份，导出备份   | 支持持久化至 Google LevelDB                                  | AMQ、KahaDB、JDBC、LevelDB                                   | 等待                                               |
| 扩展方式   | Webhook、Trigger、Plugin 等，支持 Erlang 与 Lua、Java、Python 扩展开发，支持 Webhook 开发，侵入性不强 | Trigger、Plugin 等，使用 Java 技术栈开发，提供方便开发的 SDK | Trigger、Plugin 等，支持 Erlang 与 Lua 扩展开发              | Java 扩展                                                    | 等待                                               |
| 数据存储   | 仅**企业版**适配数据库：Redis、Mysql、PostgreSQL、MongoDB、Cassandra、OpenTSDB、TimescaleDB、InfluxDB 适配消息队列：Kakfa、RabbitMQ、Pulsar 桥接模式：支持桥接至标准 MQTT 协议消息服务 开源版支持 HTTP 将数据同步、存储 | 适配数据库：无，提供 Java SDK 开发进行适配 消息队列：Kafka 桥接模式：支持桥接至标准 MQTT 协议消息服务 | 适配数据库：无，提供 Erlang 和 Lua 扩展开发 适配消息队列：无 桥接模式：支持桥接至标准 MQTT 协议消息服务 | 适配数据库：JDBC、KahaDB、LevelDB 适配消息队列：无 桥接模式：支持通过 JMS 桥接 | 等待                                               |
| 管理监控   | 支持可视化的 Dashboard，实现集群与节点的统一集中管理 支持第三方监控工具 Prometheus ，提供可视化 Grafana 界面模板 | 支持可视化的 HiveMQ Control Center，实现集群与节点统一管理 支持第三方监控工具 Prometheus ，可提供可视化 Grafana 界面 支持 InfluxDB 监控 | 内置简单状态管理可视化界面 支持第三方监控工具 Prometheus ，可提供可视化 Grafana 界面 | 支持可视化的监控界面 支持第三方监控工具 Prometheus ，可提供可视化 Grafana 界面 | 通过 MQTT 订阅系统主题                             |
| 规则引擎   | 支持规则引擎，基于 SQL 的规则引擎给予 Broker 超越一般消息中间件的能力。除了在接受转发消息之外，规则引擎还可以解析消息的格式（**企业版**）。 规则引擎由消息的订阅，发布，确认的事件触发，根据消息的负载来执行相应的动作，降低应用开发的复杂度。 | 不支持                                                       | 不支持                                                       | 不支持                                                       | 不支持                                             |
| 开发集成   | 支持通过 REST API 进行常用的业务管理操作如： 调整设置、获取 Broker 状态信息、进行消息发布、代理订阅与取消订阅、断开指定客户端、查看客户端列表、规则引擎管理、插件管理，提供 Java SDK、Python SDK 直接编码处理业务逻辑 | 无，提供 Java SDK 在应用系统在编码的层面操作进程，非常灵活但耦合性高 | 提供少量 REST API，用于监控与状态管理、客户端管理等。 缺乏代理订阅、业务管理等功能和 API | 提供少量队列管理 REST API                                    | 等待                                               |
| 适用场景   | 优势在于高并发连接与高吞吐消息的服务能力，以及物联网协议栈支持的完整性；扩展能力较强，无需过多开发 | 有一定高并发连接与高吞吐消息的服务能力，物联网协议栈的完整性较弱仅支持 MQTT 协议；缺乏开箱即用的功能插件，功能必须编码使用 | 基础的并发连接与高吞吐消息的服务能力，物联网协议栈的完整性较弱仅支持 MQTT 协议；扩展能力较差，基础的业务组件支持度不够，商业成熟度不足客户量较少，缺乏开箱即用的功能插件 | 核心是消息队列系统，主要用于支持异构应用之间的消息通信，比如用于企业消息总线等；后面支持了部分物联网协议。ActiveMQ 比较适合系统既要支持传统的异构应用之间需要通信，也需要支持小型物联网接入支持的用户。 | 轻量简便的 MQTT Broker，工控、网关或小规模接入项目 |

## 性能测试对比

MQTT Broker 性能测试对比，包含快速使用的测试工具。

- MQTTBox 客户端工具提供了一个 MQTT 连接与性能测试功能，但是这个是基于 JS 来做的且比较简单，功能与性能都一般。

- [EMQ](https://www.emqx.io/) 提供了一个性能测试工具 [emqtt-benck](https://github.com/emqx/emqtt-bench)，采用 Erlang 编写，在 MacBook 2015款上能跑出 2K 连接，10K消息吞吐，把电脑压榨到死机，适合来做 MQTT Broker 性能测试。
  - 连接：指定连接数、连接速率，测试 MQTT Broker 的连接性能（速率、响应时间、错误数）
  - 订阅：指定连接数、主题数、订阅速率，QoS、测试 MQTT Broker 的订阅性能（速率、响应时间、错误数）
  - 发布：指定连接数、消息发布速率、消息大小、QoS，测试 MQTT Broker 的消息吞吐性能（速率、响应时间、错误数）
- 开源性能测试框架 JMeter 也有 MQTT 性能测试的功能，基于此的 XMeter 提供性能测试商业服务。

### 性能测试场景

此处采用 HiveMQ 提供的 broker.hivemq.com 在线服务器作为测试 Broker，场景如下：

- 消息发布吞吐：1000ms/10ms * 100 连接 = 10K/秒
- 消息转发吞吐：100 连接 *1 订阅* 10K/s = 1000K = 100W

下图为结果，HiveMQ 应该是做了相应的限制，PUB 测试会报错，实际测试自己的 MQTT Broker 性能时应当按需调节。

![image-20200623175355400](https://tva1.sinaimg.cn/large/007S8ZIlgy1gg2dfwzuk6j31ji0g6dpw.jpg)

### 下载安装测试工具

复制

```
git clone https://github.com/emqx/emqtt-bench.git

cd emqtt-bench

./emqtt-bench sub --help
```

### MQTT 连接性能测试

建立 100 个客户端连接

复制

```
./emqtt_bench conn -c 100 -h broker.hivemq.com
```

### MQTT 订阅性能测试

建立 100 个客户端连接，每 10ms 建立一个连接，每个连接均订阅 testtopic/# 主题，QoS 为 2

复制

```
./emqtt_bench sub -c 100 -i 10 -t testtopic/# -q 2 -h broker.hivemq.com
```

### MQTT 发布性能测试

建立 100 个客户端连接，每 10ms 建立一个连接，每个连接 10ms 发布一次消息，每个连接均向 testtopic/${clientid} 主题发布消息，单条消息尺寸为 256 Bytes，消息 QoS 为 2

复制

```
./emqtt_bench pub -c 100 -i 10 -I 10 -t testtopic/%i -s 256 -q 2 -h broker.hivemq.com
```

## MQTT Broker 职责与需求

**消息队列**与**消息中间件**适用场景不一样。

MQTT 与消息队列有一定的区别，队列是一种**先进先出**的数据结构，消息队列常用于应用服务层面，实现参考如 RabbitMQ Kafka RocketMQ；

MQTT 是传输协议，绝大部分 MQTT Broker 不保证消息顺序（Queue），常用语物联网、消息传输等，MQTT Broker 的常见需求可参考：[共享行业的分布式 MQTT 设计](https://blog.csdn.netesttopic/java060515/article/details/80129549)

### 消息队列与MQTT异同

| 场景                          | 部署端   | MQTT                                         | 消息队列                           |
| :---------------------------- | :------- | :------------------------------------------- | :--------------------------------- |
| 设备端上报状态数据、设备通信  | 移动终端 | √                                            | ×                                  |
| 接收并处理分析设备的上报数据  | 移动终端 | ×                                            | √                                  |
| 对多个设备下发控制指令        | 服务器   | ×                                            | √                                  |
| 直播、弹幕、聊天 App 收发消息 | 应用     | √                                            | ×                                  |
| 服务端接收并分析聊天消息      | 服务器   | ×                                            | √                                  |
| 客户端连接数                  |          | 客户端规模庞大，百万甚至千万级               | 一般服务器规模较小，极少数万级     |
| 单客户端消息量                |          | 单个客户端需要处理的消息少，一般定时收发消息 | 单个客户端处理消息量大，注重吞吐量 |

## EMQ X Docker 安装

复制

```
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8083:8083 -p 8084:8084 -p 18083:18083 emqx/emqx
```

启动之后打开 [http://localhost:18083](http://localhost:18083/)

EMQ X Dashboard 默认用户名密码：

用户名：admin

密码：public

## HiveMQ Docker 安装

复制

```
docker run -p 8080:8080 -p 1883:1883 -p 8083:8083 hivemq/hivemq4
```

启动之后打开 [http://localhost:8080](http://localhost:8080/)

HiveMQ 默认用户名密码：

用户名：admin

密码：hivemq

## VerneMQ Docker 安装

复制

```
docker run -p 1883:1883 -e "DOCKER_VERNEMQ_ACCEPT_EULA=yes" --name vernemq1 -d vernemq/vernemq
```

## Mosquitto Docker 安装

复制

```
docker run -it --name=mosquitto -p 1883:1883 -p 9001:9001 -d eclipse-mosquitto
```

9001 是 Mosquitto WebSocket 端口。

## 全部 MQTT Broker 与 MQTT 服务列表

- [EMQ X](https://github.com/emqx/emqx). Scalable and Reliable Real-time MQTT 5.0 Message Broker for IoT in 5G Era.
- [Adafruit IO](http://io.adafruit.com/)
- [HiveMQ](http://www.hivemq.com/)
- [ActiveMQ](http://activemq.apache.org/)
- [ActiveMQ Artemis](http://activemq.apache.org/artemis)
- [RabbitMQ](https://www.rabbitmq.com/)
- [Mosquitto](http://mosquitto.org/)
- [flespi](https://flespi.com/mqtt-broker)
- [IBM MessageSight](http://www-03.ibm.com/software/products/en/iot-messagesight)
- [Mosca](http://www.mosca.io/). More recently by the same author: [Aedes](https://www.npmjs.com/package/aedes)
- [MQTT Dashboard](http://mqttdashboard.com/dashboard)
- [Eclipse IoT](http://iot.eclipse.org/)
- [VerneMQ](https://verne.mq/)
- [Solace](http://dev.solacesystems.com/)
- [CloudMQTT](https://www.cloudmqtt.com/)
- [Wave](https://github.com/gbour/wave)
- [vertx-mqtt-broker](https://github.com/GruppoFilippetti/vertx-mqtt-broker)
- [JoramMQ](http://www.scalagent.com/en/jorammq-33/products/overview)
- [Moquette MQTT](https://github.com/andsel/moquette)
- [MQTTnet](https://github.com/chkr1011/MQTTnet). Embedded MQTT broker, C#
- [MyQttHub](https://myqtthub.com/)
- [Jmqtt](https://github.com/Cicizz/jmqtt)