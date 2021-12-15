[kubernetes使用ceph存储](https://www.cnblogs.com/yuezhimi/p/13066663.html)



# PV、PVC概述

管理存储是管理计算的一个明显问题。PersistentVolume子系统为用户和管理员提供了一个API，用于抽象如何根据消费方式提供存储的详细信息。于是引入了两个新的API资源：PersistentVolume和PersistentVolumeClaim

> PersistentVolume（PV）是集群中已由管理员配置的一段网络存储。 集群中的资源就像一个节点是一个集群资源。 PV是诸如卷之类的卷插件，但是具有独立于使用PV的任何单个pod的生命周期。 该API对象包含存储的实现细节，即NFS，iSCSI或云提供商特定的存储系统。

> PersistentVolumeClaim（PVC）是用户存储的请求。 它类似于pod。Pod消耗节点资源，PVC消耗存储资源。 pod可以请求特定级别的资源（CPU和内存）。 权限要求可以请求特定的大小和访问模式。

> 虽然PersistentVolumeClaims允许用户使用抽象存储资源，但是常见的是，用户需要具有不同属性（如性能）的PersistentVolumes，用于不同的问题。 管理员需要能够提供多种不同于PersistentVolumes，而不仅仅是大小和访问模式，而不会使用户了解这些卷的实现细节。 对于这些需求，存在StorageClass资源。

> StorageClass为集群提供了一种描述他们提供的存储的“类”的方法。 不同的类可能映射到服务质量级别，或备份策略，或者由群集管理员确定的任意策略。 Kubernetes本身对于什么类别代表是不言而喻的。 这个概念有时在其他存储系统中称为“配置文件”

## POD动态供给

> 动态供给主要是能够自动帮你创建pv，需要多大的空间就创建多大的pv。k8s帮助创建pv，创建pvc就直接api调用存储类来寻找pv。

> 如果是存储静态供给的话，会需要我们手动去创建pv，如果没有足够的资源，找不到合适的pv，那么pod就会处于pending等待的状态。而动态供给主要的一个实现就是StorageClass存储对象，其实它就是声明你使用哪个存储，然后帮你去连接，再帮你去自动创建pv。

## POD使用RBD做为持久数据卷

### 安装与配置

RBD支持ReadWriteOnce，ReadOnlyMany两种模式

1、配置rbd-provisioner

github仓库链接https://github.com/kubernetes-incubator/external-storage



```bash
# git clone https://github.com/kubernetes-incubator/external-storage.git
# cd external-storage/ceph/rbd/deploy
# NAMESPACE=kube-system
# sed -r -i "s/namespace: [^ ]+/namespace: $NAMESPACE/g" ./rbac/clusterrolebinding.yaml ./rbac/rolebinding.yaml
# kubectl -n $NAMESPACE apply -f ./rbac

# kubectl get pod -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-575bd6d498-n995v           1/1     Running   1          13d
kube-flannel-ds-amd64-gplmm        1/1     Running   1          13d
kube-flannel-ds-amd64-jrrb9        1/1     Running   1          13d
kube-flannel-ds-amd64-ttcx4        1/1     Running   1          13d
rbd-provisioner-75b85f85bd-vr7t5   1/1     Running   0          72s
```



2、配置k8s访问ceph的用户



```bash
1、创建pod时，kubelet需要使用rbd命令去检测和挂载pv对应的ceph image，所以要在所有的worker节点安装ceph客户端ceph-common。
将ceph的ceph.client.admin.keyring和ceph.conf文件拷贝到master的/etc/ceph目录下
yum -y install ceph-common
2、创建 osd pool 在ceph的mon或者admin节点
ceph osd pool create kube 128 128 
ceph osd pool ls
3、创建k8s访问ceph的用户 在ceph的mon或者admin节点
ceph auth get-or-create client.kube mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=kube' -o ceph.client.kube.keyring
4、查看key 在ceph的mon或者admin节点
[root@ceph-node01 my-cluster]# ceph auth get-key client.admin
AQCBrJ9eV/U5NBAAoDlM4gV3a+KNQDBOUqVxdw==
[root@ceph-node01 my-cluster]# ceph auth get-key client.kube
AQCZ96BeUgPkDhAAhxbWarZh9kTx2QbFCDM/rA==

5、创建 admin secret
kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
--from-literal=key=AQCBrJ9eV/U5NBAAoDlM4gV3a+KNQDBOUqVxdw== \
--namespace=kube-system
# kubectl get secret -n kube-system |grep ceph
ceph-secret                   kubernetes.io/rbd                     1      91s
6、在 default 命名空间创建pvc用于访问ceph的 secret
kubectl create secret generic ceph-user-secret --type="kubernetes.io/rbd" \
--from-literal=key=AQCZ96BeUgPkDhAAhxbWarZh9kTx2QbFCDM/rA== \
--namespace=default
# kubectl get secret
NAME                  TYPE                                  DATA   AGE
ceph-user-secret      kubernetes.io/rbd                     1      114s
```



3、配置StorageClass



```bash
# vim storageclass-ceph-rdb.yaml 
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: dynamic-ceph-rdb
provisioner: ceph.com/rbd
parameters:
  monitors: 192.168.0.246:6789,192.168.0.247:6789,192.168.0.248:6789
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: kube
  userId: kube
  userSecretName: ceph-user-secret
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"

[root@k8s-master yaml]# kubectl apply -f storageclass-ceph-rdb.yaml
storageclass.storage.k8s.io/dynamic-ceph-rdb created
[root@k8s-master yaml]# kubectl get sc
NAME               PROVISIONER    RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
dynamic-ceph-rdb   ceph.com/rbd   Delete          Immediate           false                  12s
```



创建 pvc测试



```bash
# cat >ceph-rdb-pvc-test.yaml<<EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: ceph-rdb-claim
spec:
  accessModes:     
    - ReadWriteOnce
  storageClassName: dynamic-ceph-rdb
  resources:
    requests:
      storage: 2Gi
EOF
# kubectl apply -f ceph-rdb-pvc-test.yaml
# kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS       REASON   AGE
persistentvolume/pvc-908ec99d-5029-4c62-952f-016ca11ab08c   2Gi        RWO            Delete           Bound    default/ceph-rdb-claim   dynamic-ceph-rdb            8s

NAME                                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
persistentvolumeclaim/ceph-rdb-claim   Bound    pvc-908ec99d-5029-4c62-952f-016ca11ab08c   2Gi        RWO            dynamic-ceph-rdb   9s
```



创建 nginx pod 挂载测试



```bash
# cat >nginx-pod.yaml<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    name: nginx-pod1
spec:
  containers:
  - name: nginx-pod1
    image: nginx:alpine
    ports:
    - name: web
      containerPort: 80
    volumeMounts:
    - name: ceph-rdb
      mountPath: /usr/share/nginx/html
  volumes:
  - name: ceph-rdb
    persistentVolumeClaim:
      claimName: ceph-rdb-claim
EOF
# kubectl apply -f nginx-pod.yaml

[root@k8s-master yaml]# kubectl get pod -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-pod1   1/1     Running   0          43s   10.244.2.3   k8s-master1   <none>           <none>
[root@k8s-master yaml]# kubectl exec -ti nginx-pod1 -- /bin/sh -c 'echo this is from Ceph RBD!!! > /usr/share/nginx/html/index.html'
[root@k8s-master yaml]# curl 10.244.2.3
this is from Ceph RBD!!!
```



清理

```bash
kubectl delete -f nginx-pod.yaml
kubectl delete -f ceph-rdb-pvc-test.yaml
```

# POD使用CephFS做为持久数据卷

CephFS方式支持k8s的pv的3种访问模式ReadWriteOnce，ReadOnlyMany ，ReadWriteMany

## Ceph端创建CephFS pool

1、如下操作在ceph的mon或者admin节点 CephFS需要使用两个Pool来分别存储数据和元数据



```bash
1、如下操作在ceph的mon或者admin节点CephFS需要使用两个Pool来分别存储数据和元数据
ceph osd pool create fs_data 128
ceph osd pool create fs_metadata 128
ceph osd lspools

2、创建一个CephFS
ceph fs new cephfs fs_metadata fs_data

3、查看
# ceph fs ls
name: cephfs, metadata pool: fs_metadata, data pools: [fs_data ]
```



## 部署 cephfs-provisioner

使用社区提供的cephfs-provisioner



```bash
# cat >external-storage-cephfs-provisioner.yaml<<EOF
apiVersion: v1
kind: Namespace
metadata:
   name: cephfs
   labels:
     name: cephfs
 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cephfs-provisioner
  namespace: kube-system
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cephfs-provisioner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["create", "get", "delete"]
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
 
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
  namespace: kube-system
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["kube-dns","coredns"]
    verbs: ["list", "get"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "create", "delete"]
  - apiGroups: ["policy"]
    resourceNames: ["cephfs-provisioner"]
    resources: ["podsecuritypolicies"]
    verbs: ["use"]
 
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cephfs-provisioner
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cephfs-provisioner
subjects:
- kind: ServiceAccount
  name: cephfs-provisioner
 
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cephfs-provisioner
subjects:
  - kind: ServiceAccount
    name: cephfs-provisioner
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cephfs-provisioner
  apiGroup: rbac.authorization.k8s.io
 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cephfs-provisioner
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: cephfs-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: cephfs-provisioner
    spec:
      containers:
      - name: cephfs-provisioner
        image: "quay.io/external_storage/cephfs-provisioner:latest"
        env:
        - name: PROVISIONER_NAME
          value: ceph.com/cephfs
        command:
        - "/usr/local/bin/cephfs-provisioner"
        args:
        - "-id=cephfs-provisioner-1"
        - "-disable-ceph-namespace-isolation=true"
      serviceAccount: cephfs-provisioner
EOF
# kubectl apply -f external-storage-cephfs-provisioner.yaml
# kubectl get pod -n kube-system |grep cephfs
cephfs-provisioner-847468fc-5k8vx   1/1     Running   0          7m39s
```



配置secret



```yaml
查看key 在ceph的mon或者admin节点
ceph auth get-key client.admin

创建 admin secret
# kubectl create secret generic ceph-secret --type="kubernetes.io/rbd" \
--from-literal=key=AQCBrJ9eV/U5NBAAoDlM4gV3a+KNQDBOUqVxdw== \
--namespace=kube-system

查看
# kubectl get secret ceph-secret -n kube-system -o yaml
apiVersion: v1
data:
  key: QVFDQnJKOWVWL1U1TkJBQW9EbE00Z1YzYStLTlFEQk9VcVZ4ZHc9PQ==
kind: Secret
metadata:
  creationTimestamp: "2020-06-08T08:17:09Z"
  name: ceph-secret
  namespace: kube-system
  resourceVersion: "42732"
  selfLink: /api/v1/namespaces/kube-system/secrets/ceph-secret
  uid: efec109a-17de-4f72-afd4-d126f4d4f8d6
type: kubernetes.io/rbd
```



配置 StorageClass



```yaml
# vim storageclass-cephfs.yaml 
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: dynamic-cephfs
provisioner: ceph.com/cephfs
parameters:
    monitors: 192.168.0.246:6789,192.168.0.247:6789,192.168.0.248:6789
    adminId: admin
    adminSecretName: ceph-secret
    adminSecretNamespace: "kube-system"
    claimRoot: /volumes/kubernetes

[root@k8s-master yaml]# kubectl apply -f storageclass-cephfs.yaml
storageclass.storage.k8s.io/dynamic-cephfs created
[root@k8s-master yaml]# kubectl get sc
NAME               PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
dynamic-ceph-rdb   ceph.com/rbd      Delete          Immediate           false                  59m
dynamic-cephfs     ceph.com/cephfs   Delete          Immediate           false                  5s
```



创建pvc测试



```yaml
# cat >cephfs-pvc-test.yaml<<EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cephfs-claim
spec:
  accessModes:     
    - ReadWriteOnce
  storageClassName: dynamic-cephfs
  resources:
    requests:
      storage: 2Gi
EOF
# kubectl apply -f cephfs-pvc-test.yaml 
persistentvolumeclaim/cephfs-claim created
[root@k8s-master yaml]# kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS     REASON   AGE
persistentvolume/pvc-df8768d7-5111-4e14-a0cf-dd029b00469b   2Gi        RWO            Delete           Bound    default/cephfs-claim   dynamic-cephfs            12s

NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
persistentvolumeclaim/cephfs-claim   Bound    pvc-df8768d7-5111-4e14-a0cf-dd029b00469b   2Gi        RWO            dynamic-cephfs   13s
```

创建 nginx pod 挂载测试

```yaml
# cat >nginx-pod.yaml<<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod2
  labels:
    name: nginx-pod2
spec:
  containers:
  - name: nginx-pod2
    image: nginx
    ports:
    - name: web
      containerPort: 80
    volumeMounts:
    - name: cephfs
      mountPath: /usr/share/nginx/html
  volumes:
  - name: cephfs
    persistentVolumeClaim:
      claimName: cephfs-claim
EOF
# kubectl apply -f nginx-pod.yaml
[root@k8s-master yaml]# kubectl get pod -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
nginx-pod2   1/1     Running   0          37s   10.244.2.5   k8s-master1   <none>           <none>
[root@k8s-master yaml]# kubectl exec -ti nginx-pod2 -- /bin/sh -c 'echo This is from CephFS!!! > /usr/share/nginx/html/index.html'
[root@k8s-master yaml]# curl 10.244.2.5
This is from CephFS!!!
###查看挂载信息
[root@k8s-master yaml]# kubectl exec -ti nginx-pod2 -- /bin/sh -c 'df -h'
Filesystem                                                                                                                                           Size  Used Avail Use% Mounted on
overlay                                                                                                                                               76G  4.1G   72G   6% /
tmpfs                                                                                                                                                 64M     0   64M   0% /dev
tmpfs                                                                                                                                                1.4G     0  1.4G   0% /sys/fs/cgroup
/dev/sda3                                                                                                                                             76G  4.1G   72G   6% /etc/hosts
shm                                                                                                                                                   64M     0   64M   0% /dev/shm
192.168.0.246:6789,192.168.0.247:6789,192.168.0.248:6789:/volumes/kubernetes/kubernetes/kubernetes-dynamic-pvc-e957596e-a969-11ea-8966-cec7bd638523   85G     0   85G   0% /usr/share/nginx/html
tmpfs                                                                                                                                                1.4G   12K  1.4G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs                                                                                                                                                1.4G     0  1.4G   0% /proc/acpi
tmpfs                                                                                                                                                1.4G     0  1.4G   0% /proc/scsi
tmpfs                                                                                                                                                1.4G     0  1.4G   0% /sys/firmware
```

