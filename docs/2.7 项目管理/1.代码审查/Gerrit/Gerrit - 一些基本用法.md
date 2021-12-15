- [Gerrit - 一些基本用法](https://www.cnblogs.com/anliven/p/12019989.html)

# 1 - 主配置文件

主配置文件位于`$GERRIT_SITE/etc/gerrit.config`目录

```
[gerrit@mt101 ~]$ cat gerrit_testsite/etc/gerrit.config
[gerrit]
    basePath = git
    canonicalWebUrl = http://192.168.16.101:8083/
    serverId = 0b911b9e-195a-46b0-a5cd-b407b776b344
[container]
    javaOptions = "-Dflogger.backend_factory=com.google.common.flogger.backend.log4j.Log4jBackendFactory#getInstance"
    javaOptions = "-Dflogger.logging_context=com.google.gerrit.server.logging.LoggingContext#getInstance"
    user = root
    javaHome = /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre
[index]
    type = lucene
[auth]
    type = HTTP
[receive]
    enableSignedPush = false
[sendemail]
    smtpServer = localhost
[sshd]
    listenAddress = *:29418
[httpd]
    listenUrl = http://192.168.16.101:8083/
[cache]
    directory = cache
[gerrit@mt101 ~]$
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 2 - Gerrit的用户和群组

Gerrit是基于群组来进行权限控制的，不同的群组具有不同的权限。
每个用户属于一个或者多个群组。

Gerrit系统自带群组

- Anonymous Users：所有用户自动属于该群组，默认只有Read权限
- Change Owner：某个提交的拥有者，具备所属变更的权限
- Project Owners：项目拥有者，具备所属项目的权限
- Registered Users：所有成功登录的用户自动属于该群组，具备投票权限（CodeReview +1-1）

Gerrit预先定义的群组

- Administrators：该群组的成员可以管理所有项目和Gerrit的系统配置
- Non-Interactive Users：该群组的成员可以通过Gerrit界面进行操作，一般用于和第三方系统集成
  ![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214003648705-530433012.png)

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 3 - 进程和服务控制

```
[gerrit@mt101 ~]$ ll gerrit_testsite/bin/gerrit.sh
-rwxr-xr-x 1 root root 16109 Dec 10 14:43 gerrit_testsite/bin/gerrit.sh
[gerrit@mt101 ~]$ 
[gerrit@mt101 ~]$ $GERRIT_SITE/bin/gerrit.sh
Usage: gerrit.sh {start|stop|restart|check|status|run|supervise|threads} [-d site]
[gerrit@mt101 ~]$
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 4 - 查看日志

日志所在目录：`$GERRIT_SITE/logs/`

```
[gerrit@mt101 logs]$ pwd
/home/gerrit/gerrit_testsite/logs
[gerrit@mt101 logs]$ ll
total 28
-rw-r--r-- 1 root root  3052 Dec 10 15:12 error_log
-rw-r--r-- 1 root root     0 Dec 10 14:44 gc_log
-rw-r--r-- 1 root root     5 Dec 10 14:43 gerrit.pid
-rw-r--r-- 1 root root    16 Dec 10 14:44 gerrit.run
-rw-r--r-- 1 root root 13067 Dec 10 15:12 httpd_log
-rw-r--r-- 1 root root     0 Dec 10 14:44 sshd_log
[gerrit@mt101 logs]$
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 5 - war包的命令

war包在命令行下用很多可用命令。

```
[gerrit@mt101 ~]$ ll
total 67556
-rw-r--r--  1 gerrit gerrit 69172528 Dec 10 13:04 gerrit-3.1.0.war
-rwxr-xr-x  1 root   root         91 Dec 10 14:52 gerrit.password
drwxr-xr-x 14 root   root        150 Dec 10 14:44 gerrit_testsite
[gerrit@mt101 ~]$ 
[gerrit@mt101 ~]$ 
[gerrit@mt101 ~]$ sudo java -jar gerrit-3.1.0.war
Gerrit Code Review 
usage: java -jar gerrit-3.1.0.war command [ARG ...]

The most commonly used commands are:
  init            Initialize a Gerrit installation
  reindex         Rebuild the secondary index
  daemon          Run the Gerrit network daemons
  version         Display the build version number
  passwd          Set or change password in secure.config

  ls              List files available for cat
  cat FILE        Display a file from the archive

[gerrit@mt101 ~]$ 
[gerrit@mt101 ~]$ sudo java -jar gerrit-3.1.0.war init -h
init [--batch (-b)] [--delete-caches] [--dev] [--help (-h)] [--install-all-plugins] [--install-plugin VAL] [--list-plugins] [--no-auto-start] [--no-reindex] [--secure-store-lib VAL] [--show-stack-trace] [--site-path (-d) VAL] [--skip-all-downloads] [--skip-download VAL] [--skip-plugins]

 --batch (-b)           : Batch mode; skip interactive prompting (default:
                          false)
 --delete-caches        : Delete all persistent caches without asking (default:
                          false)
 --dev                  : Setup site with default options suitable for
                          developers (default: false)
 --help (-h)            : display this help text (default: true)
 --install-all-plugins  : Install all plugins from war without asking (default:
                          false)
 --install-plugin VAL   : Install given plugin without asking
 --list-plugins         : List available plugins (default: false)
 --no-auto-start        : Don't automatically start daemon after init (default:
                          false)
 --no-reindex           : Don't automatically reindex any entities (default:
                          false)
 --secure-store-lib VAL : Path to jar providing SecureStore implementation class
 --show-stack-trace     : display stack trace on failure (default: false)
 --site-path (-d) VAL   : Local directory containing site data
 --skip-all-downloads   : Don't download libraries (default: false)
 --skip-download VAL    : Don't download given library
 --skip-plugins         : Don't install plugins (default: false)


