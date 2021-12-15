# HBase设计的实践经验

原文地址：掘金：https://juejin.cn/post/6987036728594792484



#### HBASE是什么？

HBase是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。

HBase是Apache的Hadoop项目的子项目。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

#### **（1）数据模型**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/deddef94f9fc4b6795d48ea13c9f251c~tplv-k3u1fbpfcp-watermark.image)

这就是一张表，我们可以根据行键（roykey）获取一列族数据或多列族数据，每一个列族下面有不限量的列，每一个列上可以存储数据，每一个数据都是有版本的，可以通过时间戳来区别。所以我们在一张表中，知道行键，列族，列，版本时间戳可以确定一个唯一的值

#### **（2）逻辑架构**

任何一张表，他的rowkey是全局有序的，由于对物理存储上的考虑，我们把它放在多个机器上，我们按照大小或者其他策略，分为多个region。每一个region负责表示一份数据，region会在物理机器上，保证是一个均衡的状态。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07552d8b016747acb71bd784e6e1216e~tplv-k3u1fbpfcp-watermark.image)

#### **（3）系统架构**

首先它也是一套标准的存储架构。他的**Hmaster**主要负责简单的协调服务，比如**region的转移，均衡，以及错误的恢复**，实际上他并不参与查询，真正的**查询是发生在region server**。region server事负责存储的，刚才我们说过，每一个表会分为几个region。然后存储在region server。

这里最重要的部分是hlog。为了保证数据一致性，首先会写一份日志文件，这是数据库系统里面以来的一种特性，创建了日志以后，我们才能写入成功。我们刚才提到HBase里面有很多column-family列族，没个列族在一个region里对应一个store，store分别包含storefile和menstore。

为了后续对HBase可以优化，我们首先考虑把**文件写入menstore**里面，随着menstore里面的数据满了之后，会把数据**分发到磁盘里**，然后storefile和memstore整体的话，依赖一个数据模型，叫做**lmstree**。

然后，数据是采用append方式写入的，无论是插入，修改，删除。实际上都是不断的append的。比如说你的更新，删除的操作方式，都是以**打标记方式写入**，所以它**避免了磁盘的随机io**，提高了写入性能，当然的话，它的底层的话是建立在hdfs之上。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d13dbe1cbba94eef91966b751ffebc1b~tplv-k3u1fbpfcp-watermark.image)

HBase 使用 Zookeeper 做分布式管理服务，来维护集群中所有服务的状态。Zookeeper 维护了哪些 servers 是健康可用的，并且在 server 故障时做出通知。Zookeeper 使用一致性协议来保证分布式状态的一致性。注意这需要三台或者五台机器来做一致性协议。

zk的分布式协议还是必须要掌握的，毕竟大数据中的香饽饽flink,hbase这些都是是用zk来做分布式协议的。

#### **（4）怎么读？**

1. 定位，从 Meta table 获取 rowkey 属于哪个 Region Server 管理
2. 相应的 Region Server 读写数据
3. Meta table，保存了系统中所有的 region 列表，结构如下

- Key：table, region start key, region id
- Value：region server

#### **（5）Region Server 是什么，存的什么？**

Region Server 运行在 HDFS DataNode 上，由以下组件组成：

- WAL：Write Ahead Log （**写前日志**）是分布式文件系统上的一个文件，用于存储新的还未被持久化存储的数据，它被用来做故障恢复。
- BlockCache：这是**读缓存**，在内存中存储了最常访问的数据，是 LRU（Least Recently Used）缓存。
- MemStore：这是**写缓存**，在内存中存储了新的还未被持久化到硬盘的数据。当被写入硬盘时，数据会首先被排序。注意每个 Region 的每个 Column Family 都会有一个 MemStore。
- HFile 在硬盘上（HDFS）存储 **HBase 数据**，以有序 KeyValue 的形式。

#### **（6）怎么写数据？**

