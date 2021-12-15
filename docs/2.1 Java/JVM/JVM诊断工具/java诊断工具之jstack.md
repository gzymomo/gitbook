[TOC]

# 1、简介
jstack需要指定具体的进程，打印虚拟机里所有线程的**堆栈跟踪信息**，大概可以知道以下这些信息：**java线程、虚拟机内部线程、本地堆栈帧、死锁检查**。

所有线程的堆栈跟踪可用于诊断许多问题，比如死锁或挂起。 使用-l选项可以在堆栈中查看同步器的情况，并打印相关java.util.concurrent.locks信息，如果没有设置-l选项，那么只显示监视器上的一些信息，也可以使用thread.getAllStackTraces方法来获取Thread dumps，或在调试器中使用调试器选项打印所有线程堆栈。
如果发现系统出现问题，那么可以jstack多次，比较每次的堆栈信息是否相同，如果相同的话就可能死循环了。

# 2、强制stack dump
有时候存在挂起的进程，导致jstack pid没有响应，那么可以使用-F选项来解决，可以强制stack dump，可以看示例：
`$ jstack -F 8321`

# 3、核心dump获取stack堆栈
为了获取核心dump的stack堆栈，可以从核心文件上执行jstack命令，如示例：
`jstack $JAVA_HOME/bin/java core`
注意：jdk12开始，可以使用jhsdb命令查看核心dump文件
`jhsdb jstack --exe java-home/bin/java --core core-file`

# 4、混合stack
jstack还可以打印混合stack，也就是说，他除了打印java堆栈之外，还可以打印本地堆栈帧，本地帧是c/c++编写的vm代码或JNI/native代码。打印混合stack可以使用-m参数，如示例：
`$ jstack -m 21177`

# 5、jstack相关选项
```java
jstack [options] pid
jstack [options] executable core
jstack [options] [server-id@] remote-hostname-or-IP
```
- options：命令行选项。
- pid：打印堆栈的进程id，这个进程必须是java进程，可以用个jps获取机器上所有的java进行列表。
- executable：生成核心dump的Java可执行文件。
- core：要打印堆栈跟踪的核心文件。
- remote-hostname-or-IP：远程调式服务器主机名或者ip地址。
- server-id@：当多个调试服务器运行在同一个远程主机上时，可以使用一个可选的惟一ID。

## 5.1 options解释
- -F：当jstack pid没有响应时强制进行堆栈dump。
- -l：可以打印一些附加信息，如java.util.concurrent的信息。如果使用了该配置，并且线程存在死锁，那么控制台会输出如下信息：
`Locked ownable synchronizers: - <0x000000076c637cd0> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)`
如果不存在死锁，那么输出：
`Locked ownable synchronizers: - None`
- -m：打印java和c++混合的堆栈信息。
- -h/-help：打印帮助信息。

## 5.2 输出到日志文件
可以把堆栈信息输出到具体的日志文件中，如
`jstack pid >>xxx.log`

# 6、脚本输出dump
```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo >&2 "Usage: jstackSeries  [ count [ delay ] ]"
    echo >&2 "    Defaults: count = 10, delay = 1 (seconds)"
    exit 1
fi

pid=$1          # required
count=${2:-10}  # defaults to 10 times
delay=${3:-1} # defaults to 1 second

while [ $count -gt 0 ]
do
    jstack $pid >jstack.$pid.$(date +%H%M%S.%N)
    sleep $delay
    let count--
    echo -n "."
done
```
调用规则：脚本名称 pid 统计次数 延迟几秒
可以这样使用脚本：
` jstackSeries  10 5 `