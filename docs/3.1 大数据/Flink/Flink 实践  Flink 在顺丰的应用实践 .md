- [Flink 实践 | Flink 在顺丰的应用实践](https://mp.weixin.qq.com/s/xc-Yni0eIT07kh_AzWwmiA)

# 一、建设背景

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9ZsaxF7cBOYDqv42GOU8DpNp4V5HnqhGlgmiaHxZ8PrYJp7gbcUA8Tpw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

顺丰是国内领先的快递物流综合服务商，经过多年的发展，顺丰使用大数据技术支持高质量的物流服务。以下是一票快件的流转过程，可以看到从客户下单到最终客户收件的整个过程是非常长的，其中涉及的一些处理逻辑也比较复杂。为了应对复杂业务的挑战，顺丰进行了数据仓库的探索。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR93oXejFv8aw1j1YsqTevkXGHMsmXqMmxx0KhfPeyibEab9vgXqoxHaFA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



传统数仓主要分为离线和实时两个部分：

- 离线部分以固定的计算逻辑，通过定时调度，完成数据抽取，清洗，计算，最后产出报表；
- 而实时部分则是需求驱动的，用户需要什么，就马上着手开发。

这种数仓架构在数据量小、对实时性要求不高的情况下运行得很好。然而随着业务的发展，数据规模的扩大和实时需求的不断增长，传统数仓的缺点也被放大了。

- #### **从业务指标的开发效率来看**

  #### 实时指标采用的是需求驱动的、纵向烟囱式的开发模式，需要用户手写 Flink 任务进行开发，这种开发方式效率低门槛高，输出的指标很难统一管理与复用。

- #### **从技术架构方面来看**

  #### 离线和实时两套架构是不统一的，开发方式、运维方式、元数据方面都存在差异。传统架构整体还是以离线为主，实时为辅，依赖离线 T+1  调度导出报表，这些调度任务通常都运行在凌晨，导致凌晨时集群压力激增，可能会导致报表的产出不稳定；如果重要的报表产出有延迟，相应的下游的报表产出也会出现延迟。这种以离线为主的架构无法满足精细化、实时化运营的需要。

- #### **从平台管理的角度来看**

  #### 传统数仓的实时指标开发是比较粗放的，没有 Schema 的规范，没有元数据的管理，也没有打通实时和离线数据之间的联系。

  #### 为了解决传统数仓的问题，顺丰开始了实时数仓的探索。实时数仓和离线数仓实际上解决的都是相同的业务问题，最大的区别就在于时效性。

- - #### 离线数仓有小时级或天级的延迟；

  - #### 而实时数仓则是秒级或分钟级的延迟。

其他特性，比如数据源、数据存储以及开发方式都是比较相近的。因此，我们希望：

- 用户能从传统数仓平滑迁移到实时数仓，保持良好的体验；
- 同时统一实时和离线架构，加快数据产出，减少开发的撕裂感；
- 加强平台治理，降低用户使用门槛，提高开发效率也是我们的目标。

# 二、建设思路

经过总结，我们提炼出以下 3 个实时数仓的建设思路。

1. 首先是通过统一数仓标准、元数据以及开发流程，使得用户达到开发体验上的批流统一。
2. 随后，引入 Hudi 加速数仓宽表，基于 Flink SQL 建设我们的实时数仓。
3. 最后是加强平台治理，进行数仓平台化建设，实现数据统一接入、统一开发、以及统一的元数据管理。

## 1. 批流统一的实时数仓

建设批流统一的实时数仓可以分为以下 3 个阶段：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR92zSnDpzLzDwnryWAq3QSkW77LXWibc46wbD58rG2NiaCn0H2Bq2qreJA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 1.1 统一数仓规范

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR99qdZ2s8wUXldJZbA7nkZyUa2kCUnHDdJuPRvJib269fX4GIAPZdtMpQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先，无规矩不成方圆，建设数仓必须有统一的数仓规范。统一的数仓规范包括以下几个部分：

- 设计规范
- 命名规范
- 模型规范
- 开发规范
- 存储规范
- 流程规范

统一好数仓规范之后，开始数仓层级的划分，将实时和离线统一规划数仓层级，分为 ODS、DWD、DWS、ADS 层。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9swe5q9My9OLIgZy2b9SoqahKhknribQuY83ibeW5SGASRvYhvvcdqKcg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 1.2 统一元数据

基于以上统一的数仓规范和层级划分模型，可以将实时和离线的元数据进行统一管理。下游的数据治理过程，比如数据字典、数据血缘、数据质量、权限管理等都可以达到统一。这种统一可以沉淀实时数仓的建设成果，使数仓能更好的落地实施。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9WdteOm5l2gHEBlHqp7ZLU9W2xLyHjEV7Z7uKQtpQPeWIERujTx8EtQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 1.3 基于 SQL 统一开发流程

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9UBleSUXFzk2wkn4JtJPUmB7bBhPecxpY8ut80rzp2qNOA8wkQhME0Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

开发人员都知道，使用 DataStream API 开发 Flink 任务是比较复杂的。在数据量比较大的情况下，如果用户使用 API  不规范或者开发能力不足，可能会导致性能和稳定性的问题。如果我们能将实时开发的过程统一到 SQL  上，就可以达到减少用户开发成本、学习成本以及运维成本的目的。

之前提到过我们已经统一了实时和离线的元数据，那么就可以将上图左边的异构数据源和数据存储抽象成统一的 Table ，然后使用 SQL 进行统一的数仓开发，也就是将离线批处理、实时流处理以及 OLAP 查询统一 SQL 化。

### 1.4 实时数仓方案对比

完成了数仓规范、元数据、开发流程的统一之后，我们开始探索数仓架构的具体架构方案。业界目前的主流是 Lambda 架构和 Kappa 架构。

- **Lambda 架构**

  Lambda 架构是在原有离线数仓的基础上，将对实时性要求比较高的部分剥离出来，增加了一个实时速度层。Lambda 架构的缺点是需要维护实时和离线两套架构和两套开发逻辑，维护成本比较高，另外两套架构带来的资源消耗也是比较大的。


  ![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9nBEicMbVDdicjxicfFS9S7SacgPbDSKh7ydicFIBdNlVHv4XgxDrLzZIVQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- **Kappa 架构**

  为了应对 Lambda 架构的缺陷，Jay Kreps 提出了 Kappa 架构，Kappa 架构移除了原有的离线部分，使用纯流式引擎开发。Kappa 架构的最大问题是，流数据重放处理时的吞吐能力达不到批处理的级别，导致重放时产生一定的延时。


  ![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9z04TgIibowmKOTJ3jYnrTxkfprTqQA0jGcZJtf5iaYfHwaY8CYm4Y5Uw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- **实时数仓方案对比与实际需求**

  在真实的生产实践中，并不是一定要严格遵循规范的 Lambda 架构或 Kappa 架构，可以是两者的混合。比如大部分指标使用流式引擎开发，少部分重要的指标使用批处理开发，并增加数据校对的过程。

  在顺丰的业务场景中，并非所有用户都需要纯实时的表，许多用户的报表还是依赖离线 T+1 调度产出的宽表，如果我们能够加速宽表的产出，那么其他报表的时效性也能相应地得到提高。

  另外，这个离线 T+1 调度产出的宽表，需要聚合 45 天内多个数据源的全量数据，不管是 Lambda 架构还是 Kappa 架构，都需要对数据进行全量聚合，如果能够直接更新宽表，就可以避免全量重新计算，大大降低资源消耗和延时。


  ![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9sm9qBpC4xODZVskZf55tiaZEctuxucjnIaicUjtT8x1NP1J2GPksGTbQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2. 引入 Hudi 加速宽表

之前说过，维护 Lambda 架构的复杂性在于需要同时维护实时和离线两套系统架构。而对于这个缺点，我们可以通过批流统一来克服。

经过权衡，我们决定改造原有 Lambda 架构，通过加速它的离线部分来建设数仓宽表。此时，就需要一个工具来实时快速的更新和删除 Hive 表，支持 ACID  特性，支持历史数据的重放。基于这样的需求，我们调研了市面上的三款开源组件：Delta Lake、Iceberg、Hudi，最后选择 Hudi  来加速宽表。

### 2.1 Hudi 关键特性

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR94M7KXwHlP85c9akqJMxZYl2nVqj8Cd9AvAlMLJLoRs0o1KPiaIkkDXg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hudi 的关键特性包括：可回溯历史数据，支持在大规模数据集中根据主键更新删除数据；支持数据增量消费；支持 HDFS 小文件压缩。这些特性恰好能满足我们的需求。

### 2.2 引入 Hudi 加速宽表

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9UJglVreRQJpykzmMsLorNgkYhib7bXHfr1CQycHSWBhBjJj8zTfm2Sw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

引入 Hudi 有两种方式加速数仓。首先，在 ODS 层引入 Hudi 实现实时数据接入，将 ODS 层 T+1 的全量数据抽取改成 T+0 的实时接入，从数据源头实现 Hive 表的加速。

另外，使用 Flink 消费 Kafka 中接入的数据，进行清洗聚合，通过 Hudi 增量更新 DWD 层的 Hive 宽表，将宽表从离线加速成准实时。

### 2.3 构建实时数仓宽表示例

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9Ge6IMyoXibDMfY2Kic8gdWoNntXDJvcSicDBsVCJCnT6u1L5ZIf4zt0Tw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里通过一个例子介绍如何构建实时数仓宽表。

假设运单宽表由运单表，订单表和用户表组成，分别包含运单号、运单状态、订单号、订单状态、用户 ID、用户名等字段。

首先将运单表数据插入宽表，运单号作为宽表主键，并且将运单号和订单号的映射存入临时表。当订单表数据更新后，首先关联用户维表，获取用户名，再从临时表中获取对应运单号。最后根据运单号将订单表数据增量插入宽表，以更新宽表状态。

## 3. 最终架构

引入 Hudi 后，基于 Lambda 架构，我们定制化的实时数仓最终架构如下图所示。实时速度层通过 CDC 接入数据到 Kafka，采用  Flink SQL 处理 Kafka 中的数据，并将 ODS 层 Kafka 数据清洗计算后通过 Hudi 准实时更新 DWD  层的宽表，以加速宽表的产出。离线层采用 Hive 存储及处理。最后由 ADS 层提供统一的数据存储与服务。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9fN8lvOIRQ91cFzV788Z6ibOG8y4MqvOzKEEAC1O31jO68OUTfRLz6cA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

除了制定数仓标准和构建数仓架构，我们还需要构建数仓平台来约束开发规范和流程，提升开发效率，提高用户体验。

站在数据开发人员的角度，我们不仅要提供快速的数据接入能力，还需要关注开发效率以及统一的元数据治理。因此可以基于 Table 和 SQL 抽象，对数据接入、数据开发、元数据管理这三个主要功能进行平台化，为实时数仓用户提供统一、便捷、高效的体验。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9JDpjha1QINmLAjwwnX4ZXics6nqEdiaDhnu8vyuKG82JXiadVJm08RCKw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 三、落地实践

## 1. Hudi On Flink

顺丰是最早将 Hudi On Flink 引入生产实践的公司，顺丰内部使用版本基于 T3 出行的内部分支进行了许多修改和完善，大大提升了 Hudi on Flink 的性能和稳定性。

### 1.1 实现原理

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9yZMePml9ZibfEDpvicuZBflOHmBxURKKDyPTqGqVKDVtFx6Xfiavm54ibw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里介绍下 Hudi On Flink 的原理。Hudi 原先与 Spark 强绑定，它的写操作本质上是批处理的过程。为了解耦 Spark 并且统一  API ，Hudi On Flink 采用的是在 Checkpoint 期间攒批的机制，在 Checkpoint 触发时将这一批数据Upsert 到 Hive，根据 Upsert 结果统一提交或回滚。

Hudi On Flink 的实现流可以分解为几个步骤：

1. 首先使用 Flink 消费 Kafka 中的 Binlog 类型数据，将其转化为 Hudi Record。
2. Hudi Record 进入 InstantTime Generator，该 Operator 并不对数据做任何处理，只负责转发数据。它的作用是每次  Checkpoint 时在 Hudi 的 Timeline 上生成全局唯一且递增的 Instant，并下发。
3. 随后，数据进入 Partitioner ，根据分区路径以及主键进行二级分区。分区后数据进入 File Indexer ，根据主键找到在 HDFS  上需要更新的对应文件，将这个对应关系按文件 id 进行分桶，并下发到下游的 WriteProcessOperator 。
4. WriteProcessOperator 在 Checkpoint 期间会积攒一批数据，当 Checkpoint 触发时，通过 Hudi 的 Client 将这批数据 Upsert 到 HDFS 中，并且将 Upsert 的结果下发到下游的 CommitSink 。
5. CommitSink 会收集上游所有算子的 upsert 结果，如果成功的个数和上游算子的并行度相等时，就认为本次 commit 成功，并将 Instant 的状态设置为 success ，否则就认为本次 commit 失败并进行回滚。

### 1.2 优化

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR93MN5ntqpJcEHNhC3AtbX6OaxTjRgop44H5tbtFZJIMTq3KL58aDHPA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



顺丰基于社区代码对 Hudi On Flink 进行了一些优化，主要目的是增强性能和提升稳定性。

- **二级分区**

  对于增量写入的场景，大部分的数据都写入当天的分区，可能会导致数据倾斜。因此，我们使用分区路径和主键 id 实现二级分区，避免攒批过程中单个分区数据过多，解决数据倾斜问题。

- **文件索引**

  Hudi 写入过程的瓶颈在于如何快速找到记录要写入的文件并更新。为此 Hudi 提供了一套索引机制，该机制会将一个记录的键 +  分区路径的组合映射到一个文件 ID. 这个映射关系一旦记录被写入文件组就不会再改变。Hudi 当前提供了 HBase、Bloom Filter  和内存索引 3 种索引机制。然而经过生产实践，HBase 索引需要依赖外部的组件，内存索引可能存在 OOM 的问题，Bloom Filter  存在一定的误算率。

  我们研究发现，在 Hudi 写入的 parquet 文件中存在一个隐藏的列，通过读取这个列可以拿到文件中所有数据的主键，因此可以通过文件索引获取到数据需要写入的文件路径，并保存到 Flink 算子的 state 中，也避免了外部依赖和 OOM 的问题。

- **索引写入分离**

  原先 Hudi 的 Upsert 过程，写入和索引的过程是在一个算子中的，算子的并行度只由分区路径来决定。我们将索引和写入的过程进行分离，这样可以提高 Upsert 算子的并行度，提高写入的吞吐量。

- **故障恢复**

  最后我们将整个流程的状态保存到 Flink State 中，设计了一套基于 State 的故障恢复机制，可以保证端到端的 exactly-once 语义。

## 2. 实时数仓的产品化

在实时数仓产品化方面，我们也做了一些工作。提供了包括数据接入、元数据管理、数据处理在内的数仓开发套件。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9q3rYbZiaDdNWstfOqeDOhWhe4j6rzedmscdOaSMm2XcTrbNSRM8My9w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.1 实时数据接入

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9pnFXXZLMnsWmqGZz5j53VWtsUbAJtUOuUMfan5WZj1pnka0bPI7t3g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实时数据接入采用的是表单式的流程接入方式，屏蔽了复杂的底层技术，用户只需要经过简单的操作就可以将外部数据源接入到数仓体系。以 MySQL 为例，用户只需要选择 MySQL 数据源，平台就会自动抽取并展示 Schema ，用户确认 Schema 之后，就会将  Schema 插入到平台元数据中。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9K7RwfFvWmO5DFDciayNh7AHbiabpdJQvjhF1p9r5RSGV5XTEmwhZyibibA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

随后，用户选择有权限的集群，设置 Hive 表的主键 ID 和分区字段，提交申请之后，平台就会自动生成 Flink 任务，抽取数据到 Kafka 并自动落入 Hive  表中。对数据库类型的数据源，还支持分库分表功能，将分库分表的业务数据写入 ODS  层的同一张表。另外也支持采集主从同步的数据库，从从库中查询存量数据，主库拉取 Binlog，在减轻主库压力的同时降低数据同步延迟。

### 2.2 实时元数据更新

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9vuAhWiaxdicmmibO0DS6TbUt3fsxBdLK0iaiba36H5fFzUTmztyOp75VIlA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实时元数据更新的过程，还是以 MySQL 为例。CDC Source 会抽取数据库中的 Binlog ，区分 DDL 和 DML 语句分别处理，DDL  语句会上报到元数据中心，DML 语句经过转化变成 avro 格式的 Binlog 数据发送到 Kafka ，如果下游有写入到 Hive  的需求，就消费 Kafka 的数据通过 Hudi Sink 写入到 Hive 。

### 2.3 数据资产管理体系

基于实时数据的统一接入，并将其与现有的离线数仓结合，我们构建了数据资产管理体系。包括规范数仓标准，统一管理元数据，提升数据质量，保障数据安全，盘点数据资产。

## 3. 实时计算平台架构

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9deymoPntniaT4yQ6dUT1S675pxa2v3gGu6D3icUwzjUibBURO7kXzbTyQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有了数据统一接入的基础和数据资产资产管理体系的保驾护航，我们还需要一个数据开发套件，将整个数据开发的过程整合到实时计算平台。实时计算平台的最底层是数据接入层，支持 Kafka 和 Binlog 等数据源。上一层是数据存储层，提供了 Kafka 、ES、HBase、Hive、ClickHouse、MySQL 等存储组件。支持 JStorm 、Spark Streaming、Flink 计算引擎。并进行了框架封装和公共组件打包。

### 3.1 多种开发模式 - JAR & DRAG

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9icjaGicGH89URXEFrVg7YhBmbEiaLVxTRgKaoxp1XQs5omvFiaFpbo61jA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实时计算平台提供了多种开发模式供不同用户选择。以 Flink 为例，Flink JAR 模式由用户编写 Flink 任务代码，打成 jar 包上传到平台，满足高级用户的需求。Flink  DRAG 模式则是图形化的拖拽式开发，由平台封装好公共组件之后，用户只需要拖拽公共组件，将其组装成一个 Flink 任务，提交至集群运行。

### 3.2 多种开发模式 - SQL

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9cYsXFyFMgVzp1aVWzlk9naBYgMnLQJGxYTyxT1UAS1hj6ibqMcWWMLg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实时计算平台同样提供 SQL 开发模式，支持手动建表，根据元数据自动识别表及设置表属性。支持创建 UDF、自动识别 UDF、执行 DML 等。

### 3.3 任务管控

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9ecEibeUqbHVgPjpDAnVLjibhVcZgrBsNR4dLXbtMQ1WhjrWaLUzcPzuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在任务管控方面，实时计算平台尽量简化任务的配置，屏蔽了一些复杂的配置。用户开发完成之后，只需要选择集群，填写资源，就能将任务提交到集群中运行。对每个任务，平台还提供了历史版本控制能力。

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9DjY2edBKNe4D3HIQak1CHR2QeSBqkqFtw26RMjP8BfYaQGZS7u8oWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当用户操作任务时，平台会自动解析任务的配置，根据不同的组件提供不同的选项。比如选择了 Kafka 数据源，启动的时候，可以选择从上次消费位置、最早位置、最新位置或指定位置启动。

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR97kZj3PYy3KJff2gibLrIibrQia9TKDM9e80RQFbwOKKJrMXOfJPNB4GyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

任务恢复方面，用户可以选择从 Savepoint 启动已停止的 Flink 任务，便于快速恢复历史状态。

### 3.4 任务运维

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9Xpbqsiaich76jY81c5Kxs5hHRnH1jtU2oxgDkibIgL67p4Dk9Nuh2iapSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对实时任务来说，任务运维是一个难点也是一个痛点。平台提供了日志查询功能，采集历史的启动日志和任务运行日志，用户可以方便的进行对比和查询。

当任务启动之后，平台会自动采集并上报任务的指标，用户可以根据这些指标自定义告警配置，当告警规则被触发时，平台会通过各种方式告警到用户。最后，平台提供了指标的实时监控看板，当然用户也可以自行在 Grafana 中配置监控看板。

通过采集日志、指标以及监控告警，以及过往的历史经验，我们实现了一个智能的机器客服，可以实现任务故障的一些自助诊断。这些举措大大降低了任务的运维成本，减轻平台研发人员的压力。

### 3.5 Flink 任务稳定性保障

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9FnBYwAH8lQlaPwUVx2LVfp74T3yJ1Yrz9Tqg6osF5aicAaPcE6CUOAg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实时作业运维最关注的是稳定性，在保障 Flink  任务稳定性上我们也有一些实践。首先提供多种异常检测和监控告警的功能，方便用户快速的发现问题。每个任务都会定时的生成任务快照，保存任务历史的  Savepoint，以方便任务回滚和故障恢复。任务可能会由于某种异常原因导致任务失败，任务失败之后会被平台重新拉新，并指定从上次失败的位置开始重新消费。

基于 Zookeeper 的高可用机制，以保障 JobManager 的可用性。支持多集群、多机房的容灾切换能力，可以将任务一键切换至容灾集群上运行。实现了一套实时离线集群隔离、队列管理的资源隔离系统。

# 四、应用案例

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9BYN8pyMMVcpaKqNpDXLSZpH4jsm7PiabAX2WEyib20aexVNL2GsZXLWA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

以业务宽表计算为例，需要获取 45 天内的多个数据源的数据，进行计算聚合。如果使用离线数仓，大概需要 3000 核的 CPU、12000G 的内存，耗时 120 ～ 150 min 完成计算，处理的数据量大概为 450T。如果使用实时数仓，大概需要 2500 核的 CPU、1400G 的内存，更新宽表大概有 2～5 min 的延时，处理的数据量约为 18T。

# 五、未来规划

顺丰的实时数仓建设取得了一些成果，但未来仍需要进行不断的优化。

## 1. 增强 SQL 能力

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9sxqJ7CjKksZnUjFo6MBn09cJSLRGBld5yGeSr3YWnOk9UKbUrXJy3w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先，希望能够支持更多 SQL 的语法和特性，支持更多可用的连接器，以及实现 SQL 任务的自动调优等。

## 2. 精细化资源管理

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9zthicfSsibX0YpJqexHTgiajicQmkKcibh4ARKoTlqxEFXVHt9XH7bJSd5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其次，基于 Flink On Kubernets 、任务的自动弹性扩缩容，Task 级别的细粒度资源调度实现精细化的资源调度管理，使得 Flink 任务达到全面的弹性化和云原生化。

## 3. 流批一体

![图片](https://mmbiz.qpic.cn/mmbiz_png/8AsYBicEePu4VicWj32pBjoDqLBjX0biaR9TYLmEMicnK9uHjgqrichrd8p7mTYiaqJUfrZqObrWTDUctxOdZcRfRJZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后，希望能够实现流批一体，通过统一的高度兼容性的 SQL ，经过 SQL 解析以及引擎的适配，通过 Flink 统一的引擎去处理流和批。

