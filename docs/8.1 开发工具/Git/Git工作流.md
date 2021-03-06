- [5 个 Git 工作流，改善你的开发流程](https://www.cnblogs.com/xueweihan/p/13524162.html)



## 1. 基本的 Git 工作流

最基本的 Git 工作流是只有一个分支 - master 分支的模式。开发人员直接提交 master 分支并使用它来部署到预发布和生产环境。

![img](https://img2020.cnblogs.com/blog/759200/202008/759200-20200818161329729-796929090.png)

上图为基本的 Git 工作流，所有提交都直接添加到 master 分支。

通常不建议使用此工作流，除非你正在开发一个 side 项目并且希望快速开始。

由于只有一个分支，因此这里实际上没有任何流程。这样一来，你就可以轻松开始使用 Git。但是，使用此工作流时需要记住它的一些缺点：

1. 在代码上进行协作将导致多种冲突。
2. 生产环境出现 bug 的概率会大增。
3. 维护干净的代码将更加困难。

## 2. Git 功能分支工作流

当你有多个开发人员在同一个代码库上工作时，Git 功能分支工作流将成为必选项。

假设你有一个正在开发一项新功能的开发人员。另一个开发人员正在开发第二个功能。现在，如果两个开发人员都向同一个分支提交代码，这将使代码库陷入混乱，并产生大量冲突。

![img](https://img2020.cnblogs.com/blog/759200/202008/759200-20200818161338583-1267594128.png)

上图为具有功能分支的 Git 工作流模型。

为避免这种情况，两个开发人员可以分别从 master 分支创建两个单独的分支，并分别开发其负责的功能。完成功能后，他们可以将各自的分支合并到 master 分支，然后进行部署，而不必等待对方的功能开发完成。

使用此工作流的优点是，Git 功能分支工作流使你可以在代码上进行协作，而不必担心代码冲突。

## 3. 带有 Develop 分支的 Git 功能分支工作流

此工作流是开发团队中比较流行的工作流之一。它与 Git 功能分支工作流相似，但它的 develop 分支与 master 分支并行存在。

在此工作流中，master 分支始终代表生产环境的状态。每当团队想要部署代码到生产环境时，他们都会部署 master 分支。

Develop 分支代表针对下一版本的最新交付的代码。开发人员从 develop 分支创建新分支，并开发新功能。功能开发完毕后，将对其进行测试，与 develop 分支合并，在合并了其他功能分支的情况下使用 develop 分支的代码进行测试，然后与 master 分支合并。

![img](https://img2020.cnblogs.com/blog/759200/202008/759200-20200818161349949-489956003.png)

上图为具有 develop 分支的 Git 功能分支工作流模型。

此工作流的优点是，它使团队能够一致地合并所有新功能，在预发布阶段对其进行测试并部署到生产环境中。尽管这种工作流让代码维护变得更加容易，但是对于某些团队来说，这样做可能会感到有些疲倦，因为频繁的 Git 操作可能会让你感到乏味。

## 4. Gitflow 工作流

Gitflow 工作流与我们之前讨论的工作流非常相似，我们将它们与其他两个分支（ release 分支和 hot-fix 分支）结合使用。

### 4.1 Hot-Fix 分支

Hot-fix 分支是唯一一个从 master 分支创建的分支，并且直接合并到 master 分支而不是 develop 分支。仅在必须快速修复生产环境问题时使用。该分支的一个优点是，它使你可以快速修复并部署生产环境的问题，而无需中断其他人的工作流，也不必等待下一个发布周期。

将修复合并到 master 分支并进行部署后，应将其合并到 develop 和当前的 release 分支中。这样做是为了确保任何从 develop 分支创建新功能分支的人都具有最新代码。

### 4.2 Release 分支

在将所有准备发布的功能的代码成功合并到 develop 分支之后，就可以从 develop 分支创建 release 分支了。

Release 分支不包含新功能相关的代码。仅将与发布相关的代码添加到 release 分支。例如，与此版本相关的文档，错误修复和其他关联任务才能添加到此分支。

一旦将此分支与 master 分支合并并部署到生产环境后，它也将被合并回 develop 分支中，以便之后从 develop 分支创建新功能分支时，新的分支能够具有最新代码。

![img](https://img2020.cnblogs.com/blog/759200/202008/759200-20200818161400164-1833173904.png)

上图为具有 hot-fix 和 release 分支的 Gitflow 工作流模型

此工作流由 [Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/) 首次发布并广受欢迎，已被具有预定发布周期的组织广泛使用。

由于 git-flow 是对 Git 的包装，因此你可以为当前代码库安装 git-flow。git-flow 非常简单，除了为你创建分支外，它不会更改代码库中的任何内容。

要在 Mac 机器上安装 ，请在终端中执行 `brew install git-flow` 。

要在 Windows 机器上安装，你需要 [下载并安装 git-flow](https://git-scm.com/download/win) 。安装完成后，运行 `git flow init` 命令，就可以在项目中使用它了。

## 5. Git Fork 工作流

Fork 工作流在使用开源软件的团队中很流行。

该流程通常如下所示：

1. 开发人员 fork 开源软件的官方代码库。在他们的帐户中创建此代码库的副本。
2. 然后，开发人员将代码库从其帐户克隆到本地系统。
3. 官方代码库的远端源已添加到克隆到本地系统的代码库中。
4. 开发人员创建一个新的功能分支，该分支将在其本地系统中创建，进行更改并提交。
5. 这些更改以及分支将被推送到其帐户上开发人员的代码库副本。
6. 从该新功能分支创建一个 pull request，提交到官方代码库。
7. 官方代码库的维护者检查 pull request 中的修改并批准将这些修改合并到官方代码库中。