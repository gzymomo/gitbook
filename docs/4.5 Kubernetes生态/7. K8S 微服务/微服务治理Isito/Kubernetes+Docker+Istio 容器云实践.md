- [Kubernetes+Docker+Istio 容器云实践](https://www.yisu.com/zixun/23427.html)

### 一、Microservices

#### 1.1 解决大应用微服务化后的问题

现在各大企业都在谈论微服务，在微服务的大趋势之下技术圈里逢人必谈微服务，及微服务化后的各种解决方案。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53818.jpg)

#### 1.2 当我们在讨论微服务的时候我们在讨论什么？

使用微服务架构有很多充分的理由，但天下没有免费的午餐，微服务虽有诸多优势，同时也增加了复杂性。团队应该积极应对这种复杂性，前提是应用能够受益于微服务。

##### 1.2.1 如何微服务化的问题

- 微服务要如何拆分
- 业务API规则
- 数据一致性保证
- 后期可扩展性考虑

当然这不是本文主要讨论的问题，我不讲微服务具体要如何拆分，每个企业每个应用的情况都不太一样，适合自己的方案就是最好的拆分方案。我们主要来解决微服务化后所带来的一些问题。

##### 1.2.2 微服务化后带来的问题

- 环境一致性
- 如何对资源快速分配
- 如何快速度部署
- 怎么做基本监控
- 服务注册与发现
- [负载均衡](https://www.yisu.com/slb/)如何做

以上都是大应用微服务化所需要解决的基础问题，如果还按照传统的方式使用虚拟机来实现，资源开支将会非常大。那么这些问题要怎么解决呢？比如: 

- 流量管理
- 服务降级
- 认证、授权

当然面对上述这些问题我们广大的猿友们肯定是有解决方案的。

#### 1.3 Service governance

##### 1.3.1 Java 体系

假设我们是Java体系的应用，那解决起来就很方便了，比如我们可以考虑使用SpringCloud全家桶系列。也可以拆分使用: 

- Eureka
- Hystrix
- Zuul
- Spring-cloud
- Spring-boot
- ZipKin

Java体系下能很方便的做以我们微服务化后的基础部分，但依然不能非常舒服地解决环境一致性，并且如果有其他语系的服务将很难融入进去。

我们来看基础编程语言一般有什么组合方式来解决基础问题。

##### 1.3.2 其他体系

- Consul
- Kong
- Go-kit
- Jaeger/Zipkin

假设我们是使用Golang语言，这里再捧一下Golang语言。go语言简直就是天生为微服务而生的语言，实在不要太方便了。高效的开发速度及相当不错的性能，简单精悍。

跑题了~我们使用上面这些工具也可以组成一套还不错的微服务架构。

- Consul: 当作服务发现及配置中心来使
- Kong: 作为服务网关
- Jaeger: 作为链路追踪来使
- Go-kit: 开发组件

但是这种方案也有问题，对服务的侵入性太强了，每个服务都需要嵌入大量代码，这还是很头疼的。

### 二、Docker & Kubernetes

基于Docker+k8s搭建平台的实践方案。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53819.jpg)

#### 2.1 Docker

Docker 是一个非常强大的容器。

- 资源利用率的提升
- 环境一致性、可移植性
- 快速度扩容伸缩
- 版本控制

使用了Docker之后，我们发现可玩的东西变多了，更加灵活了。不仅仅是资源利用率提升、环境一致性得到了保证，版本控制也变得更加方便了。

以前我们使用Jenkins进行构建，需要回滚时，又需要重新走一次jenkins Build过程，非常麻烦。如果是Java应用，它的构建时间将会变得非常长。

使用了Docker之后，这一切都变得简单了，只需要把某个版本的镜像拉下来启动就完事了(如果本地有缓存直接启动某个版本就行了)，这个提升是非常高效的。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53820.jpg)

(图片来源网络)

既然使用了Docker容器作为服务的基础，那我们肯定需要对容器进行编排，如果没有编排那将是非常可怕的。而对于Docker容器的编排，我们有多种选择：Docker Swarm、Apache Mesos、Kubernetes，在这些编排工具之中，我们选择了服务编排王者Kubernetes。

##### 2.1.1 Docker VS VM

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53821.jpg)

- VM: 创建虚拟机需要1分钟，部署环境3分钟，部署代码2分钟。
- Docker: 启动容器30秒内。

#### 2.2 Why choose Kubernetes

我们来对比这三个容器编排工具。

##### 2.2.1 Apache Mesos

