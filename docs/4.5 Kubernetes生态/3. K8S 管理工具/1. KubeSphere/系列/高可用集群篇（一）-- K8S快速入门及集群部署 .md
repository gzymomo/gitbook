- [高可用集群篇（一）-- k8s集群部署](https://juejin.cn/post/6940887645551591455)
- [高可用集群篇（二）-- KubeSphere安装与使用](https://juejin.cn/post/6942282048107347976)
- [高可用集群篇（三）-- MySQL主从复制&ShardingSphere读写分离分库分表](https://juejin.cn/post/6944142563532079134)
- [高可用集群篇（四）-- Redis、ElasticSearch、RabbitMQ集群](https://juejin.cn/post/6945360597668069384)
- [高可用集群篇（五）-- k8s部署微服务](https://juejin.cn/post/6946396542097948702)

# 一、K8s快速入门

- [集群搭建所需的部分yaml文件以及安装包](https://github.com/free3growth/touch-air-mall/tree/main/doc)
- [gitee地址](https://gitee.com/OK12138/touch-air-mall)

## 1.1 简介

- `Kubernetes`简称k8s；用于自动化部署，扩展和管理容器后应用程序的开源系统
- [中文官网](https://kubernetes.io/zh/)
- [中文社区](https://www.kubernetes.org.cn/)
- [官方文档](https://kubernetes.io/zh/docs/home/)

### 1.1.1 Kubernetes是什么

- Kubernetes 是一个<font color='red'>可移植的、可扩展的开源平台，用于管理容器化的工作负载和服务，可促进声明式配置和自动化</font>； Kubernetes 拥有一个庞大且快速增长的生态系统。Kubernetes 的服务、支持和工具广泛可用。
- 容器因具有许多优势而变得流行起来。下面列出的是容器的一些好处：
  - 敏捷应用程序的创建和部署：与使用 VM 镜像相比，提高了容器镜像创建的简便性和效率
  - 持续开发、集成和部署：通过快速简单的回滚（由于镜像不可变性），支持可靠且频繁的 容器镜像构建和部署
  - 关注开发与运维的分离：在构建/发布时而不是在部署时创建应用程序容器镜像， 从而将应用程序与基础架构分离
  - 可观察性不仅可以显示操作系统级别的信息和指标，还可以显示应用程序的运行状况和其他指标信号
  - 跨开发、测试和生产的环境一致性：在便携式计算机上与在云中相同地运行
  - 跨云和操作系统发行版本的可移植性：可在 Ubuntu、RHEL、CoreOS、本地、 Google Kubernetes Engine 和其他任何地方运行
  - 以应用程序为中心的管理：提高抽象级别，从在虚拟硬件上运行 OS 到使用逻辑资源在 OS 上运行应用程序
  - 松散耦合、分布式、弹性、解放的微服务：应用程序被分解成较小的独立部分， 并且可以动态部署和管理 - 而不是在一台大型单机上整体运行
  - 资源隔离：可预测的应用程序性能
  - 资源利用：高效率和高密度

### 1.1.2 为什么要使用Kubernetes

容器是打包和运行应用程序的好方式。在生产环境中，你需要管理运行应用程序的容器，并确保不会停机。 例如，如果一个容器发生故障，则需要启动另一个容器。如果系统处理此行为，会不会更容易？

这就是 Kubernetes 来解决这些问题的方法！ Kubernetes 为你提供了一个<font color='red'>可弹性运行分布式系统的框架 Kubernetes 会满足你的扩展要求、故障转移、部署模式等</font>。 例如，Kubernetes 可以轻松管理系统的 Canary 部署

Kubernetes 为你提供：

- **服务发现和负载均衡**

  Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

- **存储编排**

  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等

- **自动部署和回滚**

  你可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态 更改为期望状态。例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

- **自动完成装箱计算**

  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来管理容器的资源

- **自我修复**

  Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的 运行状况检查的容器，并且在准备好服务之前不将其通告给客户端

- **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥

### 1.1.3 Kubernetes不是什么

<font color='red'>Kubernetes 不是传统的、包罗万象的 PaaS（平台即服务）系统。 由于 Kubernetes 在容器级别而不是在硬件级别运行，它提供了 PaaS 产品共有的一些普遍适用的功能， 例如部署、扩展、负载均衡、日志记录和监视</font>。

 但是，Kubernetes 不是单体系统，默认解决方案都是可选和可插拔的。 Kubernetes 提供了构建开发人员平台的基础，但是在重要的地方保留了用户的选择和灵活性

Kubernetes：

- **不限制支持的应用程序类型**。 Kubernetes 旨在支持极其多种多样的工作负载，包括无状态、有状态和数据处理工作负载。 如果应用程序可以在容器中运行，那么它应该可以在 Kubernetes 上很好地运行。
- **不部署源代码，也不构建你的应用程序**。 持续集成(CI)、交付和部署（CI/CD）工作流取决于组织的文化和偏好以及技术要求。
- **不提供应用程序级别的服务作为内置服务**，例如中间件（例如，消息中间件）、 数据处理框架（例如，Spark）、数据库（例如，mysql）、缓存、集群存储系统 （例如，Ceph）。这样的组件可以在 Kubernetes 上运行，并且/或者可以由运行在 Kubernetes 上的应用程序通过可移植机制（例如， [开放服务代理](https://openservicebrokerapi.org/)）来访问。
- **不要求日志记录、监视或警报解决方案**。 它提供了一些集成作为概念证明，并提供了收集和导出指标的机制。
- **不提供或不要求配置语言/系统（例如 jsonnet）**，它提供了声明性 API， 该声明性 API 可以由任意形式的声明性规范所构成。
- 不提供也不采用任何全面的机器配置、维护、管理或自我修复系统。
- 此外，Kubernetes 不仅仅是一个编排系统，实际上它消除了编排的需要。 编排的技术定义是执行已定义的工作流程：首先执行 A，然后执行 B，再执行 C。 相比之下，Kubernetes 包含一组独立的、可组合的控制过程， 这些过程连续地将当前状态驱动到所提供的所需状态。 如何从 A 到 C 的方式无关紧要，也不需要集中控制，这使得系统更易于使用 且功能更强大、系统更健壮、更为弹性和可扩展

### 1.1.4 Kubernetes工作示例

- 自动部署

  ![image-20210313120220816](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cb5ba66bb4d48f18e86e7a7d7bd3f5e~tplv-k3u1fbpfcp-zoom-1.image)

- 自动恢复

  ![image-20210313120248509](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa16d30889b94aca862dd6637efa6aaa~tplv-k3u1fbpfcp-zoom-1.image)

- 水平伸缩

  ![image-20210313120339866](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/086c6a6742bb4f0b9f870011290d6cb2~tplv-k3u1fbpfcp-zoom-1.image)

## 1.2 架构原理&核心概念

### 1.2.1 整体主从方式

- ![image-20210313125941792](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a7b843c951e40b39d41f93d9108c661~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210313130046227](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26cfa9e14e1a4fa08e9385b804e56179~tplv-k3u1fbpfcp-zoom-1.image)

### 1.2.2 Master节点架构

- 架构图

  ![image-20210313130252843](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96fda524446844eeb208d152cd827e4b~tplv-k3u1fbpfcp-zoom-1.image)

  - ```
    kube-apiserver
    ```

    - 对外暴露k8s的api接口，是外界进行资源操作的唯一入口
    - 提供认证、授权、访问控制、API注册和发现等机制

  - ```
    etcd
    ```

    - `etcd`是兼具一致性和高可用性的键值数据库，可以作为保存`Kubernetes`所有集群数据的后台数据库
    - `Kubernetes`集群的 `etcd`数据库通常需要有个备份计划

  - ```
    kube-scheduler
    ```

    - 主节点上的组件，该组件监视那些新创建的未指定运行节点的Pod，并选择节点让Pod在上面运行
    - 所有对k8s的集群操作，都必须经过主节点进行调度

  - ```
    kube-controller-manager
    ```

    - 在主节点上运行控制器的组件
    - 这些控制器包括：
      - 节点控制器（Node Controller）:负责在节点出现故障时进行通知和响应
      - 副本控制器（Replication Controller）:负责为系统中的每个副本控制器对象维护正确数量的Pod
      - 端点控制器（Endpoints Controller）：填充端点（Endpoints）对象（即加入Service 与 Pod）
      - 服务账户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认账户和API访问令牌

### 1.2.3 Node节点架构

- 节点组件在每个节点上运行，维护运行的Pod并提供Kubernetes运行环境

- 架构图

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f392b037341476ea723034ffc16ba9f~tplv-k3u1fbpfcp-zoom-1.image)

  - ```
    kubelet
    ```

    - 一个集群中每个节点上运行的代理；它保证容器都运行在Pod中
    - 负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）的管理

  - ```
    kube-proxy
    ```

    - 负责为Service提供cluster内部的**服务发现和负载均衡**

  - 容器运行环境（Container Runtime）

    - 容器运行环境是负责运行容器的软件
    - Kubernetes**支持多个容器运行环境**：`docker、containerd、cri-o、rktlet`以及任何实现`Kubernetes CRI`(容器运行环境接口)

  - ```
    fluentd
    ```

    - 是一个守护进程，它有助于提供集群层面日志

## 1.3 完整概念

- 整体架构图

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b05ce3a0f1414a52a8c79f98249f1bea~tplv-k3u1fbpfcp-zoom-1.image)

  - `Container`：容器，可以是docker启动的一个容器

  - `Pod`：

    - k8s使用Pod来组织一组容器
    - 一个Pod中的所有容器共享同一网络
    - Pod是k8s中的**最小部署单元**

  - `Volume`

    - 声明在Pod容器中可访问的文件目录
    - 可以被挂载在Pod中一个或多个容器指定路径下
    - 支持多种后端存储抽象（本地存储、分布式存储、云存储...）

  - `Controllers`：更高层次对象，部署和管理Pod

    - `ReplicaSet`：确保预期的Pod副本数量
    - `Deplotment`：无状态应用部署
    - `StatefulSet`：有状态应用部署
    - `DaemonSet`：确保所有Node都运行在一个指定Pod
    - `Job`：一次性任务
    - `Cronjob`：定时任务

  - `Deployment`：

    - 定义一组Pod的副本数目、版本等

    - 通过控制器（Controller）维持Pod数目（自动回复失败的Pod）

    - 通过控制器以指定的策略控制版本（滚动升级、回滚等）

      ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd5665ecd59441bd8a487dfcfc3eda5f~tplv-k3u1fbpfcp-zoom-1.image)

  - `Service`

    - 定义一组Pod的访问策略

    - Pod的负载均衡，提供一个或者多个Pod的稳定访问地址

    - 支持多种方式（`ClusterIP、NodePort、LoadBalance`）

      ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/869e4703f37e4690b99ebb3ced969c03~tplv-k3u1fbpfcp-zoom-1.image)

  - `Label`：标签，用于对象资源的查询、筛选

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ad7cd3cea0b44c593c432bcfbd83208~tplv-k3u1fbpfcp-zoom-1.image)

  - `Namespace`：命名空间，逻辑隔离

    - 一个集群内部的逻辑隔离机制（鉴权、资源）
    - 每个资源都属于一个namespace
    - 同一个namespace所有资源名不能重复
    - 不同namespace可以资源名重复

- `API`：

  我们通过`kubernetes`的API来操作整个集群

  可以通过 `kubectl`、`ui`、`curl`最终发送 `http+json/yaml`方式的请求给API server,然后控制k8s集群；**k8s里的所有资源对象都可以采用 `yaml` 或 `JSON`格式的文件定义或者描述**

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a297705164574198ba8b97d00f8780ee~tplv-k3u1fbpfcp-zoom-1.image)

## 1.4 流程叙述

- 1、通过`Kubectl`提交一个创建`RC（Replication Controller）`的请求，该请求通过`APIServer`被写入`etcd`中
- 2、此时 `Controller Manager` 通过 API Server的监听资源变化的接口监听到此RC事件
- 3、分析之后，发现当前集群中还没有它所对应的Pod实例
- 4、于是根据RC里的Pod模板定义生成一个Pod对象，通过APIServer写入etcd
- 5、此事件被Scheduler发现，它立即执行一个复杂的调度流程，为这个新Pod选定一个落户的Node，然后通过API Server将这一结果写入到`etcd`中
- 6、目标Node上运行的`Kubelet`进程通过API Server监测到这个新生的Pod，并按照它的定义，启动该Pod并任劳任怨的负责它的下半生，直到Pod生命结束
- 7、随后我们通过`Kubectl`提交一个新的映射到该Pod的Service的创建请求
- 8、`ControllerManager` 通过Label标签查询到关联的Pod实例，然后生成Service的Endpoints信息，并通过APIServer写入到`etcd`中
- 9、接下来所有Node上运行的Proxy进程通过API Server查询并监听 Service对象与其对应的Endpoints信息，建立一个软件方式的负载均衡器来实现Service访问到后端Pod的流量转发功能

> k8s里的所有资源对象都可以采用 yaml 或 JSON 格式的文件定义或描述

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d83282df6e744f3b592fac2a90d2caa~tplv-k3u1fbpfcp-zoom-1.image)



# 二、k8s集群安装

## 2.1 Kubeadm

- `kubeadm`是官方社区推出的一个用于快速部署`kubernetes`集群的工具；这个工具能通过两条指令完成一个`kubernetes`集群的部署

  [kubeadm安装文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

  - 1、创建一个Master节点

    ```bash
    $ kubeadm init
    ```
    
  - 2、将一个Node节点加入到当前集群中
  
    ```bash
    $ kubeadm join <Master节点的IP和端口>
    ```

### 2.1.1 前置要求

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，**2个CPU**或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

### 2.1.2部署步骤

- 1、在所有节点上安装 Docker 和 kubeadm

- 2、部署Kubernetes Master

- 3、部署容器网络插件

- 4、部署 kubernetes Node，将节点加入 Kuberbetes集群中

- 5、部署 Dashboard Web 页面，可视化查看Kubernetes资源


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b7bd20732a74ecd9d0fac4af17cc1e3~tplv-k3u1fbpfcp-zoom-1.image)



### 2.1.3 环境准备

#### 2.1.3.1 准备工作

- VMware 克隆两个虚拟机，准备三个虚拟机

  ```bash
  # vi /etc/sysconfig/network-scripts/ifcfg-ens33
  
  192.168.83.133
  192.168.83.134
  192.168.83.135
  ```

#### 2.1.3.2 设置linux网络环境（三个节点都执行）

- 关闭防火墙

  ```bash
  systemctl stop firewalld
  systemctl disable firewalld
  ```
  
- 关闭 selinux

  ```bash
  #设置安全策略暂不开启
  sed -i 's/enforcing/diabled/' /etc/selinux/config
  setenforce 0
  ```
  
- 关闭swap（内存交换）

  ```bash
  #临时
  swapoff -a  
  
  #永久
  sed -ri 's/.*swap.*/#&/' /etc/fstab
  验证是否生效： free -g ，swap必须为0
  ```
  
- 添加主机名与ip对应关系

  ```bash
  #设置对应的hostname
  hostnamectl set-hostname k8s-node1
  
  
  vim /etc/hosts
  
  192.168.83.133  k8s-node1
  192.168.83.134  k8s-node2
  192.168.83.135  k8s-node3
  ```
  
  确保在每个虚拟机中，ping各自的节点名称都能ping通
  
  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f47368f3dce41ac96615fc834aa5231~tplv-k3u1fbpfcp-zoom-1.image)
  
  
  
  
  
  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf0f245986404ef5a6ccf82d9af2ea1c~tplv-k3u1fbpfcp-zoom-1.image)
  
- 将桥接的ipv4流量传递到iptables的链

  ```bash
  cat > /etc/sysctl.d/k8s.conf << EOF   
  net.bridge.bridge-nf-call-ip6tables=1
  net.bridge.bridge-nf-call-iptables=1
  EOF
  
  sysctl --system
  ```


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/049db541a2654c49b47f40fad550a406~tplv-k3u1fbpfcp-zoom-1.image)



#### 2.1.3.3 所有节点安装

- `docker、kubeadm、kubelet、kubectl`

  Kubernetes 默认 CRI（容器运行时）为Docker ，因此先按照Docker

- 添加阿里云yum源

  ```bash
  cat > /etc/yum.repos.d/kubernetes.repo << EOF
  [kubernetes]
  name=Kubernetes
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
  enabled=1
  gpgcheck=0
  repo_gpgcheck=0
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
  EOF
  ```
  
- 指定版本安装

  ```bash
  yum list|grep kube
  #基于1.1.7版本的kubernetes
  yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3
  
  #安装完成执行
  systemctl enable kubelet
  systemctl start kubelet
  ```

### 2.1.4 部署 k8s-master

#### 2.1.4.1 master节点初始化

- 将k8s资源文件拷贝进k8s-node1

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b0a702f77644e90809588e85658d98a~tplv-k3u1fbpfcp-zoom-1.image)

  

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8359731f97c471f984e926891603674~tplv-k3u1fbpfcp-zoom-1.image)

  

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0493945c619b409a9477124943eea023~tplv-k3u1fbpfcp-zoom-1.image)

