- [KubeKey](https://gitee.com/kubesphere/kubekey)

# 一、软件简介

[KubeKey](https://github.com/kubesphere/kubekey)（由 Go 语言开发）是一种全新的安装工具，替代了以前使用的基于 ansible 的安装程序。KubeKey 为您提供灵活的安装选择，您可以仅安装 Kubernetes，也可以同时安装 Kubernetes 和 KubeSphere。

KubeKey 的几种使用场景：

- 仅安装 Kubernetes；
- 使用一个命令同时安装 Kubernetes 和 KubeSphere；
- 扩缩集群；
- 升级集群；
- 安装 Kubernetes 相关的插件（Chart 或 YAML）。

# 二、KubeKey 如何运作

下载 KubeKey 之后，您可以使用可执行文件 `kk` 来进行不同的操作。无论您是使用它来创建，扩缩还是升级集群，都必须事先使用 `kk` 准备配置文件。此配置文件包含集群的基本参数，例如主机信息、网络配置（CNI 插件以及 Pod 和 Service CIDR）、仓库镜像、插件（YAML 或 Chart）和可插拔组件选项（如果您安装 KubeSphere）。有关更多信息，请参见[示例配置文件](https://github.com/kubesphere/kubekey/blob/master/docs/config-example.md)。

准备好配置文件后，您需要使用 `./kk` 命令以及不同的标志来进行不同的操作。这之后，KubeKey 会自动安装 Docker，并拉取所有必要的镜像以进行安装。安装完成后，您还可以检查安装日志。

## 2.1 为什么选择 KubeKey

- 以前基于 ansible 的安装程序依赖于许多软件，例如 Python。KubeKey 由 Go 语言开发，可以消除在多种环境中出现的问题，确保成功安装。
- KubeKey 支持多种安装选项，例如 [All-in-One](https://kubesphere.com.cn/docs/quick-start/all-in-one-on-linux/)、[多节点安装](https://kubesphere.com.cn/docs/installing-on-linux/introduction/multioverview/)以及[离线安装](https://kubesphere.com.cn/docs/installing-on-linux/introduction/air-gapped-installation/)。
- KubeKey 使用 Kubeadm 在节点上尽可能多地并行安装 Kubernetes 集群，使安装更简便，提高效率。与旧版的安装程序相比，它极大地节省了安装时间。
- KubeKey 旨在将群集作为对象来进行安装，即 CaaO。

## 2.2 下载 KubeKey

- [如果您能正常访问 GitHub/Googleapis](https://kubesphere.com.cn/docs/installing-on-linux/introduction/kubekey/#)
- [如果您访问 GitHub/Googleapis 受限](https://kubesphere.com.cn/docs/installing-on-linux/introduction/kubekey/#)

从 [GitHub Release Page](https://github.com/kubesphere/kubekey/releases) 下载 KubeKey 或者直接运行以下命令。

```
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.0.1 sh - 
```

备注

通过以上的命令，可以下载 KubeKey 的最新版本 (v1.0.1)。您可以更改命令中的版本号来下载特定的版本。

## 2.3 支持的环境

### 2.3.1 Linux 发行版

- **Ubuntu** *16.04, 18.04*
- **Debian** *Buster, Stretch*
- **CentOS/RHEL** *7*
- **SUSE Linux Enterprise Server** *15*

### 2.3.2 Kubernetes 版本

- **v1.15**:   *v1.15.12*
- **v1.16**:   *v1.16.13*
- **v1.17**:   *v1.17.9* (默认)
- **v1.18**:   *v1.18.6*

> 查看更多支持的版本[点击这里](https://gitee.com/kubesphere/kubekey/blob/master/docs/kubernetes-versions.md)

> 注意: KubeSphere目前暂不支持运行在k8s 1.19.x之上。

## 2.4 要求和建议

- 最低资源要求（仅对于最小安装 KubeSphere）： 
  - 2 核虚拟 CPU
  - 4 GB 内存
  - 20 GB 储存空间

> /var/lib/docker 主要用于存储容器数据，在使用和操作过程中会逐渐增大。对于生产环境，建议 /var/lib/docker 单独挂盘。

- 操作系统要求： 
  - `SSH` 可以访问所有节点。
  - 所有节点的时间同步。
  - `sudo`/`curl`/`openssl` 应在所有节点使用。
  - `docker` 可以自己安装，也可以通过 KubeKey 安装。
  - `Red Hat` 在其 `Linux` 发行版本中包括了`SELinux`，建议[关闭SELinux](https://gitee.com/kubesphere/kubekey/blob/master/docs/turn-off-SELinux_zh-CN.md)或者将[SELinux的模式切换](https://gitee.com/kubesphere/kubekey/blob/master/docs/turn-off-SELinux_zh-CN.md)为Permissive[宽容]工作模式

> - 建议您的操作系统环境足够干净 (不安装任何其他软件)，否则可能会发生冲突。
> - 如果在从 dockerhub.io 下载镜像时遇到问题，建议准备一个容器镜像仓库 (加速器)。[为 Docker 守护程序配置镜像加速](https://docs.docker.com/registry/recipes/mirror/#configure-the-docker-daemon)。
> - 默认情况下，KubeKey 将安装 [OpenEBS](https://openebs.io/) 来为开发和测试环境配置 LocalPV，这对新用户来说非常方便。对于生产，请使用 NFS/Ceph/GlusterFS 或商业化存储作为持久化存储，并在所有节点中安装[相关的客户端](https://gitee.com/kubesphere/kubekey/blob/master/docs/storage-client.md) 。
> - 如果遇到拷贝时报权限问题Permission denied,建议优先考虑查看[SELinux的原因](https://gitee.com/kubesphere/kubekey/blob/master/docs/turn-off-SELinux_zh-CN.md)。

- 依赖要求:

KubeKey 可以同时安装 Kubernetes 和 KubeSphere。根据 KubeSphere 所安装版本的不同，您所需要安装的依赖可能也不同。请参考以下表格查看您是否需要提前在节点上安装有关的依赖。

|             | Kubernetes 版本 ≥ 1.18 | Kubernetes 版本 < 1.18 |
| ----------- | ---------------------- | ---------------------- |
| `socat`     | 必须安装               | 可选，但推荐安装       |
| `conntrack` | 必须安装               | 可选，但推荐安装       |
| `ebtables`  | 可选，但推荐安装       | 可选，但推荐安装       |
| `ipset`     | 可选，但推荐安装       | 可选，但推荐安装       |

- 网络和 DNS 要求： 
  - 确保 `/etc/resolv.conf` 中的 DNS 地址可用。否则，可能会导致群集中出现某些 DNS 问题。
  - 如果您的网络配置使用防火墙或安全组，则必须确保基础结构组件可以通过特定端口相互通信。建议您关闭防火墙或遵循链接配置：[网络访问](https://gitee.com/kubesphere/kubekey/blob/master/docs/network-access.md)。

# 三、用法

## 3.1 获取安装程序可执行文件

- 下载KubeKey可执行文件 [Releases page](https://github.com/kubesphere/kubekey/releases)

  下载解压后可直接使用。

- 从源代码生成二进制文件

  ```
  git clone https://github.com/kubesphere/kubekey.git
  cd kubekey
  ./build.sh
  ```

> 注意：
>
> - 在构建之前，需要先安装 Docker。
> - 如果无法访问 `https://proxy.golang.org/`，比如在大陆，请执行 `build.sh -p`。

## 3.2 创建集群

### 3.2.1 快速开始

快速入门使用 `all-in-one` 安装，这是熟悉 KubeSphere 的良好开始。

命令

```
./kk create cluster [--with-kubernetes version] [--with-kubesphere version]
```

例子

- 使用默认版本创建一个纯 Kubernetes 集群

  ```
  ./kk create cluster
  ```

- 创建指定一个（[支持的版本](https://gitee.com/kubesphere/kubekey#KubernetesVersions)）的 Kubernetes 集群

  ```
  ./kk create cluster --with-kubernetes v1.17.9
  ```

- 创建一个部署了 KubeSphere 的 Kubernetes 集群 （例如 `--with-kubesphere v3.0.0`）

  ```
  ./kk create cluster --with-kubesphere [version]
  ```

### 3.2.2 高级用法

您可以使用高级安装来控制自定义参数或创建多节点群集。具体来说，通过指定配置文件来创建集群。

1. 首先，创建一个示例配置文件

   ```
   ./kk create config [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]
   ```

   **例子：**

   - 使用默认配置创建一个示例配置文件。您也可以指定文件名称或文件所在的文件夹。

     ```
     ./kk create config [-f ~/myfolder/config-sample.yaml]
     ```

   - 同时安装 KubeSphere

     ```
     ./kk create config --with-kubesphere
     ```

2. 根据您的环境修改配置文件 config-sample.yaml

> 当指定安装KubeSphere时，要求集群中有可用的持久化存储。默认使用localVolume，如果需要使用其他持久化存储，请参阅 [addons](https://gitee.com/kubesphere/kubekey/blob/master/docs/addons.md) 配置。

1. 使用配置文件创建集群。

   ```
   ./kk create cluster -f ~/myfolder/config-sample.yaml
   ```

### 3.2.3 启用多集群管理

默认情况下，Kubekey 将仅安装一个 Solo 模式的单集群，即未开启 Kubernetes 多集群联邦。如果您希望将 KubeSphere 作为一个支持多集群集中管理的中央面板，您需要在 [config-example.yaml](https://gitee.com/kubesphere/kubekey/blob/master/docs/config-example.md) 中设置 `ClusterRole`。关于多集群的使用文档，请参考 [如何启用多集群](https://github.com/kubesphere/community/blob/master/sig-multicluster/how-to-setup-multicluster-on-kubesphere/README_zh.md)。

### 3.2.4 开启可插拔功能组件

KubeSphere 从 2.1.0 版本开始对 Installer  的各功能组件进行了解耦，快速安装将默认仅开启最小化安装（Minimal Installation），Installer  支持在安装前或安装后自定义可插拔的功能组件的安装。使最小化安装更快速轻量且资源占用更少，也方便不同用户按需选择安装不同的功能组件。

KubeSphere 有多个可插拔功能组件，功能组件的介绍可参考 [配置示例](https://gitee.com/kubesphere/kubekey/blob/master/docs/config-example.md)。您可以根据需求，选择开启安装 KubeSphere 的可插拔功能组件。我们非常建议您开启这些功能组件来体验 KubeSphere 完整的功能以及端到端的解决方案。请在安装前确保您的机器有足够的 CPU 与内存资源。开启可插拔功能组件可参考 [开启可选功能组件](https://github.com/kubesphere/ks-installer/blob/master/README_zh.md#安装功能组件)。

## 3.3 添加节点

将新节点的信息添加到群集配置文件，然后应用更改。

```
./kk add nodes -f config-sample.yaml
```

## 3.4 删除节点

通过以下命令删除节点，nodename指需要删除的节点名。

```
./kk delete node <nodeName> -f config-sample.yaml
```

## 3.5 删除集群

您可以通过以下命令删除集群：

- 如果您以快速入门（all-in-one）开始：

```
./kk delete cluster
```

- 如果从高级安装开始（使用配置文件创建的集群）：

```
./kk delete cluster [-f config-sample.yaml]
```

## 3.6 集群升级

### 3.6.1 单节点集群

升级集群到指定版本。

```
./kk upgrade [--with-kubernetes version] [--with-kubesphere version] 
```

- `--with-kubernetes` 指定kubernetes目标版本。
- `--with-kubesphere` 指定kubesphere目标版本。

### 3.6.2 多节点集群

通过指定配置文件对集群进行升级。

```
./kk upgrade [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]
```

- `--with-kubernetes` 指定kubernetes目标版本。
- `--with-kubesphere` 指定kubesphere目标版本。
- `-f` 指定集群安装时创建的配置文件。

> 注意: 升级多节点集群需要指定配置文件. 如果集群非kubekey创建，或者创建集群时生成的配置文件丢失，需要重新生成配置文件，或使用以下方法生成。

Getting cluster info and generating kubekey's configuration file (optional).

```
./kk create config [--from-cluster] [(-f | --file) path] [--kubeconfig path]
```

- `--from-cluster` 根据已存在集群信息生成配置文件.
- `-f` 指定生成配置文件路径.
- `--kubeconfig` 指定集群kubeconfig文件.
- 由于无法全面获取集群配置，生成配置文件后，请根据集群实际信息补全配置文件。

## 3.7 启用 kubectl 自动补全

KubeKey 不会启用 kubectl 自动补全功能。请参阅下面的指南并将其打开：

**先决条件**：确保已安装 `bash-autocompletion` 并可以正常工作。

```
# 安装 bash-completion
apt-get install bash-completion

# 将 completion 脚本添加到你的 ~/.bashrc 文件
echo 'source <(kubectl completion bash)' >>~/.bashrc

# 将 completion 脚本添加到 /etc/bash_completion.d 目录
kubectl completion bash >/etc/bash_completion.d/kubectl
```