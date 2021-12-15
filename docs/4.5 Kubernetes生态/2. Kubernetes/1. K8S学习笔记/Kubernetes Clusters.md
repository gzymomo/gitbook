# [Kubernetes Clusters](https://www.cnblogs.com/cjsblog/p/12050618.html)



1. create a Kubernetes cluster
2. Deploy an app
3. Explore your app
4. Expose your app publicly
5. Scale up your app
6. update your app



# 1. 创建集群

**Kubernetes集群** 

**Kubernetes协调一个高可用的计算机集群，作为一个单独的单元来一起工作。**有了这种抽象，在Kubernetes中你就可以将容器化的应用程序部署到集群中，而不必将它们特定地绑定到单独的机器上。为了利用这种新的部署模型，应用程序需要以一种将它们与单个主机解耦的方式打包：它们需要被容器化。与过去的部署模型（PS：应用程序被直接安装到特定的机器上）相比，容器化应用程序更加灵活和可用。Kubernetes以更高效的方式自动化分发和调度应用容器。Kubernetes是一个开源平台，可以投入生产。

Kubernetes集群由两类资源组成：

- **Master** 协调整个集群
- **Nodes** 运行应用程序的Workers 

集群大概是这样的：

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191225184219669-698078266.png)

**Master负责管理集群。**Master协调集群中的所有活动，比如调度应用程序、维护应用程序所需的状态、扩展应用程序和推送新的更新。

**在一个Kubernetes集群中，节点是一个虚拟机或物理机，它是作为worker存在的。**每个节点都有一个Kubelet，它是一个代理，用于管理该节点和master之间的通信。生产环境中，Kubernetes集群应该至少有三个节点。

当你部署应用程序到Kubernetes上时，你实际上是告诉master启动应用程序容器。master调度容器在集群节点上运行。节点和master之间的通信通过Kubernetes API来完成。终端用户还可以直接使用Kubernetes API与集群交互。



一个集群中有一个Master和多个Node。Master管理集群，负责集群中的所有活动。Node负责具体任务的执行，它是worker。每个Node上都有一个Kubelet，用于和Master通信。



# 2. 部署应用

**Kubernetes Deployments**

为了将容器化的应用部署到Kubernetes集群中，需要创建一个Kubernetes Deployment配置。Deployment指示Kubernetes如何创建和更新应用程序的实例。一旦创建了Deployment，Kubernetes master调度就会将应用程序实例放到集群中的各个节点上。 

应用程序实例被创建以后，Kubernetes Deployment Controller将会持续监视这些实例。如果承载实例的节点宕机或被删除，部署控制器将使用集群中另一个节点上的实例替换该实例。这就提供了一种自我修复机制来处理机器故障或维护。

在预先编排的世界中，安装脚本通常用于启动应用程序，但它们无法从机器故障中恢复。通过创建应用程序实例并让它们跨节点运行，Kubernetes部署为应用程序管理提供了一种完全不同的方法。

可以使用Kubernetes命令行接口Kubectl来创建和管理部署。Kubectl使用Kubernetes API与集群交互。

创建部署的时候，需要为应用程序指定容器镜像和要运行的副本数量。当然，后续可以通过更新部署来更改该信息。

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191225184154064-619175868.png)



**Deployment**就是将容器化的应用程序部署到Kubernetes集群中。这个很好理解，就是我们平时开发完打jar包部署到服务器上。

# 3. 查看应用

## 3.1. Pods

创建部署时，Kubernetes会创建一个Pod来承载应用程序实例。**一个Pod是一个Kubernetes抽象，它表示一组（一个或多个）应用程序容器（例如：Docker），以及这些容器的一些共享资源**。这些资源包括： 

- Shared storage, as Volumes
- Networking, as a unique cluster IP address
- Information about how to run each container, such as the container image version or specific ports to use 

Pod为一个特定应用程序的“逻辑主机”建模，并可以包含不同的应用程序容器，这些容器是相对紧密耦合的。例如，一个Pod可能包含Node.js应用程序的容器和一个不同的容器，后者提供Node.js web服务器要发布的数据。Pod中的容器共享一个IP地址和端口空间，总是同时定位和同时调度，并在同一节点上的共享上下文中运行。

Pods是Kubernetes平台上的原子单位。当我们在Kubernetes上创建部署的时候，这个部署会创建包含容器的Pods（而不是直接创建容器）。每个Pod都绑定到预定的节点，并一直保持到终止（根据重启策略）或删除。万一节点出现故障，集群中其他可用节点上调度相同的Pods将会被调度。

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191225191946476-553912242.png)

## 3.2. Nodes

**Pod总是在节点上运行**。节点是Kubernetes中的工作机，根据集群的不同，它可以是虚拟机，也可以是物理机。每个节点由Master管理。一个节点可以有多个pod， Kubernetes Master跨集群节点自动处理调度pod。Master的自动调度考虑到每个节点上的可用资源。

每个Kubernetes节点至少运行：

- Kubelet, a process responsible for communication between the Kubernetes Master and the Node; it manages the Pods and the containers running on a machine.
- A container runtime (like Docker, rkt) responsible for pulling the container image from a registry, unpacking the container, and running the application.

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191225192930342-1578038422.png) 

常用的kubelet命令：

```
# list resources
kubectl get
# show detailed information about a resource
kubectl describe 
# print the logs from a container in a pod
kubectl logs 
# execute a command on a container in a pod
kubectl exec
```

一个Pod是一组应用程序容器，包括运行这组容器所需的共享存储、IP等资源。可以这样理解，将一组容器打包在一起就是一个Pod。

Pod总是运行在Node上的。Node是一台物理或虚拟机。

