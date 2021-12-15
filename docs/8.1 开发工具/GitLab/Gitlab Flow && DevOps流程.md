# [Gitlab Flow && DevOps流程](https://www.cnblogs.com/JulianHuang/p/13676065.html)



长话短说，本文全景呈现我司项目组gitlab flow && devops

> - Git Flow定义了一个项目发布的分支模型，为管理具有预定发布周期的大型项目提供了一个健壮的框架。
> - DevOps 强调的是团队通过自动化的工具协作和高效地沟通来完成软件的生命周期管理，从而更快、更频繁地交付更稳定的软件。开发关注代码，运维关注部署，效率和质量都能得到提升。

## 一. 团队背景[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#一-团队背景)

- 项目组10人小团队也在实践敏捷开发;
- 每个sprint周期一般包含2-3个功能;
- 采用前后端开发，生产均使用k8s部署;
- 每个sprint上线周期均经历 intergate Test--->alpha--->prod。

现代Devops技术基于**容器技术**、**自动化脚本**实现了依赖环境的打包、版本管理、敏捷部署。

## 二.我司操作[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#二我司操作)

为在迭代便利性、部署严谨性上取得平衡，项目组(其实是我。。啦)设计了如下Gitlab flow & DevOps流程。

[![img](https://img2020.cnblogs.com/blog/587720/202009/587720-20200918100448056-1372003370.png)](https://img2020.cnblogs.com/blog/587720/202009/587720-20200918100448056-1372003370.png)

一个完整的功能迭代上线周期：

### 第①阶段： 开发阶段[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#第①阶段：-开发阶段)

- 开发人员从develop切出feature分支，项目经理梳理本sprint需要上线的feature分支，自测完成后合并到develop；
- 此时会打出"develop分支"镜像，自动部署到集成测试环境，理论上还属于代码躁动的阶段；
- 开发人员应该关注集成测试环境，QA人员可酌情参与。

### 第②阶段：测试阶段[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#第②阶段：测试阶段)

- 集成测试环境验证之后， 可从develop切出release-1.0.0预发布分支，此处会打出"release-1.0.0预发布分支" 镜像，自动部署到alpha环境；
- 此处QA会重点花时间在这个环境上测试， 发现问题，开发人员迅速响应；
- 从release-1.0.0分支上切出bugfix分支，修复完后迅速合并回release-1.0.0 分支，同样会自动部署到alpha，QA快速验证；
- .....
- 这个阶段我们保持趋近一个稳定的release-1.0.0的分支。

### 第③阶段： 部署阶段[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#第③阶段：-部署阶段)

- 从稳定的release-1.0.0分支打出对应的git tags: v1.0.0， 此处会打出ImageTag:v1.0.0的镜像，需要手动部署到prod；
- QA线上测试，出现修复不的问题，迅速使用之前的ImageTag回滚；
- 上线之后若发现不能回退的bug，此时需要hotfix，还是从release-1.0.0切出hoxfix分支，修复完合并回release-1.0.0，alpha环境测试通过；打出git tags：v1.0.0-hotfix1 重新部署到prod；
- .....
- 确认上线成功，将release-1.0.0分支合并回develop、master分支

> 这里为什么保留master分支， 是因为理论上当feature分支合并回develop分支，develop已经被污染了，这里保留master只为兜底。

后续就是开始新的sprint周期了，git release分支名/tag标签名跟随迭代。

## 三.Gitlab Flow与容器结合[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#三gitlab-flow与容器结合)

整个过程贯彻了git flow 预发布分支release,hotfix的核心用法， 同时在部署方式上也有一定的改进。

- 集成测试和alpha环境使用git分支名（develop或 release-1.0.0）作为镜像Tag，快速迭代，自动部署.

针对分支的持续自动部署，本来尝试使用分支名作为镜像名，实际上git分支不像git tag， git 分支的代码是会变动的，所以这里我使用了一个骚操作：

找到此次分支提交的快照（commit_id），打出＇branchName：{commit_id} ＇这样的镜像tag，该带后缀的镜像Tag定位了此次代码快照！

[![img](https://img2020.cnblogs.com/blog/587720/202012/587720-20201201190329220-223006372.png)](https://img2020.cnblogs.com/blog/587720/202012/587720-20201201190329220-223006372.png)

- prod上要求从release分支上打出git标签，git标签名作为镜像Tag，这是常规操作。手动点击部署，放置随意部署！

## 四. 作业小抄[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#四-作业小抄)

集成测试采用docker-compose部署； alpha,prod是采用k8s部署；

> 根据十二要素App方法论，一份代码，多处部署！

**alpha,prod是不同的机器环境，在不同环境下编写部署合适的脚本，ssh远程登录机器后，执行放置的部署脚本**。

### gitlab-ci 文件[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#gitlab-ci-文件)

```

stages:
  - build
  - build_image
  - deploy

build:
  stage: build
  script: 
    - pwd
    - "for d in $(ls app/src);do echo $d;pro=$(pwd)/app/src/$d/$d.csproj; dotnet build $pro; done"
  tags:
    - my-tag

build_image:EAPWebsite:
  stage: build_image
  script:
    - dotnet publish app/src/EAP.Web/EAP.Web.csproj  -c release -o container/app/publish/
    - if [ $CI_COMMIT_REF_NAME != "$CI_COMMIT_TAG" ]; then CI_COMMIT_REF_NAME=$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA; fi;  # 如果不是正常的git tag，则骚操作定义镜像Tag
    - docker build -t $DOCKER_REGISTRY_HOST/eap/website:$CI_COMMIT_REF_NAME  container/app       
    - docker login $DOCKER_REGISTRY_HOST -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
    - docker push $DOCKER_REGISTRY_HOST/eap/website:$CI_COMMIT_REF_NAME
  tags: 
    - my-tag
  only:     
    - tags
    - develop
    - master
    - /^release-.*$/i                 # 针对git tag 和git branch打出镜像

deploy:intergate-test:
  stage: deploy
  script:
    - if [ $CI_COMMIT_REF_NAME != "$CI_COMMIT_TAG" ]; then CI_COMMIT_REF_NAME=$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA; fi;  # 如果不是正常的git tag，则骚操作定义镜像Tag
    -  echo $CI_COMMIT_REF_NAME
    - ssh -t testUser@10.202.42.252 "cd /home/eap/website && export TAG=$CI_COMMIT_REF_NAME && docker-compose pull website && docker-compose -f docker-compose.yml up -d"
  dependencies:
    - build_image:EAPWebsite
  tags:
    - my-tag
  only:
    - develop                        # develop分支，自动部署到集成测试环境

deploy:alpha:
  stage: deploy
  script:
    - if [ $CI_COMMIT_REF_NAME != "$CI_COMMIT_TAG" ]; then CI_COMMIT_REF_NAME=$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA; fi;  # 如果不是正常的git tag，则骚操作定义镜像Tag
    -  echo $CI_COMMIT_REF_NAME
    - ssh -t wd-deploy@10.202.83.148 "export TAG=$CI_Deploy_ImageTag && ./eapwebsite_cd.sh" 
  dependencies:
    - build_image:EAPWebsite
  tags:
    - my-tag
  only:
    - /^release-.*$/i                # release-1.0.0分支，自动部署到alpha测试环境

deploy:prod:
  stage: deploy
  script:
    - if [ $CI_COMMIT_REF_NAME != "$CI_COMMIT_TAG" ]; then CI_COMMIT_REF_NAME=$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA; fi;  # 如果不是正常的git tag，则骚操作定义镜像Tag
    -  echo $CI_COMMIT_REF_NAME
    - ssh -t wd-deploy@10.202.42.20 "export TAG=$CI_COMMIT_REF_NAME && ./eapwebsite_cd.sh"
  dependencies:
    - build_image:EAPWebsite
  tags:
    - my-tag
  only:
    - tags
  when: manual                       # prod环境，人工点击部署
```

1. 使用ssh远程部署，请参阅  https://www.cnblogs.com/JulianHuang/p/13374066.html
2. 基于docoer-compose完成的Gitlab-ci，请参阅 https://www.cnblogs.com/JulianHuang/p/11346615.html
3. 如果是对git分支试图制作镜像，既要保留分支名称，又要保留本次提交动作的快照，
    我们尝试使用分支名和CommitId共同作为镜像Tag，
    于是修改`CI_COMMIT_REF_NAME=$CI_COMMIT_BRANCH.$CI_COMMIT_SHORT_SHA`, 在每个Job上作为镜像tag，
    [![img](https://img2020.cnblogs.com/blog/587720/202012/587720-20201201190514988-1287376849.png)](https://img2020.cnblogs.com/blog/587720/202012/587720-20201201190514988-1287376849.png)
4. 后续部署的Job从`build.env` 读取镜像TAG

> https://docs.gitlab.com/ee/ci/variables/README.html#inherit-environment-variables

### 测试环境自动部署脚本[#](https://www.cnblogs.com/JulianHuang/p/13676065.html#测试环境自动部署脚本)

Ssh远程登录之后，进入的是wd-deploy的主目录，执行此处放置的部署脚本。

自动部署利用的是原生的`kustomize`命令，

```

#!/bin/sh

cd /home/wd-deploy/localdeploy/wd/overlays/
kustomize edit set image  repository.gridsum.com:8443/eap/website=repository.gridsum.com:8443/eap/website:${TAG}
kustomize build . | kubectl apply -f -

```

`kustomize edit set image` 可即时修改kustomization.yml文件内容，修改之后再应用，起到了部署的效果！

------

以上是一个完整的【集成测试环境---->  针对release分支打包镜像到alpha环境（有骚操作）---> 针对生产自动部署tag】的devops流程， 有效实用！。