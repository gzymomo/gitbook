[TOC]

# 一、Zookeeper
## 1.1 启动zookeeper集群：
```shell
zkServer.sh start
```
## 1.2 查看zookeeper的状态
```shell
zkServer.sh status
```
# 二、Kafka
## 2.1 后台启动kafka集群服务
```shell
bin/kafka-server-start.sh config/server.properties &
```
## 2.2 创建新的topic
```bash
bin/kafka-topics.sh --create --zookeeper alary001:2181/home/hadoop/app/kafka_2.12-2.2.0,alary002:2181/home/hadoop/app/kafka_2.12-2.2.0,alary003:2181/home/hadoop/app/kafka_2.12-2.2.0 --replication-factor 3 --partitions 3 --topic SparkKafka
```

选型说明：
--topic：定义topic名
--replication-factor：定义副本数
--partitions：定义分区数

## 2.3 Slave-Kafka:
Kafka服务器：slave1,slave2,slave3
Kafka安装地址：/usr/etc/kafka-1.0.1
生产者：`./kafka-console-producer.sh --broker-list slave1:9092,slave2:9092,slave3:9092 --topic xxx`
消费者：`./kafka-console-consumer.sh --zookeeper slave1:2181,slave2:2181,slave3:2181 --topic xxx`
### 2.3.1 查看Topic列表：
```bash
./kafka-topics.sh --list --zookeeper slave1:2181
./kafka-topics.sh --list --zookeeper slave1:2181,slave2:2181,slave3:2181
./kafka-topics.sh --zookeeper slave1,slave2,slave3 --list
```
### 2.3.2 创建Topic：
```bash
./kafka-topics.sh --create --zookeeper slave1:2181,slave2:2181,slave3:2181 --replication-factor 3 --partitions 3 --topic xxx
```
### 2.3.3 Kafka消费者消息查看：
```bash
./kafka-console-consumer.sh --bootstrap-server slave1:9092,slave2:9092,slave3:9092 --from-beginning --topic xxx
```
### 2.3.4 查看topic副本信息
```bash
bin/kafka-topics.sh --describe --zookeeper alary001:2181/home/hadoop/app/kafka_2.12-2.2.0,alary002:2181/home/hadoop/app/kafka_2.12-2.2.0,alary003:2181/home/hadoop/app/kafka_2.12-2.2.0 --topic TestTopic
```
### 2.3.5 查看已经创建的topic信息
```bash
kafka-topics.sh --list --zookeeper alary001:2181/home/hadoop/app/kafka_2.12-2.2.0,alary002:2181/home/hadoop/app/kafka_2.12-2.2.0,alary003:2181/home/hadoop/app/kafka_2.12-2.2.0
```
### 2.3.6 测试生产者发送消息
```bash
bin/kafka-console-producer.sh --broker-list alary001:9092,alary002:9092,alary003:9092 --topic TestTopic
```
### 2.3.7 测试消费者消费消息
```bash
kafka-console-consumer.sh --bootstrap-server alary001:9092,alary002:9092,alary003:9092  --from-beginning --topic TestTopic
```
### 2.3.8 删除topic
```bash
bin/kafka-topics.sh --zookeeper alary001:2181/home/hadoop/app/kafka_2.12-2.2.0,alary002:2181/home/hadoop/app/kafka_2.12-2.2.0,alary003:2181/home/hadoop/app/kafka_2.12-2.2.0  --delete --topic TestTopic
```
需要server.properties中设置delete.topic.enable=true否则只是标记删除或者直接重启。
### 2.3.9 停止kafka服务
`bin/kafka-server-stop.sh stop`

# 三、Hadoop
Hadoop启动停止
分别启动hdfs组件： `hadoop-daemon.sh start|stop   namenode|datanode|secondartnamenode`
启动yarn：`yarn-daemon.sh		start|stop	resourecemanager|nodemanager`

各个模块分开启动：（配置ssh是前提）
```shell
start|stop-dfs.sh		start|stop-yarn.sh
```
# 四、Phoenix命令行模式
Phoenix命令模式：
`/usr/etc/phoenix-4.13.1-Hbase-1.3/bin/sqlline.py slave1:2181`
或：
`./sqlline.py slave1:2181`
Phoenix命令：
`!table，查看表		!describe table，查看表描述		!help，命令帮助		!exit，退出# `

# 五、Spark：
Spark启动：`./start`
Spark的Jar包存放路径：`/usr/etc/spark-2.3.0/jars`
Spark执行Jar命令：
```bash
nohup ./spark-submit --class com.xx.xx  --supervise --num-executors 3 --total-executor-cores 3 /root/program/xx.jar > /root/logs/zystorestreaming.log.txt 2>&1 &

nohup ./spark-submit --class com.xx.xx --master spark://master:7077 --supervise --num-executors 3 --total-executor-cores 3 --executor-memory 2g /root/program/xx.jar > /root/logs/xx.log.txt 2>&1 &
```

# 六、服务器挂掉后启动顺序：Zookeeper、Kafka、DataNode，HBase
## 6.1 启动Zookeeper：
```bash
./zkServer.sh ../conf/zoo.cfg
./zkServer.sh start
./zkCli.sh     # 启动客户端
```
## 6.2 启动Kafa：
```shell
cd /usr/etc/kafka-1.0.1/
./kafka-server-start.sh -daemon ../config/server.properties
bin/kafka-server-start.sh -daemon config/server.properties &
```
## 6.3 启动Datanode：
```bash
# 进去到Hadoop的sbin下：
./start-all.sh

```
## 6.4 启动HBase：
```bash
./hbase-daemon.sh start regionserver
```