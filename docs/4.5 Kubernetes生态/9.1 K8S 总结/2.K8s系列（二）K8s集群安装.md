相关博文：

- [k8s集群部署高可用完整版](https://www.cnblogs.com/yuezhimi/p/12931002.html)
- [kubernetes 1.16 二进制集群高可用安装实操踩坑篇](https://blog.csdn.net/sfdst/article/details/105813485)
- [冰河教你一次性成功安装K8S集群（基于一主两从模式）](https://www.cnblogs.com/binghe001/p/14083312.html)
- [K8S集群的安装](https://www.cnblogs.com/MarkGuo/p/14163469.html)
- [Kubernetes容器集群管理环境 - Node节点的移除与加入](https://www.cnblogs.com/kevingrace/p/11302555.html)
- [Kubernetes容器集群管理环境 - 完整部署（上篇）](https://www.cnblogs.com/kevingrace/p/10961264.html)
- [Kubernetes容器集群管理环境 - 完整部署（中篇）](https://www.cnblogs.com/kevingrace/p/11043042.html)
- [Kubernetes容器集群管理环境 - 完整部署（下篇）](https://www.cnblogs.com/kevingrace/p/10995648.html)



# 一、服务器规划

| IP              | 主机名    | 节点       | 操作系统        |
| --------------- | --------- | ---------- | --------------- |
| 192.168.175.101 | binghe101 | K8S Master | CentOS 8.0.1905 |
| 192.168.175.102 | binghe102 | K8S Worker | CentOS 8.0.1905 |
| 192.168.175.103 | binghe103 | K8S Worker | CentOS 8.0.1905 |

# 二、安装环境版本

| 软件名称       | 软件版本 | 说明                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| Docker         | 19.03.8  | 提供容器环境                                                 |
| docker-compose | 1.25.5   | 定义和运行由多个容器组成的应用                               |
| K8S            | 1.8.12   | 是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。 |

# 三、服务器免密码登录

在各服务器执行如下命令。

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

将binghe102和binghe103服务器上的id_rsa.pub文件复制到binghe101服务器。

```bash
[root@binghe102 ~]# scp /root/.ssh/id_rsa.pub binghe101:/root/.ssh/102
[root@binghe103 ~]# scp /root/.ssh/id_rsa.pub binghe101:/root/.ssh/103
```

在binghe101服务器上执行如下命令。

```bash
cat /root/.ssh/142 >> ~/.ssh/authorized_keys
cat /root/.ssh/144 >> ~/.ssh/authorized_keys
```

然后将authorized_keys文件分别复制到binghe102、binghe103服务器。

```bash
[root@binghe101 ~]# scp /root/.ssh/authorized_keys binghe102:/root/.ssh/authorized_keys
[root@binghe101 ~]# scp /root/.ssh/authorized_keys binghe103:/root/.ssh/authorized_keys
```

删除binghe101节点上~/.ssh下的102和103文件。

```bash
rm ~/.ssh/102
rm ~/.ssh/103
```

# 四、部署nginx负载均衡（与Haproxy+Keepalive二选一）

```bash
rpm -vih http://nginx.org/packages/rhel/7/x86_64/RPMS/nginx-1.16.0-1.el7.ngx.x86_64.rpm
# vim /etc/nginx/nginx.conf
……
stream {

    log_format  main  '$remote_addr $upstream_addr - [$time_local] $status $upstream_bytes_sent';

    access_log  /var/log/nginx/k8s-access.log  main;

    upstream k8s-apiserver {
                server 192.168.0.131:6443;
                server 192.168.0.132:6443;
            }
    
    server {
       listen 6443;
       proxy_pass k8s-apiserver;
    }
}
……

#启动nginx
systemctl start nginx
systemctl enable nginx
```

## 4.1 Nginx+keepalived高可用

```bash
###主节点
# yum install keepalived
# vi /etc/keepalived/keepalived.conf
global_defs { 
   notification_email { 
     acassen@firewall.loc 
     failover@firewall.loc 
     sysadmin@firewall.loc 
   } 
   notification_email_from Alexandre.Cassen@firewall.loc  
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30 
   router_id NGINX_MASTER
} 

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
}

vrrp_instance VI_1 { 
    state MASTER 
    interface ens32
    virtual_router_id 51 # VRRP 路由 ID实例，每个实例是唯一的 
    priority 100    # 优先级，备服务器设置 90 
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒 
    authentication { 
        auth_type PASS      
        auth_pass 1111 
    }  
    virtual_ipaddress { 
        192.168.0.130/24
    } 
    track_script {
        check_nginx
    } 
}

# cat /etc/keepalived/check_nginx.sh 
#!/bin/bash
count=$(ps -ef |grep nginx |egrep -cv "grep|$$")

if [ "$count" -eq 0 ];then
    exit 1
else
    exit 0
fi
# systemctl start keepalived
# systemctl enable keepalived

###备节点
#vim /etc/keepalived/keepalived.conf
global_defs { 
   notification_email { 
     acassen@firewall.loc 
     failover@firewall.loc 
     sysadmin@firewall.loc 
   } 
   notification_email_from Alexandre.Cassen@firewall.loc  
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30 
   router_id NGINX_BACKUP
} 

vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
}

vrrp_instance VI_1 { 
    state BACKUP 
    interface ens32
    virtual_router_id 51 # VRRP 路由 ID实例，每个实例是唯一的 
    priority 90    # 优先级，备服务器设置 90 
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒 
    authentication { 
        auth_type PASS      
        auth_pass 1111 
    }  
    virtual_ipaddress { 
        192.168.0.130/24
    } 
    track_script {
        check_nginx
    } 
}

# cat /etc/keepalived/check_nginx.sh 
#!/bin/bash
count=$(ps -ef |grep nginx |egrep -cv "grep|$$")

if [ "$count" -eq 0 ];then
    exit 1
else
    exit 0
fi

# systemctl start keepalived
# systemctl enable keepalived

###测试VIP是否正常工作
curl -k --header "Authorization: Bearer 8762670119726309a80b1fe94eb66e93" https://192.168.0.130:6443/version
{
  "major": "1",
  "minor": "18",
  "gitVersion": "v1.18.2",
  "gitCommit": "52c56ce7a8272c798dbc29846288d7cd9fbae032",
  "gitTreeState": "clean",
  "buildDate": "2020-04-16T11:48:36Z",
  "goVersion": "go1.13.9",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```



## 4.2 Haproxy+keepalive搭建高可用

### 4.2.1 安装配置haproxy服务

```bash
1.安装haproxy
[root@k8s-master01 ~]# yum install -y  haproxy
2.配置haproxy
cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.ori
cat /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /var/run/haproxy-admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
    nbproc 1
defaults
    log global
    timeout connect 5000
    timeout client 10m
    timeout server 10m
listen admin_stats
    bind 0.0.0.0:10080
    mode http
    log 127.0.0.1 local0 err
    stats refresh 30s
    stats uri /status
    stats realm welcome login\ Haproxy
    stats auth along:along123
    stats hide-version
    stats admin if TRUE
listen kube-master
    bind 0.0.0.0:8443
    mode tcp
    option tcplog
    balance source
    server 192.168.10.11 192.168.10.11:6443 check inter 2000 fall 2 rise 2 weight 1
    server 192.168.10.12 192.168.10.12:6443 check inter 2000 fall 2 rise 2 weight 1
    server 192.168.10.13 192.168.10.13:6443 check inter 2000 fall 2 rise 2 weight 1

3.启动haproxy
systemctl restart haproxy
[root@k8s-master01 ~]# systemctl enable haproxy
Created symlink from /etc/systemd/system/multi-user.target.wants/haproxy.service to /usr/lib/systemd/system/haproxy.service.
[root@k8s-master01 ~]# systemctl status haproxy
● haproxy.service - HAProxy Load Balancer
   Loaded: loaded (/usr/lib/systemd/system/haproxy.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2020-04-10 16:35:46 CST; 24s ago
 Main PID: 10235 (haproxy-systemd)
   CGroup: /system.slice/haproxy.service
           ├─10235 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
           ├─10236 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
           └─10237 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
```



# 五、装Docker环境

**本文档基于Docker 19.03.8 版本搭建Docker环境。**

在所有服务器上创建install_docker.sh脚本，脚本内容如下所示。

```bash
#!/bin/bash
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com
dnf install yum*
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
dnf install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.1.el7.x86_64.rpm
#指定Docker的版本进行安装 
yum install -y docker-ce-19.03.8 docker-ce-cli-19.03.8

systemctl enable docker.service
systemctl start docker.service


#配置docker镜像加速
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zz3sblpi.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

使用如下命令赋予auto_install_docker.sh文件可执行权限。

```bash
chmod a+x ./auto_install_docker.sh
```



# 六、安装K8S集群环境

**本文档基于K8S 1.8.12版本来搭建K8S集群**

## 6.1 安装K8S基础环境

在所有服务器上创建install_k8s.sh脚本文件，脚本文件的内容如下所示。

```bash
#!/bin/bash
#设置hosts
cat >> /etc/hosts << EOF
192.168.0.140 master
192.168.0.142 slave3
192.168.0.144 slave4
EOF

#安装nfs-utils
yum install -y nfs-utils
yum install -y wget

#同步系统时间：
yum install -y ntpdate
ntpdate time.windows.com

#启动nfs-server
systemctl start nfs-server
systemctl enable nfs-server

#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#关闭SeLinux
#临时关闭
setenforce 0	
#永久关闭
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap
# 临时关闭
swapoff -a
#永久关闭
sed -i 's/.*swap.*/#&/' /etc/fstab
cat /etc/fstab

#修改 /etc/sysctl.conf
# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.disable_ipv6.*#net.ipv6.conf.all.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.default.disable_ipv6.*#net.ipv6.conf.default.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.lo.disable_ipv6.*#net.ipv6.conf.lo.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.forwarding.*#net.ipv6.conf.all.forwarding=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p

# 配置K8S的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 卸载旧版本K8S
yum remove -y kubelet kubeadm kubectl

# 安装kubelet、kubeadm、kubectl，这里我安装的是1.18.2版本
yum install -y kubelet-1.18.2 kubeadm-1.18.2 kubectl-1.18.2

# 修改docker Cgroup Driver为systemd
# # 将/usr/lib/systemd/system/docker.service文件中的这一行 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# # 修改为 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
# 如果不修改，在添加 worker 节点时可能会碰到如下错误
# [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 
# Please follow the guide at https://kubernetes.io/docs/setup/cri/
sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service

# 设置 docker 镜像，提高 docker 镜像下载速度和稳定性
# 如果访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤
# curl -sSL https://kuboard.cn/install-script/set_mirror.sh | sh -s ${REGISTRY_MIRROR}

# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet

docker version
```

在每台服务器上为install_k8s.sh脚本赋予可执行权限，并执行脚本即可。



## 6.2 综合安装脚本

```bash
#!/bin/bash
#安装Docker
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com
dnf install yum*
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
dnf install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.1.el7.x86_64.rpm
#指定Docker的版本进行安装 
yum install -y docker-ce-19.03.8 docker-ce-cli-19.03.8

systemctl enable docker.service
systemctl start docker.service


#配置docker镜像加速
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zz3sblpi.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker


#基础环境配置
#设置hosts
cat >> /etc/hosts << EOF
192.168.0.140 master
192.168.0.142 slave3
192.168.0.144 slave4
EOF

#安装nfs-utils
yum install -y nfs-utils
yum install -y wget

#同步系统时间：
yum install -y ntpdate
ntpdate time.windows.com

#启动nfs-server
systemctl start nfs-server
systemctl enable nfs-server

#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#关闭SeLinux
#临时关闭
setenforce 0	
#永久关闭
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap
# 临时关闭
swapoff -a
#永久关闭
sed -i 's/.*swap.*/#&/' /etc/fstab
cat /etc/fstab

#修改 /etc/sysctl.conf
# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.disable_ipv6.*#net.ipv6.conf.all.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.default.disable_ipv6.*#net.ipv6.conf.default.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.lo.disable_ipv6.*#net.ipv6.conf.lo.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.forwarding.*#net.ipv6.conf.all.forwarding=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p

# 配置K8S的yum源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 卸载旧版本K8S
yum remove -y kubelet kubeadm kubectl

# 安装kubelet、kubeadm、kubectl，这里我安装的是1.18.2版本
yum install -y kubelet-1.18.2 kubeadm-1.18.2 kubectl-1.18.2

# 修改docker Cgroup Driver为systemd
# # 将/usr/lib/systemd/system/docker.service文件中的这一行 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
# # 修改为 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd
# 如果不修改，在添加 worker 节点时可能会碰到如下错误
# [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 
# Please follow the guide at https://kubernetes.io/docs/setup/cri/
sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service

# 设置 docker 镜像，提高 docker 镜像下载速度和稳定性
# 如果访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤
# curl -sSL https://kuboard.cn/install-script/set_mirror.sh | sh -s ${REGISTRY_MIRROR}

# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet

docker version
```



# 七、初始化Master节点

**只在binghe101服务器上执行的操作。**

## 7.1 初始化Master节点的网络环境

注意：下面的命令需要在命令行手动执行。

```bash
# 只在 master 节点执行
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=192.168.175.101
# 替换 k8s.master 为 您想要的 dnsName
export APISERVER_NAME=k8s.master
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于物理网络中
export POD_SUBNET=172.18.0.1/16
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
```

## 7.2 初始化Master节点

在binghe101服务器上创建init_master.sh脚本文件，文件内容如下所示。

```bash
#!/bin/bash
# 脚本出错时终止执行
set -e

if [ ${#POD_SUBNET} -eq 0 ] || [ ${#APISERVER_NAME} -eq 0 ]; then
  echo -e "\033[31;1m请确保您已经设置了环境变量 POD_SUBNET 和 APISERVER_NAME \033[0m"
  echo 当前POD_SUBNET=$POD_SUBNET
  echo 当前APISERVER_NAME=$APISERVER_NAME
  exit 1
fi


# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
rm -f ./kubeadm-config.yaml
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.2
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "${APISERVER_NAME}:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "${POD_SUBNET}"
  dnsDomain: "cluster.local"
EOF

# kubeadm init
# 根据服务器网速的情况，您需要等候 3 - 10 分钟
kubeadm init --config=kubeadm-config.yaml --upload-certs

# 配置 kubectl
rm -rf /root/.kube/
mkdir /root/.kube/
cp -i /etc/kubernetes/admin.conf /root/.kube/config

# 安装 calico 网络插件
# 参考文档 https://docs.projectcalico.org/v3.13/getting-started/kubernetes/self-managed-onprem/onpremises
echo "安装calico-3.13.1"
rm -f calico-3.13.1.yaml
wget https://kuboard.cn/install-script/calico/calico-3.13.1.yaml
kubectl apply -f calico-3.13.1.yaml
```

赋予init_master.sh脚本文件可执行权限并执行脚本。

## 7.3 查看Master节点的初始化结果

**（1）确保所有容器组处于Running状态**

```bash
# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态
watch kubectl get pod -n kube-system -o wide
```

具体执行如下所示。

```bash
[root@binghe101 ~]# watch kubectl get pod -n kube-system -o wide
Every 2.0s: kubectl get pod -n kube-system -o wide                                                                                                                          binghe101: Sun May 10 11:01:32 2020

NAME                                       READY   STATUS    RESTARTS   AGE    IP                NODE        NOMINATED NODE   READINESS GATES          
calico-kube-controllers-5b8b769fcd-5dtlp   1/1     Running   0          118s   172.18.203.66     binghe101   <none>           <none>          
calico-node-fnv8g                          1/1     Running   0          118s   192.168.175.101   binghe101   <none>           <none>          
coredns-546565776c-27t7h                   1/1     Running   0          2m1s   172.18.203.67     binghe101   <none>           <none>          
coredns-546565776c-hjb8z                   1/1     Running   0          2m1s   172.18.203.65     binghe101   <none>           <none>          
etcd-binghe101                             1/1     Running   0          2m7s   192.168.175.101   binghe101   <none>           <none>          
kube-apiserver-binghe101                   1/1     Running   0          2m7s   192.168.175.101   binghe101   <none>           <none>          
kube-controller-manager-binghe101          1/1     Running   0          2m7s   192.168.175.101   binghe101   <none>           <none>          
kube-proxy-dvgsr                           1/1     Running   0          2m1s   192.168.175.101   binghe101   <none>           <none>          
kube-scheduler-binghe101                   1/1     Running   0          2m7s   192.168.175.101   binghe101   <none>           <none>
```

**（2） 查看 Master 节点初始化结果**

```bash
kubectl get nodes -o wide
```

具体执行如下所示。

```bash
[root@binghe101 ~]# kubectl get nodes -o wide
NAME        STATUS   ROLES    AGE     VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION         CONTAINER-RUNTIME
binghe101   Ready    master   3m28s   v1.18.2   192.168.175.101   <none>        CentOS Linux 8 (Core)   4.18.0-80.el8.x86_64   docker://19.3.8
```

# 八、初始化Worker节点

## 8.1 获取join命令参数

在Master节点（binghe101服务器）上执行如下命令获取join命令参数。

```bash
kubeadm token create --print-join-command
```

具体执行如下所示。

```bash
[root@binghe101 ~]# kubeadm token create --print-join-command
W0510 11:04:34.828126   56132 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
kubeadm join k8s.master:6443 --token 8nblts.62xytoqufwsqzko2     --discovery-token-ca-cert-hash sha256:1717cc3e34f6a56b642b5751796530e367aa73f4113d09994ac3455e33047c0d 
```

其中，有如下一行输出。

```bash
kubeadm join k8s.master:6443 --token 8nblts.62xytoqufwsqzko2     --discovery-token-ca-cert-hash sha256:1717cc3e34f6a56b642b5751796530e367aa73f4113d09994ac3455e33047c0d 
```

这行代码就是获取到的join命令。

> 注意：join命令中的token的有效时间为 2 个小时，2小时内，可以使用此 token 初始化任意数量的 worker 节点。

## 8.2 初始化Worker节点

针对所有的 worker 节点执行，在这里，就是在binghe102服务器和binghe103服务器上执行。

在命令分别手动执行如下命令。

```bash
# 只在 worker 节点执行
# 192.168.175.101 为 master 节点的内网 IP
export MASTER_IP=192.168.175.101
# 替换 k8s.master 为初始化 master 节点时所使用的 APISERVER_NAME
export APISERVER_NAME=k8s.master
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# 替换为 master 节点上 kubeadm token create 命令输出的join
kubeadm join k8s.master:6443 --token 8nblts.62xytoqufwsqzko2     --discovery-token-ca-cert-hash sha256:1717cc3e34f6a56b642b5751796530e367aa73f4113d09994ac3455e33047c0d 
```

具体执行如下所示。

```bash
[root@binghe102 ~]# export MASTER_IP=192.168.175.101
[root@binghe102 ~]# export APISERVER_NAME=k8s.master
[root@binghe102 ~]# echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
[root@binghe102 ~]# kubeadm join k8s.master:6443 --token 8nblts.62xytoqufwsqzko2     --discovery-token-ca-cert-hash sha256:1717cc3e34f6a56b642b5751796530e367aa73f4113d09994ac3455e33047c0d 
W0510 11:08:27.709263   42795 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING FileExisting-tc]: tc not found in system path
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

根据输出结果可以看出，Worker节点加入了K8S集群。

> 注意：kubeadm join…就是master 节点上 kubeadm token create 命令输出的join。

## 8.3 查看初始化结果

在Master节点（binghe101服务器）执行如下命令查看初始化结果。

```bash
kubectl get nodes -o wide
```

具体执行如下所示。

```bash
[root@binghe101 ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE     VERSION
binghe101   Ready    master   20m     v1.18.2
binghe102   Ready    <none>   2m46s   v1.18.2
binghe103   Ready    <none>   2m46s   v1.18.2
```

> 注意：kubectl get nodes命令后面加上-o wide参数可以输出更多的信息。

> 

# 九、K8S安装ingress-nginx

**注意：在Master节点（binghe101服务器上执行）**

## 9.1 创建ingress-nginx命名空间

创建ingress-nginx-namespace.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    name: ingress-nginx
```

执行如下命令创建ingress-nginx命名空间。

```bash
kubectl apply -f ingress-nginx-namespace.yaml
```

## 9.2 安装ingress controller

创建ingress-nginx-mandatory.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: default-http-backend
  labels:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx
  namespace: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: default-http-backend
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: default-http-backend
        app.kubernetes.io/part-of: ingress-nginx
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: default-http-backend
          # Any image is permissible as long as:
          # 1. It serves a 404 page at /
          # 2. It serves 200 on a /healthz endpoint
          image: registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/defaultbackend-amd64:1.5
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 30
            timeoutSeconds: 5
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: 10m
              memory: 20Mi
            requests:
              cpu: 10m
              memory: 20Mi

---
apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx
spec:
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app.kubernetes.io/name: default-http-backend
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: registry.cn-qingdao.aliyuncs.com/kubernetes_xingej/nginx-ingress-controller:0.20.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1

---
```

执行如下命令安装ingress controller。

```bash
kubectl apply -f ingress-nginx-mandatory.yaml
```

## 9.3 安装K8S SVC：ingress-nginx

主要是用来用于暴露pod：nginx-ingress-controller。

创建service-nodeport.yaml文件，文件内容如下所示。ls

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
      nodePort: 30443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

执行如下命令安装。

```bash
kubectl apply -f service-nodeport.yaml
```

## 9.4 访问K8S SVC：ingress-nginx

查看ingress-nginx命名空间的部署情况，如下所示。

```bash
[root@binghe101 k8s]# kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS    RESTARTS   AGE
default-http-backend-796ddcd9b-vfmgn        1/1     Running   1          10h
nginx-ingress-controller-58985cc996-87754   1/1     Running   2          10h
```

在命令行服务器命令行输入如下命令查看ingress-nginx的端口映射情况。

```bash
kubectl get svc -n ingress-nginx 
```

具体如下所示。

```bash
[root@binghe101 k8s]# kubectl get svc -n ingress-nginx 
NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
default-http-backend   ClusterIP   10.96.247.2   <none>        80/TCP                       7m3s
ingress-nginx          NodePort    10.96.40.6    <none>        80:30080/TCP,443:30443/TCP   4m35s
```

所以，可以通过Master节点（binghe101服务器）的IP地址和30080端口号来访问ingress-nginx，如下所示。

```bash
[root@binghe101 k8s]# curl 192.168.175.101:30080       
default backend - 404
```

也可以在浏览器打开http://192.168.175.101:30080 来访问ingress-nginx，如下所示。
![img](https://img-blog.csdnimg.cn/20200521003920255.jpg#pic_center)

# 十、重启K8S集群引起的问题

## 10.1 Worker节点故障不能启动

Master 节点的 IP 地址发生变化，导致 worker 节点不能启动。需要重新安装K8S集群，并确保所有节点都有固定的内网 IP 地址。

## 10.2 Pod崩溃或不能正常访问

重启服务器后使用如下命令查看Pod的运行状态。

```bash
kubectl get pods --all-namespaces
```

发现很多 Pod 不在 Running 状态，此时，需要使用如下命令删除运行不正常的Pod。

```bash
kubectl delete pod <pod-name> -n <pod-namespece>
```

> 注意：如果Pod 是使用 Deployment、StatefulSet 等控制器创建的，K8S 将创建新的 Pod 作为替代，重新启动的 Pod 通常能够正常工作。

# 十一、kubernetes的node重新加入

注意:以下操作在node下操作

## 11.1 停掉kubelet

```bash
systemctl stop kubelet
```

## 11.2 删除之前的相关文件

```bash
rm -rf /etc/kubernetes/*
```

## 11.3 加入集群

```bash
kubeadm join 192.168.233.3:6443 
```