- [K8SEASY：一键安装K8S高可用集群](https://blog.csdn.net/yanggd1987/article/details/115257453)

# 一、简述

kubeadm 二进制部署 k8s集群，整个部署过程相对比较繁琐，但是通过安装有助于入门者初步了解k8s的组件、网络等信息，因此还是需要了解的。

而本文要推荐的是一款可快速部署多节点高可用K8S集群的软件：K8SEASY,同时内置了Dashboard、Prometheus、Grafana、node-exporter等组件，简单易用。

其他优点如下：

- 一键安装，安装一个系统只需要 3 分钟, 安装好以后完整的监控也一并装好，可以直接使用。（不止支持单master 还支持3master 高可用方案)
- 新版本发布，支持 1.19.x 新增nginx-ingress, helm 组件， 支持自动发现。
- 完全不需要配置，一键完成，包括集群也是一键生成。
- 不需要下载任何镜像，所有需要的模块组件都已经准备。
- 安装所有的组件是以系统服务方式安装，启动停止方便。
- 完美支持Centos7.0 和Ubuntu 16.04/18.04。

# 二、内核升级

## 2.1 离线升级内核

```bash
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-5.4.107-1.el7.elrepo.x86_64.rpm
wget https://elrepo.org/linux/kernel/el7/x86_64/RPMS/kernel-lt-devel-5.4.107-1.el7.elrepo.x86_64.rpm
yum localinstall -y kernel-lt-5.4.107-1.el7.elrepo.x86_64.rpm kernel-lt-devel-5.4.107-1.el7.elrepo.x86_64.rpm
```

## 2.2 设置grub内核版本

```bash
# vim /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
#由saved改为0
GRUB_DEFAULT=0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

## 2.3 重新创建内核配置

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 2.4 重启并查看内核

```bash
# uname -a
Linux prod-166-203-252 5.4.107-1.el7.elrepo.x86_64 #1 SMP Sat Mar 20 09:47:18 EDT 2021 x86_64 x86_64 x86_64 GNU/Linux
```

# 三、集群部署

## 3.1 下载k8seasy组件

```bash
wget http://dl.k8seasy.com/installer 
wget http://dl.k8seasy.com/kubernetes-server-linux-amd64.tar.gz
wget http://dl.k8seasy.com/pack.2020.10.02.bin
```

其中：

- installer 是安装工具。
- kubernetes-server-linux-amd64.tar.gz 可官网下载，须是amd64平台。
- pack.2020.10.02.bin 是镜像文件。

## 3.2 创建密钥

```bash
./installer --genkey -hostlist=10.166.203.252
```

此时会在installer同级目录下生成密钥k8skey.pem，此密钥将用于后续新节点的加入。

## 3.3 创建集群

```bash
#如果是多个master，可以使用如下命令
./installer   -kubernetestarfile kubernetes-server-linux-amd64.tar.gz -masterip 10.166.203.252,10.166.203.253,10.166.203.254

# 单master
./installer   -kubernetestarfile kubernetes-server-linux-amd64.tar.gz -masterip 10.166.203.252
# 以下是单master部署过程打印信息
master:  10.166.203.252
IP: 10.166.203.252  Eth: ens192
1> Upgrade system.
2> Install etcd.
3> Install flanneld.
4> Install haproxy.
5> Update kubernets env.
6> Install K8S API Server.
7> Install K8S Controller Server.
8> Install K8S Scheduler Server.
9> Install Docker.
...............
10> Update Kubeadm env.
11> Install kubelet.
12> Install K8S Proxy.
13> Install coredns.
14> Install monitoring & dashboard.
15> Install Helm.
16> Install nginx-ingress.
17> Install localwebproxy.


请使用浏览器 访问 alertmanager  http://10.166.203.252:8080 
请使用浏览器 访问 grafana  http://10.166.203.252:8081   默认用户名 admin  默认密码 admin
请使用浏览器 访问 prometheus http://10.166.203.252:8082
请使用浏览器 访问 node_export http://10.166.203.252:9100
请使用浏览器 访问 dashboard http://10.166.203.252:10000

You can find more detail about alertmanager by visiting http://10.166.203.252:8080
You can find more detail about grafana by visiting http://10.166.203.252:8081   the default user: admin   pass: admin
You can find more detail about prometheus by visiting http://10.166.203.252:8082
You can find more detail about node_export by visiting http://10.166.203.252:9100
You can find more detail about dashboard by visiting http://10.166.203.252:10000
18> Install localwebproxy.
已经复制了一个配置文件 lens.kubeconfig 在当前目录，这个配置文件可以被 各种管理工具（如lens) 直接使用
请在 /etc/hosts 文件里加入 如下内容
10.166.203.252  s7400036261.lens.k8seasy.com
其中10.166.203.252 为本机的IP, 也可以为任何管理工具直接能访问的本机IP （如本机有多个ip 可以任选一个

