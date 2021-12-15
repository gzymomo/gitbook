- [【TiDB】——框架理解](https://blog.csdn.net/yye894817571/article/details/89394355)

# 整体框架

TiDB主要分为3个核心组件：TiDB Server ,PD Server 和TiKV Server，还有用于解决用户复杂OLAP需求的TiSpark组件。部署一个单机版的TiDB，这三个组件都需要启动。如果用生产环境，需要使用Ansible部署TiDB集群。

一个完整的TiDB集群框架如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190418201855148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwNDQwMjk=,size_16,color_FFFFFF,t_70)

## TiKV Server

TiKV Server 负责存储数据，对于数据存储需要保证实现以下功能：

- 支持跨数据中心的容灾；

- 写入速度足够快；

- 读取速度方便；

- 支持数据修改与并发修改数据；

- 多条记录修改后保证原子性。

TiKV采用Key-Value模型存储数据，并且提供有序遍历方法。TiKV是一个巨大的Map，TiKV存储的是key-value pair，key-value pair按照key的二进制顺序有序，查找到某个key的位置，可以不断地调用Next方法以递增的顺序获取比这个key大的key-value。

TiKV的存储模型与SQL中Table无关，TiKV就是一个高性能高可靠性的巨大的（分布式）的map。

TiKV通过RocksDB将数据持久化到磁盘上，而不是直接向磁盘上写数据，也就是说具体的数据落地是用RocksDB负责。RokcsDB 是一个高性能的单机引擎，有FaceBook的团队做持续优化。

如果要做到数据不丢失，支持跨数据中心的容灾，就需要将数据负责到多台机器上，但是这个时候就涉及到数据一致性的问题了。TiDB采用Raft协议来保证数据一致性，Raft是一个一致性算法，PingCAP公司对Raft协议的实现做了大量的优化来保证这一协议切实可行。

Raft是一个一致性协议，提供了以下几个重要的功能：

- Leader选举；
- 成员变更；

- 日志复制；



TiKV利用Raft来做数据复制，每个数据变更都会落地为一条Raft日志，通过Raft的日志复制功能，将数据安全可靠地同步到Group的多数节点中，以防单机失效。数据的写入是通过Raft这一层的接口写入，而不是直接写RocksDB。通过Raft实现，我们拥有一个分布式的巨大Map，也就不用担心某台机器挂掉。

下图为数据的存储流程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019041820575172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwNDQwMjk=,size_16,color_FFFFFF,t_70)



经过前面的理解，可以将TiKV看作是一个kv系统，TiKV是以Region为单位做存储与复制，将key-value分段存储在节点上，每一段是一系列连续的key,也就是分Range，每一段就是一个Region。每个Region中存储的数据不超过一定的大小（默认是64mb),每一个Region都可以用StartKey到EndKey这样一个左闭右开区间来描述。

系统会通过一个组件来负责将Region尽可能均匀的散步在集群中所有的节点上，这样一方面实现了存储容量的水平扩展，另一方面也实现了负载均衡。为了保证上层客户端能够访问所需要的数据，系统会有一个组件记录Region在节点上面的分布情况，可以通过任意一个key就能查询到这个key在哪个Region中，以及这个Region在哪个节点上。

TiKV以Region为单位做数据的复制，也就是一个Region的数据会保存多个副本，每个副本叫做一个Replica.Replica之间是通过Raft来保证数据的一致性，一个Region的多个Replica会保存在不同的节点上，构成一个Raft Group。其中Replica会作为这个Group的leader，其他的Replica作为Follower。所有的读和写都是通过Leader进行，在由leader复制给Follower。

如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019041821175420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwNDQwMjk=,size_16,color_FFFFFF,t_70)

小结：TiKV是一个分布式key-value存储系统，一个巨大的分布式Map系统，一个全局有序的分布式key-value引擎。

## PD Server

Placement Driver（简称PD）是TiDB里面全局中心总控节点，是整个集群的管理模块，负责整个集群的调度。

TiDB作为一个分布式高可用存储系统，系统需要具备多副本容错，动态扩容、缩容，容忍节点掉线以及自动错误恢复的功能，且整个系统负载均与，方便管理。需要满足这些功能，TiDB就需要收集足够的信息，比如每个节点的状态、每个Raft Group的信息，业务访问操作的统计等。PD根据这些信息以及调度的策略，置顶出了尽量满足这些需求的调度计划，并提供基本操作来完成这个计划。

### 信息收集

调度依赖于这个集群信息的收集，PD需要知道每个TiKV节点的状态以及每个Region的状态。TiKV集群会向PD汇报两类信息。

#### 一、每个TiKV节点会定期向PD汇报节点的整体信息。

TiKV节点（store)与PD之间存在心跳包，一方面PD通过心跳包检测每个Store是否存活，以及是否有新加入的Store；另一方面也会携带这个Store的状态信息，主要包括：

- 总磁盘容量
- 可用磁盘容量
- 承载的Region数量
- 数据写入速度
- 发送/接受的Snapshot数量（Replica之间可能会通过Snapshot同步数据）
- 是否过载
- 标签信息（标签是具备层级关系的一系列Tag）

