# Netdata

## Netdata介绍

Netdata是一款Linux系统性能实时监控工具。是一个高度优化的Linux守护进程，可以对Linux系统、应用程序(包括但不限于Web服务器，数据库等)、SNMP服务等提供实时的性能监控。

Netdata用可视化的手段，将其被监控的信息展现出来，以便你清楚的了解到你的系统、程序、应用的实时运行状态，而且还可以与Prometheus，Graphite，OpenTSDB，Kafka，Grafana等相集成。

Netdata是免费的开源软件，目前可在Linux，FreeBSD和macOS以及从它们衍生的其他系统（例如Kubernetes和Docker）上运行。

Netdata仓库地址：[https://github.com/netdata/ne...](https://github.com/netdata/netdata)

## Netdata特性

- 1、友好、美观的可视化界面
- 2、可自定义的控制界面
- 3、安装快速且高效
- 4、配置简单，甚至可零配置
- 5、零依赖
- 6、可扩展，自带插件API
- 7、支持的系统平台广

## Netdata是如何工作的？

Netdata是一个高效，高度模块化的指标管理引擎。它的无锁设计使其非常适合度量标准上的并发操作。

![img](https://segmentfault.com/img/remote/1460000024541645)

上图的各个组件的作用描述，有兴趣的可以参考官方的说明，这里不再赘述了。

## Netdata可监控什么？

Netdata可以收集来自200多种流行服务和应用程序的指标，以及数十种与系统相关的指标，例如CPU，内存，磁盘，文件系统，网络等。我们将这些收集器称为，它们由插件管理，该插件支持多种编程语言，包括Go和Python。

流行的收集器包括Nginx，Apache，MySQL，statsd，cgroups（容器，Docker，Kubernetes，LXC等），Traefik，Web服务器access.log文件等。

详细的支持列表请参考下面的说明：[https://github.com/netdata/ne...](https://github.com/netdata/netdata/blob/master/collectors/COLLECTORS.md)

## Netdata安装

1、直接安装

首先需要更新升级系统内核和一些依赖库文件

```
[root@CentOS7-1 ~]# yum update -y
```

如果没有操作，你在直接安装时会出现下面的提示，输入y让系统自动更新也是可以的。

![img](https://segmentfault.com/img/remote/1460000024541646)

执行完更新操作后，直接执行下面的命令进行安装Netdata。

```
[root@CentOS7-1 ~]# bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

然后，程序会自动执行安装动作，去下载一系列的包进行安装，中间需要确认操作一次，如下：

![img](https://segmentfault.com/img/remote/1460000024541650)

可能会由于访问国外的资源，和根据你的网络关系，等待的时间或长或短。

![img](https://segmentfault.com/img/remote/1460000024541647)

一些关键的信息，从安装过程中也是可以看的出来的，如上图。

![img](https://segmentfault.com/img/remote/1460000024541649)

从上图信息可以看出访问方法，启动、停止服务的命令。

安装完成如下图

![img](https://segmentfault.com/img/remote/1460000024541648)

显示Netdata已经启动完成，我们可以使用命令来查看一下是否启动完成？

```
[root@CentOS7-1 ~]# lsof -i :19999
COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
netdata 14787 netdata    4u  IPv4  27995      0t0  TCP *:dnp-sec (LISTEN)
netdata 14787 netdata    5u  IPv6  27996      0t0  TCP *:dnp-sec (LISTEN)
[root@CentOS7-1 ~]# ps -ef|grep netdata
netdata   14787      1  2 23:24 ?        00:00:06 /usr/sbin/netdata -P /var/run/netdata/netdata.pid -D
netdata   14800  14787  0 23:24 ?        00:00:00 /usr/sbin/netdata --special-spawn-server
netdata   14954  14787  0 23:24 ?        00:00:01 bash /usr/libexec/netdata/plugins.d/tc-qos-helper.sh 1
netdata   14974  14787  0 23:24 ?        00:00:02 /usr/bin/python /usr/libexec/netdata/plugins.d/python.d.plugin 1
root      14975  14787  1 23:24 ?        00:00:04 /usr/libexec/netdata/plugins.d/ebpf.plugin 1
netdata   14976  14787  0 23:24 ?        00:00:01 /usr/libexec/netdata/plugins.d/go.d.plugin 1
netdata   14977  14787  1 23:24 ?        00:00:05 /usr/libexec/netdata/plugins.d/apps.plugin 1
root      15277   1149  0 23:29 pts/0    00:00:00 grep --color=auto netdata
```

2、Docker方式安装

首先准备Docker环境，然后直接执行下面的命令即可完成安装操作。

```
docker run -d --name=netdata 
  -p 19999:19999 
  -v netdatalib:/var/lib/netdata 
  -v netdatacache:/var/cache/netdata 
  -v /etc/passwd:/host/etc/passwd:ro 
  -v /etc/group:/host/etc/group:ro 
  -v /proc:/host/proc:ro 
  -v /sys:/host/sys:ro 
  -v /etc/os-release:/host/etc/os-release:ro 
  --restart unless-stopped 
  --cap-add SYS_PTRACE 
  --security-opt apparmor=unconfined 
  netdata/netdata
```

安装完成后，就可以通过下面的方式进行访问了。

![img](https://segmentfault.com/img/remote/1460000024541651)

![img](https://segmentfault.com/img/remote/1460000024541652)

## 界面展示

1、总体数据界面

![img](https://segmentfault.com/img/remote/1460000024541654)

2、内存

![img](https://segmentfault.com/img/remote/1460000024541653)

3、CPU

![img](https://segmentfault.com/img/remote/1460000024541656)

4、磁盘

![img](https://segmentfault.com/img/remote/1460000024541655)

5、网络

![img](https://segmentfault.com/img/remote/1460000024541657)

6、应用

![img](https://segmentfault.com/img/remote/1460000024541658)

7、网络接口

![img](https://segmentfault.com/img/remote/1460000024541660)

8、数据同步功能

Netdata仪表板上的图表彼此同步，没有主图表。可以随时平移或缩放任何图表，其他所有图表也将随之出现。

![img](https://segmentfault.com/img/remote/1460000024541659)

通过使用鼠标拖动可以平移图表。当鼠标指针悬停在图表上时，可以使用SHIFT+ 放大/缩小mouse wheel图表。

## Netdata强大之处

之所以如此强大，是因为它与各类应用的配合与支持，直接上图说明：

![img](https://segmentfault.com/img/remote/1460000024541661)

## Netdata集群管理方案

上面展示的只是单一服务器的监控数据，而且netdata有一个缺点就是所有被监控的服务器都需要安装agent，所以，这里就是出现一个问题，就是如何将监控数据统一管理与展示？

netdata官方并没设计主从模式，像zabbix那样，可以一台做为主服务器，其它的做为从服务器，将数据收集到主服务器统一处理与展示，但是，官方也给出了相关的解决方案。

- 1、netdata.cloud

使用自带的 netdata.cloud，也就是每一个安装节点WEB界面右上角的signin。只要我们使用同一个账号登录netdata.cloud（需要kexue上网），之后各个节点之间就可以轻松通过一个账号控制。每个节点开启19999端口与允许管理员查看数据，然后控制中心通过前端从各节点的端口收集的数据，传给netdata.cloud记录并展示。

这是一种被动的集群监控，本质上还是独立的机器，且不方便做自定义的集群dashboard。

- 2、stream 插件

所以，为了解决上面这种方案的弊端，netdata又提供了另一种方法，将各节点的数据集中汇总到一台（主）服务器，数据处理也在这台服务器上，其它节点无需开放19999端口。 算是一种主动传输模式，把收集到的数据发送到主服务器上，这样在主服务器上可以进行自定义的dashboard开发。

缺点：主服务器流量、负载都会比较大（在集群服务器数量较多的情况下），如果主服务器负载过高，我们可以通过设置节点服务器的数据收集周期（update every）来解决这个问题。

## Netdata集群监控配置

**很多文章都只是介绍了其安装与一些界面的展示结果，并没有提供集群监控这一解决方案与其具体的配置，民工哥也是查了很多的资料，现在将其配置过程分享给大家。**

对于streaming的配置不熟悉的可以参考官方的文档说明：[https://docs.netdata.cloud/st...](https://docs.netdata.cloud/streaming/)

1、节点服务器配置

```
[root@CentOS7-1 ~]# cd /etc/netdata/
[root@CentOS7-1 netdata]# vim netdata.conf
#修改配置如下
[global]
    memory mode = none
    hostname = [建议修改成你的主机名]
[web]
    mode = none
```

然后，在/etc/netdata/目录下新建一个文件stream.conf，然后将其配置为如下：

```
[stream]
    enabled = yes
    destination = MASTER_SERVER_IP:PORT
    api key = xxxx-xxxx-xxxx-xxxx-xxxx
    
#参数说明如下
 destination = MASTER_SERVER_IP:PORT  主服务器地址与端口
 api key 必需为uuid的字符串，Linux系统中可以使用下面的命令自动生成。
 [root@CentOS7-1 netdata]# uuidgen
 480fdc8c-d1ac-4d6f-aa26-128eba744089
```

配置完成之后，需要重启节点的netdata服务即可完成整个配置。

```
[root@CentOS7-1 ~]# systemctl restart netdata
```

2、主服务器配置

在netdata.conf的同一目录下新建stream.conf并写入如下配置：

```
[API_KEY]/[480fdc8c-d1ac-4d6f-aa26-128eba744089]
    enabled = yes
    default history = 3600
    default memory mode = save
    health enabled by default = auto
    allow from = *
[API_KEY]
    enabled = yes
    default history = 3600
    default memory mode = save
    health enabled by default = auto
    allow from = *
#其中，API_KEY对应节点服务器的api key(字符串)，allow from可以设置数据流的允许来源以保证安全。
#如果有多个节点服务器，则一起写在stream.conf里面
```

完成配置后重启netdata：

```
systemctl restart netdata
```

所有的配置完成后，就可以在主服务器的WEB界面右上角看到下拉菜单（主机名），点击即可看到相关的监控信息了。

如果需要自定义控制面板，可以参考官方的文档，去修改xml文件。 原文地址：[https://docs.netdata.cloud/we...](https://docs.netdata.cloud/web/gui/custom/)