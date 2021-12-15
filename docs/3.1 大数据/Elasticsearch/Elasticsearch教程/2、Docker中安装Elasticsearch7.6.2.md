# 一、Docker中安装Elasticsearch7.6.2

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

访问结果

## 插件安装

安装ik 分词器
下载对应的版本：elasticsearch-analysis-ik

为什么安装IK，轻量级。配置好词库也是可以用来中文分词，HanLP重量级，内置算法较多，不适合单独分词使用。

```bash
# 离线安装，下载对应插件zip
# https://github.com/medcl/elasticsearch-analysis-ik
docker cp /opt/elasticsearch-analysis-ik-7.6.2.zip elasticsearch:/opt
docker exec -it elasticsearch bash
cd plugins/
mkdir analysis-ik
unzip -d /usr/share/elasticsearch/plugins/analysis-ik/ /opt/elasticsearch-analysis-ik-7.6.2.zip 
exit
docker restart elasticsearch
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619164351529.png)

自定义词库

- 自定义字典
- 远程词库

参考：https://blog.csdn.net/qq_15973399/article/details/105793781

# 二、常用维护命令

```bash
# 查看所有索引信息
GET /_cat/indices?pretty
# 节点监控
GET /_cat/health?pretty
# 安装了哪些插件
GET _cat/plugin
```



# 三、监控和开发工具Kibana

Kibana 是为 Elasticsearch设计的开源分析和可视化平台。你可以使用 Kibana 来搜索，查看存储在 Elasticsearch 索引中的数据并与之交互。你可以很容易实现高级的数据分析和可视化，以图标的形式展现出来。

我们的服务器IP是172.18.63.211

```bash
docker run  --restart=always --link elasticsearch:elasticsearch --name kibana -p 5601:5601 -d kibana:7.6.2
```

进入容器修改配置文件kibana.yml

```bash
docker exec  -it kibana bash
vi config/kibana.yml
########################
# 指定es的地址
elasticsearch.hosts: ["http://172.18.63.211:9200"]
# 中文化
i18n.locale: "zh-CN"
# 修改外网访问 可选
server.host: "0.0.0.0"
exit
########################
docker restart kibana
```

打开地址：http://172.18.63.211:5601

测试分词工具

```json
POST _analyze
{
  "text": "检测甘蓝型油菜抗磺酰脲类除草剂基因BnALS3R的引物与应用",
  "analyzer": "hanlp"
}
```

新增索引库

```json
PUT achievement
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}


PUT achievement/_mapping
{
  "properties": {
    "id": {
      "type": "text"
    },
    "owner": {
      "type": "text"
    },
    "title": {
      "type": "text",
      "analyzer": "hanlp"
    },
    "description": {
      "type": "text",
      "analyzer": "hanlp"
    },
    "update_time":{
      "type": "date"
    }
  }
}
```

# 四、数据同步Logstash

用于收集、解析和转换日志，同步数据等。

## 4.1 安装

    docker pull logstash:7.5.0 

配置文件目录

```
mkdir -p /usr/local/logstash/config
cd /usr/local/logstash/config
touch logstash.yml
vi log4j2.properties
#####添加以下内容
logger.elasticsearchoutput.name = logstash.outputs.elasticsearch
logger.elasticsearchoutput.level = debug
#####
vi pipelines.yml
####
- pipeline.id: logstash-match
  path.config: "/usr/share/logstash/config/*.conf"
  pipeline.workers: 3
####
```

同时需要将MySQL的驱动包放入配置文件中。
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8yMTM2NjMzMy1iODI4ZDVlYjQzZjdjOTVkLnBuZw?x-oss-process=image/format,png)

再创建配置文件即可
这里给一个例子，是定时同步mysql数据到es中的。*

# logstash-mysql-es.conf
```yaml
# logstash-mysql-es.conf
input{
  jdbc{
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://172.18.63.211:3306/open_intelligence?characterEncoding=utf8&serverTimezone=Asia/Shanghai"
    jdbc_user => "docker"
    jdbc_password => "docker@12345"
    jdbc_paging_enabled => true
    jdbc_page_size => 10000
    jdbc_fetch_size => 10000
    connection_retry_attempts => 3
    connection_retry_attempts_wait_time => 1
    jdbc_pool_timeout => 5
    use_column_value => true
    tracking_column => "update_time"
    tracking_column_type => "timestamp"
    record_last_run => true
    last_run_metadata_path => "/usr/share/logstash/mysql/goods_achievement"
    statement => "select * from goods_achievement where update_time > :sql_last_value"
    schedule => "* */30 * * * *"
  }
}

filter{
  mutate {
    split => { "feature1" => ";" }
  }
  mutate {
    split => { "feature2" => ";" }
  }
  mutate {
    split => { "feature3" => ";" }
  }
}

output {
  elasticsearch {
    document_id => "%{id}"
    index => "goods_achievement"
    hosts => ["http://172.18.63.211:9200"]
  }
}


```

启动

```bash
docker run -d -p 5044:5044 -p 9600:9600 -it \
-e TZ=Asia/Shanghai \
--name logstash --restart=always \
-v /usr/local/logstash/config/:/usr/share/logstash/config/ \
-v /usr/local/logstash/mysql/:/usr/share/logstash/mysql/ \
--privileged=true \
logstash:7.6.2
```

如果报错了

    Error: com.mysql.cj.jdbc.Driver not loaded. :jdbc_driver_library is not set, are you sure you included the proper driver client libraries in your classpath?

可以尝试将驱动器即mysql-connector-java-xxxx-bin.jar拷贝到 logstash目录\logstash-core\lib\jars 下
如：

```bash
cd /usr/local/logstash/config
docker cp mysql-connector-java-8.0.17.jar logstash:/usr/share/logstash/logstash-core/lib/jars
```

检测配置文件

    bin/logstash -f /usr/local/logstash/config/mysql-es-patent.conf -t

完成，你可以进行开发了。