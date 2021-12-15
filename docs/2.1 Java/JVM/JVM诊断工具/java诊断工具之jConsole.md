[TOC]

# 1、简介
jConsole监控工具也是比较有用的，该工具与JMX兼容，该工具使用JVM内置的JMX来监控运行程序的性能和资源消耗情况，也可以监视JRE的应用。
jConsole可以应用到任何java应用程序中，以便可以查看以下信息：线程使用情况、内存消耗、类加载的详细信息、运行时编译、操作系统。还可以诊断高级别的问题，如：内存泄露、过度的类加载、运行中的线程。对系统的调优和堆大小很有帮助。
除了监控之外，jConsole还支持动态修改几个参数，
如：`-verbose:gc，可以在运行时动态的开启或者关闭垃圾收集跟踪信息。`

# 2、故障排除
下面列出jConsole可以列出哪些数据，下面的模块都对应jConsole工具里面的一个tab。
## 2,1 概览
该窗口显示所有的图表，如堆内存使用，线程数，CPU使用情况。
![](https://img.hacpai.com/file/2019/08/image-6d78d10d.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2,2 内存
1. 用于选定的内存区域（堆、非堆、各种内存池）
 - 显示内存随时间使用情况的图表
 - 当前内存大小
 - 提交的内存量
 - 最大内存大小
2. 垃圾收集信息，包括执行的收集数量、执行垃圾收集总花费时间
3. 显示当前堆和非堆使用内存百分比图表

此外，还可以在该界面上执行垃圾收集。
![](https://img.hacpai.com/file/2019/08/image-16e7ccd0.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2.3 线程
- 显示线程随时间的使用情况的图表。
- 活动线程:活动线程的当前数量。
- Peak: JVM启动以来活动线程的最高数量。

对于选定的线程，以及对于阻塞的线程、线程正在等待获取的同步器和拥有锁的线程，分别执行名称、状态和堆栈跟踪。
死锁检测按钮向目标应用程序发送一个请求来执行死锁检测，并在一个单独的选项卡中显示每个死锁周期。
![](https://img.hacpai.com/file/2019/08/image-2d942545.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2.4 类
- 显示随时间推移加载的类的数量的图表
- 当前装入内存的类的数量
- 自JVM启动以来加载到内存中的类的总数，包括随后卸载的类
- 自JVM启动以来从内存中卸载的类的总数。
![](https://img.hacpai.com/file/2019/08/image-0060f8dd.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2.5 VM概要
一般信息，如JConsole连接数据、JVM的正常运行时间、JVM消耗的CPU时间、编译器名称和总编译时间等等。
- 线程和类摘要信息。
- 内存和垃圾收集信息，包括挂起的对象数量等。
- 有关操作系统的信息，包括物理特性、用于运行进程的虚拟内存数量和交换空间。
- 关于JVM本身的信息，例如参数和类路径。
![](https://img.hacpai.com/file/2019/08/image-d5b18882.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2.6 MBean
此窗格显示一个树结构，显示在连接的JMX代理中注册的所有平台和应用程序mbean，当你在树中选择一个MBean时，将显示它的属性、操作、通知和其他信息。
1. 你可以调用操作(如果有的话)，例如，HotSpotDiagnostic MBean的dumpHeap操作，它位于com.sun.management域下，执行堆转储，此操作的输入参数是目标VM所在机器上堆转储文件的路径名。
2. 你可以设置可写属性的值。例如，你可以通过调用HotSpotDiagnostic MBean的setVMOption操作来设置、取消设置或更改某些VM标志的值，这些标志由DiagnosticOptions属性的值列表表示。
3. 如果有通知，可以使用订阅和取消按钮订阅。
![](https://img.hacpai.com/file/2019/08/image-3ca15e3f.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

![](https://img.hacpai.com/file/2019/08/image-d7b99d56.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 2.7 监控远程或者本地应用
jConsole可以监控远程或者本地应用，如果通过JMX代理去进行连接，那么jConsole将自动开始监控应用。如果监控本地应用，那么使用jconsole pid ，pid是应用的进程号；如果监控远程应用，那么使用jconsole hostname:portnumber ，hostname是远程应用的host，portnumber是远程应用指定的JMX代理。如果使用jconsole启动工具，那么你可以在操作界面上指定连接远程还是本地，可以在Connection菜单中任意设置连接地址，如下示例显示了内存的使用情况：
![](https://img.hacpai.com/file/2019/08/image-f6ebb972.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

# 3、语法
## 3.1 概要
`jconsole [ options ] [ connection ... ]`
optiosns：
命令行选项
`connection = pid | host:port | jmxURL`
- pid表示JVM进程id，必须用启动JVM相同的登录用户来启动jConsole；host:port表示系统的域名地址，port是启动是指定的com.sun.management.jmxremote.port的值；jmxURL是一个JMX的代理地址。

## 3.2 选项
- -interval=n
更新间隔设置多少秒，默认是4秒。
- -notile
是否需要平铺窗口
- -pluginpath plugins
指定要搜索JConsole插件的目录或JAR文件列表。插件路径应该包含一个META-INF/services/com.sun.tools.jconsole.JConsolePlugin提供程序配置文件，还必须要实现com.sun.tools.jconsole.JConsolePlugin类。
- -version
显示发布信息和退出。
- -help
显示帮助消息。
- -Jflag
将标志传递给运行jconsole命令的JVM。

## 3.3 示例
设置本地监控
`jconsole 111`
设置远程监控
`jconsole http://127.0.01:8080`