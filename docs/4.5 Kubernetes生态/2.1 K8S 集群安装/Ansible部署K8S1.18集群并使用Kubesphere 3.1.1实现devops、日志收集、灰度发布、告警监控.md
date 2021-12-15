- [ansible部署K8S1.18集群并使用Kubesphere 3.1.1实现devops、日志收集、灰度发布、告警监控](https://blog.csdn.net/FizzX1/article/details/107714417?spm=1001.2014.3001.5501)

# 一、离线安装集群

参考 https://github.com/easzlab/kubeasz/blob/master/docs/setup/offline_install.md

## 1.1 离线文件准备

在一台能够访问互联网的服务器上执行：

- 下载工具脚本easzup，举例使用kubeasz版本3.1.0

```bash
export release=3.1.0
curl -C- -fLO --retry 3 https://github.com/easzlab/kubeasz/releases/download/${release}/ezdown
chmod +x ./ezdown
```

- 使用工具脚本下载

默认下载最新推荐k8s/docker等版本，使用命令`./easzup` 查看工具脚本的帮助信息

```bash
# 举例使用 k8s 版本 v1.18.6，docker 19.03.5
./ezdown -D -d 19.03.14 -k v1.18.6
# 下载离线系统软件包
./ezdown -P
执行成功后，所有文件均已整理好放入目录/etc/ansible
，只要把该目录整体复制到任何离线的机器上，即可开始安装集群，离线文件包括：
```

- `/etc/ansible` 包含 kubeasz 版本为 ${release} 的发布代码
- `/etc/ansible/bin` 包含 k8s/etcd/docker/cni 等二进制文件
- `/etc/ansible/down` 包含集群安装时需要的离线容器镜像
- `/etc/ansible/down/packages` 包含集群安装时需要的系统基础软件

离线文件不包括：

- 管理端 ansible 安装，但可以使用 kubeasz 容器运行 ansible 脚本
- 其他更多 kubernetes 插件镜像

## 1.2 离线安装

上述下载完成后，把`/etc/ansible`整个目录复制到目标离线服务器相同目录，然后在离线服务器上运行：

- 离线安装 docker，检查本地文件，正常会提示所有文件已经下载完成

```bash
./easzup -D
```

- 启动 kubeasz 容器

```bash
./easzup -S
```

- 设置参数允许离线安装

```bash
sed -i 's/^INSTALL_SOURCE.*$/INSTALL_SOURCE: "offline"/g' /etc/ansible/roles/chrony/defaults/main.yml
sed -i 's/^INSTALL_SOURCE.*$/INSTALL_SOURCE: "offline"/g' /etc/ansible/roles/ex-lb/defaults/main.yml
sed -i 's/^INSTALL_SOURCE.*$/INSTALL_SOURCE: "offline"/g' /etc/ansible/roles/kube-node/defaults/main.yml
sed -i 's/^INSTALL_SOURCE.*$/INSTALL_SOURCE: "offline"/g' /etc/ansible/roles/prepare/defaults/main.yml
```

在kubeasz容器内执行k8s的安装

```bash
docker exec -it kubeasz ezctl start-aio
```

如果需要配置1主多从的集群

```bash
#配置ssh登录node节点
ssh-copy-id root@192.168.11.11
#进入kubeasz容器中执行添加节点命令
docker exec -it kubeasz bash
#执行添加节点命令
ezctl add-node default 192.168.11.11
```

如果当前linux内核版本在4.4以下，kube-proxy的ipvs可能会有问题，导致dns无法解析

解决问题

1、升级系统内核版本

升级 Kubernetes 集群各个节点的 CentOS 系统内核版本：

```bash
## 载入公钥 

$ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 

## 安装 ELRepo 最新版本

$ yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm 

## 列出可以使用的 kernel 包版本 

$ yum list available --disablerepo=* --enablerepo=elrepo-kernel 

## 安装指定的 kernel 版本： 

$ yum install -y kernel-lt-4.4.231-1.el7.elrepo --enablerepo=elrepo-kernel

 ## 设置开机从新内核启动

$ grub2-set-default "CentOS Linux (4.4.231-1.el7.elrepo.x86_64) 7 (Core)"

## 查看内核启动项 

$ grub2-editenv list saved_entry=CentOS Linux (4.4.231-1.el7.elrepo.x86_64) 7 (Core)

重启系统使内核生效：

$ reboot

启动完成查看内核版本是否更新：

$ uname -r 4.4.231-1.el7.elrepo.x86_64
```

参考：http://www.mydlq.club/article/78/



# 二、部署NFS服务，添加为k8s默认storageclass

## 2.1 nfs服务端安装

```bash
使用 yum 安装 NFS 安装包。

$ sudo yum install nfs-utils

注意

只安装 nfs-utils 即可，rpcbind 属于它的依赖，也会安装上。

服务端配置

设置 NFS 服务开机启动

$ sudo systemctl enable rpcbind $ sudo systemctl enable nfs

启动 NFS 服务

$ sudo systemctl start rpcbind $ sudo systemctl start nfs


配置共享目录

服务启动之后，我们在服务端配置一个共享目录

$ sudo mkdir /data $ sudo chmod 755 /data

根据这个目录，相应配置导出目录

$ sudo vi /etc/exports

添加如下配置

/data/ 192.168.0.0/24(rw,sync,no_root_squash,no_all_squash)

/data: 共享目录位置。
192.168.0.0/24: 客户端 IP 范围，* 代表所有，即没有限制。
rw: 权限设置，可读可写。
sync: 同步共享目录。
no_root_squash: 可以使用 root 授权。
no_all_squash: 可以使用普通用户授权。
:wq 保存设置之后，重启 NFS 服务。

$ sudo systemctl restart nfs

可以检查一下本地的共享目录

$ showmount -e localhost Export list for localhost: /data 192.168.0.0/24

这样，服务端就配置好了，接下来配置客户端，连接服务端，使用共享目录。
```



## 2.2 为k8s添加动态PV

```yaml
#编辑自定义配置文件：
$ vim /etc/kubeasz/roles/cluster-storage/defaults/main.yml
# 比如创建nfs provisioner
storage:
  nfs:
    enabled: "yes"
    server: "192.168.1.8"
    server_path: "/data/nfs"
    storage_class: "nfs-dynamic-class"
    provisioner_name: "nfs-provisioner-01"
#创建 nfs provisioner
$ docker exec -it kubeasz bash
$ ansible-playbook /etc/kubeasz/roles/cluster-storage/cluster-storage.yml
$ kubectl patch storageclass nfs-dynamic-class -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
# 执行成功后验证
$ kubectl get pod --all-namespaces |grep nfs-prov
kube-system   nfs-provisioner-01-6b7fbbf9d4-bh8lh        1/1       Running   0          1d
注意 k8s集群可以使用多个nfs provisioner，重复上述步骤1、2：修改使用不同的nfs server nfs_storage_class nfs_provisioner_name后执行创建即可。
```



# 三、开始部署Kubesphere 3.1.1正式版本

参考：https://github.com/kubesphere/ks-installer

## 3.1 准备工作

集群现有的可用内存至少在 `10G` 以上。 如果是执行的 `allinone` 安装，那么执行 `free -g` 可以看下可用资源

```bash
$ free -g
              total        used        free      shared  buff/cache   available
Mem:              16          4          10           0           3           2
Swap:             0           0           0
```

## 3.2 部署 KubeSphere

### 3.2.1 最小化快速部署

```bash
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.1.1/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.1.1/cluster-configuration.yaml


 # 查看部署进度及日志
 $ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

部署完成后可执行如下命令查看控制台的服务端口，使用 `IP:consolePort(default: 30880)` 访问 KubeSphere UI 界面，默认的集群管理员账号为 `admin/P@88w0rd`。

```bash
kubectl get svc/ks-console -n kubesphere-system
```

以上为最小化部署，如需开启更多功能，请参考如下步骤配置相关依赖：

### 3.2.2 安装可插拔功能组件

1. 编辑 ClusterConfiguration 开启可插拔的功能组件:

```bash
#编辑k8s的clusterconfig，将未开启的日志收集、告警监控、devops等功能按需开启（对应enable设置为true）
kubectl edit cc ks-installer -n kubesphere-system
```

> 按功能需求编辑配置文件之后，退出等待生效即可，如长时间未生效请使用如下命令查看相关日志:

```bash
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

