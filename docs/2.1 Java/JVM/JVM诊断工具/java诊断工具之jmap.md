[TOC]

# 1、简介
jmap命令用来统计运行的vm或者核心文件的内存相关信息，
该实用程序还可以使用jsadebugd守护进程查询远程机器上的进程或核心文件。注意:在这种情况下，输出需要更长的时间来打印。
在jdk8中可以使用jcmd命令来代替jmap命令，jcmd可以增强诊断功能并降低性能开销。
如果使用jmap的时候可以指定选项，那么会打印加载的共享对象列表，可以指定以下选项来查看指定信息：-heap，-histo，-permstat。
在jdk7中可以指定`-dump:format=b,file=filename`选项，该选项可以把dump文件输出到指定文件夹中，然后可以使用jhat工具来进行分析。
如果使用jmap时，进行被hung住了没有响应，那么可以使用-F选项来强制执行。

# 2、堆配置和使用
-heap可以获取以下java堆信息：
1. gc算法的信息，包含gc算法的名称（如：parallel GC），具体算法的详情（如：parallel GC的线程数）
2. 堆配置，可以指定为命令行选项，也可以由VM根据机器配置选择。
3. 堆使用概要：对于每一代(堆的区域)，该工具打印总堆容量、正在使用的内存和可用的空闲内存，例如像新生代会显示他内存的概要信息。
查看示例：
```java
$ jmap -heap 29620

Attaching to process ID 29620, please wait...

Debugger attached successfully.

Client compiler detected.
JVM version is 1.6.0-rc-b100

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 67108864 (64.0MB)
   NewSize          = 2228224 (2.125MB)
   MaxNewSize       = 4294901760 (4095.9375MB)
   OldSize          = 4194304 (4.0MB)
   NewRatio         = 8
   SurvivorRatio    = 8
   PermSize         = 12582912 (12.0MB)
   MaxPermSize      = 67108864 (64.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 2031616 (1.9375MB)
   used     = 70984 (0.06769561767578125MB)
   free     = 1960632 (1.8698043823242188MB)
   3.4939673639112905% used
Eden Space:
   capacity = 1835008 (1.75MB)
   used     = 36152 (0.03447723388671875MB)
   free     = 1798856 (1.7155227661132812MB)
   1.9701276506696428% used
From Space:
   capacity = 196608 (0.1875MB)
   used     = 34832 (0.0332183837890625MB)
   free     = 161776 (0.1542816162109375MB)
   17.716471354166668% used
To Space:
   capacity = 196608 (0.1875MB)
   used     = 0 (0.0MB)
   free     = 196608 (0.1875MB)
   0.0% used
tenured generation:
   capacity = 15966208 (15.2265625MB)
   used     = 9577760 (9.134063720703125MB)
   free     = 6388448 (6.092498779296875MB)
   59.98769400974859% used
Perm Generation:
   capacity = 12582912 (12.0MB)
   used     = 1469408 (1.401336669921875MB)
   free     = 11113504 (10.598663330078125MB)
   11.677805582682291% used
```

# 3、堆直方图
使用-histo可以获取类的堆直方图， 当在运行的进程上执行该命令时，该工具将打印对象的数量、内存大小(以字节为单位)以及每个类的完全限定类名，Java HotSpot VM中的内部类用尖括号括起来。直方图对理解堆的使用很有帮助，要获得对象的大小，必须将总大小除以该对象类型的计数。
查看示例：
```java
$ jmap -histo 29620
num   #instances    #bytes  class name
--------------------------------------
  1:      1414     6013016  [I
  2:       793      482888  [B
  3:      2502      334928  <constMethodKlass>
  4:       280      274976  <instanceKlassKlass>
  5:       324      227152  [D
  6:      2502      200896  <methodKlass>
  7:      2094      187496  [C
  8:       280      172248  <constantPoolKlass>
  9:      3767      139000  [Ljava.lang.Object;
 10:       260      122416  <constantPoolCacheKlass>
 11:      3304      112864  <symbolKlass>
 12:       160       72960  java2d.Tools$3
 13:       192       61440  <objArrayKlassKlass>
 14:       219       55640  [F
 15:      2114       50736  java.lang.String
 16:      2079       49896  java.util.HashMap$Entry
 17:       528       48344  [S
 18:      1940       46560  java.util.Hashtable$Entry
 19:       481       46176  java.lang.Class
 20:        92       43424  javax.swing.plaf.metal.MetalScrollButton
... more lines removed here to reduce output...
1118:         1           8  java.util.Hashtable$EmptyIterator
1119:         1           8  sun.java2d.pipe.SolidTextRenderer
Total    61297    10152040
```
当jmap -histo命令在核心文件上执行时，该工具将打印每个类的大小、数量和类名，Java HotSpot VM中的内部类以星号(*)作为前缀。
在核心文件中使用jmap命令：
```java
& jmap -histo /net/koori.sfbay/onestop/jdk/6.0/promoted/all/b100/binaries/solaris-sparcv9/bin/java core

Attaching to core core from executable /net/koori.sfbay/onestop/jdk/6.0/
promoted/all/b100/binaries/solaris-sparcv9/bin/java, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 1.6.0-rc-b100
Iterating over heap. This may take a while...
Heap traversal took 8.902 seconds.

Object Histogram:

Size    Count    Class description
-------------------------------------------------------
4151816    2941    int[]
2997816    26403    * ConstMethodKlass
2118728    26403    * MethodKlass
1613184    39750    * SymbolKlass
1268896    2011    * ConstantPoolKlass
1097040    2011    * InstanceKlassKlass
882048    1906    * ConstantPoolCacheKlass
758424    7572    char[]
733776    2518    byte[]
252240    3260    short[]
214944    2239    java.lang.Class
177448    3341    * System ObjArray
176832    7368    java.lang.String
137792    3756    java.lang.Object[]
121744    74    long[]
72960    160    java2d.Tools$3
63680    199    * ObjArrayKlassKlass
53264    158    float[]
... more lines removed here to reduce output...
```

