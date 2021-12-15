Serverless 时代 DevOps 的最佳打开方式

作者 | 许成铭（竞霄）

DevOps 简析





传统软件开发过程中，开发和运维是极其分裂的两个环节，运维人员不关心代码是怎样运作的，开发人员也不知道代码是如何运行的。

而对于互联网公司而言，其业务发展迅速，需要快速更新以满足用户差异化的需求或者竞对的产品策略，需要进行产品的快速迭代，通过小步快跑的方式进行敏捷开发。

对于这种每周发布 n 次甚至每天发布 n 次的场景，高效的协作文化就显得尤为重要。DevOps 就在这种场景下应运而生，它打破了开发人员和运维人员之间的壁垒。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5j3YR5IslqdjibIJD5OSjWhfoBaBpibDfetYlhok7MBdxHbH1aSjtnJQA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

DevOps 是一种重视“软件开发人员（Dev）”和“IT 运维技术人员（Ops）”之间沟通合作的文化、运动或惯例。通过自动化“软件交付”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5ibVPHK8l4egFq7PYxNf3l5FyKWge6OQLZRThAZtwyPCUnNaFeQZglmQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



上图是一个完整的软件开发生命周期，DevOps 运动的主要特点是倡导对构建软件的整个生命周期进行全面的管理。



**DevOps 工程师的职责：**

- 管理应用的全生命周期，比如需求、设计、开发、QA、发布、运行；
- 关注全流程效率提升，挖掘瓶颈点并将其解决；
- 通过标准化、自动化、平台化的工具来解决问题。



**DevOps 的关注点在于缩短开发周期，增加部署频率，更可靠地发布。**通过将 DevOps 的理念引入到整个系统的开发过程中，能够显著提升软件的开发效率，缩短软件交付的周期，更加适应当今快速发展的互联网时代。





# Serverless 简析



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5g0qCYxg4OFr97cYe4o064uQjBc1pcgGToRlPhc4uAMribIgQF9QOHuA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



上图左侧是谷歌趋势，对比了 Serverless 和微服务的关键词趋势走向，可看出随着时间变化，**Serverless 的热度已经逐渐超过微服务**，这说明全世界的开发人员及公司对 Serverless 非常青睐。



那 Serverless 究竟是什么？上图右侧是软件逻辑架构图，有开发工程师写的应用，也有应用部署的 Server（服务器），还有 Server  的维护操作，比如资源申请、环境搭建、负载均衡、扩缩容、监控、日志、告警、容灾、安全、权限等。而 Serverless 实际上是把 Server  的维护工作屏蔽了，对于开发者是黑盒，这些工作都由平台方支持，对业务来说只需关注核心逻辑即可。



总得来说，Serverless 架构是“无服务器”架构，是云计算时代的一种架构模式，能够让开发者在构建应用的过程中无需关注计算资源的获取和运维，降低运营成本，缩短上线时间。



Serverless 时代 DevOps 的变化



#  

## **1. Serverless 的特性**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5ZIFiaxIufZ3icZGQibhGzINPK2FzeicibsTXwqwvw58HEIAePZTe7zkH2WQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



上图左侧为 2020 年中国云原生用户调查报告中 Serverless 技术在国内的采用情况，图中显示近三成用户已经把 Serverless  应用在生产环境中，16% 的用户已经将 Serverless 应用在核心业务的生产环境，12% 的用户也已经在非核心业务的生产环境中用到  Serverless，可见国内对 Serverless 接受度较高。



上图右侧为咨询公司 O'Reilly 对全球不同地区不同行业的公司进行的调查报告结果，图中显示一马当先使用 Serverless 架构的就是 DevOps 人员。



那么当 Serverless 遇上 DevOps，会发生哪些变化呢？首先我们看一下云原生架构白皮书中对 Serverless 特性的归纳总结：



- **全托管的计算服务**  



