## Spark介绍

按照官方的定义，Spark 是一个通用，快速，适用于大规模数据的处理引擎。

- 通用性：我们可以使用Spark SQL来执行常规分析， Spark Streaming 来流数据处理， 以及用Mlib来执行机器学习等。Java，python，scala及R语言的支持也是其通用性的表现之一。
- 快速：这个可能是Spark成功的最初原因之一，主要归功于其基于内存的运算方式。当需要处理的数据需要反复迭代时，Spark可以直接在内存中暂存数据，而无需像Map Reduce一样需要把数据写回磁盘。官方的数据表明：它可以比传统的Map Reduce快上100倍。
- 大规模：原生支持HDFS，并且其计算节点支持弹性扩展，利用大量廉价计算资源并发的特点来支持大规模数据处理。

### 我们能用它做什么

那我们能用Spark来做什么呢？场景数不胜数。
最简单的可以只是统计一下某一个页面多少点击量，复杂的可以通过机器学习来预测。

*个性化* 是一个常见的案例，比如说，Yahoo的网站首页使用Spark来实现快速的用户兴趣分析。应该在首页显示什么新闻？原始的做法是让用户选择分类；聪明的做法就是在用户交互的过程中揣摩用户可能喜欢的文章。另一方面就是要在新闻进来时候进行分析并确定什么样的用户是可能的受众。新闻的时效性非常高，按照常规的MapReduce做法，对于Yahoo几亿用户及海量的文章，可能需要计算一天才能得出结果。Spark的高效运算可以在这里得到充分的运用，来保证新闻推荐在数十分钟或更短时间内完成。另外，如美国最大的有线电视商Comcast用它来做节目推荐，最近刚和滴滴联姻的uber用它实时订单分析，优酷则在Spark上实现了商业智能的升级版。

### Spark生态系统