K8Seasy is a tools that can help you quickly deploy Kubernetes clusters to local and cloud environmental.
K8S version: 1.11.3-1.18
installer version: 1.0.0.004
Packfile version: 2020.10.02.001
Packfile: pack.2020.10.02.bin
Packfile md5sum: 06222ec7e9dfd17efe318e94540c019d
Build Time: 
Git Hash: 
Website:  http://www.k8seasy.com  https://github.com/xiaojiaqi/k8seasy_release_page
Feature Request/Bug Report Form: https://github.com/xiaojiaqi/k8seasy_release_page/issues
Email:  k8seasy@gmail.com
License: Personal non-commercial use. License cannot be used either directly or indirectly for business purposes.
```

正如部署信息提示，可通过以下地址登录相关组件：

```bash
# 访问 alertmanager  
http://10.166.203.252:8080 
# 访问 grafana，默认用户名 admin  默认密码 admin
http://10.166.203.252:8081   
# 访问 prometheus 
http://10.166.203.252:8082
# 访问 node_export 
http://10.166.203.252:9100
# 访问 dashboard 
http://10.166.203.252:10000
```

## 3.4 查看资源

```bash
# 查看api-server，由于是单master，因此只有一个。
# service kube-apiserver status
Redirecting to /bin/systemctl status kube-apiserver.service
● kube-apiserver.service - Kubernetes API Server
   Loaded: loaded (/etc/systemd/system/kube-apiserver.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-03-26 18:43:55 CST; 42min ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 13094 (kube-apiserver)
    Tasks: 16
   Memory: 347.8M
   CGroup: /system.slice/kube-apiserver.service
           └─13094 /etc/k8s/k8sapi/api/kube-apiserver --advertise-address=10.166.203.252 --default-not-ready-toleration-seconds=360 --default-unreachable-toleration-seconds=360 --feature-gates=DynamicAuditi...
           


# kubectl get svc -A
NAMESPACE              NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
default                kubernetes                  ClusterIP   10.254.0.1       <none>        443/TCP                        29m
kube-system            kube-dns                    ClusterIP   10.254.0.2       <none>        53/UDP,53/TCP,9153/TCP         28m
kube-system            kubelet                     ClusterIP   None             <none>        10250/TCP,10255/TCP,4194/TCP   27m
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.254.68.28     <none>        8000/TCP                       27m
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.254.241.174   <none>        443:10001/TCP,80:10000/TCP     27m
monitoring             alertmanager-main           ClusterIP   10.254.215.113   <none>        9093/TCP                       28m
monitoring             alertmanager-operated       ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP     27m
monitoring             grafana                     ClusterIP   10.254.103.193   <none>        3000/TCP                       28m
monitoring             kube-state-metrics          ClusterIP   None             <none>        8443/TCP,9443/TCP              28m
monitoring             node-exporter               ClusterIP   None             <none>        9100/TCP                       28m
monitoring             prometheus-adapter          ClusterIP   10.254.23.219    <none>        443/TCP                        28m
monitoring             prometheus-k8s              ClusterIP   10.254.47.42     <none>        9090/TCP                       27m
monitoring             prometheus-operated         ClusterIP   None             <none>        9090/TCP                       27m
monitoring             prometheus-operator         ClusterIP   None             <none>        8443/TCP                       28m
```

从service中可以看到dashboard、alertmanager、grafana、node-exporter都已经内置。

## 3.5 加入新节点

```bash
#将密钥复制到新节点
cp ../k8skey.pem ./
#将机器加入集群
./installer   -kubernetestarfile kubernetes-server-linux-amd64.tar.gz -masterip 10.166.203.252

#如果是多个master集群，可以使用如下命令加入集群
./installer   -kubernetestarfile kubernetes-server-linux-amd64.tar.gz -masterip 10.166.203.252,10.166.203.253,10.166.203.254
```

注意：新节点也要提前准备installer、kubernetes-server-linux-amd64.tar.gz、pack.2020.10.02.bin等安装资源。

## 3.6 一键卸载

```bash
# ./installer -uninstall
1>  uninstall kube-apiserver
2>  uninstall kube-controller-manager
3>  uninstall kube-proxy
4>  uninstall kube-scheduler
5>  uninstall kubelet
6>  uninstall flanneld
7>  uninstall haproxy
8>  uninstall docker
9>  uninstall etcd
10>  uninstall localwebproxy
It's done.
```