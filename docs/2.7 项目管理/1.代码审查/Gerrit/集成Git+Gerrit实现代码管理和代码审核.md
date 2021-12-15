# 集成Git+Gerrit实现代码管理和代码审核

# 管理员操作

在此就略过git gitlab gerrit的安装过程和管理员的激活了，网上教程很多的。
记得把Gerrit服务器SSH KEY配置到gitlab上

## 1、新建GitLab工程

![在新建Gitlab上新建工程](https://img-blog.csdnimg.cn/20190801135620408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
用gerrit的用户创建新工程，记住命名空间Namespace，并把权限级别设为private，这样可以禁止别的用户通过本地跳过代码审核直接上传到gitlab，如果不做强制代码审核要求的，可以设为public。
新建工程后切记不要clone到本地 不要init仓库。不然可能会在gerrit初始同步时出错。

## 2、新建Gerrit工程

![新建Gerrit工程](https://img-blog.csdnimg.cn/20190801141101200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
用管理员登录Gerrit的UI页面，在browse-responsitories 中点击create new新建工程，工程名与Gitlab中的工程名保持一致。同时，指定owner的时候指定一个用户组或者单个用户，不然没有提交权限，只有查看权限（工程首页不会出现下载并附带提交hook的脚本）

## 3、配置同步插件

![配置同步插件](https://img-blog.csdnimg.cn/20190801141651694.png)
进入gerrit安装目录下的etc文件夹，找到replication.config文件，配置remote.如果已经存在对应命名空间的配置，无需操作。
重启之后等待第一次同步完成后即可在gitlab上看到项目完成初始化 并创建了配置的几个分支

# 开发人员操作

此处也不描述如何注册账号，上传SSH KEY等操作了

## 1、代码下载

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801142452418.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
打开Gerrit 页面，选择工程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801142524516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
选择SSH方式下载，如果你没配置SSH KEY 也可以用HTTP方式。
如果当前用户没有在这个项目的owner组里面，是不会看到上面这个脚本的，只有下面的，即 只能读，不能写。

复制代码下载地址，在git客户端中执行，即可下载代码。

## 代码提交

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164638159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
执行完正常的git add, git commit之后，点击gerrit首页的CREATE CHANGE。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164720902.png)
选择工程和对应提交的分支，点击VIEW COMMANDS，获取推送的git命令。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164737978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
需要审核的代码需要复制第三个推送命令，推送到gerrit仓库后才能进行code review

当然， 可以直接push 到refs/for.master的分支，这个分支就是需要代码审核的。

## 查看审核状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164929718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
进入首页，即可看到刚提交的代码。点击相应的提交记录即可查看对应的代码审查状态。

## 审查代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801165033429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
点击具体的修改，进入详情页面，可以查看改动的文件，点击具体文件，可查看改动内容。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801165053692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)
点击左上角的gerrit自动生成的commit-id可以返回当前提交详情页。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801165116638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1MDQxMjA5,size_16,color_FFFFFF,t_70)

Reply：答复，可以进行打分，分数+2为审核通过提交，-2为驳回，+1和-1为需要再次审核。

Code-Review+2 : 直接打+2分，提交更改
Cherry Pick：切换分支，当执行push操作之后如果发现分支不对，需要切换，可以在这里点击Cherry Pick按钮进行分支切换操作。
Rebase:”变基”,把分叉的提交历史“整理”成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了。
Abandon：撤销push操作，当push之后，如果发现代码有问题，需要修改，可以点击这个按钮，然后修改代码，修改完成后，再次push，然后登陆Gerrit Web UI，会有一个Restore按钮，这时候点击Restore，就可以重新申请Code Review。
Delete Change: 删除修改
Follow-Up: 跟随，需要指定的变更提交后，才会进行提交

在审核人员选择+2后，会出现submit按钮，点击提交后，在下次replication插件同步后，就能在gitlab上看到审核通过的代码了。