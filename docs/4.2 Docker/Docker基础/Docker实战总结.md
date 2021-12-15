- [Docker实战总结（非常全面，建议收藏）](https://segmentfault.com/a/1190000024505902)

# **一、  Docker简介**

Docker是一个开源的应用容器引擎，开发者可以打包自己的应用到容器里面，然后迁移到其他机器的docker应用中，可以实现快速部署。

简单的理解，docker就是一个软件集装箱化平台，就像船只、火车、卡车运输集装箱而不论其内部的货物一样，软件容器充当软件部署的标准单元，其中可以包含不同的代码和依赖项。

按照这种方式容器化软件，开发人员和 IT 专业人员只需进行极少修改或不修改，即可将其部署到不同的环境，如果出现的故障，也可以通过镜像，快速恢复服务。

![img](https://segmentfault.com/img/remote/1460000024505905)

# 二、Docker优势

## 1. 特性优势

![img](https://segmentfault.com/img/remote/1460000024505906)

2.　资源优势

![img](https://segmentfault.com/img/remote/1460000024505907)

# 三、Docker基本概念

**Client（客户端）**：是Docker的用户端，可以接受用户命令和配置标识，并与Docker daemon通信。

**Images（镜像）**：是一个只读模板，含创建Docker容器的说明，它与操作系统的安装光盘有点像。

**Containers（容器）**：镜像的运行实例，镜像与容器的关系类比面向对象中的类和对象。

**Registry（仓库）：**是一个集中存储与分发镜像的服务。最常用的Registry是官方的Docker Hub 。

![img](https://segmentfault.com/img/remote/1460000024505908)

# 四、Docker安装使用

- **操作系统：CentOS 7**

**1、安装依赖**

> yum install -y yum-utils device-mapper-persistent-data lvm2

**2、添加软件源**

> yum-config-manager --add-repo [http://mirrors.aliyun.com/doc...](http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo)　　# 指定阿里云镜像源

**3、安装docker-ce**（对系统内核有一定要求，centos6不支持）

> yum clean all  yum makecache fast    # 重新生成缓存
>
> **yum -y install docker-ce docker-ce-cli containerd.io**

**4、设置自启并启动**

> systemctl enable docker
>
> **systemctl start docker**

**5、查看版本**

> docker version

![img](https://segmentfault.com/img/remote/1460000024505911)

- **运行示例：Nginx**

## 1、搜索并下载镜像

> docker search nginx
>
> docker pull nginx

![img](https://segmentfault.com/img/remote/1460000024505909)

## 2、启动一个容器并映射端口到本地

> docker run -d -p 8080:80 --name Nginx nginx　　 # 参数详解见下文

![img](https://segmentfault.com/img/remote/1460000024505910)

## 3、访问本地映射端口

![img](https://segmentfault.com/img/remote/1460000024505914)

# 五、Docker常用命令

## 1.镜像控制

> 搜索镜像：docker search [OPTIONS] TERM
>
> 上传镜像：docker push [OPTIONS] NAME[:TAG]
>
> 下载镜像：docker pull [OPTIONS] NAME[:TAG]
>
> 提交镜像：docker commit [OPTIONS] CONTAINER NAME[:TAG]
>
> 构建镜像：docker build [OPTIONS] PATH
>
> 删除镜像：docker rmi [OPTIONS] IMAGE [IMAGE...]
>
> 增加镜像标签：docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
>
> 查看所有镜像：docker images [OPTIONS] [REPOSITORY[:TAG]]

![img](https://segmentfault.com/img/remote/1460000024505912)

## 2.容器控制

> 启动/重启容器：docker start/restart CONTAINER
>
> 停止/强停容器：docker stop/ kill CONTAINER
>
> 删除容器：docker rm [OPTIONS] CONTAINER [CONTAINER...]
>
> 重命名容器：docker rename CONTAINER CONTAINER_NEW
>
> 进入容器：docker attach CONTAINER
>
> 执行容器命令：docker exec CONTAINER COMMAND
>
> 查看容器日志：docker logs [OPTIONS] CONTAINER
>
> 查看容器列表：docker ps [OPTIONS]

![img](https://segmentfault.com/img/remote/1460000024505913)

## 3.容器启动

> docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

**-d :** 后台运行容器，并返回容器ID

**-i：**以交互模式运行容器，通常与 -t 同时使用

**-t：**为容器重新分配一个伪输入终端，通常与 -i 同时使用

**-v：**绑定挂载目录

**--name="mycontainer":** 为容器指定一个名称

**--net="bridge":** 指定容器的网络连接类型，支持如下：

bridge / host / none / container:<name|id>

**-p/-P :**端口映射，格式如图：

![img](https://segmentfault.com/img/remote/1460000024505918)

## 4. 其他命令

> 查看docker信息：docker info
>
> docker命令帮助：docker run --help
>
> 复制文件到容器：docker cp custom.conf Nginx:/etc/nginx/conf.d/
>
> 更新容器启动项：docker container update --restart=always nginx
>
> 查看docker日志：tail -f /var/log/messages

![img](https://segmentfault.com/img/remote/1460000024505915)

![img](https://segmentfault.com/img/remote/1460000024505916)

# 六、Docker镜像构建

## 1.Docker commit（1运行2修改3保存）

**a）  运行容器**

> docker run -dit -p 8080:80 --name Nginx nginx

**b）  修改容器**（这里我只是做个演示，所以就复制一下文件，具体修改需要根据你实际情况）

> docker cp custom.conf Nginx:/etc/nginx/conf.d/

**c）  将容器保存为新的镜像**

> docker commit Nginx zwx/nginx

![img](https://segmentfault.com/img/remote/1460000024505917)

## 2.   Dockerfile（1编写2构建）

**a）编写Dockerfile文件**

> vim Dockerfile

![img](https://segmentfault.com/img/remote/1460000024505920)

**b）执行Dockerfile文件**

> docker build -t zwx/nginx .　　　　# 后面有个点，代表当前目录下dockerfile文件

![img](https://segmentfault.com/img/remote/1460000024505919)

## 3. Dockerfile 常用指令

![img](https://segmentfault.com/img/remote/1460000024505921)

# 七、Docker本地仓库

**1、拉取镜像仓库**

> docker search registry
>
> docker pull registry

**2、启动镜像服务**

> docker run -dit
>
> --name=Registry 　　　　# 指定容器名称
>
> -p 5000:5000 　　　　　 # 仓库默认端口是5000，映射到宿主机，这样可以使用宿主机地址访问
>
> --restart=always         # 自动重启，这样每次docker重启后仓库容器也会自动启动
>
> --privileged=true        # 增加安全权限，一般可不加
>
> -v /usr/local/my_registry:/var/lib/registry 　　　　# 把仓库镜像数据保存到宿主机
>
> registry

**3、注册https协议**（需要通过本地仓库下载镜像，均需要配置）

> vim /etc/docker/daemon.json　　　　　　　　# 默认无此文件，需自行添加，有则追加一下内容。
>
>   { "insecure-registries":[" xx.xx.xx.xx:5000"] }　　# 指定ip地址或域名

**4、新增tag指明仓库地址**

> docker **tag** zwx/nginx x.xx.xx.xx:5000/zwx/nginx　　# 如果构建时已经指定仓库地址，则可以省略

**5、上传镜像到本地仓库**

> docker **push** x.xx.xx.xx:5000/zwx/nginx

**6、查看本地仓库**

> curl -XGET [http://x.xx.xx.xx](http://x.xx.xx.xx/):5000/v2/_catalog

![img](https://segmentfault.com/img/remote/1460000024505923)

# 八、Docker与图形管理工具Portainer

## 1.简介

Portainer是Docker的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、

事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。

![img](https://segmentfault.com/img/remote/1460000024505922)

## 2.安装使用

**a） 搜索并下载镜像**

> docker search portainer
>
> docker pull portainer/portainer

**b） 单机方式运行**

> docker run -d
>
> -p 9000:9000 　　　# portainer默认端口是9000，映射到本地9000端口，通过本地地址访问
>
> --restart=always 　　# 设置自动重启
>
> -v /var/run/docker.sock:/var/run/docker.sock 　　# 单机必须指定docker.sock
>
> --name Prtainer portainer/portainer

**c） 访问[http://localhost](http://localhost/):9000**

首次登陆需要注册用户，给admin用户设置密码，然后单机版选择local连接即可。

![img](https://segmentfault.com/img/remote/1460000024505924)

**d） 控制管理**

![img](https://segmentfault.com/img/remote/1460000024505925)



