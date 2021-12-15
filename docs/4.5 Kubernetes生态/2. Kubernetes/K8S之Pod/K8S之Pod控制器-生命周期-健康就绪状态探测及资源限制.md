博客园：

- [容器编排系统之Pod生命周期、健康/就绪状态探测以及资源限制](https://www.cnblogs.com/qiuhom-1874/p/14143610.html)



# 1. Pod

## 1.1. Pod介绍

### 1.1.1. Pod简介

Pod 是 Kubernetes 的基本构建块，它是 Kubernetes 对象模型中创建或部署的**最小和最简单的单元**。 Pod 表示集群上正在运行的进程。Pod 封装了应用程序容器（或者在某些情况下封装多个容器）、存储资源、唯一网络 IP 以及控制容器应该如何运行的选项。 Pod 表示部署单元：Kubernetes 中应用程序的单个实例，它可能由单个容器或少量紧密耦合并共享资源的容器组成。

一个pod内部一般仅运行一个pod，也可以运行多个pod，如果存在多个pod时，其中一个为主容器，其它作为辅助容器，也被称为边车模式。同一个pod共享一个网络名称空间和外部存储卷。



1. 最小部署单元
2. 包含多个容器（一组容器的集合）
3. 一个pod中容器共享网络命令空间
4. pod是短暂的，重启后ip会变



Pod存在意义：

1. 创建容器使用docker，一个docker对应是一个容器，一个容器有进程，一个容器运行一个应用程序。
2. Pod是多进程设计，运行多个应用程序（一个pod有多个容器，一个容器里面运行一个应用程序）
3. Pod存在是为了亲密性应用（两个应用之间交互。网络之间调用。两个应用需要频繁调用）



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1579339190487-a1f2c303-a092-4f16-a7e2-2bba306fe3d2.png)

### 1.1.2. Pod生命周期

Pod的生命周期中可以经历多个阶段，在一个Pod中在主容器(Main Container)启动前可以由init container来完成一些初始化操作。初始化完毕后，init Container 退出，Main Container启动。

在主容器启动后可以执行一些特定的指令，称为启动后钩子(PostStart)，在主容器退出前也可以执行一些特殊指令完成清理工作，称为结束前钩子(PreStop)。

在主容器工作周期内，并不是刚创建就能对外提供服务，容器内部可能需要加载相关配置，因此可以使用特定命令确定容器是否就绪，称为就绪性检测(ReadinessProbe)，完成就绪性检测才能成为Ready状态。

主容器对外提供服务后，可能出现意外导致容器异常，虽然此时容器仍在运行，但是不具备对外提供业务的能力，因此需要对其做存活性探测(LivenessProbe)。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1579445077801-c30c2449-41d5-43e6-9892-025fbcf78aaf.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