1. 首先是将数据写入到 WAL 中(WAL 是在文件尾部追加,性能高)
2. 加入到 MemStore 即写缓存, 服务端就可以向客户端返回 ack 表示写数据完成

#### **（7）MemStore 即写缓存是个什么东西？**

- 缓存 HBase 的数据，这和 HFile 中的存储形式一样
- 更新都以 Column Family 为单位进行排序

#### **（8）缓存写完了怎么刷盘呢，总要写到磁盘上去吧？**

1. MemStore 中累积了足够多
2. 整个有序数据集就会被写入一个新的 HFile 文件到 HDFS 上（顺序写）
3. 这也是为什么 HBase 要限制 Column Family 数量的一个原因(列族不能太多）
4. 维护一个最大序列号，这样就知道哪些数据被持久化了

#### **（9）HFILE是什么鬼？**

HFile 使用多层索引来查询数据而不必读取整个文件，这种多层索引类似于一个 B+ tree：

- KeyValues 有序存储。
- rowkey 指向 index，而 index 则指向了具体的 data block，以 64 KB 为单位。
- 每个 block 都有它的叶索引。
- 每个 block 的最后一个 key 都被存储在中间层索引。
- 索引根节点指向中间层索引。

**trailer** 指向原信息数据块，它是在数据持久化为 HFile 时被写在 **HFile 文件尾部**。trailer 还包含例如**布隆过滤器和时间范围**等信息。

布隆过滤器用来**跳过那些不包含指定 rowkey 的文件**，**时间范围信息则是根据时间来过滤**，跳过那些不在请求的时间范围之内的文件。

刚才讨论的索引，在 HFile 被打开时会被载入内存，这样数据查询只要一次硬盘查询。

### 二、读放大、写放大、故障恢复

#### **（10）什么时候会触发读合并？**

上一篇说了，当我们读取数据时，首先是定位，从 Meta table 获取 rowkey 属于哪个 Region Server 管理，而Region Server又有读缓存、写缓存、HFILE

因此我们读一个数据，它有可能在读缓存（LRU），也有可能刚刚才写入还在写缓存，或者都不在，则HBase 会使用 Block Cache 中的索引和布隆过滤器来加载对应的 HFile 到内存，因此数据可能来自读缓存、scanner 读取写缓存和HFILE，这就叫左HBASE的**读合并**

之前说过写缓存可能存在多个HFILE,因此一个读请求可能会读多个文件，影响性能，这也被称为**读放大**（read amplification）。

#### （11）既然存在读放大，那么有没有少去读多个文件的办法？ --写合并

简单的说就是，HBase 会自动合并一些小的 HFile，重写成少量更大的 HFiles

它使用归并排序算法，将小文件合并成大文件，有效减少 HFile 的数量

这个过程被称为 写合并（minor compaction）

#### （12）写合并针对的是哪些HFILE?

1.它合并重写每个 列族（Column Family） 下的所有的 HFiles

2.在这个过程中，被删除的和过期的 cell 会被真正从物理上删除，这能提高读的性能

3.但是因为 major compaction 会重写所有的 HFile，**会产生大量的硬盘 I/O 和网络开销**。这被称为**写放大**（Write Amplification）。

4.HBASE默认是自动调度，因为存在写放大，建议在凌晨或周末进行

5.Major compaction 还能将因为服务器 crash 或者负载均衡导致的数据迁移重新移回到离 Region Server 的地方，这样就能恢复本地数据（ data locality）。

#### （13）不是说Region Server管理多个Region，到底管理几个呢，Region什么时候会扩容？ --Region分裂

我们再来回顾一下 region 的概念：

- HBase Table 被水平切分成一个或数个 regions。每个 region 包含了连续的，有序的一段 rows，以 start key 和 end key 为边界。
- 每个 region 的默认大小为 1GB。
- region 里的数据由 Region Server 负责读写，和 client 交互。
- 每个 Region Server 可以管理约 1000 个 regions（它们可能来自一张表或者多张表）。

一开始每个 table 默认**只有一个 region**。当一个 region 逐渐变得很大时，它会**分裂（split）成两个子 region**，每个子 region 都包含了原来 region 一半的数据，这两个子 region 并行地在原来这个 region server 上创建，这个分裂动作会被报告给 HMaster。处于负载均衡的目的，HMaster 可能会将新的 region 迁移给其它 region server。

#### （14）因为分裂了，为了负载均衡可能在多个region server，造成了读放大，直到写合并的到来，重新迁移或合并到离 region server 节点附近的地方

#### （15）HBASE的数据怎么备份的？

所有的读写都发生在 HDFS 的主 DataNode 节点上。HDFS 会自动备份 WAL（写前日志） 和 HFile 的文件 blocks。HBase 依赖于 HDFS 来保证数据完整安全。当数据被写入 HDFS 时，一份会写入本地节点，另外两个备份会被写入其它节点。

WAL 和 HFiles 都会持久化到硬盘并备份。那么 HBase 是怎么恢复 MemStore 中还未被持久化到 HFile 的数据呢？

#### （16）HBASE的数据怎么宕机恢复的？

1. 当某个 Region Server 发生 crash 时，它所管理的 region 就无法被访问了，直到 crash 被检测到，然后故障恢复完成，这些 region 才能恢复访问。Zookeeper 依靠心跳检测发现节点故障，然后 HMaster 会收到 region server 故障的通知。
2. 当 HMaster 发现某个 region server 故障，HMaster 会将这个 region server 所管理的 regions 分配给其它健康的 region servers。为了恢复故障的 region server 的 MemStore 中还未被持久化到 HFile 的数据，HMaster 会**将 WAL 分割成几个文**件，将它们保存在**新的 region server** 上。每个 region server 然后回放各自拿到的 WAL 碎片中的数据，来为它所分配到的新 region 建立 MemStore。
3. WAL 包含了一系列的修改操作，每个修改都表示一个 **put 或者 delete** 操作。这些修改按照时间顺序依次写入，持久化时它们被依次写入 WAL 文件的尾部。
4. 当数据仍然在 MemStore 还未被持久化到 HFile 怎么办呢？WAL 文件会被回放。操作的方法是读取 WAL 文件，排序并添加所有的修改记录到 MemStore，最后 MemStore 会被刷写到 HFile。

HBASE基本讲解就到此结束了，开始讲解实战演练吧！

------

#### 三、实战经验

#### 怎么设计rowkey

加班回来才能开始写文章，好了，开始了。

在告警业务场景中，一般分为两类场景

1. 瞬时事件类型 --通常开始就结束。
2. 持续事件类型 --通常开始一段时间在结束。

针对这两种情况，我们可以涉及rowkey为 ： 唯一标识id + 时间 + 告警类型

我们对id做一个md5,做一个哈希，这样可以保证数据的分配均衡.

#### **指标平台**

第二个场景叫做指标平台，我们使用kylin做了一层封装，在这上面可以选择我们在HBase存储好的数据，可以选择哪些维度，去查询哪些指标。比如这个成交数据，可以选择时间，城市。就会形成一张图，进而创建一张报表。然后这张报表可以分享给其他人使用。

为什么会选择Kylin呢，因为Kylin是一个**molap引擎**，他是一个运算模型，他满足我们的需求，对页面的相应的话，需要亚秒级的响应。

第二，他对并发有一定的要求，原始的数据达到了百亿的规模。另外需要具有一定的灵活性，最好有sql接口，以离线为主。综合考虑，我们使用的是Kylin。

#### **Kylin简介**

Kylin给他家简单介绍一下，Apache Kylin™是一个开源的分布式分析引擎，提供Hadoop之上的SQL查询接口及多维分析（OLAP）能力以支持超大规模数据，最初由eBay Inc.开发并贡献至开源社区。它能在亚秒内查询巨大的Hive表。他的原理比较简单，基于一个并运算模型，我预先知道，我要从那几个维度去查询某个指标。在预定好的指标和维度的情况下去把所有的情况遍历一遍。利用molap把所有的结果都算出来，在存储到HBase中。然后根据sql查询的维度和指标直接到HBase中扫描就行了。为什么能够实现亚秒级的查询，这就依赖于HBase的计算。

#### **kylin架构**

和刚才讲的逻辑是一致的，左边是数据仓库。所有的数据都在数据仓库中存储。中间是计算引擎，把每天的调度做好，转化为HBase的KY结构存储在HBase中，对外提供sql接口，提供路由功能，解析sql语句，转化为具体的HBase命令。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06de3809621a4864a38fe94b5dbb0c3a~tplv-k3u1fbpfcp-watermark.image)

