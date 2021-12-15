[TOC]

# 1、关于JVM配置：
设置jvm内存的参数有四个：
```
-Xmx 设置堆(Java Heap)最大值，默认值为物理内存的1/4，最佳设值应该视物理内存大小及计算机内其他内存开销而定。
-Xms 设置初始堆(Java Heap)初始值，Server端JVM最好将-Xms和-Xmx设为相同值，开发测试机JVM可以保留默认值。
-Xmn 设置年轻代(Java Heap Young)区大小，在整个堆内存大小确定的情况下，增大年轻代将会减小老年代，反之亦然。不熟悉最好保留默认值。
-Xss 每个线程的栈（Stack）大小，不熟悉最好保留默认值；
-XX:NewSize=1024m 设置年轻代初始值
-XX:MaxNewSize=1024m 设置年轻代最大值
-XX:PermSize=256m 设置持久代初始值
-XX:MaxPermSize=256m 设置持久代初始值
-XX:NewRatio=4 设置年轻代（包括一个Eden区和两个Survivor区）和年老代的比值。
-XX:SurivivorRatio=4 设置Survivor区和Eden区的比值。
-XX:MaxTenuringThreshold=7 表示一个对象如果在Survivor区（救助空间）移动了7次还没有被垃圾回收就进入年老代。
```

```shell
一般用到最多的是：
-Xms512m （堆最大大小） 设置jvm促使内存为512M，此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存
-Xmx512m （堆默认大小） 设置jvm最大可用内存为512M
-Xmn200m （新生代大小） 设置年轻代大小为200M。整个堆大小=年轻代大小+年老代大小+持久代大小。持久代一般固定大小为64m。所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8
-Xss128k 设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右
```

# 2、Linux JVM设置：
	在Linux系统的服务器上面，启动各个spring cloud的微服务jar包的时候，需要在java -jar 的命令中间增加jvm的内存参数设置：-Xms64m -Xmx128m

```shell
nohup java -Xms64m -Xmx128m -jar xxx.xxxx-xxx-xxxxxx-0.0.1-SNAPSHOT.jar &
```

# 3、GC日志收集-配置参数
```java
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails  输出GC的详细日志
-XX:+PrintGCTimestamps   输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps   输出GC的时间戳（以日期的形式）
-XX:+PrintHeapAtGC   在进行GC的前后打印出堆的信息
```

# 4、调优策略
两个基本原则：

- 将转移到老年代的对象数量降到最少。
- 减少Full GC的执行时间。目标是Minor GC时间在100ms以内，Full GC时间在1s以内。
- 主要调优参数：

设定堆内存大小，这是最基本的。

- -Xms：启动JVM时的堆内存空间。
- -Xmx：堆内存最大限制。

设定新生代大小。
新生代不宜太小，否则会有大量对象涌入老年代。
- -XX:NewRatio：新生代和老年代的占比。
- -XX:NewSize：新生代空间。
- -XX:SurvivorRatio：伊甸园空间和幸存者空间的占比。
- -XX:MaxTenuringThreshold：对象进入老年代的年龄阈值。

设定垃圾回收器：

- 年轻代：-XX:+UseParNewGC。
- 老年代：-XX:+UseConcMarkSweepGC。

CMS可以将STW时间降到最低，但是不对内存进行压缩，有可能出现“并行模式失败”。比如老年代空间还有300MB空间，但是一些10MB的对象无法被顺序的存储。这时候会触发压缩处理，但是CMS GC模式下的压缩处理时间要比Parallel GC长很多。
G1采用”标记-整理“算法，解决了内存碎片问题，建立了可预测的停顿时间类型，能让使用者指定在一个长度为M毫秒的时间段内，消耗在垃圾收集上的时间不得超过N毫秒。



