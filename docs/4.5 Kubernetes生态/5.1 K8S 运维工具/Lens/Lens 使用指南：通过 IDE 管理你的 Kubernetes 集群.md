- [Lens 使用指南：通过 IDE 管理你的 Kubernetes 集群](https://blog.csdn.net/jiangyou0k/article/details/105784868)



`Lens` 是一个开源的管理 Kubernetes 集群的 IDE，支持 MacOS， Windows 和 Linux。通过 Lens，我们可以很方便地管理多个 Kubernetes 集群。

本文演示环境为 Windows X64，Lens 版本为 3.3.1，连接的 Kubernetes 集群托管在阿里云上。

# 一、下载安装

到 releases 下载对应的安装包。我用的是祖传 Windows 系统，所以这里下载 `Lens-Setup-3.3.1.exe`：

![img](https:////upload-images.jianshu.io/upload_images/3821938-6477c3d3cd2c7a9f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



安装后可以看到：



![img](https:////upload-images.jianshu.io/upload_images/3821938-7fb7bba61c8cb64f.png?imageMogr2/auto-orient/strip|imageView2/2/w/535)

lens.png

点击 `+` 并选择要连接的集群：

![img](https:////upload-images.jianshu.io/upload_images/3821938-d0e24ed3cb34c434.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162)



我本地配置过 `kubeconfig`，所以添加集群的时候能够看到配置。如果之前没配过，可以选择 `Custom` 手动添加。选好后点击 `Add Cluster`，就可以看到集群了：

![img](https:////upload-images.jianshu.io/upload_images/3821938-24808d0b9e35b65b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)



# 二、安装 Metrics

可以看到，目前集群上没有 Metrics 数据。在集群图标上右键然后点击 `Settings`：

![img](https:////upload-images.jianshu.io/upload_images/3821938-caa131b548a7bbc5.png?imageMogr2/auto-orient/strip|imageView2/2/w/624)



点击 `Install` 安装：

![img](https:////upload-images.jianshu.io/upload_images/3821938-ed63006f26f24c7b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1113)



之后在 Cluster 界面就可以看到 Metrics 数据了：



![img](https:////upload-images.jianshu.io/upload_images/3821938-b105898d69602b4b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

安装 Metrics 时会新建命名空间 `lens-metrics`，并通过 `Prometheus` 采集数据。如果之后不需要的话，可以在设置界面点击 `Uninstall` 卸载。

# 三、使用指南

本章将演示一些 Lens 的基本操作，包括：

> - 新建 namespace
> - 添加 Deployment
> - 调整 Deployment 的副本数
> - 进入 Pod 内部

## Namespace

平时用命令行新建命名空间 `test`，需要执行：



作者：Xpitz
链接：https://www.jianshu.com/p/7382b2e2163c