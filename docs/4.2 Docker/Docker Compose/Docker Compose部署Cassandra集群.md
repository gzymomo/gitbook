- [docker创建Cassandra集群](https://www.cnblogs.com/xiao987334176/p/13219163.html)



# 一、概述

## 简介

- Cassandra是一个开源分布式NoSQL数据库系统。
- 它最初由Facebook开发，用于储存收件箱等简单格式数据，集GoogleBigTable的数据模型与Amazon  Dynamo的完全分布式的架构于一身。Facebook于2008将 Cassandra  开源，此后，由于Cassandra良好的可扩展性，被Digg、Twitter等知名Web  2.0网站所采纳，成为了一种流行的分布式结构化数据存储方案。
- 不过国内并未流行起来，除了最早的淘宝和360在用，加上阿里巴巴后来一直在推崇HBase，就GG了。。。

 ![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200701145140240-843046930.png)

 

## 特点

- Cassandra的主要特点就是它不是一个数据库，而是由一堆数据库节点共同构成的一个分布式网络服务，对Cassandra  的一个写操作，会被复制到其他节点上去，对Cassandra的读操作，也会被路由到某个节点上面去读取。对于一个Cassandra集群来说，扩展性能是比较简单的事情，只管在群集里面添加节点就可以了。
- 上面的话太官方了，哈哈哈。，简单来说呢就是说它是一个P2P去中心化的东西，咱门平时传统用的数据库都会有Master/Slave，在复杂的场景下对于Master进行扩展是个非常麻烦的事，而Cassandra帮助我们解决了这个麻烦。
- 它是一个面向列的数据库，不向传统结构式数据库是用表来模拟关系，也就是说你可以随意扩展你的字段。你可以想象cassandra是一个连续嵌套的Map结构。如下图所示

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200701145239473-1709084174.png)

 

# 二、docker搭建

## 环境说明

| 操作系统   | docker版本 | ip地址         | 配置  |
| ---------- | ---------- | -------------- | ----- |
| centos 7.6 | 19.03.12   | 192.168.31.229 | 4核8g |

 

## 下载镜像

官方地址：https://hub.docker.com/_/cassandra

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200701145454057-1448128223.png)

 

 目前最新版本是3.11.6

 

下载镜像

```
docker pull cassandra
```

 

## Docker Compose部署

Cassandra采用去中心化的集群架构，没有master节点的概念；但是会有seed节点在新节点连入时通知当前集群。

下面的Docker Compose模板将为你创建一个包含3个节点的Cassandra集群，其中第一个容器“cassandra-1”为seed节点。

 

新建目录/opt/cassandra，结构如下：

```
./
├── cassandra.yaml
└── docker-compose.yaml
```

### cassandra.yaml 

cassandra.yaml 是从cassandra容器里面拷贝出来的。

先运行一个单节点的cassandra

```
docker run -it cassandra /bin/bash
```

再开一个新的窗口，拷贝配置文件。

```
cd /opt/cassandra
docker cp tmp-cassandra:/opt/cassandra/conf/cassandra.yaml .
```

 

修改cassandra.yaml，将

```
authenticator: AllowAllAuthenticator
```

修改为：

```
authenticator: PasswordAuthenticator
```

这样做的目的是为了，可以使用用户名和密码，进行远程连接。默认的策略，好像只能本地连接。

 

### docker-compose.yaml

