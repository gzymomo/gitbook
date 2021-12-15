[TOC]

相关内容参考链接：

- 微信公众号：芋道源码

- [SpringBoot项目优化和Jvm调优](https://www.cnblogs.com/jpfss/p/9753215.html)
- [Springboot jar包启动及JVM参数设置](https://blog.csdn.net/qq_35573959/article/details/106827648)



# 1、项目调优
SpringBoot项目配置Tomcat和JVM参数。
## 1.1 修改配置文件
SpringBoot项目详细的配置文件修改文档
其中比较重要的有：
```yml
server.tomcat.max-connections=0 # Maximum number of connections that the server accepts and processes at any given time.
server.tomcat.max-http-header-size=0 # Maximum size, in bytes, of the HTTP message header.
server.tomcat.max-http-post-size=0 # Maximum size, in bytes, of the HTTP post content.
server.tomcat.max-threads=0 # Maximum number of worker threads.
server.tomcat.min-spare-threads=0 # Minimum number of worker threads.
```
# 2、Jvm调优
## 2.1 设置Jvm参数
例如要配置JVM参数：
`-XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms1024m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC1`
方式一：如果你用的是IDEA等开发工具，来启动运行项目，那么要调试JDK就方便太多了。只需要将参数值设置到VM options中即可。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/e315666c3043ccdf21ddd22e782f0c79?showdoc=.jpg)

方式二：适用于在项目部署后，在启动的时候，采用脚本或者命令行运行的时候设置。
先在项目路径下，给项目打包：清理就项目
`mvn clean`
打包新项目：
`mvn package -Dmaven.test.skip=true`
打包完成后进入可运行Jar包的路径下,执行启动设置Jvm参数的操作。
`$ java -jar -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms1024m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC newframe-1.0.0.jar`

## 2.2 JYM参数说明
 - -XX:MetaspaceSize=128m （元空间默认大小）
 - -XX:MaxMetaspaceSize=128m （元空间最大大小）
 - -Xms1024m （堆最大大小）
 - -Xmx1024m （堆默认大小）
 - -Xmn256m （新生代大小）
 - -Xss256k （棧最大深度大小）
 - -XX:SurvivorRatio=8 （新生代分区比例 8:2）
 - -XX:+UseConcMarkSweepGC （指定使用的垃圾收集器，这里使用CMS收集器）
 - -XX:+PrintGCDetails （打印详细的GC日志）

JDK8之后把-XX:PermSize 和 -XX:MaxPermGen移除了，取而代之的是 -XX:MetaspaceSize=128m （元空间默认大小） -XX:MaxMetaspaceSize=128m （元空间最大大小） JDK 8开始把类的元数据放到本地化的堆内存(native heap)中，这一块区域就叫Metaspace，中文名叫元空间。使用本地化的内存有什么好处呢？最直接的表现就是java.lang.OutOfMemoryError: PermGen 空间问题将不复存在，因为默认的类的元数据分配只受本地内存大小的限制，也就是说本地内存剩余多少，理论上Metaspace就可以有多大（貌似容量还与操作系统的虚拟内存有关？这里不太清楚），这解决了空间不足的问题。不过，让Metaspace变得无限大显然是不现实的，因此我们也要限制Metaspace的大小：使用-XX:MaxMetaspaceSize参数来指定Metaspace区域的大小。JVM默认在运行时根据需要动态地设置MaxMetaspaceSize的大小。

# 3、**常用JVM参数**

| 参数                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| -XX:+AlwaysPreTouch                   | JVM启动时分配内存，堆的每个页面都在初始化期间按需置零，而不是在应用程序执行期间递增 |
| -XX:Errorfile = filename              | 错误日志                                                     |
| -XX:+TraceClassLoading 跟踪类加载信息 |                                                              |
| -XX:+PrintClassHistogram              | 按下Ctrl+Break后打印类信息                                   |
| -Xmx -Xms                             | 最大堆 最小堆                                                |
| -xx:permSize                          | 永久代大小                                                   |
| -xx:metaspaceSize                     | 元数据空间大小                                               |
| -XX:+HeapDumpOnOutOfMemoryError       | 当抛出OOM时进行HeapDump                                      |
| -XX:+HeapDumpPath                     | OOM时堆导出的路径                                            |
| -XX:OnOutOfMemoryError                | 当发生OOM时执行用户指定的命令                                |

命令： java -XX:+PrintFlagsFinal -version 会 打印所有的-XX参数及其默认值



# 4、**GC调优思路**

1. 分析场景，如：启动速度慢，偶尔出现响应慢于平均水平或出现卡顿
2. 确定目标，如：内存占用，低延时，吞吐量
3. 收集日志，如：通过参数配置收集GC日志，通过JDK工具查看GC状态
4. 分析日志，如：使用工具辅助分析日志，查看GC次数，GC时间
5. 调整参数，如：切换垃圾收集器或者调整垃圾收集器参数

# 5、常用GC参数

| 参数                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| -XX:ParallelGCThreads     | 并行GC线程数量                                               |
| -XX:ConcGcThreads         | 并发GC线程数量                                               |
| -XX:MaxGCPauseMillis      | 最大停顿时间，单位毫秒，GC尽力保证回收时间不超过设定值       |
| -XX:GCTimeRatio           | 垃圾收集时间占总时间的比值，取值0-100，默认99，即最大允许1%的时间做GC |
| -XX:SurvivorRatio         | 设置eden区大小和survivor区大小的比例，8表示两个survivor:eden=2:8，即一个survivor占年轻代的1/10 |
| -XX:NewRatio              | 新生代和老年代的比，4表示新生代:老年代=1:4，即年轻代占堆的1/5 |
| -verbose:gc，-XX:+PrintGC | 打印GC的简要信息                                             |
| -XX:+PrintGCDetails       | 打印GC详细信息（JDK9之后不再使用）                           |
| -XX:+PrintGCTimeStamps    | 打印GC发生的时间戳（JDK9之后不再使用）                       |
| -Xloggc:log/gc.log        | 指定GC log的位置，以文件输出                                 |
| -XX:PrintHeapAtGC         | 每次GC后都打印堆信息                                         |

# 6、垃圾收集器Parallel参数调优

 Parallel垃圾收集器在JDK8中是JVM默认的垃圾收集器，它是以吞吐量优先的垃圾收集器。其可调节的参数如下：

| 参数                       | 描述                     |
| -------------------------- | ------------------------ |
| -XX:+UseParallelGC         | 新生代使用并行垃圾收集器 |
| -XX:+UseParallelOldGC      | 老年代使用并行垃圾收集器 |
| -XX:ParallelGCThreads      | 设置用于垃圾回收的线程数 |
| -XX:+UseAdaptiveSizePolicy | 打开自适应GC策略         |

# 7、垃圾收集器CMS参数调优

 CMS垃圾收集器是一个响应时间优先的垃圾收集器，Parallel收集器无法满足应用程序延迟要求时再考虑使用CMS垃圾收集器，从JDK9开始CMS收集器已不建议使用，默认用的是G1垃圾收集器。

| 参数                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| -XX:+UseConcMarkSweepGC            | 新生代使用并行收集器，老年代使用CMS+串行收集器               |
| -XX:+UseParNewGC                   | 新生代使用并行收集器，老年代CMS收集器默认开启                |
| -XX:CMSInitiatingOccupanyFraction  | 设置触发GC的阈值，默认68%，如果内存预留空间不够，就会引起concurrent mode failure |
| -XX:+UseCMSCompactAtFullCollection | Full GC后，进行一次整理，整理过程是独占的，会引起停顿时间变长 |
| -XX:+CMSFullGCsBeforeCompaction    | 设置进行几次Full GC后进行一次碎片整理                        |
| -XX:+CMSClassUnloadingEnabled      | 允许对类元数据进行回收                                       |
| -XX:+UseCMSInitiatingOccupanyOnly  | 表示只在达到阈值的时候才进行CMS回收                          |
| -XX:+CMSIncrementalMode            | 使用增量模式，比较适合单CPU                                  |

# 8、垃圾收集器G1参数调优

 G1收集器是一个兼顾吞吐量和响应时间的收集器，如果是大堆（如堆的大小超过6GB）,堆的使用率超过50%，GC延迟要求稳定且可预测的低于0.5秒，建议使用G1收集器。

| 参数                                  | 描述                                                    |
| ------------------------------------- | ------------------------------------------------------- |
| -XX:G1HeapRegionSize                  | 设置Region大小，默认heap/2000                           |
| -XX:G1MixedGCLiveThresholdPercent     | 老年代依靠Mixed GC, 触发阈值                            |
| -XX:G1OldSetRegionThresholdPercent    | 定多包含在一次Mixed GC中的Region比例                    |
| -XX:+ClassUnloadingWithConcurrentMark | G1增加默认开启，在并发标记阶段结束后，JVM即进行类型卸载 |
| -XX:G1NewSizePercent                  | 新生代的最小比例                                        |
| -XX:G1MaxNewSizePercent               | 新生代的最大比列                                        |
| -XX:G1MixedGCCountTraget              | Mixed GC数量控制                                        |