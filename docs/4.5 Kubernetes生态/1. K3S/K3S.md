## 1. K3S介绍

K3S 是轻量级的Kubernetes，适用于少量机器的场景，如边缘节点(Edge)、物联网设备(IoT)、CI、开发环境和ARM平台。K3S特点：

- k3s是一个高可用的、经过CNCF认证的Kubernetes发行版，专为无人值守、资源受限、偏远地区或物联网设备内部的生产工作负载而设计。
- k3s被打包成单个小于60MB的二进制文件，从而减少了运行安装、运行和自动更新生产Kubernetes集群所需的依赖性和步骤。
- ARM64和ARMv7都支持二进制文件和多源镜像。k3s在小到树莓派或大到 AWS a1.4xlarge 32GiB服务器的环境中均能出色工作。



- K3S 默认是基于 sqlite3 作为数据存储，在高可用场景中，可以使用 MySQL、Postgres、etcd3 以及 DQLite 进行数据存储，Kubernetes采用的是 etcd
- K3s 同时提供以下功能：

- - containerd 作为默认的 runtime，也可以替换为 docker
  - flannel 作为默认的 CNI 插件，可以替换为 calico
  - CoreDNS
  - Traefik 作为默认Ingress控制器，可以替换为其它的ingress controller
  -  服务负载均衡、Helm 控制器

## 2. K3S安装

### 2.1. 单节点