Mesos的目的是建立一个高效可扩展的系统，并且这个系统能够支持各种各样的框架，不管是现在的还是未来的框架，它都能支持。这也是现今一个比较大的问题：类似Hadoop和MPI这些框架都是独立开的，这导致想要在框架之间做一些细粒度的分享是不可能的。

但它的基础语言不是Golang，不在我们的技术栈里，我们对它的维护成本将会增高，所以我们首先排除了它。

##### 2.2.2 Docker Swarm

Docker Swarm是一个由Docker开发的调度框架。由Docker自身开发的好处之一就是标准Docker API的使用。Swarm的架构由两部分组成：

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53822.jpg)

(图片来源网络)

它的使用，这里不再具体进行介绍。

#### 2.2.3 Kubernetes

Kubernetes是一个Docker容器的编排系统，它使用label和pod的概念来将容器换分为逻辑单元。Pods是同地协作（co-located）容器的集合，这些容器被共同部署和调度，形成了一个服务，这是Kubernetes和其他两个框架的主要区别。相比于基于相似度的容器调度方式（就像Swarm和Mesos），这个方法简化了对集群的管理.

不仅如此，它还提供了非常丰富的API，方便我们对它进行操作，及玩出更多花样。其实还有一大重点就是符合我们的Golang技术栈，并且有大厂支持。

Kubernetes 的具体使用这里也不再过多介绍，网站上有大把资料可以参考。

#### 2.3 Kubernetes in kubernetes

kubernetes（k8s）是自动化容器操作的开源平台，这些操作包括部署、调度和节点集群间扩展。

- 自动化容器的部署和复制
- 随时扩展或收缩容器规模
- 将容器组织成组，并且提供容器间的负载均衡
- 很容易地升级应用程序容器的新版本
- 提供容器弹性，如果容器失效就替换它，等等...

#### 2.4 Kubernetes is not enough either

到这里我们解决了以下问题:

- Docker: 环境一致性、快速度部署。
- Kubernetes: 服务注册与发现、负载均衡、对资源快速分配。

当然还有监控，这个我们后面再说。我们先来看要解决一些更高层次的问题该怎么办呢？

在不对服务进行侵入性的代码修改的情况下，服务认证、链路追踪、日志管理、断路器、流量管理、错误注入等等问题要怎么解决呢？

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53824.jpg)

这两年非常流行一种解决方案：Service Mesh。

### 三、Service Mesh

处理服务间通信的基础设施层，用于在云原生应用复杂的服务拓扑中实现可靠的请求传递。

- 用来处理服务间通讯的专用基础设施层，通过复杂的拓扑结构让请求传递的过程变得更可靠。
- 作为一组轻量级高性能网络代理，和程序部署在一起，应用程序不需要知道它的存在。

在云原生应用中可靠地传递请求可能非常复杂，通过一系列强大技术来管理这种复杂性: 链路熔断、延迟感知、负载均衡，服务发现、服务续约及下线与剔除。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53825.jpg)

市面上的ServiceMesh框架有很多，我们选择了站在风口的Istio。

#### 3.1 Istio

连接、管理和保护微服务的开放平台。

- 平台支持: Kubernetes, Mesos, Cloud Foundry。
- 可观察性:Metrics, logs, traces, dependency 。visualisation。
- Service Identity & Security: 为服务、服务到服务的身份验证提供可验证的标识。
- Traffic 管理: 动态控制服务之间的通信、入口/出口路由、故障注入。
- Policy 执行: 前提检查，服务之间的配额管理。

#### 3.2 我们为什么选择Istio？

因为有大厂支持~其实主要还是它的理念是相当好的。

虽然它才到1.0版本，我们是从 0.6 版本开始尝试体验，测试环境跑，然后0.7.1版本出了，我们升级到0.7.1版本跑，后来0.8.0LTS出了，我们开始正式使用0.8.0版本，并且做了一套升级方案。

目前最新版已经到了1.0.4, 但我们并不准备升级，我想等到它升级到1.2之后，再开始正式大规模应用。0.8.0LTS在现在来看小规模还是可以的。

#### 3.3 Istio 架构

我们先来看一下Istio的架构。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53828.jpg)

其中Istio控制面板主要分为三大块，Pilot、Mixer、Istio-Auth。

- Pilot: 主要作为服务发现和路由规则，并且管理着所有Envoy，它对资源的消耗是非常大的。
- Mixer: 主要负责策略请求和配额管理，还有Tracing，所有的请求都会上报到Mixer。
- Istio-Auth: 升级流量、身份验证等等功能，目前我们暂时没有启用此功能，需求并不是特别大，因为集群本身就是对外部隔离的。

每个Pod都会被注入一个Sidecar，容器里的流量通过iptables全部转到Envoy进行处理。

