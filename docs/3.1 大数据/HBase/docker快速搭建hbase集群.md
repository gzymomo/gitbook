- [docker快速搭建hbase集群](https://www.cnblogs.com/xiao987334176/p/13230925.html)



# 一、概述

HBase是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang  所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File  System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase是Apache的Hadoop项目的子项目。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200703155540990-1325645516.png)

## Hadoop生太圈

通过Hadoop生态圈，可以看到HBase的身影，可见HBase在Hadoop的生态圈是扮演这一个重要的角色那就是 **实时、分布式、高维数据** 的数据存储；

## HBase简介

- HBase – Hadoop Database，是一个**高可靠性、高性能、面向列、可伸缩、 实时读写的分布式数据库** 
- 利用Hadoop HDFS作为其文件存储系统,利用Hadoop MapReduce来处理 HBase中的海量数据,利用Zookeeper作为其分布式协同服务
- 主要用来存储非结构化和半结构化的松散数据（列存NoSQL数据库）

## HBase数据模型

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200703155757178-1840791610.png)

 以关系型数据的思维下会感觉，上面的表格是一个5列4行的数据表格，但是在HBase中这种理解是错误的，其实在HBase中上面的表格只是一行数据；

**Row Key:**

　　　　– 决定一行数据的唯一标识

　　　　– RowKey是按照字典顺序排序的。

　　　　– Row key最多只能存储64k的字节数据。

　　**Column Family列族（CF1、CF2、CF3） & qualifier列：**

　　　　– HBase表中的每个列都归属于某个列族，列族必须作为表模式(schema) 定义的一部分预先给出。如create ‘test’, ‘course’；

　　　　– 列名以列族作为前缀，每个“列族”都可以有多个列成员(column，每个列族中可以存放几千~上千万个列)；如 CF1:q1, CF2:qw,

　　　　  新的列族成员（列）可以随后按需、动态加入，Family下面可以有多个Qualifier，所以可以简单的理解为，HBase中的列是二级列，

　　　　　也就是说Family是第一级列，Qualifier是第二级列。两个是父子关系。

　　　　– 权限控制、存储以及调优都是在列族层面进行的；

　　　　– HBase把同一列族里面的数据存储在同一目录下，由几个文件保存。

　　　　– 目前为止HBase的列族能能够很好处理最多不超过3个列族。

　　**Timestamp时间戳：**

　　　　– 在HBase每个cell存储单元对同一份数据有多个版本，根据唯一的时间 戳来区分每个版本之间的差异，不同版本的数据按照时间倒序排序，

　　　　　最新的数据版本排在最前面。

　　　　– 时间戳的类型是64位整型。

　　　　– 时间戳可以由HBase(在数据写入时自动)赋值，此时时间戳是精确到毫 秒的当前系统时间。

　　　　– 时间戳也可以由客户显式赋值，如果应用程序要避免数据版本冲突， 就必须自己生成具有唯一性的时间戳。

　　**Cell单元格：**

　　　　– 由行和列的坐标交叉决定；

　　　　– 单元格是有版本的（由时间戳来作为版本）；

　　　　– 单元格的内容是未解析的字节数组（Byte[]），cell中的数据是没有类型的，全部是字节码形式存贮。

　　　　　• 由{row key，column(=<family> +<qualifier>)，version}唯一确定的单元。

 

## HBase体系架构

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200703160012916-1627347010.png)

**Client**

　　　　　• 包含访问HBase的接口并维护cache来加快对HBase的访问

　　　　**Zookeeper**

　　　　　• 保证任何时候，集群中只有一个master

　　　　　• 存贮所有Region的寻址入口。

　　　　　• 实时监控Region server的上线和下线信息。并实时通知Master

　　　　　• 存储HBase的schema和table元数据

　　　　**Master**

　　　　　• 为Region server分配region

　　　　　• 负责Region server的负载均衡

　　　　　• 发现失效的Region server并重新分配其上的region

　　　　　• 管理用户对table的增删改操作

　　　　**RegionServer**

　　　　　• Region server维护region，处理对这些region的IO请求

　　　　　• Region server负责切分在运行过程中变得过大的region　

 

　　　  **HLog(WAL log)：**

　　　　　　– HLog文件就是一个普通的Hadoop Sequence File，Sequence File 的Key是 HLogKey对象，HLogKey中记录了写入数据的归属信息，

　　　　  　　除了table和 region名字外，同时还包括sequence number和timestamp，timestamp是” 写入时间”，sequence number的起始值为0，

