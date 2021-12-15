原创：王科怀（行松）：微信公众号： [ Serverless ]

## 如何通过 Serverless 提高 Java 微服务治理效率？



微服务治理面临的挑战



#  

在业务初期，因人手有限，想要快速开发并上线产品，很多团队使用单体的架构来开发。但是随着公司的发展，会不断往系统里面添加新的业务功能，系统越来越庞大，需求不断增加，越来越多的人也会加入到开发团队，代码库也会增速的膨胀，慢慢的单体应用变得越来越臃肿，可维护性和灵活性逐渐降低，维护成本越来越高。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1sLhnYWMLNgiafxzkxPZwvmuIHRzBGB02Oe0OoUL1Giafs4CXVEva8PnGy2BPiaCjIsNicpMO28fFAmlg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这个时候很多团队会把单体应用架构改为微服务的架构，解决单体应用的问题。但随着微服务越来越多，运维投入会越来越大，需要保证几十甚至几百个服务正常运行与协作，这给运维带来了很大的挑战，下面从软件生命周期的角度来分析这些挑战：



- 开发测试态



- - 如何实现开发、测试、线上环境隔离？
  - 如何快速调试本地变更？
  - 如何快速部署本地变更？



- 发布态



- - 如何设计服务发布策略？
  - 如何无损下线旧版本服务？
  - 如何实现对新版本服务灰 度测试？



- 运行态



- - 线上问题如何排查？有什么工具可以利用呢？
  - 对于服务质量差的节点如何处理？
  - 对于完全不工作的实例我们如何恢复？



面对以上问题，Serverless 应用引擎在这方面都做了哪些工作？



Serverless 应用引擎



#  

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1sLhnYWMLNgiafxzkxPZwvmuc2SRI9Sh28Knp6Y9fL8OOH5iaqRHkD1udhwQ6kviagmkmiaicFlMxrsiafg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



如上图所示，Serverless 应用引擎（SAE）基于神龙 + ECI + VPC + SLB + NAS 等 IaaS 资源，构建了一个 Kubernetes  集群，在此之上提供了应用管理和微服务治理的一些能力。它可以针对不同应用类型进行托管，比如 Spring Cloud 应用、Dubbo  应用、HSF 应用、Web 应用和多语言应用。并且支持 Cloudtoolkit 插件、云效 RDC / Jenkins 等开发者工具。在  Serverless 应用引擎上，零代码改造就可以把 Java 微服务的应用迁移到 Serverless。



总的来说，Serverless 应用引擎能够提供成本更优、效率更高的一站式应用托管方案，零门槛、零改造、零容器基础，即可享受 Serverless + K8s + 微服务带来的技术红利。



微服务治理实践



#  

## **1. 开发态实践**

###  

### **1）多环境管理**



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



- 多租户共有一个注册中心，通过不同的租户对流量进行隔离；更进一步可以通过网络 VPC 进行环境隔离；
- 提供环境级别的运维操作，比如一键停止和拉起整个环境的功能；
- 提供环境级别的配置管理；
- 提供环境级别的网关路由流量管理。



### **2）云端联调**



Serverless 应用引擎（SAE）基于 Alibaba CloudToolkit 插件+ 跳板机可以实现：



- 本地服务订阅并注册到云端 SAE 内置的注册中心；
- 本地服务可以和云端 SAE 服务互相调用。



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



如上图所示，在实现的时候用户需要有一个 ECS 代理服务器，实际注册的是 ECS 代理服务器到 SAE 的注册中心，IDEA 在安装 Cloudtoolkit  插件以后，在启动进程时，会在本地拉起一个通道服务，这个通道服务会连上 ECS 代理服务器，本地所有的请求都会转到 ECS  代理服务器上，云端对服务的调用也会通过 ECS 代理转到本地，这样就可以以最新的代码在本地断点调试，这就是云端联调的实现。

###  

### **3）构建快速开发体系**



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



代码在本地完成联调以后，要能快速地通过 Maven 插件和 IDEA-plugin，可以很快地一键部署到云端的开发环境。



## **2. 发布态实践**



### **1）应用发布三板斧**



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



