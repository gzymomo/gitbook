- [当微服务遇上Serverless：解读云原生微服务最佳实践](https://mp.weixin.qq.com/s/EDWqMq8SBnIRjWO73aOQgw)

# 背景

微服务作为一种更灵活、可靠、开放的架构，近年来得到迅速发展，和容器技术的结合可以轻松实现微服务化后的 DevOps，越来越多的企业寻求微服务容器化落地之道来让企业应用更好的上云。然而因 K8s  本身的学习曲线、运维复杂度、适配微服务的服务注册发现、版本管理、灰度策略，已有会话处理等，让这些客户望而却步，爱而不得。

阿里云 Serverless  应用引擎（SAE）就是在这个背景下诞生的，初衷是让客户不改任何代码，不改变应用部署方式，就可以享受到微服务+K8s+Serverless  的完整体验，开箱即用免运维。底层基于统一的 K8s 底座，帮用户屏蔽 IaaS 和 K8s 集群运维，WAR/JAR/PHP zip  包无需容器化改造直接部署。在应用层，给用户提供了全栈的能力，重点包括应用管理和微服务治理。在开发者工具/SaaS 方面也做了良好的集成，可以说  SAE 覆盖了应用上云的完整场景。

![图片](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5IiafvnHrmdkzF00HkvyI4oiaodeyiaCkjxsdFvynBXtiajwe1IzatpQOo4DoCYCfJ8IPVyrHCHicvu6BybODQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 微服务治理能力业界领先

SAE 深度集成了微服务引擎（MSE），将阿里深耕十余年历经双 11 考验的微服务最佳实践产品化。在开源 Spring Cloud/Dubbo  的基础上，提供了更多免费的高级治理能力。如微服务金丝雀/灰度流量能力，能让应用发新版时，基于 header/cookie  等各种纬度进行精准灰度，控制最小爆炸半径；微服务的无损下线和无损上线能力，能在 Provider 升级过程中，通过 SAE 应用内挂载的  agent 主动刷新服务列表和主动通知，Consumer  不会出现调用报错。服务启动过程中，无论发布/扩容都实现流量平滑和稳定。还有杀手锏的全链路灰度能力，能实现从七层入口流量到后端一系列微服务的级联流量灰度，极大的降低了客户多套环境搭建成本，提升灰度效果。

![图片](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5IiafvnHrmdkzF00HkvyI4oiaodeyW78CibdqC51R3xCW5cMfNokcaheLf8C2TuAeKVibqjP2XzicpBf5tkhcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# SAE 突破 Java 冷启动瓶颈

Java 冷启动效率慢一直是困扰开发者多年的难题，加载的类较多，依赖包大，会严重拖慢效率。SAE  除了镜像加速、镜像预热效率优化手段外，也在极力打造极致的 Java 应用启动效率：基于 Alibaba Dragonwell 11 增强的  AppCDS 启动加速技术，将应用第一次启动的过程生成缓存保存起来，后续直接通过缓存启动应用。同比标准的 OpenJDK，在冷启动耗时场景下提升 40%，极大提升了应用启动和弹性效率 。该项技术已大范围应用于集团生产业务，也收到了多数企业用户的频频点赞。

# SAE 业界首发混合弹性策略

SAE 提供了业界最丰富的弹性指标，最灵活的弹性策略。不同的场景使用不同的弹性策略。除 K8s 标准提供的 cpu/mem 外，SAE  新增支持应用监控指标如 QPS、RT、TCP 连接数等，基于业务来弹更精准。除定时弹性和监控指标自动弹性外，SAE  新增支持混合弹性策略，解决了在线教育、互娱、文化传媒等行业中定时弹性和监控弹性互斥，不能同时启用的痛点问题，并且在手工干预扩容后，还能系统恢复自动弹性能力。

# SAE 提供面向大促的高可用解决方案

Serverless 应用引擎（SAE）尤其适用于电商、新零售、互娱、在线教育、餐饮、出行、文化传媒等时有突发流量的行业，能做到精准容量+极致弹性+限流降级。

![图片](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5IiafvnHrmdkzF00HkvyI4oiaodeyIxVgBiaK9BajcuMAs1GHXr0X6TLh6zuy4RGO8hIbISVTUUPiaOIzmRZA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

有人说微服务跑在 Serverless 上是异想天开，也有人说 Serverless 与微服务是天作之合，相信随着  Serverless应用引擎(SAE)这款产品的不断发展，这类争论会逐渐地消失，而 SAE 也会成为微服务容器化的最短路径和微服务 on  Serverless 的最佳实践。

