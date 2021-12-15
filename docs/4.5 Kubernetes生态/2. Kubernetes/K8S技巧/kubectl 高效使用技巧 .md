- [kubectl 高效使用技巧](https://mp.weixin.qq.com/s/nqcZRsm4c9JITgXh8jYvTg)

kubectl 是 Kubernetes 集群的控制工具，它可以让你执行所有可能的 Kubernetes 操作。

从技术角度上看，kubectl 是 Kubernetes API 的客户端，Kubernetes API 是一个 HTTP REST API，这个 API 是真正的  Kubernetes 用户界面，Kubernetes完全受这个 API 控制，这意味着每个 Kubernetes 操作都作为 API  端点暴露，并且可以由对此端点的 HTTP 请求执行，因此，kubectl 的主要工作是执行对 Kubernetes API 的 HTTP 请求：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2851g3nkHIh2mJ1oTdDlWmaMAJyDK9BXYpvdkabAylcQEkolicRVCb1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Kubernetes 是一个完全以资源为中心的系统，Kubernetes 维护资源的内部状态并且所有的 Kubernetes 操作都是针对这些资源的 CRUD（增加、查询、更新、删除）操作，你完全可以通过操控这些资源来控制Kubernetes。

比如我们想要创建一个 `ReplicaSet` 资源，在一个名为 `replicaset.yaml` 的文件中定义 ReplicaSet 资源对象，然后运行以下命令：

```
kubectl create -f replicaset.yaml
```

Kubernetes 有一个创建 ReplicaSet 的操作，并且它和其他所有 Kubernetes 操作一样，都会作为 API 端点暴露出去，对于我们这里的操作而言，该 API 端点如下：

```
POST /apis/apps/v1/namespaces/{namespace}/replicasets
```

当我们执行上述命令时，kubectl 将向上述 API 端点发出一个 HTTP POST 请求。ReplicaSet 的资源清单内容在请求的 body 中传递。这就是  kubectl 与 Kubernetes 集群交互的所有命令的最基本的工作方式，kubectl 只需向对应的Kubernetes API  端点发出 HTTP 请求即可。

所以我们完全完全也可以使用 curl、postman 之类的工具来控制 Kubernetes，Kubectl 只是让我们可以更轻松地访问 Kubernetes API。

执行了上面的资源创建命令之后，API server 将会在 etcd 保存 ReplicaSet 资源定义。然后会触发 controller manager 中的  ReplicaSet controller，后者会一直watch ReplicaSet 资源的创建、更新和删除。ReplicaSet  controller 为每个 ReplicaSet 副本创建了一个 Pod 定义（根据在 ReplicaSet  定义中的Pod模板创建）并将它们保存到存储后端。Pod 创建后会触发了 scheduler，它一直 watch 尚未被分配给 worker  节点的 Pod。

Scheduler 为每个 Pod 选择一个合适的 worker 节点，并在存储后端中添加该信息到 Pod 定义中。这触发了在 Pod 所调度到的 worker  节点上的kubelet，它会监视调度到其 worker 节点上的 Pod。Kubelet 从存储后端读取 Pod 定义并指示容器运行时来运行在  worker 节点上的容器。这样我们的 ReplicaSet 应用程序就运行起来了。

熟悉了这些流程概念后会在很大程度上帮助我们更好地理解 kubectl 并利用它。接下来，我们来看一下具体的技巧，来帮助你提升 kubectl 的生产力。

## 命令补全

`命令补全`是提高 kubectl 生产率的最有用但经常被忽略的技巧之一。命令补全功能使你可以使用 Tab 键自动完成 kubectl 命令的各个部分。这适用于子命令、选项和参数，包括诸如资源名称之类难以键入的内容。命令补全可用于 Bash 和 Zsh Shell。

官方文档 https://kubernetes.io/docs/tasks/tools/#enabling-shell-autocompletion 中其实就有关于命令补全的说明。

命令补全是通过补全脚本而起作用的 Shell 功能，补全脚本本质上是一个 shell 脚本，它为特定命令定义了补全行为。通过输入补全脚本可以补全相应的命令。Kubectl 可以使用以下命令为 Bash 和 Zsh 自动生成并 `print out` 补全脚本：

```
kubectl completion bash
# or
kubectl completion zsh
```

理论上，在合适的 shell（Bash或Zsh）中提供此命令的输出将会启用 kubectl 的命令补全功能。在 Bash 和 Zsh 之间存在一些细微的差别（包括在 Linux 和 macOS 之间也存在差别）。

### Bash

#### Linux

Bash 的补全脚本主要依赖 bash-completion 项目，所以你需要先安装它。我们可以使用不同的软件包管理器安装 `bash-completion`，如：

```
# ubuntu
apt-get install bash-completion
# or centos
yum install bash-completion
```

你可以使用以下命令测试 `bash-completion` 是否正确安装：

```
type _init_completion
```

如果输出的是 shell 的代码，那么 `bash-completion` 就已经正确安装了。如果命令输出的是 not found 错误，你必须添加以下命令行到你的 `~/.bashrc` 文件：

```
source /usr/share/bash-completion/bash_completion
```

你是否需要添加这一行到你的 `~/.bashrc` 文件中，取决于你用于安装 `bash-completion` 的软件包管理器，对于 APT 来说，这是必要的，对于 yum 则不是。

`bash-completion` 安装完成之后，你需要进行一些设置，以便在所有的 shell 会话中获得 kubectl 补全脚本。一种方法是将以下命令行添加到 `~/.bashrc` 文件中：

```
source <(kubectl completion bash)
```

另一种是将 kubectl 补充脚本添加到 `/etc/bash_completion.d` 目录中（如果不存在，则需要先创建它）：

```
kubectl completion bash  > /etc/bash_completion.d/kubectl
```

`/etc/bash_completion.d` 目录中的所有补全脚本均由 `bash-completion` 自动提供。

以上两种方法都是一样的效果。重新加载 shell 后，kubectl 命令补全就能正常工作了，这个时候我们可以使用 tab 来补全信息了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2mwOz85BbbEHD1Wz0ldWqGQGkFF9o1H2Xw4VwYh7Z4hyfEXPsjKBdzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### Mac

使用 macOS 时，会有些复杂，因为默认的 Bash 版本是3.2，而 kubectl 补全脚本至少需要 Bash 4.1，苹果依旧在 macOS 上默认使用过时的 Bash 版本是因为更新版本的 Bash 使用的是 GPLv3 license，而苹果不支持这一 license。

所以要在 macOS 上使用 kubectl 命令补全功能，你必须安装较新版本的 Bash。我们可以用 Homebrew 安装/升级：

```
brew install bash
```

重新加载 shell，并验证所需的版本已经生效：

```
echo $BASH_VERSION $SHELL
```

在继续剩下的步骤之前，确保你现在已经在使用 Bash 4.1 或更高的版本（可以使用 `bash --version` 查看版本）。

同上一部分一样，Bash 的补全脚本主要依赖 `bash-completion` 项目，所以你需要先安装它。我们可以使用 Homebrew 安装  bash-completion：

```
brew install bash-completion@2
```

`@2` 代表 bash-completion v2 版本，Kubectl 补全脚本要求 bash-completion v2，而  bash-completion v2 要求至少是Bash 4.1，这就是你不能在低于 4.1 的 Bash 版本上使用 kubectl  补全脚本的原因。

`brew intall` 命令的输出包括一个 `Caveats` 部分，其中的说明将以下行添加 `~/.bash_profile` 文件：

```
export BASH_COMPLETION_COMPAT_DIR=/usr/local/etc/bash_completion.d
[[ -r "/usr/local/etc/profile.d/bash_completion.sh"  ]]  &&  .  "/usr/local/etc/profile.d/bash_completion.sh"
```

必须执行此操作才能完成 bash-completion 的安装，当然最好将上面的内容添加到 `~/.bashrc` 文件中。重新加载 shell 之后，你可以使用以下命令测试 bash-completion 是否正确安装：

```
type _init_completion
```

如果输出为 shell 功能的代码，意味着一切都设置完成。现在，你需要进行一些设置，以便在所有的 shell 会话中获得 kubectl 补全脚本。一种方法是将以下命令行添加到 `~/.bashrc` 文件中：

```
source <(kubectl completion bash)
```

另一种方法是将 kubectl 补全脚本添加到 `/usr/local/etc/bash_completion.d` 目录中：

```
kubectl completion bash  >/usr/local/etc/bash_completion.d/kubectl
```

仅当你使用 Homebrew 安装了 bash-completion 时，此方法才有效。在这种情况下，bash-completion  会在此目录中提供所有补全脚本。如果你还用 Homebrew 安装了kubectl，则甚至不必执行上述步骤，因为补全脚本应该已经由 kubectl Homebrew 放置在 `/usr/local/etc/bash_completion.d` 目录中。在这种情况下，kubectl 补全应该在安装 bash-completion 后就可以生效了。

重新加载 shell 后，kubectl 自动补全也就生效了。

### Zsh

Zsh 的补全脚本没有任何依赖项，所以配置要简单很多，我们可以通过添加以下命令到你的 `~/.zshrc` 文件中来实现这一效果：

```
source <(kubectl completion zsh)
```

如果在重新加载你的 shell 之后，你获得了 `command not found: compdef` 错误，你需要启动内置的 `compdef`，你可以在将以下行添加到开始的 `~/.zshrc` 文件中：

```
autoload -Uz compinit
compinit
```

此外我们还可以为 kubectl 定义一个别名，比如定义成 k：

```
echo 'alias k=kubectl' >> ~/.zshrc
```

如果定义了别名也可以通过扩展 shell 补全来兼容该别名：

```
echo 'complete -F __start_kubectl k' >> ~/.zshrc
```

另外还推荐配置 zsh 下面的 zsh-autosuggestions、zsh-syntax-highlighting、kubectl 这几个插件，可以自动提示之前我们使用过的一些历史命令，在 `~/.zshrc` 中添加插件配置：

```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting kubectl)
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2iawQgA9icCX2pJy8unicTmViaZUgAW435PDztLAFz61HKm2hLvicGLv7zHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 查看资源规范

当你创建 YAML 资源定义时，你需要知道字段以及这些资源的意义，kubectl 提供了 `kubectl explain` 命令，它可以在终端中正确地输入所有资源规范。

kubectl explain的用法如下：

```
kubectl explain resource[.field]...
```

该命令输出所请求资源或字段的规范，默认情况下，`kubectl explain` 仅显示单个级别的字段，你可以使用 `--recursive` 标志来显示所有级别的字段：

```
kubectl explain deployment.spec --recursive
```

如果你不确定哪个资源名称可以用于 `kubectl explain`，你可以使用以下命令查看：

```
kubectl api-resources
```

该命令以复数形式显示资源名称（如 deployments），它同时显示资源名称的缩写（如 deploy），这些名称对于 kubectl 都是等效的，我们可以使用它们中的任何一个。例如，以下命令效果都是一样的：

```
kubectl explain deployments.spec
# or
kubectl explain deployment.spec
# or
kubectl explain deploy.spec
```

## 自定义列输出格式

`kubectl get` 命令默认的输出方式如下（该命令用于读取资源）：

```
➜  ~ kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-54f4485448-kwr45   1/1     Running   1          67d
nginx-674ff86d-t6gbd                      1/1     Running   0          67d
```

默认的输出格式包含有限的信息，我们可以看到每个资源仅显示了一些字段，而不是完整的资源定义。此时，自定义列输出格式就非常有用了，它使你可以自由定义列和想在其中显示的数据，你可以选择资源的任何字段，使其在输出中显示为单独的列。自定义列输出选项的用法如下：

```
-o custom-columns=<header>:<jsonpath>[,<header>:<jsonpath>]...
```

你必须将每个输出列定义为 `<header>：<jsonpath>` 对：

- `<header>` 是列的名称，你可以选择任何所需的内容
- `<jsonpath>` 是一个选择资源字段的表达式

让我们看一个简单的例子：

```
➜  ~ kubectl get pods -o custom-columns='NAME:metadata.name'
NAME
nfs-client-provisioner-54f4485448-kwr45
nginx-674ff86d-t6gbd
```

在这里，输出包括显示所有 Pod 名称的一列，选择 Pod 名称的表达式是 `meta.name`，因为 Pod 的名称是在 Pod 资源的 metadata 属性下面的 name 字段中定义的（我们可以使用 `kubectl explain pod.metadata.name` 进行查找）。

现在，假设你想在输出中添加一个附加列，比如显示每个 Pod 在其上运行的节点，那么我们只需在自定义列选项中添加适当的列规范即可：

```
➜  ~ kubectl get pods -o custom-columns='NAME:metadata.name,NODE:spec.nodeName'
NAME                                      NODE
nfs-client-provisioner-54f4485448-kwr45   172.27.0.2
nginx-674ff86d-t6gbd                      172.27.0.2
```

选择节点名称的表达式是 `spec.nodeName`，这是因为已将 Pod 调度的节点保存在 Pod 的 `spec.nodeName` 字段中了。不过需要请注意的是 Kubernetes 资源字段是区分大小写的。

我们可以通过这种方式将资源的任何字段设置为输出列，只需浏览资源规范并尝试使用任何你喜欢的字段即可。当然我们需要对字段选择的 JSONPath 表达式要有一定的了解。

### JSONPath 表达式

用于选择资源字段的表达式基于 JSONPath 的。JSONPath 是一种用于从 JSON 文档提取数据的语言（类似于 `XPath for XML`），选择单个字段只是 JSONPath 的最基本用法，它还有很多其他功能，例如 list selector、filter 等。

但是，`kubectl explain` 仅支持 JSONPath 功能的子集，下面我们通过一些示例用法来总结下这些使用规则：

**选择一个列表的所有元素**

```
# 获取Pod下面的所有容器镜像
➜  ~ kubectl get pods -o custom-columns='IMAGES:spec.containers[*].image'
IMAGES
cnych/nfs-subdir-external-provisioner:v4.0.2
nginx:latest
```

**选择一个列表的指定元素**

```
# 获取Pod下面第一个容器的镜像
➜  ~ kubectl get pods -o custom-columns='IMAGE:spec.containers[0].image'
IMAGE
cnych/nfs-subdir-external-provisioner:v4.0.2
nginx:latest
```

**选择匹配过滤表达式的列表元素**

```
➜  ~ kubectl get pods -o custom-columns='DATA:spec.containers[?(@.image!="nginx:latest")].image'
DATA
cnych/nfs-subdir-external-provisioner:v4.0.2
<none>
```

**选择指定位置下的所有字段**

```
➜  ~ kubectl get pods -o custom-columns='DATA:metadata.*'
DATA
default,[map[apiVersion:apps/v1 blockOwnerDeletion:true controller:true kind:ReplicaSet name:nfs-client-provisioner-54f4485448 uid:39912344-d707-4029-8da8-5269cfcae9e9]],4926994155,map[app:nfs-client-provisioner pod-template-hash:54f4485448],2021-07-06T04:08:48Z,nfs-client-provisioner-54f4485448-,[map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:generateName:map[] f:labels:map[.:map[] f:app:map[] f:pod-template-hash:map[]] f:ownerReferences:map[.:map[] k:{"uid":"39912344-d707-4029-8da8-5269cfcae9e9"}:map[.:map[] f:apiVersion:map[] f:blockOwnerDeletion:map[] f:controller:map[] f:kind:map[] f:name:map[] f:uid:map[]]]] f:spec:map[f:containers:map[k:{"name":"nfs-client-provisioner"}:map[.:map[] f:env:map[.:map[] k:{"name":"NFS_PATH"}:map[.:map[] f:name:map[] f:value:map[]] k:{"name":"NFS_SERVER"}:map[.:map[] f:name:map[] f:value:map[]] k:{"name":"PROVISIONER_NAME"}:map[.:map[] f:name:map[] f:value:map[]]] f:image:map[] f:imagePullPolicy:map[] f:name:map[] f:resources:map[] f:terminationMessagePath:map[] f:terminationMessagePolicy:map[] f:volumeMounts:map[.:map[] k:{"mountPath":"/persistentvolumes"}:map[.:map[] f:mountPath:map[] f:name:map[]]]]] f:dnsPolicy:map[] f:enableServiceLinks:map[] f:restartPolicy:map[] f:schedulerName:map[] f:securityContext:map[] f:serviceAccount:map[] f:serviceAccountName:map[] f:terminationGracePeriodSeconds:map[] f:volumes:map[.:map[] k:{"name":"nfs-client-root"}:map[.:map[] f:name:map[] f:nfs:map[.:map[] f:path:map[] f:server:map[]]]]]] manager:kube-controller-manager operation:Update time:2021-07-06T04:08:48Z] map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:status:map[f:conditions:map[.:map[] k:{"type":"PodScheduled"}:map[.:map[] f:lastProbeTime:map[] f:lastTransitionTime:map[] f:message:map[] f:reason:map[] f:status:map[] f:type:map[]]]]] manager:kube-scheduler operation:Update time:2021-07-06T04:08:49Z] map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:metadata:map[f:annotations:map[.:map[] f:tke.cloud.tencent.com/networks-status:map[]]]] manager:multus operation:Update time:2021-07-06T04:09:24Z] map[apiVersion:v1 fieldsType:FieldsV1 fieldsV1:map[f:status:map[f:conditions:map[k:{"type":"ContainersReady"}:map[.:map[] f:lastProbeTime:map[] f:lastTransitionTime:map[] f:status:map[] f:type:map[]] k:{"type":"Initialized"}:map[.:map[] f:lastProbeTime:map[] f:lastTransitionTime:map[] f:status:map[] f:type:map[]] k:{"type":"Ready"}:map[.:map[] f:lastProbeTime:map[] f:lastTransitionTime:map[] f:status:map[] f:type:map[]]] f:containerStatuses:map[] f:hostIP:map[] f:phase:map[] f:podIP:map[] f:podIPs:map[.:map[] k:{"ip":"172.16.0.87"}:map[.:map[] f:ip:map[]]] f:startTime:map[]]] manager:kubelet operation:Update time:2021-08-02T23:00:33Z]],nfs-client-provisioner-54f4485448-kwr45,/api/v1/namespaces/default/pods/nfs-client-provisioner-54f4485448-kwr45,9c445349-42ce-4e38-b20a-41bb47587d7e,map[tke.cloud.tencent.com/networks-status:[{
    "name": "tke-bridge",
    "ips": [
        "172.16.0.87"
    ],
    "default": true,
    "dns": {}
}]]
......
```

**选择所有具有指定名称的字段，无论其位置如何**

```
➜  ~ kubectl get pods -o custom-columns='IMAGE:..image'
IMAGE
cnych/nfs-subdir-external-provisioner:v4.0.2,cnych/nfs-subdir-external-provisioner:v4.0.2
nginx:latest,nginx:latest
```

其中的 `[ ]` 运算符特别重要，Kubernetes 资源的许多字段都是列表，使用此运算符可以选择这些列表中的某一些元素，它通常与通配符一起使用 `[*]`，以选择列表中的所有项目。

### 示例应用程序

使用自定义列输出格式有无限可能，因为你可以在输出中显示资源的任何字段或字段组合。以下是一些示例应用程序，但你可以自己探索并找到对你有用的应用程序。

> 提示：如果你经常使用这些命令，则可以为其创建一个 shell 别名。

**显示 Pod 的容器镜像**

```
➜  ~ kubectl get pods -o custom-columns='NAME:metadata.name,IMAGES:spec.containers[*].image'
NAME                                      IMAGES
nfs-client-provisioner-54f4485448-kwr45   cnych/nfs-subdir-external-provisioner:v4.0.2
nginx                                     nginx
nginx-674ff86d-t6gbd                      nginx:latest
```

此命令显示每个 Pod 的所有容器镜像的名称。因为一个 Pod 可能包含多个容器。在这种情况下，单个 Pod 的容器镜像在同一列中显示为由逗号分隔的列表。

**显示节点的可用区**

```
➜  ~ kubectl get nodes \
  -o custom-columns='NAME:metadata.name,ZONE:metadata.labels.failure-domain\.beta\.kubernetes\.io/zone'
