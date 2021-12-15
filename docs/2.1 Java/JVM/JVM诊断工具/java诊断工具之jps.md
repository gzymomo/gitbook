[TOC]

# 1、简介
jps会列出当前用户下的所有的java进程，jps在嵌入式VM中非常有用，使用JNI调用API而不是java启动程序启动它，在这些环境中，要识别java进程还是比较困难的。

# 2、示例
```bash
$ jps
16217 MyApplication
16342 jps
```
该实用程序列出了用户具有访问权限的虚拟机，这是由特定于操作系统的访问控制机制决定的，例如，在Oracle Solaris操作系统上，如果一个非root用户执行jps实用程序，那么输出的是用该用户的uid启动的虚拟机列表。除了列出PID之外，该实用程序还提供了一些选项来输出传递给应用程序主方法的参数、VM参数的完整列表和应用程序主类的完整包名。如果远程系统正在运行jstatd守护进程，jps实用程序还可以列出远程系统上的进程。
如果你在一个系统上运行几个Java Web Start应用程序，它们看起来是一样的，如：
```bash
$ jps
1271 jps
     1269 Main
     1190 Main
```
在上面例子中，使用jps -m来区分它们，如
```bash
$ jps -m
1271 jps -m
     1269 Main http://bugster.central.sun.com/bugster.jnlp
     1190 Main http://webbugs.sfbay/IncidentManager/incident.jnlp
```
列出远程jvm进程，如
```bash
jps -l remote.domain
3002 /opt/jdk1.7.0/demo/jfc/Java2D/Java2Demo.JAR
2857 sun.tools.jstatd.jstatd
```
下面的示例列出远程主机上的检测jvm，该主机具有RMI注册表的非默认端口。本例假设jstatd服务器在远程主机上运行，并将内部RMI注册表绑定到端口2002。本例还使用-m选项来包含传递给列出的每个Java应用程序的主方法的参数，如
```bash
jps -m remote.domain:2002
3002 /opt/jdk1.7.0/demo/jfc/Java2D/Java2Demo.JAR
3102 sun.tools.jstatd.jstatd -p 2002
```
# 3、语法
`jps [options] [hostid]`
- options：命令行选项。
- hostid：主机的唯一标识，应该为哪个进程生成报告。该值包含通信协议，端口号，具体的路径。
如果没有指定hostid，那么jps命令在本机上搜索java进程，如果指定了，那么会列出远程服务的java进程。jps会显示所有找到的java进程的唯一标识符（lvmid），lvmid通常是(但不一定是)JVM进程的操作系统进程标识符。没有设置选项，那么会列出lvmid和应用类的简称或者jar包名称，jps命令使用Java启动程序查找传递给主方法的类名和参数。如果目标JVM是用自定义启动程序启动的，那么类或JAR文件名以及主方法的参数是不可用的，此时类名或者包名会输出Unknown。

## 3.1 选项
-q：抑制传递给主方法的类名、JAR文件名和参数的输出，只生成本地JVM标识符列表。
-m：显示传递给主方法的参数。对于嵌入式jvm，输出可能为null。
-l：显示应用程序的主类的完整包名或应用程序JAR文件的完整路径名。
-v：显示传递给JVM的参数。
-V：抑制传递给主方法的类名、JAR文件名和参数的输出，只生成本地JVM标识符列表。
-Joption：将选项传递给JVM，其中选项是Java应用程序启动程序参考页面中描述的选项之一，例如：-J-Xms48m将启动内存设置为48mb。

## 3.2 主机标识符
主机标识符或hostid是指示目标系统的字符串，hostid字符串的语法对应于URI的语法：
`[protocol:][[//]hostname][:port][/servername]`
- protocol：通信协议。如果省略了协议，并且没有指定主机名，那么默认协议是特定于平台的、经过优化的本地协议。如果省略了协议并指定了主机名，则默认协议是rmi。
- hostname：指示目标主机的主机名或IP地址，如果省略了hostname参数，那么目标主机就是本地主机。
- port：与远程服务器通信的默认端口。如果省略主机名参数或协议参数指定优化的本地协议，则忽略端口参数。否则，端口参数的处理是特定于实现的。对于默认的rmi协议，port参数指示远程主机上rmiregistry的端口号。如果省略端口参数，协议参数指示rmi，则使用默认的rmiregistry端口(1099)。
- servername：该参数的处理取决于实现。对于优化的本地协议，该字段将被忽略。对于rmi协议，这个参数是一个字符串，表示远程主机上rmi远程对象的名称。

## 3.3 输出格式
jps命令的输出遵循以下模式：
`lvmid [ [ classname | JARfilename | "Unknown"] [ arg* ] [ jvmarg* ] ]`
所有输出标记都用空格分隔。当试图将参数映射到它们的实际位置参数时，包含嵌入式空白的arg值会产生歧义。