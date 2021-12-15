[TOC]

主流配置中心的比较 Spring Cloud Config、Apollo、Nacos.

# 1、Nacos
- Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您实现动态服务发现、服务配置管理、服务及流量管理。
- Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构(例如微服务范式、云原生范式)的服务基础设施。

Nacos 支持基于 DNS 和基于 RPC 的服务发现。在Spring Cloud中使用Nacos，只需要先下载 Nacos 并启动 Nacos server，Nacos只需要简单的配置就可以完成服务的注册发现。

Nacos除了服务的注册发现之外，还支持动态配置服务。动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。

## 1.1 Nacos主要提供以下四大功能
1. 服务发现与服务健康检查
Nacos使服务更容易注册自己并通过DNS或HTTP接口发现其他服务。Nacos还提供服务的实时健康检查，以防止向不健康的主机或服务实例发送请求。

2. 动态配置管理
动态配置服务允许您在所有环境中以集中和动态的方式管理所有服务的配置。Nacos消除了在更新配置时重新部署应用程序和服务的需要，这使配置更改更加高效和灵活。

3. 动态DNS服务
Nacos支持加权路由，使您可以更轻松地在数据中心的生产环境中实施中间层负载平衡，灵活的路由策略，流量控制和简单的DNS解析服务。它可以帮助您轻松实现基于DNS的服务发现，并防止应用程序耦合到特定于供应商的服务发现API。

4. 服务和元数据管理
Nacos提供易于使用的服务仪表板，可帮助您管理服务元数据，配置，kubernetes DNS，服务运行状况和指标统计。

# 2、Spring Cloud Config
为服务端和客户端提供了分布式系统的外部配置支持，配置服务器为各应用的所有环境提供了一个中心化的外部配置。Spring Cloud 配置服务器默认采用 Git 来存储配置信息，其配置存储、版本管理、发布等功能都基于 Git 或其他外围系统来实现。

支持Profile的方式隔离多个环境，通过在Git上配置多个Profile的配置文件，客户端启动时指定Profile就可以访问对应的配置文件。单独创建一个配置文件的项目，在config server的git中配置即可。

Spring Cloud Config原生不支持配置的实时推送，需要依赖Git的WebHook、Spring Cloud Bus和客户端/bus/refresh端点:

1. 基于Git的WebHook，配置变更触发server端refresh
2. Server端接收到请求并发送给Spring Cloud Bus
3. Spring Cloud Bus接到消息并通知给客户端
4. 客户端接收到通知，请求Server端获取最新配置

# 3、Apollo
Apollo： 携程开源的配置管理中心，具备规范的权限、流程治理等特性。能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

- Apollo有 单独的管理界面，并且不用整合gitee/gitlab，配置简单。
- 用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序。
- 支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例。
- Apollo自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。
- 配置中心作为基础服务，可用性要求非常高，这就要求Apollo对外部依赖尽可能地少，目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来，Apollo还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数。



# 4、Eureka，Apollo，Spring Cloud Config三者对比
![](https://pics2.baidu.com/feed/55e736d12f2eb938e919ed1b2e15f133e4dd6f65.jpeg?token=a7d3087ca0c954a2131754f345984771)


## 4.1 Nacos vs Spring Cloud
- Nacos = Spring Cloud Eureka + Spring Cloud Config
- Nacos 可以与 Spring, Spring Boot, Spring Cloud 集成，并能代替 Spring Cloud Eureka, Spring Cloud Config。
  1. 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-config 实现配置的动态变更。
  2. 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现。

## 4.2 Apollo 与 Nacos 功能对比
- nacos配置文件支持比较多的格式，支持yaml、text、json、xml、html、Properties，apollo只支持xml、text、Properties的格式，没有兼容springboot中比较通用的yaml配置。
- apollo用户管理以及权限管理做的比较好和全面，适合做部门或者公司级的配置中心。nacos比较简洁明了（也可以说没有做权限这一块的开发），适合做小组内，或者小型java团体使用。
- apollo区分多环境是直接通过环境指定，可以直接对比和切换，而nacos是通过命名空间进行区分的。
- nacos是支持多格式的配置文件，但是解析上没有apollo做的好，apollo虽然支持的配置格式较少，不过会进行解析，使每个配置看起来比较直观，修改的时候比较直观，可以对单个进行修改。

![](https://img-blog.csdnimg.cn/20200102104318659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NyZWVwaW5f,size_16,color_FFFFFF,t_70)

### 4.2.1 Nacos与Apollo对比结论
1. Nacos部署简化，Nacos整合了注册中心、配置中心功能，且部署相比apollo简单，方便管理和监控。
2. apollo容器化较困难，Nacos有官网的镜像可以直接部署，总体来说，Nacos比apollo更符合KISS原则
3. 性能方面，Nacos读写tps比apollo稍强一些

## 4.3 Nacos与Eureka注册中心对比
![](https://img-blog.csdnimg.cn/20200102103624430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NyZWVwaW5f,size_16,color_FFFFFF,t_70)

**阿里的nacos : 性能最好**

1. 他同时支持AP和CP模式,他根据服务注册选择临时和永久来决定走AP模式还是CP模式,
2. 他这里支持CP模式对于我的理解来说,应该是为了配置中心集群,因为nacos可以同时作为注册中心和配置中心,
3. 因为他的配置中心信息是保存在nacos里面的,假如因为nacos其中一台挂掉后,还没有同步配置信息,就可能发生配置不一致的情况.,
4. 配置中心的配置变更是服务端有监听器,配置中心发生配置变化,然后服务端会监听到配置发生变化,从而做出改变

**eureka+spring cloud config:**

1. 性能也不差,对于服务数量小于上千台来说,性能没有问题
2. eureka: 可以做注册中心,完全AP,支持注册中心之间的节点复制,同时支持服务端同时注册多个注册中心节点, 所以不存节点信息不一致的情况
3. config: 单独服务,是从git仓库拉取配置信息,然后服务端从config服务里面拉取配置信息缓存到本地仓库， 这里配置的变更比较麻烦,他需要结合bus组件,同时约束了只能用rabbitmq和kafka来进行通知服务端进行配置变更。
4. 但是保证了数据的一致性,因为他的配置信息在git仓库上,git仓库只有一个,就会数据一致

### 4.3.1 Nacos与Eureka对比结论
1. Nacos具备服务优雅上下线和流量管理（API+后台管理页面），而Eureka的后台页面仅供展示，需要使用api操作上下线且不具备流量管理功能。
2. 从部署来看，Nacos整合了注册中心、配置中心功能，把原来两套集群整合成一套，简化了部署维护
3. 从长远来看，Eureka开源工作已停止，后续不再有更新和维护，而Nacos在以后的版本会支持SpringCLoud+Kubernetes的组合，填补 2 者的鸿沟，在两套体系下可以采用同一套服务发现和配置管理的解决方案，这将大大的简化使用和维护的成本。同时来说,Nacos 计划实现 Service Mesh，是未来微服务的趋势
4. 从伸缩性和扩展性来看Nacos支持跨注册中心同步，而Eureka不支持，且在伸缩扩容方面，Nacos比Eureka更优（nacos支持大数量级的集群）。
5. Nacos具有分组隔离功能，一套Nacos集群可以支撑多项目、多环境。

