**一、下载安装**

Github地址：https://github.com/adejoux/nmon2influxdb

入门文档：http://nmon2influxdb.org/

**1、RPM安装**

```
# 下载tar包
wget https://github.com/adejoux/nmon2influxdb/releases/download/v2.1.6/nmon2influxdb_2.1.6_linux_64-bit.tar.gz
# 解压tar包
tar -zxvf nmon2influxdb_2.1.6_linux_64-bit.tar.gz
# 查看帮助说明
./nmon2influxdb -h
```

**2、GZ包安装**

下载地址：[nmon2influxdb](https://github.com/adejoux/nmon2influxdb/releases)

去上述地址，下载对应操作系统的安装包，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201906/983980-20190628132102598-6404074.png)

利用FTP或者其他方式上传到服务器，然后输入命令 gunzip nmon2influxdb_2.1.6_linux_64-bit.tar.gz 解压，查看帮助说明，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201906/983980-20190628132708635-1603323506.png)

PS：上图标红的几点，需要修改对应的配置文件为实际的参数，谨记！

 

**二、配置部署**

**1、修改配置文件**

通过上文可知，配置文件nmon2influxdb.cfg的地址在家目录下，去对应目录修改配置文件，命令如下：

```
# 从当前目录到家目录
cd ~
#查找配置文件
ls -alrth
# 编辑配置文件
vi .nmon2influxdb.cfg
```

要修改的配置文件参数如下图所示：

![img](https://img2018.cnblogs.com/blog/983980/201906/983980-20190628134016368-530860222.png)

**2、导入数据验证**

**PS**：我用的是influxdb作为数据存储服务，因此执行这一步之前，需要安装influxdb，如何安装使用可参考这里：[时序数据库influxDB：简介及安装](https://www.cnblogs.com/imyalost/p/9689209.html)。

首先，输入nmon命令 ./nmon -ft -s 10 -c 20 ，生成一定的采样数据；（如何安装使用nmon，可参考这里：[服务端监控工具：Nmon使用方法](https://www.cnblogs.com/imyalost/p/9689213.html)）

然后，输入命令 ./nmon2influxdb import $server.nmon ，将采集的数据导入（命令中的$server为采样文件的名称）influxdb对应的库中（如配置文件所示，默认库为nmon_reports）；

进入服务端，输入命令，查看数据是否入库，相关命令如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
#进入数据库操作
influx
# 查看目前已有的数据库
show databases
# 查看数据库数据保存策略
show retention policies on nmon_reports
# 使用nmon_reports库
use nmon_reports
# 显示nmon_reports库所有的表
show measurements
# 查询数据
select * from CPU_ALL
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2018.cnblogs.com/blog/983980/201906/983980-20190628140747819-1393293252.png)

 

**三、监控数据可视化**

启动grafana，配置对应的Dashboard、Data Sources，然后选择配置好的仪表盘，查看可视化的监控数据（如何配置grafana，请看这里：[可视化工具Grafana：简介及安装](https://www.cnblogs.com/imyalost/p/9873641.html)）。

![img](https://img2018.cnblogs.com/blog/983980/201906/983980-20190628141512014-2117701834.png)