要实现在 Jenkins 中的构建工作，可以有多种方式，我们这里采用比较常用的 Pipeline  这种方式。Pipeline，简单来说，就是一套运行在 Jenkins  上的工作流框架，将原来独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

# 一、概况

## 1.1 Jenkins Pipeline 核心概念：

- Node：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行环境，比如我们之前动态运行的 Jenkins Slave 就是一个 Node 节点
- Stage：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如：Build、Test、Deploy，Stage 是一个逻辑分组的概念，可以跨多个 Node
- Step：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像，由各类 Jenkins 插件提供，比如命令：sh 'make'，就相当于我们平时 shell 终端中执行 make 命令一样。

# 二、如何创建 Jenkins Pipline 呢？

- Pipeline 脚本是由 Groovy 语言实现的，但是我们没必要单独去学习 Groovy，当然你会的话最好
- Pipeline 支持两种语法：Declarative(声明式)和 Scripted Pipeline(脚本式)语法
- Pipeline 也有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一个 Jenkinsfile 脚本文件放入项目源码库中
- 一般我们都推荐在 Jenkins 中直接从源代码控制(SCMD)中直接载入 Jenkinsfile Pipeline 这种方法

我们这里来给大家快速创建一个简单的 Pipeline，直接在 Jenkins 的 Web UI 界面中输入脚本运行。

- 新建任务：在 Web UI 中点击 `新建任务` -> 输入名称：pipeline-demo -> 选择下面的 `流水线` -> 点击 `确定`

- 配置：在最下方的 Pipeline 区域输入如下 Script 脚本，然后点击保存。

  ```yaml
  node {
    stage('Clone') {
        echo "1.Clone Stage"
    }
    stage('Test') {
        echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Stage"
    }
    stage('Deploy') {
        echo "4. Deploy Stage"
    }
  }
  ```

- 构建：点击左侧区域的 `立即构建`，可以看到 Job 开始构建了

隔一会儿，构建完成，可以点击左侧区域的 `Console Output`，我们就可以看到如下输出信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23ianicBUb55CRGwkGDqCn1gZ7dmIu7nmUqiaaqMZewo2LibG7zssztUicia1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

console output 我们可以看到上面我们 Pipeline 脚本中的4条输出语句都打印出来了，证明是符合我们的预期的。

如果大家对 Pipeline 语法不是特别熟悉的，可以前往输入脚本的下面的链接 流水线语法 中进行查看，这里有很多关于 Pipeline 语法的介绍，也可以自动帮我们生成一些脚本。

## 2.1 在 Slave 中构建任务

上面我们创建了一个简单的 Pipeline 任务，但是我们可以看到这个任务并没有在 Jenkins 的 Slave 中运行，那么如何让我们的任务跑在 Slave  中呢？还记得上节课我们在添加 Slave Pod 的时候，一定要记住添加的 label 吗？没错，我们就需要用到这个  label，我们重新编辑上面创建的 Pipeline 脚本，给 node 添加一个 label 属性，如下：

```yaml
node('ydzs-jnlp') {
  stage('Clone') {
    echo "1.Clone Stage"
  }
  stage('Test') {
    echo "2.Test Stage"
  }
  stage('Build') {
    echo "3.Build Stage"
  }
  stage('Deploy') {
    echo "4. Deploy Stage"
  }
}
```

我们这里只是给 node 添加了一个 `ydzs-jnlp` 这样的一个 label，然后我们保存，构建之前查看下 kubernetes 集群中的 Pod：

```bash
$ kubectl get pods -n kube-ops
NAME                                           READY   STATUS    RESTARTS   AGE
jenkins-587b78f5cd-47hf8                       1/1     Running   2          4d21h
......
```

然后重新触发立刻构建：

```bash
$ kubectl get pods -n kube-ops
NAME                                           READY   STATUS    RESTARTS   AGE
jenkins-587b78f5cd-47hf8                       1/1     Running   2          4d21h
jenkins-agent-6gw0w                            1/1     Running   0          4s
```

我们发现多了一个名叫 `jenkins-agent-6gw0w` 的 Pod 正在运行，隔一会儿这个 Pod 就不再了。这也证明我们的 Job 构建完成了，同样回到 Jenkins 的 Web UI 界面中查看 Console Output，可以看到如下的信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23HEanomFDwCLWCZiaDDOHlocG6hUYGXDQtMSnXvY2No0qyImjlg91hhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`pipeline demo#2` 是不是也证明我们当前的任务在跑在上面动态生成的这个 Pod 中，也符合我们的预期。我们回到 Job 的主界面，也可以看到大家可能比较熟悉的 `阶段视图` 界面：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