如果容器之间是紧密耦合的，并且需要共享磁盘等资源，那么应该将它们放在单个（同一个）Pod中。

综上所述，我们不难理解：

- Node是一台物理机或虚拟机，是真正干活的工作机（worker） 
- 多个应用程序容器组成Pod
- Pod运行在Node上
- 多个Node组成一个集群
- Master负责管理集群，可以跨集群节点调度

也就是说，在节点上我们看到的是一个一个的Pod，而Pod里面是一个一个的容器



# 4. 发布应用

Pods也是有生命周期的。当一个工作节点（worker node）死亡时，该节点上运行的Pods也会随之丢失。然后，副本集可能通过创建新pod来动态地将集群恢复到所需的状态，以保持应用程序的运行。Kubernetes集群中的每个Pod都有一个惟一的IP地址，即使是在同一个节点上的Pods，因此需要一种方法来自动协调Pods之间的变化，以保证应用程序正常运行。

Kubernetes中的Service是一个抽象，它定义了一组逻辑Pods和访问它们的策略。Services支持有依赖关系的Pods之间的松散耦合。与所有Kubernetes对象一样，Service是使用YAML（首选）或JSON定义的。哪些Pods被选中用来组成一个Service通常是由标签选择器决定的。

**尽管每个Pod都有唯一的IP地址，但是如果没有Service，这些IP不会暴露在集群之外**。Service允许应用程序接收流量。通过在ServiceSpec中指定类型，可以以不同的方式公开服务：

- ClusterIP (default) - 在集群中的内网IP上公开服务。这种类型使得服务只能从集群内部访问。
- NodePort - 使用NAT在集群中每个选定节点的相同端口上公开服务。使用<NodeIP>:<NodePort>从集群外部访问服务。
- LoadBalancer - 在当前云中创建一个外部负载均衡器（如果支持），并为服务分配一个固定的外网IP。
- ExternalName - 通过返回带有名称的CNAME记录，使用任意名称（在规范中由externalName指定）公开服务。

## 4.1. Services and Labels

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226155704569-817512336.png) 

Service在一组Pod之间路由流量。Services是允许Pod在Kubernetes中死亡和复制而又不影响应用程序的抽象。Pods之间的发现和路由，由Kubernetes Services来处理。

Services通过使用标签和选择器匹配Pods。标签是附加到对象上的键/值对，可以以多种方式使用：

- 为开发、测试和生产指定对象
- 嵌入版本标记
- 使用标记对对象进行分类 

标签可以在对象创建时或以后附加到对象，可以随时修改它们。

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226155922542-1563861629.png) 

Kubernetes服务是一个抽象层，它定义一组Pods作为一个逻辑单元并为这些Pod启用外部流量公开，负载平衡和服务发现。

多个容器组成一个Pod，多个Pod组成一个服务。服务是一种更高层次的抽象，它通过标签和选择器匹配一些Pods，这些被选中的Pods形成一个服务，共同对外提供服务。



# 5. 扩展应用（扩容、伸缩）

## 5.1. Scaling an application

部署只创建了一个Pod来运行我们的应用程序。当流量增加时，我们需要扩展应用程序以满足用户需求。

**扩展是通过更改部署中的副本数量来实现的。**

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226171957424-1779716141.png) ![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226172032829-462927692.png)

扩展部署将确保创建新Pod并将其调度到具有可用资源的节点上。缩放会将Pod的数量增加到新的所需状态。Kubernetes还支持Pods的自动缩放。缩放到零也是可能的，它将终止指定Deployment的所有Pod。

运行一个应用程序的多个实例将需要一种将流量分配给所有实例的方法。 服务具有集成的负载均衡器，可以将网络流量分发到公开部署的所有Pod。服务将使用端点连续监视正在运行的Pod，以确保流量仅发送到可用Pod。

一旦运行了一个应用程序的多个实例，就可以在不停机的情况下进行滚动更新。

Scale（伸缩）是通过改变副本数量来实现的。伸缩会将Pod的数量增加到新的所需的状态。当应用从一个实例变成多个实例后，服务自带的负载均衡器会将流量分发到所有公开的Pods上。

# 6. 更新应用

## 6.1. Rolling updates

用户期望应用程序一直可用，而开发人员期望每天多次部署它们的新版本。在Kubernetes中，这是通过滚动更新来完成的。 滚动更新允许通过用新的Pod实例增量更新Pod实例，从而在零停机时间内进行Deployment的更新。新的Pod将被调度到具有可用资源的节点上。

默认情况下，在更新过程中不可用的Pod的最大数量和可以创建的新Pod的最大数量为1。这两个选项都可以配置为数字或百分比（按Pod）。 在Kubernetes中，对更新进行版本控制，并且任何部署更新都可以还原为先前（稳定）的版本。

![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226183406611-910850539.png)![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226183450360-968044074.png)

 ![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226183609699-982962007.png)![img](https://img2018.cnblogs.com/blog/874963/201912/874963-20191226183640756-433873444.png)

与应用程序扩展类似，如果公开部署，则该服务将在更新过程中仅将流量负载均衡到可用Pod。可用的Pod是可供应用程序用户使用的实例。 

滚动更新允许执行以下操作：

- 将应用程序从一种环境升级到另一种环境（通过容器镜像更新）
- 回滚到以前的版本
- 持续集成和持续交付应用程序，停机时间为零

滚动更新允许通过用新的Pod实例增量更新Pod实例，从而在实现在不停机（服务不中断）的情况下内进行Deployment的更新。

如果部署是公开公开的，则该服务将在更新期间仅将流量负载平衡到可用的Pod。

总之，一句话，**滚动更新可以实现平滑升级（平滑上线）**。（PS：可以联想一下Nginx） 