## Containerd 概述

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrnlnRIxN921dS6lwibp9yOAlGiaPYCf37IQ1h4aQicD1wOyIb9cNicskJs8EwSRuwHJxpuddJF5q8HrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

很早之前的 [Docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247509907&idx=1&sn=c78499a0684f7d7ef82bfdcedd4b4150&chksm=e918c28fde6f4b993d5a3ea0600b3a06b6cca768037425dae7142833c8f4cadfbae5dd9d0fc3&scene=21#wechat_redirect) Engine 中就有了containerd，只不过现在是将 containerd 从 Docker Engine  里分离出来，作为一个独立的开源项目，目标是提供一个更加开放、稳定的容器运行基础设施。分离出来的 containerd  将具有更多的功能，涵盖整个容器运行时管理的所有需求，提供更强大的支持。

简单的来说，containerd 是一个工业级标准的容器运行时，它强调简单性、健壮性和可移植性。**containerd可以在宿主机中管理完整的容器生命周期，包括容器镜像的传输和存储、容器的执行和管理、存储和网络等**。

地址：https://github.com/containerd/containerd/

## containerd 架构

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrnlnRIxN921dS6lwibp9yOAKrClAKsh7ibWicXT2ibXHW9KGMffX3ibjm8MgvXOuxGuRRLlwicUiaaqNFVA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中，grpc 模块向上层提供服务接口，metrics 则提供监控数据(cgroup 相关数据)，两者均向上层提供服务。containerd 包含一个守护进程，该进程通过本地 UNIX 套接字暴露 grpc 接口。

storage 部分负责镜像的存储、管理、拉取等 metadata 管理容器及镜像的元数据，通过bootio存储在磁盘上 task --  管理容器的逻辑结构，与 low-level 交互 event -- 对容器操作的事件，上层通过订阅可以知道发生了什么事情 Runtimes -- low-level runtime（对接 runc）

## Containerd 能做什么？？

- 管理容器的生命周期(从创建容器到销毁容器)
- 拉取/推送容器镜像
- 存储管理(管理镜像及容器数据的存储)
- 调用 runC 运行容器(与 runC 等容器运行时交互)
- 管理容器网络接口及网络

从 k8s 的角度看，选择 containerd作为运行时的组件，它调用链更短，组件更少，更稳定，占用节点资源更少。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrnlnRIxN921dS6lwibp9yOAF6gJ6dU5XpLaFzc1j5fU1yHgEicCmIsslibCxicKzuyiczEQDcwtCqNKIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上图来源 containerd官方网站，containerd可用作 Linux 和 Windows 的守护程序。它管理其主机系统的完整容器生命周期，从图像传输和存储到容器执行和监督，再到低级存储到网络附件等等。

## 安装

下载地址：https://containerd.io/downloads/

```
[root@centos7 ~]# wget https://github.com/containerd/containerd/releases/download/v1.5.2/containerd-1.5.2-linux-amd64.tar.gz
[root@centos7 ~]# tar zxf containerd-1.5.2-linux-amd64.tar.gz -C /usr/local/

#通过上面的操作，将containerd 安装至/usr/local/bin目录下
[root@centos7 ~]# cd /usr/local/bin/
[root@centos7 bin]# ll
total 98068
-rwxr-xr-x 1 root root   214432 Mar 29 05:20 bpytop
-rwxr-xr-x 1 1001  116 49049696 May 19 12:56 containerd
-rwxr-xr-x 1 1001  116  6434816 May 19 12:56 containerd-shim
-rwxr-xr-x 1 1001  116  8671232 May 19 12:57 containerd-shim-runc-v1
-rwxr-xr-x 1 1001  116  8683520 May 19 12:57 containerd-shim-runc-v2
-rwxr-xr-x 1 1001  116 27230976 May 19 12:56 ctr
lrwxrwxrwx 1 root root        6 Mar 28 00:13 nc -> netcat
-rwxr-xr-x 1 root root   126800 Mar 28 00:13 netcat
```

生成默认配置文件

```
[root@centos7 bin]# containerd config default > /etc/containerd/config.toml
[root@centos7 bin]# ll /etc/containerd/config.toml 
-rw-r--r-- 1 root root 6069 Jun  4 14:47 /etc/containerd/config.toml
```

配置 containerd 作为服务运行

```
[root@centos7 ~]# touch /lib/systemd/system/containerd.service
[root@centos7 bin]# vim /lib/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Delegate=yes
KillMode=process
LimitNOFILE=1048576
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

启动服务

```
[root@centos7 ~]# systemctl daemon-reload
[root@centos7 ~]# systemctl start containerd.service
[root@centos7 ~]# systemctl status containerd.service
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrnlnRIxN921dS6lwibp9yOAGjN42gENEsPKibhlae9Oib15TWOldJK8kd3kXLOo5kORHQRaHhauaaXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## containerd 使用

其实，[史上最轻量 Kubernetes 发行版 K3s](https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247510850&idx=2&sn=da57a46e7c0c472c71bce4823413659b&chksm=e918ce5ede6f474870b7e7d00f7c6152b760944100db7666a2fae673308184349723e1a411cb&token=581298630&lang=zh_CN&scene=21#wechat_redirect) 默认就包括了 containerd、Flannel、CoreDNS 组件。

- ctr：是containerd本身的CLI
- crictl ：是Kubernetes社区定义的专门CLI工具

```
[root@centos7 ~]# ctr version
Client:
  Version:  v1.5.2
  Revision: 36cc874494a56a253cd181a1a685b44b58a2e34a
  Go version: go1.16.4

Server:
  Version:  v1.5.2
  Revision: 36cc874494a56a253cd181a1a685b44b58a2e34a
  UUID: ebe42dac-40ae-4af1-99b0-52e61728c918
```

帮助信息

```
[root@centos7 ~]# ctr --help
NAME:
   ctr - 
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


USAGE:
   ctr [global options] command [command options] [arguments...]

VERSION:
   v1.5.2

DESCRIPTION:
   
ctr is an unsupported debug and administrative client for interacting
with the containerd daemon. Because it is unsupported, the commands,
options, and operations are not guaranteed to be backward compatible or
stable from release to release of the containerd project.

COMMANDS:
   plugins, plugin            provides information about containerd plugins
   version                    print the client and server versions
   containers, c, container   manage containers
   content                    manage content
   events, event              display containerd events
   images, image, i           manage images
   leases                     manage leases
   namespaces, namespace, ns  manage namespaces
   pprof                      provide golang pprof outputs for containerd
   run                        run a container
   snapshots, snapshot        manage snapshots
   tasks, t, task             manage tasks
   install                    install a new package
   oci                        OCI tools
   shim                       interact with a shim directly
   help, h                    Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                      enable debug output in logs
   --address value, -a value    address for containerd's GRPC server (default: "/run/containerd/containerd.sock") [$CONTAINERD_ADDRESS]
   --timeout value              total timeout for ctr commands (default: 0s)
   --connect-timeout value      timeout for connecting to containerd (default: 0s)
   --namespace value, -n value  namespace to use with commands (default: "default") [$CONTAINERD_NAMESPACE]
   --help, -h                   show help
   --version, -v                print the version
```

查看与删除

```
[root@centos7 ~]# ctr container list
CONTAINER    IMAGE                             RUNTIME                  
nginx        docker.io/library/nginx:alpine    io.containerd.runc.v2    
[root@centos7 ~]# ctr container del nginx
[root@centos7 ~]# ctr container list
CONTAINER    IMAGE    RUNTIME   
```

pull镜像文件

```
[root@centos7 ~]# ctr images pull docker.io/library/nginx:alpine
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrnlnRIxN921dS6lwibp9yOATbSEK3hPw61jkKXN2qC9mJISdxbFWDZoPflXCwmF8gSkwqtbo31ROQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

查看镜像文件列表

```
[root@centos7 ~]# ctr images list
REF                            TYPE                                                      DIGEST                                                                  SIZE    PLATFORMS                                                                                LABELS 
docker.io/library/nginx:alpine application/vnd.docker.distribution.manifest.list.v2+json sha256:0f8595aa040ec107821e0409a1dd3f7a5e989501d5c8d5b5ca1f955f33ac81a0 9.4 MiB linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64/v8,linux/ppc64le,linux/s390x - 
```

运行容器

```
[root@centos7 ~]# ctr run -d docker.io/library/nginx:alpine nginx
[root@centos7 ~]# ctr container list
CONTAINER    IMAGE                             RUNTIME                  
nginx        docker.io/library/nginx:alpine    io.containerd.runc.v2 
```

一圈使用下来，基本上与[docker的命令](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247504153&idx=1&sn=208009779ddb18d84fd198770d67656c&chksm=e918b405de6f3d1321f11b4a1be539f01a7bd79f7a37560caec825bb3055844910d2576b371e&scene=21#wechat_redirect)相差无几，使用上没有什么大的学习成本，所以，无论是 Kubernetes 是否支持 [docker](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247502654&idx=1&sn=9458abf0f3adc055759f4498a97b50f9&chksm=e918ae22de6f273421893f8e0203130b1ca4a90805cb315cca236760b32abe37b19e4cfe9c9f&scene=21#wechat_redirect)，对于我们使用者来讲，问题不大。