用户只需要编写自己的代码来构建应用，无需关注同质化的复杂的基础设施的开发运维工作。



- **通用性**



能够在云上构建普适的各种类型应用。



- **自动的弹性伸缩**



用户无需对资源进行预先的容量规划，业务如果有明显的波峰波谷或临时容量需求，Serverless 平台都能够及时且稳定地提供对应资源。



- **按量计费**



企业可以使成本管理更加有效，不用为闲置资源付费。



Serverless 让运维行为对开发透明，开发人员只需关注核心业务逻辑的开发，进而精益整个产品开发流程，快速适应市场变化。而上述 Serverless 的这些特性与 DevOps 的文化理念及目标是天然契合的。

 

## **2. Serverless 开发运维体验**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC52IRvOdmQBcicauY2WMCwpMicylQVxhgryAvQdSAtAnvuJbtMl5w3qcRQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



传统应用构建的流程中，DevOps 人员管理整个生命周期的步骤非常多：



- 在资源准备阶段，要购买 ECS 进行机器初始化等系列操作；
- 在研发部署阶段，需要把业务应用、监控系统、日志系统等旁路系统部署在 ECS 上；
- 在运维阶段，你不仅需要运维自己的应用，还需运维 Iaas 及其他旁路的监控、日志、告警组件。



而如果迁到 Serverless，其开发体验是怎么样的呢？



- 在资源准备阶段，不需要任何资源准备，因为 Serverless 是按需使用、按量付费的，不用关注底层 Server；
- 在研发部署阶段，只需要将自己的业务部署到对应的 Serverless 平台上；
- 在运维阶段，彻底做到了免运维。



可以看到，传统应用构建流程中的 Iaas 及监控、日志、告警，在 Serverless 上完全没有，它以全托管、免运维的形式展现给用户。



Serverless 时代 DevOps 最佳实践





上文介绍的体验其实就是基于阿里云的一款 Serverless 产品——SAE 来实现的。Serverless 应用引擎（SAE）是阿里云 Serverless 产品矩阵中提供的 DevOps 最佳实践。先简单介绍一下 SAE：



## **1. Serverless 应用引擎（SAE）**



SAE 是一款面向应用 Serverless PaaS 平台，支持 Spring Cloud、Dubbo、HSF  等主流的应用开发框架。用户可以零代码改造，直接将应用部署到 SAE 上，并且按需使用、按量付费、秒级弹性，可以充分发挥 Serverless  的优势，为用户节省闲置的资源成本。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5fI3QAdpiaWCRrRM6WAh4x5eh6tAw0TL9ZicUzZfufFl6IAZDMeII1FqA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



在体验上，SAE 采用全托管、免运维的方式，用户可以聚焦于核心的业务逻辑开发，而应用的整个生命周期管理，如监控、日志、告警，这些都由 SAE  完成。可以说，SAE 提供了一个成本更优、效率更高的一站式应用托管方案，用户可以做到零门槛、零改造、零容器基础就可以享受到 Serverless 带来的技术红利。



Serverless 应用引擎（SAE）三大特点：



- **0 代码改造**：微服务无缝迁移，开箱即用，支持 War/Jar 自动构建镜像；
- **15s 弹性效率**：应用端到端快速扩容，应对突发流量；
- **57% 降本提效**：多套环境按需启停，降本且提效。



## **2. 构建高效闭环的 DevOps 体系**



SAE 内构建了高效闭环 DevOps 体系，覆盖开发态、部署态和运维态的整个过程。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5kicwdpZRz3VGhXia1fmo4ZcEx7rGASzBB1ruVRibPficC8OA0iaRdgHCLTQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



中大型企业一般都使用企业级的 CICD 工具（如 Jenkins 或云效）部署到 SAE，从而完成从源码到构建再到部署的整个流程。



对于个人开发者或者中小企业，更倾向于使用 Maven 插件或 IDEA 插件一键部署到云端，方便本地调试，也提升了整个的用户体验。



