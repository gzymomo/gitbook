[Docker-搭建日志监控系统](https://www.cnblogs.com/1ssqq1lxr/p/14622927.html)



## 项目中常用集中日志收集工具

- ### Logstash

Logstash是一个开源数据收集引擎，具有实时管道功能。Logstash可以动态地将来自不同数据源的数据统一起来，并将数据标准化到你所选择的目的地。

- 优点

  Logstash 主要的有点就是它的灵活性，主要因为它有很多插件，详细的文档以及直白的配置格式让它可以在多种场景下应用。我们基本上可以在网上找到很多资源，几乎可以处理任何问题。

- 缺点

  Logstash 致命的问题是它的性能以及资源消耗(默认的堆大小是  1GB)。尽管它的性能在近几年已经有很大提升，与它的替代者们相比还是要慢很多的。这里有 Logstash 与 rsyslog  性能对比以及Logstash 与 filebeat 的性能对比。它在大数据量的情况下会是个问题。

- ### Filebeat

作为 Beats 家族的一员，Filebeat 是一个轻量级的日志传输工具，它的存在正弥补了 Logstash 的缺点：Filebeat 作为一个轻量级的日志传输工具可以将日志推送到中心 Logstash。

- 优点

  Filebeat  只是一个二进制文件没有任何依赖。它占用资源极少，尽管它还十分年轻，正式因为它简单，所以几乎没有什么可以出错的地方，所以它的可靠性还是很高的。它也为我们提供了很多可以调节的点，例如：它以何种方式搜索新的文件，以及当文件有一段时间没有发生变化时，何时选择关闭文件句柄。

- 缺点

  Filebeat 的应用范围十分有限，所以在某些场景下我们会碰到问题。例如，如果使用 Logstash  作为下游管道，我们同样会遇到性能问题。正因为如此，Filebeat 的范围在扩大。开始时，它只能将日志发送到 Logstash 和  Elasticsearch，而现在它可以将日志发送给 Kafka 和 Redis，在 5.x 版本中，它还具备过滤的能力。

- ### Fluentd （Docker日志驱动支持）

Fluentd 创建的初衷主要是尽可能的使用 JSON 作为日志输出，所以传输工具及其下游的传输线不需要猜测子字符串里面各个字段的类型。这样，它为几乎所有的语言都提供库，这也意味着，我们可以将它插入到我们自定义的程序中。

- 优点

  和多数 Logstash 插件一样，Fluentd 插件是用 Ruby 语言开发的非常易于编写维护。所以它数量很多，几乎所有的源和目标存储都有插件(各个插件的成熟度也不太一样)。这也意味这我们可以用 Fluentd 来串联所有的东西。

- 缺点

  因为在多数应用场景下，我们会通过 Fluentd  得到结构化的数据，它的灵活性并不好。但是我们仍然可以通过正则表达式，来解析非结构化的数据。尽管，性能在大多数场景下都很好，但它并不是***的，和 syslog-ng 一样，它的缓冲只存在与输出端，单线程核心以及 Ruby GIL  实现的插件意味着它大的节点下性能是受限的，不过，它的资源消耗在大多数场景下是可以接受的。对于小的或者嵌入式的设备，可能需要看看 Fluent  Bit，它和 Fluentd 的关系与 Filebeat 和 Logstash 之间的关系类似。

## 使用Docker-Compose搭建EFK收集中心

1. ### 创建docker-compose.yml

新建一个efk目录，然后进入目录下：

```
version: '3'
services:
  web:
    image: httpd
    ports:
      - "80:80"
    links:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: httpd.access

  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links:
      - "elasticsearch"
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.10.2
    environment:
      - "discovery.type=single-node"
    expose:
      - "9200"
    ports:
      - "9200:9200"

  kibana:
    image: kibana:7.10.1
    links:
      - "elasticsearch"
    ports:
      - "5601:5601"
```

1. ### 创建fluentd镜像以及配置config与插件

新建 fluentd/Dockerfile

```
FROM fluent/fluentd:v1.12.0-debian-1.0
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "4.3.3"]
USER fluent
```

新建 fluentd/conf/fluent.conf

```
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *.**>
  @type copy

  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>

  <store>
    @type stdout
  </store>
</match>
```

1. ### 启动服务

```
docker-compose up
```

1. ### 多次请求httpd服务生成日志

```
$ curl localhost:80
```

1. 验证日志收集

打开浏览器访问http://localhost:5601

初始化创建fluentd-*索引

![创建索引](https://files.mdnice.com/user/11463/d19c1041-3f65-493e-ba95-ceca69d62a7d.png)创建索引

此时可以看到Httpd 生成的日志已经被收集

![log](https://files.mdnice.com/user/11463/104d5097-c800-4a47-ac8d-c79c21b9be13.png)log

## 使用fluentd收集关键点

1. 如何指定fluentd驱动

- 修改daemon.json（全局）

  ```
  "log-driver":"fluentd",
  "log-opts":{
   "fluentd-address":"192.168.0.133:24224"
  },
  ```

- 单个容器

  ```
  # 启动增加 
  --fluentd-address=localhost:24224  --log-driver=fluentd
  #注意：注意，此时如果fluentd服务挂了 服务启动不起来的，可以在服务启动时候 加上
  --log-opt=fluentd-async-connect
  ```