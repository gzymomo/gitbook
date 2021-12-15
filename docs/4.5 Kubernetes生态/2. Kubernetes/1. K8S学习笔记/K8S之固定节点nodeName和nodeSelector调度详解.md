[Kubernetes K8S之固定节点nodeName和nodeSelector调度详解](https://www.cnblogs.com/zhanglianghhh/p/14077078.html)

# nodeName调度

nodeName是节点选择约束的最简单形式，但是由于其限制，通常很少使用它。nodeName是PodSpec的领域。

pod.spec.nodeName将Pod直接调度到指定的Node节点上，会【跳过Scheduler的调度策略】，该匹配规则是【强制】匹配。可以越过Taints污点进行调度。

nodeName用于选择节点的一些限制是：

- 如果指定的节点不存在，则容器将不会运行，并且在某些情况下可能会自动删除。
- 如果指定的节点没有足够的资源来容纳该Pod，则该Pod将会失败，并且其原因将被指出，例如OutOfmemory或OutOfcpu。
- 云环境中的节点名称并非总是可预测或稳定的。

# nodeName示例

获取当前的节点信息

```bash
[root@k8s-master scheduler]# kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k8s-master   Ready    master   42d   v1.17.4   172.16.1.110   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
k8s-node01   Ready    <none>   42d   v1.17.4   172.16.1.111   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
k8s-node02   Ready    <none>   42d   v1.17.4   172.16.1.112   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8
```

 

## 当nodeName指定节点存在

要运行的yaml文件

```yaml
[root@k8s-master scheduler]# pwd
/root/k8s_practice/scheduler
[root@k8s-master scheduler]# cat scheduler_nodeName.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scheduler-nodename-deploy
  labels:
    app: nodename-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      # 指定节点运行
      nodeName: k8s-master
```

运行yaml文件并查看信息

```bash
[root@k8s-master scheduler]# kubectl apply -f scheduler_nodeName.yaml
deployment.apps/scheduler-nodename-deploy created
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get deploy -o wide
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodename-deploy   0/5     5            0           6s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get rs -o wide
NAME                                  DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodename-deploy-d5c9574bd   5         5         5       15s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=d5c9574bd
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get pod -o wide
NAME                                        READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
scheduler-nodename-deploy-d5c9574bd-6l9d8   1/1     Running   0          23s   10.244.0.123   k8s-master   <none>           <none>
scheduler-nodename-deploy-d5c9574bd-c82cc   1/1     Running   0          23s   10.244.0.119   k8s-master   <none>           <none>
scheduler-nodename-deploy-d5c9574bd-dkkjg   1/1     Running   0          23s   10.244.0.122   k8s-master   <none>           <none>
scheduler-nodename-deploy-d5c9574bd-hcn77   1/1     Running   0          23s   10.244.0.121   k8s-master   <none>           <none>
scheduler-nodename-deploy-d5c9574bd-zstjx   1/1     Running   0          23s   10.244.0.120   k8s-master   <none>           <none>
```

由上可见，yaml文件中nodeName: k8s-master生效，所有pod被调度到了k8s-master节点。如果这里是nodeName: k8s-node02，那么就会直接调度到k8s-node02节点。

## 当nodeName指定节点不存在

要运行的yaml文件

```yaml
[root@k8s-master scheduler]# pwd
/root/k8s_practice/scheduler
[root@k8s-master scheduler]# cat scheduler_nodeName_02.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scheduler-nodename-deploy
  labels:
    app: nodename-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      # 指定节点运行，该节点不存在
      nodeName: k8s-node08
```

运行yaml文件并查看信息

```bash
[root@k8s-master scheduler]# kubectl apply -f scheduler_nodeName_02.yaml
deployment.apps/scheduler-nodename-deploy created
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get deploy -o wide
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodename-deploy   0/5     5            0           4s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get rs -o wide
NAME                                   DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodename-deploy-75944bdc5d   5         5         0       9s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=75944bdc5d
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get pod -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP       NODE         NOMINATED NODE   READINESS GATES
scheduler-nodename-deploy-75944bdc5d-c8f5d   0/1     Pending   0          13s   <none>   k8s-node08   <none>           <none>
scheduler-nodename-deploy-75944bdc5d-hfdlv   0/1     Pending   0          13s   <none>   k8s-node08   <none>           <none>
scheduler-nodename-deploy-75944bdc5d-q9qgt   0/1     Pending   0          13s   <none>   k8s-node08   <none>           <none>
scheduler-nodename-deploy-75944bdc5d-q9zl7   0/1     Pending   0          13s   <none>   k8s-node08   <none>           <none>
scheduler-nodename-deploy-75944bdc5d-wxsnv   0/1     Pending   0          13s   <none>   k8s-node08   <none>           <none>
```

由上可见，如果指定的节点不存在，则容器将不会运行，一直处于Pending 状态。

 

# nodeSelector调度

nodeSelector是节点选择约束的最简单推荐形式。nodeSelector是PodSpec的领域。它指定键值对的映射。

Pod.spec.nodeSelector是通过Kubernetes的label-selector机制选择节点，由调度器调度策略匹配label，而后调度Pod到目标节点，该匹配规则属于【强制】约束。由于是调度器调度，因此不能越过Taints污点进行调度。

 

# nodeSelector示例

获取当前的节点信息

```bash
[root@k8s-master ~]# kubectl get node -o wide --show-labels
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME   LABELS
k8s-master   Ready    master   42d   v1.17.4   172.16.1.110   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node01   Ready    <none>   42d   v1.17.4   172.16.1.111   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux
k8s-node02   Ready    <none>   42d   v1.17.4   172.16.1.112   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node02,kubernetes.io/os=linux
```

## 添加label标签

运行kubectl get nodes以获取群集节点的名称。然后可以对指定节点添加标签。比如：k8s-node01的磁盘为SSD，那么添加disk-type=ssd；k8s-node02的CPU核数高，那么添加cpu-type=hight；如果为Web机器，那么添加service-type=web。怎么添加标签可以根据实际规划情况而定。

```bash
### 给k8s-node01 添加指定标签
[root@k8s-master ~]# kubectl label nodes k8s-node01 disk-type=ssd
node/k8s-node01 labeled
#### 删除标签命令 kubectl label nodes k8s-node01 disk-type-
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get node --show-labels
NAME         STATUS   ROLES    AGE   VERSION   LABELS
k8s-master   Ready    master   42d   v1.17.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node01   Ready    <none>   42d   v1.17.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disk-type=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux
k8s-node02   Ready    <none>   42d   v1.17.4   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node02,kubernetes.io/os=linux
```

由上可见，已经为k8s-node01节点添加了disk-type=ssd 标签。

 

## 当nodeSelector标签存在

要运行的yaml文件

```yaml
[root@k8s-master scheduler]# pwd
/root/k8s_practice/scheduler
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# cat scheduler_nodeSelector.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scheduler-nodeselector-deploy
  labels:
    app: nodeselector-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      # 指定节点标签选择，且标签存在
      nodeSelector:
        disk-type: ssd
```

运行yaml文件并查看信息

```bash
[root@k8s-master scheduler]# kubectl apply -f scheduler_nodeSelector.yaml
deployment.apps/scheduler-nodeselector-deploy created
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get deploy -o wide
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodeselector-deploy   5/5     5            5           10s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get rs -o wide
NAME                                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodeselector-deploy-79455db454   5         5         5       14s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=79455db454
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get pod -o wide
NAME                                             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
scheduler-nodeselector-deploy-79455db454-745ph   1/1     Running   0          19s   10.244.4.154   k8s-node01   <none>           <none>
scheduler-nodeselector-deploy-79455db454-bmjvd   1/1     Running   0          19s   10.244.4.151   k8s-node01   <none>           <none>
scheduler-nodeselector-deploy-79455db454-g5cg2   1/1     Running   0          19s   10.244.4.153   k8s-node01   <none>           <none>
scheduler-nodeselector-deploy-79455db454-hw8jv   1/1     Running   0          19s   10.244.4.152   k8s-node01   <none>           <none>
scheduler-nodeselector-deploy-79455db454-zrt8d   1/1     Running   0          19s   10.244.4.155   k8s-node01   <none>           <none>
```

由上可见，所有pod都被调度到了k8s-node01节点。当然如果其他节点也有disk-type=ssd 标签，那么pod也会调度到这些节点上。

 

## 当nodeSelector标签不存在

要运行的yaml文件

```yaml
[root@k8s-master scheduler]# pwd
/root/k8s_practice/scheduler
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# cat scheduler_nodeSelector_02.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scheduler-nodeselector-deploy
  labels:
    app: nodeselector-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      # 指定节点标签选择，且标签不存在
      nodeSelector:
        service-type: web
```

运行yaml文件并查看信息

```bash
[root@k8s-master scheduler]# kubectl apply -f scheduler_nodeSelector_02.yaml
deployment.apps/scheduler-nodeselector-deploy created
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get deploy -o wide
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodeselector-deploy   0/5     5            0           26s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get rs -o wide
NAME                                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
scheduler-nodeselector-deploy-799d748db6   5         5         0       30s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=799d748db6
[root@k8s-master scheduler]#
[root@k8s-master scheduler]# kubectl get pod -o wide
NAME                                             READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
scheduler-nodeselector-deploy-799d748db6-92mqj   0/1     Pending   0          40s   <none>   <none>   <none>           <none>
scheduler-nodeselector-deploy-799d748db6-c2w25   0/1     Pending   0          40s   <none>   <none>   <none>           <none>
scheduler-nodeselector-deploy-799d748db6-c8tlx   0/1     Pending   0          40s   <none>   <none>   <none>           <none>
scheduler-nodeselector-deploy-799d748db6-tc5n7   0/1     Pending   0          40s   <none>   <none>   <none>           <none>
scheduler-nodeselector-deploy-799d748db6-z8c57   0/1     Pending   0          40s   <none>   <none>   <none>           <none>
```

由上可见，如果nodeSelector匹配的标签不存在，则容器将不会运行，一直处于Pending 状态。