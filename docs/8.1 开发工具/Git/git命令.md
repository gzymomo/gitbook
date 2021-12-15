[TOC]

- [掘金：秋天不落叶：三年 Git 使用心得 & 常见问题整理](https://juejin.im/post/5ee649ff51882542ea2b5108#heading-34)
- [程序员必备基础：Git 命令全方位学习](https://juejin.cn/post/6844904200111915015)
- [Git实用技巧记录](https://www.escapelife.site/posts/f6ffe82b.html)
- [GitHub官方Action自动化工具](https://www.escapelife.site/posts/1eb5a495.html)



# Git基本概念

![Git基本命令](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b675e7bb00d24232a2338f87d85d00af~tplv-k3u1fbpfcp-zoom-1.image)



基于上面的图，我们就有接下来一些概念👇

- 版本库👉`.git`
  - 当我们使用git管理文件时，比如`git init`时，这个时候，会多一个`.git`文件，我们把这个文件称之为版本库。
  - `.git文件`另外一个作用就是它在创建的时候，会自动创建master分支，并且将HEAD指针指向master分支。
- 工作区 
  - 本地项目存放文件的位置
  - 可以理解成图上的workspace
- 暂存区 (Index/Stage) 
  - 顾名思义就是暂时存放文件的地方，通过是通过add命令将工作区的文件添加到缓冲区
- 本地仓库（Repository）
  - 通常情况下，我们使用commit命令可以将暂存区的文件添加到本地仓库
  - 通常而言，HEAD指针指向的就是master分支
- 远程仓库（Remote）
  - 举个例子，当我们使用GitHub托管我们项目时，它就是一个远程仓库。
  - 通常我们使用clone命令将远程仓库代码拷贝下来，本地代码更新后，通过push托送给远程仓库。



# 1、Git命令

## 1、Git global配置

```shell
# 配置全局用户
$ git config --global user.name "用户名" 
$ git config --global user.email "git账号"
# 配置别名
$ git config --global alias.co checkout
$ git config --global alias.ss status
$ git config --global alias.cm commit
$ git config --global alias.br branch
$ git config --global alias.rg reflog
# 这里只是美化 log 的输出，实际使用时可以在 git lg 后面加命令参数，如： git lg -10 显示最近10条提交
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
# 删除全局配置
$ git config --global --unset alias.xxx
$ git config --global --unset user.xxx
```

git修改完密码后，重置git bash密码。
解决方法：打开电脑的控制面板–>用户账户–>管理Windows凭据（win10可以直接搜索** 凭据管理器**）

## 1.1 查看git信息

![Git配置命令](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29f0c70414b14fe1986b376f7b303959~tplv-k3u1fbpfcp-zoom-1.image)

```shell
# 查看系统配置
$ git config --list
# 查看用户配置
$ cat ~/.gitconfig
# 查看当前项目的 git 配置
$ cat .git/config
# 查看暂存区的文件
$ git ls-files
# 查看本地 git 命令历史
$ git reflog
# 查看所有 git 命令
$ git --help -a
# 查看当前 HEAD 指向
$ cat .git/HEAD

# git 中 D 向下翻一行  F 向下翻页  B 向上翻页  Q 退出
# 查看提交历史
$ git log --oneline
          --grep="关键字"
          --graph
          --all
          --author "username"
          --reverse
          -num
          -p
          --before=  1  day/1  week/1  "2019-06-06"
          --after= "2019-06-06"
          --stat
          --abbrev-commit
          --pretty=format:"xxx"
# oneline -> 将日志记录一行一行的显示
# grep="关键字" -> 查找日志记录中(commit提交时的注释)与关键字有关的记录
# graph -> 记录图形化显示 ！！！
# all -> 将所有记录都详细的显示出来
# author "username" -> 查找这个作者提交的记录
# reverse -> commit 提交记录顺序翻转
# before -> 查找规定的时间(如:1天/1周)之前的记录
# num -> git log -10 显示最近10次提交 ！！！
# stat -> 显示每次更新的文件修改统计信息，会列出具体文件列表 ！！！
# abbrev-commit -> 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符 ！！！
# pretty=format:"xxx" ->  可以定制要显示的记录格式 ！！！
# p -> 显示每次提交所引入的差异（按 补丁 的格式输出）！！！
```

## 1.2 Git 常用命令
```bash
# 查看工作区和暂存区的状态
$ git status
# 将工作区的文件提交到暂存区
$ git add .
# 提交到本地仓库
$ git commit -m "本次提交说明"
# add和commit的合并，便捷写法（未追踪的文件无法直接提交到暂存区/本地仓库）
$ git commit -am "本次提交说明"
# 将本地分支和远程分支进行关联
$ git push -u origin branchName
# 将本地仓库的文件推送到远程分支
$ git push
# 拉取远程分支的代码
$ git pull origin branchName
# 合并分支
$ git merge branchName
# 查看本地拥有哪些分支
$ git branch
# 查看所有分支（包括远程分支和本地分支）
$ git branch -a
# 切换分支
$ git checkout branchName
# 临时将工作区文件的修改保存至堆栈中
$ git stash
# 将之前保存至堆栈中的文件取出来
$ git stash pop
```

## 1.3 Git分支管理

![Git分支管理](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bff7ddbc6a145f993c0841eb81c8998~tplv-k3u1fbpfcp-zoom-1.image)



- 查看本地分支

```bash
git branch
```

- 查看远程分支

```bash
git branch -r
```

- 查看本地和远程分支

```bash
git branch -a
```

- 从当前分支，切换到其他分支

```bash
git checkout <branch-name>
// 举个例子
git checkout feature/tiantian
```

- 创建并切换到新建分支

```bash
git checkout -b <branch-name>
// 举个例子👇
git checkout -b feature/tiantian
```

- 删除分支

```bash
git branch -d <branch-name>
// 举个例子👇
git branch -d feature/tiantian
```

- 当前分支与指定分支合并

```bash
git merge <branch-name>
// 举个例子👇
git merge feature/tiantian
```

- 查看哪些分支已经合并到当前分支

```bash
git branch --merged
```

- 查看哪些分支没有合并到当前分支

```bash
git branch --no-merged
```

- 查看各个分支最后一个提交对象的信息

```bash
git branch -v
```

- 删除远程分支

```bash
git push origin -d <branch-name>
```

- 重命名分支

```bash
git branch -m <oldbranch-name> <newbranch-name>
```

- 拉取远程分支并创建本地分支

```bash
git checkout -b 本地分支名x origin/远程分支名x

// 另外一种方式,也可以完成这个操作。
git fetch origin <branch-name>:<local-branch-name>
// fetch这个指令的话,后续会梳理
```

## 1.4 fetch指令

![Git命令fetch](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c666ec139fe4dc5a08df6b811b9803d~tplv-k3u1fbpfcp-zoom-1.image)



我理解的就是将远程仓库内容更新到本地，最近与师姐开发项目过程中，使用的就是这个命令。

具体是这样子的👇

### fetch推荐写法

```bash
git fetch origin <branch-name>:<local-branch-name>
复制代码
```

- 一般而言，这个origin是远程主机名，一般默认就是origin。
- `branch-name` 你要拉取的分支
- `local-branch-name` 通常而言，就是你本地新建一个新分支，将origin下的某个分支代码下载到本地分支。

举个例子👇

```bash
git fetch origin feature/template_excellent:feature/template_layout
// 你的工作目录下，就会有feature/template_layout
// 一般情况下,我们需要做的就是在这个分支上开发新需求
// 完成代码后,我们需要做的就是上传我们的分支
复制代码
```

### fetch其他写法

- 将某个远程主机的更新，全部取回本地。

```bash
git fetch <远程主机名> 
```

- 这样子的话，取回的是所有的分支更新，如果想取回特定分支，可以指定分支名👇

```bash
git fetch <远程主机名> <分支名>
```

- 当你想将某个分支的内容取回到本地下某个分支的话，如下👇

```
git fetch origin :<local-branch-name>
// 等价于👇
git fetch origin master:<local-branch-name>
```

## 1.5 状态查询

- 查看状态
  - git status
- 查看历史操作记录
  - git reflog
- 查看日志
  - git log



## 1.6 文件暂存

![Git命令文件暂存](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b229cb4872e4991b33181cdad72b59d~tplv-k3u1fbpfcp-zoom-1.image)



- 添加改动到stash
  - git stash save -a “message”
- 删除暂存
  - git stash drop [stash@{ID}](mailto:stash@{ID})
- 查看stash列表
  - git stash list
- 删除全部缓存
  - git stash clear
- 恢复改动
  - git stash pop [stash@{ID}](mailto:stash@{ID})



## 1.7 差异比较

![Git文件比较](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c779e736198247bfb0795b50dced0814~tplv-k3u1fbpfcp-zoom-1.image)



- 比较工作区与缓存区
  - git diff
- 比较缓存区与本地库最近一次commit内容
  - git diff -- cached
- 比较工作区与本地最近一次commit内容
  - git diff HEAD
- 比较两个commit之间差异
  - git diff



# 2、Git思维导图

## Git数据流向图



![](https://www.showdoc.cc/server/api/common/visitfile/sign/1407a1916ff926d3ae69b667ec0e1af1?showdoc=.jpg)

![](https://user-gold-cdn.xitu.io/2020/6/15/172b390eab77fcbd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库



## Git导图

![Git脑图](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/145b0cdfa98a4a9cb724d745a1466c47~tplv-k3u1fbpfcp-zoom-1.image)



Git通常的操作流程👇



![Git经典流程图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1d538d63559402fbcfd82d68b08061c~tplv-k3u1fbpfcp-zoom-1.image)

# 3、GitLab Service
![](https://www.showdoc.cc/server/api/common/visitfile/sign/3a4b814c78dcef21bb49de4749a6e3f0?showdoc=.jpg)

# 4、图解Git命令
## 4.1 git merge
fast-forward模式
![](https://segmentfault.com/img/bVbGazO)

no-fast-forward模式
![](https://segmentfault.com/img/bVbGazQ)

合并冲突修复的过程 ，动画演示如下：
![](https://segmentfault.com/img/bVbGazR)

## 4.2 git rebase
git rebase 指令会复制当前分支的所有最新提交，然后将这些提交添加到指定分支提交记录之上。
![](https://segmentfault.com/img/bVbGazU)

git rebase还提供了 6 种操作模式：
- reword：修改提交信息
- edit：修改此提交
- squash：将当前提交合并到之前的提交中
- fixup：将当前提交合并到之前的提交中，不保留提交日志消息
- exec：在每一个需要变基的提交上执行一条命令
- drop：删除提交
以 drop 为例：
![](https://segmentfault.com/img/bVbGazW)

以 squash 为例：
![](https://segmentfault.com/img/bVbGazW)

## 4.3 git reset
以下图为例：9e78i 提交添加了 style.css 文件，035cc 提交添加了 index.js 文件。使用软重置，我们可以撤销提交记录，但是保留新建的 style.css 和 index.js 文件。
![](https://segmentfault.com/img/bVbGaz6)

**Hard reset硬重置**
硬重置时：无需保留提交已有的修改，直接将当前分支的状态恢复到某个特定提交下。需要注意的是，硬重置还会将当前工作目录（working directory）中的文件、已暂存文件（staged files）全部移除！如下图所示：
![](https://segmentfault.com/img/bVbGaAk)

## 4.4 git revert
举个例子，我们在 ec5be 上添加了 index.js 文件。之后发现并不需要这个文件。那么就可以使用 git revert ec5be 指令还原之前的更改。如下图所示：
![](https://segmentfault.com/img/bVbGaAn)

## 4.5 git fetch
使用 git fetch 指令将远程分支上的最新的修改下载下来。
![](https://segmentfault.com/img/bVbGaAr)

## 4.6 git pull
git pull 指令实际做了两件事：git fetch 和 git merge。
如下图所示：
![](https://segmentfault.com/img/bVbGaAs)

## 4.7 git reflog
git reflog 用于显示所有已执行操作的日志！包括合并、重置、还原，也就是记录了对分支的一切更改行为。
![](https://segmentfault.com/img/bVbGaAw)

如果，你不想合并 origin/master 分支了。就需要执行 git reflog 命令，合并之前的仓库状态位于 HEAD@{1} 这个地方，所以我们使用 git reset 指令将 HEAD 头指向 HEAD@{1}就可以了。
![](https://segmentfault.com/img/bVbGaAG)

# 5、Git常用命令详解
## 5.1 add
将工作区的文件添加到暂存区。

```bash
# 添加指定文件到暂存区（追踪新增的指定文件）
$ git add [file1] [file2] ...
# 添加指定目录到暂存区，包括子目录
$ git add [dir]
# 添加当前目录的所有文件到暂存区（追踪所有新增的文件）
$ git add .
# 删除工作区/暂存区的文件
$ git rm [file1] [file2] ...
# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]
# 改名工作区/暂存区的文件
$ git mv [file-original] [file-renamed]

# Git 2.0 以下版本
#只作用于文件的新增和修改
$ git add .
#只作用于文件的修改和删除
$ gti add -u
#作用于文件的增删改
$ git add -A

# Git 2.0 版本
$ git add . 等价于 $ git add -A
```

- git add . ：操作的对象是“当前目录”所有文件变更，"."  表示当前目录。会监控工作区的状态树，使用它会把工作区的所有变化提交到暂存区，包括文件内容修改（modified）以及新文件（new），但不包括被删除的文件。
- git add -u ：操作的对象是整个工作区已经跟踪的文件变更，无论当前位于哪个目录下。仅监控已经被 add 的文件（即 tracked file），它会将被修改的文件（包括文件删除）提交到暂存区。git add -u 不会提交新文件（untracked file）。（git add --update 的缩写）
- git add -A ：操作的对象是“整个工作区”所有文件的变更，无论当前位于哪个目录下。是上面两个功能的合集（git add --all 的缩写）。

## 5.2 status
```bash
# 查看工作区和暂存区的状态
$ git status
```

## 5.3 commit
```bash
# 将暂存区的文件提交到本地仓库并添加提交说明
$ git commit -m "本次提交的说明"

# add 和 commit 的合并，便捷写法
# 和 git add -u 命令一样，未跟踪的文件是无法提交上去的
$ git commit -am "本次提交的说明"

# 跳过验证继续提交
$ git commit --no-verify
$ git commit -n

# 编辑器会弹出上一次提交的信息，可以在这里修改提交信息
$ git commit --amend
# 修复提交，同时修改提交信息
$ git commit --amend -m "本次提交的说明"
# 加入 --no-edit 标记会修复提交但不修改提交信息，编辑器不会弹出上一次提交的信息
$ git commit --amend --no-edit
```

- git commit --amend 既可以修改上次提交的文件内容，也可以修改上次提交的说明。会用一个新的 commit 更新并替换最近一次提交的 commit 。如果暂存区有内容，这个新的 commit 会把任何修改内容和上一个 commit 的内容结合起来。如果暂存区没有内容，那么这个操作就只会把上次的 commit 消息重写一遍。永远不要修复一个已经推送到公共仓库中的提交，会拒绝推送到仓库

## 5.3 push & pull
分支推送顺序的写法是 <来源地>:<目的地>
```bash
# 将本地仓库的文件推送到远程分支
# 如果远程仓库没有这个分支，会新建一个同名的远程分支
# 如果省略远程分支名，则表示两者同名
$ git push <远程主机名> <本地分支名>:<远程分支名>
$ git push origin branchname

# 如果省略本地分支名，则表示删除指定的远程分支
# 因为这等同于推送一个空的本地分支到远程分支。
$ git push origin :master
# 等同于
$ git push origin --delete master

# 建立当前分支和远程分支的追踪关系
$ git push -u origin master
# 如果当前分支与远程分支之间存在追踪关系
# 则可以省略分支和 -u
$ git push

# 不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机
$ git push --all origin

# 拉取所有远程分支到本地镜像仓库中
$ git pull
# 拉取并合并项目其他人员的一个分支
$ git pull origin branchname
# 等同于 fetch + merge
$ git fetch origin branchName
$ git merge origin/branchName

# 如果远程主机的版本比本地版本更新，推送时 Git 会报错，要求先在本地做 git pull 合并差异，
# 然后再推送到远程主机。这时，如果你一定要推送，可以使用 –-force 选项 
# （尽量避免使用）
$ git push --force origin | git push -f origin
```

## 5.4 branch
```bash
# 查看本地分支
$ git branch | git branch -l
# 查看远程分支
$ git branch -r
# 查看所有分支（本地分支+远程分支）
$ git branch -a
# 查看所有分支并带上最新的提交信息
$ git branch -av
# 查看本地分支对应的远程分支
$ git branch -vv

# 新建分支
# 在别的分支下新建一个分支，新分支会复制当前分支的内容
# 注意：如果当前分支有修改，但是没有提交到仓库，此时修改的内容是不会被复制到新分支的
$ git branch branchname
# 切换分支(切换分支时，本地工作区，仓库都会相应切换到对应分支的内容)
$ git checkout branchname
# 创建一个 aaa 分支，并切换到该分支 （新建分支和切换分支的简写）
$ git checkout -b aaa
# 可以看做是基于 master 分支创建一个 aaa 分支，并切换到该分支
$ git checkout -b aaa master

# 新建一条空分支（详情请看问题列表）
$ git checkout --orphan emptyBranchName
$ git rm -rf .

# 删除本地分支,会阻止删除包含未合并更改的分支
$ git brnach -d branchname
# 强制删除一个本地分支，即使包含未合并更改的分支
$ git branch -D branchname
# 删除远程分支
# 推送一个空分支到远程分支，其实就相当于删除远程分支
$ git push origin  :远程分支名
# 或者
$ git push origin --delete 远程分支名

# 修改当前分支名
$ git branch -m branchname
```

## 5.5 merge 三种常用合并方法
```bash
# 默认 fast-forward ，HEAD 指针直接指向被合并的分支
$ git merge
# 禁止快进式合并
$ git merge --no-ff
$ git merge --squash
```
![](https://user-gold-cdn.xitu.io/2020/6/15/172b390eac6f9586?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- fast-forward：会在当前分支的提交历史中添加进被合并分支的提交历史（得先理解什么时候会发生快速合并，并不是每次 merge 都会发生快速合并）；
- --no-ff：会生成一个新的提交，让当前分支的提交历史不会那么乱；
- --squash：不会生成新的提交，会将被合并分支多次提交的内容直接存到工作区和暂存区，由开发者手动去提交，这样当前分支最终只会多出一条提交记录，不会掺杂被合并分支的提交历史

## 5.6 stash

- 能够将所有未提交的修改保存至堆栈中，用于后续恢复当前工作区内容
- 如果文件没有提交到暂存区（使用 git add . 追踪新的文件），使用该命令会提示 No local changes to save ，无法将修改保存到堆栈中

**使用场景**： 当你接到一个修复紧急 bug 的任务时候，一般都是先创建一个新的 bug 分支来修复它，然后合并，最后删除。但是，如果当前你正在开发功能中，短时间还无法完成，无法直接提交到仓库，这时候可以先把当前工作区的内容 git stash 一下，然后去修复 bug，修复后，再 git stash pop，恢复之前的工作内容。

```bash
# 将所有未提交的修改（提交到暂存区）保存至堆栈中
$ git stash
# 给本次存储加个备注，以防时间久了忘了
$ git stash save "存储"
# 存储未追踪的文件
$ git stash -u

# 查看存储记录
$ git stash list

在 Windows 上和 PowerShell 中，需要加双引号
# 恢复后，stash 记录并不删除
$ git stash apply "stash@{index}"
# 恢复的同时把 stash 记录也删了
$ git stash pop "stash@{index}"
# 删除 stash 记录
$ git stash drop "stash@{index}"
# 删除所有存储的进度
$ git stash clear
# 查看当前记录中修改了哪些文件
$ git stash show "stash@{index}"
# 查看当前记录中修改了哪些文件的内容
$ git stash show -p "stash@{index}"
```

## 5.7 diff
```bash
# 查看工作区和暂存区单个文件的对比
$ git diff filename
# 查看工作区和暂存区所有文件的对比
$ git diff
# 查看工作区和暂存区所有文件的对比，并显示出所有有差异的文件列表
$ git diff --stat
# 注意：
# 1.你修改了某个文件，但是没有提交到暂存区，这时候会有对比的内容
# 一旦提交到暂存区，就不会有对比的内容(因为暂存区已经更新)
# 2.如果你新建了一个文件，但是没有提交到暂存区，这时候 diff 是没有结果的

# 查看暂存区与上次提交到本地仓库的快照（即最新提交到本地仓库的快照）的对比
$ git diff --cached/--staged
# 查看工作区与上次提交到本地仓库的快照（即最新提交到本地仓库的快照）的对比
$ git diff branchname
# 查看工作区与 HEAD 指向（默认当前分支最新的提交）的对比
$ git diff HEAD

# 查看两个本地分支中某一个文件的对比
$ git diff branchname..branchname filename
# 查看两个本地分支所有的对比
$ git diff branchname..branchname
# 查看远程分支和本地分支的对比
$ git diff origin/branchname..branchname
# 查看远程分支和远程分支的对比
$ git diff origin/branchname..origin/branchname

# 查看两个 commit 的对比
$ git diff commit1..commit2
```

## 5.8 remote
```bash
# 查看所有远程主机
$ git remote
# 查看关联的远程仓库的详细信息
$ git remote -v
# 删除远程仓库的 “关联”
$ git remote rm projectname
# 设置远程仓库的 “关联”
$ git remote set-url origin <newurl>
```

## 5.9 tag
```bash
# 默认在 HEAD 上创建一个标签
$ git tag v1.0
# 指定一个 commit id 创建一个标签
$ git tag v0.9 f52c633
# 创建带有说明的标签，用 -a 指定标签名，-m 指定说明文字
$ git tag -a v0.1 -m "version 0.1 released"

# 查看所有标签
# 注意：标签不是按时间顺序列出，而是按字母排序的。
$ git tag

# 查看单个标签具体信息
$ git show <tagname>

# 推送一个本地标签
$ git push origin <tagname>
# 推送全部未推送过的本地标签
$ git push origin --tags

# 删除本地标签
# 因为创建的标签都只存储在本地，不会自动推送到远程。
# 所以，打错的标签可以在本地安全删除。
$ git tag -d v0.1
# 删除一个远程标签（先删除本地 tag ，然后再删除远程 tag）
$ git push origin :refs/tags/<tagname>
```

## 5.10 删除文件
```bash
# 删除暂存区和工作区的文件
$ git rm filename
# 只删除暂存区的文件，不会删除工作区的文件
$ git rm --cached filename
```

# 6、新建一个 Git 项目的两种方式
## 6.1 本地新建好 Git 项目，然后关联远程仓库
```bash
# 初始化一个Git仓库
$ git init 
# 关联远程仓库
$ git remote add <name> <git-repo-url>
# 例如
$ git remote add origin https://github.com/xxxxxx
```

## 6.2 clone 远程仓库
```bash
# 新建好远程仓库，然后 clone 到本地
$ git clone <git-repo-url>

# 将远程仓库下载到（当前 git bash 启动位置下面的）指定文件中，如果没有会自动生成
$ git clone <git-repo-url> <project-name>
```

# 7、分支命名

![Git分支管理规范](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd8abe5e5605411d8dbe5c4faa0054aa~tplv-k3u1fbpfcp-zoom-1.image)



**master分支**

1. 主分支，用于部署生产环境的分支，确保稳定性。
2. master分支一般由develop以及hotfix分支合并，任何情况下都不能直接修改代码。

**develop 分支**

1. develop为开发分支，通常情况下，保存最新完成以及bug修复后的代码。
2. 开发新功能时，feature分支都是基于develop分支下创建的。

**feature分支**

1. 开发新功能，基本上以develop为基础创建feature分支。
2. 分支命名：feature/ 开头的为特性分支， 命名规则: feature/user_module、 feature/cart_module。

**这点我深有体会，我在网易，mentor就是这么教我的，**通常建一个feature分支。

**release分支**

1. release 为预上线分支，发布提测阶段，会release分支代码为基准提测。

**hotfix分支**

1. 分支命名：hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似。
2. 线上出现紧急问题时，需要及时修复，以master分支为基线，创建hotfix分支，修复完成后，需要合并到master分支和develop分支。



# 8、基本操作

创建本地仓库 git init

> git init

链接本地仓库与远端仓库

> git remote add  origin 
>
> origin默认是远端仓库别名  url 可以是**可以使用https或者ssh的方式新建**

检查配置信息

- git config --list

Git user name 与email

> git config --global user.name "yourname"
>
> git config --global user.email  "your_email"

生成SSH密钥

> ssh-keygen -t rsa -C "这里换上你的邮箱"
>
> cd ~/.ssh 里面有一个文件名为id_rsa.pub,把里面的内容复制到git库的我的SSHKEYs中

常看远端仓库信息 

- git remote -v

远端仓库重新命名 

- git remote rename old new

提交到缓存区 

- git add .  全部上传到缓存区
- git add   指定文件

提交到本地仓库

- git commit -m 'some message'

提交远程仓库

- git push <远程主机名> <本地分支名>:<远程分支名>

查看分支

- git  branch

创建新分支

- git branch 

切换分支

- git checkout 

创建分支并切换

- git checkout -b 

删除分支

- git branch -d 

删除远程分支

- git push -d  

切换分支

- git checkout


