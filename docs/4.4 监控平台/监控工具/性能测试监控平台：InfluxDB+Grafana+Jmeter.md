**前言**

性能测试工具jmeter自带的监视器对性能测试结果的实时展示，在Windows系统下的GUI模式运行，渲染和效果不是太好，在linux环境下又无法实时可视化。

因此如果有一个性能测试结果实时展示的页面，可以提高我们对系统性能表现的掌握程度，另一方面也提高了我们的测试效率。

InfluxDB+Telegraf+Grafana+Jmeter的框集成，就很好的解决了这些问题。网上关于这些开源组建的介绍已经很多了，目前我所在的性能团队内部就使用的该套框架。

这篇博客，就介绍下如何集成这些开源工具，搭建属于自己的性能测试监控平台。。。

 

**一、安装环境**

| 组件名称 | 版本说明       |
| -------- | -------------- |
| 服务器   | Centos7.4 64位 |
| jmeter   | 3.2            |
| JDK      | 1.8            |
| InfluxDB | 1.0.2          |
| Grafana  | 5.3.2          |

 

**二、jmeter和JDK安装**

linux环境下，jmeter和JDK的安装，请看这里：[linux环境运行jmeter并生成报告](https://www.cnblogs.com/imyalost/p/9808079.html)

 

**三、InfluxDB安装**

linux环境下，安装influxdb，请看这里：[时序数据库InfluxDB：简介及安装](https://www.cnblogs.com/imyalost/p/9689209.html)

安装后，新建数据库，命令如下：

```
# 新建一个名为zwgdb的数据库
create database zwgdb
# 创建数据保存策略，这里数据保存时间为7天，默认采用此策略保留数据
create retention policy "zwgdb_7d" on "zwgdb" duration 7d replication 1 default
# 查看数据库数据保存策略
show retention policies on zwgdb
```

![img](https://img2018.cnblogs.com/blog/983980/201811/983980-20181103174029336-474968416.png)

 

**四、Grafana安装**

linux环境下，安装grafana，请看这里：[可视化工具Grafana：简介及安装](https://www.cnblogs.com/imyalost/p/9873641.html)

**PS**：安装后，可根据使用目的和使用者类型，进行分组，为了使每个成员使用平台进行监控时操作互相独立，又可以互相查看对方的数据，可以在influxdb中新建多个数据库。

在grafana中为每个成员创建各自的登录账号，如下：

![img](https://img2018.cnblogs.com/blog/983980/201811/983980-20181103174833081-1300710023.png)

然后，为每个成员添加数据源，如下：

![img](https://img2018.cnblogs.com/blog/983980/201811/983980-20181103175217785-280144578.png)

**PS：**如何添加数据源，请看前面的关于Grafana的安装使用的博客。

 

**五、测试实践**

**1.启动jmeter，新建测试脚本**

![img](https://img2018.cnblogs.com/blog/983980/201811/983980-20181103175519912-833189540.png)

**2、运行脚本，实时监控测试结果**

![img](https://img2018.cnblogs.com/blog/983980/201811/983980-20181103181847356-488906578.png)