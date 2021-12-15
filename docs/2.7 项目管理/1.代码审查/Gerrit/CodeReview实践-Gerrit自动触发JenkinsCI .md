- [CodeReview实践-Gerrit自动触发JenkinsCI](https://mp.weixin.qq.com/s/7zVA_Y3BBGL9RfcGp7Dznw)

## Gerrit + Jenkins

### 背景

当前团队使用Gerrit来做代码管理、CodeReview。计划实现当review提交到了Gerrit并且review通过（merged）自动触发Jenkins流水线。以前接触Gitlab比较多，Gerrit还是第一次开始用，踩了点坑记录下来。本文主要讲述Gerrit Trigger流水线配置，关于服务器配置等细节问题暂不研究，降低复杂性。

### Gerrit 配置

我们可以通过Docker的方式快速启动一个Gerrit实例，默认Gerrit使用的是HTTP 8080端口、SSH29418端口。通过`CANONICAL_WEB_URL`参数指定服务器网页地址。

```bash
docker run --name gerrit -itd \
-p 8088:8080 \
-p 29418:29418 \
-e CANONICAL_WEB_URL=http://192.168.1.200:8088 gerritcodereview/gerrit
```

启动成功后，默认打开的是一个插件安装的页面，此时可以根据个人需要安装相关插件，也可以跳过。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqDtu37MO26Fv4CozO4w5mXSkKItfh09evECwpzrqoF7x6qVFGGLT7sA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

默认登录就是admin, 创建一个Jenkins用户。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqcxkgvK5xBME6ECAcDxrSwLSJVIic8ibVkjUJoCNomHK0drCXtLwSib5zQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



登录Jenkins用户然后配置SSH-KEY，创建ssh-key添加到jenkins用户配置中。

```bash
[root@zeyang-nuc-service ~]# kubectl exec -it jenkins-6ccf555769-sfdw6 -n devops bash
bash-4.2$ id
uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins)
bash-4.2$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):
Created directory '/var/jenkins_home/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/jenkins_home/.ssh/id_rsa.
Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:nGqkSVAUuc2xrGe8Bz/xuWcQ/YVrDISPJux+tCZkJgI jenkins@jenkins-6ccf555769-sfdw6
The key's randomart image is:
+---[RSA 2048]----+
|   .+o     .     |
|   .. .   . .    |
|  .  = +   =   . |
|  E.. =.o.+ + . .|
|   ..o..So . + o |
|   .o+*.* o   =  |
|    o+oX + + .   |
|     .. * * o    |
|       . =.+     |
+----[SHA256]-----+
```

默认的key在JENKINS_HOME目录中`/var/jenkins_home/.ssh/id_rsa`。

```bash
bash-4.2$ cat /var/jenkins_home/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCb+BcXnBXG4f4T3MSDsL/aNLm4zlMkX5xn5pwC4eaep+XMe9kXMsYJZ3xuQ1dxUTAeTHAYX33IsclpE63H0nXdNj8cgcC9dnyXFYGieKfSx44JeP3O4rcMFN+cPGlEcIVJdTF8RfpvDANObCUJ0fnsw7f/yVImdwqGbXaBsU11+s6uRuCghXUw1JhA4H+mVp89YZN7ilhif4I8rol/cUkcKnQhxM0ziClWL5VLBTfpO5QNhj+vy2JICMSgU93EEs0LgBUdT2Q+1tduQo3R7fNOkQm46y1oonoUMzXTr9/kOlcAxZR9kIT7WYPxGQGCoyf2AiMP3VKwowv98MenDCFZ jenkins@jenkins-6ccf555769-sfdw6
```

这里使用的是id_rsa.pub，复制文件内容，然后添加到Gerrit Jenkins用户中。（记得点击ADD）

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqC2FMj40VS2kVpslkPXWQYbqPRKQkZrI5ibSmXz4wCKapIOaucibf3fLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



将Jenkins用户加入`Non-interactive Users`组。`BROWSE>Groups>Non-Interactive Users>Members`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqoZU90w89iaVlKfdfNMiaxib76NYdSFZAzwsLpDb3lGr0xVeiazaVdJYVMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



创建一个仓库，然后简单的设置下repo权限：

```bash
refs/* ：read Non-interactive Users
refs/heads/* : Label Code-Review Non-interactive Users
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqTlhRyYia5NCKl6mWKZiczvKynlJtiazNibhlOGD4RxMdmnoQcOK0d2ibDGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Gerrit 2.7+  创建一个组`Event Streaming Users`，将Jenkins用户加入。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoq2ib72mQsLzUibh7FmLbMMN2TpIlJLB1s7emXnHkTntHOqNtsuHYjB59w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



设置`All-projects` access 权限， `BROWSE> repos>All-Projects>Access>Global Capabilities >Stream Events` 。

- 

```
allow Event Streaming Users
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqbHl5G9U0qEcib5YpHicHAHh6I37VicXfW9cEPBIicTGcUeJt3icvNC8icqyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



到此，Gerrit配置基本上已经完成了，页面样式很简洁。

------

### Jenkins配置

首先我们安装Gerrit Hook插件，然后进入系统管理会看到gerrit的图标。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqw0ibf29BQ5wibUAppwdjTofqomkB9HwNYM9fBJ9Tyc7QocGfaunPBFAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqm40jwtIc8zpSmIcV2cXY0KbJ0f7TQ8fd4twMHVrsmNR7sO5ry0GsHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





**Connection error : com.jcraft.jsch.JSchException: Auth fail** 错误一般是ssh-key问题。

在流水线项目中添加`Gerrit Trigger`.

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqyG8P4R1GVknFsCVoyNKZA5e4h2Ik5AaKLPZnrl2ibKjL8h5ia9nic3v2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Ok，Jenkins的配置完成了。接下来开始测试自动触发。

------

### 创建codereview

```bash
[root@zeyang-nuc-service devops]# ls
aa,txt  aasss,txt  sss  test.txt
[root@zeyang-nuc-service devops]# echo 123 >test.txt
[root@zeyang-nuc-service devops]# git add .
[root@zeyang-nuc-service devops]# git commit -m "init"
[master 77f6474] init
 1 file changed, 1 insertion(+), 1 deletion(-)
[root@zeyang-nuc-service devops]# git push origin HEAD:refs/for/master
Username for 'http://192.168.1.200:8088': admin
Password for 'http://admin@192.168.1.200:8088':
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 253 bytes | 253.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 1, done
remote:
remote: SUCCESS
remote:
remote:   http://192.168.1.200:8088/c/devops/+/21 init [NEW]
remote:
To http://192.168.1.200:8088/devops
 * [new branch]      HEAD -> refs/for/master
```

merge 测试

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqmFMEXU0iaXjXibvkCkdl7gBqSHiawbrUxhevF8yHxR20JHQoO5tJdnDzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqMicKaylaNAGVfGC2qV2IC0ia5wZWow5D1suGwcFicTpUgexzuTRmdNamQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Gerrit传递的参数还是挺多的，可以很方便的获取。基本上这些参数就够用了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTN0szN9RkK3RFh6UkrKCYoqia9yC6ric5iclbuAy3QRwhrNySXib3Y1kPsp5tpVoIZ17DGnjY9pvQRFicg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



------

### Pipeline As Code

```bash
//Pipeline params
String BRANCH_NAME = "${env.GERRIT_BRANCH}"
String PROJECT_NAME = "devops"
String PROJECT_URL = "http://192.168.1.200:8088/devops"
currentBuild.description = "Trigger By ${BRANCH_NAME}"

//Pipeline
pipeline{
    agent {
        node {  label "build"   //指定运行节点的标签或者名称
        }
    }

    options{
        skipDefaultCheckout()
    }

    triggers {
        //配置gerrit触发器
        gerrit  customUrl: '',
                gerritProjects: [[branches: [[compareType: 'ANT', pattern: '**']],
                compareType: 'PLAIN',
                disableStrictForbiddenFileVerification: false,
                pattern: "${PROJECT_NAME}"]],
                serverName: 'devops',
                triggerOnEvents: [changeMerged()]
    }

    stages{

        stage("GetCode"){
            steps{
                echo "========executing GetCode========"
                //下载代码
                checkout([$class: 'GitSCM', branches: [[name: "${BRANCH_NAME}"]],
                                      doGenerateSubmoduleConfigurations: false,
                                      extensions: [],
                                      submoduleCfg: [],
                                      userRemoteConfigs: [[url: "${PROJECT_URL}"]]])
            }
        }
    }
    post{
        always{
            echo "========always========"
            cleanWs()
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
```

到此基本上触发就已经完成了，后续添加构建和发布步骤。Gerrit进行CodeReview还是很方便的，现在每次提交的代码、Jenkinsfile都需要先进行CodeReview才能进行merge。