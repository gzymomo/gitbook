官网地址：https://karmada-io.slack.com/

Github地址：https://github.com/karmada-io/karmada



Karmada（Kubernetes Armada）是一个 Kubernetes 管理系统，基于 Kubernetes  Federation v1 和 v2 开发，它可以跨多个 Kubernetes  集群和云运行云原生应用程序，而无需对应用程序进行更改。通过直接使用 Kubernetes 原生 API 并提供高级调度功能，Karmada  可以实现真正的开放式多云 Kubernetes。

Karmada 旨在为多云和混合云场景下的多集群应用程序管理提供 turnkey 自动化，其关键功能包括集中式多云管理、高可用性、故障恢复和流量调度。

# 一、Karmada 的优势

- 兼容 K8s 原生 API
  - 零修改实现从单集群到多集群的升级
  - 无缝集成现有 K8s 工具链
- 开箱即用
  - 内置针对不同场景的策略集，包括：Active-Active、远程灾难恢复、地理冗余等
  - 支持跨集群应用程序在多集群上的自动扩展、故障转移和负载均衡
- 避免供应商锁定
  - 支持与主流云厂商集成
  - 支持自动分配和跨集群迁移
  - 不依赖于某家专有供应商编排
- 集中管理
  - 支持位置不可知集群管理
  - 支持公有云、本地端（on-prem）或边缘端集群管理
- 高效的多集群调度策略
  - 支持集群亲和性调度，多集群拆分和重新平衡
  - 多维 HA：区域/可用区/集群/提供商
- 开放中立
  - 由多家互联网、金融、制造业、电信、云服务厂商共同发起
  - 以捐赠给 CNCF 进行开放治理为目标

# 二、Karmada 的架构设计

![image.png](https://segmentfault.com/img/remote/1460000039902047)

Karmada 的控制面板包含 API 服务器（API Server）、控制器管理器（Controller Manager ）和调度器三大组件。

ETCD 存储 karmada API 对象，API 服务器作为 REST 端点，可以与所有其他组件通信，而 Karmada 控制器管理器将根据用户创建的 API 对象执行操作。

Karmada 控制器管理器运行各种控制器，这些控制器监视 karmada 的对象，然后与基础集群的 API 服务器对话以创建常规的 Kubernetes 资源。

1. 集群控制器：将 Kubernetes 集群添加到 Karmada，通过创建集群对象来管理集群的生命周期；
2. 策略控制器：监控 PropagationPolicy 对象，当有新增 PropagationPolicy 对象时，它将选择与 resourceSelector 匹配的一组资源，并为每个资源对象创建 ResourceBinding；
3. 绑定控制器：监控 ResourceBinding 对象，并使用单个资源清单创建与每个集群相对应的 Work 对象；
4. 执行控制器：监控 Work 对象，当出现新创建的 Work 对象时，它负责将资源分配给成员集群。

# 三、Karmada 概念

资源模板：Karmada 使用 Kubernetes 本机 API 定义的联合资源模板，以使其易于与 Kubernetes 上已采用的现有工具集成

传播策略：Karmada 提供独立的传播（放置）策略 API，以定义多集群调度和传播需求。

- 支持策略 1：n 映射：工作负载，用户无需在每次创建联合应用程序时都指出调度约束。
- 使用默认策略，用户可以仅与 K8s API 进行交互

覆盖策略：Karmada 提供独立的覆盖策略 API，用于专门针对与群集相关的配置自动化。例如：

- 根据成员群集区域覆盖图像前缀
- 根据云提供商覆盖 StorageClass

下图显示了将资源传播到成员集群时如何使用 Karmada 资源。

![image.png](https://segmentfault.com/img/remote/1460000039902048)