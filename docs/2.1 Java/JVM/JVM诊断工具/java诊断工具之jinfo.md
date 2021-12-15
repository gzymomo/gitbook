[TOC]

# 1、简介
jinfo命令从运行的java进程或者dump文件中获取配置信息，并打印系统属性或者命令行用于启动JVM的命令行标志。
在jdk8中建议使用jcmd命令来查看这些信息。
jinfo还可以使用jsadebugd守护进程查询远程机器上的进程或核心文件，注意:在这种情况下，输出需要更长的时间来打印。
使用-flag选项，jinfo可以为指定的Java进程动态设置、取消设置或更改特定JVM标志的值。

# 2、基本语法
```java
jinfo [ option ] pid
jinfo [ option ] executable core
jinfo [ option ] [ servier-id ] remote-hostname-or-IP
```
option:命令行选项。
- pid：需要打印信息的进程id，这个进程必须是java进程，可以使用jps命令获取java进程列表。
- executable：可以执行dump文件。
- core：要打印配置信息的核心文件。
- remote-hostname-or-IP：远程服务ip或者hostname。
- server-id：当多个调试服务器运行在同一个远程主机上时，可以使用一个可选的惟一ID。
- jinfo命令打印指定Java进程或核心文件或远程调试服务器的Java配置信息。配置信息包括Java系统属性和Java虚拟机(JVM)命令行标志。如果指定的进程在64位JVM上运行，那么可能需要指定-J-d64选项，例如:jinfo -J-d64 -sysprops pid。

## 2.1 选项
- no-option：没有设置选项，同时打印命令行标志和系统属性名称-值对。
- -flag name：打印指定命令行标志的名称和值。
- -flag [+|-]name：启用或禁用指定的布尔命令行标志。
- -flag name=value：将指定的命令行标志设置为指定的值。
- -flags：打印传递给JVM的命令行标志。
- -sysprops：将Java系统属性打印为名称-值对。
- -h：打印帮助信息。
- -help：打印帮助信息。

# 3、示例
## 3.1 示例1
以下示例不设置任何选项
```java
$ jinfo 29620

Attaching to process ID 29620, please wait...
Debugger attached successfully.
Client compiler detected.
JVM version is 1.6.0-rc-b100
Java System Properties:

java.runtime.name = Java(TM) SE Runtime Environment
sun.boot.library.path = /usr/jdk/instances/jdk1.6.0/jre/lib/sparc
java.vm.version = 1.6.0-rc-b100
java.vm.vendor = Sun Microsystems Inc.
java.vendor.url = http://java.sun.com/
path.separator = :
java.vm.name = Java HotSpot(TM) Client VM
file.encoding.pkg = sun.io
sun.java.launcher = SUN_STANDARD
sun.os.patch.level = unknown
java.vm.specification.name = Java Virtual Machine Specification
user.dir = /home/js159705
java.runtime.version = 1.6.0-rc-b100
java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
java.endorsed.dirs = /usr/jdk/instances/jdk1.6.0/jre/lib/endorsed
os.arch = sparc
java.io.tmpdir = /var/tmp/
line.separator =

java.vm.specification.vendor = Sun Microsystems Inc.
os.name = SunOS
sun.jnu.encoding = ISO646-US
java.library.path = /usr/jdk/instances/jdk1.6.0/jre/lib/sparc/client:/usr/jdk/instances/jdk1.6.0/jre/lib/sparc:
/usr/jdk/instances/jdk1.6.0/jre/../lib/sparc:/net/gtee.sfbay/usr/sge/sge6/lib/sol-sparc64:
/usr/jdk/packages/lib/sparc:/lib:/usr/lib
java.specification.name = Java Platform API Specification
java.class.version = 50.0
sun.management.compiler = HotSpot Client Compiler
os.version = 5.10
user.home = /home/js159705
user.timezone = US/Pacific
java.awt.printerjob = sun.print.PSPrinterJob
file.encoding = ISO646-US
java.specification.version = 1.6
java.class.path = /usr/jdk/jdk1.6.0/demo/jfc/Java2D/Java2Demo.jar
user.name = js159705
java.vm.specification.version = 1.0
java.home = /usr/jdk/instances/jdk1.6.0/jre
sun.arch.data.model = 32
user.language = en
java.specification.vendor = Sun Microsystems Inc.
java.vm.info = mixed mode, sharing
java.version = 1.6.0-rc
java.ext.dirs = /usr/jdk/instances/jdk1.6.0/jre/lib/ext:/usr/jdk/packages/lib/ext
sun.boot.class.path = /usr/jdk/instances/jdk1.6.0/jre/lib/resources.jar:
/usr/jdk/instances/jdk1.6.0/jre/lib/rt.jar:/usr/jdk/instances/jdk1.6.0/jre/lib/sunrsasign.jar:
/usr/jdk/instances/jdk1.6.0/jre/lib/jsse.jar:
/usr/jdk/instances/jdk1.6.0/jre/lib/jce.jar:/usr/jdk/instances/jdk1.6.0/jre/lib/charsets.jar:
/usr/jdk/instances/jdk1.6.0/jre/classes
java.vendor = Sun Microsystems Inc.
file.separator = /
java.vendor.url.bug = http://java.sun.com/cgi-bin/bugreport.cgi
sun.io.unicode.encoding = UnicodeBig
sun.cpu.endian = big
sun.cpu.isalist =

VM Flags:
```