#### 二、每个Raft Group的Leader会定期向PD汇报信息

每个Raft Group的Leader和PD之间存在心跳包，用于汇报这个Region的状态，主要包括下面几点信息：

- leader的位置

- Followers的位置

- 掉线Replica的个数

- 数据写入/读取的速度

PD不断的通过这两类心跳消息收集整个集群的信息，再以这些信息座位决策的依据。除此之外，PD还可以通过管理接口接受额外的信息，用来做更准确的决策。比如当某个Store的心跳包中断的时候，PD并不能判断这个节点是临时失效还是永久失效，只能经过一段时间的等待（默认是30分钟），如果一直没有心跳包，就认为是Store已经下线，再决定需要将这个Store上面的Region都调度走。但是有的时候，是运维人员主动将某台机器下线，这个时候，可以通过PD的管理接口通知PD改Store不可用，PD就可以马上判断判断需要将这个Store上面的Region都调度走。

## 调度的策略

PD收集了这些信息后，还需要一些策略来制定具体的调度计划。

### 一、一个Region的Replica数量正确

当PD通过某个Region Leader的心跳包发现这个Region的Replica数量不满足要求时，需要通过Add/Remove Replica 操作调整Replica数量。

### 二、一个Raft Group中的多个Replica不在同一个位置

### 三、副本在Store之间的分布均匀分配

每个副本中存储的数据容量上限是固定的，所以维持每个节点上面副本数量的均衡，会使得总体负载更均衡。

### 四、Leader数量在Store之间均匀分配

Raft协议要读取和写入都通过Leader进行，所以计算的负载主要在Leader上面，PD会尽可能讲Leader在节点之间分散。

### 五、访问热点数量在Store之间均匀分配

每个Store以及Region Leader在上报信息是携带了当前访问负载的信息，比如Key的读取/写入速度。PD会检测出访问热点，且将其在节点之间分散。

### 六、各个Store的存储空间占用大致相等

每个Store启动的时候都会指定一个Capacity参数，表明这个Store的存储空间上限，PD在做调度的时候，会考虑节点的存储空间剩余量。

### 七、控制调度速度，避免影响在线服务

调度操作需要耗费CPU、内存、磁盘IO以及网络带宽，我们需要避免对线上服务造成太大影响。PD会对当前正在进行的操作数量进行控制，默认的速度控制是比较保守的，如果希望加快调度（比如已经停服务升级，增加新节点，希望尽快调度），那么可以通过pd-ctl手动加快调度速度。

### 八、支持手动下线节点

当通过pd-ctl手动下线节点后，PD会在一定速率控制下，将节点上的数据调度走。当调度完成后，就会将这个节点置为下线状态。



小结：作为中心中控节点，PD通过集成etcd，自动得支持auto failover，无须担心单点故障问题。同时PD也通过etcd的raft，保证了数据强一致性，不用担心数据丢失问题。除此之外，PD还负责全局ID的生成，以及全局时间戳TSO的生成，保存整个集群TiKV的元信息，负责给client提供路由功能。

## TiDB Server

TiDB Server负责接收应用成发送过来的SQL请求，处理SQL相关的逻辑，并通过PD找到存储所需数据的TiKV地址，与TiKV交互获取数据，最终返回结果。TiDB Server是无状态的，其本身并不存储数据，只负责计算，可以无限水平扩展，可以通过负载均衡组件（如LVS、HAProxy或F5）对外提供统一得结束地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190419094342445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTMwNDQwMjk=,size_16,color_FFFFFF,t_70)

TiDB本身并不存储数据，节点之间完全对等，TiDB Server这一层最重要的工作是处理用户请求，执行SQL运算逻辑。

因为TiKV是一个key-value的存储引擎，需要做到SQL到kv的映射，这里可以去具体了解它的映射方案。

用户的SQL请求会直接或者通过Load Balancer 发送到TiDB-Server，TiDB会解析MySQLProtocol Packet，获取请求内容，然后做语法分析、查询计划指定和优化、执行查询计划获取和处理数据。数据全部存储在TiKV集群中，这个过程中TiDB-server会和TiKV-Server交互，获取数据，最后TiDB-Server需要将查询结果返回给用户。

## TiSpark

TiSpark就是Spark SQL on TiKV，是解决用户复杂OLAP需求的主要组件,将Spark SQL 直接运行在TiDB存储层上，同时融合TiKV 分布集群的优势，和 TiDB 一起为用户一站式解决 HTAP （Hybrid Transactional/Analytical Processing）需求。TiSPark依赖于TiKV集群和PD的存在。如果需要用到TiSPark，也需要搭建一个Spark集群。由于目前项目中没有用到TiSPark，在这里就不深入研究。

# 总结

TiKV Server负责存储，PD Server 负责调度，TiDB Server负责计算，三者中间有个至关重要的协议Raft，这个协议保证了TiDB这个分布式数据库的数据安全一致。
