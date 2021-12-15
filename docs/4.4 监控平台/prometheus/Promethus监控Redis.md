- [promethus监控Redis](https://www.cnblogs.com/xiao987334176/p/12101496.html)
- [Prometheus 监控 Redis 集群的正确姿势](https://www.cnblogs.com/fsckzy/p/12053604.html)



# 一、概述

Prometheus exporter for Redis metrics.

github地址：

https://github.com/oliver006/redis_exporter

# 二、安装redis_exporter

下载最新版本：

https://github.com/oliver006/redis_exporter/releases/download/v1.3.5/redis_exporter-v1.3.5.linux-amd64.tar.gz

 

登录到redis服务器，解压安装

```
tar zxvf redis_exporter-v1.3.5.linux-amd64.tar.gz -C /data
mv /data/redis_exporter-v1.3.5.linux-amd64 /data/redis_exporter
```

 

redis_exporter 用法

解压后只有一个二进制程序就叫 redis_exporter 通过 -h 可以获取到帮助信息，下面列出一些常用的选项：

```
-redis.addr：指明一个或多个 Redis 节点的地址，多个节点使用逗号分隔，默认为 redis://localhost:6379
-redis.password：验证 Redis 时使用的密码；
-redis.file：包含一个或多个redis 节点的文件路径，每行一个节点，此选项与 -redis.addr 互斥。
-web.listen-address：监听的地址和端口，默认为 0.0.0.0:9121
```

 

运行 redis_exporter 服务

```
## 无密码
nohup ./redis_exporter redis//192.168.111.11:6379 &
## 有密码
nohup ./redis_exporter  -redis.addr 192.168.111.11:6379  -redis.password 123456 &
```

 

# 三、配置 prometheus.yml

## 单机版

添加监控目标

```
vim /data/prometheus/prometheus.yml
```

最后一行添加

```
  - job_name: 'redis_exporter'
    static_configs:
    - targets: ['192.168.10.147:9121']
      labels:
        instance: 生产实例1
    - targets: ['192.168.10.148:9121']
      labels:
        instance: 生产实例2
    - targets: ['192.168.10.149:9121']
      labels:
        instance: 生产实例3
    - targets: ['192.168.10.150:9121']
      labels:
        instance: 生产实例4
    - targets: ['192.168.10.151:9121']
      labels:
        instance: 生产实例5
    - targets: ['192.168.10.152:9121']
      labels:
        instance: 生产实例6
```

 

## 集群版

运行 redis_exporter 服务，只需要连接其中一个节点即可。

```
## 无密码
nohup ./redis_exporter redis//192.168.111.11:7000 &
## 有密码
nohup ./redis_exporter  -redis.addr 192.168.111.11:7000  -redis.password 123456 &
```

 最后一行添加

```
  - job_name: 'redis_cluster'
    static_configs:
      - targets:
        - redis://192.168.111.11:7000
        - redis://192.168.111.11:7001
        - redis://192.168.111.11:7002
        - redis://192.168.111.11:7003
        - redis://192.168.111.11:7004
        - redis://192.168.111.11:7005
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.111.11:9121
```

 

重启prometheus即可。

 

# 四、配置 Grafana 的模板

redis_exporter 在 Grafana 上为我们提供好了 Dashboard 模板：[https://grafana.com/dashboards/763](http://www.eryajf.net/go?url=https://grafana.com/dashboards/763)

下载后在 Grafana 中导入 json 模板就可以看到官方这样的示例截图啦：

![img](https://img2018.cnblogs.com/i-beta/1341090/201912/1341090-20191226142608846-137569674.png)

 

注意：Memory Usage这个图表，一直是N/A。是因为redis_memory_max_bytes 获取的值为0

导致 redis_memory_used_bytes / redis_memory_max_bytes 结果不正常。

 

解决办法：将redis_memory_max_bytes 改为服务器的真实内存大小。

所以我更改计算公式

```
redis_memory_used_bytes{instance=~"$instance"}  / 8193428
```

 

本文参考链接：

- http://www.eryajf.net/2497.html

- https://www.cnblogs.com/fsckzy/p/12053604.html

