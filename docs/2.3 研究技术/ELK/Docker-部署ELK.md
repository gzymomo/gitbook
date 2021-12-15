[TOC]

 [散尽浮华](https://home.cnblogs.com/u/kevingrace/)—[ELK实时日志分析平台环境部署--完整记录](https://www.cnblogs.com/kevingrace/p/5919021.html)



# 1、ELK概述

ELK是Elasticsearch、Logstash、Kibana三大开源框架首字母大写简称。
 - Elasticsearch是一个基于Lucene、分布式、通过Restful方式进行交互的近实时搜索平台框架。像类似百度、谷歌这种大数据全文搜索引擎的场景都可以使用Elasticsearch作为底层支持框架.
 - Logstash是ELK的中央数据流引擎，用于从不同目标（文件/数据存储/MQ）收集的不同格式数据，经过过滤后支持输出到不同目的地（文件/MQ/redis/elasticsearch/kafka等）。
 - Kibana可以将elasticsearch的数据通过友好的页面展示出来，提供实时分析的功能。
 - FileBeat，它是一个轻量级的日志收集处理工具(Agent)，Filebeat占用资源少，适合于在各个服务器上搜集日志后传输给Logstash，官方也推荐此工具。

通过Logstash去收集每台服务器日志文件，然后按定义的正则模板过滤后传输到Kafka或redis，然后由另一个Logstash从KafKa或redis读取日志存储到elasticsearch中创建索引，最后通过Kibana展示给开发者或运维人员进行分析。这样大大提升了运维线上问题的效率。除此之外，还可以将收集的日志进行大数据分析，得到更有价值的数据给到高层进行决策。



# 2、Docker容器中运行ES,Kibana,Cerebro和Logstash安装与数据导入ES

## 一、Docker容器中运行ES,Kibana,Cerebro

#### 1、所需环境

```
Docker + docker-compose
```

首先环境要部署好 `Docker` 和 `docker-compose`

**检验是否成功**

**命令** `docker —version`

```
xubdeMacBook-Pro:~ xub$ docker --version
Docker version 17.03.1-ce-rc1, build 3476dbf
```

**命令** `docker-compose —version`

```
xubdeMacBook-Pro:~ xub$ docker-compose --version
docker-compose version 1.11.2, build dfed245
```

#### 2、docker-compose.yml

我们可以简单把docker-compose.yml理解成一个类似Shell的脚本，这个脚本定义了运行多个容器应用程序的信息。

```yaml
version: '2.2'
services:
  cerebro:
    image: lmenezes/cerebro:0.8.3
    container_name: cerebro
    ports:
      - "9000:9000"
    command:
      - -Dhosts.0.host=http://elasticsearch:9200
    networks:
      - es7net
  kibana:
    image: docker.elastic.co/kibana/kibana:7.1.0
    container_name: kibana7
    environment:
      - I18N_LOCALE=zh-CN
      - XPACK_GRAPH_ENABLED=true
      - TIMELION_ENABLED=true
      - XPACK_MONITORING_COLLECTION_ENABLED="true"
    ports:
      - "5601:5601"
    networks:
      - es7net
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_01
    environment:
      - cluster.name=xiaoxiao
      - node.name=es7_01
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - es7net
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.1.0
    container_name: es7_02
    environment:
      - cluster.name=xiaoxiao
      - node.name=es7_02
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.seed_hosts=es7_01,es7_02
      - cluster.initial_master_nodes=es7_01,es7_02
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - es7data2:/usr/share/elasticsearch/data
    networks:
      - es7net

volumes:
  es7data1:
    driver: local
  es7data2:
    driver: local

networks:
  es7net:
    driver: bridge
```

启动命令

```
docker-compose up      #启动
docker-compose down    #停止容器
docker-compose down -v #停止容器并且移除数据
```

#### 3、查看是否成功

**es访问地址**

```
localhost:9200  #ES默认端口为9200
```

![img](https://img2018.cnblogs.com/blog/1090617/201908/1090617-20190829211410397-404455124.png)

**kibana访问地址**

```
localhost:5601 #kibana默认端口5601
```

![img](https://img2018.cnblogs.com/blog/1090617/201908/1090617-20190829211522044-1950731908.png)

**cerebro访问地址**

```
localhost:9000 #cerebro默认端口9000
```

![img](https://img2018.cnblogs.com/blog/1090617/201908/1090617-20190829230050305-1074367035.png)

整体这样就安装成功了。

`说明` 项目是在Mac系统部署成功的,尝试在自己的阿里云服务进行部署但是因为内存太小始终无法成功。



## 二、 Logstash安装与数据导入ES

`注意` Logstash和kibana下载的版本要和你的elasticsearch的版本号一一致。

#### 1、配置movices.yml

这个名称是完全任意的

```yaml
# input代表读取数据 这里读取数据的位置在data文件夹下，文件名称为movies.csv
input {
  file {
    path => "/Users/xub/opt/logstash-7.1.0/data/movies.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
    separator => ","
    columns => ["id","content","genre"]
  }

  mutate {
    split => { "genre" => "|" }
    remove_field => ["path", "host","@timestamp","message"]
  }

  mutate {

    split => ["content", "("]
    add_field => { "title" => "%{[content][0]}"}
    add_field => { "year" => "%{[content][1]}"}
  }

  mutate {
    convert => {
      "year" => "integer"
    }
    strip => ["title"]
    remove_field => ["path", "host","@timestamp","message","content"]
  }

}
# 输入位置 这里输入数据到本地es ,并且索引名称为movies
output {
   elasticsearch {
     hosts => "http://localhost:9200"
     index => "movies"
     document_id => "%{id}"
   }
  stdout {}
}
```

**启动命令** : 启动命令会和配置文件movices.yml的摆放位置有关，进入bin目录

```
./logstash ../movices.yml 
```

**movices.yml存放的位置**

![img](https://img2018.cnblogs.com/blog/1090617/201908/1090617-20190829211811236-890093644.png)

**启动成功**

![img](https://img2018.cnblogs.com/blog/1090617/201908/1090617-20190829211819906-1388950755.jpg)

这个时候你去cerebro可视化界面可以看到,已经有名称为`movies的索引`存在的,上面的图片其实已经存在movies索引了，因为我是Logstash数据导入ES成功才截的图。

`总结`总的来说这里还是简单的，之前通过Logstash将Mysql数据数据迁移到es会相对复杂点，毕竟它还需要一个数据库驱动包。

这样环境就已经搭建成功了，接下来的学习都在这个的基础上进行演示。