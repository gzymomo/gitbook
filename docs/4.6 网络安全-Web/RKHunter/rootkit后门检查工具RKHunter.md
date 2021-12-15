- [rootkit后门检查工具RKHunter](https://www.cnblogs.com/xiao987334176/p/11957974.html)



# 一、概述

## 简介

中文名叫”Rootkit猎手”,

rkhunter是Linux系统平台下的一款开源入侵检测工具，具有非常全面的扫描范围，除了能够检测各种已知的rootkit特征码以外，还支持端口扫描、常用程序文件的变动情况检查。

rkhunter的官方网站位于http://www.rootkit.nl/，注意：官网不能直接打开，必须要能访问谷歌才行。

 

目前最新的版本是rkhunter-1.4.6。

源码下载链接：

```
http://jaist.dl.sourceforge.net/project/rkhunter/rkhunter/1.4.6/rkhunter-1.4.6.tar.gz
```

 

## rootkit是什么？

  rootkit是Linux平台下最常见的一种木马后门工具，它主要通过替换系统文件来达到入侵和和隐蔽的目的，这种木马比普通木马后门更加危险和隐蔽，普通的检测工具和检查手段很难发现这种木马。rootkit攻击能力极强，对系统的危害很大，它通过一套工具来建立后门和隐藏行迹，从而让攻击者保住权限，以使它在任何时候都可以使用root 权限登录到系统。

​      rootkit主要有两种类型：文件级别和内核级别。

  文件级别的rootkit:  一般是通过程序漏洞或者系统漏洞进入系统后，通过修改系统的重要文件来达到隐藏自己的目的。在系统遭受rootkit攻击后，合法的文件被木马程序替代，变成了外壳程序，而其内部是隐藏着的后门程序。通常容易被rootkit替换的系统程序有login、ls、ps、ifconfig、du、find、netstat等。文件级别的rootkit，对系统维护很大，目前最有效的防御方法是定期对系统重要文件的完整性进行检查，如Tripwire、aide等。 

  内核级rootkit:  是比文件级rootkit更高级的一种入侵方式，它可以使攻击者获得对系统底层的完全控制权，此时攻击者可以修改系统内核，进而截获运行程序向内核提交的命令，并将其重定向到入侵者所选择的程序并运行此程序。内核级rootkit主要依附在内核上，它并不对系统文件做任何修改。以防范

为主。

 

# 二、安装rkhunter

## 环境说明

操作系统：centos 6.9

目前有一台服务器，负载比较高，cpu使用率为：85%。

使用top查看进程，发现最高的进程，在1%左右。一直不明白，为何负载如此之高，怀疑中了木马程序。

 

## 安装

直接yum安装

```
yum install -y rkhunter
```

 

## 在线升级rkhunter

rkhunter是通过一个含有rootkit名字的数据库来检测系统的rootkits漏洞, 所以经常更新该数据库非常重要, 你可以通过下面命令来更新该数据库:

```
rkhunter --update
```

 

为基本系统程序建立校对样本，建议系统安装完成后就建立

```
rkhunter --propupd
```

 

## 运行rkhunter检查系统

它主要执行下面一系列的测试:

  \1. MD5校验测试, 检测任何文件是否改动.

  \2. 检测rootkits使用的二进制和系统工具文件.

  \3. 检测特洛伊木马程序的特征码.

  \4. 检测大多常用程序的文件异常属性.

  \5. 执行一些系统相关的测试 - 因为rootkit hunter可支持多个系统平台.

  \6. 扫描任何混杂模式下的接口和后门程序常用的端口.

  \7. 检测如/etc/rc.d/目录下的所有配置文件, 日志文件, 任何异常的隐藏文件等等. 例如, 在检测/dev/.udev和/etc/.pwd.lock文件时候, 我的系统被警告.

  \8. 对一些使用常用端口的应用程序进行版本测试. 如: Apache Web Server, Procmail等.

 

如果您不想要每个部分都以 Enter 来继续，想要让程序自动持续执行，可以使用：

```
rkhunter --check --sk
```

 

输出如下：

```
[ Rootkit Hunter version 1.4.6 ]
...
/sbin/chkconfig                                          [ OK ]
/sbin/depmod                                             [ OK ]
...
/bin/chown                                               [ Warning ]
...
/bin/date                                                [ Warning ]

...
/usr/bin/top                                             [ Warning ]
...
/usr/bin/vmstat                                          [ Warning ]
...
System checks summary
=====================

File properties checks...
    Files checked: 134
    Suspect files: 4

Rootkit checks...
    Rootkits checked : 502
    Possible rootkits: 0

Applications checks...
    All checks skipped

The system checks took: 1 minute and 57 seconds

All results have been written to the log file: /var/log/rkhunter/rkhunter.log

One or more warnings have been found while checking the system.
Please check the log file (/var/log/rkhunter/rkhunter.log)
```

 

从以上信息可以看出，已经有4个系统命令，被串改了。

 

# 三、一些解决思路

如果您的系统经过 rkhunter 的检测之后，却发现很多的『红字』时，该怎么办？很简单， 可以参考这个网页提供的方法：

http://www.rootkit.nl/articles/rootkit_hunter_faq.html

 

  基本上，官方网站与一般网管老手的建议都一样，如果被 rootkit 之类的程序包攻击后 (  也就是上一节的检测表中的第二部分所攻击时 )，那么最好最好直接重新安装系统， 不要存在说可以移除 rootkit  或者木马程序的幻想，因为，『隐藏』本来就是 rootkit 与木马程序的拿手好戏！ 我们不知道到底这个 rootkit  或者木马程序有多剽悍，为了保险起见，还是重灌系统吧！如何重灌？简单的说：

  1.将原主机的网络线拔除；

  2.备份您的数据，最好备份成两部分，一部份是全部的系统内容，越详尽越好，包括 binary files 与 logfile 等等， 至于另一部份则可以考虑仅备份重要的数据文件即可！

  3.将上个步骤的数据备份(仅重要数据部分！)进行整体的检查，察看是否有怪异的数据存在(这部分可能会花去不少时间！)

  4.重新安装一部完整的系统，这包括：

  o仅安装需要的套件在服务器上面；

  o先进行 简单的防火墙 设定后才进行联机；

  o以 APT/YUM 之类的工具进行在线更新；

  o执行类似 rkhunter/nessus 之类的软件，检验系统是否处在较为安全的状态

  5.将原本的重要数据移动至上个步骤安装好的系统当中，并启动原本服务器上面的各项服务；

  6.以 rkhunter/nessus 之类的软件检验系统是否处在较为安全的环境，并且加强防火墙的机制！

  7.最后，将原本完整备份的数据拿出来进行分析，尤其是 logfile 部分，试图找出 cracker 是藉由那个服务？那个时间点？ 以那个远程 IP 联机进入本机等等的信息，并针对该信息研拟预防的方法，并应用在已经运作的机器上。

 

所以，只有重新安装系统，服务重新部署才是最妥当的办法。

 

本文参考链接：

https://www.cnblogs.com/cp-miao/p/6141025.html