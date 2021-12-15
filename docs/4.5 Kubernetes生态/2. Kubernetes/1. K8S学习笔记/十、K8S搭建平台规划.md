# ca1、搭建K8S环境平台规划

## 1.1 单master集群

![](..\..\img\singlemaster.png)



## 1.2 多master集群(高可用)

![](..\..\img\multimaster.png)







# 2、服务器硬件配置要求

## 2.1 测试环境

master：

- 2核 4G 20G

node：

- 4核 8G 40G



## 2.2 生产环境

更高的要求：

master：





# 3、搭建K8S集群部署方式

## 3.1 kubeadm方式

​	Kubeadm是一个K8S部署工具，提供kubeadm init 和kubeadm join，用于快速部署Kubernetes集群。



### 3.1.1 安装前要求

禁止swap分区



### 3.1.2 kubeadm部署

1. 创建一个Master节点： kubeadm init
2. 将Node节点加入到当前集群中 kubeadm join <Master节点的IP和端口>



### 3.1.3 安装操作初始化步骤



1. 关闭防火墙：`systemctl stop firewalld`临时关闭，`systemctl disable firewalld`永久关闭。（master和node都执行。）
2. 关闭selinux：临时关闭：`setenforce 0`，永久关闭：`sed -i 's/enforcing/disabled/' /etc/selinux/config`。（master和node都执行。）
3. 关闭swap：临时关闭：`swapoff -a`，永久关闭：`sed -ri 's/.*swap.*/#&/' /etc/fstab`。（master和node都执行。）
4. 根据规划设置主机名：`hostnamectl set-hostname <hostname>`。（master和node都执行。）
5. 在master添加hosts：（只在master执行）

```bash
cat >> /etc/hosts << EOF
192.168.44.141 master
192.168.44.142 node1
192.168.44.143 node2
EOF
```

6. 将桥接的IPv4流量传递到iptables的链（master和node都执行。）

```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后在执行：（master和node都执行。）

```bash
sysctl --system #生效
```

7. 时间同步（master和node都执行。）

```bash
yum install ntpdate -y
ntpdate time.windows.com
```



### 3.1.4 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Dokcer。

1. 安装Docker



配置Docker镜像阿里云加速器：

```bash
cat > /etc/docker/daemon.json << EOF
{
	"regostry-mirros": ["  "]
}
EOF
```



2. 添加阿里云YUM软件源

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[Kubernetes]
name=Kubernetes
baseurl=
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=

EOF
```



3. 安装kubeadm，kubelet和kubectl

由于版本更新频繁，指定版本号部署：

```bash
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
systemctl enable kubelet
```

### 3.1.5 部署Kubernetes Master

在Master上执行：

```bash
kubeadm init \
--apiserver-advertise-address=192.168.44.141 \   # 当前节点的IP
--image-repository registry.aliyuns.com/google_containers \	# 镜像
--kubernetes-version v1.18.0 \ # 版本
--service-cidr=10.96.0.0/12 \ #只要不与当前IP冲突即可
--pod-networ-cidr=10.244.0.0/16 #只要不与当前IP冲突即可
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

```bash
kubeadm init \
--apiserver-advertise-address=192.168.44.141 \
--image-repository registry.aliyuns.com/google_containers \
--kubernetes-version v1.18.0 \ 
--service-cidr=10.96.0.0/12 \
--pod-networ-cidr=10.244.0.0/16
```



使用kubectl工具：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```



![](..\..\img\load.png)

### 3.1.6 加入Kubernetes Node

​	在Node执行：

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```bash
kubeadm join ip:6443 --token xxxxxxxxxxxxx \
--discover-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

默认token有效期为24小时，当过期之后，该token就不可用了。

这时就需要重新创建token，操作如下：

```bash
kubeadm token create --print-join-command
```



### 3.1.7. 部署CNI网络插件

```bash
wget https://raw.githubusercontenct.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```bash
kubectl apply -f https://raw.githubuser.content.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods -n kube-system
```

可以看到Status为Running状态。



### 3.1.8 测试Kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行。

```bash
# 连网去拉取nginx镜像
kubectl create deployment nginx --image=nginx

# 对外暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看当前对外端口
kubectl get pod,svc
```

访问地址：http://NodeIp:Port



## 3.2 二进制包

​	从github下载发行版的二进制包，手动部署每个组件，组成Kubernetes集群。

​	Kubeadm降低部署门槛，但屏蔽了很多细节，遇到问题难排查。



### 3.2.1 操作系统的初始化



1. 关闭防火墙：`systemctl stop firewalld`临时关闭，`systemctl disable firewalld`永久关闭。（master和node都执行。）
2. 关闭selinux：临时关闭：`setenforce 0`，永久关闭：`sed -i 's/enforcing/disabled/' /etc/selinux/config`。（master和node都执行。）
3. 关闭swap：临时关闭：`swapoff -a`，永久关闭：`sed -ri 's/.*swap.*/#&/' /etc/fstab`。（master和node都执行。）
4. 根据规划设置主机名：`hostnamectl set-hostname <hostname>`。（master和node都执行。）
5. 在master添加hosts：（只在master执行）

```bash
cat >> /etc/hosts << EOF
192.168.44.141 master
192.168.44.142 node1
192.168.44.143 node2
EOF
```

6. 将桥接的IPv4流量传递到iptables的链（master和node都执行。）

```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

然后在执行：（master和node都执行。）

```bash
sysctl --system #生效
```

7. 时间同步（master和node都执行。）

```bash
yum install ntpdate -y
ntpdate time.windows.com
```



### 3.2.2 为etcd和APIServer准备自签证书

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```



------生成证书文件



### 3.2.3 部署etcd集群

Etcd是一个分布式键值存储系统，Kubernetes使用Etcd进行数据存储，所以先准备一个Etcd数据库，为解决Etcd单点故障，应采用集群方式部署，这里使用3台组件集群，可容忍一台机器故障，当然，也可以使用5台组件集群，容忍2台集群故障。



### 3.2.4 部署master组件

​	包括kube-apiserver，kube-controller-manager，kube-scheduler，etcd。



### 3.2.6 部署Node组件

​	包括kubelet，kube-proxy，docker，etcd。



### 3.2.7 部署集群网络