- **可灰度**：应用在发布的过程中，运维平台一定要有发布策略，包括单批、分批、金丝雀等发布策略；同时还要支持流量的灰度；批次间也要允许自动/手动任选。
- **可观测**：发布过程可监控，白屏化实时查看发布的日志和结果，及时定位问题。
- **可回滚**：允许人工介入控制发布流程：异常中止、一键回滚。



通过这三点可以让应用发布做到可灰度、可观测、可回滚。



### **2）微服务无损下线**



在版本更换的过程中，SAE 是如何保证旧版本的微服务流量可以无损地下线掉？



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



上图是微服务注册和发行的整个流程，图中有服务消费者和服务提供者，服务提供者分别有 B1、B2 两台实例，服务消费者分别有 A1、A2 两台实例。



B1、B2 把自己注册到注册中心，消费者从注册中心刷新服务列表，发现服务提供者 B1、B2，正常情况下，消费者开始调用 B1 或者 B2，服务提供者 B  需要发布新版本，先对其中一个节点进行操作，如 B1，首先停止 Java  进程，服务停止过程又分为主动销毁和被动销毁，主动销毁是准实时的，被动销毁的时间由不同的注册中心决定，最差的情况可能需要一分钟。



如果应用是正常停止，Spring Cloud 和 Dubbo 框架的 ShutdownHook  能正常被执行，这一步的耗时基本上是可以忽略不计的。如果应用是非正常停止，比如说直接 Kill-9 的一个停止，或者是 Docker  镜像构建的时候，Java 进程不是一号进程，且没有把 Kill  信号传递给应用的话，那么服务提供者不会主动去注销节点，它会等待注册中心去发现、被动地去感知服务下线的过程。



当微服务注册中心感知到服务下线以后，会通知服务消费者其中一个服务节点已下线，这里有两种方式：注册中心的推送和消费者的轮巡。注册中心刷新服务列表，感知到提供者已经下线一个节点，这一步对于 Dubbo 框架来说不存在，但对于 Spring Cloud 来说，它最差的刷新时间是 30  秒。等消费者的服务列表更新以后，就不再调用下线节点 B。从第 2 步到第 6 步的过程中，注册中心如果是  Eureka，最差的情况需要消耗两分钟；如果是 Nacos，最差的情况需要消耗 50 秒。



在这个时间内请求都有可能出现问题，所以发布的时候会出现各种报错。



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



经过上面的分析，在传统的发布流程中，客户端有一个服务端调用报错期，这是由于客户端没有及时感知到服务端下线的实例造成的，这种情况主要是因为服务提供者借助微服务，通知消费者来更新服务提供的列表造成的。



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



那能否绕过注册中心，服务提供者直接通知服务消费者？答案是肯定的。SAE 做了两件事情，第一，服务提供者在应用发布前，会主动向服务注册中心注销应用，并将应用标记为已下线状态，将原来停止进程阶段的注销变成了 preStop 阶段注销进程。



在接收到服务消费者的请求时，首先会正常处理本次请求，并且通知服务消费者此节点已经下线，在此之后消费者收到通知后，会立即刷新自己的服务列表，在此之后服务消费者就不会再把请求发到服务提供者 B1 的实例上。



通过上面这个方案，就使得下线感知时间大大缩短，从原来的分钟级别做到准实时的，确保你的应用在下线时能够做到业务无损。



### **3）基于标签的灰度发布**



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



发布策略分为分批发布和灰度发布，如何实现流量的灰度？从上面的架构图中可以看到，在应用发布之前，要配置一个灰度规则，比如按 uid 的取模余值 =20  来作为灰度流量的规则，当应用发布的时候，会对已发布的节点标识为一个灰度的版本，在这样的情况下，当有流量进来时，微服务网关和消费者都会通过配置中心拿到在治理中心配置的灰度规则。



消费者的 Agent 也会从注册中心拉取它所依赖的服务的一些信息，当一个流量进到消费者时，会按照灰度规则来做匹配，如果是灰度的流量，它会转化到灰度的机器上；如果是正常流量，它会转到正常的机器上，这是基于标签实现的灰度发布的具体逻辑。



## **3. 运行态实践**



