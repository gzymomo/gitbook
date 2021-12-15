- [Docker部署ElasticSearch集群](https://www.cnblogs.com/npeng/p/14293416.html)             
- [Docker Elasticsearch 集群配置](https://www.cnblogs.com/yxy-ngu/p/14445776.html)



### 1.前言

为什么要用ElasticSearch？我们的应用经常需要添加检索功能，开源的Elastic  Search是目前全文检索引擎的首选。它可以快速的存储、搜索和分析海量数据。ElasticSearch是一个分布式搜索框架，提供RestfulAPI，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能。
 Elasticsearch: 权威指南(中文)：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

### 2.拉取ElasticSearch镜像

在centos窗口中，执行如下命令：

```shell
docker pull elasticsearch:5.6.8
```

当前ES镜像版本信息：

```json
 {
  "name" : "WlwFyqU",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "78UDZtviQqiWmzmenGpSrQ",
  "version" : {
    "number" : "5.6.8",
    "build_hash" : "cfe3d9f",
    "build_date" : "2018-09-10T20:12:43.732Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
 }
```

### 3.创建数据挂在目录，以及配置ElasticSearch集群配置文件

#### 3.1.创建数据文件挂载目录,并开放通信端口

在centos窗口中，执行如下操作：

```shell
[root@localhost soft]# pwd
/root/soft
[root@localhost soft]# mkdir -p ES/config
[root@localhost soft]# cd  ES 
[root@localhost ES]# mkdir data1
[root@localhost ES]# mkdir data2
[root@localhost ES]# mkdir data3
[root@localhost ES]# cd ES/config/
[root@localhost ES]# firewall-cmd --add-port=9300/tcp
success
[root@localhost ES]# firewall-cmd --add-port=9301/tcp
success
[root@localhost ES]# firewall-cmd --add-port=9302/tcp
success
```

> 注：如果ELK选的6.X版本的,且非管理员，那么读者需将data1 data2 data3 开启777权限=> chmod 777 data1 data2 data3

#### 3.2.创建ElasticSearch配置文件

在centos窗口中，使用vim命令分别创建如下文件：es1.yml,es2.yml,es3.yml

**es1.yml**

```shell
cluster.name: my-elasticsearch
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.70.129
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.70.129:9300","192.168.70.129:9301","192.168.70.129:9302"]
discovery.zen.minimum_master_nodes: 2
```

**es2.yml**

```shell
cluster.name: my-elasticsearch
node.name: es-node2
network.bind_host: 0.0.0.0
network.publish_host: 192.168.70.129
http.port: 9201
transport.tcp.port: 9301
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.70.129:9300","192.168.70.129:9301","192.168.70.129:9302"]
discovery.zen.minimum_master_nodes: 2
```

**es3.yml**

```shell
cluster.name: my-elasticsearch
node.name: es-node3
network.bind_host: 0.0.0.0
network.publish_host: 192.168.70.129
http.port: 9202
transport.tcp.port: 9302
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.70.129:9300","192.168.70.129:9301","192.168.70.129:9302"]
discovery.zen.minimum_master_nodes: 2
```

> 注：本机虚拟机ip：192.168.70.129 读者请自行更改

#### 3.3.调高JVM线程数限制数量

在centos窗口中，修改配置sysctl.conf

```shell
vim /etc/sysctl.conf
```

加入如下内容：

```shell
vm.max_map_count=262144 
```

启用配置：

```shell
sysctl -p
```

> 注：这一步是为了防止启动容器时，报出如下错误：
>  bootstrap checks failed max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]

### 4.启动ElasticSearch集群容器

启动ElasticSearch集群容器
 在centos窗口中，执行如下命令：

```shell
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 -v /root/soft/ES/config/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /root/soft/ES/data1:/usr/share/elasticsearch/data --name ES01 elasticsearch:5.6.8

docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9201:9201 -p 9301:9301 -v /root/soft/ES/config/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /root/soft/ES/data2:/usr/share/elasticsearch/data --name ES02 elasticsearch:5.6.8

docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9202:9202 -p 9302:9302 -v /root/soft/ES/config/es3.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /root/soft/ES/data3:/usr/share/elasticsearch/data --name ES03 elasticsearch:5.6.8
```

> 注：设置-e ES_JAVA_OPTS="-Xms256m -Xmx256m" 是因为/etc/elasticsearch/jvm.options 默认jvm最大最小内存是2G，读者启动容器后 可用docker stats命令查看

### 5.验证是否搭建成功

在浏览器地址栏访问http://192.168.70.129:9200/_cat/nodes?pretty 查看节点状态
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200412150509697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JydWNlTGl1X2NvZGU=,size_16,color_FFFFFF,t_70)

> 注：节点名称带表示为主节点*

### 6.ElasticSearch配置文件说明

```shell
cluster.name: my-elasticsearch
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.70.129
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true 
node.data: true  
discovery.zen.ping.unicast.hosts: ["192.168.70.129:9300","192.168.70.129:9301","192.168.70.129:9302"]
discovery.zen.minimum_master_nodes: 2
```

参数说明:

```shell

cluster.name：用于唯一标识一个集群，不同的集群，其 cluster.name 不同，集群名字相同的所有节点自动组成一个集群。如果不配置改属性，默认值是：elasticsearch。
node.name：节点名，默认随机指定一个name列表中名字。集群中node名字不能重复
index.number_of_shards: 默认的配置是把索引分为5个分片
index.number_of_replicas:设置每个index的默认的冗余备份的分片数，默认是1
network.bind_host: 设置可以访问的ip,可以是ipv4或ipv6的，默认为0.0.0.0，这里全部设置通过
network.publish_host:设置其它结点和该结点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址
http.port:设置对外服务的http端口，默认为9200
transport.tcp.port: 设置节点之间交互的tcp端口，默认是9300
http.cors.enabled: 是否允许跨域REST请求
http.cors.allow-origin: 允许 REST 请求来自何处
node.master: true 配置该结点有资格被选举为主结点（候选主结点），用于处理请求和管理集群。如果结点没有资格成为主结点，那么该结点永远不可能成为主结点；如果结点有资格成为主结点，只有在被其他候选主结点认可和被选举为主结点之后，才真正成为主结点。
node.data: true 配置该结点是数据结点，用于保存数据，执行数据相关的操作（CRUD，Aggregation）；
discovery.zen.minimum_master_nodes: //自动发现master节点的最小数，如果这个集群中配置进来的master节点少于这个数目，es的日志会一直报master节点数目不足。（默认为1）为了避免脑裂，个数请遵从该公式 => (totalnumber of master-eligible nodes / 2 + 1)。 * 脑裂是指在主备切换时，由于切换不彻底或其他原因，导致客户端和Slave误以为出现两个active master，最终使得整个集群处于混乱状态*
discovery.zen.ping.unicast.hosts： 集群个节点IP地址，也可以使用es-node等名称，需要各节点能够解析
```

**附录：**

1.查看容器内存

```shell
docker stats $(docker ps --format={{.Names}})
```

2.查看容器日志

```shell
docker logs 容器名/容器ID
```

