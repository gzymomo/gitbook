**一、生成HTML测试报告的两种方式**

**1、利用已有.jtl文件生成报告**

之前的博客介绍过如何在[linux环境运行jmeter并生成报告](https://www.cnblogs.com/imyalost/p/9808079.html)，如果已经有经过测试生成的.jtl文件，可以利用该文件直接生成HTML可视化测试报告。

进入jmeter的bin目录下，输入如下命令：

```
jmeter -g test.jtl -o /path
# -g：后跟test.jtl文件所在的路径
# -o：后跟生成的HTML文件存放的路径
```

**PS：**如果是在Windows环境命令行运行，必须指定生成的HTML文件存放文件夹，否则会报错；如果是linux环境，如指定路径下不存在该文件夹，会生成对应的文件夹存放报告文件！

 

**2、无.jtl文件生成测试报告**

如果还未生成.jtl文件，则可以通过如下命令，一次性完成测试执行和生成HTML可视化报告的操作，进入jmeter的bin目录下，输入如下命令：

```
jmeter -n -t test.jmx -l test.jtl -e -o /path
# -n：以非GUI形式运行Jmeter 
# -t：source.jmx 脚本路径 
# -l：result.jtl 运行结果保存路径（.jtl）,此文件必须不存在 
# -e：在脚本运行结束后生成html报告 
# -o：用于存放html报告的目录
```

我本地Windows环境执行截图如下：

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190108211725639-1373603449.png)

**PS：**（linux系统和windows系统命令一样）需要注意的是，生成的**.jtl**文件路径下，不能存在同名的**.jtl**文件，否则会执行失败。

执行完毕后，用浏览器打开生成的文件目录下的index文件，效果展示如下：

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190108212803345-2030078543.png)

 

**二、图表信息详解**

测试报告分为两部分，Dashboard和Charts，下面分开解析。

**1、Dashboard（概览仪表盘）**

**①、Test and Report informations**

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190108214635483-1600436571.png)

**②、APDEX (应用性能指标)**

关于APDEX的相关信息，请参考这里：[应用性能指标](http://oneapm.udesk.cn/hc/articles/515)；英文原文，参考这里：[Apdex-Wikipedia](https://en.wikipedia.org/wiki/Apdex)

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190108215348337-273403282.png)

**③、Requests Summary**

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190108215644132-1505880418.png)

 

**2、Charts（详细信息图表）**

**PS：**由于详细信息图表有点多，这里我挑几个性能测试过程中比较关键的图表解析！

**Over Time**

**①、Response Times Over Time（脚本运行期间的响应时间变化趋势图）**

说明：可以根据响应时间和变化和TPS以及模拟的并发数变化，判断性能拐点的范围。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109094111342-1489934097.png)

**②、 Response Time Percentiles Over Time (successful responses)**

说明：脚本运行期间成功的请求响应时间百分比分布图，可以理解为聚合报告里面不同%的数据，图形化展示的结果。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109094529173-1868847214.png)

**③、Bytes Throughput Over Time（脚本运行期间的吞吐量变化趋势图）**

说明：在容量规划、可用性测试和大文件上传下载场景中，吞吐量是很重要的一个监控和分析指标。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109112528071-162666554.png)

**④、 Latencies Over Time（脚本运行期间的响应延时变化趋势图）**

说明：在高并发场景或者强业务强数据一致性场景，延时是个很严重的影响因素。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109114954469-1258401112.png)

 

**Throughput**

**①、Transactions Per Second（每秒事务数）**

说明：每秒事务数，即TPS，是性能测试中很重要的一个指标，它是用来衡量系统处理能力的一个重要指标。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109132330006-165754605.png)

 

**Response Times**

**①、 Response Time Percentiles（响应时间百分比分布曲线图）**

说明：即响应时间在某个范围内的请求在所有请求数中所占的比率，相比于平均响应时间，这个值更适合用来衡量系统的稳定性。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109132806769-75726845.png)

**②、Time Vs Threads（平均响应时间和线程数的对应变化曲线）**

说明：可以通过这个对应的变化曲线来作为确定性能拐点的一个参考值。

![img](https://img2018.cnblogs.com/blog/983980/201901/983980-20190109133049288-1188263361.png)

 

以上内容，即为jmeter生成HTML格式测试报告的方法以及报告内容解析，个人觉得这个图表可以进行再次开发，变得更灵活和易用。。。