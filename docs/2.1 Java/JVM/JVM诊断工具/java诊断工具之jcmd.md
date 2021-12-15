[TOC]

# 1、简介
jcmd被用于发送诊断命令给JVM，这些请求对控制java飞行记录、排查故障和诊断JVM和java应用。JVM必须运行在相同的机器上，并具有启动JVM相同的有效用户和组标识符。

# 2、jcmd比较有用的命令
在不同的HotSpot VM版本中jcmd命令有所不同，因此可以使用jcmd <process id/main class> help命令来查看所有可用选项，以下命令是在JDK8中最有用的命令，可以使用jcmd <process id/main class> help <command>命令来查看具体命令的更多选项。

## 2.1 打印所有HotSpot和JDK版本id
`jcmd <process id/main class> VM.version`
## 2.2 打印所有VM的系统属性
可以显示数百行信息
`jcmd <process id/main class> VM.system_properties`
## 2.3 打印VM使用的所有标志
即使您没有提供任何标志，也会打印一些默认值，例如初始值和最大堆大小。
`jcmd <process id/main class> VM.flags`
## 2.4 以秒为单位打印正常运行时间
`jcmd <process id/main class> VM.uptime`
## 2.5 创建一个类直方图
输出内容会很多，可以把他输出到文件中，内部类和特定于应用程序的类都包含在列表中，占用内存最多的类列在顶部，类按降序排列。
`jcmd <process id/main class> GC.class_histogram`
## 2.6 创建一个堆dump文件
与jmap -dump:file=<file> <pid>生成堆dump文件相同，但是建议使用jcmd命令，这里生成hprof文件。
`jcmd GC.heap_dump filename=Myheapdump`

## 2.7 创建堆直方图
`jcmd <process id/main class> GC.class_histogram filename=Myheaphistogram`
与jmap -histo <pid>命令相似，但是推荐使用jcmd命令。

## 2.8 打印所有带有堆栈跟踪的线程
`jcmd <process id/main class> Thread.print`


# 3、使用jcmd进行故障排除
jcmd提供以下排查故障的命令

## 3.1 开始记录
比如：对进程7060开始一个2分钟的记录，文件名字叫myrecording.jfr
`jcmd 7060 JFR.start name=MyRecording settings=profile delay=20s duration=2m filename=C:\TEMP\myrecording.jfr`
## 3.2 检查记录
JFR.check命令检查正在运行的记录
`jcmd 7060 JFR.check`
## 3.3 停止记录
JFR.stop诊断命令停止正在运行的记录，并具有丢弃记录数据的选项
`jcmd 7060 JFR.stop`
## 3.4 dump一个记录
诊断命令停止正在运行的记录，并具有将录音dump到文件中的选项
`jcmd 7060 JFR.dump name=MyRecording filename=C:\TEMP\myrecording.jfr`
## 3.5 创建堆dump
创建堆dump的首选方法是
`jcmd <pid> GC.heap_dump filename=Myheapdump`
## 3.6 创建堆直方图
创建堆直方图的首选方法是
`jcmd <pid> GC.class_histogram filename=Myheaphistogram`
# 4、基本命令
```java
jcmd [-l|-h|-help]
jcmd pid|main-class PerfCounter.print
jcmd pid|main-class -f filename
jcmd pid|main-class command[ arguments]
```
使用jcmd命令时没有参数或者-l，那么将输出所有java应用的pid和主类。

![](https://img.hacpai.com/file/2019/11/image-e5d09bd1.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

如果指定pid或者主类，那么jcmd将发送诊断命令请求到改应用，如果pid为0，表示发送命令到所有可用的java应用，使用下面其中一个作为诊断请求：
- Perfcounter.print
打印指定Java进程可用的性能计数器，性能计数器的列表可能因Java进程而异。
- -f filename
要从中读取诊断命令并将其发送到指定Java进程的文件的名称。
- command [arguments]
要发送到指定Java进程的命令，可以通过向该进程发送help命令来获得给定进程的可用诊断命令列表。

## 4.1 选项
- -h或-help：打印帮助信息
- -l：打印带有主类和命令行参数的运行Java进程标识符的列表。
- -f filename：从指定文件中读取命令。只有在将流程标识符或主类指定为第一个参数时，才能使用此选项。文件中的每个命令必须写在一行上。以数字符号(#)开头的行将被忽略。当读取所有行或包含stop关键字的行时，文件处理结束。
如图可以使用以下命令
![](https://img.hacpai.com/file/2019/11/image-3c55ab69.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

# 5、示例
`jcmd <process id/main class> PerfCounter.print`
在进程中打印所有性能计算器
![](https://img.hacpai.com/file/2019/11/image-80dd6a8c.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

`jcmd <process id/main class> <command> [options]`
将实际命令发送给JVM。

以下是使用jcmd实用工具向JVM显示诊断命令请求。
```java
> jcmd
5485 sun.tools.jcmd.JCmd
2125 MyProgram

> jcmd MyProgram help (or "jcmd 2125 help")
2125:
The following commands are available:
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
help

> jcmd MyProgram help Thread.print
2125:
Thread.print
Print all threads with stacktraces.

Impact: Medium: Depends on the number of threads.

Permission: java.lang.management.ManagementPermission(monitor)

Syntax : Thread.print [options]

Options: (options must be specified using the <key> or <key>=<value> syntax)
        -l : [optional] print java.util.concurrent locks (BOOLEAN, false)

> jcmd MyProgram Thread.print
2125:
2014-07-04 15:58:56
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.0-b69 mixed mode):

```