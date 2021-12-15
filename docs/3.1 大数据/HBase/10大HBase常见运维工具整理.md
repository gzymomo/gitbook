## **HBase组件介绍**

HBase作为当前比较热门和广泛使用的NoSQL数据库，由于本身设计架构和流程上比较复杂，对大数据经验较少的运维人员门槛较高，本文对当前HBase上已有的工具做一些介绍以及总结。

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWSibFcUo4EfWLxIMsNCaGUnhbza7pWQBwmUzibKXUBkK4o93JMy5qMu0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**写在前面的说明：**

1） 由于HBase不同版本间的差异性较大（如HBase2.x上移走了hbck工具），本文使用的所有命令行运行的环境为MRS_1.9.3，对应的HBase版本为1.3.1，部分命令在HBase2上不支持（有时间的话会对HBase2做单独的介绍）。

2） 本文所涉及的HBase工具均为开源自带工具，不涉及厂商自研的优化和运维工具。

## **Canary工具**

HBase Canary是检测HBase集群当前状态的工具，用简单的查询来检查HBASE上的region是否可用（可读）。它主要分为两种模式

1） region模式（默认），对每个region下每个CF随机查询一条数据，打印是否成功以及查询时延。



```
#对t1和tsdb-uid表进行检查
hbase org.apache.hadoop.hbase.tool.Canary t1 tsdb-uid
#注意：不指定表时扫所有region
```

2） regionserver模式，对每个regionserver上随机选一个表进行查询，打印是否成功以及查询时延。



```
#对一个regionserver进行检查
hbase org.apache.hadoop.hbase.tool.Canary -regionserver node-ana-coreQZLQ0002.1432edca-3d6f-4e17-ad52-098f2adde2e6.com
#注意：不指定regionserver时扫所有regionserver
```

```
Canary还可以指定一些简单的参数，可以参考如下
```

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTW7vKPqUWAY6LGf38nwgTUJZDv3UCtF1ayE6AQoeLtjJQAZUxVJaeUVQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：2星（只是简单的读操作，region个数极多的时候会占用少部分请求吞吐）
- 实用性：2星



## **HFile工具**

HBase HFile查看工具，主要用来检查当前某个具体的HFile的内容/元数据。当业务上发现某个region无法读取，在regionserver上由于文件问题无法打开region或者读取某个文件出现异常时，可用此工具单独来检查HFile是否有问题

```
#查看t1表下的其中一个HFile的详情，打印KV
hbase org.apache.hadoop.hbase.io.hfile.HFile -v -m -p -f /hbase/data/default/t1/4dfafe12b749999fdc1e3325f22794d0/cf1/06e102be436c449693734b222b9e9aab
```

使用参数如下：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWz3gic4pes07Cu1sYeqhrnnxU2AveufoktKA86qKJoroIwgplNIZdEaQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：1星（此工具不走HBase通道，只是单纯的读取文件，不影响集群）
- 实用性：4星（可精确判断具体的HFile内容是否有问题）

 

## **RowCounter和CellCounter工具**

RowCounter 是用MapReduce任务来计算表行数的一个统计工具。而和 RowCounter类似，但会收集和表相关的更细节的统计数据，包括：表的行数、列族数、qualifier数以及对应出现的次数等。两个工具都可以指定row的起止位置和timestamp来进行范围查询



```
# RowCounter扫描t1
hbase org.apache.hadoop.hbase.mapreduce.RowCounter t1
#用CellCounter扫描t1表并将结果写入HDFS的/tmp/t1.cell目录
hbase org.apache.hadoop.hbase.mapreduce.CellCounter t1 /tmp/t1.cell
```

使用参数如下：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWFLj4elic3xsibT9rQfEFPjUKAnWyotylFUsOrliaoWJ9u3FhQreGengyA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTW1750K5ic8ywpuX5zU67SFBt6SZCRfp7JEYq3vf5yGdksFel6ebrjCaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：3星（需要起MapReduce对表所有region进行scan，占用集群资源）
- 实用性：3星（HBase统计自身表行数的唯一工具， hbase shell中count效率比较低）