![image](https://cdn.nlark.com/yuque/0/2020/svg/378176/1592485364605-6a33de7b-325d-45b5-93e7-2c60fd174baf.svg)

![image](https://cdn.nlark.com/yuque/0/2020/png/378176/1592485430345-a2e87dda-f73f-4563-8a4b-81798233a2ae.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

#### 2.1.1. 主机清单

| **主机名**               | **IP地址** | **系统版本**                  | **角色**                           |
| ------------------------ | ---------- | ----------------------------- | ---------------------------------- |
| devops-7-3.host.com      | 10.4.7.3   | CentOS Linux release 7.5.1804 | DNS服务器，跳板机，Nginx负载均衡器 |
| k3s-master-7-91.host.com | 10.4.7.91  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-node-7-94.host.com   | 10.4.7.94  | CentOS Linux release 7.5.1804 | k3s node 节点                      |

#### 2.1.2. 部署流程

```
# master 节点
[root@k3s-master-7-91 ~]# export http_proxy="10.4.7.1:10080"              # 配置代理服服务器，为下载脚本和软件包加速，参考: https://www.yuque.com/duduniao/docker/cseut0#WSrRD
[root@k3s-master-7-91 ~]# export https_proxy="10.4.7.1:10080"             # 配置代理服服务器，为下载脚本和软件包加速
[root@k3s-master-7-91 ~]# curl -sfL https://get.k3s.io | sh -s - server   # 使用官方的shell脚本进行安装
[INFO]  Finding release for channel stable
[INFO]  Using v1.18.4+k3s1 as release      # 默认使用最新的稳定版本
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.18.4+k3s1/sha256sum-amd64.txt 
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.18.4+k3s1/k3s # 二进制文件
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s   
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s  # 软连接
[INFO]  Creating /usr/local/bin/crictl symlink to k3s   # 软连接
[INFO]  Creating /usr/local/bin/ctr symlink to k3s      # 软连接
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env # 创建环境服务启动的环境变量,这里面的代理环境变量根据情况考虑删除
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service     # 服务管理脚本
[INFO]  systemd: Enabling k3s unit
Created symlink from /etc/systemd/system/multi-user.target.wants/k3s.service to /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

[root@k3s-master-7-91 ~]# ll /usr/local/bin/  # 命令行工具
total 52356
lrwxrwxrwx 1 root root        3 2020-06-19 04:43:52 crictl -> k3s     # 提供和docker相同功能 
lrwxrwxrwx 1 root root        3 2020-06-19 04:43:52 ctr -> k3s
-rwxr-xr-x 1 root root 53604352 2020-06-19 04:43:52 k3s
-rwxr-xr-x 1 root root     1710 2020-06-19 04:43:52 k3s-killall.sh    # 停止服务和容器
-rwxr-xr-x 1 root root      881 2020-06-19 04:43:52 k3s-uninstall.sh  # 卸载k3s
lrwxrwxrwx 1 root root        3 2020-06-19 04:43:52 kubectl -> k3s    # 提供和kubectl相同功能
[root@k3s-master-7-91 ~]# kubectl get node 
NAME                       STATUS   ROLES    AGE   VERSION
k3s-master-7-91.host.com   Ready    master   53s   v1.18.4+k3s1
[root@k3s-master-7-91 ~]# kubectl get cs
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok        
scheduler            Healthy   ok        
[root@k3s-master-7-91 ~]# kubectl get pod -A  # 启动完毕
NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
kube-system   metrics-server-7566d596c8-hmwb4          1/1     Running     0          114s
kube-system   local-path-provisioner-6d59f47c7-7rtjs   1/1     Running     0          114s
kube-system   helm-install-traefik-2qgtf               0/1     Completed   0          115s
kube-system   coredns-8655855d6-4k9hw                  1/1     Running     0          114s
kube-system   svclb-traefik-7m87l                      2/2     Running     0          77s
kube-system   traefik-758cd5fc85-pkph6                 1/1     Running     0          77s

[root@k3s-master-7-91 ~]# kubectl config view  # 认证文件存储在 /etc/rancher/k3s/k3s.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: 394d8ab1c7004a7c24ca5ca12f06548f
    username: admin
# node 节点，如无必要请删除/etc/systemd/system/k3s-agent.service.env中代理的环境变量
[root@k3s-master-7-91 ~]# cat /var/lib/rancher/k3s/server/node-token  # 获取token，用于node注册
K10e6f83f3a37b883cd3ba58c07984ac540adfe365d24a2b0c336e89a5c0f7365b6::server:4aa7fc8d02f7b004bce3b66daba7c806
[root@k3s-node-7-94 ~]# export http_proxy="10.4.7.1:10080"
[root@k3s-node-7-94 ~]# export https_proxy="10.4.7.1:10080"
[root@k3s-node-7-94 ~]# curl -sfL https://get.k3s.io | K3S_URL=https://10.4.7.91:6443 K3S_TOKEN='K10e6f83f3a37b883cd3ba58c07984ac540adfe365d24a2b0c336e89a5c0f7365b6::server:4aa7fc8d02f7b004bce3b66daba7c806' sh -
[root@k3s-master-7-91 ~]# kubectl get node 
NAME                       STATUS   ROLES    AGE   VERSION
k3s-master-7-91.host.com   Ready    master   27m   v1.18.4+k3s1
k3s-node-7-94.host.com     Ready    <none>   29s   v1.18.4+k3s1
```

### 2.2. 高可用架构

高可用有两个方面，一个是数据库的高可用，一个是Master节点高可用，因此高可用架构如下:

- 使用外部的数据库进行数据存储，如MySQL，etcd3等
- 多个master使用一个固定访问入口，如VIP，L4的负载均衡，DNS等

![image](https://cdn.nlark.com/yuque/0/2020/svg/378176/1592485493081-a64759e5-d03a-4596-907b-d83b5de12c8f.svg)

除此之外，K3S 还有一个处于实验性阶段的高可用方案，即使用DQLite作为数据库，采用2n+1个master节点，每个节点部署对应的数据，采用 raft 算法选择主节点：[https://rancher.com/docs/k3s/latest/en/installation/ha-embedded](https://rancher.com/docs/k3s/latest/en/installation/ha-embedded/)

#### 2.2.1. 主机清单

| **主机名**               | **IP地址** | **系统版本**                  | **角色**                           |
| ------------------------ | ---------- | ----------------------------- | ---------------------------------- |
| devops-7-3.host.com      | 10.4.7.3   | CentOS Linux release 7.5.1804 | DNS服务器，跳板机，Nginx负载均衡器 |
| k3s-db-7-90.host.com     | 10.4.7.90  | CentOS Linux release 7.5.1804 | MySQL数据库                        |
| k3s-master-7-91.host.com | 10.4.7.91  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-master-7-92.host.com | 10.4.7.92  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-master-7-93.host.com | 10.4.7.93  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-node-7-94.host.com   | 10.4.7.94  | CentOS Linux release 7.5.1804 | k3s node 节点                      |
| k3s-node-7-95.host.com   | 10.4.7.95  | CentOS Linux release 7.5.1804 | k3s node 节点                      |

#### 2.2.2. 中间件准备

```
# 使用外部高可用数据库，生产环境中可配置主从
mysql> create database if not exists k3s character set utf8mb4 ;
mysql> grant all privileges on k3s.* to k3s@'10.4.7.%' identified by 'Duduniao.2020';
mysql> flush privileges ;
# 固定master访问路径：
# 1. 配置dns解析到 nginx 所在的服务器,nginx本身应该是高可用的，可以和内网其它服务共用
# 2. 使用 keepalived + lvs 实现VIP访问
本实验使用nginx服务器: k3s-api.od.com --> 10.4.7.3
[root@devops-7-3 ~]# cat /etc/nginx/nginx.conf # 追加以下内容, 配置完毕重载nginx配置文件
stream {
    log_format proxy '$time_local|$remote_addr|$upstream_addr|$status|'
                     '$upstream_connect_time|$bytes_sent|$upstream_bytes_sent|$upstream_bytes_received' ;

    upstream kube-apiserver {
        server 10.4.7.91:6443     max_fails=3 fail_timeout=30s;
        server 10.4.7.92:6443     max_fails=3 fail_timeout=30s;
        server 10.4.7.93:6443     max_fails=3 fail_timeout=30s;
    }
    server {
        listen 6443;
        proxy_connect_timeout 2s;
        proxy_timeout 900s;
        proxy_pass kube-apiserver;
        access_log /var/log/nginx/proxy.log proxy;
    }
}
```

#### 2.2.3. 部署master

```
[root@k3s-master-7-91 ~]# export http_proxy="10.4.7.1:10080"
[root@k3s-master-7-91 ~]# export https_proxy="10.4.7.1:10080"
# 数据库格式: --datastore-endpoint='mysql://${username}:${password}@tcp(${mysql_server}:${mysql_port})/${db_name}'
# --datastore-endpoint 是作为server端的启动参数，在 /etc/systemd/system/k3s.service 中可查看到
[root@k3s-master-7-91 ~]# curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s'
---
[root@k3s-master-7-92 ~]# export http_proxy="10.4.7.1:10080"
[root@k3s-master-7-92 ~]# export https_proxy="10.4.7.1:10080"
[root@k3s-master-7-92 ~]# curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s'
---
[root@k3s-master-7-93 ~]# export http_proxy="10.4.7.1:10080" 
[root@k3s-master-7-93 ~]# export https_proxy="10.4.7.1:10080"
[root@k3s-master-7-93 ~]# curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s'
---
[root@k3s-master-7-91 ~]# kubectl get node
NAME                       STATUS   ROLES    AGE     VERSION
k3s-master-7-91.host.com   Ready    master   3m16s   v1.18.4+k3s1
k3s-master-7-93.host.com   Ready    master   23s     v1.18.4+k3s1
k3s-master-7-92.host.com   Ready    master   91s     v1.18.4+k3s1
[root@k3s-master-7-91 ~]# kubectl get cs
NAME                 STATUS    MESSAGE   ERROR
scheduler            Healthy   ok        
controller-manager   Healthy   ok  
```

#### 2.2.4. 部署node

```
[root@k3s-master-7-91 ~]# cat /var/lib/rancher/k3s/server/node-token # 获取token
K10d1c48eb3ae030c4534780e76f85267e0c61bfb200db782e037a79f99a28ab048::server:44131bb099846847f45a22dc51b1922b
[root@k3s-node-7-94 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-node-7-94 ~]# curl -sfL https://get.k3s.io | K3S_URL=https://10.4.7.91:6443 K3S_TOKEN='K10d1c48eb3ae030c4534780e76f85267e0c61bfb200db782e037a79f99a28ab048::server:44131bb099846847f45a22dc51b1922b' sh - 
---
[root@k3s-node-7-95 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-node-7-95 ~]# curl -sfL https://get.k3s.io | K3S_URL=https://10.4.7.91:6443 K3S_TOKEN='K10d1c48eb3ae030c4534780e76f85267e0c61bfb200db782e037a79f99a28ab048::server:44131bb099846847f45a22dc51b1922b' sh -
---
[root@k3s-master-7-91 ~]# kubectl get node
NAME                       STATUS   ROLES    AGE     VERSION
k3s-master-7-91.host.com   Ready    master   4m57s   v1.18.4+k3s1
k3s-master-7-92.host.com   Ready    master   3m12s   v1.18.4+k3s1
k3s-master-7-93.host.com   Ready    master   2m4s    v1.18.4+k3s1
k3s-node-7-94.host.com     Ready    <none>   77s     v1.18.4+k3s1
k3s-node-7-95.host.com     Ready    <none>   12s     v1.18.4+k3s1
[root@k3s-master-7-91 ~]# kubectl get pod -A -o wide
NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE     IP          NODE                       NOMINATED NODE   READINESS GATES
kube-system   metrics-server-7566d596c8-swpmv          1/1     Running     0          6m12s   10.42.0.5   k3s-master-7-91.host.com   <none>           <none>
kube-system   helm-install-traefik-f4vvw               0/1     Completed   0          6m12s   10.42.0.2   k3s-master-7-91.host.com   <none>           <none>
kube-system   coredns-8655855d6-bzmhg                  1/1     Running     0          6m12s   10.42.0.3   k3s-master-7-91.host.com   <none>           <none>
kube-system   svclb-traefik-pj5m6                      2/2     Running     0          5m22s   10.42.0.7   k3s-master-7-91.host.com   <none>           <none>
kube-system   traefik-758cd5fc85-wbndp                 1/1     Running     0          5m22s   10.42.0.6   k3s-master-7-91.host.com   <none>           <none>
kube-system   local-path-provisioner-6d59f47c7-kw94k   1/1     Running     0          6m12s   10.42.0.4   k3s-master-7-91.host.com   <none>           <none>
kube-system   svclb-traefik-9dj5h                      2/2     Running     0          4m31s   10.42.1.2   k3s-master-7-92.host.com   <none>           <none>
kube-system   svclb-traefik-5gsg7                      2/2     Running     0          3m23s   10.42.3.2   k3s-master-7-93.host.com   <none>           <none>
kube-system   svclb-traefik-qbbrr                      2/2     Running     0          2m35s   10.42.4.2   k3s-node-7-94.host.com     <none>           <none>
kube-system   svclb-traefik-69fl9                      2/2     Running     0          91s     10.42.5.2   k3s-node-7-95.host.com     <none>           <none>
---
[root@k3s-master-7-91 ~]# kubectl label node k3s-node-7-94.host.com node-role.kubernetes.io/node=
[root@k3s-master-7-91 ~]# kubectl label node k3s-node-7-95.host.com node-role.kubernetes.io/node=
[root@k3s-master-7-91 ~]# kubectl get node
NAME                       STATUS   ROLES    AGE     VERSION
k3s-master-7-93.host.com   Ready    master   5m39s   v1.18.4+k3s1
k3s-master-7-91.host.com   Ready    master   8m32s   v1.18.4+k3s1
k3s-master-7-92.host.com   Ready    master   6m47s   v1.18.4+k3s1
k3s-node-7-94.host.com     Ready    node     4m52s   v1.18.4+k3s1
k3s-node-7-95.host.com     Ready    node     3m47s   v1.18.4+k3s1
```

### 2.3. 修改k3s组件

k3s 默认采用了 containerd 作为 runtime，部分场景中，比如开发环境，采用 docker 可能更加合适。另外CNI网络插件flannel可以修改其工作模式，比如改为 host-gw ，甚至替换为 calico 作为插件。

本实验将修改 runtime 为 docker-ce，修改网络插件为 calico，虽然可以在集群部署完毕后更改配置，但是在有选择的情况下，优先考虑在安装过程中(确切来说是在k3s初次启动前)修改，数据库仍然采用外置的 mysql。

#### 2.3.1. 主机清单

| **主机名**               | **IP地址** | **系统版本**                  | **角色**                           |
| ------------------------ | ---------- | ----------------------------- | ---------------------------------- |
| devops-7-3.host.com      | 10.4.7.3   | CentOS Linux release 7.5.1804 | DNS服务器，跳板机，Nginx负载均衡器 |
| k3s-db-7-90.host.com     | 10.4.7.90  | CentOS Linux release 7.5.1804 | MySQL数据库                        |
| k3s-master-7-91.host.com | 10.4.7.91  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-master-7-92.host.com | 10.4.7.92  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-master-7-93.host.com | 10.4.7.93  | CentOS Linux release 7.5.1804 | k3s master 节点                    |
| k3s-node-7-94.host.com   | 10.4.7.94  | CentOS Linux release 7.5.1804 | k3s node 节点                      |
| k3s-node-7-95.host.com   | 10.4.7.95  | CentOS Linux release 7.5.1804 | k3s node 节点                      |

#### 2.3.2. 安装docker-ce

```
# scan_host.sh 是运维脚本，里面是用for循环实现的多进程批量执行命令
[root@devops-7-3 ~]# scan_host.sh cmd -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 "curl -sfL https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo > /etc/yum.repos.d/docker-ce.repo"
[root@devops-7-3 ~]# scan_host.sh cmd -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 "yum install -q -y docker-ce; echo $?"
[root@devops-7-3 ~]# scan_host.sh cmd -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 "docker version 2>&1| grep Version" | xargs -n 3
10.4.7.94 Version: 19.03.11
10.4.7.95 Version: 19.03.11
10.4.7.93 Version: 19.03.11
10.4.7.92 Version: 19.03.11
10.4.7.91 Version: 19.03.11
# k3s不支持systemd驱动的cgroup : "exec-opts": ["native.cgroupdriver=systemd"] 
[root@devops-7-3 ~]# vim daemon.json 
{
  "graph": "/data/docker",
  "storage-driver": "overlay2",
  "insecure-registries": ["harbor.od.com"],
  "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
  "live-restore": true
}
[root@devops-7-3 ~]# scan_host.sh cmd -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 "mkdir /etc/docker"
[root@devops-7-3 ~]# scan_host.sh push -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 daemon.json /etc/docker/
[root@devops-7-3 ~]# scan_host.sh cmd -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 "systemctl enable docker ; systemctl start docker"
```

#### 2.3.3. k3s 部署准备

```
# 中间件准备
https://www.yuque.com/duduniao/k8s/luxw91#fnF9w
# 安装脚本准备
# 因为本实验会采用代理下载软件，而代理会被脚本写入k3s启动环境中，进而导致calico网络异常
# 如果没有配置代理，可以忽略该步骤
[root@devops-7-3 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@devops-7-3 ~]# curl -sfL https://get.k3s.io > k3s_install.sh 
[root@devops-7-3 ~]# vim k3s_install.sh  # 在create_env_file函数中注释掉代理配置
create_env_file() {
    info "env: Creating environment file ${FILE_K3S_ENV}"
    UMASK=$(umask)
    umask 0377
    env | grep '^K3S_' | $SUDO tee ${FILE_K3S_ENV} >/dev/null
    # env | egrep -i '^(NO|HTTP|HTTPS)_PROXY' | $SUDO tee -a ${FILE_K3S_ENV} >/dev/null
    umask $UMASK
}
# calico 清单文件准备
[root@devops-7-3 ~]# curl -sfL https://docs.projectcalico.org/manifests/calico.yaml > calico.yaml
[root@devops-7-3 ~]# vim calico.yaml # 在configmap中增加container_settings对象
......
"ipam": {
    "type": "calico-ipam"
},
"container_settings": { 
    "allow_ip_forwarding": true
},
......
[root@devops-7-3 ~]# vim calico.yaml # 修改CALICO_IPV4POOL_CIDR，k3s的pod-cird默认为10.42.0.0/16
......
- name: CALICO_IPV4POOL_CIDR
  value: "10.42.0.0/16"
......
# 批量下发文件
[root@devops-7-3 ~]# scan_host.sh push -h 10.4.7.91 10.4.7.92 10.4.7.93 10.4.7.94 10.4.7.95 calico.yaml k3s_install.sh /tmp/
10.4.7.94     calico.yaml k3s_install.sh --> /tmp/ Y
10.4.7.91     calico.yaml k3s_install.sh --> /tmp/ Y
10.4.7.95     calico.yaml k3s_install.sh --> /tmp/ Y
10.4.7.92     calico.yaml k3s_install.sh --> /tmp/ Y
10.4.7.93     calico.yaml k3s_install.sh --> /tmp/ Y
```

#### 2.3.4. 部署master

```
# master-1
[root@k3s-master-7-91 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-master-7-91 ~]# cat /tmp/k3s_install.sh | sh -s - server --docker --flannel-backend=none --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s
[root@k3s-master-7-91 ~]# kubectl get node -A
NAME                       STATUS     ROLES    AGE   VERSION
k3s-master-7-91.host.com   NotReady   master   42s   v1.18.4+k3s1
[root@k3s-master-7-91 ~]# kubectl get pod -A  # 因为没有 CNI 插件，所有pod都是Pending状态
NAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
kube-system   coredns-8655855d6-ptq24                  0/1     Pending   0          5s
kube-system   local-path-provisioner-6d59f47c7-59h7h   0/1     Pending   0          5s
kube-system   metrics-server-7566d596c8-fvpcp          0/1     Pending   0          5s
kube-system   helm-install-traefik-f5sxf               0/1     Pending   0          5s
[root@k3s-master-7-91 ~]# kubectl apply -f /tmp/calico.yaml # 应用calico 资源清单
[root@k3s-master-7-91 ~]# kubectl get pod -A  # 等待 calico 初始化完毕后，容器就能正常启动
NAMESPACE     NAME                                       READY   STATUS     RESTARTS   AGE
kube-system   coredns-8655855d6-ptq24                    0/1     Pending    0          94s
kube-system   local-path-provisioner-6d59f47c7-59h7h     0/1     Pending    0          94s
kube-system   metrics-server-7566d596c8-fvpcp            0/1     Pending    0          94s
kube-system   helm-install-traefik-f5sxf                 0/1     Pending    0          94s
kube-system   calico-node-c6hx8                          0/1     Init:0/3   0          28s
kube-system   calico-kube-controllers-76d4774d89-d6987   0/1     Pending    0          28s
# master-2
[root@k3s-master-7-92 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-master-7-92 ~]# cat /tmp/k3s_install.sh | sh -s - server --docker --flannel-backend=none --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s'
# master-3
[root@k3s-master-7-93 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-master-7-93 ~]# cat /tmp/k3s_install.sh | sh -s - server --docker --flannel-backend=none --datastore-endpoint='mysql://k3s:Duduniao.2020@tcp(10.4.7.90:3306)/k3s'
```

#### 2.3.5. 部署node

```
[root@k3s-master-7-91 ~]# cat /var/lib/rancher/k3s/server/node-token 
K101f067e74ca73472e4f23777e74a28caa5980142837ff7689d8a92f611a1fce33::server:369ee527230cc7f821e7e73399ca4678
# node-1
[root@k3s-node-7-94 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-node-7-94 ~]# cat /tmp/k3s_install.sh | K3S_URL=https://k3s-api.od.com:6443 K3S_TOKEN='K101f067e74ca73472e4f23777e74a28caa5980142837ff7689d8a92f611a1fce33::server:369ee527230cc7f821e7e73399ca4678' sh -s - --docker

# node-2
[root@k3s-node-7-95 ~]# export http_proxy="10.4.7.1:10080" ; export https_proxy="10.4.7.1:10080"
[root@k3s-node-7-95 ~]# cat /tmp/k3s_install.sh | K3S_URL=https://k3s-api.od.com:6443 K3S_TOKEN='K101f067e74ca73472e4f23777e74a28caa5980142837ff7689d8a92f611a1fce33::server:369ee527230cc7f821e7e73399ca4678' sh -s - --docker
```

#### 2.3.6. 查看集群状态

```
[root@k3s-master-7-91 ~]# kubectl get pod -A -o wide
NAMESPACE     NAME                                       READY   STATUS      RESTARTS   AGE     IP              NODE                       NOMINATED NODE   READINESS GATES
kube-system   calico-node-c6hx8                          1/1     Running     0          32m     10.4.7.91       k3s-master-7-91.host.com   <none>           <none>
kube-system   metrics-server-7566d596c8-fvpcp            1/1     Running     0          33m     10.42.215.129   k3s-master-7-91.host.com   <none>           <none>
kube-system   calico-kube-controllers-76d4774d89-d6987   1/1     Running     0          32m     10.42.215.130   k3s-master-7-91.host.com   <none>           <none>
kube-system   coredns-8655855d6-ptq24                    1/1     Running     0          33m     10.42.215.131   k3s-master-7-91.host.com   <none>           <none>
kube-system   local-path-provisioner-6d59f47c7-59h7h     1/1     Running     0          33m     10.42.215.132   k3s-master-7-91.host.com   <none>           <none>
kube-system   calico-node-rlzwl                          1/1     Running     0          26m     10.4.7.92       k3s-master-7-92.host.com   <none>           <none>
kube-system   calico-node-wsz8x                          1/1     Running     0          25m     10.4.7.93       k3s-master-7-93.host.com   <none>           <none>
kube-system   helm-install-traefik-f5sxf                 0/1     Completed   0          33m     10.42.215.133   k3s-master-7-91.host.com   <none>           <none>
kube-system   svclb-traefik-76nxx                        2/2     Running     0          14m     10.42.252.1     k3s-master-7-92.host.com   <none>           <none>
kube-system   svclb-traefik-rl8st                        2/2     Running     0          14m     10.42.215.134   k3s-master-7-91.host.com   <none>           <none>
kube-system   traefik-758cd5fc85-wfqlw                   1/1     Running     0          14m     10.42.161.65    k3s-master-7-93.host.com   <none>           <none>
kube-system   svclb-traefik-r4448                        2/2     Running     0          14m     10.42.161.66    k3s-master-7-93.host.com   <none>           <none>
kube-system   calico-node-ts825                          1/1     Running     0          18m     10.4.7.94       k3s-node-7-94.host.com     <none>           <none>
kube-system   svclb-traefik-9f4b8                        2/2     Running     0          13m     10.42.45.65     k3s-node-7-94.host.com     <none>           <none>
kube-system   calico-node-84tnl                          1/1     Running     0          10m     10.4.7.95       k3s-node-7-95.host.com     <none>           <none>
kube-system   svclb-traefik-bfml5                        2/2     Running     0          8m16s   10.42.115.65    k3s-node-7-95.host.com     <none>           <none>
```

### 2.4. 离线安装

K3S 离线安装用于解决软件包无法下载或者下载速度太慢问题，安装过程分为三个步骤

- 导入二进制文件和离线镜像包
- 编写 service 脚本和 shell 脚本
- 启动k3s服务

下面以 单机版 k3s master 为例，node 节点操作差别仅在 service 脚本不同。

#### 2.4.1. 导入离线包

下载地址: https://github.com/rancher/k3s/releases

下载 k3s 二进制文件及其 [k3s-airgap-images-amd64.tar](https://github.com/rancher/k3s/releases/download/v1.18.4%2Bk3s1/k3s-airgap-images-amd64.tar) 镜像包

```
[root@devops-7-3 ~]# scp k3s-airgap-images-amd64.tar k3s 10.4.7.91:/tmp/

[root@k3s-master-7-91 ~]# mv /tmp/k3s /usr/local/bin/
[root@k3s-master-7-91 ~]# chmod 755 /usr/local/bin/k3s        
[root@k3s-master-7-91 ~]# ln -s /usr/local/bin/k3s /usr/local/bin/kubectl
[root@k3s-master-7-91 ~]# ln -s /usr/local/bin/k3s /usr/local/bin/crictl
[root@k3s-master-7-91 ~]# ln -s /usr/local/bin/k3s /usr/local/bin/ctr
[root@k3s-master-7-91 ~]# ll /usr/local/bin
total 52348
lrwxrwxrwx 1 root root       18 2020-06-25 04:31:48 crictl -> /usr/local/bin/k3s
lrwxrwxrwx 1 root root       18 2020-06-25 04:31:55 ctr -> /usr/local/bin/k3s
-rwxr-xr-x 1 root root 53604352 2020-06-25 04:30:18 k3s
lrwxrwxrwx 1 root root       18 2020-06-25 04:31:18 kubectl -> /usr/local/bin/k3s
[root@k3s-master-7-91 ~]# mkdir -p /var/lib/rancher/k3s/agent/images/
[root@k3s-master-7-91 ~]# mv /tmp/k3s-airgap-images-amd64.tar /var/lib/rancher/k3s/agent/images/
```

#### 2.4.2. 配置脚本

```
[root@k3s-master-7-91 ~]# vim /etc/systemd/system/k3s.service
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
Type=notify
KillMode=process
Delegate=yes
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0
Restart=always
RestartSec=5s
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s server
[root@k3s-master-7-91 ~]# vim /usr/local/bin/k3s-killall.sh
#!/bin/sh
[ $(id -u) -eq 0 ] || exec sudo $0 $@

for bin in /var/lib/rancher/k3s/data/**/bin/; do
    [ -d $bin ] && export PATH=$PATH:$bin:$bin/aux
done

set -x

for service in /etc/systemd/system/k3s*.service; do
    [ -s $service ] && systemctl stop $(basename $service)
done

for service in /etc/init.d/k3s*; do
    [ -x $service ] && $service stop
done

pschildren() {
    ps -e -o ppid= -o pid= | \
    sed -e 's/^\s*//g; s/\s\s*/\t/g;' | \
    grep -w "^$1" | \
    cut -f2
}

pstree() {
    for pid in $@; do
        echo $pid
        for child in $(pschildren $pid); do
            pstree $child
        done
    done
}

killtree() {
    kill -9 $(
        { set +x; } 2>/dev/null;
        pstree $@;
        set -x;
    ) 2>/dev/null
}

getshims() {
    ps -e -o pid= -o args= | sed -e 's/^ *//; s/\s\s*/\t/;' | grep -w 'k3s/data/[^/]*/bin/containerd-shim' | cut -f1
}

killtree $({ set +x; } 2>/dev/null; getshims; set -x)

do_unmount() {
    { set +x; } 2>/dev/null
    MOUNTS=
    while read ignore mount ignore; do
        MOUNTS="$mount\n$MOUNTS"
    done </proc/self/mounts
    MOUNTS=$(printf $MOUNTS | grep "^$1" | sort -r)
    if [ -n "${MOUNTS}" ]; then
        set -x
        umount ${MOUNTS}
    else
        set -x
    fi
}

do_unmount '/run/k3s'
do_unmount '/var/lib/rancher/k3s'
do_unmount '/var/lib/kubelet/pods'
do_unmount '/run/netns/cni-'

# Delete network interface(s) that match 'master cni0'
ip link show 2>/dev/null | grep 'master cni0' | while read ignore iface ignore; do
    iface=${iface%%@*}
    [ -z "$iface" ] || ip link delete $iface
done
ip link delete cni0
ip link delete flannel.1
rm -rf /var/lib/cni/
iptables-save | grep -v KUBE- | grep -v CNI- | iptables-restore
[root@k3s-master-7-91 ~]# vim /usr/local/bin/k3s-uninstall.sh
#!/bin/sh
set -x
[ $(id -u) -eq 0 ] || exec sudo $0 $@

/usr/local/bin/k3s-killall.sh

if which systemctl; then
    systemctl disable k3s
    systemctl reset-failed k3s
    systemctl daemon-reload
fi
if which rc-update; then
    rc-update delete k3s default
fi

rm -f /etc/systemd/system/k3s.service
rm -f /etc/systemd/system/k3s.service.env

remove_uninstall() {
    rm -f /usr/local/bin/k3s-uninstall.sh
}
trap remove_uninstall EXIT

if (ls /etc/systemd/system/k3s*.service || ls /etc/init.d/k3s*) >/dev/null 2>&1; then
    set +x; echo 'Additional k3s services installed, skipping uninstall of k3s'; set -x
    exit
fi

for cmd in kubectl crictl ctr; do
    if [ -L /usr/local/bin/$cmd ]; then
        rm -f /usr/local/bin/$cmd
    fi
done

rm -rf /etc/rancher/k3s
rm -rf /var/lib/rancher/k3s
rm -rf /var/lib/kubelet
rm -f /usr/local/bin/k3s
rm -f /usr/local/bin/k3s-killall.sh
```

#### 2.4.3. 启动服务

```
[root@k3s-master-7-91 ~]# systemctl daemon-reload 
[root@k3s-master-7-91 ~]# systemctl start k3s ; systemctl enable k3s
[root@k3s-master-7-91 ~]# kubectl get node
NAME                       STATUS   ROLES    AGE   VERSION
k3s-master-7-91.host.com   Ready    master   23s   v1.18.4+k3s1
[root@k3s-master-7-91 ~]# kubectl get cs
NAME                 STATUS    MESSAGE   ERROR
scheduler            Healthy   ok        
controller-manager   Healthy   ok        
[root@k3s-master-7-91 ~]# kubectl get pod -A
NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
kube-system   metrics-server-7566d596c8-lxtkl          1/1     Running     0          40s
kube-system   local-path-provisioner-6d59f47c7-hnxqp   1/1     Running     0          40s
kube-system   helm-install-traefik-kbght               0/1     Completed   0          40s
kube-system   svclb-traefik-d4j9x                      2/2     Running     0          21s
kube-system   coredns-8655855d6-7cm4p                  1/1     Running     0          40s
kube-system   traefik-758cd5fc85-n6pwb                 1/1     Running     0          21s
```