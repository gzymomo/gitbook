来源：

- 微信公众号：aoho求索，作者aoho
- 原文地址：[Serverless 架构到底要不要服务器？](https://mp.weixin.qq.com/s/X7fP-2XqaBhcAd_XU8Tpww#)



# Serverless 架构到底要不要服务器?



# Serverless 是什么

Serverless 架构是不是就不要服务器了？回答这个问题，我们需要了解下 Serverless 是什么。 



Serverless 架构近几年频繁出现在一些技术架构大会的演讲标题中，很多人对于  Serverless，只是从字面意义上理解——无服务器架构，但是它真正的含义是开发者再也不用过多考虑服务器的问题，当然，这并不代表完全去除服务器，而是我们依靠第三方资源服务器后端，从 2014 年开始，经过这么多年的发展，各大云服务商基本都提供了 Serverless 服务。比如使用 Amazon Web  Services(AWS) Lambda 计算服务来执行代码。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1vy3IZRqFFjxicbyAzFVc2wAY00ib8pQmRoe5ZmaPlXXRIiba3ZUILcqvUM6mfhiaIjPqarPRUkQl9nSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

国内 Serverless 服务的发展相对 AWS 要晚一点，目前也都有对 Serverless 的支持。比较著名的云服务商有阿里云、腾讯云。它们提供的服务也大同小异：函数计算、对象存储、API 网关等，非常容易上手。



# 架构是如何演进到 Serverless ？



看看过去几十年间，云计算领域的发展演进历程。总的来说，云计算的发展分为三个阶段：虚拟化的出现、虚拟化在云计算中的应用以及容器化的出现。云计算的高速发展，则集中在近十几年。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1vy3IZRqFFjxicbyAzFVc2wAD7P3LH1y1CRa2uyJKBOoniccRyO7WBJ49nFGyVpiadGV2oeMBRhc1c5Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



总结来说有如下的里程碑事件：

- 通过虚拟化技术将大型物理机虚拟成单个的 VM 资源。
- 将虚拟化集群搬到云计算平台上，只做简单运维。
- 把每一个 VM 按照运行空间最小化的原则切分成更细的 Docker 容器。
- 基于 Docker 容器构建不用管理任何运行环境、仅需编写核心代码的 Serverless 架构。



从裸金属机器的部署应用，到 Openstack 架构和虚拟机的划分，再到容器化部署，这其中典型的就是近些年 Docker 和 Kubernetes 的流行，进一步发展为使用一个微服务或微功能来响应一个客户端的请求 ，这种方式是云计算发展的自然过程。



这个发展历程也是一场 IT 架构的演进，期间经历了一系列代际的技术变革，把资源切分得更细，让运行效率更高，让硬件软件维护更简单。IT 架构的演进主要有以下几个特点：

- 硬件资源使用颗粒度变小
- 资源利用率越来越高
- 运维工作逐步减少
- 业务更聚焦在代码层面



## 1. Serverless 架构的组成

Serverless架构分为 Backend as a Service(BaaS) 和 Functions as a Service(FaaS)  两种技术，Serverless 是由开发者实现的服务端逻辑运行在无状态的计算容器中，它是由事件触发、完全被第三方管理的。



## 2. 什么是 BaaS?

Baas 的英文翻译成中文的含义：后端即服务，它的应用架构由大量第三方云服务器和 API  组成，使应用中关于服务器的逻辑和状态都由服务提供方来管理。比如我们的典型的单页应用 SPA 和移动 APP 富客户端应用，前后端交互主要是以  RestAPI 调用为主。只需要调用服务提供方的 API 即可完成相应的功能，比如常见的身份验证、云端数据/文件存储、消息推送、应用数据分析等。



## 3. 什么是 FaaS?

FaaS 可以被叫做：函数即服务。开发者可以直接将服务业务逻辑代码部署，运行在第三方提供的无状态计算容器中，开发者只需要编写业务代码即可，无需关注服务器，并且代码的执行是由事件触发的。其中 AWS Lambda 是目前最佳的 FaaS 实现之一。



Serverless 的应用架构是将 BaaS 和 FaaS 组合在一起的应用，用户只需要关注应用的业务逻辑代码，编写函数为粒度将其运行在 FaaS 平台上，并且和 BaaS 第三方服务整合在一起，最后就搭建了一个完整的系统。整个系统过程中完全无需关注服务器。



# Serverless 架构的特点

总得来说，Serverless 架构主要有以下特点：

- 实现了细粒度的计算资源分配
- 不需要预先分配资源
- 具备真正意义上的高度扩容和弹性
- 按需使用，按需计费



由于 Serverless 应用与服务器的解耦，购买的是云服务商的资源，使得 Serverless 架构降低了运维的压力，也无需进行服务器硬件等预估和购买。



Serverless 架构使得开发人员更加专注于业务服务的实现，中间件和硬件服务器资源都托管给了云服务商。这同时降低了开发成本，按需扩展和计费，无需考虑基础设施。



Serverless 架构给前端也带来了便利，大前端深入到业务端的成本降低，开发者只需要关注业务逻辑，前端工程师轻松转为全栈工程师。



# Serverless 有哪些应用场景？

应用场景与 Serverless 架构的特点密切相关，根据 Serverless 的这些通用特点，我们归纳出下面几种典型使用场景：弹性伸缩、大数据分析、事件触发等。



## 1. 弹性伸缩

由于云函数事件驱动及单事件处理的特性，云函数通过自动的伸缩来支持业务的高并发。针对业务的实际事件或请求数，云函数自动弹性合适的处理实例来承载实际业务量。在没有事件或请求时，无运行实例，不占用资源。如视频直播服务，直播观众不固定，需要考虑适度的并发和弹性。直播不可能 24 小时在线，有较为明显的业务访问高峰期和低谷期。直播是事件或者公众点爆的场景，更新速度较快，版本迭代较快，需要快速完成对新热点的技术升级。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1vy3IZRqFFjxicbyAzFVc2wAF8LqYibWXz7Sa8lHBx41j3jz8ZiaMozo9xLIwJlauoz5LtheELZbxviaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 2. 大数据分析

数据统计本身只需要很少的计算量，离线计算生成图表。在空闲的时候对数据进行处理，或者不需要考虑任何延时的情况下。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1vy3IZRqFFjxicbyAzFVc2wATEGick2nqaQ8licxxDnVFCzYEYRfQOmGYSI6sKf0f3cYZddLsXBBERGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



- 开发者编写代码，目前支持的语言 Java、NodeJS、Python 等语言；
- 把代码上传到函数计算上，上传的方式有通过 API 或者 SDK 上传，也可以通过控制台页面上传，还可以通过命令行工具 Fcli 上传；
- 通过 API&SDK 来触发函数计算执行，同样也可以通过云产品的事件源来触发函数计算执行；
- 函数计算在执行过程中，会根据用户请请求量动态扩容函数计算来保证请求峰值的执行，这个过程对用户是透明无感知的；
- 函数执行结束。

## 3. 事件触发

事件触发即云函数由事件驱动，事件的定义可以是指定的 http 请求，或者数据库的 binlog 日志、消息推送等。通过 Serverless  架构，在控制台上配置事件源通知，编写业务代码。业务逻辑添加到到函数计算里，业务高峰期函数计算会动态伸缩，这个过程不需要管理软硬件环境。常见的场景如视频、OSS 图片，当上传之后，通过进行后续的过滤、转换和分析，触发一系列的后续处理，如内容不合法、容量告警等。



# 小结

Serverless 架构不是不要服务器了，而是依托第三方云服务平台，服务端逻辑运行在无状态的计算容器中，其业务层面的状态则被开发者使用的数据库和存储资源所记录。



Serverless 无服务器架构有其适合应用的场景，但是也存在局限性。总得来说，Serverless 架构还不够成熟，很多地方尚不完善。Serverless  依赖云服务商提供的基础设施，目前来说云服务商还做不到真正的平台高可用。Serverless  资源虽然便宜，但是构建一个生产环境的应用系统却比较复杂。



云计算还在不断发展，基础设施服务日趋完善，开发者将会更加专注于业务逻辑的实现。云计算将平台、中间件、运维部署的责任进行了转移，同时也降低了中小企业上云的成本。让我们一起期待 Serverless 架构的未来。



参考：

1. 阿里云文档
2. *https://blog.csdn.net/cc18868876837/article/details/90672971*



# Serverless Devs

Serverless Devs 是一个开源开放的 Serverless 开发者平台，致力于为开发者提供强大的工具链体系。通过该平台，开发者可以一键体验多云 Serverless 产品，极速部署 Serverless 项目。

- Github 地址：

  *https://github.com/serverless-devs*

- Gitee 地址：

  *https://gitee.com/organizations/serverless-devs/projects*

- Serverless Devs 官网：

  *https://www.serverless-devs.com*