- 初始化master节点

  ```bash
  kubeadm init \
  --apiserver-advertise-address=192.168.83.133 \
  --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
  --kubernetes-version v1.17.3 \
  --service-cidr=10.96.0.0/16 \
  --pod-network-cidr=10.244.0.0/16
  ```
  

执行结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b48b3982955d4e508484010810dc43a0~tplv-k3u1fbpfcp-zoom-1.image)



继续按照控制台提示执行提示命令

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef2ff9ded3974182a966bb84c2366c39~tplv-k3u1fbpfcp-zoom-1.image)

  

  

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ca0267127f84603ac7bc9a36fc7d0b5~tplv-k3u1fbpfcp-zoom-1.image)

  ```bash
  #启动完成，这句话拷贝出来备用（*），两小时内有效
  kubeadm join 192.168.83.133:6443 --token f9s477.9qh5bg4gd7xy9f67 \
      --discovery-token-ca-cert-hash sha256:8d6007a15b9dfa0940d2ca8fbf2929a108c391541ae59f9c75d66352bdd0aba6
  ```

### 2.1.5 安装Pod网络插件

- 利用k8s文件夹中的  `kube-flannel.yml`

  ```bash
  kubectl apply -f kube-flannel.yml
  ```
  
  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cab7800df4594c298e8d266b0757c6ac~tplv-k3u1fbpfcp-zoom-1.image)
  
  ```bash
  kubectl get pods --all-namespaces
  ```


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc13e435f2c34f2cba3ca6fbdd5521c6~tplv-k3u1fbpfcp-zoom-1.image)