# 三、部署 Kubernetes 应用

上面我们已经知道了如何在 Jenkins Slave 中构建任务了，那么如何来部署一个原生的 Kubernetes 应用呢？要部署 Kubernetes 应用，我们就得对我们之前部署应用的流程要非常熟悉才行，我们之前的流程是怎样的：

- 编写代码
- 测试
- 编写 Dockerfile
- 构建打包 Docker 镜像
- 推送 Docker 镜像到仓库
- 编写 Kubernetes YAML 文件
- 更改 YAML 文件中 Docker 镜像 TAG
- 利用 kubectl 工具部署应用

我们之前在 Kubernetes 环境中部署一个原生应用的流程应该基本上是上面这些流程吧？现在我们就需要把上面这些流程放入 Jenkins  中来自动帮我们完成(当然编码除外)，从测试到更新 YAML 文件属于 CI 流程，后面部署属于 CD  的流程。如果按照我们上面的示例，我们现在要来编写一个 Pipeline 的脚本，应该怎么编写呢？

```yaml
node('ydzs-jnlp') {
    stage('Clone') {
      echo "1.Clone Stage"
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
      echo "3.Build Docker Image Stage"
    }
    stage('Push') {
      echo "4.Push Docker Image Stage"
    }
    stage('YAML') {
      echo "5.Change YAML File Stage"
    }
    stage('Deploy') {
      echo "6.Deploy Stage"
    }
}
```

现在我们创建一个流水线的作业，直接使用上面的脚本来构建，同样可以得到正确的结果：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23Daiaud8544UE1sdvBWxmg8PPwOE9tZ1b7ibcFslGicUMedFPn51Oh009w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这里我们来将一个简单 golang 程序，部署到 kubernetes  环境中，代码链接：https://github.com/cnych/drone-k8s-demo。我们将代码推送到我们自己的 GitLab  仓库上去，地址：http://git.k8s.local/course/devops-demo，这样让 Jenkins 和 Gitlab  去进行连接进行 CI/CD。

如果按照之前的示例，我们是不是应该像这样来编写 Pipeline 脚本：

第一步，clone 代码 第二步，进行测试，如果测试通过了才继续下面的任务 第三步，由于 Dockerfile 基本上都是放入源码中进行管理的，所以我们这里就是直接构建 Docker 镜像了 第四步，镜像打包完成，就应该推送到镜像仓库中吧 第五步，镜像推送完成，是不是需要更改 YAML 文件中的镜像 TAG 为这次镜像的 TAG 第六步，万事俱备，只差最后一步，使用 kubectl 命令行工具进行部署了

到这里我们的整个 CI/CD 的流程是不是就都完成了。我们同样可以用上面的我们自定义的一个 jnlp 的镜像来完成我们的整个构建工作，但是我们这里的项目是  golang 代码的，构建需要相应的环境，如果每次需要特定的环境都需要重新去定制下镜像这未免太麻烦了，我们这里来采用一种更加灵活的方式，自定义 `podTemplate`。我们可以直接在 Pipeline 中去自定义 Slave Pod 中所需要用到的容器模板，这样我们需要什么镜像只需要在 `Slave Pod Template` 中声明即可，完全不需要去定义一个庞大的 Slave 镜像了。

这里我们需要使用到 `gitlab` 的插件，用于 Gitab 侧代码变动后触发 Jenkins 的构建任务：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23g4fKFkWrcuicZrQQsZsIKyD53XHzFdmWJjkLUhyjTXthBMAYNLKYA6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



然后新建一个名为 `devops-demo` 类型为`流水线`的任务，在 `构建触发器` 区域选择 `Build when a change is pushed to GitLab`，后面的 `http://jenkins.k8s.local/project/devops-demo` 是我们需要在 Gitlab 上配的 Webhook 地址:

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23xfib0Hn3M1vx1FM9P7uicvicLVGHXgcicicITsNgIqhPZickHYAvl86TTzTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



