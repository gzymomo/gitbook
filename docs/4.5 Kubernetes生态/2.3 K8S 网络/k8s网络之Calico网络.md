博客园：

- [k8s网络之Calico网络](https://www.cnblogs.com/goldsunshine/p/10701242.html)
- [kubernetes集群网络](https://www.cnblogs.com/yuezhimi/p/13042037.html)



# 一、简介

## Kubernetes网络模型

Kubernetes 要求所有的网络插件实现必须满足如下要求：

- 一个Pod一个IP
- 所有的 Pod 可以与任何其他 Pod 直接通信，无需使用 NAT 映射
- 所有节点可以与所有 Pod 直接通信，无需使用 NAT 映射
- Pod 内部获取到的 IP 地址与其他 Pod 或节点与其通信时的 IP 地址是同一个。

## Docker容器网络模型

先看下Linux网络名词：

- **网络的命名空间：**Linux在网络栈中引入网络命名空间，将独立的网络协议栈隔离到不同的命令空间中，彼此间无法通信；Docker利用这一特性，实现不同容器间的网络隔离。
- **Veth设备对：**Veth设备对的引入是为了实现在不同网络命名空间的通信。
- **Iptables/Netfilter：**Docker使用Netfilter实现容器网络转发。
- **网桥：**网桥是一个二层网络设备，通过网桥可以将Linux支持的不同的端口连接起来，并实现类似交换机那样的多对多的通信。
- **路由：**Linux系统包含一个完整的路由功能，当IP层在处理数据发送或转发的时候，会使用路由表来决定发往哪里。

Docker容器网络示意图如下：

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604101615637-1824228148.png)

 

## Pod 网络

**问题：**Pod是K8S最小调度单元，一个Pod由一个容器或多个容器组成，当多个容器时，怎么都用这一个Pod IP？

**实现：**k8s会在每个Pod里先启动一个infra container小容器，然后让其他的容器连接进来这个网络命名空间，然后其他容器看到的网络试图就完全一样了。即网络设备、IP地址、Mac地址等。这就是解决网络共享的一种解法。在Pod的IP地址就是infra container的IP地址。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604101755019-1974608469.png)

在 Kubernetes 中，每一个 Pod 都有一个真实的 IP 地址，并且每一个 Pod 都可以使用此 IP 地址与 其他 Pod 通信。

Pod之间通信会有两种情况：

- 两个Pod在同一个Node上
- 两个Pod在不同Node上

**先看下第一种情况：两个Pod在同一个Node上**

同节点Pod之间通信道理与Docker网络一样的，如下图：

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604101857742-926691651.gif)

 

1. 对 Pod1 来说，eth0 通过虚拟以太网设备（veth0）连接到 root namespace；
2. 网桥 cbr0 中为 veth0 配置了一个网段。一旦数据包到达网桥，网桥使用ARP 协议解析出其正确的目标网段 veth1；
3. 网桥 cbr0 将数据包发送到 veth1；
4. 数据包到达 veth1 时，被直接转发到 Pod2 的 network namespace 中的 eth0 网络设备。

**再看下第二种情况：两个Pod在不同Node上**

K8S网络模型要求Pod IP在整个网络中都可访问，这种需求是由第三方网络组件实现。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604102028160-1693817067.gif)

 

## CNI（容器网络接口）

CNI（Container Network Interface，容器网络接口)：是一个容器网络规范，Kubernetes网络采用的就是这个CNI规范，CNI实现依赖两种插件，一种CNI Plugin是负责容器连接到主机，另一种是IPAM负责配置容器网络命名空间的网络。

CNI插件默认路径：

```bash
# ls /opt/cni/bin/
```

地址：https://github.com/containernetworking/cni

当你在宿主机上部署Flanneld后，flanneld 启动后会在每台宿主机上生成它对应的CNI 配置文件（它其实是一个 ConfigMap），从而告诉Kubernetes，这个集群要使用 Flannel 作为容器网络方案。

CNI配置文件路径：

```bash
/etc/cni/net.d/10-flannel.conflist
```

当 kubelet 组件需要创建 Pod 的时候，先调用dockershim它先创建一个 Infra 容器。然后调用 CNI 插件为 Infra 容器配置网络。

这两个路径在kubelet启动参数中定义：

```bash
 --network-plugin=cni \
 --cni-conf-dir=/etc/cni/net.d \
 --cni-bin-dir=/opt/cni/bin
```

 

## Kubernetes网络组件之 Flannel

Flannel是CoreOS维护的一个网络组件，Flannel为每个Pod提供全局唯一的IP，Flannel使用ETCD来存储Pod子网与Node IP之间的关系。flanneld守护进程在每台主机上运行，并负责维护ETCD信息和路由数据包。

### Flannel 部署

https://github.com/coreos/flannel

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Flannel工作模式及原理

Flannel支持多种数据转发方式：

- UDP：最早支持的一种方式，由于性能最差，目前已经弃用。
- VXLAN：Overlay Network方案，源数据包封装在另一种网络包里面进行路由转发和通信
- Host-GW：Flannel通过在各个节点上的Agent进程，将容器网络的路由信息刷到主机的路由表上，这样一来所有的主机都有整个容器网络的路由数据了。
- Directrouting(vxlan+host-gw)

