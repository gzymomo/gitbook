[官方文档](https://kubesphere.com.cn/docs/quick-start/)

- [Linux 上的 All-in-one 安装](https://kubesphere.com.cn/docs/quick-start/all-in-one-on-linux/)

- [创建企业空间、项目、帐户和角色](https://kubesphere.com.cn/docs/quick-start/create-workspace-and-project/)




# KuberSphere简介

[KubeSphere](https://kubesphere.io/) 是在目前主流容器调度平台 [Kubernetes](https://kubernetes.io/) 之上构建的企业级分布式多租户容器平台，提供简单易用的操作界面以及向导式操作方式，在降低用户使用容器调度平台学习成本的同时，极大减轻开发、测试、运维的日常工作的复杂度，旨在解决 Kubernetes 本身存在的存储、网络、安全和易用性等痛点。除此之外，平台已经整合并优化了多个适用于容器场景的功能模块，以完整的解决方案帮助企业轻松应对敏捷开发与自动化运维、微服务治理、多租户管理、工作负载和集群管理、服务与网络管理、应用编排与管理、镜像仓库管理和存储管理等业务场景。

相比较易捷版，KubeSphere 高级版提供企业级容器应用管理服务，支持更强大的功能和灵活的配置，满足企业复杂的业务需求。比如支持 Master 和 etcd 节点高可用、可视化 CI/CD 流水线、多维度监控告警日志、多租户管理、LDAP 集成、新增支持 HPA (水平自动伸缩) 、容器健康检查以及 Secrets、ConfigMaps 的配置管理等功能，新增微服务治理、灰度发布、s2i、代码质量检查等，后续还将提供和支持多集群管理、大数据、人工智能等更为复杂的业务场景。



## 1.1 功能介绍

KubeSphere®️ 作为企业级的全栈化容器平台，为用户提供了一个具备极致体验的 Web 控制台，让您能够像使用任何其他互联网产品一样，快速上手各项功能与服务。KubeSphere 目前提供了工作负载管理、微服务治理、DevOps 工程、Source to Image、多租户管理、多维度监控、日志查询与收集、告警通知、服务与网络、应用管理、基础设施管理、镜像管理、应用配置密钥管理等功能模块，开发了适用于适用于物理机部署 Kubernetes 的 [负载均衡器插件 Porter](https://github.com/kubesphere/porter)，并支持对接多种开源的存储与网络方案，支持高性能的商业存储与网络服务。

![产品功能 - 图1](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/2b5f8f7b9635698e5a0579f5aa758bc1.png)

以下从专业的角度为您详解各个模块的功能服务：

### Kubernetes 资源管理

对底层 Kubernetes 中的多种类型的资源提供极简的图形化向导式 UI 实现工作负载管理、镜像管理、服务与应用路由管理 (服务发现)、密钥配置管理等，并提供弹性伸缩 (HPA) 和容器健康检查支持，支持数万规模的容器资源调度，保证业务在高峰并发情况下的高可用性。

![产品功能 - 图2](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/2a1ef9f86170016b9f391232b0d5c072.png)

### 微服务治理

- 灵活的微服务框架：基于 Istio 微服务框架提供可视化的微服务治理功能，将 Kubernetes 的服务进行更细粒度的拆分
- 完善的治理功能：支持熔断、灰度发布、流量管控、限流、链路追踪、智能路由等完善的微服务治理功能，同时，支持代码无侵入的微服务治理

![产品功能 - 图3](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/a2f9b7a1e8d1db35389b90b15c89b03f.png)

### 多租户管理

- 多租户：提供基于角色的细粒度多租户统一认证与三层级权限管理
- 统一认证：支持与企业基于 LDAP / AD 协议的集中认证系统对接，支持单点登录 (SSO)，以实现租户身份的统一认证
- 权限管理：权限等级由高至低分为集群、企业空间与项目三个管理层级，保障多层级不同角色间资源共享且互相隔离，充分保障资源安全性

![产品功能 - 图4](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/675e7f0d3774e4871734b69f6e769fdb.png)

### DevOps 工程

- 开箱即用的 DevOps：基于 Jenkins 的可视化 CI / CD 流水线编辑，无需对 Jenkins 进行配置，同时内置丰富的 CI/CD 流水线插件
- CI/CD 图形化流水线提供邮件通知功能，新增多个执行条件
- 端到端的流水线设置：支持从仓库 (GitHub / SVN / Git)、代码编译、镜像制作、镜像安全、推送仓库、版本发布、到定时构建的端到端流水线设置
- 安全管理：支持代码静态分析扫描以对 DevOps 工程中代码质量进行安全管理
- 日志：日志完整记录 CI / CD 流水线运行全过程

![产品功能 - 图5](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/17780fb1f1cc5bb267aa7712efc85fd2.png)

### Source to Image

提供 Source to Image (s2i) 的方式从已有的代码仓库中获取代码，并通过 Source to Image 构建镜像的方式来完成部署，并将镜像推送至目标仓库，每次构建镜像的过程将以任务 (Job) 的方式去完成。

![产品功能 - 图6](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/c2f12060313da2a8926bd17381228567.png)

### 多维度监控

- KubeSphere 全监控运维功能可通过可视化界面操作，同时，开放标准接口，易于对接企业运维系统，以统一运维入口实现集中化运维
- 立体化秒级监控：秒级频率、双重维度、十六项指标立体化监控
  - 在集群资源维度，提供 CPU 利用率、内存利用率、CPU 平均负载、磁盘使用量、inode 使用率、磁盘吞吐量、IOPS、网卡速率、容器组运行状态、ETCD 监控、API Server 监控等多项指标
  - 在应用资源维度，提供针对应用的 CPU 用量、内存用量、容器组数量、网络流出速率、网络流入速率等五项监控指标。并支持按用量排序和自定义时间范围查询，快速定位异常
- 提供按节点、企业空间、项目等资源用量排行
- 提供服务组件监控，快速定位组件故障

![产品功能 - 图7](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/c85436e68257d634783b81fd2ea71e12.png)

### 自研多租户告警系统

- 支持基于多租户、多维度的监控指标告警，目前告警策略支持集群管理员对节点级别和租户对工作负载级别等两个层级
- 灵活的告警策略：可自定义包含多个告警规则的告警策略，并且可以指定通知规则和重复告警的规则
- 丰富的监控告警指标：提供节点级别和工作负载级别的监控告警指标，包括容器组、CPU、内存、磁盘、网络等多个监控告警指标
- 灵活的告警规则：可自定义某监控指标的检测周期长度、持续周期次数、告警等级等
- 灵活的通知发送规则：可自定义发送通知时间段及通知列表，目前支持邮件通知
- 自定义重复告警规则：支持设置重复告警周期、最大重复次数并和告警级别挂钩

![产品功能 - 图8](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/cff8e9de832ffaa6daf7153c14ad8a6a.png)

### 日志查询与收集

- 提供多租户日志管理，在 KubeSphere 的日志查询系统中，不同的租户只能看到属于自己的日志信息
- 多级别的日志查询 (项目/工作负载/容器组/容器以及关键字)、灵活方便的日志收集配置选项等
- 支持多种日志收集平台，如 Elasticsearch、Kafka、Fluentd

![产品功能 - 图9](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/98efe5e138ac66aceb41f1a5d52a8036.png)

### 应用管理与编排

- 使用开源的 [OpenPitrix](https://openpitrix.io/) 提供应用商店和应用仓库服务，为用户提供应用全生命周期管理功能
- 用户基于应用模板可以快速便捷地部署一个完整应用的所有服务

![产品功能 - 图10](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/2014a39a74b96712c37b3e257926935d.png)

### 基础设施管理

提供存储类型管理、主机管理和监控、资源配额管理，并且支持镜像仓库管理、权限管理、镜像安全扫描。内置 Harbor 镜像仓库，支持添加 Docker 或私有的 Harbor 镜像仓库。

![产品功能 - 图11](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/7a71704c309b817728ff65f66841c4d8.png)

### 多存储类型支持

- 支持 GlusterFS、CephRBD、NFS 等开源存储方案，支持有状态存储
- NeonSAN CSI 插件对接 QingStor NeonSAN，以更低时延、更加弹性、更高性能的存储，满足核心业务需求
- QingCloud CSI 插件对接 QingCloud 云平台各种性能的块存储服务

![产品功能 - 图12](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/37f82fa93ef7942fedefd11e98bf4e79.png)

### 多网络方案支持

- 支持 Calico、Flannel 等开源网络方案
- 开发了适用于适用于物理机部署 Kubernetes 的 [负载均衡器插件 Porter](https://github.com/kubesphere/porter)



## 1.2 优势

通过 KubeSphere 可以快速管理 Kubernetes 集群、部署应用、服务发现、CI/CD 流水线、集群扩容、微服务治理、日志查询和监控告警。换句话说，Kubernetes 是一个很棒的开源项目（或被认为是一个框架），但是 KubeSphere 是一款非常专业的企业级平台产品，专注于解决用户在复杂业务场景中的痛点，提供更友好更专业的用户体验。



## 1.3 为什么选择 KubeSphere ？

KubeSphere 为企业用户提供高性能可伸缩的容器应用管理服务，旨在帮助企业完成新一代互联网技术驱动下的数字化转型，加速业务的快速迭代与交付，以满足企业日新月异的业务需求。

### 极简体验，向导式 UI

- 面向开发、测试、运维友好的用户界面，向导式用户体验，降低 Kubernetes 学习成本的设计理念
- 用户基于应用模板可以一键部署一个完整应用的所有服务，UI 提供全生命周期管理

### 业务高可靠与高可用

- 自动弹性伸缩：部署 (Deployment) 支持根据访问量进行动态横向伸缩和容器资源的弹性扩缩容，保证集群和容器资源的高可用
- 提供健康检查：支持为容器设置健康检查探针来检查容器的健康状态，确保业务的可靠性

### 容器化 DevOps 持续交付

- 简单易用的 DevOps：基于 Jenkins 的可视化 CI/CD 流水线编辑，无需对 Jenkins 进行配置，同时内置丰富的 CI/CD 流水线模版
- Source to Image (s2i)：从已有的代码仓库中获取代码，并通过 s2i 自动构建镜像完成应用部署并自动推送至镜像仓库，无需编写 Dockerfile
- 端到端的流水线设置：支持从仓库 (GitHub / SVN / Git)、代码编译、镜像制作、镜像安全、推送仓库、版本发布、到定时构建的端到端流水线设置
- 安全管理：支持代码静态分析扫描以对 DevOps 工程中代码质量进行安全管理
- 日志：日志完整记录 CI / CD 流水线运行全过程

### 开箱即用的微服务治理

- 灵活的微服务框架：基于 Istio 微服务框架提供可视化的微服务治理功能，将 Kubernetes 的服务进行更细粒度的拆分，支持无侵入的微服务治理
- 完善的治理功能：支持灰度发布、熔断、流量监测、流量管控、限流、链路追踪、智能路由等完善的微服务治理功能

灵活的持久化存储方案

- 支持 GlusterFS、CephRBD、NFS 等开源存储方案，支持有状态存储
- NeonSAN CSI 插件对接 QingStor NeonSAN，以更低时延、更加弹性、更高性能的存储，满足核心业务需求
- QingCloud CSI 插件对接 QingCloud 云平台各种性能的块存储服务

灵活的网络方案支持

- 支持 Calico、Flannel 等开源网络方案
- 分别开发了 [QingCloud 云平台负载均衡器插件](https://github.com/yunify/qingcloud-cloud-controller-manager) 和适用于物理机部署 Kubernetes 的 [负载均衡器插件 Porter](https://github.com/kubesphere/porter)
- 商业验证的 SDN 能力：可通过 QingCloud CNI 插件对接 QingCloud SDN，获得更安全、更高性能的网络支持

### 多维度监控日志告警

- KubeSphere 全监控运维功能可通过可视化界面操作，同时，开放标准接口对接企业运维系统，以统一运维入口实现集中化运维
- 可视化秒级监控：秒级频率、双重维度、十六项指标立体化监控；提供服务组件监控，快速定位组件故障
- 提供按节点、企业空间、项目等资源用量排行
- 支持基于多租户、多维度的监控指标告警，目前告警策略支持集群节点级别和工作负载级别等两个层级
- 提供多租户日志管理，在 KubeSphere 的日志查询系统中，不同的租户只能看到属于自己的日志信息

## 1.4 架构说明

KubeSphere 采用了**前后端分离的架构**，实现了**面向云原生的设计**，后端的各个功能组件可通过 REST API 对接外部系统，可参考 [API 文档](https://www.bookstack.cn/read/kubesphere-v2.0-zh/$api-reference-api-docs)。KubeSphere 无底层的基础设施依赖，可以运行在任何 Kubernetes、私有云、公有云、VM 或物理环境（BM）之上。

![架构说明 - 图1](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/7169dd15ec00b5fe9157df927df64381.png)

| 后端组件              | 功能说明                                                     |
| :-------------------- | :----------------------------------------------------------- |
| ks-account            | 提供用户、权限管理相关的 API                                 |
| ks-apiserver          | 整个集群管理的 API 接口和集群内部各个模块之间通信的枢纽，以及集群安全控制 |
| ks-apigateway         | 负责处理服务请求和处理 API 调用过程中的所有任务              |
| ks-console            | 提供 KubeSphere 的控制台服务                                 |
| ks-controller-manager | 实现业务逻辑的，例如创建企业空间时，为其创建对应的权限；或创建服务策略时，生成对应的 Istio 配置等 |
| Metrics-server        | Kubernetes 的监控组件，从每个节点的 Kubelet 采集指标信息     |
| Prometheus            | 提供集群、节点、工作负载、API 对象等相关监控数据与服务       |
| Elasticsearch         | 提供集群的日志索引、查询、数据管理等服务，在安装时也可对接您已有的 ES 减少资源消耗 |
| Fluent Bit            | 提供日志接收与转发，可将采集到的⽇志信息发送到 ElasticSearch、Kafka |
| Jenkins               | 提供 CI/CD 流水线服务                                        |
| SonarQube             | 可选安装项，提供代码静态检查与质量分析                       |
| Source-to-Image       | 将源代码自动将编译并打包成 Docker 镜像，方便快速构建镜像     |
| Istio                 | 提供微服务治理与流量管控，如灰度发布、金丝雀发布、熔断、流量镜像等 |
| Jaeger                | 收集 Sidecar 数据，提供分布式 Tracing 服务                   |
| OpenPitrix            | 提供应用模板、应用部署与管理的服务                           |
| Alert                 | 提供集群、Workload、Pod、容器级别的自定义告警服务            |
| Notification          | 通用的通知服务，目前支持邮件通知                             |
| redis                 | 将 ks-console 与 ks-account 的数据存储在内存中的存储系统     |
| MySQL                 | 集群后端组件的数据库，监控、告警、DevOps、OpenPitrix 共用 MySQL 服务 |
| PostgreSQL            | SonarQube 和 Harbor 的后端数据库                             |
| OpenLDAP              | 负责集中存储和管理用户账号信息与对接外部的 LDAP              |
| 存储                  | 内置 CSI 插件对接云平台存储服务，可选安装开源的 NFS/Ceph/Gluster 的客户端 |
| 网络                  | 可选安装 Calico/Flannel 等开源的网络插件，支持对接云平台 SDN |

除了上述列表的组件，KubeSphere 还支持 Harbor 与 GitLab 作为可选安装项，您可以根据项目需要进行安装。以上列表中每个功能组件下还有多个服务组件，关于服务组件的说明，可参考 [服务组件说明](https://www.bookstack.cn/read/kubesphere-v2.0-zh/$infrastructure-components)。

![架构说明 - 图2](https://static.bookstack.cn/projects/kubesphere-v2.0-zh/423bbc835bfd6c7891468d44b5ce80c7.png)

## 1.5 应用场景

KubeSphere®️ 适用于企业在数字化转型时所面临的敏捷开发与自动化运维、微服务应用架构与流量治理、自动弹性伸缩和业务高可用、DevOps 持续集成与交付等应用场景。

### 一步升级容器架构，助力业务数字化转型

企业用户部署于物理机、传统虚拟化环境的业务系统，各业务模块会深度耦合，资源不能灵活的水平扩展。 KubeSphere 帮助企业将 IT 环境容器化并提供完整的运维管理功能，同时依托青云QingCloud 为企业提供强大的网络、存储支持，并可高效对接企业原监控、运维系统，一站式高效完成企业 IT 容器化改造。

### 多维管控 Kubernetes，降低运维复杂度

无论将业务架构在 Kubernetes 平台上的用户，还是使用多套来自不同厂商提供的 Kubernetes 平台的用户，复杂的运维管理使企业压力倍增。KubeSphere 可提供统一平台纳管异构 Kubernetes 集群，支持应用自动化部署，减轻日常运维压力。同时，完善的监控告警与日志管理系统有效节省运维人工成本，使企业能够将更多精力投放到业务创新上。

### 敏捷开发与自动化运维，推动企业 DevOps 落地

DevOps 将开发团队与运营团队通过一套流程或方法建立更具协作性、更高效的的关系，使得开发、测试、发布应用能够更加敏捷、高效、可靠。KubeSphere CI / CD 功能可为企业DevOps 提供敏捷开发与自动化运维。同时， KubeSphere 的微服务治理功能，帮助企业以一种细粒度的方式开发、测试和发布服务，有效推动企业 DevOps 落地。

### 灵活的微服务解决方案，一步升级云原生架构

微服务架构可轻量级构建冗余，可扩展性强，非常适合构建云原生应用程序。KubeSphere 基于主流微服务解决方案 Istio，提供无代码侵入的微服务治理平台。后续将集成 SpringCloud，便于企业构建 Java 应用，助力企业一步实现微服务架构，实现应用云原生转型。

### 基于物理环境构建全栈容器架构，释放硬件最大效能

支持在全物理环境部署全栈容器架构，利用物理交换机，为 KubeSphere 提供负载均衡器服务，同时，通过 KubeSphere 与 QingCloud VPC 以及QingStor NeonSAN 的组合，可打通负载均衡、容器平台、网络、存储全栈功能，实现真正意义上的物理环境一体化多租户容器架构解决方案，并实现自主可控、统一管理。避免虚拟化带来的性能损耗，释放硬件最大效能。

## 1.6 名词解释

| KubeSphere   | Kubernetes 对照释义                                          |
| :----------- | :----------------------------------------------------------- |
| 项目         | Namespace， 为 Kubernetes 集群提供虚拟的隔离作用，详见 [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)。 |
| 容器组       | Pod，是 Kubernetes 进行资源调度的最小单位，每个 Pod 中运行着一个或多个密切相关的业务容器 |
| 部署         | Deployments，表示用户对 Kubernetes 集群的一次更新操作，详见 [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)。 |
| 有状态副本集 | StatefulSets，用来管理有状态应用，可以保证部署和 scale 的顺序，详见 [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)。 |
| 守护进程集   | DaemonSets，保证在每个 Node 上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用，详见 [Daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)。 |
| 任务         | Jobs，在 Kubernetes 中用来控制批处理型任务的资源对象，即仅执行一次的任务，它保证批处理任务的一个或多个 Pod 成功结束。任务管理的 Pod 根据用户的设置将任务成功完成就自动退出了。比如在创建工作负载前，执行任务，将镜像上传至镜像仓库。详见 [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)。 |
| 定时任务     | CronJob，是基于时间的 Job，就类似于 Linux 系统的 crontab，在指定的时间周期运行指定的 Job，在给定时间点只运行一次或周期性地运行。详见 [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) |
| 服务         | Service， 一个 Kubernete 服务是一个最小的对象，类似 Pod，和其它的终端对象一样，详见 [Service](https://kubernetes.io/docs/concepts/services-networking/service/)。 |
| 应用路由     | Ingress，是授权入站连接到达集群服务的规则集合。可通过 Ingress 配置提供外部可访问的 URL、负载均衡、SSL、基于名称的虚拟主机等，详见 [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)。 |
| 镜像仓库     | Image Registries，镜像仓库用于存放 Docker 镜像，Docker 镜像用于部署容器服务， 详见 [Images](https://kubernetes.io/docs/concepts/containers/images/)。 |
| 存储卷       | PersistentVolumeClaim（PVC），满足用户对于持久化存储的需求，用户将 Pod 内需要持久化的数据挂载至存储卷，实现删除 Pod 后，数据仍保留在存储卷内。Kubesphere 推荐使用动态分配存储，当集群管理员配置存储类型后，集群用户可一键式分配和回收存储卷，无需关心存储底层细节。详见 [Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)。 |
| 存储类型     | StorageClass，为管理员提供了描述存储 “Class（类）” 的方法，包含 Provisioner、 ReclaimPolicy 和 Parameters 。详见 [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)。 |
| 流水线       | Pipeline，简单来说就是一套运行在 Jenkins 上的 CI/CD 工作流框架，将原来独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。 |
| 企业空间     | Workspace，是 KubeSphere 实现多租户模式的基础，是您管理项目、 DevOps 工程和企业成员的基本单位。 |