## 1. Volume

Docker 中也有一个 volume 的概念，尽管它稍微宽松一些，管理也很少。在 Docker 中，卷就像是磁盘或是另一个容器中的一个目录。它的生命周期不受管理，直到最近才有了 local-disk-backed 卷。Docker 现在提供了卷驱动程序，但是功能还非常有限（例如Docker1.7只允许每个容器使用一个卷驱动，并且无法给卷传递参数）。

另一方面，Kubernetes 中的卷有明确的寿命——与封装它的 Pod 相同。所f以，卷的生命比 Pod 中的所有容器都长，当这个容器重启时数据仍然得以保存。当然，当 Pod 不再存在时，卷也将不复存在。也许更重要的是，Kubernetes 支持多种类型的卷，Pod 可以同时使用任意数量的卷。

卷的核心是目录，可能还包含了一些数据，可以通过 pod 中的容器来访问。该目录是如何形成的、支持该目录的介质以及其内容取决于所使用的特定卷类型。要使用卷，需要为 pod 指定为卷（spec.volumes 字段）以及将它挂载到容器的位置（ spec.containers.volumeMounts 字段 ）。

容器中的进程看到的是由其 Docker 镜像和卷组成的文件系统视图。Docker 镜像位于文件系统层次结构的根目录，任何卷都被挂载在镜像的指定路径中。卷无法挂载到其他卷上或与其他卷有硬连接。Pod 中的每个容器都必须独立指定每个卷的挂载位置。

Kubernetes支持的存储卷类型很多，有公有云环境、有本地宿主机目录、有分布式文件系统等，具体可参考：https://jimmysong.io/kubernetes-handbook/concepts/volume.html，主要有以下几类：

- emptyDir：临时存储，一般用于多个container之间共享
- hostPath：使用宿主机目录
- gitRepo：在容器启动前，将git仓库中的内容拉取到本地并挂载到容器中
- 网络存储：

- - SAN
  - NAS
  - 分布式存储
  - 云存储

### 1.1. 模板

```
pod.spec
    containers
        volumeMounts                <[]Object>              # 挂载的卷和挂载点
            name                    <string> -required-     # 指定被挂载的卷名，对应 spec.volumes.name
            mountPath               <string> -required-     # 挂载点
            readOnly                <boolean>               # 是否只读，默认false
    volumes                         <[]Object>              # 指定存储卷
        name                        <string> -required-     # 存储卷名称
        configMap                   <Object>                # 配置类存储卷
            name                    <string>                # 引用的configmap名称
            defaultMode             <integer>               # 挂载到Pod中后，文件权限，如0444
            items                   <[]Object>              # 指定具体哪个key会被挂载，默认所有
                key                 <string> -required-     # configMap中的key
                path                <string> -required-     # 挂载路径，不可以使用 ..
                mode                <integer>               # 文件权限
            optional                <boolean>               # 当key不存在时是否报错，默认true
        emptyDir                    <Object>                # 空目录型存储卷，使用 emptyDir: {} 表使用默认值
            medium                  <string>                # 存储介质，支持Memory和空字符串(默认)
            sizeLimit               <string>                # 大小限制，默认不限制。一般在使用Memory时会限制
        hostPath                    <Object>                # 使用宿主机路径
            path                    <string> -required-     # 指定宿主机的路径，如果指定软连接，则使用软连接的目标路径
            type                    <string>                # 指定hostPath类型，默认为""
                # 具体类型可参考：https://kubernetes.io/docs/concepts/storage/volumes/#hostpath
        nfs                         <Object>                # 指定NFS存储类型
            server                  <string> -required-     # NFS服务器地址
            path                    <string> -required-     # 共享路径
            readOnly                <boolean>               # 是否只读，默认false
        persistentVolumeClaim       <Object>                # 指定PVC
            claimName               <string> -required-     # PVC对象名称，必须要和当前pod在同一个名称空间
            readOnly                <boolean>               # 是否只读，默认false
```

### 1.2. emptyDir

#### 1.2.1. 介绍

