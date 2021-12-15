Docker仓库，重点想讲一讲Harbor，对于Registry私服个人感觉用的还是少，而且不是很方便，对于Harbor自己在上手实践也在公司项目落地过程中，均觉得十分不错，故Docker系列中专门讲解一下Harbor的相关内容。



# 一、Harbor概述

Harbor是由VMware公司开源的企业级的Docker Registry管理项目，它包括**权限管理(RBAC)、LDAP、日志审核、管理界面、自我注册、镜像复制和中文支持**等功能。

容器的核心在于镜象的概念，由于可以将应用打包成镜像，并快速的启动和停止，因此容器成为新的炙手可热的基础设施CAAS，并为敏捷和持续交付包括DevOps提供底层的支持。

而Habor和Docker Registry所提供的容器镜像仓库，就是容器镜像的存储和分发服务。

之所以会有这样的服务存在，是由于以下三个原因：

- 提供分层传输机制，优化网络传输
  - Docker镜像是是分层的，而如果每次传输都使用全量文件（所以用FTP的方式并不适合），显然不经济。必须提供识别分层传输的机制，以层的UUID为标识，确定传输的对象。
- 提供WEB界面，优化用户体验
  - 只用镜像的名字来进行上传下载显然很不方便，需要有一个用户界面可以支持登陆、搜索功能，包括区分公有、私有镜像。
- 支持水平扩展集群
  - 当有用户对镜像的上传下载操作集中在某服务器，需要对相应的访问压力作分解。

## 1.1 Harbor的安全机制

在Harbor中，用户主要分为两类。一类为管理员，另一类为普通用户。两类用户都可以成为项目的成员。而管理员可以对用户进行管理。

成员是对应于项目的概念，分为三类：管理员、开发者、访客。管理员可以对开发者和访客作权限的配置和管理。测试和运维人员可以访客身份读取项目镜像，或者公共镜像库中的文件。
从项目的角度出发，显然项目管理员拥有最大的项目权限，如果要对用户进行禁用或限权等，可以通过修改用户在项目中的成员角色来实现，甚至将用户移除出这个项目。

