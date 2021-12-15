- [Prometheus监控Minio集群](https://www.cnblogs.com/xiao987334176/p/13259257.html)



# 一、概述

Minio支持集成prometheus，用以监控CPU、硬盘、网络等数据。

# 二、修改docker-compose.yaml

官方的给docker-compose.yaml，默认是不能访问metric数据的。

这里配置用是"public"类型，无身份认证。

需要在docker-compose.yaml中，增加一个环境变量即可。

```
MINIO_PROMETHEUS_AUTH_TYPE: public
```

还没有配置主机目录映射，因此，这里就一并修改了，完整内容如下：

```yaml
version: '3.7'

# starts 4 docker containers running minio server instances. Each
# minio server's web interface will be accessible on the host at port
# 9001 through 9004.
services:
  minio1:
    image: minio/minio:RELEASE.2020-07-02T00-15-09Z
    volumes:
      - /data/minio-cluster/minio1/data1:/data1
      - /data/minio-cluster/minio1/data2:/data2
    ports:
      - "9001:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_PROMETHEUS_AUTH_TYPE: public
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio2:
    image: minio/minio:RELEASE.2020-07-02T00-15-09Z
    volumes:
      - /data/minio-cluster/minio2/data1:/data1
      - /data/minio-cluster/minio2/data2:/data2
    ports:
      - "9002:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_PROMETHEUS_AUTH_TYPE: public
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio3:
    image: minio/minio:RELEASE.2020-07-02T00-15-09Z
    volumes:
      - /data/minio-cluster/minio3/data1:/data1
      - /data/minio-cluster/minio3/data2:/data2
    ports:
      - "9003:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_PROMETHEUS_AUTH_TYPE: public
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio4:
    image: minio/minio:RELEASE.2020-07-02T00-15-09Z
    volumes:
      - /data/minio-cluster/minio4/data1:/data1
      - /data/minio-cluster/minio4/data2:/data2
    ports:
      - "9004:9000"
    environment:
      MINIO_ACCESS_KEY: minio
      MINIO_SECRET_KEY: minio123
      MINIO_PROMETHEUS_AUTH_TYPE: public
    command: server http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

创建主机目录

```
mkdir -p /data/minio-cluster/minio{1,2,3,4}/data{1,2}
```

 

启动docker-compose

```
docker-compose up -d
```

# 三、访问metric

```
http://192.168.31.34:9001/minio/prometheus/metrics
```

效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707100645376-383529363.png)

将端口改为9002~9004，也是同样的结果。

# 四、Prometheus配置

修改prometheus.yml，增加job_name

```yaml
  - job_name: minio
    metrics_path: /minio/prometheus/metrics
    scrape_interval: 10s
    scheme: http
    static_configs:
      - targets: ['192.168.31.34:9001','192.168.31.34:9002','192.168.31.34:9003','192.168.31.34:9004']
```

修改完成后，重启prometheus

 

访问targets，确保都是UP状态

# 五、Grafana导入模板

## 模板选择

推荐使用模板：https://grafana.com/grafana/dashboards/12063

这个模板执行选择Minio节点，而且还是中文显示的。

导入模板后，效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707101455166-1462234778.png)

但是发现关于s3相关图表，数据是空的。

**需要修改图表中的metrics计算公式才行。**

先来看S3接口总请求，对应的metrics计算公式为：

```
sum(s3_requests_total{instance="172.16.62.150:9000",job="minio-metrics"}) by (api)
```

它需要key为s3_requests_total的值。

 

我们再去这几个metrics中去查找

```
http://192.168.31.34:9001/minio/prometheus/metrics
http://192.168.31.34:9002/minio/prometheus/metrics
http://192.168.31.34:9003/minio/prometheus/metrics
http://192.168.31.34:9004/minio/prometheus/metrics
```

 

发现只有第一个才有s3_requests_total的值。

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707131207907-875410709.png)

 

因此metrics的计算公式为：

```
sum(s3_requests_total{api="listobjectsv1"}) by (api)
```

修改完成之后，图表数据就有了。

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707134329293-1447220990.png)

 

附上其他图表的正确计算公式：

```
S3接口当前总请求数
sum(s3_requests_current{api="listobjectsv1"}) by (api)

S3接口总错误请求数
sum(s3_errors_total{api="listobjectsv1",job="$job"}) by (api)

sum(s3_requests_current{api="listobjectsv1",job="$job"}) by (api)

S3接口延迟统计
s3_ttfb_seconds_sum{api="listobjectsv1"}
```

 

对于S3接口总错误请求数，需要修改一下Legend，做一下标识区分。

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707134848797-1305342582.png)

 

最终总体效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202007/1341090-20200707135230980-179388741.png)

 

本文参考链接：

https://www.cnblogs.com/rongfengliang/p/12017914.html

https://blog.csdn.net/kuang1144/article/details/105302960