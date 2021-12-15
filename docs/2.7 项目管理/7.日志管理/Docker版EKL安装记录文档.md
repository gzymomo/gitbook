# Docker版EKL安装记录文档

- 拉取以下三个镜像

```
docker.io/logstash        7.5.2               b6518c95ed2f        6 months ago        805 MB
docker.io/kibana          7.5.2               a6e894c36481        6 months ago        950 MB
docker.io/elasticsearch   7.5.2               929d271f1798        6 months ago        779 MB
```

# 部署ES

- 修改系统配置文件

[官方参考链接](http://www.neohope.org/2016/12/28/elasticsearch-docker官方镜像无法运行/)

## elasticsearch docker官方镜像无法运行

elasticsearch5.1 docker官方镜像运行时会报错：

```bash
ERROR: bootstrap checks failed
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

那是因为vm.max_map_count达不到es的最低要求262144，修改方式有两种：

```bash
#一次生效
sudo sysctl -w vm.max_map_count=262144
```

```bash
#永久生效
sudo vi /etc/sysctl.conf
#添加这一行
vm.max_map_count=262144
 
#加载配置
sudo sysctl -p
```

**注意一**

> ⚠️准备 config,data,logs三个文件夹，文件夹里需要有配置的就config文件夹。

**注意二**

> ⚠️ config 是elasticsearch容器默认的配置文件，我们需要修改elasticsearch.yml文件，不然还是启动不了，配置如下：



```yaml
#cluster.name: "docker-cluster"
#network.host: 0.0.0.0
cluster.name: "docker-cluster"
node.name: node-1
network.host: 0.0.0.0
bootstrap.system_call_filter: false
cluster.initial_master_nodes: ["node-1"]
xpack.license.self_generated.type: basic
```

**注意三**

> ⚠️ data,logs文件夹可手动创建，需要给他们root账号的读写执行权限，我这给的是775 权限足够了，权限可参考如下：



```bash
[root@localhost elasticsearch]# ls -al
总用量 0
drwxr-xr-x  6 root root  89 7月  21 16:45 .
drwxr-xr-x  5 root root  73 7月  21 16:25 ..
drwxr-xr-x  2 root root 178 7月  21 16:44 config
drwxrwxr-x  3 root root  27 7月  21 16:53 data
drwxrwxr-x 10 root root 283 7月  21 16:40 elasticsearch.bak
drwxrwxr-x  2 root root 133 7月  21 16:55 logs
```

- 最后docker run



```bash
docker run --name elasticsearch -p 9200:9200 -v /data/EKL/elasticsearch/config:/usr/share/elasticsearch/config  -v /data/EKL/elasticsearch/data:/usr/share/elasticsearch/data  -v /data/EKL/elasticsearch/logs:/usr/share/elasticsearch/logs   -v /etc/localtime:/etc/localtime:ro   -itd elasticsearch:7.5.2
```

**注意四**

> ⚠️端口可以自己改，我这就默认了
> 确认一下是否起来了



```bash
[root@localhost elasticsearch]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                              NAMES
1b0763e52708        elasticsearch:7.5.2   "/usr/local/bin/do..."   14 minutes ago      Up 9 minutes        0.0.0.0:9200->9200/tcp, 9300/tcp   elasticsearch
```

> 起来了后用浏览器访问一下 127.0.0.1:9200 (127.0.0.1是你自己宿主机的IP地址)
> [![-w806](https://img2020.cnblogs.com/blog/2151193/202011/2151193-20201119195246292-669697386.jpg)](https://img2020.cnblogs.com/blog/2151193/202011/2151193-20201119195246292-669697386.jpg)

# 部署kibana

- 准备 kibana的 config 文件夹，配置内容如下

```yaml
#
## ** THIS IS AN AUTO-GENERATED FILE **
##
#
## Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://127.0.0.1:9200" ]  # 127.0.0.1:9200改为你自己的elasticsearch地址
xpack.monitoring.ui.container.elasticsearch.enabled: true  # 解释链接https://blog.csdn.net/u011311291/article/details/100041912
i18n.locale: zh-CN  #汉化
kibana.index: ".kibana"  #配置本地索引
```

- docker run 启动

```bash
docker run -itd --name kibana -p 5601:5601 -v /data/EKL/kibana/config:/usr/share/kibana/config  -v /etc/localtime:/etc/localtime:ro kibana:7.5.2
```

**注意五**

> ⚠️开放所用到的端口，我这里是9200，5601，浏览器访问127.0.0.1:5601 (127.0.0.1是你自己宿主机的IP)
> [![-w1410](https://img2020.cnblogs.com/blog/2151193/202011/2151193-20201119195246325-1712065424.jpg)](https://img2020.cnblogs.com/blog/2151193/202011/2151193-20201119195246325-1712065424.jpg)

# 部署logstash

- 准备好logstash的 config跟pipeline文件

**注意六**

> ⚠️修改config下的 logstash.yml ； logstash-sample.conf 两个配置文件中的 elasticsearch 地址配置地址如下



```yaml
[root@test config]# cat logstash.yml 
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://127.0.0.1:9200" ]

[root@test config]# cat pipelines.yml 
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
  path.config: "/usr/share/logstash/pipeline"     #这是服务配置路径，可改。我这暂且不改。


[root@test config]# cat logstash-sample.conf

# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
}
```

**注意七**

> ⚠️：以上 127.0.0.1:9200 需改为自己宿主机的地址
> 其实到这一步就差不多了，接下来就是对监控的logs做相关的配置了，我是对java的logs做个监控，需在 pipeline 下创建一个 java.conf （命名可改随你喜欢）,配置如下

## logstash日志配置



```yaml
[root@test pipeline]# vim java.conf 

input {
    file{
      path => "/DATA/logs/chenfan-base.log"   #log路径
      type => "base"                        # 打个标签
      start_position => "beginning"
        }
    }

output {
    if [type] == "base" {               # 条件
        elasticsearch {
            hosts => ["127.0.0.1:9200"]  #elasticsearch 服务地址
            index => "base-%{+YYYY.MM.dd}"  # 索引名字命名
            codec => "json"                                 #需将java日志装成json格式
                }
            }
}
```



```bash
docker run -itd --name logstash -p 5044:5044 -p 9600:9600  -v /DATA/logstash/docker/logstash/config:/usr/share/logstash/config -v /DATA/logstash/docker/logstash/pipeline:/usr/share/logstash/pipeline  -v /DATA/logstash/log:/DATA/logs  -v /etc/localtime:/etc/localtime:ro  logstash:7.5.2
```