[gerrit@mt101 ~]$
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 6 - Gerrit解决冲突的提交

如果不同的开发人员同时修改同一个文件并提交，那么这些提交都不会进入代码库。
Gerrit会在页面显示“Conflicts With”或“Cannot Merge”信息来提示有冲突。

处理方式1：

- 简单粗暴地直接取消有冲突的提交，在需要修改时重新提交一个。

处理方式2：

1. 在本地执行git fetch命令更新最新的远端代码
2. 执行git rebase命令获取具体的冲突信息
3. 执行git mergetool命令手动解决冲突
4. 执行git add指令重新添加修改的文件
5. 执行git rebase -continue命令完成rebase过程
6. 重新提交

获取命令的用法帮助信息

```
git fetch -h
git rebase -h
git rmergetool -h
git add -h
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 7 - 为Gerrit项目创建和删除分支

Gerrit和GitLab集成后，在Gerrit上创建分支，GitLab也会自动同步该分支。
但只能是单项同步（Gerrit--》GitLab），也就是说直接在GitLab上创建的分支不会自动同步到Gerrit上。
建议在Gerrit和GitLab集成后，所有的操作都在Gerrit上完成。



## 7.1 查看已有分支

![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214003910382-1530685340.png)



## 7.2 创建新分支

![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214003921865-459220562.png)
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214003937594-357922155.png)



## 7.3 删除分支

点击要删除分支一行的DETELE按钮，根据提示操作即可。
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214003944974-1095172632.png)

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 8 - 为Gerrit项目添加默认代码审核人

一般情况下，每次提交时都需要手工添加Code Reviewer。
通过reviewers插件，可以为指定项目或分支设置默认的Code Reviewer，在有代码提交时，Code Reviewer会接收到代码审核通知邮件。



## 8.1 找到reviewers插件

在GerritForge（https://gerrit-ci.gerritforge.com/），找到对应gerrit 版本的reviewers插件
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004004542-31761230.png)

reviewers插件：
https://gerrit-ci.gerritforge.com/job/plugin-reviewers-bazel-master/
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004013005-406691094.png)

获得对应的jar下载地址
https://gerrit-ci.gerritforge.com/job/plugin-reviewers-bazel-master/lastSuccessfulBuild/artifact/bazel-bin/plugins/reviewers/reviewers.jar



## 8.2 放置插件并重启Gerrit服务

将下载的插件（jar包）放置在`$GERRIT_SITE/plugins`目录下，然后重启Gerrit服务（`$GERRIT_SITE/bin/gerrit.sh restart`），会自动加载此目录下的插件。

```
[gerrit@mt101 ~]$ cd gerrit_testsite/plugins/
[gerrit@mt101 plugins]$ pwd
/home/gerrit/gerrit_testsite/plugins
[gerrit@mt101 plugins]$ wget https://gerrit-ci.gerritforge.com/job/plugin-reviewers-bazel-master/lastSuccessfulBuild/artifact/bazel-bin/plugins/reviewers/reviewers.jar
--2019-12-11 11:55:16--  https://gerrit-ci.gerritforge.com/job/plugin-reviewers-bazel-master/lastSuccessfulBuild/artifact/bazel-bin/plugins/reviewers/reviewers.jar
Resolving gerrit-ci.gerritforge.com (gerrit-ci.gerritforge.com)... 8.26.94.23
Connecting to gerrit-ci.gerritforge.com (gerrit-ci.gerritforge.com)|8.26.94.23|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41911 (41K) [application/java-archive]
Saving to: ‘reviewers.jar’

