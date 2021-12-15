# 1. Service

- 定义一组pod的访问规则

Kubernetes Service定义了这样一种抽象：逻辑上的一组 Pod，一种可以访问它们的策略 —— 通常被称为微服务。这一组 Pod 能够被 Service 访问到，通常是通过 selector实现的。

举例：考虑一个图片处理 backend，它运行了3个副本。这些副本是可互换的 —— frontend 不需要关心它们调用了哪个 backend 副本。 然而组成这一组 backend 程序的 Pod 实际上可能会发生变化，frontend 客户端不应该也没必要知道，而且也不需要跟踪这一组 backend 的状态。Service 定义的抽象能够解耦这种关联。



Service可以提供负载均衡的能力，但是使用上存在如下限制：

- 只能提供4层负载均衡能力，而没有7层功能。有时我们可能需要更多的匹配规则来转发请求，这点上4层负载均衡是不支持的



Service是Kubernetes的核心概念，通过创建Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址，并且将请求负载分发到后端的各个容器应用上。

**Service从逻辑上代表了一组Pod，具体是哪组Pod则是由label来挑选的**

**在Kubernetes中Service的Cluster IP实现数据报文请求的转发，都离不开node上部署的重要组件 kube-proxy**



kube-proxy作用

- 实时监听kube-api，获取建立service的建立，升级信息，增加或删除pod信息。来获取Pod和VIP的映射关系
- 维护本地Netfileter iptables IPVS内核组件
- 通过修改和更新ipvs规则来实现数据报文的转发规则
- 构建路由信息，通过转发规则转发报文到对应的pod上



## 1.1 service存在意义

- 防止pod失联（服务发现）

![](..\..\img\pod.png)



- 定义一组Pod访问策略（负载均衡）

![](..\..\img\service.png)

## 1.2 Pod和Service关系

- 根据label和selector标签建立关联的

- 通过serivice实现Pod的负载均衡

![](..\..\img\podservice.png)

## 1.3 VIP和Service代理

在 Kubernetes 集群中，每个 Node 运行一个 kube-proxy 进程。kube-proxy 负责为 Service 实现了一种 VIP（虚拟 IP）的形式，而不是 ExternalName 的形式。

从Kubernetes v1.0开始，已经可以使用 userspace代理模式。Kubernetes v1.1添加了 iptables 代理模式，在 Kubernetes v1.2 中kube-proxy 的 iptables 模式成为默认设置。Kubernetes v1.8添加了 ipvs 代理模式。

**为什么不使用 DNS 轮询？**

原因如下：

- DNS 实现的历史由来已久，它不遵守记录 TTL，并且在名称查找到结果后会对其进行缓存。
- 有些应用程序仅执行一次 DNS 查找，并无限期地缓存结果。
- 即使应用和库进行了适当的重新解析，DNS 记录上的 TTL 值低或为零也可能会给 DNS 带来高负载，从而使管理变得困难。

总之就是因为有缓存，因此不合适。

 

## userspace代理模式

这种模式，kube-proxy 会监视 Kubernetes master 对 Service 对象和 Endpoints 对象的添加和移除。 对每个 Service，它会在本地 Node 上打开一个端口（随机选择）。 任何连接到“代理端口”的请求，都会被代理到 Service 的backend Pods 中的某个上面（如 Endpoints 所报告的一样）。 使用哪个 backend Pod，是 kube-proxy 基于 SessionAffinity 来确定的。

最后，它配置 iptables 规则，捕获到达该 Service 的 clusterIP（是虚拟 IP）和 Port 的请求，并重定向到代理端口，代理端口再代理请求到 backend Pod。

默认情况下，userspace模式下的kube-proxy通过循环算法选择后端。

默认的策略是，通过 round-robin 算法来选择 backend Pod。

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200917000552912-742898966.png)

 

## iptables 代理模式

这种模式，kube-proxy 会监视 Kubernetes 控制节点对 Service 对象和 Endpoints 对象的添加和移除。 对每个 Service，它会配置 iptables 规则，从而捕获到达该 Service 的 clusterIP 和端口的请求，进而将请求重定向到 Service 的一组 backend 中的某个上面。对于每个 Endpoints 对象，它也会配置 iptables 规则，这个规则会选择一个 backend 组合。

默认的策略是，kube-proxy 在 iptables 模式下随机选择一个 backend。