## 3.2 示例2
使用核心文件
`$ jinfo $JAVA_HOME/bin/java core.29620`

## 3.3 示例3
```java
$ jinfo -flags 1843
Attaching to process ID 1843, please wait...
Debugger attached successfully.
Client compiler detected.
JVM version is 25.5-b02
Non-default VM flags: -XX:+AlwaysPreTouch -XX:CMSInitiatingOccupancyFraction=75 -XX:ErrorFile=null -XX:GCLogFileSize=67108864 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=null -XX:InitialHeapSize=1073741824 -XX:MaxHeapSize=1073741824 -XX:MaxNewSize=67108864 -XX:MaxTenuringThreshold=6 -XX:MinHeapDeltaBytes=131072 -XX:NewSize=67108864 -XX:NumberOfGCLogFiles=32 -XX:OldPLABSize=16 -XX:OldSize=1006632960 -XX:-OmitStackTraceInFastThrow -XX:+PrintGC -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:ThreadStackSize=1024 -XX:+UseCMSInitiatingOccupancyOnly -XX:+UseConcMarkSweepGC -XX:+UseGCLogFileRotation -XX:+UseParNewGC 
Command line:  -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/tmp/elasticsearch-7009776053354731844 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime -Xloggc:logs/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=32 -XX:GCLogFileSize=64m -Dio.netty.allocator.type=unpooled -Des.path.home=/home/elasticsearch-7.1.1 -Des.path.conf=/home/elasticsearch-7.1.1/config -Des.distribution.flavor=default -Des.distribution.type=tar -Des.bundled_jdk=true
```

## 3.4 示例4
`$ jinfo -flag CMSInitiatingOccupancyFraction 1843
-XX:CMSInitiatingOccupancyFraction=75`

