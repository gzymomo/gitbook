- [Apache Openwhisk学习（一）](https://www.cnblogs.com/junjiang3/p/9613698.html)

# 一、OpenWhisk概述

Github项目地址：https://github.com/apache/openwhisk

OpenWhisk 是属于 Apache 基金会的开源 FaaS 计算平台[官网链接](https://openwhisk.apache.org/), 由 IBM 在2016年公布并贡献给开源社区（[github页面](https://github.com/apache/incubator-openwhisk)），IBM Cloud 本身也提供完全托管的 OpenWhisk FaaS服务 [IBM Cloud Function](https://console.bluemix.net/openwhisk/)。从业务逻辑上看，OpenWhisk 同 AWS Lambda 一样，为用户提供基于事件驱动的无状态的计算模型，并直接支持多种编程语言（理论上可以将任何语言的 runtime 打包上传，间接调用）。

OpenWhisk 是一个由 IBM 开源的、事件驱动的无服务器计算平台，你可以将操作代码发送给 OpenWhisk，然后提供  OpenWhisk 代码要处理的数据流。OpenWhisk  负责处理计算资源的扩展，这些资源是处理工作负载所需要的；你只需要处理操作代码以及触发这些操作的数据。

OpenWhisk  简化了微服务的部署，消除了管理自己的消息代理或部署自己的工作服务器的需求。OpenWhisk  适用于你不希望管理任何基础架构的项目，只需为已完成的工作付费，不会将金钱浪费在空闲的服务器上。OpenWhisk  很容易管理活动峰值，因为它可以外扩来满足该需求。

由于运行 OpenWhisk 操作需要资源，所以最好使用 OpenWhisk 执行以下不是很频繁的计算任务，比如：

1. 处理上传的图像来创建缩略图，将它们保存到对象存储
2. 从移动应用程序获取地理位置数据，并调用 Weather API 来扩充它

OpenWhisk 对处理具有很高的并发性水平的系统也很有用，比如：

1. 将数据发送到云的移动应用程序
2. 物联网部署，其中需要存储和处理传入的传感器数据

![img](https://images2018.cnblogs.com/blog/1159663/201809/1159663-20180909134432252-860623606.png)

Openwhisk的特点：

（1）高性能、高扩展性的分布式Faas计算平台

（2）函数的代码及运行是全部在Docker容器中进行，利用Docker引擎实现Faas函数运行的管理、负载均衡和扩展

（3）同时，Openwhisk架构中的所有其他组件（API网关、控制器、触发器）也全部运行在Docker容器中，这使得其全栈可以比较容易的部署在IAAS/PAAS平台上 。

（4）更重要的是，相比其他Faas实现，Openwhisk更像是一套完整Serverless解决方案，除了容易调用和函数管理，Openwhisk还包括了身份验证/鉴权、函数异步触发等功能。

## 1.1 Openwhisk架构

![openwhisk架构图](https://img-blog.csdnimg.cn/20190819113048968.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTUwMjI5NA==,size_16,color_FFFFFF,t_70)

openwhisk是一个事件驱动的计算平台，也被用在serveless和fass领域，来响应事件调用。事件通常包括：

- 数据库记录的修改
- IoT传感器数据上传
- GitHub代码仓库的新提交
- HTTP调用

这些内部或者外部的事件通过trigger和rules，最终到达actions来响应这些事件。
Actions 通常是一些代码片段, 或者是在一个 Docker container里的二进制文件。Actions在OpenWhisk里

通常在触发器触发的时候才会被调用。通过OpenWhisk API, CLI, or iOS SDK，也可以绕过trigger直接调用action。Action还可以组成Chain被顺序调用。

开发者可以用packages来实现与外部服务和事件的对接。Package是一些 feeds的组合。Feed是一段外部事件引发触发器的代码。

# 二、系统概览

 Openwhisk是一个事件驱动的计算平台，它运行代码以响应事件或直接调用，下图显示了其 体系结构：

![img](https://images2018.cnblogs.com/blog/1159663/201809/1159663-20180909135738185-1135708669.png)

在上述架构中，代码时基于事件（Event）触发的。事件产生于事件源（feed），而可以用于触发函数的事件源可以是多种多样的，例如数据库的更改，超过特定温度的IoT传感器读数，新提交代码到Github存储库等。事件与对应的函数代码，通过规则（Rule）绑定。通过匹配事件对应的规则，Openswhisk会触发对应的行为（Action）。值得注意的是，多个Action可以串联，完成复杂的操作。

# 三、Openwhisk的工作原理

作为一个开源项目，openwhisk集成了很多其他组件Nginx，Kafka，Docker，CouchDB等，为了更加详细的解释所有组件，我们可以使用下面这个例子，来追踪整个系统的调用过程。

假设我们有包含以下代码块的文件action.js：

```js
function main() {
    console.log('Hello World');
    return { hello: 'world' };
}
```

使用下面命令创建动作

```bash
wsk action create myAction action.js
```

现在，我们可以使用以下命令调用该操作：

```
wsk action create myAction action.js
```

那么我现在来看看在Openwhisk中具体发生了什么，整个过程如下图所示：

![img](https://images2018.cnblogs.com/blog/1159663/201809/1159663-20180909142023086-469618209.png)

## （1）Nginx

Openwhisk面向用户的API完全基于HTTP，遵循Restful设计，因此，通过wsk-cli发送的命令本质上是向其发送HTTP请求，上面的命令大致可以翻译为：

```bash
POST /api/v1/namespaces/$userNamespace/actions/myAction
Host: $openwhiskEndpoint
```

此处的Nginx主要用于接受Http请求，并将处理后的Http请求直接转发给controller

## （2）控制器（Controller）

控制器是真正开始处理请求的地方。控制器使用Scala语言实现，并提供了对应的Rest API，接受Nginx转发的请求。Controller分析请求内容，进行下一步处理。下面的很多个步骤都会和其有关系。

## （3）身份验证和鉴权：CouchDB

继续用上一步用户发出的Post请求为例，控制器首先需要验证用户的身份和权限。用户的身份信息（credentials）保存在CouchDB的用户身份数据库中，验证无误后，控制器进行下一步处理。

## （4）再次CouchDB，得到对应的Action的代码及配置

身份验证通过后，Controller需要从CouchDB中加载此操作（在本例中为myAction）。操作记录主要要执行的代码和要传递给操作的默认参数，并与实际调用请求中包含的参数合并。它还包含执行时对其施加的资源限制，例如允许使用的内存。

## （5）Consul和负载均衡

到了这一步，控制器已经有了触发函数所需要的全部信息，在将数据发送给触发器（Invoker）之前，控制器需要和 Consul 确认，从  Consul 获取处于空闲状态的触发器的地址。Consul 是一个开源的服务注册/发现系统，在 OpenWhisk 中 Consul  负责记录跟踪所有触发器的状态信息。当控制器向 Consul 发出请求，Consul  从后台随机选取一个空闲的触发器信息，并返回。值得注意的是：无论是同步还是异步触发模式，控制器都不会直接调用触发器API，所有触发请求都会通过  Kafka 传递。

## （6）发送请求进Kafka

考虑使用kafka主要是担心发生以下两种状况：

1、系统崩溃，丢失调用请求

2、系统可能处于繁重的负载之下，调用需要等待其它调用首先完成。

Openwhisk考虑到异步情况的发生，考虑异步触发的情况，当控制器得到 Kafka  收到请求消息的的确认后，会直接向发出请求的用户返回一个 ActivationId，当用户收到确认的  ActivationId，即可认为请求已经成功存入到 Kafka 队列中。用户可以稍后通过 ActivationId 索取函数运行的结果。

## （7）Invoker运行用户的代码

Invoker从对应的Kafka topic中接受控制器传来的请求，会生成一个Docker容器，注入动作代码，试用传递给他的参数执行它，获取结果，消灭容器。这也是进行了大量性能优化以减少开销并缩短响应时间的地方。

## （8）CouchDB存储请求结果

Invoker的执行结果最终会被保存在CouchDB的whisk数据库中，格式如下所示：

```json
{
   "activationId": "31809ddca6f64cfc9de2937ebd44fbb9",
   "response": {
       "statusCode": 0,
       "result": {
           "hello": "world"
       }
   },
   "end": 1474459415621,
   "logs": [
       "2016-09-21T12:03:35.619234386Z stdout: Hello World"
   ],
   "start": 1474459415595,
}
```

保存的结果中包括用户函数的返回值，及日志记录。对异步触发用户，可以通过步骤6中返回的 activationID  取回函数运行结果。同步触发的的结果和异步触发一样保存在 CouchDB 里，控制器在得到触发结束的确认后，从 CouchDB  中取得运行结果，直接返回给用户。

 

本文主要参考了：

（1）https://github.com/apache/incubator-openwhisk/blob/master/docs/about.md

（2）https://blog.xinkuo.me/post/apache-openwhisk.html#apache-openwhisk%E7%AE%80%E4%BB%8B