### **1）强大的应用监控 & 诊断能力**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1sLhnYWMLNgiafxzkxPZwvmuSROZ1t89alTjiakGju60FSo85KLZQYIJc4aforYdraiamwhX96GkcoEQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



运行态的实例，服务的运行过程中会出现这样或者那样的问题，怎么去排查和解决它？



排查和解决的前提是必须具有强大的应用监控能力和诊断能力，SAE 集成了云产品 ARMS，能够让跑在上面的 Java 微服务看到应用的调用关系拓扑图，可以定位到你的 MySQL 慢服务方法的调用堆栈，进而定位到代码级别的问题。



比如一个请求响应慢，业务出现问题，它可以定位到是哪个请求、哪个服务、服务的哪行代码出现了问题，这样就能为解决问题带来很多便利。总的来说，就是我们要先有监控报警的能力，才能帮助我们更好地诊断服务运营过程中的问题。



### **2）故障隔离和服务恢复**



上面说到我们通过监控、报警来排查、解决遇到的问题，那我们的系统能否主动去做一些事情呢？SAE 作为一个 Serverless 平台，具备很多自运维的能力，下图中有两个场景：



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1sLhnYWMLNgiafxzkxPZwvmuIicDMvpzzaAo2cBSlQnqhOLtWciaUSQjNhBfFOmibIial68f4UsRrRxEOA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



- **场景1**：某应用运营过程中，某几台机器由于磁盘满或者宿主机资源争抢，导致 load 很高或网络状态差，客户端出现调用超时或者报错。



面对这种情况，SAE 提供了服务治理能力，即**离群摘除**，它可以配置，当网络超时严重或者后端服务 5xx 报错达到一定比例时，可以选择把该节点从消费端服务列表中摘除，从而使得有问题的机器不再响应业务的请求，很好地保证业务的 SLA。



- **场景2**：某应用运行过程中，因突发流量导致内存耗尽，触发 OOM。



这种情况下，通过 SAE 这种 Serverless 应用引擎，节点在配置**健康检查**以后，节点里的容器是可以重新拉起的，可以做到快速对进程进行恢复。

 

### **3）精准容量+限流降级+极致弹性**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1sLhnYWMLNgiafxzkxPZwvmu8v3GD95IQQY2F1l6c08PGHxttibicyZWAX50qzcXQpTTGUk4iazJcfO6Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



基于 Serverless Paas 平台 SAE 和其他产品的互动，来达到整个运维态的闭环。



用户在使用的时候，可以运用 PTS 压测工具构造场景，然后得出来一些阈值。比如可以对流量高峰所需要消耗的资源进行预估，这时就可以根据这些阈值设计弹性策略。当业务系统达到请求比例时，就可以按照所设置的弹性策略来扩缩容自己的机器。



扩缩容在时间上，有可能还跟不上处理大批量的请求，这时可以通过和 AHAS 的互动，配置限流降级的能力。当有突发大流量时，首先可以用 AHAS 的能力把一些流量挡在门外，然后同时触发 SAE  上应用的扩容策略去扩容实例，当这些实例扩容完成之后，整个机器的平均负载会下降，流量又重新放进来。从突发大流量到限流降级再到扩容，最后到流量达到正常状态，这就是“精准容量+限流降级+极致弹性”的最佳实践模型。



总结



#  

本文首先按照提出问题、解决问题的思路，阐述微服务在开发、发布和运行态是如何解决问题的；再介绍如何通过 Serverless 产品和其他产品的互动，从而实现精准流量、限流降级和极致弹性。



- 开发测试态



- - 通过注册中心多租户和网络环境的隔离，并提供环境级别的能力；
  - 通过云端联调技术来快速调式本地变更；
  - 如果 IDE 插件快速部署本地变更。



- 发布态



- - 运维平台针对应用发布需要具备可灰度、可观测、 可回滚；
  - 通过 MSE agent 能力实现服务无损下线；
  - 通过标签路由提供了线上流量灰度测试的能力。



- 运行态



- - 建立强大应用监控和诊断能力；
  - 对服务质量差的节点具备离群摘除能力；
  - 对已经不工作的实例通过配置健康检查能够做到实例重启恢复业务；
  - 提供了精准容量+限流降级+极致弹性模型。