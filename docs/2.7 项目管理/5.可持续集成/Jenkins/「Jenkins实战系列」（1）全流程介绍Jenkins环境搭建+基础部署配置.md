# 「Jenkins实战系列」（1）全流程介绍Jenkins环境搭建+基础部署配置

原文地址：https://blog.51cto.com/alex4dream/2994604

# 背景

- **在实际开发中，我们经常要一边开发一边测试，当然这里说的测试并不是程序员对自己代码的单元测试，而是同组程序员将代码提交后，由测试人员测试；**
- **前后端分离后，经常会修改接口，然后重新部署；这些情况都会涉及到频繁的打包部署；**

## 手动打包常规步骤：

1. git commit + git push 提交代码
2. 问一下同组小伙伴有没有要提交的代码
3. 拉取代码并打包（war包，或者jar包）
4. 上传到Linux服务器
5. 查看当前程序是否在运行
6. 关闭当前程序
7. 启动新的jar包
8. 观察日志看是否启动成功
9. 如果有同事说，自己还有代码没有提交…再次重复1到8的步骤。

> **基于以上的痛点，有一种工具能够实现，将代码提交到git后就自动打包部署勒，答案是肯定的：现在这里主要介绍jenkins**

> **当然除了Jenkins以外，也还有其他的工具可以实现自动化部署，如Hudson、gitlab CI/CD等。只是Jenkins相对来说，使用得更广泛。**

## 文章结构：

