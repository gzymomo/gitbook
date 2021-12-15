[TOC]

Jenkins部分内容原文地址：
CSDN：heiPony：[centos7 安装 jenkins](https://blog.csdn.net/qq_36493822/article/details/106805482)

# 一、JDK安装

 1. 新建一个目录：`mkdir /usr/jdk`
 2. 上传tar包到该目录：`rz + 包；`
 3. 解压tar包：`tar -zxvf jdk-8-linux-x64.tar.gz`
 4. 修改环境变量：`vi /etc/profile`

新服务器一般没有 JDK ，可以使用 java -version 命令查看。如果没有，则通过 yum 命令安装之，如果有但版本不对也可以先卸载再安装

```bash
卸载
rpm -qa | grep java | xargs rpm -e --nodeps
安装 1.8
yum install java-1.8.0-openjdk* -y
```

修改内容如下：
```shell
export JAVA_HOME=/usr/jdk/jdk1.8.0_171
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

然后重置环境：

`source /etc/profile`


4. 查看环境变量内容：`echo $PATH`
5. 检测是否安装成功：`java -version`

# 二、Git安装
```shell
sudo yum install -y git
```



# 三、Docker安装使用：

 1. 安装依赖：`yum install -y yum-utils device-mapper-persistent-data lvm2`
 2. 添加软件源：`yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`　　# 指定阿里云镜像源
 3. 安装docker-ce：`yum clean all`
 `yum makecache fast` # 重新生成缓存
`yum -y install docker-ce docker-ce-cli containerd.io`
 4. 设置自启并启动：`systemctl enable docker`
`systemctl start docker`
 5. 查看版本：`docker version`



# 四、maven安装及配置

1. 下载安装包，传至服务器后，解压。
2. 配置环境变量：

```bash
vi /etc/profile

export MAVEN_HOME=/usr/local/apache-maven-3.6.3
export PATH=$MAVEN_HOME/bin:$PATH
```

3. 刷新环境变量
4. 检查maven版本

```bash
mvn -v
```




# 五、Jenkins的安装
## 4.1 安装方法一
https://pkg.jenkins.io/redhat-stable/
根据需要选择要安装的jenkins版本，
```bash
wget https://pkg.jenkins.io/redhat-stable/jenkins-2.204.1-1.1.noarch.rpm

sudo rpm -ih jenkins-2.204.1-1.1.noarch.rpm
```

## 4.2 安装方法二

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

配置 jenkins端口及权限：
```yml
# vim /etc/sysconfig/jenkins
// 监听端口
JENKINS_PORT="8080"

//修改配置,这里是为了防止权限不足
# JENKINS_USER="root"
```

修改jenkins权限：
```bash
# chown -R root:root /var/lib/jenkins
# chown -R root:root /var/cache/jenkins
# chown -R root:root /var/log/jenkins
```

链接jdk：
```bash
ln -s /usr/local/java/jdk1.8.0_171/bin/java /usr/bin/java
```
或者配置 jdk路径：
```bash
vi /etc/init.d/jenkins
```
![](https://img-blog.csdnimg.cn/20200617135407554.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NDkzODIy,size_16,color_FFFFFF,t_70#pic_center)

Jenkins的工作路径：
`/var/lib/jenkins/`

启动 、重启、停止
```bash
service jenkins start
service jenkins restart
service jenkins stop
```

登陆后配置：系统管理、全局工具配置
JDK配置：
![](https://img-blog.csdnimg.cn/2020061917384323.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NDkzODIy,size_16,color_FFFFFF,t_70)

Git配置：
![](https://img-blog.csdnimg.cn/20200619173853259.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NDkzODIy,size_16,color_FFFFFF,t_70)

Maven配置：
![](https://img-blog.csdnimg.cn/20200619173902534.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NDkzODIy,size_16,color_FFFFFF,t_70)

Jenkins项目打包发布脚本：
```bash
#!/bin/bash

//传入的war包名称
name=$1
//war包所在目录
path=$2
//上传的war包位置
path_w=$3
//如果项目正在运行就杀死进程
if [ -f "$path/$name" ];then
	echo "delete the file $name"
	rm -f  $path/$name
else
	echo "the file $name is not exist"
fi
//把jenkins上传的war包拷贝到我们所在目录
cp $path_w/$name $path/
echo "copy the file $name from $path_w to $path"

//获取该项目正在运行的pid
pid=$(ps -ef | grep java | grep $name | awk '{print $2}')
echo "pid = $pid"
//如果项目正在运行就杀死进程
if [ $pid ];then
	kill -9 $pid
	echo "kill the process $name pid = $pid"
else
	echo "process is not exist"
fi
//要切换到项目目录下才能在项目目录下生成日志
cd $path
//防止被jenkins杀掉进程 BUILD_ID=dontKillMe
BUILD_ID=dontKillMe
//启动项目
nohup java -server -Xms256m -Xmx512m -jar -Dserver.port=20000 $name >> nohup.out 2>&1 &
//判断项目是否启动成功
pid_new=$(ps -ef | grep java | grep $name | awk '{print $2}')
if [ $? -eq 0 ];then
echo "this application  $name is starting  pid_new = $pid_new"
else
echo "this application  $name  startup failure"
fi
echo $! > /var/run/myClass.pid
echo "over"
```

# 六、Linux、Centos7服务运维操作
## 6.1 重启服务：
```shell
systemctl daemon-reload
systemctl restart jenkins.service
```

## 6.2 uname -r 命令查看你当前的内核版本

```shell
wget http://mirrors.aliyun.com/repo/Centos-7.repo
yum-config-manager --add-repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
yum clean all
yum install -y net-tools
yum install -y lrzsz
yum install -y gcc
```

## 6.3 查看网关、端口开放情况

```bash
ip  r  查看网关
lsof -i : 22查看22端口使用情况
ss -tnlp 查看端口开启情况
查看端口使用情况！
netstat -ntlp | grep 80
```

## 6.4 修改镜像源
```shell
修改默认yum源为国内的阿里云yum源。官方的yum源在国内访问效果不佳，需要改为国内比较好的阿里云或者网易的yum源：
	#下载wget
	yum install -y wget
	#备份当前的yum源
	mv /etc/yum.repos.d /etc/yum.repos.d.backup
	#新建空的yum源设置目录
	mkdir /etc/yum.repos.d
	#下载阿里云的yum源配置
	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
重建缓存：
	yum clean all
	yum makecache
```

## 6.5 检查服务器时间
一般新服务器时间都会与网络时间不一致，这时就需要我们先同步一下服务器时间
date/timedatectl 命令可用于查看系统当前的时间，如果和网络时间不一致
```bash
# 安装日期工具
yum -y install ntp ntpdate
# 同步时间
ntpdate cn.pool.ntp.org
# 将系统时间写入硬件时间
hwclock --systohc
```