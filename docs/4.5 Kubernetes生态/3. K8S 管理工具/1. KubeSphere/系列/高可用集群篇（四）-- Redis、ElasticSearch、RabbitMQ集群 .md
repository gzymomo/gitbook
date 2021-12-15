- [高可用集群篇（一）-- k8s集群部署](https://juejin.cn/post/6940887645551591455)
- [高可用集群篇（二）-- KubeSphere安装与使用](https://juejin.cn/post/6942282048107347976)
- [高可用集群篇（三）-- MySQL主从复制&ShardingSphere读写分离分库分表](https://juejin.cn/post/6944142563532079134)
- [高可用集群篇（四）-- Redis、ElasticSearch、RabbitMQ集群](https://juejin.cn/post/6945360597668069384)
- [高可用集群篇（五）-- k8s部署微服务](https://juejin.cn/post/6946396542097948702)



## Redis集群搭建

### Redis集群形式

#### 数据分区方案

##### 客户端分区

- 客户端分区方案的代表为 Redis Sharding、Redis Sharding 是 Redis Cluster出来之前，业界普遍使用的Redis多实例集群方法；Java的Redis客户端驱动库Jedis，支持Redis Sharding 功能，即 ShardedJedis 以及结合缓存池的 ShardedJedisPool

  ![image-20210320161547744](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71922685f01b40daad47fa0dc0548679~tplv-k3u1fbpfcp-zoom-1.image)

- 优点：不使用第三方中间件，分区逻辑可控，配置简单，节点之间无关联，容易线性扩展，灵活性强

- 缺点：客户端无法动态增删服务节点，客户端需要自行维护分发逻辑，客户端之间无法连接共享，会造成资源浪费

##### 代理分区

- 带来分区常用方案有 Twemproxy 和 Codis

  ![image-20210320161633138](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e42608efe3145ba9c3b19d1d2b0b0f1~tplv-k3u1fbpfcp-zoom-1.image)

##### redis-cluster

#### 高可用方式

##### Sentinel（哨兵机制）支持高可用

- 前面介绍了主从机制，但是从运维角度来看，主节点出现了问题我们还需要通过人工干预的方式把从节点设为主节点，还要通知应用程序更新主节点地址，这种方式非常繁琐笨重，而且主节点的读写能力都十分有限，有没有较好的办法解决这两个问题，哨兵机制就是针对第一个问题的有效解决方案，第二个问题则依赖于集群；哨兵的作用就是监控Redis系统的运行状况，其功能主要包括以下三个：

  - **监控**（Monitoring）：哨兵会不断地检查你的Master和Slave是否运作正常
  - **提醒**（Notification）：当被监控的某个Redis出现问题时，哨兵可以通过API向管理员或者其他应用程序发送通知
  - **自动故障迁移**（Automatic failover）：当主数据库出现故障时自动将从数据库转换为主数据库

  - ![image-20210321120005360](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4e039d9741c4a73b49d7601cc7733b4~tplv-k3u1fbpfcp-zoom-1.image)
  
  ##### 

##### 哨兵的原理

- Redis哨兵的三个定时任务，Redis哨兵判定一个Redis节点故障不可达主要就是**通过三个定时监控任务来完成**的：

  - 每隔10秒每隔哨兵节点会向主节点和从节点发送 `info replication`命令来获取最新的拓扑结构

    ![image-20210321120241404](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71564ead8a224126b8c7bbd3d1a45cca~tplv-k3u1fbpfcp-zoom-1.image)

  - 每隔2秒每隔哨兵节点回向Redis节点的`_sentinel_:hello`频道发送自己对主节点是否故障的判断以及自身的节点信息，并且其他的哨兵节点也会订阅这个频道来了解其他哨兵节点的信息以及对主节点的判断

    ![image-20210321120358665](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21595a74fc274e38b81a1f5865c9858d~tplv-k3u1fbpfcp-zoom-1.image)

  - 每隔1秒每个哨兵节点会向主节点、从节点、其他的哨兵节点发送一个 `ping`命令来做心跳检测

    ![image-20210321120648877](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e60c1d1ad41745db99d1c218c74f54c8~tplv-k3u1fbpfcp-zoom-1.image)

    如果在定时Job3检测不到节点的心跳，会判断为主观下线；如果该节点还是主节点那么还会通知其他的哨兵对该主节点进行心跳检测，这时主观下线的票数就超过了数时，那么这个主节点确实就可能是故障不可达了，这时就由原来的主观下线变为客观下线

  - 故障转移和Leader选举

    如果主节点被判定为客观下线之后，就要选取一个哨兵节点来完成后面的故障转移工作，选举一个leader，这里采用的选举算法为Raft；选举出来的哨兵leader就要来完成故障转移工作，也就是在从节点中选举出一个节点来当心的主节点

### Redis-Cluster

- [redis-cluster文档](https://redis.io/topics/cluster-tutorial)

- Redis的官方多机器部署方案：Redis Cluster；一组 Redis Cluster是由多个Redis实例组成，官方推荐我们是6实例，其中三个为主节点，三个为从节点；一旦有主节点发生故障的时候，Redis Cluster可以选举出对应的从节点成为新的主节点，继续对外服务，从而保证服务的高可用性；

- 那么对于客户端来说，怎么知道对应的key是要路由到哪一个节点呢？

  Redis Cluster把所有的数据划分为**16384个不同的槽位**，可以根据机器的性能把不同的槽位分配给不同的Redis实例，对于Redis实例来说，他们只会存储部分的Redis数据，当然，槽的数据是可以迁移的，不听的实例之间可以通过一定的协议，进行数据迁移

  - ![image-20210321133537139](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11831b67856c4fee9601aa51040b6a76~tplv-k3u1fbpfcp-zoom-1.image)

#### 槽

- Redis集群的功能限制；Redis集群相对于单机在功能上存在一些限制，需要开发人员提前了解，在使用时做好规避；**JAVA CRC16校验算法**

  ![image-20210321134106832](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/601eafdca4604350bbd4d1b5936dd771~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210321134132985](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94812211fba74d828cfdb9837ae7008d~tplv-k3u1fbpfcp-zoom-1.image)

- 可以批量操作 支持有限

  - 类似mset、mget操作，目前只支持对具有相同slot值的key执行批量操作；对于映射为不同slot值的key由于执行mget、mset等操作可能存在于多个节点上，因此不被支持

- key事务操作 支持有限

  - 只支持多key在同一节点上的事务操作，当多个key分布在不同节点上时无法使用事务功能

- 可以作为数据分区的最小粒度

- 不能将一个大的键值对象 如 hash、list等映射到不同的节点

- 不支持多数据库空间

  - 单机下的Redis可以支持16个数据库（db0 - db15），集群模式下只能使用一个数据库空间，即db0

- 复制结构，只支持一层

  - 从节点只能复制主节点，不支持嵌套树状复制结构

- 命令大多会重定向，耗时多

  - ![image-20210321134605852](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1aaf69b6673434c8ff7132ea53bd45e~tplv-k3u1fbpfcp-zoom-1.image)
  
  

#### 一致性hash

- 一致性hash可以很好地解决稳定性的问题，可以将所有的存储节点排列在首尾相接的Hash环上，每个key在计算Hash后会顺时针找到临接的存储节点存放；而当有节点加入或退出时，仅仅影响该节点在Hash环上顺时针相邻的后续节点

  ![image-20210321135249822](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2494977e7cf74e45bfb9261c3b7bb092~tplv-k3u1fbpfcp-zoom-1.image)

- Hash倾斜

  如果节点很少，容易出现倾斜，负载不均衡问题；一致性哈希算法，引入了虚拟节点，在整个环上，均衡增加若干个节点；比如a1、a2、b1、b2、c1、c2；a1和a2都是属于A节点的，解决了hash倾斜问题

### 部署Cluster

#### 创建6个redis节点

- 三主三从方式，从为了同步备份，主进行slot数据分片

  ![image-20210321143120398](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dedd84e4c8ed498b8bad060a6f49356e~tplv-k3u1fbpfcp-zoom-1.image)

  

  - 创建目录与配置，并启动容器

    ```shell
    for port in $(seq 7001 7006); \
    do \
    mkdir -p /var/mall/redis/node-${port}/conf
    touch /var/mall/redis/node-${port}/conf/redis.conf
    cat <<EOF>> /var/mall/redis/node-${port}/conf/redis.conf
    port ${port}
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-announce-ip 192.168.83.133
    cluster-announce-port ${port}
    cluster-announce-bus-port 1${port}
    appendonly yes
    EOF
    docker run -p ${port}:${port} -p 1${port}:1${port} --name redis-${port} \
    --restart=always \
    -v /var/mall/redis/node-${port}/data:/data \
    -v /var/mall/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
    -d redis redis-server /etc/redis/redis.conf; \
    done
    
    ```

    ![image-20210321142255590](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/070c992ee33f4f9993d58a3c11880d3d~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210321142412697](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05b55288f5944576bf6c9fef337ae361~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210321142345881](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94a92fe8cc684529bf16f93cbe24ab71~tplv-k3u1fbpfcp-zoom-1.image)

    
    
    ```shell
    #停止这6个redis容器
    docker stop $(docker ps -a |grep redis-700 | awk '{print $1}')
    #删除这6个redis容器
    docker rm $(docker ps -a | grep redis-700 | awk '{print $1}')
    
    ```

#### 使用redis建立集群

- 进入任一master节点的redis容器

  ```shell
  #进入容器
  docker exec -it redis-7001 /bin/bash
  #建立集群，如果不指定master，会自己随机指定
  redis-cli --cluster create 192.168.83.133:7001 192.168.83.133:7002 192.168.83.133:7003 192.168.83.133:7004 192.168.83.133:7005 192.168.83.133:7006 --cluster-replicas 1
  
  ```

  ![image-20210321143726656](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e65a1203b364418ae14a5674b7ec8a3~tplv-k3u1fbpfcp-zoom-1.image)

  输入`yes`，接受配置，集群搭建完成

  - ![image-20210321143940702](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1af839718dc0441baeaec933d8f03947~tplv-k3u1fbpfcp-zoom-1.image)
  
  #### 

#### 测试redis集群

- 连接redis客户端，保存数据（**分片存储**）

  ```
  #集群模式下，比单机多一个 -c
  redis-cli -c  -h 192.168.83.133 -p 7001
  
  set hello 1
  set a aaa
  set bb aa
  
  ```

  ![image-20210321144535451](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97231bd7a48741b7baf5fc47ad5a46e4~tplv-k3u1fbpfcp-zoom-1.image)

- 查看集群状态

  ```
  cluster info
  
  ```

  ![image-20210321145716188](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f1dfbdd5c2346a9918c802e60903b24~tplv-k3u1fbpfcp-zoom-1.image)

- 查看节点信息

  ```
  cluster nodes
  
  ```

  ![image-20210321145818299](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c17e326d957048f9aa05413658fe7dd1~tplv-k3u1fbpfcp-zoom-1.image)

- 模拟一个主节点宕机

  ```
  docker stop redis-7001
  #进入7002容器
  docker exec -it redis-7002 /bin/bash
  #连接redis客户端
  redis-cli -c  -h 192.168.83.133 -p 7002
  #获取之前存在redis7001 上的 hello的值，这时会发现，7001的副本（从机）会替代宕机的7001主机
  #7005被集群提升为主节点
  get hello
  #查看节点信息
  cluster nodes
  
  ```

  ![image-20210321150203048](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea5a75fe833a4241bdb6e1df7b898ada~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210321150530220](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/630395eb466945d3a94b6df6f88e20a4~tplv-k3u1fbpfcp-zoom-1.image)

- 这时，宕机的7001再次启动，观察节点的变化（**故障切换**）

  ```
  #重启redis-7001容器
  docker restart redis-7001
  #查看节点信息
  cluster nodes
  
  ```

  - ![image-20210321151029146](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/977e1f86dca44d49aa469027c3aeed9e~tplv-k3u1fbpfcp-zoom-1.image)
  
  ## 

## ElasticSearch集群搭建

### es集群原理

- ElasticSearch是天生支持集群的，它不需要依赖其他的服务发现和注册的组件，如zookeeper这些，因为它内置了一个名字叫作`ZenDiscovery`的模块，是ElasticSearch自己实现的一套用于节点发现和选主等功能的组件，所以ElasticSearch做起集群来非常简单，不需要太多额外的配置的安装额外的第三方组件

#### 单节点

- 一个运行中的ElasticSearch实例称为一个节点，而集群是由一**个或者多个拥有相同cluster.name配置的节点组成**；它们共同承担数据和负载的压力，当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据
- 当一个节点被选举称为**主节点**时，它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等；而主节点并不需要涉及到文档级别的变更和搜索等操作，所有当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈，任何节点都可以成为主节点；我们的示例集群就只有一个节点，所以它同时也成为了主节点
- 作为用户，我们可以将请求发送到集群中的任何节点，包括主节点；每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点；无论我们将请求发送到哪个子节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回给客户端，ElasticSearch对这一切的管理都是透明的

#### 集群健康

- ElasticSearch的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是集群健康，它在status字段中展示为green、yellow或者red

  ```
  GET /_cluster/health
  
  ```

  `status`：指示这当前集群在总体上是否工作正常，它的三种颜色含义如下：

  ​	`green`：所有的主分片和副本分片都正常运行

  ​	`yellow`：所有的主分片都正常运行，但不是所有的副本分片都正常运行

  ​	`red`：有主分片没能正常运行

#### 分片

- 一个**分片**是一个底层的工作单元，它仅保存了全部数据中的一部分；我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互；分片就任务是一个数据区

- 一个分片可以是主分片或者副本分片；索引内任意一个文档都归属于一个主分片，所有主分片的数目决定着索引能够保存的最大数据量

- 在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改

- 让我们在包含一个空节点的集群内创建名为blogs的索引，索引在默认情况下会被分配5个主分片，但是为了演示目的：我们将分配3个主分片和一份副本（每个主分片拥有一个副本分片）

  ```
  PUT /blogs{
  	"settings":{
  		"number_of_shards": 3,
  		"number_of_replicas": 1
  	}
  }
  
  ```

- 拥有一个索引的单节点集群

  ![image-20210321154512225](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcdde12f07df4693a4d8335e62b592e6~tplv-k3u1fbpfcp-zoom-1.image)

  此时集群的健康状况为 `yellow` 则表示全部 **主分片都正常运行**（集群可以正常服务所有请求），但是**副本分片没有全部处在正常状态**；实际上，所有三个副本分片都是 `unassigned` --- 它们都没有被分配带任何节点；

  **在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据**

  当我们的集群是正常运行的，但是在硬件故障是有丢失数据的风险

#### 新增节点

- 当你在同一台机器上启动了第二个节点时，只有它和第一个节点有同样的`cluster.name`配置，它就会自动发现集群并加入到其中；但是在不同机器上启动节点的时候，为了加入到同一集群，你需要配置一个可连接到的单播主机列表；详细信息请查看最好使用单播代替组播

- 拥有两个节点的集群 -- 所有主分片和副本分片都已被分配

  ![image-20210321155401965](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd89691db8e441e7b433d2794753a0a1~tplv-k3u1fbpfcp-zoom-1.image)

  此时，cluster-health现在展示的状态为green，这表示所有6个分片（包括3个主分片和3个副本分片）都在正常运行；我们的集群现在不仅仅是正常运行的，并且还始终处于**可用**状态

#### 水平扩容 -- 启动第三个节点

- 用于三个节点的集群 -- 为了分散负载而对分片进行重新分配

  ![image-20210321155829355](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3108e6c4f964351af12a4a730d56b26~tplv-k3u1fbpfcp-zoom-1.image)

  Node1 和 Node2 上各有一个分片被迁移到新的 Node3 节点，现在每个节点上都拥有2个分片，而不是之前的3个；这表示每个节点的硬件资源（CPU、RAM、I/O）将被更少的分片所共享，每个分片的性能将会得到提升

  在运行的集群上可以动态调整副本分片数目的，我们可以按需伸缩集群；让我们把副本数从默认的1增加到2

  ```
  PUT /blogs/_settings
  {
  	"number_of_replicas": 2
  }
  
  ```

  blogs 索引现在拥有9个分片：3个主分片和6个副本分片；这意味着我们可以将集群扩容到9个节点，每个节点上一个分片；相比原来的3个节点时，集群的搜索性能可以提升3倍

  - ![image-20210321160529979](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/faa0a92cd5004a2b8bd075ea8af22646~tplv-k3u1fbpfcp-zoom-1.image)
  
  #### 

#### 应对故障

- 关闭一个节点后的集群

  ![image-20210321160617533](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7888a63271bb42bf92e608a10fc0484d~tplv-k3u1fbpfcp-zoom-1.image)

- 我们关闭的节点是一个主节点；而集群必须拥有一个主节点来保证正常工作，所以发生的第一件事情就是选举一个新的主节点：Node2

- 在我们关闭Node1的同时也失去了主分片1和2，并且在缺失主分片的时候索引也不能正常工作；如果此时来检查集群的状况，我们看到的状态将会是red：不是所有主分片都在正常工作

- 幸运的是：在其它节点上存在着这两个主分片的完整副本，所以新的主节点立即将这些分片在Node2和Node3上对应的副本提升为主分片，此时集群的状态将会为yellow，这个提升主分片的过程是瞬间发生的，如同按下一个开关一般

- 为什么我们**集群状态是yellow而不是green呢**？虽然我们拥有所有的三个主分片但是同时设置了每个主分片需要对应的两个副本分片，而此时只存在一份副本分片；所以集群不是green的状态

  不过我们不必过于担心：如果我们同样关闭了Node2，我们的程序依然可以保持在不丢失任何数据的情况下运行，因为Node3为每一个分片都保留着一份副本

- 如果我们重新启动Node1，集群可以将缺失的副本分片再次进行分配；如果Node1依然拥有着之前的分片，它将尝试去重用它们，同时仅从主分片复制发生了修改的数据文件

#### 问题解决

##### 主节点

- 主节点负责创建索引、删除索引、分配分片、追踪集群中的节点状态等工作；ElasticSearch中的主节点的工作量相对较轻，用户的请求可以发往集群中任何一个节点，由该节点负责分发和返回结果，而不需要经过主节点转发；而**主节点是由候选主节点通过ZenDiscovery机制选举出来的，所有想要成为主节点，首先要先成为候选主节点**

##### 候选主节点

- 在ElasticSearch集群初始化或者主节点宕机的情况下，由候选主节点中选举其中一个作为主节点，指定候选主节点的配置为：`node.master:true`

  当主节点负载压力过大，或者集群环境中的网络问题，导致其他节点与主节点的通讯的时候，主节点没来得及响应，这样的话，某些节点就认为主节点宕机，重新选择新的主节点，这样的话整个集群的工作就有问题了；比如我们集群中有10个节点，其中7个候选主节点，1个候选主节点成为了主节点，这种情况是正常情况；但是如果现在出现了上述我们所说的主节点响应不及时，导致其他某些节点任认为主节点宕机而重选主节点，那就有问题了，这剩下的6个候选主节点可能有3个候选主节点去重选主节点，最后集群中就出现了两个主节点的情况，这种情况官方称为**脑裂现象**

  集群中不同的节点对应master的选择出现了分歧，出现了多个master竞争，导致主分片和副本的识别也发生了分歧，**对一些分歧中的分片标识为了坏片**

##### 数据节点

- 数据节点负责数据的存储与相关具体操作，比如CRUD、搜索、聚合；所以，数据节点对机器的配置要求比较高，首先需要有足够的磁盘空间来存储数据，其次数据操作对系统CPU、Memory和IO的性能消耗都很大；通常随着集群的扩大，需要增加更多的数据节点来提高可用性；指定数据节点的配置：`node.data:true`

  ElasticSearch是允许一个节点既做候选主节点也做数据节点的，但是数据节点的负载较重，所以需要考虑将二者分离开，设置专用的候选主节点和数据节点，避免因数据节点负载重导致主节点不响应

##### 客户端节点

- 客户端节店就是既不做候选主节点也不做数据节点的节点，只负责请求的分发、汇总等等，但是这样的工作，其实任何一个节点都可以完成，因为在ElasticSearch中一个集群内的节点都可以执行任何请求，其会负责将请求转发给对应的节点进行处理；所以单独增加这样的节点更多的是为了负载均衡；指定该节点的配置为：

  `node.master:false`

  `node.data:false`

##### 脑裂问题可能的成因

- 1、网络问题：集群键的网络延迟导致了一些节点访问不到master，任务master节点挂掉了从而选举出新的master，并对master上的分片和副本标红，分配新的主分片

- 2、节点负载：主节点的角色既为master又为data，访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点等不到主节点的响应认为主节点挂掉了，会重新选取主节点

- 3、内存回收：data节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应

- 脑裂问题解决方案：

  - **角色分离**：即master节点与data节点分离，限制角色；数据节点是需要承担存储和搜索的工作的，压力会很大；所以如果该节点同时作为候选主节点和数据节点，那么一旦选上它作为主节点了，这时主节点的工作压力会非常大，出现脑裂现象的概率就增加了

  - **减少误判**：配置主节点的响应时间，在默认情况下，主节点3秒没有响应，其他节点就任务主节点宕机看，那我们可以把该时间设置的长一点，该配置是：`discover.zen.ping_timeout:5`

  - 选举触发

    ：

    ```
    discovery.zen.minimum_master_nodes:1
    ```

    （默认是1），该属性定义的是为了形成一个集群，有主节点资格并互相连接的节点的最小数目

    - 一个有10个节点的集群，且每个节点都有成为主节点的资格，`discovery.zen.minimum_master_nodes`参数设置为6
    - 正常情况下，10个节点，互相连接，大于6，就可以形成一个集群
    - 若某个时刻，其中有3个节点断开连接，剩下7个节点，大于6，继续运行之前的集群，而断开的3个节点，小于6，不能形成一个集群
    - **该参数就是为了防止脑裂的产生**
    - 建议设置为（候选主节点数 / 2）+ 1

#### 集群结构

- 以三台物理机为例，在这三台物理机上，搭建了6个ES的节点，三个data节点，三个master节点（每台物理机分别起了一个data和一个master），3个master节点，目的是达到（n/2）+1等于2的要求，这样挂掉一台master后（不考虑data），n等于2，满足参数，其他两个master节点都认为master节点挂掉之后开始重新选举

  ```
  #master节点上
  node.master=true
  node.data=false
  discovery.zen.minimum_master_nodes=2
  
  #data节点上
  node.master=falsse
  node.data=true
  
  
  ```

  - ![image-20210322091009844](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7046d6e6a0434ad4a62a2b1b33e7198c~tplv-k3u1fbpfcp-zoom-1.image)
  
  ### 

### es集群搭建

- es集群：三个主节点，三个数据节点

  ![image-20210322092736711](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c70a39feaa1f43ffa805f8e8b7415d5b~tplv-k3u1fbpfcp-zoom-1.image)

- 所有操作之前先运行

  ```
  #防止JVM报错
  #临时修改
  sysctl -w vm.max_map_count=262144
  #永久修改
  echo vm.max_map_count=262144>>/etc/sysctl.conf
  sysctl -p
  
  ```

#### 准备docker网络

- Docker创建容器时默认采用`bridge`网络，自行分配ip，不允许自己指定

  在实际部署中，我们需要指定容器ip，不允许其自行分配ip，尤其是搭建集群时，固定ip是必须的；我们可以创建自己的`bridge`网络：`mynet`，创建容器的时候指定网络为`mynet`并指定ip即可

  查看网络模式：`docker network ls`

  创建一个新的`bridge`网络

  ```
  #创建mynet
  docker network create --driver bridge --subnet=172.18.12.0/16 --gateway=172.18.1.1 mynet
  #查看网络信息
  docker network inspect mynet
  
  ```

  ![image-20210322101740636](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e415d9c24c214292942232c320c02934~tplv-k3u1fbpfcp-zoom-1.image)

- 以后使用 `--network=mynet --ip 172.18.12.x`，指定ip

#### 创建三个Msater节点

- 脚本生成配置并启动容器

  ```shell
  #discovery.zen.minimum_master_nodes:2 
  #设置这个参数来保证集群中的节点可以知道其他N个有master资格的节点；官方推荐 （N/2）+ 1
  for port in $(seq 1 3); \
  do \
  mkdir -p /var/mall/elasticsearch/master-${port}/config
  mkdir -p /var/mall/elasticsearch/master-${port}/data
  chmod -R 777 //var/mall/elasticsearch/master-${port}
  cat<<EOF> /var/mall/elasticsearch/master-${port}/config/jvm.options
  -Xms128m
  -Xmx128m
  EOF
  cat<<EOF> /var/mall/elasticsearch/master-${port}/config/elasticsearch.yml
  cluster.name: my-es #集群的名称，同一个集群值必须设置成一样的
  node.name: es-master-${port} #该节点的名字
  node.master: true #该节点有机会成为master节点
  node.data: false #该节点可以存储数据
  network.host: 0.0.0.0
  http.host: 0.0.0.0 #所有http均可访问
  http.port: 920${port}
  transport.tcp.port: 930${port}
  discovery.zen.ping_timeout: 10s #设置集群中自动发现其他节点时ping的超时时间
  #设置集群中的Master节点的初始化列表，可以通过这些节点来自动发现其他新加入集群的节点，es7的新增配置
  discovery.seed_hosts: ["172.18.12.21:9301","172.18.12.22:9302","172.18.12.23:9303"]
  cluster.initial_master_nodes: ["172.18.12.21"] #新集群初始时的候选主节点，es7新增
  EOF
  docker run -d --name=es-node-${port} \
  -p 920${port}:920${port} -p 930${port}:930${port} --network mynet --ip 172.18.12.2${port} \
  -v /var/mall/elasticsearch/master-${port}/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /var/mall/elasticsearch/master-${port}/config/jvm.options:/usr/share/elasticsearch/config/jvm.options \
  -v /var/mall/elasticsearch/master-${port}/data:/usr/share/elasticsearch/data \
  -v /var/mall/elasticsearch/master-${port}/plugins:/usr/share/elasticsearch/plugins \
  elasticsearch:7.6.1;\
  done
  
  ```

  ```
  #停止全部es容器
  docker stop $(docker ps -a |grep es-node-*|awk '{print $1}')
  #删除全部es容器
  docker rm -f $(docker ps -a |grep es-node-*|awk '{print $1}')
  
  ```

  - ![image-20210322143827246](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61f68e1f383d49cbad4f9011d06d0fba~tplv-k3u1fbpfcp-zoom-1.image)
  
  #### 

#### 创建三个Data节点

- 创建脚本与Master不同之处

  ![image-20210322144802351](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0351aefb7f3d42cfa59c3ea0d6e1ff3b~tplv-k3u1fbpfcp-zoom-1.image)

  ```shell
  
  ```
  
  ```shell
  for port in $(seq 4 6); \
  do \
  mkdir -p /var/mall/elasticsearch/node-${port}/config
  mkdir -p /var/mall/elasticsearch/node-${port}/data
  chmod -R 777 //var/mall/elasticsearch/node-${port}
  cat<<EOF> /var/mall/elasticsearch/node-${port}/config/jvm.options
  -Xms128m
  -Xmx128m
  EOF
  cat<<EOF> /var/mall/elasticsearch/node-${port}/config/elasticsearch.yml
  cluster.name: my-es #集群的名称，同一个集群值必须设置成一样的
  node.name: es-node-${port} #该节点的名字
  node.master: false #该节点有机会成为master节点
  node.data: true #该节点可以存储数据
  network.host: 0.0.0.0
  http.host: 0.0.0.0 #所有http均可访问
  http.port: 920${port}
  transport.tcp.port: 930${port}
  discovery.zen.ping_timeout: 10s #设置集群中自动发现其他节点时ping的超时时间
  #设置集群中的Master节点的初始化列表，可以通过这些节点来自动发现其他新加入集群的节点，es7的新增配置
  discovery.seed_hosts: ["172.18.12.21:9301","172.18.12.22:9302","172.18.12.23:9303"]
  cluster.initial_master_nodes: ["172.18.12.21"] #新集群初始时的候选主节点，es7新增
  EOF
  docker run -d --name=es-node-${port} \
  -p 920${port}:920${port} -p 930${port}:930${port} --network mynet --ip 172.18.12.2${port} \
  -v /var/mall/elasticsearch/node-${port}/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /var/mall/elasticsearch/node-${port}/config/jvm.options:/usr/share/elasticsearch/config/jvm.options \
  -v /var/mall/elasticsearch/node-${port}/data:/usr/share/elasticsearch/data \
  -v /var/mall/elasticsearch/node-${port}/plugins:/usr/share/elasticsearch/plugins \
  elasticsearch:7.6.1;\
  done
  
  ```
  
  ```
  #查看es容器日志信息
  #主节点
  docker logs es-node-1
  #数据节点
  docker logs es-node-4
  
  ```
  
  ![image-20210322160539096](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fe16f0ead6040308bdf3f2ab778996f~tplv-k3u1fbpfcp-zoom-1.image)
  
  ![image-20210322161711340](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0de609b90c749af95dd9cd02460fbfd~tplv-k3u1fbpfcp-zoom-1.image)

#### 检查es集群的工作状况

- 1、浏览器访问 `192.168.83.133:9201 -- 9206` 都可以正常访问

  ![image-20210322161918898](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/405fad49bd464ae1a94004a63b658eba~tplv-k3u1fbpfcp-zoom-1.image)

- 2、部分命令

  - `/_nodes/process?pretty` ：查看节点状况

  - `/_cluster/stats?pretty`：查看集群状态

  - `/_cluster/health?pretty`：查看集群健康状况

  - `/_cat/nodes`：查看各个节点信息

    ![image-20210322162615183](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38ab120d3c4a4f8eab3c40573d0029e9~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210322162628675](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a34e04a0873c4291bda615c4556b25bf~tplv-k3u1fbpfcp-zoom-1.image)

    

    - 这时可以看到 主节点是 `es-master-3`，尝试停止 `es-master-3`容器，观察结果：

      `docker stop es-node-3`

      **会发现触发选主机制，`es-master-1`节点被选举为主节点**

      ![image-20210322163232346](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ec252790e6249619f857aef1dcf890b~tplv-k3u1fbpfcp-zoom-1.image)

      再次启动 `es-master-3`，这时集群会认为这是一个新添节点，`es-master-1`节点仍然是主节点

      `docker restart  es-node-3`

      ![image-20210322163708884](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98115a54afe1442cb744f7c877051a27~tplv-k3u1fbpfcp-zoom-1.image)

## RabbitMQ集群搭建

### 集群形式

- RabbitMQ是Erlang开发的，集群非常方便，因为Erlang天生就是一门分布式语言，但其本身并不支持负载均衡
- RabbitMQ集群中节点包括**内存节点（RAM）、磁盘节点（Disk、消息持久化）**，集群中至少有一个Disk节点

#### 普通模式（默认）

- 对于普通模式，集群中各个节点有相同的队列结构，但消息只会存在于集群中的一个节点；对于消费者来说，若消息进入A节点的Queue中，当从B节点拉取时，RabbitMQ会将消息从A中取出，并经过B发送给消费者
- 应用场景：该模式各适合于消息无需持久化的场合，如日志队列；当队列非持久化，且创建该队列的节点宕机，客户端才可以重连集群其他节点，并重新创建队列；若为持久化，只能等故障节点恢复

#### 镜像模式

- 与普通模式不同之处是消息实体会自动在镜像节点间同步，而不是在取数据时临时拉取，高可用；该模式下，mirror queue 有一套选举算法，即1个master、n个salve，生产者、消费者的请求都会转至master

- 应用场景：可靠性要求较高场合，如下单、库存队列

  缺点：若镜像队列过多，且消息体量大，集群内部的网络带宽将会被此种同步通讯所消耗

  - 1、镜像集群也是基于普通集群，即只有先搭建普通集群，然后才能设置镜像队列
  - 2、若消费过程中，master挂掉，则选举新master,若未来得及确认，则可能会重复消费

### 搭建MQ集群

- 创建目录

  ```
  mkdir -p /var/mall/rabbitmq/rabbitmq01
  mkdir -p /var/mall/rabbitmq/rabbitmq02
  mkdir -p /var/mall/rabbitmq/rabbitmq03
  
  ```

- 启动mq容器

  ```shell
  #rabbitmq01
  docker run -d --hostname rabbitmq01 --name rabbitmq01 -v /var/mall/rabbitmq/rabbitmq01:/var/lib/rabbitmq  -p 15671:15672 -p 5671:5672 -e RABBITMQ_ERLANG_COOKIE='mall' rabbitmq:management
  
  #rabbitmq02
  docker run -d --hostname rabbitmq02 --name rabbitmq02 -v /var/mall/rabbitmq/rabbitmq02:/var/lib/rabbitmq  -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='mall' --link rabbitmq01:rabbitmq01  rabbitmq:management
  
  #rabbitmq03
  docker run -d --hostname rabbitmq03 --name rabbitmq03 -v /var/mall/rabbitmq/rabbitmq03:/var/lib/rabbitmq  -p 15673:15672 -p 5673:5672 -e RABBITMQ_ERLANG_COOKIE='mall' --link rabbitmq01:rabbitmq01  --link rabbitmq02:rabbitmq02 rabbitmq:management
  
  # --hostname 设置容器的主机名
  RABBITMQ_ERLANG_COOKIE 节点认证作用，部署集群时，需要同步该值
  
  ```

  - ![image-20210323083243934](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf0e05a883b44169b9d0ffcc52a99aab~tplv-k3u1fbpfcp-zoom-1.image)
  
  ### 

### 节点加入集群

- 进入rabbitmq01容器

  ```
  docker exec -it rabbitmq01 /bin/bash
  
  #清空、初始化mq
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl start_app
  exit
  
  ```

  ![image-20210323084521683](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7d1dd2f6e934c7da0d3947c66ad2b85~tplv-k3u1fbpfcp-zoom-1.image)

- 进入第二个节点 rabbitmq02

  ```
  docker exec -it rabbitmq02 /bin/bash
  
  #清空、初始化mq，并加入集群
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl join_cluster --ram rabbit@rabbitmq01
  rabbitmqctl start_app
  exit
  
  ```

  ![image-20210323084847937](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8d0703a9194405499d02cc97d93eba0~tplv-k3u1fbpfcp-zoom-1.image)

- 进入第二个节点 rabbitmq03

  ```
  docker exec -it rabbitmq03 /bin/bash
  
  #清空、初始化mq，并加入集群
  rabbitmqctl stop_app
  rabbitmqctl reset
  rabbitmqctl join_cluster --ram rabbit@rabbitmq01
  rabbitmqctl start_app
  exit
  
  ```

  ![image-20210323085104178](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2802069e3a7f43478e3d4f1e4b21fe62~tplv-k3u1fbpfcp-zoom-1.image)

  **默认集群是普通集群，普通集群存在单点故障的隐患**

  ![image-20210323085245997](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd475c0cb60d49eca9db1095e1fc2fe2~tplv-k3u1fbpfcp-zoom-1.image)

### 实现镜像集群

- 进入任一mq容器

  ```
  docker exec -it rabbitmq01 /bin/bash
  #可以使用 rabbitmqctl list_policies -p /; 查看vhost/下面所有policy
  #设置策略（所有队列都是高可用模式并且自动同步）
  rabbitmqctl set_policy -p / ha "^" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
  
  ```

  ```
  #在cluster中任意节点启用策略，策略会自动同步到集群节点
  rabbitmqctl set_policy -p / ha-all "^" '{"ha-mode":"all"}'
  #策略模式all即复制到所有节点，包含新增节点，策略正则表达式为"^",表示匹配所有
  
  ```

  ![image-20210323090646337](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07190304f8f444bd880a9fd11b036fce~tplv-k3u1fbpfcp-zoom-1.image)

### 验证镜像集群

- 在master节点上创建队列

  理论结果：rabbitmq02和rabbitmq03 会自动同步队列

  ![image-20210323090830419](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/634e55d002e04c9b97e0aedf6c7413e0~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210323090853712](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e200c80ae2d34ad5af928ca8e6ceab3e~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210323090918770](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2e774f82d1f4f26bef8b35068715cbd~tplv-k3u1fbpfcp-zoom-1.image)

- 测试数据同步

  ![image-20210323091024286](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d62f187b26647d9b07f7cff38572adb~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210323091125956](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b63b443f404945eb98c16a75546d573d~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210323091208086](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f70403e3aee4dd9b2fad427096c9ff4~tplv-k3u1fbpfcp-zoom-1.image)