使用 iptables 处理流量具有较低的系统开销，因为流量由 Linux netfilter 处理，而无需在用户空间和内核空间之间切换。 这种方法也可能更可靠。

如果 kube-proxy 在 iptables模式下运行，并且所选的第一个 Pod 没有响应，则连接失败。 这与userspace模式不同：在这种情况下，kube-proxy 将检测到与第一个 Pod 的连接已失败，并会自动使用其他后端 Pod 重试。

我们可以使用 Pod readiness 探测器 验证后端 Pod 是否可以正常工作，以便 iptables 模式下的 kube-proxy 仅看到测试正常的后端。这样做意味着可以避免将流量通过 kube-proxy 发送到已知已失败的Pod。

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200917000609683-1386303374.png)

 

## IPVS 代理模式

在 ipvs 模式下，kube-proxy监视Kubernetes服务(Service)和端点(Endpoints)，调用 netlink 接口相应地创建 IPVS 规则， 并定期将 IPVS 规则与 Kubernetes服务(Service)和端点(Endpoints)同步。该控制循环可确保 IPVS 状态与所需状态匹配。访问服务(Service)时，IPVS　将流量定向到后端Pod之一。

IPVS代理模式基于类似于 iptables 模式的 netfilter 挂钩函数，但是使用哈希表作为基础数据结构，并且在内核空间中工作。 这意味着，与 iptables 模式下的 kube-proxy 相比，IPVS 模式下的 kube-proxy 重定向通信的延迟要短，并且在同步代理规则时具有更好的性能。与其他代理模式相比，IPVS 模式还支持更高的网络流量吞吐量。

IPVS提供了更多选项来平衡后端Pod的流量。这些是：

- rr: round-robin
- lc: least connection (smallest number of open connections)
- dh: destination hashing
- sh: source hashing
- sed: shortest expected delay
- nq: never queue

注意：要在 IPVS 模式下运行 kube-proxy，必须在启动 kube-proxy 之前使 IPVS Linux 在节点上可用。 当 kube-proxy 以 IPVS 代理模式启动时，它将验证 IPVS 内核模块是否可用。 如果未检测到 IPVS 内核模块，则 kube-proxy 将退回到以 iptables 代理模式运行。

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200917000627912-624638194.png)



## 1.4 常用Service类型

Kubernetes 中Service有以下4中类型：

### ClusterIP（默认）

- 集群内部使用，默认类型，自动分配一个仅Cluster内部可以访问的虚拟IP

### NodePort

- 对外访问应用使用，对外暴露，访问端口
- 通过每个 Node 上的 IP 和静态端口（NodePort）暴露服务。以ClusterIP为基础，NodePort 服务会路由到 ClusterIP 服务。通过请求 `<NodeIP>:<NodePort>`，可以从集群的外部访问一个集群内部的 NodePort 服务。

### LoadBalancer

- 对外访问应用使用，公有云
- 使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。

### ExternalName

- 通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容（例如，foo.bar.example.com）。
- 没有任何类型代理被创建。



需要注意的是：Service 能够将一个接收 port 映射到任意的 targetPort。默认情况下，targetPort 将被设置为与 port 字段相同的值。

Service域名格式：`$(service name).$(namespace).svc.cluster.local`，其中 cluster.local 为指定的集群的域名。



node内网部署应用，外网一般不能访问。

- 找到一台可以进行外网访问机器，安装nginx，反向代理
- 手动把可以访问节点添加到nginx里面



LoadBalancer：公有云，负载均衡，控制器

# 2. Service资源定义

```yaml
apiVersion: v1
kind: Service
metadata:
 name: nginx-svc
 labels:
   app: nginx
spec:
 type: ClusterIP
 ports:
   - port: 80
      targetPort: 80
 selector:
   app: nginx
```

## 2.1 Service Type

根据创建Service的type不同 可以分为以下几种类型

- **ClusterIP**
  默认方式，根据是否生成ClusterIP又可以分为普通Service和Headless Service两类

  此方式仅用于集群内部之间实现通信的

- **NodePort**

  NodePort模式除了使用cluster ip外，也将service的port映射到每个node的一个指定内部port上，映射的每个node的内部port都一样。可以通过访问Node节点的IP实现外部通信

- **LoadBalancer**
  要配合支持公有云负载均衡使用比如GCE、AWS。其实也是NodePort，只不过会把:自动添加到公有云的负载均衡当中

