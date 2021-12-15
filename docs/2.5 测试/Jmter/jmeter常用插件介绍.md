jmeter作为一个开源的接口性能测试工具，其本身的小巧和灵活性给了测试人员很大的帮助，但其本身作为一个开源工具，相比于一些商业工具（比如LoadRunner），在功能的全面性上就稍显不足。

这篇博客，就介绍下jmeter的第三方插件**jmeter-plugins.org**和其中常用的几种插件使用方法。

 

**一、下载安装及使用**

下载地址：[jmeter-plugins.org](https://jmeter-plugins.org/downloads/all/)

安装：下载后文件为[plugins-manager.jar](https://jmeter-plugins.org/get/)格式，将其放入jmeter安装目录下的lib/ext目录，然后重启jmeter，即可。

启动jemter，点击选项，最下面的一栏，如下图所示：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171107004149903-1073083008.png)

打开后界面如下：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171107004322825-635646935.png)

**Installed Plugins**（已安装的插件）：即插件jar包中已经包含的插件，可以通过选中勾选框，来使用这些插件；

**Available Plugins**（可下载的插件）：即该插件扩展的一些插件，可以通过选中勾选框，来下载你所需要的插件；

**Upgrades**（可更新的插件）：即可以更新到最新版本的一些插件，一般显示为加粗斜体，可以通过点击截图右下角的Apply Changes and Restart Jmeter按钮来下载更新；

**PS**：一般不建议进行更新操作，因为最新的插件都有一些兼容问题，而且很可能导致jmeter无法使用（经常报加载类异常）！！！

　　建议使用jmeter最新的3.2版本来尝试更新这些插件。。。

 

**二、Transactions per Second**

即**TPS：每秒事务数**，性能测试中，最重要的2个指标之一。该插件的作用是在测试脚本执行过程中，监控查看服务器的TPS表现————比如**整体趋势、实时平均值走向、稳定性**等。

jmeter本身的安装包中，监视器虽然提供了比如[聚合报告](http://www.cnblogs.com/imyalost/p/5804359.html)这种元件，也能提供一些实时的数据，但相比于要求更高的性能测试需求，就稍显乏力。

通过上面的下载地址下载安装好插件后，重启jmeter，从监视器中就可以看到该插件，如下图所示：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171107233305263-1730024319.png)

某次压力测试TPS变化展示图：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108110751794-1362459450.png)

 

**三、Response Times Over Time**

即**TRT：事务响应时间**，性能测试中，最重要的两个指标的另外一个。该插件的主要作用是在测试脚本执行过程中，监控查看**响应时间的实时平均值、整体响应时间走向**等。

使用方法如上，下载安装配置好插件之后，重启jmeter，添加该监视器，即可实时看到实时的TRT数值及整体表现。

某次压力测试TRT变化展示图：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108110842809-709007936.png)

 

**四、PerfMon Metrics Collector**

即**服务器性能监控数据采集器**。在性能测试过程中，除了监控TPS和TRT，还需要监控服务器的资源使用情况，比如**CPU、memory、I/O**等。该插件可以在性能测试中实时监控服务器的各项资源使用。

下载地址：http://jmeter-plugins.org/downloads/all/或链接：[http://pan.baidu.com/s/1skZS0Zb](https://www.cnblogs.com/imyalost/p/链接：http://pan.baidu.com/s/1skZS0Zb) 密码：isu5

下载界面如下：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108000311403-97876486.png)

其中JMeterPlugins-Standard和JMeterPlugins-Extras是**客户端**的插件，ServerAgent是**服务端**的插件。

下载成功后，复制**JmeterPlugins-Extras.jar**和**JmeterPlugins-Standard.jar**两个文件，放到jmeter安装文件中的lib/ext中，重启jmeter，即可看到该监视器插件。如下图：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108001125106-452288317.png)

将ServerAgent-2.2.1.jar上传到被测服务器，解压，进入目录，Windows环境，双击ServerAgent.bat启动；linux环境执ServerAgent.sh启动，默认使用4444端口。

如出现如下图所示情况，即表明服务端配置成功：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108001556763-1493728509.png)

**1、服务端启动校验**

CMD进入命令框，观察是否有接收到消息，如果有，即表明ServerAgent成功启动。

**2、客户端监听测试**

给测试脚本中添加jp@gc - PerfMon Metrics Collector监听器，然后添加需要监控的服务器资源选项，启动脚本，即可在该监听器界面看到资源使用的曲线变化。如下图所示：

![img](https://images2017.cnblogs.com/blog/983980/201711/983980-20171108003044059-468808623.png)

在脚本启动后，即可从界面看到服务器资源使用的曲线变化，Chart表示主界面显示，Rows表示小界面以及不同资源曲线所代表的颜色，Settings表示设置，可选择自己需要的配置。

**PS：**注意测试脚本需要持续运行一段时间，才可以看到具体的曲线变化，否则ServerAgent端会断开连接！