100%[==========================================>] 41,911       118KB/s   in 0.3s   

2019-12-11 11:55:23 (118 KB/s) - ‘reviewers.jar’ saved [41911/41911]

[gerrit@mt101 plugins]$ 
[gerrit@mt101 plugins]$ chmod 755 reviewers.jar 
[gerrit@mt101 plugins]$ ll
total 44
-rwxr-xr-x 1 gerrit gerrit 41911 Nov 16 02:03 reviewers.jar
[gerrit@mt101 plugins]$ 
[gerrit@mt101 plugins]$ cd
[gerrit@mt101 ~]$ sudo sh gerrit_testsite/bin/gerrit.sh restart
Stopping Gerrit Code Review: OK
Starting Gerrit Code Review: OK
[gerrit@mt101 ~]$
```



## 8.3 查看插件是否安装成功

![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004104575-671917888.png)



## 8.4 配置Reviewers

![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004121357-788873638.png)
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004134410-1734311529.png)

Filter部分："*"表示所有分支改动
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004143990-1792660256.png)

Reviewer部分：自动提示支持的用户名、邮箱名、群组名
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004153226-1321573191.png)
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004158616-582095894.png)

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 9 - 手动同步GitLab代码到Gerrit

Gerrit和GitLab集成后，在Gerrit上创建分支，GitLab也会自动同步该分支。
但只能是单项同步（Gerrit--》GitLab），也就是说直接在GitLab上创建的分支不会自动同步到Gerrit上。
建议在Gerrit和GitLab集成后，所有的操作都在Gerrit上完成。

如果不小心在GitLab端进行了代码的更新操作，就需要手工执行同步代码的命令。

```
cd /home/gerrit/gerrit_testsite/git/${project}
git fetch origin +refs/heads/*:refs/heads/* +refs/heads/*:refs/heads/* --prune
```

[回到顶部](https://www.cnblogs.com/anliven/p/12019989.html#_labelTop)

# 10 - 删除Gerrit上的项目

为防止误操作，在Gerrit界面无法直接删除项目。
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004229805-1793011763.png)

可以在后台将项目目录删除，然后刷新缓存，才可以完全移除。

```
[gerrit@mt101 git]$ pwd
/home/gerrit/gerrit_testsite/git
[gerrit@mt101 git]$ ll
total 0
drwxr-xr-x 7 gerrit gerrit 119 Dec 10 14:43 All-Projects.git
drwxr-xr-x 7 gerrit gerrit 119 Dec 11 12:26 All-Users.git
drwxr-xr-x 7 gerrit gerrit 138 Dec 11 12:45 testrepo.git
[gerrit@mt101 git]$ 
[gerrit@mt101 git]$ rm -rf testrepo.git/
[gerrit@mt101 git]$ 
[gerrit@mt101 git]$ ssh -p 29418 admin@192.168.16.101 gerrit flush-caches --all
[gerrit@mt101 git]$
```

刷新页面
![img](https://img2018.cnblogs.com/blog/819128/201912/819128-20191214004300769-375386630.png)


**Action is the antidote to despair!**