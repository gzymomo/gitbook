整理自：[云原生 Meetup | KubeSphere & Friends 2021-上海站](https://mudu.tv/live/watch/general?id=me1ezxbl&time=1621058977484



# KubeSphere带你远航

周小四：KubeSphere容器平台研发负责人

# 什么是云原生？

## 狭义

云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。云原生的代表技术包括容器、服务网格、微服务、不可变基础设施和声明式API。这些技术能够构建容错性好、易于管理和便于观察的松精合系统。结合可靠的自动化手段，云原生技术使工程师能够轻松地对系统作出频繁和可预测的重大变更。

## 广义

充分利用云计算的交付方式创建、部署、运行应用的一种途径。

区别于cloud-hosted或cloud-based,如Snowflake。

# KubeSphere产品向产品家族的演进

![image-20210515142225517](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515142225517.png)



![image-20210515142358039](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515142358039.png)





# 混合云霞的K8S多集群管理尉应用部署

李宇：KubeSphere研发工程师

## 单集群

![image-20210515142619231](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515142619231.png)



## 多集群场景

![image-20210515142653986](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515142653986.png)



## 多集群现有的解决方案

Control Plane

- Federation V1(Deprecated)
- Federation V2(Kubefed)
- GitOps（ArgoCD/FluxCD）



Network Centric

- Cilium Mesh
- Istio Multi-Cluster
- Linkerd Service Mirroring

## KubeSphere with Kubefed

![image-20210515143025959](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143025959.png)

![image-20210515143338423](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143338423.png)



![image-20210515143355359](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143355359.png)

## Tower工作流程

![image-20210515143446039](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143446039.png)

## 多集群下的多租户支持

![image-20210515143508031](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143508031.png)



![image-20210515143523088](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143523088.png)

## 统一单集群/多集群ns管理

![image-20210515143546559](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143546559.png)



## 应用一键部署 可配置的集群差异化设置 屏蔽kubefed底层

![image-20210515143620775](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143620775.png)

## 联邦资源的状态收集

![image-20210515143647903](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143647903.png)



## KubeSphere多集群架构

![image-20210515143716830](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143716830.png)

## liqo 去中心化的多集群解决方案

![image-20210515143823655](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143823655.png)

![image-20210515143836526](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143836526.png)

![image-20210515143852952](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143852952.png)



# KubeSphere在媒体直播行业的实践

唐明：苏州广播电视总台	系统架构师/运维负责人

## IT进化论

![image-20210515143958488](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515143958488.png)

## KubeSphere的应用——慢直播

实现音频文件与视频流的混合，推送至直播平台

技术难点：信号数量多，信源不稳定，随时会变化

方案迭代

1. 尝试使用ffmpeg进行编码
2. 使用docker以容器方式运行
3. 使用KubeSphere进行统一管理



KubeSphere的作用：

- 统一管理
- 任务编排
- 状态监控
- 出错报警
- 日志统计

![image-20210515144152389](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515144152389.png)

# 在云原生场景下构建企业级存储方案

杨兴祥：QingStor 高级软件工程师

## 云原生存储的挑战

![image-20210515144302237](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515144302237.png)

## 常见云原生存储解决方案

![image-20210515144340997](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515144340997.png)



## NeonIO为什么适合云原生存储

NeonIO：一款支持容器化部署的企业级分布式块存储系统，能够给Kubernetes平台上提供动态创建（dynamic provisioning）持久存储卷（persistent volume）的能力。

![image-20210515144513055](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515144513055.png)



### 易用性

- 组件容器化：服务组件、CSI、Portal容器化
- 支持CSI：提供标准的IO接入能力，可静态、动态创建PV
- UI界面，运维方便：存储运维操作界面化、告警、监控可视管理，有基于PV粒度的性能监控，如IOPS、吞吐量，快速定位到热点PV，有基于PV粒度的Qos

![image-20210515144708534](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515144708534.png)

### 与云原生高度融合

1. 支持Promethus:通过ServiceMonitor把NeonlO的采集指标暴露给Promethus,同时UI界面可与Promethus对接，展示其他云原生监控的指标，比如node-exporter采集到节点磁盘IO负载

2. 平台化的运维方式：存储的扩容、升级、灾难恢复运维操作、只需要k8s的一些命令即可实现，K需要额外掌握过多的存储相关的运维知识
3. 服务发现、分布式协调支持etcd、元数据的管理，使用CRD的方式；

### 一键式部署

`helm install neonio ./neonio --namespace kube-system`

### 部署灵活

充分利用k8s的编排能力，部署简单、灵活

![image-20210515145045709](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515145045709.png)

### 高性能：单卷IOPS 10K，时延 亚毫秒

全闪的分布式存储架构

1. 集群中所有节点共同承担压力，IO性能随着节点增加而线性增长
2. 存储介质支持NVME SSD
3. 支持RDMA：通过高速的RDMA技术将节点连接

### 极短的IO路径

抛弃文件系统，自研元数据管理系统，使IO路径极短

![image-20210515145309687](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515145309687.png)

### HostNetwork

优势：Strore CSI Pod使用HostNetwork，直接使用物理网络，减少网络层次

管理网络、前端网络、数据同步网络分离，避免网络竞争；

![image-20210515145413302](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515145413302.png)

### 高可用

服务组件可靠性与可用性

1. NeonIO管理服务默认使用3副本POD
2. 使用探针检测POD服务是否可用，是否存活，检测到POD服务部可用剔除组件服务，检测到POD死掉后重启POD，使其重新启动服务

### 数据的可靠性与可用性

![image-20210515145616324](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515145616324.png)

### 敏捷性

Pod跨节点重建高效：2000PV的挂载/卸载16s

批量创建PV能力：2000PV的创建5min

## NeonIO应用场景

- Devops场景：批量快速创建/销毁PV能力，2000PV创建5min
- 数据库场景：WEB网站后端数据库MySQL等提供稳定的持久化存储，提供高IOPS、低时延
- 大数据应用分析场景：提供超大容量，PV可扩容到100TB



# 中通快递关键业务和复杂架构挑战下的Kubernetes集群服务暴露实践

王文虎：ZKE容器平台研发

## ZKE容器管理平台

![image-20210515145927964](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515145927964.png)

## Kubernetes集群服务暴露方案

1. Dubbo服务之间访问
2. 泛域名方式访问
3. 自定义域名方式访问

## Kubernetes集群服务暴漏方案

### Dubbo服务之间访问

为兼容dubbo服务虚拟机和容器混布场景，通过bgp协议打通了pod网段和物理网络，解决了dubbo服务互相调用问题。

![image-20210515150129579](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515150129579.png)



### 泛域名方式访问

1. 减少域名运维工作量
2. 用户方便定义域名
3. 适用于开发和测试环境

![image-20210515150219204](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210515150219204.png)

### 自定义域名访问

1. 用户需要https域名
2. 用户对域名格式有要求
3. 适用于生产环境

## 服务暴漏方案踩坑实践

### Ingress Nginx Controller服务踩坑实践

![image-20210517083526798](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083526798.png)

![image-20210517083540287](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083540287.png)

![image-20210517083600901](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083600901.png)

### Calico关闭natOutgoing

![image-20210517083626590](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083626590.png)



# MySQL on K8S 开源开放的高可用容器编排方案

高日耀：资深MySQL内核研发

## 为什么要做MySQL容器化？

![image-20210517083809141](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083809141.png)

## MySQL容器化需要考虑的问题

![image-20210517083833487](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083833487.png)

![image-20210517083843442](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517083843442.png)

## 什么是RadonDB?

RadonDB现已升级为青云数据库产品的品牌。

涵盖MySQL、PostgreSQL、ClickHouse等主流数据库。

## RadonDB MySQL容器化

在KubeSphere和Kubernetes上安装部署和管理

自动执行尉RadonDB MySQL集群有关的任务。

## 什么是RadonDB MySQL？

基于MySQL的开源、高可用、云原生集群解决方案。

支持一主多从高可用架构。

![image-20210517084100095](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084100095.png)



![image-20210517084115777](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084115777.png)

## 部署效果

![image-20210517084130578](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084130578.png)

# 基于云原生架构下的DevOps实践

蒋立杰：江苏苏宁银行云计算负责人

## 传统DevOps如何技术演进为云原生DevOps

![image-20210517084318645](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084318645.png)



## 苏宁银行云平台总体架构

![image-20210517084350294](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084350294.png)

## 苏宁银行DevOps总体架构

![image-20210517084406868](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084406868.png)

## 苏宁银行DevOps系统选型分析

![image-20210517084501059](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084501059.png)

![image-20210517084515130](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084515130.png)

## 苏宁银行Tekton+ArgoCD实践-场景

![image-20210517084543945](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084543945.png)

![image-20210517084556867](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084556867.png)

## 基于Argocd-Rollouts进行blue/green canary发布

![image-20210517084627764](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084627764.png)

## 新旧灰度对比

![image-20210517084646974](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210517084646974.png)