## 3.5 示例5
```java
$ jinfo -sysprops 1843
Attaching to process ID 1843, please wait...
Debugger attached successfully.
Client compiler detected.
JVM version is 25.5-b02
java.runtime.name = Java(TM) SE Runtime Environment
sun.boot.library.path = /home/jdk1.8.0_05/jre/lib/i386
java.vm.version = 25.5-b02
es.path.home = /home/elasticsearch-7.1.1
log4j.shutdownHookEnabled = false
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = :
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) Client VM
sun.os.patch.level = unknown
user.country = US
sun.java.launcher = SUN_STANDARD
es.networkaddress.cache.negative.ttl = 10
jna.nosys = true
java.vm.specification.name = Java Virtual Machine Specification
user.dir = /home/elasticsearch-7.1.1
java.runtime.version = 1.8.0_05-b13
java.awt.graphicsenv = sun.awt.X11GraphicsEnvironment
java.endorsed.dirs = /home/jdk1.8.0_05/jre/lib/endorsed
os.arch = i386
java.io.tmpdir = /tmp/elasticsearch-7009776053354731844
line.separator =

es.networkaddress.cache.ttl = 60
es.logs.node_name = localhost.localdomain
java.vm.specification.vendor = Oracle Corporation
os.name = Linux
io.netty.noKeySetOptimization = true
sun.jnu.encoding = UTF-8
java.library.path = /usr/java/packages/lib/i386:/lib:/usr/lib
sun.nio.ch.bugLevel = 
es.logs.cluster_name = elasticsearch
java.specification.name = Java Platform API Specification
java.class.version = 52.0
sun.management.compiler = HotSpot Client Compiler
os.version = 2.6.32-431.el6.i686
user.home = /home/elsearch
user.timezone = PRC
java.awt.printerjob = sun.print.PSPrinterJob
file.encoding = UTF-8
java.specification.version = 1.8
es.distribution.type = tar
io.netty.recycler.maxCapacityPerThread = 0
io.netty.allocator.type = unpooled
user.name = elsearch
es.logs.base_path = /home/elasticsearch-7.1.1/logs
java.class.path = /home/elasticsearch-7.1.1/lib/jackson-core-2.8.11.jar:/home/elasticsearch-7.1.1/lib/log4j-1.2-api-2.11.1.jar:/home/elasticsearch-7.1.1/lib/lucene-core-8.0.0.jar:/home/elasticsearch-7.1.1/lib/spatial4j-0.7.jar:/home/elasticsearch-7.1.1/lib/lucene-misc-8.0.0.jar:/home/elasticsearch-7.1.1/lib/lucene-analyzers-common-8.0.0.jar:/home/elasticsearch-7.1.1/lib/lucene-spatial-extras-8.0.0.jar:/home/elasticsearch-7.1.1/lib/jackson-dataformat-yaml-2.8.11.jar:/home/elasticsearch-7.1.1/lib/lucene-queryparser-8.0.0.jar:/home/elasticsearch-7.1.1/lib/log4j-core-2.11.1.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-7.1.1.jar:/home/elasticsearch-7.1.1/lib/snakeyaml-1.17.jar:/home/elasticsearch-7.1.1/lib/joda-time-2.10.1.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-x-content-7.1.1.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-geo-7.1.1.jar:/home/elasticsearch-7.1.1/lib/lucene-spatial-8.0.0.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-core-7.1.1.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-secure-sm-7.1.1.jar:/home/elasticsearch-7.1.1/lib/java-version-checker-7.1.1.jar:/home/elasticsearch-7.1.1/lib/lucene-join-8.0.0.jar:/home/elasticsearch-7.1.1/lib/lucene-highlighter-8.0.0.jar:/home/elasticsearch-7.1.1/lib/lucene-queries-8.0.0.jar:/home/elasticsearch-7.1.1/lib/lucene-suggest-8.0.0.jar:/home/elasticsearch-7.1.1/lib/jackson-dataformat-smile-2.8.11.jar:/home/elasticsearch-7.1.1/lib/lucene-grouping-8.0.0.jar:/home/elasticsearch-7.1.1/lib/jna-4.5.1.jar:/home/elasticsearch-7.1.1/lib/jts-core-1.15.0.jar:/home/elasticsearch-7.1.1/lib/hppc-0.7.1.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-launchers-7.1.1.jar:/home/elasticsearch-7.1.1/lib/log4j-api-2.11.1.jar:/home/elasticsearch-7.1.1/lib/jackson-dataformat-cbor-2.8.11.jar:/home/elasticsearch-7.1.1/lib/lucene-backward-codecs-8.0.0.jar:/home/elasticsearch-7.1.1/lib/elasticsearch-cli-7.1.1.jar:/home/elasticsearch-7.1.1/lib/plugin-classloader-7.1.1.jar:/home/elasticsearch-7.1.1/lib/t-digest-3.2.jar:/home/elasticsearch-7.1.1/lib/lucene-spatial3d-8.0.0.jar:/home/elasticsearch-7.1.1/lib/HdrHistogram-2.1.9.jar:/home/elasticsearch-7.1.1/lib/lucene-sandbox-8.0.0.jar:/home/elasticsearch-7.1.1/lib/jopt-simple-5.0.2.jar:/home/elasticsearch-7.1.1/lib/lucene-memory-8.0.0.jar
es.path.conf = /home/elasticsearch-7.1.1/config
java.vm.specification.version = 1.8
java.home = /home/jdk1.8.0_05/jre
sun.java.command = org.elasticsearch.bootstrap.Elasticsearch
sun.arch.data.model = 32
io.netty.noUnsafe = true
user.language = en
java.specification.vendor = Oracle Corporation
awt.toolkit = sun.awt.X11.XToolkit
java.vm.info = mixed mode
java.version = 1.8.0_05
java.ext.dirs = /home/jdk1.8.0_05/jre/lib/ext:/usr/java/packages/lib/ext
sun.boot.class.path = /home/jdk1.8.0_05/jre/lib/resources.jar:/home/jdk1.8.0_05/jre/lib/rt.jar:/home/jdk1.8.0_05/jre/lib/sunrsasign.jar:/home/jdk1.8.0_05/jre/lib/jsse.jar:/home/jdk1.8.0_05/jre/lib/jce.jar:/home/jdk1.8.0_05/jre/lib/charsets.jar:/home/jdk1.8.0_05/jre/lib/jfr.jar:/home/jdk1.8.0_05/jre/classes
java.awt.headless = true
java.vendor = Oracle Corporation
es.bundled_jdk = true
file.separator = /
java.vendor.url.bug = http://bugreport.sun.com/bugreport/
es.distribution.flavor = default
sun.io.unicode.encoding = UnicodeLittle
sun.cpu.endian = little
log4j2.disable.jmx = true
sun.cpu.isalist =
```