```yaml
# kubeadm部署指定Pod网段
kubeadm init --pod-network-cidr=10.244.0.0/16

# 二进制部署指定
cat /opt/kubernetes/cfg/kube-controller-manager.conf
--allocate-node-cidrs=true \
--cluster-cidr=10.244.0.0/16 \

#配置文件 kube-flannel.yml
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```



为了能够在二层网络上打通“隧道”，VXLAN 会在宿主机上设置一个特殊的网络设备作为“隧道”的两端。这个设备就叫作 VTEP，即：VXLAN Tunnel End Point（虚拟隧道端点）。

 如果Pod 1访问Pod 2，源地址10.244.2.250，目的地址10.244.1.33 ，数据包传输流程如下：



```bash
1、容器路由：容器根据路由表从eth0发出
# kubectl get pod -o wide
NAME                                      READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
busybox-6cd57fd969-4n6fn                  1/1     Running   0          61s     10.244.2.50   k8s-master1   <none>           <none>
web-5675686b8-rc9bf                       1/1     Running   0          7m12s   10.244.1.33   k8s-node01    <none>           <none>
web-5675686b8-rrc6f                       1/1     Running   0          7m12s   10.244.2.49   k8s-master1   <none>           <none>
# kubectl exec -it busybox-6cd57fd969-4n6fn sh
/ # ip route
default via 10.244.2.1 dev eth0 
10.244.0.0/16 via 10.244.2.1 dev eth0 
10.244.2.0/24 dev eth0 scope link  src 10.244.2.51

2、主机路由：数据包进入到宿主机虚拟网卡cni0，根据路由表转发到flannel.1虚拟网卡，也就是，来到了隧道的入口。
# ip route
default via 192.168.0.1 dev ens32 proto static metric 100 
10.244.0.0/24 via 10.244.0.0 dev flannel.1 onlink 
10.244.1.0/24 via 10.244.1.0 dev flannel.1 onlink 

3、VXLAN封装：而这些VTEP设备（二层）之间组成二层网络必须要知道目的MAC地址。这个MAC地址从哪获取到呢？其实在flanneld进程启动后，就会自动添加其他节点ARP记录，可以通过ip命令查看，如下所示：
# ip neigh show dev flannel.1
10.244.0.0 lladdr 06:6a:9d:15:ac:e0 PERMANENT
10.244.1.0 lladdr 36:68:64:fb:4f:9a PERMANENT

4、二次封包：知道了目的MAC地址，封装二层数据帧（容器源IP和目的IP）后，对于宿主机网络来说这个帧并没有什么实际意义。接下来，Linux内核还要把这个数据帧进一步封装成为宿主机网络的一个普通数据帧，好让它载着内部数据帧，通过宿主机的eth0网卡进行传输。

5、封装到UDP包发出去：现在能直接发UDP包嘛？到目前为止，我们只知道另一端的flannel.1设备的MAC地址，却不知道对应的宿主机地址是什么。
flanneld进程也维护着一个叫做FDB的转发数据库，可以通过bridge fdb命令查看：
# bridge fdb show  dev flannel.1
06:6a:9d:15:ac:e0 dst 192.168.0.134 self permanent
36:68:64:fb:4f:9a dst 192.168.0.133 self permanent
可以看到，上面用的对方flannel.1的MAC地址对应宿主机IP，也就是UDP要发往的目的地。使用这个目的IP进行封装。

6、数据包到达目的宿主机：Node1的eth0网卡发出去，发现是VXLAN数据包，把它交给flannel.1设备。flannel.1设备则会进一步拆包，取出原始二层数据帧包，发送ARP请求，经由cni0网桥转发给container。
```



### Host-GW

host-gw模式相比vxlan简单了许多， 直接添加路由，将目的主机当做网关，直接路由原始封包。



```yaml
# kube-flannel.yml
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```



当你设置flannel使用host-gw模式,flanneld会在宿主机上创建节点的路由表：

```bash
# ip route
default via 192.168.0.1 dev ens32 proto static metric 100
10.244.0.0/24 via 192.168.0.134 dev ens32
10.244.1.0/24 via 192.168.0.133 dev ens32
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.0.0/24 dev ens32 proto kernel scope link src 192.168.0.132 metric 100
```

目的 IP 地址属于 10.244.1.0/24 网段的 IP 包，应该经过本机的 eth0 设备发出去（即：dev eth0）；并且，它下一跳地址是 192.168.0.132。一旦配置了下一跳地址，那么接下来，当 IP 包从网络层进入链路层封装成帧的时候，eth0 设备就会使用下一跳地址对应的 MAC 地址，作为该数据帧的目的 MAC 地址。

而 Node 2 的内核网络栈从二层数据帧里拿到 IP 包后，会“看到”这个 IP 包的目的 IP 地址是container-2 的 IP 地址。这时候，根据 Node 2 上的路由表，该目的地址会匹配到第二条路由规则，从而进入 cni0 网桥，进而进入到 container-2 当中。

