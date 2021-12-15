# 为什么要总结k8s yaml

1.k8s 环境部署应用后因为缺少某种配置参数导致应用出现不稳定问题
2.yaml配置项多而杂乱每一个参数在什么场景中使用都未知.
3.发布应用统一使用YAML模板

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ops-nginx-api  # Deployment 对象的名称，与应用名称保持一致
  namespace: default    #命名空间
  labels:
    appName: ops-nginx-api  # 应用名称
spec:
  selector:
    matchLabels:
      app: ops-nginx-api  #app 标签名称
  replicas: 5 #Pod
  strategy: #部署策略更多策略 1.https://www.qikqiak.com/post/k8s-deployment-strategies/
    type: RollingUpdate #其他类型如下 1.重建(Recreate) 开发环境使用 2.RollingUpdate(滚动更新)
    rollingUpdate:
       maxSurge: 25%        #一次可以添加多少个Pod
       maxUnavailable: 25%  #滚动更新期间最大多少个Pod不可用
  template:
    metadata:
      labels:
        app: 'ops-nginx-api'
    spec:
      terminationGracePeriodSeconds: 120 #优雅关闭时间，这个时间内优雅关闭未结束，k8s 强制 kill
      affinity:
        podAntiAffinity:  # pod反亲和性，尽量避免同一个应用调度到相同node
          preferredDuringSchedulingIgnoredDuringExecution: #硬需求 1.preferredDuringSchedulingIgnoredDuringExecution 软需求
            - weight: 100
              #weight 字段值的 范围是 1-100。 对于每个符合所有调度要求（资源请求、RequiredDuringScheduling 亲和性表达式等） 的节点，调度器将遍历该字段的元素来计算总和，并且如果节点匹配对应的 MatchExpressions，则添加“权重”到总和。 然后将这个评分与该节点的其他优先级函数的评分进行组合。 总分最高的节点是最优选的。
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - ops-nginx-api
                topologyKey: "kubernetes.io/hostname"

      initContainers:  #初始化容器
        - name: sidecar-sre #init 容器名称
          image: breaklinux/sidecar-sre:201210129  #docker hup仓库镜像
          imagePullPolicy: IfNotPresent  #镜像拉取策略 1.IfNotPresent如果本地存在镜像就优先使用本地镜像。2.Never直接不再去拉取镜像了,使用本地的.如果本地不存在就报异常了。
          #3.imagePullPolicy 未被定义为特定的值，默认值设置为 Always 本地是否存在都会去仓库拉取镜像.
          env:
            #Downward API官网示例 https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/
            - name: CHJ_NODE_IP
              valueFrom:
                fieldRef:  #这两种呈现 Pod 和 Container 字段的方式统称为 Downward API。
                  fieldPath: status.hostIP #获取pod 所在node IP地址设置为CHJ_NODE_IP

            - name: CHJ_POD_IP  #变量名称
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP #获取pod自身ip地址设置为CHJ_POD_IP变量名称

            - name: CHJ_APP_NAME   #应用名称EVN
              value: 'ops-nginx-api' #环境变量值

          command: ["/bin/sh","-c"]  #shell 执行
          args: ["mkdir -p /scratch/.sidecar-sre && cp /sidecar/post-start.sh /scratch/.sidecar-sre/post-start.sh && /scratch/.sidecar-sre/post-start.sh"]

          volumeMounts: #挂载日志卷和存在的空镜像
            - name: log-volume
              mountPath: /tmp/data/log/ops-nginx-api  #容器内挂载镜像路径
            - name: sidecar-sre
              mountPath: /scratch  #空镜像

          resources:  #qos 设置
            limits:
              cpu: 100m #pod 占单核cpu 1/10
              memory: 100Mi #内存100M
        #hostPath 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。
        volumes:
          - name: log-volume  #卷名称
            hostPath:         #卷类型详细见:https://kubernetes.io/zh/docs/concepts/storage/volumes/
              path: /data/logs/prod/ops-nginx-api  #宿主机存在的目录路径
              type: DirectoryOrCreate #如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息
          - name: sidecar-sre  #
            emptyDir: {} #emptyDir 卷的存储介质（磁盘、SSD 等）是由保存 kubelet 数据的根目录 （通常是 /var/lib/kubelet）的文件系统的介质确定。

      containers:
        - name: ops-nginx-api # 容器名称，与应用名称保持一致
          image: breaklinux/op-flask-api:v1 #遵守镜像命名规范
          imagePullPolicy: Always  #镜像拉取策略 1.IfNotPresent如果本地存在镜像就优先使用本地镜像。2.Never直接不再去拉取镜像了,使用本地的.如果本地不存在就报异常了。
          #3.imagePullPolicy 未被定义为特定的值，默认值设置为 Always 本地是否存在都会去仓库拉取镜像.
          lifecycle: #Kubernetes 支持 postStart 和 preStop 事件。 当一个容器启动后，Kubernetes 将立即发送 postStart 事件；在容器被终结之前， Kubernetes 将发送一个 preStop 事件。
            postStart: #容器创建成功后，运行前的任务，用于资源部署、环境准备等。
              exec:
                command:
                  - /bin/sh
                  - -c
                  - echo 'Hello from the postStart handler' >> /var/log/nginx/message
            preStop: #在容器被终止前的任务，用于优雅关闭应用程序、通知其他系统等等
              exec:
                command:
                  - sh
                  - -c
                  - sleep 30

          livenessProbe:  #存活探针器配置
            failureThreshold: 3 #处于成功时状态时，探测操作至少连续多少次的失败才被视为检测不通过，显示为#failure属性.默认值为3,最小值为 1,存活探测情况下的放弃就意味着重新启动容器。
            httpGet: #1.存活探针器三种方式 1.cmd命令方式进行探测 2.http 状态码方式 3.基于tcp端口探测
              path: /healthy #k8s源码中healthz 实现 https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go
              port: 8080    #应用程序监听端口
            initialDelaySeconds: 600 #存活性探测延迟时长,即容器启动多久之后再开始第一次探测操作,显示为delay属性.默认值为0，即容器启动后立刻便开始进行探测.
            periodSeconds: 10  #执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1秒,过高的频率会对Pod对象带来较大的额外开销,而过低的频率又会使得对错误的反应不及时.
            successThreshold: 1 #处于失败状态时,探测操作至少连续多少次的成功才被人为是通过检测，显示为#success属性,默认值为1，最小值也是1
            timeoutSeconds: 3 #存活性探测的超时时长,显示为timeout属性,默认值1s,最小值也是1s

          readinessProbe:  #定义就绪探测器
            failureThreshold: 3 #处于成功时状态时，探测操作至少连续多少次的失败才被视为检测不通过，显示为#failure属性.默认值为3,最小值为  就绪探测情况下的放弃 Pod 会被打上未就绪的标签.

            tcpSocket: # 1.就绪探针三种方式 1.cmd命令方式进行探测 2.http 状态码方式 3.基于tcp端口探测
              port: 8080 #应用程序监听端口
            initialDelaySeconds: 10 #执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1秒,过高的频率会对Pod对象带来较大的额外开销,而过低的频率又会使得对错误的反应不及时.
            periodSeconds: 10  #执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1秒,过高的频率会对Pod对象带来较大的额外开销,而过低的频率又会使得对错误的反应不及时
            successThreshold: 1 #处于失败状态时,探测操作至少连续多少次的成功才被人为是通过检测，显示为#success属性,默认值为1，最小值也是1
            timeoutSeconds: 3  #存活性探测的超时时长,显示为timeout属性,默认值1s,最小值也是1s
          ports:
            - containerPort: 19201 #应用监听的端口
              protocol: TCP #协议 tcp和 udp
          env:  #应用配置中心环境变量
            - name: ENV
              value: test
            - name: apollo.meta
              value: http://saos-apollo-config-service:8080
            - name: CHJ_APP_NAME
              value: 'ops-nginx-api'
            - name: LOG_BASE
              value: '/chj/data/log'
            - name: RUNTIME_CLUSTER
              value: 'default'
          #Request: 容器使用的最小资源需求，作为容器调度时资源分配的判断依赖。只有当节点上可分配资源量>=容器资源请求数时才允许将容器调度到该节点。但Request参数不限制容器的最大可使用资源。
          #Limit: 容器能使用资源的资源的最大值，设置为0表示使用资源无上限。
          resources:  #qos限制 1.QoS 主要分为Guaranteed、Burstable 和 Best-Effort三类，优先级从高到低
            requests:
              memory: 4096Mi #内存4G
              cpu: 500m #cpu 0.5
            limits:
              memory: 4096Mi
              cpu: 500m
          volumeMounts: #挂载苏宿主机目录到制定
            - name: log-volume  # sidecar-sre
              mountPath: /tmp/data/log/ops-nginx-api #该目录作为程序日志sidecar路径收集
            - name: sidecar-sre
              mountPath: /chj/data  #苏主机挂载目录
      imagePullSecrets:  #在 Pod 中设置 ImagePullSecrets 只有提供自己密钥的 Pod 才能访问私有仓库
        - name: lixiang-images-pull  #镜像Secrets需要在集群中手动创建
```