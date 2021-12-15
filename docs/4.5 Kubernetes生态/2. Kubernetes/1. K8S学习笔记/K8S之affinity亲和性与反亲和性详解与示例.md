[Kubernetes K8S之affinity亲和性与反亲和性详解与示例](https://www.cnblogs.com/zhanglianghhh/p/13922945.html)



# 亲和性和反亲和性

nodeSelector提供了一种非常简单的方法，将pods约束到具有特定标签的节点。而亲和性/反亲和性极大地扩展了可表达的约束类型。关键的增强是：

1、亲和性/反亲和性语言更具表达性。除了使用逻辑AND操作创建的精确匹配之外，该语言还提供了更多的匹配规则；

2、可以指示规则是优选项而不是硬要求，因此如果调度器不能满足，pod仍将被调度；

3、可以针对节点（或其他拓扑域）上运行的pods的标签进行约束，而不是针对节点的自身标签，这影响哪些Pod可以或不可以共处。

亲和特性包括两种类型：node节点亲和性/反亲和性 和 pod亲和性/反亲和性。pod亲和性/反亲和性约束针对的是pod标签而不是节点标签。

拓扑域是什么：多个node节点，拥有相同的label标签【节点标签的键值相同】，那么这些节点就处于同一个拓扑域。★★★★★

 

# node节点亲和性

当前有两种类型的节点亲和性，称为requiredDuringSchedulingIgnoredDuringExecution和 preferredDuringSchedulingIgnoredDuringExecution，可以将它们分别视为“硬”【必须满足条件】和“软”【优选满足条件】要求。

前者表示Pod要调度到的节点必须满足规则条件，不满足则不会调度，pod会一直处于Pending状态；后者表示优先调度到满足规则条件的节点，如果不能满足再调度到其他节点。

名称中的 IgnoredDuringExecution 部分意味着，与nodeSelector的工作方式类似，如果节点上的标签在Pod运行时发生更改，使得pod上的亲和性规则不再满足，那么pod仍将继续在该节点上运行。

在未来，会计划提供requiredDuringSchedulingRequiredDuringExecution，类似requiredDuringSchedulingIgnoredDuringExecution。不同之处就是pod运行过程中如果节点不再满足pod的亲和性，则pod会在该节点中逐出。

节点亲和性语法支持以下运算符：In，NotIn，Exists，DoesNotExist，Gt，Lt。可以使用NotIn和DoesNotExist实现节点的反亲和行为。

运算符关系：

- In：label的值在某个列表中
- NotIn：label的值不在某个列表中
- Gt：label的值大于某个值
- Lt：label的值小于某个值
- Exists：某个label存在
- DoesNotExist：某个label不存在

**其他重要说明：**

1、如果同时指定nodeSelector和nodeAffinity，则必须满足两个条件，才能将Pod调度到候选节点上。

2、如果在nodeAffinity类型下指定了多个nodeSelectorTerms对象【对象不能有多个，如果存在多个只有最后一个生效】，那么只有最后一个nodeSelectorTerms对象生效。

3、如果在nodeSelectorTerms下指定了多个matchExpressions列表，那么只要能满足其中一个matchExpressions，就可以将pod调度到某个节点上【针对节点硬亲和】。

![img](https://img2020.cnblogs.com/blog/1395193/202011/1395193-20201103220541486-420729092.png)

4、如果在matchExpressions下有多个key列表，那么只有当所有key满足时，才能将pod调度到某个节点【针对硬亲和】。

![img](https://img2020.cnblogs.com/blog/1395193/202011/1395193-20201103220558287-501828620.png)

5、在key下的values只要有一个满足条件，那么当前的key就满足条件

![img](https://img2020.cnblogs.com/blog/1395193/202011/1395193-20201103220616076-269149310.png)

6、如果pod已经调度在该节点，当我们删除或修该节点的标签时，pod不会被移除。换句话说，亲和性选择只有在pod调度期间有效。

7、preferredDuringSchedulingIgnoredDuringExecution中的weight（权重）字段在1-100范围内。对于每个满足所有调度需求的节点(资源请求、RequiredDuringScheduling亲和表达式等)，调度器将通过迭代该字段的元素来计算一个总和，如果节点与相应的匹配表达式匹配，则向该总和添加“权重”。然后将该分数与节点的其他优先级函数的分数结合起来。总得分最高的节点是最受欢迎的。

 

# node节点亲和性示例

## 准备事项

给node节点打label标签

```bash
###  --overwrite覆盖已存在的标签信息
kubectl label nodes k8s-node01 disk-type=ssd --overwrite
kubectl label nodes k8s-node01 cpu-num=12

kubectl label nodes k8s-node02 disk-type=sata
kubectl label nodes k8s-node02 cpu-num=24
```

查询所有节点标签信息

```bash
[root@k8s-master ~]# kubectl get node -o wide --show-labels
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME   LABELS
k8s-master   Ready    master   43d   v1.17.4   172.16.1.110   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node01   Ready    <none>   43d   v1.17.4   172.16.1.111   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,cpu-num=12,disk-type=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux
k8s-node02   Ready    <none>   43d   v1.17.4   172.16.1.112   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,cpu-num=24,disk-type=sata,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node02,kubernetes.io/os=linux
```

参见上面，给k8s-node01打了cpu-num=12,disk-type=ssd标签；给k8s-node02打了cpu-num=24,disk-type=sata标签。

 

## node硬亲和性示例

必须满足条件才能调度，否则不会调度

要运行的yaml文件

```yaml
[root@k8s-master nodeAffinity]# pwd
/root/k8s_practice/scheduler/nodeAffinity
[root@k8s-master nodeAffinity]# cat node_required_affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-affinity-deploy
  labels:
    app: nodeaffinity-deploy
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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              # 表示node标签存在 disk-type=ssd 或 disk-type=sas
              - key: disk-type
                operator: In
                values:
                - ssd
                - sas
              # 表示node标签存在 cpu-num且值大于6
              - key: cpu-num
                operator: Gt
                values:
                - "6"
```

运行yaml文件并查看状态

```bash
[root@k8s-master nodeAffinity]# kubectl apply -f node_required_affinity.yaml
deployment.apps/node-affinity-deploy created
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get deploy -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy   5/5     5            5           6s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get rs -o wide
NAME                              DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy-5c88ffb8ff   5         5         5       11s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5c88ffb8ff
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get pod -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
node-affinity-deploy-5c88ffb8ff-2mbfl   1/1     Running   0          15s   10.244.4.237   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-9hjhk   1/1     Running   0          15s   10.244.4.235   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-9rg75   1/1     Running   0          15s   10.244.4.239   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-pqtfh   1/1     Running   0          15s   10.244.4.236   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-zqpl8   1/1     Running   0          15s   10.244.4.238   k8s-node01   <none>           <none>
```

由上可见，再根据之前打的标签，很容易推断出当前pod只能调度在k8s-node01节点。

即使我们删除原来的rs，重新生成rs后pod依旧会调度到k8s-node01节点。如下：

```bash
[root@k8s-master nodeAffinity]# kubectl delete rs node-affinity-deploy-5c88ffb8ff
replicaset.apps "node-affinity-deploy-5c88ffb8ff" deleted
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get rs -o wide
NAME                              DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy-5c88ffb8ff   5         5         2       4s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5c88ffb8ff
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get pod -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
node-affinity-deploy-5c88ffb8ff-2v2tb   1/1     Running   0          11s   10.244.4.241   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-gl4fm   1/1     Running   0          11s   10.244.4.240   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-j26rg   1/1     Running   0          11s   10.244.4.244   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-vhzmn   1/1     Running   0          11s   10.244.4.243   k8s-node01   <none>           <none>
node-affinity-deploy-5c88ffb8ff-xxj8m   1/1     Running   0          11s   10.244.4.242   k8s-node01   <none>           <none>
```

## node软亲和性示例

优先调度到满足条件的节点，如果都不满足也会调度到其他节点。

要运行的yaml文件

```yaml
[root@k8s-master nodeAffinity]# pwd
/root/k8s_practice/scheduler/nodeAffinity
[root@k8s-master nodeAffinity]# cat node_preferred_affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-affinity-deploy
  labels:
    app: nodeaffinity-deploy
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
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              # 表示node标签存在 disk-type=ssd 或 disk-type=sas
              - key: disk-type
                operator: In
                values:
                - ssd
                - sas
          - weight: 50
            preference:
              matchExpressions:
              # 表示node标签存在 cpu-num且值大于16
              - key: cpu-num
                operator: Gt
                values:
                - "16"
```

运行yaml文件并查看状态

```yaml
[root@k8s-master nodeAffinity]# kubectl apply -f node_preferred_affinity.yaml
deployment.apps/node-affinity-deploy created
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get deploy -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy   5/5     5            5           9s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get rs -o wide
NAME                             DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy-d5d9cbc8d   5         5         5       13s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=d5d9cbc8d
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get pod -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
node-affinity-deploy-d5d9cbc8d-bv86t   1/1     Running   0          18s   10.244.2.243   k8s-node02   <none>           <none>
node-affinity-deploy-d5d9cbc8d-dnbr8   1/1     Running   0          18s   10.244.2.244   k8s-node02   <none>           <none>
node-affinity-deploy-d5d9cbc8d-ldq82   1/1     Running   0          18s   10.244.2.246   k8s-node02   <none>           <none>
node-affinity-deploy-d5d9cbc8d-nt74q   1/1     Running   0          18s   10.244.4.2     k8s-node01   <none>           <none>
node-affinity-deploy-d5d9cbc8d-rt5nb   1/1     Running   0          18s   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，再根据之前打的标签，很容易推断出当前pod会【优先】调度在k8s-node02节点。

 

## node软硬亲和性联合示例

硬亲和性与软亲和性一起使用

要运行的yaml文件

```yaml
[root@k8s-master nodeAffinity]# pwd
/root/k8s_practice/scheduler/nodeAffinity
[root@k8s-master nodeAffinity]# cat node_affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-affinity-deploy
  labels:
    app: nodeaffinity-deploy
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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              # 表示node标签存在 cpu-num且值大于10
            - matchExpressions:
              - key: cpu-num
                operator: Gt
                values:
                - "10"
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 50
            preference:
              matchExpressions:
                # 表示node标签存在 disk-type=ssd 或 disk-type=sas
              - key: disk-type
                operator: In
                values:
                - ssd
                - sas
```

运行yaml文件并查看状态

```bash
[root@k8s-master nodeAffinity]# kubectl apply -f node_affinity.yaml
deployment.apps/node-affinity-deploy created
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get deploy -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy   5/5     5            5           9s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get rs -o wide
NAME                             DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
node-affinity-deploy-f9cb9b99b   5         5         5       13s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=f9cb9b99b
[root@k8s-master nodeAffinity]#
[root@k8s-master nodeAffinity]# kubectl get pod -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
node-affinity-deploy-f9cb9b99b-8w2nc   1/1     Running   0          17s   10.244.4.10    k8s-node01   <none>           <none>
node-affinity-deploy-f9cb9b99b-csk2s   1/1     Running   0          17s   10.244.4.9     k8s-node01   <none>           <none>
node-affinity-deploy-f9cb9b99b-g42kq   1/1     Running   0          17s   10.244.4.8     k8s-node01   <none>           <none>
node-affinity-deploy-f9cb9b99b-m6xbv   1/1     Running   0          17s   10.244.4.7     k8s-node01   <none>           <none>
node-affinity-deploy-f9cb9b99b-mxbdp   1/1     Running   0          17s   10.244.2.253   k8s-node02   <none>           <none>
```

由上可见，再根据之前打的标签，很容易推断出k8s-node01、k8s-node02都满足必要条件，但当前pod会【优先】调度在k8s-node01节点。

 

# Pod亲和性

与节点亲和性一样，当前有Pod亲和性/反亲和性都有两种类型，称为requiredDuringSchedulingIgnoredDuringExecution和 preferredDuringSchedulingIgnoredDuringExecution，分别表示“硬”与“软”要求。对于硬要求，如果不满足则pod会一直处于Pending状态。

Pod的亲和性与反亲和性是基于Node节点上已经运行pod的标签(而不是节点上的标签)决定的，从而约束哪些节点适合调度你的pod。

规则的形式是：如果X已经运行了一个或多个符合规则Y的pod，则此pod应该在X中运行(如果是反亲和的情况下，则不应该在X中运行）。当然pod必须处在同一名称空间，不然亲和性/反亲和性无作用。从概念上讲，X是一个拓扑域。我们可以使用topologyKey来表示它，topologyKey 的值是node节点标签的键以便系统用来表示这样的拓扑域。当然这里也有个隐藏条件，就是node节点标签的键值相同时，才是在同一拓扑域中；如果只是节点标签名相同，但是值不同，那么也不在同一拓扑域。★★★★★

也就是说：Pod的亲和性/反亲和性调度是根据拓扑域来界定调度的，而不是根据node节点。★★★★★

 

**注意事项**

1、pod之间亲和性/反亲和性需要大量的处理，这会明显降低大型集群中的调度速度。不建议在大于几百个节点的集群中使用它们。

2、Pod反亲和性要求对节点进行一致的标记。换句话说，集群中的每个节点都必须有一个匹配topologyKey的适当标签。如果某些或所有节点缺少指定的topologyKey标签，可能会导致意外行为。

requiredDuringSchedulingIgnoredDuringExecution中亲和性的一个示例是“将服务A和服务B的Pod放置在同一区域【拓扑域】中，因为它们之间有很多交流”；preferredDuringSchedulingIgnoredDuringExecution中反亲和性的示例是“将此服务的 pod 跨区域【拓扑域】分布”【此时硬性要求是说不通的，因为你可能拥有的 pod 数多于区域数】。

Pod亲和性/反亲和性语法支持以下运算符：In，NotIn，Exists，DoesNotExist。

 

**原则上，topologyKey可以是任何合法的标签键。但是，出于性能和安全方面的原因，topologyKey有一些限制：**

1、对于Pod亲和性，在requiredDuringSchedulingIgnoredDuringExecution和preferredDuringSchedulingIgnoredDuringExecution中topologyKey都不允许为空。

2、对于Pod反亲和性，在requiredDuringSchedulingIgnoredDuringExecution和preferredDuringSchedulingIgnoredDuringExecution中topologyKey也都不允许为空。

3、对于requiredDuringSchedulingIgnoredDuringExecution的pod反亲和性，引入了允许控制器LimitPodHardAntiAffinityTopology来限制topologyKey的kubernet.io/hostname。如果你想让它对自定义拓扑可用，你可以修改许可控制器，或者干脆禁用它。

4、除上述情况外，topologyKey可以是任何合法的标签键。

Pod 间亲和通过 PodSpec 中 affinity 字段下的 podAffinity 字段进行指定。而 pod 间反亲和通过 PodSpec 中 affinity 字段下的 podAntiAffinity 字段进行指定。

Pod亲和性/反亲和性的requiredDuringSchedulingIgnoredDuringExecution所关联的matchExpressions下有多个key列表，那么只有当所有key满足时，才能将pod调度到某个区域【针对Pod硬亲和】。

 

# pod亲和性与反亲和性示例

为了更好的演示Pod亲和性与反亲和性，本次示例我们会将k8s-master节点也加入进来进行演示。

## 准备事项

给node节点打label标签

```bash
# 删除已存在标签
kubectl label nodes k8s-node01 cpu-num-
kubectl label nodes k8s-node01 disk-type-
kubectl label nodes k8s-node02 cpu-num-
kubectl label nodes k8s-node02 disk-type-


###  --overwrite覆盖已存在的标签信息
# k8s-master 标签添加
kubectl label nodes k8s-master busi-use=www --overwrite
kubectl label nodes k8s-master disk-type=ssd --overwrite
kubectl label nodes k8s-master busi-db=redis

# k8s-node01 标签添加
kubectl label nodes k8s-node01 busi-use=www
kubectl label nodes k8s-node01 disk-type=sata
kubectl label nodes k8s-node01 busi-db=redis

# k8s-node02 标签添加
kubectl label nodes k8s-node02 busi-use=www
kubectl label nodes k8s-node02 disk-type=ssd
kubectl label nodes k8s-node02 busi-db=etcd
```

查询所有节点标签信息

```bash
[root@k8s-master ~]# kubectl get node -o wide --show-labels
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME   LABELS
k8s-master   Ready    master   28d   v1.17.4   172.16.1.110   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,busi-db=redis,busi-use=www,disk-type=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master,kubernetes.io/os=linux,node-role.kubernetes.io/master=
k8s-node01   Ready    <none>   28d   v1.17.4   172.16.1.111   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,busi-db=redis,busi-use=www,disk-type=sata,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node01,kubernetes.io/os=linux
k8s-node02   Ready    <none>   28d   v1.17.4   172.16.1.112   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64   docker://19.3.8     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,busi-db=etcd,busi-use=www,disk-type=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-node02,kubernetes.io/os=linux
```

如上所述：k8s-master添加了disk-type=ssd,busi-db=redis,busi-use=www标签

k8s-node01添加了disk-type=sata,busi-db=redis,busi-use=www标签

k8s-node02添加了disk-type=ssd,busi-db=etcd,busi-use=www标签

通过deployment运行一个pod，或者直接运行一个pod也可以。为后续的Pod亲和性与反亲和性测验做基础。

```bash
### yaml文件
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]# cat web_deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  labels:
    app: myweb-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-web
  template:
    metadata:
      labels:
        app: myapp-web
        version: v1
    spec:
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
[root@k8s-master podAffinity]#
### 运行yaml文件
[root@k8s-master podAffinity]# kubectl apply -f web_deploy.yaml
deployment.apps/web-deploy created
[root@k8s-master podAffinity]#
### 查看pod标签
[root@k8s-master podAffinity]# kubectl get pod -o wide --show-labels
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES   LABELS
web-deploy-5ccc9d7c55-kkwst   1/1     Running   0          15m   10.244.2.4   k8s-node02   <none>           <none>            app=myapp-web,pod-template-hash=5ccc9d7c55,version=v1
```

当前pod在k8s-node02节点；其中pod的标签app=myapp-web,version=v1会在后面pod亲和性/反亲和性示例中使用。

 

## pod硬亲和性示例

要运行的yaml文件

```bash
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]# cat pod_required_affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-podaffinity-deploy
  labels:
    app: podaffinity-deploy
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # 允许在master节点运行
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              # 由于是Pod亲和性/反亲和性；因此这里匹配规则写的是Pod的标签信息
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp-web
            # 拓扑域  若多个node节点具有相同的标签信息【标签键值相同】，则表示这些node节点就在同一拓扑域
            # 请对比如下两个不同的拓扑域，Pod的调度结果
            #topologyKey: busi-use
            topologyKey: disk-type
```

运行yaml文件并查看状态

```bash
[root@k8s-master podAffinity]# kubectl apply -f pod_required_affinity.yaml
deployment.apps/pod-podaffinity-deploy created
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get deploy -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-deploy   6/6     6            6           48s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
web-deploy               1/1     1            1           22h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get rs -o wide
NAME                                DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-deploy-848559bf5b   6         6         6       52s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=848559bf5b
web-deploy-5ccc9d7c55               1         1         1       22h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web,pod-template-hash=5ccc9d7c55
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-podaffinity-deploy-848559bf5b-8kkwm   1/1     Running   0          54s   10.244.0.80    k8s-master   <none>           <none>
pod-podaffinity-deploy-848559bf5b-8s59f   1/1     Running   0          54s   10.244.2.252   k8s-node02   <none>           <none>
pod-podaffinity-deploy-848559bf5b-8z4dv   1/1     Running   0          54s   10.244.2.253   k8s-node02   <none>           <none>
pod-podaffinity-deploy-848559bf5b-gs7sb   1/1     Running   0          54s   10.244.0.79    k8s-master   <none>           <none>
pod-podaffinity-deploy-848559bf5b-sm6nz   1/1     Running   0          54s   10.244.0.78    k8s-master   <none>           <none>
pod-podaffinity-deploy-848559bf5b-zbr6v   1/1     Running   0          54s   10.244.2.251   k8s-node02   <none>           <none>
web-deploy-5ccc9d7c55-khhrr               1/1     Running   3          22h   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，yaml文件中为topologyKey: disk-type；虽然k8s-master、k8s-node01、k8s-node02都有disk-type标签；但是k8s-master和k8s-node02节点的disk-type标签值为ssd；而k8s-node01节点的disk-type标签值为sata。因此k8s-master和k8s-node02节点属于同一拓扑域，Pod只会调度到这两个节点上。

 

## pod软亲和性示例

要运行的yaml文件

```yaml
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# cat pod_preferred_affinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-podaffinity-deploy
  labels:
    app: podaffinity-deploy
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # 允许在master节点运行
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                # 由于是Pod亲和性/反亲和性；因此这里匹配规则写的是Pod的标签信息
                matchExpressions:
                - key: version
                  operator: In
                  values:
                  - v1
                  - v2
              # 拓扑域  若多个node节点具有相同的标签信息【标签键值相同】，则表示这些node节点就在同一拓扑域
              topologyKey: disk-type
```

运行yaml文件并查看状态

```bash
[root@k8s-master podAffinity]# kubectl apply -f pod_preferred_affinity.yaml
deployment.apps/pod-podaffinity-deploy created
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get deploy -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-deploy   6/6     6            6           75s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
web-deploy               1/1     1            1           25h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get rs -o wide
NAME                                DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-deploy-8474b4b586   6         6         6       79s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=8474b4b586
web-deploy-5ccc9d7c55               1         1         1       25h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web,pod-template-hash=5ccc9d7c55
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-podaffinity-deploy-8474b4b586-57gxh   1/1     Running   0          83s   10.244.2.4     k8s-node02   <none>           <none>
pod-podaffinity-deploy-8474b4b586-kd5l4   1/1     Running   0          83s   10.244.2.3     k8s-node02   <none>           <none>
pod-podaffinity-deploy-8474b4b586-mlvv7   1/1     Running   0          83s   10.244.0.84    k8s-master   <none>           <none>
pod-podaffinity-deploy-8474b4b586-mtk6r   1/1     Running   0          83s   10.244.0.86    k8s-master   <none>           <none>
pod-podaffinity-deploy-8474b4b586-n5jpj   1/1     Running   0          83s   10.244.0.85    k8s-master   <none>           <none>
pod-podaffinity-deploy-8474b4b586-q2xdl   1/1     Running   0          83s   10.244.3.22    k8s-node01   <none>           <none>
web-deploy-5ccc9d7c55-khhrr               1/1     Running   3          25h   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，再根据k8s-master、k8s-node01、k8s-node02的标签信息；很容易推断出Pod会优先调度到k8s-master、k8s-node02节点。

 

## pod硬反亲和性示例

要运行的yaml文件

```yaml
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# cat pod_required_AntiAffinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-podantiaffinity-deploy
  labels:
    app: podantiaffinity-deploy
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # 允许在master节点运行
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              # 由于是Pod亲和性/反亲和性；因此这里匹配规则写的是Pod的标签信息
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp-web
            # 拓扑域  若多个node节点具有相同的标签信息【标签键值相同】，则表示这些node节点就在同一拓扑域
            topologyKey: disk-type
```

运行yaml文件并查看状态

```bash
[root@k8s-master podAffinity]# kubectl apply -f pod_required_AntiAffinity.yaml
deployment.apps/pod-podantiaffinity-deploy created
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get deploy -o wide
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podantiaffinity-deploy   6/6     6            6           68s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
web-deploy                   1/1     1            1           25h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get rs -o wide
NAME                                    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podantiaffinity-deploy-5fb4764b6b   6         6         6       72s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5fb4764b6b
web-deploy-5ccc9d7c55                   1         1         1       25h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web,pod-template-hash=5ccc9d7c55
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get pod -o wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-podantiaffinity-deploy-5fb4764b6b-b5bzd   1/1     Running   0          75s   10.244.3.28    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-5fb4764b6b-b6qjg   1/1     Running   0          75s   10.244.3.23    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-5fb4764b6b-h262g   1/1     Running   0          75s   10.244.3.27    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-5fb4764b6b-q98gt   1/1     Running   0          75s   10.244.3.24    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-5fb4764b6b-v6kpm   1/1     Running   0          75s   10.244.3.25    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-5fb4764b6b-wtmm6   1/1     Running   0          75s   10.244.3.26    k8s-node01   <none>           <none>
web-deploy-5ccc9d7c55-khhrr                   1/1     Running   3          25h   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，由于是Pod反亲和测验，再根据k8s-master、k8s-node01、k8s-node02的标签信息；很容易推断出Pod只能调度到k8s-node01节点。

 

## pod软反亲和性示例

要运行的yaml文件

```yaml
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# cat pod_preferred_AntiAffinity.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-podantiaffinity-deploy
  labels:
    app: podantiaffinity-deploy
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # 允许在master节点运行
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                # 由于是Pod亲和性/反亲和性；因此这里匹配规则写的是Pod的标签信息
                matchExpressions:
                - key: version
                  operator: In
                  values:
                  - v1
                  - v2
              # 拓扑域  若多个node节点具有相同的标签信息【标签键值相同】，则表示这些node节点就在同一拓扑域
              topologyKey: disk-type
```

运行yaml文件并查看状态

```bash
[root@k8s-master podAffinity]# kubectl apply -f pod_preferred_AntiAffinity.yaml
deployment.apps/pod-podantiaffinity-deploy created
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get deploy -o wide
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podantiaffinity-deploy   6/6     6            6           9s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
web-deploy                   1/1     1            1           26h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get rs -o wide
NAME                                    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podantiaffinity-deploy-54d758ddb4   6         6         6       13s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=54d758ddb4
web-deploy-5ccc9d7c55                   1         1         1       26h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web,pod-template-hash=5ccc9d7c55
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get pod -o wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-podantiaffinity-deploy-54d758ddb4-58t9p   1/1     Running   0          17s   10.244.3.31    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-54d758ddb4-9ntd7   1/1     Running   0          17s   10.244.3.32    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-54d758ddb4-9wr6p   1/1     Running   0          17s   10.244.2.5     k8s-node02   <none>           <none>
pod-podantiaffinity-deploy-54d758ddb4-gnls4   1/1     Running   0          17s   10.244.3.30    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-54d758ddb4-jlftn   1/1     Running   0          17s   10.244.3.29    k8s-node01   <none>           <none>
pod-podantiaffinity-deploy-54d758ddb4-mvplv   1/1     Running   0          17s   10.244.0.87    k8s-master   <none>           <none>
web-deploy-5ccc9d7c55-khhrr                   1/1     Running   3          26h   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，由于是Pod反亲和测验，再根据k8s-master、k8s-node01、k8s-node02的标签信息；很容易推断出Pod会优先调度到k8s-node01节点。

 

## pod亲和性与反亲和性联合示例

要运行的yaml文件

```yaml
[root@k8s-master podAffinity]# pwd
/root/k8s_practice/scheduler/podAffinity
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# cat pod_podAffinity_all.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-podaffinity-all-deploy
  labels:
    app: podaffinity-all-deploy
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # 允许在master节点运行
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              # 由于是Pod亲和性/反亲和性；因此这里匹配规则写的是Pod的标签信息
              matchExpressions:
              - key: app
                operator: In
                values:
                - myapp-web
            # 拓扑域  若多个node节点具有相同的标签信息【标签键值相同】，则表示这些node节点就在同一拓扑域
            topologyKey: disk-type
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: version
                  operator: In
                  values:
                  - v1
                  - v2
              topologyKey: busi-db
```

运行yaml文件并查看状态

```bash
[root@k8s-master podAffinity]# kubectl apply -f pod_podAffinity_all.yaml
deployment.apps/pod-podaffinity-all-deploy created
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get deploy -o wide
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-all-deploy   6/6     6            1           5s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
web-deploy                   1/1     1            1           28h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get rs -o wide
NAME                                    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
pod-podaffinity-all-deploy-5ddbf9cbf8   6         6         6       10s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5ddbf9cbf8
web-deploy-5ccc9d7c55                   1         1         1       28h   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp-web,pod-template-hash=5ccc9d7c55
[root@k8s-master podAffinity]#
[root@k8s-master podAffinity]# kubectl get pod -o wide
NAME                                          READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-podaffinity-all-deploy-5ddbf9cbf8-5w5b7   1/1     Running   0          15s   10.244.0.91    k8s-master   <none>           <none>
pod-podaffinity-all-deploy-5ddbf9cbf8-j57g9   1/1     Running   0          15s   10.244.0.90    k8s-master   <none>           <none>
pod-podaffinity-all-deploy-5ddbf9cbf8-kwz6w   1/1     Running   0          15s   10.244.0.92    k8s-master   <none>           <none>
pod-podaffinity-all-deploy-5ddbf9cbf8-l8spj   1/1     Running   0          15s   10.244.2.6     k8s-node02   <none>           <none>
pod-podaffinity-all-deploy-5ddbf9cbf8-lf22c   1/1     Running   0          15s   10.244.0.89    k8s-master   <none>           <none>
pod-podaffinity-all-deploy-5ddbf9cbf8-r2fgl   1/1     Running   0          15s   10.244.0.88    k8s-master   <none>           <none>
web-deploy-5ccc9d7c55-khhrr                   1/1     Running   3          28h   10.244.2.245   k8s-node02   <none>           <none>
```

由上可见，根据k8s-master、k8s-node01、k8s-node02的标签信息；很容易推断出Pod只能调度到k8s-master、k8s-node02节点，且会优先调度到k8s-master节点。