**一、认识nmon**

**1、简介**

nmon是一种在AIX与各种Linux操作系统上广泛使用的监控与分析工具，它能在系统运行过程中实时地捕捉系统资源的使用情况，记录的信息比较全面，

并且能输出结果到文件中，然后通过nmon_analyzer工具产生数据文件与图形化结果。

**2、nmon可监控的数据类型**

内存使用情况

磁盘适配器

文件系统中的可用空间

CPU使用率

页面空间和页面速度

异步I/O，仅适用于AIX

网络文件系统（NFS）

磁盘I/O速度和读写比率

服务器详细信息和资源

内核统计信息

消耗资源最多的进程

运行队列信息

**3、特点**

①、占用系统资源少（一般不到2%）

②、功能强大（监控数据类型全面）

③、结合grafana之类的仪表图，可以更直观的实时展示所监控的数据

④、移植性、兼容性较好

 

**二、检查安装环境**

```
# 查看操作系统的信息
uname -a 
# 查看linux发行版本 
lsb_release -a
```

如下图，我的操作系统为64位，linux版本为CentOS7.4版本：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[root@izbp1jbg0c2bbcmcba0exoz ~]# uname -a
Linux izbp1jbg0c2bbcmcba0exoz 3.10.0-693.2.2.el7.x86_64 #1 SMP Tue Sep 12 22:26:13 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@izbp1jbg0c2bbcmcba0exoz ~]# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID:    CentOS
Description:    CentOS Linux release 7.4.1708 (Core) 
Release:    7.4.1708
Codename:    Core
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**三、nmon下载安装**

**1、官方地址**：http://nmon.sourceforge.net/pmwiki.php?n=Site.Download

根据我的操作系统和linux版本，选择对应的支持版本，如下：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013013754000-838618535.png)

**2、下载方式**

①、下载到本地，通过FTP上传到服务器

②、命令行 wget http://sourceforge.net/projects/nmon/files/nmon16e_mpginc.tar.gz 

**3、安装**

下载完成后，执行以下命令：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# 新建一个nmon文件夹
mkdir nmon
# 解压
tar xvfz nmon16e_mpginc.tar.gz
# 改名
mv nmon_x86_64_centos7 /root/nmon
# 给工具授权
chmod -x nmon 777
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**四、运行nmon**

完成上面的操作后，执行 ./nmon 命令，出现如下界面，说明安装成功：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013013239998-1034085493.png)

常用快捷命令说明：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# c
查看CPU相关信息
# m
查看内存相关信息
# d          
查看磁盘相关信息
# n          
查看网络相关信息
# t
查看相关进程信息
# h          
查看帮助相关信息
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

输入如上几种命令，结果如下图显示：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013205702111-994375377.png)

 

**五、采集数据**

nmon通过命令行启动监控，捕获服务器的各项数据，命令如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
./nmon -ft -s 10 -c 60 -m /root/nmon 
# 参数说明 
-f   监控结果以文件形式输出，默认机器名+日期.nmon格式 
-F   指定输出的文件名，比如test.nmon 
-s   指的是采样的频率，单位为毫秒 
-c   指的是采样的次数，即以上面的采样频率采集多少次 
-m   指定生成的文件目录 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**PS：**一般来说不建议对稳定性测试使用nmon监控，因为生成的nmon文件超过10M时，分析工具会由于内存不足导致报错。

如果必须进行的话，建议加大采样频次，降低采样次数（低于330次）。

 

**六、监控结果分析**

**1、下载分析工具**

nmon监控捕获的信息，一般用nmon_analyser来进行分析。nmon_analyser 由IBM提供， 使用excel的宏命令分析加载生成excel图表，展示资源占用的各项信息。

**官网地址：[nmon_analyser](https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power Systems/page/nmon_analyser)**

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013010626366-839900566.png)

下载你需要的版本，然后解压，解压后出现如下2个文件：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013010720194-2028759813.png)

**2、使用nmon analyser工具**

打开**.xlsm**文件，点击**Analyze nmon data**，打开你需要进行分析的nmon监控文件：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013214910356-967728657.png)

PS：如果提示分析文件不可用，从“工具-宏-安全性”启动宏，然后再次打开文件，即可使用该分析文件。

**3、生成各种图表数据**

通过分析工具生成的监控数据结果如下图：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013215613524-977379276.png)

红色标注区域为采集的监控数据，选择自己需要的类型（比如cpu），然后筛选对应的服务Pid（比如1314），选择对应的数据类型（比如CPU使用率占比），

通过excel提供的各种图形生成工具，生成直观的分析结果图。比如：

![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013220131905-779859331.png)![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013220139725-435210663.png)![img](https://img2018.cnblogs.com/blog/983980/201810/983980-20181013220155157-1629382074.png)