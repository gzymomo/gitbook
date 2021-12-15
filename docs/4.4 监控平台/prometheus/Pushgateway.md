- [基于Prometheus的Pushgateway实战](https://www.cnblogs.com/xiao987334176/p/9933963.html)
- [Pushgateway 介绍](https://www.cnblogs.com/shhnwangjian/p/10706660.html)
- [Prometheus基于pushgateway方式数据采集实战](https://baijiahao.baidu.com/s?id=1653813369632155595&wfr=spider&for=pc)



# 一、Pushgateway概述

Pushgateway是一个独立的服务，Pushgateway位于应用程序发送指标和Prometheus服务器之间。

Pushgateway接收指标，然后将其作为目标被Prometheus服务器拉取。

可以将其看作代理服务，或者与blackbox exporter的行为相反， 它接收度量，而不是探测它们。



[Pushgateway](https://github.com/prometheus/pushgateway) 是 Prometheus 生态中一个重要工具，使用它的原因主要是：

- Prometheus 采用 pull 模式，可能由于不在一个子网或者防火墙原因，导致 Prometheus 无法直接拉取各个 target 数据。
- 在监控业务数据的时候，需要将不同数据汇总, 由 Prometheus 统一收集。

由于以上原因，不得不使用 pushgateway，但在使用之前，有必要了解一下它的一些弊端：

- 将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大。
- Prometheus 拉取状态 `up` 只针对 pushgateway, 无法做到对每个节点有效。
- Pushgateway 可以持久化推送给它的所有监控数据。

因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据。



被动的数据采集方式，获取监控数据资源的prometheus插件。可以单独运行再任何节点的插件。用户开发自定义脚本把数据发送给pushgateway，pushgateway提供给prometheus服务器。



拓扑图如下：

![img](https://img2018.cnblogs.com/blog/1341090/201811/1341090-20181109120151807-1018788353.png)

 