当 Pod 被分配给节点时，首先创建 emptyDir 卷，并且只要该 Pod 在该节点上运行，该卷就会存在。正如卷的名字所述，它最初是空的。Pod 中的容器可以读取和写入 emptyDir 卷中的相同文件，尽管该卷可以挂载到每个容器中的相同或不同路径上。当出于任何原因从节点中删除 Pod 时，emptyDir 中的数据将被永久删除。**容器崩溃不会从节点中移除 pod，因此 emptyDir 卷中的数据在容器崩溃时是安全的。**emptyDir 的用法有：

- 暂存空间，例如用于基于磁盘的合并排序
- 用作长时间计算崩溃恢复时的检查点
- Web服务器容器提供数据时，保存内容管理器容器提取的文件

#### 1.2.2. 案例

```
[root@hdss7-200 volume]# vim /data/k8s-yaml/base_resource/volume/emptydir.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: pod-emptydir
  namespace: app
  labels:
    tier: volume
    role: empty-dir
spec:
  containers:
  - name: main-container
    image: harbor.od.com/public/busybox:v1.31.1
    volumeMounts:
    - name: web-root
      mountPath: /tmp
    command:
    - httpd
    args:
    - -f
    - -h 
    - "/tmp/"
  - name: sidecar-container
    image: harbor.od.com/public/centos:7
    volumeMounts:
    - name: web-root
      mountPath: /tmp
    command:
    - /bin/bash
    args:
    - -c
    - "while :;do date +'%F %T' > /tmp/index.html;sleep 1;done"
  volumes:
  - name: web-root
    emptyDir: {}
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/emptydir.yaml
[root@hdss7-21 ~]# kubectl get pod pod-emptydir -n app -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
pod-emptydir   2/2     Running   0          71s   172.7.22.11   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# kubectl describe pod pod-emptydir -n app
Name:         pod-emptydir
Namespace:    app
...
Containers:
  main-container:
...
    Mounts:
      /tmp from web-root (rw)
...
  sidecar-container:
...
    Mounts:
      /tmp from web-root (rw)
...
Volumes:
  web-root:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>

[root@hdss7-21 ~]# while :;do curl -s 172.7.22.11;sleep 1;done
2020-01-26 12:31:09
2020-01-26 12:31:10
2020-01-26 12:31:11
2020-01-26 12:31:12
```

### 1.3. hostPath

使用宿主机目录作为存储卷去挂载到pod内部

```
[root@hdss7-200 volume]# vim /data/k8s-yaml/base_resource/volume/hostpath.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: pod-hostpath
  namespace: app
  labels:
    tier: volume
    role: hostpath
spec:
  containers:
  - name: main-container
    image: harbor.od.com/public/busybox:v1.31.1
    volumeMounts:
    - name: web-root
      mountPath: /data/web/html
    command:
    - httpd
    args:
    - -f
    - -h 
    - "/data/web/html"
  volumes:
  - name: web-root
    hostPath:
      path: /tmp/pod-hostpath/html
      type: DirectoryOrCreate
  nodeSelector:
    kubernetes.io/hostname: hdss7-21.host.com
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/hostpath.yaml
[root@hdss7-21 ~]# kubectl get pod pod-hostpath -o wide -n app
NAME           READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
pod-hostpath   1/1     Running   0          19s   172.7.21.11   hdss7-21.host.com   <none>           <none>
[root@hdss7-21 ~]# echo hello world > /tmp/pod-hostpath/html/index.html
[root@hdss7-21 ~]# curl -s 172.7.21.11
hello world
```

### 1.4. nfs

```
[root@hdss7-200 volume]# showmount -e  # 待挂载nfs磁盘
Export list for hdss7-200.host.com:
/tmp/data/volume 10.4.7.0/24
[root@hdss7-200 volume]# hostname > /tmp/data/volume/index.html
```



