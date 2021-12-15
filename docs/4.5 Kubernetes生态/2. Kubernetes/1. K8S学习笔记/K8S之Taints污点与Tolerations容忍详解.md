[Kubernetes K8S之Taints污点与Tolerations容忍详解](https://www.cnblogs.com/zhanglianghhh/p/14022018.html)

# Taints污点和Tolerations容忍概述

节点和Pod亲和力，是将Pod吸引到一组节点【根据拓扑域】（作为优选或硬性要求）。污点（Taints）则相反，它们允许一个节点排斥一组Pod。

容忍（Tolerations）应用于pod，允许（但不强制要求）pod调度到具有匹配污点的节点上。

污点（Taints）和容忍（Tolerations）共同作用，确保pods不会被调度到不适当的节点。一个或多个污点应用于节点；这标志着该节点不应该接受任何不容忍污点的Pod。

说明：我们在平常使用中发现pod不会调度到k8s的master节点，就是因为master节点存在污点。

 

# Taints污点

## Taints污点的组成

使用kubectl taint命令可以给某个Node节点设置污点，Node被设置污点之后就和Pod之间存在一种相斥的关系，可以让Node拒绝Pod的调度执行，甚至将Node上已经存在的Pod驱逐出去。

每个污点的组成如下：

```
key=value:effect
```

每个污点有一个key和value作为污点的标签，effect描述污点的作用。当前taint effect支持如下选项：

- NoSchedule：表示K8S将不会把Pod调度到具有该污点的Node节点上
- PreferNoSchedule：表示K8S将尽量避免把Pod调度到具有该污点的Node节点上
- NoExecute：表示K8S将不会把Pod调度到具有该污点的Node节点上，同时会将Node上已经存在的Pod驱逐出去

 

## 污点taint的NoExecute详解

taint 的 effect 值 NoExecute，它会影响已经在节点上运行的 pod：

- 如果 pod 不能容忍 effect 值为 NoExecute 的 taint，那么 pod 将马上被驱逐
- 如果 pod 能够容忍 effect 值为 NoExecute 的 taint，且在 toleration 定义中没有指定 tolerationSeconds，则 pod 会一直在这个节点上运行。
- 如果 pod 能够容忍 effect 值为 NoExecute 的 taint，但是在toleration定义中指定了 tolerationSeconds，则表示 pod 还能在这个节点上继续运行的时间长度。

 

## Taints污点设置

### 污点（Taints）查看

k8s master节点查看

```
kubectl describe node k8s-master
```

![img](https://img2020.cnblogs.com/blog/1395193/202011/1395193-20201122222540281-1632397196.png)

 

k8s node查看

```
kubectl describe node k8s-node01
```

![img](https://img2020.cnblogs.com/blog/1395193/202011/1395193-20201122222602664-1130917242.png)

 

### 污点（Taints）添加

```bash
[root@k8s-master taint]# kubectl taint nodes k8s-node01 check=zhang:NoSchedule
node/k8s-node01 tainted
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl describe node k8s-node01
Name:               k8s-node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cpu-num=12
                    disk-type=ssd
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-node01
                    kubernetes.io/os=linux
                    mem-num=48
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"3e:15:bb:f8:85:dc"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.0.0.111
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 12 May 2020 16:50:54 +0800
Taints:             check=zhang:NoSchedule   ### 可见已添加污点
Unschedulable:      false
```

在k8s-node01节点添加了一个污点（taint），污点的key为check，value为zhang，污点effect为NoSchedule。这意味着没有pod可以调度到k8s-node01节点，除非具有相匹配的容忍。

 

### 污点（Taints）删除

```bash
[root@k8s-master taint]# kubectl taint nodes k8s-node01 check:NoExecute-
##### 或者
[root@k8s-master taint]# kubectl taint nodes k8s-node01 check=zhang:NoSchedule-
node/k8s-node01 untainted
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl describe node k8s-node01
Name:               k8s-node01
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cpu-num=12
                    disk-type=ssd
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-node01
                    kubernetes.io/os=linux
                    mem-num=48
Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"3e:15:bb:f8:85:dc"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 10.0.0.111
                    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Tue, 12 May 2020 16:50:54 +0800
Taints:             <none>   ### 可见已删除污点
Unschedulable:      false
```

# Tolerations容忍

设置了污点的Node将根据taint的effect：NoSchedule、PreferNoSchedule、NoExecute和Pod之间产生互斥的关系，Pod将在一定程度上不会被调度到Node上。

但我们可以在Pod上设置容忍（Tolerations），意思是设置了容忍的Pod将可以容忍污点的存在，可以被调度到存在污点的Node上。

**pod.spec.tolerations示例**

```bash
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
---
tolerations:
- key: "key"
  operator: "Exists"
  effect: "NoSchedule"
---
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

重要说明：

- 其中key、value、effect要与Node上设置的taint保持一致
- operator的值为Exists时，将会忽略value；只要有key和effect就行
- tolerationSeconds：表示pod 能够容忍 effect 值为 NoExecute 的 taint；当指定了 tolerationSeconds【容忍时间】，则表示 pod 还能在这个节点上继续运行的时间长度。

 

## 当不指定key值时

当不指定key值和effect值时，且operator为Exists，表示容忍所有的污点【能匹配污点所有的keys，values和effects】

```
tolerations:
- operator: "Exists"
```

 

## 当不指定effect值时

当不指定effect值时，则能匹配污点key对应的所有effects情况

```
tolerations:
- key: "key"
  operator: "Exists"
```

 

## 当有多个Master存在时

当有多个Master存在时，为了防止资源浪费，可以进行如下设置：

```bash
kubectl taint nodes Node-name node-role.kubernetes.io/master=:PreferNoSchedule
```

 

# 多个Taints污点和多个Tolerations容忍怎么判断

可以在同一个node节点上设置多个污点（Taints），在同一个pod上设置多个容忍（Tolerations）。Kubernetes处理多个污点和容忍的方式就像一个过滤器：从节点的所有污点开始，然后忽略可以被Pod容忍匹配的污点；保留其余不可忽略的污点，污点的effect对Pod具有显示效果：特别是：

- 如果有至少一个不可忽略污点，effect为NoSchedule，那么Kubernetes将不调度Pod到该节点
- 如果没有effect为NoSchedule的不可忽视污点，但有至少一个不可忽视污点，effect为PreferNoSchedule，那么Kubernetes将尽量不调度Pod到该节点
- 如果有至少一个不可忽视污点，effect为NoExecute，那么Pod将被从该节点驱逐（如果Pod已经在该节点运行），并且不会被调度到该节点（如果Pod还未在该节点运行）

 

# 污点和容忍示例

## Node污点为NoExecute的示例

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：
k8s-node02 污点为：
```

 

污点添加操作如下：
「无，本次无污点操作」

污点查看操作如下：

```
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

除了k8s-master默认的污点，在k8s-node01、k8s-node02无污点。

 

**污点为NoExecute示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat noexecute_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: noexec-tolerations-deploy
  labels:
    app: noexectolerations-deploy
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
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
       # 有容忍并有 tolerationSeconds 时的格式
#      tolerations:
#      - key: "check-mem"
#        operator: "Equal"
#        value: "memdb"
#        effect: "NoExecute"
#        # 当Pod将被驱逐时，Pod还可以在Node节点上继续保留运行的时间
#        tolerationSeconds: 30
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f noexecute_tolerations.yaml
deployment.apps/noexec-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
noexec-tolerations-deploy   6/6     6            6           10s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
noexec-tolerations-deploy-85587896f9-2j848   1/1     Running   0          15s   10.244.4.101   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-jgqkn   1/1     Running   0          15s   10.244.2.141   k8s-node02   <none>           <none>
noexec-tolerations-deploy-85587896f9-jmw5w   1/1     Running   0          15s   10.244.2.142   k8s-node02   <none>           <none>
noexec-tolerations-deploy-85587896f9-s8x95   1/1     Running   0          15s   10.244.4.102   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-t82fj   1/1     Running   0          15s   10.244.4.103   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-wx9pz   1/1     Running   0          15s   10.244.2.143   k8s-node02   <none>           <none>
```

由上可见，pod是在k8s-node01、k8s-node02平均分布的。

 

**添加effect为NoExecute的污点**

```
kubectl taint nodes k8s-node02 check-mem=memdb:NoExecute
```

 

此时所有节点污点为

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：
k8s-node02 污点为：check-mem=memdb:NoExecute
```

 

之后再次查看pod信息

```bash
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                         READY   STATUS    RESTARTS   AGE    IP             NODE         NOMINATED NODE   READINESS GATES
noexec-tolerations-deploy-85587896f9-2j848   1/1     Running   0          2m2s   10.244.4.101   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-ch96j   1/1     Running   0          8s     10.244.4.106   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-cjrkb   1/1     Running   0          8s     10.244.4.105   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-qbq6d   1/1     Running   0          7s     10.244.4.104   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-s8x95   1/1     Running   0          2m2s   10.244.4.102   k8s-node01   <none>           <none>
noexec-tolerations-deploy-85587896f9-t82fj   1/1     Running   0          2m2s   10.244.4.103   k8s-node01   <none>           <none>
```

由上可见，在k8s-node02节点上的pod已被驱逐，驱逐的pod被调度到了k8s-node01节点。

 

## Pod没有容忍时（Tolerations）

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：check-nginx=web:PreferNoSchedule
k8s-node02 污点为：check-nginx=web:NoSchedule
```

 

污点添加操作如下：

```bash
kubectl taint nodes k8s-node01 check-nginx=web:PreferNoSchedule
kubectl taint nodes k8s-node02 check-nginx=web:NoSchedule
```

 

污点查看操作如下：

```bash
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

 

**无容忍示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat no_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: no-tolerations-deploy
  labels:
    app: notolerations-deploy
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
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f no_tolerations.yaml
deployment.apps/no-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
no-tolerations-deploy   5/5     5            5           9s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
no-tolerations-deploy-85587896f9-6bjv8   1/1     Running   0          16s   10.244.4.54   k8s-node01   <none>           <none>
no-tolerations-deploy-85587896f9-hbbjb   1/1     Running   0          16s   10.244.4.58   k8s-node01   <none>           <none>
no-tolerations-deploy-85587896f9-jlmzw   1/1     Running   0          16s   10.244.4.56   k8s-node01   <none>           <none>
no-tolerations-deploy-85587896f9-kfh2c   1/1     Running   0          16s   10.244.4.55   k8s-node01   <none>           <none>
no-tolerations-deploy-85587896f9-wmp8b   1/1     Running   0          16s   10.244.4.57   k8s-node01   <none>           <none>
```

由上可见，因为k8s-node02节点的污点check-nginx 的effect为NoSchedule，说明pod不能被调度到该节点。此时k8s-node01节点的污点check-nginx 的effect为PreferNoSchedule【尽量不调度到该节点】；但只有该节点满足调度条件，因此都调度到了k8s-node01节点。

 

## Pod单个容忍时（Tolerations）

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：check-nginx=web:PreferNoSchedule
k8s-node02 污点为：check-nginx=web:NoSchedule
```

 

污点添加操作如下：

```bash
kubectl taint nodes k8s-node01 check-nginx=web:PreferNoSchedule
kubectl taint nodes k8s-node02 check-nginx=web:NoSchedule
```

 

污点查看操作如下：

```bash
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

 

**单个容忍示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat one_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: one-tolerations-deploy
  labels:
    app: onetolerations-deploy
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
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      tolerations:
      - key: "check-nginx"
        operator: "Equal"
        value: "web"
        effect: "NoSchedule"
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f one_tolerations.yaml
deployment.apps/one-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
one-tolerations-deploy   6/6     6            6           3s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
one-tolerations-deploy-5757d6b559-gbj49   1/1     Running   0          7s    10.244.2.73   k8s-node02   <none>           <none>
one-tolerations-deploy-5757d6b559-j9p6r   1/1     Running   0          7s    10.244.2.71   k8s-node02   <none>           <none>
one-tolerations-deploy-5757d6b559-kpk9q   1/1     Running   0          7s    10.244.2.72   k8s-node02   <none>           <none>
one-tolerations-deploy-5757d6b559-lsppn   1/1     Running   0          7s    10.244.4.65   k8s-node01   <none>           <none>
one-tolerations-deploy-5757d6b559-rx72g   1/1     Running   0          7s    10.244.4.66   k8s-node01   <none>           <none>
one-tolerations-deploy-5757d6b559-s8qr9   1/1     Running   0          7s    10.244.2.74   k8s-node02   <none>           <none>
```

由上可见，此时pod会尽量【优先】调度到k8s-node02节点，尽量不调度到k8s-node01节点。如果我们只有一个pod，那么会一直调度到k8s-node02节点。

 

## Pod多个容忍时（Tolerations）

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：check-nginx=web:PreferNoSchedule, check-redis=memdb:NoSchedule
k8s-node02 污点为：check-nginx=web:NoSchedule, check-redis=database:NoSchedule
```

 

污点添加操作如下：

```bash
kubectl taint nodes k8s-node01 check-nginx=web:PreferNoSchedule
kubectl taint nodes k8s-node01 check-redis=memdb:NoSchedule
kubectl taint nodes k8s-node02 check-nginx=web:NoSchedule
kubectl taint nodes k8s-node02 check-redis=database:NoSchedule
```

 

污点查看操作如下：

```bash
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

 

**多个容忍示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat multi_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-tolerations-deploy
  labels:
    app: multitolerations-deploy
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
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      tolerations:
      - key: "check-nginx"
        operator: "Equal"
        value: "web"
        effect: "NoSchedule"
      - key: "check-redis"
        operator: "Exists"
        effect: "NoSchedule"
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f multi_tolerations.yaml
deployment.apps/multi-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
multi-tolerations-deploy   6/6     6            6           5s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                        READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
multi-tolerations-deploy-776ff4449c-2csnk   1/1     Running   0          10s   10.244.2.171   k8s-node02   <none>           <none>
multi-tolerations-deploy-776ff4449c-4d9fh   1/1     Running   0          10s   10.244.4.116   k8s-node01   <none>           <none>
multi-tolerations-deploy-776ff4449c-c8fz5   1/1     Running   0          10s   10.244.2.173   k8s-node02   <none>           <none>
multi-tolerations-deploy-776ff4449c-nj29f   1/1     Running   0          10s   10.244.4.115   k8s-node01   <none>           <none>
multi-tolerations-deploy-776ff4449c-r7gsm   1/1     Running   0          10s   10.244.2.172   k8s-node02   <none>           <none>
multi-tolerations-deploy-776ff4449c-s8t2n   1/1     Running   0          10s   10.244.2.174   k8s-node02   <none>           <none>
```

由上可见，示例中的pod容忍为：check-nginx=web:NoSchedule；check-redis=:NoSchedule。因此pod会尽量调度到k8s-node02节点，尽量不调度到k8s-node01节点。

 

## Pod容忍指定污点key的所有effects情况

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：check-redis=memdb:NoSchedule
k8s-node02 污点为：check-redis=database:NoSchedule
```

 

污点添加操作如下：

```bash
kubectl taint nodes k8s-node01 check-redis=memdb:NoSchedule
kubectl taint nodes k8s-node02 check-redis=database:NoSchedule
```

 

污点查看操作如下：

```bash
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

 

**多个容忍示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat key_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: key-tolerations-deploy
  labels:
    app: keytolerations-deploy
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
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      tolerations:
      - key: "check-redis"
        operator: "Exists"
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f key_tolerations.yaml
deployment.apps/key-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
key-tolerations-deploy   6/6     6            6           21s   myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
key-tolerations-deploy-db5c4c4db-2zqr8   1/1     Running   0          26s   10.244.2.170   k8s-node02   <none>           <none>
key-tolerations-deploy-db5c4c4db-5qb5p   1/1     Running   0          26s   10.244.4.113   k8s-node01   <none>           <none>
key-tolerations-deploy-db5c4c4db-7xmt6   1/1     Running   0          26s   10.244.2.169   k8s-node02   <none>           <none>
key-tolerations-deploy-db5c4c4db-84rkj   1/1     Running   0          26s   10.244.4.114   k8s-node01   <none>           <none>
key-tolerations-deploy-db5c4c4db-gszxg   1/1     Running   0          26s   10.244.2.168   k8s-node02   <none>           <none>
key-tolerations-deploy-db5c4c4db-vlgh8   1/1     Running   0          26s   10.244.4.112   k8s-node01   <none>           <none>
```

由上可见，示例中的pod容忍为：check-nginx=:；仅需匹配node污点的key即可，污点的value和effect不需要关心。因此可以匹配k8s-node01、k8s-node02节点。

 

## Pod容忍所有污点

记得把已有的污点清除，以免影响测验。

**节点上的污点设置（Taints）**

实现如下污点

```
k8s-master 污点为：node-role.kubernetes.io/master:NoSchedule 【k8s自带污点，直接使用，不必另外操作添加】
k8s-node01 污点为：check-nginx=web:PreferNoSchedule, check-redis=memdb:NoSchedule
k8s-node02 污点为：check-nginx=web:NoSchedule, check-redis=database:NoSchedule
```

 

污点添加操作如下：

```bash
kubectl taint nodes k8s-node01 check-nginx=web:PreferNoSchedule
kubectl taint nodes k8s-node01 check-redis=memdb:NoSchedule
kubectl taint nodes k8s-node02 check-nginx=web:NoSchedule
kubectl taint nodes k8s-node02 check-redis=database:NoSchedule
```

 

污点查看操作如下：

```bash
kubectl describe node k8s-master | grep 'Taints' -A 5
kubectl describe node k8s-node01 | grep 'Taints' -A 5
kubectl describe node k8s-node02 | grep 'Taints' -A 5
```

 

**所有容忍示例**

yaml文件

```yaml
[root@k8s-master taint]# pwd
/root/k8s_practice/scheduler/taint
[root@k8s-master taint]#
[root@k8s-master taint]# cat all_tolerations.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: all-tolerations-deploy
  labels:
    app: alltolerations-deploy
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
      containers:
      - name: myapp-pod
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 80
      tolerations:
      - operator: "Exists"
```

运行yaml文件

```bash
[root@k8s-master taint]# kubectl apply -f all_tolerations.yaml
deployment.apps/all-tolerations-deploy created
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get deploy -o wide
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
all-tolerations-deploy   6/6     6            6           8s    myapp-pod    registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp
[root@k8s-master taint]#
[root@k8s-master taint]# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
all-tolerations-deploy-566cdccbcd-4klc2   1/1     Running   0          12s   10.244.0.116   k8s-master   <none>           <none>
all-tolerations-deploy-566cdccbcd-59vvc   1/1     Running   0          12s   10.244.0.115   k8s-master   <none>           <none>
all-tolerations-deploy-566cdccbcd-cvw4s   1/1     Running   0          12s   10.244.2.175   k8s-node02   <none>           <none>
all-tolerations-deploy-566cdccbcd-k8fzl   1/1     Running   0          12s   10.244.2.176   k8s-node02   <none>           <none>
all-tolerations-deploy-566cdccbcd-s2pw7   1/1     Running   0          12s   10.244.4.118   k8s-node01   <none>           <none>
all-tolerations-deploy-566cdccbcd-xzngt   1/1     Running   0          13s   10.244.4.117   k8s-node01   <none>           <none>
```

后上可见，示例中的pod容忍所有的污点，因此pod可被调度到所有k8s节点。