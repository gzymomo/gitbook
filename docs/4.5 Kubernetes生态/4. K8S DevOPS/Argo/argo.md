【Github】：https://github.com/argoproj/argo-workflows/

【Website】：https://argoproj.github.io/



# 一、什么是Argo Workflows?

Argo workflow是一个开源的容器本地工作流引擎，用于在Kubernetes上编排并行作业。Argo工作流被实现为Kubernetes CRD(自定义资源定义)。

- 定义工作流，其中每个步骤都是一个容器。

- 将多步骤工作流建模为任务序列，或者使用有向无环图(DAG)捕获任务之间的依赖关系。

- 使用Kubernetes上的Argo工作流，轻松地运行机器学习或数据处理的计算密集型作业。

- 在Kubernetes上本机运行CI/CD管道，无需配置复杂的软件开发产品。

Argo是一个云本地计算基金会(CNCF)托管的项目。

# 二、为什么选择Argon Workflows？

- 完全为容器设计，没有遗留VM和基于服务器的环境的开销和限制。

- 与云无关，可以在任何Kubernetes集群上运行。

- 在Kubernetes上轻松编排高度并行的工作。

- Argo工作流将云级超级计算机放在您的指尖!

# 三、快速开始

要了解Argo是如何工作的，您可以安装它并运行简单工作流的示例，以及使用工件的工作流。

首先，您需要一个Kubernetes集群和kubectl设置。

## 3.1 安装Argo Workflows

为了快速入门，您可以使用快速入门清单，它将安装Argo工作流以及一些常用的组件:

```bash
kubectl create ns argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/quick-start-postgres.yaml
```

注意：在GKE上，您可能需要授予您的帐户创建新的clusterrole的能力。

```bash
kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin --user=YOUREMAIL@gmail.com
```

如果你在本地运行Argo工作流(例如使用Minikube或桌面Docker)，打开一个端口转发，这样你就可以访问命名空间:

```bash
kubectl -n argo port-forward deployment/argo-server 2746:2746
```

这将为http://localhost:2746上的用户界面提供服务



如果你在远程集群上使用运行Argo工作流(例如在EKS或GKE上)，请遵循以下说明。

说明：https://github.com/argoproj/argo-workflows/blob/master/docs/argo-server.md#access-the-argo-workflows-ui



## 3.2 下载安装

接下来，下载最新的Argo CLI。

Linux：

```bash
# Download the binary
curl -sLO https://github.com/argoproj/argo/releases/download/v3.0.0-rc6/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```

## 3.3 Argo Controller

```bash
kubectl create namespace argo
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo/v3.0.0-rc6/manifests/install.yaml
```



最后，提交一个工作流示例:

```bash
argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml
argo list -n argo
argo get -n argo @latest
argo logs -n argo @latest
```

# 四、Argo Server

Argo服务器是一个为工作流公开API和UI的服务器。如果你想卸载大型工作流或工作流存档，你需要使用这个。

您可以在“托管”或“本地”模式下运行。

它取代了Argo UI。



## 4.1 Offloading Large Workflows

https://github.com/argoproj/argo-workflows/blob/master/docs/offloading-large-workflows.md 

Argo将工作流存储为Kubernetes资源(即EtcD内)。这将限制它们的大小，因为资源必须小于1MB。每个资源包含每个节点的状态，存储在资源的/status/nodes字段中。这可以超过1MB。如果发生这种情况，我们尝试压缩节点状态并将其存储在/status/compressedNodes中。如果状态仍然太大，我们将尝试将其存储在SQL数据库中。

要启用该特性，在配置中持久性下配置Postgres或MySQL数据库，并设置nodeStatusOffLoad: true。



## 4.2 Workflow Archive

https://github.com/argoproj/argo-workflows/blob/master/docs/workflow-archive.md

在许多情况下，您可能希望长时间保持工作流。Argo可以将完成的工作流保存到SQL数据库中。

要启用这个特性，在你的配置中配置一个Postgres或MySQL(>= 5.7.8)数据库，并设置archive: true。



## 4.3 托管模式：Hosted Mode

在以下情况下使用此模式:

- 你想要一个顶替Argo UI。

- 如果需要防止用户直接访问数据库。

