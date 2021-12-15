Controller（StatefulSet）-部署有状态应用

# 什么是控制器

kubernetes中内建了很多controller（控制器），这些相当于一个状态机，用来控制pod的具体状态和行为。

**部分控制器类型如下：**

- ReplicationController 和 ReplicaSet
- Deployment
- DaemonSet
- StatefulSet
- Job/CronJob
- HorizontalPodAutoscaler



# StatefulSet

StatefulSet 是用来管理有状态应用的工作负载 API 对象。

StatefulSet 中的 Pod 拥有一个具有黏性的、独一无二的身份标识。这个标识基于 StatefulSet 控制器分配给每个 Pod 的唯一顺序索引。Pod 的名称的形式为<statefulset name>-<ordinal index> 。例如：web的StatefulSet 拥有两个副本，所以它创建了两个 Pod：web-0和web-1。

和 Deployment 相同的是，StatefulSet 管理了基于相同容器定义的一组 Pod。但和 Deployment 不同的是，StatefulSet 为它们的每个 Pod 维护了一个固定的 ID。这些 Pod 是基于相同的声明来创建的，但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

【使用场景】StatefulSets 对于需要满足以下一个或多个需求的应用程序很有价值：

- 稳定的、唯一的网络标识符，即Pod重新调度后其PodName和HostName不变【当然IP是会变的】
- 稳定的、持久的存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC实现
- 有序的、优雅的部署和缩放
- 有序的、自动的滚动更新

如上面，稳定意味着 Pod 调度或重调度的整个过程是有持久性的。

如果应用程序不需要任何稳定的标识符或有序的部署、删除或伸缩，则应该使用由一组无状态的副本控制器提供的工作负载来部署应用程序，比如使用 Deployment 或者 ReplicaSet 可能更适用于无状态应用部署需要。

## 限制

- 给定 Pod 的存储必须由 PersistentVolume 驱动 基于所请求的 storage class 来提供，或者由管理员预先提供。
- 删除或者收缩 StatefulSet 并不会删除它关联的存储卷。这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。
- StatefulSet 当前需要 headless 服务 来负责 Pod 的网络标识。你需要负责创建此服务。
- 当删除 StatefulSets 时，StatefulSet 不提供任何终止 Pod 的保证。为了实现 StatefulSet 中的 Pod 可以有序和优雅的终止，可以在删除之前将 StatefulSet 缩放为 0。
- 在默认 Pod 管理策略(OrderedReady) 时使用滚动更新，可能进入需要人工干预才能修复的损坏状态。

## 有序索引

对于具有 N 个副本的 StatefulSet，StatefulSet 中的每个 Pod 将被分配一个整数序号，从 0 到 N-1，该序号在 StatefulSet 上是唯一的。

StatefulSet 中的每个 Pod 根据 StatefulSet 中的名称和 Pod 的序号来派生出它的主机名。组合主机名的格式为$(StatefulSet 名称)-$(序号)。

## 部署和扩缩保证

- 对于包含 N 个 副本的 StatefulSet，当部署 Pod 时，它们是依次创建的，顺序为 0~(N-1)。
- 当删除 Pod 时，它们是逆序终止的，顺序为 (N-1)~0。
- 在将缩放操作应用到 Pod 之前，它前面的所有 Pod 必须是 Running 和 Ready 状态。
- 在 Pod 终止之前，所有的继任者必须完全关闭。

StatefulSet 不应将 pod.Spec.TerminationGracePeriodSeconds 设置为 0。这种做法是不安全的，要强烈阻止。

### 部署顺序

在下面的 nginx 示例被创建后，会按照 web-0、web-1、web-2 的顺序部署三个 Pod。在 web-0 进入 Running 和 Ready 状态前不会部署 web-1。在 web-1 进入 Running 和 Ready 状态前不会部署 web-2。

如果 web-1 已经处于 Running 和 Ready 状态，而 web-2 尚未部署，在此期间发生了 web-0 运行失败，那么 web-2 将不会被部署，要等到 web-0 部署完成并进入 Running 和 Ready 状态后，才会部署 web-2。

### 收缩顺序

如果想将示例中的 StatefulSet 收缩为 replicas=1，首先被终止的是 web-2。在 web-2 没有被完全停止和删除前，web-1 不会被终止。当 web-2 已被终止和删除；但web-1 尚未被终止，如果在此期间发生 web-0 运行失败，那么就不会终止 web-1，必须等到 web-0 进入 Running 和 Ready 状态后才会终止 web-1。



# 无状态和有状态区别

## 无状态

- 认为Pod都是一样的
- 没有顺序要求
- 不用考虑在哪个node运行
- 随意进行伸缩和扩展



## 有状态

- 无状态中所有内容都需考虑到
- 让每个Pod独立的，保持Pod启动顺序和唯一性（唯一网络标识符，持久存储区分的）
  - 有序，比如mysql主从



# 部署有状态应用

## 无头service

- ClusterIP：none