在我们开始谈MongoDB 和Spark 之前，我们首先来了解一下Spark的生态系统。Spark 作为一个大型分布式计算框架，需要和其他组件一起协同工作。
![img](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OB8tysAYh45cSDL2ggQeBxIBpuZDy4Ks41WtAONK5mfvLThhGVk8tT06icnWnDIns6oMYYgqqBiacw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
在Hdaoop里面，HDFS是其核心，作为一个数据层。

Spark是Hadoop生态系统的一颗新星，原生就支持HDFS。大家知道HDFS是用来管理大规模非结构化数据的存储系统，具有高可用和巨大的横向扩展能力。

而作为一个横向扩展的分布式集群，资源管理是其核心必备的能力，Spark 可以通过YARN或者MESOS来负责资源（CPU）分配和任务调度。如果你不需要管理节点的高可用需求，你也可以直接使用Spark standalone。

在有了数据层和资源管理层后， 接下来就是我们真正的计算引擎。

Hadoop技术的两大基石之一的MapReduce就是用来实现集群大规模并行计算。而现在就多了一个选项：Spark。Map Reduce的特点是，用4个字来概括，简单粗暴。采用divide & conquer战术，我们可以用Map Reduce来处理PB级的数据。而Spark 作为打了鸡血的Map Reduce增强版，利用了内存价格大量下降的时代因素，充分把计算所用变量和中间结果放到内存里，并且提供了一整套机器学习的分析算法，在加上很多语言的支持，使之成为一个较之于Map Reduce更加优秀的选择。

由于MapReduce 是一个相对并不直观的程序接口，所以为了方便使用，一系列的高层接口如Hive或者Pig应运而生。Hive可以让我们使用非常熟悉的SQL语句的方式来做一些常见的统计分析工作。同理，在Spark 引擎层也有类似的封装，如Spark SQL、RDD以及2.0版本新推出的Dataframe等。

所以一个完整的大数据解决方案，包含了存储，资源管理，计算引擎及接口层。那么问题来了：我们画了这么大这么圆的大饼，MongoDB可以吃哪一块呢？

![img](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OB8tysAYh45cSDL2ggQeBxHxksZoXcOI8seibBkticgqGLekla9fkVlaKsmJPyUaozKdBdhHO7pR9A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
MongoDB是个什么？是个database。所以自然而然，MongoDB可以担任的角色，就是数据存储的这一部分。在和 Spark一起使用的时候，MongoDB就可以扮演HDFS的角色来为Spark提供计算的原始数据，以及用来持久化分析计算的结果。

### HDFS vs. MongoDB

既然我们说MongoDB可以用在HDFS的地方，那我们来详细看看两者之间的差异性。

在说区别之前，其实我们可以先来注意一下两者的共同点。HDFS和MongoDB都是基于廉价x86服务器的横向扩展架构，都能支持到TB到PB级的数据量。数据会在多节点自动备份，来保证数据的高可用和冗余。两者都支持非结构化数据的存储，等等。

但是，HDFS和MongoDB更多的是差异点：

- 如在存储方式上 HDFS的存储是以文件为单位，每个文件64MB到128MB不等。而MongoDB则是细颗粒化的、以文档为单位的存储。
- HDFS不支持索引的概念，对数据的操作局限于扫描性质的读，MongoDB则支持基于二级索引的快速检索。
- MongoDB可以支持常见的增删改查场景，而HDFS一般只是一次写入后就很难进行修改。
- 从响应时间上来说，HDFS一般是分钟级别而MongoDB对手请求的响应时间通常以毫秒作为单位。

### 一个日志的例子

如果说刚才的比较有些抽象，我们可以结合一个实际一点的例子来理解。

比如说，一个比较经典的案例可能是日志记录管理。在HDFS里面你可能会用日期范围来命名文件，如7月1日，7月2日等等，每个文件是个日志文本文件，可能会有几万到几十万行日志。

而在MongoDB里面，我们可以采用一个JSON的格式，每一条日志就是一个JSON document。我们可以对某几个关心的字段建索引，如时间戳，错误类型等。

我们来考虑一些场景，加入我们相对7月份所有日志做一些全量的统计，比如每个页面的所有点击量，那么这个HDFS和MongoDB都可以正常处理。

如果有一天你的经理告诉你：他想知道网站上每天有多少404错误在发生，这个时候如果你用HDFS，就还是需要通过全量扫描所有行，而MongoDB则可以通过索引，很快地找到所有的404日志，可能花数秒钟就可以解答你经理的问题。

又比如说，如果你希望对每个日志项加一个自定义的属性，在进行一些预处理后，MongoDB就会比较容易地支持到。而一般来说，HDFS是不支持更新类型操作的。

好的，我们了解了MongoDB为什么可以替换HDFS并且为什么有这个必要来做这个事情，下面我们就来看看Spark和MongoDB怎么玩！

## Spark + MongoDB

Spark的工作流程可以概括为三部曲：创建并发任务，对数据进行transformation操作，如map， filter，union，intersect等，然后执行运算，如reduce，count，或者简单地收集结果。
![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
这里是Spark和MongoDB部署的一个典型架构。
Spark任务一般由Spark的driver节点发起，经过Spark Master进行资源调度分发。比如这里我们有4个Spark worker节点，这些节点上的几个executor 计算进程就会同时开始工作。一般一个core就对应一个executor。

每个executor会独立的去MongoDB取来原始数据，直接套用Spark提供的分析算法或者使用自定义流程来处理数据，计算完后把相应结果写回到MongoDB。

我们需要提到的是：在这里，所有和MongoDB的交互都是通过一个叫做Mongo-Spark的连接器来完成的。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
另一种常见的架构是结合MongoDB和HDFS的。Hadoop在非结构化数据处理的场景下要比MongoDB的普及率高。所以我们可以看到不少用户会已经将数据存放在HDFS上。这个时候你可以直接在HDFS上面架Spark来跑，Spark从HDFS取来原始数据进行计算，而MongoDB在这个场景下是用来保存处理结果。为什么要这么麻烦？几个原因：

- Spark处理结果数量可能会很大，比如说，个性化推荐可能会产生数百万至数千万条记录，需要一个能够支持每秒万级写入能力的数据库
- 处理结果可以直接用来驱动前台APP，如用户打开页面时获取后台已经为他准备好的推荐列表。

### Mongo Spark Connector 连接器

在这里我们在介绍下MongoDB官方提供的Mongo Spark连接器 。目前有3个连接器可用，包括社区第三方开发的和之前Mongo Hadoop连接器等，这个Mongo-Spark是最新的，也是我们推荐的连接方案。
![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
这个连接器是专门为Spark打造的，支持双向数据，读出和写入。但是最关键的是**条件下推**，也就是说：如果你在Spark端指定了查询或者限制条件的情况下，这个条件会被下推到MongoDB去执行，这样可以保证从MongoDB取出来、经过网络传输到Spark计算节点的数据确实都是用得着的。没有下推支持的话，每次操作很可能需要从MongoDB读取全量的数据，性能体验将会很糟糕。拿刚才的日志例子来说，如果我们只想对404错误日志进行分析，看那些错误都是哪些页面，以及每天错误页面数量的变化，如果有条件下推，那么我们可以给MongoDB一个限定条件：错误代码=404， 这个条件会在MongoDB服务器端执行，这样我们只需要通过网络传输可能只是全部日志的0.1%的数据，而不是没有条件下推情况下的全部数据。

另外，这个最新的连接器还支持和Spark计算节点Co-Lo 部署。就是说在同一个节点上同时部署Spark实例和MongoDB实例。这样做可以减少数据在网络上的传输带来的资源消耗及时延。当然，这种部署方式需要注意内存资源和CPU资源的隔离。隔离的方式可以通过Linux的cgroups。

## Spark + MongoDB 成功案例

目前已经有很多案例在不同的应用场景中使用Spark+MongoDB。
法国航空是法国最大的航空公司，为了提高客户体验，在最近施行的360度客户视图中，使用Spark对已经收集在MongoDB里面的客户数据进行分类及行为分析，并把结果（如客户的类别、标签等信息）写回到MongoDB内每一个客户的文档结构里。

Stratio是美国硅谷一家著名的金融大数据公司。他们最近在一家在31个国家有分支机构的跨国银行实施了一个实时监控平台。该银行希望通过对日志的监控和分析来保证客户服务的响应时间以及实时监测一些可能的违规或者金融欺诈行为。在这个应用内， 他们使用了：

Stratio是美国硅谷一家著名的金融大数据公司。他们最近在一家在31个国家有分支机构的跨国银行实施了一个实时监控平台。该银行希望通过对日志的监控和分析来保证客户服务的响应时间以及实时监测一些可能的违规或者金融欺诈行为。在这个应用内， 他们使用了：

- Apache Flume 来收集log
- Spark来处理实时的log
- MongoDB来存储收集的log以及Spark分析的结果，如Key Performance Indicators等

东方航空最近刚完成一个Spark运价的POC测试。

### 东方航空的挑战

东方航空作为国内的3大行之一，每天有1000多个航班，服务26万多乘客。过去，顾客在网站上订购机票，平均资料库查询200次就会下单订购机票，但是现在平均要查询1.2万次才会发生一次订购行为，同样的订单量，查询量却成长百倍。按照50%直销率这个目标计算，东航的运价系统要支持每天16亿的运价请求。

### 思路：空间换时间

当前的运价是通过实时计算的，按照现在的计算能力，需要对已有系统进行100多倍的扩容。另一个常用的思路，就是采用空间换时间的方式。与其对每一次的运价请求进行耗时300ms的运算，不如事先把所有可能的票价查询组合穷举出来并进行批量计算，然后把结果存入MongoDB里面。当需要查询运价时，直接按照 出发+目的地+日期的方式做一个快速的DB查询，响应时间应该可以做到几十毫秒。

那为什么要用MongoDB？因为我们要处理的数据量庞大无比。按照1000多个航班，365天，26个仓位，100多渠道以及数个不同的航程类型，我们要实时存取的运价记录有数十亿条之多。这个已经远远超出常规RDBMS可以承受的范围。

MongoDB基于内存缓存的数据管理方式决定了对并发读写的响应可以做到很低延迟，水平扩展的方式可以通过多台节点同时并发处理海量请求。
事实上，全球最大的航空分销商，管理者全世界95%航空库存的Amadeus也正是使用MongoDB作为其1000多亿运价缓存的存储方案。

### Spark + MongoDB 方案

我们知道MongoDB可以用来做我们海量运价数据的存储方案，在大规模并行计算方案上，就可以用到崭新的Spark技术。
![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
这里是一个运价系统的架构图。左边是发起航班查询请求的客户端，首先会有API服务器进行预处理。一般航班请求会分为库存查询和运价查询。库存查询会直接到东航已有的库存系统（Seat Inventory），同样是实现在MongoDB上面的。在确定库存后根据库存结果再从Fare Cache系统内查询相应的运价。

Spark集群则是另外一套计算集群，通过Spark MongoDB连接套件和MongoDB Fare Cache集群连接。Spark 计算任务会定期触发（如每天一次或者每4小时一次），这个任务会对所有的可能的运价组合进行全量计算，然后存入MongoDB，以供查询使用。右半边则把原来实时运算的集群换成了Spark+MongoDB。Spark负责批量计算一年内所有航班所有仓位的所有价格，并以高并发的形式存储到MongoDB里面。每秒钟处理的运价可以达到数万条。
当来自客户端的运价查询达到服务端以后，服务端直接就向MongoDB发出按照日期，出发到达机场为条件的mongo查询。

### 批处理计算流程

![img](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OB8tysAYh45cSDL2ggQeBxCxicawGMyeCgAWhDria0gGrGuss27o8SYibpZ2Aibpz0SylX0iaTKRwT5hg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
这里是Spark计算任务的流程图。需要计算的任务，也就是所有日期航班仓位的组合，事先已经存放到MongoDB里面。
任务递交到master，然后预先加载所需参考数据，broadcast就是把这些在内存里的数据复制到每一个Spark计算节点的JVM，然后所有计算节点多线程并发执行，从Mongodb里取出需要计算的仓位，调用东航自己的运价逻辑，得出结果以后，并保存回MongoDB。

### Spark 任务入口程序

Spark和MongoDB的连接使用非常简单，下面就是一个代码示例：

```
// initialization dependencies including base prices, pricing rules and some reference dataMap dependencies = MyDependencyManager.loadDependencies();// broadcasting dependencies
javaSparkContext.broadcast(dependencies);// create job rdd
cabinsRDD = MongoSpark.load(javaSparkContext).withPipeline(pipeline)// for each cabin, date, airport pair, calculate the price
cabinsRDD.map(function calc_price);// collect the result, which will cause the data to be stored into MongoDB
cabinsRDD.collect()
cabinsRDD.saveToMongo()
```

### 处理能力和响应时间比较

这里是一个在东航POC的简单测试结果。从吞吐量的角度，新的API服务器单节点就可以处理3400个并发的运价请求。在显著提高了并发的同时，响应延迟则降低了10几倍，平均10ms就可以返回运价结果。按照这个性能，6台 API服务器就可以应付将来每天16亿的运价查询。
![img](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OB8tysAYh45cSDL2ggQeBxnibsShgzXueqHibQ5VjTp8OvpUdW1qyw0cCRsicSQiakJvCPNvvfPcnWZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Spark ＋ MongoDB演示

接下来是一个简单的Spark+MongoDB演示。

### 安装 Spark

```
# curl -OL http://d3kbcqa49mib13.cloudfront.net/spark-1.6.0-bin-hadoop2.6.tgz# mkdir -p ~/spark# tar -xvf spark-1.6.0-bin-hadoop2.6.tgz -C ~/spark --strip-components=1
```

### 测试连接器

```
# cd ～／spark# ./bin/spark-shell \--conf "spark.mongodb.input.uri=mongodb://127.0.0.1/flights.av" \
--conf "spark.mongodb.output.uri=mongodb://127.0.0.1/flights.output" \
--packages org.mongodb.spark:mongo-spark-connector_2.10:1.0.0import com.mongodb.spark._
import org.bson.DocumentMongoSpark.load(sc).take(10).foreach(println)
```

### 简单分组统计

数据：365天，所有航班库存信息，500万文档
任务：按航班统计一年内所有余票量

```
MongoSpark.load(sc)
     .map(doc=>(doc.getString("flight") ,doc.getLong("seats")))
     .reduceByKey((x,y)=>(x+y))
      .take(10)
     .foreach(println)
```

### 简单分组统计加条件过滤

数据：365天，所有航班库存信息，500万文档
任务：按航班统计一年内所有库存，但是只处理昆明出发的航班

```
import org.bson.DocumentMongoSpark.load(sc)
          .withPipeline(Seq(Document.parse("{ $match: { orig :  'KMG'  } }")))
    .map(doc=>(doc.getString("flight") ,doc.getLong("seats")))
    .reduceByKey((x,y)=>(x+y))
    .take(10)
    .foreach(println)
```

### 性能优化事项

- 使用合适的chunksize (MB)
  Total data size / chunksize = chunks = RDD partitions = spark tasks
- 不要将所有CPU核分配给Spark
  预留1-2个core给操作系统及其他管理进程
- 同机部署
  适当情况可以同机部署Spark+MongoDB，利用本地IO提高性能

## 总结

上面只是一些简单的演示，实际上Spark + MongoDB的使用可以通过Spark的很多种形式来使用。我们来总结一下Spark ＋ Mongo的应用场景。在座的同学可能很多人已经使用了MongoDB，也有些人已经使用了Hadoop。我们可以从两个角度来考虑这个事情：

- 对那些已经使用MongoDB的用户，如果你希望在你的MongoDB驱动的应用上提供个性化功能，比如说像Yahoo一样为你找感兴趣的新闻，能够在你的MongoDB数据上利用到Spark强大的机器学习或者流处理，你就可以考虑在MongoDB集群上部署Spark来实现这些功能。
- 如果你已经使用Hadoop而且数据已经在HDFS里面，你可以考虑使用Spark来实现更加实时更加快速的分析型需求，并且如果你的分析结果有数据量大、格式多变以及这些结果数据要及时提供给前台APP使用的需求，那么MongoDB可以用来作为你分析结果的一个存储方案。