Kylin中有一个概念叫Cube和Cubold，其实这个逻辑也非常简单，比如已经知道查询的维度有A，b,c,d四个。那abcd查询的时候，可以取也可以不取。一共有16种组合，整体叫做，cube。其中每一种组合叫做Cuboid。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa6f0195108240678dca98651d27f0bc~tplv-k3u1fbpfcp-watermark.image)

#### Kylin如何在Hbase中进行物理存储的 --这里引用的是贝壳的用例

首先定义一张原始表，有两个维度，year和city。

在定义一个指标，比如总价。下面是所有的组合，刚才说到Kylin里面有很多cuboid组合，比如说前面三行有一个cuboid：00000011组合，他们在HBase中的RowKey就是cuboid加上各维度的取值。

这里面会做一点小技巧，对维度的值做一些编码，如果把程式的原始值放到Rowkey里，会比较长。

Rowkey也会存在一个sell里任何一个版本的值都会存在里面。如果rowkey变的非常长，对HBase的压力会非常大，所有通过一个字典编码去减少长度，通过这种方法，就可以把kylin中的计算数据存储到HBase中。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd255398523a40c4bea97ac1d1cc0cba~tplv-k3u1fbpfcp-watermark.image)

用比特位来做一些特征，多个维度统计分析，速度是比较快的，也是大数据分析常用的手段！！！

