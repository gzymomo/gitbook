## Cilium 简介

Cilium 是一个用于容器网络领域的开源项目，主要是面向容器而使用，用于提供并透明地保护应用程序工作负载（如应用程序容器或进程）之间的网络连接和负载均衡。

Cilium 在第 3/4 层运行，以提供传统的网络和安全服务，还在第 7 层运行，以保护现代应用协议（如 HTTP, gRPC 和 Kafka）的使用。Cilium 被集成到常见的容器编排框架中，如 Kubernetes 和 Mesos。

Cilium 的底层基础是 BPF，Cilium 的工作模式是生成内核级别的 BPF 程序与容器直接交互。区别于为容器创建 overlay  网络，Cilium 允许每个容器分配一个 IPv6 地址（或者 IPv4  地址），使用容器标签而不是网络路由规则去完成容器间的网络隔离。它还包含创建并实施 Cilium 规则的编排系统的整合。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicRroK9oXnOWGukTA2sanyWTsqMEPxvibl4kib2ZgXr2J9lDMR4hX93C0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 以上简介来源于 OSCHINA

对 Cilium 的性能比较感兴趣的读者可以参考这篇文章：



[![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWic9nevzBt0OcDeaUxh2v1LBVQVXicZOHn1lwXwNBYEmtYibL2icZLYJ4RDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](http://mp.weixin.qq.com/s?__biz=Mzg4NTU0MzEyMg==&mid=2247487213&idx=1&sn=b075da3a3ab61aec5fd4098cf874ae2f&chksm=cfa61750f8d19e4635d90d4dee63720282968feeb4d15402852d62531e329a4a7c12cceb03b8&scene=21#wechat_redirect)

最强 CNI 基准测试：Cilium 网络性能分析（点击上方图片即可访问）

## KubeKey 介绍

KubeKey（由 Go 语言开发）是一种全新的安装工具，替代了以前使用的基于 ansible 的安装程序。KubeKey 为您提供灵活的安装选择，您可以仅安装 Kubernetes，也可以同时安装 Kubernetes 和 KubeSphere。

KubeKey 的几种使用场景：

- 仅安装 Kubernetes；
- 使用一个命令同时安装 Kubernetes 和 KubeSphere；
- 扩缩集群；
- 升级集群；
- 安装 Kubernetes 相关的插件（Chart 或 YAML）。

KubeKey 目前已集成和支持使用 Cilium 作为 CNI 来部署 Kubernetes 和 KubeSphere 集群，使用 KubeKey 来部署 Cilium 将会更加简单高效，极大地节省安装时间。

## 系统要求

Linux Kernel >= 4.9.17 更多信息请查看 **Cilium 系统要求**[1]

## 环境

以一台 Ubuntu Server 20.04.1 LTS 64bit 为例。

| name  | ip           | role                 |
| :---- | :----------- | :------------------- |
| node1 | 10.160.6.136 | etcd, master, worker |

## 下载安装包

```
sudo wget https://github.com/kubesphere/kubekey/releases/download/v1.1.0/kubekey-v1.1.0-linux-64bit.deb
```

## 使用 cilium 作为网络插件部署 KubeSphere

1.安装 KubeKey。

```
sudo dpkg -i kubekey-v1.1.0-linux-64bit.deb
```

2.生成配置文件。

```
sudo kk create config --with-kubernetes v1.19.8
```

3.修改配置文件，将网络插件修改为 cilium 注意将 spec.network.plugin 的值修改为 **cilium。**

```
sudo vi config-sample.yaml
apiVersion: kubekey.kubesphere.io/v1alpha1
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: node1, address: 10.160.6.136, internalAddress: 10.160.6.136, user: ubuntu, password: ********}
  roleGroups:
    etcd:
    - node1
    master:
    - node1
    worker:
    - node1
  controlPlaneEndpoint:
    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.19.8
    imageRepo: kubesphere
    clusterName: cluster.local
  network:
    plugin: cilium
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    registryMirrors: []
    insecureRegistries: []
  addons: []
```

4.部署依赖。

```
sudo kk init os -f config-sample.yaml
```

5.部署 KubeSphere。

```
sudo kk create cluster -f config-sample.yaml --with-kubesphere v3.1.0
```

看到如下提示说明安装完成。

```
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://10.160.6.136:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2021-05-18 17:15:03
#####################################################
INFO[17:15:16 CST] Installation is complete.
```

6.登陆 KubeSphere console。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicicYZnUguffJUic5mEQEIia0ia8R63HYBwHS5k0S2rXV9KohMU4cyVrKTJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

7.检查状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicPENPmTohXyMQIIdiaibQPayJjpQR3nicuJAibuhQPq23mzGicEoIBVhN2pw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 安装 hubble UI

Hubble 是专门为网络可视化设计的，能够利用 Cilium 提供的 eBPF 数据路径，获得对 Kubernetes  应用和服务的网络流量的深度可见性。这些网络流量信息可以对接 Hubble CLI、UI 工具，可以通过交互式的方式快速诊断如与 DNS  相关的问题。除了 Hubble 自身的监控工具，还可以对接主流的云原生监控体系——Prometheus 和  Grafana，实现可扩展的监控策略。

Hubble 的安装很简单，直接执行以下命令：

```
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.9.7/install/kubernetes/quick-hubble-install.yaml
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWic3kxwiaEOAPm3JJBtKKrYkJVzXn1DJIz16UbjAic9g1shqTsDbsC71cyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

检查状态。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicRK7OBCz3NdJbChjBcnJI1S1uDia5iaucOkLZnDzk1wCdMEicXvKXuTojA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 安装 demo 服务，并在 hubble UI 查看服务依赖关系

1.安装 demo。

```
kubectl create -f https://raw.githubusercontent.com/cilium/cilium/1.9.7/examples/minikube/http-sw-app.yaml
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicCsHBWQPpItAtq2Vzv80Lia3JLQ6R5nGSp17mlj0lsB9x4Gkv09qc6VQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2.将 hubble UI 服务类型修改为 NodePort。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWic4icrmrPIcyJ2NSDgdrTU39cYKkWYNhZbibNR4Hqzxs9ccbwqPKvWpukA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicQDQNzF1YDrnNKVFeQicC8TsALibicZFVxh1WarwUibFCSYU5siamic4HjhIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3.访问 demo

```
kubectl exec xwing -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
kubectl exec tiefighter -- curl -s -XPOST deathstar.default.svc.cluster.local/v1/request-landing
Ship landed
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWic4MqoRmibib6G1RWFANmKjDAicpKRkzhO6icviaY0YyrN9CEK73M9TI7cdfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4.在 hubble 上 查看服务依赖关系。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEXrHiafhZlwqMZQq0EXsNQWicGUibzlDJiaoKwrJyZzic3S2dgezBzK906yUZXoSibs8u82D7Fp6kWgraYQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果想开启网络 7 层的可视化观察，就需要对目标 Pod 进行 annotations ，感兴趣可以看**Cilium 的官方文档**[2]。

## 总结

从使用体验来看，Cilium 已经可以满足绝大多数的容器网络需求，特别是 Hubble 使用原生的方式实现了数据平面的可视化，比 Istio 高明多了。相信用不了多久，Cilium 便会成为 Kubernetes 社区使用最多的网络方案。

### 脚注

[1]Cilium 系统要求: *https://docs.cilium.io/en/v1.9/operations/system_requirements/*

[2]Cilium 的官方文档: *http://docs.cilium.io/en/stable/policy/visibility/*