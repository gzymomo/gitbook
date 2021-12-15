- [prometheus 监控docker](https://www.cnblogs.com/xiao987334176/p/12340743.html)



# 一、概述

cAdvisor（Container Advisor）用于收集正在运行的容器资源使用和性能信息。

**使用Prometheus监控cAdvisor**

cAdvisor将容器统计信息公开为Prometheus指标。

默认情况下，这些指标在/metrics HTTP端点下提供。

可以通过设置-prometheus_endpoint命令行标志来自定义此端点。

要使用Prometheus监控cAdvisor，只需在Prometheus中配置一个或多个作业，这些作业会在该指标端点处刮取相关的cAdvisor流程。

- 使用文档：https://github.com/google/cadvisor
- 图表模板：https://grafana.com/dashboards/193



# 二、运行cAdvisor

## 启动cAdvisor容器

运行单个cAdvisor来监控整个Docker主机，被监控端安装完Docker后，添加启动cAdvisor容器

```bash
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --restart=always \
  google/cadvisor:latest
```

 

## 配置Promethus

修改配置文件prometheus.yml，最后一行添加

```yaml
  - job_name: 'docker'
    static_configs:
    - targets: ['192.168.31.138:8080']
      labels:
        instance: docker测试
```

修改配置文件后，重启prometheus

 

访问prometheus targets，确保是up状态

![img](https://img2018.cnblogs.com/i-beta/1341090/202002/1341090-20200221113506838-1145919374.png)

 

 

# 三、Granfana 导入 Docker 监控图表

推荐图标ID：https://grafana.com/dashboards/193

 ![img](https://img2018.cnblogs.com/common/1341090/202002/1341090-20200221113655293-1007622749.png)

 

输入导入图标ID等待3秒弹出如下，修改后保存

![img](https://img2018.cnblogs.com/common/1341090/202002/1341090-20200221113727380-769483957.png)

 

查看图标监控仪表盘

![img](https://img2018.cnblogs.com/common/1341090/202002/1341090-20200221113755218-1214796204.png)

 

但是这个模板，无法选择根据主机选择。推荐另外一个模板，它是可以选择主机的。

https://grafana.com/grafana/dashboards/10566



本文参考链接：

https://www.cnblogs.com/xiangsikai/p/11289518.html