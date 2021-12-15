# 【Couchbase的优势】

## 一.web界面

  Couchbase提供了良好的管理界面，集配置，管理，监控和告警于一身。Couchbase的web界面提供了版本提醒，ssl证书配置，用户管理，邮件告警等一系列丰富的功能，大大简化了运维的工作；也web界面可以直观的观测OPS，磁盘写入队列，内存数据量，Compaction和Ejection实时状况，为开发和测试提供了直观的数据参考。对性能测试至关重要。而redis就仅有第三方提供的一些简单客户端产品，用于观测数据存储情况，配置优化相关的工作也需要在配置文件中操作。



如果考虑到后期性能测试以及运维的可操作性，couchbase是更好的选择。



##  二. 三高

 这里的三高指的是高性能，高可用性和高伸缩性。


 Redis从一开始就是单点解决方案，直到Redis3.0后才出来官方的集群方案，而且仍存在一些架构上的问题，其高可用性目前还没有在线上被证明，第三方的集群方案像豌豆荚的Codi又缺少官方的后续支持。


 相比而言，Couchbase从一开始就内建集群系统，即使是节点重启，数据未完全载入内存，也能照常提供服务，这得益于每份数据的metedata，其中包含近期的操作信息，Couchbase借此来区分热数据，只要热数据加载到最低水位即可立即提供服务。


  如果出现节点失效，集群可在指定的时间里自动发现并启动failover，这里不同Redis的哨兵系统，Couchbase采用激活失效节点备份将请求分摊给幸存节点的方案，恢复时间更快；如果节点新增，Couchbase会将内存数据复制，一份用于提供服务，一份用于重分配并时刻保证数据一致性，即集群扩容不会导致任何业务中断。此外，couchbase的异步持久化和备份同步（通过维护一个持久化队列）也要优于redis的RDB快照和AOF日志方案。因此，从三高的角度来看，高可用性和高伸缩性上Couchbase显然是更加可靠的。


 Couchbase的集群方案相比Redis，对用户屏蔽了更多细节，集群更具弹性，且经过多年的生产线上验证。因此，如果高可用和弹性是重要考虑因素，那Couchbase无疑是更稳妥的方案。
 
 两款产品直观的优势如上，大家可以根据本文结合实际业务场景进行选择。有兴趣的同学推荐阅读《Seven Databases in Seven Weeks》。



# 【Redis vs Couchbase内存管理分析】

对于像Redis和Couchbase基于内存的数据库系统来说，内存管理的效率高低是影响系统性能的关键因素。



## Couchbase内存管理分析

 Couchbase默认使用Slab  Allocation机制管理内存，其主要思想是按照预先规定的大小，将分配的内存分割成特定长度的块以存储相应长度的key-value数据记录，以完全解决内存碎片问题。（其实是把外部碎片转化为了内部碎片）Slab Allocation机制只为存储外部数据而设计，也就是说所有的key-value数据都存储在Slab  Allocation系统里，而Couchbase的其它内存请求则通过普通的malloc/free来申请，因为这些请求的数量和频率决定了它们不会对整个系统的性能造成影响。