# 4、永久代信息统计
永久代是堆的区域，它包含虚拟机本身的所有反射数据，比如类和方法对象，在Java虚拟机规范中，这个区域也称为“方法区域”。
对于动态生成和加载大量类的应用配置永久代的大小非常重要（如java服务或者web容器），如果一个应用加载了太大的类，那么会报以下的错误：
`Exception in thread thread_name java.lang.OutOfMemoryError: PermGen space`
为了获取更多的永久代信息，你可以指定-permstat选项。
查看示例：
```java
$ jmap -permstat 29620

Attaching to process ID 29620, please wait...
Debugger attached successfully.
Client compiler detected.
JVM version is 1.6.0-rc-b100
12674 intern Strings occupying 1082616 bytes.
finding class loader instances ..Unknown oop at 0xd0400900
Oop's klass is 0xd0bf8408
Unknown oop at 0xd0401100
Oop's klass is null
done.
computing per loader stat ..done.
please wait.. computing liveness.........................................done.
class_loader    classes bytes   parent_loader   alive?  type

<bootstrap>     1846 5321080  null        live   <internal>
0xd0bf3828  0      0      null         live    sun/misc/Launcher$ExtClassLoader@0xd8c98c78
0xd0d2f370  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0c99280  1   1440      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0b71d90  0      0   0xd0b5b9c0    live java/util/ResourceBundle$RBClassLoader@0xd8d042e8
0xd0d2f4c0  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0b5bf98  1    920   0xd0b5bf38      dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0c99248  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f488  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0b5bf38  6   11832  0xd0b5b9c0      dead    sun/reflect/misc/MethodUtil@0xd8e8e560
0xd0d2f338  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f418  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f3a8  1    904     null          dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0b5b9c0  317 1397448 0xd0bf3828     live    sun/misc/Launcher$AppClassLoader@0xd8cb83d8
0xd0d2f300  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f3e0  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0ec3968  1   1440      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0e0a248  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0c99210  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f450  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0d2f4f8  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50
0xd0e0a280  1    904      null         dead    sun/reflect/DelegatingClassLoader@0xd8c22f50

total = 22      2186    6746816   N/A   alive=4, dead=18       N/A
```
对于每个类加载器，会打印以下信息：
1. 类加载器对象在运行实用程序时快照处的地址
2. 类加载器的数量
3. 类加载器加载的所有类的metadata所消耗的字节数
4. 父类加载器的地址(如果有的话)
5. 指示加载器对象将来是否将被垃圾收集的活动或死活动
6. 这个类加载器的类名

# 5、基本语法
打印进程、核心文件或远程调试服务器的共享对象内存映射或堆内存详细信息。
```java
jmap [ options ] pid
jmap [ options ] executable core
jmap [ options ] [ pid ] server-id@ ] remote-hostname-or-IP
```
options：命令行选项
- pid：要打印内存映射的进程ID，这个进程必须是java进程，可以使用jps获取所有java进程。
- executable：生成核心dump文件。
- core：要打印内存映射的核心文件。
- remote-hostname-or-IP：远程服务地址或ip地址
- server-id：当多个调试服务器运行在同一个远程主机上时，可以使用一个可选的惟一ID。

## 5.1 描述
jmap命令打印共享对象内存或者堆内存信息为指定的进程、核心文件、远程服务，如果指定的进程运行在64位Java虚拟机(JVM)上，那么你可能需要指定-J-d64选项，如：jmap -J-d64 -heap pid
注意：该命令在之后的版本中可能会被弃用。

## 5.2 选项
- <no option>：当没有选项被使用时，jmap命令打印共享对象映射，对于目标JVM中加载的每个共享对象，将打印开始地址、映射大小和共享对象文件的完整路径。与pmap命令很相似。
- -dump:[live,] format=b, file=filename：将hprof二进制格式的Java堆转储到文件名，live子选项是可选的，但在指定时，仅dump堆中的活动对象。如果想要浏览堆dump文件，你可以使用jhat命令。
- -finalizerinfo：打印关于等待结束的对象的信息。
- -heap：打印使用的垃圾收集、头配置和generation-wise堆使用情况，此外，还打印了字符串的数量和大小。
- -histo[:live]：打印堆的直方图，对于每个Java类，打印对象的数量、以字节为单位的内存大小和完全限定的类名，JVM内部类名用星号(*)前缀打印，如果指定了live子选项，则只计算活动对象。
- -clstats：打印Java堆的类加载器wise统计信息，对于每个类加载器，将打印它的名称、它的活动程度、地址、父类加载器以及它加载的类的数量和大小。
- -F：强制，当pid没有响应的时候，可以使用该选项，此模式不支持live子选项。
- -h：打印帮助信息。
- -help：打印帮助信息。
- -Jflag：将标志传递到运行jmap命令的Java虚拟机。