### 2.1.6 加入k8s集群

- 查看master状态

  ```bash
  kubectl get nodes
  ```
  
  ![image-20210313182140812](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0476e705bf5e4cf293d5ebb3d80b1c5c~tplv-k3u1fbpfcp-zoom-1.image)
  
- node2和node3节点加入集群

  ```bash
  #master启动时，日志打印出的内容
  kubeadm join 192.168.83.133:6443 --token f9s477.9qh5bg4gd7xy9f67 \
      --discovery-token-ca-cert-hash sha256:8d6007a15b9dfa0940d2ca8fbf2929a108c391541ae59f9c75d66352bdd0aba6
  ```
  
  - ![image-20210313182434819](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d65cf77db824142bdad23eadf02f246~tplv-k3u1fbpfcp-zoom-1.image)
  
    ![image-20210313182520080](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5518ab03214e4f149884c0bd1f2eb24d~tplv-k3u1fbpfcp-zoom-1.image)
  
  ![image-20210313182544620](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1794ec319b64555af08fea4bd81c649~tplv-k3u1fbpfcp-zoom-1.image)
  
  
  
- 如果存在NotReady状态，网络问题，可以执行以下命令，监控pod进度

  ```bash
  kubectl get pod -n kube-system -o wide
  ```
  
![image-20210313183028711](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/731c1b19a1a6459892d4dbdf8a4256f6~tplv-k3u1fbpfcp-zoom-1.image)
  
