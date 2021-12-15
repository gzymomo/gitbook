[云边一体 | 基于Kubernetes打造边缘云计算](https://developer.aliyun.com/article/739188)



> 云原生的理念如今正如火如荼。它不仅仅是一种技术，更是随着云生态的发展而被逐渐提炼出的一系列技术、最佳实践与方法论的集合；它带来了资源利用率提升、分布式系统的弹性扩展与可靠性等能力，能够让IT系统最大程度的享受云计算红利，业界全面拥抱云原生就是最好的佐证。伴随5G、IoT的发展，边缘计算正在成为云计算的新边界，而规模和复杂度的日益提升对边缘计算的效率，可靠性，资源利用率等一系列能力又有了新的诉求。试想，如果能将云能力从中心往边缘触达，上述问题是不是将迎刃而解？那么在云原生时代构建云到边的触达通路，保持云边一致性体验，我们的抓手又在哪里呢？本次分享将一一为你揭晓；



# 引言

云原生的理念如今正如火如荼。它不仅仅是一种技术，更是随着云生态的发展而被逐渐提炼出的一系列技术、最佳实践与方法论的集合；它带来了资源利用率提升、分布式系统的弹性扩展与可靠性等能力，能够让IT系统最大程度的享受云计算红利，业界全面拥抱云原生就是最好的佐证。

伴随5G、IoT的发展，边缘计算正在成为云计算的新边界，而规模和复杂度的日益提升对边缘计算的效率，可靠性，资源利用率等一系列能力又有了新的诉求。试想，如果能将云能力从中心往边缘触达，上述问题是不是将迎刃而解？那么在云原生时代构建云到边的触达通路，保持云边一致性体验，我们的抓手又在哪里呢？本次分享将一一为你揭晓。

# 云原生概念

云原生的概念最早是在2013年被提出，经过这几年的发展，尤其是从 2015 年 Google 牵头成立 CNCF 以来，云原生技术开始进入公众的视线并逐渐演变成包括  DevOps、持续交付、微服务、容器、基础设施，Serverless，FaaS等一系列的技术，实践和方法论集合。伴随着技术的普及，与之相配套的团队建设，技术文化，组织架构和管理方法也呼之欲出。越来越多的企业选择云原生构建其应用来获得更好的资源效率和持续的服务能力。相比较过往着力云原生概念的普及、理解和力求共识，云原生落地已经成为现如今I/CT日常主旋律。

云原生的技术范畴包括了以下几个方面：云应用定义与开发、云应用的编排与管理、监控与可观测性、云原生的底层技术（比如容器运行时、云原生存储技术、云原生网络技术等）、云原生工具集、Serverless。虽然，CNCF以目前200多个项目和产品（CNCF云原生全景https://github.com/cncf/landscape） 的巨大体量保持高速发展，不断壮大云原生体系的技术集合，但所有项目基本都在上述技术范畴中，紧守云原生理念不放；

那么云原生的核心优势到底有哪些，能带给我们真实体感又是什么？

引用CNCF的重新定义："Cloud native technologies empower organizations to build and run scalable  applications in modern, dynamic environments such as public, private,  and hybrid clouds. Containers, service meshes, microservices, immutable  infrastructure, and declarative APIs exemplify this approach.These  techniques enable loosely coupled systems that are resilient,  manageable, and observable. Combined with robust automation, they allow  engineers to make high-impact changes frequently and predictably with  minimal toil."
![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwyWktKibAdc0pEjjKt07ro8YdicPSkQ0kSa9WiciaHoeVJDvWvp04A3zQzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 云原生和边缘基础设施

在云计算真正普及之前，获取基础设施能力（比如服务发现、流量控制、监控与可观测性、访问控制、网络控制、存储层抽象等）需要应用通过某种抽象或接口方式，使得两者之间是非常紧密的耦合关系，应用本身的能力和演进需要强依赖基础设施。而在云原生时代，类似kubernetes这样的标准化资源抽象、编排、整合平台促进基础设施能力正在不断下沉，云能力和应用之间的隔阂也正在云原生技术体系下被不断瓦解。“面向云架构”、“面向云编程”的呼声日渐高涨的同时，cloudnative的云基础设施越来越“友好”。但是，云原生基础设施概念远远超出公有云上运行的基础设施范畴。

“软硬件技术的成熟、巨大的社会价值和伟大的商业模式”造就了云计算的兴起和蓬勃发展，迅猛的势头也催生了各种新型技术架构的诞生，笔者认为云原生的出现同样是顺应了技术和商业的浪潮。

放眼当下，随着互联网智能终端设备数量的急剧增加和数据、业务下沉的诉求增多，边缘计算规模和业务复杂度已经发生了翻天覆地的变化，边缘智能、边缘实时计算、边缘分析等新型业务不断涌现。传统云计算中心集中存储、计算的模式已经无法满足边缘设备对于时效、容量、算力的需求，如何打造边缘基础设施成了一个新课题。

本次分享也是从这个问题出发，和大家一起探讨云原生基础设施在边缘计算落地的可能性。
试想，从云到端，是否能将云计算的能力下沉到边缘侧、设备侧，并通过中心进行统一交付、运维、管控，通过粘合云计算核心能力和边缘算力，构筑在边缘基础设施之上的云计算平台？在回答这个问题之前，我们先梳理一下边缘计算可能面临的一些挑战：

- 云边端协同：缺少统一的交付、运维、管控标准。
- 安全：边缘服务和边缘数据的安全风险控制难度较高。
- 网络：边缘网络的可靠性和带宽限制。
- 异构资源：对不同硬件架构、硬件规格、通信协议的支持，以及基于异构资源、网络、规模等差异化提供标准统一的服务能力的挑战。

这些问题云原生模式能解吗，云原生的技术底座kubernetes能撑起大梁吗？众所周知，以Kubernetes为基础的云原生技术，核心价值之一是通过统一的标准实现在任何基础设施上提供和云上一致的功能和体验：那么借助云原生技术，可以实现云-边-端一体化的应用分发，解决在海量边、端设备上统一完成大规模应用交付、运维、管控的诉求是不是就有可能；同时在安全方面，云原生技术可以提供容器等更加安全的工作负载运行环境，以及流量控制、网络策略等能力，能够有效提升边缘服务和边缘数据的安全性；而依托云原生领域强大的社区和厂商支持，云原生技术对异构资源的适用性逐步提升，在物联网领域，云原生技术已经能够很好的支持多种CPU架构(x86-64/arm/arm64)和通信协议，并实现较低的资源占用。k8s虽好，但落地边缘计算场景好像还有一段路要走。

# Kubernetes概念

## 基本概念

“Kubernetes——让容器应用进入大规模工业生产，已然成了容器编排系统的事实标准；”下图是kubernetes架构图（图文引用自https://jimmysong.io/kubernetes-handbook/cloud-native/from-kubernetes-to-cloud-native.html）

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwKcRjj8icf8RZwyglEL7p4zX1Ky31fTqpruhw3f45mzNLn4VKtPIfU3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层：kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
- Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS应用、ChatOps等
- Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

## Kubernetes落地形态

随着kubernetes的普及，越来越来的上层系统开始搭建在kubernetes底座之上，比如CICD、Serverless、IoT、PaaS等业务。过往经验告诉我们，在享受kubernetes带来的方便快捷集成体验的同时，kubernetes的自身运维也带来了额外的巨大工作量：kubernetes的升级、高可用部署，资源对接、监控、事件告警、多租管理等等。因此，专门的运维团队，运维工具，以及公有云托管服务就应运而生了，这也是当前业界kubernetes几种常见的服务形态。

先说说公共云托管kubernetes服务，目前主流云厂商基本上都推出了XKS/XCK的服务，虽然底层实现各异，但是都有着类似的用户体验。用户只需要在云厂商界面提交集群配置，点击购买按钮，一个高可用的，无缝对接各种云资源，云服务的kubernetes集群就能够在3-5分钟内被创建出来；且后续的集群升级、运维、扩缩容等等运维操作都是动动小手的白屏化操作，体验简洁到极致。kubernetes强大接口能力也能够让用户快速便捷的按需使用上各厂商的IaaS资源。综合各种因素，公共云托管kubernetes服务正成为大多数用户的选择。但往往也有出于各种数据安全、集群规模、价格因素等等考虑，部分用户也会选择自建或者使用云厂商的专有云kubernetes版本、混合云版本。如果从kubernetes层面考量，三者本质都大同小异，依赖第三方提供一个稳定、兼容、安全可以持续发展的商业化kubernetes底座，以期得到规模化、标准化的k8s服务能力。

重新回到前面的问题，边缘计算基础设施既要云原生，又要云边一体，还要一致性体验，那么使用云端托管、边缘定制会不会是一个很好的选择呢？接下来展开叙述。

# 云端管控、边缘自治

“云边端一体化协同”作为标准化的一个构想，将标准化的云原生能力向边缘端复制，需要分为三个层次：

- 第一个层次是能够在云端提供标准化的接口、管控能力，或者是标准的云服务和云资源的接入能力。其中我们能够看到 Kubernetes 的身影；
- 第二个层次是能够高效的管理处在整个边缘端的众多资源，其中包括在边缘端应用的效率问题。
- 第三层次是典型的 IoT 场景中的端设备，例如智慧楼宇智能停车设备，蓝牙设备、人脸识别设备等。
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwayWicpvWSRLpJksj4eZhNicxWASkH9zB6RhoxxRYoMoh0SkHtic2VvH0g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 云端托管原生k8s

先看最核心的云端托管层，在云边一体的设计理念中，自然而然的能够联想到将原生的标准k8s托管在公共云上，开箱即用各种被集成的云能力必定是不二之选。几点原因，k8s免运维上面已经谈到，原生的k8s便于被上层业务系统集成却常常被忽视，通过云厂商将kubernetes和其他云能力（弹性、日志监控、应用市场，镜像服务等）打通，无论是终端用户直接使用，还是做新业务创新，复杂度都大大降低；此外，将云管控作为中心式服务，通过提供统一管控能力，反而非常适合管理边缘场景中零散分布的计算资源和应用，比如CDN服务、IoT业务等；最后，云原生的方式又可以让用户以往K8s经验用于新的边缘计算业务。

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwMYWFQoo8GZPDHVStB6xjABGwfPAMsfGYCKlKiaDELReQG8EEdwuHkkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图所示的管控架构也符合前文提到的云边分层结构：“分而自治，中心管控”。

## 适度定制适配边缘场景

k8s除了上述诸般特征之外，还有一个强大的插件机制，CxI自不必说，CRD、Operator更是让k8s如虎添翼。在面向边缘场景，自然而然会有一些别样的逻辑需要适配，例如：边缘节点和云端管控走弱连接公网交互带来的管理逻辑适配；边缘节点应用驱逐逻辑适配；边缘节点支持原生运维接口适配等等。插件化的非侵入方式可以让原生k8s在免受任何一丁点的冲击的同时，扩展更多的逻辑，比如用来做边缘单元化管理的EdgeUnit Operator，再比如做边缘节点自治管理的EdgeNode Operator。

前文提到，边缘托管集群要将运维能力统一收编到云端，中心式的运维能力高效又便捷；所以，除了各种控制器之外，云端还有各种运维控制器来做日志、监控数据等请求的转运，如：日志控制器，MetricServer等；EdgeTunnelServer提供了一条稳健的反向数据通道，很多云到边的请求都要从此经过，至于它的用途后面分解；
![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwSKeur19Bl6hwRGYce3zlia3Aq10dt1LDUOt18ic6ZaG55icAb1zTvUGsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 自治

如上图所示，每个Edge节点上配备了一个EdgeHub组件，边缘节点自治离不开它。

先说说背景，众所周知k8s原生设计为数据中心内部的容器编排系统（DCOS），默认网络可靠、可信。管控端（apiserver、调度器，资源控制器）源源不断收集节点心跳做相应的调度决策，若worker节点和管控断链，原生k8s管控就要将节点置为不可用、并驱逐应用（容忍时间到了之后）。而这些看似再正常不过的业务逻辑，在边缘场景确是万万不可接受的。

在“分而自治，中心管控”的设计理念下，woker节点和中心管控走弱链接的公网交互，各种环境、网络、人为因素都会带来网络的抖动，甚至不可用。但是此时，边缘节点上的业务和agent却要能够持续提供服务，这就是自治。概括起来还包括，worker节点和云端断连后：

- 节点上agent（kubelet、proxy等）无感知持续运行，就像还稳稳的连着master（其实是要骗骗它们）
- 节点间东西向流量持续稳定交互（老师不在，同学们自己上自习）；
- 节点重启后，节点上管控元信息和应用原信息能够恢复原样，因为没法通过管控配置中心同步（邮局关门，我不能给你写信，我就在原地等你--mac地址保持、podIP保持等）；
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwRHvu8eV3bNkZDSqb2VxgC6QEEbq0ZwI3h965z4sCwicA0pVFc6OaibOA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

EdgeHub作为节点上的临时配置中心，在断链情况下，持续为节点上所有设备提供断网前一刻的配置服务。数据持久化在本地磁盘，和节点共存亡。EdgeHub上实现了诸多k8s api，除了给agent用，节点上承载的业务pod也可以使用，轻量又便捷；

### 原生运维支持

说完边缘端，再来说说云边链接的“脐带”--EdgeTunnel。用过k8s命令行工具--kubectl的同学，肯定会对“kubectl exec，kubectl logs， kubectl attach，kubectl  top”等运维命令直呼过瘾，原理也很简单，大致就是apiserver建联到kubelet提供长链接支持。但是在边缘场景，由于大多数边缘节点没有暴露在公网之上，主动的云到边的网络建链突然变成不可能，所有的原生运维api黯然失效。

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwFPficBBQvAkicI1cZ2M9a7zB14licOnicxJV9RNTT5S5iaZnaMBs2ls0jqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

EdgeTunnel的设计目的就是为了弥补这份缺失，它通过在管控与边缘worker节点之间建立反向通道，并和worker节点的生命周期完整联动，支持证书配置，支持websocktet等等。我们看到最终apiserver通过它按需中转，metricserver（k8s原生运维工具）通过它按需中转......至此，这条宽似太平洋的通道让运维接口又转动起来，既安全又高效，皆大欢喜。

### 边缘单元

在真实落地过程中，我们发现边缘场景还真的是新奇又多彩。先卖个关子：

客户A，IoT厂商，它们负责交付安防设备到商场，机场、火车站等；但是各场所的资源、应用管理诉求不同，客户希望能够一次提交，单元化管理，且要有“逻辑多租能力”：流量单一场地闭环，部署按场地调度；客户B，CDN厂商，机房遍布世界各地，海外和国内业务部署差异明显，资源规格各异；希望能够将资源分组，为后续开展不同业务铺垫。

单元化，单元化流量管理，单元化调度、批量管理？k8s好像没有这个能力。千真万确，是没有，但是k8s的自定义资源控制器（CRD&Operator）提供了一条方便的解决途径。
![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwjyDUibxVzpCokOeJcbepUWHfbeu7m7yG2jqFnibC0NdfYvU0jgAMGCjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过k8s的扩展机制，在原生k8s基础上我们增加了edgeunit的管控逻辑，简言之：

- 分属同一个edgeunit的worker节点，可以批量控制节点上的标签、annotation，设置节点调度状态等等；
- 用户提交应用，可以按照单元化部署；如用户提交一份“deployment”配置，通过边缘单元的调度控制，可以在每个edgeunit中被克隆部署；除了批量管理之外，也可以赋予“单元”的独立控制能力，灵活高效。
- 用户的服务流量控制，可以在单元内闭环。大家都知道，原生k8s  service提供的集群内东西流量交互是不会感知边缘场景下节点位置属性而做额外处理的，但是出于数据安全考虑或天然的网络隔离等原因，我们需要将东西流量限制在一个边缘单元内，这也是edgeunit在service管理上需要额外设计的逻辑；

### 轻量化

上文提到，边缘算力的涵义颇广，包括：规模较大的CDN资源，通常被称之为“另一朵云”的边缘基础设施；也有浩如烟海的IoT的边缘设备，算力规模小但是数量庞大。在处理IoT场景云原生转型问题上，轻量化是绕不开的一环。我们知道，IoT业务场景充斥着大量的异构、小规格的边缘算力，像各种智能终端、设备，它们对资源的约束是极致的，难于接受额外的过多资源占用。所以，首当其冲的必然是管控层面的轻量化，简单介绍下目前常见的技术尝试：

- 管控组件的轻量化替代和压缩，如containerd替代docker，以及减少额外node sidecar的部署和开销的方案等等；
- 管控组件的裁剪，在k8s体系下对kubelet相关功能的裁剪和模块化，社区也有类似方案；
  ![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwQEDIsNLdHqZyJBgFMAzCZDHTQGdrqty2EQVFibU4cL2fSvTnjQicymvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 边缘k8s用在哪里

那么回头再看，似否还有一个问题我们还没有讨论到，边缘容器/k8s到底用在哪里？一言以蔽之，Edge kubernetes适用于需要通过中心统一管控远端机房、服务器、设备的场景，轻松实现云、边、端一体的应用交付、运维、管控能力。

- 在IOT领域，可用于工厂、仓库、楼宇、园区、高速收费站等场景。
- 在CDN领域，可以支持视频直播、视频监控、在线教育等行业客户就近计算和存储的需求。
- 在ENS边缘计算产品上，提供了针对ENS边缘实例的DevOps能力，方便针对边缘的应用分发及部署运维。

# 阿里云容器服务边缘托管ACK@Edge

人们常说“技术的发展有其自身的内在直接动力”，而阿里云边缘云原生的产品化落地动力确是实实在在的客户场景催生。大致分几个阶段：

- 众所周知，阿里云在两大边缘计算领域CDN和IoT深耕已久，边缘规模和业务复杂度的日益攀升带来了不小的效能问题。在云原生大有爆发之势的2018年，阿里云容器服务先是和IoT业务团队联合推出Kubernetes Edge定制版本，作为云上IoT边缘托管服务底座，支持海量边缘网关节点接入，深度融合IoT云端市场、云端FaaS、消息、运维等服务，`云边一体概念萌生`

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwGRGCwxREdovw6LVlh1dQ7icZZB5Naqp4lDvRhXcYUBiaJHJoiczDT5Hdg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在这个过程中，云原生让IoT托管如虎添翼，持续落地智慧楼宇、智慧工厂、智慧仓储项目，云边端分层结构也日益显现，这里和大家分享两个案例：亲橙里和XX仓储：

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLw62N9RLEYXdQgYYfg75gWHwZeUmgQBhhBd3yrGOjcSDqr9WzFtZJqHQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 随着IoT业务规模增长，Kubernetes  Edge定制版作为独立底座的维护成本越来越高，产品化势在必行。在2019年4月，阿里云容器服务ACK推出EdgeKubernetes的托管服务（ACK@Edge），就是上文提到的k8s的托管服务，只不过兼具了满足边缘计算场景的能力补齐，云边一体威力初现。

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLwP1WbbeHOGnzqJvRtKKe5msh7w0A73ZVfUqzWDdxjpfGC2FnVbM3EgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而这一次，ACK@Edge又在产品化之初选择和CDN联姻。阿里云有着国内最大体量的CDN业务，并正在从以内容分发服务为主转变为边缘计算，以ACK@Edge为依托构建云原生的EdgePaaS服务，而CDN的海量节点也考验了
ACK@Edge的大规模服务能力。

- 随着ACK@Edge的技术成熟，越来越多的内外部客户选择云边一体的云原生标准k8s托管服务。这里有一个优酷的case，优酷是国内最大的视频平台，随着业务的快速发展，需要将原来部署在若干 IDC 内的集中式架构，演进到云+边缘计算的架构。这就势必需要业务方能够有一种方式来统一管理阿里云十几个 region  和众多的边缘节点。通过边缘k8s架构，统一管理云与边缘的节点，实现了应用发布和弹性扩缩容的统一。通过弹性能力，节省了机器成本  50%以上；用户终端就近访问边缘节点，让端到端网络延迟降低了 75%

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KcZRDicjWo9HJHYnP5yanKLw3Ufr28bdhbibuAbh10KYYpTBG5QbPOHBbGibibaoBsHiboWPptXSic3namw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 边缘云原生未来

云原生的未来是共同的期待和技术演进，在消除边缘和云的差异之后，云边一体的架构体系会在未来将边缘计算牢牢坚定在“云计算新边界”的理念之上。新业务的开展将和云上保持高度一致：serverless，安全沙箱技术，函数计算等新的业务形态都将在不远的将来落地。ACK@Edge将和大家一起见证。