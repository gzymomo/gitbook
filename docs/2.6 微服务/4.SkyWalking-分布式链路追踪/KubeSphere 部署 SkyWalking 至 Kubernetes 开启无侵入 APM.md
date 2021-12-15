[KubeSphere 部署 SkyWalking 至 Kubernetes 开启无侵入 APM](https://www.qedev.com/cloud/216340.html)



# Kubernetes + Spring Cloud 集成链路追踪 SkyWalking

原文地址：51CTO：https://blog.51cto.com/u_15181572/2961802

## 一、概述

### 1、什么是 SkyWalking ？

分布式系统的应用程序性能监视工具，专为微服务、云原生架构和基于容器（Docker、K8s、Mesos）架构而设计。提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案。

官网地址：http://skywalking.apache.org/

### 2、SkyWalking 特性

- 多种监控手段，语言探针和 Service Mesh
- 多语言自动探针，Java，.NET Core和Node.JS
- 轻量高效，不需要大数据
- 模块化，UI、存储、集群管理多种机制可选
- 支持告警
- 优秀的可视化方案

### 3、整体结构

![9a91ca357a4080faff187ad44575982c.png](https://s4.51cto.com/images/blog/202106/30/9a91ca357a4080faff187ad44575982c.png)整个架构，分成上、下、左、右四部分：

考虑到让描述更简单，我们舍弃掉 Metric 指标相关，而着重在 Tracing 链路相关功能。

- 上部分 Agent ：负责从应用中，收集链路信息，发送给 SkyWalking OAP 服务器。目前支持 SkyWalking、Zikpin、Jaeger 等提供的 Tracing 数据信息。而我们目前采用的是，SkyWalking Agent 收集 `SkyWalking Tracing`数据，传递给服务器。
- 下部分 `SkyWalking OAP` ：负责接收 Agent 发送的 Tracing 数据信息，然后进行分析(Analysis Core) ，存储到外部存储器( Storage )，最终提供查询( Query)功能。
- 右部分 Storage ：Tracing 数据存储。目前支持 ES、MySQL、Sharding Sphere、TiDB、H2 多种存储器。而我们目前采用的是 ES ，主要考虑是 SkyWalking 开发团队自己的生产环境采用 ES 为主。
- 左部分 `SkyWalking UI` ：负责提供控台，查看链路等等

简单概况原理为下图：![3b752149768b084f00e57475bd0fb0a8.png](https://s4.51cto.com/images/blog/202106/30/3b752149768b084f00e57475bd0fb0a8.png)

## 二、搭建 skywalking

### 1、环境准备

- Mkubernetes 版本：1.18.5
- Nginx Ingress 版本：2.2.8
- Helm 版本：3.2.4
- 持久化存储驱动：NFS

### 2、使用 chart 部署

本文主要讲述的是如何使用 Helm Charts 将 SkyWalking 部署到 Kubernetes 集群中，相关文档可以参考skywalking-kubernetes

目前推荐的四种方式：

- 使用 helm 3 提供的 helm serve 启动本地 helm repo
- 使用本地 chart 文件部署
- 使用 harbor 提供的 repo 功能
- 直接从官方 repo 进行部署（暂不满足）

> ❝
>
> 注意：目前 skywalking 的 chart 还没有提交到官方仓库，请先参照前三种方式进行部署
>
> ❞

#### 2.1、 下载 chart 文件

可以直接使用本地文件部署 skywalking,按照上面的步骤将`skywalking chart`下载完成之后，直接使用以下命令进行部署：

```
git clone https://github.com/apache/skywalking-kubernetes
cd skywalking-kubernetes/chart
helm repo add elastic https://helm.elastic.co
helm dep up skywalking
export SKYWALKING_RELEASE_NAME=skywalking  # 定义自己的名称
export SKYWALKING_RELEASE_NAMESPACE=default  # 定义自己的命名空间
```

#### 2.2、定义已存在es参数文件

修改`values-my-es.yaml`:

```
oap:
  image:
    tag: 8.1.0-es7      # Set the right tag according to the existing Elasticsearch version
  storageType: elasticsearch7

ui:
  image:
    tag: 8.1.0

elasticsearch:
  enabled: false
  config:               # For users of an existing elasticsearch cluster,takes effect when `elasticsearch.enabled` is false
    host: elasticsearch-client
    port:
      http: 9200
    user: "elastic"         # [optional]
    password: "admin@123"     # [optional]
```

#### 2.3、helm 安装

```
helm install "${SKYWALKING_RELEASE_NAME}" skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  -f ./skywalking/values-my-es.yaml
```

安装完成后，我们核实下安装情况：

```
$ kubectl get deployment -n skywalking
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
my-skywalking-oap   2/2     2            2           9m
my-skywalking-ui    1/1     1            1           9m
```

## 三、使用 Skywalking Agent

Java 中使用 agent ，提供了以下三种方式供你选择

- 使用官方提供的基础镜像
- 将 agent 包构建到已经存在的基础镜像中
- sidecar 模式挂载 agent（推荐）

### 1、使用官方提供的基础镜像

查看官方 docker hub 提供的基础镜像，只需要在你构建服务镜像是 From 这个镜像即可，直接集成到 Jenkins 中可以更加方便

### 2、将 agent 包构建到已经存在的基础镜像中

提供这种方式的原因是：官方的镜像属于精简镜像，并且是 openjdk ，可能很多命令没有，需要自己二次安装，这里略过。

### 3、sidecar 模式挂载 agent

由于服务是部署在 Kubernetes 中，使用这种方式来使用 Skywalking Agent  ,这种方式的好处在不需要修改原来的基础镜像，也不用重新构建新的服务镜像，而是以sidecar 模式，通过共享 volume 的方式将 agent 所需的相关文件挂载到已经存在的服务镜像中。

#### 3.1、构建 skywalking agent image

自己构建,参考：https://hub.docker.com/r/prophet/skywalking-agent

通过以下 dockerfile 进行构建：

```
FROM alpine:3.8

LABEL maintainer="zuozewei@hotmail.com"

ENV SKYWALKING_VERSION=8.1.0

ADD http://mirrors.tuna.tsinghua.edu.cn/apache/skywalking/${SKYWALKING_VERSION}/apache-skywalking-apm-${SKYWALKING_VERSION}.tar.gz /

RUN tar -zxvf /apache-skywalking-apm-${SKYWALKING_VERSION}.tar.gz && \
    mv apache-skywalking-apm-bin skywalking && \
    mv /skywalking/agent/optional-plugins/apm-trace-ignore-plugin* /skywalking/agent/plugins/ && \
    echo -e "\n# Ignore Path" >> /skywalking/agent/config/agent.config && \
    echo "# see https://github.com/apache/skywalking/blob/v8.1.0/docs/en/setup/service-agent/java-agent/agent-optional-plugins/trace-ignore-plugin.md" >> /skywalking/agent/config/agent.config && \
    echo 'trace.ignore_path=${SW_IGNORE_PATH:/health}' >> /skywalking/agent/config/agent.config
docker build -t 172.16.106.237/monitor/skywalking-agent:8.1.0 .
```

待 docker build 完毕后，push 到仓库即可。

#### 3.2、使用 sidecar 挂载

示例配置文件如下：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-skywalking
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-skywalking
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: demo-skywalking
    spec:
      initContainers:
        - name: init-skywalking-agent
          image: 172.16.106.237/monitor/skywalking-agent:8.1.0
          command:
            - 'sh'
            - '-c'
            - 'set -ex;mkdir -p /vmskywalking/agent;cp -r /skywalking/agent/* /vmskywalking/agent;'
          volumeMounts:
            - mountPath: /vmskywalking/agent
              name: skywalking-agent
      containers:
        - image: nginx:1.7.9
          imagePullPolicy: Always
          name: nginx
          ports:
            - containerPort: 80
              protocol: TCP
          volumeMounts:
            - mountPath: /opt/skywalking/agent
              name: skywalking-agent
      volumes:
        - name: skywalking-agent
          emptyDir: {}
```

以上是挂载 sidecar 的 `deployment.yaml` 文件，以 nginx 作为服务为例，主要是通过共享 volume 的方式挂载 agent，首先 initContainers 通过 skywalking-agent 卷挂载了 sw-agent-sidecar 中的 `/vmskywalking/agent`，并且将上面构建好的镜像中的 agent 目录 cp 到了 `/vmskywalking/agent` 目录，完成之后 nginx 启动时也挂载了 `skywalking-agent` 卷，并将其挂载到了容器的 `/opt/skywalking/agent` 目录，这样就完成了共享过程。

## 四、改造 Spring Cloud 应用

### 1、docker打包并推送到仓库

修改下 dockerfile 配置，集成 skywalking agent：

```
FROM insideo/centos7-java8-build
VOLUME /tmp
ADD mall-admin.jar app.jar
RUN bash -c 'touch /app.jar'
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime && echo Asia/Shanghai > /etc/timezone
ENTRYPOINT ["java","-Dapp.id=svc-mall-admin","-javaagent:/opt/skywalking/agent/skywalking-agent.jar","-Dskywalking.agent.service_name=svc-mall-admin","-Dskywalking.collector.backend_service=my-skywalking-oap.skywalking.svc.cluster.local:11800","-jar","-Dspring.profiles.active=prod","-Djava.security.egd=file:/dev/./urandom","/app.jar"]
```

改好了，直接运行 `maven package` 就能将这个项目打包成镜像。

注意：

> ❝
>
> k8s 创建 Service 时，它会创建相应的 DNS 条目。此条目的格式为 `<service-name>.<namespace-name>.svc.cluster.local`，这意味着如果容器只使用`<service-name>`，它将解析为本地服务到命名空间。如果要跨命名空间访问，则需要使用完全限定的域名。
>
> ❞

### 2、编写 k8s的yaml版本的部署脚本

这里我以其中某服务举例：

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: svc-mall-admin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: svc-mall-admin
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: svc-mall-admin
    spec:
      initContainers:
        - name: init-skywalking-agent
          image: 172.16.106.237/monitor/skywalking-agent:8.1.0
          command:
            - 'sh'
            - '-c'
            - 'set -ex;mkdir -p /vmskywalking/agent;cp -r /skywalking/agent/* /vmskywalking/agent;'
          volumeMounts:
            - mountPath: /vmskywalking/agent
              name: skywalking-agent
      containers:
        - image: 172.16.106.237/mall_repo/mall-admin:1.0
          imagePullPolicy: Always
          name: mall-admin
          ports:
            - containerPort: 8180
              protocol: TCP
          volumeMounts:
            - mountPath: /opt/skywalking/agent
              name: skywalking-agent
      volumes:
        - name: skywalking-agent
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: svc-mall-admin
spec:
  ports:
    - name: http
      port: 8180
      protocol: TCP
      targetPort: 8180
  selector:
    app: svc-mall-admin
```

然后就可以直接运行了,它就可以将的项目全部跑起来了。

## 五、测试验证

完事，可以去 SkyWalking UI 查看是否链路收集成功。

### 1、 测试应用 API

首先，请求下 Spring Cloud 应用提供的 API。因为，我们要追踪下该链路。![5490b3cfe1d9d0d5b81d5ca910d7e5c4.png](https://s4.51cto.com/images/blog/202106/30/5490b3cfe1d9d0d5b81d5ca910d7e5c4.png)

### 2、 查看 SkyWalking UI 界面

![7f56f83590d2ca6ae3bfdd0a9f225249.png](https://s4.51cto.com/images/blog/202106/30/7f56f83590d2ca6ae3bfdd0a9f225249.png)在这里插入图片描述

这里，我们会看到 SkyWalking 中非常重要的三个概念：

- 服务(Service) ：表示对请求提供相同行为的一系列或一组工作负载。在使用 Agent 或 SDK  的时候，你可以定义服务的名字。如果不定义的话，SkyWalking 将会使用你在平台（例如说 Istio）上定义的名字。这里，我们可以看到  Spring Cloud  应用的服务为 `svc-mall-admin`，就是我们在 agent 环境变量  `service_name` 中所定义的。
- 服务实例(Service Instance) ：上述的一组工作负载中的每一个工作负载称为一个实例。就像 Kubernetes 中的  pods 一样, 服务实例未必就是操作系统上的一个进程。但当你在使用 Agent 的时候,  一个服务实例实际就是操作系统上的一个真实进程。这里，我们可以看到 Spring Cloud  应用的服务为 `UUID@hostname`，由  Agent 自动生成。
- 端点(Endpoint) ：对于特定服务所接收的请求路径, 如 HTTP 的 URI 路径和 gRPC 服务的类名 + 方法签名。

这里，我们可以看到 Spring Cloud 应用的一个端点，为  API 接口 `/mall-admin/admin/login`。

更多 agent 参数介绍参考：https://github.com/apache/skywalking/blob/v8.1.0/docs/en/setup/service-agent/java-agent/README.md

点击「拓扑图」菜单，进入查看拓扑图的界面：![57d9abb257c5a93f5c2cd878e30abdb6.png](https://s4.51cto.com/images/blog/202106/30/57d9abb257c5a93f5c2cd878e30abdb6.png)点击「追踪」菜单，进入查看链路数据的界面：![18d1fc9f1686091b50acaef281302912.png](https://s4.51cto.com/images/blog/202106/30/18d1fc9f1686091b50acaef281302912.png)

## 六、小结

本文详细介绍了如何使用 Kubernetes + Spring Cloud 集成  SkyWalking，顺便说下调用链监控在目前的微服务系统里面是必不可少的组件，分布式追踪、服务网格遥测分析、度量聚合和可视化还是挺好用的，这里我们选择了 Skywalking，具体原因和细节的玩法就不在此详述了。