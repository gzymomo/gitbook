[kubernetes监控终极方案-kube-promethues](https://www.cnblogs.com/skyflask/p/11480988.html)

项目地址为：[https://github.com/coreos/kube-prometheus](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcoreos%2Fkube-prometheus)

*![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907092904269-1984828303.png)*

这个仓库包括：kubernetes清单、granfana dashboard以及promethues rules。同时还包括容易上手的安装脚本。

组件包括：

- The [Prometheus Operator](https://github.com/coreos/prometheus-operator)
- 高可用[Prometheus](https://prometheus.io/)
- 高可用[Alertmanager](https://github.com/prometheus/alertmanager)
- [Prometheus node-exporter](https://github.com/prometheus/node_exporter)
- [Prometheus Adapter for Kubernetes Metrics APIs](https://github.com/DirectXMan12/k8s-prometheus-adapter)
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
- [Grafana](https://grafana.com/)



## kube-promethues架构

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907093555092-1273384033.png)

上图是`Prometheus-Operator`官方提供的架构图，其中`Operator`是最核心的部分，作为一个控制器，他会去创建`Prometheus`、`ServiceMonitor`、`AlertManager`以及`PrometheusRule`4个`CRD`资源对象，然后会一直监控并维持这4个资源对象的状态。

其中创建的`prometheus`这种资源对象就是作为`Prometheus Server`存在，而`ServiceMonitor`就是`exporter`的各种抽象，`exporter`前面我们已经学习了，是用来提供专门提供`metrics`数据接口的工具，`Prometheus`就是通过`ServiceMonitor`提供的`metrics`数据接口去 pull 数据的，当然`alertmanager`这种资源对象就是对应的`AlertManager`的抽象，而`PrometheusRule`是用来被`Prometheus`实例使用的报警规则文件。

这样我们要在集群中监控什么数据，就变成了直接去操作 Kubernetes 集群的资源对象了，是不是方便很多了。上图中的 Service 和 ServiceMonitor 都是  Kubernetes 的资源，一个 ServiceMonitor 可以通过 labelSelector 的方式去匹配一类  Service，Prometheus 也可以通过 labelSelector 去匹配多个ServiceMonitor。



## kube-promethues部署

下载安装源码

```
git clone https:``//github.com/coreos/kube-prometheus.git
```

　　安装文件都在kube-prometheus/manifests/ 目录下。

官方把所有文件都放在一起,这里我复制了然后分类下

```
mkdir prometheus``cp kube-prometheus/manifests/* prometheus/``cd prometheus/``mkdir -p ``operator` `node-exporter alertmanager grafana kube-state-metrics prometheus serviceMonitor adapter``mv *-serviceMonitor* serviceMonitor/``mv 0prometheus-``operator``* ``operator``/``mv grafana-* grafana/``mv kube-state-metrics-* kube-state-metrics/``mv alertmanager-* alertmanager/``mv node-exporter-* node-exporter/``mv prometheus-adapter* adapter/``mv prometheus-* prometheus/
```

　　注意：新版本的默认label变了，需要修改选择器为`beta.kubernetes.io/os，不然安装的时候会卡住。`

修改选择器

```
sed -ri ``'/linux/s#kubernetes.io#beta.&#'` `\``  ``alertmanager/alertmanager-alertmanager.yaml \``  ``prometheus/prometheus-prometheus.yaml \``  ``node-exporter/node-exporter-daemonset.yaml \``  ``kube-state-metrics/kube-state-metrics-deployment.yaml
```

 

镜像使用dockerhub上的

```
sed -ri ``'/quay.io/s#quay.io/prometheus#prom#'` `\`` ``alertmanager/alertmanager-alertmanager.yaml \`` ``prometheus/prometheus-prometheus.yaml \`` ``node-exporter/node-exporter-daemonset.yaml
```

　　

使用能拉取到的谷歌镜像

```
find -type f -exec sed -ri ``'s#k8s.gcr.io#gcr.azk8s.cn/google_containers#'` `{} \; 
```

　　

当前文件目录：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907151056439-1714099461.png)

 

 

1、生成namespace

```
kubectl apply -f .
```

　

2、安装operater

```
kubectl apply -f ``operator``/
```

　　

3、依次安装其他组件

```
kubectl apply -f adapter/``kubectl apply -f alertmanager/``kubectl apply -f node-exporter/``kubectl apply -f kube-state-metrics/``kubectl apply -f grafana/``kubectl apply -f prometheus/``kubectl apply -f serviceMonitor/
```

　　

4、查看整体状态

```
kubectl -n monitoring ``get` `all
```

　　

```
[root@vm10-0-0-12 prometheus]# kubectl -n monitoring ``get` `all``NAME                    READY  STATUS  RESTARTS  AGE``pod/alertmanager-main-0          2/2   Running  0     27h``pod/alertmanager-main-1          2/2   Running  0     27h``pod/alertmanager-main-2          2/2   Running  0     27h``pod/grafana-7b86fd9ffd-sslwf        1/1   Running  0     27h``pod/kube-state-metrics-688965c565-knjbz  4/4   Running  0     27h``pod/node-exporter-4vtgl          2/2   Running  0     27h``pod/node-exporter-5bnfw          2/2   Running  0     27h``pod/node-exporter-9nnsp          2/2   Running  0     27h``pod/node-exporter-fd8ng          2/2   Running  0     27h``pod/node-exporter-nh5q9          2/2   Running  0     27h``pod/node-exporter-z69fb          2/2   Running  0     27h``pod/prometheus-adapter-66fc7797fd-qpt4d  1/1   Running  0     27h``pod/prometheus-k8s-0            3/3   Running  1     27h``pod/prometheus-k8s-1            3/3   Running  1     27h``pod/prometheus-``operator``-78678c7494-6954s  1/1   Running  0     27h` `NAME              TYPE      CLUSTER-IP    EXTERNAL-IP   PORT(S)           AGE``service/alertmanager-main    ClusterIP   10.254.160.2   <none>      9093/TCP           27h``service/alertmanager-operated  ClusterIP   None       <none>      9093/TCP,9094/TCP,9094/UDP  27h``service/grafana         LoadBalancer  10.254.2.98   120.92.212.201  3000:31423/TCP        27h``service/kube-state-metrics   ClusterIP   None       <none>      8443/TCP,9443/TCP      27h``service/node-exporter      ClusterIP   None       <none>      9100/TCP           27h``service/prometheus-adapter   ClusterIP   10.254.225.221  <none>      443/TCP           27h``service/prometheus-k8s     LoadBalancer  10.254.23.154  120.92.92.56   9090:32361/TCP        27h``service/prometheus-operated   ClusterIP   None       <none>      9090/TCP           27h``service/prometheus-``operator`   `ClusterIP   None       <none>      8080/TCP           27h` `NAME              DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR         AGE``daemonset.apps/node-exporter  6     6     6    6      6      beta.kubernetes.io/os=linux  27h` `NAME                 READY  UP-TO-DATE  AVAILABLE  AGE``deployment.apps/grafana        1/1   1      1      27h``deployment.apps/kube-state-metrics  1/1   1      1      27h``deployment.apps/prometheus-adapter  1/1   1      1      27h``deployment.apps/prometheus-``operator`  `1/1   1      1      27h` `NAME                       DESIRED  CURRENT  READY  AGE``replicaset.apps/grafana-7b86fd9ffd        1     1     1    27h``replicaset.apps/kube-state-metrics-688965c565  1     1     1    27h``replicaset.apps/kube-state-metrics-758f8b9855  0     0     0    27h``replicaset.apps/prometheus-adapter-66fc7797fd  1     1     1    27h``replicaset.apps/prometheus-``operator``-78678c7494  1     1     1    27h` `NAME                 READY  AGE``statefulset.apps/alertmanager-main  3/3   27h``statefulset.apps/prometheus-k8s   2/2   27h
```

　　

 



## kube-promethues配置

1、kube-controller-manager 和 kube-scheduler组件配置

我们可以看到大部分的配置都是正常的，只有两三个没有管理到对应的监控目标，比如 kube-controller-manager 和 kube-scheduler 这两个系统组件，这就和 ServiceMonitor  的定义有关系了，我们先来查看下 kube-scheduler 组件对应的 ServiceMonitor  资源的定义：(prometheus-serviceMonitorKubeScheduler.yaml)

 

```
apiVersion: monitoring.coreos.com/v1``kind: ServiceMonitor``metadata:`` ``labels:``  ``k8s-app: kube-scheduler`` ``name: kube-scheduler`` ``namespace``: monitoring``spec:`` ``endpoints:`` ``- interval: 30s # 每30s获取一次信息``  ``port: http-metrics # 对应service的端口名`` ``jobLabel: k8s-app`` ``namespaceSelector: # 表示去匹配某一命名空间中的service，如果想从所有的``namespace``中匹配用any: ``true``  ``matchNames:``  ``- kube-system`` ``selector: # 匹配的 Service 的labels，如果使用mathLabels，则下面的所有标签都匹配时才会匹配该service，如果使用matchExpressions，则至少匹配一个标签的service都会被选择``  ``matchLabels:``   ``k8s-app: kube-scheduler
```

　　上面是一个典型的 ServiceMonitor 资源文件的声明方式，上面我们通过`selector.matchLabels`在 kube-system 这个命名空间下面匹配具有`k8s-app=kube-scheduler`这样的 Service，但是我们系统中根本就没有对应的 Service，所以我们需要手动创建一个 Service：（prometheus-kubeSchedulerService.yaml）

cat prometheus-kubeSchedulerService.yaml

```
apiVersion: v1``kind: Service``metadata:`` ``namespace``: kube-system`` ``name: kube-scheduler`` ``labels:``  ``k8s-app: kube-scheduler``spec:`` ``type: ClusterIP`` ``clusterIP: None`` ``ports:`` ``- name: port``  ``port: 10251``  ``protocol: TCP``---``apiVersion: v1``kind: Endpoints``metadata:`` ``labels:``  ``k8s-app: kube-scheduler`` ``name: kube-scheduler`` ``namespace``: kube-system``subsets:``- addresses:`` ``- ip: 10.0.0.5`` ``- ip: 10.0.0.15`` ``- ip: 10.0.0.20`` ``ports:`` ``- name: http-metrics``  ``port: 10251``  ``protocol: TCP
```

　　

　　同理prometheus-kubeControllerManagerService.yaml也需要修改一下：

```
apiVersion: v1``kind: Service``metadata:`` ``namespace``: kube-system`` ``name: kube-controller-manager`` ``labels:``  ``k8s-app: kube-controller-manager``spec:`` ``selector:``  ``component: kube-controller-manager`` ``type: ClusterIP`` ``clusterIP: None`` ``ports:`` ``- name: http-metrics``  ``port: 10252``  ``targetPort: 10252``  ``protocol: TCP` `---``apiVersion: v1``kind: Endpoints``metadata:`` ``labels:``  ``k8s-app: kube-controller-manager`` ``name: kube-controller-manager`` ``namespace``: kube-system``subsets:``- addresses:`` ``- ip: 10.0.0.5`` ``- ip: 10.0.0.15`` ``- ip: 10.0.0.20`` ``ports:`` ``- name: http-metrics``  ``port: 10252``  ``protocol: TCP
```

　　

 2、coredns配置

我在测试环境和线上环境都遇到了coredns无法发现的问题。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907161740355-1677813924.png)

 

 

 出现这个问题的原因在于/data/monitor/kube-prometheus/manifests/prometheus-serviceMonitorCoreDNS.yaml这个文件：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907161932547-1076268584.png)

 

 

 上面的matchLabels为kube-dns，而实际上不是这个label：

```
pod/coredns-58d6869b44-ddczz           1/1   Running  0     4d4h  10.8.5.2  10.0.0.5   <none>      <none>``pod/coredns-58d6869b44-zrkx4           1/1   Running  0     4d4h  10.8.4.2  10.0.0.15   <none>      <none>` `service/coredns          ClusterIP  10.254.0.10   <none>     53/UDP,53/TCP,9153/TCP  4d4h  k8s-app=coredns
```

　　实际上是coredns，所以需要修改。

同时，rules里面也需要修改。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907162956379-1455895187.png)

 

 

 ![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163029358-1478768185.png)

 

 

 修改完成后：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163105890-1937097340.png)

 

 

 

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163126378-320218009.png)

 

 

 恢复正常。

 

3、自定义监控项

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

```
apiVersion: monitoring.coreos.com/v1``kind: ServiceMonitor``metadata:`` ``name: jx3recipe`` ``namespace``: monitoring`` ``labels:``  ``k8s-app: jx3recipe``spec:`` ``jobLabel: k8s-app`` ``endpoints:`` ``- port: port``  ``interval: 30s``  ``scheme: http`` ``selector:``  ``matchLabels:``   ``run: jx3recipe`` ``namespaceSelector:``  ``matchNames:``  ``- ``default
```

　　

新建service文件

cat prometheus-jx3recipeService.yaml

```
apiVersion: v1``kind: Service``metadata:`` ``name: jx3recipe`` ``namespace``: ``default`` ``labels:``  ``run: jx3recipe``spec:`` ``type: ClusterIP`` ``clusterIP: None`` ``ports:`` ``- name: port``  ``port: 7000``  ``protocol: TCP` `---``apiVersion: v1``kind: Endpoints``metadata:`` ``name: jx3recipe `` ``namespace``: ``default`` ``labels:``  ``run: jx3recipe``subsets:``- addresses:`` ``- ip: 10.8.0.19``  ``nodeName: jx3recipe-01`` ``- ip: 10.8.2.17``  ``nodeName: jx3recipe-02`` ``ports:`` ``- name: port``  ``port: 7000``  ``protocol: TCP
```

　　**这里有个问题，我把EP写死了，可能容器重启后，IP地址改变的话，就不行了，所以这里有点问题。**

```
kubectl apply -f prometheus-jx3recipeService.yaml``kubectl apply -f prometheus-serviceMonitorJx3recipe.yaml
```

　　

在promethues的targets里面可以看到效果：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907163946095-894043990.png)

 

 

随便挑个item查看数据：

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907164045516-1947960201.png)

 

 

 一切OK，后续我们就可以使用grafana定制dashboard了。



## kube-promethues对外暴露

上面环境搭建完成后，服务都是对内的，我们需要对外进行暴露，由于环境是搭建在金山云上，所以直接使用公有云的LB即可。

grafana暴露

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165400353-1298079404.png)

 

 只需要把service的type改为LB。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165502359-1040100564.png)

 

 直接通过外网IP访问即可。http://120.92.*.*:3000访问grafana。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165640164-1322300985.png)

 

 

进入后可以看到已经自带了很多dashboard，比较丰富，包括cluster、node、pod以及k8s的各个组件的监控数据。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165758359-1008182064.png)

 

 

　

Node

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165826744-1724181362.png)

 

 

 promethues本身

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907165853356-431740999.png)

 

 

当然，我们还可以从grafana导入一些更加炫酷的dashboard，便于我们实时掌握资源使用情况。

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907170710367-338912633.png)

 

 相关文件已经上传到了github上，可以直接导入使用：

https://github.com/loveqx/k8s-study

![img](https://img2018.cnblogs.com/blog/462684/201909/462684-20190907170755709-1141894134.png)

 