- **ExternalName**

  外部IP;如果集群外部需要有一个服务需要我们进行访问；那么就需要在service中指定外部的IP让service与外部的那个服务进行访问;那么接下的集群内部到外部那个数据包走向便是:数据包先到service然后由service交给外部那个服务；回来的数据包是:交给node node交给service service交给Pod



下面说下生产常用的类型定义以及使用

## 2.2 Deployment的yaml信息

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat myapp-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: v1
  template:
    metadata:
      labels:
        app: myapp
        release: v1
        env: test
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
```

启动Deployment并查看状态

```bash
[root@k8s-master service]# kubectl apply -f myapp-deploy.yaml
deployment.apps/myapp-deploy created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get deploy -o wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy   3/3     3            3           31h   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,release=v1
[root@k8s-master service]#
[root@k8s-master service]# kubectl get rs -o wide
NAME                      DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy-5695bb5658   3         3         3       31h   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5695bb5658,release=v1
[root@k8s-master service]#
[root@k8s-master service]# kubectl get pod -o wide --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy-5695bb5658-2866m   1/1     Running   2          31h   10.244.2.116   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy-5695bb5658-dcfw7   1/1     Running   2          31h   10.244.4.105   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy-5695bb5658-n2b5w   1/1     Running   2          31h   10.244.2.115   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
```

curl访问



## 2.3 ClusterIP

定义一个web应用

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myappnginx
      release: stable
  template:
    metadata:
      labels:
        app: myappnginx
        release: stable
    spec:
      containers:
      - name: nginxweb
        image: nginx:1.14-alpine
        imagePullPolicy: IfNotPresent
```

### **创建service资源基于ClusterIP类型**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webservice          #service名字;也就是后面验证基于主机名访问的名字
  namespace: default        #名称空间要与刚才创建的Pod的名称空间一致，service资源也是基于namespace隔离的
spec:
  selector:                         #标签选择器很重要决定是要关联某个符合标签的Pod
    app: myappnginx         #标签要与刚才定义Pod的标签一致;因为service是通过标签与Pod关联的
    release: stable      
  type: ClusterIP               #类型是ClusterIP
  ports:                          #暴露的端口设置
  - port: 88                     #service暴露的端口
    targetPort: 80             #容器本身暴露的端口，和dockerfile中的expose意思一样
```

### 查看service状态

```bash
[root@master replicaset]# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   3d9h   <none>
webservice   ClusterIP   10.102.99.133   <none>        88/TCP    13s    app=myappnginx,release=stable