Slab Allocation的原理相当简单。  如图所示，它首先从操作系统申请一大块内存，并将其分割成各种尺寸的块Chunk，并把尺寸相同的块分成组Slab  Class。其中，Chunk就是用来存储key-value数据的最小单位。每个Slab  Class的大小，可以在Couchbase启动的时候通过制定Growth Factor来控制。假定图中Growth  Factor的取值为1.25，如果第一组Chunk的大小为88个字节，第二组Chunk的大小就为110个字节，依此类推。 
 ![img](https://rdc.hundsun.com/portal/data/upload/201705/f_54188e84323cd85a6a364d5553a53cc7.png)

当Couchbase接收到客户端发送过来的数据时首先会根据收到数据的大小选择一个最合适的Slab Class，然后通过查询Couchbase保存着的该Slab  Class内空闲Chunk的列表就可以找到一个可用于存储数据的Chunk。当一条数据库过期或者丢弃时，该记录所占用的Chunk就可以回收，重新添加到空闲列表中。


  从以上过程我们可以看出Couchbase的内存管理制效率高，而且不会造成内存碎片，但是它最大的缺点就是会导致空间浪费。因为每个Chunk都分配了特定长度的内存空间，所以变长数据无法充分利用这些空间。如图  所示，将100个字节的数据缓存到128个字节的Chunk中，剩余的28个字节就浪费掉了（这就是内部碎片，但相比外部碎片是可控的，也是可再利用的）。
 ![img](https://rdc.hundsun.com/portal/data/upload/201705/f_9879e7b3a8c3f1ff5530fa52828e497b.png)

## Redis内存管理分析

Redis的内存管理主要通过源码中zmalloc.h和zmalloc.c两个文件来实现的。Redis为了方便内存的管理，在分配一块内存之后，会将这块内存的大小存入内存块的头部。



如图所示，real_ptr是Redis调用malloc后返回的指针。Redis将内存块的大小size存入头部，size所占据的内存大小是已知的，为size_t类型的长度，然后返回ret_ptr。当需要释放内存的时候，ret_ptr被传给内存管理程序。通过ret_ptr，程序可以很容易的算出real_ptr的值，然后将real_ptr传给free释放内存。

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_38758be2f5490ceafe0d3583b303aade.png)

Redis通过定义一个数组来记录所有的内存分配情况，这个数组的长度为ZMALLOC_MAX_ALLOC_STAT。数组的每一个元素代表当前程序所分配的内存块的个数，且内存块的大小为该元素的下标。在源码中，这个数组为zmalloc_allocations。zmalloc_allocations[16]代表已经分配的长度为16bytes的内存块的个数。zmalloc.c中有一个静态变量used_memory用来记录当前分配的内存总大小。所以，总的来看，Redis采用的是包装的mallc/free，相较于Couchbase的内存管理方法来说，要简单很多。


  总的来说，Couchbase的内存管理机制以每次分配冗余空间为代价，避免了内存碎片。如果程序需要频繁短时效的百字节以上的内存空间，比如动态令牌，Couchbase显然是更好的选择；如果仅仅使用长效的计数器或几个字节的标识字段，那么使用Couchbase反而造成内存浪费，Redis却是更好的选择。



# 【集群管理】

## 一. Couchbase集群管理

  Couchbase本身并不支持分布式，因此只能在客户端通过像一致性哈希这样的分布式算法来实现Couchbase的分布式存储，Couchbase会通过在集群内部和客户端直接共享vbucket和节点映射关系，客户端每次操作前需要对数据的key进行计算，以确定数据落入的vbucket编号，再根据映射表确定数据所在节点，然后直接与指定节点通信，不需要Redis的节点重定位方案，对于集群变更对外也只需要更新vbucket和节点映射关系即可。下图是Couchbase的分布式存储实现架构。
![img](https://rdc.hundsun.com/portal/data/upload/201705/f_f7a60532c37d0d8bb0ee049a0292529b.png)



现在我们模拟一下Couchbase的失效备援方案，假设当前客户端的vbucket和节点映射关系如下：

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_30ca052492447ff8b2caca0a292c6cf5.png)



那么当D节点失效后，集群只需要激活ABC上D节点的数据副本，然后更新vbucket和节点映射关系如下：

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_75f8a12eb5e4cf54c3a55748386a1dd0.png)

此后所有的数据请求就被分摊到了ABC之上，即使客户端的配置文件里还存在节点D的地址，也不会再产生交互了。
 

##  二. Redis集群管理

  相较于Couchbase只能采用客户端实现分布式存储，Redis更偏向于在服务器端构建分布式存储。最新版本的Redis已经支持了分布式存储功能。Redis Cluster是一个实现了分布式且允许单点故障的Redis高级版本，它没有中心节点，具有线性可伸缩的功能。

下图是Redis Cluster的分布式存储架构：
 ![img](https://rdc.hundsun.com/portal/data/upload/201705/f_3a53d8578f2c4c4311d41a2f9df37e3f.png)

其中节点与节点之间通过二进制协议进行通信，节点与客户端之间通过ascii协议进行通信。在数据的放置策略上，Redis Cluster将整个key的数值域分成4096个哈希槽，每个节点上可以存储一个或多个哈希槽，也就是说当前Redis  Cluster支持的最大节点数就是4096。Redis Cluster使用的分布式算法也很简单：crc16( key ) %  HASH_SLOTS_NUMBER。



为了保证单点故障下的数据可用性，Redis  Cluster引入了Master节点和Slave节点。在Redis  Cluster中，每个Master节点都会有对应的两个用于冗余的Slave节点。这样在整个集群中，任意两个节点的宕机都不会导致数据的不可用。当Master节点退出后，集群会自动选择一个Slave节点成为新的Master节点。

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_a0c4d85018cbd15609a9d1bb288eedc6.png)



总而言之，Couchbase把数据分布计算分摊给客户端执行，节省了缓存服务器的CPU，并且客户端直接和数据所在节点通信节省了带宽并缩短了响应时间。且相比Redis至少需要6个实例才能组成集群的限制，Couchbase的集群方案更加灵活（但Redis可以一机多个实例，Couchbase单机只能部署一个）。



因此，如果机器资源充足，Couchbase可以提供更好的服务体验（由于客户端分摊计算成本，这里的机器不需要过多CPU），如果只有机器资源紧张，redis是部署上的轻量级方案，但前面提到的，当访问量爆发时，可能会考验缓存服务器的CPU。



# 【该怎么选择】

在我看来，Conchbase和Redis本就是定位不同的两款产品。Redis的亮点在于业务契合度高，Couchbase的亮点在于高可用。如果不是二选一的场景，它们是可以相辅相成的，Redis的定位是一个带有丰富数据结构的内存数据库，充分利用其数据结构可以简化许多业务场景下的开发，如利用队列实现消息队列，利用有序集合实现排行榜；而Couchbase的定位是一个专业的分布式缓存系统，将一些业务关键信息，如鉴权信息和会话信息存储其中，能最大限度保证业务的安全性和高可用性。



Redis&Conchbase的缓存系统选型：

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_8349cd6fea5e35ecd84694e1546eb011.png)

![img](https://rdc.hundsun.com/portal/data/upload/201705/f_14caf512cb435680b621ba18c5301fbc.png)

​              