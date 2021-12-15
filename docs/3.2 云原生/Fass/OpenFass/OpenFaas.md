- [openfaas/faas 环境搭建和开发使用](https://www.jianshu.com/p/7d11d9e8792a)
- [OpenFaaS概览](https://zhuanlan.zhihu.com/p/40641182)

# 一、OpenFaas概述

[OpenFaaS](https://github.com/openfaas/faas)一款高人气的开源的faas框架，可以直接在Kubernetes上运行，也可以基于Swarm或容器运行。

> 无服务器函数变得简单。

![img](https://pic1.zhimg.com/80/v2-a178741a752cb2ad7b135fee42549194_720w.jpg)

## 1.1 函数看门狗

- 你可以通过添加*函数看门狗* (一个小型的Golang HTTP服务)把任何一个Docker镜像变成无服务器函数。
- *函数看门狗*是允许HTTP请求通过STDIN转发到目标进程的入口点。响应会从你应用写入STDOUT返回给调用者。

## 1.2 API网关/UI门户

- API网关为你的函数提供外部路由，并通过Prometheus收集云原生指标。
- 你的API网关将会根据需求更改Docker Swarm 或 Kubernetes API中的服务副本数来实现伸缩性。
- UI是允许你在浏览器中调用函数或者根据需要创建新的函数。

> API网关是一个RESTful形式的微服务，你可以在这里查看[Swagger文档](https://link.zhihu.com/?target=https%3A//github.com/openfaas/faas/tree/master/api-docs)。

## 1.3 命令行

Docker中的任何容器或者进程都可以是FaaS中的一个无服务器函数。使用[FaaS CLI](https://link.zhihu.com/?target=http%3A//github.com/openfaas/faas-cli) ，你可以快速的部署函数。

可以从Node.js, Python, [Go](https://link.zhihu.com/?target=https%3A//blog.alexellis.io/serverless-golang-with-openfaas/) 或者更多的语言模板中创建新的函数。如果你无法找到一个合适的模板，甚至可以使用一个Dockerfile。

> CLI实际上是API网关的一个RESTful客户端。

## 1.4 函数示例

你可以通过 使用FaaS-CLI和其内置的模板创建新函数，也可以在Docker中使用Windows或Linux的二进制文件。

- Python示例：

```python
import requests

def handle(req):
    r =  requests.get(req, timeout = 1)
    print(req +" => " + str(r.status_code))
```

*handler.py*

- Node.js示例：

```js
"use strict"

module.exports = (callback, context) => {
    callback(null, {"message": "You said: " + context})
}
```