[root@master replicaset]# kubectl describe svc webservice
Name:              webservice
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"webservice","namespace":"default"},"spec":{"ports":[{"port":88,"t...
Selector:          app=myappnginx,release=stable
Type:              ClusterIP
IP:                10.102.99.133        
Port:              <unset>  88/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.27:80,10.244.2.27:80
Session Affinity:  None
Events:            <none>
```

### 连接一个客户端Pod进行测试

```bash
[root@master replicaset]# kubectl exec -it web-deploy-75bfb496f9-fm29g -- /bin/sh
/ # wget -O - webservice:88   #基于coredns进行解析的
Connecting to webservice:88 (10.102.99.133:88)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```

## 2.4 无头service

无头IP只能使用CluserIP类型就是没有clusterip 而是将解析的IP 解析到后端的Pod之上

该服务不会分配Cluster IP，也不通过kube-proxy做反向代理和负载均衡。而是通过DNS提供稳定的网络ID来访问，DNS会将headless service的后端直接解析为podIP列表。主要供StatefulSet使用



有时不需要或不想要负载均衡，以及单独的 Service IP。遇到这种情况，可以通过指定 Cluster IP（spec.clusterIP）的值为 “None” 来创建 Headless Service。

这对headless Service 并不会分配 Cluster IP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。

**使用场景**

- 第一种：自主选择权，有时候client想自己来决定使用哪个Real Server，可以通过查询DNS来获取Real Server的信息。
- 第二种：Headless Services还有一个用处（PS：也就是我们需要的那个特性）。Headless Service对应的每一个Endpoints，即每一个Pod，都会有对应的DNS域名；这样Pod之间就可以互相访问。【结合statefulset有状态服务使用，如Web、MySQL集群】



```bash
[root@master replicaset]# cat svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: webservice
  namespace: default
spec:
  selector:
    app: myappnginx
    release: stable
  clusterIP: None           定义cluserIP为空
  ports:
  - port: 88
    targetPort: 80


[root@master replicaset]# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d10h   <none>
webservice   ClusterIP   None         <none>        88/TCP    7s      app=myappnginx,release=stable
[root@master replicaset]# kubectl exec -it web-deploy-75bfb496f9-fm29g -- /bin/sh
/ # cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
/ # nslookup webservice
nslookup: can't resolve '(null)': Name does not resolve

Name:      webservice
Address 1: 10.244.2.27 10-244-2-27.webservice.default.svc.cluster.local
Address 2: 10.244.1.27 web-deploy-75bfb496f9-fm29g
```

## 2.5 NodePort

```yaml
[root@k8s-master01 daem]# cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: default
  name: nginxapp
  labels:
    app: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mynginx
  template:
    metadata:
      labels:
        app: mynginx
    spec:
      containers:
      - name: nginxweb1
        image: nginx:1.15-alpine
您在 /var/spool/mail/root 中有新邮件
[root@k8s-master01 daem]# cat svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx-svc
spec:
  ports:
  - name: http
    port: 80                #service暴露的端口，可以基于内部集群访问
    protocol: TCP
    nodePort: 30001   #node节点的映射端口 可以通过外部访问
    targetPort: 80
  selector:
    app: mynginx
  sessionAffinity: None
  type: NodePort
```

可以基于内部集群访问

```bash
[root@k8s-master01 daem]# kubectl get svc 
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        8d
nginx-svc    NodePort    10.99.184.91   <none>        80:30001/TCP   5s
您在 /var/spool/mail/root 中有新邮件
[root@k8s-master01 daem]# curl 10.99.184.91 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@k8s-master01 daem]# 
```

外部浏览器也可以进行访问。



## 2.6 sessionAffinity实现源地址session绑定

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webservice
  namespace: default
spec:
  selector:
    app: myappnginx
    release: stable
  sessionAffinity: ClientIP     将来自同意客户端的请求调度到后端的同一个Pod上
  type: NodePort
  ports:
  - port: 88
    nodePort: 30001
    targetPort: 80
```

**直接通过Pod的IP地址和端口号可以访问到容器应用内的服务，但是Pod的IP地址是不可靠的，例如当Pod所在的Node发生故障时，Pod将被Kubernetes重新调度到另一个Node，Pod的IP地址将发生变化。更重要的是，如果容器应用本身是分布式的部署方式，通过多个实例共同提供服务，就需要在这些实例的前端设置一个负载均衡器来实现请求的分发。Kubernetes中的Service就是用于解决这些问题的核心组件。**

# 3、ClusterIP类型示例

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat myapp-svc-ClusterIP.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip
  namespace: default
spec:
  type: ClusterIP  # 可以不写，为默认类型
  selector:
    app: myapp
    release: v1
  ports:
  - name: http
    port: 80
    targetPort: 80
```

启动Service并查看状态

```bash
[root@k8s-master service]# kubectl apply -f myapp-svc-ClusterIP.yaml
service/myapp-clusterip created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get svc -o wide
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   22d   <none>
myapp-clusterip   ClusterIP   10.106.66.120   <none>        80/TCP    15s   app=myapp,release=v1
```

查看pod信息

```bash
[root@k8s-master service]# kubectl get pod -o wide --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy-5695bb5658-2866m   1/1     Running   2          31h   10.244.2.116   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy-5695bb5658-dcfw7   1/1     Running   2          31h   10.244.4.105   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy-5695bb5658-n2b5w   1/1     Running   2          31h   10.244.2.115   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
```

查看ipvs信息

```bash
[root@k8s-master service]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
………………
TCP  10.106.66.120:80 rr
  -> 10.244.2.115:80              Masq    1      0          0
  -> 10.244.2.116:80              Masq    1      0          0
  -> 10.244.4.105:80              Masq    1      0          0
```

curl访问

```bash
[root@k8s-master service]# curl 10.106.66.120
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master service]#
[root@k8s-master service]# curl 10.106.66.120/hostname.html
myapp-deploy-5695bb5658-2866m
[root@k8s-master service]#
[root@k8s-master service]# curl 10.106.66.120/hostname.html
myapp-deploy-5695bb5658-n2b5w
[root@k8s-master service]#
[root@k8s-master service]# curl 10.106.66.120/hostname.html
myapp-deploy-5695bb5658-dcfw7
[root@k8s-master service]#
[root@k8s-master service]# curl 10.106.66.120/hostname.html
myapp-deploy-5695bb5658-2866m
```

# 4、Headless Services

有时不需要或不想要负载均衡，以及单独的 Service IP。遇到这种情况，可以通过指定 Cluster IP（spec.clusterIP）的值为 “None” 来创建 Headless Service。

这对headless Service 并不会分配 Cluster IP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。

**使用场景**

- 第一种：自主选择权，有时候client想自己来决定使用哪个Real Server，可以通过查询DNS来获取Real Server的信息。
- 第二种：Headless Services还有一个用处（PS：也就是我们需要的那个特性）。Headless Service对应的每一个Endpoints，即每一个Pod，都会有对应的DNS域名；这样Pod之间就可以互相访问。【结合statefulset有状态服务使用，如Web、MySQL集群】

**示例**

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat myapp-svc-headless.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-headless
  namespace: default
spec:
  selector:
    app: myapp
    release: v1
  clusterIP: "None"
  ports:
  - port: 80
    targetPort: 80
```

启动Service并查看状态和详情

```bash
[root@k8s-master service]# kubectl apply -f myapp-svc-headless.yaml
service/myapp-headless created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes       ClusterIP   10.96.0.1    <none>        443/TCP   22d   <none>
myapp-headless   ClusterIP   None         <none>        80/TCP    6s    app=myapp,release=v1
[root@k8s-master service]#
[root@k8s-master service]# kubectl describe svc/myapp-headless
Name:              myapp-headless
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"myapp-headless","namespace":"default"},"spec":{"clusterIP":"None"...
Selector:          app=myapp,release=v1
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.2.115:80,10.244.2.116:80,10.244.4.105:80  # 后端的Pod信息
Session Affinity:  None
Events:            <none>
```

service只要创建成功就会写入到coredns。我们得到coredns IP的命令如下：

```bash
[root@k8s-master service]# kubectl get pod -o wide -A | grep 'coredns'
kube-system            coredns-6955765f44-c9zfh                     1/1     Running   29         22d    10.244.0.62    k8s-master   <none>           <none>
kube-system            coredns-6955765f44-lrz5q                     1/1     Running   29         22d    10.244.0.61    k8s-master   <none>           <none>
```

在宿主机安装nslookup、dig命令安装

```bash
yum install -y bind-utils
```

coredns记录信息如下

```bash
# 其中 10.244.0.61 为 coredns IP
# myapp-headless.default.svc.cluster.local 为Headless Service域名。格式为:$(service name).$(namespace).svc.cluster.local，其中 cluster.local 指定的集群的域名
[root@k8s-master service]# nslookup myapp-headless.default.svc.cluster.local 10.244.0.61
Server:        10.244.0.61
Address:    10.244.0.61#53

Name:    myapp-headless.default.svc.cluster.local
Address: 10.244.2.116
Name:    myapp-headless.default.svc.cluster.local
Address: 10.244.4.105
Name:    myapp-headless.default.svc.cluster.local
Address: 10.244.2.115

[root@k8s-master service]#
### 或使用如下命令
[root@k8s-master service]# dig -t A myapp-headless.default.svc.cluster.local. @10.244.0.61

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-16.P2.el7_8.6 <<>> -t A myapp-headless.default.svc.cluster.local. @10.244.0.61
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7089
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;myapp-headless.default.svc.cluster.local. IN A

;; ANSWER SECTION:
myapp-headless.default.svc.cluster.local. 14 IN    A 10.244.2.116
myapp-headless.default.svc.cluster.local. 14 IN    A 10.244.4.105
myapp-headless.default.svc.cluster.local. 14 IN    A 10.244.2.115

;; Query time: 0 msec
;; SERVER: 10.244.0.61#53(10.244.0.61)
;; WHEN: Wed Jun 03 22:34:46 CST 2020
;; MSG SIZE  rcvd: 237
```

# 5、NodePort类型示例

如果将 type 字段设置为 NodePort，则 Kubernetes 控制层面将在 --service-node-port-range 标志指定的范围内分配端口（默认值：30000-32767）。

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat myapp-svc-NodePort.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
  namespace: default
spec:
  type: NodePort
  selector:
    app: myapp
    release: v1
  ports:
  - name: http
    # 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值。
    port: 80         # Service对外提供服务端口
    targetPort: 80   # 请求转发后端Pod使用的端口
    nodePort: 31682  # 可选字段，默认情况下，为了方便起见，Kubernetes 控制层面会从某个范围内分配一个端口号（默认：30000-32767）
```

启动Service并查看状态

```bash
[root@k8s-master service]# kubectl apply -f myapp-svc-NodePort.yaml
service/myapp-nodeport created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get svc -o wide
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes       ClusterIP   10.96.0.1     <none>        443/TCP        22d   <none>
myapp-nodeport   NodePort    10.99.50.81   <none>        80:31682/TCP   6s    app=myapp,release=v1
```

由上可见，类型变为了NodePort

查看ipvs信息

```bash
[root@k8s-master service]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
………………
TCP  10.99.50.81:80 rr
  -> 10.244.2.115:80              Masq    1      0          0
  -> 10.244.2.116:80              Masq    1      0          0
  -> 10.244.4.105:80              Masq    1      0          0
```

端口查看，可见在本地宿主机监听了相应的端口（备注：集群所有机器都监听了该端口）

```bash
# 集群所有机器都可以执行查看
[root@k8s-master service]# netstat -lntp | grep '31682'
tcp6       0      0 :::31682                :::*                    LISTEN      3961/kube-proxy
```

curl通过ClusterIP访问

```bash
# 通过ClusterIP访问
[root@k8s-master service]# curl 10.99.50.81
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master service]#
[root@k8s-master service]# curl 10.99.50.81/hostname.html
myapp-deploy-5695bb5658-2866m
[root@k8s-master service]#
[root@k8s-master service]# curl 10.99.50.81/hostname.html
myapp-deploy-5695bb5658-n2b5w
[root@k8s-master service]#
[root@k8s-master service]# curl 10.99.50.81/hostname.html
myapp-deploy-5695bb5658-dcfw7
```

curl通过节点IP访问

```bash
# 通过集群节点IP访问
[root@k8s-master service]# curl 172.16.1.110:31682
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master service]#
[root@k8s-master service]# curl 172.16.1.110:31682/hostname.html
myapp-deploy-5695bb5658-2866m
[root@k8s-master service]#
[root@k8s-master service]# curl 172.16.1.110:31682/hostname.html
myapp-deploy-5695bb5658-n2b5w
[root@k8s-master service]#
[root@k8s-master service]# curl 172.16.1.110:31682/hostname.html
myapp-deploy-5695bb5658-dcfw7
# 访问集群其他节点。每台机器都有LVS，和相关调度
[root@k8s-master service]# curl 172.16.1.111:31682/hostname.html
myapp-deploy-5695bb5658-dcfw7
[root@k8s-master service]#
[root@k8s-master service]# curl 172.16.1.112:31682/hostname.html
myapp-deploy-5695bb5658-dcfw7
```

访问日志查看

```bash
kubectl logs -f svc/myapp-nodeport
```

# 6、ExternalName类型示例

这种类型的Service通过返回CNAME和它的值，可以将服务映射到externalName字段的内容（例如：my.k8s.example.com；可以实现跨namespace名称空间访问）。ExternalName Service是Service的特例，它没有selector，也没有定义任何的端口和Endpoint。相反的，对于运行在集群外部的服务，它通过返回该外部服务的别名这种方式提供服务。

具体使用参见：「[Kubernetes K8S之Pod跨namespace名称空间访问Service服务](https://www.cnblogs.com/zhanglianghhh/p/13663476.html)」

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat myapp-svc-ExternalName.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-externalname
  namespace: default
spec:
  type: ExternalName
  externalName: my.k8s.example.com
```