![img](https://img2020.cnblogs.com/blog/1395193/202008/1395193-20200812220935447-1350637895.png)

每个Pod里运行着一个特殊的被称之为Pause的容器，其他容器则为业务容器，这些业务容器共享Pause容器的网络栈和Volume挂载卷，因此他们之间通信和数据交换更为高效。在设计时可以充分利用这一特性，将一组密切相关的服务进程放入同一个Pod中；同一个Pod里的容器之间仅需通过localhost就能互相通信。

**kubernetes中的pause容器主要为每个业务容器提供以下功能：**

PID命名空间：Pod中的不同应用程序可以看到其他应用程序的进程ID。

网络命名空间：Pod中的多个容器能够访问同一个IP和端口范围。

IPC命名空间：Pod中的多个容器能够使用System V IPC或POSIX消息队列进行通信。

UTS命名空间：Pod中的多个容器共享一个主机名；Volumes（共享存储卷）。

Pod中的各个容器可以访问在Pod级别定义的Volumes。

#### Init Container容器

Pod可以包含多个容器，应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。

如果为一个 Pod 指定了多个 Init 容器，这些Init容器会按顺序逐个运行。每个 Init 容器都必须运行成功，下一个才能够运行。当所有的 Init 容器运行完成时，Kubernetes 才会为 Pod 初始化应用容器并像平常一样运行。

 

#### **Init容器与普通的容器非常像，除了以下两点：**

1、Init容器总是运行到成功完成且正常退出为止

2、只有前一个Init容器成功完成并正常退出，才能运行下一个Init容器。

如果Pod的Init容器失败，Kubernetes会不断地重启Pod，直到Init容器成功为止。但如果Pod对应的restartPolicy为Never，则不会重新启动。

在所有的 Init 容器没有成功之前，Pod 将不会变成 Ready 状态。 Init 容器的端口将不会在 Service 中进行聚集。 正在初始化中的 Pod 处于 Pending 状态，但会将条件 Initializing 设置为 true。

如果 Pod 重启，所有 Init 容器必须重新执行。

在 Pod 中的每个应用容器和 Init 容器的名称必须唯一；与任何其它容器共享同一个名称，会在校验时抛出错误。



#### Init 容器能做什么？

因为 Init 容器是与应用容器分离的单独镜像，其启动相关代码具有如下优势：

1、Init 容器可以包含一些安装过程中应用容器不存在的实用工具或个性化代码。例如，在安装过程中要使用类似 sed、 awk、 python 或 dig 这样的工具，那么放到Init容器去安装这些工具；再例如，应用容器需要一些必要的目录或者配置文件甚至涉及敏感信息，那么放到Init容器去执行。而不是在主容器执行。

2、Init 容器可以安全地运行这些工具，避免这些工具导致应用镜像的安全性降低。

3、应用镜像的创建者和部署者可以各自独立工作，而没有必要联合构建一个单独的应用镜像。

4、Init 容器能以不同于Pod内应用容器的文件系统视图运行。因此，Init容器可具有访问 Secrets 的权限，而应用容器不能够访问。

5、由于 Init 容器必须在应用容器启动之前运行完成，因此 Init 容器提供了一种机制来阻塞或延迟应用容器的启动，直到满足了一组先决条件。一旦前置条件满足，Pod内的所有的应用容器会并行启动。



### 1.1.3. Pod状态

- Pending: Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建。
- Running: 该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
- Succeeded: Pod 中的所有容器都被成功终止，并且不会再重启。
- Failed: Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。
- Unknown: 因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败。

![image](https://cdn.nlark.com/yuque/0/2020/jpeg/378176/1579446234452-ef9fedb8-e923-4aef-a8c3-6862d4729009.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

### 1.1.4. Pod创建过程

[![img](https://img2020.cnblogs.com/blog/1503305/202012/1503305-20201216132714632-1243109247.png)](https://img2020.cnblogs.com/blog/1503305/202012/1503305-20201216132714632-1243109247.png)

　　提示：首先用户通过客户端工具将请求提交给apiserver，apiserver收到用户的请求以后，它会尝试将用户提交的请求内容存进etcd中，etcd存入完成后就反馈给apiserver写入数据完成，此时apiserver就返回客户端，说某某资源已经创建；随后apiserver要发送一个watch信号给scheduler，说要创建一个新pod，请你看看在那个节点上创建合适，scheduler收到信号以后，就开始做调度，并把调度后端结果反馈给apiserver，apiserver收到调度器的调度信息以后，它就把对应调度信息保存到etcd中，随后apiServer要发送一个watch信号给对应被调度的主机上的kubelet，对应主机上的kubelet收到消息后，立刻调用docker，并把对应容器跑起来；当容器运行起来以后，docker会向kubelet返回容器的状体；随后kubelet把容器的状态反馈给apiserver,由apiserver把容器的状态信息保存到etcd中；最后当etcd中的容器状态信息更新完成后，随后apiserver把容器状态信息更新完成的消息发送给对应主机的kubelet；



### 1.1.5. Pod终止过程

[![img](https://img2020.cnblogs.com/blog/1503305/202012/1503305-20201216145048637-28038042.jpg)](https://img2020.cnblogs.com/blog/1503305/202012/1503305-20201216145048637-28038042.jpg)

　　提示：用户通过客户端工具想APIserver发送删除pod的指令，在APIserver收到用户发来的删除指令后，首先APIserver会把对应的操作写到etcd中，并设置其宽限期，然后etcd把对应数据写好以后，响应APIserver，随后APIserver响应客户端说对应容器已经标记为terminating状态；随后APIserver会发送一个把对应pod标记为terminating状态的消息给endpoint端点控制，让其删除与当前要删除pod相关的所有service，（其实在k8s上我们创建service关联pod不是直接关联pod，是现关联endpoint端点控制器，然后端点控制器再关联pod），随后APIserver会向对应要删除pod所在主机上的kubelet发送将pod标记为terminating状态的消息，当对应主机收到APIserver发送的标记pod为terminating状态消息后，对应主机上的kubelet会向对应pod里运行的容器发送TERM信号，随后再执行preStop中定义的操作；随后等待宽限期超时，如果对应的pod还没有被删除，此时APIserver就会向对应pod所在主机上的kubelet发送宽限期超时的消息，此时对应kubelet会向对应容器发送SIGKILL信号来强制删除对应的容器，随后docker把对应容器删除后，把删除完容器的消息响应给APIserver，此时APIserver会向etcd发送删除对应pod在etcd中的所有信息；



## 1.2. Pod清单

### 1.2.1. apiversion/kind

```bash
apiVersion: v1
kind: Pod
```

### 1.2.2. metadata

```bash
metadata
    name        <string>            # 在一个名称空间内不能重复
    namespace   <string>            # 指定名称空间，默认defalut
    labels      <map[string]string> # 标签
    annotations <map[string]string> # 注释，不能作为被筛选
```

### 1.2.3. spec

```bash
spec
    containers  <[]Object> -required-   # 必选参数
        name    <string> -required-     # 指定容器名称，不可更新
        image   <string> -required-     # 指定镜像
        imagePullPolicy <string>        # 指定镜像拉取方式
            # Always: 始终从registory拉取镜像。如果镜像标签为latest，则默认值为Always
            # Never: 仅使用本地镜像
            # IfNotPresent: 本地不存在镜像时才去registory拉取。默认值
        env     <[]Object>              # 指定环境变量，使用 $(var) 引用,参考: configmap中模板
        command <[]string>              # 以数组方式指定容器运行指令，替代docker的ENTRYPOINT指令
        args    <[]string>              # 以数组方式指定容器运行参数，替代docker的CMD指令
        ports   <[]Object>              # 指定容器暴露的端口
            containerPort <integer> -required-  # 容器的监听端口
            name    <string>            # 为端口取名，该名称可以在service种被引用
            protocol  <string>          # 指定协议，默认TCP
            hostIP    <string>          # 绑定到宿主机的某个IP
            hostPort  <integer>         # 绑定到宿主机的端口
        readinessProbe <Object>         # 就绪性探测，确认就绪后提供服务
            initialDelaySeconds <integer>   # 容器启动后到开始就绪性探测中间的等待秒数
            periodSeconds <integer>     # 两次探测的间隔多少秒，默认值为10
            successThreshold <integer>  # 连续多少次检测成功认为容器正常，默认值为1。不支持修改
            failureThreshold <integer>  # 连续多少次检测成功认为容器异常，默认值为3
            timeoutSeconds   <integer>  # 探测请求超时时间
            exec    <Object>            # 通过执行特定命令来探测容器健康状态
                command <[]string>      # 执行命令，返回值为0表示健康，不自持shell模式
            tcpSocket <Object>          # 检测TCP套接字
                host <string>           # 指定检测地址，默认pod的IP
                port <string> -required-# 指定检测端口
            httpGet <Object>            # 以HTTP请求方式检测
                host    <string>        # 指定检测地址，默认pod的IP
                httpHeaders <[]Object>  # 设置请求头
                path    <string>        # 设置请求的location
                port <string> -required-# 指定检测端口
                scheme <string>         # 指定协议，默认HTTP
        livenessProbe   <Object>        # 存活性探测，确认pod是否具备对外服务的能力
            # 该对象中字段和readinessProbe一致
        lifecycle       <Object>        # 生命周期
            postStart   <Object>        # pod启动后钩子，执行指令或者检测失败则退出容器或者重启容器
                exec    <Object>        # 执行指令，参考readinessProbe.exec
                httpGet <Object>        # 执行HTTP，参考readinessProbe.httpGet
                tcpSocket <Object>      # 检测TCP套接字，参考readinessProbe.tcpSocket
            preStop     <Object>        # pod停止前钩子，停止前执行清理工作
                # 该对象中字段和postStart一致
    hostname    <string>                # 指定pod主机名
    nodeName    <string>                # 调度到指定的node节点
    nodeSelector    <map[string]string> # 指定预选的node节点
    hostIPC <boolean>                   # 使用宿主机的IPC名称空间，默认false
    hostNetwork <boolean>               # 使用宿主机的网络名称空间，默认false
    serviceAccountName  <string>        # Pod运行时的服务账号
    imagePullSecrets    <[]Object>      # 当拉取私密仓库镜像时，需要指定的密码密钥信息
        name            <string>        # secrets 对象名
```

### 1.2.4. k8s和image中的命令

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1579444125794-e27d08bb-7811-44c1-806b-86745f8f9093.png)

### 1.2.4. 就绪性探测和存活性探测

- 就绪性探测失败不会重启pod，只是让pod不处于ready状态。存活性探测失败会触发pod重启。
- 就绪性探测和存活性探测会持续进行下去，直到pod终止。



## 1.3 Pod实现机制



### 1.3.1 共享网络

容器本身之间是相互隔离的，是通过namespace和group实现隔离。

一个Pod中会存在多个容器，容器之间如何实现网络共享。

前提条件：容器在同一个ns里面。

Pod实现网络共享机制：

![](..\..\img\pod_network.png)

共享网络：

- 通过Pause容器，把其他业务容器加入到Pause容器里面，让所有业务容器在同一个名称空间中，可以实现网络共享



### 1.3.2 共享存储

引入数据卷概念Volume，使用数据卷进行持久化存储。

![](..\....\..\img\volume.png)

## 1.4 Pod镜像拉取策略

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx
      image: nginx:1.14
      imagePullPolicy: Always
```



- IfNotPresent：默认值，镜像在宿主机不存在时才拉取
- Always：每次创建Pod都会重新拉取一次镜像
- Nerver：Pod永远不会主动拉取这个镜像



## 1.5 Pod资源限制

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx
      image: nginx:1.14
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```



![](..\....\..\img\resources.png)

## 1.6 Pod重启机制

容器重启策略：

```yaml
spec:
  restartPolicy: [Always|Never|OnFailure] 
```

- Always：Pod一旦终止运行，kubelet都会进行重启，这也是默认值。
- Never：不会进行重启
- OnFailure：容器非正常退出（即是退出码不为0），kubelet会重启容器，反之不会重启。



PodSpec 中有一个 restartPolicy 字段，可能的值为 Always、OnFailure 和 Never。默认为 Always。

Always表示一旦不管以何种方式终止运行，kubelet都将重启；OnFailure表示只有Pod以非0退出码退出才重启；Nerver表示不再重启该Pod。

restartPolicy 适用于 Pod 中的所有容器。restartPolicy 仅指通过同一节点上的 kubelet 重新启动容器。失败的容器由 kubelet 以五分钟为上限的指数退避延迟（10秒，20秒，40秒…）重新启动，并在成功执行十分钟后重置。如 Pod 文档中所述，一旦pod绑定到一个节点，Pod 将永远不会重新绑定到另一个节点。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox
      image: busibox:1.28.4
      args:
      - /bin/sh
      - -c
      - sleep 36000
    restartPolicy: Never
```

- Always：当容器终止退出后，总是容器容器，默认策略
- OnFailure：当容器异常退出（退出状态码非0）时，才重启容器
- Never：当容器终止退出，从不重启容器。（场景：执行批量任务，只需执行一次）

## 1.7 Pod健康检查

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: busybox
      image: busibox:1.28.4
      args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
```

livenessProbe（存活检查）

- 如果检查失败，将杀死容器，根据Pod的restartPolicy来进行后续操作。

readinessProbe（就绪检查）

- 如果检查失败，Kubernetes会把Pod从service endpoints中剔除。



Probe支持以下三种检查方法：

- httpGet：发送HTTP请求，返回200-400范围状态码为成功。
- exec：执行Shell命令返回状态码是0位成功。
- tcpSocket：发起TCP Socket建立成功。

## 1.8 Pod调度策略



#### 创建Pod流程

![](..\....\..\img\createpod.png)



#### 影响Pod调度-资源限制、节点亲和性）

资源限制：Pod资源限制对Pod调用产生影响

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250"
```

根据request找到足够node节点进行调度。



#### 影响Pod调度-节点选择器

节点选择器标签：

1. 首先对节点创建标签

```bash
kubectl label node node1 env_role=prod
```

2. 在yaml文件中进行配置

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
spec:
  nodeSelector:  # 节点选择器
    env_role: dev
  containers:
  - name: nginx
    image: nginx:1.15
```

![](..\....\..\img\nodeselector.png)

#### 影响Pod调度-节点亲和性

节点亲和性：

nodeAffinity和之前的nodeSelector基本一样的，根据节点上标签约束来决定Pod调度到哪些节点上。

功能相较于nodeSelector更强大。里面还支持各种操作符。

1. 硬亲和性（约束条件必须满足）（支持常用操作符，In,NotIn，Exists，Gt，Lt，DoesNotExists。）

2. 软亲和性（尝试满足，不保证）

![](..\....\..\img\affinity.png)



#### 影响Pod调度-污点和污点容忍

nodeSelecor和nodeAffinity：Pod调度到某些节点上，Pod属性，调度时候实现

Taint污点：节点不做普通分配调度，是节点属性。



场景：

- 专用节点
- 配置特点硬件节点
- 基于Taint驱逐



示例：

1. 查看节点污点情况

```bash
kubectl describe node [node] | grep Taint
```

污点值有三个：

- NoSchedule：一定不被调度
- PreferNoSchdule：尽量不被调度
- NoExecute：不会调度，并且还会驱逐Node已有Pod

2. 为节点添加污点

```bash
kebuctl taint node [node] key=value:污点三个值
```

3. 删除污点

```bash
kubectl taint node [node] key:NoSchedule-node/[node] untainted
```



**污点容忍**

```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
    
  containers:
  - name: webdemo
    images: nginx
```



# 2、容器探针

探针是由 kubelet 对容器执行的定期诊断。要执行诊断，则需kubelet 调用由容器实现的 Handler。探针有三种类型的处理程序：

- ExecAction：在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。
- CPSocketAction：对指定端口上的容器的 IP 地址进行 TCP 检查。如果端口打开，则诊断被认为是成功的。
- HTTPGetAction：对指定的端口和路径上的容器的 IP 地址执行 HTTP Get 请求。如果响应的状态码大于等于200 且小于 400，则诊断被认为是成功的。

每次探测都将获得以下三种结果之一：

- 成功：容器通过了诊断。
- 失败：容器未通过诊断。
- 未知：诊断失败，因此不会采取任何行动。

Kubelet 可以选择是否在容器上运行三种探针执行和做出反应：

- livenessProbe：指示容器是否正在运行。如果存活探测失败，则 kubelet 会杀死容器，并且容器将受到其重启策略的影响。如果容器不提供存活探针，则默认状态为 Success。
- readinessProbe：指示容器是否准备好服务请求【对外接受请求访问】。如果就绪探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success。
- startupProbe: 指示容器中的应用是否已经启动。如果提供了启动探测(startup probe)，则禁用所有其他探测，直到它成功为止。如果启动探测失败，kubelet 将杀死容器，容器服从其重启策略进行重启。如果容器没有提供启动探测，则默认状态为成功Success。



## 2.1 存活（liveness）和就绪（readiness）探针的使用场景

如果容器中的进程能够在遇到问题或不健康的情况下自行崩溃，则不一定需要存活探针；kubelet 将根据 Pod 的restartPolicy 自动执行正确的操作。

如果你希望容器在探测失败时被杀死并重新启动，那么请指定一个存活探针，并指定restartPolicy 为 Always 或 OnFailure。

如果要仅在探测成功时才开始向 Pod 发送流量，请指定就绪探针。在这种情况下，就绪探针可能与存活探针相同，但是 spec 中的就绪探针的存在意味着 Pod 将在没有接收到任何流量的情况下启动，并且只有在探针探测成功后才开始接收流量。



## 2.2 Pod phase（阶段）

Pod 的 status 定义在 PodStatus 对象中，其中有一个 phase 字段。

Pod 的运行阶段（phase）是 Pod 在其生命周期中的简单宏观概述。该阶段并不是对容器或 Pod 的综合汇总，也不是为了做为综合状态机。

Pod 相位的数量和含义是严格指定的。除了本文档中列举的内容外，不应该再假定 Pod 有其他的 phase 值。

 

![img](https://img2020.cnblogs.com/blog/1395193/202008/1395193-20200816231550945-1643987940.png)

下面是 phase 可能的值：

- 挂起（Pending）：Pod 已被 Kubernetes 系统接受，但有一个或者多个容器镜像尚未创建。等待时间包括调度 Pod 的时间和通过网络下载镜像的时间，这可能需要花点时间。
- 运行中（Running）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
- 成功（Succeeded）：Pod 中的所有容器都被成功终止，并且不会再重启。
- 失败（Failed）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。
- 未知（Unknown）：因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败。



## 2.3 检测探针-就绪检测

pod yaml脚本

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat readinessProbe-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-httpdget-pod
  namespace: default
  labels:
    test: readiness-httpdget
spec:
  containers:
  - name: readiness-httpget
    image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
    imagePullPolicy: IfNotPresent
    readinessProbe:
      httpGet:
        path: /index1.html
        port: 80
      initialDelaySeconds: 5  #容器启动完成后，kubelet在执行第一次探测前应该等待 5 秒。默认是 0 秒，最小值是 0。
      periodSeconds: 3  #指定 kubelet 每隔 3 秒执行一次存活探测。默认是 10 秒。最小值是 1
```

创建 Pod，并查看pod状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f readinessProbe-httpget.yaml 
pod/readiness-httpdget-pod created
[root@k8s-master lifecycle]# kubectl get pod -n default -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
readiness-httpdget-pod   0/1     Running   0          5s    10.244.2.25   k8s-node02   <none>           <none>
```

查看pod详情

```bash
[root@k8s-master lifecycle]# kubectl describe pod readiness-httpdget-pod
Name:         readiness-httpdget-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 16:10:04 +0800
Labels:       test=readiness-httpdget
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"readiness-httpdget"},"name":"readiness-httpdget-pod","names...
Status:       Running
IP:           10.244.2.25
IPs:
  IP:  10.244.2.25
Containers:
  readiness-httpget:
    Container ID:   docker://066d66aaef191b1db08e1b3efba6a9be75378d2fe70e99400fc513b91242089c
………………
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 23 May 2020 16:10:05 +0800
    Ready:          False   ##### 状态为False
    Restart Count:  0
    Readiness:      http-get http://:80/index1.html delay=5s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False   ##### 为False
  ContainersReady   False   ##### 为False
  PodScheduled      True
Volumes:
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From                 Message
  ----     ------     ----               ----                 -------
  Normal   Scheduled  <unknown>          default-scheduler    Successfully assigned default/readiness-httpdget-pod to k8s-node02
  Normal   Pulled     49s                kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal   Created    49s                kubelet, k8s-node02  Created container readiness-httpget
  Normal   Started    49s                kubelet, k8s-node02  Started container readiness-httpget
  Warning  Unhealthy  2s (x15 over 44s)  kubelet, k8s-node02  Readiness probe failed: HTTP probe failed with statuscode: 404
```

由上可见，容器未就绪。

我们进入pod的第一个容器，然后创建对应的文件

```bash
[root@k8s-master lifecycle]# kubectl exec -it readiness-httpdget-pod -c readiness-httpget bash
root@readiness-httpdget-pod:/# cd /usr/share/nginx/html
root@readiness-httpdget-pod:/usr/share/nginx/html# ls
50x.html  index.html
root@readiness-httpdget-pod:/usr/share/nginx/html# echo "readiness-httpdget info" > index1.html
root@readiness-httpdget-pod:/usr/share/nginx/html# ls
50x.html  index.html  index1.html
```

之后看pod状态与详情

```bash
[root@k8s-master lifecycle]# kubectl get pod -n default -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
readiness-httpdget-pod   1/1     Running   0          2m30s   10.244.2.25   k8s-node02   <none>           <none>
[root@k8s-master lifecycle]# kubectl describe pod readiness-httpdget-pod
Name:         readiness-httpdget-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 16:10:04 +0800
Labels:       test=readiness-httpdget
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"readiness-httpdget"},"name":"readiness-httpdget-pod","names...
Status:       Running
IP:           10.244.2.25
IPs:
  IP:  10.244.2.25
Containers:
  readiness-httpget:
    Container ID:   docker://066d66aaef191b1db08e1b3efba6a9be75378d2fe70e99400fc513b91242089c
………………
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 23 May 2020 16:10:05 +0800
    Ready:          True     ##### 状态为True
    Restart Count:  0
    Readiness:      http-get http://:80/index1.html delay=5s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True   ##### 为True
  ContainersReady   True   ##### 为True
  PodScheduled      True
Volumes:
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                   From                 Message
  ----     ------     ----                  ----                 -------
  Normal   Scheduled  <unknown>             default-scheduler    Successfully assigned default/readiness-httpdget-pod to k8s-node02
  Normal   Pulled     2m33s                 kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal   Created    2m33s                 kubelet, k8s-node02  Created container readiness-httpget
  Normal   Started    2m33s                 kubelet, k8s-node02  Started container readiness-httpget
  Warning  Unhealthy  85s (x22 over 2m28s)  kubelet, k8s-node02  Readiness probe failed: HTTP probe failed with statuscode: 404
```

由上可见，容器已就绪。



## 2.4 检测探针-存活检测

### 存活检测-执行命令

pod yaml脚本

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat livenessProbe-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec-pod
  labels:
    test: liveness
spec:
  containers:
  - name: liveness-exec
    image: registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    imagePullPolicy: IfNotPresent
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5   # 第一次检测前等待5秒
      periodSeconds: 3   # 检测周期3秒一次
```

这个容器生命的前 30 秒，/tmp/healthy 文件是存在的。所以在这最开始的 30 秒内，执行命令 cat /tmp/healthy 会返回成功码。30 秒之后，执行命令 cat /tmp/healthy 就会返回失败状态码。

 

创建 Pod

```bash
[root@k8s-master lifecycle]# kubectl apply -f livenessProbe-exec.yaml 
pod/liveness-exec-pod created
```

在 30 秒内，查看 Pod 的描述：

```bash
[root@k8s-master lifecycle]# kubectl get pod -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
liveness-exec-pod   1/1     Running   0          17s   10.244.2.21   k8s-node02   <none>           <none>
[root@k8s-master lifecycle]# kubectl describe pod liveness-exec-pod
Name:         liveness-exec-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
………………
Events:
  Type    Reason     Age   From                 Message
  ----    ------     ----  ----                 -------
  Normal  Scheduled  25s   default-scheduler    Successfully assigned default/liveness-exec-pod to k8s-node02
  Normal  Pulled     24s   kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24" already present on machine
  Normal  Created    24s   kubelet, k8s-node02  Created container liveness-exec
  Normal  Started    24s   kubelet, k8s-node02  Started container liveness-exec
```

输出结果显示：存活探测器成功。

 

35 秒之后，再来看 Pod 的描述：

```bash
[root@k8s-master lifecycle]# kubectl get pod -o wide   # 显示 RESTARTS 的值增加了 1
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
liveness-exec-pod   1/1     Running   1          89s   10.244.2.22   k8s-node02   <none>           <none>
[root@k8s-master lifecycle]# kubectl describe pod liveness-exec-pod
………………
Events:
  Type     Reason     Age              From                 Message
  ----     ------     ----             ----                 -------
  Normal   Scheduled  42s              default-scheduler    Successfully assigned default/liveness-exec-pod to k8s-node02
  Normal   Pulled     41s              kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24" already present on machine
  Normal   Created    41s              kubelet, k8s-node02  Created container liveness-exec
  Normal   Started    41s              kubelet, k8s-node02  Started container liveness-exec
  Warning  Unhealthy  2s (x3 over 8s)  kubelet, k8s-node02  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    2s               kubelet, k8s-node02  Container liveness-exec failed liveness probe, will be restarted
```

由上可见，在输出结果的最下面，有信息显示存活探测器失败了，因此这个容器被杀死并且被重建了。

 

### 存活检测-HTTP请求

pod yaml脚本

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat livenessProbe-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-httpget-pod
  labels:
    test: liveness
spec:
  containers:
  - name: liveness-httpget
    image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    livenessProbe:
      httpGet:   # 任何大于或等于 200 并且小于 400 的返回码表示成功，其它返回码都表示失败。
        path: /index.html
        port: 80
        httpHeaders:  #请求中自定义的 HTTP 头。HTTP 头字段允许重复。
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 5
      periodSeconds: 3
```

创建 Pod，查看pod状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f livenessProbe-httpget.yaml 
pod/liveness-httpget-pod created
[root@k8s-master lifecycle]# kubectl get pod -n default -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
liveness-httpget-pod   1/1     Running   0          3s    10.244.2.27   k8s-node02   <none>           <none>
```

查看pod详情

```bash
[root@k8s-master lifecycle]# kubectl describe pod liveness-httpget-pod
Name:         liveness-httpget-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 16:45:25 +0800
Labels:       test=liveness
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"liveness"},"name":"liveness-httpget-pod","namespace":"defau...
Status:       Running
IP:           10.244.2.27
IPs:
  IP:  10.244.2.27
Containers:
  liveness-httpget:
    Container ID:   docker://4b42a351414667000fe94d4f3166d75e72a3401e549fed723126d2297124ea1a
………………
    Port:           80/TCP
    Host Port:      8080/TCP
    State:          Running
      Started:      Sat, 23 May 2020 16:45:26 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:80/index.html delay=5s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
………………
Events:
  Type    Reason     Age        From                 Message
  ----    ------     ----       ----                 -------
  Normal  Scheduled  <unknown>  default-scheduler    Successfully assigned default/liveness-httpget-pod to k8s-node02
  Normal  Pulled     5m52s      kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal  Created    5m52s      kubelet, k8s-node02  Created container liveness-httpget
  Normal  Started    5m52s      kubelet, k8s-node02  Started container liveness-httpget
```

由上可见，pod存活检测正常

 

我们进入pod的第一个容器，然后删除对应的文件

```bash
[root@k8s-master lifecycle]# kubectl exec -it liveness-httpget-pod -c liveness-httpget bash
root@liveness-httpget-pod:/# cd /usr/share/nginx/html/
root@liveness-httpget-pod:/usr/share/nginx/html# ls
50x.html  index.html
root@liveness-httpget-pod:/usr/share/nginx/html# rm -f index.html
root@liveness-httpget-pod:/usr/share/nginx/html# ls
50x.html
```

再次看pod状态和详情，可见Pod的RESTARTS从0变为了1。

```bash
[root@k8s-master lifecycle]# kubectl get pod -n default -o wide   # RESTARTS 从0变为了1
NAME                   READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
liveness-httpget-pod   1/1     Running   1          8m16s   10.244.2.27   k8s-node02   <none>           <none>
[root@k8s-master lifecycle]# kubectl describe pod liveness-httpget-pod
Name:         liveness-httpget-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 16:45:25 +0800
Labels:       test=liveness
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"liveness"},"name":"liveness-httpget-pod","namespace":"defau...
Status:       Running
IP:           10.244.2.27
IPs:
  IP:  10.244.2.27
Containers:
  liveness-httpget:
    Container ID:   docker://5d0962d383b1df5e59cd3d1100b259ff0415ac37c8293b17944034f530fb51c8
………………
    Port:           80/TCP
    Host Port:      8080/TCP
    State:          Running
      Started:      Sat, 23 May 2020 16:53:38 +0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 23 May 2020 16:45:26 +0800
      Finished:     Sat, 23 May 2020 16:53:38 +0800
    Ready:          True
    Restart Count:  1
    Liveness:       http-get http://:80/index.html delay=5s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                 From                 Message
  ----     ------     ----                ----                 -------
  Normal   Scheduled  <unknown>           default-scheduler    Successfully assigned default/liveness-httpget-pod to k8s-node02
  Normal   Pulled     7s (x2 over 8m19s)  kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal   Created    7s (x2 over 8m19s)  kubelet, k8s-node02  Created container liveness-httpget
  Normal   Started    7s (x2 over 8m19s)  kubelet, k8s-node02  Started container liveness-httpget
  Warning  Unhealthy  7s (x3 over 13s)    kubelet, k8s-node02  Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    7s                  kubelet, k8s-node02  Container liveness-httpget failed liveness probe, will be restarted
```

由上可见，当liveness-httpget检测失败，重建了Pod容器

 

### 存活检测-TCP端口

pod yaml脚本

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat livenessProbe-tcp.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp-pod
  labels:
    test: liveness
spec:
  containers:
  - name: liveness-tcp
    image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 3
```

#### TCP探测正常情况

创建 Pod，查看pod状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f livenessProbe-tcp.yaml
pod/liveness-tcp-pod created
[root@k8s-master lifecycle]# kubectl get pod -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
liveness-tcp-pod   1/1     Running   0          50s   10.244.4.23   k8s-node01   <none>           <none>
```

查看pod详情

```bash
[root@k8s-master lifecycle]# kubectl describe pod liveness-tcp-pod
Name:         liveness-tcp-pod
Namespace:    default
Priority:     0
Node:         k8s-node01/172.16.1.111
Start Time:   Sat, 23 May 2020 18:02:46 +0800
Labels:       test=liveness
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"test":"liveness"},"name":"liveness-tcp-pod","namespace":"default"}...
Status:       Running
IP:           10.244.4.23
IPs:
  IP:  10.244.4.23
Containers:
  liveness-tcp:
    Container ID:   docker://4de13e7c2e36c028b2094bf9dcf8e2824bfd15b8c45a0b963e301b91ee1a926d
………………
    Port:           80/TCP
    Host Port:      8080/TCP
    State:          Running
      Started:      Sat, 23 May 2020 18:03:04 +0800
    Ready:          True
    Restart Count:  0
    Liveness:       tcp-socket :80 delay=5s timeout=1s period=3s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                 Message
  ----    ------     ----       ----                 -------
  Normal  Scheduled  <unknown>  default-scheduler    Successfully assigned default/liveness-tcp-pod to k8s-node01
  Normal  Pulling    74s        kubelet, k8s-node01  Pulling image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17"
  Normal  Pulled     58s        kubelet, k8s-node01  Successfully pulled image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17"
  Normal  Created    57s        kubelet, k8s-node01  Created container liveness-tcp
  Normal  Started    57s        kubelet, k8s-node01  Started container liveness-tcp
```

以上是正常情况，可见存活探测成功。

 

#### 模拟TCP探测失败情况

将上面yaml文件中的探测TCP端口进行如下修改：

```yaml
livenessProbe:
  tcpSocket:
    port: 8090  # 之前是80
```

删除之前的pod并重新创建，并过一会儿看pod状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f livenessProbe-tcp.yaml 
pod/liveness-tcp-pod created
[root@k8s-master lifecycle]# kubectl get pod -o wide   # 可见RESTARTS变为了1，再过一会儿会变为2，之后依次叠加
NAME               READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
liveness-tcp-pod   1/1     Running   1          25s   10.244.2.28   k8s-node02   <none>           <none>
```

pod详情

```bash
[root@k8s-master lifecycle]# kubectl describe pod liveness-tcp-pod
Name:         liveness-tcp-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 18:08:32 +0800
Labels:       test=liveness
………………
Events:
  Type     Reason     Age                From                 Message
  ----     ------     ----               ----                 -------
  Normal   Scheduled  <unknown>          default-scheduler    Successfully assigned default/liveness-tcp-pod to k8s-node02
  Normal   Pulled     12s (x2 over 29s)  kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal   Created    12s (x2 over 29s)  kubelet, k8s-node02  Created container liveness-tcp
  Normal   Started    12s (x2 over 28s)  kubelet, k8s-node02  Started container liveness-tcp
  Normal   Killing    12s                kubelet, k8s-node02  Container liveness-tcp failed liveness probe, will be restarted
  Warning  Unhealthy  0s (x4 over 18s)   kubelet, k8s-node02  Liveness probe failed: dial tcp 10.244.2.28:8090: connect: connection refused
```



由上可见，liveness-tcp检测失败，重建了Pod容器。

 

### 检测探针-启动检测

有时候，会有一些现有的应用程序在启动时需要较多的初始化时间【如：Tomcat服务】。这种情况下，在不影响对触发这种探测的死锁的快速响应的情况下，设置存活探测参数是要有技巧的。

技巧就是使用一个命令来设置启动探测。针对HTTP 或者 TCP 检测，可以通过设置 failureThreshold * periodSeconds 参数来保证有足够长的时间应对糟糕情况下的启动时间。

**示例如下：**

pod yaml文件

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat startupProbe-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
  labels:
    test: startup
spec:
  containers:
  - name: startup
    image: registry.cn-beijing.aliyuncs.com/google_registry/tomcat:7.0.94-jdk8-openjdk
    imagePullPolicy: IfNotPresent
    ports:
    - name: web-port
      containerPort: 8080
      hostPort: 8080
    livenessProbe:
      httpGet:
        path: /index.jsp
        port: web-port
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 1
    startupProbe:
      httpGet:
        path: /index.jsp
        port: web-port
      periodSeconds: 10      #指定 kubelet 每隔 10 秒执行一次存活探测。默认是 10 秒。最小值是 1
      failureThreshold: 30   #最大的失败次数
```

启动pod，并查看状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f startupProbe-httpget.yaml 
pod/startup-pod created
[root@k8s-master lifecycle]# kubectl get pod -o wide
NAME          READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
startup-pod   1/1     Running   0          8m46s   10.244.4.26   k8s-node01   <none>           <none>
```

查看pod详情

```bash
[root@k8s-master ~]# kubectl describe pod startup-pod
```

有启动探测，应用程序将会有最多 5 分钟(30 * 10 = 300s) 的时间来完成它的启动。一旦启动探测成功一次，存活探测任务就会接管对容器的探测，对容器死锁可以快速响应。 如果启动探测一直没有成功，容器会在 300 秒后被杀死，并且根据 restartPolicy 来设置 Pod 状态。

## 2.5 探测器配置详解

使用如下这些字段可以精确的控制存活和就绪检测行为：

- initialDelaySeconds：容器启动后要等待多少秒后存活和就绪探测器才被初始化，默认是 0 秒，最小值是 0。
- periodSeconds：执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1。
- timeoutSeconds：探测的超时时间。默认值是 1 秒。最小值是 1。
- successThreshold：探测器在失败后，被视为成功的最小连续成功数。默认值是 1。存活探测的这个值必须是 1。最小值是 1。
- failureThreshold：当探测失败时，Kubernetes 的重试次数。存活探测情况下的放弃就意味着重新启动容器。就绪探测情况下的放弃 Pod 会被打上未就绪的标签。默认值是 3。最小值是 1。

HTTP 探测器可以在 httpGet 上配置额外的字段：

- host：连接使用的主机名，默认是 Pod 的 IP。也可以在 HTTP 头中设置 “Host” 来代替。
- scheme ：用于设置连接主机的方式（HTTP 还是 HTTPS）。默认是 HTTP。
- path：访问 HTTP 服务的路径。
- httpHeaders：请求中自定义的 HTTP 头。HTTP 头字段允许重复。
- port：访问容器的端口号或者端口名。如果数字必须在 1 ～ 65535 之间。



# 3、主容器生命周期事件的处理函数

Kubernetes 支持 postStart 和 preStop 事件。当一个主容器启动后，Kubernetes 将立即发送 postStart 事件；在主容器被终结之前，Kubernetes 将发送一个 preStop 事件。

 

## 3.1 postStart 和 preStop 处理函数示例

pod yaml文件

```yaml
[root@k8s-master lifecycle]# pwd
/root/k8s_practice/lifecycle
[root@k8s-master lifecycle]# cat lifecycle-events.yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo-pod
  namespace: default
  labels:
    test: lifecycle
spec:
  containers:
  - name: lifecycle-demo
    image: registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
    imagePullPolicy: IfNotPresent
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo 'Hello from the postStart handler' >> /var/log/nginx/message"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "echo 'Hello from the preStop handler'   >> /var/log/nginx/message"]
    volumeMounts:         #定义容器挂载内容
    - name: message-log   #使用的存储卷名称，如果跟下面volume字段name值相同，则表示使用volume的nginx-site这个存储卷
      mountPath: /var/log/nginx/  #挂载至容器中哪个目录
      readOnly: false             #读写挂载方式，默认为读写模式false
  initContainers:
  - name: init-myservice
    image: registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    command: ["/bin/sh", "-c", "echo 'Hello initContainers'   >> /var/log/nginx/message"]
    volumeMounts:         #定义容器挂载内容
    - name: message-log   #使用的存储卷名称，如果跟下面volume字段name值相同，则表示使用volume的nginx-site这个存储卷
      mountPath: /var/log/nginx/  #挂载至容器中哪个目录
      readOnly: false             #读写挂载方式，默认为读写模式false
  volumes:              #volumes字段定义了paues容器关联的宿主机或分布式文件系统存储卷
  - name: message-log   #存储卷名称
    hostPath:           #路径，为宿主机存储路径
      path: /data/volumes/nginx/log/    #在宿主机上目录的路径
      type: DirectoryOrCreate           #定义类型，这表示如果宿主机没有此目录则会自动创建
```

启动pod，查看pod状态

```bash
[root@k8s-master lifecycle]# kubectl apply -f lifecycle-events.yaml 
pod/lifecycle-demo-pod created
[root@k8s-master lifecycle]# kubectl get pod -o wide
NAME                 READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
lifecycle-demo-pod   1/1     Running   0          5s    10.244.2.30   k8s-node02   <none>           <none>
```

查看pod详情

```bash
[root@k8s-master lifecycle]# kubectl describe pod lifecycle-demo-pod
Name:         lifecycle-demo-pod
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Sat, 23 May 2020 22:08:04 +0800
Labels:       test=lifecycle
………………
Init Containers:
  init-myservice:
    Container ID:  docker://1cfabcb60b817efd5c7283ad9552dafada95dbe932f92822b814aaa9c38f8ba5
    Image:         registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    Image ID:      docker-pullable://registry.cn-beijing.aliyuncs.com/ducafe/busybox@sha256:f73ae051fae52945d92ee20d62c315306c593c59a429ccbbdcba4a488ee12269
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      echo 'Hello initContainers'   >> /var/log/nginx/message
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 23 May 2020 22:08:06 +0800
      Finished:     Sat, 23 May 2020 22:08:06 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx/ from message-log (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Containers:
  lifecycle-demo:
    Container ID:   docker://c07f7f3d838206878ad0bfeaec9b4222ac7d6b13fb758cc1b340ac43e7212a3a
    Image:          registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
    Image ID:       docker-pullable://registry.cn-beijing.aliyuncs.com/google_registry/nginx@sha256:7ac7819e1523911399b798309025935a9968b277d86d50e5255465d6592c0266
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 23 May 2020 22:08:07 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx/ from message-log (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  message-log:
    Type:          HostPath (bare host directory volume)
    Path:          /data/volumes/nginx/log/
    HostPathType:  DirectoryOrCreate
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From                 Message
  ----    ------     ----       ----                 -------
  Normal  Scheduled  <unknown>  default-scheduler    Successfully assigned default/lifecycle-demo-pod to k8s-node02
  Normal  Pulled     87s        kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24" already present on machine
  Normal  Created    87s        kubelet, k8s-node02  Created container init-myservice
  Normal  Started    87s        kubelet, k8s-node02  Started container init-myservice
  Normal  Pulled     86s        kubelet, k8s-node02  Container image "registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17" already present on machine
  Normal  Created    86s        kubelet, k8s-node02  Created container lifecycle-demo
  Normal  Started    86s        kubelet, k8s-node02  Started container lifecycle-demo
```

此时在k8s-node02查看输出信息如下：

```bash
[root@k8s-node02 log]# pwd
/data/volumes/nginx/log
[root@k8s-node02 log]# cat message 
Hello initContainers
Hello from the postStart handler
```

由上可知，init Container先执行，然后当一个主容器启动后，Kubernetes 将立即发送 postStart 事件。

停止该pod

```bash
[root@k8s-master lifecycle]# kubectl delete pod lifecycle-demo-pod
pod "lifecycle-demo-pod" deleted
```

 

此时在k8s-node02查看输出信息如下：

```bash
[root@k8s-node02 log]# pwd
/data/volumes/nginx/log
[root@k8s-node02 log]# cat message
Hello initContainers
Hello from the postStart handler
Hello from the preStop handler
```

由上可知，当在容器被终结之前， Kubernetes 将发送一个 preStop 事件。



# 4、Pod-Containers

containers是Pod中的容器列表，数组类型。

```yaml
spec:
  containers:  #容器列表
  - name: string  #容器名称
    image: string  #所用镜像
    imagePullPolicy: [Always|Never|IfNotPresent]  #镜像拉取策略
    command: [string]  #容器的启动命令列表
    args: [string]  #启动命令参数列表
    workingDir: string  #工作目录
    volumeMounts:  #挂载在容器内部的存储卷配置
    - name: string  #共享存储卷名称
      mountPath: string  #存储卷绝对路径
      readOnly: boolean  #是否只读
    ports:  #容器需要暴露的端口号列表
    - name: string  #端口名称
      containerPort: int  #容器监听端口
      hostPort: int  #映射宿主机端口
      protocol: string  #端口协议
    env:  #环境变量
    - name: string
      value: string
    resources:  #资源限制
      limits:
        cpu: string  #单位是core
        memory: string  #单位是MiB、GiB
    livenessProbe:  #探针，对Pod各容器健康检查的设置，如几次无回应，则会自动重启
      exec:
        command: [string]
      httpGet:
        path: string
        port: number
        host: string
        scheme: string
        httpHeaders:
        - name: string
          value: string
      tcpSocket:
        port: number
      initialDelaySeconds: 0  #启动后多久进行检测
      timeoutSeconds: 0  #超时时间
      periodSeconds: 0  #间隔时间
      successThreshold: 0  #
      failureThreshold: 0
    securityContext: #权限设置
      privileged: false  #是否允许创建特权模式的Pod
```

探针测试：

列出文件或文件夹aaa（此目录是不存在的），容器启动后5s开始执行探针，每隔5s执行一次。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    app: nginx-test
spec:
  containers:
  - name: nginx-test
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
    livenessProbe:
      exec:
        command: ["ls","aaa"]
      initialDelaySeconds: 5
      timeoutSeconds: 5
```

# 5、nodeSelector

指定Pod被调度到哪个节点运行。

```
spec:
  nodeSelector:
    K: V
```

　　

比如想把一个Pod调度给cnode-2节点运行：

获取集群中所有节点列表：

![img](https://p26-tt.byteimg.com/img/pgc-image/833f97cb0b104bd29ea23419c386372e~tplv-tt-shrink:640:0.image)

 

给cnode-2节点打标签：

```
kubectl label nodes/cnode-2 name=cnode-2
```

　　

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/d5d27b62994544c78761c7080019abfe~tplv-tt-shrink:640:0.image)

 

查看cnode-2节点标签信息：

![img](https://p9-tt-ipv6.byteimg.com/img/pgc-image/e87e1487dd8d47d2a8cb24917825283c~tplv-tt-shrink:640:0.image)

 

定义Pod的yaml文件：nginx-ns.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    app: nginx-test
spec:
  containers:
  - name: nginx-test
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
  nodeSelector:
    name: cnode-2
```

　　

使用如下命令创建Pod：

```
kubectl create -f nginx-ns.yaml
```

　　

查看这个Pod运行在哪个节点：

![img](https://p26-tt.byteimg.com/img/pgc-image/a4eaf0ba402449e4a3ff8054bebc496f~tplv-tt-shrink:640:0.image)

# 6、imagePullSecrets

拉取镜像时使用的Secret名称，以name：secretKey格式指定

```yaml
spec:
  imagePullSecrets:
    name: secretKey
```

Secret是用来保存私密凭据的，比如密码等信息



# 7、hostNetwork

是否使用主机网络模式

```yaml
spec:
  hostNetwork: true|false
```

　　

如果使用主机网络模式的话，Pod的IP就是跟宿主机IP是一样的

例如：创建下列Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  labels:
    app: nginx-test
spec:
  containers:
  - name: nginx-test
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
  nodeSelector:
    name: cnode-3
  hostNetwork: true
```

　　

然后查看Pod被分配的IP与主机IP是否相同

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/f9dcc2b31dae405eb1dd646520ae40e3~tplv-tt-shrink:640:0.image)

# 8、volumes

Pod上定义的共享存储列表：

```yaml
spec:
  volumes:  #存储卷
  - name: string
    emptyDir: {}  #表示与Pod同生命周期的一个临时目录
    hostPath:  #宿主机Host
      path: string
    secret: #挂载集群预定义的secret对象到容器内部
      secretName: string
      items:
      - key: string
        path: string
    configMap: #挂载集群预定义的configMap对象到容器内部
      name: string
      items:
      - key: string
        path: string
```



#  案例

**一般不会单独创建pod，而是通过控制器的方式创建。**

## 1. 创建简单pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: app
  labels:
    app: centos7
    release: stable
    environment: dev
spec:
  containers:
  - name: centos
    image: harbor.od.com/public/centos:7
    command:
    - /bin/bash
    - -c
    - "sleep 3600"
```



```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/pods/myapp.yaml
[root@hdss7-21 ~]# kubectl get pod -o wide -n app
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE                NOMINATED NODE   READINESS GATES
pod-demo   1/1     Running   0          16s   172.7.22.2   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# kubectl exec pod-demo -n app -- ps uax
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0   4364   352 ?        Ss   04:41   0:00 sleep 3600
root         11  0.0  0.0  51752  1696 ?        Rs   04:42   0:00 ps uax
```



```bash
[root@hdss7-21 ~]# kubectl describe pod pod-demo -n app | tail
Events:
  Type    Reason     Age    From                        Message
  ----    ------     ----   ----                        -------
  Normal  Scheduled  3m46s  default-scheduler           Successfully assigned app/pod-demo to hdss7-22.host.com
  Normal  Pulling    3m45s  kubelet, hdss7-22.host.com  Pulling image "harbor.od.com/public/centos:7"
  Normal  Pulled     3m45s  kubelet, hdss7-22.host.com  Successfully pulled image "harbor.od.com/public/centos:7"
  Normal  Created    3m45s  kubelet, hdss7-22.host.com  Created container centos
  Normal  Started    3m45s  kubelet, hdss7-22.host.com  Started container centos
```

## 2. 带健康检测的pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod-01
  namespace: app
  labels:
    app: centos7
    release: stable
    version: t1
spec:
  containers:
  - name: centos
    image: harbor.od.com/public/centos:7
    command:
    - /bin/bash
    - -c
    - "echo 'abc' > /tmp/health;sleep 60;rm -f /tmp/health;sleep 600"
    livenessProbe:
      exec:
        command:
        - /bin/bash
        - -c
        - "[ -f /tmp/health ]"
```


