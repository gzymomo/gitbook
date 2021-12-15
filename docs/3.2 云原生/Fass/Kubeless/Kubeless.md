- [Kubeless 学习手册（1）：自建 Serverless 平台](https://www.jianshu.com/p/5c55f9c4ee18)
- 

# 一、Kubeless概述

Github地址：https://github.com/kubeless/kubeless

Kubeless 是一个基于 Kubernetes 的 Serverless 框架，允许您部署少量代码，而无需担心底层基础架构管道。它利用 Kubernetes 资源提供自动扩展、API 路由、监控、故障排除等功能。

Kuberless 包含以下部分：

- 持 Python、Node.js、Ruby、PHP、Golang、.NET、Ballerina 和自定义运行时
- CLI 兼容 AWS Lambda CLI
- 事件触发器使用 Kafka 消息传递系统和 HTTP 事件
- Prometheus 默认监控函数调用和函数延迟
- Serverless Framework 插件

# 二 Kubeless架构

## 1 Kubeless基本组成

Kubeless主要由以下三部分组成：

- Functions
- Triggers
- Runtime

下面针对这三个组成部分，进行详细介绍。

### Functions

Functions 表示要执行的代码，即为函数，在Kubeless中函数包含有关其运行时的依赖、构建等元数据。函数具有独立生命周期，并支持以下方法：
 （1） Deploy： Kubeless 将函数部署为 Pod的形式运行在Kubernetes集群中，此步骤会涉及构建函数镜像等操作。
 （2）Execute：执行函数，不通过任何事件源调用。
 （3）Update：修改函数元数据。
 （4）Delete：在Kubernetes集群中删除函数的所有相关资源。
 （5）List：显示函数列表。
 （6）Logs：函数实例在Kubernetes中生成及运行的日志。

### Triggers

Triggers表示函数的事件源，当事件发生时，Kubeless确保最多调用一次函数，Triggers可以与单个功能相关联，也可与多个功能相关联，具体取决于事件源类型。Triggers与函数的生命周期解耦，可以进行如下操作：
 （1）Create：使用事件源和相关功能的详细信息创建 Triggers。
 （2）Update: 更新 Triggers元数据。
 （3）Delete：删除Triggers及为其配置的任何资源。
 （4）List：显示Triggers列表。

### Runtime

函数使用语言因不同用户的喜好通常多样化， Kubeless 为用户带来了几乎所有的主流函数运行时， 目前含有[3]：
 （1） Python: 支持2.7、3.4、3.6版本。
 （2） NodeJS: 支持6、8版本。

（3） Ruby: 支持2.4版本。
 （4） PHP: 支持7.2版本。
 （5） Golang: 支持1.10版本。
 （6） .NET: 支持2.0版本。

（7） Ballerina: 支持0.975.0版本。
 在Kubeless中，每个函数运行时都会以镜像的方式封装在容器镜像中，通过在Kubeless配置中引用这些镜像来使用，可以通过 Docker CLI 查看源代码。

## 2 Kubeless设计方式

与其它开发框架一样， Kubeless也有自己的设计方式，Kubeless利用Kubernetes中的许多概念来完成对函数实例的部署，主要使用了 Kubernetes以下特性【2】 ：

（1） CRD（ 自定义资源） 用于表示函数。
 （2） 每个事件源都被当作为一个单独的 Trigger CRD 对象。
 （3） CRD Controller 用于处理与 CRD 对象相应的 CRUD 操作。
 （4） Deployment/Pod 运行相应的运行时。
 （5） ConfigMap 将函数的代码注入运行时的 Pod。
 （6） Init-container 加载函数的依赖项。
 （7） 使用Service在集群中暴露函数（ ClusterIP）。
 （8） 使用Ingress资源对象暴露函数到外部。
 Kubernetes CRD 和 CRD Controller 构成了 Kubeless 的设计宗旨，对函数和 Triggers 使用不同的 CRD 可以明确区分关键点，使用单独的 CRD Controller 可以使代码解耦并模块化。

部署Kubeless之后，集群中Kubeless对应的namespace中会出现三个CRD以代表Kubeless架构中的Functions和Triggers，如图 2所示，在此之后每通过Kubeless CLI创建的 Functions、 Triggers 均隶属于这三个CRD端点下，

![img](http://blog.nsfocus.net/wp-content/uploads/2018/10/ccbe3312cc80d9f33ba0c9f40aa3f862.png)

图2 Kubeless的CRD

在Kubeless上部署函数的过程可分为以下三步【2】【2】：2

- Kubeless CLI 读取用户输入的函数运行配置， 产生一个 Function 对象
   并将其提交给Kubernetes API Server。
- Kubeless Function Controller（运行在Kubeless Controller Manager中，  安装完Kubeless后在集群中默认存在的 Deployment, 用于监听及处理函数的相应事件）  监测到有新的函数被创建并且读取函数信息，由提供的函数信息  Kubeless首先产生一个带有函数代码及其依赖关系的ConfigMap，再产生一个用于内部通过HTTP或其它方式访问的  Service，最后产生一个带有基本镜像的Deployment ，以上生成的顺序十分重要，因为若 Kubeless  中的Controller无法部署ConfigMap 或 Service，则不会创建Deployment。任何步骤中的失败都会中止该过程。
- 创建完函数对应的Deployment后， 集群中会跑一个对应的 Pod， Pod在启动时会动态的读取函数中的内容。