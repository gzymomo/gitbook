- [基于Jenkins+Gitlab+Harbor+Rancher+k8s CI/CD实现](https://www.cnblogs.com/xiao987334176/p/13074198.html)
- [Jenkins+harbor+gitlab+k8s 部署maven项目](https://www.cnblogs.com/xiao987334176/p/11434849.html)
- [Jenkins+GitLab+SonnarQube搭建CI/CD全流程](https://www.cnblogs.com/mpolaris/p/14275351.html)



### 1. Jenkins 介绍

Jenkins 是一款著名的可扩展的用于自动化部署的开源 CI/CD 工具。Jenkins 是完全用 Java 编写的，是在 MIT 许可下发布的。它有一组强大的功能，可以将软件的构建、测试、部署、集成和发布等相关任务自动化。这款用于测试的自动化 CI/CD 工具可以在 macOS、Windows 和各种 UNIX 版本（例如 OpenSUSE、Ubuntu、Red Hat 等）系统上使用。除了通过本地安装包安装，它还可以在任何安装过 Java 运行时环境（Java Runtime Environment，JRE）的机器上单独安装或者作为一个 Docker 安装。Jenkins 团队还有一个子项目叫做 Jenkins X，专门运行一个与 Kubernetes 无缝衔接的开箱即用的 pipeline。Jenkins X 巧妙地集成了 Helm、Jenkins CI/CD 服务器、Kubernetes 以及其它一些工具，来提供一个内置最佳实践的规范的 CI/CD 工具 pipeline，例如使用 GitOps 来管理环境。使用 Jenkins 的一个加分点是，其脚本结构良好、易于理解并且可读性很强。Jenkins 团队已经开发了近 1000 个插件，使得应用程序可以与其它熟悉的技术混合使用。除此之外，还可以使用 Credentials Command 之类的插件。这使得向脚本中添加隐藏的身份验证凭证等变得简单可行。一旦 Jenkins pipeline 开始运行，你还可以验证每个阶段通过与否以及每个阶段的总数。但是，你不能在提供的图形化概览中检查特定作业的状态。你可以做的是跟踪终端中的作业进度。

### 2. Jenkins 核心特性

Jenkins 以其易于配置、自动化构建过程和它向用户提供的大量文档而闻名。当谈到 DevOps 测试时，Jenkins 被认为是非常可靠的，而且没必要监视整个构建过程，而对于其它 CI/CD 工具则不会这么放心。

让我们看看 Jenkins 提供的一些最重要的特性:

1. 免费、开源且易安装

Jenkins 在 macOS、Unix、Windows 等平台上都非常容易安装。它可以与 Docker 结合，为自动化作业带来更高的一致性和额外的速度。它可以可以作为一个 servlet 运行在 Apache Tomcat 和 GlassFish 这样的 Java 容器中。你可以找到许多支持和文档来指导整个安装过程。

2. 广泛的插件生态系统

这个工具的插件生态系统相比于其它 CI/CD 工具来说更成熟。目前，这个生态系统提供了 1500+ 插件。由于这些插件的范围从特定语言开发工具到构建工具，这使得定制化变得非常简单便利。因此，你不需要购买昂贵的插件。Jenkins 插件集成也适用于一些 DevOps 测试工具。

3. 易于安装和配置

这个工具的配置过程非常简单，只需要在安装时操作一些步骤。Jenkins 的升级过程也不麻烦且非常直接。而且其提供的支持文档对于你根据自己的需求配置工具也帮助很大。

4. 有用的社区

如你所知，这是一个开源项目，拥有一个庞大的插件生态系统，所有插件的功能都得到了大量社区贡献的支持。伴随 Jenkins 的惊人的社区参与度也是促进其成熟的一个主要原因。

5. 提供 REST API

Jenkins 提供了 REST 风格的应用程序接口来便于扩展。Jenkins 的远程接入 API 有三种不同的风格——Python、XML 以及 JSON（支持 JSONP）。Jenkins 网站中有一个页面有关于 Jenkins API 的描述性文档，有助于扩展。

6. 支持并行执行

Jenkins 支持并行测试。你可以轻松将它与不同的工具集成并得到构建是否成功的通知。开发者甚至可以在不同的虚拟机上并行执行多个构建来加速测试过程。

7. 轻松分配工作

它可以毫不费力地运行分布式工作，即任务在不同的机器上运行，而不会对 GUI（用户图形界面）造成影响。值得一提的是，与其它 CI/CD 工具相比，只有这款工具能够使用与运行 GUI 相关任务的同一个实例。

### 3. GitLab CI/CD 介绍

在所有用于测试的 CI/CD 工具中，GitLab CI/CD 毫无疑问是最新且最受赞赏的选择。它是一款免费且自托管的内置于 GitLab CI/CD 的持续集成工具。GitLab CI/CD 有一个社区版本，提供了 git 仓库管理、问题跟踪、代码评审、wiki 和活动订阅。许多公司在本地安装 GitLab CI/CD，并将它与 Active Directory 和 LDAP 服务器连接来进行安全授权和身份验证。GitLab CI/CD 先前是作为一个独立项目发布的，并从 2015 年 9 月发布的 GitLab 8.0 正式版开始集成到 GitLab 主软件。一个单独的 GitLab CI/CD 服务器可以管理 25000 多个用户，它还可以与多个活跃的服务器构成一个高可用性的配置。GitLab CI/CD 和 GitLab 是用 Ruby 和 Go 编写的，并在 MIT 许可下发布。除了其它 CI/CD 工具关注的 CI/CD 功能之外，GitLab CI/CD 还提供了计划、打包、源码管理、发布、配置和审查等功能。GitLab CI/CD 还提供了仓库，因此 GitLab CI/CD 的集成非常简单直接。在使用 GitLab CI/CD 时，phase 命令包含一系列阶段，这些阶段将按照精确的顺序实现或执行。在实现后，每个作业都被描述和配置了各种选项。每个作业都是一个阶段的一个部分，会在相似的阶段与其它作业一起自动并行运行。一旦你那样做，作业就被配置好了，你就可以运行 GitLab CI/CD 管道了。其结果会稍后演示，而且你可以检查某个阶段你指定的每一个作业的状态。这也是 GitLab CI/CD 与其它用于 DevOps 测试的 CI/CD 工具的不同之处。

### 4. GitLab CI/CD：核心特性

GitLab CI/CD 是最受欢迎的用于 DevOps 测试的 CI/CD 工具之一。GitLab CI/CD 文档丰富、易于控制且用户体验好。如果你刚接触 GitLab CI/CD，我列举了 GitLab CI/CD 的主要功能，会有助于你了解它。来看看吧。

1. 高可用性部署

它被广泛采用，是最新可用的开源 CI/CD 工具之一。GitLab CI/CD 的安装和配置都很简单。它是内置于 GitLab 的免费且自托管的持续集成工具。GitLab CI/CD 逐渐发展成最受欢迎的用于自动化部署的免费 CI/CD 工具之一。

2. Jekyll 插件支持

Jekyll 插件是一个静态网站生成器，对 GitHub Pages 有比较好的支持，它使得构建过程更简单。Jekyll 插件支持使用 HTML 文件和 Markdown，基于你的布局偏好，创建一个完全静态的站点。你可以通过编辑你的 _config.yml 文件来很容易地配置大部分 Jekyll 设置，例如，你的网站的插件和主题。

3. 里程碑设置

工具中的里程碑设置是跟踪问题、改进系列问题、绘制仓库的请求的一种很好的方法。你可以轻易将项目里程碑分配给任何问题，或者合并项目中不常见的请求，或者将组里程碑分配给一组问题，或者合并该组中任何项目的请求。

4. 自动伸缩的持续集成运行器

自动伸缩的 GitLab 持续集成运行器可以轻松管理和节省 90%  EC2 成本。这真的非常重要，特别是对于并行测试环境。而且，对于组件级别或者项目级别的运行器，可以跨代码库使用。

5. 问题跟踪和问题讨论

由于其强大的问题跟踪和问题讨论功能，GitLab 是无数开源项目首选的 CI/CD 工具。它巧妙地允许你并行测试拉取请求和分支。为了简单方便地监控，测试结果被显示在 GitHub UI 上。由于简单的用户界面，相比于 Jenkins，它使用起来更加友好。

6. 使用访问控制管理 Git 仓库

你可以通过访问权限轻松管理 git 仓库。你可以轻松地向单个仓库的协作者授予写入 / 读取访问权限，甚至特定组织的成员也可以对组织的仓库进行更细粒度的访问控制。

7. 活跃的社区支持

活跃且进步的社区是 GitLab CI/CD 的一个主要加分点。提供的所有支持都是开箱即用的，不需要在额外的插件安装中进行修改。

8. 代码评审和合并请求

GitLab CI/CD 不仅仅用于构建代码，还用于评审代码。它允许使用简单的合并请求和合并管理系统来进行改进协作。它几乎支持所有的版本控制系统和构建环境。在 GitHub 项目下实现了大量协作方案，这些项目有助于 GitLab CI/CD 的扩展。

### 5. Jenkins vs GitLab CI/CD 的功能对比

Jenkins 和 GitLab CI/CD 都有它们非常擅长的领域和各自的技术追随者。然而，在讨论 Jenkins vs GitLab CI/CD 之争时，会讨论许多功能。下图是这两个 CI/CD 工具提供的所有功能的比较。![img](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VOlOKu8dzOx1nicHZCqtSo0hj1ggVl69dcvU9X82icoFIVeiaUdial8miaZUJzmDLNibAtRKondtG1LiakVw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 6. Jenkins vs GitLab CI/CD 之间的区别

既然你已经看了 Jenkins vs GitLab CI/CD 之间的功能对比，那也是时候来看看这两个 DevOps 测试工具之间的差别。这些差别将帮助你理解 Jenkins vs GitLab CI/CD 之争背后的真正原因。在 GitLab CI/CD 的帮助下，你可以通过对分支和其它一些方面的完全控制来控制 Git 仓库，从而使你的代码免受突然的威胁。然而，使用 Jenkins 时，你虽然可以控制代码库，但只有几个方面。Jenkins 不允许完全控制分支和其它方面。Jenkins 是“内部托管的”和“免费开源的”，这也是程序员选择它的原因。另一方面，GitLab CI/CD 是“自托管的”和“免费的”，这就是为什么开发人员更喜欢它。在 GitLab CI/CD 中，每一个项目都有一个跟踪程序，它将跟踪问题并进行代码评审来提高效率。而在 Jenkins 工具中，它改变了一些设置支持和一个简单的安装配置过程。

### 7. Jenkins vs GitLab CI/CD 优缺点

我希望你现在理解 Jenkins vs GitLab CI/CD 这两个工具。为了更进一步，我列举了与 Jenkins vs GitLab CI/CD 有关的主要优点和缺点。我知道你已经决定了你要使用的 DevOps 测试工具，本节将帮您增强选择正确的 CI/CD 工具的信念。

Jenkins 的优点

- 大量插件库
- 自托管，例如对工作空间的完全控制
- 容易调试运行，由于对工作空间的绝对控制
- 容易搭建节点
- 容易部署代码
- 非常好的凭证管理
- 非常灵活多样的功能
- 支持不同的语言
- 非常直观

Jenkins 的缺点

- 插件集成复杂
- 对于比较小的项目开销比较大，因为你需要自己搭建
- 缺少对整个 pipeline 跟踪的分析

GitLab CI/CD 的优点

- 更好的 Docker 集成
- 运行程序扩展或收缩比较简单
- 阶段内的作业并行执行
- 有向无环图 pipeline 的机会
- 由于并发运行程序而非常易于扩展收缩
- 合并请求集成
- 容易添加作业
- 容易处理冲突问题
- 良好的安全和隐私政策

GitLab CI/CD 的缺点

- 需要为每个作业定义构建并上传 / 下载
- 在实际合并发生之前测试合并状态是不可能的
- 还不支持细分阶段

### 8. Jenkins vs GitLab CI/CD 如何选

Jenkins 和 GitLab CI/CD 都有它们各自的优点和缺点，你在这两个工具之间的最终选择取决于项目需求和规格。其中每一个 CI/CD 工具都有它自己的优势和劣势，发布时都实现了完全相同的需求：自动化 CI/CD（持续集成和交付）的过程。Jenkins 用于持续集成，而 GitLab CI/CD 用于代码协作和版本控制。在选择最佳的用于 DevOps 测试的 CI/CD 工具时，除了突出的特性，你还应该查看价格列表和内部熟练度。