**当然我们在实际中也配合apache Phoenix提供查询功能**

### 四、优化经验

我先给出一个图

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef98abaf988c4f1e93e1ef555207b717~tplv-k3u1fbpfcp-watermark.image)

#### 1. scan缓存是否设置合理？

优化原理：在解释这个问题之前，首先需要解释什么是scan缓存，通常来讲一次scan会返回大量数据，因此客户端发起一次scan请求，实际并不会一次就将所有数据加载到本地，而是分成多次RPC请求进行加载，这样设计一方面是因为大量数据请求可能会导致网络带宽严重消耗进而影响其他业务，另一方面也有可能因为数据量太大导致本地客户端发生OOM。在这样的设计体系下用户会首先加载一部分数据到本地，然后遍历处理，再加载下一部分数据到本地处理，如此往复，直至所有数据都加载完成。数据加载到本地就存放在scan缓存中，默认100条数据大小。通常情况下，默认的scan缓存设置就可以正常工作的。但是在一些大scan（一次scan可能需要查询几万甚至几十万行数据）来说，每次请求100条数据意味着一次scan需要几百甚至几千次RPC请求，这种交互的代价无疑是很大的。因此可以考虑将scan缓存设置增大，比如设为500或者1000就可能更加合适。笔者之前做过一次试验，在一次scan扫描10w+条数据量的条件下，将scan缓存从100增加到1000，可以有效降低scan请求的总体延迟，延迟基本降低了25%左右。

**优化建议：大scan场景下将scan缓存从100增大到500或者1000，用以减少RPC次数**

#### 2. get请求是否可以使用批量请求？

优化原理：HBase分别提供了单条get以及批量get的API接口，使用批量get接口可以减少客户端到RegionServer之间的RPC连接数，提高读取性能。另外需要注意的是，批量get请求要么成功返回所有请求数据，要么抛出异常。