### Directrouting



```bash
# kubectl edit cm kube-flannel-cfg  -n kube-system
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
        "Directrouting": true
      }
    }

# kubectl get cm kube-flannel-cfg -o json -n kube-system
        "net-conf.json": "{\n  \"Network\": \"10.244.0.0/16\",\n  \"Backend\": {\n    \"Type\": \"vxlan\"\n    \"Directrouting\": true\n  }\n}\n"
```



小结：
1、vxlan 不受网络环境限制，只要三层可达就行
2、vxlan 需要二层解封包，降低工作效率
3、hostgw 基于路由表转发，效率更高
4、hostgw 只适用于二层网络（本身网络架构受限，节点数量也受限）



# 二、Calico

## 2.1 Calico简介

Calico 是一个纯三层的数据中心网络方案，是一种容器之间互通的网络方案。在虚拟化平台中，比如 OpenStack、Docker 等都需要实现 workloads 之间互连，但同时也需要对容器做隔离控制，就像在 Internet 中的服务仅开放80端口、公有云的多租户一样，提供隔离和管控机制。而在多数的虚拟化平台实现中，通常都使用二层隔离技术来实现容器的网络，这些二层的技术有一些弊端，比如需要依赖 VLAN、bridge 和隧道等技术，其中 bridge 带来了复杂性，vlan 隔离和 tunnel 隧道则消耗更多的资源并对物理环境有要求，随着网络规模的增大，整体会变得越加复杂。我们尝试把 Host 当作 Internet 中的路由器，同样使用 BGP 同步路由，并使用 iptables 来做安全访问策略，最终设计出了 Calico 方案。



Calico 在每一个计算节点利用 Linux Kernel 实现了一个高效的虚拟路由器（ vRouter） 来负责数据转发，而每个 vRouter 通过 BGP 协议负责把自己上运行的 workload 的路由信息向整个 Calico 网络内传播。

此外，Calico 项目还实现了 Kubernetes 网络策略，提供ACL功能。



**适用场景**：k8s环境中的pod之间需要隔离

**设计思想**：Calico 不使用隧道或 NAT 来实现转发，而是巧妙的把所有二三层流量转换成三层流量，并通过 host 上路由配置完成跨 Host 转发。

**设计优势**：

1.更优的资源利用

二层网络通讯需要依赖广播消息机制，广播消息的开销与 host 的数量呈指数级增长，Calico 使用的三层路由方法，则完全抑制了二层广播，减少了资源开销。

另外，二层网络使用 VLAN 隔离技术，天生有 4096 个规格限制，即便可以使用 vxlan 解决，但 vxlan 又带来了隧道开销的新问题。而 Calico 不使用 vlan 或 vxlan 技术，使资源利用率更高。

2.可扩展性

Calico 使用与 Internet 类似的方案，Internet 的网络比任何数据中心都大，Calico 同样天然具有可扩展性。

3.简单而更容易 debug

因为没有隧道，意味着 workloads 之间路径更短更简单，配置更少，在 host 上更容易进行 debug 调试。

4.更少的依赖

Calico 仅依赖三层路由可达。

5.可适配性

Calico 较少的依赖性使它能适配所有 VM、Container、白盒或者混合环境场景。



## 2.2 Calico架构

架构图：

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413152300545-538840176.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413152300545-538840176.png)

 

Calico网络模型主要工作组件：

1.Felix：运行在每一台 Host 的 agent 进程，主要负责网络接口管理和监听、路由、ARP 管理、ACL 管理和同步、状态上报等。

2.etcd：分布式键值存储，主要负责网络元数据一致性，确保Calico网络状态的准确性，可以与kubernetes共用；

3.BGP Client（BIRD）：Calico 为每一台 Host 部署一个 BGP Client，使用 BIRD 实现，BIRD 是一个单独的持续发展的项目，实现了众多动态路由协议比如 BGP、OSPF、RIP 等。在 Calico 的角色是监听 Host 上由 Felix 注入的路由信息，然后通过 BGP 协议广播告诉剩余 Host 节点，从而实现网络互通。

4.BGP Route Reflector：在大型网络规模中，如果仅仅使用 BGP client 形成 mesh 全网互联的方案就会导致规模限制，因为所有节点之间俩俩互联，需要 N^2 个连接，为了解决这个规模问题，可以采用 BGP 的 Router Reflector 的方法，使所有 BGP Client 仅与特定 RR 节点互联并做路由同步，从而大大减少连接数。

 

## 2.3 Felix

Felix会监听ECTD中心的存储，从它获取事件，比如说用户在这台机器上加了一个IP，或者是创建了一个容器等。用户创建pod后，Felix负责将其网卡、IP、MAC都设置好，然后在内核的路由表里面写一条，注明这个IP应该到这张网卡。同样如果用户制定了隔离策略，Felix同样会将该策略创建到ACL中，以实现隔离。

 

## 2.4 BIRD

