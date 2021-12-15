微信公众号：史上最全、最详细的Docker学习资料

## 一、Docker 简介

#### Docker 两个主要部件：

Docker: 开源的容器虚拟化平台

- Docker Hub: 用于分享、管理 Docker 容器的 Docker SaaS 平台 -- Docker Hub
- Docker 使用客户端-服务器 (C/S) 架构模式。Docker 客户端会与 Docker 守护进程进行通信。Docker  守护进程会处理复杂繁重的任务，例如建立、运行、发布你的 Docker 容器。Docker  客户端和守护进程可以运行在同一个系统上，当然你也可以使用 Docker 客户端去连接一个远程的 Docker 守护进程。Docker  客户端和守护进程之间通过 socket 或者 RESTful API 进行通信。

#### Docker 守护进程

如上图所示，Docker 守护进程运行在一台主机上。用户并不直接和守护进程进行交互，而是通过 Docker 客户端间接和其通信。

#### Docker 客户端

Docker 客户端，实际上是 docker 的二进制程序，是主要的用户与 Docker 交互方式。它接收用户指令并且与背后的 Docker 守护进程通信，如此来回往复。

#### Docker 内部

要理解 Docker 内部构建，需要理解以下三种部件：

- Docker 镜像 - Docker images
- Docker 仓库 - Docker registeries
- Docker 容器 - Docker containers

#### Docker 镜像

Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成。Docker 使用 UnionFS  来将这些层联合到单独的镜像中。UnionFS  允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。正因为有了这些层的存在，Docker  是如此的轻量。当你改变了一个 Docker  镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新 的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速。

#### Docker 仓库

Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。公有的 Docker 仓库名字是  Docker Hub。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。

#### Docker 容器

Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。每一个 Docker 容器都是从 Docker  镜像创建的。Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是  Docker 的运行部分。

## 二：Docker使用场景介绍

## [**Docker，你到底知道多少？**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486470&idx=2&sn=657fcc159ce64b3b7ffa8fc09ccdc217&chksm=e91b691ade6ce00cbe163389e6cc90018514fb1b3018b0760a202ae832932fc241543c2c6846&scene=21#wechat_redirect) 

## 三：Docker生态介绍

## [**Docker 生态概览**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486990&idx=2&sn=4b65be91263a2f8abb3c9807dfc28004&chksm=e91b6b12de6ce204d353eeedb787e803f22c9dea48a172de259d4785a9550f7360638f92677b&scene=21#wechat_redirect) 

## 四：Docker安装

[**Docker容器技术入门（一）**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247485299&idx=1&sn=8873fcee47e12af8dd92bda8cf61ef1e&chksm=e91b626fde6ceb794057913c6b61fa0184b2aa48cb2a54f1f1b32e8c4b2f201a5bff5eb5c01f&scene=21#wechat_redirect)

## 五：Docker网络与磁盘

[**【容器技术】Docker容器技术入门（二）**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247485318&idx=1&sn=f4d43728b03ef6d09af7a07d66035b03&chksm=e91b629ade6ceb8c17b666a6d1c43d8ac3602c47d113fbfd40bee9ac80ce94446a3cbb33f0d0&scene=21#wechat_redirect)

## 六：Docker命令大全

[**这20个Docker Command，有几个是你会的？**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247485839&idx=1&sn=842a3d5ef663ac07d9397d0d0cff8ebb&chksm=e91b6c93de6ce585418d974dcc6991c23df1bb24a1c1f65b66260055cbcebdd634184a848141&scene=21#wechat_redirect)

## 七：Docker容器技术之Docker file

[**Docker容器技术之Docker file**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486864&idx=1&sn=4e4a252aada8dfdb0de4e5e92de02058&chksm=e91b688cde6ce19af4f61b42454759c701bf2e250a46563948d6962be3a1883d63bd9c57beaf&scene=21#wechat_redirect)

## 八：Docker容器技术之Docker-machine

[**容器技术｜Docker三剑客之docker-machine**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486291&idx=1&sn=937f1fdb7d05838e37fd256233e45b7e&chksm=e91b6e4fde6ce759f66f2435feea0bc876fcdb7b2761096ce086465a395179ef0d86804911da&scene=21#wechat_redirect)

## 九：Docker容器技术之Docker-compose

[**容器技术｜Docker三剑客之Compose**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486005&idx=1&sn=8588567be4bb8dddd042a4d260d49a59&chksm=e91b6f29de6ce63f15d237b19492974c1a0e436bf1b35f8e5a0b87dc78896b5fd11fec84bbf7&scene=21#wechat_redirect)

## 十：Docker容器技术之Docker-swarm

[**容器技术｜Docker三剑客之docker-swarm**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247487131&idx=1&sn=ab26a01288355ca29fe16a50346ec8cd&chksm=e91b6b87de6ce291af3ad05d9842b6ad45e62d9f9d86d7474a3f38f7ba8cc4feea9817d22060&scene=21#wechat_redirect)

## 十一：Docker-register构建私有镜像仓库

[**使用docker Registry快速搭建私有镜像仓库(内附干货)**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247487199&idx=1&sn=c67f88f8a4be597054120e4750bfc4ea&chksm=e91b6bc3de6ce2d50bec65dbcd868d2e86666acb006207edbb49ddcd6bfa3a82e35b2181f34a&scene=21#wechat_redirect)

## 十二：打造高逼格Docker容器监控平台

[**打造高逼格、可视化的Docker容器监控系统平台**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247486129&idx=1&sn=986d170f115071cbe676d211a0458008&chksm=e91b6fadde6ce6bb271dedda23acef2c031ee3bdd7d9e2034ceab9a9e2c65f2caa98932e9491&scene=21#wechat_redirect)

## 十三：Jenkins与Docker的自动化CI/CD实战

[**Jenkins与Docker的自动化CI/CD实战**](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247487061&idx=1&sn=df9fd5ce9ebb784ddd280f813bb6454c&chksm=e91b6b49de6ce25f40edd7a5068cd93a90184148b589e17e3a83dcf36c4e4215dc53bdb8eff6&scene=21#wechat_redirect)