**优化建议：使用批量get进行读取请求**

### 3. 请求是否可以显示指定列族或者列？

优化原理：HBase是典型的列族数据库，意味着同一列族的数据存储在一起，不同列族的数据分开存储在不同的目录下。如果一个表有多个列族，只是根据Rowkey而不指定列族进行检索的话不同列族的数据需要独立进行检索，性能必然会比指定列族的查询差很多，很多情况下甚至会有2倍～3倍的性能损失。

**优化建议：可以指定列族或者列进行精确查找的尽量指定查找**

#### 4. 离线批量读取请求是否设置禁止缓存？

优化原理：通常离线批量读取数据会进行一次性全表扫描，一方面数据量很大，另一方面请求只会执行一次。这种场景下如果使用scan默认设置，就会将数据从HDFS加载出来之后放到缓存。可想而知，大量数据进入缓存必将其他实时业务热点数据挤出，其他业务不得不从HDFS加载，进而会造成明显的读延迟毛刺

**优化建议：离线批量读取请求设置禁用缓存，scan.setBlockCache(false)**

#### 5. RowKey必须进行散列化处理（比如MD5散列），同时建表必须进行预分区处理

#### 6.JVM内存配置量 < 20G，BlockCache策略选择LRUBlockCache；否则选择BucketCache策略的offheap模式；期待HBase 2.0的到来！

#### 7. 观察RegionServer级别以及Region级别的storefile数，确认HFile文件是否过多

优化建议：

hbase.hstore.compactionThreshold设置不能太大，默认是3个；设置需要根据Region大小确定，通常可以简单的认为hbase.hstore.compaction.max.size = RegionSize / hbase.hstore.compactionThreshold

#### 8. Compaction是否消耗系统资源过多？

（1）Minor Compaction设置：hbase.hstore.compactionThreshold设置不能太小，又不能设置太大，因此建议设置为5～6；hbase.hstore.compaction.max.size = RegionSize / hbase.hstore.compactionThreshold（2）Major Compaction设置：大Region读延迟敏感业务（ 100G以上）通常不建议开启自动Major Compaction，手动低峰期触发。小Region或者延迟不敏感业务可以开启Major Compaction，但建议限制流量；（3）期待更多的优秀Compaction策略，类似于stripe-compaction尽早提供稳定服务

#### 9.Bloomfilter是否设置？是否设置合理？

任何业务都应该设置Bloomfilter，通常设置为row就可以，除非确认业务随机查询类型为row+cf，可以设置为rowcol

BloomFilter是启用在每个ColumnFamily上的，我们可以使用JavaAPI

```
//我们可以通过ColumnDescriptor来指定开启的BloomFilter的类型
HColumnDescriptor.setBloomFilterType() //可选NONE、ROW、ROWCOL
复制代码
```

我们还可以在创建Table的时候指定BloomFilter

```
hbase> create 'mytable',{NAME => 'colfam1', BLOOMFILTER => 'ROWCOL'}
复制代码
```

#### 10.开启Short Circuit Local Read功能，具体配置戳这里[官网](https://link.juejin.cn/?target=https%3A%2F%2Fhadoop.apache.org%2Fdocs%2Fr2.7.2%2Fhadoop-project-dist%2Fhadoop-hdfs%2FShortCircuitLocalReads.html)

优化原理：当前HDFS读取数据都需要经过DataNode，客户端会向DataNode发送读取数据的请求，DataNode接受到请求之后从硬盘中将文件读出来，再通过TPC发送给客户端。Short Circuit策略允许客户端绕过DataNode直接读取本地数据。

#### 11. Hedged Read功能是否开启？