## **Clean工具**

clean命令是用来清除HBase在ZooKeeper合HDFS上数据的工具。当集群想清理或铲除所有数据时，可以让HBase恢复到最初的状态。

```
#清除HBase下的所有数据hbase clean --cleanAll使用参数如下：
```

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

总结：

- 对集群影响：**5星**（删除HBase集群上所有数据）
- 实用性：2星（除开需要重新设置HBase数据的场景如要切换到HBase on OBS，平时很少会用到）



## **HBCK工具**

HBase的hbck工具是日常运维过程中使用最多的工具，它可以检查集群上region的一致性。由于HBase的RIT状态较复杂也最容易出现问题，日常运维过程中经常会遇到region不在线/不一致等问题，此时就可以根据hbck不同的检查结果使用相应的命令进行修复。



```
#检查t1表的region状态
hbase hbck t1
#修复t1表的meta并重新assign分配
hbase hbck -fixMeta -fixAssignments t1
```

由于该工具使用的场景太多太细，此处就不作展开介绍了，可以查看参数的描述来对各种异常场景进行修复。注意：在不清楚异常原因的情况下，千万不要乱使用修复命令病急乱投医，很有可能会使问题本身更糟糕。

使用参数如下：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWQJ5pu2suwlLy0pT7kYWhnQDeeLxKV6vjuLQR5zRlGnOMgQib76ibRPIw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：4星（个别meta相关命令对集群影响极大）
- 实用性：**5星**（hbck是HBase运维人员的最基本运维工具）



## **RegionSplitter工具**

RegionSplitter是HBase的Pre-splitting工具，在table初始化的时候如果不配置pre-split的话，HBase不知道如何去split region，这就很大可能会造成后续的region/regionserver的热点，最好的办法就是首先预测split的切分点，在建表的时候做pre-splitting，保证一开始的业务访问总体负载均衡。RegionSplitter能够通过具体的split算法在建表的时候进行pre-split，自带了两种算法：

- **HexStringSplit**

使用8个16进制字符来进行split，适合row key是十六进制的字符串(ASCII)作为前缀的时候

- **UniformSplit**

使用一个长度为8的byte数组进行split，按照原始byte值（从0x00~0xFF）右边以00填充。以这种方式分区的表在Put数据的时候需要对rowkey做一定的修饰， 比如原来的rowkey为rawStr，则需要对其取hashCode，然后进行按照byte位反转后放在最初rowkey串的前面



```
#创建test_table表，并使用HexStringSplit算法预分区10个
hbase org.apache.hadoop.hbase.util.RegionSplitter test_table HexStringSplit -c 10 -f f1
#Tips：此操作等价于在hbase shell中create ' test_table ', { NAME => 'f1'},{NUMREGIONS => 10, SPLITALGO => 'HexStringSplit'}
```

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTW5eVtYHr9B9A4JtHP06quYOicw1zoyeiatzcwdAlNohgFQIh7oF1e72eg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

不管是HBase自带的哪一种pre-split算法，都是建立在表数据本身的rowkey符合它约定格式的条件下，实际用户还是需要按业务来设计rowkey，并实现自己的pre-split算法（实现SplitAlgorithm接口）

- 对集群影响：1星（创建表操作，不影响其他集群业务）
- 实用性：3星（实际pre-split都是按实际业务来的，对于测试来说可以使用HBase默认的split算法来构造rowkey格式）



## **FSHLog工具**

FSHLog是HBase自带的一个WALs文件检查和split工具，它主要分为两部分功能

- **dump**

将某个WAL文件中的内容dump出来具体的内容

- **split**

触发某个WAL文件夹的WAL split操作

```bash
#dump出某个当前的WALs文件中的内容
hbase org.apache.hadoop.hbase.regionserver.wal.FSHLog --dump /hbase/WALs/node-ana-coreqzlq0002.1432edca-3d6f-4e17-ad52-098f2adde2e6.com,16020,1591846214733/node-ana-coreqzlq0002.1432edca-3d6f-4e17-ad52-098f2adde2e6.com%2C16020%2C1591846214733.1592184625801
```



