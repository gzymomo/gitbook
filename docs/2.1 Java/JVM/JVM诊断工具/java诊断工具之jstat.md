[TOC]

# 1、概要
jstat是VM内置的统计工具，主要对应用的性能和资源消耗提供信息，对诊断应用性能问题、堆大小、垃圾收集等问题很有用，启动jstat不需要特殊的option，在Java HotSpot VM中默认是启动的。
jstat实用程序使用虚拟机标识符(VMID)来标识目标进程，文档描述了VMID的语法，但是它唯一需要的组件是本地虚拟机标识符(LVMID)，LVMID通常(但并不总是)是操作系统针对目标JVM进程的PID。
jstat工具提供的数据类似于vmstat和iostat工具在Oracle Solaris和Linux操作系统上提供的数据。对于数据的图形表示，可以使用visualgc工具。

# 2、语法
## 2.1基本语法
`jstat [ generalOption | outputOptions vmid [ interval[s|ms] [ count ] ]`
- generalOption：一个通用命令行参数，值为：-help或者-options，如下图显示
![](https://img.hacpai.com/file/2019/09/image-f5e274bb.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

- outputOptions：一个或多个选项，包括statOption选项，加上-t，-h，-J中的其中一个。
- vmid：虚拟机标识，表示需要统计的目标JVM，一般的语法如下：[protocol:][//]lvmid[@hostname[:port]/servername]，vmid可以是简单的进程id，也可以是一个协议，指定host和端口。
- interval [s|ms]：在指定单元内进行循环间隔，单位为毫秒或者秒，默认是毫秒，必须是正整数，如果指定了，那么jstat将每隔一定时间进行输出。
- count：要显示的样本数量，默认是无穷大，直到JVM关闭或者jstat命令终止，值必须的正整数。

## 2.2 描述
jstat可以统计应用的性能情况，可以通过vmid来指定。

## 2.3 虚拟机标识符
vmid的语法与URI的语法一样，
`[protocol:][//]lvmid[@hostname[:port]/servername]`

- protocol：通信协议，如果省略了协议值，并且没有指定主机名，则默认协议是特定于平台的优化本地协议，如果没有指定协议，但是指定了host，那么默认协议是rmi。
- lvmid：当前JVM的本地虚拟机标识符，lvmid是一个特定于平台的值，它惟一地标识系统上的JVM，lvmid是虚拟机标识符惟一需要的组件。lvmid通常是(但不一定是)目标JVM进程的操作系统进程标识符。可以使用jps命令来确定lvmid值，此外，还可以使用ps命令在Solaris、Linux和OS X平台上确定lvmid，并使用Windows任务管理器在Windows上确定lvmid。
- hostname：指定目标主机的主机名或者ip地址，如果省略了hostname值，那么目标主机就是本地主机。
- port：与远程服务器通信的默认端口，如果省略主机名值或协议值指定优化的本地协议，则忽略端口值，对于默认的rmi协议，端口值指示远程主机上rmiregistry的端口号，如果省略端口值，协议值指示rmi，则使用缺省的rmiregistry端口(1099)。
- servername：servername参数的处理取决于实现，对于优化的本地协议，这个字段被忽略，对于rmi协议，它表示远程主机上rmi远程对象的名称。

# 3、选项
jstat支持两种选项，通用选项和输出选项，通用选项显示jstat的简单使用和版本信息，输出选项决定输出的内容和格式。

## 3.1 通用选项
如果指定了通用选项，那么不能在指定其他选项或者参数。
- -help：显示帮助信息
- -options：显示静态选项列表。

## 3.2 输出选项
如果没有指定通用选项，那么就可以指定输出选项，它决定了输出的内容和格式，它由statOption组成，并加上-h，-t，-J中的一个。
输出格式为一个表，列之间以空格隔开，标题行描述每列的值，使用-h选项设置显示标题的频率，列标题名称在不同选项之间是一致的。通常，如果两个选项提供具有相同名称的列，则这两个列的数据源是相同的。
使用-t选项显示时间戳列，将时间戳标记为输出的第一列。Timestamp列包含目标JVM启动以来的运行时间(以秒为单位)，时间戳的分辨率依赖于各种因素，并且由于负载过重的系统上的线程调度延迟，时间戳的分辨率可能会发生变化。
使用interval和count参数分别确定jstat命令显示输出的频率和次数。

### 3.2.1 -statOption
确定jstat命令显示的统计信息。下面列出可用的选项：
- class: 显示有关类加载器行为的统计信息。
- compiler: 显示有关Java HotSpot VM即时编译器行为的统计信息。
- gc: 显示关于垃圾收集堆的行为的统计信息。
- gccapacity: 显示有关新生代/老年代的容量及其对应空间的统计信息。
- gccause: 显示关于垃圾收集统计信息的摘要(与-gcutil相同)，以及上次和当前垃圾收集事件的原因。
- gcnew: 显示新生代行为的统计信息。
- gcnewcapacity: 显示关于新生代的大小及其对应空间的统计信息。
- gcold: 显示关于老年代行为的统计信息和元空间统计信息。
- gcoldcapacity: 显示关于老年代大小的统计信息。
- gcmetacapacity: 显示有关元空间大小的统计信息。
- gcutil: 显示关于垃圾收集统计信息的摘要。
- printcompilation: 显示Java HotSpot VM编译方法统计信息。

### 3.2.2 -h n
每n个样本(输出行)显示一个列标题，其中n是一个正整数，默认值为0，它显示列标题第一行数据。

### 3.2.3 -t
在第一行中显示时间戳列，时间戳是目标JVM启动时间之后的时间。

### 3.2.4 -JjavaOption
将javaOption传递给Java应用程序启动程序。例如，-J-Xms48m将启动内存设置为48mb。

# 4、统计选项和输出
以下信息总结了jstat命令为每个statOption输出的列：

## 4.1 -class 选项
- class loader统计信息
- Loaded: 加载的类的数量。
- Bytes: 加载的kBs数量。
- Unloaded: 卸载的类的数量。
-Bytes: 卸载的字节数。
- Time: 用于执行类加载和卸载操作的时间。

## 4.2 -compiler 选项
Java HotSpot VM即时编译器统计数据。
- Compiled: 执行编译任务的数量。
- Failed: 编译任务失败的次数。
- Invalid: 无效编译任务的数目。
- Time: 用于执行编译任务的时间。
- FailedType:上次编译失败的编译类型。
- FailedMethod: 上次编译失败的类名和方法。

## 4.3 -gc 选项
堆垃圾收集统计数据。
- S0C: 当前survivor0的空间容量，单位为kb。
- S1C: 当前survivor1的空间容量，单位为kb。
- S0U: survivor0空间使用大小，单位为kb。
- S1U: survivor1空间使用大小，单位为kb。
- EC: 当前eden空间容量，单位为kb。
- EU: 当前eden使用大小，单位为kb。
- OC: 当前old空间大小容量，单位为kb。
- OU: old使用大小，单位为kb。
- MC: Metaspace容量大小，单位为kb。
- MU: Metaspace使用大小，单位为kb。
- CCSC: 压缩类空间的大小容量，单位为kb。
- CCSU: 压缩类空间的使用，单位为kb。
- YGC: 年轻代垃圾收集事件的数量。
- YGCT: 年轻代垃圾收集时间。
- FGC: full GC的数量。
- FGCT: full gc的时间。
- GCT: 垃圾收集总时间。

## 4.4 -gccapacity 选项
显示各个代的容量和使用情况
- NGCMN: 新生代最小容量，单位为kb。
- NGCMX: 新生代最大容量，单位为kb。
- NGC: 当前新生代容量，单位为kb。
- S0C: 当前survivor0的容量，单位为kb。
- S1C:当前survivor1的容量 ，单位为kb。
- EC: 当前eden容量，单位为kb。
- OGCMN: 老年代最小容量，单位为kb。
- OGCMX: 老年代最大容量，单位为kb。
- OGC: 当前老年代容量，单位为kb。
- OC: 当前老年代容量，单位为kb。
- MCMN: metaspace最小容量，单位为kb。
- MCMX: metaspace最大容量，单位为kb。
- MC: 当前metaspace容量，单位为kb。
- CCSMN: 压缩类空间最小容量，单位为kb。
- CCSMX: 压缩类空间最大容量，单位为kb。
- CCSC: 当前压缩类空间容量，单位为kb。
- YGC: 新生代GC发生次数。
- FGC: full gc发生次数。

## 4.5 -gccause 选项
此选项显示与-gcutil选项相同，都是显示垃圾收集统计信息摘要，但包括上次垃圾收集事件的原因，以及当前垃圾收集事件的原因，除了为-gcutil列出的列之外，该选项还添加了以下列：
- LGCC: 上次gc的原因。
- GCC: 当前gc的原因。

## 4.6 -gcnew 选项
新生代统计信息。
- S0C: 当前survivor0的容量大小，单位为KB。
- S1C: 当前survivor1的容量大小，单位为KB。
- S0U: 当前survivor0的使用大小，单位为KB。
- S1U: 当前survivor1的使用大小，单位为KB。
- TT: 晋升阀值。
- MTT: 最大晋升阀值。
- DSS: 预期的survivor大小，单位为KB。
- EC: 当前eden容量大小，单位为KB。
- EU: eden的使用大小，单位为KB。
- YGC: 新生代gc次数。
- YGCT: 新生代gc时间。

## 4.7 -gcnewcapacity 选项
新生代空间大小统计。
- NGCMN: 最小新生代容量，单位为KB。
- NGCMX: 最大新生代容量，单位为KB。
- NGC: 当前新生代容量，单位为KB。
- S0CMX: 最大survivor0空间容量，单位为KB。
- S0C: 当前survivor0空间容量，单位为KB。
- S1CMX: 最大survivor1空间容量，单位为KB。
- S1C: 当前survivor1空间容量，单位为KB。
- ECMX: 最大eden空间容量，单位为KB。
- EC: 当前eden空间容量，单位为KB。
- YGC: 新生代gc次数。
- FGC: full gc次数。

## 4.8 -gcold 选项
老年代和元空间行为信息统计
- MC: metaspace容量，单位为KB。
- MU: metaspace使用情况，单位为KB。
- CCSC: 压缩类空间容量，单位为KB。
- CCSU: 压缩类空间使用情况，单位为KB。
- OC: 当前老年代容量，单位为KB。
- OU: 老年代空间使用情况，单位为KB。
- YGC: 新生代gc次数。
- FGC: full gc次数。
- FGCT: full gc时间。
- GCT: gc总时间。

## 4.9 -gcoldcapacity 选项
老年代大小统计信息。
- OGCMN: 最小老年代容量，单位为KB。
- OGCMX: 最大老年代容量，单位为KB。
- OGC: 当前老年代容量，单位为KB。
- OC: 当前老年代容量，单位为KB。
- YGC: 新生代gc次数。
- FGC: full gc 次数。
- FGCT: full gc时间。
- GCT: gc总时间。

## 4.9 -gcmetacapacity 选项
metaspace大小统计信息。
- MCMN: 最小metaspace容量，单位为KB。
- MCMX: 最大metaspace容量，单位为KB。
- MC: metaspace容量，单位为KB。
- CCSMN: 压缩类空间最小容量，单位为KB。
- CCSMX: 压缩类空间最大容量，单位为KB。
- YGC: 新生代gc次数。
- FGC: full gc次数。
- FGCT: full gc时间。
- GCT: gc总时间。

## 4.10 -gcutil 选项
gc统计数据摘要。
- S0: survivor0的利用率占空间当前容量的百分比。
- S1: survivor1的利用率占空间当前容量的百分比。
- E: eden的利用率占空间当前容量的百分比。
- O: 老年代的利用率占空间当前容量的百分比。
- M: metaspace的利用率占空间当前容量的百分比。
- CCS: 压缩类空间利用率的百分比。
- YGC: 新生代gc次数。
-  YGCT: 新生代gc时间。
- FGC: full gc次数。
- FGCT: full gc时间。
- GCT: gc总时间。

## 4.11 -printcompilation 选项
Java HotSpot VM编译器方法统计。
- Compiled: 最近编译的方法执行的编译任务的数量。
- Size: 最近编译的方法的字节大小。
- Type: 最近编译的方法的编译类型。
- Method: 最近编译的方法的类名、方法名。类名使用斜杠(/)代替点(.)作为名称空间分隔符，方法名是指定类中的方法，这两个属性值的格式跟-XX:+PrintCompilation是一致的。

# 5、可视化
可以使用visualgc来进行可视化界面展开，
![](https://img.hacpai.com/file/2019/10/image-3f7ac1dc.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

[下载地址](https://www.oracle.com/java/technologies/java-archive-jvm-downloads.html#jvmstat-3_0-mr-oth-JPR)

# 6、示例
## 6.1 示例1
`jstat -gcutil 2834 250 7`
-gcutil表示查看垃圾回收，每个250毫秒间隔获取样本，一共获取7次。
![](https://img.hacpai.com/file/2019/09/image-8d022a58.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

这个示例的输出显示，在第三个和第四个示例之间发生了一个年轻代集合。该集合耗时0.078秒，将对象从eden空间(E)提升到old空间(O)，使old空间利用率从66.80%提升到68.19%。收集前的幸存者空间利用率为97.02%，收集后的幸存者空间利用率为91.03%。

## 6.2 示例2
`jstat -gcnew -h3 2834 250`
-gcnew表示收集年轻代，-h3表示每隔三行显示标题，
![](https://img.hacpai.com/file/2019/09/image-84ccade4.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

除了显示重复的头字符串外，这个示例还显示了在第4和第5个示例之间发生了一个年轻代收集，其持续时间为0.02秒。该集合找到了足够的活动数据，以致幸存者空间1利用率(S1U)将超过所需的幸存者大小(DSS)。结果，对象被提升到老一代(在这个输出中不可见)，并且 tenuring阀值(TT)从15降低到1。

## 6.3 示例3
jstat -gcoldcapacity -t 21891 250 3
-gcoldcapacity表示对老年代信息查看，-t表示在第一列中显示时间戳。
![](https://img.hacpai.com/file/2019/09/image-58ecb648.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

Timestamp列以秒为单位报告自目标JVM启动以来的运行时间。此外，-gcoldcapacity输出显示随着堆扩展以满足分配或提升需求，老年代容量(OGC)和老的空间容量(OC)会增加。在第81次FGC之后，OGC已经从11696 KB增长到13820 KB。代和空间最大的容量是60544 KB (OGCMX)，因此仍然有扩展的空间。