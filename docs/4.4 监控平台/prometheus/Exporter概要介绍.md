[TOC]

CSDN：liumiaocn：[Prometheus：监控与告警：6: Exporter概要介绍](https://blog.csdn.net/liumiaocn/article/details/103957805)
# 1、exporter是什么？
![](https://www.showdoc.cc/server/api/common/visitfile/sign/bdc914fafe1f79841838c6efa14fc9b6?showdoc=.jpg)

为Prometheus提供监控数据源的应用都可以被成为Exporter，比如Node Exporter则用来提供节点相关的资源使用状况，而Prometheus从这些不同的Exporter中获取监控数据，然后可以在诸如Grafana这样的可视化工具中进行结果的显示。

# 2、社区场景的exporter
## 2.1 数据库
 - MongoDB exporter
 - MSSQL server exporter
 - MySQL server exporter (official)
 - OpenTSDB Exporter
 - Oracle DB Exporter
 - PostgreSQL exporter
 - Redis exporter
 - ElasticSearch exporter
 - RethinkDB exporter
 - Consul exporter (official)

## 2.2 消息队列
 - Kafka exporter
 - IBM MQ exporter
 - RabbitMQ exporter
 - RocketMQ exporter
 - NSQ exporter
 - Gearman exporter

## 2.3 存储
 - Ceph exporter
 - Gluster exporter
 - Hadoop HDFS FSImage exporter

## 2.4 硬件相关
 - Node/system metrics exporter (official)
 - Dell Hardware OMSA exporter
 - IoT Edison exporter
 - IBM Z HMC exporter
 - NVIDIA GPU exporter

## 2.5 问题追踪与持续集成
 - Bamboo exporter
 - Bitbucket exporter
 - Confluence exporter
 - Jenkins exporter
 - JIRA exporter

## 2.6 HTTP服务
 - Apache exporter
 - HAProxy exporter (official)
 - Nginx metric library
 - Nginx VTS exporter
 - Passenger exporter
 - Squid exporter
 - Tinyproxy exporter
 - Varnish exporter
 - WebDriver exporter

## 2.7 API服务
 - AWS ECS exporter
 - AWS Health exporter
 - AWS SQS exporter
 - Cloudflare exporter
 - DigitalOcean exporter
 - Docker Cloud exporter
 - Docker Hub exporter
 - GitHub exporter
 - InstaClustr exporter
 - Mozilla Observatory exporter
 - OpenWeatherMap exporter
 - Pagespeed exporter
 - Rancher exporter
 - Speedtest exporter
 - Tankerkönig API Exporter

## 2.8 日志
 - Fluentd exporter
 - Google’s mtail log data extractor
 - Grok exporter

## 2.9 监控系统
 - Akamai Cloudmonitor exporter
 - Alibaba Cloudmonitor exporter
 - AWS CloudWatch exporter (official)
 - Azure Monitor exporter
 - Cloud Foundry Firehose exporter
 - Collectd exporter (official)
 - Google Stackdriver exporter
 - Graphite exporter (official)
 - Huawei Cloudeye exporter
 - InfluxDB exporter (official)
 - JavaMelody exporter
 - JMX exporter (official)
 - Nagios / Naemon exporter
 - Sensu exporter
 - SNMP exporter (official)
 - TencentCloud monitor exporter
 - ThousandEyes exporter

## 2.10 其他
 - BIND exporter
 - Bitcoind exporter
 - cAdvisor
 - Dnsmasq exporter
 - Ethereum Client exporter
 - JFrog Artifactory Exporter
 - JMeter plugin
 - Kibana Exporter
 - kube-state-metrics
 - OpenStack exporter
 - PowerDNS exporter
 - Script exporter
 - SMTP/Maildir MDA blackbox prober
 - WireGuard exporter
 - Xen exporter

# 3、使用方式
Prometheus Server提供PromQL查询语言能力、负责数据的采集和存储等主要功能，而数据的采集主要通过周期性的从Exporter所暴露出来的HTTP服务地址（一般是/metrics）来获取监控数据。而Exporter在实际运行的时候根据其支持的方式也会分为：
 - 独立运行的Exporter应用，通过HTTP服务地址提供相应的监控数据（比如Node Exporter）
 - 内置在监控目标中，通过HTTP服务地址提供相应的监控数据（比如kubernetes）