![image-20210204143556213](http://lovebetterworld.com/image-20210204143556213.png)

## 1.2 Harbor的镜像同步

**为什么需要镜像同步**
由于对镜像的访问是一个核心的容器概念，在实际使用过程中，一个镜像库可能是不够用的，下例情况下，我们可能会需要部署多个镜像仓库：

- 国外的公有镜像下载过慢，需要一个中转仓库进行加速
- 容器规模较大，一个镜像仓库不堪重负
- 对系统稳定性要求高，需要多个仓库保证高可用性
- 镜像仓库有多级规划，下级仓库依赖上级仓库

**更常用的场景是，在企业级软件环境中，会在软件开发的不同阶段存在不同的镜像仓库，**

- 在开发环境库，开发人员频繁修改镜像，一旦代码完成，生成稳定的镜像即需要同步到测试环境。
- 在测试环境库，测试人员对镜像是只读操作，测试完成后，将镜像同步到预上线环境库。
- 在预上线环境库，运维人员对镜像也是只读操作，一旦运行正常，即将镜像同步到生产环境库。
- 在这个流程中，各环境的镜像库之间都需要镜像的同步和复制。

## 1.3 Harbor的镜像同步机制

有了多个镜像仓库，在多个仓库之间进行镜像同步马上就成为了一个普遍的需求。比较传统的镜像同步方式，有两种：

- 第一种方案，使用Linux提供的RSYNC服务来定义两个仓库之间的镜像数据同步。
- 第二种方案，对于使用IaaS服务进行镜像存储的场景，利用IaaS的配置工具来对镜像的同步进行配置。

这两种方案都依赖于仓库所在的存储环境，而需要采用不同的工具策略。Harbor则提供了更加灵活的方案来处理镜像的同步，其核心是三个概念：

- 用Harbor自己的API来进行镜像下载和传输，作到与底层存储环境解耦。
- 利用任务调度和监控机制进行复制任务的管理，保障复制任务的健壮性。在同步过程中，如果源镜像已删除，Harbor会自动同步删除远端的镜像。在镜像同步复制的过程中，Harbor会监控整个复制过程，遇到网络等错误，会自动重试。
- 提供复制策略机制保证项目级的复制需求。在Harbor中，可以在项目中创建复制策略，来实现对镜像的同步。与Docker Registry的不同之处在于，Harbor的复制是推（PUSH）的策略，由源端发起，而Docker Registry的复制是拉（PULL）的策略，由目标端发起。

![image-20210204143625510](http://lovebetterworld.com/image-20210204143625510.png)

## 1.4 Harbor的多级部署

在实际的企业级生产运维场景，往往需要跨地域，跨层级进行镜像的同步复制，比如集团企业从总部到省公司，由省公司再市公司的场景。
这一部署场景可简化如下图：



![](https://www.kubernetes.org.cn/img/2017/03/20170314213455.jpg)

更复杂的部署场景如下图：



![](https://www.kubernetes.org.cn/img/2017/03/20170314213503.jpg)

# 二、安装

## 2.1 docker安装

```bash
# yum 包更新
[root@centos7 ~]# yum update

# 卸载旧版本 Docker
[root@centos7 ~]# yum remove docker docker-common docker-selinux docker-engine

# 安装软件包
[root@centos7 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加 Docker yum源
[root@centos7 ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker
[root@centos7 ~]# yum -y install docker-ce

# 启动 Docker
[root@centos7 ~]# systemctl start docker

# 查看 Docker 版本号
[root@centos7 ~]# docker --version
```

开启docker远程访问：

```bash
vim /lib/systemd/system/docker.service

# 找到ExecStart行，修改成下边这样
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock  
```

## 2.2 docker compose安装

### 2.2.1 方式一

```bash
# 安装 epel-release
[root@centos7 ~]# yum install epel-release

# 安装 python-pip
[root@centos7 ~]# yum install -y python-pip

# 安装 docker-compose
[root@centos7 ~]# pip install docker-compose

# 安装 git
[root@centos7 ~]# yum install -y git

# 查看 docker-compose 版本号
[root@centos7 ~] docker-compose -version

# docker compose卸载
sudo rm /usr/local/bin/docker-compose
```

### 2.2.2 方式二

```bash
yum install epel-release
yum install -y python-pip
pip install docker-compose
yum install git
```

![](https://img2018.cnblogs.com/blog/1580998/201912/1580998-20191224091913658-1191936961.png)


## 2.3 harbor安装

GitHub地址：https://github.com/goharbor/harbor/releases

```bash
wget https://github.com/goharbor/harbor/releases/download/v1.9.3/harbor-offline-installer-v1.9.3.tgz
```

下载后解压：
` tar -xvf harbor-offline-installer-v1.9.0.tgz `

修改harbor.yml：

```yml
hostname: example.xxxx.cn  # 写你自己的网址或IP，公网访问要写公网IP
https:
   # https port for harbor, default is 443
   port: 443
   # The path of cert and key files for nginx
   certificate: /root/harbor/Nginx/1.crt
   private_key: /root/harbor/Nginx/2.key
harbor_admin_password: Harbor12345 # 管理员密码建议修改
database:
  password: root123    # 数据库密码也建议修改
```

![](https://upload-images.jianshu.io/upload_images/13965490-90a9339c70e2b80f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)


执行安装：
` sh ./install.sh `

## 2.4 常用命令

```bash
# 停止harbor
$ sudo docker-compose stop

# 启动harbor
$ sudo docker-compose start

# 重新配置harbor
$ sudo docker-compose down -v
$ vim harbor.yml
$ sudo prepare
$ sudo docker-compose up -d

# 删除Harbor的容器，同时将镜像数据和Harbor的数据库文件保留在文件系统中：
$ sudo docker-compose down -v

# 删除Harbor的数据库和镜像数据以进行重新安装：
$ rm -r /data/database
$ rm -r /data/registry

# 如果要一起安装Notary，Clair和图表存储库服务，则应在prepare命令中包括所有组件：
$ sudo docker-compose down -v
$ vim harbor.yml
$ sudo prepare --with-notary --with-clair --with-chartmuseum
$ sudo docker-compose up -d
```

# 三、Harbor使用

## 3.1 项目

以项目的维度划分镜像，可以理解为镜像组，相同镜像的不同版本可以放在一个项目里，同样项目里有完成的仓库、成员、标签、日志的管理。

## 3.2 系统管理

- 用户管理 用于操作用户的增删、密码重置
- 仓库管理 拉取其他服务器镜像到本地
- 同步管理 可定时去拉取最新镜像

# 四、镜像推送和拉取

镜像推送和拉取

```bash
#从私服拉取镜像 docker pull 私服地址/仓库项目名/镜像名：标签
```

镜像推送

```bash
#推送
docker login 服务器地址:port

#镜像打标签 ,要重新打标签，标签默认是官网地址
docker tag 镜像名:标签 私服地址/仓库项目名/镜像名:标签

#推送指令
docker push 私服地址/仓库项目名/镜像名：标签
```

# 五、坑点

## 5.1 上传项目时修改http请求为https

在我们上传项目的时候可能会出现一些问题：

```bash
docker login 10.0.86.193
Username: admin
Password:
Error response from daemon: Get https://10.0.86.193/v1/users/: dial tcp 10.0.86.193:443: getsockopt: connection refused
```

在我们进行登录上传代码的时候，会报出这样的错误
这是因为docker1.3.2版本开始默认docker registry使用的是https，我们设置Harbor默认http方式，所以当执行用docker login、pull、push等命令操作非https的docker regsitry的时就会报错。

**解决办法**：
如果是在Harbor本机登录可以这样做如下解决
在/etc/docker/daemon.json 加上如下内容(注意是json字符串)

```yml
     {
      "insecure-registries": [
        "10.0.86.193"
      ]
    }
```

打开docker-compose.yml添加如下内容，注意前边的空格
![](https://img-blog.csdn.net/2018092610475038?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA4MjYzNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后我们执行docker-compose stop
./install.sh

**如果是远程登录的话，也会出现这个错误**
查找Docker的服务文件：登录到已经安装Docker的服务器，输入 systemctl status docker查看Docker的service文件
![](https://img-blog.csdn.net/20180926170307401?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA4MjYzNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

编辑docker.service文件：在ExecStart处添加 –insecure-registry 参数。
![](https://img-blog.csdn.net/20180926170436296?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjA4MjYzNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

远程也可直接通过以下方式解决：

```bash
vi  /etc/docker/daemon.json
# 增加一个daemon.json文件

{ "insecure-registries":["192.168.1.100:port"] }
# 此处port指的是harbor服务的port，不知为何，看到很多人设置的是5000端口，我测试后并不好使，然后改为harbor服务端口，原本是80，我本地调试改为81，即可远程登录私服。

# harbor配置文件，harbor.yml，查看其中的http，port。
```

重启docker服务

```bash
systemctl daemon-reload && systemctl restart docker
```



相关内容参考原文链接：

- Kubernetes中文社区：[Harbor用户机制、镜像同步和与Kubernetes的集成实践]：   https://www.kubernetes.org.cn/1738.html
- [Harbor镜像仓库（含clair镜像扫描） - 完整部署记录]：   https://www.cnblogs.com/kevingrace/p/13970578.html



# Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)

