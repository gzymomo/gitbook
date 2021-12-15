# gerrit是什么?

Gerrit，一种免费、开放源代码的代码审查软件，使用网页界面。

# gerrit背景

Gerrit，一种免费、开放[源代码](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E6%BA%90%E4%BB%A3%E7%A0%81%2F3969)的代码审查软件，使用网页界面。利用[网页浏览器](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E7%BD%91%E9%A1%B5%E6%B5%8F%E8%A7%88%E5%99%A8)，同一个团队的软件程序员，可以相互审阅彼此修改后的程序代码，决定是否能够提交，退回或者继续修改。它使用[Git](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FGit)作为底层版本控制系统。它分支自Rietveld，作者为[Google](https://links.jianshu.com/go?to=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FGoogle)公司的Shawn Pearce，原先是为了管理Android计划而产生。

Gerrit 官网: [https://www.gerritcodereview.com/](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.gerritcodereview.com%2F)

Gerrit 官方文档: [https://gerrit-documentation.storage.googleapis.com/Documentation/3.0.3/index.html](https://links.jianshu.com/go?to=https%3A%2F%2Fgerrit-documentation.storage.googleapis.com%2FDocumentation%2F3.0.3%2Findex.html)



`Gerrit` 之前的系统架构：

![img](https:////upload-images.jianshu.io/upload_images/20146081-3ff016ed9ac2d6f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)

image.png

`Gerrit` 之后的系统架构：

![img](https:////upload-images.jianshu.io/upload_images/20146081-70787463af1343b0.png?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)

image.png

通过 `Gerrit` 机制将代码做分隔。

工作流程

![img](https:////upload-images.jianshu.io/upload_images/20146081-3976f2873c8b1f13.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

使用过 git 的同学，都知道，当我们`git add --> git commit --> git push` 之后，你的代码会被直接提交到 `repo`，也就是代码仓库中，就是图中橘红色箭头指示的那样。

那么 `gerrit` 就是上图中的那只鸟，普通成员的代码是被先 `push` 到 `gerrit` 服务器上，然后由代码审核人员，就是左上角的 `integrator` 在 `web` 页面进行代码的审核 (`review`)，可以单人审核，也可以邀请其他成员一同审核，当代码审核通过(`approve`) 之后，这次代码才会被提交 (`submit`) 到代码仓库 (`repo`) 中去。

无论有新的代码提交待审核，代码审核通过或被拒绝，代码提交者 (`Contributor`) 和所有的相关代码审核人员 (`Integrator`) 都会收到邮件提醒。`gerrit` 还有自动测试的功能，和主线有冲突或者测试不通过的代码，是会被直接拒绝掉的，这个功能是通过右下角那个老头 (`Jenkins`) 的来实现的,这里不过多讨论。

整个流程就是上面这样。 在使用过程中，有两点需要特别注意下：

- 当进行 `commit` 时，必须要生成一个 `Change-Id`，否则，`push` 到 `gerrit` 服务器时，会收到一个错误提醒。

- 提交者不能直接把代码推到远程的 `master` 主线 (或者其他远程分支) 上去。这样就相当于越过了 `gerrit`了。 `gerrit` 必须依赖于一个`refs/for/*` 的分支。

  > 假如我们远程只有一个 `master` 主线，那么只有当你的代码被提交到`refs/for/master`分支时，`gerrit` 才会知道，我收到了一个需要审核的代码推送，需要通知审核员来审核代码了。当审核通过之后，`gerrit` 会自动将这条分支合并到 `master` 主线上，然后邮件通知相关成员，`master` 分支有更新，需要的成员再去 `pull` 就好了。而且这条`refs/for/master`分支，是透明的，也就是说普通成员其实是不需要知道这条线的，如果你正确配置了 `sourceTree`，你也应该是看不到这条线的。