首先得有一个无头的Service，即ClusterIP:none

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```



1. SatefulSet部署有状态应用

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels: 
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
  ..........
```



2. 部署应用

```bash
# 部署应用
kubectl applf -f sts.yaml

# 查看
kubectl get pods
```

3. 查看pod，每个都是唯一名称

![](..\..\img\pods.png)

4. 查看创建无头的service

```bash
kubectl get svc
```

![](..\..\img\service1.png)



# deployment和statefulset区别

- 有身份的（唯一标识的）
- 根据主机名+按照一定规则生成域名
- 每个pod有唯一主机名
- 唯一域名：
  - 格式：主机名称.service名称.名称空间.svc.cluster.local



# StatefulSet示例

yaml文件

```yaml
[root@k8s-master controller]# pwd
/root/k8s_practice/controller
[root@k8s-master controller]# cat statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: http
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10   # 默认30秒
      containers:
      - name: nginx
        image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
        ports:
        - containerPort: 80
          name: http
```

启动StatefulSet和Service，并查看状态

```bash
[root@k8s-master controller]# kubectl apply -f statefulset.yaml
service/nginx created
statefulset.apps/web created
[root@k8s-master controller]# kubectl get service -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17d   <none>
nginx        ClusterIP   None         <none>        80/TCP    87s   app=nginx
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get statefulset -o wide
NAME   READY   AGE   CONTAINERS   IMAGES
web    3/3     15m   nginx        registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          16m   10.244.2.95    k8s-node02   <none>           <none>
web-1   1/1     Running   0          16m   10.244.3.103   k8s-node01   <none>           <none>
web-2   1/1     Running   0          16m   10.244.3.104   k8s-node01   <none>           <none>
```

由上可见，StatefulSet 中的pod是有序的。有N个副本，那么序列号为0~(N-1)。

 

## 查看StatefulSet相关的域名信息

启动一个pod

```yaml
[root@k8s-master test]# pwd
/root/k8s_practice/test
[root@k8s-master test]# cat myapp_demo.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-demo
  namespace: default
  labels:
    k8s-app: myapp
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: httpd
      containerPort: 80
      protocol: TCP
[root@k8s-master test]#
[root@k8s-master test]# kubectl apply -f myapp_demo.yaml
pod/myapp-demo created
[root@k8s-master test]#
[root@k8s-master test]# kubectl get pod -o wide | grep 'myapp'
myapp-demo   1/1     Running   0          3m24s   10.244.2.101   k8s-node02   <none>           <none>
```

进入pod并查看StatefulSet域名信息

```bash
# 进入一个k8s管理的myapp镜像容器。
[root@k8s-master test]# kubectl exec -it myapp-demo sh
/ # nslookup 10.244.2.95
nslookup: can't resolve '(null)': Name does not resolve

Name:      10.244.2.95
Address 1: 10.244.2.95 web-0.nginx.default.svc.cluster.local
/ #
/ #
/ # nslookup 10.244.3.103
nslookup: can't resolve '(null)': Name does not resolve

Name:      10.244.3.103
Address 1: 10.244.3.103 web-1.nginx.default.svc.cluster.local
/ #
/ #
/ # nslookup 10.244.3.104
nslookup: can't resolve '(null)': Name does not resolve

Name:      10.244.3.104
Address 1: 10.244.3.104 web-2.nginx.default.svc.cluster.local
/ #
/ #
##### nginx.default.svc.cluster.local   为service的域名信息
/ # nslookup nginx.default.svc.cluster.local
nslookup: can't resolve '(null)': Name does not resolve

Name:      nginx.default.svc.cluster.local
Address 1: 10.244.3.104 web-2.nginx.default.svc.cluster.local
Address 2: 10.244.3.103 web-1.nginx.default.svc.cluster.local
Address 3: 10.244.2.95 web-0.nginx.default.svc.cluster.local
```

## StatefulSet网络标识与PVC

有上文可得如下信息：

1、匹配StatefulSet的Pod name(网络标识)的模式为：$(statefulset名称)-$(序号)，比如StatefulSet名称为web，副本数为3。则为：web-0、web-1、web-2

2、StatefulSet为每个Pod副本创建了一个DNS域名，这个域名的格式为：$(podname).(headless service name)，也就意味着服务之间是通过Pod域名来通信而非Pod IP。当Pod所在Node发生故障时，Pod会被漂移到其他Node上，Pod IP会发生改变，但Pod域名不会变化

3、StatefulSet使用Headless服务来控制Pod的域名，这个Headless服务域名的为：$(service name).$(namespace).svc.cluster.local，其中 cluster.local 指定的集群的域名

4、根据volumeClaimTemplates，为每个Pod创建一个PVC，PVC的命令规则为：$(volumeClaimTemplates name)-$(pod name)，比如volumeClaimTemplates为www，pod name为web-0、web-1、web-2；那么创建出来的PVC为：www-web-0、www-web-1、www-web-2

5、删除Pod不会删除对应的PVC，手动删除PVC将自动释放PV。