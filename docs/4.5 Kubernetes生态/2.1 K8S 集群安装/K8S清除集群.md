- [Kubernetes集群之清除集群](https://www.cnblogs.com/weifeng1463/p/12034701.html)

- [Kubernetes容器集群管理环境 - Node节点的移除与加入](https://www.cnblogs.com/kevingrace/p/11302555.html)



# 清除K8s集群的Etcd集群

##### 暂停相关服务

```bash
systemctl stop etcd
```

##### 清除相关文件

```bash
# 删除 etcd 的工作目录和数据目录
rm -rf /var/lib/etcd

# 删除etcd.service文件
rm -rf /etc/systemd/system/etcd.service

# 删除程序文件
rm -rf /root/local/bin/etcd

# 删除TLS证书文件
rm -rf /etc/etcd/ssl/*
```

# 清除K8s集群的Master节点

##### 暂停相关服务

```bash
systemctl stop kube-apiserver kube-controller-manager kube-scheduler flanneld
```

##### 清除相关文件

```bash
# 删除kube-apiserver工作目录
rm -rf /var/run/kubernetes

# 删除service文件
rm -rf /etc/systemd/system/{kube-apiserver,kube-controller-manager,kube-scheduler,flanneld}.service

# 删除程序文件
rm -rf /root/local/bin/{kube-apiserver,kube-controller-manager,kube-scheduler,flanneld,mk-docker-opts.sh}

# 删除证书文件
rm -rf /etc/flanneld/ssl /etc/kubernetes/ssl

# 删除kubelet缓存
rm -rf ~/.kube/cache ~/.kube/schema
```

# 清除K8s集群的Node节点

##### 暂停相关服务

```bash
systemctl stop kubelet kube-proxy flanneld docker
```

##### 清除相关文件

```bash
# umount kubelet 挂载的目录
mount | grep '/var/lib/kubelet'| awk '{print $3}'|xargs sudo umount

# 删除kubelet工作目录
rm -rf /var/lib/kubelet

# 删除docker工作目录
rm -rf /var/lib/docker

# 删除flanneld写入的网络配置文件
rm -rf /var/run/flannel/

# 删除service文件
rm -rf /etc/systemd/system/{kubelet,docker,flanneld}.service

# 删除程序文件
rm -rf /root/local/bin/{kubelet,docker,flanneld,mk-docker-opts.sh}

# 删除证书文件
rm -rf /etc/flanneld/ssl /etc/kubernetes/ssl
```

##### 清除Iptables

```bash
iptables -F && sudo iptables -X && sudo iptables -F -t nat && sudo iptables -X -t nat
```

##### 清除网桥

```bash
ip link del flannel.1

ip link del docker0
```

# Node节点的移除与加入

## 从Kubernetes集群中移除Node

比如从集群中移除k8s-node03这个Node节点，做法如下：

```bash
1）先在master节点查看Node情况
[root@k8s-master01 ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-node01   Ready    <none>   47d   v1.14.2
k8s-node02   Ready    <none>   47d   v1.14.2
k8s-node03   Ready    <none>   47d   v1.14.2
 
2）接着查看下pod情况
[root@k8s-master01 ~]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
dnsutils-ds-5sc4z           1/1     Running   963        40d   172.30.56.3    k8s-node02   <none>           <none>
dnsutils-ds-h546r           1/1     Running   963        40d   172.30.72.5    k8s-node03   <none>           <none>
dnsutils-ds-jx5kx           1/1     Running   963        40d   172.30.88.4    k8s-node01   <none>           <none>
kevin-nginx                 1/1     Running   0          27d   172.30.72.11   k8s-node03   <none>           <none>
my-nginx-5dd67b97fb-69gvm   1/1     Running   0          40d   172.30.72.4    k8s-node03   <none>           <none>
my-nginx-5dd67b97fb-8j4k6   1/1     Running   0          40d   172.30.88.3    k8s-node01   <none>           <none>
nginx-7db9fccd9b-dkdzf      1/1     Running   0          27d   172.30.88.8    k8s-node01   <none>           <none>
nginx-7db9fccd9b-t8njb      1/1     Running   0          27d   172.30.72.10   k8s-node03   <none>           <none>
nginx-7db9fccd9b-vrp9f      1/1     Running   0          27d   172.30.56.6    k8s-node02   <none>           <none>
nginx-ds-4lf8z              1/1     Running   0          41d   172.30.56.2    k8s-node02   <none>           <none>
nginx-ds-6kfsw              1/1     Running   0          41d   172.30.72.2    k8s-node03   <none>           <none>
nginx-ds-xqdgw              1/1     Running   0          41d   172.30.88.2    k8s-node01   <none>           <none>
 
3）封锁k8s-node03这个node节点，排干该node节点上的pod资源
[root@k8s-master01 ~]# kubectl drain k8s-node03 --delete-local-data --force --ignore-daemonsets
node/k8s-node03 cordoned
WARNING: deleting Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet: default/kevin-nginx; ignoring DaemonSet-managed Pods: default/dnsutils-ds-h546r, default/nginx-ds-6kfsw, kube-system/node-exporter-zmb68
evicting pod "metrics-server-54997795d9-rczmc"
evicting pod "kevin-nginx"
evicting pod "nginx-7db9fccd9b-t8njb"
evicting pod "coredns-5b969f4c88-pd5js"
evicting pod "kubernetes-dashboard-7976c5cb9c-4jpzb"
evicting pod "my-nginx-5dd67b97fb-69gvm"
pod/my-nginx-5dd67b97fb-69gvm evicted
pod/coredns-5b969f4c88-pd5js evicted
pod/nginx-7db9fccd9b-t8njb evicted
pod/kubernetes-dashboard-7976c5cb9c-4jpzb evicted
pod/kevin-nginx evicted
pod/metrics-server-54997795d9-rczmc evicted
node/k8s-node03 evicted
 
4）接着删除k8s-node03这个节点
[root@k8s-master01 ~]# kubectl delete node k8s-node03
node "k8s-node03" deleted
 
5）再查看pod情况，发现原来在k8s-node03上的pod已经调度到其他留存的node节点上了
[root@k8s-master01 ~]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
dnsutils-ds-5sc4z           1/1     Running   963        40d   172.30.56.3   k8s-node02   <none>           <none>
dnsutils-ds-jx5kx           1/1     Running   963        40d   172.30.88.4   k8s-node01   <none>           <none>
my-nginx-5dd67b97fb-8j4k6   1/1     Running   0          40d   172.30.88.3   k8s-node01   <none>           <none>
my-nginx-5dd67b97fb-kx2pc   1/1     Running   0          98s   172.30.56.7   k8s-node02   <none>           <none>
nginx-7db9fccd9b-7vbhq      1/1     Running   0          98s   172.30.88.7   k8s-node01   <none>           <none>
nginx-7db9fccd9b-dkdzf      1/1     Running   0          27d   172.30.88.8   k8s-node01   <none>           <none>
nginx-7db9fccd9b-vrp9f      1/1     Running   0          27d   172.30.56.6   k8s-node02   <none>           <none>
nginx-ds-4lf8z              1/1     Running   0          41d   172.30.56.2   k8s-node02   <none>           <none>
nginx-ds-xqdgw              1/1     Running   0          41d   172.30.88.2   k8s-node01   <none>           <none>
 
[root@k8s-master01 ~]# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-node01   Ready    <none>   47d   v1.14.2
k8s-node02   Ready    <none>   47d   v1.14.2
 
6）最后在k8s-node03节点上执行清理操作：
[root@k8s-node03 ~]# systemctl stop kubelet kube-proxy flanneld docker
  
[root@k8s-node03 ~]# source /opt/k8s/bin/environment.sh
[root@k8s-node03 ~]# mount | grep "${K8S_DIR}" | awk '{print $3}'|xargs sudo umount
[root@k8s-node03 ~]# rm -rf ${K8S_DIR}/kubelet
[root@k8s-node03 ~]# rm -rf ${DOCKER_DIR}
[root@k8s-node03 ~]# rm -rf /var/run/flannel/
[root@k8s-node03 ~]# rm -rf /var/run/docker/
[root@k8s-node03 ~]# rm -rf /etc/systemd/system/{kubelet,docker,flanneld,kube-nginx}.service
[root@k8s-node03 ~]# rm -rf /opt/k8s/bin/*
[root@k8s-node03 ~]# rm -rf /etc/flanneld/cert /etc/kubernetes/cert
  
[root@k8s-node03 ~]# iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat
[root@k8s-node03 ~]# ip link del flannel.1
[root@k8s-node03 ~]# ip link del docker0
```

## 向Kubernetes集群中加入Node节点

比如将之前移除的k8s-node03节点重新加入到k8s集群中 （下面操作都在k8s-master01节点上完成）

```bash
1）修改变量脚本文件/opt/k8s/bin/environment.sh里的NODE节点为k8s-node03节点，然后进行分发。
[root@k8s-master01 ~]# cp /opt/k8s/bin/environment.sh /opt/k8s/bin/environment.sh.bak1
[root@k8s-master01 ~]# vim /opt/k8s/bin/environment.sh
........
# 集群中所有node节点集群IP数组
export NODE_NODE_IPS=(172.16.60.246)
# 集群中node节点IP对应的主机名数组
export NODE_NODE_NAMES=(k8s-node03)
   
[root@k8s-master01 ~]# diff /opt/k8s/bin/environment.sh /opt/k8s/bin/environment.sh.bak1
17c17
< export NODE_NODE_IPS=(172.16.60.246)
---
> export NODE_NODE_IPS=(172.16.60.244 172.16.60.245 172.16.60.246)
19c19
< export NODE_NODE_NAMES=(k8s-node03)
---
> export NODE_NODE_NAMES=(k8s-node01 k8s-node02 k8s-node03)
   
2）将之前在k8s-master01节点上生产的证书文件分发到新加入的node节点上
[root@k8s-master01 ~]# cd /opt/k8s/work/
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "mkdir -p /etc/kubernetes/cert"
    scp ca*.pem ca-config.json root@${node_node_ip}:/etc/kubernetes/cert
  done
   
3) Flannel容器网络
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp flannel/{flanneld,mk-docker-opts.sh} root@${node_node_ip}:/opt/k8s/bin/
    ssh root@${node_node_ip} "chmod +x /opt/k8s/bin/*"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "mkdir -p /etc/flanneld/cert"
    scp flanneld*.pem root@${node_node_ip}:/etc/flanneld/cert
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp flanneld.service root@${node_node_ip}:/etc/systemd/system/
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "systemctl daemon-reload && systemctl enable flanneld && systemctl restart flanneld"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "systemctl status flanneld|grep Active"
  done
   
4）部署node节点运行组件
   
->  安装依赖包
[root@k8s-master01 ~]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 ~]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "yum install -y epel-release"
    ssh root@${node_node_ip} "yum install -y conntrack ipvsadm ntp ntpdate ipset jq iptables curl sysstat libseccomp && modprobe ip_vs "
  done
   
->  部署docker组件
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp docker/*  root@${node_node_ip}:/opt/k8s/bin/
    ssh root@${node_node_ip} "chmod +x /opt/k8s/bin/*"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp docker.service root@${node_node_ip}:/etc/systemd/system/
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "mkdir -p  /etc/docker/ ${DOCKER_DIR}/{data,exec}"
    scp docker-daemon.json root@${node_node_ip}:/etc/docker/daemon.json
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "systemctl daemon-reload && systemctl enable docker && systemctl restart docker"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "systemctl status docker|grep Active"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "/usr/sbin/ip addr show flannel.1 && /usr/sbin/ip addr show docker0"
  done
   
->  部署kubelet组件
[root@k8s-master01 ~]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp kubernetes/server/bin/kubelet root@${node_node_ip}:/opt/k8s/bin/
    ssh root@${node_node_ip} "chmod +x /opt/k8s/bin/*"
  done
   
->  创建token（之前创建的已经过期，token有效期只有24h，即有效期只有一天！）
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
       
    # 创建 token
    export BOOTSTRAP_TOKEN=$(kubeadm token create \
      --description kubelet-bootstrap-token \
      --groups system:bootstrappers:${node_node_name} \
      --kubeconfig ~/.kube/config)
       
    # 设置集群参数
    kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/cert/ca.pem \
      --embed-certs=true \
      --server=${KUBE_APISERVER} \
      --kubeconfig=kubelet-bootstrap-${node_node_name}.kubeconfig
       
    # 设置客户端认证参数
    kubectl config set-credentials kubelet-bootstrap \
      --token=${BOOTSTRAP_TOKEN} \
      --kubeconfig=kubelet-bootstrap-${node_node_name}.kubeconfig
       
    # 设置上下文参数
    kubectl config set-context default \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=kubelet-bootstrap-${node_node_name}.kubeconfig
       
    # 设置默认上下文
    kubectl config use-context default --kubeconfig=kubelet-bootstrap-${node_node_name}.kubeconfig
  done
   
查看 kubeadm 为各新节点创建的 token：
[root@k8s-master01 work]# kubeadm token list --kubeconfig ~/.kube/config
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION               EXTRA GROUPS
sdwq5g.llzr9ytm32h1mnh1   23h       2019-08-06T11:47:47+08:00   authentication,signing   kubelet-bootstrap-token   system:bootstrappers:k8s-node03
   
[root@k8s-master01 work]# kubectl get secrets  -n kube-system|grep bootstrap-token
bootstrap-token-sdwq5g                           bootstrap.kubernetes.io/token         7      77s
   
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
    scp kubelet-bootstrap-${node_node_name}.kubeconfig root@${node_node_name}:/etc/kubernetes/kubelet-bootstrap.kubeconfig
  done
   
->  分发 bootstrap kubeconfig 文件到新增node节点
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
    scp kubelet-bootstrap-${node_node_name}.kubeconfig root@${node_node_name}:/etc/kubernetes/kubelet-bootstrap.kubeconfig
  done
   
->  分发 kubelet 参数配置文件
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    sed -e "s/##NODE_NODE_IP##/${node_node_ip}/" kubelet-config.yaml.template > kubelet-config-${node_node_ip}.yaml.template
    scp kubelet-config-${node_node_ip}.yaml.template root@${node_node_ip}:/etc/kubernetes/kubelet-config.yaml
  done
   
->  分发 kubelet systemd unit 文件
[root@k8s-master01 work]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
    sed -e "s/##NODE_NODE_NAME##/${node_node_name}/" kubelet.service.template > kubelet-${node_node_name}.service
    scp kubelet-${node_node_name}.service root@${node_node_name}:/etc/systemd/system/kubelet.service
  done
   
->  启动 kubelet 服务
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "mkdir -p ${K8S_DIR}/kubelet/kubelet-plugins/volume/exec/"
    ssh root@${node_node_ip} "/usr/sbin/swapoff -a"
    ssh root@${node_node_ip} "systemctl daemon-reload && systemctl enable kubelet && systemctl restart kubelet"
  done
  
-> 部署 kube-proxy 组件
[root@k8s-master01 ~]# cd /opt/k8s/work
[root@k8s-master01 work]# source /opt/k8s/bin/environment.sh
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    scp kubernetes/server/bin/kube-proxy root@${node_node_ip}:/opt/k8s/bin/
    ssh root@${node_node_ip} "chmod +x /opt/k8s/bin/*"
  done
   
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
    scp kube-proxy.kubeconfig root@${node_node_name}:/etc/kubernetes/
  done
  
=================================================================================================================================================
特别注意（如果是完全新增node节点，则这里需要添加下面操作）：
由于这里是恢复之前移除的k8s-node03节点，故这里不需要重新根据kube-proxy配置模板生成对应的新增node节点的配置文件(因为之前已经生成过了)
[root@k8s-master01 work]# ll kube-proxy-config-k8s-node*
-rw-r--r-- 1 root root 500 Jun 24 20:27 kube-proxy-config-k8s-node01.yaml.template
-rw-r--r-- 1 root root 500 Jun 24 20:27 kube-proxy-config-k8s-node02.yaml.template
-rw-r--r-- 1 root root 500 Jun 24 20:27 kube-proxy-config-k8s-node03.yaml.template
  
如果是完全新增加的节点，比如新增加的node节点172.16.60.240 (主机名: k8s-node04)，
则这一步还需要拷贝已存在node节点的配置文件为新增node节点的配置文件，然后分发过去
[root@k8s-master01 work]# cp kube-proxy-config-k8s-node03.yaml.template kube-proxy-config-k8s-node04.yaml.template
[root@k8s-master01 work]# sed -i 's/172.16.60.246/172.16.60.240/g' kube-proxy-config-k8s-node04.yaml.template
[root@k8s-master01 work]# sed -i 's/k8s-node03/k8s-node04/g' kube-proxy-config-k8s-node04.yaml.template
[root@k8s-master01 work]# scp kube-proxy-config-k8s-node04.yaml.template root@k8s-node04:/etc/kubernetes/kube-proxy-config.yaml
  
如果是新增多个node节点，则同样是拷贝已存在node节点的配置文件为各个新增node节点的配置文件，然后分发过去
=================================================================================================================================================
  
[root@k8s-master01 work]# for node_node_name in ${NODE_NODE_NAMES[@]}
  do
    echo ">>> ${node_node_name}"
    scp kube-proxy.service root@${node_node_name}:/etc/systemd/system/
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "mkdir -p ${K8S_DIR}/kube-proxy"
    ssh root@${node_node_ip} "modprobe ip_vs_rr"
    ssh root@${node_node_ip} "systemctl daemon-reload && systemctl enable kube-proxy && systemctl restart kube-proxy"
  done
   
[root@k8s-master01 work]# for node_node_ip in ${NODE_NODE_IPS[@]}
  do
    echo ">>> ${node_node_ip}"
    ssh root@${node_node_ip} "systemctl status kube-proxy|grep Active"
  done
   
->  手动 approve server cert csr
[root@k8s-master01 work]# kubectl get csr
NAME        AGE     REQUESTOR                 CONDITION
csr-5fwlh   3m34s   system:bootstrap:sdwq5g   Approved,Issued
csr-t547p   3m21s   system:node:k8s-node03    Pending
   
[root@k8s-master01 work]# kubectl certificate approve csr-t547p
certificatesigningrequest.certificates.k8s.io/csr-t547p approved
   
[root@k8s-master01 work]# kubectl get csr
NAME        AGE     REQUESTOR                 CONDITION
csr-5fwlh   3m53s   system:bootstrap:sdwq5g   Approved,Issued
csr-t547p   3m40s   system:node:k8s-node03    Approved,Issued
   
-> 查看集群状态，发现k8s-node03节点已经被重新加入到集群中了，并且已经分配了pod资源。
[root@k8s-master01 work]# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-node01   Ready    <none>   47d   v1.14.2
k8s-node02   Ready    <none>   47d   v1.14.2
k8s-node03   Ready    <none>   1s    v1.14.2
   
[root@k8s-master01 work]# kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE    IP            NODE         NOMINATED NODE   READINESS GATES
dnsutils-ds-5sc4z           1/1     Running   965        40d    172.30.56.3   k8s-node02   <none>           <none>
dnsutils-ds-gc8sb           1/1     Running   1          94m    172.30.72.2   k8s-node03   <none>           <none>
dnsutils-ds-jx5kx           1/1     Running   966        40d    172.30.88.4   k8s-node01   <none>           <none>
my-nginx-5dd67b97fb-8j4k6   1/1     Running   0          40d    172.30.88.3   k8s-node01   <none>           <none>
my-nginx-5dd67b97fb-kx2pc   1/1     Running   0          174m   172.30.56.7   k8s-node02   <none>           <none>
nginx-7db9fccd9b-7vbhq      1/1     Running   0          174m   172.30.88.7   k8s-node01   <none>           <none>
nginx-7db9fccd9b-dkdzf      1/1     Running   0          27d    172.30.88.8   k8s-node01   <none>           <none>
nginx-7db9fccd9b-vrp9f      1/1     Running   0          27d    172.30.56.6   k8s-node02   <none>           <none>
nginx-ds-4lf8z              1/1     Running   0          41d    172.30.56.2   k8s-node02   <none>           <none>
nginx-ds-jn759              1/1     Running   0          94m    172.30.72.3   k8s-node03   <none>           <none>
nginx-ds-xqdgw              1/1     Running   0          41d    172.30.88.2   k8s-node01   <none>           <none>
   
[root@k8s-master01 work]# kubectl top node
NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s-node01   96m          2%     2123Mi          55%    
k8s-node02   133m         3%     1772Mi          46%    
k8s-node03   46m          1%     4859Mi          61%
   
=======================================================================================================================================
注意一
如果是添加全新的节点到上述k8s集群中，做法如下：
1）做好node节点的环境初始化准备，如做好K8s-master01到新增节点的ssh无密码登录的信任关系；etc/hosts里做好绑定；关闭防火墙等。
2）在/opt/k8s/bin/environment.sh变量脚本里，将NODE_NODE_IPS和NODE_NODE_NAMES变量改成新增node节点的对应信息
3）按照上面添加k8s-node03节点的一系列添加步骤全部执行一遍即可
   
======================================================================================================================================
注意二
上面使用的是二进制方式按照k8s集群。如果使用kubeadmin工具创建的k8s集群，则重新使node加入集群的操作如下：
   
使节点加入集群的命令格式（node节点上操作，使用root用户）：
# kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
   
如果忘记了Master节点的token，可以使用下面命令查看（master节点上操作）：
# kubeadm token list
   
默认情况下，token的有效期是24小时，如果token已经过期的话，可以使用下面命令重新生成（master节点上操作）;
# kubeadm token create
   
如果找不到--discovery-token-ca-cert-hash的值，可以使用以下命令生成（master节点上操作）：
# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
   
加入节点后，稍等一会儿，即可看到节点已加入（master节点上操作）
```