### 四、Kubernetes & Istio

Istio可以独立部署，但显然它与Kuberntes结合是更好的选择。基于Kubernetes的小规模架构。有人担心它的性能，其实经过生产测试，上万的QPS是完全没有问题的。

#### 4.1 Kubernetes Cluster

在资源紧缺的情况下，我们的k8s集群是怎么样的？

##### 4.1.1 Master集群

- Master Cluster:
  - ETCD、Kube-apiserver、kubelet、Docker、kube-proxy、kube-scheduler、kube-controller-manager、Calico、 keepalived、 IPVS。

##### 4.1.2 Node节点

- Node:
  - Kubelet、 kube-proxy 、Docker、Calico、IPVS。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53829.jpg)

(图片来源网络)

我们所调用的Master的API都是通过 keepalived 进行管理，某一master发生故障，能保证顺滑的飘到其他master的API，不影响整个集群的运行。

当然我们还配置了两个边缘节点。

##### 4.1.3 Edge Node

- 边缘节点
- 流量入口

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53830.jpg)

边缘节点的主要功能是让集群提供对外暴露服务能力的节点，所以它也不需要稳定，我们的IngressGateway 就是部署在这两个边缘节点上面，并且通过Keeplived进行管理。

#### 4.2 外部服务请求流程

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53832.jpg)

最外层是[DNS](https://www.yisu.com/dns/)，通过泛解析到Nginx，Nginx将流量转到集群的VIP，VIP再到集群的HAproxy，将外部流量发到我们的边缘节点Gateway。

每个VirtualService都会绑定到Gateway上，通过VirtualService可以进行服务的负载、限流、故障处理、路由规则及金丝雀部署。再通过Service最终到服务所在的Pods上。

这是在没有进行Mixer跟策略检测的情况下的过程，只使用了Istio-IngressGateway。如果使用全部Istio组件将有所变化，但主流程还是这样的。

#### 4.3 Logging

日志收集我们采用的是低耦合、扩展性强、方便维护和升级的方案。

- 节点Filebeat收集宿主机日志。
- 每个Pods注入Filebeat容器收集业务日志。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53833.jpg)

Filebeat会跟应用容器部署在一起，应用也不需要知道它的存在，只需要指定日志输入的目录就可以了。Filebeat所使用的配置是从ConfigMap读取，只需要维护好收集日志的规则。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53835.jpg)

上图是我们可以从Kibana上看到所采集到的日志。

#### 4.4 Prometheus + Kubernetes

- 基于时间序列的监控系统。
- 与kubernetes无缝集成基础设施和应用等级。
- 具有强大功能的键值数据模型。
- 大厂支持。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53836.jpg)

##### 4.4.1 Grafana

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53837.jpg)

##### 4.4.2 Alarm

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53838.jpg)

目前我们支持的报警有Wechat、kplcloud、Email、IM。所有报警都可在平台上配置发送到各个地方。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53840.jpg)

#### 4.4.3 整体架构

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53841.jpg)

整个架构由外围服务及集群内的基础服务组成，外围服务有:

- Consul作为配置中心来使用。
- Prometheus+Grafana用来监控K8s集群。
- Zipkin提供自己定义的链路追踪。
- ELK日志收集、分析，我们集群内的所有日志会推送到这里。
- Gitlab代码仓库。
- Jenkins用来构建代码及打包成Docker镜像并且上传到仓库。
- Repository 镜像仓库。

集群有:

- HAProxy+keeprlived 负责流量转发。
- 网络是Calico, Calico对kube-proxy的ipvs代理模式有beta级支持。如果Calico检测到kube-proxy正在该模式下运行，则会自动激活Calico ipvs支持，所以我们启用了IPVS。
- 集群内部的DNS是 CoreDNS。
- 我们部署了两个网关，主要使用的是Istio的 IngressGateway，TraefikIngress备用。一旦IngressGateway挂了我们可以快速切换到TraefikIngress。
- 上面是Istio的相关组件。
- 最后是我们的APP服务。
- 集群通过Filebeat收集日志发到外部的ES。
- 集群内部的监控有：
  - State-Metrics 主要用来自动伸缩的监控组件
  - Mail&Wechat 自研的报警的服务
  - Prometheus+Grafana+AlertManager 集群内部的监控，主要监控服务及相关基础组件
  - InfluxDB+Heapster 流数据库存储着所有服务的监控信息

#### 4.5 有了Kubernetes那怎么部署应用呢？

##### 4.5.1 研发打包成镜像、传仓库、管理版本

- 学习Docker。
- 学习配置仓库、手动打包上传麻烦。
- 学习k8s相关知识。