其中`Comment (regex) for triggering a build`是说在 git 仓库，发送包含 `jenkins build` 这样的关键字的时候会触发执行此 build 构建。然后点击下面的`高级`可以生成 token。这里的 url 和 token 是 jenkins 的 api，可以提供给 GtiLab 使用，在代码合并/提交commit/push代码等操作时，通知 Jenkins 执行 build 操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23EicVktjr1nOJRiaFSOfZBdTd1svshWKsWJxSWdIquFSCSIqORaJRFHOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



> 注: 复制出 URL 和 Token，我们后面配置 Gitlab 的 Webhook 会用到。

然后在下面的流水线区域我们可以选择 `Pipeline script` 然后在下面测试流水线脚本，我们这里选择 `Pipeline script from SCM`，意思就是从代码仓库中通过 `Jenkinsfile` 文件获取 `Pipeline script` 脚本定义，然后选择 SCM 来源为 Git，在出现的列表中配置上仓库地址 `http://git.k8s.local/course/devops-demo.git`，由于我们是在一个 Slave Pod 中去进行构建，所以如果使用 SSH 的方式去访问 Gitlab 代码仓库的话就需要频繁的去更新 `SSH-KEY`，所以我们这里采用直接使用用户名和密码的形式来方式：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23q3PsplJEfmDmhjL8ulWiaN8aYBibVs8MUsicEpK9h22VrwVdWgttEZsZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们可以看到有一个明显的错误 `Could not resolve host: git.k8s.local` 提示不能解析我们的 GitLab 域名，这是因为我们的域名都是自定义的，我们可以通过在 CoreDNS 中添加自定义域名解析来解决这个问题（如果你的域名是外网可以正常解析的就不会出现这个问题了）：

```yaml
$ kubectl edit cm coredns -n kube-system
apiVersion: v1
data:
  Corefile: |
    .:53 {
        log
        errors
        health {
          lameduck 5s
        }
        ready
        hosts {  # 添加自定义域名解析
          192.168.31.30 git.k8s.local
          192.168.31.30 jenkins.k8s.local
          192.168.31.30 harbor.k8s.local
          fallthrough
        }
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
......
```

修改完成后，隔一小会儿，CoreDNS 就会自动热加载，我们就可以在集群内访问我们自定义的域名了。然后肯定没有权限，所以需要配置帐号认证信息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23yl6HvKPu4Z8pzKIpKK2FtjibMjpoUhscSEAOUR6W608vJB7pkaUVcEA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



在 `Credentials` 区域点击添加按钮添加我们访问 Gitlab 的用户名和密码：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23pShqTKJxnSV53eZ176By0cI2qiaAplg6KKHrMI05iaEw46cP4J5q58uw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



然后需要我们配置用于构建的分支，如果所有的分支我们都想要进行构建的话，只需要将 `Branch Specifier` 区域留空即可，一般情况下不同的环境对应的分支才需要构建，比如 master、dev、test 等，平时开发的 feature 或者 bugfix 的分支没必要频繁构建，我们这里就只配置 master 个分支用于构建。

最后点击保存，至此，Jenkins 的持续集成配置好了，还需要配置 Gitlab 的 Webhook，用于代码提交通知 Jenkins。前往 Gitlab 中配置项目  devops-demo 的 Webhook，settings -> Webhooks，填写上面得到的 trigger 地址：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea2387cuRvgiaRO0lfTjgsk4G7iauZncicx8mwNknSYhyycXnh0dNJgtdw9uA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们这里都是自定义的域名，也没有配置 https 服务，所以记得取消配置下面的 `启用SSL验证`。

保存后，如果出现 `Urlis blocked: Requests to the local network are not allowed` 这样的报警信息，则需要进入 `GitLab Admin -> 设置 -> 网络` -> 勾选 `外发请求`，然后保存配置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea235KI09tb0fN52Uk11Q66OKM5MzkibkLZFUDFbCmnwpR49AJOeo3XGRNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



现在就可以正常保存了，可以直接点击 `测试 -> Push Event` 测试是否可以正常访问 Webhook 地址，出现了 `Hook executed successfully: HTTP 200` 则证明 Webhook 配置成功了，否则就需要检查下 Jenkins 的安全配置是否正确了。

由于当前项目中还没有 `Jenkinsfile` 文件，所以触发过后会构建失败，接下来我们直接在代码仓库根目录下面添加 `Jenkinsfile` 文件，用于描述流水线构建流程，整体实现流程如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23wEv6NaRNQicfHMtJ84C482kovFw3e0pFGptJpwH02KxnMrn0gIUBlng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



