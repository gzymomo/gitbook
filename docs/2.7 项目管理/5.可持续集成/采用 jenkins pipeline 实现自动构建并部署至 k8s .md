微信公众号：民工哥技术之路：采用 jenkins pipeline 实现自动构建并部署至 k8s

[Jenkins API+Pipeline深度实践之input的自动化](https://mp.weixin.qq.com/s?__biz=MzkxNzAzNDA3Ng==&mid=2247484622&idx=1&sn=ef9bdc890fafdd3a079a666d0cbaf96f&chksm=c14781fdf63008eb48301a2d5e85d03b4040bcdcaf5baf99353f35c8d318eebf54c202e65a21&mpshare=1&scene=24&srcid=0324cquPD0cnS1I3OPJw6ohm&sharer_sharetime=1616542819519&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)

## 前言

在日常开发中，经常会有发布的需求，而且经常会碰到各种环境，比如：开发环境、测试环境、生产环境。虽然可以使用手动构建、上传服务器部署的方式，但在微服务架构下一个项目经常包含多个微服务的部署，如果用手动方式就会非常繁琐而且容易出错。使用jenkins结合SCM可以实现代码的整个自动化构建部署过程。

本文中自动构建部署过程大致完成了以下步骤：

- 提交spring boot项目代码并打上git tag，上传代码及tag至gitlab
- gitlab通过webhook自动触发jenkins执行任务
- jenkins获取代码，执行代码编译、构建docker镜像、上传docker镜像至harbor镜像仓库、执行kubectl命令部署至k8s。

本文中采用jenkins pipeline执行整个jenkins的构建过程，在pipeline中使用docker执行maven构建。文中构建的docker镜像的tag直接采用git中的tag。

下面的示例中，jenkins版本为2.121.3，gitlab版本为10.0.2-ce，如果使用的版本不一致可能会有部分设置差异。

## 部署Jenkins

这里采用docker的方式部署jenkins。

1. 在终端中运行docker命令，使用jenkinsci/blueocean镜像运行容器。

```
docker run \
  -d \
  -u root \
  -p 8080:8080 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```

1. 访问 http://localhost:8080 地址，等待出现下面解锁界面。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0LGHibaeybxR106Dic103ZeqibzQd6Ficd6WcTjd8eTTK3BmDtQm81vNJkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. 使用docker logs 命令从日志信息中 复制自动生成的密码(在两组星号之间)。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

1. 在 解锁Jenkins 页面, 粘贴密码并继续。
2. 解锁jenkins后，在界面中选择“安装建议的插件”。
3. 最后，jenkins要求创建管理员用户。创建新用户或使用admin用户，按照步骤完成后即可登录使用jenkis了。

注：由于后面jenkins  pipeline任务会上传docker镜像至harbor仓库，如果harbor使用http需要在/etc/docker/daemon.json中将地址配置到insecure-registries；如果harbor使用自签名https证书，需要将ca证书放入/etc/docker/certs.d/目录下。设置完需要重启docker服务。

## 准备java示例工程

下面新建spring boot示例工程，示例工程的代码地址为：https://gitee.com/tinylk/pipeline-demo

###### 创建spring boot示例工程

1.通过 https://start.spring.io/ 生成spring boot基础工程，添加一个示例Controller类。package com.example.demo.controller;

```
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomeController {

    @GetMapping("")
    public String hello() {
        return "Hello!";
    }
}
```

2.修改application配置文件，设置端口。

```
spring.application.name=pipeline-demo
server.port=40080
```

3.编译运行，访问 http://localhost:40080 地址可以看到示例运行结果。

**添加Dockerfile**

在工程根目录创建Dockerfile，用来构建docker镜像。其中${JAR_FILE}参数在pipeline执行docker build时，通过build-arg参数传入。

```
FROM openjdk:8-jdk-alpine
#构建参数
ARG JAR_FILE
ARG WORK_PATH="/opt/demo"
# 环境变量
ENV JAVA_OPTS="" \
    JAR_FILE=${JAR_FILE}

#设置时区
RUN apk update && apk add ca-certificates && \
    apk add tzdata && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

COPY target/$JAR_FILE $WORK_PATH/

WORKDIR $WORK_PATH

ENTRYPOINT exec java $JAVA_OPTS -jar $JAR_FILE
```

**添加k8s的Deployment配置**

在工程根目录创建k8s-deployment.tpl文件，此文件用来作为k8s的yaml文件模板。在jenkens  pipeline执行时，会先将tpl文件中{}括起来的自定义参数用sed命令替换为实际的内容。（由于对k8s的了解有限，不知道有没有更好的方式来实现）

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {APP_NAME}-deployment
  labels:
    app: {APP_NAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {APP_NAME}
  template:
    metadata:
      labels:
        app: {APP_NAME}
    spec:
      containers:
      - name: {APP_NAME}
        image: {IMAGE_URL}:{IMAGE_TAG}
        ports:
        - containerPort: 40080
        env:
          - name: SPRING_PROFILES_ACTIVE
            value: {SPRING_PROFILE}
```

**添加Jenkinsfile**

在工程根目录创建Jenkinsfile，用来执行jenkins pipeline任务。Jenkinsfile文件的大概内容描述如下：

- environment中变量说明，environment的文档说明参见：https://jenkins.io/zh/doc/book/pipeline/jenkinsfile/#%E5%A4%84%E7%90%86%E5%87%AD%E8%AF%81 。

- - HARBOR_CREDS为harbor镜像仓库的用户密码，数据保存为jenkins的“username and  password”类型的凭据，用credentials方法从凭据中获取。使用时通过HARBOR_CREDS_USR获取用户名，HARBOR_CREDS_PSW获取密码。
  - K8S_CONFIG为k8s中kubectl命令的yaml配置文件内容，数据保存为jenkins的“Secret  Text”类型的凭据，用credentials方法从凭据中获取。这里保存的yaml配置文件内容以base64编码格式保存，在设置凭据时先要进行base64编码。（此base64编码是非必须的，如果直接保存原文，下面Jenkinsfile中需要去掉base64 -d 解码）
  - GIT_TAG变量通过执行sh命令获取当前git的tag值。由于后面构建docker镜像时使用git的tag作为镜像的标签，所以这个变量也不能为空。

- parameters中变量说明

```
HARBOR_HOST：harbor镜像仓库地址。
DOCKER_IMAGE：docker镜像名，包含harbor项目名称。
APP_NAME：k8s中的标签名称，对应k8s的yaml模板中的{APP_NAME}。
K8S_NAMESPACE：k8s中的namespace名称，执行kubectl命令会部署至此命名空间。
```

- stages说明：

- - Maven Build：使用docker的方式执行maven命令，args参数中将.m2目录映射出来，避免执行时重复从远端获取依赖；stash步骤中将jar文件保存下来，供后面的stage使用。
  - Docker Build：unstash获取jar文件。通过sh依次执行docker命令登录harbor、构建镜像、上传镜像、移除本地镜像。构建镜像时，会获取jar文件名传入JAR_FILE参数。
  - Deploy：使用docker的方式执行kubectl命令。在执行前先将K8S_CONFIG中的内容进行base64解密并存为~/.kube/config配置文件，然后执行sed命令将k8s-deployment.tpl文件中“{参数名}”形式参数替换为实际的参数值，最后执行kubectl命令部署至k8s。

parameters中的参数在pipeline任务执行一次后，第二次执行时可以在界面上修改参数值。

这里的stage都是一个接一个自动执行，如果需要手动执行可以使用input指令实现。参见：https://jenkins.io/zh/doc/book/pipeline/syntax/#input

```
// 需要在jenkins的Credentials设置中配置jenkins-harbor-creds、jenkins-k8s-config参数

pipeline {
    agent any
    environment {
        HARBOR_CREDS = credentials('jenkins-harbor-creds')
        K8S_CONFIG = credentials('jenkins-k8s-config')
        GIT_TAG = sh(returnStdout: true,script: 'git describe --tags --always').trim()
    }
    parameters {
        string(name: 'HARBOR_HOST', defaultValue: '172.23.101.66', description: 'harbor仓库地址')
        string(name: 'DOCKER_IMAGE', defaultValue: 'tssp/pipeline-demo', description: 'docker镜像名')
        string(name: 'APP_NAME', defaultValue: 'pipeline-demo', description: 'k8s中标签名')
        string(name: 'K8S_NAMESPACE', defaultValue: 'demo', description: 'k8s的namespace名称')
    }
    stages {
        stage('Maven Build') {
            when { expression { env.GIT_TAG != null } }
            agent {
                docker {
                    image 'maven:3-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -Dfile.encoding=UTF-8 -DskipTests=true'
                stash includes: 'target/*.jar', name: 'app'
            }

        }
        stage('Docker Build') {
            when { 
                allOf {
                    expression { env.GIT_TAG != null }
                }
            }
            agent any
            steps {
                unstash 'app'
                sh "docker login -u ${HARBOR_CREDS_USR} -p ${HARBOR_CREDS_PSW} ${params.HARBOR_HOST}"
                sh "docker build --build-arg JAR_FILE=`ls target/*.jar |cut -d '/' -f2` -t ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG} ."
                sh "docker push ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
                sh "docker rmi ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
            }
            
        }
        stage('Deploy') {
            when { 
                allOf {
                    expression { env.GIT_TAG != null }
                }
            }
            agent {
                docker {
                    image 'lwolf/helm-kubectl-docker'
                }
            }
            steps {
                sh "mkdir -p ~/.kube"
                sh "echo ${K8S_CONFIG} | base64 -d > ~/.kube/config"
                sh "sed -e 's#{IMAGE_URL}#${params.HARBOR_HOST}/${params.DOCKER_IMAGE}#g;s#{IMAGE_TAG}#${GIT_TAG}#g;s#{APP_NAME}#${params.APP_NAME}#g;s#{SPRING_PROFILE}#k8s-test#g' k8s-deployment.tpl > k8s-deployment.yml"
                sh "kubectl apply -f k8s-deployment.yml --namespace=${params.K8S_NAMESPACE}"
            }
            
        }
        
    }
}
```

## 配置jenkins pipeline任务

创建jenkins pipeline任务，并设置需要的参数。

###### 新建pipeline任务

点击“新建任务”，输入名称并选择“流水线”（pipeline），然后点击确定。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###### 配置 pipeline任务 

进入任务的配置界面，在流水线（pipeline）设置部分，选择“Pipeline script from SCM”。SCM选项选为“Git”，配置好工程的git地址以及获取代码的凭证信息。然后在“Additional  Behaviours”中添加“Clean before checkout”。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0n1IVYv3WMhib2Uu67PL9OXeicCuOkwxYgyCZgzVxdrgIoXw2FCG2zfSA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

###### 配置harbor账号与密码

选择“凭据”，然后在下图所示位置点击“添加凭据”。在新凭据设置界面，类型选择为“Username with  password”，ID设置为“jenkins-harbor-creds”（此处的ID必须与Jenkinsfile中的保持一致）。Username与Password分别设置为harbor镜像私库的用户名和密码。![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0wwW2tXEILoj3tibjOJd2gFoKvtE49ictibUCeBzCsKYND9PGV9IU8bAMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)添加凭证![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0KUoB386s6efIEKKg3HFr2YXTh1rX5ysibL4QUbBwOvz8KdkKUsrBYrA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)设置habor账号密码

###### 配置k8s的kube.config配置信息

k8s中使用kubectl命令时需要yaml格式的服务器及授权信息配置文件。这里将kubectl的yaml配置文件的内容以base64编码后保存在jenkins的凭据中。pipeline任务执行时，先从jenkins凭据中获取内容，进行base64解码后将配置保存为~/.kube/config文件。kubectl的配置文件的内容如下：

```
apiVersion: v1
kind: Config
clusters:
- name: "test"
  cluster:
    server: "https://xxxxx"
    api-version: v1
    certificate-authority-data: "xxxxxx"

users:
- name: "user1"
  user:
    token: "xxxx"

contexts:
- name: "test"
  context:
    user: "user1"
    cluster: "test"

current-context: "test"
```

可以在linux中采用下面命令将kubectl的yaml配置文件进行base64编码。

```
base64 kube-config.yml > kube-config.txt
```

然后类似上一步，在jenkins凭据中增加配置文件内容。在凭据设置界面，类型选择为“Secret  text”，ID设置为“jenkins-k8s-config”（此处的ID必须与Jenkinsfile中的保持一致），Secret设置为上面经过base64编码后的配置文件内容。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0T3v9ibrCBSUbflcOK9L7micUwib8xQWmCgrLzEWI9lvpK7qWibk8cQEyVg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 测试pipeline任务 

在创建的pipeline任务中，点击“立即构建”即可立即执行pipeline任务。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg01FLfpJBsKMibicNVzpJ4DnRMb2g2ibdx6D4Fn9tDSPxkauQibD1eWhibUOA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在jenkins中存在一个名为Blue Ocean的新界面，在新界面中也可以执行pipeline任务，而且新的界面中查看任务的执行结果更加清晰，所以执行结果日志建议在新界面中查看。点击“Open Blue Ocean”菜单进入新界面。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0qhicSq6DicfmQCjHbBNicPvcqOOKDFghf4ericeSZX0TAySyaq03sBmOkQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在Blue Ocean新界面中，可以点击“运行”执行pipeline任务。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在执行pipeline任务，执行“git describe --tags”命令出错，这是由于上面示例的Jenkinsfile中要求必须有git的tag。在git中增加tag，并提交上传至gitlab后即可解决。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在git中增加tag，并提交上传至gitlab后重新执行pipeline任务，所有步骤都执行成功。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0RhiaFSvlMoicxGib95mYTMotia3GUiasibg9FRWocoRysaYicK1zlOdib1vicAQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

执行成功后，查看harbor镜像仓库，docker镜像成功上传至harbor。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0WvPiactk410WZCibTjPLwsmcWWJOIL7gbfcOKOvX54vFHjh6wlSyGIXw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

执行成功后，查看k8s中pod运行日志，服务启动成功。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0MtGVIUXHRK1N9XGtlmEA56KH6HRafIqm19yPTkCVySPx8gFNzb24rQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 设置gitlab自动触发jenkins

前面的步骤中已经完成了手动执行jenkins执行pipeline任务完成构建部署任务，下面说明如何在代码提交后让gitlab自动触发jenkins执行pipeline任务。

###### jenkins中安装gitlab插件

要实现gitlab自动触发jenkins任务，需要在jenkins中安装gitlab插件。从jenkins的“系统管理”-“管理插件”中找到gitlab插件并下载安装，重启jenkins后生效。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

##### 在gitlab中创建访问token 

安装的gitlab插件在配置时，需要gitlab的访问token，先要在gitlab中创建访问的token。点击“用户设置”-“Access Tokens”菜单，进入访问token设置界面。然后输入“Name”，并在“Scopes”中勾选“api”，点击“Create  ...”按钮即可创建访问token。创建成功后，一定要将生成的token保存下来，后面将无法再查看生成的token，如果未保存后面只能重新生成。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg06BibgkEC19jnibYHlAq4o39EPqVtF3QIOaMCjxakM2ffRdic09uh2BgIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## jenkins中配置gitlab插件

在jenkins中，进入“系统管理”-“系统设置”-“Gitlab”配置。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0YXKeiaMQbic55tiaF5MOEO0VmLMNHFfkibIibj0HsDIXOFREic81FSIUvVKw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

输入“Connection name”、“Gitlab host URL”，在“Credentials”中点击“Add”添加gitlab的授权token。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0PmH6LwZLAun2cN7dGbDhibPrN3AF24pQ73lVOIeW93qBtIkZOIaFgAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击添加后，会进入添加凭据界面。在添加界面，选择类型为“GitLab API token”，API token设置为前面在gitlab中创建的token值。然后点击“添加”按钮创建凭据。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

添加完gitlab api token，在Gitlab设置界面“Credentials”中选择前面添加的token，然后点击“Test Connection”测试是否成功。测试成功后保存。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 配置pipeline任务，启用触发器

进入jenkins的任务设置界面，在“构建触发器”中，勾上“Build when a change pushed to Gitlab.Gitlab webhook URL ...”（这里的webhook  URL在后面配置gitlab时需要），根据自己的需要设置其它的选项。点击“高级”按钮，然后点击“Generate”按钮生成Secret  token（这里的token后面配置gitlab时需要）。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0ZtW55hjhEq9hqL2cAJGw04Hpa2k451E0BWQLmIY7urjSGaQjXtDm0w/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg07OxahsBwBKTQOSiaRFSUh20rl3FCq4exyn7wZ98Xibcehdib5rribYgVgQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## gitlab配置webhook 

在gitlab的项目设置中选择“Integrations”，然后在URL和Secret token中填入上一步jenkins设置中的到的内容。由于本文中是在git上打了tag后才触发jenkins执行任务，所以这里勾选“Tag  push events”，然后去掉“Enable SSL verification”的勾选项，并点击“Add  webhook”按钮添加。如果需要每次提交代码都触发则应该选“Push events”（注意：**这里选择Push events后，Jenkinsfile中也需要做相应的修改，去掉git的tag相关内容，否则执行时会出错）。**

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0voicR49xQU5292bJ7R1oewxicVzibqQL6wwDUfibPkavy45cPCT14XRuow/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## webhook测试 

添加完成后，在下面可以看到刚才添加的webhook，点击“Test”按钮在弹出的菜单中选择“Tag push events”发送测试请求，发送成功后会显示“Hook executed successfully”信息。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0zicRr31zgG1rCOxpr3Kf3SZFgAbic1pGCFGySvBah7ib2WA2DHxfncDmw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Tag push Test

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0QaBm9U0AiaKViaOt0ib9mgTnyjwg2nwofqsSrY5m4reQVSibx1ovlVRXbg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)push成功

webhook发送成功后，到jenkins中可以看到正在执行触发的任务。配置成功后，后面只要有git的tag提交到gitlab即可直接触发jenkins执行pipline任务。

![图片](https://mmbiz.qpic.cn/mmbiz/tuSaKc6SfPpy5SBtMtUqemZ3iaiaia19Qg0X5Q5HicVaFXAlTLlEmlNHAiaFWtmvtu9icEaDkzMSDCKsUT2zVMDo6SqQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

webhook触发执行

> *作者：tinylk*
>
> *原文：www.jianshu.com/p/2d89fd1b4403*