等待3-10分钟，完全都是running以后，再次查看
  
![image-20210313182811099](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c684064a723d46f19ae820e611641ae9~tplv-k3u1fbpfcp-zoom-1.image)


# 三、入门操作Kubernetes集群

## 3.1 基本操作体验

### 3.1.1 部署一个tomcat

```bash
#主节点 运行创建一个deployment部署
kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 

#获取到tomcat信息
kubectl get pods -o wide
```

- 查看节点 和 pod 信息

  - ![image-20210314105950139](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1ab5399a3514c0caf4f45652af7d0f9~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210314110018197](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cabe87659fa54d10a693381832f5dfdc~tplv-k3u1fbpfcp-zoom-1.image)
  

#### 3.1.1.1 模拟tomcat容器宕机（手动stop 容器）

- 观察是否会重新拉起tomcat容器

  - ![image-20210314110507427](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/826bb8d9f9b04c70acfcc90c820ff639~tplv-k3u1fbpfcp-zoom-1.image)
  

#### 3.1.1.2 模拟node3 节点宕机（关闭虚拟机）

- 检测过程可能会比较慢，大约5分钟左右，耐心等待

  - ![image-20210314111538865](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd6d7913ec0d425db262790fcce11a8c~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210314111611425](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e5a294eba734ced8ade71ec4143cd7d~tplv-k3u1fbpfcp-zoom-1.image)
  

