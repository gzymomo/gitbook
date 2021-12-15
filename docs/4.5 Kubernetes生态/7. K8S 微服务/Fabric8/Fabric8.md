**Fabric8** 是开源 Java Containers(JVMs) 深度管理集成平台。

有了 fabric8 可以非常方便的从 UI 和 UX 一致的中央位置进行自动操作，配置和管理。

fabric8 同时提供一些非功能性需求，比如配置管理，服务发现故障转移，集中化监控，自动化等等。

官网地址：http://fabric8.io/

# 一、Fabric8简介

**fabric8**是一个开源**集成开发平台**，为基于[Kubernetes](http://kubernetes.io/)和[Jenkins](https://jenkins.io/)的微服务提供[持续发布](http://fabric8.io/guide/cdelivery.html)。

使用fabric可以很方便的通过[Continuous Delivery pipelines](http://fabric8.io/guide/cdelivery.html)创建、编译、部署和测试微服务，然后通过Continuous Improvement和[ChatOps](http://fabric8.io/guide/chat.html)运行和管理他们。

[Fabric8微服务平台](http://fabric8.io/guide/fabric8DevOps.html)提供：

- [Developer Console](http://fabric8.io/guide/console.html)，是一个[富web应用](http://www.infoq.com/cn/news/2014/11/seven-principles-rich-web-app)，提供一个单页面来创建、编辑、编译、部署和测试微服务。

- [Continuous Integration and Continous Delivery](http://fabric8.io/guide/cdelivery.html)，使用 [Jenkins](https://jenkins.io/) with a [Jenkins Workflow Library](http://fabric8.io/guide/jenkinsWorkflowLibrary.html)更快和更可靠的交付软件。

- [Management](http://fabric8.io/guide/management.html)，集中式管理[Logging](http://fabric8.io/guide/logging.html)、[Metrics](http://fabric8.io/guide/metrics.html), [ChatOps](http://fabric8.io/guide/chat.html)、[Chaos Monkey](http://fabric8.io/guide/chaosMonkey.html)，使用[Hawtio](http://hawt.io/)和[Jolokia](http://jolokia.org/)管理Java Containers。

- [Integration](http://fabric8.io/guide/ipaas.html) *Integration Platform As A Service* with [deep visualisation](http://fabric8.io/guide/console.html) of your [Apache Camel](http://camel.apache.org/) integration services, an [API Registry](http://fabric8.io/guide/apiRegistry.html) to view of all your RESTful and SOAP APIs and [Fabric8 MQ](http://fabric8.io/guide/fabric8MQ.html) provides *Messaging As A Service* based on [Apache ActiveMQ](http://activemq.apache.org/)。

- Java Tools

   帮助Java应用使用

  Kubernetes

  :  

  - [Maven Plugin](http://fabric8.io/guide/mavenPlugin.html) for working with [Kubernetes](http://kubernetes.io/) ，这真是极好的
  - [Integration and System Testing](http://fabric8.io/guide/testing.html) of [Kubernetes](http://kubernetes.io/) resources easily inside [JUnit](http://junit.org/) with [Arquillian](http://arquillian.org/)
  - [Java Libraries](http://fabric8.io/guide/javaLibraries.html) and support for [CDI](http://fabric8.io/guide/cdi.html) extensions for working with [Kubernetes](http://kubernetes.io/).

# 二、Fabric8微服务平台

Fabric8提供了一个完全集成的开源微服务平台，可在任何的[Kubernetes](http://kubernetes.io/)和[OpenShift](http://www.openshift.org/)环境中开箱即用。

整个平台是基于微服务而且是模块化的，你可以按照微服务的方式来使用它。

微服务平台提供的服务有：

- 开发者控制台，这是一个富Web应用程序，它提供了一个单一的页面来创建、编辑、编译、部署和测试微服务。
- 持续集成和持续交付，帮助团队以更快更可靠的方式交付软件，可以使用以下开源软件：  
  - [Jenkins](https://jenkins.io/)：CI／CD pipeline
  - [Nexus](http://www.sonatype.org/nexus/)： 组件库
  - [Gogs](http://gogs.io/)：git代码库
  - [SonarQube](http://www.sonarqube.org/)：代码质量维护平台
  - [Jenkins Workflow Library](http://fabric8.io/guide/jenkinsWorkflowLibrary.html)：在不同的项目中复用[Jenkins Workflow scripts](https://github.com/fabric8io/jenkins-workflow-library)
  - [Fabric8.yml](http://fabric8.io/guide/fabric8YmlFile.html)：为每个项目、存储库、聊天室、工作流脚本和问题跟踪器提供一个配置文件
- [ChatOps](http://fabric8.io/guide/chat.html)：通过使用[hubot](https://hubot.github.com/)来开发和管理，能够让你的团队拥抱DevOps，通过聊天和系统通知的方式来[approval of release promotion](https://github.com/fabric8io/fabric8-jenkins-workflow-steps#hubotapprove)
- [Chaos Monkey](http://fabric8.io/guide/chaosMonkey.html)：通过干掉[pods](http://fabric8.io/guide/pods.html)来测试系统健壮性和可靠性
- 管理
  - [日志](http://fabric8.io/guide/logging.html) 统一集群日志和可视化查看状态
  - [metris](http://fabric8.io/guide/metrics.html) 可查看历史metrics和可视化

# 参考

[fabric8：容器集成平台——伯乐在线](http://hao.jobbole.com/fabric8/)

[Kubernetes部署微服务速成指南——*2017-03-09 徐薛彪* 容器时代微信公众号](http://mp.weixin.qq.com/s?__biz=MzI0NjI4MDg5MQ==&mid=2715290731&idx=1&sn=f1fcacb9aa4f1f3037918f03c29c0465&chksm=cd6d0bbffa1a82a978ccc0405afa295bd9265bd9f89f2217c80f48e1c497b25d1f24090108af&mpshare=1&scene=1&srcid=0410RTk3PKkxlFlLbCVlOKMK#rd)

上面那篇文章是翻译的，英文原文地址：[Quick Guide to Developing Microservices on Kubernetes and Docker](http://www.eclipse.org/community/eclipse_newsletter/2017/january/article2.php)

[fabric8官网](https://fabric8.io/)

[fabric8 get started](http://fabric8.io/guide/getStarted/gofabric8.html)