NAME         ZONE
172.27.0.2   160001
```

如果你的 Kubernetes 集群部署在公有云上，则此命令很有用，它显示每个节点所在的可用区。每个节点的可用区均通过特殊的 `failure-domain.beta.kubernetes.io/zone` 标签获得，如果集群在公有云基础架构上运行，则将自动创建此标签，并将其值设置为节点的可用性区域的名称。

标签不是 Kubernetes 资源规范的一部分，但是，如果将节点输出为 YAML 或 JSON，则可以看到它的相关信息：

```
kubectl get nodes -o yaml
# or
kubectl get nodes -o json
```

## 多集群和命名空间切换

当 kubectl 向 Kubernetes API 发出请求时，它将读取 `kubeconfig` 文件，以获取它需要访问的所有连接参数并向 APIServer 发出请求。默认的 kubeconfig 文件是 `~/.kube/config`，在使用多个集群时，在 kubeconfig 文件中配置了多个集群的连接参数，所以我们需要一种方法来告诉 kubectl  要将其连接到哪个集群中。在集群中，我们可以设置多个命名空间，Kubectl 还可确定 kubeconfig  文件中用于请求的命名空间，所以同样我们需要一种方法来告诉 kubectl 要使用哪个命名空间。

此外我们还可以通过在 `KUBECONFIG` 环境变量来设置它们，还可以为每个 kubectl 命令使用 `--kubeconfig` 选项覆盖默认的 kubeconfig 文件。

### kubeconfig

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2a31PiatFGtAWc878JzAibYWQibwIzeq2bR5THvFSYDIAJnibibX92jOVEGQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

kubeconfig 文件由一组上下文组成，上下文包含以下三个元素：

- Cluster：集群的 API server 地址
- User：集群中特定用户的身份验证凭据
- Namespace：连接到集群时要使用的命名空间

通常大部分用户在其 kubeconfig 文件中为每个集群使用单个上下文，但是，每个集群也可以有多个上下文，它们的用户或命名空间不同，但并不太常见，因此集群和上下文之间通常存在一对一的映射。

在任何指定时间，这些上下文其中之一都可以被设置为当前上下文：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2nsiceWR4SoxyrB1kYGx4bhFkic8nX5SfcC2DDoBrQo0iaGwxd8SGqicIEA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当 kubectl 读取 kubeconfig 文件时，它总是使用当前上下文中的信息，所以在上面的示例中，kubectl 将连接到 Hare 集群。因此，要切换到另一个集群时，你只需在 kubeconfig 文件中更改当前上下文即可：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2ItQlekb0YI0iaaOn102ahXjucYDy9ysOibX0rID9IxHpjaLVZU6eBic2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样 kubectl 现在将连接到 Fox 集群，并切换到同一集群中的另一个命名空间，可以更改当前上下文的命名空间元素的值：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2RgCuRkH7E3xtDb2JrlyVnyqYmmorYwvWRyXwtLrmFnow23x7UIbPmQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在上面的示例中，kubectl 现在将在 Fox 集群中使用 Prod 命名空间，而不是之前设置的 Test 命名空间了。理论上讲，我们可以通过手动编辑 kubeconfig 文件来进行这些更改，`kubectl config` 也命令提供了用于编辑 kubeconfig 文件的子命令：

- kubectl config get-contexts：列出所有上下文
- kubectl config current-context：获取当前上下文
- kubectl config use-context：更改当前上下文
- kubectl config set-context：更改上下文的元素

比如现在我有两个 kubeconfig 文件，分别连接两个集群，现在我们可以使用下面的命令来合并两个 kubeconfig 文件：

```
➜  ~ cp $HOME/.kube/config $HOME/.kube/config.backup.$(date +%Y-%m-%d.%H:%M:%S)
KUBECONFIG=$HOME/.kube/config:$HOME/.kube/ydzs-config kubectl config view --merge --flatten > $HOME/.kube/merged_kubeconfig && mv $HOME/.kube/merged_kubeconfig $HOME/.kube/config
➜  ~ kubectl config get-contexts
CURRENT   NAME                           CLUSTER   AUTHINFO           NAMESPACE
          cls-9kl736yn-context-default   tke       admin
