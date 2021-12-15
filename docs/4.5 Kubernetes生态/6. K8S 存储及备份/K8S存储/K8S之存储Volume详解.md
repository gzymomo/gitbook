- [Kubernetes K8S之存储Volume详解](https://www.cnblogs.com/zhanglianghhh/p/13844062.html)

- [K8s之Volume的基础使用](https://www.cnblogs.com/qiuhom-1874/p/14180752.html)



# 一、Volume概述

在容器中的文件在磁盘上是临时存放的，当容器关闭时这些临时文件也会被一并清除。这给容器中运行的特殊应用程序带来一些问题。

首先，当容器崩溃时，kubelet 将重新启动容器，容器中的文件将会丢失——因为容器会以干净的状态重建。

其次，当在一个 Pod 中同时运行多个容器时，常常需要在这些容器之间共享文件。

Kubernetes 抽象出 Volume 对象来解决这两个问题。

Kubernetes Volume卷具有明确的生命周期——与包裹它的 Pod 相同。 因此，Volume比 Pod 中运行的任何容器的存活期都长，在容器重新启动时数据也会得到保留。 当然，当一个 Pod 不再存在时，Volume也将不再存在。更重要的是，Kubernetes 可以支持许多类型的Volume卷，Pod 也能同时使用任意数量的Volume卷。

使用卷时，Pod 声明中需要提供卷的类型 (.spec.volumes 字段)和卷挂载的位置 (.spec.containers.volumeMounts 字段).



# 二、Volume类型

Kubernetes 支持下列类型的卷：

```yaml
awsElasticBlockStore
azureDisk
azureFile
cephfs
cinder
configMap
csi
downwardAPI
emptyDir
fc (fibre channel)
flexVolume
flocker
gcePersistentDisk
gitRepo (deprecated)
glusterfs
hostPath
iscsi
local
nfs
persistentVolumeClaim
projected
portworxVolume
quobyte
rbd
scaleIO
secret
storageos
vsphereVolume
```

K8S上支持的存储接口还是很多，每一个存储接口都是一种类型；对于这些存储类型我们大致可以分为云存储，分布式存储，网络存储、临时存储，节点本地存储，特殊类型存储、用户自定义存储等等；比如awsElasticBlockStore、azureDisk、azureFile、gcePersistentDisk、vshperVolume、cinder这些类型划分为云存储；cephfs、glusterfs、rbd这些划分为分布式存储；nfs、iscsi、fc这些划分为网络存储；enptyDIR划分为临时存储；hostPath、local划分为本地存储；自定义存储csi；特殊存储configMap、secret、downwardAPId；持久卷申请persistentVolumeClaim等等；

# 三、mptyDir卷

当 Pod 指定到某个节点上时，首先创建的是一个 emptyDir 卷，并且只要 Pod 在该节点上运行，卷就一直存在。就像它的名称表示的那样，卷最初是空的。

尽管 Pod 中每个容器挂载 emptyDir 卷的路径可能相同也可能不同，但是这些容器都可以读写 emptyDir 卷中相同的文件。

如果Pod中有多个容器，其中某个容器重启，不会影响emptyDir 卷中的数据。当 Pod 因为某些原因被删除时，emptyDir 卷中的数据也会永久删除。

注意：容器崩溃并不会导致 Pod 被从节点上移除，因此容器崩溃时 emptyDir 卷中的数据是安全的。

 

## 3.1 emptyDir的一些用途

- 缓存空间，例如基于磁盘的归并排序
- 为耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行
- 在 Web 服务器容器服务数据时，保存内容管理器容器获取的文件

 

## 3.2 emptyDir示例

yaml文件

```yaml
[root@k8s-master emptydir]# pwd
/root/k8s_practice/emptydir
[root@k8s-master emptydir]# cat pod_emptydir.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir
  namespace: default
spec:
  containers:
  - name: myapp-pod
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  - name: busybox-pod
    image: registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - mountPath: /test/cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

启动pod，并查看状态

```bash
[root@k8s-master emptydir]# kubectl apply -f pod_emptydir.yaml
pod/pod-emptydir created
[root@k8s-master emptydir]#
[root@k8s-master emptydir]# kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-emptydir   2/2     Running   0          10s   10.244.2.166   k8s-node02   <none>           <none>
[root@k8s-master emptydir]#
[root@k8s-master emptydir]# kubectl describe pod pod-emptydir
Name:         pod-emptydir
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Fri, 12 Jun 2020 22:49:11 +0800
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"pod-emptydir","namespace":"default"},"spec":{"containers":[{"image":"...
Status:       Running
IP:           10.244.2.166
IPs:
  IP:  10.244.2.166
Containers:
  myapp-pod:
    Container ID:   docker://d45663776b40a24e7cfc3cf46cb08cf3ed6b98b023a5d2cb5f42bee2234c7338
    Image:          registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    Image ID:       docker-pullable://10.0.0.110:5000/k8s-secret/myapp@sha256:9eeca44ba2d410e54fccc54cbe9c021802aa8b9836a0bcf3d3229354e4c8870e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 12 Jun 2020 22:49:12 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /cache from cache-volume (rw)  ##### 挂载信息
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
  busybox-pod:
    Container ID:  docker://c2917ba30c3322fb0caead5d97476b341e691f9fb1990091264364b8cd340512
    Image:         registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    Image ID:      docker-pullable://registry.cn-beijing.aliyuncs.com/ducafe/busybox@sha256:f73ae051fae52945d92ee20d62c315306c593c59a429ccbbdcba4a488ee12269
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      sleep 3600
    State:          Running
      Started:      Fri, 12 Jun 2020 22:49:12 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /test/cache from cache-volume (rw)  ##### 挂载信息
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  cache-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                 Message
  ----    ------     ----  ----                 -------
  Normal  Scheduled  3s    default-scheduler    Successfully assigned default/pod-emptydir to k8s-node02
  Normal  Pulled     2s    kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1" already present on machine
  Normal  Created    2s    kubelet, k8s-node02  Created container myapp-pod
  Normal  Started    2s    kubelet, k8s-node02  Started container myapp-pod
  Normal  Pulled     2s    kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24" already present on machine
  Normal  Created    2s    kubelet, k8s-node02  Created container busybox-pod
  Normal  Started    2s    kubelet, k8s-node02  Started container busybox-pod
```

## 3.3 emptyDir验证

在pod中的myapp-pod容器内操作

```bash
[root@k8s-master emptydir]# kubectl exec -it pod-emptydir -c myapp-pod -- sh
/ # cd /cache
/cache #
/cache # pwd
/cache
/cache #
/cache # date >> data.info
/cache # ls -l
total 4
-rw-r--r--    1 root     root            29 Jun 12 14:53 data.info
/cache # cat data.info
Fri Jun 12 14:53:27 UTC 2020
```

在pod中的busybox-pod容器内操作

```bash
[root@k8s-master emptydir]# kubectl exec -it pod-emptydir -c busybox-pod -- sh
/ # cd /test/cache
/test/cache # ls -l
total 4
-rw-r--r--    1 root     root            29 Jun 12 14:53 data.info
/test/cache # cat data.info
Fri Jun 12 14:53:27 UTC 2020
/test/cache #
/test/cache # echo "===" >> data.info
/test/cache # date >> data.info
/test/cache # cat data.info
Fri Jun 12 14:53:27 UTC 2020
===
Fri Jun 12 14:56:05 UTC 2020
```

由上可见，一个Pod中多个容器可共享同一个emptyDir卷。

 

# 四、hostPath卷

hostPath 卷能将主机node节点文件系统上的文件或目录挂载到你的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

 

## 4.1 hostPath 的一些用法有

- 运行一个需要访问 Docker 引擎内部机制的容器；请使用 hostPath 挂载 /var/lib/docker 路径。
- 在容器中运行 cAdvisor 时，以 hostPath 方式挂载 /sys。
- 允许 Pod 指定给定的 hostPath 在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。

## 4.2 支持类型

除了必需的 path 属性之外，用户可以选择性地为 hostPath 卷指定 type。支持的 type 值如下：

| 取值              | 行为                                                         |
| ----------------- | ------------------------------------------------------------ |
|                   | 空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查 |
| DirectoryOrCreate | 如果指定的路径不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 Kubelet 相同的组和所有权 |
| Directory         | 给定的路径必须存在                                           |
| FileOrCreate      | 如果给定路径的文件不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 Kubelet 相同的组和所有权【前提：文件所在目录必须存在；目录不存在则不能创建文件】 |
| File              | 给定路径上的文件必须存在                                     |
| Socket            | 在给定路径上必须存在的 UNIX 套接字                           |
| CharDevice        | 在给定路径上必须存在的字符设备                               |
| BlockDevice       | 在给定路径上必须存在的块设备                                 |



 

## 4.3 注意事项

当使用这种类型的卷时要小心，因为：

- 具有相同配置（例如从 podTemplate 创建）的多个 Pod 会由于节点上文件的不同而在不同节点上有不同的行为。
- 当 Kubernetes 按照计划添加资源感知的调度时，这类调度机制将无法考虑由 hostPath 卷使用的资源。
- 基础主机上创建的文件或目录只能由 root 用户写入。需要在 特权容器 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 hostPath 卷。

 

## 4.4 hostPath示例

yaml文件

```yaml
[root@k8s-master hostpath]# pwd
/root/k8s_practice/hostpath
[root@k8s-master hostpath]# cat pod_hostpath.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
  namespace: default
spec:
  containers:
  - name: myapp-pod
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: hostpath-dir-volume
      mountPath: /test-k8s/hostpath-dir
    - name: hostpath-file-volume
      mountPath: /test/hostpath-file/test.conf
  volumes:
  - name: hostpath-dir-volume
    hostPath:
      # 宿主机目录
      path: /k8s/hostpath-dir
      # hostPath 卷指定 type，如果目录不存在则创建(可创建多层目录)
      type: DirectoryOrCreate
  - name: hostpath-file-volume
    hostPath:
      path: /k8s2/hostpath-file/test.conf
      # 如果文件不存在则创建。 前提：文件所在目录必须存在  目录不存在则不能创建文件
      type: FileOrCreate
```

启动pod，并查看状态

```yaml
[root@k8s-master hostpath]# kubectl apply -f pod_hostpath.yaml
pod/pod-hostpath created
[root@k8s-master hostpath]#
[root@k8s-master hostpath]# kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-hostpath   1/1     Running   0          17s   10.244.4.133   k8s-node01   <none>           <none>
[root@k8s-master hostpath]#
[root@k8s-master hostpath]# kubectl describe pod pod-hostpath
Name:         pod-hostpath
Namespace:    default
Priority:     0
Node:         k8s-node01/172.16.1.111
Start Time:   Sat, 13 Jun 2020 16:12:15 +0800
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"pod-hostpath","namespace":"default"},"spec":{"containers":[{"image":"...
Status:       Running
IP:           10.244.4.133
IPs:
  IP:  10.244.4.133
Containers:
  myapp-pod:
    Container ID:   docker://8cc87217fb483288067fb6d227c46aa890d02f75cae85c6d110646839435ab96
    Image:          registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    Image ID:       docker-pullable://registry.cn-beijing.aliyuncs.com/google_registry/myapp@sha256:9eeca44ba2d410e54fccc54cbe9c021802aa8b9836a0bcf3d3229354e4c8870e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 13 Jun 2020 16:12:17 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /test-k8s/hostpath-dir from hostpath-dir-volume (rw)
      /test/hostpath-file/test.conf from hostpath-file-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  hostpath-dir-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /k8s/hostpath-dir
    HostPathType:  DirectoryOrCreate
  hostpath-file-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /k8s2/hostpath-file/test.conf
    HostPathType:  FileOrCreate
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                 Message
  ----    ------     ----       ----                 -------
  Normal  Scheduled  <unknown>  default-scheduler    Successfully assigned default/pod-hostpath to k8s-node01
  Normal  Pulled     12m        kubelet, k8s-node01  Container image "registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1" already present on machine
  Normal  Created    12m        kubelet, k8s-node01  Created container myapp-pod
  Normal  Started    12m        kubelet, k8s-node01  Started container myapp-pod
```

## 4.5 hostPath验证

**宿主机操作**

根据pod，在k8s-node01节点宿主机操作【因为Pod分配到了该节点】

```bash
# 对挂载的目录操作
[root@k8s-node01 hostpath-dir]# pwd
/k8s/hostpath-dir
[root@k8s-node01 hostpath-dir]# echo "dir" >> info
[root@k8s-node01 hostpath-dir]# date >> info
[root@k8s-node01 hostpath-dir]# cat info
dir
Sat Jun 13 16:22:37 CST 2020
# 对挂载的文件操作
[root@k8s-node01 hostpath-file]# pwd
/k8s2/hostpath-file
[root@k8s-node01 hostpath-file]# echo "file" >> test.conf
[root@k8s-node01 hostpath-file]# date >> test.conf
[root@k8s-node01 hostpath-file]#
[root@k8s-node01 hostpath-file]# cat test.conf
file
Sat Jun 13 16:23:05 CST 2020
```



**在Pod 容器中操作**

```bash
# 进入pod 中的指定容器【如果只有一个容器，那么可以不指定容器】
[root@k8s-master hostpath]# kubectl exec -it pod-hostpath -c myapp-pod -- /bin/sh
##### 对挂载的目录操作
/ # cd /test-k8s/hostpath-dir
/test-k8s/hostpath-dir # ls -l
total 4
-rw-r--r--    1 root     root            33 Jun 13 08:22 info
/test-k8s/hostpath-dir # cat info
dir
Sat Jun 13 16:22:37 CST 2020
/test-k8s/hostpath-dir #
/test-k8s/hostpath-dir # date >> info
/test-k8s/hostpath-dir # cat info
dir
Sat Jun 13 16:22:37 CST 2020
Sat Jun 13 08:26:10 UTC 2020
##### 对挂载的文件操作
# cd /test/hostpath-file/
/test/hostpath-file # cat test.conf
file
Sat Jun 13 16:23:05 CST 2020
/test/hostpath-file # echo "file====" >> test.conf
/test/hostpath-file # cat test.conf
file
Sat Jun 13 16:23:05 CST 2020
file====
```



# 五、网络文件系统nfs

`yml`文件中配置如下

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: goserver
spec:
  replicas: 2
  template:
    metadata:
      labels:
        run: goserver
    spec:
      containers:
      - name: goserver
        image: registry.cn-hangzhou.aliyuncs.com/magina-centos7/goserver:1.0
        ports:
        - containerPort: 4040
        volumeMounts:
        - mountPath: /mnt/logs
          name: go-logs
  volumes:
  - name: go-log
    nfs:
      server: nfs4.yinnote.com
      path: /prod/logs/goserver
```

> 这里使用了nfs标签，也就是将当前目录挂载到了远程文件系统，这里的server指的是远程文件系统路径，需要自己去配置，或者直接买其他云服务厂商的文件系统，这样做的好处是，不管哪个节点，哪个pod，都可以将日志打到统一的地方

另外，如果我们使用了nfs文件系统，必须要在每台节点上面安装nfs-utils工具包，否则pod会无法启动

> yum install nfs-utils