```yaml
version: '3'
services:
  cassandra-1:
    image: cassandra
    container_name: cassandra-1
    volumes:
      - ./cassandra.yaml:/opt/cassandra/conf/cassandra.yaml
      - /data/cassandra-cluster/cassandra-1/cassandra:/var/lib/cassandra
    environment:
      - CASSANDRA_BROADCAST_ADDRESS=cassandra-1
    ports:
      - "7000:7000"
      - "9042:9042"
    restart: always
  cassandra-2:
    image: cassandra
    container_name: cassandra-2
    volumes:
      - ./cassandra.yaml:/opt/cassandra/conf/cassandra.yaml
      - /data/cassandra-cluster/cassandra-2/cassandra:/var/lib/cassandra
    environment:
      - CASSANDRA_BROADCAST_ADDRESS=cassandra-2
      - CASSANDRA_SEEDS=cassandra-1
    ports:
      - "7001:7000"
      - "9043:9042"
    depends_on:
      - cassandra-1
    restart: always
  cassandra-3:
    image: cassandra
    container_name: cassandra-3
    volumes:
      - ./cassandra.yaml:/opt/cassandra/conf/cassandra.yaml
      - /data/cassandra-cluster/cassandra-3/cassandra:/var/lib/cassandra
    environment:
      - CASSANDRA_BROADCAST_ADDRESS=cassandra-3
      - CASSANDRA_SEEDS=cassandra-1
    ports:
      - "7002:7000"
      - "9044:9042"
    depends_on:
      - cassandra-2
    restart: always
```

说明：

cassandra.yaml 挂载到容器中，开启用户远程登录。

/data/cassandra-cluster 是本地目录，用来做持久化的。

CASSANDRA_BROADCAST_ADDRESS 此变量用于控制向其他节点播发哪个IP地址。

CASSANDRA_SEEDS 这个变量是用逗号分隔的IP地址列表，gossip 用来引导加入集群的新节点。

 

### cassandra 常用端口

- 7199 - JMX（8080 pre Cassandra 0.8.xx）
- 7000 - 节点间通信（如果启用了TLS，则不使用）
- 7001 - TLS节点间通信（使用TLS时使用）
- 9160 - Thrift客户端API
- 9042 - CQL本地传输端口

 

在上面的docker-compose.yaml中，映射了2个端口。我一般只使用9042端口，用来做远程连接！

 

## 调整系统参数

```
vi /etc/sysctl.conf
```

修改参数

```
vm.max_map_count=1048575
```

刷新参数

```
sysctl -p
```

 

如果不做这一步，启动Cassandra集群时，会有警告信息

```
WARN  [main] 2020-07-01 05:59:39,699 StartupChecks.java:311 - Maximum number of memory map areas per process (vm.max_map_count) 65530 is too low, recommended value: 1048575, you can change it with sysctl.
```

 

## 启动docker-compose

创建持久化目录

```
mkdir -p /data/cassandra-cluster/cassandra-{1,2,3}
```

 

现在，我们可以轻松利用 docker-compose 命令来启动Cassandra集群了

```
docker-compose up -d
```

启动之后，需要等待1分钟左右。

 

这个时候，如果使用docker logs命令查看日志，会发现它会有一些报错，请不必理会！

因为我把数据目录映射了出来，默认是空的。所以第一次启动时，会报错。不过没有关系，docker会自动重新启动几次。

在第3次时，就会启动成功了。

 

## 查看日志

```
docker logs -f cassandra-1
```

输出：

```
...
INFO  [main] 2020-07-01 06:11:12,670 Server.java:159 - Starting listening for CQL clients on /0.0.0.0:9042 (unencrypted)...
INFO  [main] 2020-07-01 06:11:12,791 CassandraDaemon.java:556 - Not starting RPC server as requested. Use JMX (StorageService->startRPCServer()) or nodetool (enablethrift) to start it
INFO  [OptionalTasks:1] 2020-07-01 06:11:13,588 CassandraRoleManager.java:372 - Created default superuser role 'cassandra'
```

看到了9042，说明集群已经工作正常了！

 

## 查看集群状态

查看容器运行状态

```
# docker-compose ps
   Name                  Command               State                            Ports                          
---------------------------------------------------------------------------------------------------------------
cassandra-1   docker-entrypoint.sh cassa ...   Up      0.0.0.0:7000->7000/tcp, 7001/tcp, 7199/tcp,             
                                                       0.0.0.0:9042->9042/tcp, 9160/tcp                        
cassandra-2   docker-entrypoint.sh cassa ...   Up      0.0.0.0:7001->7000/tcp, 7001/tcp, 7199/tcp,             
                                                       0.0.0.0:9043->9042/tcp, 9160/tcp                        
cassandra-3   docker-entrypoint.sh cassa ...   Up      0.0.0.0:7002->7000/tcp, 7001/tcp, 7199/tcp,             
                                                       0.0.0.0:9044->9042/tcp, 9160/tcp     
```

 