*         kubernetes-admin@kubernetes    local     kubernetes-admin   default
```

通过上面的命令可以将两个 kubeconfig 合并到一起，我们可以看到现在有两个集群和两个上下文，在操作资源对象的时候可以通过使用参数 `--context` 来指定操作的集群：

```
➜  ~ kubectl get pods --context=cls-9kl736yn-context-default
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-54f4485448-kwr45   1/1     Running   1          67d
nginx-674ff86d-t6gbd                      1/1     Running   0          67d
➜  ~ kubectl get pods --context=kubernetes-admin@kubernetes
NAME                                      READY   STATUS    RESTARTS   AGE
nginx                                     1/1     Running   0          26d
```

我们可以看到操作的时候是非常繁琐的，下面我们使用其他的工具来帮助我们自动进行这些更改。

### Kubectx

Kubectx 可以有效帮助我们在集群和命名空间之间进行切换，该工具提供了 `kubectx` 和 `kubens` 命令，使我们可以分别更改当前上下文和命名空间。如果每个集群只有一个上下文，则更改当前上下文意味着更改集群。

如果我们已经安装过 kubectl 插件管理工具 Krew，则直接使用下面的命令来安装 Kubectx 插件即可：

```
kubectl krew install ctx
kubectl krew install ns
```

安装完成后，可以使用 `kubectl ctx` 和 `kubectl ns` 命令进行操作。但是需要注意这种方式不会安装 shell 自动补全脚本，如果需要，可以使用另外的方式进行安装，比如 macOS 下面使用 Homebrew 进行安装：

```
brew install kubectx
```

此安装命令将自动设置 bash/zsh/fish 自动补全脚本，由于经常需要切换不同的集群，很可能会误操作集群，这个时候有个提示就很棒了，我们可以使用 `kube-ps1` 工具来修改 `PS1`。

不过由于我这里本地使用的是 `oh-my-zsh`，所以可以不用安装，直接在 `~/.zshrc` 开启 plugin 加上 `kube-ps1` 就可以了，然后自定义一下，重新 source 下即可：

```
PROMPT='$(kube_ps1)'$PROMPT
KUBE_PS1_PREFIX=""
KUBE_PS1_SYMBOL_DEFAULT=""
KUBE_PS1_DIVIDER="-"
KUBE_PS1_SUFFIX=" "
```

现在我们只需要输入 kubectx 命令就可以切换集群了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YvCBLyBDow1xeDJ5kwdCNz2c4Tc9vY4AlAh35NxUVQZOQfEgKENJN7pzVngMxrVKKcNLFlSlljHOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由于我们配置了 `kube-ps1`，所以在操作的终端前面也直接显示了当前操作的集群，防止操作集群错误。

> kubectx 的另一个十分有用的功能是交互模式，这需要与 fzf 工具一起工作（安装 fzf 会自动启用kubectx交互模式）。交互式模式允许你通过交互式模糊搜索界面选择目标上下文或命名空间。

## kubectl 插件

从1.12版开始，kubectl 就提供了插件机制，可让你使用自定义命令扩展 kubectl，Kubectl 插件作为简单的可执行文件分发，名称形式为 `kubectl-x`，前缀 `kubectl-` 是必填项，其后是允许调用插件的新的 kubectl 子命令。

要安装插件，你只需要将 `kubectl-x` 文件复制到 PATH 中的任何目录并使其可执行，之后，你可以立即使用 `kubectl x` 调用插件。你可以使用以下命令列出系统上当前安装的所有插件：

```
kubectl plugin list
```

Kubectl 插件可以像软件包一样共享和重用，但是在哪里可以找到其他人共享的插件？Krew 项目旨在为共享、查找、安装和管理 kubectl  插件提供统一的解决方案。Krew 根据 kubectl 插件进行索引，你可以从中选择和安装。当然 krew 本身就是一个 kubectl  插件，这意味着，安装 krew 本质上就像安装其他任何 kubectl  插件一样。你可以在GitHub页面上找到krew的详细安装说明：https://github.com/kubernetes-sigs/krew。

下面是一些重要的 krew 命令：

```
# Search the krew index (with an optional search query)
kubectl krew search [<query>]

