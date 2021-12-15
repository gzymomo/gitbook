# Hbase在用户画像上的应用

来源：微信公众号：浪尖聊大数据

## 背景

公司要做C端的用户画像，方便运营人员根据标签做用户圈选，然后对这部分人群做精准广告投放，所以需要线上接口实时调用库中数据，并快速返回结果，并将结果反馈到推送平台进行推送。

目前公司的方案是全部走ES。每天将Hive统计的各个维度的用户标签结果数据从Hive推送到ES，然后ES进行筛选，推送。这个流程虽然速度快，但是不可避免的遇到一个问题，后续数据量会越来越大，如果跨多个维度的用户圈选，可能会导致ES处理不过来，所以准备采取第二套方案，在组合标签查询对应的用户人群场景中，首先通过组合标签的条件在Elasticsearch中查询对应的索引数据，然后通过索引数据去HBase中批量获取rowkey对应的数据。

优化后方案流程图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUOLvaapXicWDiciaxUBGicCiaBlaE79nfVJbZc8x7DQSkAPLFwwPgUNmJ8DQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 介绍

Hbase的简介

HBase是一个高性能、列存储、可伸缩、实时读写的分布式存储系统，同样运行在HDFS之上。与Hive不同的是，HBase能够在数据库上实时运行，而不是跑MapReduce任务，适合进行大数据的实时查询。

> Hadoop本质上是：分布式文件系统(HDFS) + 分布式计算框架(Mapreduce) + 调度系统Yarn搭建起来的分布式大数据处理框架。
> Hive：是一个基于Hadoop的数据仓库，适用于一些高延迟性的应用（离线开发），可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能。
> Hive可以认为是MapReduce的一个包装，把好写的HQL转换为的MapReduce程序，本身不存储和计算数据，它完全依赖于HDFS和MapReduce，Hive中的表是纯逻辑表。hive需要用到hdfs存储文件，需要用到MapReduce计算框架。
> HBase：是一个Hadoop的数据库，一个分布式、可扩展、大数据的存储。hbase是物理表，不是逻辑表，提供一个超大的内存hash表，搜索引擎通过它来存储索引，方便查询操作。
> HBase可以认为是HDFS的一个包装。他的本质是数据存储，是个NoSql数据库；HBase部署于HDFS之上，并且克服了hdfs在随机读写方面的缺点，提高查询效率。

HMaster

- - 管理HRegionServer,实现负载均衡。
  - 管理和分配HRegion，比如在HRegion split时分配新的HRegion；在HRegionServer下线时迁移其内的HRegion到其他HRegionServer上。
  - 实现DDL操作
  - 管理namespace和table的元数据
  - 权限控制（ACL）

HRegionServer

- - 管理HRegion
  - 读写HDFS，管理Table中的数据。
  - Client直接通过HRegionServer读写数据（从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后）

ZooKeeper

- - 存放整个 HBase集群的元数据以及集群的状态信息。
  - 实现HMaster主从节点的failover。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUxXCR0OVI0LvncSicTLYdS98YxNZhL5b6A5Gbfrv5Um9INia1et6us9wA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Rowkey

用来表示唯一一行记录的主键，HBase的数据是按照row key的字典顺序进行全局排列的。访问HBase中的行只有3种方式

- 通过单个row key访问
- 通过row key的正则访问
- 全表扫描

由于HBase通过rowkey对数据进行检索，而rowkey由于长度限制的因素不能将很多查询条件拼接在rowkey中，因此HBase无法像关系数据库那样根据多种条件对数据进行筛选。一般地，HBase需建立二级索引来满足根据复杂条件查询数据的需求。

Rowkey设计时需要遵循三大原则：

- 唯一性原则：rowkey需要保证唯一性，不存在重复的情况。在画像中一般使用用户id作为rowkey。
- 长度原则：rowkey的长度一般为10-100bytes。
- 散列原则：rowkey的散列分布有利于数据均衡分布在每个RegionServer，可实现负载均衡。

通常来说，RowKey 只能针对条件中含有其首字段的查询给予令人满意的性能支持，在 查询其他字段时，表现就差强人意了，在极端情况下某些字段的查询性能可能会退化为全表  扫描的水平，这是因为字段在 RowKey 中的地位是不等价的，它们在 RowKey 中的排位决  定了它们被检索时的性能表现，排序越靠前的字段在查询中越具有优势，特别是首位字段  具有特别的先发优势，如果查询中包含首位字段，检索时就可以通过首位字段的值确定 RowKey  的前缀部分，从而大幅度地收窄检索区间，如果不包含则只能在全体数据的 RowKey 上逐一查找，由此可以想见两者在性能上的差距。

columns family

列簇，HBase中的每个列都归属于某个列簇，列簇是表的schema的一部分，必须在使用表之前定义。划分columns family的原则如下：

- 是否具有相似的数据格式；
- 是否具有相似的访问类型。

## Hbase伪分布式搭建

1、下载Hbase

https://www.apache.org/dyn/closer.lua/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz

www.apache.org