启动Service并查看状态

```bash
[root@k8s-master service]# kubectl apply -f myapp-svc-ExternalName.yaml
service/myapp-externalname created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get svc -o wide
NAME                 TYPE           CLUSTER-IP   EXTERNAL-IP          PORT(S)   AGE   SELECTOR
kubernetes           ClusterIP      10.96.0.1    <none>               443/TCP   21d   <none>
myapp-externalname   ExternalName   <none>       my.k8s.example.com   <none>    21s   <none>
```

由上可见，类型变为了ExternalName

宿主机dig命令安装

```bash
yum install -y bind-utils
```

coredns记录信息如下

```bash
# 其中 10.244.0.61 为 coredns IP
# myapp-externalname.default.svc.cluster.local 为Service域名。格式为:$(service name).$(namespace).svc.cluster.local，其中 cluster.local 指定的集群的域名
##### 通过 nslookup 访问
[root@k8s-master service]# nslookup myapp-externalname.default.svc.cluster.local 10.244.0.61
Server:        10.244.0.61
Address:    10.244.0.61#53

myapp-externalname.default.svc.cluster.local    canonical name = my.k8s.example.com.
** server can't find my.k8s.example.com: NXDOMAIN

[root@k8s-master service]#
##### 通过 dig 访问
[root@k8s-master service]# dig -t A myapp-externalname.default.svc.cluster.local. @10.244.0.61

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-16.P2.el7_8.6 <<>> -t A myapp-externalname.default.svc.cluster.local. @10.244.0.61
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39541
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;myapp-externalname.default.svc.cluster.local. IN A

;; ANSWER SECTION:
myapp-externalname.default.svc.cluster.local. 30 IN CNAME my.k8s.example.com.

;; Query time: 2072 msec
;; SERVER: 10.244.0.61#53(10.244.0.61)
;; WHEN: Wed Jun 03 23:15:47 CST 2020
;; MSG SIZE  rcvd: 149
```

