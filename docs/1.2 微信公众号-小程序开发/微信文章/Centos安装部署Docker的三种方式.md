# Centos部署Docker环境

## 1.1 方式一：yum方式安装

```bash
# 更新yum源
yum update
# 安装所需环境
yum install -y yum-utils device-mapper-persistent-data lvm2
# 配置yum仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装Docker
yum install docker-ce
# 启动Docker
systemctl start docker
systemctl enable docker
```



## 1.2 方式二：（推荐安装方式）

```bash
curl -sSL https://get.daocloud.io/docker | sh
```



## 1.3 离线安装

1. 下载docker的安装文件：https://download.docker.com/linux/static/stable/x86_64/

![image-20210201092144600](http://lovebetterworld.com/image-20210201092144600.png)

2. 将下载后的tgz文件传至服务器，通过FTP工具上传即可
3. 解压`tar -zxvf docker-19.03.8-ce.tgz`
4. 将解压出来的docker文件复制到 /usr/bin/ 目录下：`cp docker/* /usr/bin/`
5. 进入**/etc/systemd/system/**目录,并创建**docker.service**文件

```bash
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
```

6. 打开**docker.service**文件,将以下内容复制

```bash
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

7. 给docker.service文件添加执行权限：`chmod 777 /etc/systemd/system/docker.service `

8. 重新加载配置文件：`systemctl daemon-reload `

9. 启动Docker `systemctl start docker`

10. 设置开机启动：`systemctl enable docker.service`

11. 查看Docker状态：`systemctl status docker`

    ![image-20210201092621496](http://lovebetterworld.com/image-20210201092621496.png)

如出现如图界面，则表示安装成功！

