[TOC]

- [CSDN：Moshow郑锴：基于ELK打造强大的日志收集分析系统（springboot2+logback+logstash+elasticsearch+kibana）](https://blog.csdn.net/moshowgame/article/details/98851656)

- [ElasticSearch实战系列九: ELK日志系统介绍和安装](https://www.cnblogs.com/xuwujing/p/13870806.html)





springboot2+logback+logstash+elasticsearch的日志分析系统，借助es强大的生态圈以及全文搜索能力，实现日志收集/分析/检索不再是难事。

日志收集分为两种情况：

- logback直接输出到logstash，通过Tcp/Socket等传输（网络开销），当量太大时Logstash会做一个缓存队列处理，减少了filebeat去读取抓取日志的步骤。
- logback输出日志到日志文件，filebeat实时抓取日志文件（性能开销），再传输到logstash。

第二种情况的旧版flume方案就存在很大的缺点：在日志的产生端LogServer服务器重，部署FlumeAgent，然后实时监控产生的日志，再发送至Kafka。每一个FlumeAgent都占用了较大的系统资源，有时候LogServer性能开销大，CPU资源尤其紧张，所以实时收集分析日志，就必须交给一个更轻量级、占用资源更少的日志收集框架，例如filebeat。

# 1、什么是ELK？
![](https://img-blog.csdnimg.cn/20190812140738488.png)

从功能上看，ElasticSearch负责数据的存储和检索，Kibana提供图形界面便于管理，Logstash是个日志中转站负责给ElasticSearch输出信息。

![](https://img-blog.csdnimg.cn/2019081019103940.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly96aGVuZ2thaS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70)

从流程上看，Logstash收集AppServer产生的Log，并存放到ElasticSearch集群中，而Kibana则从ES集群中查询数据生成图表，再返回给Browser。



