- [Jenkins-slave分布式跨网络发布](https://www.cnblogs.com/xiao987334176/p/13152229.html)



# 一、Jenkins主从配置概述

Jenkins的Master-Slave分布式架构主要是为了解决Jenkins单点构建任务多、负载较高、性能不足的场景。Master-Slave相当于Server和Agent的概念。

Master提供web接口让用户来管理job和Slave，job可以运行在Master本机或者被分配到Slave上运行构建。

一个Master（Jenkins服务所在机器）可以关联多个Slave用来为不同的job或相同的job的不同配置来服务。

## 环境说明

| 系统版本   | 主机名     | ip地址         | 说明          |
| ---------- | ---------- | -------------- | ------------- |
| centos 7.6 | jenkins    | 10.212.82.86   | jenkins服务器 |
| centos 7.6 | office-145 | 192.168.31.145 | 办公室测试    |

说明：

jenkins-->office-145 网络是不通的。

office-145-->jenkins 网络是通的。

 

现在要求jenkins能一键发布到office-145，那么很明显一个问题。网络是不通的，怎么实现。

后来我研究发现，通过Jenkins-slave就能实现跨网络发布。



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715174907250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h0OTk5OWk=,size_16,color_FFFFFF,t_70)

## 配置前准备

jenkins服务器已经安装好了，请参考链接：

https://www.cnblogs.com/xiao987334176/p/13032339.html

 

office-145需要安装一下jdk

解压jdk

```
mkdir /data
tar zxvf jdk-8u211-linux-x64.tar.gz -C /data/
```

 

添加环境变量

```
vim /etc/profile
```

 

最后一行添加

```
set java environment
JAVA_HOME=/data/jdk1.8.0_211/
JRE_HOME=/data/jdk1.8.0_211/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

 

重新加载环境变量

```
source /etc/profile
```

 

查看java版本

```
java -version
```

# 二、新建节点

## 安全配置

登录Jenkins服务器-->Manage Jenkins-->Configure Global Security

找到代理，勾选随机端口

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617155340638-1612075159.png)

 

 

## 新建节点

登录Jenkins服务器-->Manage Jenkins-->Manage Nodes and Clouds

点击新建节点

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617150720715-733587227.png)

 

 

输入节点名称，第一次配置只能选这个选项，表示所有配置重新填写。

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617150810038-1834530310.png)

 

填写相关信息

 ![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617153358104-1852261106.png)

 

说明：

名称：节点名称，上一步新建时的名字

描述：节点描述，主要说明这个节点机器主要用来做什么工作，可随意填写。

并发构建数：此机器可同时执行任务的数量

远程工作目录：这个目录就填写Jenkins服务器的安装目录即可，其实也可以指定其他目录

标签：标记节点机器的一个标记，后面会用到这个名字，可随意填写。

用法：此项根据根据自己的需求选择即可。

​      Only build jobs with label expressions matching this node 表示仅生成标签表达式与此节点匹配的作业。注意：我这里是执行特定的任务，不是执行所有任务。

​      如果需要执行所有Jenkins任务，选择：Use this node as much as possible

启动方式：此项是说明节点链接Jenkins时的方式，不同版本略有不同。这个启动方式大体意思是通过代理连接服务器，但是后期你会发现和java web启动是一样的（具体有啥区别就不清楚了，也许就是java web启动吧）

 

点击保存

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617153702062-1078627988.png)

 

点击节点

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617153800312-343255495.png)

 

跳转页面，下载2个文件，分别是slave-agent.jnlp和agent.jar 

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617153854713-1498960872.png)

上面的命令任选其一，我这里选择第一个。

 

登录主机office-145，创建目录

```
mkdir -p /data/jenkins
mkdir -p /data/jenkins-slave
```

 

将下载好的2个文件，上传到/data/jenkins-slave目录。

启动代理

```
cd /data/jenkins-slave/
java -jar agent.jar -jnlpUrl http://10.212.82.86:8080/computer/office-145/slave-agent.jnlp -secret f6bfb3d0a0e58f71704aebb4af9417bdf3a85b105ab587b808f7339e6aaf7d15 -workDir "/data/jenkins"
```

 

输出

```java
六月 17, 2020 4:00:03 下午 org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
信息: Using /data/jenkins/remoting as a remoting work directory
六月 17, 2020 4:00:03 下午 org.jenkinsci.remoting.engine.WorkDirManager setupLogging
信息: Both error and output logs will be printed to /data/jenkins/remoting
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main createEngine
信息: Setting up agent: office-145
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener <init>
信息: Jenkins agent is running in headless mode.
六月 17, 2020 4:00:04 下午 hudson.remoting.Engine startEngine
信息: Using Remoting version: 4.2.1
六月 17, 2020 4:00:04 下午 org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
信息: Using /data/jenkins/remoting as a remoting work directory
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Locating server among [http://10.212.82.86:8080/]
六月 17, 2020 4:00:04 下午 org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
信息: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Agent discovery successful
  Agent address: 10.212.82.86
  Agent port:    20251
  Identity:      1d:f0:4e:5f:34:5f:87:63:60:42:13:e5:38:b1:1b:f0
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Handshaking
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Connecting to 10.212.82.86:20251
六月 17, 2020 4:00:04 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Trying protocol: JNLP4-connect
六月 17, 2020 4:00:05 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Remote identity confirmed: 1d:f0:4e:5f:34:5f:87:63:60:42:13:e5:38:b1:1b:f0
六月 17, 2020 4:00:08 下午 hudson.remoting.jnlp.Main$CuiListener status
信息: Connected
```

查看状态。提示已经同步了

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617160129074-1058712828.png)

#  三、构建配置

## 自由风格

接下来配置一下Job，测试一下项目在节点主机上是否能够成功构建并执行

新建项目，选择自由风格。

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617164155619-1916520889.png)

 

 

 

配置general

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617164305300-486477096.png)

 

 

 标签表达式输入的是之前配置的节点标签名。

 

添加构建步骤

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617164609284-1827901283.png)

 

 这里的命令是查看主机名

 

保存之后，执行一下构建。

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617164739712-1322313513.png)

 

 

查看控制台输出

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617164825135-1857478934.png)

 

 发现输出的主机名是正确的。

 

## 流水线

 ![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617165004300-728771937.png)

 

 

配置greneral，**注意：这里是不能选择slave节点的。**

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617165051407-842992344.png)

 

 

流水线

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617172918901-782224434.png)

 

 完整代码如下：

```
pipeline{
    agent{ label '192.168.31.145'}
    stages{
        stage('test cmd'){
            steps{
                sh 'hostname'
            }
        }
    }
}
```

这里指定了slave节点为192.168.31.145

 

保存配置之后，点击构建

 ![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617165249940-1283987785.png)

 

 

查看输出结果

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200617173028566-437461202.png)

 

 可以看到输出结果正确

 

本文参考链接：

https://www.cnblogs.com/linuxchao/p/linunx-Jenkins-slave.html

https://blog.csdn.net/tellmewhyto/article/details/81546477