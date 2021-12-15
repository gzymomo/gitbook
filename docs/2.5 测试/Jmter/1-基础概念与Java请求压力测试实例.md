# 基础概念

## QPS

QPS是Queries Per Second的缩写，是特定服务器每秒钟处理完的查询相关的查询的数量，反映了机器性能上的吞吐能力。

## TPS

TPS是Transactions Per Second 的缩写，是特定服务器每秒处理完的事务数量。需要注意的这些都是处理完的事务数。

### TPS vs QPS

简单地来说，以特定的实例来说明TPS和QPS的区别，两个都是衡量单位时间所处理完的请求的能力，但是计数的粒度和方式有所不同，比如一个事务中如果包含3次查询操作的话，TPS计数为1，而QPS则为3.

### RPS vs QPS

RPS是Requests Per Second 的缩写，是特定服务器每秒处理完的请求数量，是衡量机器吞吐性能的参数。RPS和QPS统计方式基本相同，角度和叫法不同。

### PV

PV是Page View的缩写，指的是页面浏览量，用于衡量网站页面的访问次数。同一用户对于同一页面连续刷新的情况下，此值会连续累计。

### UV

UV是Unique Visitor的缩写，指的是不重复的访客独立访客数量，一般用于统计 1 天内访问某站点的用户数(比如以cookie为依据)，每台机器会被计算为一个独立的访客。一天之内的同一访客的多次访问在UV上只会被计算一次。

### VV

VV是Visit Vie的缩写，指的访客的访问数量，一般用于统计1天之内所有访客访问的次数。一次访问被定义为访问完页面并关闭浏览器，同一访客一天之内如果有多次访问，访问次数会进行累计。

### IP

IP是Internet Protocol的缩写，本来指的是IP协议，而这里指的是不重复的IP数量。一般用于统计1 天内多少个不重复的 IP 有过访问，实际上是从IP的角度来来确认实际用户的数量。同一个IP一天之内无论访问了多少个页面，只会被计算一次。

### RT

RT是Response Time的缩写，指的是单次请求的响应时间。一般普通页面的响应速度应该在毫秒级别，超过3秒，用户会感到较为缓慢。

### 并发用户数

并发用户数指的是系统可以承载的并行使用用户的数量，在实际的情况下，以网站的使用情况为例，为有注册用户数、在线用户数和并发请求用户数这样的统计数据，因为在线用户也不会所有人都一个频度在访问页面，所以在线用户的数量并不能明确反映性能状况，比如大量用户在线但是什么都不做的情况下跟所有人都在频繁操作会有很大区别。在实际的工具中类似的功能或者概念的叫法一般有所不同，在LR（LoadRunner）中被成为虚拟用户VU（Virtual User），而在Apache JMeter中则被称之为线程数。

# JMeter使用示例

## 确认内容

在压力测试工具中这个计算公式应该会非常清楚，在接下来的示例中我们来确认这一点。

> 计算公式：并发用户数 = QPS x RT

## 环境准备

关于Apache JMeter的概要介绍与安装的方法，可参看如下内容：

- https://liumiaocn.blog.csdn.net/article/details/101264380

### 语言设定

关于JMeter的语言设定，有两种方式，可以通过菜单选项直接设定，也可以在启动脚本中进行定制化修改和设定。

- 方法1: 选择Options菜单的Choose Language下拉菜单中简体中文Chinese ( Simplified ) 即可即使起效，但是退出之后不会保存，需要每次设定
- 方法2: 修改jmeter.properties文件，将language设定为zh_CN，修改示例如下所示

```
liumiaocn:apache-jmeter-5.1.1 liumiao$ grep language= bin/jmeter.properties 
#language=en
liumiaocn:apache-jmeter-5.1.1 liumiao$ vi bin/jmeter.properties 
liumiaocn:apache-jmeter-5.1.1 liumiao$ grep language= bin/jmeter.properties 
language=zh_CN
liumiaocn:apache-jmeter-5.1.1 liumiao$
123456
```

启动之后将会看到如下中文界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925093131523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## 验证准备

使用如下步骤准备测试验证的前提准备：

- 步骤1: 在测试计划下添加一个线程组，选择菜单信息如下图所示：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925093509480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)
- 步骤2: 在刚刚创建的线程组上添加一个Java请求的取样器，选择菜单信息如下图所示：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925093703596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)
- 步骤3: 在测试计划中添加一个聚合报告，选择菜单信息如下图所示：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925093932159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

## 验证1: 并发用户数100、循环10次、持续时间10秒的内置Java请求验证

设定信息如下：

| 设定项   | 设定值 |
| -------- | ------ |
| 线程数   | 100    |
| 循环次数 | 10     |
| 持续时间 | 10s    |

点击绿色的启动按钮开始执行，然后点击聚合报告可以看到实时的信息，执行结束后结果信息如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190925094555794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)
公式确认：

> 计算公式：并发用户数 = QPS x RT

- 并发用户数：相当于线程数，次例中为100
- QPS：就是此处的吞吐量，值为278.4/秒
- RT：响应时间，此处的平均值列即为所用内容，单位是毫秒，所以说明此Java请求为平均响应时间是232毫秒。

结果确认：

> QPS * RT = 278.4 * 0.232 = 64.5888

显然这个结果距离实际设定的线程数100还是具有不少的差距的，实际上在数量小的时候损耗的影响还是很大的，如果把这个数字加大，时间拉长，效果就比较明显了。

## 验证2: 并发用户数100、循环360次、持续时间180秒的内置Java请求验证

设定信息如下：

| 设定项   | 设定值 |
| -------- | ------ |
| 线程数   | 100    |
| 循环次数 | 360    |
| 持续时间 | 180s   |

执行之前右键单击，然后选择清楚选项，去除第一次结果的执行的影响。接下来再点击绿色的启动按钮开始执行，然后点击聚合报告可以看到实时的信息，执行结束后结果信息如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019092509590387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9saXVtaWFvY24uYmxvZy5jc2RuLm5ldA==,size_16,color_FFFFFF,t_70)

公式确认：

> 计算公式：并发用户数 = QPS x RT

- 并发用户数：相当于线程数，次例中为100
- QPS：就是此处的吞吐量，值为422.9/秒
- RT：响应时间，此处的平均值列即为所用内容，单位是毫秒，所以说明此Java请求为平均响应时间是228毫秒。

结果确认：

> QPS * RT = 422.9 * 0.228 = 96.4212

显然这个结果距离实际设定的线程数100已经比较接近了，在一定程度上已经能够证明这个公式的正确性了。