当部署到 SAE 之后，可以进行可视化的智能运维操作，比如高可用运维（服务治理、性能压测、限流降级等）、应用诊断（线程诊断、日志诊断、数据库诊断等）以及数据化运营。以上操作都是部署到 SAE 之后，用户可以开箱即用的现成的功能。



用户通过 SAE 可以非常方便地实现整体的开发运维流程，感受 Serverless 带来的全方位体验和效率上的提升。下面介绍几个 SAE 的最佳实践：



## **3. 部署态最佳实践：CICD**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5lDIUibaibRtonicvMgqls3icuHic7dIzia6fHOurelbMPnNwRGeXDnXPHvoQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**SAE 目前支持三种部署方式，分别是 War、Jar 和镜像。**



如果用户使用 Spring Cloud、Dubbo 或 HSF 这类应用，可以直接打包或者填写对应的 URL 地址，就可以直接部署到 SAE 上。而对于非  Java 语言的场景，可以通过镜像方式进行部署。后续我们也会支持其他的语言包以自动化构建的方式进行部署。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5oBicdlvwuPkTwHZfDG1TzfOFlhYtxibEunDBnWBicJJ4Xzia4QfKjy6Pgg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**除了直接部署之外，SAE 也支持本地部署、云效部署和自建部署这三种方式。**



本地部署依赖 CloudToolkit 插件，对 IDEA/Eclipse 进行了支持，用户可以在 IDEA 里一键部署到 SAE 上，无需登录，方便地进行自动化操作。



云效部署是阿里云提供的企业级一体化 CICD 平台型产品，通过云效可以监听代码库的变动，如果进行 Push  操作，就会触发云效的整个发布流程。比如进行代码检查或者单元测试，在对这个代码进行编译、打包、构建，构建好后会生成对应的构建物，之后它会调用  SAE 的 API，然后执行整体的部署操作。这一整套流程也是开箱即用的，用户只需要在云效控制台上进行可视化配置就可以把整个流程串起来。



自建部署指用户的公司如果是直接通过 Jenkins 进行构建的话，也可以直接使用 SAE。Jenkins 作为开源的最大的 CICD 平台，我们也提供了有力支持，许多用户也通过 Jenkins 成功地部署到 SAE 上。

 

## **4. 部署态最佳实践：应用发布三板斧**



**应用发布三板斧包括：可灰度、可监控、可回滚。**在阿里内部所有的变更都需要严格做到上述的“三板斧”，而 SAE 作为一款云产品，也是把阿里巴巴的最佳实践对外输出进行产品化的集成。



- **可灰度**：支持单批、分批、金丝雀等多种发布策略；支持按流量灰度，批次间自动/手动发布，分批间隔等多种发布选项；
- **可监控**：发布过程中清晰对比不同批次基础监控与应用监控指标异动，及时暴露问题，定位变更风险；
- **可回滚**：在发布过程中，允许人工介入控制发布流程，如异常中止、一键回滚。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5u74CeAecibEsOiaRZHcNE3vpjickmXBWib3fl4nGNBicQCA7QUgJWLq7iaiaw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5Q3I0E2nkic4m1MvpCYBYbS5Y3CS1RN3icrDqGibcGfzsOb5nL1tbzcBOg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



上图为控制台截图，可以看到在部署上我们支持单批、分批和灰度三种方式进行发布。



执行发布的过程都是通过发布端进行，每个发布端都有具体的步骤，首先进行构建镜像，然后初始化环境，接着创建和更新部署配置。用户可以清晰地看到发布端当前的运行进度与状态，方便排查。

 

## **5. 运维态最佳实践：全方位可观测**



