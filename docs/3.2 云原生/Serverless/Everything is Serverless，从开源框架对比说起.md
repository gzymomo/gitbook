[Everything is Serverless，从开源框架对比说起](https://www.cnblogs.com/huaweiyun/p/14516489.html)

在众多云计算解决方案中，**Serverless** 逐渐崭露头角，受到了很多关注并且发展迅猛，今天就关于serverless 开源框架细说二三。

## 什么是serverless computing

- serverless computing = FaaS (Function as a Service) + BaaS (Backedn as a Service)
- serverless是云原生应用的业务需求，是云计算形态的进一步发展，是云计算的下一代计算范式，Everything is Serverless

## 无服务器和传统云计算之间的三个基本区别是：

- 解耦计算和存储；它们分别缩放并独立定价, 通常存储由独立服务提供，计算是无状态的
- 执行一段代码而不是分配执行代码的资源的抽象。用户提供一段代码，云端自动配置资源来执行代码(NoOPS，传统云计算是devops)
- 支付代码执行费用（Pay as you Run, 传统云计算是Pay as You Use），而不是支付为执行代码分配的资源。比如按执行时间计费，而不是按分配的虚机大小数量计费

## Serverless 典型产品

![img](https://pic3.zhimg.com/80/v2-962107ea121cc0ff4c7bb5c70aed020a_720w.jpg)

## 函数服务主要开源项目

![img](https://pic4.zhimg.com/80/v2-d597c6d8f7a7b21e2c4ad9a6005efb33_720w.jpg)

## 开源项目对比

### ServerLess 框架比较

![img](https://pic1.zhimg.com/80/v2-43ce215866b44a547284ba3d617df430_720w.jpg)

### 使用场景

![img](https://pic3.zhimg.com/80/v2-2d7e32c9f5525e300be712b6bfe69e2e_720w.jpg)

## 架构

以AWS为例

![img](https://pic4.zhimg.com/80/v2-240bf6b06e478f2182fd2dbb5ca2559b_720w.jpg)

## 两条支持异构硬件的路径

- Serverless 包含多种实例类型，不同的硬件使用不同的价格
- 提供商自动选择基于语言的加速器和DSA（Domain Specific  Architecture），比如GPU硬件用于CUDA代码，TPU硬件用于TensorFlow代码（对于python或者js等高级语言，软硬件co-design提供language specific 处理器； 对于编译型语言，编译器应该建议使用何种硬件架构）

## 当前技术局限

![img](https://pic1.zhimg.com/80/v2-e316500eb4d6712786eac21a620e18a8_720w.jpg)

## 挑战

- 计算抽象（屏蔽计算资源，解决数据依赖）
- 系统使能（函数状态的高速存储，函数间高速信令，函数极速启动）
- 安全性 (应用级隔离，分布式安全)
- 适应性 （异构硬件使能，微服务演进）
- 成本不可以预测： 需要提供成本预测能力
- 容易产生Vendor lock-in: 需要提供API标准规范，类似POSIX为操作系统做的事情，Google的Knative project在向这个方向努力