# 7、ExternalIP示例

如果外部的 IP 路由到集群中一个或多个 Node 上，Kubernetes Service 会被暴露给这些 externalIPs。通过外部 IP（作为目的 IP 地址）进入到集群，打到 Service 端口上的流量，将会被路由到 Service 的 Endpoint 上。

externalIPs 不会被 Kubernetes 管理，它属于集群管理员的职责范畴。

根据 Service 的规定，externalIPs 可以同任意的 ServiceType 来一起指定。在下面的例子中，my-service 可以在【模拟外网IP】“10.0.0.240”(externalIP:port) 上被客户端访问。

yaml文件

```yaml
[root@k8s-master service]# pwd
/root/k8s_practice/service
[root@k8s-master service]# cat  myapp-svc-externalIP.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-externalip
  namespace: default
spec:
  selector:
    app: myapp
    release: v1
  ports:
  - name: http
    port: 80
    targetPort: 80
  externalIPs:
    - 10.0.0.240
```

启动Service并查看状态

```bash
[root@k8s-master service]# kubectl apply -f myapp-svc-externalIP.yaml
service/myapp-externalip created
[root@k8s-master service]#
[root@k8s-master service]# kubectl get svc -o wide
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP   22d   <none>
myapp-externalip   ClusterIP   10.107.186.167   10.0.0.240    80/TCP    8s    app=myapp,release=v1
```

