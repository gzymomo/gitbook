- [k8s环境下Skywalking容器化部署](https://www.jianshu.com/p/4f4c182bcbd8)

- [kubernetes部署skywalking集群和JAVA服务的接入](https://blog.csdn.net/nangonghen/article/details/110290450)

- [使用helm3在k8s上部署SkyWalking](https://blog.csdn.net/duanqing_song/article/details/106229486)

- [k8s 部署 skywalking 并将 pod 应用接入链路追踪](https://segmentfault.com/a/1190000039878680)

  

# 一、打包agent的镜像

[k8s上运行我们的springboot服务之——skywalking监控我们的springboot](https://blog.csdn.net/weiranaixi/article/details/112508593)

```bash
vi Dockerfile
```

```bash
FROM busybox:latest
ENV LANG=C.UTF-8
RUN set -eux && mkdir -p /opt/skywalking/agent/
ADD agent/ /opt/skywalking/agent/
WORKDIR /
```

# 1.1 修改Yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: demo-istio
    service: demo-istio
  name: demo-istio
  namespace: default
spec:
  ports:
  - name: demo-istio
    port: 8070
  selector:
    app: demo-istio
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: demo-istio
    version: v1
  name: demo-istio-v1
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo-istio
      version: v1
  template:
    metadata:
      annotations:
        prometheus.io/scrape: false
        prometheus.io/path: /actuator/prometheus
        prometheus.io/port: 8070
      labels:
        app: demo-istio
        version: v1
    spec:
      containers:
      - env:
        - name: LIMITS_MEMORY
          valueFrom:
            resourceFieldRef:
              divisor: 1Mi
              resource: limits.memory
        - name: JAVA_OPTS
          value: -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Duser.timezone=Asia/Shanghai
        image: 192.168.10.59:8080/frame/demo-frame-istio:1.0.RELEASE
        imagePullPolicy: Always
        name: demo-istio
        ports:
        - containerPort: 8070
        resources:
          limits:
            cpu: 2048m
            memory: 2048Mi
          requests:
            cpu: 1024m
            memory: 1024Mi
        volumeMounts:
        - mountPath: /sidecar
          name: sidecar
      hostAliases:
      - hostnames:
        - www.zipkin.com
        ip: 192.168.10.80
      imagePullSecrets:
      - name: regsecret
      initContainers:
      - command:
        - cp
        - -r
        - /opt/skywalking/agent
        - /sidecar
        image: 192.168.10.59:8080/frame/skywalking-agent:v1
        imagePullPolicy: Always
        name: sidecar
        volumeMounts:
        - mountPath: /sidecar
          name: sidecar
      volumes:
      - emptyDir: {}
        name: sidecar
```







# 二、使用 helm 部署 skywalking

在 k8s 中使用 helm 的前提是需要先安装 helm 客户端，关于 helm 的安装可以查看官方文档。

> 安装 helm 官方文档地址：[https://helm.sh/docs/intro/in...](https://helm.sh/docs/intro/install/)

这里介绍两种方式部署 skywalking , 一种是使用 [Artifact Hub](https://artifacthub.io/) 提供的 chart，另一种是通过 GitHub 提供的源文件进行部署。这两种部署方式的本质是一样的，不同的是通常情况下 GitHub 上面的通常更新先于 [Artifact Hub](https://artifacthub.io/)。

## 1、使用 [Artifact Hub](https://artifacthub.io/) 提供的 chart 部署 skywalking

这是使用 [Artifact Hub](https://artifacthub.io/) 提供的 chart，搜索 skywalking 可以看到如下图所示：
![image.png](https://segmentfault.com/img/remote/1460000039879110)

如下图所示，点击 INSTALL 查看并添加 helm 仓库：

![image.png](https://segmentfault.com/img/remote/1460000039879111)

添加 skywalking chart 仓库的命令如下：

```
helm repo add choerodon https://openchart.choerodon.com.cn/choerodon/c7n
```

使用下面的命令查看添加的 repo，这里一并列出了我添加的其他仓库：

```
helm repo list
NAME                    URL                                               
oteemocharts            https://oteemo.github.io/charts                   
kubeview                https://benc-uk.github.io/kubeview/charts         
oteemo-charts           https://oteemo.github.io/charts                   
jenkinsci               https://charts.jenkins.io/                        
ygqygq2                 https://ygqygq2.github.io/charts/                 
prometheus-community    https://prometheus-community.github.io/helm-charts
my-chart                https://wangedison.github.io/k8s-helm-chart/      
carlosjgp               https://carlosjgp.github.io/open-charts/          
choerodon               https://openchart.choerodon.com.cn/choerodon/c7n  
```

使用下面的命令在仓库中搜索 skywalking：

```
helm search repo skywalking
NAME                        CHART VERSION    APP VERSION    DESCRIPTION                 
choerodon/skywalking        6.6.0            6.6.0          Apache SkyWalking APM System
choerodon/skywalking-oap    0.1.3            0.1.3          skywalking-oap for Choerodon
choerodon/skywalking-ui     0.1.4            0.1.4          skywalking-ui for Choerodon 
choerodon/chart-test        1.0.0            1.0.0          skywalking-ui for Choerodon 
```

为了做一些自定义的配置，比如修改版本号等，使用下面的命令下载 chart 到本地：

```
# 下载 chart 到本地
[root@k8s-node01 chart-test]# helm pull choerodon/skywalking
# 查看下载的内容
[root@k8s-node01 chart-test]# ll
total 12
-rw-r--r-- 1 root root 10341 Apr 21 11:12 skywalking-6.6.0.tgz
# 解压 chart 包
[root@k8s-node01 chart-test]# tar -zxvf skywalking-6.6.0.tgz 
skywalking/Chart.yaml
skywalking/values.yaml
skywalking/templates/_helpers.tpl
skywalking/templates/istio-adapter/adapter.yaml
skywalking/templates/istio-adapter/handler.yaml
skywalking/templates/istio-adapter/instance.yaml
skywalking/templates/istio-adapter/rule.yaml
skywalking/templates/mysql-init.job.yaml
skywalking/templates/oap-clusterrole.yaml
skywalking/templates/oap-clusterrolebinding.yaml
skywalking/templates/oap-deployment.yaml
skywalking/templates/oap-role.yaml
skywalking/templates/oap-rolebinding.yaml
skywalking/templates/oap-serviceaccount.yaml
skywalking/templates/oap-svc.yaml
skywalking/templates/ui-deployment.yaml
skywalking/templates/ui-ingress.yaml
skywalking/templates/ui-svc.yaml
skywalking/.auto_devops.sh
skywalking/.choerodon/.docker/config.json
skywalking/.gitlab-ci.yml
skywalking/.helmignore
skywalking/Dockerfile
skywalking/README.md
```

自定义配置，可以修改 `values.yaml`文件：

```
vim skywalking/values.yaml
```

可以看到，这里的 skywalking 默认使用的是 mysql 数据库，使用的 skywalking 的版本是 6.6.0，可以根据自己的需求修改对应的版本号，修改完成后保存退出。更多的配置修改说明可以查看 [skywalking 官方 chart 说明](https://artifacthub.io/packages/helm/choerodon/skywalking)。

可以使用下面的命令安装 chart，并指定自定义的配置文件：

```
# Usage:  helm install [NAME] [CHART] [flags]
helm install  skywalking skywaling -f skywalking/values.yaml
```

使用上面的命令会将 skywalking 安装在 `default` 命名空间，我们可以使用 `-n namespace` 参数指定到特定的命名空间，前提是这个命名空间必须存在。

安装成功后可以使用下面的命令查看安装的 chart，安装后的 chart 叫做 release：

```
helm list
```

可以使用下面的命令卸载 chart：

```
# Usage:  helm uninstall RELEASE_NAME [...] [flags]
helm uninstall skywalking -n default
```

## 2、从 GitHub 下载 chart 源文件部署 skywalking

> skywalking chart 的 GitHub地址为：[https://github.com/apache/sky...](https://github.com/apache/skywalking-kubernetes)

在 Linux 系统中使用下面的命令安装 `git`：

```
yum install -y git
```

使用下面的命令 clone 代码：

```
git clone https://github.com/apache/skywalking-kubernetes
```

进到 skywalking 的 chart 目录，可以看到下面的内容：

```
[root@k8s-node01 chart-test]# cd skywalking-kubernetes/chart/skywalking/
[root@k8s-node01 skywalking]# ll
total 84
-rw-r--r-- 1 root root  1382 Apr 21 11:35 Chart.yaml
drwxr-xr-x 3 root root  4096 Apr 21 11:35 files
-rw-r--r-- 1 root root   877 Apr 21 11:35 OWNERS
-rw-r--r-- 1 root root 42593 Apr 21 11:35 README.md
drwxr-xr-x 3 root root  4096 Apr 21 11:35 templates
-rw-r--r-- 1 root root  1030 Apr 21 11:35 values-es6.yaml
-rw-r--r-- 1 root root  1031 Apr 21 11:35 values-es7.yaml
-rw-r--r-- 1 root root  1366 Apr 21 11:35 values-my-es.yaml
-rw-r--r-- 1 root root 10184 Apr 21 11:35 values.yaml
```

可以看到作者非常贴心的为我们定义了三个自定义配置文件：`values-es6.yaml` 、`values-es7.yaml` 和 `values-my-es.yaml`，分别对应使用 es6、es7 和 外部 es 存储的配置。因为我这里使用的是外部自有的 es 集群，并且 es 的版本是 6.7.0，所以我需要修改 `values-my-es.yaml` 文件如下：

```
# Default values for skywalking.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

oap:
  image:
    tag: 8.5.0-es6      # Set the right tag according to the existing Elasticsearch version
  storageType: elasticsearch # elasticsearch 对应 es6 ，elasticsearch7 对应 es7

ui:
  image:
    tag: 8.5.0

elasticsearch:
  enabled: false # 由于使用 外部的 es，所以这里需要设置为 false，因为设置为 true 会在 k8s 中部署 es
  config:               # For users of an existing elasticsearch cluster,takes effect when `elasticsearch.enabled` is false
    host: your.elasticsearch.host.or.ip
    port:
      http: 9200
    user: "xxx"         # [optional]
    password: "xxx"     # [optional]
```

还可以修改 `values.yaml` 文件，比如开启 ingress，更多详细的配置可以查看 GitHub 中 [skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes) 的说明，根据需要配置好之后就可以使用下面的命令安装 chart：

```
helm install "${SKYWALKING_RELEASE_NAME}" skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}"  -f ./skywalking/values-my-es.yaml
```

安装完成以后，可以通过下面的命令查看 pod 是否正常启动：

```
[root@k8s-node01 ~]# kubectl get pod -n default
NAME                                     READY   STATUS      RESTARTS   AGE
skywalking-es-init-v6sbn                 0/1     Completed   0          1h
skywalking-oap-5c4d5bf887-4cvjk          1/1     Running     0          1h
skywalking-oap-5c4d5bf887-g75fj          1/1     Running     0          1h
skywalking-ui-6cd4bbd858-sbpvt           1/1     Running     0          1h
```

# 二、使用 sidecar 将 pod 接入链路追踪

前面简单介绍了使用 helm 部署 skywalking，下面介绍如何使用 sidecar 将 pod 接入链路追踪。Java微服务接入skywalking 可以使用 `SkyWalking Java Agent` 来上报监控数据，这就需要 java 微服务在启动参数中通过 `-javaagent:<skywalking-agent-path>` 指定 skywalking agent 探针包，通常有以下三种方式集成：

- 使用官方提供的基础镜像 `skywalking-base`；
- 将 agent 包构建到已存在的镜像中；
- 通过 sidecar 模式挂载 agent；

前面两种方式在前面的文章中有简单介绍，这里主要介绍如何使用 sidecar 将 pod  接入链路追踪，这种方式不需要修改原来的基础镜像，也不需要重新构建新的服务镜像，而是会以sidecar模式，通过共享的 volume 将  agent 所需的相关文件直接挂载到已经存在的服务镜像中。sidecar模式原理很简单，就是在 pod  中再部署一个初始容器，这个初始容器的作用就是将 skywalking agent 和 pod 中的应用容器共享。

## 1、什么是初始化容器 init container

Init Container 就是用来做初始化工作的容器，可以是一个或者多个，如果有多个的话，这些容器会按定义的顺序依次执行，只有所有的 Init  Container 执行完后，主容器才会被启动。我们知道一个Pod里面的所有容器是共享数据卷和网络命名空间的，所以 Init Container 里面产生的数据可以被主容器使用到的。

## 2、自定义 skywalking agent 镜像

在开始以 sidecar 方式将一个 java 微服务接入 skywalking 之前，我们需要构建 skywalking agent 的公共镜像，具体步骤如下：

- 使用下面的命令下载 skywalking agent 并解压：

  ```
  # 下载 skywalking-8.5.0 for es6 版本的发布包，与部署的 skywalking 后端版本一致
  wget https://www.apache.org/dyn/closer.cgi/skywalking/8.5.0/apache-skywalking-apm-8.5.0.tar.gz
  # 将下载的发布包解压到当前目录
  tar -zxvf apache-skywalking-apm-8.5.0.tar.gz
  ```

- 在前面步骤中解压的 skywalking 发行包的同级目录编写 Dockerfile 文件，具体内容如下：

  ```
  FROM busybox:latest
  LABEL maintainer="xiniao"
  COPY apache-skywalking-apm-bin/agent/ /usr/skywalking/agent/
  ```

  在上述 Dockefile 文件中使用的基础镜像是 bosybox 镜像，而不是 SkyWalking 的发行镜像，这样可以确保构建出来的sidecar镜像保持最小。

- 使用下面的命令构建镜像：

  ```
  docker build -t skywalking-agent-sidecar:8.5.0 .
  ```

  使用下面的命令查看构建的镜像：

  ```
  docker images |grep agent
  skywalking-agent-sidecar          8.5.0             98290e961b49        5 days ago          32.6MB
  ```

- 使用下面的命令给镜像打标签，这里推送到我的阿里云镜像仓库：

  ```
  docker tag skywalking-agent-sidecar:8.5.0 registry.cn-shenzhen.aliyuncs.com/devan/skywalking-agent-sidecar:8.5.0
  ```

  使用下面的命令推送镜像到远程仓库：

  ```
  docker push registry.cn-shenzhen.aliyuncs.com/devan/skywalking-agent-sidecar:8.5.0
  ```

  ## 3、sidecar 模式接入 skywalking

  上面我们通过手工构建的方式构建了 SkyWalking Java Agent 的公共 Docker 镜像，接下来我们将演示如何通过编写 Kubernetes 服务发布文件，来将 Java 服务发布到 K8s 集群的过程中自动以 SideCar 的形式集成Agent 并接入 SkyWalking 服务。

- 创建一个 `deploy-skywalking.yaml`文件，内容如下：

  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
  name: spring-boot-skywalking-demo
  namespace: default
  labels:
    app: spring-boot-skywalking-demo
  spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-boot-skywalking-demo
  template:
    metadata:
      labels:
        app: spring-boot-skywalking-demo
    spec:
      #构建初始化镜像(通过初始化镜像的方式集成SkyWalking Agent)
      initContainers:
        - image: registry.cn-shenzhen.aliyuncs.com/devan/skywalking-agent-sidecar:8.5.0
          name: sw-agent-sidecar
          imagePullPolicy: IfNotPresent
          command: [ "sh" ]
          args:
            [
                "-c",
                "cp -R /usr/skywalking/agent/* /skywalking/agent",
            ]
          volumeMounts:
            - mountPath: /skywalking/agent
              name: sw-agent
      containers:
        - name: spring-boot-skywalking-demo
          image: ${ORIGIN_REPO}/spring-boot-skywalking-demo:${IMAGE_TAG}
          imagePullPolicy: Always
          env:
            - name: TZ
              value: Asia/Shanghai
            - name: BUILD_TAG
              value: ${BUILD_TAG}
            - name: NAMESPACE
              value: default
            #这里通过JAVA_TOOL_OPTIONS，而不是JAVA_OPTS可以实现不通过将agent命令加入到java应用jvm参数而实现agent的集成
            - name: JAVA_TOOL_OPTIONS
              value: -javaagent:/usr/skywalking/agent/skywalking-agent.jar
            - name: SW_AGENT_NAME
              value: spring-boot-skywalking-demo
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
              # FQDN: servicename.namespacename.svc.cluster.local
              value: skywalking-oap.default.svc:11800
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 200m
              memory: 500Mi
          volumeMounts:
            - mountPath: /usr/skywalking/agent
              name: sw-agent
      volumes:
        - name: sw-agent
          emptyDir: { }
  
  
  ---
  apiVersion: v1
  kind: Service
  metadata:
  name: spring-boot-skywalking-demo
  namespace: default
  labels:
    app: spring-boot-skywalking-demo
  spec:
  ports:
    - name: port
      port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: spring-boot-skywalking-demo
  type: ClusterIP
  ```

  `spec.volumes` 指的是 pod 中的卷，`spec.containers.volumeMounts` 是将指定的卷 mount 到容器指定的位置，相当于 docker 里面的 `-v 宿主机目录：容器目录`，我们这里使用的是   `emptyDir{}`，这个就相当于一个共享卷，是一个临时的目录，生命周期等同于Pod的生命周期。初始容器执行的命令是 `sh -c cp -R /usr/skywalking/agent/* /skywalking/agent`， 意思是将 skywalking agent 复制到共享目录，主容器关联了共享目录，所以主容器就可以访问 skywalking agent。

使用下面的命令部署应用：

```
 kubectl apply -f deploy-skywalking.yaml
```

# 总结

这篇文章简单介绍了使用 helm 部署 skywalking，关于 helm 的使用以及如何自定义  chart，后面可以写一篇文章介绍一下，如果想要详细了解，建议还是查看官方文档。还有简单介绍了 pod 应用以 SideCar  模式接入SkyWalking 服务，主要是理解初始容器 initContainers  的作用，初始容器是在主容器启动之前执行，可以和主容器共享数据卷共享网络命名空间。