# Rancher RKE 和 KubeOperator 的对比

|            |          | KubeOperator v2.4.43                                         | Rancher RKE v1.0.6                                           | 备注（RKE + Rancher）                                        |
| ---------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Day 0 规划 | 集群模式 | 1 个 Master 节点 n 个 Worker 节点模式：适合开发测试用途；    | 节点分为 controlplane / worker / etcd 三种角色               | 未看到确切节点规划                                           |
|            |          | 3 个 Master 节点 n 个 Worker 节点模式：适合生产用途          |                                                              |                                                              |
|            | 计算方案 | 独立主机：支持自行准备的虚机、公有云主机和物理机             | 独立主机：支持自行准备的虚机、公有云主机和物理机             |                                                              |
|            |          | vSphere 平台：支持自动创建主机（使用 Terraform）             | vSphere 平台：不支持                                         | 整合在 Rancher                                               |
|            |          | Openstack 平台：支持自动创建主机 （使用 Terraform）          | Openstack 平台：不支持                                       | 整合在 Rancher                                               |
|            | 存储方案 | 独立主机：支持 NFS / Ceph RBD / Local Volume                 | 无                                                           | 存储相关信息只存在于 Rancher，RKE 中未看到任何有关存储的说明 |
|            |          | vSphere 平台：支持 vSphere Datastore （vSAN 及 vSphere 兼容的集中存储） |                                                              |                                                              |
|            |          | Openstack 平台：支持 Openstack Cinder （Ceph 及 Cinder 兼容的集中存储） |                                                              |                                                              |
|            | 网络方案 | 支持 Flannel / Calico 网络插件                               | 支持 Flannel / Calico / Canal / Weave 网络插件               | F5 Big IP 与 Traefik 存在于                                  |
|            |          | 支持通过 F5 Big IP 对外暴露服务                              |                                                              | Rancher 中                                                   |
|            |          | 支持 Traefik                                                 |                                                              |                                                              |
|            |          | 支持 CoreDNS                                                 | 支持 CoreDNS                                                 |                                                              |
|            | GPU 方案 | 支持 NVIDIA GPU                                              | 不支持                                                       | Rancher 上也未来到相关功能                                   |
|            | 操作系统 | 支持 CentOS 7.4 / 7.5 / 7.6 / 7.7                            | CentOS 7 / RHEL 7 / Ubuntu / Windows Sever 等支持 docker 的几乎所有系统 |                                                              |
| Day 1 部署 | 部署     | 提供离线环境下的完整安装包                                   | 离线部署需自行下载镜像，并搭建私有仓库                       | 部署前需手动配置系统环境                                     |
|            |          | 支持可视化方式展示部署过程                                   | 不提供可视化部署                                             | 部署时需先手动对节点角色，IP 等信息                          |
|            |          | 支持一键自动化部署（使用 Ansible）                           | 不支持一键部署                                               | 手动生成 yml 文件                                            |
| Day 2 运营 | 管理     | 支持用户权限管理，支持对接 LDAP/AD                           | 无                                                           | 管理功能整合在 Rancher                                       |
|            |          | 对外开放 REST API                                            |                                                              |                                                              |
|            |          | 内置 K8s Dashboard 管理应用                                  |                                                              |                                                              |
|            |          | 内置 Weave Scope 管理应用                                    |                                                              |                                                              |
|            |          | 提供 Web Kubectl 界面                                        |                                                              |                                                              |
|            |          | 内置 Helm                                                    |                                                              |                                                              |
|            | 可观察性 | 内置 Promethus，支持对集群、节点、Pod、Container的全方位监控和告警 | 无                                                           | 整合在 Rancher                                               |
|            |          | 内置 Loki 日志方案                                           |                                                              |                                                              |
|            |          | 内置 Grafana 作为监控和日志展示                              |                                                              |                                                              |
|            |          | 支持消息中心，通过钉钉、微信通知各种集群异常事件             |                                                              |                                                              |
|            | 升级     | 支持集群升级                                                 | 支持集群升级                                                 |                                                              |
|            | 伸缩     | 支持增加或者减少 Worker 节点                                 | 支持对 worker 和 controlplane 节点增加或减少                 |                                                              |
|            | 备份     | 支持 etcd 定期备份                                           | 支持 etcd 定期备份                                           |                                                              |
|            | 合规     | 支持集群合规检查并可视化展示结果                             | 无                                                           | 整合在 Rancher                                               |
|            | 应用商店 | 集成 KubeApps Plus 应用商店，快速部署 CI/CD、AI 深度学习等应用 | 无                                                           | 整合在 Rancher                                               |