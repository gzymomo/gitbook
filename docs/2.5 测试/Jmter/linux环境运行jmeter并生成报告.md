# [linux环境运行jmeter并生成报告](https://www.cnblogs.com/imyalost/p/9808079.html)

jmeter是一个java开发的利用多线程原理来模拟并发进行性能测试的工具，一般来说，GUI模式只用于创建脚本以及用来debug，执行测试时建议使用非GUI模式运行。

这篇博客，介绍下在linux环境利用jmeter进行性能测试的方法，以及如何生成测试报告。。。

 

**一、为什么要非GUI模式运行**

jmeter是java语言开发，实际是运行在JVM中的，GUI模式运行需要耗费较多的系统资源，一般来说，GUI模式要占用10%-25%的系统资源。

而使用非GUI模式（即linux或dos命令）可以降低对资源的消耗，提升单台负载机所能模拟的并发数。

启动jmeter，提醒如下：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181017235354528-1410994690.png)

 

**二、环境准备**

**1、安装JDK**

关于如何在linux环境安装JDK，可参考我之前的博客：[linux下安装JDK](https://www.cnblogs.com/imyalost/p/8709578.html)

**2、安装jmeter**

官方地址：[Download Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi)

下载压缩包，然后将安装包上传至linux服务器，一般有以下2种方式：

①、通过FileZilla或其他类似工具上传至linux服务器；

②、直接将zip文件拖至linux服务器；

方法如下：

输入命令 yum install -y lrzsz ，安装linux下的上传和下载功能包，然后将jmeter压缩包拖进去即可，示例如下：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181018000448933-287232329.png)

然后输入命令 unzip jmeter-3.2.zip ，进行解压。

**3、配置环境变量**

输入命令 vim /etc/profile ，在最下面添加如下内容：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
export JAVA_HOME=/usr/java/1.8.0_181
export CLASSPATH=.:JAVA_HOME/lib/dt.jar:$JAVA_HOME.lib/tools.jar:$JRE_HOME.lib
export PATH=$PATH:$JAVA_HOME/lib
export PATH=/jmeter/apache-3.2/bin/:$PATH
export JMETER_HOME=/jmeter/jmeter-3.2
export CLASSPATH=$JMETER_HOME/lib/ext/ApacheJMeter_core.jar:$JMETER_HOME/lib/jorphan.jar:$CLASSPATH
export PATH=$JMETER_HOME/bin:$PATH
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 保存后，输入命令 source /etc/profile ,使修改的配置生效。

**4、授予权限**

在执行jmeter脚本执行，首先要确保监控工具、jmeter以及相关的文件有相应的权限，否则会报错，常见的报错如下：

①、文件没有权限

②、无法打开目录下的文件

③、编码格式错误

查看文件或工具是权限的命令如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 查看当前目录下所有文件的权限
ls -l     
# 查看当前目录下所有文件的权限
ll
# 查看某个文件的权限
ls -l filename
# 查看某个目录的权限
ls ld /path
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 **5、linux文件颜色代表的含义**

在linux中，不同颜色的文件代表不同的含义，下面是linux中不同颜色的文件代表的含义：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 白色：普通的文件
# 蓝色：目录
# 绿色：可执行的文件
# 红色：压缩文件或者包文件
# 青色：连接文件
# 黄色：设备文件
# 灰色：其他的文件
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**三、运行jmeter**

**1、启动jmeter，创建脚本**

这里以访问我博客首页为例：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181018001153966-2055522873.png)

脚本保存为test.jmx，然后将文件上传至linux服务器。

**2、运行脚本**

将脚本上传至linux服务器，然后进入jmeter的bin目录下，输入命令 jmeter -n -t test.jmx -l test.jtl ，运行jmeter脚本。

PS：常用命令解析：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 常见命令说明
-h 帮助：打印出有用的信息并退出
-n 非 GUI 模式：在非 GUI 模式下运行 JMeter
-t 测试文件：要运行的 JMeter 测试脚本文件
-l 日志文件：记录结果的文件
-r 远程执行：启动远程服务
-H 代理主机：设置 JMeter 使用的代理主机
-P 代理端口：设置 JMeter 使用的代理主机的端口号
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 运行结果如下图：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181019002930250-378046106.png)

**3、查看测试报告**

启动jmeter，新建一个线程组，添加所需的监听器，导入脚本运行产生的.jtl文件，如下：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181019003254348-730225769.png)