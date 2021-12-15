[Kubernetes K8S之调度器kube-scheduler详解](https://www.cnblogs.com/zhanglianghhh/p/13875203.html)

# kube-scheduler调度概述

在 Kubernetes 中，调度是指将 Pod 放置到合适的 Node 节点上，然后对应 Node 上的 Kubelet 才能够运行这些 pod。

调度器通过 kubernetes 的 watch 机制来发现集群中新创建且尚未被调度到 Node 上的 Pod。调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 上来运行。调度器会依据下文的调度原则来做出调度选择。

调度是容器编排的重要环节，需要经过严格的监控和控制，现实生产通常对调度有各类限制，譬如某些服务必须在业务独享的机器上运行，或者从灾备的角度考虑尽量把服务调度到不同机器，这些需求在Kubernetes集群依靠调度组件kube-scheduler满足。

kube-scheduler是Kubernetes中的关键模块，扮演管家的角色遵从一套机制——为Pod提供调度服务，例如基于资源的公平调度、调度Pod到指定节点、或者通信频繁的Pod调度到同一节点等。容器调度本身是一件比较复杂的事，因为要确保以下几个目标：

- 公平性：在调度Pod时需要公平的进行决策，每个节点都有被分配资源的机会，调度器需要对不同节点的使用作出平衡决策。
- 资源高效利用：最大化群集所有资源的利用率，使有限的CPU、内存等资源服务尽可能更多的Pod。
- 效率问题：能快速的完成对大批量Pod的调度工作，在集群规模扩增的情况下，依然保证调度过程的性能。
- 灵活性：在实际运作中，用户往往希望Pod的调度策略是可控的，从而处理大量复杂的实际问题。因此平台要允许多个调度器并行工作，同时支持自定义调度器。

为达到上述目标，kube-scheduler通过结合Node资源、负载情况、数据位置等各种因素进行调度判断，确保在满足场景需求的同时将Pod分配到最优节点。显然，kube-scheduler影响着Kubernetes集群的可用性与性能，Pod数量越多集群的调度能力越重要，尤其达到了数千级节点数时，优秀的调度能力将显著提升容器平台性能。

 

# kube-scheduler调度流程

kube-scheduler的根本工作任务是根据各种调度算法将Pod绑定（bind）到最合适的工作节点，整个调度流程分为两个阶段：预选策略（Predicates）和优选策略（Priorities）。

**预选（Predicates）：**输入是所有节点，输出是满足预选条件的节点。kube-scheduler根据预选策略过滤掉不满足策略的Nodes。例如，如果某节点的资源不足或者不满足预选策略的条件如“Node的label必须与Pod的Selector一致”时则无法通过预选。

**优选（Priorities）：**输入是预选阶段筛选出的节点，优选会根据优先策略为通过预选的Nodes进行打分排名，选择得分最高的Node。例如，资源越富裕、负载越小的Node可能具有越高的排名。

通俗点说，调度的过程就是在回答两个问题：1. 候选有哪些？2. 其中最适合的是哪个？

值得一提的是，如果在预选阶段没有节点满足条件，Pod会一直处在Pending状态直到出现满足的节点，在此期间调度器会不断的进行重试。

 

## 预选策略（Predicates）

官网地址：[调度器预选、优选策略](https://kubernetes.io/docs/reference/scheduling/policies/)

过滤条件包含如下：

- PodFitsHostPorts：检查Pod容器所需的HostPort是否已被节点上其它容器或服务占用。如果已被占用，则禁止Pod调度到该节点。
- PodFitsHost：检查Pod指定的NodeName是否匹配当前节点。
- PodFitsResources：检查节点是否有足够空闲资源（例如CPU和内存）来满足Pod的要求。
- PodMatchNodeSelector：检查Pod的节点选择器(nodeSelector)是否与节点(Node)的标签匹配
- NoVolumeZoneConflict：对于给定的某块区域，判断如果在此区域的节点上部署Pod是否存在卷冲突。
- NoDiskConflict：根据节点请求的卷和已经挂载的卷，评估Pod是否适合该节点。
- MaxCSIVolumeCount：决定应该附加多少CSI卷，以及该卷是否超过配置的限制。
- CheckNodeMemoryPressure：如果节点报告内存压力，并且没有配置异常，那么将不会往那里调度Pod。
- CheckNodePIDPressure：如果节点报告进程id稀缺，并且没有配置异常，那么将不会往那里调度Pod。
- CheckNodeDiskPressure：如果节点报告存储压力(文件系统已满或接近满)，并且没有配置异常，那么将不会往那里调度Pod。
- CheckNodeCondition：节点可以报告它们有一个完全完整的文件系统，然而网络不可用，或者kubelet没有准备好运行Pods。如果为节点设置了这样的条件，并且没有配置异常，那么将不会往那里调度Pod。
- PodToleratesNodeTaints：检查Pod的容忍度是否能容忍节点的污点。
- CheckVolumeBinding：评估Pod是否适合它所请求的容量。这适用于约束和非约束PVC。

如果在predicates(预选)过程中没有合适的节点，那么Pod会一直在pending状态，不断重试调度，直到有节点满足条件。

经过这个步骤，如果有多个节点满足条件，就继续priorities过程，最后按照优先级大小对节点排序。

 

## 优选策略（Priorities）

包含如下优选评分条件：

- SelectorSpreadPriority：对于属于同一服务、有状态集或副本集（Service，StatefulSet or ReplicaSet）的Pods，会将Pods尽量分散到不同主机上。
- InterPodAffinityPriority：策略有podAffinity和podAntiAffinity两种配置方式。简单来说，就说根据Node上运行的Pod的Label来进行调度匹配的规则，匹配的表达式有：In, NotIn, Exists, DoesNotExist，通过该策略，可以更灵活地对Pod进行调度。
- LeastRequestedPriority：偏向使用较少请求资源的节点。换句话说，放置在节点上的Pod越多，这些Pod使用的资源越多，此策略给出的排名就越低。
- MostRequestedPriority：偏向具有最多请求资源的节点。这个策略将把计划的Pods放到整个工作负载集所需的最小节点上运行。
- RequestedToCapacityRatioPriority：使用默认的资源评分函数模型创建基于ResourceAllocationPriority的requestedToCapacity。
- BalancedResourceAllocation：偏向具有平衡资源使用的节点。
- NodePreferAvoidPodsPriority：根据节点注释scheduler.alpha.kubernet .io/preferAvoidPods为节点划分优先级。可以使用它来示意两个不同的Pod不应在同一Node上运行。
- NodeAffinityPriority：根据preferredduringschedulingignoredingexecution中所示的节点关联调度偏好来对节点排序。
- TaintTolerationPriority：根据节点上无法忍受的污点数量，为所有节点准备优先级列表。此策略将考虑该列表调整节点的排名。
- ImageLocalityPriority：偏向已经拥有本地缓存Pod容器镜像的节点。
- ServiceSpreadingPriority：对于给定的服务，此策略旨在确保Service的Pods运行在不同的节点上。总的结果是，Service对单个节点故障变得更有弹性。
- EqualPriority：赋予所有节点相同的权值1。
- EvenPodsSpreadPriority：实现择优 pod的拓扑扩展约束

 

# 自定义调度器

除了Kubernetes自带的调度器，我们也可以编写自己的调度器。通过spec.schedulername参数指定调度器名字，可以为Pod选择某个调度器进行调度。

如下Pod选择my-scheduler进行调度，而不是默认的default-scheduler

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulername: my-scheduler
  containers:
  - name: pod-with-second-annotation-container
    image: gcr.io/google_containers/pause:2.0
```

至于调度器如何编写，我们这里就不详细说了，工作中几乎不会使用到，有兴趣的同学可以自行查阅官网或其他资料。