# Display information about a plugin
kubectl krew info <plugin>

# Install a plugin
kubectl krew install <plugin>

# Upgrade all plugins to the newest versions
kubectl krew upgrade

# List all plugins that have been installed with krew
kubectl krew list

# Uninstall a plugin
kubectl krew remove <plugin>
```

需要注意 `kubectl krew list` 命令仅列出已与 krew 一起安装的插件，而 `kubectl plugin list` 命令列出了所有插件，即与 krew 一起安装的插件和以其他方式安装的插件。

### 创建插件

我们也可以很方便创建自己的 kubectl 插件，只需要创建一个执行所需操作的可执行文件，将其命名为 `kubectl-x`，然后按照如上所述的方式安装即可。可执行文件可以是任何类型，可以是 Bash 脚本、已编译的 Go 程序、Python 脚本，这些类型实际上并不重要。唯一的要求是它可以由操作系统直接执行。

让我们现在创建一个示例插件。前面我们使用 kubectl 命令列出每个 Pod 的容器镜像，我们可以轻松地将此命令转换为可以使用 `kubectl img` 调用的插件。只需创建一个名为 `kubectl-img` 的文件，其内容如下：

```
#!/bin/bash
kubectl get pods -o custom-columns='NAME:metadata.name,IMAGES:spec.containers[*].image'
```

现在，使用 `chmod + x kubectl-img` 使该文件可执行，并将其移动到 PATH 中的任何目录，之后，你可以立即将插件与 `kubectl img` 一起使用了：

```
➜  ~ kubectl img
NAME                                      IMAGES
nfs-client-provisioner-54f4485448-kwr45   cnych/nfs-subdir-external-provisioner:v4.0.2
nginx-674ff86d-t6gbd                      nginx:latest
```

kubectl 插件可以用任何编程或脚本语言编写，如果使用 Shell 脚本，则具有可以轻松从插件调用 kubectl  的优势。但是，你可以使用真实的编程语言编写更复杂的插件，例如使用 Kubernetes 客户端库，如果使用 Go，还可以使用  cli-runtime 库，该库专门用于编写 kubectl 插件。