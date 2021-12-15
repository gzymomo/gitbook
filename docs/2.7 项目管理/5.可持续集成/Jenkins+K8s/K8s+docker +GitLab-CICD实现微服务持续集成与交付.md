- [K8s+docker +GitLab-CI/CD实现微服务持续集成与交付](https://blog.51cto.com/xiaozhagn/3544165)

**K8s+docker +GitLab-ci/cd持续集成与交付**

 

# 一、部署流程

开发人员把项目代码通过git推送到gitlab，触发gitla-runner自动从拉取gitlab上面拉取代码下来，然后进行build，编译、生成镜像、然后把镜像推送到Harbor仓库；然后在部署的时候通过k8s拉取Harbor上面的代码进行创建容器和服务，最终发布完成，然后可以用外部访问

更多的K8s技术文档可以关注-公众号-[ it小疯子](https://s4.51cto.com/images/blog/202108/19/b9fb3390d7fd660f17acbee6848cda5e.jpeg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

部署流程如下：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/859f8a84a2216eaa13a7f421255cbd32.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

环境准备：

| IP          | 角色                         |
| ----------- | ---------------------------- |
| 172.25.0.30 | master                       |
| 172.25.0.31 | node1、Gitlab                |
| 172.25.0.32 | node2、Harbor、gitlab-runner |

 

# 二、K8s 安装

## 1. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

 

- 一台或多台机器，操作系统x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 可以访问外部网洛，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止swap分区

## 2. 准备环境

 

| 角色       | IP          | k8s版本          |
| ---------- | ----------- | ---------------- |
| k8s-master | 172.25.0.30 | kubernetes1.21.0 |
| k8s-node1  | 172.25.0.31 | kubernetes1.21.0 |
| k8s-node2  | 172.25.0.32 | kubernetes1.21.0 |

 

\# 关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

\# 关闭selinux

```bash
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久

setenforce 0  # 临时1.2.3.
```

\# 关闭swap

```bash
swapoff -a  # 临时

sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久1.2.3.
```

\# 根据规划设置主机名

```bash
hostnamectl set-hostname <hostname>
```

\# 在master添加hosts

```bash
cat >> /etc/hosts << EOF

172.25.0.30 k8s-master

172.25.0.31 k8s-node1

172.25.0.32 k8s-node2

EOF
```

\# 将桥接的IPv4流量传递到iptables的链

```bash
cat > /etc/sysctl.d/k8s.conf << EOF

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

vm.swappiness=0

net.ipv4.ip_forward=1

net.ipv4.tcp_tw_recycle=0

EOF

sysctl --system  # 生效
```

 

\# 时间同步

```bash
yum install ntpdate -y

ntpdate -u pool.ntp.org
```

添加定时

```bash
crontab  -l

*/20 * * * * /sbin/ntpdate -u pool.ntp.org > /dev/null 2>&1
```

## 3. 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

### 3.1 安装Docker

```bash
#yum install -y yum-utils \

           device-mapper-persistent-data \

           lvm2



#yum-config-manager \

    --add-repo \

    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo



#yum install docker-ce -y
1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.
#cat > /etc/docker/daemon.json << EOF


  {"registry-mirrors": ["https://he1np13j.mirror.aliyuncs.com"]}

EOF
```

### 3.2 添加阿里云YUM软件源

```bash
$ cat > /etc/yum.repos.d/kubernetes.repo << EOF

[kubernetes]

name=Kubernetes

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=0

repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF
```

 

### 3.3 安装kubeadm，kubelet和kubectl

由于版本更新频繁，这里指定版本号部署：

```bash
$ yum install -y kubelet-1.21.0 kubeadm-1.21.0 kubectl-1.21.0

$ systemctl enable kubelet
```

## 4. 部署Kubernetes Master

在172.25.0.30（Master）执行。

```bash
$ kubeadm init \

  --apiserver-advertise-address=172.25.0.30 \

  --image-repository registry.aliyuncs.com/google_containers \

  --kubernetes-version v1.21.0 \

  --service-cidr=10.96.0.0/12 \

  --pod-network-cidr=10.244.0.0/16 --upload-certs
```





安装1.21版本时报错发现coredns，无法下载

手动拉去

```bash
docker  pull registry.aliyuncs.com/google_containers/coredns:1.8.0
```

重命名

```bash
docker  tag  registry.aliyuncs.com/google_containers/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns/coredns:v1.8.0
```

重新初始化

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

使用kubectl工具：

```bash
mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

$ kubectl get nodes
```

## 5. 加入Kubernetes Node

在172.25.0.31/32（Node）执行。

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```bash
$ kubeadm join 172.25.0.30:6443 --token amdbyn.a02my1ugmoblwy4q --discovery-token-ca-cert-hash sha256:18462463a7db86052399e97b18efe3f12edc5999293abdccf7529669df0ad3fa
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```bash
kubeadm token create --print-join-command
```

## 6. 部署CNI网络插件

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


kubectl apply -f kube-flannel.yml
```

 

默认镜像地址无法访问，sed命令修改为docker hub镜像仓库。

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

查看k8s pod状态

```bash
kubectl get pods -n kube-system

NAME                          READY   STATUS    RESTARTS   AGE

coredns-545d6fc579-2cgr8      1/1     Running   0          72s
```

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/e6951474317ef8fbbf31e93e7a0ac566.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 7. 测试kubernetes集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```bash
$ kubectl create deployment nginx --image=nginx

$ kubectl expose deployment nginx --port=80 --type=NodePort

$ kubectl get pod,svc
```

访问地址：http://NodeIP:Port 

# 三、部署gitlab

## 1、使用docker方式部署

```bash
docker run -d --hostname gitlab.xxx.cn \

--publish 443:443 --publish 80:80 --publish 2222:22 \

--name gitlab --restart always --volume /srv/gitlab/config:/etc/gitlab \

--volume /srv/gitlab/logs:/var/log/gitlab \

--volume /srv/gitlab/data:/var/opt/gitlab \

gitlab/gitlab-ce:latest
```

## 2、配置gitlab

按上面的方式，gitlab容器运行没问题，但在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb

```bash
vim  /srv/gitlab/config/gitlab.rb
```

\# 配置http协议所使用的访问地址,不加端口号默认为80

```bash
external_url 'http://172.25.0.31'
```

\# 配置ssh协议所使用的访问地址和端口

```bash
gitlab_rails['gitlab_ssh_host'] = '172.25.0.31'

gitlab_rails['gitlab_shell_ssh_port'] = 2222 # 此端口是run时22端口映射的2222端口
```

gitlab默认使用的内存越来越大，我们需要优化一下

减少数据库内存大小

```bash
sed -i 's/# puma['worker_processes'] = 2/puma['worker_processes'] = 2/g' /srv/gitlab/config/gitlab.rb
```

减少数据库并发数

```bash
sed -i 's/# postgresql['shared_buffers'] = "256MB"/postgresql['shared_buffers'] = "64MB"/g' /srv/gitlab/config/gitlab.rb

sed -i 's/# postgresql['max_worker_processes'] = 8/postgresql['max_worker_processes'] = 5/g' /srv/gitlab/config/gitlab.rb
```

 

\# 重启gitlab容器

```bash
$ docker restart gitlab
```

## 3、配置管理员密码

默认密码，如果无法登陆

```bash
root 5iveL!fe
```

### 1、进入容器

```bash
docker exec -it gitlab /bin/bash
```

启用gitlab的ruby

```bash
gitlab-rails console -e production
```

### 2、进入管理员用户

```bash
user = User.where(id: 1).first
```

### 3、更改密码

```bash
user.password = 'abcd1234'

user.password_confirmation = 'abcd1234'
```

### 4、保存

```bash
user.save!
```

配置过程

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/bf5b46b2d02ad1df889fbd514b6252ce.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4、访问gitlab

登陆，如下图所示：

账号：root 密码：abcd1234

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/75d95ddeb76799a08adf40a3a20da4ad.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/74f5db33b4e7fd653f50f2ac208cb789.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 

# 四、部署harbor镜像仓库 注意：如果额外的还需要 安装docker

## 1、docker-compose 安装

docker-compose安装

```bash
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/bin/docker-compose
```

赋予权限：

```bash
chmod a+x /usr/bin/docker-compose
```

## 2、Harbor配置、ssl证书生产

### 1、下载包，这里是v2.2.3：

 

```bash
wget https://github.com/goharbor/harbor/releases/download/v2.2.3/harbor-offline-installer-v2.2.3.tgz
```

 

其他版本 ：https://github.com/goharbor/harbor/releases

### 2、解压harbor

```bash
tar -zxvf harbor-offline-installer-v2.2.3.tgz && mv harbor /usr/local/1.
```


![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/3e466a53d76d6e4c52b20415d82ffcbf.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3、配置并安装

准备配置文件：

```bash
cp harbor.yml.tmpl harbor.yml
```

配置harbor.yml

```bash
vim harbor.yml
```

配置域名

```bash
hostname: hub.sx.com #也可以是ip
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80
# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /data/cert/server.crt
  private_key: /data/cert/server.key
harbor_admin_password: harbor12345
database:
  # The password for the root user of Harbor DB. Change this before any production use.
  password: root123
 data_volume: /data/harbor #自定义路
```

\#创建目录

```bash
mkdir -p /data/{cert,harbor}
```

### 4、证书生成

```bash
mkdir -p /data/cert && cd /data/cert
```

生成ssl证书

```bash
openssl genrsa -des3 -out server.key 2048
```





输入两次相同的密码即可：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/8d033912a058c71b2697dd42c5398e8d.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

创建证书请求csr

```bash
openssl req -new -key server.key -out server.csr
```

按步骤 输入server.key密码、国家如CN、省如BJ、城市如BJ、组织如sx、机构如：sx、hostname：hub.sx.com、邮箱123@qq.com。

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/6eee9d01a7f2b917b0b6344b9a89f33f.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

备份ssl证书

```bash
cp server.key server.key.org
```

推出密码，引导证书有密码时，会有问题，需要解锁密码

```bash
openssl rsa -in server.key.org -out server.key
```


![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/b9ea93a10b5da362885c1e42e4c4297f.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

证书请求签名

```bash
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
```

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/160a642eea4fa5a0b4902a37bbee004c.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

赋予权限

```bash
chmod a+x *
```

## 3、harbor安装

### 1、运行prepare文件

```bash
cd /usr/local/harbor

./prepare
```

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/1f97ce2934c06640910983f91b7d817e.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)



### 2、执行脚本安装

```bash
./install.sh
```


![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/13de76538482614bc7df1dbaa0ed53fa.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/fca01f6f2d2a7fdb4c8336e142bb2a42.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4、登录访问

### 1、页面登录

域名绑定hosts

https://hub.sx.com (hub.sx.com为自己的机名harbor.yaml的hostname) 。

管理员用户名/密码为 admin / harbor12345

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/c116c713a4cebf7125f62a9ab3abb5c1.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

登录后如下：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/f8c096a0e712a28e27e7e6eea492404f.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

创建项目

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/0979d0ee66f1a45cb56b35f443a64878.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

如果你运行正常，查看容器会如下：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/7f4f4e96cca0a386803a59fb27ed7580.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 2、docker登录

所以节点配置hosts

```bash
echo 172.25.0.32 hub.sx.com >> /etc/hosts
```

dokcer登录

```bash
docker login https://hub.sx.com  
```

账号密码：admin / harbor12345

从官方文档提示，客户端要使用tls与Harbor通信，使用的还是自签证书，那么必须建立一个目录：/etc/docker/certs.d

在这个目录下建立签名的域名的目录，比如域名为hub.sx.com， 那么整个目录为: /etc/docker/certs.d/hub.sx.com, 然后把harbor的证书拷贝到这个目录即可。

创建目录

```bash
mkdir -p /etc/docker/certs.d/hub.sx.com
```

复制证书到域名目录下

```bash
rsync -avz root@172.25.0.32:/data/cert/server.crt   /etc/docker/certs.d/hub.sx.com/
```

登录测试

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/83c2f7d6c96c14ecbe8220292f4e4075.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# 五、gitlab-runner

安装gitlab-runner

## 1、添加gitlab-runner库

```bash
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

\# For RHEL/CentOS/Fedora

```bash
sudo yum install -y gitlab-runner
```

找到setting -->CI/CD-->runner Expand

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/318e2dffbd6cc1b29f542d05205b0d18.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

下面URL与token为注册runner准备

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/313410ef7409f06b231a9d1c4d5b402f.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 

## 2、注册 gitlab-runner

官网：https://docs.gitlab.com/runner/register/

 

```bash
# gitlab-runner register --clone-url http://172.25.0.31
Runtime platform                                    arch=amd64 os=linux pid=116330 revision=8925d9a0 version=14.1.0
Running in system-mode.                           
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://172.25.0.31/  #你的gitlab地址 ，注意该地址可以被gitlab-runner主机访问
Enter the registration token:  #项目的token
7-TksKUqs4xtxjM1DNjk
Enter a description for the runner:  #描述，如： test
[k8s-node2]: test
Enter tags for the runner (comma-separated):  # 项目标签，如：builder; 线上环境可以设置为 tags，测试环境设置为test
test
Registering runner... succeeded                     runner=cDjz56GH
Enter an executor: ssh, virtualbox, docker-ssh+machine, custom, docker-ssh, shell, kubernetes, docker, parallels, docker+machine: #执行方式：这里选择docker
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

也可以直接运行

```bash
gitlab-runner register \
  --non-interactive \
  --url "http://172.25.0.31/" \ #该参数为gitlab服务器的位置
  --registration-token "7-TksKUqs4xtxjM1DNjk" \ #该以管理员身份从gitlab获取的registration token
  --executor "shell" \ #以shell方式执行任务，支持docker，docker machine，ssh 等等各种执行方式
  --docker-image alpine:latest \ #默认的任务镜像
  --description "test" \ #描述
  --tag-list "test" \ #标记在.gitlab-ci.yml会用到
  --run-untagged \ #可以运行在非tag代码上
  --locked="false"  #是否锁定该执行器
```

gitlab可以查看已创建的runner

 

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/efc31c19af9f2301fe6967e96bb4ba56.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 

修改 gitlab-runner 并发执行个数

```bash
$ vim /etc/gitlab-runner/config.toml
concurrent = 5
```

# 六、构建部署项目

## 1、拉取微服务代码

```bash
git clone https://github.com/hyperf/hyperf-skeleton.git1.
```

## 2、修改部署文件

添加deploy部署文件

```bash
vim hyperf.yaml
apiVersion: v1
kind: Service
metadata:
  name: transport
  labels:
    app: transport
spec:
  type: NodePort
  ports:
  - name: http
    port: 9501                      #服务端口
    targetPort: 9501
    nodePort: 30018                 #NodePort方式暴露 
  selector:
    app: transport
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: transport
spec:
  selector:
    matchLabels:
      app: transport
  replicas: 2
  template:
    metadata:
      labels:
        app: transport
    spec:
      imagePullSecrets:
        - name: registry-pull-secret3
      containers:
      - name: transport
        image: hub.sx.com/hyperf/hyperf
        imagePullPolicy: Always
        securityContext:                    
          runAsUser: 0                      #设置以ROOT用户运行容器
          privileged: true                  #拥有特权
        ports:
        - name: http
          containerPort: 9501
        resources:
          limits:
            memory: 2Gi
            cpu: "1000m"
          requests:
            memory: 500Mi
            cpu: "500m"
        volumeMounts:
        - mountPath:  /opt/www/runtime      
          name: test-volume  
      volumes:
      - name: test-volume   
        hostPath:  
          path: /data/transport/logs # directory location on host          
          type: DirectoryOrCreate # this field is optional
```

修改.gitlab-ci.yml

```bash
stages:
  - docker-pkg
  - k8s-deploy
variables:
  PROJECT_NAME: hyperf #gitlab项目名
  REGISTRY_URL:  hub.sx.com/hyperf #仓库url
docker-pkg:
  stage: docker-pkg
  before_script:
  - docker login -u admin -p harbor12345  hub.sx.com
  script:
  - docker build  -t  $REGISTRY_URL  .
  - docker push   $REGISTRY_URL/$PROJECT_NAME 
  allow_failure: false
  only:
  - test
  tags:
  - test
  when: manual
  retry: 2
k8s-deploy:
  stage: k8s-deploy
  script:
  - kubectl apply -f ./hyperf.yaml
  allow_failure: false
  dependencies:
  - docker-pkg
  only:
  - test
  tags:
  - test
  when: manual
  retry: 2
```

添加配置.env

```bash
APP_NAME=skeleton
APP_ENV=dev
DB_DRIVER=mysql
DB_HOST=10.10.10.12
DB_PORT=3306
DB_DATABASE=hyperf
DB_USERNAME=hyperf
DB_PASSWORD=123456
DB_CHARSET=utf8mb4
DB_COLLATION=utf8mb4_unicode_ci
DB_PREFIX=
REDIS_HOST=10.10.10.15
REDIS_AUTH=
REDIS_PORT=6379
REDIS_DB=0
```

##  3、推送代码到自己gitlab项目

```bash
git add .
git commit -m 'add file'
git push
```





![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/2b3ee41e0a35f34d27f31d923e4331aa.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 4、gitlab查看ci/cd构建

 

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/75882da54a603c271bf3d88d439b8f28.jpeg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

成功的终端输出如下：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/74f8b1602453ef0dd77dd28176fe811a.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/c90fa9e42894198031fd85fe591376aa.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 5、查看节点，访问是否成功

 

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/de4a5a6c3c4573862921a4a7f734b936.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

访问测试：

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202108/20/2178820ce0189b181455039e1164af0a.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# 七、构建错误以及处理办法

## 错误1：

```bash
. dial unix /var/run/docker.sock: connect: permission denied
```

报错信息：

```bash
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.27/info: dial unix /var/run/docker.sock: connect: permission denied
```

原因：gitlab-runner账号权限不足，不能访问/var/run/docker.sock。

解决方案：

\# 将gitlab-runner用户加入docker组

```bash
# usermod -aG docker gitlab-runner
```

\# 查看

```bash
# groups gitlab-runner
```

## 错误2：

```bash
Reinitialized existing Git repository in /home/gitlab-runner/builds/SoPpKTDd/0/root/git-runner-demo/.git/
fatal: git fetch-pack: expected shallow list
fatal: The remote end hung up unexpectedly
ERROR: Job failed: exit status 1
```

\#删除旧版本git

```bash
# yum remove git -y
```

\##安装第三方yum源,centos7 基础仓库，提供的 git 版本只有到 1.8.3，沒办法使用 git 2 的一些新功能

```bash
# yum install http://opensource.wandisco.com/centos/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm
```

\##安装最新版git 2.x

```bash
# yum install git gitlab-runner
```

\##查看版本

```bash
# git version
git version 2.22.0
```