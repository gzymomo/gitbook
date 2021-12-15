- [sealos + NFS 部署 kubesphere 3.0](https://www.cnblogs.com/fsckzy/p/14600473.html)

## 背景

使用sealos+NFS部署一个带有持久化存储的k8s1.18.17，并在其之上部署kubesphere。

### sealos 简介

https://github.com/fanux/sealos
 一条命令离线安装kubernetes，超全版本，支持国产化，生产环境中稳如老狗，99年证书，0依赖，去haproxy keepalived，v1.20支持containerd！

### NFS 简介

没啥好说的

### kubesphere 简介

https://kubesphere.com.cn/
 KubeSphere 是在 Kubernetes 之上构建的面向云原生应用的分布式操作系统，完全开源，支持多云与多集群管理，提供全栈的 IT  自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。

作为全栈的多租户容器平台，KubeSphere  提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。KubeSphere 为用户提供构建企业级 Kubernetes 环境所需的多项功能，例如多云与多集群管理、Kubernetes  资源管理、DevOps、应用生命周期管理、微服务治理（服务网格）、日志查询与收集、服务与网络、多租户管理、监控告警、事件与审计查询、存储管理、访问权限控制、GPU 支持、网络策略、镜像仓库管理以及安全管理等。

总之是kubernetes图形界面就是了。

![img](https://img2020.cnblogs.com/blog/891189/202103/891189-20210331113258501-1241183845.png)

## 使用sealos部署k8s1.18.17

准备3个master节点，N个node节点。系统初始化不讲了，时间要同步，`/var/lib/docker`建议单独挂盘，主机名记得修改，下面简单介绍下怎么批量修改主机名。

### Ansible修改主机名

首先编辑 hosts，将你想要设置的主机名写在ip后面

```yaml
[m]
172.26.140.151 hostname=k8sm1
172.26.140.145 hostname=k8sm2
172.26.140.216 hostname=k8sm3

[n]
172.26.140.202 hostname=k8sn1
172.26.140.185 hostname=k8sn2
172.26.140.156 hostname=k8sn3
```

剧本如下

```shell
---

- hosts: n
  gather_facts: no
  tasks:
    - name: change name
      raw: "echo {{hostname|quote}} > /etc/hostname"
    - name:
      shell: hostname {{hostname|quote}}
```

运行

```shell
ansible-playbook main.yaml
```

### 安装k8s

```shell
sealos init --passwd '123456' --master 172.26.140.151  --master 172.26.140.145  --master 172.26.140.216 --node 172.26.140.202 --node 172.26.140.185 --user root --pkg-url /root/kube1.18.17.tar.gz --version v1.18.17
```

过个几分钟就会输出：

```shell
[root@k8sm1 ks]# kubectl get nodes
NAME    STATUS   ROLES    AGE   VERSION
k8sm1   Ready    master   22m   v1.18.17
k8sm2   Ready    master   22m   v1.18.17
k8sm3   Ready    master   22m   v1.18.17
k8sn1   Ready    <none>   21m   v1.18.17
k8sn2   Ready    <none>   21m   v1.18.17
```

要增加node节点

```shell
sealos join --node 172.26.140.156 --node 172.26.140.139
```

## helm & NFS

NFS服务器怎么搭建不讲了。下面将如何将NFS设置为k8s的默认存储，需要用到Helm。

### Helm

到这下载二进制文件，https://github.com/helm/helm/releases

```shell
wget https://get.helm.sh/helm-v3.4.2-linux-amd64.tar.gz
tar zxvf helm-v3.4.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/helm
```

### NFS

```shell
docker pull heegor/nfs-subdir-external-provisioner:v4.0.0
docker tag heegor/nfs-subdir-external-provisioner:v4.0.0 gcr.io/k8s-staging-sig-storage/nfs-subdir-external-provisioner:v4.0.0
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner     --set nfs.server=1.2.3.4     --set nfs.path=/xx/k8s
```

## 设置默认StorageClass

```shell
[root@k8sm1 ~]# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   114s
[root@k8sm1 ~]# kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/nfs-client patched
[root@k8sm1 ~]# kubectl get storageclass
NAME                   PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client (default)   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   2m5s
```

## kubesphere

### 修改配置文件

kubesphere-installer.yaml不用改动

cluster-configuration.yaml 主要修改两项

使用外置的elasticsearch，需要新增

```shell
      externalElasticsearchUrl: 192.168.1.1
      externalElasticsearchPort: 9200
```

还需要设置etcd集群的地址

```shell
    endpointIps: 172.26.140.151,172.26.140.145,172.26.140.216  # etcd cluster EndpointIps, it can be a bunch of IPs here.
```

完整配置如下：

```yaml
---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.0.0
spec:
  persistence:
    storageClass: ""        # If there is not a default StorageClass in your cluster, you need to specify an existing StorageClass here.
  authentication:
    jwtSecret: ""           # Keep the jwtSecret consistent with the host cluster. Retrive the jwtSecret by executing "kubectl -n kubesphere-system get cm kubesphere-config -o yaml | grep -v "apiVersion" | grep jwtSecret" on the host cluster.
  etcd:
    monitoring: true       # Whether to enable etcd monitoring dashboard installation. You have to create a secret for etcd before you enable it.
    endpointIps: 172.26.140.151,172.26.140.145,172.26.140.216  # etcd cluster EndpointIps, it can be a bunch of IPs here.
    port: 2379              # etcd port
    tlsEnable: true
  common:
    mysqlVolumeSize: 20Gi # MySQL PVC size.
    minioVolumeSize: 20Gi # Minio PVC size.
    etcdVolumeSize: 20Gi  # etcd PVC size.
    openldapVolumeSize: 2Gi   # openldap PVC size.
    redisVolumSize: 2Gi # Redis PVC size.
    es:   # Storage backend for logging, events and auditing.
      elasticsearchMasterReplicas: 1   # total number of master nodes, it's not allowed to use even number
      elasticsearchDataReplicas: 1     # total number of data nodes.
      elasticsearchMasterVolumeSize: 4Gi   # Volume size of Elasticsearch master nodes.
      elasticsearchDataVolumeSize: 20Gi    # Volume size of Elasticsearch data nodes.
      logMaxAge: 7                     # Log retention time in built-in Elasticsearch, it is 7 days by default.
      elkPrefix: logstash              # The string making up index names. The index name will be formatted as ks-<elk_prefix>-log.
      externalElasticsearchUrl: 192.168.1.1
      externalElasticsearchPort: 9200
  console:
    enableMultiLogin: true  # enable/disable multiple sing on, it allows an account can be used by different users at the same time.
    port: 30880
  alerting:                # (CPU: 0.3 Core, Memory: 300 MiB) Whether to install KubeSphere alerting system. It enables Users to customize alerting policies to send messages to receivers in time with different time intervals and alerting levels to choose from.
    enabled: false
  auditing:                # Whether to install KubeSphere audit log system. It provides a security-relevant chronological set of records，recording the sequence of activities happened in platform, initiated by different tenants.
    enabled: false
  devops:                  # (CPU: 0.47 Core, Memory: 8.6 G) Whether to install KubeSphere DevOps System. It provides out-of-box CI/CD system based on Jenkins, and automated workflow tools including Source-to-Image & Binary-to-Image.
    enabled: false
    jenkinsMemoryLim: 2Gi      # Jenkins memory limit.
    jenkinsMemoryReq: 1500Mi   # Jenkins memory request.
    jenkinsVolumeSize: 8Gi     # Jenkins volume size.
    jenkinsJavaOpts_Xms: 512m  # The following three fields are JVM parameters.
    jenkinsJavaOpts_Xmx: 512m
    jenkinsJavaOpts_MaxRAM: 2g
  events:                  # Whether to install KubeSphere events system. It provides a graphical web console for Kubernetes Events exporting, filtering and alerting in multi-tenant Kubernetes clusters.
    enabled: false
    ruler:
      enabled: true
      replicas: 2
  logging:                 # (CPU: 57 m, Memory: 2.76 G) Whether to install KubeSphere logging system. Flexible logging functions are provided for log query, collection and management in a unified console. Additional log collectors can be added, such as Elasticsearch, Kafka and Fluentd.
    enabled: true
    logsidecarReplicas: 2
  metrics_server:                    # (CPU: 56 m, Memory: 44.35 MiB) Whether to install metrics-server. IT enables HPA (Horizontal Pod Autoscaler).
    enabled: false
  monitoring:
    # prometheusReplicas: 1            # Prometheus replicas are responsible for monitoring different segments of data source and provide high availability as well.
    prometheusMemoryRequest: 400Mi   # Prometheus request memory.
    prometheusVolumeSize: 20Gi       # Prometheus PVC size.
    # alertmanagerReplicas: 1          # AlertManager Replicas.
  multicluster:
    clusterRole: none  # host | member | none  # You can install a solo cluster, or specify it as the role of host or member cluster.
  networkpolicy:       # Network policies allow network isolation within the same cluster, which means firewalls can be set up between certain instances (Pods).
    # Make sure that the CNI network plugin used by the cluster supports NetworkPolicy. There are a number of CNI network plugins that support NetworkPolicy, including Calico, Cilium, Kube-router, Romana and Weave Net.
    enabled: false
  notification:        # Email Notification support for the legacy alerting system, should be enabled/disabled together with the above alerting option.
    enabled: false
  openpitrix:          # (2 Core, 3.6 G) Whether to install KubeSphere Application Store. It provides an application store for Helm-based applications, and offer application lifecycle management.
    enabled: false
  servicemesh:         # (0.3 Core, 300 MiB) Whether to install KubeSphere Service Mesh (Istio-based). It provides fine-grained traffic management, observability and tracing, and offer visualization for traffic topology.
    enabled: false
```

### 安装

```shell
kubectl apply -f kubesphere-installer.yaml 
kubectl apply -f cluster-configuration.yaml 
```

然后过几分钟就安装etcd证书，不然Prometheus-0、Prometheus-1会安装不上

### etcd证书

```shell
 kubectl -n kubesphere-monitoring-system create secret generic kube-etcd-client-certs  \
--from-file=etcd-client-ca.crt=/etc/kubernetes/pki/etcd/ca.crt  \
--from-file=etcd-client.crt=/etc/kubernetes/pki/etcd/healthcheck-client.crt  \
--from-file=etcd-client.key=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

### 验证

```shell
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

如果日志卡住了，就去看pod有啥问题

```shell
kubectl get pod -A
```

看到pod不是running状态的，比如`crashloopbackoff`、`init 0/1`之类的就desc或看日志

在根据情况去处理

最后会输出：

```shell
**************************************************
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://172.26.140.151:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After logging into the console, please check the
     monitoring status of service components in
     the "Cluster Management". If any service is not
     ready, please wait patiently until all components 
     are ready.
  2. Please modify the default password after login.

#####################################################
https://kubesphere.io             2021-03-27 09:35:17
#####################################################
```