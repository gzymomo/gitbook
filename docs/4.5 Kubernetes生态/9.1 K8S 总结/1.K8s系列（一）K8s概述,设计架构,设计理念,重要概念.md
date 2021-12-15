相关参考文章原文链接：

- [毛江云，腾讯 CSIG应用开发工程师](https://tinyurl.com/ya3ennxf)



# 一、Kubernetes概述

## 1.1 K8s是什么？

K8S 是Kubernetes的全称，官方称其是：

> Kubernetes is an open source system for managing containerized applications across multiple hosts. It provides basic mechanisms for deployment,  maintenance, and scaling of applications.
>
> 用于自动部署、扩展和管理“容器化（containerized）应用程序”的开源系统。

翻译成大白话就是：“**K8S 是 负责自动化运维管理多个 Docker 程序的集群**”。



[**Kubernetes**](https://www.kubernetes.org.cn/)是一个开源的，用于管理云平台中多个主机上的容器化的应用，Kubernetes的目标是让部署容器化的应用简单并且高效（powerful）,Kubernetes提供了应用部署，规划，更新，维护的一种机制。简单点说就是，kubernetes是一种开源的容器编排工具，通过调度系统维持用户预期数量和状态的容器正常运行。



## 1.2 K8s能干什么？

K8S 要做的事情：自动化运维管理 Docker（容器化）程序。



<font color='blue'>Kubernetes是一个可移植的、可扩展的、用于管理容器化工作负载和服务的开源平台，它简化（促进）了声明式配置和自动化。</font>



Kubernetes一个核心的特点就是能够自主的管理容器来保证云平台中的容器按照用户的期望状态运行着（比如用户想让apache一直运行，用户不需要关心怎么去做，Kubernetes会自动去监控，然后去重启，新建，总之，让apache一直提供服务），管理员可以加载一个微型服务，让规划器来找到合适的位置，同时，Kubernetes也系统提升工具以及人性化方面，让用户能够方便的部署自己的应用。



现在Kubernetes着重于不间断的服务状态（比如web服务器或者缓存服务器）和原生云平台应用（Nosql）,在不久的将来会支持各种生产云平台中的各种服务，例如，分批，工作流，以及传统数据库。



Kubernetes提供的功能：

- 服务发现和负载均衡：Kubernetes 可以使用 DNS 名称或自己的 IP 地址公开容器，如果到容器的流量很大，Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。
- 存储编排：Kubernetes 允许您自动挂载您选择的存储系统，例如本地存储、公共云提供商等

- 自动部署和回滚：您可以使用 Kubernetes 描述已部署容器的所需状态，它可以以受控的速率将实际状态更改为所需状态。例如，您可以自动化 Kubernetes 来为您的部署创建新容器，删除现有容器并将它们的所有资源用于新容器。

- 自我修复：Kubernetes 重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器，并且在准备好服务之前不将其通告给客户端。

- 密钥与配置管理：Kubernetes 允许您存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。您可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。



## 1.3 K8s怎么做?

K8S 的核心功能：自动化运维管理多个容器化程序。

![image-20210202104400297](http://lovebetterworld.com/image-20210202104400297.png)

K8S 是属于**主从设备模型（Master-Slave 架构）**，即有 Master 节点负责核心的调度、管理和运维，Slave 节点则在执行用户的程序。但是在 K8S 中，主节点一般被称为**Master Node 或者 Head Node**，而从节点则被称为**Worker Node 或者 Node**。

要注意一点：<font color='red'>Master Node 和 Worker Node 是分别安装了 K8S 的 Master 和 Woker 组件的实体服务器，每个 Node  都对应了一台实体服务器（虽然 Master Node 可以和其中一个 Worker Node 安装在同一台服务器，但是建议 Master  Node 单独部署），**所有 Master Node 和 Worker Node 组成了 K8S 集群**，同一个集群可能存在多个 Master Node 和 Worker Node。</font>



首先来看**Master Node**都有哪些组件：

- **API Server**。**K8S 的请求入口服务**。API Server 负责接收 K8S 所有请求（来自 UI 界面或者 CLI 命令行工具），然后，API Server 根据用户的具体请求，去通知其他组件干活。
- **Scheduler**。**K8S 所有 Worker Node 的调度器**。当用户要部署服务时，Scheduler 会选择最合适的 Worker Node（服务器）来部署。
- **Controller Manager**。**K8S 所有 Worker Node 的监控器**。Controller Manager 有很多具体的 Controller，在文章Components of Kubernetes Architecture中提到的有 Node Controller、Service Controller、Volume Controller 等。Controller  负责监控和调整在 Worker Node 上部署的服务的状态，比如用户要求 A 服务部署 2  个副本，那么当其中一个服务挂了的时候，Controller 会马上调整，让 Scheduler 再选择一个 Worker Node  重新部署服务。
- **etcd**。**K8S 的存储服务**。etcd 存储了 K8S 的关键配置和用户配置，K8S 中仅 API Server 才具备读写权限，其他组件必须通过 API Server 的接口才能读写数据。

接着来看**Worker Node**的组件：

- **Kubelet**。**Worker Node 的监视器，以及与 Master Node 的通讯器**。Kubelet 是 Master Node 安插在 Worker Node 上的“眼线”，它会定期向 Worker Node 汇报自己 Node 上运行的服务的状态，并接受来自 Master Node 的指示采取调整措施。
- **Kube-Proxy**。**K8S 的网络代理**。私以为称呼为 Network-Proxy 可能更适合？Kube-Proxy 负责 Node 在 K8S 的网络通讯、以及对外部网络流量的负载均衡。
- **Container Runtime**。**Worker Node 的运行环境**。即安装了容器化所需的软件环境确保容器化程序能够跑起来，比如 Docker Engine。大白话就是帮忙装好了 Docker 运行环境。
- **Logging Layer**。**K8S 的监控状态收集器**。私以为称呼为 Monitor 可能更合适？Logging Layer 负责采集 Node 上所有服务的 CPU、内存、磁盘、网络等监控项信息。
- **Add-Ons**。**K8S 管理运维 Worker Node 的插件组件**。有些文章认为 Worker Node 只有三大组件，不包含 Add-On，但笔者认为 K8S 系统提供了 Add-On 机制，让用户可以扩展更多定制化功能，是很不错的亮点。

总结来看，**K8S 的 Master Node 具备：请求入口管理（API Server），Worker Node  调度（Scheduler），监控和自动调节（Controller Manager），以及存储功能（etcd）；而 K8S 的 Worker  Node 具备：状态和监控收集（Kubelet），网络和负载均衡（Kube-Proxy）、保障容器化运行环境（Container  Runtime）、以及定制化功能（Add-Ons）。**



# 二、Kubernetes设计架构

![image-20210202110209533](http://lovebetterworld.com/image-20210202110209533.png)

![image-20210202110250028](http://lovebetterworld.com/image-20210202110250028.png)



Kubernetes主要由以下几个核心组件组成：

- etcd：保存了整个集群的状态；
- apiserver：提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet：负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
- Container runtime：负责镜像管理以及Pod和容器的真正运行（CRI）；
- kube-proxy：负责为Service提供cluster内部的服务发现和负载均衡；

除了核心组件，还有一些推荐的Add-ons：

- kube-dns：负责为整个集群提供DNS服务
- Ingress Controller：为服务提供外网入口
- Heapster：提供资源监控
- Dashboard：提供GUI
- Federation：提供跨可用区的集群
- Fluentd-elasticsearch：提供集群日志采集、存储与查询



### Master组件说明

Master组件说明：

- API Server：集群统一入口，以Restful方式，交给etcd存储。
- Scheduler：节点调度，选择node节点部署应用
- Controller-Manage：处理集群中的常规后台任务，一个资源对应一个控制器。
- etcd：存储系统，用于保存集群相关的数据。



![image-20210202104124192](http://lovebetterworld.com/image-20210202104124192.png)



### Node组件说明

- kubelet：master派到node节点代表，管理本机容器

- kube-proxy：提供网络代理，负载均衡等操作。

- **Container Runtime**

  　　容器运行时是负责运行容器的软件。

    　　Kubernetes支持多个容器运行时：Docker、containerd、crio、rktlet和Kubernetes CRI(容器运行时接口)的任何实现。 



## 分层架构

Kubernetes设计理念和功能其实就是一个类似Linux的分层架构，如下图所示

![image-20210202104104918](http://lovebetterworld.com/image-20210202104104918.png)

- 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层：kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
  - Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等
  - Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等



# 三、Kubernetes设计理念

K8s系统最核心的两个设计理念：一个是**容错性**，一个是**易扩展性**。容错性实际是保证K8s系统稳定性和安全性的基础，易扩展性是保证K8s对变更友好，可以快速迭代增加新功能的基础。



## 3.1 API设计原则

对于云计算系统，系统API实际上处于系统设计的统领地位，正如本文前面所说，K8s集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的API对象，支持对该功能的管理操作，理解掌握的API，就好比抓住了K8s系统的牛鼻子。K8s系统API的设计有以下几条原则：

1. **所有API应该是声明式的**。正如前文所说，声明式的操作，相对于命令式操作，对于重复操作的效果是稳定的，这对于容易出现数据丢失或重复的分布式环境来说是很重要的。另外，声明式操作更容易被用户使用，可以使系统向用户隐藏实现的细节，隐藏实现的细节的同时，也就保留了系统未来持续优化的可能性。此外，声明式的API，同时隐含了所有的API对象都是名词性质的，例如Service、Volume这些API都是名词，这些名词描述了用户所期望得到的一个目标分布式对象。
2. **API对象是彼此互补而且可组合的**。这里面实际是鼓励API对象尽量实现面向对象设计时的要求，即“高内聚，松耦合”，对业务相关的概念有一个合适的分解，提高分解出来的对象的可重用性。事实上，K8s这种分布式系统管理平台，也是一种业务系统，只不过它的业务就是调度和管理容器服务。
3. **高层API以操作意图为基础设计**。如何能够设计好API，跟如何能用面向对象的方法设计好应用系统有相通的地方，高层设计一定是从业务出发，而不是过早的从技术实现出发。因此，针对K8s的高层API设计，一定是以K8s的业务为基础出发，也就是以系统调度管理容器的操作意图为基础设计。
4. **低层API根据高层API的控制需要设计**。设计实现低层API的目的，是为了被高层API使用，考虑减少冗余、提高重用性的目的，低层API的设计也要以需求为基础，要尽量抵抗受技术实现影响的诱惑。
5. **尽量避免简单封装，不要有在外部API无法显式知道的内部隐藏的机制**。简单的封装，实际没有提供新的功能，反而增加了对所封装API的依赖性。内部隐藏的机制也是非常不利于系统维护的设计方式，例如PetSet和ReplicaSet，本来就是两种Pod集合，那么K8s就用不同API对象来定义它们，而不会说只用同一个ReplicaSet，内部通过特殊的算法再来区分这个ReplicaSet是有状态的还是无状态。
6. **API操作复杂度与对象数量成正比**。这一条主要是从系统性能角度考虑，要保证整个系统随着系统规模的扩大，性能不会迅速变慢到无法使用，那么最低的限定就是API的操作复杂度不能超过O(N)，N是对象的数量，否则系统就不具备水平伸缩性了。
7. **API对象状态不能依赖于网络连接状态**。由于众所周知，在分布式环境下，网络连接断开是经常发生的事情，因此要保证API对象状态能应对网络的不稳定，API对象的状态就不能依赖于网络连接状态。
8. **尽量避免让操作机制依赖于全局状态，因为在分布式系统中要保证全局状态的同步是非常困难的**。

## 3.2 控制机制设计原则

- **控制逻辑应该只依赖于当前状态**。这是为了保证分布式系统的稳定可靠，对于经常出现局部错误的分布式系统，如果控制逻辑只依赖当前状态，那么就非常容易将一个暂时出现故障的系统恢复到正常状态，因为你只要将该系统重置到某个稳定状态，就可以自信的知道系统的所有控制逻辑会开始按照正常方式运行。
- **假设任何错误的可能，并做容错处理**。在一个分布式系统中出现局部和临时错误是大概率事件。错误可能来自于物理系统故障，外部系统故障也可能来自于系统自身的代码错误，依靠自己实现的代码不会出错来保证系统稳定其实也是难以实现的，因此要设计对任何可能错误的容错处理。
- **尽量避免复杂状态机，控制逻辑不要依赖无法监控的内部状态**。因为分布式系统各个子系统都是不能严格通过程序内部保持同步的，所以如果两个子系统的控制逻辑如果互相有影响，那么子系统就一定要能互相访问到影响控制逻辑的状态，否则，就等同于系统里存在不确定的控制逻辑。
- **假设任何操作都可能被任何操作对象拒绝，甚至被错误解析**。由于分布式系统的复杂性以及各子系统的相对独立性，不同子系统经常来自不同的开发团队，所以不能奢望任何操作被另一个子系统以正确的方式处理，要保证出现错误的时候，操作级别的错误不会影响到系统稳定性。
- **每个模块都可以在出错后自动恢复**。由于分布式系统中无法保证系统各个模块是始终连接的，因此每个模块要有自我修复的能力，保证不会因为连接不到其他模块而自我崩溃。
- **每个模块都可以在必要时优雅地降级服务**。所谓优雅地降级服务，是对系统鲁棒性的要求，即要求在设计实现模块时划分清楚基本功能和高级功能，保证基本功能不会依赖高级功能，这样同时就保证了不会因为高级功能出现故障而导致整个模块崩溃。根据这种理念实现的系统，也更容易快速地增加新的高级功能，以为不必担心引入高级功能影响原有的基本功能。



# 四、Kubernetes重要概念

## 4.1 Pod实例

官方对于**Pod**的解释是：

> **Pod**是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

**Pod 就是 K8S 中一个服务的闭包**。

简单来说，**Pod 可以被理解成一群可以共享网络、存储和计算资源的容器化服务的集合**。再打个形象的比喻，在同一个 Pod 里的几个 Docker 服务/程序，好像被部署在同一台机器上，可以通过 localhost 互相访问，并且可以共用 Pod  里的存储资源（这里是指 Docker 可以挂载 Pod 内的数据卷，数据卷的概念”）。



**同一个 Pod 之间的 Container 可以通过 localhost 互相访问，并且可以挂载 Pod 内所有的数据卷；但是不同的 Pod 之间的 Container 不能用 localhost 访问，也不能挂载其他 Pod 的数据卷**。

![image-20210202104053613](http://lovebetterworld.com/image-20210202104053613.png)



K8S 中所有的对象都通过 yaml 来表示，从官方网站摘录了一个最简单的 Pod 的 yaml：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

- `apiVersion`记录 K8S 的 API Server 版本，现在看到的都是`v1`，用户不用管。

- `kind`记录该 yaml 的对象，比如这是一份 Pod 的 yaml 配置文件，那么值内容就是`Pod`。

- `metadata`记录了 Pod 自身的元数据，比如这个 Pod 的名字、这个 Pod 属于哪个 namespace（“同一个命名空间内的对象互相可见”）。

- `spec`记录了 Pod 内部所有的资源的详细信息：

- - `containers`记录了 Pod 内的容器信息，`containers`包括了：`name`容器名，`image`容器的镜像地址，`resources`容器需要的 CPU、内存、GPU 等资源，`command`容器的入口命令，`args`容器的入口参数，`volumeMounts`容器要挂载的 Pod 数据卷等。可以看到，**上述这些信息都是启动容器的必要和必需的信息**。
  - `volumes`记录了 Pod 内的数据卷信息。

  

## 4.2 Volume 数据卷

  K8S 支持很多类型的 volume 数据卷挂载，具体请参见K8S 卷。

**数据卷 volume 是 Pod 内部的磁盘资源**。

**volume 是 K8S 的对象，对应一个实体的数据卷；而 volumeMounts 只是 container 的挂载点，对应 container 的其中一个参数**。但是，**volumeMounts 依赖于 volume**，只有当 Pod 内有 volume 资源的时候，该 Pod 内部的 container 才可能有 volumeMounts。



## 4.3 Container 容器

**一个 Pod 内可以有多个容器 container**。

在 Pod 中，容器也有分类：

- **标准容器 Application Container**。
- **初始化容器 Init Container**。
- **边车容器 Sidecar Container**。
- **临时容器 Ephemeral Container**。

一般来说，我们部署的大多是**标准容器（ Application Container）**。



## 4.4 Deployment 和 ReplicaSet（简称 RS）

什么是 Deployment 呢？官方给出了一个要命的解释：

> 一个 *Deployment* 控制器为 Pods 和 ReplicaSets 提供声明式的更新能力。
>
> 你负责描述 Deployment 中的 *目标状态*，而 Deployment 控制器以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment，并通过新的 Deployment 收养其资源。



**Deployment 的作用是管理和控制 Pod 和 ReplicaSet，管控它们运行在用户期望的状态中**。

打个形象的比喻，**Deployment 就是包工头**，主要负责监督底下的工人 Pod 干活，确保每时每刻有用户要求数量的 Pod 在工作。如果一旦发现某个工人 Pod 不行了，就赶紧新拉一个 Pod 过来替换它。



那什么是 ReplicaSets 呢？

> ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。



ReplicaSet 的作用就是管理和控制 Pod，管控他们好好干活。但是，ReplicaSet 受控于 Deployment。形象来说，**ReplicaSet 就是总包工头手下的小包工头**。



在 K8S 中还有一个对象 --- **ReplicationController（简称 RC）**，官方文档对它的定义是：

> *ReplicationController* 确保在任何时候都有特定数量的 Pod 副本处于运行状态。换句话说，ReplicationController 确保一个 Pod 或一组同类的 Pod 总是可用的。

在Deployments, ReplicaSets, and  pods教程中说“ReplicationController 是 ReplicaSet 的前身”，官方也推荐用 Deployment 取代  ReplicationController 来部署服务。



## 4.5 Service 和 Ingress

**Service 和 Ingress 则负责管控 Pod 网络服务**。

官方文档中 Service 的定义：

> 将运行在一组 Pods 上的应用程序公开为网络服务的抽象方法。
>
> 使用 Kubernetes，您无需修改应用程序即可使用不熟悉的服务发现机制。Kubernetes 为 Pods 提供自己的 IP 地址，并为一组 Pod 提供相同的 DNS 名， 并且可以在它们之间进行负载均衡。



<font color='red'>K8S 中的服务（Service）并不是我们常说的“服务”的含义，而更像是网关层，是若干个 Pod 的流量入口、流量均衡器。</font>



**为什么要 Service 呢**？

官方文档讲解地非常清楚：

> Kubernetes Pod 是有生命周期的。它们可以被创建，而且销毁之后不会再启动。如果您使用 Deployment 来运行您的应用程序，则它可以动态创建和销毁 Pod。
>
> 每个 Pod 都有自己的 IP 地址，但是在 Deployment 中，在同一时刻运行的 Pod 集合可能与稍后运行该应用程序的 Pod 集合不同。
>
> 这导致了一个问题：如果一组 Pod（称为“后端”）为群集内的其他 Pod（称为“前端”）提供功能， 那么前端如何找出并跟踪要连接的 IP 地址，以便前端可以使用工作量的后端部分？

**Service 是 K8S 服务的核心，屏蔽了服务细节，统一对外暴露服务接口，真正做到了“微服务”**。举个例子，我们的一个服务 A，部署了 3 个备份，也就是 3 个 Pod；对于用户来说，只需要关注一个 Service 的入口就可以，而不需要操心究竟应该请求哪一个 Pod。优势非常明显：**一方面外部用户不需要感知因为 Pod 上服务的意外崩溃、K8S 重新拉起 Pod 而造成的 IP 变更，外部用户也不需要感知因升级、变更服务带来的 Pod 替换而造成的 IP 变化，另一方面，Service 还可以做流量负载均衡**。



Service 主要负责 K8S 集群内部的网络拓扑。那么集群外部怎么访问集群内部呢？这个时候就需要 Ingress 了，官方文档中的解释是：

> Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。
>
> Ingress 可以提供负载均衡、SSL 终结和基于名称的虚拟托管。

Ingress 是整个 K8S 集群的接入层，复杂集群内外通讯。



## 4.6 namespace 命名空间

 namespace 是为了服务整个 K8S 集群的。

namespace 是什么呢？

官方文档定义：

> Kubernetes 支持多个虚拟集群，它们底层依赖于同一个物理集群。这些虚拟集群被称为名字空间。

**namespace 是为了把一个 K8S 集群划分为若干个资源不可共享的虚拟集群而诞生的**。



**可以通过在 K8S 集群内创建 namespace 来分隔资源和对象**。比如我有 2 个业务 A 和 B，那么我可以创建 ns-a 和 ns-b 分别部署业务 A 和 B 的服务，如在 ns-a 中部署了一个  deployment，名字是 hello，返回用户的是“hello a”；在 ns-b 中也部署了一个 deployment，名字恰巧也是  hello，返回用户的是“hello b”（要知道，在同一个 namespace 下 deployment 不能同名；但是不同  namespace 之间没有影响）。