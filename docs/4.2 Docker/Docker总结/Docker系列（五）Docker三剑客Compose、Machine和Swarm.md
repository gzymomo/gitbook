Docker三剑客，在我个人工作的过程中，使用过Compose和Swarm，均是个人研究使用，Compose还好，在很多场景下有着良好的适用性，以及一些软件的安装如果通过Compose方式来编排还是比较容易管理的。对于Swarm，是用来管理Docker集群的，因为之前公司服务器大部分都安装了Docker，每个服务器单独管理，所以一直设想如何通过一个集群来管理Docker，后来对比了解了Swarm和Kubernetes，最终还是选择K8s。Docker三剑客的内容，个人的见解也没那么深刻，所以转载网上的相关优秀博文来做分享。



> 原文作者：wfs1994
>
> 原文链接：https://blog.csdn.net/wfs1994/article/details/80601027



Docker三大编排工具：

- `Docker Compose`：是用来组装多容器应用的工具，可以在 Swarm集群中部署分布式应用。
- `Docker Machine`：是支持多平台安装Docker的工具，使用 Docker Machine，可以很方便地在笔记本、云平台及数据中心里安装Docker。
- `Docker Swarm`：是Docker社区原生提供的容器集群管理工具。



# 一、Docker Compose

Github地址： https://github.com/docker/compose

Compose是用来<font color='red'>定义和运行一个或多个容器应用的工具</font>。使用compaose可以简化容器镜像的建立及容器的运行。
 Compose使用python语言开发，非常适合在单机环境里部署一个或多个容器，并自动把多个容器互相关联起来。

Compose 中有两个重要的概念：

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

Compose是使用YML文件来定义多容器应用的，它还会用 `docker-compose up`  命令把完整的应用运行起来。

docker-compose up  命令为应用的运行做了所有的准备工作。

从本质上讲，Compose把YML文件解析成docker命令的参数，然后调用相应的docker命令行接口，从而把应用以容器化的方式管理起来。它通过解析容器间的依赖关系来顺序启动容器。

而容器间的依赖关系则可以通过在 `docker-compose.yml`文件中使用 `links` 标记指定。

