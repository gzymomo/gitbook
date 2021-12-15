# SparkSQL在字节跳动的应用实践和优化实战

原文地址：微信公众号：大数据技术与架构 

**场景描述：**面对大量复杂的数据分析需求，提供一套稳定、高效、便捷的企业级查询分析服务具有重大意义。本次演讲介绍了字节跳动基于SparkSQL建设大数据查询统一服务TQS（Toutiao Query Service）的一些实践以及在执行计划调优、数据读取剪枝、SQL兼容性等方面对SparkSQL引擎的一些优化。

**关键词：**SparkSQL优化 字节跳动



# 一、目标和能力

为公司内部提供 Hive 、 Spark - SQL 等 OLAP 查询引擎服务支持。

- 提供全公司大数据查询的统一服务入口，支持丰富的API接口，覆盖Adhoc、ETL等SQL查询需求
- 支持多引擎的智能路由、参数的动态优化
- Spark-SQL/Hive引擎性能优化

针对SparkSQL，主要做了以下优化：

## 1.1 执行计划自动调优

​    **•基于AE的 ShuffledHashJoin调整**

​    **•Leftjoinbuildleftmap技术**

## 1.2 数据读取剪枝

​    **•Parquetlocalsort**

​    **•BloomFilter&BitMap**

​    **•Prewhere**

## 1.3 一些其它优化

# 二、执行计划调优

## 2.1 执行计划的自动调优

Spark Adaptive Execution （  Intel®Software）,简称SparkAE，总体思想是将sparksql生成的1个job中的所有stage单独执行，为每一个stage单独创建一个子job，子job执行完后收集该stage相关的统计信息（主要是数据量和记录数），并依据这些统计信息优化调整下游stage的执行计划。

目前SparkAE主要支持的功能：

（1）数据倾斜的调整

（2）小task的合并

（3）sortmerge-> broadcase

Spark 有3种join方式：Broadcastjoin、ShuffledHashJoin、SortMergeJoin

普通leftjoin无法build 左表

## 2.2 优化点

在AE的框架下，根据shuffle数据量大小，自动调整join执行计划：SortMergeJoin调整为 ShuffledHashJoin•扩展支持left-join时将左表build成HashMap。

省去了大表join小表的情况下对shuffle数据的排序过程、join过程以HashMap完成，实现join提速。

- **SortMergeJoin调整为ShuffledHashJoin**



