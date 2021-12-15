部分内容参考链接：

- [Docker实战总结（非常全面，建议收藏）](https://segmentfault.com/a/1190000024505902)
- [Docker核心技术从入门到精通](https://juejin.cn/post/6850418119304413197)



# 一、  Docker概述

Docker是一个开源的应用容器引擎（基于Go语言开发），让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。简言之，就是可以在Linux上镜像使用的这么一个容器。



Docker可以在容器内部快速自动化部署应用，并可以通过内核虚拟化技术（namespaces及cgroups等）来提供容器的资源隔离与安全保障等。

> Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送水果的船和专门运送化学品的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都运走。



简单的理解，<font color='red'>docker就是一个软件集装箱化平台，就像船只、火车、卡车运输集装箱而不论其内部的货物一样，软件容器充当软件部署的标准单元，其中可以包含不同的代码和依赖项。</font>

按照这种方式容器化软件，开发人员和 IT 专业人员只需进行极少修改或不修改，即可将其部署到不同的环境，如果出现的故障，也可以通过镜像，快速恢复服务。

## 1.1 Docker的优势

![image-20210202112442718](http://lovebetterworld.com/image-20210202112442718.png)

## 1.2 Docker理念

1. Docker是基于GO语言实现的云开源项目
2. Docker的主要目标是：“Build，Ship and Run Any App，Anywhere”。也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，即“一次封装，到处运行”



## 1.3 比较Docker和传统虚拟化方式的不同

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一套完整的操作系统，在该系统上运行所需要的应用进程；（分钟级）
- 而容器的应用进程之间运行于宿主的内核容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。（秒级）
- 每个容器之间相互隔离，每个容器都有自己的文件系统，容器之间进程不会相互影响，能区分计算资源。





# 二、从容器化技术说起

## 2.1 背景

在虚拟机和云计算较为成熟的时候，各家公司想在云服务器上部署应用，通常都是像部署物理机那样使用脚本或手动部署，但由于本地环境和云环境不一致，往往会出现各种小问题。

这时候有个叫Paas的项目，就是专注于解决本地环境与云端环境不一致的问题，并且提供了**应用托管**的功能。简单得说，就是在云服务器上部署Paas对应的服务端，然后本机就能一键push，将本地应用部署到云端机器。然后由于云服务器上，一个Paas服务端，会接收多个用户提交的应用，所以其底层提供了一套隔离机制，为每个提交的应用创建一个**沙盒**，每个沙盒之间彼此隔离，互不干涉。

看看，这个沙盒是不是和docker很类似呢？**实际上，容器技术并不是docker的专属，docker只是众多实现容器技术中的一个而已**。



## 2.2 docker实现原理

说起docker，很多人都会将它与虚拟机进行比较，

![image-20210202111548325](http://lovebetterworld.com/image-20210202111548325.png)

其中左边是虚拟机的结构，右边是docker容器的结构。

在虚拟机中，通过Hypervisor对硬件资源进行虚拟化，在这部分硬件资源上安装操作系统，从而可以让上层的虚拟机和底层的宿主机相互隔离。



但docker是没有这种功能的，我们在docker容器中看到的与宿主机相互隔离的**沙盒**环境（文件系统，资源，进程环境等），**本质上是通过Linux的Namespace机制，CGroups（Control Groups）和Chroot等功能实现的。实际上Docker依旧是运行在宿主机上的一个进程（进程组）**，只是通过一些障眼法让docker以为自己是一个独立环境。

所以，容器=Cgroup+Namespace+rootfs+容器引擎（用户态工具）

- Cgroup：资源控制。
- Namespace：访问隔离。
- rootfs：文件系统隔离。
- 容器引擎：生命周期控制。



## 2.3 Cgroup

Cgroup是control group的简写，属于Linux内核提供的一个特性，用于限制和隔离一组进程对系统资源的使用，也就是做资源QoS，这些资源主要包括CPU、内存、block I/O和网络带宽。

- devices：设备权限控制。
- cpuset：分配指定的CPU和内存节点。
- cpu：控制CPU占用率。
- cpuacct：统计CPU使用情况。
- memory：限制内存的使用上限。
- freezer：冻结（暂停）Cgroup中的进程。
- net_cls：配合tc（traffic controller）限制网络带宽。
- net_prio：设置进程的网络流量优先级。
- huge_tlb：限制HugeTLB的使用。
- perf_event：允许Perf工具基于Cgroup分组做性能监测。



## 2.4 NameSpace

Namespace又称为命名空间（也称为名字空间），它是将内核的全局资源做封装，使得每个Namespace都有一份独立的资源，因此不同的进程在各自的Namespace内对同一种资源的使用不会互相干扰。

- IPC：隔离System V IPC和POSIX消息队列。
- Network：隔离网络资源。
- Mount：隔离文件系统挂载点。
- PID：隔离进程ID。
- UTS：隔离主机名和域名。
- User：隔离用户ID和组ID。



## 2.5 彻底了解docker隔离机制

如果在一个docker容器里面，使用ps命令查看进程，可能只会看到如下的输出：

```bash
/ # ps
PID  USER   TIME COMMAND
  1 root   0:00 /bin/bash
  10 root  0:00 ps
```

在容器中执行ps，只会看到1号进程/bin/bash和10号进程ps。前面有说到，docker容器本身只是Linux中的一个进程（组），也就是说在宿主机上，这个/bin/bash的pid可能是100或1000，那为什么在docker里面看到的这个/bin/bash进程的pid是1呢？**答案是linux提供的Namespace机制，将/bin/bash这个进程的进程空间隔离开了**。



具体的做法呢，就是在创建进程的时候添加一个可选的参数，比如下面这样：

```
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```

那样后，创建的线程就会有一个新的命名空间，在这个命名空间中，它的pid就是1，当然在宿主机的真实环境中，它的pid还是原来的值。



上面的这个例子，其实只是pid Namespace（进程命名空间），除此之外，还有network Namespace（网络命名空间），mount Namespace（文件命名空间，就是将整个容器的根目录root挂载到一个新的目录中，然后在其中放入内核文件看起来就像一个新的系统了）等，用以将整个容器和实际宿主机隔离开来。而这其实也就是容器基础的基础实现了。



但是，上述各种Namespace其实还不够，还有一个比较大的问题，**那就是系统资源的隔离**，比如要控制一个容器的CPU资源使用率，内存占用等，否则一个容器就吃尽系统资源，其他容器怎么办。

**而Linux实现资源隔离的方法就是Cgroups**。Cgroups主要是提供文件接口，即通过修改 /sys/fs/cgroup/下面的文件信息，比如给出pid，CPU使用时间限制等就能限制一个容器所使用的资源。

**所以，docker本身只是linux中的一个进程，通过Namespace和cgroup将它隔离成一个个单独的沙盒**。明白这点，就会明白docker的一些特性，比如说太过依赖内核的程序在docker上可能执行会出问题，比如无法在低版本的宿主机上安装高本版的docker等，因为本质上还是执行在宿主机的内核上。



# 三、Docker的核心：镜像、容器、仓库



## 3.1 Docker的基本组成

- docker架构图

  

  ![image-20200925114421569](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5a2787685a44c65b51d6ecc23b0eb46~tplv-k3u1fbpfcp-zoom-1.image)



## 3.2 镜像

镜像是一个可执行包，包含运行应用程序所需的所有内容——代码、运行时、库、环境变量和配置文件。



docker镜像就是一个只读模板。镜像可以用来创建Docker容器，一个镜像可以创建很多容器。

例子： 1.  Person a=new Person() 2.  Person b=new Person() 3.  Person c=new Person() 4.  a、b、c三个不同的实例，就是三个不同的容器。但都来自于同一个模板（类） 就是docker中的镜像。




## 3.3 容器

**容器：**

- 容器是通过运行镜像启动容器，是镜像的运行时实例。镜像实际上就是一个容器的模板，通过这个模板可以创建很多相同的容器。
  - 容器就是一个认为只有其本身在运行状态的linux程序，只服从用户指定的命令。（容器程序有自己的IP地址；一个可访问网络的独立设备）



通过Java去类比理解Docker的一些概念：
![](https://img-blog.csdnimg.cn/20200128042708714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0V2YW5fTGV1bmc=,size_16,color_FFFFFF,t_70)

- Class文件 - 相当于Docker镜像，定义了类的一些所需要的信息
- 对象 - 相当于容器，通过Class文件创建出来的实例
- JVM - 相当于Docker引擎，可以让Docker容器屏蔽底层复杂逻辑，实现跨平台操作



Docker里有容器（Container）独立运行一个或一组应用。容器是用镜像创建的运行实例。

它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做一个简易的Linux环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

容器的定义和镜像几乎一模一样，也是一对层的统一视角，**唯一区别**在于容器的最上面那一层是**可读可写**。

## 3.4 仓库

存放镜像的地方，和git仓库类似。

仓库（repository）是集中存放镜像文件的场所。

仓库和仓库注册服务器（registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含多个镜像，每个镜像有不同的标签（Tag）。

仓库分为公开仓库和私有仓库两种形式。

最大的公开仓库是Docker Hub（[hub.docker.com/）,存放了数量庞大的镜…](https://hub.docker.com/）,存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云、网易云等。)




1. Docker本身就是一个容器运行载体或者称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看作是容器模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个溶蚀运行的容器实例。
2. image文件生成的容器实例，本身也是一个文件，称为镜像文件

3. 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

4. 至于仓库，就是存放了一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来



# 四、Docker的安装方式

## 4.1 方式一：yum方式安装

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



## 4.2 方式二：curl

```bash
curl -sSL https://get.daocloud.io/docker | sh
```



## 4.3 离线安装

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

# 五、Docker是怎么工作的

1. Docker是一个Client-Server的结构系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。**容器，是一个运行时环境，就是我们最初说的集装箱**

   ![image-20201012151434264](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa601c6aaa314c00a149c00a26fb21f4~tplv-k3u1fbpfcp-zoom-1.image)



# 六、Docker为什么比VM快

1. docker有着比虚拟机更少的抽象层。由于Docker不需要`Hypervisor`实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此`cpu`、内存利用率上docker将会在效率上有明显的优势。

2. docker利用的宿主机内核，而不需要Guest OS。因此，当建立一个容器时，docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免寻求、加载操作系统内核比较费时费资源的过程。当新建一个虚拟机时，虚拟机软件需加载Guest OS，返个新建过程是**分钟级别**的。而docker由于直接利用宿主机的操作系统，则省略了返个过程，因此**新建一个docker容器只需要几秒钟**。

   |            | Docker容器              | 虚拟机（VM）                |
   | ---------- | ----------------------- | --------------------------- |
   | 操作系统   | 与宿主机共享OS          | 宿主机OS上运行虚拟机OS      |
   | 存储大小   | 镜像小，便于存储与传输  | 镜像庞大（vmdk、vdi等）     |
   | 运行性能   | 几乎无额外性能损失      | 操作系统额外的CPU、内存消耗 |
   | 移植性     | 轻便、灵活，适用于Linux | 笨重，与虚拟化技术耦合度高  |
   | 硬件亲和性 | 面向软件开发者          | 面向硬件运维者              |
   | 部署速度   | 快速，秒级              | 较慢，10s以上               |

# 七、Docker常用命令

### 7.1 帮助命令

- docker version   查看docker版本信息
- docker info         docker详细信息
- docker  --           ` 类似linux中的man命令  查看命令帮助`

### 7.2 镜像命令

1. docker images [OPTIONS]         列出本地主机上的镜像

   ```
    1. 各个选项说明
    * REPOSITORY：表示镜像的仓库源
    * TAG：镜像的标签
    * IMAGE ID：镜像ID
    * CREATED：镜像创建的时间
    * SIZE： 镜像大小
   ```
   
1. OPTIONS参数说明：
      - -a：列出本地所有的镜像（含中间映像层）
      - -q：显示镜像ID
      - `-qa： 显示所有镜像的ID`
      - --digests ： 显示镜像的摘要信息
      - `--no-trunc：显示完整的镜像信息（不要截取）`
   
2. docker search [OPTIONS]  某个镜像的名字

   1. 网站 ：[hub.docker.com](https://hub.docker.com)
   2. 命令：docker search [OPTIONS]  镜像的名字
      - `--no --trunc : 显示完整的镜像描述`
      - -s :列出收藏不小于指定值的镜像（点赞数）
      - `--autmated：只列出automated build类型的镜像`

3. `docker pull   某个XXX镜像的名字`

   - `docker  pull tomcat  等价于 docker pull tomcat：latest`

4. `docker rmi  某个镜像的名字ID`

   - 删除单个 ` docker rmi -f 镜像ID`
   - 删除多个 ` docker  rmi -f  镜像名1：TAG  镜像名2：TAG`
   - 删除全部  `docker rmi -f  $（docker images -qa）`

### 7.3 容器命令

1. 有镜像才能创建容器，这是根本前提（下载一个`CentOS`的镜像演示 ）`docker pull centos`

2. 新建并启动容器 docker run [OPTIONS] IMAGE [COMMAND]

   - OPTIONS说明(常用):有些是一个 -号  有些是-- 两个

     1. --name="容器的新名字"：为容器指定一个名称

     2. **-d：后台运行容器，并返回容器ID，也即启动守护式容器**

     3. **-i：已交互模式运行容器，通常与-t同时使用**

     4. **-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用**

     5. -P：随机端口映射

     6. -p：指定端口映射，有以下四种格式 * 

        ```
        ip：hostPort:containerPort
        ```

        - `ip::containerPort`
        - `hostPort:containerPort`
        - `containerPort`

3. 列出当前所有**正在运行**的容器

   - `docker ps[OPTIONS]`

     options：

     1. -l 上次运行的
        1. -a 全部状态的
        2. -n 3  上三次运行过得（number）
        3. -q   只显示容器编号
        4. `-lq 上次运行的容器编号`

4. 退出容器

   - exit    容器停止退出
   - `ctrl+P+Q  容器不停止退出`

5. 启动容器

   - docker start 容器ID或者容器名字

6. 重启容器

   - docker restart 容器ID

7. 停止容器

   - docker stop 容器ID

8. 强制停止容器

   - docker kill  容器ID

9. 删除已停止的容器

   - docker rm
   - 删除并停止 docker rm -f

10.一次性删除多个容器

- `docker rm -f $(docker ps -a -q)`
- `docker ps -a -q |xargs docker rm`

### 7.4 重要

1. docker run -d 容器名  **启动守护式容器**  以后台模式启动一个容器

   ​	很重要的 要说明的一点：**Docker容器后台运行，就必须有一个前台进程。**容器运行的命令如果不是那些

   一直挂起的命令（比如运行top，tail），就是会**自动退出**的。

   ​	这是docker的机制问题，比如你的web容器，我们以`nginx`为例，正常情况下，我们配置启动服务只需要启动响应式service即可。例如`service nginx start`，但这样做，`nginx`为后台进程模式运行，就导致docker前台没有运行的应用，这样的容器后台启动后，会立即自杀因为他觉得他没事可做了。

   **所以，最佳的解决方案是，将你要运行的程序以前台进程的形式运行。**

2. docker logs -f -t --tail 容器ID  查看容器的日志

   - -t   是加入时间戳
   - -f    跟随最新当然日志打印
   - --tail  数字 显示最后多少条

3. docker top 容器ID  查看容器内的进程

4. docker inspect 容器ID  查看容器内部细节

   **docker 镜像是一层套一层 就像一个同心圆一样，整个容器以一个嵌套json串的形式来描述**

5. **进入正在运行的容器并以命令行交互**

   1. `docker exec -it 容器ID bashshell   ` `（linux中 exec命令执行脚本）`
      - `docker exec **-it**  容器ID  /bin/bash  ` 进入容器终端
      - `docker exec **-t** 容器ID   ls -l /tmp`   不进入容器 直接执行脚本
   2. docker attach 容器ID   重新进入

   两者的区别：**1的操作不进入容器  直接返回shell脚本操作的结果**

   ​					  **2的操作 直接进入容器的命令终端**

6. `docker cp` 容器ID：容器内路径 目的主机路径   从容器内拷贝文件到主机上

### 7.5 小总结（常用命令）

```bash
attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
commit    Create a new image from a container changes   # 提交当前容器为新的镜像
cp        Copy files/folders from the containers filesystem to the host path   #从容器中拷贝指定文件或者目录到宿主机中
create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
diff      Inspect changes on a container's filesystem   # 查看 docker 容器变化
events    Get real time events from the server          # 从 docker 服务获取容器实时事件
exec      Run a command in an existing container        # 在已存在的容器上运行命令
export    Stream the contents of a container as a tar archive   # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
history   Show the history of an image                  # 展示一个镜像形成历史
images    List images                                   # 列出系统当前镜像
import    Create a new filesystem image from the contents of a tarball # 从tar包中的内容创建一个新的文件系统映像[对应export]
info      Display system-wide information               # 显示系统相关信息
inspect   Return low-level information on a container   # 查看容器详细信息
kill      Kill a running container                      # kill 指定 docker 容器
load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
login     Register or Login to the docker registry server    # 注册或者登陆一个 docker 源服务器
logout    Log out from a Docker registry server          # 从当前 Docker registry 退出
logs      Fetch the logs of a container                 # 输出当前容器日志信息
port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT    # 查看映射端口对应的容器内部源端口
pause     Pause all processes within a container        # 暂停容器
ps        List containers                               # 列出容器列表
pull      Pull an image or a repository from the docker registry server   # 从docker镜像源服务器拉取指定镜像或者库镜像
push      Push an image or a repository to the docker registry server    # 推送指定镜像或者库镜像至docker源服务器
restart   Restart a running container                   # 重启运行的容器
rm        Remove one or more containers                 # 移除一个或者多个容器
rmi       Remove one or more images             # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
run       Run a command in a new container              # 创建一个新的容器并运行一个命令
save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
start     Start a stopped containers                    # 启动容器
stop      Stop a running containers                     # 停止容器
tag       Tag an image into a repository                # 给源中镜像打标签
top       Lookup the running processes of a container   # 查看容器中运行的进程信息
unpause   Unpause a paused container                    # 取消暂停容器
version   Show the docker version information           # 查看 docker 版本号
wait      Block until a container stops, then print its exit code   # 截取容器停止时的退出状态值
```



# 八、Docker镜像原理

## 8.1 是什么

1. 镜像是一种轻量级，可执行的独立软件包，用来打包软件运行环境和机遇运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

2. `UnionFS`（联合文件系统）

   - Union文件系统（`UnionFS`）是一种分层、轻量级并且高性能的文件系统，它**支持对文件系统的修改作为一次提交来一层层的叠加**，同时可以将不同目录挂载到同一个虚拟文件系统下(`unite several directories into a single virtual filesystem`)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像
   - 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

3. Docker镜像加载原理

   - docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统`UnionFS`

     > `bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel(Linux内核), Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs`   相当于Linux内核
     >
     > `rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等` 相当于Linux的版本选择

   ### 问题：平时我们安装进虚拟机的`CentOS`都是好几个G，为什么docker这里才`200M`？

   ​	对于一个精简的OS，`rootfs`可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为**底层直接用Host的kernel**(**宿主机的内核**)，自己只需要提供 `rootfs `就行了。由此可见对于不同的`linux`发行版, `bootfs`基本是一致的, `rootfs`会有差别, 因此不同的发行版可以公用`bootfs`

4. 分层的镜像

   - 以我们的pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载

5. 为什么Docker镜像要采用这种分层的结构呢

   - 最大的一个好处就是 - **共享资源**

     比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像， 同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享

   ![image-20201012152044180](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3e0196221534ce788f8c8f16e6994a5~tplv-k3u1fbpfcp-zoom-1.image)

## 8.2 特点

Docker镜像都是只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。 这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

## 8.3 Docker镜像commit操作补充

1. docker commit 提交容器副本使之成为一个新的镜像

2. docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

3. 案例演示：

   1. 从hub上下载tomcat镜像到本地并成功运行
   2. 故意删除上一步镜像生产tomcat容器的文档docs
   3. 以删除后的容器为模板，commit一个没有docs的tomcat 新镜像` tomcat/日期`

   ```shell
   docker commit -a="bin.wang" -m="tomcat without docs" c005584f57c5 touchair/tomcat:1.0
   ```
   
1. 启动我们的新镜像并和原来的对比
      - 启动 tomcat/日期 ，他没有docs
      - 新启动的原来的tomcat ，他有docs
   
4. **注意：** 新版本Tomcat 默认`webapps`中没有主页文件 在`webapps.dist`中

![image-20200929152825261](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/838e022eaefd40caaa15ee2cf9268366~tplv-k3u1fbpfcp-zoom-1.image)



# 九、Docker容器数据卷

## 9.1 是什么？（有点类似`Redis`中的`rdb`和`aof`文件）

1. 先来看看Docker的理念：
   - 将应用与运行环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
   - 容器之间希望有可能共享数据
2. Docker容器产生的数据，如果不通过docker commit 生成新的镜像，使得数据作为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了
3. 为了能保存数据在docker中 我们使用卷

## 9.2 能干什么？（容器的持久化、容器间的继承＋共享数据）

1. 卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过`UNION File System`提供的一些用于持续存储或共享数据的特性：

   卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

2. 特点：

   - 数据卷可在容器之间共享或重用数据
   - 卷中的更改可以直接生效
   - 数据卷中的更改不会包含在镜像更新中
   - 数据卷的生命周期一直持续到没有容器使用它为止

## 9.3 数据卷（容器内添加）

### 9.3.1 直接命令添加

- 命令： docker run -it -v/宿主机绝对路径目录:/容器内目录  镜像名

- ```shell
  docker run -it -v /var/myDataVolume:/dataVolumeContainer centos
  ```

##### 查看数据卷是否挂载成功

- docker inspect 容器id

  ![image-20201009133025195](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/274daa4df25d4451b366ef86fd460c2d~tplv-k3u1fbpfcp-zoom-1.image)

##### 容器和宿主机之间数据共享

- 第一步：在主机的 `/var/myDataVolume` 目录下 添加 host.txt 文件

- 第二步：进入容器，可以看到 /dataVolumeContainer/ 目录下 多出了 host.txt 文件

- 第三步：vi host.txt  添加内容 container update

- 第四步：主机查看 host.txt 文件 ，多了 container update

  ![image-20201009133600858](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95842bc719414b7bb8a2527f37d15dd3~tplv-k3u1fbpfcp-zoom-1.image)

##### 容器停止退出后，主机修改后数据是否同步？

- 第一步：exit 退出容器
- 第二步：主机 vim host.txt；添加 host update；保存退出
- 第三步：重新启动容器，查看容器内的 /dataVolume/Container/ 目录；查看 host.txt文件
- 第四步：cat host.txt  可以看到 host update
- 结论：**完全同步**

##### 命令带权限

- ```shell
  docker run -it -v /宿主机绝对路径目录：/容器内目录：ro 镜像名
  ```
  
- ro : read-only  只允许主机单方面写操作，容器只能查看 并不能增删改

### 9.3.2 DockerFile添加

##### 是什么

- Docker images  =====> `DockerFile`  (对镜像的源码级描述)
- 类比java中的：`JavaEE Hello.java`  ----> `Hello.class`

##### 编写dockerfile

1. 根目录下新建`var/mydocker`文件夹并进入；

2. 可在`Dockerfile`中使用VOLUME指令来给镜像添加一个或多个数据卷

   - 说明：出于可移植和分享的考虑，用-v主机目录：容器目录这种方法不能够直接在`DockerFile`中实现。由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。

3. File构建 vim DockerFile

   ```shell
   # volume test
   FROM centos
   VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
   CMD echo "finished,-------success1"
   CMD /bin/bash
   ```
   
4. build后生成镜像，获得一个新镜像 touchair/centos

   - ```shell
     docker build -f /var/mydocker/Dockerfile -t touchair/centos .
     ```
     
![image-20201009141827393](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e319ea0172b947b48383098cec20c524~tplv-k3u1fbpfcp-zoom-1.image)
     
- docker images 查看生成的新镜像
   
  ![image-20201009141902476](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26916fd0543d4427b238820d43cb0e0c~tplv-k3u1fbpfcp-zoom-1.image)
   
5. run容器

   ```shell
   docker run -it touchair/centos /bin/bash
   ```
   
![image-20201009142206678](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f94e4200f06420b8a920a3b1bf3c6e7~tplv-k3u1fbpfcp-zoom-1.image)
   
6. 通过上述步骤，容器内的卷目录地址已经知道对应的主机目录地址在哪？？

   - 主机对应默认地址  （通过`docker inspect 容器ID ` 查看）

     ![image-20201009142605774](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2348f0bcd02a4947bf574008ad0c83a4~tplv-k3u1fbpfcp-zoom-1.image)

   - ![image-20201009142725844](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da7f63bfe3934639a22a781d4626b155~tplv-k3u1fbpfcp-zoom-1.image)

### 9.3.3 备注

- Docker挂载主机目录，Docker访问出现cannot open directory：Permission denied
- 解决办法：在挂载目录后多加一个 --privileged=true 参数即可

## 9.4 数据卷容器

### 9.4.1 是什么

- 命名的容器挂载数据卷，其它容器通过挂载这个（父容器）实现数据共享，挂载数据卷的容器，称之为数据卷容器

### 9.4.2 总体介绍

- 以上一步新建的镜像 touchair/centos 为模板并运行容器 dc01/dc02/dc03
- 她们已经具有容器卷
  - /dataVolumeContainer1
  - /dataVolumeContainer2

### 9.4.3 容器间传递共享(--volumes-from)

- 第一步：运行容器 dc01

  ```shell
  docker run -it --name dc01 touchair/centos
  ```
  
在dc01容器中的 dataVolumeContainer2 目录下添加 dc01_add.txt 文件![image-20201010101422602](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18a86034c72b4628aab8be1bfe2742b6~tplv-k3u1fbpfcp-zoom-1.image)
  
- 第二步：运行容器dc02 dc01作为数据卷容器

  ```shell
  docker run -it --name dc02 --volumes-from dc01 touchair/centos
  ```
  
在dc02容器中的 dataVolumeContainer2 目录下添加 dc02_add.txt 文件![image-20201010102007560](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd1e7402266246609627634edfa540d6~tplv-k3u1fbpfcp-zoom-1.image)
  
- 第三步：运行容器 dc03 dc01作为数据容器卷

  ```shell
  docker run -it --name dc03 --volumes-from dc01 touchair/centos
  ```
  
![image-20201010112659621](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea1039705c4d4d52b2cec417d7403865~tplv-k3u1fbpfcp-zoom-1.image)
  
- 第四步：回到dc01可以看到02/03各自添加的都能共享了![image-20201010114107689](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57cec3df91204bcca657868b79d89b06~tplv-k3u1fbpfcp-zoom-1.image)

- 第五步：删除 dc01, dc02 修改后 dc03 可否访问？ 可以！![image-20201010114712516](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/976732f107fe4fc59ceba3e08659d39d~tplv-k3u1fbpfcp-zoom-1.image)

- 第六步：删除02后，03依旧可以访问

- 第七步：新建 dc04 继承 dc03 后再删除 dc03

  ```shell
  docker run -it --name dc04 --volumes-from dc03 touchair/centos
  ```
  
![image-20201010115048973](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80d1e707b76d48859c2a21ee41718fb9~tplv-k3u1fbpfcp-zoom-1.image)

> **结论**：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止





# 附录：Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)