> 简单的容灾恢复测试

## 3.2 暴露访问

- Pod的80映射容器的8080；service会代理Pod的80

  ```bash
  kubectl expose deployment tomcat6 --port=80 --target-port=8080 --type=NodePort
  ```
  
- 获取service

  ```bash
  kubectl get svc
  ```
  
  - ![image-20210314113547955](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ccd00d629b64700b2c5c8028197d9fa~tplv-k3u1fbpfcp-zoom-1.image)
  
    ![image-20210314113606731](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/899ecfa3da0243e4b6fc3b0092bdd3ea~tplv-k3u1fbpfcp-zoom-1.image)
  

## 3.3 动态扩容测试

- 获取部署信息

  ```bash
  kubectl get deployment
  ```
  

![image-20210314114606206](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ce92b72d56549fd9e0568a55236f1c8~tplv-k3u1fbpfcp-zoom-1.image)

- 扩容

  ```bash
  kubectl scale --replicas=3 deployment tomcat6
  ```
  

扩容了多份，所以无论访问那个node的指定端口，都可以访问到tomcat6

![image-20210314114139240](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b350f09d98b04587bda9a2107ab96cfb~tplv-k3u1fbpfcp-zoom-1.image)

![image-20210314114210487](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c7ec316dd8a452eb68005036548e54d~tplv-k3u1fbpfcp-zoom-1.image)

