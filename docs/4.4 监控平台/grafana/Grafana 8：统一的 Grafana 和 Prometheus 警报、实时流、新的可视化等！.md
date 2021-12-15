- [Grafana 8：统一的 Grafana 和 Prometheus 警报、实时流媒体、新的可视化等！](https://juejin.cn/post/6972119275725144095#heading-9)

## 前言

Grafana v8.0 的**重大变更**包括对告警系统的重构；新的可视化改进，包括状态时间线、状态历史和直方图面板； **实时流**； 可以重用的**库面板**； 和**细粒度的访问控制**，允许企业客户确保其组织中的每个人都具有适当的**访问级别**。

我们还对 Grafana 的性能和功能进行了升级。 用户界面的改进带来了焕然一新的外观和感觉； Enterprise 中的数据源查询缓存用来**加快仪表盘加载**； 由于初始下载数据的大幅减少，**更好的启动和加载性能**，这意味着您可以更快地工作并享受与仪表盘的响应更快的交互。

要了解更多信息，请收听 6 月 9 日在 [GrafanaCONline 上举行的 Grafana 8.0 深入探讨会议](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fgo%2Fgrafanaconline%2F2021%2Fgrafana-8%2F%3Fpg%3Dblog)。在本次演讲中，Grafana 团队成员将演示此版本中的更多新功能。 您还可以在我们新的 [Grafana Play 仪表盘](https://link.juejin.cn?target=https%3A%2F%2Fplay.grafana.org%2Fd%2FYI95GyqMz%2F1-new-features-in-v8-0%3ForgId%3D1)中查看 v8 中的新功能。 

最后，您可以通过 [Grafana Cloud](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fproducts%2Fcloud%2F) 在几分钟内开始使用 Grafana。 我们有免费和付费的 Grafana Cloud 计划来满足每个用例——[立即免费注册](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fauth%2Fsign-up%2Fcreate-user%3Fpg%3Dblog)。

现在让我们来看看Grafana8.0中所有令人兴奋的新特性！

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b8da306d4714aa79fc6f59889895bfd~tplv-k3u1fbpfcp-watermark.image)

## 一、告警

多年来，Grafana 社区提出的最多需求都是警报相关的。去年9月，我们在 Grafana Cloud 中引入了 Prometheus 风格的告警，在 Grafana 实例中嵌入了一个简单的 UI 来管理警报。

在此基础上，我们在 8.0 对 Grafana 告警系统进行了全面的改进，将 Prometheus 告警和 Grafana 告警统一在同一个用户界面中，用于查看和编辑告警。这为所有 Grafana 用户和数据源提供了一种常见的告警体验——无论您使用的是开源版本还是企业和云堆栈。

Grafana 托管告警和来自 Prometheus 兼容数据源的告警都受支持，因此您可以为 Grafana 托管告警、Cortex 告警和 Loki 告警创建和编辑告警规则，还可以在单个可搜索视图中查看来自 Prometheus 兼容数据源的告警信息。这是 8.0 的一个选择加入功能。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/264de22d8c5b458a829b2956d5b1a4f8~tplv-k3u1fbpfcp-watermark.image)

警报现在已与仪表盘解耦，我们还添加了对多维警报的支持、用于大规模管理通知的通知策略，以及功能齐全的API。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00943567b234408dbb1deedada26ef45~tplv-k3u1fbpfcp-watermark.image)

有关我们新的警报功能的更多信息，请务必在6月16日的 GrafanaCONline 上观看“[Grafana中告警的下一步](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fgo%2Fgrafanaconline%2F2021%2Falerting%2F)”课程。

## 二、值映射

使用新的值映射编辑器，可以将字符串和布尔状态直接映射到颜色和可选显示文本。这将在所有Grafana可视化中工作，包括新的状态时间表面板（见下文）。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cccde16ea60f465eba8a1166ab7075be~tplv-k3u1fbpfcp-watermark.image)

## 三、状态时间轴面板

“状态时间线”面板可以随时间显示字符串或布尔值状态。使用上述新的值映射功能，可以为每个值指定颜色。通过使用阈值，您还可以将时间序列可视化为随时间变化的离散状态，这样就可以很容易地一眼看到每个阈值括号中花费的持续时间。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83719bd7b9db4732a20291cebaf0932e~tplv-k3u1fbpfcp-watermark.image)