## 1.1 安装Docker compose

 [官方文档](https://docs.docker.com/compose/install/)

```bash
pip安装：
pip install docker-compose
	
从github安装：
curl -L --fail https://github.com/docker/compose/releases/download/1.17.0/run.sh -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 1.2 卸载

```bash
rm /usr/local/bin/docker-compose # 使用curl安装的
pip uninstall docker-compose # 使用pip卸载
```

**[命令说明](https://yeasy.gitbooks.io/docker_practice/content/compose/commands.html)**

```bash
docker-compose --help
```

# 二、Docker Machine

Github地址： https://github.com/docker/machine/
 Docker Machine 是 Docker 官方编排（Orchestration）项目之一，负责在多种平台上快速安装 Docker 环境。
 Docker Machine 项目基于 Go 语言实现，目前在 [Github](https://github.com/docker/machine) 上进行维护。

## 2.1 安装

```bash
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
chmod +x /usr/local/bin/docker-machine
```

完成后，查看版本信息：

```bash
# docker-machine -v
docker-machine version 0.13.0, build 9ba6da9
```

为了得到更好的体验，我们可以安装 `bash completion script`，这样在 bash 能够通过 tab 键补全 docker-mahine 的子命令和参数。
 下载方法：

```bash
base=https://raw.githubusercontent.com/docker/machine/v0.13.0
for i in docker-machine-prompt.bash docker-machine-wrapper.bash docker-machine.bash
do
  sudo wget "$base/contrib/completion/bash/${i}" -P /etc/bash_completion.d
done
```

确认版本将其放置到 `/etc/bash_completion.d` 目录下。
 然后在你的bash终端中运行如下命令，告诉你的设置在哪里可以找到`docker-machine-prompt.bash`你以前下载的文件 。

```bash
source /etc/bash_completion.d/docker-machine-prompt.bash
```

要启用`docker-machine shell`提示符，请添加 `$(__docker_machine_ps1)`到您的PS1设置中`~/.bashrc`。

```bash
PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '
```

## 2.2 使用

> Docker Machine 支持多种后端驱动，包括虚拟机、本地主机和云平台等

创建本地主机实例：
 使用 `virtualbox` 类型的驱动，创建一台 Docker主机，命名为test

*安装VirtualBox：*

```bash
配置源：
# cat /etc/yum.repos.d/virtualbox.repo 
[virtualbox]
name=Oracle Linux / RHEL / CentOS-$releasever / $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc

#安装VirtualBox：
yum search VirtualBox
yum install -y VirtualBox-5.2
/sbin/vboxconfig    # 重新加载VirtualBox服务
```

*创建主机：*

```bash
#  docker-machine create -d virtualbox test
Running pre-create checks...
Creating machine...
(test) Copying /root/.docker/machine/cache/boot2docker.iso to /root/.docker/machine/machines/test/boot2docker.iso...
(test) Creating VirtualBox VM...
(test) Creating SSH key...
(test) Starting the VM...
(test) Check network to re-create if needed...
(test) Found a new host-only adapter: "vboxnet0"
(test) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env test
```

也可以在创建时加上如下参数，来配置主机或者主机上的 Docker。

```bash
--engine-opt dns=114.114.114.114   #配置 Docker 的默认 DNS
--engine-registry-mirror https://registry.docker-cn.com #配置 Docker 的仓库镜像
--virtualbox-memory 2048 #配置主机内存
--virtualbox-cpu-count 2 #配置主机 CPU
```

更多参数请使用 `docker-machine create --driver virtualbox --help` 命令查看。

*查看主机：*

```bash
# docker-machine ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
test   -        virtualbox   Running   tcp://192.168.99.100:2376           v18.05.0-ce  		
```

创建主机成功后，可以通过 env 命令来让后续操作目标主机：

```bash
docker-machine env test
```

可以通过 SSH 登录到主机：

```bash
docker-machine ssh test
```

## 2.3 官方支持驱动

```bash
# 通过 -d 选项可以选择支持的驱动类型。
amazonec2
azure
digitalocean
exoscale
generic
google
hyperv
none
openstack
rackspace
softlayer
virtualbox
vmwarevcloudair
vmwarefusion
vmwarevsphere
```

第三方驱动请到 [第三方驱动列表](https://github.com/docker/docker.github.io/blob/master/machine/AVAILABLE_DRIVER_PLUGINS.md) 查看

## 2.4 常用操作命令

```bash
active   查看活跃的 Docker 主机
config   输出连接的配置信息
create   创建一个 Docker 主机
env      显示连接到某个主机需要的环境变量
inspect  输出主机更多信息
ip       获取主机地址
kill     停止某个主机
ls       列出所有管理的主机
provision 重新设置一个已存在的主机
regenerate-certs 为某个主机重新生成 TLS 认证信息
restart  重启主机
rm       删除某台主机
ssh SSH  到主机上执行命令
scp      在主机之间复制文件
mount    挂载主机目录到本地
start    启动一个主机
status   查看主机状态
stop     停止一个主机
upgrade  更新主机 Docker 版本为最新
url      获取主机的 URL
version  输出 docker-machine 版本信息
help     输出帮助信息
```

每个命令，又带有不同的参数，可以通过`docker-machine COMMAND --help`查看。

# 三、Docker Swarm

Docker Swarm 是 Docker 官方三剑客项目之一，<font color='red'>提供 Docker 容器集群服务，是 Docker  官方对容器云生态进行支持的核心方案。使用它，用户可以将多个 Docker 主机封装为单个大型的虚拟 Docker 主机，快速打造一套容器云平台。</font>

> Docker 1.12 Swarm mode 已经内嵌入 Docker 引擎，成为了 docker 子命令 docker swarm。请注意与旧的 Docker Swarm 区分开来。

Swarm mode内置kv存储功能，提供了众多的新特性，比如：<font color='red'>具有容错能力的去中心化设计、内置服务发现、负载均衡、路由网格、动态伸缩、滚动更新、安全传输等。</font>使得 Docker 原生的 Swarm 集群具备与 Mesos、Kubernetes 竞争的实力。

## 3.1 基本概念

Swarm 是使用 [SwarmKit](https://github.com/docker/swarmkit/) 构建的 Docker 引擎内置（原生）的集群管理和编排工具。

### 节点:

 运行 Docker 的主机可以主动初始化一个 Swarm 集群或者加入一个已存在的 Swarm 集群，这样这个运行 Docker 的主机就成为一个 Swarm 集群的节点 (node)

节点分为管理 (`manager`) 节点和工作 (`worker`) 节点

- 管理节点：用于 Swarm 集群的管理，`docker swarm` 命令基本只能在管理节点执行（节点退出集群命令 `docker swarm leave` 可以在工作节点执行）。一个 Swarm 集群可以有多个管理节点，但只有一个管理节点可以成为 `leader`，leader 通过 raft 协议实现。
- 工作节点：是任务执行节点，管理节点将服务 (service) 下发至工作节点执行。管理节点默认也作为工作节点。也可以通过配置让服务只运行在管理节点。

集群中管理节点与工作节点的关系如下所示：
 ![这里写图片描述](https://img-blog.csdn.net/20180606213500838?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dmczE5OTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

###  服务和任务：

 任务 （`Task`）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。
 服务 （`Services`） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：

- `replicated services` 按照一定规则在各个工作节点上运行指定个数的任务。
- `global services` 每个工作节点上运行一个任务。

两种模式通过 `docker service create 的 --mode` 参数指定。
 容器、任务、服务的关系如下所示：
 ![这里写图片描述](https://img-blog.csdn.net/2018060621373610?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dmczE5OTQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

------

### swarm：

 从 v1.12 开始，集群管理和编排功能已经集成进 Docker Engine。当 Docker Engine 初始化了一个 swarm 或者加入到一个存在的 swarm 时，它就启动了 swarm mode。没启动  swarm mode 时，Docker 执行的是容器命令；运行 swarm mode 后，Docker 增加了编排 service 的能力。
 Docker 允许在同一个 Docker 主机上既运行 swarm service，又运行单独的容器。

### node：

 swarm 中的每个 Docker Engine 都是一个 node，有两种类型的 node：manager 和 worker。 
 为了向 swarm 中部署应用，我们需要在 manager node 上执行部署命令，manager node  会将部署任务拆解并分配给一个或多个 worker node 完成部署。manager node 负责执行编排和集群管理工作，保持并维护  swarm 处于期望的状态。swarm 中如果有多个 manager node，它们会自动协商并选举出一个 leader  执行编排任务。woker node 接受并执行由 manager node 派发的任务。默认配置下 manager node 同时也是一个  worker node，不过可以将其配置成 manager-only node，让其专职负责编排和集群管理工作。work node 会定期向  manager node 报告自己的状态和它正在执行的任务的状态，这样 manager 就可以维护整个集群的状态。

### service：

 service 定义了 worker node 上要执行的任务。swarm 的主要编排任务就是保证 service 处于期望的状态下。
 举一个 service 的例子：在 swarm 中启动一个 http 服务，使用的镜像是 httpd:latest，副本数为  3。manager node 负责创建这个 service，经过分析知道需要启动 3 个 httpd 容器，根据当前各 worker node  的状态将运行容器的任务分配下去，比如 worker1 上运行两个容器，worker2 上运行一个容器。运行了一段时间，worker2  突然宕机了，manager 监控到这个故障，于是立即在 worker3 上启动了一个新的 httpd 容器。这样就保证了 service  处于期望的三个副本状态。

默认配置下 manager node 也是 worker node，所以 swarm-manager 上也运行了副本。如果不希望在 manager 上运行 service，可以执行如下命令：

```bash
docker node update --availability drain master
```

------

## 3.2 创建swarm集群

Swarm 集群由管理节点和工作节点组成。现在我们来创建一个包含一个管理节点和两个工作节点的最小 Swarm 集群。

### 初始化集群：

 使用`Docker Machine`创建三个 Docker 主机，并加入到集群中

首先创建一个 Docker 主机作为管理节点：

```bash
docker-machine create -d virtualbox manger
```

使用 `docker swarm init` 在管理节点初始化一个 Swarm 集群

```bash
docker-machine ssh manger

docker@manger:~$ docker swarm init --advertise-addr 192.168.99.101
Swarm initialized: current node (fwh0yy8m8bygdxnsl7x1peioj) is now a manager.

To add a worker to this swarm, run the following command:

	docker swarm join --token SWMTKN-1-2acj2brip56iee9p4hc7klx3i6ljnpykh5lx6ea3t9xlhounnv-70knqo263hphhse02gxuvja12 192.168.99.101:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

docker@manger:~$
```

注意：如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 `--advertise-addr` 指定 IP。

> 执行 `docker swarm init` 命令的节点自动成为管理节点。

### 增加工作节点：

 在`docker swarm init` 完了之后，会提示如何加入新机器到集群，如果当时没有注意到，也可以通过下面的命令来获知 如何加入新机器到集群。

```bash
# docker swarm join-token worker [--quiet]
# docker swarm join-token manager [--quiet]
```

根据token的不同，我们来区分加入集群的是manager节点还是普通的节点。通过加入`–quiet`参数可以只输出token，在需要保存token的时候还是十分有用的。

上一步初始化了一个 Swarm 集群，拥有了一个管理节点，下面继续创建两个 Docker 主机作为工作节点，并加入到集群中。

```bash
# docker-machine create -d virtualbox worker1

# docker-machine ssh worker1
docker@worker1:~$ docker swarm join --token SWMTKN-1-2acj2brip56iee9p4hc7klx3i6ljnpykh5lx6ea3t9xlhounnv-70knqo263hphhse02gxuvja12 192.168.99.101:2377
This node joined a swarm as a worker.
docker@worker1:~$ 				


# docker-machine create -d virtualbox worker2
# docker-machine ssh worker2
docker@worker2:~$ docker swarm join --token SWMTKN-1-2acj2brip56iee9p4hc7klx3i6ljnpykh5lx6ea3t9xlhounnv-70knqo263hphhse02gxuvja12 192.168.99.101:2377
This node joined a swarm as a worker.
docker@worker2:~$ 
```

### 查看集群：

 在manager machine上执行 `docker node ls` 查看有哪些节点加入到swarm集群。

```bash
# docker-machine ssh manger
docker@manger:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
fwh0yy8m8bygdxnsl7x1peioj *   manger              Ready               Active              Leader              18.05.0-ce
0v6te4mspwu1d1b4250fp5rph     worker1             Ready               Active                                  18.05.0-ce
mld8rm9z07tveu1iknvws0lr1     worker2             Ready               Active                                  18.05.0-ce		
```

## 3.3 部署服务

使用 `docker service` 命令来管理 Swarm 集群中的服务，该命令只能在管理节点运行。

### 新建服务：

 在上一节创建的 Swarm 集群中运行一个名为 nginx 服务：

```bash
docker@manger:~$ docker service create --replicas 3 -p 80:80 --name nginx nginx
khw3av021hlxs3koanq85301j
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 
docker@manger:~$ 
```

使用浏览器，输入任意节点 IP ,即可看到 nginx 默认页面。

### 查看服务：

 使用 `docker service ls` 来查看当前 Swarm 集群运行的服务。

```bash
docker@manger:~$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
khw3av021hlx        nginx               replicated          3/3                 nginx:latest        *:80->80/tcp
docker@manger:~$			
```

使用 `docker service ps` 来查看某个服务的详情。

```bash
docker@manger:~$ docker service ps nginx
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR                   PORTS
6b900c649xyp        nginx.1             nginx:latest        worker1             Running             Running 5 minutes                                       
n55uhpjdurxs        nginx.2             nginx:latest        worker2             Running             Running 5 minutes ago                                   
l2uiyggbegsf        nginx.3             nginx:latest        manger              Running             Running 5 minutes ago                                   
docker@manger:~$	
```

使用 `docker service logs` 来查看某个服务的日志：

```bash
docker@manger:~$ docker service logs nginx
```

### 服务伸缩：

 可以使用 `docker service scale` 对一个服务运行的容器数量进行伸缩

当业务处于高峰期时，我们需要扩展服务运行的容器数量

```bash
$ docker service scale nginx=5
```

当业务平稳时，我们需要减少服务运行的容器数量

```bash
$ docker service scale nginx=2
```

### 删除服务:

 使用 `docker service rm` 来从 Swarm 集群移除某个服务。

```bash
$ docker service rm nginx
```



# 四、总结

Docker三剑客还是比较强大的，许多场景下能够解决的问题也是比较丰富的，但是在我后来使用和一直思考后，对于服务的编排还是比较认可Kubernetes的方式，在之后对Kubernetes有了更多自身的见解后，也会慢慢整理成系列文章。

但是对于Docker初学者或是Docker入门从业者，我建议还是要先掌握Docker Compose和Docker Swarm，Compose可以通过实战去锻炼，比如搭建Harbor，Jenkins或是其他服务的方式去感受，Swarm的话，可以自己搞三个虚拟机，然后自己实操锻炼锻炼，这部分基础内容还是有必要了解的。

虽然Kubernetes这两年的新版本要抛弃了Docker，但是Docker作为容器化的技术依然值得去深耕学习，通过对Docker的学习和了解之后，再去了解其他容器化技术，能够触类旁通，事半功倍。

给大家分享几个Docker三剑客的优秀文章：

- Docker三剑客之Docker Swarm：  https://www.cnblogs.com/zhujingzhi/p/9792432.html
- Docker三剑客之Docker Compose：   https://www.cnblogs.com/zhujingzhi/p/9786622.html
- Docker三剑客之Docker Machine：   https://www.cnblogs.com/zhujingzhi/p/9760198.html



之前学习三剑客时，看了这位博主的文章，写的很不错一直也有在记录学习，刻意分享一下。



# Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)