- 缩容（改变副本数量）

  ```bash
  kubectl scale --replicas=1 deployment tomcat6
  ```
  
  - ![image-20210314114510021](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5129b5fef7a4402a7b64d17a0794ae3~tplv-k3u1fbpfcp-zoom-1.image) 
  

## 3.4 删除

- 获取到所有的资源

  ```bash
  kubectl get all
  ```
  

![image-20210314114841781](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f474bdf901b84f4e92c1e2dba633e43c~tplv-k3u1fbpfcp-zoom-1.image)

- 删除部署

  ```bash
  kubectl delete deployment.apps/tomcat6
  ```
  
- 删除service

  ```bash
  kubectl delete service/tomcat6
  ```
  
- ![image-20210314115046881](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd3d3ed146b443ecb29ce1d1a77042de~tplv-k3u1fbpfcp-zoom-1.image)
  

# 四、k8s细节

## 4.1 kubectl

### 4.1.1 kubectl文档

- Kubectl是一个命令行接口，用于对Kubernetes集群进行命令

  [kubectl命令行界面](https://kubernetes.io/zh/docs/reference/kubectl/overview/)

### 4.1.2 资源类型

- [ 资源类型](https://kubernetes.io/zh/docs/reference/kubectl/overview/#资源类型)

  ![image-20210314120041363](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45bf821bc6df411e93647bcabe44b958~tplv-k3u1fbpfcp-zoom-1.image)

### 4.1.3 格式化输出

- 所有 `kubectl` 命令的默认输出格式都是人类可读的纯文本格式。要以特定格式向终端窗口输出详细信息，可以将 `-o` 或 `--output` 参数添加到受支持的 `kubectl` 命令中

  [格式化输出](https://kubernetes.io/zh/docs/reference/kubectl/overview/#格式化输出)

  ![image-20210314120259917](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70a82731ca704a609c8abe3f9d16f29a~tplv-k3u1fbpfcp-zoom-1.image)


### 4.1.4 常用操作

- 常用操作文档地址
  - `kubectl apply` - 以文件或标准输入为准应用或更新资源
  - `kubectl get` - 列出一个或多个资源
  - `kubectl describe` - 显示一个或多个资源的详细状态，默认情况下包括未初始化的资源
  - `kubectl delete` - 从文件、stdin 或指定标签选择器、名称、资源选择器或资源中删除资源
  - `kubectl exec` - 对 pod 中的容器执行命令
  - `kubectl logs` - 打印 Pod 中容器的日志

### 4.1.5 命令参考

- [kubectl命令参考](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)

# 五、yaml语法

## 5.1 yml模板

- 图解

  ![image-20210314120907601](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c71abb982f6a4cffac6be2dd737ff21c~tplv-k3u1fbpfcp-zoom-1.image)

- 查看上面的tomcat6创建命令对应的yaml内容

  ```bash
  kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 --dry-run -o yaml
  ```
  

![image-20210314121710836](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7cd20b7632c04defb6868d3dd38d2b81~tplv-k3u1fbpfcp-zoom-1.image)

- 生成yaml文件

  ```bash
  kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 --dry-run -o yaml > tomcat6.yaml
  ```
  

修改文件中的，副本1 改为3 `vim tomact6.yaml`

运行yaml文件

```bash
  kubectl apply -f tomcat6.yaml
```

  ![image-20210314122341809](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1adae322dee4ec88b7e288871cfca4e~tplv-k3u1fbpfcp-zoom-1.image)

- 端口暴露也可以使用yaml文件代替冗长命令

  ```shell
  kubectl expose deployment tomcat6 --port=80 --target-port=8080 --type=NodePort --dry-run -o yaml
  ```
  
- ![image-20210314122931059](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00211c386d41403a8be3944d68ff74b8~tplv-k3u1fbpfcp-zoom-1.image)
  

# 六、入门操作

## 6.1 Pod

- `Pod` 是 Kubernetes 应用程序的基本执行单元，即它是 Kubernetes 对象模型中创建或部署的最小和最简单的单元。Pod 表示在 [集群](https://v1-17.docs.kubernetes.io/zh/docs/reference/glossary/?all=true#term-cluster) 上运行的进程。

  Pod 封装了应用程序容器（或者在某些情况下封装多个容器）、存储资源、唯一网络 IP 以及控制容器应该如何运行的选项。 Pod 表示部署单元：*Kubernetes 中应用程序的单个实例*，它可能由单个 [容器](https://v1-17.docs.kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-containers) 或少量紧密耦合并共享资源的容器组成。

  [Docker](https://www.docker.com/) 是 Kubernetes Pod 中最常用的容器运行时，但 Pod 也能支持其他的[容器运行时](https://v1-17.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/)。

  Kubernetes 集群中的 Pod 可被用于以下两个主要用途：

  - **运行单个容器的 Pod**。”每个 Pod 一个容器”模型是最常见的 Kubernetes 用例；在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。
  - **运行多个协同工作的容器的 Pod**。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众，而另一个单独的“挂斗”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。 [Kubernetes 博客](https://kubernetes.io/blog) 上有一些其他的 Pod 用例信息。更多信息请参考：
  - [分布式系统工具包：容器组合的模式](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
  - [容器设计模式](https://kubernetes.io/blog/2016/06/container-design-patterns)

  每个 Pod 表示运行给定应用程序的单个实例。如果希望横向扩展应用程序（例如，运行多个实例），则应该使用多个 Pod，每个应用实例使用一个 Pod 。在 Kubernetes 中，这通常被称为 *副本*。通常使用一个称为控制器的抽象来创建和管理一组副本 Pod

## 6.2 控制器

- [控制器](https://v1-17.docs.kubernetes.io/zh/docs/concepts/workloads/controllers/)

  ![image-20210314131434293](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57b2e2214a9b401b83c4880fa332a0d3~tplv-k3u1fbpfcp-zoom-1.image)

- 一个 `Deployment` 控制器为 [Pods](https://v1-17.docs.kubernetes.io/docs/concepts/workloads/pods/pod/)和 [ReplicaSets](https://v1-17.docs.kubernetes.io/docs/concepts/workloads/controllers/replicaset/)提供描述性的更新方式

## 6.3 Deployment&Service

- 二者关系

  ![image-20210314131854578](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3def78789da48189263fed0d065fb88~tplv-k3u1fbpfcp-zoom-1.image)

- 简单来说

  - Deployment 是master节点保存的一个部署信息
  - Service暴露Pod，对Pod的一种负载均衡

- Service的意义：统一应用访问入口；管理一组Pod，防止Pod失联（服务发现）；定义一组Pod的访问策略

  现在`Service`我们使用`NodePort`的方式暴露，这样访问每个节点的端口，都可以访问到这个Pod，如果节点宕机，就会出现问题

## 6.4 labels和selectors

- 标签与选择器（类比javascript中控件的id、class与选择器的关系）

- 关系图解

  - ![image-20210314135750519](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30e30cc75db6427fa1c954ce01cd2e06~tplv-k3u1fbpfcp-zoom-1.image)
  

## 6.5 Ingress

- 基于nginx

- 利用yaml方式启动之前的tomcat6容器，并暴露访问端口

  `tomcat6-deployment.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: tomcat6
    name: tomcat6
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: tomcat6
    template:
      metadata:
        labels:
          app: tomcat6
      spec:
        containers:
        - image: tomcat:6.0.53-jre8
          name: tomcat
  ---
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: tomcat6
    name: tomcat6
  spec:
    ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
    selector:
      app: tomcat6
    type: NodePort
  ```
  
- 运行yaml文件
  
  ```bash
    kubectl apply -f tomcat6-deployment.yaml
  ```
  
  ![image-20210314141341694](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b803b0f97264b00a350787628c6d67c~tplv-k3u1fbpfcp-zoom-1.image)
  
  - 任意三个节点地址的30658端口，都可以访问到tomcat

- 通过Service发现Pod进行关联，基于**域名访问**；通过Ingress Controller实现Pod**负载均衡**‘；支持TCP/UDP 4层负载均衡和HTTP 7层负载均衡

  ![image-20210314141843873](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ff97e375cc44f9392ff7080d2bae7c5~tplv-k3u1fbpfcp-zoom-1.image)

  - 步骤1：运行文件（在k8s文件夹中，该文件已准备好），创建`Ingress Controller`

    ```bash
    kubectl apply -f  ingress-controller.yaml
    ```
    
    ![image-20210314142549125](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9a06e443a814809b05d51946a0600ff~tplv-k3u1fbpfcp-zoom-1.image)
    
    ![image-20210314142747487](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c0c39391fea4e08bde559502918fd2b~tplv-k3u1fbpfcp-zoom-1.image)
    
  - 步骤2：创建 `Ingress` 规则
  
    ```yaml
    #创建ingress-tomcat6.yaml文件
    apiVersion: extensions/v1beta1
    kind: Ingress 
    metadata: 
      name: web 
    spec:
      rules:
      - host: tomcat6.touch.air.mall.com
        http:
          paths:
            - backend:
               serviceName: tomcat6
               servicePort: 80
    
    ```
  
    ```bash
    #应用ingress规则
    kubectl apply -f ingress-tomcat6.yaml
    #查看信息
    kubectl get all
    ```
    
    ![image-20210314144047165](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05195bdd31704e07a858dbfb3a6afd19~tplv-k3u1fbpfcp-zoom-1.image)
    
  - 配置本地域名解析 `C:\Windows\System32\drivers\etc\hosts`

    ![image-20210314144438834](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c103dc797842a3b92566fe26ec8ae5~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210314145103220](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9010f99d94d4b9d999f96fd79ac0258~tplv-k3u1fbpfcp-zoom-1.image)