SAE 提供全方位可观测，可以对分布式系统中的任何变化进行观测。当系统出现问题时，可以便捷地定位问题、排查问题、分析问题；当系统平稳运行时，也可以提前暴露风险，预测可能出现的问题。通过 SAE 用户可以对自己的应用了如指掌。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC55z0t0Ie4icfuT96oic4uxH1dIVCpiaJ2RjIG1IiaELqmBgyCwT6ibAhSVwA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这里列举了可观测性的三个方面：Metrics、Logging 、Tracing。



- **Metrics**



代表聚合的数据，SAE 提供如下基础监控指标：



1）基础监控：CPU、MEM、Load、Network、Disk、IO；

2）应用监控：QPS、RT、异常数、HTTP 状态码、JVM 指标；

3）监控告警：丰富的告警源上报、告警收敛处理、多种告警渠道触达（如邮箱、短信、电话等）。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5YCaa1X4llR2rm8hicQAVpzu3TibTbeibuKu9Pic2iaYs38zgvf5Tu3Pssdg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

- **Logging**



代表离散的数据，提供以下功能：



1）实时日志：Stdout、Stderr 实时查看；

2）文件日志：自定义采集规则、持久化存储、高效查询；

3）事件：发布单变更事件、应用生命周期事件、事件通知回调机制。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5QUIFqvxK2ZclDTKcps4tcFiaRAsUAhrlkfeOS7C3icFGpP17HI4eemxg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

- **Tracing**



意味着可以按请求维度进行排查，提供如下开箱即用的功能：



1）请求调用链堆栈查询；

2）应用拓扑自动发现；

3）常用诊断场景的指标下钻分析；

4）事务快照查询；

5）异常事务和慢事务捕捉。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5PZabckzCZEU2SHt9RyUyfBYiaibS3Sb2asdxJbS7lSkS2RFBJ8VibkgpQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## **6. 运维态最佳实践：在线调试**



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5JXcW4wzeE0T8eX56gvLWYEYPO35w4voichPMXG51kw7gG2F0U3Sn3uw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



通过 SAE 在线调试可以访问单实例的目标端口，相当于用户在本地可以直接访问云端某个应用的某个具体实例，原理是为实例提供了端口映射的 SLB，通过这个能力用户可以实现如下功能：



- **SSH / SFTP 访问实例**



可以在本地通过 SSH 直接连到应用的具体的实例上，或者通过 SFTP 进行上传/下载文件。



- **Java retmote debug**



相当于在 IDEA 里配置一个断点，再远程连接到对应的 SAE 的实例上，这样就可以通过断点来查看整个方法的调用站与上下文信息，对线上正在运行的应用进行诊断。



- **其他诊断工具连接实例**



其他诊断工具也可以通过在线调试的手段连接到 SAE 的实例上，进而看到 Java 的一些信息，比如堆栈或者线程等。

 

**适用场景：**针对运行时在线应用的实时观测运维及问题排查求解。

 

## **7. 开发态最佳实践：端云联调**



针对微服务场景，我们提供了一个非常好用的能力：端云联调。它基于 CloudToolkit 插件+ 跳板机，**可以实现：**



1）本地服务订阅并注册到云端 SAE 内置的注册中心；

2）本地服务可以和云端 SAE 服务互相调用。

 

**适用场景：**



1）微服务应用迁移到云端 SAE，迁移过程中的开发联调；

2）本地开发测试验证。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/hIibsapxkj1stwdvNtUhANzhF6YHmdeC5bMnNLPoZaiaQRerAGOq4wULz00Vq0zkRbicqhfoMjExZ3qoq1aW0GHoA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这个功能的原理是需要在用户的 VPC 内，然后通过 ECS 代理服务器作为跳板机，ECS 可以和同一个 VPC 内的 SAE 应用进行互调，然后这台 ECS 通过反应代理的方式，可以与本地进行连接。



CloudToolkit 插件会在应用启动时就注入对应 SAE 注册中心的地址，以及微服务的一些上下文参数，使得用户本地的应用通过跳板机连到 SAE 的应用上，从而进行整个端云联调的过程。