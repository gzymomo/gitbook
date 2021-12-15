# Apache Hudi：新一代流式数据湖平台

以下文章来源于InfoQ ，作者Vinoth Chandar                                                           

早在 2016 年，我们就提出了一个大胆的新愿景 [1]，通过一个新的“**增量**”数据处理技术栈（结合现有的批处理和流式处理堆栈）重新构想批处理。虽然流处理管道进行面向行的处理，提供秒级处理延迟，但增量管道将对数据湖中的列数据应用相同的原则，高效的数据处理，及相对批处理数量级的改进，同时存储 / 计算可高度扩展。这个新的技术栈将能够毫不费力地支持批量再加工 / 回填的常规处理。Apache Hudi 是作为这一愿景的体现而建立的，它植根于 Uber 面临的真实、困难的问题 [2]，后来在开源社区中独树一帜。总之，我们已经能够在数据湖上引入完全增量的数据摄取和中等复杂的 ETL。



![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKuQb78z1kNsvZQUOgiccsdOlZ6LQ3zOuKkTVm0iblibjwouUWCJkticUdVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

今天，能够以增量方式表达几乎任何批处理管道的宏伟愿景比以往任何时候都更容易实现。流处理变得越来越成熟 [3]，发展势头也很迅猛 [4]，流处理 API 已被推广 [5] 到批处理执行模型上。Hudi 填补了增量处理的空白并提供了高效的面向流式优化的数据湖存储，与 Kafka/Pulsar 为事件流提供高效存储非常类似。数据湖中的流式处理使得许多组织 [6] 能够获取更新鲜的数据、更简化的体系结构和更低的成本。

但首先我们需要解决数据湖的基础问题——事务和可变性。在许多方面，Apache Hudi 开创了我们今天所知的事务性数据湖的先河。具体来说，在更多专用系统诞生的时候，Hudi 引入了一个无服务器的事务层，它工作在云存储 /HDFS 的通用 Hadoop 文件系统抽象之上。该模型帮助 Hudi 在构建之初就能够将写入器 / 读取器扩展到 1000 个内核，相比之下，传统数仓提供了更丰富的事务保证集，但往往受限于处理它们的 10 多个服务器的规模瓶颈。我们也很高兴看到类似的系统（例如 Delta Lake）后来采用了我们最初在 17 年初共享的无服务器事务层模型 [7]。我们有意识地引入了两种表类型 Copy On Write（具有更简单的可操作性）和 Merge On Read（以获得更大的灵活性），现在这些术语也用于 Hudi 之外的项目 [8]，指代从 Hudi 借用的类似想法。通过开源和从 Apache 孵化器毕业 [9] ，我们在将这些想法提升到整个行业 [10] 高度并通过有凝聚力的软件堆栈将它们变为现实方面取得了一些重大进展。鉴于过去一年左右令人兴奋的发展推动了数据湖进一步成为主流，我们认为一些观点可以帮助用户以正确的视角看待 Hudi，了解它代表什么，并成为其发展方向的一部分。此时，我们还想展示 180 多个贡献者 [11] 在该项目上所做的所有伟大工作，他们与 2000 多个独立用户通过 slack/github/jira 合作，贡献了 Hudi 在过去一年获得的所有功能。

这将是一篇相当长的文章，但我们会尽最大努力让它值得你花时间好好品读。来吧。

# 数据湖平台

我们注意到，Hudi 有时被定位为“表格格式”[12] 或“事务层”。虽然这没有错，但并不能完全体现 Hudi 所提供的一切。

## Hudi 是一种“格式”吗?

Hudi 并非设计为通用表格格式，用于跟踪文件 / 文件夹以进行批处理。相反，表格格式提供的功能只是 Hudi 软件堆栈中的一层。鉴于 Hive 格式的广泛流行，Hudi 旨在与 Hive 格式配合得很好。随着时间的推移，为了解决“扩展性”的挑战或引入额外的功能，我们致力于构建自己的原生表格格式，着眼于增量处理的愿景。例如，我们需要支持每隔几秒提交一次的较短事务。我们相信，随着时间的推移，这些要求将完全包含通用表格格式所要解决的挑战。但是，我们也愿意与其他开放表格格式交互，因此其他表格格式的用户也可以从 Hudi 堆栈的其余部分中受益。与文件格式不同，表格格式只是表格元数据的表示，如果用户愿意接受权衡，实际上用户可以从 Hudi 转换为其他格式 / 反之亦然。

## Hudi 是一个事务层吗?

当然，Hudi 必须提供事务来实现删除 / 更新，但 Hudi 的事务层是围绕事件日志 [13] 而设计的，该日志也与一整套内置表 / 数据服务很好地集成。例如，压缩知道已经调度的聚簇 (Clustering) 操作并通过跳过正在聚簇 (Clustering) 的文件进行优化 - 而用户幸福地不知道这一切。Hudi 还提供了用于摄取、ETL 数据等开箱即用的工具。我们一直认为 Hudi 是在围绕流处理解决数据库问题——实际上两者是非常相关的领域 [14]。事实上，流处理是由日志（捕获 / 发出事件流、回绕 / 重新处理）和数据库（状态存储、可更新的接收器）启用的。有了 Hudi 这样的框架，如果我们想构建一个支持高效更新和提取数据流的数据库，同时保持针对大批量查询的优化，则可以使用 Hudi 表作为状态存储和可更新接收器来构建增量管道。

因此，将 Apache Hudi 描述为围绕数据库内核构建的**流式数据湖平台（Streaming Data Lake Platform）**是最佳方式。这个定义中有几个关键词非常重要。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOK86EM3S94rXtB8iaA7fFrghhBLGhslBicB849icL8zS0WYZialegPMoIkAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Streaming**：其核心在于通过优化快速更新（upsert）和更改流（change stream），Hudi 为数据湖工作负载提供了原语，这些原语与 Apache Kafka[15] 为事件流所做的相当（即事件的增量生产 / 消费和用于交互式查询的状态存储） 。

**DataLake**：尽管如此，Hudi 为湖上（可以说是世界上最大的事务湖 [16]）大规模数据处理（即席查询、ML 管道、批处理管道）提供了一个优化的、自我管理的数据平面。虽然 Hudi 可用于构建 LakeHouse[17]，但鉴于其事务能力，Hudi 能做的不止于此，并进一步解锁了端到端的流处理架构。相比之下，“Streaming”这个词在 LakeHouse 论文 [18] 里只出现了 3 次，其中一次还是在谈论 Hudi 时提到的。

**Platform**：开源世界里有很多很棒的技术，但有点太多了——所有这些技术在各自领域都略有不同，最终使终端用户的集成任务变得繁重。数据湖用户应该获得与云数仓相同且出色的可用性，以及开源社区真正的额外自由和透明度。Hudi 的数据和表格服务与 Hudi“内核”紧密集成，使我们能够以可靠性和易用性提供跨层的优化。

# Hudi 组件栈

以下堆栈描绘了构成 Hudi 的软件组件层，每一层都依赖于并从下一层汲取力量。通常，数据湖用户将数据写入开放的文件格式（如 Apache Parquet[19] / ORC[20]），这些文件格式存储在高度可扩展的云存储或分布式文件系统之上。Hudi 提供了一个自管理的数据平面来摄取、转换和管理这些数据并解锁了对它们进行增量处理的方式。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOK6MnyQT34vniaDnF10OwuqZ9NmiaR1B90FS6Zwqgcj26JEQ3gZM59qXgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此外，Hudi 已经提供或计划添加组件以使所有不同的查询引擎都普遍可以访问这些数据。带 * 注释的功能代表正在进行的工作，虚线框代表规划中未来将展开的工作。虽然我们在博客中为新组件概述了假想的设计，但我们也非常欢迎来自社区的新观点。博客的其余部分将深入研究堆栈中的每一层 - 解释它的作用、它如何设计用于增量处理以及它将如何发展。

## 湖存储

Hudi 使用 Hadoop FileSystem API[21] 与湖存储交互，这使其能够兼容从 HDFS 到云存储甚至内存文件系统（如 Alluxio[22]/Ignite）的所有实现。Hudi 内部实现了自己包装过的文件系统 [23]，以提供额外的存储优化（例如：文件大小）、性能优化（例如：缓冲）和指标体系。值得一提的是，Hudi 充分利用了像 HDFS 之类的存储模式所支持的“append"特性。这有助于 Hudi 提供流式写入，而不会导致文件计数 / 表元数据激增。不幸的是，目前大多数云 / 对象存储都不提供“append”功能（Azure 除外 [24]）。未来我们计划利用主流云对象存储的低级 API，在流式摄取延迟时提供对文件计数的类似控制。

## 文件格式

Hudi 是围绕基本文件和增量日志文件的概念设计的，它们将更新 / 增量数据存储到给定的基本文件（称为文件片，file slice）。它们的格式是可插拔的，目前支持的基本文件格式包括 Parquet（列访问）和 HFile（索引访问）。增量日志以 Avro[25]（面向行）格式对数据进行编码，以实现更快的日志记录（就像 Kafka topic 一样）。展望未来，我们计划在即将发布的版本中将每种基本文件格式内联到日志块中 [26]，根据块大小提供对增量日志的列式访问。未来的计划还包括支持 ORC 基础 / 日志文件格式、非结构化数据格式（自由的 json 格式、图像），甚至使用事件流系统 /OLAP 引擎 / 数仓的分层存储层的原生文件格式。

从另外一方面看，Hudi 独特的文件布局方案将给定基础文件的所有更改编码为一系列块（数据块、删除块、回滚块），这些块被合并以派生出更新的基础文件。本质上这构成了一个自包含的重做日志，可以在上面实现有趣的功能。例如，当今的大多数数据隐私保护条例都是通过在读取湖中数据时进行动态编码的机制，在同一组记录上一遍又一遍地调用散列 / 加密算法并产生大量计算开销 / 成本来实现的。基于 Hudi 的这种设计，用户将能够在日志中保留相同键的多个预先编码 / 加密的副本，并根据策略分发正确的副本，从而避免以上提及的开销。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKicrpkH887MzUgxznPxvYd9OXJqTDGbYI5jM5orPJiaRicbeYGESdtwUGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 表格式

“表格式”这个术语比较新，对很多人来说同时意味着很多东西。与文件格式类比，表格式仅包括：表的文件布局、表的 schema 和对表更改的元数据跟踪。Hudi 并不将自己定位为一种表格式，不过它自身确实内置实现了一种格式。Hudi 使用 Avro 模式来存储、管理和演进表的 schema。目前 Hudi 强制执行 schema-on-write，虽然比 schema-on-read 更严格，但在流处理领域被广泛采用 [27]，以确保管道不会因无法向后兼容的变更而中断。

Hudi 有意识地将表 / 分区中的文件分组，并维护摄入记录的键与现有文件组之间的映射。所有更新都记录到特定于给定文件组的增量日志文件中，与 Hive ACID 等实现机制相比，这种设计确保了较低的合并开销，后者必须针对所有基本文件和所有增量记录来评估合并才能满足查询。例如，使用 uuid 做主键（场景非常广泛）所有基本文件很可能与所有增量日志所包含的记录重叠，从而使任何基于范围的剪枝优化变得无用。与状态存储非常相似，Hudi 的设计理念基于键的快速 upserts/deletes，并且只需要在每个文件组中合并 delta 日志。这种设计选择还让 Hudi 为写入 / 查询提供了更多的能力，我们将在下面解释。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOK17A6XwNMmy8BojJ9gyxX3EaPYtzk7olK7EAqMMgswhWkEUOYnk5hxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

时间线是所有 Hudi 表元数据的真实事件日志，存储在 .hoodie 文件夹下，提供对表执行的所有操作的有序日志。事件将保留在时间线上，直至配置的时间 / 活动间隔。每个文件组也专门设计了自包含的日志，这意味着即使影响文件组的操作从时间轴存档，每个文件组中的记录的正确状态也可以通过简单地在本地应用 delta 记录到基本文件来重新构建。这种设计限制了元数据大小，它只与表被写入 / 操作的频率成正比，与整个表的大小无关。这是支持频繁写入 / 提交表的关键设计元素。

最后，时间线上的新事件被消费并反映到内部元数据表上，元数据表实现为另一个提供低写入放大的 MOR 表。Hudi 能够支持对表元数据的快速更改，这与专为缓慢变动数据设计的表格式不同。此外，元数据表使用 HFile 基本文件格式 [28]，它提供键的索引查找，避免了不必要的全表扫描。它当前存储作为表的所有物理文件路径相关的元数据，以避免昂贵的云文件 listing。

今天所有表格式面临的一个关键挑战是需要决策快照过期以及控制时间旅行查询（time travel queries）的保留时长，以便它不会干扰查询计划 / 性能。未来，我们计划在 Hudi 中构建一个索引时间线，它可以跨越表的整个历史，支持几个月 / 几年的时间旅行回顾窗口。

## 索引

索引可帮助数据库规划更好的查询，从而减少 I/O 总量并提供更快的响应时间。有关表文件 listing 和列统计信息的表元数据通常足以让湖上的查询引擎快速生成优化的、引擎特定的查询计划。然而，这还不足以让 Hudi 实现快速插入。

目前 Hudi 已经支持不同的基于主键的索引方案，以快速将摄入的记录键映射到它们所在的文件组中。为此，Hudi 为写入器引入了一个可插拔的索引层，内置支持范围剪枝（当键被排序并且记录在很大程度上按顺序到达时）使用间隔树（interval trees）和布隆过滤器（例如：对基于 uuid 的键，排序几乎没有帮助）。Hudi 还实现了一个基于 HBase 的外部索引，虽然运行成本更高，但性能更好，同时支持用户自定义索引实现。Hudi 也有意识地利用表的分区信息来实现全局和非全局的索引方案。用户可以选择仅在一个分区内强制执行键约束，以换取 O(num_affected_partitions) 复杂度的 upsert 性能，而不是全局索引场景中的 O(total_partitions) 复杂度。

我们向您推荐这篇博客 [29]，它详细介绍了索引相关的内容。最终，Hudi 的写入器确保索引始终与时间轴和数据保持同步，而这如果在表格式之上手动实现既麻烦又容易出错。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKKCiaj1XVeuJ0jibpgPwzDeGvXtvaFH3olFul7ZWjYUJpsWMsORqib96cw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

将来，我们打算添加其他形式的索引作为元数据表上的新分区。让我们简要讨论一下每个部分所扮演的角色。

查询引擎通常依靠分区来减少读取的文件数量。在数据库方面，Hive 分区只不过是一个粗略的范围索引，它将一组列映射到一个文件列表。像 Iceberg/Delta Lake 这些在云中诞生的表格式，在单个文件 (json/avro) 中内置了对每个文件的列范围跟踪，这有助于避免大型 / 小型表格的规划成本。到目前为止，Hudi 表的这种需求已大大减少，因为 Hudi 会自动强制执行文件大小，这有助于降低从 Parquet 页脚读取统计信息所需的时间，例如，随着聚簇（Clustering）等功能的出现，可以先生成较小的文件，然后以查询优化的方式重新聚簇。我们计划添加索引列范围，它可以扩展到大量小文件并支持更快的变更。请参阅 RFC-27[30] 以跟踪设计过程并参与其中。

虽然 Hudi 已经针对“随机写”类型的工作负载提供了外部索引，但我们还希望在湖存储之上支持“点查”[31]，这有助于避免为许多类数据应用程序添加额外的数据库存储开销。我们还预计利用记录级索引的方案后，基于 uuid/key 的 join 以及 upsert 的性能将大大加快。我们还计划将对布隆过滤器的跟踪从文件页脚移到元数据表上的分区中 [32]。最终，我们希望在即将发布的版本中将所有这些都暴露给查询。

## 并发控制

并发控制定义了不同的写入器 / 读取器如何协调对表的访问。Hudi 确保原子写入，通过将提交原子地发布到时间线，并标记一个即时时间（instant），以标记该操作具体发生的时间。与通用的基于文件的版本控制不同，Hudi 明确区分了写入进程（执行用户的更新、插入 / 删除）、表服务（写入数据、元数据以优化执行所需的信息）和读取器（执行查询和读取数据）。Hudi 在所有三种类型的进程之间提供快照隔离，这意味着它们都对表的一致快照进行操作。Hudi 在写入器之间提供乐观并发控制 (OCC)[33]，同时在写入器和表服务之间以及不同表服务之间提供基于无锁、非阻塞的 MVCC 的并发控制。

完全依赖 OCC 的项目通常通过依赖锁或原子重命名来处理“竞争”操作。这种方法乐观地认为，真正的资源争用永远不会发生，而如果发生冲突，就会使其中一个写入操作失败，这可能会导致严重的资源浪费或操作开销。

想象一下有两个写入器进程的场景：一个每 30 分钟生成一次新数据的摄取写入器作业和一个需要 2 小时才能发出的执行 GDPR 的删除写入器作业。如果在相同的文件上有重叠（在随机删除的真实情况下很可能发生），删除作业几乎肯定会引发“饥饿”并且每次都无法提交，浪费大量的集群资源。Hudi 采用了一种非常不同的方法，我们认为这种方法更适用于湖的事务，这些事务通常是长期运行的。例如，异步压缩可以在不阻塞摄取作业的情况下在后台持有删除记录。这是通过文件级、基于日志的并发控制协议实现的，该协议根据时间线上的开始 instant 对操作进行排序。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKHfHAMNHYg9iasBy21SJo89mIsl6Pa01xPGxt3Xjw6gic5HRZgDC9CWKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们正在改进当前基于 OCC 的实现，以尽早检测并发写入器的冲突并在不消耗 CPU 资源的情况下提前终止冲突。我们还尽力在写入器之间添加完全基于日志 [34] 的非阻塞并发控制，其中写入器持续写入增量，稍后以某些确定性时间线顺序解决冲突——这也与流处理程序的写入方式非常相似。这仅是因为 Hudi 的独特设计将操作排序到有序的事件日志中，并且事务处理代码知道操作之间的关系 / 相互依赖关系。

## 写入器

Hudi 表可以用作 Spark/Flink 管道的接收器，并且 Hudi 写入实现提供了比原生的 parquet/avro sinks 更多的功能。Hudi 将写入操作细分为增量（插入、更新插入、删除）和分批 / 批量操作（插入覆盖、插入覆盖表、删除分区、批量插入），每个操作都是高性能和高内聚的。upsert 和 delete 操作都会自动处理输入流中具有相同键的记录的预合并（例如，从上游表获得的 CDC 流），然后查找索引，最后调用二进制打包算法将数据打包到文件中，同时遵循预配置的目标文件大小 [35]。另一方面，插入操作足够智能，可以避免预合并和索引查找，同时保留管道其余部分的优势。同样，“批量插入” 操作提供了多种排序模式，用于在将数据从外部表导入 Hudi 时控制初始文件大小和文件计数。其他批量写入操作提供了批量数据管道中使用的典型覆盖语义的基于 MVCC 实现，同时保留了所有事务性和增量处理功能，使得在用于常规运行的增量管道和用于回填 / 删除旧分区的批量管道之间无缝切换。写管道还包含通过溢出到 RocksDB[36] 或外部可溢出映射、多线程 / 并发 I/O 来处理大型合并的优化，以提高写入性能。

主键是 Hudi 中的一等公民，并且在更新插入 / 删除之前完成的预合并 / 索引查找可确保键在分区之间或分区内是唯一的。与其他由数据工程师使用 MERGE INTO 语句进行协调的方法相比，这种方法对于关键的用例场景可确保数据质量。Hudi 还附带了几个内置的主键生成器 [37]，可以解析所有常见的日期 / 时间戳，使用自定义的主键生成器的可扩展框架处理格式错误的数据。主键将随记录一起物化到 _hoodie_record_key 元数据列，这允许更改主键字段并对旧数据里不正确的主键数据进行修复。

最后，Hudi 提供了一个与 Flink 或 Kafka Streams 中的处理器 API 非常相似的 HoodieRecordPayload 接口，并允许在基本和增量日志记录之间表达任意合并条件。这允许用户表达部分合并（例如，仅将更新的列记录到增量日志以提高效率）并避免在每次合并之前读取所有基本记录。通常，我们发现用户在将旧数据重放 / 回填到表中时会利用这种自定义合并逻辑，同时确保不会覆盖较新的更新，从而使得表的快照能及时返回。这是通过简单地使用 HoodieDefaultPayload 来实现的，它根据数据中配置的预合并字段来选择给定键的最新值。

Hudi 写入器向每条记录添加元数据，该元数据为每个提交中的每条记录编码提交时间和序列号（类似于 Kafka 偏移量），这使得派生记录级别的更改流成为可能。Hudi 还为用户提供了在摄入数据流中指定事件时间字段并在时间轴中跟踪它们的能力。将这些映射到流处理领域的概念，Hudi 包含每个提交记录的到达时间和事件时间 [38]，这可以帮助我们为复杂的增量处理管道构建良好的水印 [39]。

在我们着手实现完整的端到端增量 ETL 管道这一宏伟目标之前，我们希望在不久的将来添加新的元数据列，用于对每条记录的源操作（插入、更新、删除）进行编码。总而言之，我们意识到许多用户可能只是想将 Hudi 用作支持事务、快速更新 / 删除的高效写入层。我们正在研究添加对虚拟主键 [40] 的支持并使元数据列可选 [41]，以降低存储开销，同时仍然使 Hudi 的其余功能（元数据表、表服务等）可用。

## 读取器

Hudi 在写入器和读取器之间提供了快照隔离，并允许所有主流的湖查询引擎（Spark、Hive、Flink、Presto、Trino、Impala）甚至云数仓（如 Redshift）在任何表快照上进行一致的查询。事实上，我们也很乐意将 Hudi 表作为外部表与 BigQuery/Snowflake 一起使用，前提是它们也更原生地支持湖的表格式。

我们围绕查询性能的设计理念是在仅读取基本列式文件（COW 快照表、MOR 读优化查询表）时使 Hudi 尽可能轻量，例如在 Presto、Trino、Spark 中使用引擎特定的向量化读取器。与维护我们自研的读取器相比，此模型的可扩展性要高得多。例如 Presto[42]、Trino[43] 都有自己的数据 / 元数据缓存。每当 Hudi 必须为查询合并基础文件和日志文件时，Hudi 都会进行控制并采用多种机制（可溢出的映射、延迟读取）来提高合并性能，同时还提供对数据的读优化查询，以权衡数据新鲜度与查询性能。

在不久的将来，我们将深度投入以通过多种方式提高 MOR 表的快照查询性能，例如内联 Parquet 数据、覆盖有效负载 / 合并的特殊处理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKlcCmq2dPV6yop63CyMMc2LCEHiaGkhibwCI24Rkjg1sTmmLHTpBicKIWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

忠于其设计目标，Hudi 提供了一些非常强大的增量查询功能，将写入过程中添加的元字段和基于文件组的存储布局联系在一起。虽然仅跟踪文件的表格式，只能提供在每个快照或提交期间更改的文件的信息，但由于跟踪了记录级事件和到达时间，Hudi 会生成给定时间线上某个时间点更改的确切记录集。此外，这种设计允许增量查询在较大数据量的提交中获取较小的数据集，完全解耦数据的写入和增量查询。时间旅行基于增量查询实现，该查询作用在时间线上过去的部分范围。由于 Hudi 确保在任何时间点将主键原子地映射到单个文件组，因此可以在 Hudi 表上支持完整的 CDC 功能，例如提供自时间 t 以来给定记录的所有可能值在 CDC 流上的“before”和“after”视图。所有这些功能都可以在每个文件组的本地构建，因为每个文件组都是一个独立的日志。我们未来在该领域的大部分工作是期望把类似 debezium[44] 所具备的能力带入到 Hudi 中来。

## 表服务

多年来定义和维持一个项目价值的通常是其基本设计原则和微妙的权衡。数据库通常由多个内部组件组成，它们协同工作以向用户提供效率、性能和出色的可操作性。为了让 Hudi 能作为增量数据管道的状态存储，我们为其设计了内置的表服务和自我管理运行时，可以编排 / 触发这些服务并在内部优化一切。事实上，如果我们比较 RocksDB（一种非常流行的流处理状态存储）和 Hudi 的组件，相似之处就很明显了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKKvyu2lUVyLW42pqZqSoYXuTTv3MTvfxKZ0SqWpSBOSm6iaYicq3E0A8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hudi 有几个内置的表服务，目标都是确保高性能的表存储布局和元数据管理，它们在每次写入操作后同步自动调用，或者作为单独的后台作业异步调用。此外，Spark（和 Flink）流写入器可以在“连续”模式下运行，并调用表服务与写入器智能地异步共享底层执行器。

**归档**（Archival）服务确保时间线为服务间协调（例如压缩服务等待其他压缩服务在同一文件组上完成）以及增量查询保留足够的历史记录。一旦事件从时间线上过期，归档服务就会清除湖存储的任何副作用（例如回滚失败的并发事务）。Hudi 的事务管理允许所有这些服务都是幂等的，因此只需简单的重试即可对故障容错。

**清理**（Cleaner[45]）服务以增量的方式工作（吃我们自己的增量设计狗粮），删除超过保留期限的用于增量查询的文件切片，同时还为长时间运行的批处理作业（例如 Hive ETL）留出足够的时间来完成运行。

**压缩**（Compaction）服务带有内置策略（基于日期分区，I/O 有界），将基本文件与一组增量日志文件合并以生成新的基本文件，同时允许对文件组进行并发写入。这得益于 Hudi 将文件分组并支持灵活的日志合并，同时在并发更新对同一组记录出现问题时执行非阻塞删除。

**聚簇**（Clustering[46]）服务的功能类似于用户在 BigQuery 或 Snowflake 中找到的功能，用户可以通过排序键将经常查询的记录组合在一起，或者通过将较小的基本文件合并为较大的文件来控制文件大小。**聚簇**能完全感知时间线上的其他操作（例如清理、压缩），并帮助 Hudi 实施智能优化，例如避免对已聚簇的文件组进行压缩，以节省 I/O。Hudi 还通过使用标记文件跟踪作为写入操作的一部分创建的其他文件，以执行对写入中断的回滚并清除湖存储中的任何未提交的数据。

最后，**引导**（Bootstrap）服务将原始 Parquet 表一次性零拷贝迁移到 Hudi，同时允许两个管道并行运行，以进行数据验证。清理服务一旦意识到这些已被引导的基本文件，可以选择性地清理它们，以确保满足 GDPR 合规性等需求场景。

我们一直在寻找以有意义的方式改进和增强表服务。在即将发布的版本中，我们正在努力建立一个更具可扩展性 [47] 的清理“部分写入”的模型，方法是使用我们的时间轴元服务器注册标记文件的创建，从而避免昂贵的全表扫描来寻找和删除未提交的文件。我们还有各种提议 [48] 来添加更多的**聚簇**方案，使用完全基于日志的并发控制以通过并发更新来解锁**聚簇**。

## 数据服务

正如本文最初所提及的，我们想让 Hudi 能对常见的端到端用例做到开箱即用，因此对一组数据服务进行了深度投入，这些服务提供了特定于数据 / 工作负载的功能，位于表服务之上，直面写入器和读取器。

该列表中最重要的是 Hudi DeltaStreamer 实用程序，它一直是一个非常受欢迎的选择，可以轻松地基于 Kafka 流以及在湖存储之上的不同格式的文件来构建数据湖。随着时间的推移，我们还构建了涵盖所有主流系统的源，例如 RDBMS/ 其他数仓的 JDBC 源、Hive 源，甚至从其他 Hudi 表中增量提取数据。该实用程序支持检查点的自动管理、跟踪源检查点作为目标 Hudi 表元数据的一部分，并支持回填 / 一次性运行。DeltaStreamer 还与 Confluent 等主要 Schema 注册表进行了集成，并提供了对其他流行机制（如 Kafka connect）的检查点转换。它还支持重复数据删除、多级配置管理系统、内置转换器，可以将任意 SQL 或强制 CDC 变更日志 [49] 转换为可写的形式，结合上述其他功能足以使其用于部署生产级的增量管道。

最后，就像 Spark/Flink 流写入器一样，DeltaStreamer 能够以连续模式运行，自动管理表服务。Hudi 还提供了其他几种工具，用于以一次性快照和增量的形式导出 Hudi 表，还可以将新表导入 / 导出 [50]/ 引导到 Hudi。Hudi 还向 Http 端点或 Kafka topic 提供关于表提交操作的通知，可用于分析或在工作流管理器（如 Airflow）中构建数据传感器以触发管道。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKMNqWksdPY6w4uPkUtY4GictLsWaS0YdbgRjB6icn7EBHDHQhk2IlZ8sQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

展望未来，我们希望为增强多表写入器 [51] 实用程序做出贡献，该实用程序可以在单个大型 Spark 应用程序中摄取整个 Kafka 集群。为了促进端到端的复杂增量管道向前发展，我们计划努力增强由多源流（而不是今天的单源流）触发的 delta streamer 实用程序及其 SQL 转换器，并大规模解锁物化视图。我们希望带来一系列有用的转换器来执行处理或数据监控，并扩展对 Hudi 表中数据输出到其他外部接收器的支持。最后，我们希望将 FlinkStreamer 和 DeltaStreamer 实用程序合并为一个可跨引擎使用的内聚实用程序。我们不断改进现有源（例如支持 DFS 源的并行 listing）并添加新的源（例如基于 DFS 的 S3 事件源）

## 时间线元数据服务

在湖存储上保存和提供表元数据是可扩展的，但与可扩展的元数据服务器的 RPC 相比，其性能可能要低得多。在大多数云仓库内部，其元数据层都建立在外部数据库之上（例如 Snowflake 使用 FoundationDB[52]）。Hudi 还提供了一个元数据服务器，称为“时间线服务器”（Timeline server），它为 Hudi 表的元数据提供了一个可替代的后备存储。

目前，时间线服务器内嵌在 Hudi 写入器进程中运行，在写入过程中从本地 RocksDB 存储 /Javalin[53] REST API 获取文件列表，无需重复在云存储上执行“listing”操作。鉴于自 0.6.0 版本以来我们已将此作为默认选项进行，我们正在考虑拆分时间线服务器进行独立部署以支持水平扩展、数据库 / 表映射、安全性，以将其转变为功能完备的高性能的湖元数据存储。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKCJL4evbwFQicwBAzJqwCydkDBt2UdvWXHIouEPjjEfZzoPYYicJrPmkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 湖缓存

如今，数据湖在更快的写入和出色的查询性能之间存在一个基本的权衡。更快的写入通常涉及写入较小的文件（然后将它们聚簇）或记录增量（然后在读取时合并）。虽然这已经提供了良好的性能，但追求出色的查询性能通常需要在湖存储上打开更少数量的文件 / 对象，并且可能需要预先实现基本文件和增量日志之间的合并。毕竟，大多数数据库都使用缓冲池 [54] 或块缓存 [55] 来分摊访问存储的成本。

Hudi 已经包含了一些有助于构建缓存层（直写或仅由增量查询填充）的设计元素，它将支持多租户，可以缓存最新文件分片的预合并镜像，并与时间线保持一致。Hudi 时间线可用于简单地与缓存策略通信，就像我们如何执行表间服务协调一样。从历史上看，缓存更接近查询引擎或通过中间内存文件系统完成。通过将缓存层与 Hudi 等事务性的湖存储更紧密地集成在一起，所有查询引擎都能够共享和分摊缓存成本，同时还支持更新 / 删除。我们期待在社区其他成员的贡献下，为湖建立一个可以跨所有主要引擎工作的缓冲池。

![图片](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOdbia8XTaAyibCibjZ8CxkGOKHofgSrpXpcOzXaa0mJic0uKjsnJIA0khCpjuMCU15Bk8XXeibooYfu2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 写在最后

这篇博客文章描绘了 Apache Hudi 的完整愿景。未来几周 / 几个月内我们计划发表博客文章深入讨论堆栈中的每一层，并沿着这些方向对我们的文档进行的大规模重构，感兴趣的用户和读者可以期待一下。

我们认为当前围绕表格式所做的努力仅仅是消除了数据湖存储 / 查询平面中存在了十年的瓶颈，这些问题已经在 Big Query/Snowflake 等云仓库中得到了很好的解决。我们想强调的是，Hudi 的愿景要大得多，在技术上也更具挑战性。作为这个领域中的一员，我们在思考许多更深刻的、开放式的问题以及怎样解决这些问题，希望在兼顾扩展性与简洁性的同时将流处理和数据湖结合起来。我们将继续把社区放在首位，与社区伙伴共同构建 / 解决这些难题。如果这些挑战让你感到兴奋，并且你也想为那个激动人心的未来而努力，请加入我们 [56]。

[1]: https://www.oreilly.com/content/ubers-case-for-incremental-processing-on-hadoop/

[2] https://eng.uber.com/uber-big-data-platform/

[3] https://flink.apache.org/blog/

[4]https://www.confluent.io/blog/every-company-is-becoming-software/

[5] https://flink.apache.org/2021/03/11/batch-execution-mode.html

[6] https://hudi.apache.org/docs/powered_by.html

[7] https://eng.uber.com/hoodie/

[8] https://github.com/apache/iceberg/pull/1862

[9]https://blogs.apache.org/foundation/entry/the-apache-software-foundation-announces64

[10] http://hudi.apache.org/docs/powered_by.html

[11] https://github.com/apache/hudi/graphs/contributors

[12]https://cloud.google.com/blog/products/data-analytics/getting-started-with-new-table-formats-on-dataproc

[13]https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying

[14] https://www.infoq.com/presentations/streaming-databases/

[15] https://kafka.apache.org/

[16] https://eng.uber.com/apache-hudi-graduation/

[17]https://databricks.com/blog/2020/01/30/what-is-a-data-lakehouse.html

[18] http://cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf

[19] http://parquet.apache.org/

[20] https://orc.apache.org/

[21]https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html

[22]https://www.alluxio.io/blog/building-high-performance-data-lake-using-apache-hudi-and-alluxio-at-t3go/

[23]https://github.com/apache/hudi/blob/9d2a65a6a6ff9add81411147f1cddd03f7c08e6c/hudi-common/src/main/java/org/apache/hudi/common/fs/HoodieWrapperFileSystem.java

[24]https://azure.microsoft.com/en-us/updates/append-blob-support-in-azure-data-lake-storage-is-now-generally-available/

[25] http://avro.apache.org/

[26] https://github.com/apache/hudi/pull/3228

[27]https://docs.confluent.io/platform/current/schema-registry/avro.html

[28]https://hbase.apache.org/2.0/devapidocs/org/apache/hadoop/hbase/io/hfile/HFile.html

[29] http://hudi.apache.org/blog/hudi-indexing-mechanisms/

[30]https://cwiki.apache.org/confluence/display/HUDI/RFC-27+Data+skipping+index+to+improve+query+performance

[31] https://github.com/apache/hudi/pull/2487

[32] https://issues.apache.org/jira/browse/HUDI-1295

[33]https://cwiki.apache.org/confluence/display/HUDI/RFC+-+22+%3A+Snapshot+Isolation+using+Optimistic+Concurrency+Control+for+multi-writers

[34]https://cwiki..org/confluence/display/HUDI/RFC+-+22+%3A+Snapshot+Isolation+using+Optimistic+Concurrency+Control+for+multi-writers#RFC22:SnapshotIsolationusingOptimisticConcurrencyControlformultiwriters-FutureWork(LockFree-ishConcurrencyControl)

[35]http://hudi.apache.apacheorg/blog/hudi-file-sizing/

[36]https://rocksdb.org/

[37]http://hudi.apache.org/blog/hudi-key-generators/

[38]https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/

[39]https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/event-time/generating_watermarks/

[40] https://github.com/apache/hudi/pull/3306

[41] https://github.com/apache/hudi/pull/3247

[42] https://prestodb.io/blog/2021/02/04/raptorx

[43] https://trino.io/docs/current/connector/hive-caching.html

[44] https://debezium.io/

[45]http://hudi.apache.org/blog/employing-right-configurations-for-hudi-cleaner/

[46] http://hudi.apache.org/blog/hudi-clustering-intro/

[47] https://github.com/apache/hudi/pull/3233

[48]https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=181307144

[49] http://hudi.apache.org/blog/hudi-meets-aws-emr-and-aws-dms/

[50] http://hudi.apache.org/blog/exporting-hudi-datasets/

[51] http://hudi.apache.org/blog/ingest-multiple-tables-using-hudi/

[52]https://www.snowflake.com/blog/how-foundationdb-powers-snowflake-metadata-forward/

[53] https://javalin.io/

[54] https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool.html

[55] https://github.com/facebook/rocksdb/wiki/Block-Cache

[56] http://hudi.apache.org/community.html



**译者介绍**：

杨华（vinoyang）Apache Hudi PMC & Committer. T3 出行大数据平台技术负责人。

**原文链接：**

http://hudi.apache.org/blog/2021/07/21/streaming-data-lake-platform

 