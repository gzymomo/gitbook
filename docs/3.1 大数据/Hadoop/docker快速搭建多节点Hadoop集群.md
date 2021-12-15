- [docker快速搭建多节点Hadoop集群](https://www.cnblogs.com/xiao987334176/p/13208915.html)



# 一、概述

## hadoop是什么

Hadoop被公认是一套行业大数据标准开源软件，在分布式环境下提供了海量数据的处理能力。几乎所有主流厂商都围绕Hadoop开发工具、开源软件、商业化工具和技术服务。今年大型IT公司，如EMC、Microsoft、Intel、Teradata、Cisco都明显增加了Hadoop方面的投入。
![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200629164713609-510937157.jpg)

## hadoop能干什么

hadoop擅长日志分析，facebook就用Hive来进行日志分析，2009年时facebook就有非编程人员的30%的人使用HiveQL进行数据分析；淘宝搜索中的自定义筛选也使用的Hive；利用Pig还可以做高级的数据处理，包括Twitter、LinkedIn  上用于发现您可能认识的人，可以实现类似Amazon.com的协同过滤的推荐效果。淘宝的商品推荐也是！在Yahoo！的40%的Hadoop作业是用pig运行的，包括垃圾邮件的识别和过滤，还有用户特征建模。（2012年8月25新更新，天猫的推荐系统是hive，少量尝试mahout！）

 

## hadoop的核心

1.HDFS: Hadoop Distributed File System 分布式文件系统

2.YARN: Yet Another Resource Negotiator  资源管理调度系统

3.Mapreduce：分布式运算框架

 

## HDFS的架构

主从结构

​    •主节点， namenode

​    •从节点，有很多个: datanode

namenode负责：

​     •接收用户操作请求

​     •维护文件系统的目录结构

​     •管理文件与block之间关系，block与datanode之间关系

datanode负责：

​     •存储文件

​     •文件被分成block存储在磁盘上

​     •为保证数据安全，文件会有多个副本

 

Secondary NameNode负责：

​      合并fsimage和edits文件来更新NameNode的metedata



# 二、docker部署

## 环境说明

| 操作系统   | docker版本 | ip地址         | 配置  |
| ---------- | ---------- | -------------- | ----- |
| centos 7.6 | 19.03.12   | 192.168.31.229 | 4核8g |

 

## 拉取镜像

这里采用dockerhub现有，镜像大小为：777MB

```
docker pull kiwenlau/hadoop-master:0.1.0
```

 

## 运行容器

下载源代码

```
cd /opt/
git clone https://github.com/kiwenlau/hadoop-cluster-docker
```

 

创建网桥

```
docker network create hadoop
```

 

运行容器

```
cd /opt/hadoop-cluster-docker/
./start-container.sh
```

运行结果：

```
start master container...
start slave1 container...
start slave2 container...
```

 

一共开启了3个容器，1个master, 2个slave。开启容器后就进入了master容器root用户的根目录（/root）。

 

查看master的root用户家目录的文件：

```
root@hadoop-master:~# ls
hdfs  input  run-wordcount.sh  start-hadoop.sh
```

start-hadoop.sh是开启hadoop的shell脚本，

run-wordcount.sh是运行wordcount的shell脚本，可以测试镜像是否正常工作。

 

## 开启hadoop

```
bash start-hadoop.sh
```

 注意：这一步会ssh连接到每一个节点，确保ssh信任是正常的。

Hadoop的启动速度取决于机器性能

 

## 运行wordcount

```
bash run-wordcount.sh
```

此脚本会连接到fdfs，并生成几个测试文件。

 

运行结果：

```
...
input file1.txt:
Hello Hadoop

input file2.txt:
Hello Docker

wordcount output:
Docker  1
Hadoop  1
Hello   2
```

wordcount的执行速度取决于机器性能

 

# 三、配置文件说明

进入hadoop-master容器，hadoop的配置文件目录为：/usr/local/hadoop/etc/hadoop

## core-site.xml

```
<?xml version="1.0"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-master:9000/</value>
    </property>
</configuration>
```

 

## hdfs-site.xml

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop-master:9000/</value>
    </property>
</configuration>
root@hadoop-master:/usr/local/hadoop/etc/hadoop# pwd
/usr/local/hadoop/etc/hadoop
root@hadoop-master:/usr/local/hadoop/etc/hadoop# cat hdfs-site.xml 
<?xml version="1.0"?>
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///root/hdfs/namenode</value>
        <description>NameNode directory for namespace and transaction logs storage.</description>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///root/hdfs/datanode</value>
        <description>DataNode directory</description>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
```

注意：这里是配置一个Master节点和两个Slave节点。所以dfs.replication配置为2。
dfs.namenode.name.dir和dfs.datanode.data.dir分别配置为NameNode和DataNode的目录路径

 

## mapred-site.xml

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

指定运行mapreduce的环境是yarn

 

## hadoop-env.sh

注意：这里必须要指定java的路径。否则启动Hadoop时，提示找不到变量JAVA_HOME

```
# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

 

# 四、测试Hadoop

## hadoop管理页面

```
http://ip地址:8088/cluster/nodes
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630142624437-1920141167.png)

 

 

## hdfs 管理页面

```
http://ip地址:50070/
```

点击datanode，效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630142848590-1775788520.png)

 

 

浏览文件系统

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630142927852-1128955533.png)

 

默认有2个文件夹，这里面的文件是看不到的。

 ![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630142956790-235605707.png)

 

 

由于默认开启了安全默认，默认是没有权限查看文件的。需要关闭安全模式才行！

### 关闭安全模式

进入hadoop-master容器，执行命令：

```
hadoop dfsadmin -safemode leave
```

 

授权tmp文件权限

```
hdfs dfs -chmod -R 755 /tmp
```

 

刷新页面，点击tmp

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630143258036-2066322164.png)

 

 

返回上一级目录，进入/user/root/input，就可以看到脚本创建的2个文件了！

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200630143341599-1579938698.png)

 

 

**注意：hdfs存放目录为：/root/hdfs。如果需要做持久化，将此目录映射出来即可！**

 

本文参考链接：

http://dockone.io/article/395

https://blog.csdn.net/sb985/article/details/82722451

https://blog.csdn.net/gwd1154978352/article/details/81095592