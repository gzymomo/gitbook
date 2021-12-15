# 第1章:Kubernetes运维管理

赠送1:从0搭建一个生产级K8s高可用集群(二进制)

赠送2:Ansible快速入门课(5小时)

1. Kubernetes容器云平台技术落地方案
2. Ansible自动化部署一个完整的K8s 集群
3. Kubernetes高可用方案之Master组件
4. Kubernetes高可用方案之Etcd数据库
5. Kubernetes数据库 Etcd备份与恢复
6. Kubelet证书自动续签，完美解决一年证书过期问题
7. Kubernetes集群常见故障排查思路


# 第2章:Kubernetes 弹性伸缩

1. 传统弹性伸缩的困境
2. 弹性伸缩的布局
3. Nodes资源扩容
   1. Cluster autoscaler
   2. Ansible
4. Pods资源扩容
   - HPA泉理与演进
   - 基于Metrics Server的HPA基于Prometheus的HPA
5. 弹性伸缩目前成熟的落地方案



# 第3章:Kubernetes应用包管理器Helm (v3)

1. 为什么需要Helm?
2.  Helm介绍
3. Helm v3重要里程碑
4. Helm客户端
5. Helm基本使用（部署应用、自定义选项、升级、回滚、删除)
6. Chart模板
   - 模板概述
   - 内置对象
   - Values
   - 管道与函数
   - 流程控制(if、with、range)
   - 变量
   - 命名模板
7. 手把手从零教你制作Chart: Java应用为例
8. 使用Harbor 作为Chart仓库



# 第4章 Kubernetes集群网络

1. 网络基础知识
   - 互联网组成
   - 交换技术、路由技术
   - OSI七层协议
   - TCP/UDP协议
2. Kubernetes网络方案之Flannel剖析
   - Docker网络模型与K8S网络模型
   - Flannel概述与描述
   - Flannel VXLAN模式及原理解析
   - Flannel Host-GW模式及原理解析
3. Kubernetes网络方案之Calico解析
   - Calico概述与部署
   - Calico BGP模式及原理解析
   - IPIP隧道模式及原理解析
   - Route Relector(RR)方案
   - 办公网络与K8S内部网络互通方案
   - CNI网络方案优缺点及最终选择
4. 网络策略（Pod ACL）

- - 为什么需要网络隔离？
  - 网络策略描述
  - 入站、出站网络流量访问控制案例



# 第5章:Kubernetes存储之Ceph分布式存储系统

1. Ceph系统架构及核心组件
2. 生产环境Ceph集群服务器规划
3. Ceph三大客户端介绍
4. Ceph核心概念（Crush、Pool、PG、OSD、Object）
5. 部署Ceph集群

- - Ceph版本选择
  - 安装ceph-deploy工具
  - ceph.conf配置文件
  - MON/OSD/MDS/MGR组件部署

6. RBD客户端安装与使用

- - RBD的工作原理与配置
  - RBD的常用命令
  - 客户端挂载RBD的几种方式

7. CephFS客户端安装与使用

- - CephFS的工作原理与配置
  - 挂载CephFS的几种方式

8. K8S使用Ceph作为存储

- - PV，PVC概述
  - PV自动供给
  - Pod使用RBD作为持久数据卷
  - Pod使用CephFS作为持久数据卷

9. Ceph监控

- - 内置Dashboard
  - Prometheus+Grafana监控Ceph

10. Ceph日常运维管理及常见问题



# 第6章:SpringCloud微服务容器化迁移

1. 从运维角度看微服务
2. 在K8S平台部署微服务项目应该考虑什么问题？
3. 在K8S中部署SpringCloud微服务想买

- - 第一步：熟悉SpringCloud微服务项目
  - 第二步：源代码编译构建
  - 第三步：构建项目镜像并推送到镜像仓库
  - 第四步：K8S服务编排
  - 第五步：在K8S中部署Eureka集群（注册中心）
  - 第六步：部署微服务网关服务
  - 第七步：部署微服务业务程序
  - 第八步：部署微服务前端
  - 第九步：微服务对外发布

4. 微服务链路监控系统

- - 全链路监控是什么？
  - 全链路监控解决什么问题？
  - 全链路监控系统选择依据
  - Pinpoint介绍
  - Pinpoint服务端部署
  - Pinpoint Agent部署
  - Pinpoint图形页面及重点关注指标

# 第7章:基于K8S构建Jenkins微服务发布平台

1. 发布流程设计及注意事项
2. 使用Gitlab作为代码仓库
3. 使用Harbor作为容器镜像仓库
4. 在K8S中部署Jenkins
5. Jenkins Pipeline基本使用
6. Jenkins Pipeline参数化构建
7. Jenkins在K8S中动态创建代理
8. 自定义构建Jenkins Slave镜像
9. Pipeline集成Helm发布微服务项目
10. 技术栈：Jenkins+Kubernetes+Gitlab+Helm+Springcloud



# 第8章:微服务治理Istio初探

1. Service Mesh
2. Istio概述
3. Istio使用场景
4. Istio架构与组件
5. 在Kubernetes中部署Istio
6. Sidecat注入（应用接入Istio）
7. 服务网关：Gateway
8. 服务网关：Gateway基于域名访问多个项目
9. 经典Bookinfo微服务案例
10. 主流发布方案概述：蓝绿、滚动、灰度（金丝雀和A/B Test）
11. Istio实现金丝雀发布和A/B Test发布
12. 可视化监控：监控指标、网络可视化、调度链跟踪



