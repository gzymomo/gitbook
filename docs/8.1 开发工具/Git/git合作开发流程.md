- [git合作开发流程](https://www.cnblogs.com/jzcn/p/15109406.html)



## 一、创建项目与管理

创建项目和管理项目都是管理账号需要做的事情，如果只是合作开发不进行管理，只需要浏览第二部分的内容即可。

#### 1.创建项目

登录代码托管网站，点击添加项目，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164123719-1079184208.png)
 填写相应的项目信息，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164154062-738652253.png)
 完成会生成项目的url，复制url后面会使用到，使用指令时需要注意每个项目的都不一样，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164258611-2081032250.png)
 在本地创建项目文件，并创建项目说明文件“README.md”，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164321362-2035979971.png)
 打开git执行如下命令操作
 初始化git bash客户端，进入创建的项目文件夹执行如下命令（也可以想项目文件夹中右键打开，省去cd命令）

```
git init
```

把文件添加到缓冲区，并添加注释信息

```
git add README.md
git commit -m "first commit"
```

*注：在 Linux 系统中，commit 信息使用单引号 **'**，Windows 系统，commit 信息使用双引号 **"**。*
 推送创建的仓库，其中url是之前复制的

```
git remote add origin url
git push -u origin master
```

执行以上命令操作后，项目便创建成功了，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164436403-442707056.png)

#### 2.添加协作者

点击仓库设置，添加协作者，及协作者的操作权限，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164515970-306952726.png)
 这样简单的git项目就创建完成了。能访问到项目的协作者便可以开始项目的编写了。

#### 3.合并请求管理

当有人发起合并请求时，会有相应的信息提醒，可以查看具体的请求说明，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164558056-1612059261.png)
 查看明细后，如果觉得没问题后，点击合并请求即可完成代码的合并。如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164640337-952125098.png)
 合并完成后，协作人员只需要拉取一下主分支的代码即更新本次更改的内容。

## 二、git仓库使用

#### 1.派生主分支

登录协作者的账号即可使用相应的项目，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164715785-2140654706.png)
 选择自己需要的项目并单击进入，此时便可以看到克隆的url，合作中不建议直接克隆主分支的项目，需要派生自己的分支，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164808364-384852383.png)
 派生完成后会发现项目的路径与主分支的不同，复制个人派生的url，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164847053-427117172.png)

#### 2.配置远程仓库

打开git bash 使用git clone url命令克隆分支仓库，其中url是个人派生出来的url

```
git clone url
```

添加远程仓库fork的上游主库，其中rul是主分支的url

```
git remote add upstream url
```

查看仓库的设置地址

```
git remote -v
```

能看到origin和upstream的地址，则说明配置成功，如图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806164939302-87369817.png)
 到此仓库配置已经完成，接下来便可以进行开发了。

#### 2.更新本地仓库

每次编写代码时，记得同步远程仓库到本地资源库，保证本地仓库和远程仓库的代码一直性

```
git pull upstream master
git pull origin master
```

注意：其中origin是更新个人分支到本地仓库，upstream是更新主分支到本地资源库，因为个人分支的代码多数只能自己更改，一般情况下个人分支的代码和本地基本一致所以更新origin的频率会少一些。主要是主分支由于协作的人较多，代码变动很大。

#### 3.提交代码

提交代码之前记得再次同步主分支的代码，也就是说执行以下步骤是记得使用`git pull upstream master`，这样能保证在合并时避免和主分支的代码产生冲突。
 添加所有更新至本地缓存

```
git add .
```

查看缓存区状态

```
git status	
```

提交到说明，便于版本管理

```
git commit -m "提交说明"
```

提交到远程个人仓库（个人仓库名+分支名）

```
git push origin master
```

这样已经完成代码的提交，提交完成后还需要将自己分支的代码合并到主分支。

#### 4.代码合并

去远程管理仓库进入到个人分支，点击创建合并请求，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806165112751-944438546.png)
 选择需要合并到的分支以及拉去代码的位置，如下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806165131156-1059049295.png)
 完成后点击创建合并请求并填写合并请求的说明已经更改代码的功能，便于管理员对代码进行管理。如
 下图所示：
 ![img](https://img2020.cnblogs.com/blog/2406897/202108/2406897-20210806165159956-1229799597.png)
 到此个人开发的流程已经完成了，最后只需要理员同意合并请求便可以在主分支看到个人更改的代码。

## 三、git其他指令

#### 1.强制拉取覆盖

强制拉取个人分支，并覆盖本地仓库，主要用于自己删除本地文件后无法通过更新下载已删除的文件时使用，当然可以回滚至上一版本。

```
git fetch --all
git reset --hard origin/master
git pull
```

#### 2.本地指令

```
git config --list									#查看配置信息
git init											#初始化仓库
git add 1.txt										#添加文件至缓存
git add .											#添加所有文件至缓存
git rm 1.txt										#删除文件
git status											#查看仓库状态
git commit –m "test"								#提交说明
git rm 1.txt										#删除文件
git commit -m “test”								#删除相应的提交	
git diff a.txt										#查看a.txt文件更改的内容
git log												#查看提交记录
git reset --hard HEAD^								#回滚上一个版本
git reset --hard HEAD~n								#回滚n个版本
git xxx --help										#查看指令帮助
```

#### 3.本地仓库上传至远程仓库

```
git pull origin master								#拉取远程主分支
git pull --rebase origin master						#拉取本地分支
git push -u origin master							#提交代码至个人分支
git push -u -f origin master						#强制上传代码至个人分支
```

#### 4.远程仓库指令

```
git clone url										#克隆仓库
git remote add										#添加/关联一个远程仓库，默认名是origin
git remote remove origin							#删除远程库的 origin 别名
git remote add upstream url							#添加一个将被同步给fork远程的上游仓库
git fetch upstream									#从上游仓库fetch分支和提交点，传送到本地，并会被存储在一个本地分支 upstream/master
git remote											#查看远程库的别名
git remote –v										#查看远程库的别名和仓库地址
git push origin master								#把本地 master 分支推送到别名为 origin 的远程库
git branch											#查看当前所有的分支，默认只有master 分支
git branch test										#创建 test 分支
git branch –d test									#删除 test 分支
git checkout test									#从当前分支切换到 test 分支
git checkout –b dev									#创建 dev 分支，并切换到 dev 分支上
git merge dev										#在当前分支上合并 dev 分支
git merge upstream/master							#把 upstream/master 分支合并到本地 master 上
git merge upstream/dev								#把 upstream/dev 分支合并到本地 dev 上
```

注：由于本次的文档是在本地编写的，后来图片不小心被我删除了，所以我在PDF文档中截屏的，导致图片有点模糊，希望小伙伴们不要建议。