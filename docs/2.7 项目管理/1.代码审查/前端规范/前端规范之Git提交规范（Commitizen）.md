- [前端规范之Git提交规范（Commitizen）](https://www.cnblogs.com/Yellow-ice/p/15353900.html)

   代码规范是软件开发领域经久不衰的话题，几乎所有工程师在开发过程中都会遇到或思考过这一问题。而随着前端应用的大型化和复杂化，越来越多的前端团队也开始重视代码规范。同样，前段时间，笔者所在的团队也开展了一波开源治理，而其中代码规范就占据了很重要的一项。接下来的几篇文章，将会对JS代码规范、CSS规范、Git提交规范、Git工作流规范以及文档规范进行详细的介绍~

  系列文章：

-   前端规范之JS代码规范（ESLint + Prettier）
-   前端规范之CSS规范（Stylelint）
-   前端规范之Git提交规范（Commitizen）
-   前端规范之Gti工作流规范（Husky + Commitlint + Lint-staged）
-   前端规范之文档规范

  本文主要介绍了前端规范之Git提交规范（Commitizen），将会对Commitizen的使用进行介绍，欢迎大家交流讨论~

# 1. 背景

  Git是目前世界上最先进的分布式版本控制系统，在我们平时的项目开发中已经广泛使用。而当我们使用Git提交代码时，都需要写Commit Message提交说明才能够正常提交。

```
git commit -m "提交"
```

  然而，我们平时在编写提交说明时，通常会直接填写如"fix"或"bug"等不规范的说明，不规范的提交说明很难让人明白这次代码提交究竟是为了什么。而在工作中，一份清晰简介规范的Commit Message能让后续代码审查、信息查找、版本回退都更加高效可靠。因此我们需要一些工具来约束开发者编写符合规范的提交说明。

# 2. 提交规范

  那么，什么样的提交说明才能符合规范的说明呢？不同的团队可以制定不同的规范，当然，我们也可以直接使用目前流行的规范，比如[Angular Git Commit Guidelines](https://zj-git-guide.readthedocs.io/zh_CN/latest/message/Angular提交信息规范/)。接下来将会对目前流行的Angular提交规范进行介绍。

## 2.1 提交格式

  符合规范的Commit Message的提交格式如下，包含了页眉（header）、正文（body）和页脚（footer）三部分。其中，header是必须的，body和footer可以忽略。

```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

## 2.2 页眉设置

  页眉（header）通常只有一行，包括了提交类型（type）、作用域（scope）和主题（subject）。其中，type和subject是必须的，scope是可选的。

  **2.2.1 提交类型**

  提交类型（type）用于说明此次提交的类型，需要指定为下面其中一个：

 [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929173447730-1762258724.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929173447730-1762258724.png)

  **2.2.2 作用域**

  作用域（scope）表示此次提交影响的范围。比如可以取值api，表明只影响了接口。

  **2.2.3 主题**

   主题（subject）描述是简短的一句话，简单说明此次提交的内容。

## 2.3 正文和页脚

  正文（body）和页眉（footer）这两部分不是必须的。

   如果是破坏性的变更，那就必须在提交的正文或脚注加以展示。一个破坏性变更必须包含大写的文本 BREAKING  CHANGE，紧跟冒号和空格。脚注必须只包含 BREAKING CHANGE、外部链接、issue  引用和其它元数据信息。例如修改了提交的流程，依赖了一些包，可以在正文写上：BREANKING CHANGE：需要重新npm  install，使用npm run cm代替git commit。

  下面给出了一个Commit Message例子，该例子中包含了header和body。

```
chore: 引入commitizen

BREANKING CHANGE：需要重新npm install，使用npm run cm代替git commit
```

  当然，在平时的提交中，我们也可以只包含header，比如我们修改了登录页面的某个功能，那么可以这样写Commit Message。

```
feat(登录）：添加登录接口
```

# 3. Commitizen

   虽然有了规范，但是还是无法保证每个人都能够遵守相应的规范，因此就需要使用一些工具来保证大家都能够提交符合规范的Commit  Message。常用的工具包括了可视化工具和信息交互工具，其中Commitizen是常用的Commitizen工具，接下来将会先介绍Commitizen的使用方法。

## 3.1 什么是Commitizen

  [Commitizen](https://github.com/commitizen/cz-cli)是一个撰写符合上面Commit Message标准的一款工具，可以帮助开发者提交符合规范的Commit Message。

## 3.2 安装Commitizen

  可以使用npm安装Commitizen。其中，cz-conventional-changelog是本地适配器。

```
npm install commitizen cz-conventional-changelog --save-dev
```

## 3.3 配置Commitizen

   安装好Commitizen之后，就需要配置Commitizen，我们需要在package.json中加入以下代码。其中，需要增加一个script，使得我们可以通过执行npm run cm来代替git commit，而path为cz-conventional-changelog包相对于项目根目录的路径。

```
”script": {  "cm: "git-cz"},"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

  配置完成之后，我们就可以通过执行npm run cm来代替git commit，接着只需要安装提示，完成header、body和footer的编写，就能够编写出符合规范的Commit Message。

 [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929191142082-1136383014.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929191142082-1136383014.png)

 

# 4. 可视化提交工具

  除了使用Commitizen信息交互工具来帮助我们规范Commit Message之外，我们也可以使用编译器自带的可视化提交工具。接下来，将会介绍VSCode可视化提交工具的使用方法。

  在VSCode的EXTENSIONS中找到[git-commit-plugin](https://marketplace.visualstudio.com/items?itemName=redjue.git-commit-plugin)插件，点击install进行安装。

  [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192049246-1651989740.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192049246-1651989740.png)

  安装完成之后，可以通过git add添加要提交的文件，接着，在Source Control点击show git commit template图标，开始编写Commit Message信息。

  [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192640897-947164044.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192640897-947164044.png)

   接下来只需要按照指引进行Commit Message的编写。

  [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192830008-1279897014.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192830008-1279897014.png)

  当编写完成之后，可以得到符合规范的Commit Message，这个时候就可以放心将Commit Message及所修改的文件进行提交啦。

  [![img](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192921212-716284637.png)](https://img2020.cnblogs.com/blog/831247/202109/831247-20210929192921212-716284637.png)