```
[root@hdss7-200 volume]# vim /data/k8s-yaml/base_resource/volume/nfs.yaml
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nfs-deploy
  namespace: app
  labels:
    tier: volume
    role: nfs
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: volume
      role: nfs
  template:
    metadata: 
      labels:
        tier: volume
        role: nfs
    spec:
      containers:
      - name: main-container
        image: harbor.od.com/public/busybox:v1.31.1
        volumeMounts:
        - name: web-root
          mountPath: /data/web/html
        command:
        - httpd
        args:
        - -f
        - -h 
        - "/data/web/html"
      volumes:
      - name: web-root
        nfs:
          server: hdss7-200
          path: /tmp/data/volume
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/nfs.yaml
deployment.apps/nfs-deploy created
[root@hdss7-21 ~]# kubectl get pods -n app -l role=nfs -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
nfs-deploy-8585f6765f-qbkzg   1/1     Running   0          25s   172.7.21.12   hdss7-21.host.com   <none>           <none>
nfs-deploy-8585f6765f-vhmhs   1/1     Running   0          25s   172.7.22.8    hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# curl -s 172.7.21.12; curl -s 172.7.22.8
hdss7-200.host.com
hdss7-200.host.com
```



## 2. PV/PVC

### 2.1. PV/PVC介绍

#### 2.1.1. 介绍

在数据持久化方面，采用Volume的传统方式管理能满足大部分场景，但是对于较大规模的集群，存储介质可能比较复杂，创建Pod的员工对存储设备了解程度不够深入，此时可以将存储介质和Pod分别定义为两种不同的资源，降低运维的难度。

PersistentVolume（PV）是由管理员设置的存储，它是群集的一部分。就像节点是集群中的资源一样，PV 也是集群中的资源。 PV 是 Volume 之类的卷插件，但具有独立于Pod 的生命周期。此 API 对象包含存储实现的细节，即 NFS、iSCSI 或特定于云供应商的存储系统。

PersistentVolumeClaim（PVC）是用户存储的请求。它与 Pod 相似。Pod 消耗节点资源，PVC 消耗 PV 资源。Pod 可以请求特定级别的资源（CPU 和内存）。声明可以请求特定的大小和访问模式（例如，可以以读/写一次或 只读多次模式挂载）。 **PV和PVC之间一一对应！**

#### 2.1.2. PV/PVC工作方式

存储工程师创建并维护各种网络存储设备，比如NFS，GlusterFS,RBD等存储设备。用户或开发工程师在创建POD时指定需要的存储设备资源，即PVC(PersistentVolumeClaim)。PVC会从可用的PV(PersistentVolume)中选择合适的PV进行绑定。PV与PVC是一一绑定的关系。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580108151295-725c800f-a221-49d4-ac3f-f9133349bc90.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580108819276-d13287e4-39b5-4096-bf82-1017b29a3d70.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

### 2.2. 模板

```
apiVersion: v1
kind: PersistentVolume
metadata
    name                <string>                # 在一个名称空间不能重复
    namespace           <string>                # 指定名称空间，默认defalut
    labels              <map[string]string>     # 标签
    annotations         <map[string]string>     # 注释
spec                    <Object>                # 与pod.spec.volumes几乎一致，指定pv的存储类型
    accessModes         <[]string>              # 指定访问模型，有三种模型，但是并不是所有存储介质都支持
        # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
        # RWO/ReadWriteOnce: 单路读写
        # ROX/ReadOnlyMany:  单路只读
        # RWX/ReadWriteMany: 多路读写
    capacity            <map[string]string>     # 指定磁盘空间大小Ei, Pi, Ti, Gi, Mi, Ki 单位表示1024进制存储空间大小
```



```
apiVersion: v1
kind: PersistentVolumeClaim
metadata
    name        <string>            # 在一个名称空间不能重复
    namespace   <string>            # 指定名称空间，默认defalut
    labels      <map[string]string> # 标签
    annotations <map[string]string> # 注释
spec
    accessModes                     <[]string>              # 访问模型,必须是pv.spec.acccessModes的子集
    resources                       <Object>                # 指定当前PVC需要的系统最小资源限制
        limits                      <map[string]string>     # 资源限制
        requests                    <map[string]string>     # 资源限制，常用为 storage: xGi
    selector                        <Object>                # 标签选择器，选择PV的标签，默认在所有PV中寻找
    storageClassName                <string>                # 指定存储类对象名称
    volumeMode                      <string>                # 指定PV类型，beta字段，不建议使用
    volumeName                      <string>                # 指定PV名称，直接绑定PV
```



