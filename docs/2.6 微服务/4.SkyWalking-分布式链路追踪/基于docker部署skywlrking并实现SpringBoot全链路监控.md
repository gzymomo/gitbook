

# 一、安装环境部署

下载镜像：

```bash
$ docker pull docker pull elasticsearch:7.10.1
$ docker pull apache/skywalking-oap-server:8.3.0-es7
$ docker pull apache/skywalking-ui:8.3.0
```



## 1.1 Docker中安装Elasticsearch7.6.2（方式一）

安装Elasticsearch

注意：使用版本为 7.6.2，你可以选择其他版本
拉取镜像

```bash
docker pull elasticsearch:7.6.2
```

启动容器

```bash
docker run --restart=always -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
--name='elasticsearch' --cpuset-cpus="1" -m 2G -d elasticsearch:7.6.2xxxxxxxxxx docker run --restart=always -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" \-e ES_JAVA_OPTS="-Xms512m -Xmx512m" \--name='elasticsearch' --cpuset-cpus="1" -m 2G -d elasticsearch:7.6.2123
```

说明：

- -v /opt/hanlp:/opt/hanlp如果使用了hanlp的分词，所以需要挂载词库
- ES_JAVA_OPTS可以设置参数
- 单节点启动

访问地址：http://172.18.63.211:9200



## 1.2 安装部署elasticsearch:7.10.1（方式二）

### 修改配置文件

（可选）修改主机配置参数：

```bash
$ vi /etc/sysctl.conf
vm.max_map_count=262144
$ sysctl -p

$ vi/etc/systemd/system.conf
DefaultLimitNOFILE=65536
DefaultLimitNPROC=32000
DefaultLimitMEMLOCK=infinity
$ systemctl daemon-reload

$ vi /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 4096
* hard nproc 4096
* hard memlock unlimited
* soft memlock unlimited
```

### 启动elasticsearch

创建持久化目录，并启动elasticsearch

```bash
$ mkdir -p /data/elasticsearch/data
$ mkdir -p /data/elasticsearch/logs
$ chmod -R 777 /data/elasticsearch/data
$ chmod -R 777 /data/elasticsearch/logs
$ docker run -d --name=es7 \
  --restart=always \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
elasticsearch:7.10.1
```

## 1.3 安装oap

**注意：等待elasticsearch完全启动之后，再启动oap**

```bash
$ docker run --name oap --restart always -d \
--restart=always \
-e TZ=Asia/Shanghai \
-p 12800:12800 \
-p 11800:11800 \
--link es7:es7 \
-e SW_STORAGE=elasticsearch7 \
-e SW_STORAGE_ES_CLUSTER_NODES=es7:9200 \
apache/skywalking-oap-server:8.3.0-es7
```

说明：这里指定elasticsearch 来存储数据

- --link： es7和你启动的es容器的name保持一致
- -e SW_STORAGE_ES_CLUSTER_NODES：es7也可改为你es服务器部署的Ip地址，即ip:9200



如果有如下报错：

```bash
[Entrypoint] Apache SkyWalking Docker Image
Current image doesn't Elasticsearch 6
```

把SW_STORAGE改成elasticsearch7即可



## 1.4 安装ui

```bash
$ docker run -d --name skywalking-ui \
--restart=always \
-e TZ=Asia/Shanghai \
-p 8080:8080 \
--link oap:oap \
-e SW_OAP_ADDRESS=oap:12800 \
apache/skywalking-ui:8.3.0
```

说明：

- -- link：注意容器的名称
- -e SW_OAP_ADDRESS：oap容器名称，也可改为ip:12800，即其他服务的IP和端口



## 1.5 下载源码包，会用到其中的agent

http://skywalking.apache.org/downloads/

