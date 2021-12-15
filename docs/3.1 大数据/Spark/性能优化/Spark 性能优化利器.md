# Spark 性能优化利器

原文地址：https://mp.weixin.qq.com/s/iMQiHayz1DspZBJCFNHVzQ

优化 Spark 代码是一项具有挑战性和耗时的工作，而无需了解任务在何处面临瓶颈以及用户希望实现的优化运行时间是多少。造成这种瓶颈的原因主要有 3 个：

- Driver 计算时间大

- 存在任务倾斜

- 任务数不够，意味着并行性问题

根据情况，可以实现两个优化目标——我们可以致力于减少应用程序的运行时间，或者我们可以通过使用更少的资源使其更好地工作来优化它。需要一种解决方案来帮助识别此类瓶颈，从而帮助我们实现所需的优化目标。

任何 Spark 应用程序的性能都可以通过 Yarn 资源管理器 UI 或 Spark Web UI  来观察，但它没有为我们提供可以指出应用程序面临的瓶颈的详细指标。有多种工具可用于 Spark 性能监控，例如  Sparklens、Sparklint、Dr. Elephant、SparkOscope 等，它们为我们提供了更详细的应用程序性能细节。

# 什么是Sparklens？

Sparklens 是 Qubole 的产品。它是一个内置 Spark Scheduler 模拟器的 Spark 分析和性能预测工具。它有助于识别 Spark 应用程序面临的瓶颈，并为我们提供关键路径时间。

Sparklens 报告为我们提供了可用于优化的后续步骤的想法。它的主要目标是让人们更容易理解 Spark 应用程序的可扩展性限制。它有助于了解给定的 Spark 应用程序使用所提供的计算资源的效率。也许一个应用程序会随着更多的执行器运行得更快，也许它不会，Sparklens  可以通过查看应用程序的单次运行来回答这个问题。它有助于将 Spark  应用程序调优作为一种定义明确的方法/过程，而不是可以通过反复试验来学习的东西，从而节省开发人员和计算时间以及成本。

# 如何使用Sparklens？ 

Sparklens 非常易于使用。没有先决条件，我们不必专门安装任何东西。我们只需要传递一组固定的命令以及下面提到的 Spark-submit shell，并且在运行后立即生成 Sparklens 报告。

```
— packages qubole:Sparklens:0.3.1-s_2.11
— conf Spark.extraListeners=com.qubole.Sparklens.QuboleJobListener
```

# Sparklens 报告提供的详细性能指标

•驱动程序与执行程序挂钟时间 - Spark 应用程序挂钟的总时间可以分为花费在驱动程序上的时间和花费在执行程序上的时间。当 Spark 应用程序在驱动程序上花费太多时间时，就会浪费执行程序的计算时间。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dutMNXxf19R95dzTxEhjLibVibRcm52qKUMq5GR3Z31bZ1iahdpB8Af83w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

导致驱动程序负载更高甚至驱动程序内存不足的常见原因是 -

•rdd.collect()•sparkContext.broadcast•spark.sql.autoBroadcastJoinThreshold 配置错误。在连接操作的情况下，Spark 使用此限制向所有节点广播关系。在第一次使用时，整个关系在驱动程序节点上具体化。有时，多个表也会作为查询执行的一部分进行广播。

我们应该尝试以这样一种方式编写应用程序，即可以避免驱动程序中的所有显式结果收集。此类任务可以委托给其中一位执行者。例如，如果某些结果必须保存到特定文件中，则可以在驱动程序处收集它们，或者我们可以指定一个执行程序来为我们执行此操作。

•理想应用程序运行时间的估计 - 理想的应用程序时间是通过假设应用程序的所有阶段的数据的理想分区（任务 == 核心和无偏斜）来计算的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5d2oax99gDOjaJe3ur6te0RL7XYkdudnEWRyKnGxtRxcu8a9lwVY6tgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



•关键路径时间 - 这是应用程序所需的最短时间，即使我们给它无限的执行程序。关键路径时间和应用程序运行时间之间的差异是 Spark 应用程序的真正弹性。如果差异很大，则需要更多的执行者。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dINWYTJUSmPpkQpibmeMcLg7nvvjicuUJtI1b4z1AHoTF8mT1XRTDtO8g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



•最佳执行者数量 - 如果启用了自动缩放或动态分配，我们可以看到在任何给定时间有多少个 executor 可用。Sparklens 绘制了应用程序中不同 Spark 作业使用的执行程序，以及可以在相同的挂钟时间内完成相同工作的最少数量的执行程序（理想情况）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5d96KE40ZoBhiash6VSTe9CCWNg4OCzS3yYbqDcdQVaPuahKhaZ0HEDOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



•计算小时数 - 它告诉我们有多少分配给应用程序的计算资源根本没有使用。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dUictwBpOERPFPCBSUXeibTTibWYsbcyRVT8Xic9JM05s1I0ibeJiauUx1uZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



所有上述参数也可以通过 Sparklens UI 以图形方式观察 -

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dtRGoDicEVf9ibo9OlCpaKNCZRIiahiag60JLMibnJ6Sleo2kzqibEd7meBLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



•倾斜和缺乏任务 - 由于缺乏任务或偏差，执行器也可能浪费计算时间。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dY5LV7lWx6pSibXXz6j7AHfz58JZBoicFjRmVvP7PKicmcHQpkITuRHt9w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dFCafggxQVIUUXsbPkGvDgfJ2GvVql0vDUM9b84ib5Pqujw6bM1L0kNg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



•常用术语 - Sparklens 报告在报告末尾提供了所提到的主要参数的详细定义，从而使其自给自足和清晰。

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dnzTRkjjt680YW9Be7icXwlY2rOvRsRBIBUOboSERgIA6k2DribOZ1vOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/3EcuPIickyyFfvB9Vklicyb1swdNDBQe5dia5VRfrl0FkVtf0icCWq29Eyh5x912BLMMAejtx11kjdMqxsSd4rrvcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 总结

Spark 作业的瓶颈可以通过报告来识别，无论是从资源、数据还是脚本方面。Sparklens  报告确定了可以达到的最短时间和固定目标时间。我们可以理解我们离优化的代码有多远。关键路径解决方案可能并不总是可以实现，但我们可以尝试尽可能接近。根据 Sparklens 的调查结果，可以制定详细的行动和实施计划。Sparklens 的实现可以跨不同的管道执行，因为 Sparklens  易于使用——某些固定的命令集可以与 Spark-submit shell 一起传递。Sparklens  报告每次都会生成类似的指标，因此更容易理解。

# 引用

- https://github.com/qubole/Sparklens

- https://docs.qubole.com/en/latest/user-guide/engines/Spark/Sparklens.html

- https://www.qubole.com/blog/introducing-quboles-Spark-tuning-tool/[1]

- https://www.youtube.com/watch?v=KS5vRZPLo6c