### 2.3. 案例

#### 2.3.1. PV/PVC定义和使用

- 准备NFS存储

```
[root@hdss7-200 volume]# showmount -e 
Export list for hdss7-200.host.com:
/tmp/data/websit/03 10.4.7.0/24
/tmp/data/websit/02 10.4.7.0/24
/tmp/data/websit/01 10.4.7.0/24
```

- 定义PV

```
[root@hdss7-200 volume]# cat /data/k8s-yaml/base_resource/volume/pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv001
  labels:
    type: SSD
    fs: nfs
    speed: fast
spec:
  accessModes:
  - ReadWriteMany
  - ReadWriteOnce
  - ReadOnlyMany
  capacity:
    storage: 5Gi
  nfs:
    server: hdss7-200
    path: /tmp/data/websit/01
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv002
  labels:
    type: SSD
    fs: nfs
    speed: slow
spec:
  accessModes:
  - ReadWriteMany
  - ReadOnlyMany
  capacity:
    storage: 5Gi
  nfs:
    server: hdss7-200
    path: /tmp/data/websit/02
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv003
  labels:
    type: SSD
    fs: nfs
    speed: slow
spec:
  accessModes:
  - ReadWriteMany
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  nfs:
    server: hdss7-200
    path: /tmp/data/websit/03
```

- 定义PVC和deployment

```
[root@hdss7-200 volume]# cat /data/k8s-yaml/base_resource/volume/pvc-depolyment.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: website-root-pvc
  namespace: app
  labels:
    tier: website
    type: nfs
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      fs: nfs
      speed: fast
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: pvc-deploy
  namespace: app
  labels:
    tier: volume
    role: nfs
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: volume
      role: nfs
  template:
    metadata: 
      labels:
        tier: volume
        role: nfs
    spec:
      containers:
      - name: main-container
        image: harbor.od.com/public/busybox:v1.31.1
        volumeMounts:
        - name: web-root
          mountPath: /data/web/html
        command:
        - httpd
        args:
        - -f
        - -h 
        - "/data/web/html"
      volumes:
      - name: web-root
        persistentVolumeClaim: 
          claimName: website-root-pvc
```

- 创建

```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/pv.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/pvc-depolyment.yaml
[root@hdss7-21 ~]# kubectl get pv --show-labels
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                  STORAGECLASS   REASON   AGE   LABELS
nfs-pv001   5Gi        RWO,ROX,RWX    Retain           Bound       app/website-root-pvc                           25m   fs=nfs,speed=fast,type=SSD
nfs-pv002   5Gi        ROX,RWX        Retain           Available                                                  25m   fs=nfs,speed=slow,type=SSD
nfs-pv003   10Gi       RWO,RWX        Retain           Available                                                  25m   fs=nfs,speed=slow,type=SSD
[root@hdss7-21 ~]# kubectl get pvc -n app
NAME               STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
website-root-pvc   Bound    nfs-pv001   5Gi        RWO,ROX,RWX                   13m
[root@hdss7-21 ~]# kubectl get pod -l tier=volume -n app -o wide
NAME                          READY   STATUS    RESTARTS   AGE     IP            NODE                NOMINATED NODE   READINESS GATES
pvc-deploy-86847c7b6b-mqvm5   1/1     Running   0          14m     172.7.21.11   hdss7-21.host.com   <none>           <none>
pvc-deploy-86847c7b6b-x4p6m   1/1     Running   0          8m56s   172.7.22.8    hdss7-22.host.com   <none>           <none>
```

- 测试