查看ipvs信息

```bash
[root@k8s-master service]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
………………
TCP  10.107.186.167:80 rr
  -> 10.244.2.115:80              Masq    1      0          0
  -> 10.244.2.116:80              Masq    1      0          0
  -> 10.244.4.105:80              Masq    1      0          0
………………
TCP  10.0.0.240:80 rr
  -> 10.244.2.115:80              Masq    1      0          0
  -> 10.244.2.116:80              Masq    1      0          0
  -> 10.244.4.105:80              Masq    1      0          0
```

curl访问，通过ClusterIP

```bash
[root@k8s-master service]# curl 10.107.186.167
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master service]#
[root@k8s-master service]# curl 10.107.186.167/hostname.html
myapp-deploy-5695bb5658-n2b5w
[root@k8s-master service]#
[root@k8s-master service]# curl 10.107.186.167/hostname.html
myapp-deploy-5695bb5658-2866m
[root@k8s-master service]#
[root@k8s-master service]# curl 10.107.186.167/hostname.html
myapp-deploy-5695bb5658-dcfw7
```

curl访问，通过ExternalIP

```bash
[root@k8s-master service]# curl 10.0.0.240
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master service]#
[root@k8s-master service]# curl 10.0.0.240/hostname.html
myapp-deploy-5695bb5658-2866m
[root@k8s-master service]#
[root@k8s-master service]# curl 10.0.0.240/hostname.html
myapp-deploy-5695bb5658-dcfw7
[root@k8s-master service]#
[root@k8s-master service]# curl 10.0.0.240/hostname.html
myapp-deploy-5695bb5658-n2b5w
```

