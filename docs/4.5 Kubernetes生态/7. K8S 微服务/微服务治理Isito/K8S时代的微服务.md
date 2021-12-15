- [K8S时代的微服务](https://www.cnblogs.com/failymao/p/14523714.html)

## K8S时代的微服务

> 使用过`Istio` 的人可能都会有下面几个疑问？
>
> 1. 为什么`Istio` 要绑定`K8s`?
> 2. `Kubernetes` 和 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 分别在云原生中扮演什么角色？
> 3. [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) 扩展了 `Kubernetes` 的哪些方面？解决了哪些问题？
> 4. `Kubernetes`、`xDS` 协议（[Envoy](https://github.com/envoyproxy/envoy)、[MOSN](https://github.com/mosn/mosn) 等）与 [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) 之间又是什么关系？
> 5. 到底该不该上 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh)？
> 6. 假如你已经使用 `K8S` 构建了稳定的微服务平台，那么如何设置服务间调用的负载均衡和流量控制？

`Kubernetes` 的本质是通过声明式配置对应用进行生命周期管理，而 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 的本质是提供应用间的流量和安全性管理以及可观察性。

[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 创造的 `xDS` 协议被众多开源软件所支持，如 [Istio](https://github.com/istio/istio)、[Linkerd](https://linkerd.io/)、[MOSN](https://www.servicemesher.com/github.com/mosn/mosn) 等。[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 对于 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 或云原生来说**最大的贡献就是定义了 xDS**

[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 本质上是一个 proxy，是可通过` API` 配置的现代版 proxy

### 重要观点

1. `k8s`的本质是应用的生命周期管理，具体来说就是部署和管理（扩缩容、自动恢复、发布）。
2. `k8s`为微服务提供了可扩展、高弹性的部署和管理平台。
3. [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 的基础是透明代理，通过 [sidecar](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#sidecar) proxy 拦截到微服务间流量后再通过控制平面配置管理微服务的行为。--流量劫持，转发
4. [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 将流量管理从 `k8s`中解耦，[Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 内部的流量无需 `kube-proxy` 组件的支持，通过为更接近微服务应用层的抽象，管理服务间的流量、安全性和可观察性。
5. `xDS` 定义了 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 配置的协议标准。
6. [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 是对 `k8s` 中的 [service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service) 更上层的抽象，它的下一步是 `serverless`。

### `K8S` vs Service Mesh

服务访问关系如下图

![img](https://www.servicemesher.com/istio-handbook/images/kubernetes-vs-service-mesh.png)

#### K8S流量转发

`Kubernetes` 集群的每个节点都部署了一个 `kube-proxy` 组件，该组件会与 `Kubernetes API Server `通信，获取集群中的 `service`信息，然后设置 `iptables` 规则，直接将对某个 `service` 的请求发送到对应的 Endpoint（属于同一组 [service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service) 的 [pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod)）上。

#### Service Mesh中服务发现

[Service Mesh]*中的服务注册*如下图

![img](https://www.servicemesher.com/istio-handbook/images/istio-service-registry.png)

1. `Istio Service Mesh` 可以使用`k8s`中的service 做服务注册， 还可以通过控制平面的平台适配器对接其他服务器发现系统， 然后生成数据平面的配置（使用 [CRD](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#crd) 声明，保存在 etcd 中）；数据平面的**透明代理**（transparent proxy）以 [sidecar](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#sidecar) 容器的形式部署在每个应用服务的 [pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod) 中，这些 proxy 都需要请求控制平面来同步代理配置。该过程` kube-proxy` 组件一样需要拦截流量，只不过 `kube-proxy` 拦截的是进出 `Kubernetes` 节点的流量，而 [sidecar](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#sidecar) proxy 拦截的是进出该 [Pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod) 的流量

#### Service Mesh 的劣势

1. 因为`k8s`每个节点上都会运行众多的的Pod, 将原先的`kube-proxy`方式的路由转发功能置于每个pod中，将导致大量的配置分发，同步和最终一致性的问题。
2. 为了细粒度的进行流量管理， 必将添加一些列的抽象， 从而进一步增加用户的学习成本

#### Service Mesh 的优势

1. `kube-proxy` 的设置是全局的(对节点而言)， 无法对每个节点中运行的pod（服务）做细粒度的控制， 而 Service Mesh 通过 Sidecar Proxy (Envoy) 的方式将`k8s`中对流量的控制从Service 层抽离出来，可以做更多的扩展

### `kube-Proxy` 组件

- 每个 Node 运行一个 `kube-proxy` 进程。`kube-proxy` 负责为 `Service` 实现了一种 VIP（虚拟 IP）的形式。
- 从 `Kubernetes v1.2 `起，默认使用 `iptables` 代理。

#### `kube-Proxy`缺陷

1. 如果转发的 pod不能正常提供服务，它不会自动尝试另一个 pod，当然这个可以通过 [`liveness probes`](https://jimmysong.io/kubernetes-handbook/guide/configure-liveness-readiness-probes.html) 来解决---（每个 [pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod) 都有一个健康检查的机制，当有 [pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod) 健康状况有问题时，kube-proxy 会删除对应的转发规则）。
2. `nodePort` 类型的服务也无法添加`TLS` 或者更复杂的报文路由机制
3. `Kube-proxy` 实现了流量在 `Kubernetes` [service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service) 多个 [pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod) 实例间的负载均衡, 但是如何对各个service间的流量做细粒度的控制， 比如按照百分比划分流量到不同的应用版本（不同版本的服务都属于同一个service, 但是位于不同的deployment上）， 做金丝雀发布和蓝绿发布？

### `K8S` Ingress vs `Istio` Gateway

Ingress 资源位于 `K8S`边缘节点（可以是一个也可以是一组），负责管理 南北流量。

Ingress 必须对接各种 `Ingress Controller` 才能使用。Ingress 只适用于 HTTP 流量，使用方式也很简单，只能对 [service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service)、port、HTTP 路径等有限字段匹配来路由流量，这导致它无法路由如 MySQL、Redis 和各种私有 `RPC` 等 `TCP` 流量

[Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) Gateway 的功能与 `Kubernetes `Ingress 类似，都是负责集群的南北向流量, [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) `Gateway` 描述的负载均衡器用于承载进出网格边缘的连接。

### `xDS`协议

`xDS` 协议是由 [Envoy](https://envoyproxy.io/) 提出的，

支持 `xDS` 协议的代理通过查询文件或管理服务器来动态发现资源。

概括地讲，对应的发现服务及其相应的 API 被称作 *xDS*

Envoy 通过 订阅方式获取资源，订阅方式有三种

- 文件订阅： 监控指定路径下的文件。发现动态资源的最简单方式就是将其保存在文件中，并将路径配置在ConfigSource 中的path参数中
- gRPC流逝订阅：每个`xDS`API可以单独配置 `ApiConfigSource`，指向 对应的上游管理服务器的集群地址
- 轮询REST-JSON订阅： 每个`xDS` API 可对REST端点进行的同步（长）轮询

#### `xDS` 协议要点

协议要点：

- CDS、EDS、LDS、RDS 是最基础的 xDS 协议，它们可以分别独立更新。
- 所有的发现服务（Discovery [Service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service)）可以连接不同的 Management Server，也就是说管理 xDS 的服务器可以是多个。
- Envoy 在原始的xDS协议的基础上进行了一些阔绰， 增加了SDS（密钥发现服务），ADS(聚合发现服务)，HDS(健康发现服务)，MS（Metric服务），RLS（速率发现服务）
- 为了保证数据一致性，若直接使用 xDS 原始 API 的话，需要保证这样的顺序更新：`CDS --> EDS --> LDS --> RDS`，这是遵循电子工程中的**先合后断**（Make-Before-Break）原则，即在断开原来的连接之前先建立好新的连接，应用在路由里就是为了防止设置了新的路由规则的时候却无法发现上游集群而导致流量被丢弃的情况，类似于电路里的断路。
- CDS 设置 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 中有哪些服务。
- EDS 设置哪些实例（Endpoint）属于这些服务（[Cluster](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#cluster)）。
- LDS 设置实例上监听的端口以配置路由。
- RDS 最终服务间的路由关系，应该保证最后更新 RDS。

### Envoy

Envoy是 [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) [Service Mesh]中默认的 [Sidecar](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#sidecar)，[Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) 在 Envoy 的基础上按照 [Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 的 xDS 协议扩展了其控制平面.

#### 基本术语

- **Downstream（下游）**：下游主机连接到 Envoy，发送请求并接收响应，即发送请求的主机。
- **Upstream（上游）**：上游主机接收来自 [Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 的连接和请求，并返回响应，即接受请求的主机。
- **Listener（监听器）**：监听器是命名网地址（例如，端口、`unix domain socket` 等)，下游客户端可以连接这些监听器。[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 暴露一个或者多个监听器给下游主机连接。
- **[Cluster](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#cluster)（集群）**：集群是指 [Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 连接的一组逻辑相同的上游主机。[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 通过[服务发现](http://www.servicemesher.com/envoy/intro/arch_overview/service_discovery.html#arch-overview-service-discovery)来发现集群的成员。可以选择通过[主动健康检查](http://www.servicemesher.com/envoy/intro/arch_overview/health_checking.html#arch-overview-health-checking)来确定集群成员的健康状态。[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 通过[负载均衡策略](http://www.servicemesher.com/envoy/intro/arch_overview/load_balancing.html#arch-overview-load-balancing)决定将请求路由到集群的哪个成员。

[Envoy](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#envoy) 中可以设置多个 Listener，每个 Listener 中又可以设置 filter chain（过滤器链表），而且过滤器是可扩展的，这样就可以更方便我们操作流量的行为，例如设置加密、私有 RPC 等。

### `Istio Service Mesh`

`Istio service mesh`结构图如下

![img](https://www.servicemesher.com/istio-handbook/images/istio-mesh-arch.png)

`Istio`是一个功能十分丰富的 `Service Mesh`，它包括如下功能：

- 流量管理： 这是`Istio`的最基本的功能
- 策略控制： 通过`Mixer`组件喝各种适配器来实现， 实现访问控制系统，遥测捕获，额度管理喝计费
- 可观测性： 通过`Mixter`来实现
- 安全认证： Citadel组件做密钥和证书管理。

### Istio中的流量管理

- gateway ： 在网络边缘运行的负载均衡， 用于收入或传出HTTP/TCP链接
- `VirtualService`: 实际上将`k8s`服务连接到Gateway. 它还可以执行更多的操作，例如定义一组流量路由规则，以便在主机被寻址时应用
- **DestinationRule**：定义的策略，决定了经过路由处理之后的流量的访问策略。简单的说就是定义流量如何路由。这些策略中可以定义负载均衡配置、连接池尺寸以及外部检测（用于在负载均衡池中对不健康主机进行识别和驱逐）配置。
- **EnvoyFilter**：[`EnvoyFilter`](https://istio.io/docs/reference/config/networking/envoy-filter/) 对象描述了针对代理服务的过滤器，这些过滤器可以定制由 [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) [Pilot](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pilot) 生成的代理配置。这个配置初级用户一般很少用到。
- **ServiceEntry**：默认情况下 [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 中的服务是无法发现 Mesh 外的服务的，[`ServiceEntry`](https://istio.io/docs/reference/config/networking/service-entry/) 能够在 [Istio](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#istio) 内部的服务注册表中加入额外的条目，从而让网格中自动发现的服务能够访问和路由到这些手工加入的服务

### 小结

如果说 `Kubernetes` 管理的对象是 [Pod](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#pod)，那么 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 中管理的对象就是一个个 [Service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service)，所以说使用 `Kubernetes` 管理微服务后再应用 [Service Mesh](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service-mesh) 就是水到渠成了，如果连 [Service](https://www.servicemesher.com/istio-handbook/GLOSSARY.html#service) 你也不想管了，那就用如 [knative](https://github.com/knative/) 这样的 `serverless` 平台。