```
[root@hdss7-21 ~]# kubectl exec pvc-deploy-86847c7b6b-mqvm5 -n app -- /bin/sh -c "echo 'hello world' > /data/web/html/index.html"
[root@hdss7-21 ~]# curl -s 172.7.21.11;curl -s 172.7.22.8  # 主页文件已生成
hello world
hello world
[root@hdss7-21 ~]# kubectl delete pod pvc-deploy-86847c7b6b-mqvm5 pvc-deploy-86847c7b6b-x4p6m -n app
pod "pvc-deploy-86847c7b6b-mqvm5" deleted
pod "pvc-deploy-86847c7b6b-x4p6m" deleted
[root@hdss7-21 ~]# kubectl get pod -l tier=volume -n app -o wide 
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE                NOMINATED NODE   READINESS GATES
pvc-deploy-86847c7b6b-7gqqs   1/1     Running   0          62s   172.7.21.12   hdss7-21.host.com   <none>           <none>
pvc-deploy-86847c7b6b-drm28   1/1     Running   0          62s   172.7.22.11   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# curl -s 172.7.21.12; curl -s 172.7.22.11 # Pod删除重建后，文件依旧存在
hello world
hello world
```



## 3. ConfigMap

### 3.1. 介绍

在容器模式下，配置文件管理方式有以下几类：

- 将配置文件固化到image中，这种对基本固定不变的配置是管用的，如Nginx的nginx.conf配置
- 通过自定义参数来实现，如Pod中args参数，这种仅用来传递一些简单参数
- 通过环境变量来传递参数，这种需要程序本身能处理环境变量，如entrypoint的shell脚本
- 通过外挂配置文件，如将整个配置文件目录在Pod启动中以volume方式挂载到容器中
- 使用ConfigMap/Secret对象来实现管理



ConfigMap是k8s中存储pod应用存储非加密配置的方式，相当于自动部署系统中的配置中心，一种k8s核心资源。Config Map是以key/value方式存储数据，有两种方式可以引用configmap中数据：

- 在环境变量中调用，这种方式不能动态更新，必须要重启容器才能生效(不常用)
- 虽然可以存储属性信息，但是主要存储的是配置文件的整个内容，如xxx.properties文件内容，然后将该文件以存储卷的方式挂载到配置文件目录中。当configmap发生变化，会动态更新挂载到的配置文件，应用程序本身应该对可能会动态更新的配置文件实现动态加载(如每隔5分钟做一次读取和更新)，这样就避免修改配置文件的时候重启Pod。

### 3.2. 模板

```
# configMap
apiVersion: v1
kind: ConfigMap
metadata
    name        <string>            # 在一个名称空间不能重复
    namespace   <string>            # 指定名称空间，默认defalut
    labels      <map[string]string> # 标签
    annotations <map[string]string> # 注释
data            <map[string]string> # key/value键值对，必须UTF-8格式
binaryData      <map[string]string> # 二进制数据
```



```
# 通过volume方式引用
pod.spec.volumes.configMap: https://www.yuque.com/duduniao/ww8pmw/vgms23#Ptdfs
# 通过env方式引用
pod.spec.containers.env     <[]Object>              # 指定环境变量
    name                    <string> -required-     # 变量名称
    value                   <string>                # 变量值
    valueFrom               <Object>                # 从其他地方引入value的值
        configMapKeyRef     <Object>                # 从configmap引入
            name            <string>                # configmap名称
            key             <string> -required-     # configmap中的key
            optional        <boolean>               # 是否key必须存在、
        secretKeyRef        <Object>                # 从secret对象读取
            name            <string>                # secret名称
            key             <string> -required-     # secret中的key
            optional        <boolean>               # 是否key必须存在
        fieldRef            <Object>                # 从特殊字段读取
        resourceFieldRef    <Object>                # 从资源字段读取
```

### 3.3. 案例

#### 3.3.1. 创建configmap对象

```
# value如果是数字，必须要转用字符串，如 8080 --> "8080"
# 针对文件内容可以使用命令行方式生成，然后粘贴到yaml文件中
# kubectl create configmap my-config --from-file=key1=file1.txt --dry-run -o yaml
[root@hdss7-200 ~]# vim /data/k8s-yaml/base_resource/volume/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: slb-vhosts-config
  namespace: app
  labels:
    tier: slb
    version: slb-v1.4
data:
  listen-port: "8080"
  server-name: www.duduniao.com
  default-vhost: |
    server {
        listen 8080 default;
        location / {
            return 200 "default-vhost!\r\n";
        }
    }
  blog-vhosts: |
    server {
        listen 8080 ;
        server_name blog.duduniao.com;
        location / {
            return 200 "blog-vhosts!\r\n";
        }
    }
```