BIRD是一个标准的路由程序，它会从内核里面获取哪一些IP的路由发生了变化，然后通过标准BGP的路由协议扩散到整个其他的宿主机上，让外界都知道这个IP在这里，你们路由的时候得到这里来。

 

## 2.5 架构特点

由于Calico是一种纯三层的实现，因此可以避免与二层方案相关的数据包封装的操作，中间没有任何的NAT，没有任何的overlay，所以它的转发效率可能是所有方案中最高的，因为它的包直接走原生TCP/IP的协议栈，它的隔离也因为这个栈而变得好做。因为TCP/IP的协议栈提供了一整套的防火墙的规则，所以它可以通过IPTABLES的规则达到比较复杂的隔离逻辑。

 

# 三、Calico 网络Node之间两种网络

## IPIP

从字面来理解，就是把一个IP数据包又套在一个IP包里，即把 IP 层封装到 IP 层的一个 tunnel。它的作用其实基本上就相当于一个基于IP层的网桥！一般来说，普通的网桥是基于mac层的，根本不需 IP，而这个 ipip 则是通过两端的路由做一个 tunnel，把两个本来不通的网络通过点对点连接起来。 

 

## BGP

边界网关协议（Border Gateway Protocol, BGP）是互联网上一个核心的去中心化自治路由协议。它通过维护IP路由表或‘前缀’表来实现自治系统（AS）之间的可达性，属于矢量路由协议。BGP不使用传统的内部网关协议（IGP）的指标，而使用基于路径、网络策略或规则集来决定路由。因此，它更适合被称为矢量性协议，而不是路由协议。BGP，通俗的讲就是讲接入到机房的多条线路（如电信、联通、移动等）融合为一体，实现多线单IP，BGP 机房的优点：服务器只需要设置一个IP地址，最佳访问路由是由网络上的骨干路由器根据路由跳数与其它技术指标来确定的，不会占用服务器的任何系统。

 实际上，Calico项目提供的网络解决方案，与Flannel的host-gw模式几乎一样。也就是说，Calico也是基于路由表实现容器数据包转发，但不同于Flannel使用flanneld进程来维护路由信息的做法，而Calico项目使用BGP协议来自动维护整个集群的路由信息。

BGP英文全称是Border Gateway Protocol，即边界网关协议，它是一种自治系统间的动态路由发现协议，与其他 BGP 系统交换网络可达信息。

为了能让你更清楚理解BGP，举个例子：

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604140123415-392007465.png)

 

 

在这个图中，有两个自治系统（autonomous system，简称为AS）：AS 1 和 AS 2。

在互联网中，一个自治系统(AS)是一个有权自主地决定在本系统中应采用何种路由协议的小型单位。这个网络单位可以是一个简单的网络也可以是一个由一个或多个普通的网络管理员来控制的网络群体，它是一个单独的可管理的网络单元（例如一所大学，一个企业或者一个公司个体）。一个自治系统有时也被称为是一个路由选择域（routing domain）。一个自治系统将会分配一个全局的唯一的16位号码，有时我们把这个号码叫做自治系统号（ASN）。

在正常情况下，自治系统之间不会有任何来往。如果两个自治系统里的主机，要通过 IP 地址直接进行通信，我们就必须使用路由器把这两个自治系统连接起来。BGP协议就是让他们互联的一种方式。

### Calico BGP实现

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200604140250724-828678165.png)

在了解了 BGP 之后，Calico 项目的架构就非常容易理解了，Calico主要由三个部分组成：

- Felix：以DaemonSet方式部署，运行在每一个Node节点上，主要负责维护宿主机上路由规则以及ACL规则。
- BGP Client（BIRD）：主要负责把 Felix 写入 Kernel 的路由信息分发到集群 Calico 网络。
- Etcd：分布式键值存储，保存Calico的策略和网络配置状态。
- calicoctl：允许您从简单的命令行界面实现高级策略和网络。

### Calico BGP 原理

Pod 1 访问 Pod 2大致流程如下：

1. 数据包从容器1出到达Veth Pair另一端（宿主机上，以cali前缀开头）；
2. 宿主机根据路由规则，将数据包转发给下一跳（网关）；
3. 到达Node2，根据路由规则将数据包转发给cali设备，从而到达容器2。

路由表：



```bash
# node1
default via 192.168.0.1 dev ens32 proto static metric 100 
10.244.0.0/24 via 192.168.0.134 dev ens32 
10.244.58.192/26 via 192.168.0.134 dev ens32 proto bird 
10.244.85.192 dev calic32fa4483b3 scope link 
blackhole 10.244.85.192/26 proto bird 
10.244.235.192/26 via 192.168.0.132 dev ens32 proto bird

# node2
default via 192.168.0.1 dev ens32 proto static metric 100 
10.244.58.192 dev calib29fd680177 scope link 
blackhole 10.244.58.192/26 proto bird 
10.244.85.192/26 via 192.168.0.133 dev ens32 proto bird 
10.244.235.192/26 via 192.168.0.132 dev ens32 proto bird
```