　　　　　　　或者是最近一次存入文件系 统中sequence number。

　　　　　　– HLog SequeceFile的Value是HBase的KeyValue对象，即对应HFile中的 KeyValue

　　　　**Region**

　　　　　　– HBase自动把表水平划分成多个区域(region)，每个region会保存一个表 里面某段连续的数据；每个表一开始只有一个region，随着数据不断插 入表，

　　　　　　　region不断增大，当增大到一个阀值的时候，region就会等分会 两个新的region（裂变）；

　　　　　　– 当table中的行不断增多，就会有越来越多的region。这样一张完整的表 被保存在多个Regionserver上。

　　　　**Memstore 与 storefile**

　　　　　　– 一个region由多个store组成，一个store对应一个CF（列族）

　　　　　　– store包括位于内存中的memstore和位于磁盘的storefile写操作先写入 memstore，当memstore中的数据达到某个阈值，

　　　　　　　hregionserver会启动 flashcache进程写入storefile，每次写入形成单独的一个storefile

　　　　　　– 当storefile文件的数量增长到一定阈值后，系统会进行合并（minor、 major compaction），在合并过程中会进行版本合并和删除工作 （majar），

　　　　　　　形成更大的storefile。

　　　　　　– 当一个region所有storefile的大小和超过一定阈值后，会把当前的region 分割为两个，并由hmaster分配到相应的regionserver服务器，实现负载均衡。

　　　　　　– 客户端检索数据，先在memstore找，找不到再找storefile

　　　　　　– HRegion是HBase中分布式存储和负载均衡的最小单元。最小单元就表 示不同的HRegion可以分布在不同的HRegion server上。

　　　　　　– HRegion由一个或者多个Store组成，每个store保存一个columns family。

　　　　　　– 每个Strore又由一个memStore和0至多个StoreFile组成。

　　　　　　　如图：StoreFile 以HFile格式保存在HDFS上。

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200703160041693-1012546638.png)

 

# 二、docker部署

## 环境说明

| 操作系统   | docker版本 | ip地址         | 配置  |
| ---------- | ---------- | -------------- | ----- |
| centos 7.6 | 19.03.12   | 192.168.31.229 | 4核8g |



## 软件版本

| 软件      | 版本                 |
| --------- | -------------------- |
| openjdk   | java-8-openjdk-amd64 |
| hadoop    | 2.9.2                |
| hbase     | 1.3.6                |
| zookeeper | 3.4.14               |

 

说明：openjdk直接用apt-get 在线安装，其他软件从官网下载即可。

 

## 目录结构

```
cd /opt/
git clone https://github.com/py3study/hadoop-hbase.git
```

 

/opt/hadoop-hbase 目录结构如下：

```
./
├── config
│   ├── core-site.xml
│   ├── hadoop-env.sh
│   ├── hbase-env.sh
│   ├── hbase-site.xml
│   ├── hdfs-site.xml
│   ├── mapred-site.xml
│   ├── regionservers
│   ├── run-wordcount.sh
│   ├── slaves
│   ├── ssh_config
│   ├── start-hadoop.sh
│   ├── yarn-site.xml
│   └── zoo.cfg
├── Dockerfile
├── hadoop-2.9.2.tar.gz
├── hbase-1.3.6-bin.tar.gz
├── README.md
├── run.sh
├── sources.list
├── start-container1.sh
├── start-container2.sh
└── zookeeper-3.4.14.tar.gz
```

 

由于软件包比较大，需要使用迅雷下载，下载地址如下：