```
[root@hdss7-22 ~]# kubectl get configmap/slb-vhosts-config -n app
NAME                DATA   AGE
slb-vhosts-config   4      83s
[root@hdss7-22 ~]# kubectl get configmap slb-vhosts-config -n app -o yaml
apiVersion: v1
data:
  blog-vhosts: |
    server {
        listen 8080 ;
        server_name blog.duduniao.com;
        location / {
            return 200 "blog-vhosts!\r\n";
        }
    }
  default-vhost: |
    server {
        listen 8080 default;
        location / {
            return 200 "default-vhost!\r\n";
        }
    }
  listen-port: "8080"
  server-name: www.duduniao.com
kind: ConfigMap
metadata:
......
```

#### 3.3.2. 使用env方式引用

```
[root@hdss7-200 ~]# vim /data/k8s-yaml/base_resource/volume/cm-pod-01.yaml
apiVersion: v1
kind: Pod
metadata: 
  name: slb-configmap-env
  namespace: app
  labels:
    tier: configmap
    role: slb
spec:
  containers:
  - name: slb-container
    image: harbor.od.com/public/nginx:v1.14
    env:
    - name: LISTEN_PORT
      valueFrom:
        configMapKeyRef:
          name: slb-vhosts-config
          key: listen-port
    - name: SERVER_NAME
      valueFrom:
        configMapKeyRef:
          name: slb-vhosts-config
          key: server-name
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/cm-pod-01.yaml
pod/slb-configmap-env created
[root@hdss7-21 ~]# kubectl get pod slb-configmap-env -n app 
NAME                READY   STATUS    RESTARTS   AGE
slb-configmap-env   1/1     Running   0          14s
[root@hdss7-21 ~]# kubectl exec slb-configmap-env -n app -- /bin/sh -c "echo \$LISTEN_PORT ; echo \$SERVER_NAME"
8080
www.duduniao.com
```

#### 3.3.3. 使用volume方式引用

```
[root@hdss7-200 ~]# cat /data/k8s-yaml/base_resource/volume/cm-pod-02.yaml 
apiVersion: v1
kind: Pod
metadata: 
  name: slb-configmap-volume
  namespace: app
  labels:
    tier: configmap
    role: slb
spec:
  containers:
  - name: slb-container
    image: harbor.od.com/public/nginx:v1.14
    volumeMounts:
    - name: nginx-conf
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-conf
    configMap:
      name: slb-vhosts-config
      defaultMode: 0444
      items:
      - key: default-vhost
        path: default.conf
      - key: blog-vhosts
        path: blog.conf
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/volume/cm-pod-02.yaml
[root@hdss7-21 ~]# kubectl get pod slb-configmap-volume  -n app -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP            NODE                NOMINATED NODE   READINESS GATES
slb-configmap-volume            1/1     Running   0          21s     172.7.21.11   hdss7-21.host.com   <none>           <none>
[root@hdss7-21 ~]# curl -s 172.7.21.11:8080
default-vhost!
[root@hdss7-21 ~]# curl -s -H 'Host: blog.duduniao.com' 172.7.21.11:8080
blog-vhosts!
```



## 4. Secret

### 4.1. 介绍

Secret解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod Spec中。Secret可以以Volume或者环境变量的方式使用。Secret有三种类型：

- Service Account(TLS)：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中
- kubernetes.io/dockerconfigjson(docker-registry): 用来存储私有docker registry的认证信息
- Opaque: 除了TLS和docker-registry之外的secret都是opaque类型。base64编码格式的Secret，用来存储密码、密钥等

### 4.2. 模板

因为涉及到加解密操作，一般不使用清单文件创建，可以直接使用命令行工具创建；操作和使用方式与ConfigMap类似。

```
[root@hdss7-22 ~]# kubectl create secret --help
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]
```