其中，这里最核心的“下一跳”路由规则，就是由 Calico 的 Felix 进程负责维护的。这些路由规则信息，则是通过 BGP Client 也就是 BIRD 组件，使用 BGP 协议传输而来的。

不难发现，Calico 项目实际上将集群里的所有节点，都当作是边界路由器来处理，它们一起组成了一个全连通的网络，互相之间通过 BGP 协议交换路由规则。这些节点，我们称为 BGP Peer。

#### Route Reflector 模式（RR）

https://docs.projectcalico.org/master/networking/bgp

Calico 维护的网络在默认是（Node-to-Node Mesh）全互联模式，Calico集群中的节点之间都会相互建立连接，用于路由交换。但是随着集群规模的扩大，mesh模式将形成一个巨大服务网格，连接数成倍增加。

```bash
# netstat -antp |grep bird
tcp        0      0 0.0.0.0:179             0.0.0.0:*               LISTEN      23124/bird          
tcp        0      0 192.168.0.134:44366     192.168.0.132:179       ESTABLISHED 23124/bird          
tcp        0      0 192.168.0.134:1215      192.168.0.133:179       ESTABLISHED 23124/bird
```

这时就需要使用 Route Reflector（路由器反射）模式解决这个问题。

确定一个或多个Calico节点充当路由反射器，让其他节点从这个RR节点获取路由信息。

具体步骤如下：

**1、关闭 node-to-node BGP网格**

添加 default BGP配置，调整 nodeToNodeMeshEnabled和asNumber：



```bash
cat << EOF | calicoctl create -f -
 apiVersion: projectcalico.org/v3
 kind: BGPConfiguration
 metadata:
   name: default
 spec:
   logSeverityScreen: Info
   nodeToNodeMeshEnabled: false  
   asNumber: 63400
EOF
```



ASN号可以通过获取 # calicoctl get nodes --output=wide

**2、配置指定节点充当路由反射器**

为方便让BGPPeer轻松选择节点，通过标签选择器匹配。

给路由器反射器节点打标签：

```bash
kubectl label node k8s-node01 route-reflector=true
```

然后配置路由器反射器节点routeReflectorClusterID：



```yaml
# calicoctl get node k8s-node01 -o yaml > rr-node.yaml
# vim rr-node.yaml 
apiVersion: projectcalico.org/v3
kind: Node
apiVersion: projectcalico.org/v3
kind: Node
metadata:
  annotations:
    projectcalico.org/kube-labels: '{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/arch":"amd64","kubernetes.io/hostname":"k8s-node01","kubernetes.io/os":"linux"}'
  creationTimestamp: 2020-06-04T07:24:40Z
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: k8s-node01
    kubernetes.io/os: linux
  name: k8s-node01
  resourceVersion: "50638"
  uid: 16452357-d399-4247-bc2b-ab7e7eb52dbc
spec:
  bgp:
    ipv4Address: 192.168.0.133/24
    routeReflectorClusterID: 244.0.0.1
  orchRefs:
  - nodeName: k8s-node01
    orchestrator: k8s
```

```bash
# calicoctl apply -f bgppeer.yaml

Successfully applied 1 'BGPPeer' resource(s)
```

现在，很容易使用标签选择器将路由反射器节点与其他非路由反射器节点配置为对等：



```yaml
# vi bgppeer.yaml
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-with-route-reflectors
spec:
  nodeSelector: all()
  peerSelector: route-reflector == 'true'
```

```bash
# calicoctl apply -f bgppeer.yaml
Successfully applied 1 'BGPPeer' resource(s)
```



查看节点的BGP连接状态：

```bash
# calicoctl node status
Calico process is running.
IPv4 BGP status
+---------------+---------------+-------+----------+-------------+
| PEER ADDRESS  |   PEER TYPE   | STATE |  SINCE   |    INFO     |
+---------------+---------------+-------+----------+-------------+
| 192.168.0.133 | node specific | up    | 09:54:05 | Established |
+---------------+---------------+-------+----------+-------------+
```

 

# 四、IPIP 工作模式

 

## 4.1 测试环境

一个msater节点，ip 172.171.5.95，一个node节点 ip 172.171.5.96 

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150641464-621788389.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150641464-621788389.png)

 

创建一个daemonset的应用，pod1落在master节点上 ip地址为192.168.236.3，pod2落在node节点上 ip地址为192.168.190.203

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150652618-320617155.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150652618-320617155.png)

 

pod1 ping pod2

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413153642465-2094530861.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413153642465-2094530861.png)

 

42. ## ping包的旅程

pod1上的路由信息

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150713064-1083686463.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150713064-1083686463.png)

根据路由信息，ping 192.168.190.203，会匹配到第一条。第一条路由的意思是：去往任何网段的数据包都发往网管169.254.1.1，然后从eth0网卡发送出去。

路由表中Flags标志的含义：

U up表示当前为启动状态

H host表示该路由为一个主机，多为达到数据包的路由

G Gateway 表示该路由是一个网关，如果没有说明目的地是直连的

D Dynamicaly 表示该路由是重定向报文修改

M 表示该路由已被重定向报文修改

 

