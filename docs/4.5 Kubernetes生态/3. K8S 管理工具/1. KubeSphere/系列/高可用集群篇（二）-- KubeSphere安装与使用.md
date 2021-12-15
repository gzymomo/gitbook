- [高可用集群篇（一）-- k8s集群部署](https://juejin.cn/post/6940887645551591455)
- [高可用集群篇（二）-- KubeSphere安装与使用](https://juejin.cn/post/6942282048107347976)
- [高可用集群篇（三）-- MySQL主从复制&ShardingSphere读写分离分库分表](https://juejin.cn/post/6944142563532079134)
- [高可用集群篇（四）-- Redis、ElasticSearch、RabbitMQ集群](https://juejin.cn/post/6945360597668069384)
- [高可用集群篇（五）-- k8s部署微服务](https://juejin.cn/post/6946396542097948702)

# 一、简介

## 1.1 概述

[KubeSphere](https://kubesphere.io) 是在 [Kubernetes](https://kubernetes.io) 之上构建的面向云原生应用的**分布式操作系统**，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。

作为全栈的多租户容器平台，KubeSphere 提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。KubeSphere 为用户提供构建企业级 Kubernetes 环境所需的多项功能，例如**多云与多集群管理、Kubernetes 资源管理、DevOps、应用生命周期管理、微服务治理（服务网格）、日志查询与收集、服务与网络、多租户管理、监控告警、事件与审计查询、存储管理、访问权限控制、GPU 支持、网络策略、镜像仓库管理以及安全管理**等。

KubeSphere 还开源了 [KubeKey](https://github.com/kubesphere/kubekey) 帮助企业一键在公有云或数据中心快速搭建 Kubernetes 集群，提供单节点、多节点、集群插件安装，以及集群升级与运维。https://kubesphere.io/)。

![功能概览](https://kubesphere.io/images/docs/zh-cn/introduction/what-is-kubesphere/kubesphere-feature-overview.jpeg)

## 1.2 开发运维友好

KubeSphere  为用户屏蔽了基础设施底层复杂的技术细节，帮助企业在各类基础设施之上无缝地部署、更新、迁移和管理现有的容器化应用。通过这种方式，KubeSphere 使开发人员能够专注于应用程序开发，使运维团队能够通过企业级可观测性功能和故障排除机制、统一监控和日志查询、存储和网络管理，以及易用的  CI/CD 流水线等来加快 DevOps 自动化工作流程和交付流程等。

## 1.3 支持在任意平台运行 KubeSphere

作为一个灵活的轻量级容器 PaaS 平台，KubeSphere 对不同云生态系统的支持非常友好，因为它对原生 Kubernetes 本身没有任何的侵入 (Hack)。换句话说，KubeSphere 可以**部署并运行在任何基础架构以及所有版本兼容的 Kubernetes 集群**之上，包括虚拟机、物理机、数据中心、公有云和混合云等。

您可以选择在公有云和托管 Kubernetes 集群（例如阿里云、AWS、青云QingCloud、腾讯云、华为云等）上安装 KubeSphere，**还可以导入和纳管已有的 Kubernetes 集群**。

KubeSphere 可以在不修改用户当前的资源或资产、不影响其业务的情况下部署在现有的 Kubernetes 平台上。有关更多信息，请参见[在 Linux 上安装](https://kubesphere.io/zh/docs/installing-on-linux/)和[在 Kubernetes 上安装](https://kubesphere.io/zh/docs/installing-on-kubernetes/)。

## 1.4 完全开源

借助开源的模式，KubeSphere 社区驱动着开发工作以开放的方式进行。KubeSphere **100% 开源免费**，已大规模服务于社区用户，广泛地应用在以 Docker 和 Kubernetes 为中心的开发、测试及生产环境中，大量服务平稳地运行在 KubeSphere 之上。您可在 [GitHub](https://github.com/kubesphere/) 上找到所有源代码、文档和讨论，所有主要的开源项目介绍可以在[开源项目列表](https://kubesphere.io/zh/projects/)中找到。

# 二、Kubesphere安装

参考Kubesphere官网进行安装和部署：https://kubesphere.io/zh/docs/

![image-20210916114606309](https://gitee.com/er-huomeng/l-img/raw/master/typora/image-20210916114606309.png)

# 三、Kubesphere进阶

详细操作文档参考Kubesphere：https://kubesphere.io/zh/docs/

## 3.1 建立多租户系统

- 本文档面向初次使用 KubeSphere 的集群管理员用户，引导新手用户创建企业空间、创建新的角色和账户，然后邀请新用户进入企业空间后，创建项目和 DevOps 工程，帮助用户熟悉多租户下的用户和角色管理，快速上手 KubeSphere

- 目前，平台的资源一共有三个层级，包括 **集群 (Cluster)、 企业空间 (Workspace)、 项目 (Project) 和 DevOps Project (DevOps 工程)**，层级关系如下图所示，即一个集群中可以创建多个企业空间，而每个企业空间，可以创建多个项目和 DevOps工程，而集群、企业空间、项目和 DevOps工程中，默认有多个不同的内置角色

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/136e0a98a154441ab3b4806265387a68~tplv-k3u1fbpfcp-zoom-1.image)

- 示例逻辑

  - ![%E5%A4%9A%E7%A7%9F%E6%88%B7%E7%AE%A1%E7%90%86%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6dd73fb0579413ea75618b0278b69bc~tplv-k3u1fbpfcp-zoom-1.image)
  

### 第一步：创建角色和账号

- 1.1、首先新建一个角色 (user-manager)，为该角色授予账号管理和角色管理的权限，然后新建一个账号并给这个账号授予 user-manager 角色

  ```
  平台角色--创建--user-manager--分配用户管理、角色管理的权限
  ```
  
- 1.2、 **平台管理 → 账户管理**，可以看到当前集群中所有用户的列表，点击 **创建** 按钮

  ```
  touch-air-hr --> 角色为 user-manager
  ```
  
  ![image-20210317132947360](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/205de5e35b6740738b8a6428d4ca2e57~tplv-k3u1fbpfcp-zoom-1.image)
  
- 1.3、**user-manager** 账户登录来创建下表中的四个账号，`ws-manager`将用于创建一个企业空间，并指定其中一个用户名为 `ws-admin`作为企业空间管理员

  | 用户名          | 集群角色           | 职责                                                         |
  | --------------- | ------------------ | ------------------------------------------------------------ |
  | ws-manager      | workspaces-manager | 创建和管理企业空间                                           |
  | ws-admin        | cluster-regular    | 管理企业空间下所有的资源 (本示例用于邀请新成员加入企业空间)  |
  | project-admin   | cluster-regular    | 创建和管理项目、DevOps 工程，邀请新成员加入                  |
  | project-regular | cluster-regular    | 将被 project-admin 邀请加入项目和 DevOps 工程， 用于创建项目和工程下的工作负载、Pipeline 等资源 |

  - ![image-20210317133918504](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce4a889e512f4a3288625e72ea3f0a7c~tplv-k3u1fbpfcp-zoom-1.image)
  

### 第二步：创建企业空间

- 企业空间 (workspace) 是 KubeSphere 实现多租户模式的基础，是用户管理项目、DevOps 工程和企业成员的基本单位

- 2.1、切换为 `ws-manager`登录 KubeSphere，ws-manager 有权限查看和管理平台的所有企业空间；点击左上角的 `平台管理`→ `企业空间`，可见新安装的环境只有一个系统默认的企业空间 **system-workspace**，用于运行 KubeSphere 平台相关组件和服务，禁止删除该企业空间；在企业空间列表点击 **创建**

  ![image-20210317140008891](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2572c9ab6a54fb0a0bcfafec661de04~tplv-k3u1fbpfcp-zoom-1.image)

  mall-workspace -- 企业成员 -- 邀请成员邀请 ws-admin，并赋予企业空间管理员的角色

  ![image-20210317140805643](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da4fbf7d17e1440897b5c69c04ec9a4a~tplv-k3u1fbpfcp-zoom-1.image)

- 2.2、企业空间 `mall-workspace`创建完成后，切换为 `ws-admin`登录 KubeSphere，点击左侧「进入企业空间」进入企业空间详情页

  ![image-20210317141133323](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e17f01f092e24396998a3e527bd35f0d~tplv-k3u1fbpfcp-zoom-1.image)

  ws-admin`可以从集群成员中邀请新成员加入当前企业空间，然后创建项目和 DevOps 工程。在左侧菜单栏选择 `企业空间管理`→ `成员管理`，点击 `邀请成员；这一步需要邀请在 **步骤 1.6.** 创建的两个用户 `project-admin`和 `project-regular`进入企业空间，且分别授予 `workspace-regular`和 `workspace-viewer`的角色，此时该企业空间一共有如下三个用户：

  ![image-20210317141209557](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/090bb033c2bd4077b2a59b2b49969e15~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210317141240935](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24c419852d7e477691a34bddbbb9e18c~tplv-k3u1fbpfcp-zoom-1.image)

  分别使用`project-admin`和 `project-regular`登录，观察不同，admin可以创建项目，regular只有浏览权限

### 第三步：创建项目

- 创建工作负载、服务和 CI/CD 流水线等资源之前，需要预先创建项目和 DevOps 工程

- 3.1、上一步将用户项目管理员 `project-admin`邀请进入企业空间后，可切换为 `project-admin`账号登录 KubeSphere，默认进入 demo-workspace 企业空间下，点击 **创建**，选择 **创建资源型项目**

  在弹窗中的 `project-regular`点击 `"+"`，在项目的内置角色中选择 `operator`角色。因此，后续在项目中创建和管理资源，都可以由 `project-regular`用户登录后进行操作

  ![image-20210317141908374](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee24a50cd52b482fa81f7fe72987c92b~tplv-k3u1fbpfcp-zoom-1.image)

  3.2、再次使用 `project-regular` 账户登录，就可以看到它维护的项目

  - ![image-20210317142138331](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c8e13684a56438984b4506b43615b05~tplv-k3u1fbpfcp-zoom-1.image)
  

### 第四步：创建 DevOps 工程

- 4.1、继续使用 `project-admin`用户创建 DevOps 工程。点击 **工作台**，在当前企业空间下，点击 **创建**，在弹窗中选择 **创建一个 DevOps 工程**。DevOps 工程的创建者 `project-admin`将默认为该工程的 Owner，拥有 DevOps 工程的最高权限

  ![image-20210317142556282](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e39f0d5cc08143c58e72f11d16fe26c8~tplv-k3u1fbpfcp-zoom-1.image)

- 4.2、. 同上，这一步需要在 `demo-devops`工程中邀请用户 `project-regular`，并设置角色为 `maintainer`，用于对工程内的 Pipeline、凭证等创建和配置等操作。菜单栏选择 **工程管理 → 工程成员**，然后点击 **邀请成员**，为用户 `project-regular`设置角色为 `maintainer`；后续在 DevOps 工程中创建 Pipeline 和凭证等资源，都可以由 `project-regular`用户登录后进行操作

  ![image-20210317142737845](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/175d9f8d5a834429a06e872e97e92b03~tplv-k3u1fbpfcp-zoom-1.image)

- 4.3、再次使用 `project-regular` 账户登录，就可以看到它拥有两个项目，一个是微服务商城、另一个是自动化部署的devops

  - ![image-20210317143015953](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd8688d17db6405d939db349b826f3d1~tplv-k3u1fbpfcp-zoom-1.image)

## 3.2 创建WordPress应用

- [创建WordPress应用](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/wordpress-deployment/)

- WordPress 是使用 PHP 开发的博客平台，用户可以在支持 PHP 和 MySQL 数据库的环境中架设属于自己的网站；本文以创建一个 [Wordpress 应用](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/wordpress-deployment/www.wordpress.com/‎) 为例，以创建 KubeSphere 应用的形式将 Wordpress 的组件（MySQL 和 Wordpress）创建后发布至 Kubernetes 中，并在集群外访问 Wordpress 服务

  一个完整的 Wordpress 应用会包括以下 Kubernetes 对象，其中 MySQL 作为后端数据库，Wordpress 本身作为前端提供浏览器访问

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d61c476925ad4875b976d5a9d7325985~tplv-k3u1fbpfcp-zoom-1.image)

  PersistentVolume（PV）是集群中由管理员配置的一段网络存储，它是集群中的资源，就像节点是集群资源一样；PV是容量插件，如Volumes,但其生命周期独立于使用PV的任何单个pod

  PersistentVolumeClaim（PVC）是由用户进行存储的请求；它类似于pod，Pod消耗节点资源，PVC消耗PV资源

  PVC和PV是一一对应的

### 前提条件

- 已创建了企业空间、项目和普通用户 `project-regular`账号（该已账号已被邀请至示例项目），并开启了外网访问，请参考 [多租户管理快速入门](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/admin-quick-start)

### 创建密钥

- MySQL 的环境变量 `MYSQL_ROOT_PASSWORD`即 root 用户的密码属于敏感信息，不适合以明文的方式表现在步骤中，因此以创建密钥的方式来代替该环境变量。创建的密钥将在创建 MySQL 的容器组设置时作为环境变量写入

### 创建MySQL密钥

- 1、以项目普通用户 `project-regular`登录 KubeSphere，在当前项目下左侧菜单栏的 **配置中心** 选择 **密钥**，点击 **创建**

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b688bdc322354d51a971d6738463a500~tplv-k3u1fbpfcp-zoom-1.image)

- 2、填写密钥的基本信息，完成后点击 **下一步**

  - 名称：作为 MySQL 容器中环境变量的名称，可自定义，例如 `mysql-secret`
  - 别名：别名可以由任意字符组成，帮助您更好的区分资源，例如 `MySQL 密钥`
  - 描述信息：简单介绍该密钥，如 `MySQL 初始密码`

- 3、密钥设置页，填写如下信息，完成后点击 **创建**

  - 类型：选择 `默认`(Opaque)
  - Data：Data 键值对填写 `MYSQL_ROOT_PASSWORD`和 `123456`

  - ![image-20210317150405010](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbf2f632f4864add82c76a80fa2af2c1~tplv-k3u1fbpfcp-zoom-1.image)
  

### 创建WordPress密钥

- 同上，创建一个 WordPress 密钥，Data 键值对填写 `WORDPRESS_DB_PASSWORD`和 `123456`

![image-20210317150553102](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db936c2352ec4000b91113fee2f987f3~tplv-k3u1fbpfcp-zoom-1.image)

#### 

### 创建存储卷

- 点击创建，配置可以都是默认的

  - ![image-20210317151003677](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffd50188c4ee49448c71dfa966c1eb7c~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210317151050263](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/356a8a45cbc0443198de4bd7d6827a7f~tplv-k3u1fbpfcp-zoom-1.image)
  
  #### 创建应用

### 添加MySQL组件

### 添加WordPress组件

[创建应用--按照指导一步步做](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/wordpress-deployment/#创建应用)

### 查看应用资源

- 1、在 `工作负载`下查看 **部署** 和 **有状态副本集** 的状态，当它们都显示为 `运行中`，说明 WordPress 应用创建成功

  - ![image-20210317154054132](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7e88a7755f64626a3f0ffede7c93419~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210317154107897](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5537b7bf2cc44bc89c91e2e4651de99~tplv-k3u1fbpfcp-zoom-1.image)
  
  

### 外网访问

- 2、访问 Wordpress 服务前，查看 wordpress 服务，将外网访问设置为 `NodePort`

  ![image-20210317154552911](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b7e4194819948bdad46c462dd16698b~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210317154650080](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64401feee65e4878bcb44bb5760ab62a~tplv-k3u1fbpfcp-zoom-1.image)

- 3、点击 `更多操作`→ `编辑外网访问`，选择 `NodePort`，然后该服务将在每个节点打开一个节点端口，通过 `点击访问`即可在浏览器访问 `WordPress`

  ![image-20210317154713036](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb4b24edbed845b98f2c62d11ebfbf7c~tplv-k3u1fbpfcp-zoom-1.image)

- 4、点击 `更多操作`→ `编辑配置文件`，可以看到kubesphere给我们自动生成的yaml文件

  - ![image-20210317155415436](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef5f1efdd9744c90a5c6f4cc1447bdf4~tplv-k3u1fbpfcp-zoom-1.image)
  
  

### 创建应用总结

- 至此，您已经熟悉了如何通过创建一个 KubeSphere 应用的方式，通过快速添加多个组件来完成一个应用的构建，最终发布至 Kubernetes；这种创建应用的形式非常适合微服务的构建，只需要将各个组件容器化以后，即可通过这种方式快速创建一个完整的微服务应用并发布 Kubernetes

  同时，这种方式还支持用户以 **无代码侵入的形式开启应用治理**，针对 **微服务、流量治理、灰度发布与 Tracing** 等应用场景，开启应用治理后会在每个组件中以 SideCar 的方式注入 Istio-proxy 容器来接管流量，后续将以一个 Bookinfo 的示例来说明如何在创建应用中使用应用治理

## 3.3 DevOps

- 项目开发需要考虑的维度
  - Dev：怎么开发
  - Ops：怎么运维
  - 高并发：怎么承担高并发
  - 高可用：怎么做到高可用

### 什么是DevOps

- 微服务：服务自治

  ![image-20210317155952043](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/837033a090574e9c9e2f6302f2d97d42~tplv-k3u1fbpfcp-zoom-1.image)

- DevOps：Development 和 Operations 的组合

  - DevOps看作开发（软件工程）、技术运营和质量保障（QA）三者的交集
  - 突出重视软件开发人员和运维人员的沟通合作，通过自动化流程来使得软件构建、测试、发布更加快捷、频繁和可靠
  - DevOps希望做到的是软件产品交付过程中IT工具链的打通，使得各个团队减少时间损耗，更加高效的协同工作，专家们总结出了下面这个DevOps能力图，良好的闭环可以大大增加整体的产出

### 什么是CI&CD

![image-20210317165353841](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92be6016e3b54469bdf55e50d7003d26~tplv-k3u1fbpfcp-zoom-1.image)



#### 持续集成（Continuous Integration）

- 持续集成是指软件个人研发的部分想软件整体部分交付，频繁的进行集成以便更快的发现其中的错误，持续集成源自于极限编程（XP），是XP最初的12种实践之一

- CI需要具备这些

  ：

  - 全面的自动化测试：这是实践持续集成&持续部署的基础，同时，选择合适的自动化测试工具也极其重要
  - 灵活的基础设施：容器、虚拟机的存在让开发人员和QA人员不必再大费周折
  - 版本控制工具：如Git、CVS、SVN等
  - 自动化的构建和软件发布流程的工具：如jenkins
  - 反馈机制：如构建/测试的失败，可以快速的反馈到相关负责人，以尽快解决达到一个更稳定的版本

#### 持续交付（Continuous Delivery）

- 持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的【类生成环境】（production-like environments）中；持续交付优先于整个产品生命周期的软件部署，建立在高水平自动化持续集成之上
- 灰度发布
- 持续交付和持续集成的有点非常相似：
  - 快速发布：能够应对业务需求，并更快的实现软件价值
  - 编码 - 测试 - 上线 - 交付的频繁迭代周期缩短，同时获得迅速反馈
  - 高质量的软件发布标准：整个交付过程标准化、可重复、可靠
  - 整个交付过程进度可视化：方便团队人员了解项目成熟度
  - 更先进的团队协作方式：从需求分析、产品的用户体验到交互设计、开发、测试、运维等角色紧密协作，相比于传统的瀑布式软件团队，更少浪费

#### 持续部署（Continuous Deployment）

- 持续部署是指当交付的代码通过评审之后，自动部署到生成环境中；持续部署是持续交付的最高阶段，这意味着所有通过了一些列的自动化测试的改动都将自动部署到生产环境，它也可被称为 Continuous Release

- 开发人员提交代码，持续集成服务器获取代码，执行单元测试，根据测试结果决定是否部署到预演环境，如果成功部署到预演环境，进行整体验收测试，如果测试通过，自动部署到产品环境，全程自动化高效运转

- 持续部署主要好处是，可以相对独立的部署新的功能，并能快速地收集真实用户的反馈

  `you build it,you run it`,这是Amazon一年可以完成5000万次部署，平均每个工程师每天部署超过50次的核心秘籍

CI&CD持续交付工具链图

![image-20210317172559183](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14da8fa4c88d48f5a26dd5b944d691d4~tplv-k3u1fbpfcp-zoom-1.image)



### kubesphere流水线

- Pipeline 是一系列的插件集合，可以通过组合它们来实现持续集成和持续交付的功能。 Pipeline DSL 为我们提供了一个可扩展的工具集，让我们可以将简单到复杂的逻辑通过代码实现

  [基于SpringBoot项目构建流水线](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/devops-online/)

#### 流水线概览

- 下面的流程图简单说明了流水线的完整的工作过程：

  ![20190512155453.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/700f368c770949178ba4dcf7835022e1~tplv-k3u1fbpfcp-zoom-1.image)

  > 
  
  > **流程说明**：
  >
  > - **阶段一. Checkout SCM**: 拉取 GitHub 仓库代码
  > - **阶段二. Unit test**: 单元测试，如果测试通过了才继续下面的任务
  > - **阶段三. SonarQube analysis**：sonarQube 代码质量检测
  > - **阶段四. Build & push snapshot image**: 根据行为策略中所选择分支来构建镜像，并将 tag 为 `SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER`推送至 Harbor (其中 `$BUILD_NUMBER`为 pipeline 活动列表的运行序号)
  > - **阶段五. Push latest image**: 将 master 分支打上 tag 为 latest，并推送至 DockerHub
  > - **阶段六. Deploy to dev**: 将 master 分支部署到 Dev 环境，此阶段需要审核
  > - **阶段七. Push with tag**: 生成 tag 并 release 到 GitHub，并推送到 DockerHub
  > - **阶段八. Deploy to production**: 将发布的 tag 部署到 Production 环境

#### 创建凭证

> **注意**：
>
> - GitHub 账号或密码带有 "@" 这类特殊字符，需要创建凭证前对其进行 urlencode 编码，可通过一些 [第三方网站](http://tool.chinaz.com/tools/urlencode.aspx)进行转换，然后再将转换后的结果粘贴到对应的凭证信息中。
> - 这里需要创建的是凭证（Credential），不是密钥（Secret）

- 1、本示例代码仓库中的 Jenkinsfile 需要用到 **DockerHub、GitHub** 和 **kubeconfig** (kubeconfig 用于访问接入正在运行的 Kubernetes 集群) 等一共 3 个凭证 (credentials) ，参考 [创建凭证](https://v2-1.docs.kubesphere.io/docs/zh-CN/devops/credential/#创建凭证) 依次创建这三个凭证

  ![image-20210318084638288](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f51c071d08e74ce48e2d2a4e75bcc846~tplv-k3u1fbpfcp-zoom-1.image)

- 2、然后参考 [访问 SonarQube 并创建 Token](https://v2-1.docs.kubesphere.io/docs/zh-CN/devops/sonarqube)，创建一个 Java 的 Token 并复制

  - 2.1、查看内置安装的 SonarQube

    ```
    kubectl get svc --all-namespaces
    
    ```

    可以看到以NodePort的形式暴露了32303，任意节点ip访问32303端口，admin&admin登录


- ![image-20210318085007211](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9518d2b7238b48fea32aae71574a7517~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210318085315560](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/330faaf48160455c9e5808fd7aba3fb8~tplv-k3u1fbpfcp-zoom-1.image)



- 3、最后在 KubeSphere 中进入 `devops-demo`的 DevOps 工程中，与上面步骤类似，在 **凭证** 下点击 **创建**，创建一个类型为 `秘密文本`的凭证，凭证 ID 命名为 **sonar-token**，密钥为上一步复制的 token 信息，完成后点击 **确定**

- ![image-20210318085454892](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd9203df30424cf785c9109f4a7a290d~tplv-k3u1fbpfcp-zoom-1.image)

##### 

#### 修改jenkinsfile

- 第一步：Fork项目：登录 GitHub，将本示例用到的 GitHub 仓库 [devops-java-sample](https://github.com/kubesphere/devops-java-sample) Fork 至您个人的 GitHub

- 第二步：修改 Jenkinsfile：

  - 2.1、Fork 至您个人的 GitHub 后，在 **根目录** 进入 **Jenkinsfile-online**

  - 2.2、在 GitHub UI 点击编辑图标，需要修改如下环境变量 (environment) 的值

    ![image-20210318085921561](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6abbe2530cb4b859a2479270161b2ab~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210318085841798](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25cb5c87ee8441de871f83acae9e9991~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210318085841798](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25cb5c87ee8441de871f83acae9e9991~tplv-k3u1fbpfcp-zoom-1.image)

    第一次使用注意去掉 `-o` 参数

  - 2.3、修改以上的环境变量后，点击 **Commit changes**，将更新提交到当前的 master 分支

  - 2.4、补充：创建完sonarqube的token之后，需要点击`continue`按钮，选择接下来将要分析的代码语言：这里选择 `Java&Maven`，然后点击finish
  
    ![image-20210318091454519](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85454653d54a468f941dcb12d1442b1c~tplv-k3u1fbpfcp-zoom-1.image)

#### 创建项目

- CI/CD 流水线会根据示例项目的 [yaml 模板文件](https://github.com/kubesphere/devops-java-sample/tree/master/deploy)，最终将示例分别部署到 `kubesphere-sample-dev`和 `kubesphere-sample-prod`这两个项目 (Namespace) 环境中，这两个项目需要预先在控制台依次创建，参考如下步骤创建该项目

- 1、第一步：创建第一个项目：

  - 1.1、使用项目管理员 

    ```
    project-admin
    ```

    账号登录 KubeSphere，在之前创建的企业空间 (demo-workspace) 下，点击 

    项目 → 创建

    ，创建一个 

    资源型项目

    ，作为本示例的开发环境，填写该项目的基本信息，完成后点击 

    下一步

    。

    - 名称：固定为 `kubesphere-sample-dev`，若需要修改项目名称则需在 [yaml 模板文件](https://github.com/kubesphere/devops-java-sample/tree/master/deploy) 中修改 namespace
    - 别名：可自定义，比如 **开发环境**
    - 描述信息：可简单介绍该项目，方便用户进一步了解

  - 1.2、本示例暂无资源请求和限制，因此高级设置中无需修改默认值，点击 **创建**，项目可创建成功

  ![image-20210318092243034](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/158fc9a754294dfa859a6432bd9d0e16~tplv-k3u1fbpfcp-zoom-1.image)

- 2、第二步：邀请成员：第一个项目创建完后，还需要项目管理员 `project-admin`邀请当前的项目普通用户 `project-regular`进入 `kubesphere-sample-dev`项目，进入「项目设置」→「项目成员」，点击「邀请成员」选择邀请 `project-regular`并授予 `operator`角色，若对此有疑问可参考 [多租户管理快速入门 - 邀请成员](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/admin-quick-start/#邀请成员)

  - ![image-20210318092105759](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9d00deeef884f7a881f5933b3656520~tplv-k3u1fbpfcp-zoom-1.image)

#### 创建流水线

- 切换`project-regular`身份登录

  [创建流水线，官方文档很详细](https://v2-1.docs.kubesphere.io/docs/zh-CN/quick-start/devops-online/#创建流水线)

- 创建完成，可以通过查看 扫描仓库日志，查看失败原因或者拉取进度

  - ![image-20210318094301572](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51db46ec7d17402e9f4493216545fcea~tplv-k3u1fbpfcp-zoom-1.image)
  
  

#### 运行流水线

- > **注意**：这里由于github上面的项目一直拉取失败，我将项目导入gitee，并修改了jenkinsfile

  - 需要修改的地方

    - 1、删除刚刚创建并运行失败的流水线，新建流水线，选择`Git`

      ![image-20210318113437903](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/195be7eb859d46fa8a1e5eea4f3b090c~tplv-k3u1fbpfcp-zoom-1.image)

    - 2、新增gitee凭证（同github）修改jenkinsfile，将file中的github-id 替换为 gitee-id，github_accout的值也改为gitee的账户名称

    ![image-20210318112728336](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70fb77ebf2d34bfc98da8463bdf5420e~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210318112925778](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10ff453459a74d2fa698e1d7fe79161c~tplv-k3u1fbpfcp-zoom-1.image)

- 查看运行日志

  ![image-20210318113122946](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/637ab932e776486ebc18164546c90914~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210318113137724](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7693d7aef71c400e9c23a94552e809eb~tplv-k3u1fbpfcp-zoom-1.image)

  审核流水线

  ![image-20210318132621954](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23ab0c2afa0b4dc0a38e82e62700f06d~tplv-k3u1fbpfcp-zoom-1.image)

#### 查看流水线

- 1、点击流水线中 `活动`列表下当前正在运行的流水线序列号，页面展现了流水线中每一步骤的运行状态，注意，流水线刚创建时处于初始化阶段，可能仅显示日志窗口，待初始化 (约一分钟) 完成后即可看到流水线。黑色框标注了流水线的步骤名称，示例中流水线共 8 个 stage，分别在 [Jenkinsfile-online](https://github.com/kubesphere/devops-java-sample/blob/master/Jenkinsfile-online) 中被定义

  ![image-20210318144348310](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1140d67445645dba4f13b02685449ad~tplv-k3u1fbpfcp-zoom-1.image)

- 2、当前页面中点击右上方的 `查看日志`，查看流水线运行日志。页面展示了每一步的具体日志、运行状态及时间等信息，点击左侧某个具体的阶段可展开查看其具体的日志。日志可下载至本地，如出现错误，下载至本地更便于分析定位问题

  ![image-20210318144421333](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/807073e033164b43be7934499be5627a~tplv-k3u1fbpfcp-zoom-1.image)

#### 验证流水线

- 1、若流水线执行成功，点击该流水线下的 `代码质量`，即可看到通过 sonarQube 的代码质量检测结果

  ![image-20210318145617057](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d5ff1b6acd74580b73dc80693913675~tplv-k3u1fbpfcp-zoom-1.image)

- 2、流水线最终 build 的 Docker 镜像也将被成功地 push 到 DockerHub 中，我们在 Jenkinsfile-online 中已经配置过 DockerHub，登录 DockerHub 查看镜像的 push 结果，可以看到 tag 为 snapshot、TAG_NAME(master-1)、latest 的镜像已经被 push 到 DockerHub，并且在 GitHub 中也生成了一个新的 tag 和 release。演示示例页面最终将以 deployment 和 service 分别部署到 KubeSphere 的 `kubesphere-sample-dev`和 `kubesphere-sample-prod`项目环境中

  ![image-20210318145259641](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96149ca93ff547d09b6d76660857acb9~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210318145926029](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c11a0cea48244839dc799a67ae7a0e8~tplv-k3u1fbpfcp-zoom-1.image)

- 3、可通过 KubeSphere 回到项目列表，依次查看之前创建的两个项目中的部署和服务的状态。例如，以下查看 `kubesphere-sample-prod`项目下的部署。

  进入该项目，在左侧的菜单栏点击 **工作负载 → 部署**，可以看到 ks-sample 已创建成功。正常情况下，部署的状态应该显示 **运行**

  在菜单栏中选择 **网络与服务 → 服务** 也可以查看对应创建的服务，可以看到该服务的 Virtual IP 为 `10.233.42.3`，对外暴露的节点端口 (NodePort) 是 `30961`

  ![image-20210318145419621](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bf3e7dd569e43ec91fc80aac90af3a4~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210318145949451](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e6409268f034a3bb2948a5c17091371~tplv-k3u1fbpfcp-zoom-1.image)

- 4、查看推送到您个人的 DockerHub 中的镜像，可以看到 `devops-java-sample`就是 APP_NAME 的值，而 tag 也是在 jenkinsfile-online 中定义的 tag

  ![image-20210318145039270](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45722b931acd4472b8a4d57183a011dc~tplv-k3u1fbpfcp-zoom-1.image)

- 5、点击 `release`，查看 Fork 到您个人 GitHub（Gitee） repo 中的 `v0.0.1`tag 和 release，它是由 jenkinsfile 中的 `push with tag`生成的

  ![image-20210318145005330](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6440ea21c264c87a42539c0c76819e6~tplv-k3u1fbpfcp-zoom-1.image)