## 四、历史状态面板

该面板旨在显示状态回顾，随着时间的推移可视化周期性数据。 您可以使用值映射为每个值添加颜色。 这适用于数字、字符串或布尔状态。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92d83a091e6242db8b95b0a60e280f08~tplv-k3u1fbpfcp-watermark.image)

## 五、条形图面板

新的条形图面板为 Grafana 增加了新的绘图功能，特别是对于非时间序列数据。它支持分类 x 或 y 字段、分组条以及水平和垂直布局。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be734c3dd5544dceb611011d390f0934~tplv-k3u1fbpfcp-watermark.image)

## 六、直方图面板

曾经是旧图形面板的隐藏功能，此直方图面板现在是一个独立的可视化。 您可以使用此面板将计算数据分布中的桶的直方图转换与条形图可视化结合起来。 此外，我们还引入了可以与任何可视化配对的直方图转换。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6525bf95885e4ca8acc438e155a84ed5~tplv-k3u1fbpfcp-watermark.image)

## 七、面板搜索和表格切换

为了改进导航，我们添加了搜索功能，以便更轻松地在长长的面板选项和覆盖列表中找到您想要的内容。 它们现在也都列在面板编辑侧栏中，而不是在选项卡中分开。 此外，还有一个新的表视图切换，可让您快速查看传递给可视化的数据。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1244e4fef0554956b0c142edc239230d~tplv-k3u1fbpfcp-watermark.image)

## 八、库面板

我们添加了一个用于重用面板的新工作流程。 您现在可以构建可跨多个仪表盘共享的库面板。 对库面板所做的更改或更新将反映在使用该库面板的每个仪表盘上。

## 九、实时流

实时流自从在 [7.4 版本的图形面板中实现预览版](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fblog%2F2021%2F02%2F04%2Fgrafana-7.4-released-next-generation-graph-panel-with-30-fps-live-streaming-prometheus-exemplar-support-trace-to-logs-and-more%2F%23next-generation-graph-panel)，在 8.0 中获得了更多功能。 这是我们在 Grafana 中为支持工业/物联网用例所做的激动人心的改变的一部分。 实时更新现在可以通过与 [MQTT](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgrafana%2Fmqtt-datasource) 数据源的 websocket 连接发送到仪表盘，也可以从 cURL 或 Telegraf 流式传输。 还可以通过将指标发布到新的实时端点  `/api/live/push` 来将事件发送到仪表盘。

![Grafana 8 live streaming demo from Grafana Labs .gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7abc1657248e4ea8b6542177d0dcd93e~tplv-k3u1fbpfcp-watermark.image)

它现在是 Grafana 的内置标准功能，可以开箱即用。 您所要做的就是推送到 API 并为您推送的数据连接面板。

## 十、loki 日志的改进

我们对探索中的日志导航进行了重大改进。 我们为日志添加了分页功能，因此您可以在达到行数限制时点击查看较旧或较新的日志。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7524d28e7f3445f96bbd883bf4808ce~tplv-k3u1fbpfcp-watermark.image)

您还可以通过面板检查器中的 Data 选项卡和 Explore 的检查器将日志结果下载为文本文件。

## 十一、更多的 traces 函数支持

您现在可以通过直接从 [Grafana Tempo](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Foss%2Ftempo%2F)（我们刚刚 GA 的分布式跟踪后端）查询 [Grafana Loki](https://link.juejin.cn?target=http%3A%2F%2Fgrafana.com%2Foss%2Floki%2F) 来搜索跟踪！

使用带有[日志的附加 Loki 数据源](https://link.juejin.cn?target=https%3A%2F%2Fgrafana.com%2Fdocs%2Ftempo%2Flatest%2Fgrafana-agent%2Fautomatic-logging%2F)，您可以通过 Tempo 更轻松地发现跟踪并快速构建 Loki 查询。 更进一步，Tempo 查询面板现在可以帮助您从 Loki 数据源日志构建查询，因此您不必成为 LogQL 专家——同时提供更统一的跟踪发现体验。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e27eb8c7df1040c79ec1e0b2b140d4f4~tplv-k3u1fbpfcp-watermark.image)

Explore 中还有更好的 Jaeger 搜索，以及支持 Jaeger、Zipkin 和 Tempo 的显示跟踪图。


作者：如梦技术
链接：https://juejin.cn/post/6972119275725144095
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。