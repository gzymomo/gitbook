# Ambari概述

Ambari是hadoop分布式集群配置管理工具，是由hortonworks主导的开源项目。它已经成为apache基金会的孵化器项目，已经成为hadoop运维系统中的得力助手，引起了业界和学术界的关注。



<font color='red'>Apache Ambari是一种基于Web的工具，支持Apache  Hadoop集群的创建、管理和监控。Ambari已支持大多数Hadoop组件，包括HDFS、MapReduce、Hive、Pig、  Hbase、Zookeeper、Sqoop和Hcatalog等；除此之外，Ambari还支持Spark、Storm等计算框架及资源调度平台YARN。</font>

<font color='blue'>Apache Ambari 从集群节点和服务收集大量信息，并把它们表现为容易使用的，集中化的接口：Ambari Web。</font>

Ambari Web显示诸如服务特定的摘要、图表以及警报信息。可<font color='red'>通过Ambari  Web对Hadoop集群进行创建、管理、监控、添加主机、更新服务配置等</font>；也可以<font color='red'>利用Ambari Web执行集群管理任务</font>，例如启用  Kerberos 安全以及执行Stack升级。任何用户都可以查看Ambari Web特性。拥有administrator-level  角色的用户可以访问比 operator-level 或 view-only 的用户能访问的更多选项。例如，Ambari  administrator 可以管理集群安全，一个 operator 用户可以监控集群，而 view-only  用户只能访问系统管理员授予他的必要的权限。



Ambari采用的不是一个新的思想和架构，也不是完成了软件的新的革命，而是充分利用了一些已有的优秀开源软件，巧妙地把它们结合起来，使其在分布式环境中做到了集群式服务管理能力、监控能力、展示能力。这些优秀开源软件有：

- 在agent端，采用了puppet管理节点;
- 在Web端，采用了ember.js作为前端的MVC构架和NodeJS相关工具，用handlebars.js作为页面渲染引擎，在CSS/HTML方面还用了Bootstrap 框架;
- 在Server端，采用了Jetty, Spring，Jetty，JAX-RS等;
- 同时利用了Ganglia，Nagios的分布式监控能力。

Ambari架构采用的是Server/Client的模式，主要由两部分组成：ambari-agent和ambari-server。ambari依赖其它已经成熟的工具，例如其ambari-server 就依赖python，而ambari-agent还同时依赖ruby, puppet，facter等工具，还有它也依赖一些监控工具nagios和ganglia用于监控集群状况。其中：

- puppet是分布式集群配置管理工具，也是典型的Server/Client模式，能够集中式管理分布式集群的安装配置部署，主要语言是ruby。
- facter是用python写的一个节点资源采集库，用于采集节点的系统信息，例如OS信息，主机信息等。由于ambari-agent主要是用python写的，因此用facter可以很好地采集到节点信息。

# 一、Ambari系统架构

<font color='red'>Ambari 自身也是一个分布式架构的软件，主要由两部分组成：Ambari Server 和 Ambari  Agent。</font>简单来说，<font color='red'>用户通过Ambari Server通知 Ambari Agent 安装对应的软件；Agent  会定时地发送各个机器每个软件模块的状态给 Ambari Server，最终这些状态信息会呈现在 Ambari 的  GUI，方便用户了解到集群的各种状态，并进行相应的维护。</font>

<font color='red'>Ambari Server 从整个集群上收集信息。每个主机上都有 Ambari Agent, Ambari Server 通过 Ambari Agent 控制每部主机。</font>



除了ambari-server和ambari-agent，ambari还提供一个界面清亮的管理监控页面ambari-web，这些页面由ambari-server提供。ambari-server开放了REST API，这些API也主要分两大类，其中一类为ambari-web提供管理监控服务，另一类用于与ambari-agent交互，接受ambari-agent向ambari-server发送的心跳请求。下图是Ambari的系统架构。其中master模块接受API和Agent Interface的请求，完成ambari-server的集中式管理监控逻辑，而每个agent节点只负责所在节点的状态采集及维护。
![52fa1fbc-e2ad-369b-9028-ecc6ce25502c](https://img-blog.csdnimg.cn/img_convert/137b96365d9456f01617aaf4ffb0a570.png)

# 二、Ambari-Agent内部架构

ambari-agent是一个无状态的。其功能主要分两部分：

- 采集所在节点的信息并且汇总发心跳汇报给ambari-server;
  处理ambari-server的执行请求。

因此它有两种队列：

- 消息队列MessageQueue，或为ResultQueue。包括节点状态信息（包括注册信息）和执行结果信息，并且汇总后通过心跳发送给ambari-server;
- 操作队列ActionQueue。用于接收ambari-server返回过来的状态操作，然后能过执行器按序调用puppet或python脚本等模块完成任务。

![09b223bb-e18c-3280-8f77-bd604518a2b5](https://img-blog.csdnimg.cn/img_convert/203df7032144a3a1912fb6885b91308f.png)

# 三、Ambari-Server内部架构

ambari-server是一个有状态的，它维护着自己的一个有限状态机FSM。同时这些状态机存储在数据库中，前期数据库主要采用postgres。如下图所示，server端主要维护三类状态：

- Live Cluster State：集群现有状态，各个节点汇报上来的状态信息会更改该状态;
- Desired State：用户希望该节点所处状态，是用户在页面进行了一系列的操作，需要更改某些服务的状态，这些状态还没有在节点上产生作用;
- Action State：操作状态，是状态改变时的请求状态，也可以看作是一种中间状态，这种状态可以辅助Live Cluster State向Desired State状态转变。

![a1ceaae2-189e-3cce-bcf3-3be37139a3ca](https://img-blog.csdnimg.cn/img_convert/cf0e07760e40a512de3e417ccdf256b1.png)

Ambari-server的Heartbeat Handler模块用于接收各个agent的心跳请求（心跳请求里面主要包含两类信息：节点状态信息和返回的操作结果），把节点状态信息传递给FSM状态机去维护着该节点的状态，并且把返回的操作结果信息返回给Action Manager去做进一步的处理。

Coordinator模块又可以称为API handler，主要在接收WEB端操作请求后，会检查它是否符合要求，stage planner分解成一组操作，最后提供给Action Manager去完成执行操作。

因此，从上图就可以看出，Ambari-Server的所有状态信息的维护和变更都会记录在数据库中，用户做一些更改服务的操作都会在数据库上做一些相应的记录，同时，agent通过心跳来获得数据库的变更历史。
