# KubeOperator介绍

​	KubeOperator 是一个 **开源** 的轻量级 Kubernetes 发行版，专注于帮助企业规划、部署和运营生产级别的 Kubernetes 集群。

> 核心功能全部开源，并且代码在github上，但针对企业版，会提供一些增强包。

​	KubeOperator 提供可视化的 Web UI，支持离线环境，支持物理机、VMware 和 OpenStack 等 IaaS 平台，支持 x86 和 ARM64 架构，支持 GPU，内置应用商店，<font color='red'>**已通过 CNCF 的 Kubernetes 软件一致性认证。**</font>

​	KubeOperator 使用 Terraform 在 IaaS 平台上自动创建主机（用户也可以自行准备主机，比如物理机或者虚机），通过 <font color='red'>**Ansible**</font> 完成自动化部署和变更操作，支持 Kubernetes 集群 从 Day 0 规划，到 Day 1 部署，到 Day 2 运营的全生命周期管理。



# 技术优势

​	

- 简单易用: 提供可视化的 Web UI，极大降低 Kubernetes 部署和管理门槛，内置 [Webkubectl](https://github.com/KubeOperator/webkubectl)
- 按需创建: 调用云平台 API，一键快速创建和部署 Kubernetes 集群
- 按需伸缩: 快速伸缩 Kubernetes 集群，优化资源使用效率
- 按需修补: 快速升级和修补 Kubernetes 集群，并与社区最新版本同步，保证安全性
- 离线部署: 支持完全离线下的 Kubernetes 集群部署
- 自我修复: 通过重建故障节点确保集群可用性
- 全栈监控: 提供从Pod、Node到集群的事件、监控、告警、和日志方案
- Multi-AZ 支持: 将 Master 节点分布在不同的故障域上确保集群高可用
- 应用商店: 内置 [KubeApps](https://github.com/kubeapps/kubeapps) 应用商店
- GPU 支持: 支持 GPU 节点，助力运行深度学习等应用



# KubeOperator的定位是什么？

​	KubeOperator 是一个开源的轻量级 Kubernetes 发行版。与 OpenShift 等重量级 PaaS 平台相比，KubeOperator 只专注于解决一个问题，就是帮助企业规划（Day 0）、部署（Day 1）、运营（Day 2）生产级别的 Kubernetes 集群，并且做到极致。

![what-is-ko](https://kubeoperator.io/docs/img/faq/what-is-ko.png)



云原生正在快速兴起，三个互相关联的领域在同步进化:

- 基础设施方面: 从 物理资源 到 虚拟化资源 到 容器化（ Kubernetes ）资源 的演进；
- 开发模式方面: 从 瀑布模型 到 敏捷开发 到 DevOps 的演进；
- 应用架构方面: 从 单体架构 到 多层次架构 到 微服务 的演进。

### 开源版和企业版的区别？

​	和同属飞致云旗下的 JumpServer 开源堡垒机一样，KubeOperator 的核心功能全部开源，坚持按月发布新版本，永久免费使用。

​	相比 KubeOperator 开源版，KubeOperator 企业版提供面向企业级应用场景的 X-Pack 增强包，以及高等级的原厂企业级支持服务，有效助力企业构建并运营生产级别的 K8s 集群。其中，X-Pack 增强包括一些企业级客户所需的附加功能，比如自定义 Logo 和主题、LDAP 对接等，X-Pack 增强包的具体功能会随新版发布持续增加。



# KubeOperator 与 Kubespray 等部署工具的区别是什么？

KubeOperator 不仅提供 Day 1 部署功能，还提供 Day 2 的 Kubernetes 集群升级、扩容、监控检查、备份恢复等功能，如下图所示。

![overview](https://kubeoperator.io/docs/img/faq/overview.png)

KubeOperator 不仅支持安装程序本身，还提供了一组工具来监视 Kubernetes 集群的持续运行。KubeOperator 的优势包括:

- 提供可视化的 Web UI，大大降低部署和管理 Kubernetes 的门槛；
- 提供离线的、经过全面验证和测试的安装包；
- 与 VMware 和 Openstack 等云平台紧密对接，能够实现一键虚机自动创建和部署（基于 Terraform 和 Ansible）；
- KubeOperator 会提供经过充分验证的成熟企业级存储和网络方案。

# KubeOperator 和 Rancher 有什么区别？

​	Rancher 是完整的容器管理平台，KubeOperator 仅专注于帮助企业规划、部署和运营生产级别的 Kubernetes 集群，和 KubeOperator 有可比性的是 Rancher RKE，而不是 Rancher 全部。

​	KubeOperator 推荐企业采纳解耦的方式来实现云原生之路，也就是说容器云平台与其之上的 DevOps 平台、微服务治理平台、AI 平台、应用商店等是解耦的。

### Kubernetes 集群中的 master 节点的推荐配置？

Kubernetes 集群中 master 节点配置取决于 worker 节点数量，推荐配置参考如下:

| worker 节点数量 | master 推荐配置 |
| :-------------- | :-------------- |
| 1-5             | 1C 4G           |
| 6-10            | 2C 8G           |
| 11-100          | 4C 16G          |
| 101-250         | 8C 32G          |
| 251-500         | 16C 64G         |
| > 500           | 32C 128G        |

# KubeOperator总体架构

KubeOperator 产品架构如下图所示

![Architecture](https://kubeoperator.io/images/screenshot/ko-framework.svg)

## 模块说明

- kubeoperator_server: 提供平台业务管理相关功能的后台服务；
- kubeoperator_ui: 提供平台业务管理相关功能的前台服务；
- kubeoperator_kobe: 提供执行 Ansible 任务创建 Kubernetes 集群的功能；
- kubeoperator_kotf: 提供执行 Terraform 任务创建虚拟机的功能；
- kubeoperator_webkubectl: 提供在 Web 浏览器中运行 kubectl 命令的功能；
- kubeoperator_nginx: 平台统一入口，并运行控制台的 Web 界面服务；
- kubeoperator_mysql: 数据库管理组件；
- kubeoperator_nexus: 仓库组件，提供 Docker、Helm、Raw、Yum等资源仓库功能；
- kubeoperator_grafana: 监控组件，提供平台监控等相关功能；