##### 4.5.2 用Jenkins来负责打包、传镜像、更新版本

- 运维工作增加了不少，应用需要进行配置、服务需要做变更都得找运维。
- 需要管理一堆的YAML文件。

有没有一种傻瓜式的，不需要学习太多的技术，可以方便使用的解决方案？

### 五、Kplcloud platform

#### 5.1 开普勒云平台

开普勒云平台是一个轻量级的PaaS平台。

- 为微服务化的项目提供一个可控的管理平台。
- 实现每个服务独立部署、维护、扩展。
- 简化流程，不再需要繁琐的申请流程，最大限度的自动化处理。
- 实现微服务的快速发布、独立监控、配置。
- 实现对微服务项目的零侵入式的服务发现、服务网关、链路追踪等功能。
- 提供配置中心，统一管理配置。
- 研发、产品、测试、运维甚至是老板都可以自己发布应用。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53843.jpg)

#### 5.2 在开普勒平台部署服务

为了降低学习成本及部署难度，在开普勒平台上部署应用很简单，只需要增加一个Dockerfile 就好了。

Dockerfile 参考：

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53844.jpg)

以上是普通模式，Jenkins代码Build及Docker build。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53845.jpg)

这是一种相对自由的部署方式，可以根据自己的需求进行定制，当然有学习成本。

##### 5.2.1 为什么不自动生成Dockerfile呢？

其实完全可以做到自动生成Dockerfile，但每个服务的要求可能不一样，有些需要增加文件、有些在Build时需要增加参数等等。我们不能要求所有的项目都是一样的，这会阻碍技术的发展。所以退而求其次，我们给出模版，研发根据自己的需求调整。

#### 5.3 工具整合

- 开普勒云平台整合了 gitlab，Jenkins，repo，k8s，istio，promtheus，email，WeChat 等API。
- 实现对服务的整个生命周期的管理。
- 提供服务管理、创建、发布、版本、监控、报警、日志已及一些周边附加功能，消息中心、配置中心、还能登陆到容器，服务下线等等。
- 可对服务进行一健调整服务模式、服务类型、一键扩容伸缩，回滚服务API管理以及存储的管理等操作。

#### 5.4 发布流程

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53846.jpg)

用户把自己的Dockerfile跟代码提交到Gitlab，然后在开普勒云平台填写一些参数创建自己的应用。

应用创建完后会在Jenkins创建一个Job，把代码拉取下来并执行Docker build（如果没有选择多阶构建会先执行go build或mvn），再把打包好的Docker image推送到镜像仓库，最后回调平台API或调用k8s通知拉取最新的版本。

用户只需要在开普勒云平台上管理好自己的应用就可以，其他的全部自动化处理。

#### 5.5 从创建一个服务开始

我们从创建一个服务开始介绍平台。

平台主界面:

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53847.jpg)

点击“创建服务”后进入创建页面。

填写基本信息:

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53848.jpg)

填写详细信息: 

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53849.jpg)

基本信息以Golang为例，当选择其他语言时所需填写的参数会略有不同。

如果选择了对外提供服务的话，会进入第三步，第三步是填写路由规则，如没有特殊需求直接默认提交就行了。

##### 5.5.1 服务详情

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53850.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53851.jpg)

Build 升级应用版本: 

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53852.jpg)

调用服务模式，可以在普通跟服务网格之间调整。

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53853.jpg)

服务是否提供对外服务的能力: 

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53854.jpg)

扩容调整CPU、内存：

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53855.jpg)

调整启动的Pod数量：

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53856.jpg)

网页版本的终端：

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53857.jpg)

##### 5.5.2 定时任务

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53858.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53859.jpg)

##### 5.5.3 持久化存储

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53860.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53861.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53862.jpg)

管理员创建StorageClass跟PersistentVolumeClaim，用户只需要在自己服务选择相关的PVC进行绑写就行了。

存储使用的是NFS。

##### 5.5.4 Tracing

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53863.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53864.jpg)

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53865.jpg)

##### 5.5.5 Consul

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53866.jpg)

Consul当作配置中心来使用，并且我们提供Golang的客户端。

```
$ go get github.com/lattecake/consul-kv-client
```

它会自动同步consul的目录配置存在内存，获取配置只需要直接从内存拿就行了。

##### 5.5.6 Repository

![Kubernetes+Docker+Istio 容器云实践](https://cache.yisu.com/upload/information/20200309/33/53867.jpg)

- Github: https://github.com/kplcloud/kplcloud
- Document: https://docs.nsini.com
- Demo: https://kplcloud.nsini.com