```
wget https://mirror.bit.edu.cn/apache/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz
```

2、解压

3、修改配置

```
1、设置JAVA_HOME
vim hbase-env.sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home
2、在hbase-site.xml设置HBase的核心配置
vim hbase-site.xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    //Here you have to set the path where you want HBase to store its files.
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
<property>
     <name>hbase.master.info.port</name>
     <value>60010</value>
 </property>
</configuration>
注意：
hbase.rootdir要和hadoop中core-site.xml中fs.default.name相同。
```

3、启动Hbase

必须先启动Hadoop，然后启动Hbase。

```
./start-hbase.sh
```

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUxBGSfpZS8ic4G8qRec6noMoVliaQ33Tia5n9REKMago5lqibicRgeBadBKQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、图形化界面

```
http://localhost:60010/master-status
```

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUjhFQrxicx2yaCZrzEqtah6n8IwtBL2h5NbicBY49K5GnVticiav1rCDm6A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5、hbase语法

```
create
语法: create '<table name>', '<column family>' 
Demo: create 'TEST.USER_INFO', 'INFO'

put
语法:put '<table name>', 'row1','<colfamily:colname>','<value>' 
demo: 
put 'TEST.USER_INFO', 'R1', 'INFO:C1', 'r1_c1_value1' 
put 'TEST.USER_INFO', 'R1', 'INFO:C2', 'r1_c2_value1' 
put 'TEST.USER_INFO', 'R1', 'INFO:C3', 'r1_c3_value1'

scan
语法：scan '<table name>' 
Demo: scan 'TEST.USER_INFO'

get
语法：get '<table name>','row1' 
Demo：get 'TEST.USER_INFO' , 'R1'

count
语法：count '<table name>' 
Demo：count 'TEST.USER_INFO'

delete
语法：delete '<table name>', '<row>', '<column name >', '<time stamp>' 
Demo：delete 'TEST.USER_INFO', 'R1', 'INFO:C1'

deleteall
语法：deleteall '<table name>', '<row>' 
Demo：deleteall 'TEST.USER_INFO', 'R1'

drop、disable
disable 'TEST.USER_INFO' 
drop 'TEST.USER_INFO'
```

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUZaDAPsjGicW3u5URGc12r4sCDQcCbB9uNXHbMdngdZEy6qSQ8sIriaYQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 用户画像使用场景

整体架构图在文章开头已经画了。

用户画像不是产生数据的源头，而是对基于数据仓库ODS层、DW层、DM层中与用户相关数据的二次建模加工。在ETL过程中将用户标签计算结果写入Hive，由于不同数据库有不同的应用场景，后续需要进一步将数据同步到MySQL、HBase、Elasticsearch等数据库中。

- Hive：存储用户标签计算结果、用户人群计算结果、用户特征库计算结果。
- MySQL：存储标签元数据，监控相关数据，导出到业务系统的数据。
- HBase：存储线上接口实时调用类数据，存储用户标签数据。
- Elasticserch：支持海量数据的实时查询分析，用于存储用户人群计算、用户群透视分析所需的用户标签数据

其中最开始的ETL和数据标签体系建立都是ETL工程师和数据分析师的工作。

- 各种源数据、中间层数据、汇总数据都存在Hive中。
- 用户标签源数据存Mysql中。
- 用户标签的结果数据、二级索引映射表存储HBase中。
- ES同时存储二级索引映射表来批量获取索引。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUjJssyM5PbTiaPICicJAFvVhq2T6YCgLR9SlzyYE5VCU90VgZJJFUapTg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

例如：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUCcv8o3W1vyCbOibCwYQ8BIovAt52UmpBtGI1gJxVaJUR9TKCImprtXg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为什么要用二级索引。

> 在线接口在查询HBase中数据时，由于HBase无法像关系数据库那样根据多种条件对数据进行筛选（类似SQL语言中的where筛选条件）。一般地HBase需建立二级索引来满足根据复杂条件查询数据的需求。
> 在组合标签查询对应的用户人群场景中，首先通过组合标签的条件在Elasticsearch中查询对应的索引数据，然后通过索引数据去HBase中批量获取rowkey对应的数据（Elasticsearch中的documentid和HBase中的rowkey都设计为用户id）。

ES做二级索引的两种构建流程：

配套基于Spark/MR的批量索引创建/更新程序， 用于初次或重建已有HBase库表的索引

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/TwK74MzofXfgTib0K454erRx7UKk0ImCUJrFegiayQAkhtW5PC2bpN8Ok3WrLtRAnic7icYxGluajucqx2gfPXibPqg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

备注：

用户画像的基本流程见：[浅谈用户画像](http://mp.weixin.qq.com/s?__biz=MzUyMDA4OTY3MQ==&mid=2247498224&idx=1&sn=1fb68260d1715dddd1de93ea221e23d6&chksm=f9ed00d8ce9a89ce00dd4b4bc489ab0299e9d3b3139dfc702fbb523133d44bb0021bac9a0636&scene=21#wechat_redirect)