首先定义最简单的流程，要注意这里和前面的不同之处，这里我们使用 `podTemplate` 来定义不同阶段使用的的容器，有哪些阶段呢？

```
Clone 代码 -> 单元测试 -> Golang 编译打包 -> Docker 镜像构建/推送 -> Kubectl 部署服务。
```

Clone 代码在默认的 Slave 容器中即可；单元测试我们这里直接忽略，有需要这个阶段的同学自己添加上即可；Golang 编译打包肯定就需要  Golang 的容器了；Docker 镜像构建/推送是不是就需要 Docker 环境了；最后的 Kubectl 更新服务是不是就需要一个有  Kubectl 的容器环境了，所以我们这里就可以很简单的定义 `podTemplate` 了，如下定义：

```yaml
def label = "slave-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'golang', image: 'golang:1.14.2-alpine3.11', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker:latest', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'cnych/kubectl', command: 'cat', ttyEnabled: true)
], serviceAccount: 'jenkins', volumes: [
  hostPathVolume(mountPath: '/home/jenkins/.kube', hostPath: '/root/.kube'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH

    stage('单元测试') {
      echo "测试阶段"
    }
    stage('代码编译打包') {
      container('golang') {
        echo "代码编译打包阶段"
      }
    }
    stage('构建 Docker 镜像') {
      container('docker') {
        echo "构建 Docker 镜像阶段"
      }
    }
    stage('运行 Kubectl') {
      container('kubectl') {
        echo "查看 K8S 集群 Pod 列表"
        sh "kubectl get pods"
      }
    }
  }
}
```

直接在 `podTemplate` 里面定义每个阶段需要用到的容器，volumes 里面将我们需要用到的 `docker.sock` 文件，需要注意的我们使用的 label 标签是是一个随机生成的，这样有一个好处就是有多个任务来的时候就可以同时构建了。正常来说我们还需要将访问集群的 `kubeconfig` 文件拷贝到 kubectl 容器的 `~/.kube/config` 文件下面去，这样我们就可以在容器中访问 Kubernetes 集群了，但是由于我们构建是在 Slave Pod 中去构建的，Pod 就很有可能每次调度到不同的节点去，这就需要保证每个节点上有 `kubeconfig` 文件才能挂载成功，所以这里我们使用另外一种方式。

通过将 `kubeconfig` 文件通过凭证上传到 Jenkins 中，然后在 Jenkinsfile 中读取到这个文件后，拷贝到 kubectl 容器中的 `~/.kube/config` 文件中，这样同样就可以正常使用 kubectl 访问集群了。在 Jenkins 页面中添加凭据，选择 `Secret file` 类型，然后上传 `kubeconfig` 文件，指定 ID 即可：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23XamVvfwcwyzwYvpABBKiblQQFicwEicrqteb22X2xZQnUicCLga2hah2uw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



然后在 `Jenkinsfile` 的 kubectl 容器中读取上面添加的 `Secret file` 文件，拷贝到 `~/.kube/config` 即可：

```yaml
stage('运行 Kubectl') {
  container('kubectl') {
    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
      echo "查看 K8S 集群 Pod 列表"
      sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
      sh "kubectl get pods"
    }
  }
}
```

现在我们直接将 `Jenkinsfile` 文件提交到 GitLab 代码仓库中，正常来说就可以触发 Jenkins 的构建了：

```bash
$ kubectl get pods -n kube-ops
NAME                                                     READY   STATUS              RESTARTS   AGE
jenkins-587b78f5cd-47hf8                                 1/1     Running             2          5d
slave-e3e34e24-721a-4c32-a83e-19033e244b9d-2h6wd-tjkkf   0/4     ContainerCreating   0          38s
```

我们可以看到生成的 slave Pod 包含了4个容器，就是我们在 podTemplate 指定的加上 slave 的镜像，运行完成后该 Pod 也会自动销毁。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23QkBqKXUTycWiansevgnsY1Uazqic8gtOp9A1o4icBQMIgyAJgLAaMg0rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 四、Pipeline

接下来我们就来实现具体的流水线。

第一个阶段：单元测试，我们可以在这个阶段是运行一些单元测试或者静态代码分析的脚本，我们这里直接忽略。