托管模式作为标准清单的一部分提供，特别是在[manifests](https://github.com/argoproj/argo-workflows/blob/master/manifests), [specifically in argo-server-deployment.yaml](https://github.com/argoproj/argo-workflows/blob/master/manifests/base/argo-server/argo-server-deployment.yaml) .



## 4.4 本地模式：Local Mode

在以下情况下使用此模式:

- 您想要的是不需要复杂设置的东西。

- 您不需要运行数据库。

本地运行:

```bash
argo server
```

这将在端口2746上启动一个服务器，您可以在http://localhost:2746上查看该端口。



## 4.5 选项：Options

### 身份验证模式：Auth Mode

你可以选择Argo Server使用的kube配置:

- server：“服务器”-在托管模式下，使用服务帐户的kube配置，在本地模式下，使用您的本地kube配置。

- client：“客户端”-要求客户提供他们的Kubernetes不记名令牌并使用它。

- “sso”——从2.9版开始，使用单点登录，这将为RBAC使用与每个“服务器”相同的服务帐户。我们希望在将来对此进行更改，以便将OAuth索赔映射到服务帐户。

默认情况下，服务器将以“server”的认证模式启动。

要改变服务器的认证模式，指定列表为多个auth-mode标志:

```bash
argo server --auth-mode sso --auth-mode ...
```



### 管理命名空间：Managed Namespace

可以在集群作用域或命名空间作用域配置中安装Argo。这规定了您是必须设置集群角色还是普通角色。

在命名空间范围配置中，必须使用`--namespace`运行Workflow Controller和Argo Server。

如果您希望工作流在一个分离的名称空间中运行，还可以添加`--managed-namespace`。

例如：

```bash
      - args:
        - --configmap
        - workflow-controller-configmap
        - --executor-image
        - argoproj/workflow-controller:v2.5.1
        - --namespaced
        - --managed-namespace
        - default
```

### 基本代理：Base href

如果服务器运行在反向代理后面，其子路径与/不同(例如，/argo)，则可以使用`--base-href`标志或`BASE_HREF`环境变量设置一个可选的子路径。



### 传输层安全：Transport Layer Security

【Site】：https://github.com/argoproj/argo-workflows/blob/master/docs/tls.md

如果你正在运行Argo Server，你有三个选项来提高传输安全性(注意-你也应该运行身份验证):

#### 纯文本：Plain Text

建议:开发环境

这是默认设置:所有内容都以明文形式发送。

为了确保UI的安全，您可以在前面使用HTTPS代理。

#### 加密：Encrypted

推荐使用于:开发和测试环境

您可以加密连接而不需要任何实际工作。

使用`--secure`标志启动Argo Server，例如:

```bash
argo server --secure
```

它将从一个自签名的证书开始，在365天后过期。

运行命令行`--secure`(或`ARGO_SECURE=true`)和`--insecure-skip-verify`(或`ARGO_INSECURE_SKIP_VERIFY=true`)。

```bash
argo --secure --insecure-skip-verify list

export ARGO_SECURE=true
export ARGO_INSECURE_SKIP_VERIFY=true
argo --secure --insecure-skip-verify list
```

提示:不要忘记更新准备就绪探测以使用HTTPS。为此，编辑你的`argo-server`部署的`readinessProbe`规范:

```yaml
readinessProbe:
    httpGet: 
        scheme: HTTPS
```

#### 加密和验证：Encrypted and Verified

推荐用于:生产环境

在Argo Server前运行您的HTTPS代理。您需要设置证书，这超出了本文档的范围。

启动带有`--secure`标志的Argo Server。

```bash
argo server --secure
```

与以前一样，它将从一个自签名证书开始，该证书在365天后到期。

只运行命令行`--secure`(或 `ARGO_SECURE=true` )。

```bash
argo --secure list

export ARGO_SECURE=true
argo list
```

#### TLS Min Version

设置TLS_MIN_VERSION为使用的最小TLS版本。默认情况下是v1.2。

它必须是这些int值之一。

This must be one of these [int values](https://golang.org/pkg/crypto/tls/).

| Version | Value |
| ------- | ----- |
| v1.0    | 769   |
| v1.1    | 770   |
| v1.2    | 771   |
| v1.3    | 772   |



### SSO

【Site】：https://github.com/argoproj/argo-workflows/blob/master/docs/argo-server-sso.md



## 4.6 访问Argo 工作流界面：Access the Argo Workflows UI

默认情况下，Argo UI服务不通过外部IP公开。要访问UI，使用以下方法之一：

### kubectl port-forward

```
kubectl -n argo port-forward svc/argo-server 2746:2746
```

然后访问: http://127.0.0.1:2746

### Expose a `LoadBalancer`

更新LoadBalancer类型的服务。

```bash
kubectl patch svc argo-server -n argo -p '{"spec": {"type": "LoadBalancer"}}'
```

然后等待外部IP可用:

```bash
kubectl get svc argo-server -n argo

NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
argo-server   LoadBalancer   10.43.43.130   172.18.0.2    2746:30008/TCP   18h
```

### Ingress

你可以通过以下方式获得入口:

在`deployment/argo-server`中添加`BASE_HREF`作为环境变量。不要忘记在后面添加'/'字符。

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argo-server
spec:
  selector:
    matchLabels:
      app: argo-server
  template:
    metadata:
      labels:
        app: argo-server
    spec:
      containers:
      - args:
        - server
        env:
          - name: BASE_HREF
            value: /argo/
        image: argoproj/argocli:latest
        name: argo-server
...
```

创建一个入口，并添加注释`ingress.kubernetes.io/rewrite-target: /`

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: argo-server
  annotations:
    ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - http:
        paths:
          - backend:
              serviceName: argo-server
              servicePort: 2746
            path: /argo(/|$)(.*)
```

