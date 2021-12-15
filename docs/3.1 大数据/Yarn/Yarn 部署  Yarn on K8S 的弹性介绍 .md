# Yarn 部署 | Yarn on K8S 的弹性介绍

原文地址：https://mp.weixin.qq.com/s/fXvcD0anjKUKj-icRiwaxw



# 一、背景介绍

## 1.1 为什么要使用 Yarn on K8S

- 作为在离线混部方案
- 充分利用在离线计算资源
- 不同集群计算资源共享，缓解“潮汐现象”
- 推进云原生方案快速落地

# 二、演进思路

## 2.1 阶段1：简单部署

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9ABGUXFZvd0zqia05Z12nWxkuxPmS4et330AP1b6MtHej7d6WJWuCsicibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

***局限性：***

- NMPod 挂载固定盘
- NM 资源固定
- 规则固定
- 人工维护成本高

## 2.2 阶段2：节点资源感知

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9AtH2QA5ib6qLNYId4icibbPic3gFfDCSic7zibXM8P8Pic1oeV5PKV1bqNANVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- NM 支持弹性资源
- 主动驱逐 Container
- RM 调度优化



- 节点资源感知-- 扩容

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9AlgciadZmcGrSq9UQh5qEBge90hXzoo3JFREf3Fn8QblD7Sm87HLic0lg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

NM 通过 list&watch 机制，获取节点可用资源，并且汇报 RM，从而实现动态扩缩容以及资源超发

- 节点资源感知-缩容

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9ApW199HONBYBneDgrbuGxuFpuPME5tTEhQDWp1DEX1tb9PpMolVZMZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.3 阶段3：存算分离

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9AkPticrOxOVdianOC3pJPtRR4Y6TCvDHHIjCv5NCdCvzqKn7wicP2MUxjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- Spark native on k8s：k8s 调度 spark driver 和 executor pod，使用 RSS 支持存算分离
- Yarn on k8s NM 存算分离：支持 Tez on RSS

## 2.4 阶段4：灵活的集群弹性伸缩

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIdSDhepu5J8W2C20sP1jV9AibdnXClRgSiazkKty2wSuCKzAfTy3ia0oQ27yg7BE1sibxaHpbyN7fCLjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

支持弹性伸缩：动态感知集群负载

# 三、总结和展望

**总结**

- 打通 K8S 节点感知和 yarn 资源动态上报，以解决节点资源使用冲突，平衡集群内的节点资源使用

- 提供 RSS 存算分离服务解决 K8S 调度节点本地盘依赖问题，更好的支持计算引擎层 native云原生

- 通过单独的 auto scaler 服务，提供集群资源横向动态扩缩容，灵活的分时错峰调度能力

  

**展望**

- 在 K8S 的基础上提供更完善的调度策略，如多级队列
- 使用 Node label 机制为不同级别的在线任务提供资源和集群扩缩容服务
- 改进 Yarn RM 在扩缩容场景下遇到的调度性能稳定性问题