![](https://lovebetterworld.com/skywalking.png)





# 二、SpringBoot集成Skywalking

## 2.1 配置

### 文件准备

将apache-skywalking-apm-bin-es7/agent文件夹拷贝到发布容器中，位置可以根据情况调整。

```bash
cp -r ./agent/*  /opt/skywalkingAgent
```

文件说明

- config/agent.config：为客户端代理配置文件，可以根据系统情况进行响应调整，这里就不详细说明。
- logs：SW agent相关运行情况日志。
- skywalking-agent.jar：agent代理jar包。

### 使用方式

优先级：探针 > JVM配置 > 系统环境变量 > agent.config

一般都使用探针方式，其他方式就不介绍了，配置方式如下：

- 格式1(推荐)：-javaagent:/path/to/skywalking-agent.jar={config1}={value1},{config2}={value2}

`-javaagent:../skywalking-agent.jar=agent.service_name=fw-gateway,collector.backend_service=127.0.0.1:11800`

- 格式2：-Dskywalking.[option1]=[value2]

```java
-javaagent:../skywalking-agent.jar -Dskywalking.agent.service_name=fw-gateway -Dskywalking.collector.backend_service=127.0.0.1:11800
```

一般配置下面两项即可：

    agent.service_name：客户端服务名，在apm系统中显示的服务名称。
    collector.backend_service：SW上传的服务地址。
访问相关服务地址后可以在SW控制台中查看相关信息

### 详细配置

```yaml
# 探针代理命名空间，跨进场传输时的header标记
# agent.namespace=${SW_AGENT_NAMESPACE:default-namespace}
 
# 代理服务名称，在ui显示的的服务名称
agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
 
# 采样率，每3秒trace几条
# 小于等于0使用默认值，每条都采样
# agent.sample_n_per_3_secs=${SW_AGENT_SAMPLE:-1}
 
# server端的认证配置
# agent.authentication = ${SW_AGENT_AUTHENTICATION:xxxx}
 
# 单个trace的span跨度，会有内存开销
# agent.span_limit_per_segment=${SW_AGENT_SPAN_LIMIT:150}
 
# 忽略span，多个逗号分隔
# agent.ignore_suffix=${SW_AGENT_IGNORE_SUFFIX:.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg}
 
# 开启debug模式，用于发现sw的bug
# agent.is_open_debugging_class = ${SW_AGENT_OPEN_DEBUG:true}
 
# 是否缓存sw的增强包
# agent.is_cache_enhanced_class = ${SW_AGENT_CACHE_CLASS:false}
 
# sw指令类的缓存模式: MEMORY or FILE
# MEMORY: 缓存在内存中，占用内容
# FILE: 缓存到/class-cache目录下, 退出自动删除
# agent.class_cache_mode = ${SW_AGENT_CLASS_CACHE_MODE:MEMORY}
 
# 操作名称的最大长度，最大不要超过190
# agent.operation_name_threshold=${SW_AGENT_OPERATION_NAME_THRESHOLD:150}
 
# 代理默认使用gRPC纯文本。
# 如果为true，则即使未检测到CA文件，SkyWalking代理也会使用TLS。
# agent.force_tls=${SW_AGENT_FORCE_TLS:false}
 
# 是否使用新的配置文件
# profile.active=${SW_AGENT_PROFILE_ACTIVE:true}
 
# 并行监听的分段数量
# profile.max_parallel=${SW_AGENT_PROFILE_MAX_PARALLEL:5}
 
# 监听分段的最大时间，超出则停止
# profile.duration=${SW_AGENT_PROFILE_DURATION:10}
 
# 线程最大跨线程数
# profile.dump_max_stack_depth=${SW_AGENT_PROFILE_DUMP_MAX_STACK_DEPTH:500}
 
# 数据快照缓冲区大小
# profile.snapshot_transport_buffer_size=${SW_AGENT_PROFILE_SNAPSHOT_TRANSPORT_BUFFER_SIZE:50}
 
# server服务地址
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}
 
# sw日志文件
logging.file_name=${SW_LOGGING_FILE_NAME:skywalking-api.log}
 
# sw日志级别
logging.level=${SW_LOGGING_LEVEL:INFO}
 
# 日志文件地址
# logging.dir=${SW_LOGGING_DIR:""}
 
# 单个日志文件大小, default: 300 * 1024 * 1024 = 314572800
# logging.max_file_size=${SW_LOGGING_MAX_FILE_SIZE:314572800}
 
# 最大日志文件数量，滚动删除，小于等于0时不删除
# logging.max_history_files=${SW_LOGGING_MAX_HISTORY_FILES:-1}
 
# 忽略异常
# statuscheck.ignored_exceptions=${SW_STATUSCHECK_IGNORED_EXCEPTIONS:}
 
# 异常trace的跟踪深度，建议小于10，对性能有影响
# statuscheck.max_recursive_depth=${SW_STATUSCHECK_MAX_RECURSIVE_DEPTH:1}
 
# 扩展插件
plugin.mount=${SW_MOUNT_FOLDERS:plugins,activations}
 
# 不加载插件
# plugin.exclude_plugins=${SW_EXCLUDE_PLUGINS:}
 
# mysql插件
# plugin.mysql.trace_sql_parameters=${SW_MYSQL_TRACE_SQL_PARAMETERS:false}
 
# Kafka地址
# plugin.kafka.bootstrap_servers=${SW_KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
 
# Match spring bean with regex expression for classname
# plugin.springannotation.classname_match_regex=${SW_SPRINGANNOTATION_CLASSNAME_MATCH_REGEX:}
```

# 参考链接

- [基于docker部署skywalking实现全链路监控](https://www.jianshu.com/p/a237d6e810c6)
- [Skywalking Agent配置及使用(2)](https://blog.csdn.net/lizz861109/article/details/107519853)
- [基于docker部署skywalking实现全链路监控](https://www.cnblogs.com/xiao987334176/p/13530575.html)

