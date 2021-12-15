# Centos安装Docker



## 方式一：yum安装

```bash
yum update
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce
systemctl start docker
systemctl enable docker
```



## 方式二：（推荐）

```bash
curl -sSL https://get.daocloud.io/docker | sh
```



## 方式三：脚本方式安装

在所有服务器上创建install_docker.sh脚本

```bash
#使用阿里云镜像中心
export REGISTRY_MIRROR=https://registry.cn-hangzhou.aliyuncs.com
#安装docker环境
yum install -y yum-utils device-mapper-persistent-data lvm2
#配置Docker的yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#安装容器插件
dnf install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.13-3.1.el7.x86_64.rpm
#指定安装docker 19.03.8版本
yum install -y docker-ce-19.03.8 docker-ce-cli-19.03.8
#设置Docker开机启动
systemctl enable docker.service
#启动Docker
systemctl start docker.service
#查看Docker版本
docker version
```

在每台服务器上为install_docker.sh脚本赋予可执行权限，并执行脚本，如下所示。

```bash
# 赋予install_docker.sh脚本可执行权限
chmod a+x ./install_docker.sh
# 执行install_docker.sh脚本
./install_docker.sh
```



# [Centos7-linux安装docker(离线方式)](https://www.cnblogs.com/helf/p/12889955.html)

1. 下载docker的安装文件

https://download.docker.com/linux/static/stable/x86_64/

下载的是：docker-18.06.3-ce.tgz 这个压缩文件

![img](https://img2020.cnblogs.com/blog/1419795/202005/1419795-20200514170642044-109054151.png)

1. 将docker-18.06.3-ce.tgz文件上传到centos7-linux系统上，用ftp工具上传即可

![img](https://img2020.cnblogs.com/blog/1419795/202005/1419795-20200514170715454-2121213609.png)

1. 解压

```shell
[root@localhost java]# tar -zxvf docker-18.06.3-ce.tgz
```

1. 将解压出来的docker文件复制到 /usr/bin/ 目录下

```shell
[root@localhost java]# cp docker/* /usr/bin/
```

1. 进入**/etc/systemd/system/**目录,并创建**docker.service**文件

```shell
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
```

1. 打开**docker.service**文件,将以下内容复制

```shell
[root@localhost system]# vi docker.service
```

**注意**： --insecure-registry=192.168.200.128 此处改为你自己服务器ip

```shell
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=192.168.200.128
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

1. 给docker.service文件添加执行权限

```shell
[root@localhost system]# chmod 777 /etc/systemd/system/docker.service 
```

1. 重新加载配置文件（每次有修改docker.service文件时都要重新加载下）

```shell
[root@localhost system]# systemctl daemon-reload 
```

1. 启动

```shell
[root@localhost system]# systemctl start docker
```

1. 设置开机启动

```
[root@localhost system]# systemctl enable docker.service
```

1. 查看docker状态

```shell
[root@localhost system]# systemctl status docker
```

出现下面这个界面就代表docker安装成功。

![img](https://img2020.cnblogs.com/blog/1419795/202005/1419795-20200514170744352-1186245935.png)

1. 配置镜像加速器,默认是到国外拉取镜像速度慢,可以配置国内的镜像如：阿里、网易等等。下面配置一下网易的镜像加速器。打开docker的配置文件: /etc/docker/**daemon.json**文件：

```shell
[root@localhost docker]# vi /etc/docker/daemon.json
```

配置如下:

```json
{"registry-mirrors": ["http://hub-mirror.c.163.com"]}
```

配置完后**:wq**保存配置并**重启docker** 一定要重启不然加速是不会生效的！！！

```shell
[root@localhost docker]# service docker restart
```