优化原理：HBase数据在HDFS中一般都会存储三份，而且优先会通过Short-Circuit Local Read功能尝试本地读。但是在某些特殊情况下，有可能会出现因为磁盘问题或者网络问题引起的短时间本地读取失败，为了应对这类问题，社区开发者提出了补偿重试机制 – Hedged Read。该机制基本工作原理为：客户端发起一个本地读，一旦一段时间之后还没有返回，客户端将会向其他DataNode发送相同数据的请求。哪一个请求先返回，另一个就会被丢弃。 优化建议：开启Hedged Read功能，具体配置参考这里[官网](https://link.juejin.cn/?target=https%3A%2F%2Fissues.apache.org%2Fjira%2Fbrowse%2FHDFS-5776)

#### 12. 数据本地率是否太低？

数据本地率：HDFS数据通常存储三份，假如当前RegionA处于Node1上，数据a写入的时候三副本为(Node1,Node2,Node3)，数据b写入三副本是(Node1,Node4,Node5)，数据c写入三副本(Node1,Node3,Node5)，可以看出来所有数据写入本地Node1肯定会写一份，数据都在本地可以读到，因此数据本地率是100%。现在假设RegionA被迁移到了Node2上，只有数据a在该节点上，其他数据（b和c）读取只能远程跨节点读，本地率就为33%（假设a，b和c的数据大小相同）。优化原理：数据本地率太低很显然会产生大量的跨网络IO请求，必然会导致读请求延迟较高，因此提高数据本地率可以有效优化随机读性能。数据本地率低的原因一般是因为Region迁移（自动balance开启、RegionServer宕机迁移、手动迁移等）,因此一方面可以通过避免Region无故迁移来保持数据本地率，另一方面如果数据本地率很低，也可以通过执行major_compact提升数据本地率到100%。优化建议：避免Region无故迁移，比如关闭自动balance、RS宕机及时拉起并迁回飘走的Region等；在业务低峰期执行major_compact提升数据本地率

#### 13. 线上bug, hbase暂时不提供服务了？

集群规模10~20台，每秒读写数据量在几十万条记录的量级

客户端与HBase集群进行RPC操作时会抛出NotServingRegionException异常，结果就导致了读写操作失败。大量的写操作被阻塞，写入了文件，系统也发出了警报！

**问题排查**

通过查看HBase Master运行日志，结合客户端抛出异常的时刻，发现当时HBase集群内正在进行Region的Split和不同机器之间的Region Balance

1）由于表中rowkey有时间字段，因此每天都需要新创建Region，同时由于写入数据量大，进一步触发了HBase的Region Split操作，这一过程一般耗时较长（测试时从线上日志来看，平均为10秒左右，Region大小为4GB），且Region Split操作触发较为频繁；

2）同时由于Region Split操作导致Region分布不均匀，进而触发HBase自动做Region Balance操作，Region迁移过程中也会导致Region下线，这一过程耗时较长（测试时从线上日志来看，平均为20秒左右）。

**解决：**  

1）对于写端，可以将未写入成功的记录，添加到一个客户端缓存中，隔一段时间后交给一个后台线程统一重新提交一次；也可以通过setAutoFlush(flase, false)保证提交失败的记录不被抛弃，留在客户端writeBuffer中等待下次writeBuffer满了后再次尝试提交，直到提交成功为止。

2）对于读端，捕获异常后，可以采取休眠一段时间后进行重试等方式。

3）将timestamp字段改成一个周期循环的timestamp，如取timestamp % TS_MODE后的值，其中TS_MODE须大于等于表的TTL时间周期，这样才能保证数据不会被覆盖掉。经过这样改造后，即可实现Region的复用，避免Region的无限上涨。对于读写端的变更也较小，读写端操作时只需将timestamp字段取模后作为rowkey进行读写，另外，读端需要考虑能适应scan扫描时处理[startTsMode, endTsMode]和[endTsMode, startTsMode]两种情况

参考文章：

[blog.csdn.net/shufangreal…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fshufangreal%2Farticle%2Fdetails%2F110539611%3Futm_medium%3Ddistribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.control%26depth_1-utm_source%3Ddistribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.control)

