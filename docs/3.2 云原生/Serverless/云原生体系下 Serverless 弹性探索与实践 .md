- [云原生体系下 Serverless 弹性探索与实践](https://mp.weixin.qq.com/s?__biz=MzI4NzI5MDM1MQ==&mid=2247491970&idx=1&sn=ecff506c137365a4259082a6c888810d&chksm=ebcd4702dcbace147bf39f42379cc321ba2024a0957db8aedd4ec43189a3dfa41f8032321d01&mpshare=1&scene=24&srcid=0810KMOdNJaftxtyCv8vbQTE&sharer_sharetime=1628589696571&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)



# Serverless 时代的来临

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOf1uPnOsoWoNgibzVBicRzicWwTavcEzG4jChMia5u9E2OjZGniarnaft9Sw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Serverless 顾名思义，是一种 “无服务器” 架构，因为屏蔽了服务器的各种运维复杂度，让开发人员可以将更多精力用于业务逻辑设计与实现。在  Serverless  的架构下，开发者只需要关注于上层应用逻辑的开发，而诸如资源申请，环境搭建，还有负载均衡，扩缩容等服务器相关的复杂操作都可以由平台来进行维护。在云原生架构白皮书中，对 Serverless 的特性有以下概括：

- 全托管的计算服务，客户只需要编写代码构建应用，无需关注同质化的、负担繁重的基于服务器等基础设施的开发、运维、安全、高可用等工作；

- 通用性，能够支撑云上所有重要类型的应用；

- 自动的弹性伸缩，让用户无需为资源使用提前进行容量规划；

- 按量计费，让企业使用成本得有效降低，无需为闲置资源付费。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOjR6bZHAibv5r0VpVr3tweHOIc6PqEsm4YV8rN9hcXlW4fj2p8uibgnVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

回顾整个 Serverless 的发展历程，我们可以看到从 2012 年首次提出 Serverless 概念为起点，再到 AWS 推出 Lambda  云产品的这段时间内，人们对 Serverless 的关注度出现了爆发式的增长，对无服务器的期待和畅想逐渐引爆整个行业，但 Serverless  的推广和生产落地的过程却不容乐观，Serverless 理念与实操生产的过程中存在 Gap ，挑战着人们固有的使用体验和习惯。阿里云坚信  Serverless 将作为云原生之后确定性的发展方向，相继推出了 FC, SAE 等多款云产品来覆盖不同领域，不同类型的应用负载来使用  Serverless 技术，并且不断在推进整个 Serverless 理念的普及与发展。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOhCAaORLYicwhMK7aZKwKricyj3HGSWsL50768CBtiarVBZLQF534qYiaoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

就当前 Serverless 整个市场格局而言，阿里云已经做到了 Serverless 产品能力中国第一，全球领先，在去年 Forrester  评测魔力象限中可以明显的看到阿里云在 Serverless 领域已经与 AWS 不相上下，于此同时，阿里云 Serverless  用户占比中国第一，在 2020 年中国云原生用户调研报告中整个阿里云 Serverless 用户占比已经达到了 66%，而在  Serverless 技术采用情况的调研中表明，已经有越来越多的开发者和企业用户将 Serverless  技术应用于核心业务或者将要应用于核心业务之中。

# Serverless 弹性探索

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOLvuQOu0fGBDic5pAcY2SXtH6IKdqJoAcqnMshiaMGwiciagVlXJHvroCmw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



弹性能力作为云的核心能力之一，所关注的问题是容量规划与实际集群负载间的矛盾，通过两幅图的对比可以看到，如果采用预先规划的方式进行资源安排，会由于资源准备量和资源需求量的不匹配导致资源浪费或者资源不足的情况，进而导致成本上的过多开销甚至业务受损，而我们期望极致弹性能力，是准备的资源和实际需求的资源几乎匹配，这样使得应用整体的资源利用率较高，成本也随业务的增减和相应的增减，同时不会出现因容量问题影响应用可用性的情况，这就是弹性的价值。弹性其实现上分为可伸缩性和故障容忍性,可伸缩性意味着底层资源可以参照指标的变化有一定的自适应能力，而故障容忍性则是通过弹性自愈确保服务中的应用或实例处于健康的状态。上述能力带来的价值收益在于降成本的同时提升应用可用性，一方面，资源使用量贴合应用实际消耗量，另一方面，提升峰值的应用可用性，进而灵活适应市场的不断发展与变化。

**下面将对当前较为普遍的三种弹性伸缩模式进行阐述和分析。**

## IaaS弹性伸缩

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOXia4SspS3JM6ChQjo6QgH7xgNj9Az2H7oaNkBxGfIMgeG9YXdjVIKmg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先是 IaaS 弹性伸缩，其代表产品是各云厂商云服务器弹性伸缩，如阿里云 ESS,可以通过配置云监控的告警规则来触发相应的 ECS  增减操作，同时支持动态增减 SLB 后端服务器和 RDS 白名单来保证可用性，通过健康检查功能实现弹性自愈能力。ESS  定义了伸缩组的概念，即弹性伸缩的基本单位，为相同应用场景的 ECS 实例的集合及关联  SLB,RDS,同时支持多种伸缩规则，如简单规则，进步规则，目标追踪规则，预测规则等，用户的使用流程为创建伸缩组和伸缩配置，创建伸缩规则，监控查看弹性执行情况。

## kubernetes弹性伸缩（水平）

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOqRNjZ7fqjbU23L7IM5CibHZOtmJIKDU918QeLEibJA7icsUmaFgXiaVJgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Kubernetes 弹性伸缩，这里主要关注于水平弹性 HPA，其代表产品为 K8s 以及其所对应的托管云产品，如阿里云容器服务，K8s  做为面向应用运维的基础设施和 Platform for  Platform,提供的内置能力主要是围绕着容器级别的管理和编排来展开的,而弹性能力聚焦于对底层 pod 的动态水平伸缩，K8s HPA  通过轮询pod的监控数据并将它与目标期望值比较进行，通过算法实时计算来产生期望的副本数，进而对 Workload  的副本数进行增减操作，用户在实际使用上需要创建并配置对应的指标源和弹性规则以及对应的 Workload，可以通过事件来查看弹性的执行情况。

## 应用画像弹性伸缩

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOXkh1XeHdiagFhFpCKkODW2libiaWSvhqCghJQTTRYEp9QKojlnnHqcusg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后介绍一下应用画像弹性伸缩，其主要用于互联网公司内部，如阿里 ASI 容量平台。容量平台提供容量预测服务和容量变更决策服务，指导底层容量变更组件如 AHPA/VPA  实现容量弹性伸缩，并根据弹性结果修正容量画像。以画像驱动为主 + 指标驱动为辅实现弹性伸缩能力，通过提前伸缩 +  实时修正来降低弹性伸缩风险。整个弹性伸缩会借助 ODPS  和机器学习能力对实例监控等数据进行处理并产生应用画像，如基准画像，弹性画像，大促画像等，并借助容量平台来完成画像注入，变更管控和故障熔断等操作。用户使用流程为应用接入，基于历史数据/经验生成对应的容量画像，实时监控指标修正画像，并监控查看弹性执行情况。

## 弹性模式



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXO6PCPT4hToBaJQXLR2v62GmuFUMZIm2tFQDh8Fic8uUXTNicToM4usnHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从对比可以看出各产品弹性伸缩功能模式上从抽象来讲基本相同，均由触发源，弹性决策和触发动作组成，触发源一般依赖外部监控系统，对节点指标，应用指标进行采集处理，弹性决策一般基于周期性轮询并算法决策，有部分基于历史数据分析预测以及用户定义的定时策略，而触发动作为对实例进行水平扩缩，并提供变更记录与对外通知。各个产品在此基础上做场景丰富度，效率，稳定性的竞争力，并通过可观测能力提升弹性系统的透明度，便于问题排查和指导弹性优化，同时提升用户使用体验与粘性。

## 弹性发展方向

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXO8EFo6GOG5ibxagGS9EeR6fBJh1lVyhrt2HibFcIEzeHD4Nn5o8yXIDUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

各产品弹性伸缩模型也存在这一定的差异，对于 IaaS 弹性伸缩，其作为老牌弹性伸缩能力，沉淀时间长，功能强大且丰富，云厂商间能力趋于同质化。弹性效率相较容器受限，且强绑定各自底层  IaaS 资源。Kubernetes  作为开源产品，通过社区力量不断优化迭代弹性能力和最佳实践，更符合绝大部分开发运维人员诉求。对弹性行为和API  进行高度抽象，但其可扩展性不强，无法支持自定义需求。而应用画像弹性伸缩具有集团内部特色，根据集团应用现状和弹性诉求进行设计，且更聚焦于资源池预算成本优化，缩容风险，复杂度等痛点。不易拷贝扩展，特别对于外部中小客户不适用。

**从终态目标上，可以看出公有云与互联网企业方向的不同：**

- 互联网企业往往由于其内部应用具有显著流量特征，应用启动依赖多，速度慢，且对整体资源池容量水位，库存财务管理，离在线混部有组织上的诸多诉求，因而更多的是以容量画像提前弹性扩容为主，基于 Metrics 计算的容量数据作为实时修正，其目标是容量画像足够精准以至于资源利用率达到预期目标。
- 公有云厂商服务于外部客户，提供更为通用，普适的能力，并通过可拓展性满足不同用户的差异化需求。尤其在 Serverless 场景，更强调应用应对突发流量的能力，其目标在于无需容量规划，通过指标监控配合极致弹性能力实现应用资源的近乎按需使用且整个过程服务可用。

# Serverless 弹性落地

## Serverless应用引擎简介

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOiaBQlfJRZmOYVLnnp0dSqRKzPyXOicG1XYRC2bJBUTjm5Ly2ibY2icltibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Serverless 作为云计算的最佳实践、云原生发展的方向和未来演进趋势，其核心价值在于快速交付、智能弹性、更低成本。

在时代背景下，SAE 应运而生，SAE 是一款面向应用的 Serverless PaaS 平台，支持 Spring Cloud、Dubbo  等主流开发框架，用户可以零代码改造直接将应用部署到 SAE，并且按需使用，按量计费，可以充分发挥 Serverless  的优势为客户节省闲置资源成本，同时体验上采用全托管，免运维的方式，用户只需聚焦于核心业务开发，而应用生命周期管理，微服务管理，日志，监控等功能交由 SAE 完成。

## 技术架构

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOaU18ibUJM4pV0MBRcrRtgeu3mueyYv2U6xjepztdLpYHMVskVwFbqfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



SAE 的技术架构如图所示，上层 Runtime 分为网关路由，应用生命周期管理与商业化，镜像构建，定时任务，集群代理等多个模块。底层  Infra为多租 Kubernetes，使用神龙裸金属安全容器、VK 对接 ECI 两种方式提供集群计算资源。用户在 SAE  中运行的应用会映射到 Kubernetes 中相应的资源。其中多租能力是借助系统隔离、数据隔离、服务隔离和网络隔离实现租户间的隔离。

## 弹性效率：分析

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOdovlA9c31UpavQ5VozDxMbyiczMa5JxibMN1mVWicWdCtlocLdDwqjZPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

弹性的竞争力主要在于场景丰富度，效率，稳定性的竞争力，先讲一下 SAE 在弹性效率上的优化。

通过对 SAE 应用的整个生命周期进行数据统计和可视化分析，其包含调度，init container  创建，拉取用户镜像，创建用户容器，启动用户容器和应用这几个阶段，示意图中对其耗时的占比进行了简化。我们可以看到整个应用生命周期耗时集中于调度，拉取用户镜像，应用冷启动这几个阶段。针对于调度阶段，其耗时主要在于 SAE 当前会执行打通用户 VPC  操作，由于该步骤强耦合于调度，本身耗时较长，且存在创建长尾超时，失败重试等情况，导致调度链路整体耗时较长。由此产生的疑问是可否优化调度速度？可否跳过调度阶段 ?  而对于拉取用户镜像，其包含拉取镜像与解压镜像的时长，特别是在大容量镜像部署的情况下尤为突出。优化的思路在于拉取镜像是否可以优化使用缓存，解压镜像是否可以优化。而对于应用冷启动，SAE 存在大量单体和微服务的 JAVA 应用，JAVA  类型应用往往启动依赖多，加载配置慢，初始化过程长，导致冷启动速往往达到分钟级。优化的方向在于可否避免冷启动流程并使用户尽量无感，应用无改造。

## 弹性效率：原地升级

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOqFRqdJ6rDP3MoPgpAmTCK5rlMR0WJfFqOGRGdtQXoFFqdUN4sibAqkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先 SAE 采用了原地升级能力，SAE 起初使用了 K8s 原生的 Deployment 滚动升级策略进行发布流程，会先创建新版本  Pod，再销毁旧版本 Pod 进行升级，而所谓原地升级，即只更新 Pod 中某一个或多个容器版本、而不影响整个 Pod  对象、其余容器的升级。其原理是通过 K8s patch 能力，实现原地升级 Container，通过 K8s readinessGates  能力，实现升级过程中流量无损。原地升级给 SAE 带来了诸多价值，其中最重要的是避免重调度，避免 Sidecar  容器（ARMS,SLS,AHAS）重建,使得整个部署耗时从消耗整个 Pod 生命周期到只需要拉取和创建业务容器，于此同时因为无需调度，可以预先在 Node 上缓存新镜像，提高弹性效率。SAE 采用阿里开源 Openkruise 项目提供的 Cloneset  作为新的应用负载，借助其提供的原地升级能力，使得整个弹性效率提升 42%。

## 弹性效率：镜像预热

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOh4Nf3cfG1xfEWmC6aRLHRlfiaic4uibTnmub4CnWKsAIHpJXfA1L6uXbg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同时 SAE 采用了镜像预热能力，其包含两种预热形式：调度前预热，SAE  会对通用的基础镜像进行全节点缓存，以避免其频繁的从远端进行拉取。与此同时对于分批的场景支持调度中预热，借助 Cloneset  原地升级能力，在升级的过程中可以感知到实例的节点分布情况，这样就可以在第一批部署新版本镜像的同时，对后面批次的实例所在节点进行镜像预拉取，进而实现调度与拉取用户镜像并行。通过这项技术，SAE 弹性效率提升了 30%。

## 弹性效率：镜像加速

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOiamTntPu5DzYUa4wUoE0tf8lJTyfNlcL6hBmzgPvcDoK6bJYTGp7wrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

刚才讲述的优化点在于拉取镜像部分，而对于解压镜像，传统容器运行需要将全量镜像数据下载后再解包，然而容器启动可能仅使用其中部分的内容，导致容器启动耗时长。SAE 通过镜像加速技术，将原有标准镜像格式自动转化为支持随机读取的加速镜像，可以实现镜像数据免全量下载和在线解压，大幅提升应用分发效率，同时利用  Acree 提供的 P2P 分发能力也可以有效减少镜像分发的时间。

## 弹性效率：Dragonwell 11

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOw1g3UV8uXDFLrUYAcPn1Pdet1OtqdO4b7Mywg7FZAdnacibRLGsYeHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于 Java 应用冷启动较慢的痛点，SAE 联合 Dragonwell 11 提供了增强的 AppCDS 启动加速策略，AppCDS 即  Application Class Data Sharing，通过这项技术可以获取应用启动时的 Classlist 并 Dump  其中的共享的类文件，当应用再次启动时可以使用共享文件来启动应用，进而有效减少冷启动耗时。映射到 SAE  的部署场景，应用启动后会生成对应的缓存文件在共享的 NAS 中，而在进行下一次发布的过程中就可以使用缓存文件进行启动。整体冷启动效率提升  45%。

## 弹性效率：自动扩缩

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOYjnpGMcjibJX2rKea14wzbuGa1QqSx4icpjZdQTeVw23H43MAdC5e9nA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

除了对整个应用生命周期的效率进行优化外，SAE  也对弹性伸缩进行了优化，整个弹性伸缩流程包括弹性指标获取，指标决策以及执行弹性扩缩操作三部分。对于弹性指标获取，基础监控指标数据已经达到了秒级获取，而对于七层的应用监控指标， SAE 正在规划采用流量透明拦截的方案保证指标获取的实时性。而弹性决策阶段，弹性组件启用了多队列并发进行  Reconcile，并实时监控队列堆积，延时情况。

## 弹性能力：架构

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXO3GIq9hnFjefXjILDWA8MpR1pLWNZe53xibWJOGOS3AMCdYP9dgQnAGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



SAE 弹性伸缩包括强大的指标矩阵，丰富的策略配置，完善的通知告警机制及全方位可观测能力,支持多种数据源：原生的  MetricsServer，MetricsAdapter，Prometheus，云产品 SLS,CMS,SLB  以及外部的网关路由等，支持多种指标类型：CPU,MEM,QPS,RT,TCP 连接数，出入字节数，磁盘使用率，Java 线程数，GC  数还有自定义指标。对指标的抓取和预处理后，可以自定义配置弹性策略来适配应用的具体场景：快扩快缩，快扩慢缩，只扩不缩，只缩不扩，DRYRUN，自适应扩缩等。同时可以进行更为精细化的弹性参数配置,如实例上下限，指标区间，步长比例范围，冷却、预热时间，指标采集周期和聚和逻辑，CORN  表达式，后续也会支持事件驱动的能力。弹性触发后会进行对应的扩缩容操作，并通过切流保证流量无损，并且可以借助完善的通知告警能力（钉钉，Webhook  ,电话，邮件，短信）来实时触达告知用户。弹性伸缩提供了全方位的可观测能力，对弹性的决策时间，决策上下文进行清晰化展现，并且做到实例状态可回溯，实例 SLA 可监控。

## 弹性能力：场景丰富度

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOslDMHaDOzwaXU9HqV1gRWcGicW9lJCKicb5su96KyJaBlcT5dO6cia4Hw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

SAE 弹性能力在场景丰富度上也有着相应的竞争力，这里重点介绍一下 **SAE 当前支持的四种场景：**

- 定时弹性：在已知应用流量负载周期的情况下进行配置，应用实例数可以按照时间，星期，日期周期进行规律化扩缩，如在早 8 点到晚 8 点的时间段保持 10 个实例数应对白天流量，而在其余时间由于流量较低则维持在 2 个实例数甚至缩  0。适用于资源使用率有周期性规律的应用场景，多用于证券、医疗、政府和教育等行业。

- 指标弹性：可以配置期望的监控指标规则，SAE 会时应用的指标稳定在所配置的指标规则内，并且默认采用快扩慢缩的模式来保证稳定性。如将应用的cpu指标目标值设置为 60%，QPS 设置为  1000，实例数范围为 2-50。这种适用于突发流量和典型周期性流量的应用场景，多用于互联网、游戏和社交平台等行业。

- 混合弹性：将定时弹性与指标弹性相结合，可以配置不同时间，星期，日期下的指标规则，进而更加灵活的应对复杂场景的需求。如早 8 点到晚 8 点的时间段 CPU 指标目标值设置为 60%，实例数范围为 10-50，而其余时间则将实例数范围降为  2-5，适用于兼备资源使用率有周期性规律和有突发流量、典型周期性流量的应用场景，多用于互联网、教育和餐饮等行业。

- 自适应弹性：SAE 针对流量突增场景进行了优化，借助流量激增窗口，计算当前指标在这个时刻上是否出现了流量激增问题，并会根据流量激增的强烈程度在计算扩容所需的实例时会增加一部分的冗余，并且在激增模式下，不允许缩容。

## 弹性能力：稳定性

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOyBHf9lU5IQ3rPcJegU7YtwPZeNq85mWXMyicaVlsj8nqiat7UibKibhkUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



稳定性是 SAE 弹性能力建设的过程中非常重要的一环，保证用户应用在弹性过程中按照预期行为进行扩缩，并保证整个过程的可用性是关注的重点。SAE 弹性伸缩整体遵循快扩慢缩的原则，通过多级平滑防抖保证执行稳定性，同时对于指标激增场景，借助自适应能力提前扩容。**SAE 当前支持四级弹性平滑配置保证稳定性：**

- 一级平滑：对指标获取周期，单次指标获取的时间窗口，指标计算聚和逻辑进行配置
- 二级平滑：对指标数值容忍度，区间弹性进行配置
- 三级平滑：对单位时间扩缩步长，百分比，上下限进行配置
- 四级平滑：对扩缩冷却窗口，实例预热时间进行配置

# Serverless 弹性最佳实践

## 弹性最佳实践：弹性伸缩准备

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOWdToZqn8EeEtqEvVNcIqbrNatSev2vcWsK9wictGUBqP22sKvrgjrxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



SAE 弹性伸缩可以有效解决瞬时流量波峰到来时应用自动扩容，波峰结束后自动缩容。高可靠性、免运维、低成本的保障应用平稳运行，在使用的过程中建议遵循以下最佳实践进行弹性配置。

### 配置健康检查和生命周期管理

建议对应用健康检查进行配置，以保证弹性扩缩过程中的应用整体可用性，确保您的应用仅在启动、运行并且准备好接受流量时才接收流量 同时建议配置生命周期管理 Prestop，以确保缩容时按照预期优雅下线您的应用。

### 采用指数重试机制

为避免因弹性不及时，应用启动不及时或者应用没有优雅上下线导致的服务调用异常，建议调用方采用指数重试机制进行服务调用。

### 应用启动速度优化

为提升弹性效率，建议您优化应用的创建速度，可以从以下方面考虑优化：

- 软件包优化：优化应用启动时间，减少因类加载、缓存等外部依赖导致的应用启动过长

- 镜像优化：精简镜像大小，减少创建实例时镜像拉取耗时，可借助开源工具 Dive，分析镜像层信息，有针对性的精简变更

- Java 应用启动优化：借助 SAE 联合 Dragonwell 11 ，为 Java 11 用户提供了应用启动加速功能

## 弹性最佳实践：弹性伸缩配置

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXO4ewA5SXJAc1ca2c1iaQaMvndFecQF3JiafqE0bOIeicKdZQ2slZ1S5PicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### 弹性伸缩指标配置

弹性伸缩指标配置，SAE 支持基础监控，应用监控多指标组合配置，您可以根据当前应用的属性（CPU 敏感 /内存敏感 / io 敏感）进行灵活选择。

可以通过对基础监控和应用监控对应指标历史数据（ 如过去 6h, 12h, 1 天,7 天峰值，P99, P95 数值）进行查看并预估指标目标值，可借助 PTS  等压测工具进行压测，了解应用可以应对的并发请求数量、需要的 CPU 和内存数量，以及高负载状态下的应用响应方式，以评估应用容量峰值大小。

**指标目标值需要权衡可用性与成本进行策略选择，如：**

可用性优化策略 配置指标值为 40%

可用性成本平衡策略 配置指标值为 50%

成本优化策略 配置指标值为 70%

同时弹性配置应考虑梳理上下游，中间件，DB 等相关依赖，配置对应的弹性规则或者限流降级手段，确保扩容时全链路可以保证可用性。

在配置弹性规则后，通过不断监视和调整弹性规则以使容量更加接近应用实际负载。

### 内存指标配置

关于内存指标，考虑部分应用类型采用动态内存管理进行内存分配（如 Java Jvm 内存管理，Glibc malloc 和 Free  操作），应用闲置内存并没有及时释放给操作系统，实例消耗的物理内存并不会及时减少且新增实例并不能减少平均内存消耗，进而无法触发缩容，针对于该类应用不建议采用内存指标。

**Java 应用运行时优化：释放物理内存，增强内存指标与业务关联性**

借助 Dragonwell 运行时环境，通过增加 JVM 参数开启 ElasticHeap 能力，支持 Java 堆内存的动态弹性伸缩，节约Java进程实际使用的物理内存占用。

### 最小实例数配置

配置弹性伸缩最小实例数建议大于等于 2，且配置多可用区 VSwitch，防止因底层节点异常导致实例驱逐或可用区无可用实例时应用停止工作，保证应用整体高可用。

### 最大实例数配置

配置弹性伸缩最大实例数时，应考虑可用区 IP 数是否充足，防止无法新增实例。可以在控制台 VSwitch 处查看当前应用可用 IP，若可用 IP 较少考虑替换或新增 VSwitch。

## 弹性最佳实践：弹性伸缩过程

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOHXbsjGVWsGqMwWIvMNJHbrhBH41uMl73dkKCGpJheDdB7ABWfdACsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 弹性到达最大值

可以通过应用概览查看当前开启弹性伸缩配置的应用，并及时发现当前实例数已经到达峰值的应用，进行重新评估其弹性伸缩最大值配置是否合理。若期望最大实例数超过产品限制（当前限制单应用50实例数，可提工单反馈提高上限）

### 可用区再均衡

弹性伸缩触发缩容后可能会导致可用区分配不均，可以在实例列表中查看实例所属可

用区，若可用区不均衡可以通过重启应用操作实现再均衡。

### 自动恢复弹性配置

当进行应用部署等变更单操作时，SAE 会停止当前应用的弹性伸缩配置避免两种操作冲突，若期望变更单完成后恢复弹性配置，可以在部署时勾选系统自动恢复。

# 弹性最佳实践：弹性伸缩可观测

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOvvibeTAqia7L9oEkRqMsCqGb9LeL9N6icX3AHRaMoibHe2qSvIjWJv694Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 弹性历史记录

SAE 弹性生效行为当前可通过事件进行查看扩缩时间，扩缩动作，以及实时，历史决策记录和决策上下文可视化功能，以便衡量弹性伸缩策略的有效性，并在必要时进行调整。

### 弹性事件通知

结合钉钉，Webhook ,短信电话等多种通知渠道，便于及时了解弹性触发状况。

## 弹性最佳实践：客户案例

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uQEKexPbgIf3VA7hTryMXOKy44uvtribxlFpJFqUarjd3CmYGs7spHjncQjtF1d4TrO7HibHa2Z1AA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后分享一个采用 SAE 弹性伸缩功能的客户案例，在 2020 新冠疫情期间，某在线教育客户业务流量暴涨 7-8  倍，硬件成本和业务稳定性面临巨大风险。如果此时采用传统的 ECS  架构，客户就需要在非常短的时间内做基础设施的架构升级，这对用户的成本及精力都是非常大的挑战。但如果采用 SAE，用户 0 改造成本即可享受  Serverless 带来的技术红利，结合 SAE 的多场景弹性策略配置，弹性自适应和实时可观测能力，保障了用户应用在高峰期的业务  SLA，并且通过极致弹性效率，节省硬件成本达到 35%。

综上，弹性发展方向上，尤其是在 Serverless  场景，更强调应对突发流量的能力，其目标在于无需容量规划，通过指标监控配合极致弹性能力实现应用资源的近乎按需使用且整个过程服务可用。SAE  通过对弹性组件和应用全生命周期的不断优化以达到秒级弹性，并在弹性能力，场景丰富度，稳定性上具备核心竞争力，是传统应用 0 改造上  Serverless 的最佳选择。