第二个阶段：代码编译打包，我们可以看到我们是在一个 golang 的容器中来执行的，我们只需要在该容器中获取到代码，然后在代码目录下面执行打包命令即可，如下所示：

```yaml
stage('代码编译打包') {
  try {
    container('golang') {
      echo "2.代码编译打包阶段"
      sh """
        export GOPROXY=https://goproxy.cn
        GOOS=linux GOARCH=amd64 go build -v -o demo-app
        """
    }
  } catch (exc) {
    println "构建失败 - ${currentBuild.fullDisplayName}"
    throw(exc)
  }
}
```

第三个阶段：构建 Docker 镜像，要构建 Docker 镜像，就需要提供镜像的名称和 tag，要推送到 Harbor 仓库，就需要提供登录的用户名和密码，所以我们这里使用到了 `withCredentials` 方法，在里面可以提供一个`credentialsId` 为 dockerhub 的认证信息，如下：

```yaml
stage('构建 Docker 镜像') {
  withCredentials([[$class: 'UsernamePasswordMultiBinding',
    credentialsId: 'docker-auth',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASSWORD']]) {
      container('docker') {
        echo "3. 构建 Docker 镜像阶段"
        sh """
          docker login ${registryUrl} -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
          docker build -t ${image} .
          docker push ${image}
          """
      }
  }
}
```

其中 `${image}` 和 `${imageTag}` 我们可以在上面定义成全局变量：

```yaml
// 获取 git commit id 作为镜像标签
def imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
// 仓库地址
def registryUrl = "harbor.k8s.local"
def imageEndpoint = "course/devops-demo"
// 镜像
def image = "${registryUrl}/${imageEndpoint}:${imageTag}"
```

