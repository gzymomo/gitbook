## **微服务平台架构设计**

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQ2NWvn24OvTn0Yj6lozuK1sD6IX3o07jqZnDAGblQBSFARMAaXGPMoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **云原生应用开发和部署的四大原则**

云原生应用所构建和运行的应用，旨在充分利用基于四大原则的云计算模型。


![img](https://mmbiz.qpic.cn/mmbiz/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQMDdcAVw4DUhmv2BBsAXIpe30TExFiaKYX26dsIFiatDJcgT7rEh8xQbg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


云原生应用开发所构建和运行的应用，旨在充分利用基于四大原则的云计算模型：

- 基于服务的架构：基于服务的架构（如微服务）提倡构建松散耦合的模块化服务。采用基于服务的松散耦合设计，可帮助企业提高应用创建速度，降低复杂性。
- 基于API 的通信：即通过轻量级 API 来进行服务之间的相互调用。通过API驱动的方式，企业可以通过所提供的API 在内部和外部创建新的业务功能，极大提升了业务的灵活性。此外，采用基于API 的设计，在调用服务时可避免因直接链接、共享内存模型或直接读取数据存储而带来的风险。
- 基于容器的基础架构：云原生应用依靠容器来构建跨技术环境的通用运行模型，并在不同的环境和基础架构（包括公有云、私有云和混合云）间实现真正的应用可移植性。此外，容器平台有助于实现云原生应用的弹性扩展。
- 基于DevOps流程：采用云原生方案时，企业会使用敏捷的方法、依据持续交付和DevOps 原则来开发应用。这些方法和原则要求开发、质量保证、安全、IT运维团队以及交付过程中所涉及的其他团队以协作方式构建和交付应用。

# **应用层微服务改造注意事项**

- 应用框架推荐使用

- - 微服务框架：spring boot、serviceComb（java、go）、service mesh (istio)
  - 东西流量使用HTTP
  - 提供应用的健康检查API：/api/v1/checkhealth

- 服务发现方法

- - 推荐使用 k8s 原生service 网络
  - 不推荐使用 spring cloud Euraka

- 应用配置管理

- - 建议使用配置中心Apollo
  - 不建议使用本地文件profile

- 应用无状态，支持横向扩容

  
  ![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQI0icqCXQ5jDibZUyialNicGE64z5Byu2LyjHWQATZACeHSibyGRFibeichicGw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

## **基础服务平台介绍**

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQZWHQibJc2EZ8cJOuvLxw6FklrFC9V9HNtv1sLsY0YzmWnPnyrBY370Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- JenkinsCI/CD
- prometheus监控系统
- ELK 日志系统
- jumpserver堡垒机

## jenkins CI/CD 流程

### 视频演示

- Jenkins 用户权限管理
- Jenkins pipeline 全流程自动化发布
- ansible 自动化发布ingress 资源

### Jenkins CI/CD 流程：

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQiaTWFZ6aCfG0nX33gp43bQiaWnZs3jz7l4UsNvrt0ibJ0XFcyPLOiapFDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. pull代码
2. 测试registry，并登陆registry.
3. 编写应用 Dockerfile
4. 构建打包 Docker 镜像
5. 推送 Docker 镜像到仓库
6. 更改 Deployment YAML 文件中参数
7. 利用 kubectl 工具部署应用
8. 检查应用状态

JenkinsCI/CD实践参考文档

### 应用资源分配规则

```
cat ftc_demo_limitRange.yaml 
apiVersion: v1
kind: LimitRange
metadata:
  name: ftc-demo-limit
  namespace: ftc-demo
spec:
  limits:
  - max :  # 在一个pod 中最高使用资源
      cpu: "2"
      memory: "6Gi"
    min: # 在一个pod中最低使用资源
      cpu: "100m"
      memory: "2Gi"
    type: Pod
  - default:    # 启动一个Container 默认资源规则
      cpu: "1"
      memory: "4Gi"
    defaultRequest:
      cpu: "200m"
      memory: "2Gi"
    max:    #匹配用户手动指定Limits 规则
      cpu: "1"
      memory: "4Gi"
    min:
      cpu: "100m"
      memory: "256Mi"
    type: Container
```

限制规则说明

- CPU 资源可以超限使用，内存不能超限使用。
- Container 资源限制总量必须 <= Pod 资源限制总和
- 在一个pod 中最高使用2 核6G 内存
- 在一个pod中最低使用100m核 2G 内存
- 默认启动一个Container服务 强分配cpu 200m，内存2G。Container 最高使用 1 cpu 4G 内存。
- 如果用户私有指定limits 规则。最高使用1 CPU 4G内存，最低使用 cpu 100m 内存 256Mi

默认创建一个pod 效果如下

```
kubectl describe pod -l app=ftc-saas-service  -n ftc-demo
Controlled By:  ReplicaSet/ftc-saas-service-7f998899c5
Containers:
  ftc-saas-service:
    Container ID:   docker://9f3f1a56e36e1955e6606971e43ee138adab1e2429acc4279d353dcae40e3052
    Image:          dl-harbor.dianrong.com/ftc/ftc-saas-service:6058a7fbc6126332a3dbb682337c0ac68abc555e
    Image ID:       docker-pullable://dl-harbor.dianrong.com/ftc/ftc-saas-service@sha256:a96fa52b691880e8b21fc32fff98d82156b11780c53218d22a973a4ae3d61dd7
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 21 Aug 2018 19:11:29 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     1
      memory:  4Gi
    Requests:
      cpu:      200m
      memory:   2Gi
```

### 生产案列

```
apiVersion: v1
kind: LimitRange
metadata:
  name: jccfc-prod-limit
  namespace: jccfc-prod
spec:
  limits:
  - max: # pod 中所有容器,Limit值和的上限,也是整个pod资源的最大Limit
      cpu: "4000m"
      memory: "8192Mi"
    min: # pod 中所有容器,request的资源总和不能小于min中的值
      cpu: "1000m"
      memory: "2048Mi"
    type: Pod
  - default:  # 默认的资源限制
      cpu: "4000m"
      memory: "4096Mi"
    defaultRequest: # 默认的资源需求
      cpu: "2000m"
      memory: "4096Mi"
    maxLimitRequestRatio:
      cpu: 2
      memory: 2
    type: Container
```

## **监控系统介绍**

- 基础监控：是针运行服务的基础设施的监控，比如容器、虚拟机、物理机等，监控的指标主要有内存的使用率
- 运行时监控：运行时监控主要有 GC 的监控包括 GC 次数、GC 耗时，线程数量的监控等等
- 通用监控：通用监控主要包括对流量和耗时的监控，通过流量的变化趋势可以清晰的了解到服务的流量高峰以及流量的增长情况
- 错误监控：错误监控: 错误监控是服务健康状态的直观体现，主要包括请求返回的错误码，如 HTTP 的错误码 5xx、4xx，熔断、限流等等

### Prometheus Arch

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQNaibCzCRaibBDj3rBHwbunkVTqsw42oMsq30lJtaATbvyjsxP6e0QfeA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQcHp7Vv1JyXgNvKd4kct2a7Qkga4VcPbW9eJbAFrXiaMjToaGTBciaia3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 日志管理系统

### jumpserver 堡垒机使用说明

- admin：root管理员权限
- Developer: 开发这权限

# Devops标准化规范

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQg9fhdySXo9UFfSS5UNd3Q3m8zO7RxSiaQODXXhyrULvlp8JttavWfxQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## **目的** 

为确保锦程业务访问的安全性、统一性和可管理性，制定本管理规范。

- 域名管理规范
- Deployment容器规范
- 日志标准化规范
- 编译和运行其标准
- 项目上线checklist

## 域名管理规范

![img](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8Ke7cGs8eTbQeFuoUY3rGgEn5fwqzRvTwXhgrfgJDd8x89bbM2ylB3ACGskLrj6ewqm9fLhvTqGOKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### k8S service 服务请求地址

请求地址格式

```
http://appid.namespace.svc.cluster.local:port
```

夸NameSpace

```
http://user.jccfc-uat.svc.cluster.local:8080/Seal/
```

同NameSpace访问地址

```
http://user:8080/raWeb/CSHttpServlet
```

## **Deployment 容器规范**

### Deployment

规范脚本在 jenkins 生成

- 应用提供健康检查API：http://cms.corp.jccfc.com/api/v1/checkHealth
- 后端应用端口：8080
- 前端应用端口：80

## **EFK实时日志查询系统**

### 概要：

实时收集和存储各类日志，方便快速查看、分析日志，提供可视化图表、以及基于日志内容监控告警

### 背景：

- 开发可登录服务器查看日志，存在安全隐患
- 查看日志需要专门人员授予不同权限，消耗人力和时间
- 各个项目日志格式自己拟定，维护难度大
- 日志没有滚动压缩，占用磁盘空间
- 日志缺乏分析和监控

### 日志平台架构：

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQJIx8VXicTS0icMBT3QUzIAvzHVbXUNiaPeZhaUGWEZCZrrnSFnP7d40MA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 功能：

- 日志自动发现、采集
- 日志自动解析，生成索引
- 日志自动生命周期管理
- 冷热数据分离
- 可视化图表
- 集群性能监控
- x-pack权限控制

### 容器内应用写入路径：

/volume_logs

### 节点挂在路径

```
/opt/app_logs/{{ appid }}/app.log  // 应用日志

/opt/app_logs/{{ appid }}/api-access.log   // restful接口访问日志

/opt/app_logs/{{ appid }}/rpc-access.log    // rpc接口访问日志
```

### 容器化应用

```
/volume_logs/app.log  

/volume_logs/api-access.log 

/volume_logs/rpc-access.log
```

## **编译和运行期标准**

### JDK Version 推荐使用1.8.0_191

| 版本      | 参数                        | 说明         |
| :-------- | :-------------------------- | :----------- |
| 1.8.0_131 | UseCGroupMemoryLimitForHeap | 堆内存限制   |
| 1.8.0_191 | ActiveProcessorCount        | CPU 核数限制 |

### maven && nexus

| 版本  | 备注     |
| :---- | :------- |
| 3.6.1 | 推荐版本 |

后端应用仓库 :https://nexus.corp.jccfc.com/repository/maven-public/

- sit: 使用snap 仓库
- uat: 使用release 仓库
- prod: 使用release 仓库

前端应用仓库地址：https://nexus.corp.jccfc.com/repository/npm_group/

## **项目上线checklist**

### **目的**

通过标准化部署要求，提升DevOps与开发团队部署应用效率。保障项目上线稳定性。
![img](https://mmbiz.qpic.cn/mmbiz_gif/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQUMxc5VBtVaMb5ZVhTIC3tzu9icSgSgJPy8cJmB53Qx8XYW0rcx4vhXw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

### **上线流程**

- 架构组审核
- DevOps 对接人分配项目负责人，明确指定项目Devops、DBA 负责人
- 完成项目上线
- 添加项目监控
- 添加日志索引

### **appid 作用域**

- 应用名称
- git_repo
- es 索引名称
- 监控服务名称
- 访问域名

### **编写部署文档**

项目开发负责人参考下图checklist要点，在项目code中编写README文档。

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQ8iawdhCw1Pl5R32AjsMoYlN5k8Acx7SHFCrVicuyicZvkKbHQCnfoJGqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **应用架构图案例**

![img](https://mmbiz.qpic.cn/mmbiz_png/x63NLUqhL5Fo3c07zFyEawuJSDfPnqyQ8S8dXianiaicnI7eenqWEYT6ibia1COemhwZJXOibBLPREwFlpYqMBHqF5Kg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **配置变更流程**

### 作用域

- 测试环境
- 生产环境

### 审批流程

- 一般事务变更

- ansible配置项修改。

- - 提交 pull request，待运维管理员审批通过
  - 执行变更

![img](https://mmbiz.qpic.cn/mmbiz_jpg/d5patQGz8Ke7cGs8eTbQeFuoUY3rGgEnw5btb4SMuuy93yeicYfemmVibNCG0XAFOWths6wl9MexcFruWyODQABQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 全局事务：

- - 邮件成员：CTO、研发负责人、产品负责人 、运维负责人
  - 全局环境信息，参考以下审批流程
  - ansible配置项修改
  - 提交 pull request ,待运维管理员审批通过
  - 邮件组成员商定变更时间
  - 确定变更时间后，全员邮件、微信组通知。既定时间内执行变更
    ![img](https://mmbiz.qpic.cn/mmbiz_jpg/d5patQGz8Ke7cGs8eTbQeFuoUY3rGgEnuDiaYib8GgoDEMZ1B0iaKia139LxDbbKsZKwhNvbianIY4ZGUMwdVPlccaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)