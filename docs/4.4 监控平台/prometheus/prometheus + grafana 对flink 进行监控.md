- [prometheus + grafana  对flink 进行监控](https://blog.51cto.com/flyfish225/2566434)



# prometheus + grafana  对flink 进行监控

标签（空格分隔）： flink系列 

------

> - 一：flink监控简介
> - 二：Flink的Metric架构
> - 三： prometheus + grafana 的 对 flink 的监控部署构建

------

## 一：flink监控简介

### 1.1 前言

```
Flink提供的Metrics可以在Flink内部收集一些指标，通过这些指标让开发人员更好地理解作业或集群的状态。由于集群运行后很难发现内部的实际状况，跑得慢或快，是否异常等，开发人员无法实时查看所有的Task日志，比如作业很大或者有很多作业的情况下，该如何处理？此时Metrics可以很好的帮助开发人员了解作业当前状况。对于很多大中型企业来讲，我们对进群的作业进行管理时，更多的是关心作业精细化实时运行状态。例如，实时吞吐量的同比环比、整个集群的任务运行概览、集群水位，或者监控利用 Flink 实现的 ETL 框架的运行情况等，这时候就需要设计专门的监控系统来监控集群的任务作业情况。
```

------

## 二： Flink的Metric架构

### 2.1 flink metric

```
Flink Metrics是Flink实现的一套运行信息收集库，我们不但可以收集Flink本身提供的系统指标，比如CPU、内存、线程使用情况、JVM垃圾收集情况、网络和IO等，还可以通过继承和实现指定的类或者接口打点收集用户自定义的指标。
通过使用Flink Metrics我们可以轻松地做到：
• 实时采集Flink中的Metrics信息或者自定义用户需要的指标信息并进行展示；
• 通过Flink提供的Rest API收集这些信息，并且接入第三方系统进行展示。
```

![image_1eiusfteq1f9u1dkq1hhr1i03g219.png-589.9kB](http://static.zybuluo.com/zhangyy/drwwwrovdfeubg0gxhegjjg7/image_1eiusfteq1f9u1dkq1hhr1i03g219.png)

### 2.2 监控架构

```
从Flink Metrics架构来看，指标获取方式有两种。一是REST-ful API，Flink Web UI中展示的指标就是这种形式实现的。二是reporter，通过reporter可以将metrics发送给外部系统。Flink内置支持JMX、Graphite、Prometheus等系统的reporter，同时也支持自定义reporter。
由于Flink Web UI所提供的metrics数量较少，也没有时序展示，无法满足实际生产中的监控需求。Prometheus+Grafana是业界十分普及的开源免费监控体系，上手简单，功能也十分完善。
```

![image_1eiusifnu6fhfc91rg016qq1p4jm.png-88.1kB](http://static.zybuluo.com/zhangyy/wb0avp3j2nbqdoxcj3scval7/image_1eiusifnu6fhfc91rg016qq1p4jm.png)

## 三：prometheus + grafana 的 对 flink 的监控部署构建

### 3.1 安装prometheus

```
Prometheus本身也是一个导出器(exporter)，提供了关于内存使用、垃圾收集以及自身性能
与健康状态等各种主机级指标。
prometheus官网下载址：

https://prometheus.io/download/
wget https://github.com/prometheus/prometheus/releases/download/v2.21.0/prometheus-2.21.0.linux-amd64.tar.gz

# tar zxvf prometheus-2.21.0.linux-amd64.tar.gz

# mv prometheus-2.21.0.linux-amd64 /usr/local/prometheus

# chmod +x /usr/local/prometheus/prom*

# cp -rp /usr/local/prometheus/promtool /usr/bin/
```

### 3.2 配置prometheus

```
最后 加上pushgateway 收集：

此处将pushgateway 与 prometheus 安装在一台机器上面

- job_name: 'linux'
  static_configs:
  - targets: ['192.168.100.15:9100']
     labels:
        app: node05
        nodename: node05.vpc.flyfish.cn
        role: node
- job_name: 'pushgateway'
  static_configs:
  - targets: ['192.168.100.15:9091']
  labels:
     instance: 'pushgateway'
```

------

```
prometheus  的开机启动：

cat > /usr/lib/systemd/system/prometheus.service <<EOF
[Unit]
Description=Prometheus
[Service]
ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml --
storage.tsdb.path=/usr/local/prometheus/data --web.enable-lifecycle --storage.tsdb.retention.time=180d
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
---

#service prometheus start 
#chkconfig prometheus on 
```

### 3.3 安装 prometheus 的node_exporter  与 pushgateway  的插件

```
  node_exporter :  

    #tar -zxvf node_exporter-1.0.1.linux-amd64.tar.gz
    #mv node_exporter-1.0.1.linux-amd64 /usr/local/node_exporter
    #/usr/local/node_exporter/node_exporter &
```

------

```
pushgateway:
     #tar -zxvf  pushgateway-1.2.0.linux-amd64.tar.gz
     #mv pushgateway-1.2.0.linux-amd64  /usr/local/pushgateway/
     # /usr/local/pushgateway/pushgateway &
```

------

\###3.4 flink metric 的配置

```
flink-conf.yaml
到最后加上
----
metrics.reporter.promgateway.class: org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter
metrics.reporter.promgateway.host: 192.168.100.15
metrics.reporter.promgateway.port: 9091
metrics.reporter.promgateway.jobName: pushgateway
metrics.reporter.promgateway.randomJobNameSuffix: true
metrics.reporter.promgateway.deleteOnShutdown: true
----

然后同步所有flink的 works 节点
```

------

```
重启flink 的集群 

./stop-cluster.sh

./start-cluster.sh 
```

#### 3.4.1 打开pushgateway

![image_1eiv1cltf11s81obm143a13g1sh9m.png-157.8kB](http://static.zybuluo.com/zhangyy/e4qg9jk1h2ut879pkhaeblqd/image_1eiv1cltf11s81obm143a13g1sh9m.png)

![image_1eiv1a49b8sq18ke1v9s1and1ive9.png-494.1kB](http://static.zybuluo.com/zhangyy/lqf8haio5l0g9cnlfhicyu5f/image_1eiv1a49b8sq18ke1v9s1and1ive9.png)

#### 3.4.2 prometheus 页面

![image_1eiv1f4do1sv39531mcgb326fn13.png-308.3kB](http://static.zybuluo.com/zhangyy/4oo09kxip2orxkdthc0gzc49/image_1eiv1f4do1sv39531mcgb326fn13.png)

![image_1eiv1fsqd1ese1d5q1vfi1gh11v3o1g.png-367kB](http://static.zybuluo.com/zhangyy/j8fp4yxqbizgqvvzo2b3f93l/image_1eiv1fsqd1ese1d5q1vfi1gh11v3o1g.png)

![image_1eiv1hrkd11t6c42129d14osu51t.png-443.8kB](http://static.zybuluo.com/zhangyy/foogklpi3xffu7jxy3zi1jhv/image_1eiv1hrkd11t6c42129d14osu51t.png)

#### 3.4.5 关于 grafana 的 prometheus 的Datasources

![image_1eiv1uthf1gph1gg41seotkurjd2a.png-237.6kB](http://static.zybuluo.com/zhangyy/gpgw415mg0vf0n4nku2fe1ql/image_1eiv1uthf1gph1gg41seotkurjd2a.png)

![image_1eiv23tbm1g481u1c9id1hqs8jv3n.png-451kB](http://static.zybuluo.com/zhangyy/ia73zg3nmrhpaax9ro6don8e/image_1eiv23tbm1g481u1c9id1hqs8jv3n.png)



