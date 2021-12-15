[kubernetes 高可用部署工具：sealos](https://fuckcloudnative.io/posts/sealos/)



sealos 项目地址：https://github.com/fanux/sealos

本项目名叫 sealos，旨在做一个简单干净轻量级稳定的 kubernetes 安装工具，能很好的支持高可用安装。 其实把一个东西做的功能强大并不难，但是做到极简且灵活可扩展就比较难。 所以在实现时就必须要遵循这些原则。

sealos旨在做一个简单干净轻量级稳定的kubernetes安装工具，能很好的支持高可用安装。 

# 一、 设计原则

## sealos 特性与优势：

- 支持离线安装，工具与资源包（二进制程序 配置文件 镜像 yaml文件等）分离,这样不同版本替换不同离线包即可
- 证书延期
- 使用简单
- 支持自定义配置
- 内核负载，极其稳定，因为简单所以排查问题也极其简单

### 为什么不用 ansible？

1.0 版本确实是用 `ansible` 实现，但是用户还是需要先装 ansile，装 ansible 又需要装 `python` 和一些依赖等，为了不让用户那么麻烦把 ansible 放到了容器里供用户使用。如果不想配置免密钥使用用户名密码时又需要 `ssh-pass` 等，总之不能让我满意，不是我想的极简。

所以我想就来一个二进制文件工具，没有任何依赖，文件分发与远程命令都通过调用 sdk 实现所以不依赖其它任何东西，总算让我这个有洁癖的人满意了。

### 为什么不用 keepalived haproxy？

`haproxy` 用 static pod 跑没有太大问题，还算好管理，`keepalived` 现在大部分开源 ansible 脚本都用 yum 或者 apt 等装，这样非常的不可控，有如下劣势：

- 源不一致可能导致版本不一致，版本不一直连配置文件都不一样，我曾经检测脚本不生效一直找不到原因，后来才知道是版本原因。
- 系统原因安装不上，依赖库问题某些环境就直接装不上了。
- 看了网上很多安装脚本，很多检测脚本与权重调节方式都不对，直接去检测 haproxy 进程在不在，其实是应该去检测 apiserver 是不是 healthz 的，如果 apiserver 挂了，即使 haproxy 进程存在，集群也会不正常了，就是伪高可用了。
- 管理不方便，通过 prometheus 对集群进行监控，是能直接监控到 static pod 的但是用 systemd 跑又需要单独设置监控，且重启啥的还需要单独拉起。不如 kubelet 统一管理来的干净简洁。
- 我们还出现过 keepalived 把 CPU 占满的情况。

所以为了解决这个问题，我把 keepalived 跑在了容器中(社区提供的镜像基本是不可用的) 改造中间也是发生过很多问题，最终好在解决了。

总而言之，累觉不爱，所以在想能不能甩开 haproxy 和 keepalived 做出更简单更可靠的方案出来，还真找到了。。。

### 本地负载为什么不使用 envoy 或者 nginx？

我们通过本地负载解决高可用问题。

**本地负载**：在每个 node 节点上都启动一个负载均衡，上游就是三个 master。

如果使用 `envoy` 之类的负载均衡器，则需要在每个节点上都跑一个进程，消耗的资源更多，这是我不希望的。ipvs 实际也多跑了一个进程 `lvscare`，但是 `lvscare` 只是负责管理 ipvs 规则，和 `kube-proxy` 类似，真正的流量还是从很稳定的内核走的，不需要再把包丢到用户态中去处理。

在架构实现上有个问题会让使用 `envoy` 等变得非常尴尬，就是 `join` 时如果负载均衡没有建立那是会卡住的，`kubelet` 就不会起来，所以为此你需要先启动 envoy，意味着你又不能用 static pod 去管理它，同上面 keepalived 宿主机部署一样的问题，用 static pod 就会相互依赖，逻辑死锁，鸡说要先有蛋，蛋说要先有鸡，最后谁都没有。

使用 `ipvs` 就不一样，我可以在 join 之前先把 ipvs 规则建立好，再去 join 就可以了，然后对规则进行守护即可。一旦 apiserver 不可访问了，会自动清理掉所有 node 上对应的 ipvs 规则， 等到 master 恢复正常时添加回来。

### 为什么要定制 kubeadm?

首先是由于 `kubeadm` 把证书过期时间写死了，所以需要定制把它改成 `99` 年，虽然大部分人可以自己去签个新证书，但是我们还是不想再依赖个别的工具，就直接改源码了。

其次就是做本地负载时修改 `kubeadm` 代码是最方便的，因为在 join 时我们需要做两个事，第一是 join 之前先创建好 ipvs 规则，第二是创建 static pod。如果这块不去定制 kubeadm 就把报静态 pod 目录已存在的错误，忽略这个错误很不优雅。 而且 kubeadm 中已经提供了一些很好用的 `sdk` 供我们去实现这个功能。

且这样做之后最核心的功能都集成到 kubeadm 中了，`sealos` 就单单变成分发和执行上层命令的轻量级工具了，增加节点时我们也就可以直接用 kubeadm 了。



# 二、快速开始安装部署

> 只需要准备好服务器，在任意一台服务器上执行下面命令即可

```
# 下载并安装sealos, sealos是个golang的二进制工具，直接下载拷贝到bin目录即可, release页面也可下载
$ wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/latest/sealos && \
    chmod +x sealos && mv sealos /usr/bin 

# 下载离线资源包
$ wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/562b5c0ae4e48d17c5ab6d49422842c5-v1.20.0/kube1.20.0.tar.gz

# 安装一个三master的kubernetes集群
$ sealos init --passwd '123456' \
	--master 192.168.0.2  --master 192.168.0.3  --master 192.168.0.4  \
	--node 192.168.0.5 \
	--pkg-url /root/kube1.20.0.tar.gz \
	--version v1.20.0
```

> 参数含义

| 参数名  | 含义                                                         | 示例                    |
| ------- | ------------------------------------------------------------ | ----------------------- |
| passwd  | 服务器密码                                                   | 123456                  |
| master  | k8s master节点IP地址                                         | 192.168.0.2             |
| node    | k8s node节点IP地址                                           | 192.168.0.3             |
| pkg-url | 离线资源包地址，支持下载到本地，或者一个远程地址             | /root/kube1.20.0.tar.gz |
| version | [资源包](https://www.sealyun.com/goodsDetail?type=cloud_kernel&name=kubernetes)对应的版本 | v1.20.0                 |

> 增加master

```
🐳 → sealos join --master 192.168.0.6 --master 192.168.0.7
🐳 → sealos join --master 192.168.0.6-192.168.0.9  # 或者多个连续IP
```

> 增加node

```
🐳 → sealos join --node 192.168.0.6 --node 192.168.0.7
🐳 → sealos join --node 192.168.0.6-192.168.0.9  # 或者多个连续IP
```

> 删除指定master节点

```
🐳 → sealos clean --master 192.168.0.6 --master 192.168.0.7
🐳 → sealos clean --master 192.168.0.6-192.168.0.9  # 或者多个连续IP
```

> 删除指定node节点

```
🐳 → sealos clean --node 192.168.0.6 --node 192.168.0.7
🐳 → sealos clean --node 192.168.0.6-192.168.0.9  # 或者多个连续IP
```

> 清理集群

```
🐳 → sealos clean --all
```