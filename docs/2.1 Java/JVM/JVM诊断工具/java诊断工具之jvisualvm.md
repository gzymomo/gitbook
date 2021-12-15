[TOC]

# 1、概述
jvisualvm也是java最新的诊断工具之一，该工具可以帮助开发人员诊断应用，并可以监控和提供应用性能。通过jvisualvm可以生成、分析堆dumps文件，跟踪内存泄露问题，执行和监控垃圾回收，执行轻量级内存和CPU分析。该工具还可以进行调优，调整堆大小，脱机分析和事后诊断。
另外，可以使用现有的插件来扩展jvisualvm功能，例如，JConsole工具的大部分功能都可以通过MBeans选项卡和JConsole插件包装器选项卡获得，可以在jvisualvm窗口的plugins菜单中选择各种插件。

# 2、使用jvisualvm进行故障诊断
1. 查看本地和远程java应用列表。
2. 查看应用配置和运行环境，对每个应用工具显示基础的运行信息：PID，host，main class，参数，JVM版本，JDK home，JVM标志，JVM参数，系统属性。
3. 可以设置当某个应用发生OutOfMemoryError异常的时候创建堆dump文件。
4. 监控应用内存消耗，运行中的线程，加载的类。
5. 立即触发垃圾收集。
![](https://img.hacpai.com/file/2019/08/image-4b4ae871.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

6. 立刻创建一个堆dump文件，可以在以下几个视图中查看堆dump信息：概要，类，实例数。还可以把堆dump文件保存到本地。
![](https://img.hacpai.com/file/2019/08/image-a250a9de.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

7. 分析应用程序性能或者分析内存分配（只支持本地应用），也可以保存分析数据。
![](https://img.hacpai.com/file/2019/08/image-4fd32cc5.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

8. 立刻创建一个线程dump（跟踪应用活跃线程数），可以查看线程dump。
![](https://img.hacpai.com/file/2019/08/image-dbc5430b.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

9. 分析核心dump(使用Oracle Solaris和Linux操作系统)。
10. 根据应用快照离线分析应用。
11. 获取社区提供的其他插件。
12. 编写并共享自己的插件。
13. 安装MBeans插件之后，可以显示并与MBeans进行交互。
jvisualvm主界面显示以下内容：显示了本地正在运行的java应用列表，远程java应用列表，之前保存查看过的快照列表，获取和保存的所有JVM核心dump的列表。
![](https://img.hacpai.com/file/2019/08/image-476cdd0b.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

# 3、语法
`jvisualvm [ options ]`
选项
- --help             查看帮助
- --jdkhome <path>   jdk的安装路径
- -J<jvm_option>     把jvm_option传给JVM
- --cp:p <classpath> 将<classpath>前置到类路径
- --cp:a <classpath> 将<classpath>追加到类路径
- --fork-java        在单独的进程中运行java
- --trace <path>     启动程序日志路径(用于故障排除)
- --console suppress supppress控制台输出
- --console new      打开新的控制台进行输出