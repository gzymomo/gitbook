- [Jenkins+GitLab+Docker自动化部署](https://lengxiaobing.github.io/2019/09/19/Jenkins+GitLab+Docker%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2/)

# Jenkins+GitLab+Docker自动化部署

> 简陋的Jenkins自动化部署

## 环境配置

### 服务器列表

| 服务器           | 操作系统      | 安装软件                                    |
| ---------------- | ------------- | ------------------------------------------- |
| 代码仓库         | CentOS_7_5_64 | docker，gitlab                              |
| 镜像仓库/Jenkins | CentOS_7_5_64 | docker，Harbor，Jenkins，jdk1.8，maven，git |
| 服务部署         | CentOS_7_5_64 | docker，jdk1.8                              |

### 服务器环境配置

**关闭防火墙**

```
systemctl stop firewalld & systemctl disable firewalld  
```

**关闭SeLinux**

```
setenforce 0 
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config 
```

**关闭Swap**

- 执行**swapoff -a**可临时关闭，但系统重启后恢复
- 编辑**/etc/fstab**，注释掉包含**swap**的那一行即可，重启后可永久关闭，如下所示：

```
sed -i '/ swap / s/^/#/' /etc/fstab 
```

## Docker

安装步骤参见：[安装docker](https://lengxiaobing.github.io/2019/01/02/kubeadm部署kubernetes单主集群/#二安装docker)

## Harbor

安装步骤参见：[安装Harbor](https://lengxiaobing.github.io/2019/01/02/kubeadm部署kubernetes单主集群/#32企业级私有仓库)

## Jenkins

- 安装jdk，maven

> 下载安装包，上传到服务器，解压，配置环境变量

```
# 修改配置文件
vim /etc/profile

export JAVA_HOME=/opt/jdk1.8.0_172
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

export MAVEN_HOME=/opt/maven/bin
export PATH=$MAVEN_HOME:$PATH

# 立即生效
source /etc/profile
```

- 安装git

```
yum install git 
```

- 安装Jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# 安装
yum install jenkins
# 启动
systemctl start jenkins
```

- 配置Jenkins

**全局工具配置**

[![img](https://lengxiaobing.github.io/img/docs-pics/20190919151047.png)](https://lengxiaobing.github.io/img/docs-pics/20190919151047.png)

**全局安全配置**

[![img](https://lengxiaobing.github.io/img/docs-pics/20190919151441.png)](https://lengxiaobing.github.io/img/docs-pics/20190919151441.png)

- 其他说明    
  - jenkins 工作目录：/var/lib/jenkins/workspace

## Gitlab

### 安装

安装步骤参见：[使用docker搭建GitLab环境](https://lengxiaobing.github.io/2019/04/22/使用docker搭建GitLab环境/)

### 配置

**开启本地webhook**

[![img](https://lengxiaobing.github.io/img/docs-pics/1568874079918.png)](https://lengxiaobing.github.io/img/docs-pics/1568874079918.png)

## 自动化部署

- 创建项目

在gitlab上创建一个项目，并编写相关的`Dockerfile`和`Jenkinsfile`文件。

- Jenkins配置流水线

![img](https://lengxiaobing.github.io/img/docs-pics/20190919152237.png)

- 构建触发器

> JENKINS_URL：表示Jenkins的ip和端口号；TOKEN_NAME：表示身份验证令牌

[![img](https://lengxiaobing.github.io/img/docs-pics/1568877886466.png)](https://lengxiaobing.github.io/img/docs-pics/1568877886466.png)

- 流水线配置

[![img](https://lengxiaobing.github.io/img/docs-pics/20190919152957.png)](https://lengxiaobing.github.io/img/docs-pics/20190919152957.png)

- gitlab项目webhook配置

> 在项目设置的集成配置里面，设置webhook的url。

[![img](https://lengxiaobing.github.io/img/docs-pics/1568878326326.png)](https://lengxiaobing.github.io/img/docs-pics/1568878326326.png)

[![img](https://lengxiaobing.github.io/img/docs-pics/20190919153552.png)](https://lengxiaobing.github.io/img/docs-pics/20190919153552.png)

## Jenkinsfile

> 本示例所用的完整Jenkinsfile文件如下所示

```
#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        IMAGE_STORE="store.iaas.biz:80"
        IMAGE_STORE_PATH="store.iaas.biz:80/dev"
        DOCKER_USER="admin"
        DOCKER_PASSWORD="Harbor12345"
        MODULE="eureka-test"
        VERSION="latest"
        DEPLOY_IP="deploy.iaas.biz"
    }

    stages {
        stage('编译') {
            steps {
                echo "==========编译开始=========="
                sh "mvn clean package -Dmaven.test.skip=true"
                echo "==========编译结束=========="
            }
        }
        stage('构建镜像') {
            steps {
                echo "==========构建镜像开始=========="
                sh "sudo docker build -t ${IMAGE_STORE_PATH}/${MODULE}:${VERSION} ."
                echo "==========构建镜像结束=========="
            }
        }
        stage('推送镜像') {
            steps {
                echo "==========推送镜像开始=========="
                sh "sudo docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${IMAGE_STORE}"
                sh "sudo docker push ${IMAGE_STORE_PATH}/${MODULE}:${VERSION}"
                echo "==========推送镜像结束=========="
            }
        }
        stage('部署服务') {
            steps {
                echo "==========部署服务开始=========="
                echo "----------拉取镜像----------"
                sh "sudo ssh ${DEPLOY_IP} 'sudo docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD} ${IMAGE_STORE}'"
                sh "sudo ssh ${DEPLOY_IP} 'sudo docker pull ${IMAGE_STORE_PATH}/${MODULE}:${VERSION}'"

                echo "----------删除老的容器----------"
                sh "sudo docker ps -f name=${MODULE} -q | xargs --no-run-if-empty docker container stop"
                sh "sudo docker container ls -a -f name=${MODULE} -q | xargs -r docker container rm"

                echo "----------启动新的容器----------"
                sh "sudo ssh ${DEPLOY_IP} 'sudo docker run --name ${MODULE} -d -p 10000:10000 ${IMAGE_STORE_PATH}/${MODULE}:${VERSION}'"
                echo "==========部署服务结束=========="
            }
        }
    }
}
```

## 遇到的坑

### maven命令无法使用

在Jenkins的使用过程中，如果在脚本中使用到maven命令，有可能出现如下所示的错误：

```
script.sh: line 1: mvn: command not found 
```

对于java或maven的路径的环境变量是放在/etc/profile中的，而/etc/profile只有在用户登录的时候才会被load，Jenkins在运行命令时，使用的是Non-login的方式，而这种方式在运行命令时，/etc/profile是不会被load进来的，所以jenkins只能在当前路径下寻找可执行文件.

**解决方式**:

在Jenkins中设置全局属性中的环境变量，jenkins主页面->Manage Jenkins->Configure  System->Global Properties 中，将Environment variables复选框选中，会出来List of  variables， 填入以下内容:

- name: JAVA_HOME        value:/opt/jdk1.8.0_172
- name: M2_HOME           value:/usr/cyz/apache-maven-3.6.1
- name: `PATH+EXTRA`   value: $M2_HOME/bin

注意最后标色的 `PATH+EXTRA`, 这表示PATH=EXTRA:$PATH, 即扩展当前的PATH变量.

### docker调用失败

在Jenkins的使用过程中，如果在脚本中使用到docker命令，有可能出现如下所示的错误：

```
dial unix /var/run/docker.sock: connect: permission denied 
```

Docker守护程序绑定到Unix socket而不是TCP端口。默认情况下，Unix socket由`root`用户拥有，其他用户需要使`sudo`。Docker守护程序始终以`root`用户身份运行。

**解决方式**:

在命令前加`sudo`。

### sudo命令无法使用

在Jenkins的使用过程中，如果在脚本中使用到sudo命令，有可能出现如下所示的错误：

```
sudo: no tty present and no askpass program specified 
```

这是因为Jenkins服务器在执行sudo命令时的上下文有误，导致这个命令执行的异常。

**解决方式**:

在Jenkins宿主服务器上运行如下命令

```
sudo visudo 
```

在文件的末尾加上一行

```
jenkins ALL=(ALL) NOPASSWD: ALL 
```

保存文件（注意保存的时候修改文件名，文件名后缀不要加上默认的.tmp，即可覆盖原文件）

```
Ctrl+O 
```

退出编辑

```
Ctrl+X 
```

重启Jenkins服务

```
systemctl restart jenkins 
```

最后，重新执行构建任务，不会出现先前的错误。

