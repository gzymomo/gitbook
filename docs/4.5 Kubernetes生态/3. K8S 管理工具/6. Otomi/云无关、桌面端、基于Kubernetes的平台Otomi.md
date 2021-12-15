# 一、Otomi介绍

Otomi官网：https://otomi.io/

Otomi-core核心模块Github地址：https://github.com/redkubes/otomi-core

Otomi是一个<font color='red'>开源的、云无关的、基于kubernetes的平台，通过类似桌面的用户界面安全地部署、运行和管理应用程序。</font>

Otomi易于安装，具有直观的桌面式UI，可以使用预先配置的内置应用程序提供开箱即用的体验。

就像您最喜欢的Linux发行版所期望的那样。在Kubernetes上安装Otomi后，您可以登录并立即开始部署应用程序。

![image-20211012182027231](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012182027231.png)

Otomi是建立在以下开源项目之上的：

- [Istio](https://github.com/istio/istio)

- [Knative Serving](https://github.com/knative/serving)

- [Nginx Ingress](https://github.com/kubernetes/ingress-nginx)

- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)

- [Grafana Loki](https://github.com/grafana/loki)

- [HashiCorp Vault](https://github.com/hashicorp/vault)

- [Gatekeeper](https://github.com/open-policy-agent/gatekeeper)

- [Keycloak](https://github.com/keycloak/keycloak)

- [Gitea](https://github.com/go-gitea/gitea)

- [Drone](https://github.com/drone/drone)

- [Harbor](https://github.com/goharbor/harbor)

- [Cert manager](https://github.com/jetstack/cert-manager)

- [External DNS](https://github.com/kubernetes-sigs/external-dns)

# 二、Otomi提供的特性

- 开发人员自助服务：团队成员可以使用Otomi Console直接访问他们需要的所有工具并创建Services，Jobs和Secrets。
- 预配置和准备使用的应用程序。
- 所有集成应用程序的应用程序配置管理，提供基本配置文件配置以支持最常见的DevOps用例。
- 多租户：创建团队并提供对共享应用程序的SSO访问。
- 实现了更好的治理和安全性的策略。清单将在运行时静态地和在集群上进行检查，以确保策略服从。
- 单点登录：自带IDP或使用Keycloak作为IDP(默认)。
- 自动进入配置：轻松配置Team服务的进入，允许公众在几分钟内访问服务。Istio网关和虚拟服务为Team服务自动生成和配置，以可预测的方式将通用入口体系结构绑定到服务端点。
- 输入/输出验证：静态地检查配置和输出清单的有效性和最佳实践。
- 自动漏洞扫描：扫描Harbor中所有已配置的Team服务容器。
- 内置对Azure、Amazon Web服务和谷歌云平台的支持。

**Otomi的目标是支持最常见的DevSecOps用例，即开箱即用，并强烈依赖于GitOps模式，其中所需的状态以代码形式反映出来，集群状态会自动更新。**

## 2.1 Otomi优势

- 很容易安装
- 自带一个直观的桌面式UI
- 自带准备使用，预配置和内置的应用程序
- 开箱即用的工作



就像您最喜欢的Linux发行版所期望的那样。在Kubernetes上安装Otomi之后，您可以登录并立即开始部署应用程序。



## 2.2 为什么选择Otomi

- 允许定制和扩展。
- 将上游Kubernetes与经过验证的开源应用程序和附加组件集成在一起。
- 单个可部署包是否具有经过行业验证的应用程序和策略，以获得更好的治理和安全性。
- 提供企业级容器平台的开箱即用体验。
- 提高开发人员的效率，使开发人员自我服务。
- 提供精心设计的合理默认值，以减少配置工作和加快时间的市场。
- 结合12因素应用方法和Kubernetes的最佳实践。

# 三、Otomi架构

Otomi由多个项目组成：

- Otomi Core：Otomi的心脏
- [Otomi Tasks](https://github.com/redkubes/otomi-tasks)：由Otomi Core组织的自主工作
- [Otomi API](https://github.com/redkubes/otomi-api)：Otomi的大脑，处理主机输入并与Otomi Core对话
- [Otomi Console](https://github.com/redkubes/otomi-console)：Otomi为管理员和团队提供的UI，与Otomi API对话
- [Otomi Clients](https://github.com/redkubes/otomi-clients)：构建和发布redkubes/ Otomi -tasks repo中使用的openapi客户端的工厂

# 四、安装部署

## 4.1 最低要求及配置

### 4.1.1 客户端二进制文件

请确保以下客户端二进制文件存在:

- Kubectl访问集群。
- Docker必须安装并运行，就像Otomi运行在容器中一样。
- 安装了Helm
- 可选：Otomi CLI客户端

### 4.1.2 Kubernetes和DNS

Otomi至少要求:

- 至少有3个工作节点的正在运行的Kubernetes集群(使用至少4个vCPU的通用实例类型)
- 访问公共DNS区域

Otomi支持Kubernetes版本1.18到1.20

按照下面的说明在你选择的云上设置一个Kubernetes集群和DNS：

#### 1. AWS[#](https://otomi.io/docs/installation/prerequisites/#aws)

Set up an EKS cluster on AWS: https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

使用kubectl访问集群：

```bash
aws eks update-kubeconfig --name $CLUSTER_NAME
```

设置外部DNS：https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/aws.md

#### 2. Azure (AKS)[#](https://otomi.io/docs/installation/prerequisites/#azure-aks)

Set up an AKS cluster on Azure: https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal

使用kubectl访问集群：

```bash
az aks get-credentials --resource-group <resource-group> --name <cluster-name> --admin
```

设置外部DNS：https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure.md

#### 3. Google (GKE)[#](https://otomi.io/docs/installation/prerequisites/#google-gke)

Set up a GKE cluster on Google Cloud Platform: https://cloud.google.com/kubernetes-engine/docs/how-to

使用kubectl访问集群：

```bash
gcloud container clusters get-credentials <cluster-name> --region <region> --project <project>
```

设置外部DNS：https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/gke.md

## 4.2 可选配置

可以选择将Otomi配置为使用外部IDP (Azure AD)，并为sop使用外部密钥管理服务(KMS)提供商。

下面你可以找到关于如何将Azure AD设置为外部IDP和配置KMS的详细说明。

我们将很快为其他idp添加更多说明，如Amazon Incognito，谷歌Identity和Okta。

## 4.3 安装图表

使用Helm安装Otomi。

有关如何使用helm图表的更多细节，请访问helm文档页面。

在开始之前，验证您是否已经满足先决条件：

### 1. 添加Otomi仓库[#](https://otomi.io/docs/installation/chart/#add-the-otomi-repository)

```bash
helm repo add otomi https://otomi.io/otomi-core
helm repo update
```

查看[helm repo](https://helm.sh/docs/helm/helm_repo/)了解更多命令说明。

### 2. 创建测试文件

查看所需的值。Yaml文件的详细注释，查看和下载图表的最新值。

运行以下命令查看所有的值：

```bash
helm show values otomi/otomi
```

使用实例测试输入值是否正确。

```bash
helm template -f values.yaml otomi/otomi
```

### 3. 安装图表

使用以下命令安装图表：

```bash
helm install -f values.yaml otomi otomi/otomi
```

监控图表安装：

图表在默认名称空间中部署一个Job (otomi)。使用kubectl监控图表安装：

```bash
# get the status of the job
kubectl get job otomi -w
# watch the helm chart install status:
watch helm list -Aa
```

### 4. 二进制安装

```bash
git clone https://github.com/redkubes/otomi-core.git
cd otomi-core
```

使用以下命令安装名称为my-otomi-release(您选择的自定义名称)的图表。

```bash
helm install -f values.yaml my-otomi-release chart/otomi
```

### 5. 卸载图表

```bash
helm uninstall my-otomi-release
```

Helm卸载只会移除用于部署Otomi的工作。它不会移除所有已安装的组件。如果您想完全卸载，我们建议首先克隆otomi/values存储库(以确保配置安全)，然后使用otomi CLI卸载。

# 五、安装后配置

安装Otomi之后，需要两个安装后配置操作。遵循这些指令：

## 5.1 登录Otomi控制台

在浏览器中打开url `https://otomi.<domainsuffix>`。</domainsuffix>domainSuffix可以在值中找到。Yaml是在安装期间提供的。

如果Otomi配置了OIDC(使用Azure AD作为IDP)，单击右边的按钮(下面示例中的redkubes-azure)。

如果没有配置OIDC，请先在Keycloak中创建用户。并将该用户添加到otomi-admin组。

![image-20211012181415978](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181415978.png)

成功登录后，您将看到Otomi Dashboard。

要了解更多关于使用Otomi控制台，请查看Otomi控制台。

![image-20211012181445789](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181445789.png)

![image-20211012181455531](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181455531.png)

## 5.2 激活Drone

Gitea和Drone是Otomi集群配置存储和更新的重要组成部分。点击控制台中的Gitea应用程序(在Platform/Otomi Apps下)。它将打开一个新的浏览器选项卡，并显示在Gitea的登录页面。使用默认的otomi-admin帐户登录。

![image-20211012181602228](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181602228.png)

在登录后，可能需要几分钟才能看到otomi/values存储库。

![image-20211012181619399](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181619399.png)

otomi/values存储库保存otomi集群配置，每当通过控制台发生新的更改时，它就会更新。

现在回到控制台激活Drone。

![image-20211012181649982](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181649982.png)

点击Drone 应用程序，它应该打开一个新标签，如下所示，

![image-20211012181722859](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181722859.png)

选择Activate，然后Activate REPOSITORY

![image-20211012181735372](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181735372.png)

保存更改，就可以开始了。

![image-20211012181754043](https://gitee.com/er-huomeng/l-img/raw/master/image-20211012181754043.png)

现在，最后一步是创建Team。有关更多信息，请参见 [Teams](https://otomi.io/docs/console/teams)页面。

