### 数据编辑器

- 工具提示中的列注释

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZNf36uMECjb9uLUibWKMnNc3gDibFZoez5y0X98Z9guiaibe4dZYniaqIEcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 智能代码

完成DataGrip提供上下文相关的代码完成，帮助您更快地编写SQL代码。完成可以识别表格结构、外键，甚至是您正在编辑的代码中创建的数据库对象。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZdHMicicgKrjOvClRTFibFISlXicC4TZGW76V1DHBgChglJbxVb8AWnan7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 即时分析和快速修复

DataGrip会检测代码中可能存在的错误，并建议动态修复它们的最佳选项。它会立即让您了解未解决的对象，使用关键字作为标识符，并始终提供解决问题的方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZlKQpatVnt233H9a2N1MCw73ibV6IAx9LQUS4w5RNz9YMJd0ib2rjwPlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 日志更新

完整的SQL日志，现在您将看到DataGrip在控制台输出中运行的每个查询。无论是您的SQL还是DataGrip需要在内部运行的东西，请查看“ 输出”选项卡以了解发生了什么。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZV8M42ibyDyDV56SZclfTgoXF1cxdj62UvC97KxtiaDFNMpwNGmiaz1WgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其次，来自IDE的所有查询现在都记录在文本文件中。要打开此文件，请转到“ 帮助”| 显示SQL日志。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZtqoQlKx5VdTkCX5kNIOLfr4K6WQwljH3BcIH6DYNmuCzYSJ1OqkK7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZy7gIDWXSwz8Y1F6ic86ClfoRE7JJO6wNrvcpQ6XZ4WqqFuo7Qu7G4IQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 新的SQL格式化程序

感谢您与我们分享您对SQL样式的想法！我们希望现在DataGrip能够容纳更多不同的代码样式。新的SQL格式化程序是我们强烈需要反馈的功能，因此请尝试一下，如果您的具体案例未涵盖，请告诉我们。我们仍在努力增加新的条款。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZxAhSkX7eOYm7aBYDEkyYUO7KCXEF6Sgsib2Z7Cq28kOwBvGiac283whg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

您可能已经知道，您可以创建自定义代码样式方案。现在，它们中的任何一个都可以专门用于每个数据源。为此，请转到数据源属性的“ 选项”选项卡：

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZ9kK9Gib5TQzVC4uN3ylvZWrgT6Zfg5qRVAUlRIPL3UjEaxFAfcF9hOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

运行存储过程

从过程的上下文菜单中选择“执行”。将生成SQL代码。输入所需参数的值，然后单击“确定”。如您所见，我们检索此mysql过程的输出，因为我们有SQL代码从JDBC驱动程序获取结果集：

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZicBHgOriaR3Cq7Z6LJSotWpXfz0a4ync4h8jJLgwpzgn6VpTrn3sp8Xg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

查询计划（优化性能的神器）

查询计划图基于图表的视图现在可用于查询计划。要查看它，请在调用说明计划后单击工具栏上的“ 显示可视化”按钮：

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZ7KCeL5oY1C6SB3SGVfJKXJ88TPnskt8Q6JG3c6Nv1F46vhMavibiax7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### SQL编辑

上下文信息在编辑包中的大型过程时，有时在其上下文中刷新内存是有用的，即现在正在编辑的特定过程或包。为此，请按Shift+Ctrl+Q以查看上下文信息。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupWdx1iaqrRl512WdRqG9c99tS2DUuRfzevmYutIcUmEVz9on5Vh4ialABrZzBaBRIZoibkyDd83Bzrnw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

跳到关闭括号/报价之外从此版本开始，您可以通过按Tab键在结束括号之外导航或关闭引号 。请注意，这仅在第一次输入参数或值时有效。要自定义Tab的此行为，请转到“首选项”| 编辑| 一般| 智能键并选择 跳转到关闭括号外/使用Tab键引用。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupWdx1iaqrRl512WdRqG9c99tMJR5VnFXlTpTdcBNtvdGy7eE5icLoFn1B8DmFmlrsZ4BxKxEVaU39tQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

### 导航

从“ 查找操作”分配快捷方式以前，如果使用 默认键盘映射，则无法从“ 查找操作”中指定快捷方式。我们已修复此错误，现在它适用于任何键盘映射和任何布局。一个很好的理由提醒你，这是可能的！

导航允许您通过相应的操作按名称跳转到任何表，视图或过程，或直接从SQL代码中的用法跳转到任何表，视图或过程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/R3InYSAIZkFAph5vkJibyH1SEe89IBHKZvgkMiaXFkd5kUpFaibhSlrXsiaNkibYhicEXibb0lib6vlW5IJE0eyMX5icIbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/6mychickmupWdx1iaqrRl512WdRqG9c99tVTia2k8uTuXm7yNcmU7KuzeOiaPnS4Q36oofbFwIpFXNVuFMaovMic2Rw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

总的来说，DataGrip是一个面向管理员和SQL开发人员的综合数据库IDE。它具有实用的功能，支持DB2、Derby、H2、MySQL、Oracle、PostgreSQL、SQL  Server、Sqllite及Sybase等网上主流的关系数据库产品，除了能执行sql、创建表、创建索引以及导出数据等常用的功能之外，还能在关键字上有高亮的提示，而且对字段的提示也是非常智能的！