```
https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
https://mirror.bit.edu.cn/apache/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz
https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

 

构建镜像

```
docker build -t hadoop-hbase:1 .
```

 

创建数据目录

```
mkdir -p /data/hadoop-cluster/master/ /data/hadoop-cluster/slave{1,2}/
```

 

创建网桥

```
docker network create hadoop
```

 

运行镜像

```
cd /opt/hadoop-hbase
bash start-container1.sh
```

 

拷贝hdfs文件到宿主机目录

```
docker cp hadoop-master:/root/hdfs /data/hadoop-cluster/master/
docker cp hadoop-slave1:/root/hdfs /data/hadoop-cluster/slave1/
docker cp hadoop-slave2:/root/hdfs /data/hadoop-cluster/slave2/
```

 

拷贝zookeeper文件到宿主机目录

```
docker cp hadoop-master:/usr/local/zookeeper/data /data/hadoop-cluster/master/zookeeper
docker cp hadoop-slave1:/usr/local/zookeeper/data /data/hadoop-cluster/slave1/zookeeper
docker cp hadoop-slave2:/usr/local/zookeeper/data /data/hadoop-cluster/slave2/zookeeper
```

 

使用第2个脚本，挂载宿主机目录，运行镜像

```
bash start-container2.sh
```

 

## 开启hadoop

启动hadoop集群

```
bash start-hadoop.sh
```

注意：这一步会ssh连接到每一个节点，确保ssh信任是正常的。

Hadoop的启动速度取决于机器性能

 

## 运行wordcount

 先等待1分钟，再执行命令：

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

 

## 关闭安全模式

执行命令：

```
hadoop dfsadmin -safemode leave
```

 

## 启动hbase

```
/usr/local/hbase/bin/start-hbase.sh 
```

**注意：等待3分钟，因为启动要一定的时间。**

 

## 进入hbase shell

```
/usr/local/hbase/bin/hbase shell
```

 

# 三、HBase的Shell命令

## 查看列表

```
hbase(main):001:0> list
TABLE                                                                                                                                          
users                                                                                                                                          
users_tmp                                                                                                                                      
2 row(s) in 0.2370 seconds
```

 

如果出现

```
ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
```

说明hbase集群还没有启动好，需要等待一段时间。

 

## 创建表

```
hbase(main):002:0> create 'users','user_id','address','info'
0 row(s) in 4.6300 seconds

=> Hbase::Table - users
```

 

## 添加记录

```
hbase(main):002:0> put 'users','xiaoming','info:birthday','1987-06-17'
0 row(s) in 0.1910 seconds
```

 

## 获取记录

取得一个id的所有数据

```
hbase(main):003:0> get 'users','xiaoming'
COLUMN                               CELL                                                                                                      
 info:birthday                       timestamp=1594003730408, value=1987-06-17                                                                 
1 row(s) in 0.0710 seconds
```

 

## 更新记录

```
hbase(main):004:0> put 'users','xiaoming','info:age' ,''
0 row(s) in 0.0150 seconds

hbase(main):005:0> get 'users','xiaoming','info:age'
COLUMN                               CELL                                                                                                      
 info:age                            timestamp=1594003806409, value=                                                                           
1 row(s) in 0.0170 seconds
```

 

## 获取单元格数据的版本数据

```
hbase(main):006:0> get 'users','xiaoming',{COLUMN=>'info:age',VERSIONS=>1}
COLUMN                               CELL                                                                                                      
 info:age                            timestamp=1594003806409, value=                                                                           
1 row(s) in 0.0040 seconds
```

 

## 全表扫描

```
hbase(main):007:0> scan 'users'
ROW                                  COLUMN+CELL                                                                                               
 xiaoming                            column=info:age, timestamp=1594003806409, value=                                                          
 xiaoming                            column=info:birthday, timestamp=1594003730408, value=1987-06-17                                           
1 row(s) in 0.0340 seconds
```

 

## 删除

删除xiaoming值的'info:age'字段：

```
hbase(main):008:0> delete 'users','xiaoming','info:age'
0 row(s) in 0.0340 seconds

hbase(main):009:0> get 'users','xiaoming'
COLUMN                               CELL                                                                                                      
 info:birthday                       timestamp=1594003730408, value=1987-06-17                                                                 
1 row(s) in 0.0110 seconds
```

 

删除整行

```
hbase(main):010:0> deleteall 'users','xiaoming'
0 row(s) in 0.0170 seconds
```

 

统计表的行数

```
hbase(main):011:0> count 'users'
0 row(s) in 0.0260 seconds
```

 

清空表：

```
hbase(main):012:0> truncate 'users'
Truncating 'users' table (it may take a while):
 - Disabling table...
 - Truncating table...
0 row(s) in 4.2520 seconds
```

 

# 四、web服务验证

## hadoop管理页面

```
http://ip地址:8088/
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200706110158752-1921228879.png)

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200706110250149-1437760979.png)

## hdfs 管理页面

```
http://ip地址:50070/
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200706110339781-894632933.png)

## hbase 管理页面

```
http://ip地址:16010/
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200706110421332-323807234.png)

 

 注意：这里出现的 2 nodes with inconsistent version，不用理会，不影响正常运行。

参考链接：https://developer.aliyun.com/ask/136178?spm=a2c6h.13159736

 

本文参考链接：

https://www.cnblogs.com/raphael5200/p/5229164.html

https://blog.csdn.net/qq_32440951/article/details/80803729

https://www.jianshu.com/p/a1524dccb1e4

https://www.bbsmax.com/A/6pdDLqOGdw/