这里定义的镜像名称为 `course/devops-demo`，所以需要提前在 Harbor 中新建一个名为 course 的私有项目：![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23w42up0Giaib21dNwcT0HjsTDtMmvibibYDwnk50bSN5OOZQvwXJ0RqLiaSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Docker 的用户名和密码信息则需要通过凭据来进行添加，进入 jenkins 首页 -> 左侧菜单凭据 -> 添加凭据，选择用户名和密码类型的，其中 ID 一定要和上面的 `credentialsId` 的值保持一致：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23F8RI3sVAvOr5Nb9GMwBLkdCe8GupMghCicz6GuvepzoEUDnzXXleA5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

不过需要注意的是我们这里使用的是 `Docker IN Docker` 模式来构建 Docker 镜像，通过将宿主机的 `docker.sock` 文件挂载到容器中来共享 Docker Daemon，所以我们也需要提前在节点上配置对 Harbor 镜像仓库的信任:

```yaml
$ vi /etc/docker/daemon.json
{
  "insecure-registries" : [  # 配置忽略 Harobr 镜像仓库的证书校验
    "harbor.k8s.local"
  ],
  "storage-driver": "overlay2",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors" : [
    "https://ot2k4d59.mirror.aliyuncs.com/"
  ],
}
$ systemctl daemon-reload
$ systemctl restart docker
```

配置生效过后我们就可以正常在流水线中去操作 Docker 命令，否则会出现如下所示的错误：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23QSNUv8IvyKLSwoq68BSKtp0BDayckVsNYrSNq8Cic1gpElDATzJQfrg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

现在镜像我们都已经推送到了 Harbor 仓库中去了，接下来就可以部署应用到 Kubernetes 集群中了，当然可以直接通过 kubectl 工具去操作 YAML  文件来部署，我们这里的示例，编写了一个 Helm Chart 模板，所以我们也可以直接通过 Helm 来进行部署，所以当然就需要一个具有  helm 命令的容器，这里我们使用 `cnych/helm` 这个镜像，这个镜像也非常简单，就是简单的将 helm 二进制文件下载下来放到 PATH 路径下面去即可，对应的 Dockerfile 文件如下所示，大家也可以根据自己的需要来进行定制：

```bash
FROM alpine
MAINTAINER cnych <icnych@gmail.com>
ARG HELM_VERSION="v3.2.1"
RUN apk add --update ca-certificates \
 && apk add --update -t deps wget git openssl bash \
 && wget https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz \
 && tar -xvf helm-${HELM_VERSION}-linux-amd64.tar.gz \
 && mv linux-amd64/helm /usr/local/bin \
 && apk del --purge deps \
 && rm /var/cache/apk/* \
 && rm -f /helm-${HELM_VERSION}-linux-amd64.tar.gz
ENTRYPOINT ["helm"]
CMD ["help"]
```

我们这里使用的是 Helm3 版本，所以要想用 Helm 来部署应用，同样的需要配置一个 kubeconfig 文件在容器中，这样才能访问到 Kubernetes 集群。所以我们可以将 `运行 Kubectl` 的阶段做如下更改：

```yaml
stage('运行 Helm') {
  withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
    container('helm') {
      sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
      echo "4.开始 Helm 部署"
      helmDeploy(
          debug       : false,
          name        : "devops-demo",
          chartDir    : "./helm",
          namespace   : "kube-ops",
          valuePath   : "./helm/my-value.yaml",
          imageTag    : "${imageTag}"
      )
      echo "[INFO] Helm 部署应用成功..."
    }
  }
}
```

其中 `helmDeploy` 方法可以在全局中进行定义封装：

```bash
def helmLint(String chartDir) {
    println "校验 chart 模板"
    sh "helm lint ${chartDir}"
}

def helmDeploy(Map args) {
    if (args.debug) {
        println "Debug 应用"
        sh "helm upgrade --dry-run --debug --install ${args.name} ${args.chartDir} -f ${args.valuePath} --set image.tag=${args.imageTag} --namespace ${args.namespace}"
    } else {
        println "部署应用"
        sh "helm upgrade --install ${args.name} ${args.chartDir} -f ${args.valuePath} --set image.tag=${args.imageTag} --namespace ${args.namespace}"
        echo "应用 ${args.name} 部署成功. 可以使用 helm status ${args.name} 查看应用状态"
    }
}
```

我们在 Chart 模板中定义了一个名为 `my-values.yaml` 的 Values 文件，用来覆盖默认的值，比如这里我们需要使用 Harbor 私有仓库的镜像，则必然需要定义 `imagePullSecrets`，所以需要在目标 namespace 下面创建一个 Harbor 登录认证的 Secret 对象：

```bash
$ kubectl create secret docker-registry harbor-auth --docker-server=harbor.k8s.local --docker-username=admin --docker-password=Harbor12345 --docker-email=admin@admin.com --namespace kube-ops
secret/harbor-auth created
```

然后由于每次我们构建的镜像 tag 都会变化，所以我们可以通过 `--set` 来动态设置。

不过需要记得在上面容器模板中添加 helm 容器：

```
containerTemplate(name: 'helm', image: 'cnych/helm', command: 'cat', ttyEnabled: true)
```

对于不同的环境我们可以使用不同的 values 文件来进行区分，这样当我们部署的时候可以手动选择部署到某个环境下面去。

```bash
def userInput = input(
  id: 'userInput',
  message: '选择一个部署环境',
  parameters: [
      [
          $class: 'ChoiceParameterDefinition',
          choices: "Dev\nQA\nProd",
          name: 'Env'
      ]
  ]
)
echo "部署应用到 ${userInput} 环境"
// 选择不同环境下面的 values 文件
if (userInput == "Dev") {
    // deploy dev stuff
} else if (userInput == "QA"){
    // deploy qa stuff
} else {
    // deploy prod stuff
}
// 根据 values 文件再去使用 Helm 进行部署
```

然后去构建应用的时候，在 Helm 部署阶段就会看到 Stage View 界面出现了暂停的情况，需要我们选择一个环境来进行部署：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23EU0HlIlNBOsKanM1LEK8ibAM4EUJ74zmuAjmocDx3ZntzqZvVrz0rkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



选择完成后再去部署应用。最后我们还可以添加一个 kubectl 容器来查看应用的相关资源对象：

```bash
stage('运行 Kubectl') {
  withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
    container('kubectl') {
      sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
      echo "5.查看应用"
      sh "kubectl get all -n kube-ops -l app=devops-demo"
    }
  }
}
```

有时候我们部署的应用即使有很多测试，但是也难免会出现一些错误，这个时候如果我们是部署到线上的话，就需要要求能够立即进行回滚，这里我们同样可以使用 Helm 来非常方便的操作，添加如下一个回滚的阶段：

```bash
stage('快速回滚?') {
  withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
    container('helm') {
      sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
      def userInput = input(
        id: 'userInput',
        message: '是否需要快速回滚？',
        parameters: [
            [
                $class: 'ChoiceParameterDefinition',
                choices: "Y\nN",
                name: '回滚?'
            ]
        ]
      )
      if (userInput == "Y") {
        sh "helm rollback devops-demo --namespace kube-ops"
      }
    }
  }
}
```

最后一条完整的流水线就完成了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YumiaZW5IfZJLcK4oKteea23sAo2yZygFhOvWJUlR1nTvUI3XNRhdgJiaDfCecW0DePMH1URZ8MfnLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们可以在本地加上应用域名 `devops-demo.k8s.local` 的映射就可以访问应用了：

```
$ curl http://devops-demo.k8s.local
{"msg":"Hello DevOps On Kubernetes"}
```

完整的 Jenkinsfile 文件如下所示：

```bash
def label = "slave-${UUID.randomUUID().toString()}"

def helmLint(String chartDir) {
    println "校验 chart 模板"
    sh "helm lint ${chartDir}"
}

def helmDeploy(Map args) {
    if (args.debug) {
        println "Debug 应用"
        sh "helm upgrade --dry-run --debug --install ${args.name} ${args.chartDir} -f ${args.valuePath} --set image.tag=${args.imageTag} --namespace ${args.namespace}"
    } else {
        println "部署应用"
        sh "helm upgrade --install ${args.name} ${args.chartDir} -f ${args.valuePath} --set image.tag=${args.imageTag} --namespace ${args.namespace}"
        echo "应用 ${args.name} 部署成功. 可以使用 helm status ${args.name} 查看应用状态"
    }
}

podTemplate(label: label, containers: [
  containerTemplate(name: 'golang', image: 'golang:1.14.2-alpine3.11', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker:latest', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'cnych/helm', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'cnych/kubectl', command: 'cat', ttyEnabled: true)
], serviceAccount: 'jenkins', volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    // 获取 git commit id 作为镜像标签
    def imageTag = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
    // 仓库地址
    def registryUrl = "harbor.k8s.local"
    def imageEndpoint = "course/devops-demo"
    // 镜像
    def image = "${registryUrl}/${imageEndpoint}:${imageTag}"

    stage('单元测试') {
      echo "测试阶段"
    }
    stage('代码编译打包') {
      try {
        container('golang') {
          echo "2.代码编译打包阶段"
          sh """
            export GOPROXY=https://goproxy.cn
            GOOS=linux GOARCH=amd64 go build -v -o demo-app
            """
        }
      } catch (exc) {
        println "构建失败 - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    stage('构建 Docker 镜像') {
      withCredentials([[$class: 'UsernamePasswordMultiBinding',
        credentialsId: 'docker-auth',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASSWORD']]) {
          container('docker') {
            echo "3. 构建 Docker 镜像阶段"
            sh """
              cat /etc/resolv.conf
              docker login ${registryUrl} -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
              docker build -t ${image} .
              docker push ${image}
              """
          }
      }
    }
    stage('运行 Helm') {
      withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
        container('helm') {
          sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
          echo "4.开始 Helm 部署"
          def userInput = input(
            id: 'userInput',
            message: '选择一个部署环境',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Dev\nQA\nProd",
                    name: 'Env'
                ]
            ]
          )
          echo "部署应用到 ${userInput} 环境"
          // 选择不同环境下面的 values 文件
          if (userInput == "Dev") {
              // deploy dev stuff
          } else if (userInput == "QA"){
              // deploy qa stuff
          } else {
              // deploy prod stuff
          }
          helmDeploy(
              debug       : false,
              name        : "devops-demo",
              chartDir    : "./helm",
              namespace   : "kube-ops",
              valuePath   : "./helm/my-values.yaml",
              imageTag    : "${imageTag}"
          )
        }
      }
    }
    stage('运行 Kubectl') {
      withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
        container('kubectl') {
          sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
          echo "5.查看应用"
          sh "kubectl get all -n kube-ops -l app=devops-demo"
        }
      }
    }
    stage('快速回滚?') {
      withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
        container('helm') {
          sh "mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config"
          def userInput = input(
            id: 'userInput',
            message: '是否需要快速回滚？',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Y\nN",
                    name: '回滚?'
                ]
            ]
          )
          if (userInput == "Y") {
            sh "helm rollback devops-demo --namespace kube-ops"
          }
        }
      }
    }
  }
}
```