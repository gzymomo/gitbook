- [Prometheus分布式监控](https://www.cnblogs.com/xiao987334176/p/12836472.html)



# 一、概述

prometheus安装在阿里云上面，监控节点在公司内部机房，2个网络直接是不互通的。

## 环境说明

阿里云服务器：

操作系统：centos 7.6

数量：1台

 

公司内部服务器

操作系统：centos 7.6

数量：1台

 

## 拓扑图

![img](https://img2020.cnblogs.com/blog/1341090/202005/1341090-20200507145519126-1420677056.png)

 

 说明：

\1. 公司内部服务器安装node-exporter插件，收集主机信息，通过调用curl命令，将收集的数据以POST方式发送给Pushgateway

\2. Pushgateway负责接收数据

\3. Prometheus从Pushgateway中拉取数据，结合Grafana做数据展示。

 

# 二、部署操作

## 阿里云服务器

Prometheus和Pushgateway，是直接docker部署的。具体安装操作，请参考链接：

https://www.cnblogs.com/xiao987334176/p/9930517.html

https://www.cnblogs.com/xiao987334176/p/9933963.html

 

这里重点要说明的是Prometheus配置Pushgateway时，必须要加一个参数honor_labels: true

```
  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['172.18.156.172:9091']
        labels:
          instance: xxx
```

如果不加，会造成推送给Pushgateway的instance和job全部加了"exported_"前缀。

那么在Grafana现有的模板中，无法展示推送到Pushgateway的数据。

 

## 公司内部服务器

### 安装node-exporter

这里推荐使用二进制方式

```
wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
tar zxvf node_exporter-0.18.1.linux-amd64.tar.gz -C /data/
mv /data/node_exporter-0.18.1.linux-amd64 /data/node_exporter
```

 

封装service

```
vi /etc/systemd/system/node-exporter.service
```

内容如下：

```
[Unit]
Description=Prometheus Node Exporter
After=network.target
[Service]
ExecStart=/data/node_exporter/node_exporter
[Install]
WantedBy=multi-user.target
```

注意：主要修改ExecStart和User

 

设置开机自启动

```
systemctl daemon-reload
systemctl enable node-exporter
systemctl start node-exporter
```

 

查看端口

```
# ss -tunlp|grep node
tcp    LISTEN     0      128      :::9100                 :::*                   users:(("node_exporter",pid=990,fd=3))
```

 

### 备注

node-exporter也可以使用docker方式安装，但是收集磁盘数据时，它采集的数据和真实物理主机是不一样的。

比如执行：df -hT

![img](https://img2020.cnblogs.com/blog/1341090/202005/1341090-20200507151953244-1539449765.png)

 

可能会比真实的多几个目录。

因此，我还是推荐使用二进制方式安装，这样数据会比较准确一点。

 

## 发送POST请求

将node_exporter收集到的数据传送监控数据到pushgateway

对于传过去的监控项会添加此处定义的标签 job=node_exporter instance=北京三里屯 hostname=192.168.2.45

```
curl 127.0.0.1:9100/metrics|curl --data-binary @- http://114.114.114.114:9091/metrics/job/node_exporter/instance/北京三里屯/hostname/192.168.2.45
```

 

访问pushgateway页面

```
http://114.114.114.114:9091
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202005/1341090-20200507151418548-1709168149.png)

 

 

 访问Grafana，查看主机信息

![img](https://img2020.cnblogs.com/blog/1341090/202005/1341090-20200507151658187-151037497.png)



本文参考链接：

https://www.cnblogs.com/huandada/p/10932953.html

https://www.jianshu.com/p/51b9338d98b0