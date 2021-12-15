[Kubernetes K8S之存储PV-PVC详解](https://www.cnblogs.com/zhanglianghhh/p/13861817.html)

# 概述

与管理计算实例相比，管理存储是一个明显的问题。PersistentVolume子系统为用户和管理员提供了一个API，该API从如何使用存储中抽象出如何提供存储的详细信息。为此，我们引入了两个新的API资源：PersistentVolume和PersistentVolumeClaim。

 

## PV概述

PersistentVolume (PV)是集群中由管理员提供或使用存储类动态提供的一块存储。它是集群中的资源，就像节点是集群资源一样。

PV是与Volumes类似的卷插件，但其生命周期与使用PV的任何单个Pod无关。由此API对象捕获存储的实现细节，不管是NFS、iSCSI还是特定于云提供商的存储系统。

 

## PVC概述

PersistentVolumeClaim (PVC) 是用户对存储的请求。它类似于Pod；Pods消耗节点资源，而PVC消耗PV资源。Pods可以请求特定级别的资源(CPU和内存)。Claim可以请求特定的存储大小和访问模式(例如，它们可以挂载一次读写或多次只读)。

虽然PersistentVolumeClaims (PVC) 允许用户使用抽象的存储资源，但是用户通常需要具有不同属性(比如性能)的PersistentVolumes (PV) 来解决不同的问题。集群管理员需要能够提供各种不同的PersistentVolumes，这些卷在大小和访问模式之外还有很多不同之处，也不向用户公开这些卷是如何实现的细节。对于这些需求，有一个StorageClass资源。

 

# volume 和 claim的生命周期

PV是集群中的资源。PVC是对这些资源的请求，并且还充当对资源的声明检查。PV和PVC之间的交互遵循以下生命周期：

 

## 供应

有两种方式配置PV：静态的或动态的。

 

**静态配置**

集群管理员创建一些PV。它们带有可供集群用户使用的实际存储的详细信息。存在于Kubernetes API中，可供使用。

 

**动态配置**

当管理员创建的静态PV没有一个与用户的PersistentVolumeClaim匹配时，集群可能会尝试动态地为PVC提供一个卷。此配置基于StorageClasses：PVC必须请求存储类，并且管理员必须已经创建并配置了该类，才能进行动态配置。声明该类为 `""`，可以有效地禁用其动态配置。

要启用基于存储级别的动态存储配置，集群管理员需要启用API Server上的DefaultStorageClass[准入控制器]。例如，通过确保DefaultStorageClass位于API Server组件的 `--enable-admission-plugins`标志，使用逗号分隔的有序值列表中，可以完成此操作。

 

## 绑定

用户创建(或者在动态配置的情况下，已经创建)具有特定存储请求量(大小)和特定访问模式的PersistentVolumeClaim。主控制器中的控制循环监视新的PV，找到匹配的PV(如果可能的话)，并将它们绑定在一起。如果PV为新的PVC动态配置，那么循环始终将该PV绑定到PVC。否则，用户始终至少得到他们所要求的，但是存储量可能会超过所要求的范围。

一旦绑定，无论是如何绑定的，PersistentVolumeClaim绑定都是互斥的。PVC到PV的绑定是一对一的映射，使用ClaimRef，它是PersistentVolume和PersistentVolumeClaim之间的双向绑定。

如果不存在匹配的卷，声明(Claims)将无限期保持未绑定。随着匹配量的增加，声明将受到约束。例如，配备有许多50Gi PV的群集将与请求100Gi的PVC不匹配。当将100Gi PV添加到群集时，可以绑定PVC。

注意：静态时PVC与PV绑定时会根据storageClassName（存储类名称）和accessModes（访问模式）判断哪些PV符合绑定需求。然后再根据存储量大小判断，首先存PV储量必须大于或等于PVC声明量；其次就是PV存储量越接近PVC声明量，那么优先级就越高（PV量越小优先级越高）。

 

## 使用

Pods使用声明(claims)作为卷。集群检查声明以找到绑定卷并为Pod挂载该卷。对于支持多种访问模式的卷，用户在其声明中作为Pod中卷使用时指定所需的模式。

一旦用户拥有一个声明并且该声明被绑定，则绑定的PV就属于该用户。用户通过在Pod的卷块中包含的persistentVolumeClaim部分来调度Pods并访问其声明的PV。

 

## 持久化声明保护

“使用中的存储对象保护” ：该功能的目的是确保在Pod活动时使用的PersistentVolumeClaims (PVC)和绑定到PVC的PersistentVolume (PV)不会从系统中删除，因为这可能会导致数据丢失。

如果用户删除了Pod正在使用的PVC，则不会立即删除该PVC；PVC的清除被推迟，直到任何Pod不再主动使用PVC。另外，如果管理员删除绑定到PVC的PV，则不会立即删除该PV；PV的去除被推迟，直到PV不再与PVC结合。

 

## 回收策略

当用户处理完他们的卷时，他们可以从允许回收资源的API中删除PVC对象。PersistentVolume的回收策略告诉集群在释放卷的声明后该如何处理它。目前，卷可以被保留、回收或删除。

 

**Retain (保留)**

保留回收策略允许手动回收资源。当PersistentVolumeClaim被删除时，PersistentVolume仍然存在，并且该卷被认为是“释放”的。但是，由于之前声明的数据仍然存在，因此另一个声明尚无法得到。管理员可以手动回收卷。

 

**Delete (删除)**

对于支持Delete回收策略的卷插件，删除操作会同时从Kubernetes中删除PersistentVolume对象以及外部基础架构中的关联存储资产，例如AWS EBS，GCE PD，Azure Disk或Cinder卷。动态配置的卷将继承其StorageClass的回收策略，默认为Delete。管理员应根据用户的期望配置StorageClass。

 

**Recycle (回收)**

如果基础卷插件支持，Recycle回收策略将rm -rf /thevolume/*对该卷执行基本的擦除并使其可用于新的声明。

 

## Persistent Volumes类型

PersistentVolume类型作为插件实现。Kubernetes当前支持以下插件：

```
GCEPersistentDisk
AWSElasticBlockStore
AzureFile
AzureDisk
CSI
FC (Fibre Channel)
FlexVolume
Flocker
NFS
iSCSI
RBD (Ceph Block Device)
CephFS
Cinder (OpenStack block storage)
Glusterfs
VsphereVolume
Quobyte Volumes
HostPath (仅用于单节点测试——本地存储不受任何方式的支持，也不能在多节点集群中工作)
Portworx Volumes
ScaleIO Volumes
StorageOS
```



# PV示例与参数说明

## PV示例

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

**Capacity：**通常，PV将具有特定的存储容量设置。当前，存储大小是可以设置或请求的唯一资源。将来的属性可能包括IOPS，吞吐量等。

**volumeMode：**可选参数，为Filesystem或Block。Filesystem是volumeMode省略参数时使用的默认模式。

**accessModes：**PersistentVolume可以通过资源提供者支持的任何方式安装在主机上。如下文表中所示，提供商将具有不同的功能，并且每个PV的访问模式都将设置为该特定卷支持的特定模式。例如，NFS可以支持多个读/写客户端，但是特定的NFS PV可能以只读方式在服务器上导出。每个PV都有自己的一组访问模式，用于描述该特定PV的功能。

访问方式为：

```
ReadWriteOnce-该卷可以被单个节点以读写方式挂载
ReadOnlyMany-该卷可以被许多节点以只读方式挂载
ReadWriteMany-该卷可以被多个节点以读写方式挂载
```

 

在CLI命令行中，访问模式缩写为：

```
RWO-ReadWriteOnce
ROX-ReadOnlyMany
RWX-ReadWriteMany
```

说明：一个卷一次只能使用一种访问模式挂载，即使它支持多种访问模式。

**storageClassName：**PV可以有一个类，通过将storageClassName属性设置为一个StorageClass的名称来指定这个类。特定类的PV只能绑定到请求该类的PVC。没有storageClassName的PV没有类，只能绑定到不请求特定类的PVC。

**persistentVolumeReclaimPolicy：**当前的回收政策是：Retain (保留)-手动回收、Recycle (回收)-基本擦除（rm -rf /thevolume/*）、Delete (删除)-删除相关的存储资产 (例如AWS EBS，GCE PD，Azure Disk或OpenStack Cinder卷)。

备注：当前，仅NFS和HostPath支持回收。AWS EBS，GCE PD，Azure Disk和Cinder卷支持删除。

 

## PV卷状态

卷将处于以下某种状态：

- Available：尚未绑定到声明(claim)的空闲资源
- Bound：卷已被声明绑定
- Released：声明已被删除，但群集尚未回收该资源
- Failed：该卷自动回收失败

CLI将显示绑定到PV的PVC的名称。

 

## PV类型与支持的访问模式

| Volume Plugin        | ReadWriteOnce         | ReadOnlyMany          | ReadWriteMany                      |
| -------------------- | --------------------- | --------------------- | ---------------------------------- |
| AWSElasticBlockStore | ✓                     | -                     | -                                  |
| AzureFile            | ✓                     | ✓                     | ✓                                  |
| AzureDisk            | ✓                     | -                     | -                                  |
| CephFS               | ✓                     | ✓                     | ✓                                  |
| Cinder               | ✓                     | -                     | -                                  |
| CSI                  | depends on the driver | depends on the driver | depends on the driver              |
| FC                   | ✓                     | ✓                     | -                                  |
| FlexVolume           | ✓                     | ✓                     | depends on the driver              |
| Flocker              | ✓                     | -                     | -                                  |
| GCEPersistentDisk    | ✓                     | ✓                     | -                                  |
| Glusterfs            | ✓                     | ✓                     | ✓                                  |
| HostPath             | ✓                     | -                     | -                                  |
| iSCSI                | ✓                     | ✓                     | -                                  |
| Quobyte              | ✓                     | ✓                     | ✓                                  |
| NFS                  | ✓                     | ✓                     | ✓                                  |
| RBD                  | ✓                     | ✓                     | -                                  |
| VsphereVolume        | ✓                     | -                     | - (works when Pods are collocated) |
| PortworxVolume       | ✓                     | -                     | ✓                                  |
| ScaleIO              | ✓                     | ✓                     | -                                  |
| StorageOS            | ✓                     | -                     | -                                  |

 

# PV-PVC示例

## 主机信息

| 服务器名称(hostname) | 系统版本  | 配置      | 内网IP       | 外网IP(模拟) | 部署模块   |
| -------------------- | --------- | --------- | ------------ | ------------ | ---------- |
| k8s-master           | CentOS7.7 | 2C/4G/20G | 172.16.1.110 | 10.0.0.110   | k8s-master |
| k8s-node01           | CentOS7.7 | 2C/4G/20G | 172.16.1.111 | 10.0.0.111   | k8s-node   |
| k8s-node02           | CentOS7.7 | 2C/4G/20G | 172.16.1.112 | 10.0.0.112   | k8s-node   |
| k8s-node03           | CentOS7.7 | 2C/2G/20G | 172.16.1.113 | 10.0.0.113   | NFS        |

存储使用NFS，在k8s-node03机器仅部署NFS服务，没有部署K8S

 

## NFS服务部署

文章参考：「[NFS 服务搭建与配置](https://www.cnblogs.com/zhanglianghhh/p/9230045.html)」

所有机器操作

```bash
# 所需安装包
yum install nfs-utils rpcbind -y
```

 

NFS服务端k8s-node03机器操作

```bash
[root@k8s-node03 ~]# mkdir -p /data/nfs1 /data/nfs2 /data/nfs3 /data/nfs4 /data/nfs5 /data/nfs6
[root@k8s-node03 ~]# chown -R nfsnobody.nfsnobody /data/
[root@k8s-node03 ~]#
[root@k8s-node03 ~]# ll /data/
total 0
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jun 14 16:30 nfs1
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jun 14 16:30 nfs2
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jun 14 16:30 nfs3
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jun 14 16:30 nfs4
drwxr-xr-x 2 nfsnobody nfsnobody 6 Jun 14 16:30 nfs5
drwxr-xr-x 2 nfsnobody nfsnobody 6 Aug 22 16:25 nfs6
[root@k8s-node03 ~]# vim /etc/exports
/data/nfs1  172.16.1.0/24(rw,sync,root_squash,all_squash)
/data/nfs2  172.16.1.0/24(rw,sync,root_squash,all_squash)
/data/nfs3  172.16.1.0/24(rw,sync,root_squash,all_squash)
/data/nfs4  172.16.1.0/24(rw,sync,root_squash,all_squash)
/data/nfs5  172.16.1.0/24(rw,sync,root_squash,all_squash)
/data/nfs6  172.16.1.0/24(rw,sync,root_squash,all_squash)
### 启动NFS服务
[root@k8s-node03 ~]# systemctl start rpcbind.service
[root@k8s-node03 ~]# systemctl start nfs.service
### 检查NFS服务 ， 其中 172.16.1.113 为服务端IP
[root@k8s-node03 ~]# showmount -e 172.16.1.113
Export list for 172.16.1.113:
/data/nfs6 172.16.1.0/24
/data/nfs5 172.16.1.0/24
/data/nfs4 172.16.1.0/24
/data/nfs3 172.16.1.0/24
/data/nfs2 172.16.1.0/24
/data/nfs1 172.16.1.0/24
```

**NFS客户端验证**

在k8s-node02机器验证

```bash
# 查看rpcbind服务，默认是启动的，如果没有启动则启动并加入开机自启动
[root@k8s-node02 ~]# systemctl status rpcbind.service
# 查看NFS服务信息
[root@k8s-node02 ~]# showmount -e 172.16.1.113
………………
# 挂载，并进行读写验证
[root@k8s-node02 ~]# mount -t nfs 172.16.1.113:/data/nfs1 /mnt
# 验证完毕，去掉NFS挂载
[root@k8s-node02 ~]# umount -lf 172.16.1.113:/data/nfs1
```

## PV部署

yaml文件

```yaml
[root@k8s-master pv-pvc]# pwd
/root/k8s_practice/pv-pvc
[root@k8s-master pv-pvc]# cat pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/nfs1
    server: 172.16.1.113
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs2
spec:
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/nfs2
    server: 172.16.1.113
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs3
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  nfs:
    path: /data/nfs3
    server: 172.16.1.113
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs4
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/nfs4
    server: 172.16.1.113
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs5
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/nfs5
    server: 172.16.1.113
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs6
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /data/nfs6
    server: 172.16.1.113
```

启动PV，并查看状态

```bash
[root@k8s-master pv-pvc]# kubectl apply -f pv.yaml
persistentvolume/pv-nfs1 created
persistentvolume/pv-nfs2 created
persistentvolume/pv-nfs3 created
persistentvolume/pv-nfs4 created
persistentvolume/pv-nfs5 created
persistentvolume/pv-nfs6 created
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pv -o wide
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE   VOLUMEMODE
pv-nfs1   1Gi        RWO            Recycle          Available           nfs                     11s   Filesystem
pv-nfs2   3Gi        RWO            Recycle          Available           nfs                     11s   Filesystem
pv-nfs3   5Gi        RWO            Recycle          Available           slow                    11s   Filesystem
pv-nfs4   10Gi       RWO            Recycle          Available           nfs                     11s   Filesystem
pv-nfs5   5Gi        RWX            Recycle          Available           nfs                     11s   Filesystem
pv-nfs6   5Gi        RWO            Recycle          Available           nfs                     11s   Filesystem
```

## StatefulSet创建并使用PVC

StatefulSet 需要 headless 服务 来负责 Pod 的网络标识，因此需要负责创建此服务。

yaml文件

```yaml
[root@k8s-master pv-pvc]# pwd
/root/k8s_practice/pv-pvc
[root@k8s-master pv-pvc]# cat sts-pod-pvc.yaml
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
      terminationGracePeriodSeconds: 100
      containers:
      - name: nginx
        image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "nfs"
      resources:
        requests:
          storage: 3Gi
```

启动pod并查看状态

```bash
[root@k8s-master pv-pvc]# kubectl apply -f sts-pod-pvc.yaml
service/nginx created
statefulset.apps/web created
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   24d   <none>
nginx        ClusterIP   None         <none>        80/TCP    17s   app=nginx
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get sts -o wide
NAME   READY   AGE   CONTAINERS   IMAGES
web    3/3     82m   nginx        registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pod -o wide
NAME    READY   STATUS              RESTARTS   AGE   IP       NODE         NOMINATED NODE   READINESS GATES
web-0   0/1     ContainerCreating   0          3s    <none>   k8s-node01   <none>           <none>
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pod -o wide
NAME    READY   STATUS              RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
web-0   1/1     Running             0          11s   10.244.4.135   k8s-node01   <none>           <none>
web-1   1/1     Running             0          6s    10.244.2.171   k8s-node02   <none>           <none>
web-2   0/1     ContainerCreating   0          3s    <none>         k8s-node01   <none>           <none>
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP             NODE         NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          8m23s   10.244.2.174   k8s-node02   <none>           <none>
web-1   1/1     Running   0          8m20s   10.244.4.139   k8s-node01   <none>           <none>
web-2   1/1     Running   0          8m17s   10.244.2.175   k8s-node02   <none>           <none>
```

## PV和PVC状态信息查看

```bash
### 注意挂载顺序
[root@k8s-master pv-pvc]# kubectl get pv -o wide
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE    VOLUMEMODE
pv-nfs1   1Gi        RWO            Recycle          Available                       nfs                     116s   Filesystem
pv-nfs2   3Gi        RWO            Recycle          Bound       default/www-web-0   nfs                     116s   Filesystem
pv-nfs3   5Gi        RWO            Recycle          Available                       slow                    116s   Filesystem
pv-nfs4   10Gi       RWO            Recycle          Bound       default/www-web-2   nfs                     116s   Filesystem
pv-nfs5   5Gi        RWX            Recycle          Available                       nfs                     116s   Filesystem
pv-nfs6   5Gi        RWO            Recycle          Bound       default/www-web-1   nfs                     116s   Filesystem
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pvc -o wide
NAME        STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
www-web-0   Bound    pv-nfs2   3Gi        RWO            nfs            87s   Filesystem
www-web-1   Bound    pv-nfs6   5Gi        RWO            nfs            84s   Filesystem
www-web-2   Bound    pv-nfs4   10Gi       RWO            nfs            82s   Filesystem
```

PVC与PV绑定时会根据storageClassName（存储类名称）和accessModes（访问模式）判断哪些PV符合绑定需求。然后再根据存储量大小判断，首先存PV储量必须大于或等于PVC声明量；其次就是PV存储量越接近PVC声明量，那么优先级就越高（PV量越小优先级越高）。

 

## curl访问验证

在NFS服务端k8s-node03（172.16.1.113）对应NFS共享目录创建文件

```
echo "pv-nfs2===" > /data/nfs2/index.html
echo "pv-nfs4+++" > /data/nfs4/index.html
echo "pv-nfs6---" > /data/nfs6/index.html
```

 

curl访问pod

```bash
[root@k8s-master pv-pvc]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP             NODE         NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          8m23s   10.244.2.174   k8s-node02   <none>           <none>
web-1   1/1     Running   0          8m20s   10.244.4.139   k8s-node01   <none>           <none>
web-2   1/1     Running   0          8m17s   10.244.2.175   k8s-node02   <none>           <none>
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# curl 10.244.2.174
pv-nfs2===
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# curl 10.244.4.139
pv-nfs6---
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# curl 10.244.2.175
pv-nfs4+++
```

即使删除其中一个pod，pod被拉起来后也能正常访问。

 

## 删除sts并回收PV

删除statefulset

```bash
[root@k8s-master pv-pvc]# kubectl delete -f sts-pod-pvc.yaml
service "nginx" deleted
statefulset.apps "web" deleted
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pod -o wide
No resources found in default namespace.
```

查看PVC和PV，并删除PVC

```bash
[root@k8s-master pv-pvc]# kubectl get pvc -o wide
NAME        STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
www-web-0   Bound    pv-nfs2   3Gi        RWO            nfs            24m   Filesystem
www-web-1   Bound    pv-nfs6   5Gi        RWO            nfs            24m   Filesystem
www-web-2   Bound    pv-nfs4   10Gi       RWO            nfs            24m   Filesystem
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl get pv -o wide
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE   VOLUMEMODE
pv-nfs1   1Gi        RWO            Recycle          Available                       nfs                     26m   Filesystem
pv-nfs2   3Gi        RWO            Recycle          Bound       default/www-web-0   nfs                     26m   Filesystem
pv-nfs3   5Gi        RWO            Recycle          Available                       slow                    26m   Filesystem
pv-nfs4   10Gi       RWO            Recycle          Bound       default/www-web-2   nfs                     26m   Filesystem
pv-nfs5   5Gi        RWX            Recycle          Available                       nfs                     26m   Filesystem
pv-nfs6   5Gi        RWO            Recycle          Bound       default/www-web-1   nfs                     26m   Filesystem
[root@k8s-master pv-pvc]#
[root@k8s-master pv-pvc]# kubectl delete pvc www-web-0 www-web-1 www-web-2
persistentvolumeclaim "www-web-0" deleted
persistentvolumeclaim "www-web-1" deleted
persistentvolumeclaim "www-web-2" deleted
```

回收PV

```yaml
### 由下可见，还有一个pv虽然声明被删除，但资源尚未回收；我们只需等一会儿即可
[root@k8s-master pv-pvc]# kubectl get pv -o wide
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE   VOLUMEMODE
pv-nfs1   1Gi        RWO            Recycle          Available                       nfs                     90m   Filesystem
pv-nfs2   3Gi        RWO            Recycle          Available                       nfs                     90m   Filesystem
pv-nfs3   5Gi        RWO            Recycle          Available                       slow                    90m   Filesystem
pv-nfs4   10Gi       RWO            Recycle          Available                       nfs                     90m   Filesystem
pv-nfs5   5Gi        RWX            Recycle          Available                       nfs                     90m   Filesystem
pv-nfs6   5Gi        RWO            Recycle          Released    default/www-web-1   nfs                     90m   Filesystem
[root@k8s-master pv-pvc]#
### 可见该pv还有引用
[root@k8s-master pv-pvc]# kubectl get pv pv-nfs6 -o yaml
apiVersion: v1
kind: PersistentVolume
metadata:
………………
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  ################### 可见仍然在被使用
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: www-web-1
    namespace: default
    resourceVersion: "1179810"
    uid: d4d8943c-6b16-45a5-8ffc-691fcefc4f88
  ###################
  nfs:
    path: /data/nfs6
    server: 172.16.1.113
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  volumeMode: Filesystem
status:
  phase: Released
```

在NFS服务端查看结果如下，可见/data/nfs6资源尚未回收，而/data/nfs2/、/data/nfs4/资源已经被回收。

```bash
[root@k8s-node03 ~]# tree /data/
/data/
├── nfs1
├── nfs2
├── nfs3
├── nfs4
├── nfs5
└── nfs6
    └── index.html

5 directories, 2 files
```

针对这种情况有两种处理方式：

1、我们什么也不用做，等一会儿集群就能回收该资源

2、我们进行手动回收，操作如下

**手动回收资源**

```bash
[root@k8s-master pv-pvc]# kubectl edit pv pv-nfs6
### 去掉claimRef: 部分
### 再次查看pv信息
[root@k8s-master pv-pvc]# kubectl get pv -o wide
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE    VOLUMEMODE
pv-nfs1   1Gi        RWO            Recycle          Available           nfs                     108m   Filesystem
pv-nfs2   3Gi        RWO            Recycle          Available           nfs                     108m   Filesystem
pv-nfs3   5Gi        RWO            Recycle          Available           slow                    108m   Filesystem
pv-nfs4   10Gi       RWO            Recycle          Available           nfs                     108m   Filesystem
pv-nfs5   5Gi        RWX            Recycle          Available           nfs                     108m   Filesystem
pv-nfs6   5Gi        RWO            Recycle          Available           nfs                     108m   Filesystem
```

![img](https://img2020.cnblogs.com/blog/1395193/202010/1395193-20201023001556996-721328438.png)

 

之后到NFS服务端操作，清除该pv下的数据

```bash
[root@k8s-node03 ~]# rm -fr /data/nfs6/*
```

到此，手动回收资源操作成功！

 

# StatefulSet网络标识与PVC

1、匹配StatefulSet的Pod name(网络标识)的模式为：`$(statefulset名称)-$(序号)`，比如StatefulSet名称为web，副本数为3。则为：web-0、web-1、web-2

2、StatefulSet为每个Pod副本创建了一个DNS域名，这个域名的格式为：`$(podname).(headless service name)`，也就意味着服务之间是通过Pod域名来通信而非Pod IP。当Pod所在Node发生故障时，Pod会被漂移到其他Node上，Pod IP会发生改变，但Pod域名不会变化

3、StatefulSet使用Headless服务来控制Pod的域名，这个Headless服务域名的为：`$(service name).$(namespace).svc.cluster.local`，其中 cluster.local 指定的集群的域名

4、根据volumeClaimTemplates，为每个Pod创建一个PVC，PVC的命令规则为：`$(volumeClaimTemplates name)-$(pod name)`，比如volumeClaimTemplates为www，pod name为web-0、web-1、web-2；那么创建出来的PVC为：www-web-0、www-web-1、www-web-2

5、删除Pod不会删除对应的PVC，手动删除PVC将自动释放PV。