相关参数

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWzm3LtyUUCUN4chJG5A3m9mfpqIRfo9bibIIRuhic7yHXvQBASh1ngt0A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：2星（触发的WAL split操作会对相应的Worker节点增加少量的负载，当需要split的WAL极大时，会对region级别的业务有影响）
- 实用性：4星（可以很好的检查WAL内容的准确性，以及适用于WAL搬迁的场景）



## **WALPlayer工具**

WALPlayer是一个将WAL文件中的log回放到HBase的工具。可以通过对某个表或者所有表进行数据回放，也可以指定相应的时间区间等条件进数据回放。



```
#回放一个WAL文件的数据到表t1
hbase org.apache.hadoop.hbase.mapreduce.WALPlayer /tmp/node-ana-coreqzlq0002.1432edca-3d6f-4e17-ad52-098f2adde2e6.com%2C16020%2C1591846214733.1592184625801 t1
```

**Q&A：**FSHLog和WALPlayer都能将WAL文件中的数据恢复到HBase中，有什么差异区别？

FSHLog是触发WAL split请求到HMaster中，会对WAL中的所有数据恢复到HBase，走的是HBase自己的WAL split流程。而WALPlayer是本身起MR任务来扫WAL文件中的数据，对符合条件的数据put到特定的表中或输出HFile到特定目录

相关参数：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTW66eXaLwy9Lp0lviaCgVag81U2bNe4ad94FmWBIVT2n8YiaMkb0YczL9w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：3星（起MR任务会占用部分集群资源）
- 实用性：4星（在某些特定的场景下实用性很高，如replication预同步，表数据恢复）



## **OfflineMetaRepair工具**

OfflineMetaRepair工具由于修复HBase的元数据。它会基于HBase在HDFS上的region/table元数据，重建HBase元数据。



```
#重新建立hbase的元数据
hbase org.apache.hadoop.hbase.util.hbck.OfflineMetaRepair
```

**Q&A：**hbck的fixMeta同样可以修复HBase的元数据，还能指定具体的表使用更加灵活，还有必要使用OfflineMetaRepair？

hbck工具是HBase的在线修复工具，如果HBase没有启动是无法使用的。OfflineMetaRepair是在离线状态修复HBase元数据

相关参数：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTW7BZU69NWl9sNQ2KFn66ZmtcO8n8RpIZiazmNRyicLRRibDOqOQV0pAXjw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：**5星**（备份原始元数据表后，会重建HBase元数据）
- 实用性：4星（当HBase由于元数据原因无法启动时，此工具可以恢复HBase）



## **Sweeper工具**

Sweeper工具（HBASE-11644）可以合并HBase集群中小的MOB文件并删除冗余的MOB文件。它会基于Column Family起相应的SweepJob任务来对相应的MOB文件进行合并。注意，此工具不能与MOB的major compaction同时运行，并且同一个Column Family的Sweeper任务不能同时有多个一起运行。



```
#对t1表执行Sweeper
hbase org.apache.hadoop.hbase.mob.mapreduce.Sweeper t1 cf1
```

相关参数：

![图片](https://mmbiz.qpic.cn/mmbiz/TwK74MzofXfsuj1KVN6LMYDEOC9bqdTWa7wsSUtQQ8n7lr11Fic7tGbkPS4iaRsibYKtxOWwa2uz1jKHN7T31mAXw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：

- 对集群影响：**5星**（合并MOB任务会占用大量的Yarn资源和IO，对业务影响很大）

- 实用性：2星（只适合MOB场景，使用MOB会存在HMaster上compact的瓶颈暂不推荐（社区HBASE3上才支持，相关jira HBASE-22749））

  以上就是此次介绍的所有HBase运维工具，其他的如Bulkload批量导入，数据迁移，测试相关的pe等暂不描述。如果有写的不对的请指正，多谢。

官方文档：hbase.apache.org/book.html