使用Cassandra自带命令，查看Cassandra集群状态

```
# docker exec -ti cassandra-1 cqlsh -u cassandra -pcassandra cassandra-2 -e "DESCRIBE CLUSTER"

Cluster: Test Cluster
Partitioner: Murmur3Partitioner
```

由于开启了密码认证。连接cassandra时，需要用户名和密码。这里的-u参数指定用户名，-p指定密码。

由此可知，默认的用户名和密码都是cassandra

 

# 三、Cassandra Cqlsh

 这里大概介绍Cassandra查询语言shell，并解释如何使用其命令。

默认情况下，Cassandra提供一个提示Cassandra查询语言shell（cqlsh），允许用户与它通信。使用此shell，您可以执行Cassandra查询语言（CQL）。

使用cqlsh，你可以

- 定义模式，
- 插入数据，
- 执行查询。

 

## 启动cqlsh

```
# docker exec -it cassandra-1 /bin/bash
root@4881bf50f2d5:/# cqlsh -u cassandra -pcassandra
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.6 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cassandra@cqlsh> 
```

 

## 查询keyspaces

列出来的相当于关系型数据的的系统数据库名

```
cassandra@cqlsh> describe keyspaces;

system_traces  system_schema  system_auth  system  system_distributed
```

 

## 创建数据库

```
cassandra@cqlsh> CREATE KEYSPACE IF NOT EXISTS mycasdb WITH REPLICATION = {'class': 'SimpleStrategy','replication_factor':3};
cassandra@cqlsh> describe keyspaces;

system_schema  system_auth  mycasdb  system  system_distributed  system_traces
```

解释：

Replication Factor : 复制因数。 表示一份数据在一个DC 之中包含几份。常用奇数~ 比如我们项目组设置的replication_factor=3

Replica placement strategy : 复制策略。 默认的是SimpleStrategy. 如果是单机架、单数据中心的模式，保持使用SimpleStrtegy即可。

 

## 创建表

在mycasdb数据库中创建一个表，首先使用use mycasdb;表示要使用此数据库，然后在使用：

```
cassandra@cqlsh> use mycasdb;
cassandra@cqlsh:mycasdb> CREATE TABLE user (id int,user_name varchar,PRIMARY KEY (id));
```

 

## 查看表

查看数据库中的表

```
cassandra@cqlsh:mycasdb> describe tables;

user
```

 

## 插入表数据

向user表中插入输入，使用：

```
cassandra@cqlsh:mycasdb> INSERT INTO user (id,user_name) VALUES (1,'sxj');
cassandra@cqlsh:mycasdb> INSERT INTO user (id,user_name) VALUES (2,'sxj12');
cassandra@cqlsh:mycasdb> INSERT INTO user (id,user_name) VALUES (3,'sxj123');
cassandra@cqlsh:mycasdb> INSERT INTO user (id,user_name) VALUES (4,'sxj1234');
```

 

## 查询表数据

```mysql
cassandra@cqlsh:mycasdb> select * from user;

 id | user_name
----+-----------
  1 |       sxj
  2 |     sxj12
  4 |   sxj1234
  3 |    sxj123

(4 rows)
```

 

## 删除语句

```
cassandra@cqlsh:mycasdb> delete from user where id=2;
cassandra@cqlsh:mycasdb> select * from user;

 id | user_name
----+-----------
  1 |       sxj
  4 |   sxj1234
  3 |    sxj123

(3 rows)
```

 

在删除语句时必须加条件，否则会出现：SyntaxException:line 1:16 mismatched input ';' expecting K_WHERE，如下所示：

```
cassandra@cqlsh:mycasdb> delete from user;
SyntaxException: line 1:16 mismatched input ';' expecting K_WHERE
```

 

本文参考链接：

https://yq.aliyun.com/articles/61950

https://blog.csdn.net/toefllitong/article/details/79041155

https://blog.csdn.net/lixinkuan328/article/details/92238083

https://www.w3cschool.cn/cassandra/cassandra_cqlsh.html