master节点上的路由信息

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150720233-1995872444.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150720233-1995872444.png)

 

当ping包来到master节点上，会匹配到路由tunl0。该路由的意思是：去往192.169.190.192/26的网段的数据包都发往网关172.171.5.96。因为pod1在5.95，pod2在5.96。所以数据包就通过设备tunl0发往到node节点上。 

 

 node节点上路由信息

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154329954-1165326319.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154329954-1165326319.png)

当node节点网卡收到数据包之后，发现发往的目的ip为192.168.190.203，于是匹配到红线的路由。该路由的意思是：192.168.190.203是本机直连设备，去往设备的数据包发往caliadce112d250。

 

 那么该设备是什么呢？如果到这里你能猜出来是什么，那说明你的网络功底是不错的。这个设备就是veth pair的一端。在创建pod2时calico会给pod2创建一个veth pair设备。一端是pod2的网卡，另一端就是我们看到的caliadce112d250。下面我们验证一下。在pod2中安装ethtool工具，然后使用ethtool -S eth0,查看veth pair另一端的设备号。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154707177-2093232618.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154707177-2093232618.png)

pod2 网卡另一端的设备好号是18，在node上查看编号为18的网络设备，可以发现该网络设备就是caliadce112d250。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413155040787-1729369707.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413155040787-1729369707.png)

所以，node上的路由，发送caliadce112d250的数据其实就是发送到pod2的网卡中。ping包的旅行到这里就到了目的地。

 

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150729776-1365068043.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150729776-1365068043.png)

 

查看一下pod2中的路由信息，发现该路由信息和pod1中是一样的。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150746241-395523835.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150746241-395523835.png) 

 

顾名思义，IPIP网络就是将IP网络封装在IP网络里。IPIP网络的特点是所有pod的数据流量都从隧道tunl0发送，并且在tunl0这增加了一层传输层的封包。

在master网卡上抓包分析该过程。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414232527057-801852995.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414232527057-801852995.png)

 

 

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150813156-308581738.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150813156-308581738.png)

 

打开ICMP 285，pod1 ping pod2的数据包，能够看到该数据包一共5层，其中IP所在的网络层有两个，分别是pod之间的网络和主机之间的网络封装。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150820715-66735737.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150820715-66735737.png)

 

根据数据包的封装顺序，应该是在pod1 ping pod2的ICMP包外面多封装了一层主机之间的数据包。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414173605263-65569449.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414173605263-65569449.png)

之所以要这样做是因为tunl0是一个隧道端点设备，在数据到达时要加上一层封装，便于发送到对端隧道设备中。 

 

 两层IP封装的具体内容

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150826196-1894311200.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150826196-1894311200.png)

 

IPIP的连接方式：

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165144848-1984358878.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165144848-1984358878.png)

 

# 五、BGP 工作模式

## 5.1 修改配置

在安装calico网络时，默认安装是IPIP网络。calico.yaml文件中，将CALICO_IPV4POOL_IPIP的值修改成 "off"，就能够替换成BGP网络。

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414174416871-1167710181.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414174416871-1167710181.png)

## 5.2 对比

BGP网络相比较IPIP网络，最大的不同之处就是没有了隧道设备 tunl0。 前面介绍过IPIP网络pod之间的流量发送tunl0，然后tunl0发送对端设备。BGP网络中，pod之间的流量直接从网卡发送目的地，减少了tunl0这个环节。

 

master节点上路由信息。从路由信息来看，没有tunl0设备。 

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113440788-275117848.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113440788-275117848.png)

 

同样创建一个daemonset，pod1在master节点上，pod2在node节点上。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114541797-340745707.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114541797-340745707.png)

 

## 5.3 ping包之旅

pod1 ping pod2。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114230073-496262441.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114230073-496262441.png)

根据pod1中的路由信息，ping包通过eth0网卡发送到master节点上。

master节点上路由信息。根据匹配到的 192.168.190.192 路由，该路由的意思是：去往网段192.168.190.192/26 的数据包，发送网段172.171.5.96。而5.96就是node节点。所以，该数据包直接发送了5.96节点。 

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114439204-1886327378.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114439204-1886327378.png)

 

node节点上的路由信息。根据匹配到的192.168.190.192的路由，数据将发送给 cali6fcd7d1702e设备，该设备和上面分析的是一样，为pod2的veth pair 的一端。数据就直接发送给pod2的网卡。

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115008617-409053518.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115008617-409053518.png)

 

当pod2对ping包做出回应之后，数据到达node节点上，匹配到192.168.236.0的路由，该路由说的是：去往网段192.168.236.0/26 的数据，发送给网关 172.171.5.95。数据包就直接通过网卡ens160，发送到master节点上。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115329941-1036275159.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115329941-1036275159.png)

 

通过在master节点上抓包，查看经过的流量，筛选出ICMP，找到pod1 ping pod2的数据包。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115718930-42251257.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115718930-42251257.png)

 

可以看到BGP网络下，没有使用IPIP模式，数据包是正常的封装。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113605760-703346562.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113605760-703346562.png)