![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlXCMUTqpQLDs1jB7sfyMa9qGeTbqXbRuX1IgK3YKT40sxxWQudUWoZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



- **Leftjoin build left sidemap**

1、初始化表A的一个匹配记录的映射表

目标：

对于Left-join的情况，可以对左表进行HashMapbuild。使得小左表leftjoin大右表的情况可以进行ShuffledHashJoin调整

难点：

Left-join语义：左表没有join成功的key，也需要输出

原理

在构建左表Map的时候，额外维持一个"是否已匹配"的映射表；在和右表join结束之后，把所有没有匹配到的key，用null进行join填充。

以 Aleft join B 为例：

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlNNTmfaQQBe9BBVSZiaBfmSIo93IEB4ibTpuibIr0MLkHib5dSmEGOzFb6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



2、join过程中，匹配到的key置为1，没有匹配到的项不变（如key3）

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlu1UxBdVSE4O3JluujSs9Wbic9YzlO89xhhpXlmibU2NkRSFKQjsVXshQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、join结束后，没有匹配到的项，生成一个补充结果集R2

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlDPblJFYjvw7WOH8MuX4qc3nCb5EZ1YCOvP0VM0mvXJn3BwIeMZgGtw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYl3JKN2l7eXGPcq4gZnuboaT23gc6CHfaI0xtl2gPj7OJduCxx59AZ6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4.合并结果集R1和结果集R2，输出最终生成的join结果R。



![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlB7ibIuAqJhuYP3VibNLJznzt00P9mNXuVF8vndqw02k3zRQY9pzo1XmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.3 优化结果

- 约95%左右的joinSQL有被调整成ShuffledHashJoin/BroadcastJoin
- 被优化的SQL整体速度提升20%~30%
- 整体执行时长缩短







![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYl15aotZiaDkro5Rbxp9Jn4uu4ibQ2g7mpoialw5dpsVV1YiaNzU6bNG4oKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

# 三、基于Parquet数据读取剪枝

以parquet格式数据为对象，在数据读取时进行适当的过滤剪枝，从而减少读取的数据量，加速查询速度

优化点：

- LocalSort
- BoomFilter
- BitMap
- Prewhere

## 3.1 基于Parquet数据读取剪枝：LocalSort

对parquet文件针对某个高频字段进行排序。从而实现读数据时RowGroup的过滤

目标：

- 自动选择排序字段
- 生成文件时自动排序

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYl3tvwUfn6KrSNLaLewK1LlEhoGggk4aHKOC25F8Du8BQMzJeP0mkqrw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Parquet文件读取原理：

  （1）每个rowgroup的元信息里，都会记录自己包含的各个列的最大值和最小值

  （2）读取时如何这个值不在最大值、最小值范围内，则跳过RowGroup

生成hive分区文件时，先读取metastore，获取它是否需要使用localsort，如果需要，选择它的高频列是哪个。

## 3.2 基于Parquet数据读取剪枝：BloomFilter&BitMap

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlnkcTDrITLN1sHDVyEpd5PRjpN8pCY2icsVJEVmPpk1ibhj8uRLh6Vopg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlNrorR980biaYuqbTAlvdgJN8HhjicwBichxUHgkPWX2wHZNRRCzvTmhdw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYlFS4stIGqdjF8nhibRaRicKGYpyyCpp7nLFuYcakuNVWOQpLJibcLZX5zQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 1. 整体优化结果

- 命中索引平均性能提升 30%
- 生成时间增加：10%
- 空间开销增加：5%

### 2. 如何选取合适的列

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYl333WnFduFqlLYVhjHujHibSjhZicACKHx8Y01y7gPLSVM7ckqeMx8YSw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Local_sort &BloomFilter & BitMap 如何自动生效**

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2NbGnhw8DL2YWjuhweTpFYloliadt1ENicYj3e9QiaVpn3NyBAwqDA5RIk8ZHlyqon5GuuR51mHQ8DXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 3.4 基于Parquet数据读取剪枝：Prewhere

基于列式存储各列分别存储、读取的特性•针对需要返回多列的SQL，先根据下推条件对RowId进行过滤、选取。再有跳过地读取其他列，从而减少无关IO和后续计算•谓词选择（简单、计算量小）:in,=,<>,isnull,isnotnull

优化结果使得：特定SQL（Project16列，where条件 2列）SQL平均性能提升20%

# 五、其他优化

- **Hive/SparkLoad**分区Move文件优化：

通过调整staging目录位置，实现在Load过程中mv文件夹，替代逐个mv文件，从而减少与NameNode的交互次数

- **Spark**生成文件合并

通过最后增加一个repartitionstage合并spark生成文件。

- **Vcore**

对于CPU使用率低的场景，通过vcore技术使得一个yarn-core可以启动多个spark-core

- **Spark 访问hivemetastore 特定filter**下推：

构造 get_partitions_by_filter实现 cast、substring等条件下推hivemetastore，从而减轻metastore返回数据量

# 六、运行期调优

在SQL执行前，通过统一的查询入口，对其进行**基于代价的预估**，选择合适的引擎和参数:

## 6.1 SQL分析

- 抽取Hiveexplain逻辑，进行SQL语法正确性检查
- 对SQL包含的算子、输入的数据量进行标注

## 6.2 自动引擎选择/自动参数优化

**标注结果自动选择执行引擎：**

- 小SQL走SparkServer（省去yarn申请资源耗时）
- 其他默认走Spark-Submit

**标注结果选择不同运行参数：**

- Executor个数/内存
- Overhead、堆外内存

调优后使得Adhoc30s以内SQL占比45%，Spark-Submit内存使用量平均减少20%。

# 七、代码层面的优化

```
使用reduceByKey/aggregateByKey替代groupByKey。

使用mapPartitions替代普通map。

mapPartitions类的算子，一次函数调用会处理一个partition所有的数据，而不是一次函数调用处理一条，性能相对来说会高一些。但是有的时候，使用mapPartitions会出现OOM（内存溢出）的问题。因为单次函数调用就要处理掉一个partition所有的数据，如果内存不够，垃圾回收时是无法回收掉太多对象的，很可能出现OOM异常。所以使用这类操作时要慎重！

使用foreachPartitions替代foreach。

原理类似于“使用mapPartitions替代map”，也是一次函数调用处理一个partition的所有数据，而不是一次函数调用处理一条数据。在实践中发现，foreachPartitions类的算子，对性能的提升还是很有帮助的。比如在foreach函数中，将RDD中所有数据写MySQL，那么如果是普通的foreach算子，就会一条数据一条数据地写，每次函数调用可能就会创建一个数据库连接，此时就势必会频繁地创建和销毁数据库连接，性能是非常低下；但是如果用foreachPartitions算子一次性处理一个partition的数据，那么对于每个partition，只要创建一个数据库连接即可，然后执行批量插入操作，此时性能是比较高的。实践中发现，对于1万条左右的数据量写MySQL，性能可以提升30%以上。

使用filter之后进行coalesce操作。

通常对一个RDD执行filter算子过滤掉RDD中较多数据后（比如30%以上的数据），建议使用coalesce算子，手动减少RDD的partition数量，将RDD中的数据压缩到更少的partition中去。因为filter之后，RDD的每个partition中都会有很多数据被过滤掉，此时如果照常进行后续的计算，其实每个task处理的partition中的数据量并不是很多，有一点资源浪费，而且此时处理的task越多，可能速度反而越慢。因此用coalesce减少partition数量，将RDD中的数据压缩到更少的partition之后，只要使用更少的task即可处理完所有的partition。在某些场景下，对于性能的提升会有一定的帮助。

使用repartitionAndSortWithinPartitions替代repartition与sort类操作。

repartitionAndSortWithinPartitions是Spark官网推荐的一个算子。官方建议，如果是需要在repartition重分区之后还要进行排序，就可以直接使用repartitionAndSortWithinPartitions算子。因为该算子可以一边进行重分区的shuffle操作，一边进行排序。shuffle与sort两个操作同时进行，比先shuffle再sort来说，性能可能是要高的。

尽量减少shuffle相关操作，减少join操作。
```

## 7.1 写入数据库时，设置批量插入，关闭事务

```
result.write.mode(SaveMode.Append).format("jdbc")
      .option(JDBCOptions.JDBC_URL,"jdbc:mysql://127.0.0.1:3306/db?rewriteBatchedStatement=true")    //开启批量处理
      .option("user","root")
      .option("password","XXX")
      .option(JDBCOptions.JDBC_TABLE_NAME,"xxx")
      .option(JDBCOptions.JDBC_TXN_ISOLATION_LEVEL,"NONE")    //不开启事务
      .option(JDBCOptions.JDBC_BATCH_INSERT_SIZE,10000)   //设置批量插入数据量
      .save()
```

## 7.2 缓存复用数据

如在代码下方反复用到了Result数据，可以考虑将此数据缓存下来。

```
val Result = spark.sql(
      """
        |SELECT * from A
        |UNION
        |SELECT * FROM B
      """.stripMargin)
Result.persist(StorageLevel.DISK_ONLY_2)
Result.registerTempTable("Result")
```

## 7.3 参数优化

注意：具体参数设置需要具体问题具体分析，并不是越大越好，需反复测试寻找最优值。另外不同Spark版本的参数可能有过期，请注意区分。

```
    //CBO优化
    sparkConf.set("spark.sql.cbo.enabled","true")
    sparkConf.set("spark.sql.cbo.joinReorder.enabled","true")
    sparkConf.set("spark.sql.statistics.histogram.enabled","true")
    //自适应查询优化(2.4版本之后)
    sparkConf.set("spark.sql.adaptive.enabled","true")
    //开启consolidateFiles
    sparkConf.set("spark.shuffle.consolidateFiles","true")
    //设置并行度
    sparkConf.set("spark.default.parallelism","150")
    //设置数据本地化等待时间
    sparkConf.set("spark.locality.wait","6s")
    //设置mapTask写磁盘缓存
    sparkConf.set("spark.shuffle.file.buffer","64k")
    //设置byPass机制的触发值
    sparkConf.set("spark.shuffle.sort.bypassMergeThreshold","1000")
    //设置resultTask拉取缓存
    sparkConf.set("spark.reducer.maxSizeInFlight","48m")
    //设置重试次数
    sparkConf.set("spark.shuffle.io.maxRetries","10")
    //设置重试时间间隔
    sparkConf.set("spark.shuffle.io.retryWait","10s")
    //设置reduce端聚合内存比例
    sparkConf.set("spark.shuffle.memoryFraction","0.5")
    //设置序列化
    sparkConf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    //设置自动分区
    sparkConf.set("spark.sql.auto.repartition","true")
    //设置shuffle过程中分区数
    sparkConf.set("spark.sql.shuffle.partitions","500")
    //设置自动选择压缩码
    sparkConf.set("spark.sql.inMemoryColumnarStorage.compressed","true")
    //关闭自动推测分区字段类型
    sparkConf.set("spark.sql.source.partitionColumnTypeInference.enabled","false")
    //设置spark自动管理内存
    sparkConf.set("spark.sql.tungsten.enabled","true")
    //执行sort溢写到磁盘
    sparkConf.set("spark.sql.planner.externalSort","true")
    //增加executor通信超时时间
    sparkConf.set("spark.executor.heartbeatInterval","60s")
    //cache限制时间
    sparkConf.set("spark.dynamicAllocation.cachedExecutorIdleTimeout","120")
    //设置广播变量
    sparkConf.set("spark.sql.autoBroadcastJoinThreshold","104857600")
    //其他设置
    sparkConf.set("spark.sql.files.maxPartitionBytes","268435456")
    sparkConf.set("spark.sql.files.openCostInBytes","8388608")
    sparkConf.set("spark.debug.maxToStringFields","500")
    //推测执行机制
    sparkConf.set("spark.speculation","true")
    sparkConf.set("spark.speculation.interval","500")
    sparkConf.set("spark.speculation.quantile","0.8")
    sparkConf.set("spark.speculation.multiplier","1.5")
```