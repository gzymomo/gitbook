# 三大容器编排工具

来源：简书，作者：杰_6343，链接：https://www.jianshu.com/p/2bb8f093a4a8

## Docker-Compose

Docker-Compose 是用来管理你的容器的，有点像一个容器的管家，想象一下当你的Docker中有成百上千的容器需要启动，如果一个一个的启动那得多费时间。有了Docker-Compose你只需要编写一个文件，在这个文件里面声明好要启动的容器，配置一些参数，执行一下这个文件，Docker就会按照你声明的配置去把所有的容器启动起来，但是Docker-Compose只能管理当前主机上的Docker，也就是说不能去启动其他主机上的Docker容器

## Docker Swarm

Docker Swarm则是由Docker 公司自行研发的一款用来管理多主机上的Docker容器的工具，可以负责帮你启动容器，监控容器状态，如果容器的状态不正常它会帮你重新帮你启动一个新的容器，来提供服务，同时也提供服务之间的负载均衡，而这些东西Docker-Compose 是做不到的。Swarm现在与Docker Engine完全集成，并使用标准API和网络。Swarm模式内置于Docker CLI中，无需额外安装，并且易于获取新的Swarm命令。



[Swarm](https://docs.docker.com/swarm/)是Docker的原生集群工具，Swarm使用标准的Docker API，这意味着容器能够使用`docker run`命令启动，Swarm会选择合适的主机来运行容器，这也意味着其他使用Docker API的工具比如`Compose`和`bespoke脚本`也能使用Swarm，从而利用集群而不是在单个主机上运行。

Swarm的基本架构很简单：每个主机运行一个Swarm代理，一个主机运行Swarm管理器（在测试的集群中，这个主机也可以运行代理），这个管理器负责主机上容器的编排和调度。Swarm能以高可用性模式（`etcd`、`Consul` 或`ZooKeeper` 中任何一个都可以用来将故障转移给后备管理器处理）运行。当有新主机加入到集群，有几种不同的方式来发现新加的主机，在Swarm中也就是`discovery`。默认情况下使用的是token，也就是在Docker Hub上会储存一个主机地址的列表。



[Docker Swarm](https://docs.docker.com/engine/swarm/) 和 Docker Compose 一样，都是 Docker 官方容器编排项目，但不同的是，Docker Compose 是一个在单个服务器或主机上创建多个容器的工具，而 Docker Swarm 则可以在多个服务器或主机上创建容器集群服务，**对于微服务的部署，显然 Docker Swarm 会更加适合**。



作者：沃尔夫我丢
链接：https://www.jianshu.com/p/71b5511f4f43
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## Kubernetes

Kubernetes它本身的角色定位是和Docker Swarm 是一样的，也就是说他们负责的工作在容器领域来说是相同的部分，当然也有自己一些不一样的特点。这个就像是Eclipse和IDEA一样，也是一个跨主机的容器管理平台。Kubernetes正在成为容器编排领域的领导者，由于其可配置性，可靠性和大型社区的支持，从而超越了Docker Swarm。Kubernetes由Google创建，作为一个开源项目，与整个Google云平台协调工作。此外，它几乎适用于任何基础设施。
 集群包括几个主要组件：
 Pod：在同一节点上一起创建，调度和部署的一个或多个容器的组。
 标签：分配用于标识窗格，服务和复制控制器的键值标签（即名称）。
 服务：服务为一组pod提供名称，充当负载均衡以将流量定向到正在运行的容器。
 复制控制器（Replication controllers）：一个框架，负责确保在任何给定时间安排和运行特定数量的pod复制副本。



来源：简书：作者：沃尔夫我丢：链接：https://www.jianshu.com/p/71b5511f4f43


- **Pods** – Pods是容器一起部署与调度的群体。Pods与其他系统的单一容器相比，它组成了Kubernetes中调度的原子单元。Pod通常会包括1-5个一起提供服务的容器。除了这些用户容器，Kubernetes还会运行其他容器来提供日志和监控服务。在Kubernetes中Pods寿命短暂;随着系统的进化他们不断地构建和销毁。

- **Flat Networking Space\*** – Kubernetes的网络是跟默认的Docker网络不同。在默认Docker网络中， 容器存在于一个私有子网络中，它需要赚翻主机上的端口或者使用代理才能与其他主机上的容器通讯。在Kubernetes，pod中的容器会分享一个IP地址，但是该地址空间跟所有的pods是“平”的，这意味着所有pods不用任何网络地址转换（NAT）就可以互相通讯。这就使得多主机群集更容易管理，不支持链接的代价使得建立单台主机（更准确地说是单个pod）网络更为棘手。由于在同一个pod中的容器共享一个IP，它们可以通过使用本地主机地址端口进行通信（这并不意味着你需要协调pod内的端口使用）。

- **Labels** – Labels是附在Kubernetes对象（主要是pods）上用于描述对象的识别特征的键值对，例如版本：开发与层级：前端。通常Labels不是唯一的；它们用来识别容器组。Labels选择器可以用来识别对象或对象组，例如设置所有在前端层的pods与环境设置为`production`。使用Labels可以很容易地处理分组任务，例如分配pods到负载均衡组或者在组织之间移动pods。

- **Services** – Services是通过名称来定位的稳定的节点。Services使用label选择器来连接pods，比如“缓存”Service可以连接到标识为label选择器“type”为“redis”的某些“redis”pods。该service将在这些pods之间自动循环地请求。以这种方式，Services可用于连接一个系统各部件。使用Services会提供一个抽象层，这意味着应用程序并不需要知道他们调用的service的内部细节，例如pods内部运行的应用程序只需要知道调用的数据库service的名称和接口，它不必关心有多少pods组成了那个数据库，或者上次它调用了哪个pod。 Kubernetes会为集群建立一个DNS服务器，用于监视新的services并允许他们在应用程序代码和配置文件中按名称定位。它也可以设置services不指向pods而是指向其他已经存在的services，比如外部API或数据库。

- **Replication Controllers** - Replication controllers是Kubernetes实例化pods的正常方式（通常情况下，在Kubernetes中不使用Docker CLI）。它们为service来控制和监视运行的pods数量（称为`replicas`）。例如，一个replication controller可以负责维持5个Redis的pods的运行。如果一个失败，它会立即启动一个新的。如果replicas的数量减少，它会停止多余的pods。虽然使用Replication Controllers来实例化所有pods会增加一层额外的配置，但是它显著提高容错性和可靠性。

  

  
  

  

## 简而言之

Docker Compose是一个基于Docker的单主机容器编排工具，功能并不像Docker Swarm和Kubernetes是基于Dcoker的跨主机的容器管理平台那么丰富。