![img](https://oscimg.oschina.net/oscnet/up-ae2953aff8647e12d0515f01e43895d6003.png)

### Jenkins服务器搭建及基本配置

#### 简介

> **Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。**

### Jenkins自动化部署实现原理

![img](https://oscimg.oschina.net/oscnet/up-a4ccb52164850189d0b027c4913767a1369.png)

### Jenkins部署环境

#### 基本环境

1. **jdk环境，Jenkins是java语言开发的，因需要jdk环境**。
2. **git/svn客户端，因一般代码是放在git/svn服务器上的，我们需要拉取代码**。
3. **maven客户端，因一般java程序是由maven工程，需要maven打包，当然也有其他打包方式，如：gradle**

> **以上是自动化部署java程序jenkins需要的基本环境，请自己提前安装好，下面着重讲解Jenkins的安装部署配置**。

### Jenkins安装

1. 下载安装包jenkins.war；
2. 在安装包根路径下，运行命令 java -jar jenkins.war --httpPort=8080，（linux环境、Windows环境都一样）；
3. 打开浏览器进入链接 http://localhost:8080.
4. 填写初始密码，激活系统

![img](https://oscimg.oschina.net/oscnet/up-34abdcd952f2083df9aaef66f4c17371d60.png)

1. 进入插件安装选择

这里建议选择，推荐安装的插件，保证基本常用的功能可以使用。

中文版
 ![img](https://oscimg.oschina.net/oscnet/up-77f6a6879e038f36f432d74b097ebb468bc.png)

英文版
 ![img](https://oscimg.oschina.net/oscnet/up-2d9bebe227b39571e0be96834f40d237505.png)

------

1. 选择后，进入插件安装页面
    ![img](https://oscimg.oschina.net/oscnet/up-771b0c960886dbe0f28c0a3a4294902bd55.png)
2. 设置初始用户和密码
    ![img](https://oscimg.oschina.net/oscnet/up-9488cd3636b519c6dd01a276665c098db79.png)
3. 进入系统，安装完成
    ![img](https://oscimg.oschina.net/oscnet/up-05c97f372407d110b562f01d7d9b70f1de9.png)

### Jenkins基本配置

#### 系统初始化配置

![img](https://oscimg.oschina.net/oscnet/up-3777d3b44d1435216e65144f696c7697875.png)

1. Configure System (系统设置)

- 在系统设置这里，我们只需要设置最后面的一项，配置远程服务器地址，
- 即我们代码最终运行的服务器地址信息，就像我们之前手动部署时使用xshell登录Linux服务器一样，
- 当然这里是可以配置多台远程Linux服务器的，配置完成后点击保存即可，为后面我们配置自动化部署做准备，配置如下图

![img](https://oscimg.oschina.net/oscnet/up-71a064211879d435912ae99853a23ac209d.png)

1. Configure  Global Security (全局安全配置)

配置用户相关的权限

![img](https://oscimg.oschina.net/oscnet/up-1261732eeedb2a680320fef7a7e994550a2.png)

配置钩子程序（当用代码更新时通知）访问权限，避免报403错误

默认是勾选上了的，这里去掉勾选
 ![img](https://oscimg.oschina.net/oscnet/up-678652c8ca78c2575fa6653ad2c58b597a4.png)

1. **Global Tool Configuration (全局工具配置 )**

配置maven的全局settings路径

![img](https://oscimg.oschina.net/oscnet/up-59dd199a5b87c02d182b3d881881bf479a8.png)

配置jdk

![img](https://oscimg.oschina.net/oscnet/up-9d359c392f72232382cf4569ed5a4ceff26.png)

配置git

![img](https://oscimg.oschina.net/oscnet/up-e14eb59e6dcfabcdcde0311b19005fd51b7.png)

配置maven的安装路径

![img](https://oscimg.oschina.net/oscnet/up-36ac25348d3371508834b093ce376213393.png)

配置必要插件

主要是检查如下这两个插件是否已安装

- 插件1：Publish over SSH
- 插件2：Deploy to container Plugin

![img](https://oscimg.oschina.net/oscnet/up-a893d6d4f8343ee98130ad4304559a44b00.png)

### Jenkins自动化部署

> **我们配置一个自动化部署的的java程序（springBoot+maven+gitHub），基本必要配置就差不多了，后面配置过程中如果需要在配置**。

#### Jenkins服务器上创建项目和配置

> **大体步骤：General(基础配置)–》源码管理–》构建触发器–》构建环境–》构建–》构建后操作**

1. 创建一个工程

![img](https://oscimg.oschina.net/oscnet/up-2670196f62cbc072156f629dc6a05cde071.png)

1. General(基础配置)

**仅需填写标准部分，其他可不填写**

![img](https://oscimg.oschina.net/oscnet/up-d9bbc177029f83e1d30ee131dc5e65a0195.png)

1. 源码管理

![img](https://oscimg.oschina.net/oscnet/up-198b10cc1751365886873f1be05ee2e9135.png)

上图中点击“添加”按钮添加一组账号和密码

![img](https://oscimg.oschina.net/oscnet/up-3c85c4c23368c56816063ff8cb27e7df2f6.png)

1. 构建触发器

![img](https://oscimg.oschina.net/oscnet/up-62ec5e0781bff0679a658ac86f8183c54fd.png)

如上图：当前项目的回调地址为：

> **http://localhost:8080/job/jenkinsSpringBootDemo/build?token=token_demo2**

> 只要执行这个地址（在浏览器上访问改地址），该项目就会发起一次构建项目，即拉取代码打包部署操作，在实际中，是由git服务器回调该地址。

1. 构建环境（无需配置）
2. 构建
    ![img](https://oscimg.oschina.net/oscnet/up-b64da088a9baffbe6f62bd3a3f02e29a9a8.png)
3. 构建后操作
   - 构建后操作的意思是，jar打包好后，要将jar发送到哪里去，发送后去和启动等。
   - 这里需要提前在需要部署的服务器上配置好路径，写好启动和停止项目的脚本，并设置为可以执行的脚本，
   - 其实就是我们平时在Linux上手动部署项目操作的脚本。

![img](https://oscimg.oschina.net/oscnet/up-26dc60237e198a4042ad2a44a1ca66912c0.png)

案例中给出的start.sh脚本如下：

```bash
#!/bin/bash
export JAVA_HOME=/usr/java/jdk1.8.0_131
echo ${JAVA_HOME}
echo 'Start the program : demo2-0.0.1-SNAPSHOT.jar'
chmod 777 /home/ldp/app/demo2-0.0.1-SNAPSHOT.jar
echo '-------Starting-------'
cd /home/ldp/app/
nohup ${JAVA_HOME}/bin/java -jar demo2-0.0.1-SNAPSHOT.jar &
echo 'start success'
```

案例中给出的stop.sh脚本如下：

```bash
#!/bin/bash
echo "Stop Procedure : demo2-0.0.1-SNAPSHOT.jar"
pid=`ps -ef | grep java | grep demo2-0.0.1-SNAPSHOT.jar | awk '{print $2}'`
echo 'old Procedure pid:'$pid
if [ -n "$pid" ]
then
kill -9 $pid
fi
```

### Linux服务器配置

在Linux 服务上，上传上文中的两个脚本，用于启动和停止

![img](https://oscimg.oschina.net/oscnet/up-89e13ab6c147528bd1cf37bd6592c59b5a4.png)

### GitHub服务器配置

在GitHub服务器上的指定项目里面配置上文中提到的回调地址

特别注意：为了保证回调地址网可以使用，

所以，下面配置的是映射地址。

![img](https://oscimg.oschina.net/oscnet/up-9cd9842424ba475a0f854e8f937bfaaed39.png)

可以参考：[官方文档](https://jenkins.io/zh/doc/)