值得注意的是mac地址的封装。192.168.236.0是pod1的ip，192.168.190.198是pod2的ip。而源mac地址是 master节点网卡的mac，目的mac是node节点的网卡的mac。这说明，在 master节点的路由接收到数据，重新构建数据包时，使用arp请求，将node节点的mac拿到，然后封装到数据链路层。

[![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113611124-513029729.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113611124-513029729.png)

 

BGP的连接方式：

 [![img](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165320714-135136611.png)](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165320714-135136611.png)

 

# 六、两种网络对比

## IPIP网络：

流量：tunlo设备封装数据，形成隧道，承载流量。

适用网络类型：适用于互相访问的pod不在同一个网段中，跨网段访问的场景。外层封装的ip能够解决跨网段的路由问题。

效率：流量需要tunl0设备封装，效率略低

## BGP网络：

流量：使用路由信息导向流量

适用网络类型：适用于互相访问的pod在同一个网段，适用于大型网络。

效率：原生hostGW，效率高

 

# 七、存在问题

(1) 缺点租户隔离问题

Calico 的三层方案是直接在 host 上进行路由寻址，那么对于多租户如果使用同一个 CIDR 网络就面临着地址冲突的问题。

 

(2) 路由规模问题

通过路由规则可以看出，路由规模和 pod 分布有关，如果 pod离散分布在 host 集群中，势必会产生较多的路由项。

 

(3) iptables 规则规模问题

1台 Host 上可能虚拟化十几或几十个容器实例，过多的 iptables 规则造成复杂性和不可调试性，同时也存在性能损耗。

 

(4) 跨子网时的网关路由问题

当对端网络不为二层可达时，需要通过三层路由机时，需要网关支持自定义路由配置，即 pod 的目的地址为本网段的网关地址，再由网关进行跨三层转发。

# 八、Calico 部署

```bash
curl https://docs.projectcalico.org/v3.9/manifests/calico-etcd.yaml -o calico.yaml
```

下载完后还需要修改里面配置项：

具体步骤如下：

- 配置连接etcd地址，如果使用https，还需要配置证书。（ConfigMap，Secret）
- 根据实际网络规划修改Pod CIDR（CALICO_IPV4POOL_CIDR）
- 选择工作模式（CALICO_IPV4POOL_IPIP），支持**BGP（Never）**、**IPIP（Always）**、**CrossSubnet**（开启BGP并支持跨子网）

删除flannel网络

```bash
# kubectl delete -f kube-flannel.yaml
# ip link delete flannel.1
# ip link delete cni0
# ip route del 10.244.2.0/24 via 192.168.0.132 dev ens32
# ip route del 10.244.1.0/24 via 192.168.0.133 dev ens32
```

应用清单：

```bash
# kubectl apply -f calico.yaml
# kubectl get pods -n kube-system
```

# 九、Calico 管理工具

下载工具：https://github.com/projectcalico/calicoctl/releases



```bash
# wget -O /usr/local/bin/calicoctl https://github.com/projectcalico/calicoctl/releases/download/v3.9.1/calicoctl
# chmod +x /usr/local/bin/calicoctl

# mkdir /etc/calico
# vim /etc/calico/calicoctl.cfg  
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "https://192.168.0.132:2379,https://192.168.0.133:2379,https://192.168.0.134:2379"
  etcdKeyFile: "/opt/etcd/ssl/server-key.pem"
  etcdCertFile: "/opt/etcd/ssl/server.pem"
  etcdCACertFile: "/opt/etcd/ssl/ca.pem"
```



使用calicoctl查看服务状态：



```bash
# calicoctl get node
NAME         
k8s-master   
k8s-node01   
k8s-node02   
# calicoctl node status
IPv4 BGP status
+---------------+-------------------+-------+----------+-------------+
| PEER ADDRESS  |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+---------------+-------------------+-------+----------+-------------+
| 192.168.0.133 | node-to-node mesh | up    | 07:24:45 | Established |
| 192.168.0.134 | node-to-node mesh | up    | 07:25:00 | Established |
+---------------+-------------------+-------+----------+-------------+
查看 IPAM的IP地址池：
# calicoctl get ippool -o wide
NAME                  CIDR            NAT    IPIPMODE   VXLANMODE   DISABLED   SELECTOR   
default-ipv4-ippool   10.244.0.0/16   true   Never      Never       false      all()
```

# 十、网络策略

## 为什么需要网络隔离？

CNI插件插件解决了不同Node节点Pod互通问题，从而形成一个扁平化网络，默认情况下，Kubernetes 网络允许所有 Pod 到 Pod 的流量，在一些场景中，我们不希望Pod之间默认相互访问，例如：

- 应用程序间的访问控制。例如微服务A允许访问微服务B，微服务C不能访问微服务A
- 开发环境命名空间不能访问测试环境命名空间Pod
- 当Pod暴露到外部时，需要做Pod白名单
- 多租户网络环境隔离

所以，我们需要使用network policy对Pod网络进行隔离。支持对Pod级别和Namespace级别网络访问控制。

Pod网络入口方向隔离

- 基于Pod级网络隔离：只允许特定对象访问Pod（使用标签定义），允许白名单上的IP地址或者IP段访问Pod
- 基于Namespace级网络隔离：多个命名空间，A和B命名空间Pod完全隔离。

Pod网络出口方向隔离

- 拒绝某个Namespace上所有Pod访问外部
- 基于目的IP的网络隔离：只允许Pod访问白名单上的IP地址或者IP段
- 基于目标端口的网络隔离：只允许Pod访问白名单上的端口

## 网络策略概述

一个NetworkPolicy例子：



```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```



配置解析：

- podSelector：用于选择策略应用到的Pod组。
- policyTypes：其可以包括任一Ingress，Egress或两者。该policyTypes字段指示给定的策略用于Pod的入站流量、还是出站流量，或者两者都应用。如果未指定任何值，则默认值为Ingress，如果网络策略有出口规则，则设置egress。
- Ingress：from是可以访问的白名单，可以来自于IP段、命名空间、Pod标签等，ports是可以访问的端口。
- Egress：这个Pod组可以访问外部的IP段和端口。

## 入站、出站网络流量访问控制案例

### **Pod访问限制**

准备测试环境，一个web pod，两个client pod



```bash
# kubectl create deployment web --image=nginx
# kubectl run client1 --generator=run-pod/v1 --image=busybox --command -- sleep 36000
# kubectl run client2 --generator=run-pod/v1 --image=busybox --command -- sleep 36000
# kubectl get pods --show-labels
NAME                   READY   STATUS    RESTARTS   AGE    LABELS
client1                1/1     Running   0          49s    run=client1
client2                1/1     Running   0          49s    run=client2
web-5886dfbb96-8mkg9   1/1     Running   0          136m   app=web,pod-template-hash=5886dfbb96
web-5886dfbb96-hps6t   1/1     Running   0          136m   app=web,pod-template-hash=5886dfbb96
web-5886dfbb96-lckfs   1/1     Running   0          136m   app=web,pod-template-hash=5886dfbb96
```



需求：将default命名空间携带run=web标签的Pod隔离，只允许default命名空间携带run=client1标签的Pod访问80端口



```yaml
# vim pod-acl.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: default
    - podSelector:
        matchLabels:
          run: client1
    ports:
    - protocol: TCP
      port: 80

# kubectl apply -f pod-acl.yaml
```



测试



```bash
# kubectl exec -it client1 sh
/ # wget 10.244.85.192
Connecting to 10.244.85.192 (10.244.85.192:80)
saving to 'index.html'
index.html           100% |**************************************************************************************************************|   612  0:00:00 ETA
'index.html' saved

# kubectl exec -it client2 sh
/ # wget 10.244.85.192
Connecting to 10.244.85.192 (10.244.85.192:80)
wget: can't connect to remote host (10.244.85.192): Connection timed out
```

隔离策略配置：

Pod对象：default命名空间携带run=web标签的Pod

允许访问端口：80

允许访问对象：default命名空间携带run=client1标签的Pod

拒绝访问对象：除允许访问对象外的所有对象

### **命名空间隔离**

需求：default命名空间下所有pod可以互相访问，但不能访问其他命名空间Pod，其他命名空间也不能访问default命名空间Pod。

```yaml
# kubectl create ns test
namespace/test created
# kubectl create deployment web --image=nginx -n test
deployment.apps/web created
# kubectl run client -n test --generator=run-pod/v1 --image=busybox --command -- sleep 36000

# vim ns-acl.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-from-other-namespaces 
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}

# kubectl apply -f ns-acl.yaml
# kubectl get pod -n test -o wide
NAME                  READY   STATUS    RESTARTS   AGE    IP               NODE          NOMINATED NODE   READINESS GATES
client                1/1     Running   0          119s   10.244.85.195    k8s-node01    <none>           <none>
web-d86c95cc9-gr88v   1/1     Running   0          10m    10.244.235.198   k8s-master1   <none>           <none>
[root@k8s-master ~]# kubectl get pod  -o wide
NAME                   READY   STATUS    RESTARTS   AGE    IP               NODE          NOMINATED NODE   READINESS GATES
web-5886dfbb96-8mkg9   1/1     Running   0          158m   10.244.235.193   k8s-master1   <none>           <none>
web-5886dfbb96-hps6t   1/1     Running   0          158m   10.244.85.192    k8s-node01    <none>           <none>
web-5886dfbb96-lckfs   1/1     Running   0          158m   10.244.58.192    k8s-node02    <none>           <none>
# kubectl exec -it client -n test sh
/ # wget 10.244.58.192
Connecting to 10.244.58.192 (10.244.58.192:80)
wget: can't connect to remote host (10.244.58.192): Connection timed out
/ # wget 10.244.235.198
Connecting to 10.244.235.198 (10.244.235.198:80)
saving to 'index.html'
index.html           100% |**************************************************************************************************************|   612  0:00:00 ETA
'index.html' saved
```

podSelector: {}：default命名空间下所有Pod

from.podSelector: {} : 如果未配置具体的规则，默认不允许