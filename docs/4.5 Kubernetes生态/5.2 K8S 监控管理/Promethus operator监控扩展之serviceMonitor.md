serviceMonitor 是通过对service 获取数据的一种方式。

1. promethus-operator可以通过serviceMonitor 自动识别带有某些 label 的service ，并从这些service 获取数据。
2. serviceMonitor 也是由promethus-operator 自动发现的。



# 一、部署 serviceMonitor

- [promethus operator监控扩展之serviceMonitor](https://www.cnblogs.com/pythonPath/p/12505457.html)

serviceMonitor 的 labbels 非常的关键，promethus 通过标签发现serviceMonitor。
 执行

```bash
kubectl get prometheus ack-prometheus-operator-prometheus  -n monitoring -o yaml
```

找到以下配置信息

```yaml
serviceMonitorSelector:
    matchLabels:
      release: ack-prometheus-operator
```

release: ack-prometheus-operator  就是serviceMonitor 需要配置的label

serviceMonitor.yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor 
metadata:
  labels:
    app: ack-prometheus-operator-mysql-exporter
    heritage: Tiller
    release: ack-prometheus-operator   # prometheus 通过该label 来发现该serviceMonitor
  name: ack-prometheus-operator-mysql-exporter
  namespace: monitoring
spec:
  jobLabel: RDS-exporter
  selector:
    matchLabels:
      app: prometheus-mysql-exporter   # 该serviceMonitor 通过标签选择器 来自动发现exporter 的sevice
  namespaceSelector:
    matchNames:
    - monitoring
  endpoints:
  - port: mysql-exporter     # service 端口
    interval: 30s 
    honorLabels: true
```

创建资源
 serviceMonitor.yaml

```bash
kubectl apply -f serviceMonitor.yaml
```



# 二、Prometheus-operator自定义监控ServiceMonitor

- [Prometheus-operator自定义监控ServiceMonitor](https://www.cnblogs.com/zhangb8042/p/13091807.html)

## 2.1 创建一个用于监控的测试项目

```yaml
[root@master monitor]# cat ServiceMonitor_test_dep.yaml
kind: Service
apiVersion: v1
metadata:
  name: example-app
  labels:
    app: example-app
spec:
  selector:
    app: example-app
  ports:
  - name: web
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-app
        image: nginx:alpine
        ports:
        - name: web
          containerPort: 80
```

## 2.2 查看

```bash
[root@master monitor]# kubectl get ep -l app=example-app
NAME          ENDPOINTS           AGE
example-app   10.244.167.179:80   60m
[root@master monitor]# curl  10.244.167.179:80 -I
HTTP/1.1 200 OK
Server: nginx/1.17.10
Date: Thu, 11 Jun 2020 02:31:14 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 14 Apr 2020 14:46:22 GMT
Connection: keep-alive
ETag: "5e95ccbe-264"
Accept-Ranges: bytes
```

## 2.3 创建ServiceMonitor

```yaml
[root@master monitor]# cat   ServiceMonitor_test.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: monitor-example-app
  namespace: default
  labels:
    release: mypro  #Prometheus所选择的标签
spec:
  namespaceSelector: #监控的pod所在名称空间
    matchNames:
    - default
  selector:  #选择监控endpoint的标签
    matchLabels:
      app: example-app
  endpoints:
  - port: web #service中对应的端口名称
```

## 2.4 浏览器查看prometheus的Targets监控

![](https://img2020.cnblogs.com/blog/1242171/202006/1242171-20200611103913403-1744996845.png)



# 三、使用ServiceMonitor自定义暴露指标

- [使用ServiceMonitor自定义暴露指标](https://blog.csdn.net/dgsfor/article/details/109987704)

## 3.1 创建ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: monitor-testtalus
  namespace: lb6
  labels:
    release: testtalus  #Prometheus所选择的标签
    release: eve-prometheus-operator # 这个必须，是prometheus发现你这个规则的标签，我怎么知道是这个规则呢？ 查看最后的拓展
spec:
  namespaceSelector: #监控的pod所在名称空间
    matchNames:
    - lb6
  selector:  #选择监控endpoint的标签
    matchLabels:
      smsvc: testtalus # 这个是刚刚svc定义的标签
  endpoints:
  - port: testtalus-port #service中对应的端口名称
    path: /talus/metrics/prometheus # service对应的路径
```



# 四、kubernetes监控终极方案-kube-promethues

- [kubernetes监控终极方案-kube-promethues](https://www.cnblogs.com/skyflask/p/11480988.html)

## 4.1 自定义监控项

除了 Kubernetes 集群中的一些资源对象、节点以及组件需要监控，有的时候我们可能还需要根据实际的业务需求去添加自定义的监控项，添加一个自定义监控的步骤也是非常简单的。

- 第一步建立一个 ServiceMonitor 对象，用于 Prometheus 添加监控项
- 第二步为 ServiceMonitor 对象关联 metrics 数据接口的一个 Service 对象
- 第三步确保 Service 对象可以正确获取到 metrics 数据

比如，在我这套环境中，业务那边在测试微服务，启动了2个pod。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163519360-903106767.png)

 同时他开放了7000端口作为数据接口：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163445393-166643844.png)

 

新建serviceMonitor文件:

cat prometheus-serviceMonitorJx3recipe.yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: jx3recipe
  namespace: monitoring
  labels:
    k8s-app: jx3recipe
spec:
  jobLabel: k8s-app
  endpoints:
  - port: port
    interval: 30s
    scheme: http
  selector:
    matchLabels:
      run: jx3recipe
  namespaceSelector:
    matchNames:
    - default
```

　　

新建service文件

cat prometheus-jx3recipeService.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jx3recipe
  namespace: default
  labels:
    run: jx3recipe
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: port
    port: 7000
    protocol: TCP
 
---
apiVersion: v1
kind: Endpoints
metadata:
  name: jx3recipe
  namespace: default
  labels:
    run: jx3recipe
subsets:
- addresses:
  - ip: 10.8.0.19
    nodeName: jx3recipe-01
  - ip: 10.8.2.17
    nodeName: jx3recipe-02
  ports:
  - name: port
    port: 7000
    protocol: TCP
```

　　**这里有个问题，我把IP写死了，可能容器重启后，IP地址改变的话，就不行了，所以这里有点问题。**

```bash
kubectl apply -f  prometheus-jx3recipeService.yaml
kubectl apply -f  prometheus-serviceMonitorJx3recipe.yaml
```

　　

在promethues的targets里面可以看到效果：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163946095-894043990.png)

 

 

随便挑个item查看数据：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907164045516-1947960201.png)

 一切OK，后续我们就可以使用grafana定制dashboard了。



# 五、Prometheus监控神技--自动发现配置

- [Prometheus监控神技--自动发现配置](https://www.cnblogs.com/skyflask/p/11498834.html)









# 问题记录

## [部署 ServiceMonitor 之后如何让 Prometheus 立即发现](https://q.cnblogs.com/q/125439/)

添加或删除 ServiceMonitor 后，向 prometheus 发一个 post 请求进行 reload ，然后等30秒就刷新了。

```bash
curl -X POST -v "http://10.0.1.21:30090/-/reload"
```

