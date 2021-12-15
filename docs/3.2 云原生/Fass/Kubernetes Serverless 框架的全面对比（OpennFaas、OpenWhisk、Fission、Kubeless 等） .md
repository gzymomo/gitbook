- [Kubernetes Serverless 框架的全面对比（OpennFaas、OpenWhisk、Fission、Kubeless 等）](http://dockone.io/article/9074)

Serverless（无服务）其实是个伪名词，它实际上是一套完全抽象出底层硬件技术的技术。很显然，这些函数或功能实际上仍在在某处的服务器上运行，但我们不用在乎这个。开发者只需要提供一个代码函数，然后再通过某种接口来消费或调用：一般是 REST，但也可以通过基于消息的技术来调用（比如 Kafka、Kinesis、Nats、SQS 等）。

下面对 Kubernetes 平台的 Serverless 框架进行了比较，并提供了一些建议：

# 对比结果

下面的表格是对 Kubernetes 无服务框架的一些比较，主要从受欢迎程度、稳定性、工具、技术和易用性这几个方面进行。

像受欢迎程度这种观点的问题是很难量化的，所以我们不得不借助一些指标。比如说，我真的很想知道每天又多少人在使用这个框架，但是像找到一个这样确切的数字是很难的，所以我们借助了 GitHub stars 或者 Google Trends 这样的指标来表示「受欢迎程度」。有关于受欢迎程度的更多信息可以看这里：[Open Source Metrics](https://opensource.guide/metrics/)。

请注意，这里有很多指标只是粗略估计。所以可能看起来某个框架会比实际情况更好或者更差，请结合整体情况来阅读。

​	[![1.jpg](http://dockone.io/uploads/article/20190710/aca2c4ce59a155371e33c04a93df6274.jpg)](http://dockone.io/uploads/article/20190710/aca2c4ce59a155371e33c04a93df6274.jpg)

# Serverless 框架的建议

根据上表的比较，我建议：

- 使用 [Serverless framwork](https://github.com/serverless/serverless) 来作为 SDK
- 在 Kubernetes 上使用 [OpenFaas](https://github.com/openfaas/faas) 或 [OpenWhisk](https://github.com/apache/incubator-openwhisk) 来管理函数
- OpenFaas 成熟、易于使用和可扩展、但和 OpenWhisk 相比，它在核心项目上的活跃开发者比较少，而且也不太受欢迎。（根据我对活跃开发者和受受欢迎程度的定义）
- OpenWhisk 很成熟，很受欢迎并得到了许多活跃开发者的支持，但也很复杂。它使用 Scala 来编写，而且得到了 IBM/Apache 的支持（可能是好事也可能是坏事，这个看自己的观点了）
  所以最后产生的技术栈可能像这样：

​	[![2.png](http://dockone.io/uploads/article/20190710/59e3c2d6d3cddd85e0420f6a51069fbd.png)](http://dockone.io/uploads/article/20190710/59e3c2d6d3cddd85e0420f6a51069fbd.png)


 另外还有一点额外的建议，就是 Serverless framwork 允许开发者将函数部署到 Lambda 或其他 Kubernetes 的无服务平台上。如果你已经有函数部署在了 Lambda 上，这非常有助于函数的迁移。**如果你想和更多Kubernetes技术专家交流，可以加我微信liyingjiese，备注『加群』。群里每周都有全球各大公司的最佳实践以及行业最新动态**。

# 各种框架的介绍

## Severless Framework

这篇文章下面会多次提到 Serverless framwork，所以应该先说下它是什么。

它不是一个平台，但它可以运行任何函数。它是无服务的一个 SDK。事实上，它本质上只是做了一层包装。但最爽的是，通过 Serverless framwork 来打包的函数，你可以将相同的代码部署到 Lambda，Google Functions，Azure  Functions，OpenWhisk，OpenFaas，Kubeless 或 Fn 中。

这么便利的特性非常吸引人，它制定了一个标准，让开发者遵循标准来构建他们的代码，也允许开发者从标准、成本、功能特性或可用性等方面来分析并决定在哪里部署它们。

此外，它也一定程度上让我们不用介意「应该使用哪个框架」。我喜欢 Kubeless 的实现方式，但它还不够成熟。如果我们是基于  Serverless Framework 的话，那么我们可以在 OpenFaas 和 Lambda 上构建我们的函数代码，以后可以轻松地移植到  Kubeless 中。

唯一的缺点是名字起得有点尴尬，很容易造成误解。还有现在支持的语言也有限，但除了这些，我认为这是最安全的选择了。

## OpenWhisk

OpenWhisk 是一个成熟的无服务框架，并且得到 Apache 基金会和 IBM 的支持。IBM 云函数服务也是基于 OpenWhisk 构建的。主要提交者都是 IBM 的员工。

OpenWhisk 利用了 CouchDB、Kafka、Nginx、Redis 和  ZooKeeper，有很多底层的组件，所以增加了一定的复杂性。好处是开发者可以清晰地关注于可伸缩和弹性的服务，缺点是开发者和使用者都需要具备这些工具的知识和学习如何使用，另一个缺点是它重复实现了一些 Kubernetes 中已经存在的特性（比如自动扩缩容）。函数最终会和框架一起运行在 Docker 容器中。

OpenWhisk 可以用 Helm chart 来安装 ，但有些步骤还是需要手动。函数应用可以用 CLI 工具或者 Serverless framework 来部署。Prometheus Metrics（用于监控函数运行各项指标）是开箱即用的。

## OpenFaas

OpenFaas 是一个受欢迎且易用的无服务框架（虽然在上表中不及 OpenWhisk）。但它不像  OpenWhisk 那么受欢迎，而且代码的提交都是基于个人进行的。除了个人开发者在业余时间的贡献外，VMWare 还聘请了一个团队在全职维护  OpenFaas。现在有一家名为 OpenFaas 的公司在英国注册成立了，但还不清楚这个公司和 OpenFaas 项目的有什么关联。

OpenFaas 的架构相对简单一些。可以通过 Kafka、SNS、CloudEvents、CRON 或其他同步/异步触发器来调用  API 网关，其中异步调用是由 NATS Streaming 来处理的。服务的弹性伸缩是使用 Prometheus 和 Prometheus  Alertmanager 来完成的，但也支持[替换成 Kubernetes 的 HorizontalPodAutoscaler](https://stefanprodan.com/2018/kubernetes-scaleway-baremetal-arm-terraform-installer/#horizontal-pod-autoscaling)。

通过 Helm 或 kubectl 可以提供完整的 Kubernetes 安装支持，包括 CRD 的 Operator (比如通过 kubectl 来获取函数)。还有一个 [Kubernetes Operator WIP](https://github.com/openfaas-incubator/openfaas-operator) 很好用：openfaas-operator。

函数应用可以通过 CLI 工具或者 Serverless framework 部署。还提供了一个 “[Funtion Store](https://blog.alexellis.io/announcing-function-store/)”，提供了许多在 OpenFaas 上使用的功能。Prometheus Metrics (用于监控函数运行各项指标) 也是开箱即用的。

## Kubeless

我对 Kubeless 非常感兴趣，因为它是基于原生 Kubernetes 的。工作原理是在原生  Kubernetes 添加了 “函数” 这种自定义资源的 CRD。除了这个实现非常聪明，也意味着它将 Kubernetes  变成了一个函数运行器，而没有像其他框架那样添加了各种复杂的功能，比如消息机制。

我喜欢像标准 Kubernetes 对象那样来管理函数，意味着 Kubernetes 所有常见的好东西都可以开箱即用（比如 Helm、Ark等）。

交互是通过标准 kubectl 进行的，所以没有额外的工具，并且内置了无服务的支持。

这听起来很完美啊，但……

不幸的是它还不够成熟，还不能用户生产。社区也不够大，文档不全（不得不依赖其他文章或帖子）。而且无服务的支持有 bug，这也意味着不能在 Amazon EKS 中使用，

从积极的角度来看，我相信在未来 6 个月内，Kubeless 会成为名副其实的 「Kubernetes serverless framework」。

## Fission

Fission 非常有趣，因为它介于 Kubeless 和 OpenWhisk 中间。它很大程度上了依赖了  Kubernetes 的很多特性，但又没有完全集成。这种方法的好处是它可以利用 Kubernetes  的长处（比如自动弹性伸缩），但在需要做一些其他不同的事情的时候可以获得更好的性能。例如，它有一个相当复杂的冷启动池机制。

Fission 由 [Platform9](https://platform9.com/) 支持，可以通过  Helm 来安装。使用了 Influxdb 来处理状态，以及提供了 FluentD 来收集日志，以便开箱即用。消息机制使用的是  Nats，缓存使用的是 Redis。如你所见，其他框架都没有提供开箱即用的缓存和日志功能，虽然手动添加这些功能也非常简单。

Fission 有一个非常好的扩展叫 [Fission Workflows](https://github.com/fission/fission-workflows/tree/master/Docs)。它是一个让开发者用函数式变成来编写函数的工具。这是一个非常有趣的方向，我很想知道能用他来做些什么。

然而，Fission 的用户很少（在 StackOverflow  上只有两个相关问题，但这以为着它很容易使用么？）。核心的贡献者也非常少，只有 6 个贡献者的提交超过了 10  次。除此之外还有一些其他的缺点，但主要还是因为缺少用户和开发人员，文档也比较缺失。这让开发者很难搞懂框架是怎么运行的，我也不知道代码是怎么进行模板化和启动 Pods 的，这可能会对未来产生一些隐患。总体来说，我对 Fission 的建设感到非常疑惑。「咱啥也不知道，咱也不敢问」

还有一点，Fission 这个名字也很难搜索得到。

## Fn

Fn 这个名字听起来有点尴尬。它是开源的，但主要的贡献者都来自于 Oracle。使用上主要依赖于 Fn CLI，函数运行在 Docker 容器中。[这篇博文](https://medium.com/fnproject/even-wider-language-support-in-fn-with-init-images-a7a1b3135a6e)中有一些关于 CLI 的信息，文档在[这里](https://github.com/fnproject/docs/blob/master/cli/how-to/create-init-image.md)。框架的一些组件可以用 [Helm 来部署](https://github.com/fnproject/fn-helm)。还有一个叫 Fn Flow 的新功能，它可以用来编排多函数，类似 OpenWhisk。

但最重要的区别是工作方式，Fn 更注重于易用新，但这显得它非常自以为是。他提供了函数的热部署（其他框架也能提供这个功能），以及「流函数」（这很独特，但是不清楚这是怎么和其他框架一起工作的）。

Fn 项目是从 2016 年开始的，所以它年纪其实和 OpenWhisk 差不多了，也拥有一定量的贡献者。尽管如此，但有些功能和  Kubernetes 是冲突的，我被这些功能拖累了（比如你不能通过 Kubernetes 来部署  AFAICT）。但这是我的诉求，所以我没有使用下去了。但是它是和 serverless framework 兼容的，一定程度上得到了一些缓解吧。

## IronFunctions

Iron Functions 得到了一家同名公司的支持。所以这里有些坑，在 github 上的  readme 上你点击「documentation」实际上跳转到了 Iron 公司的首页。然后你在页面中再点击「docs」，得到的内容又不是关于 Iron Functions 的。实际上真正的文档在仓库的 docs 目录下。

和其他框架一样，它也是基于 Docker 的。有一个有趣的特点是，它对于 AWS Lambda 有一些特殊的支持。你可以从 Lambda 中获取代码然后直接在 Iron Functions 上运行。这对迁移非常友好。

不幸的是，它并不能像 Fn 那样原生地支持通过 manifest 部署到 Kubernetes，而且也不支持 Serverless  framework。因为这种种缺点，所以我没有继续去尝试使用它了。它也不够受欢迎，没有出现在 Google Trends 上。