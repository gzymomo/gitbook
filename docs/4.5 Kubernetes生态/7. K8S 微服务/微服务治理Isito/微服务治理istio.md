- [微服务治理istio](https://www.cnblogs.com/yuezhimi/p/13100265.html)
- [istio的原理和功能介绍](https://www.cnblogs.com/JoZSM/p/10902306.html)

# 一、Service Mesh 

Service Mesh 的中文译为“服务网格”，是一个用于处理服务和服务之间通信的基础设施层，它负责为构建复杂的云原生应用传递可靠的网络请求，并为服务通信实现了微服务所需的基本组件功能，例如服务发现、负载均衡、监控、流量管理、访问控制等。在实践中，服务网格通常实现为一组和应用程序部署在一起的轻量级的网络代理，但对应用程序来说是透明的。

Service Mesh 部署网络结构图

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612110601388-532041487.png)

- 治理能力独立（Sidecar）
- 应用程序无感知
- 服务通信的基础设施层
- 解耦应用程序的重试/超时、监控、追踪和服务发现

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612110656457-183150046.png)

# 二、Istio 概述

Isito是Service Mesh的产品化落地，是目前最受欢迎的服务网格，功能丰富、成熟度高。
Linkerd是世界上第一个服务网格类的产品。

- 连接（Connect）
  - 流量管理
  - 负载均衡
  - 灰度发布
- 安全（Secure）
  - 认证
  - 鉴权
- 控制（Control）
  - 限流
  - ACL
- 观察（Observe）
  - 监控
  - 调用链

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612110839129-1740140046.png)

istio与kubernetes结合

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612111531541-601246704.png)

# 三、istio架构与组件

- 数据平面：由一组代理组成，这些代理微服务所有网络通信，并接收和实施来自Mixer的策略。
- Proxy：负责高效转发与策略实现。
- 控制平面：管理和配置代理来路由流量。此外，通过mixer实施策略与收集来自边车代理的数据。
- Mixer：适配组件，数据平面与控制平面通过它交互，为Proxy提供策略和数据上报。
- Pilot：策略配置组件，为Proxy提供服务发现、智能路由、错误处理等。
- Citadel：安全组件，提供证书生成下发、加密通信、访问控制。
- Galley：配置管理、验证、分发。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612111739757-744264041.png)

# 四、istio基本概念

Istio有4个配置资源，落地所有流量管理需求：

- VirtualService：实现服务请求路由规则的功能。
- DestinationRule：实现目标服务的负载均衡、服务发现、故障处理和故障注入的功能。
- Gateway：让服务网格内的服务，可以被全世界看到。
- ServiceEntry：让服务网格内的服务，可以看到外面的世界。

# 五、在Kubernetes 部署Istio

```bash
wget https://github.com/istio/istio/releases/download/1.4.2/istio-1.4.2-linux.tar.gz
tar zxvf istio-1.4.2-linux.tar.gz
cd istio-1.4.2
mv bin/istioctl /usr/bin

istioctl profile list
istioctl manifest apply --set profile=demoistioctl manifest generate --set profile=demo  #获取yaml
istioctl manifest generate --set profile=demo | kubectl delete -f -  #卸载
```

查看运行成功

```bash
# kubectl get pod -n istio-system
NAME                                      READY   STATUS    RESTARTS   AGE
grafana-6b65874977-crbm9                  1/1     Running   0          4m13s
istio-citadel-86dcf4c6b-6qg8f             1/1     Running   0          4m17s
istio-egressgateway-68f754ccdd-vgnnt      1/1     Running   0          4m20s
istio-galley-5fc6d6c45b-ckqhs             1/1     Running   0          4m13s
istio-ingressgateway-6d759478d8-w29kh     1/1     Running   0          4m17s
istio-pilot-5c4995d687-phbtd              1/1     Running   0          4m21s
istio-policy-57b99968f-mbxjg              1/1     Running   5          4m24s
istio-sidecar-injector-746f7c7bbb-f4jmv   1/1     Running   0          4m13s
istio-telemetry-854d8556d5-s5948          1/1     Running   4          4m18s
istio-tracing-c66d67cd9-2btc9             1/1     Running   0          4m18s
kiali-8559969566-npcrl                    1/1     Running   0          4m22s
prometheus-66c5887c86-wbftc               1/1     Running   0          4m17s
```

##### Sidercar 注入

部署httpbin Web示例：

```bash
# 手动注入
kubectl apply -f <(istioctl kube-inject -f httpbin-nodeport.yaml)
或者
istioctl kube-inject -f httpbin-nodeport.yaml|kubectl apply -f -

# 自动注入
kubectl label namespace default istio-injection=enabled
kubectl apply -f httpbin-gateway.yaml
```

```
# kubectl get pod
NAME                       READY   STATUS    RESTARTS   AGE
httpbin-5c8ff7878b-jtp8d   2/2     Running   0          4m44s
# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
httpbin      NodePort    10.0.0.8     <none>        8000:31378/TCP   23m
```

NodePort访问地址：http://192.168.0.134:31378

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612144415715-130197158.png)

```bash
[root@k8s-master httpbin]# kubectl apply -f httpbin-gateway.yaml 
gateway.networking.istio.io/httpbin-gateway created
virtualservice.networking.istio.io/httpbin created
[root@k8s-master httpbin]# kubectl get gateway
NAME              AGE
httpbin-gateway   46s
[root@k8s-master httpbin]# kubectl get virtualservice
NAME      GATEWAYS            HOSTS   AGE
httpbin   [httpbin-gateway]   [*]     73s
[root@k8s-master httpbin]# kubectl get svc -n istio-system |grep ingress
istio-ingressgateway     LoadBalancer   10.0.0.159   <pending>     15020:30192/TCP,80:31025/TCP,443:30335/TCP,15029:31780/TCP,15030:30496/TCP,15031:30126/TCP,15032:32066/TCP,15443:32306/TCP   142m
```

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612144525373-2019144540.png)

ingressgateway访问地址：http://192.168.0.134:31025/

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612145605093-464718032.png)

##### 服务网关：Gateway

Gateway为网格内服务提供负载均衡器，提供以下功能：

- L4-L7的负载均衡
- 对外的mTLS

Gateway根据流入流出方向分为：

- IngressGateway：接收外部访问，并将流量转发到网格内的服务。
- EgressGateway：网格内服务访问外部应用。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612144709389-907886853.png)

##### 部署bookinfo 微服务示例

Bookinfo 应用分为四个单独的微服务：

- productpage ：productpage 微服务会调用details 和reviews 两个微服务，用来生成页面。
- details ：这个微服务包含了书籍的信息。
- reviews ：这个微服务包含了书籍相关的评论。它还会调用ratings 微服务。
- ratings ：ratings 微服务中包含了由书籍评价组成的评级信息。

reviews 微服务有3 个版本：

- v1 版本不会调用ratings 服务。
- v2 版本会调用ratings 服务，并使用5个黑色五角星来显示评分信息。
- v3 版本会调用ratings 服务，并使用5个红色五角星来显示评分信息。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612181358935-1166127431.png)

```bash
kubectl create ns bookinfo
kubectl label namespace bookinfo istio-injection=enabled
cd istio-1.4.2/samples/bookinfo
kubectl apply -f platform/kube/bookinfo.yaml -n bookinfo
kubectl get pod -n bookinfo
kubectl apply -f networking/bookinfo-gateway.yaml -n bookinfo
```

访问地址：http://192.168.0.134:31025/productpage

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612181519441-907624305.png)

# 六、istio实现灰度发布

主流发布方案：

- 蓝绿发布
- 滚动发布
- 灰度发布（金丝雀发布）
- A/B Test

## 6.1 蓝绿发布

项目逻辑上分为AB组，在项目升级时，首先把A组从负载均衡中摘除，进行新版本的部署。B组仍然继续提供服务。A组升级完成上线，B组从负载均衡中摘除。
特点：

- 策略简单
- 升级/回滚速度快
- 用户无感知，平滑过渡

缺点：

- 需要两倍以上服务器资源
- 短时间内浪费一定资源成本
- 有问题影响范围大

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612181824512-1497636328.png)

## 6.2 滚动发布

每次只升级一个或多个服务，升级完成后加入生产环境，不断执行这个过程，直到集群中的全部旧版升级新版本。Kubernetes的默认发布策略。
特点：

- 用户无感知，平滑过渡

缺点：

- 部署周期长
- 发布策略较复杂
- 不易回滚
- 有影响范围较大

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612181938737-621207393.png)

## 6.3 灰度发布（金丝雀发布）

只升级部分服务，即让一部分用户继续用老版本，一部分用户开始用新版本，如果用户对新版本没有什么意见，那么逐步扩大范围，把所有用户都迁移到新版本上面来。
特点：

- 保证整体系统稳定性
- 用户无感知，平滑过渡

缺点：

- 自动化要求高

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612182042138-595424230.png)

## 6.4 A/B Test

灰度发布的一种方式，主要对特定用户采样后，对收集到的反馈数据做相关对比，然后根据比对结果作出决策。用来测试应用功能表现的方法，侧重应用的可用性，受欢迎程度等，最后决定是否升级。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612182140014-402606740.png)

## 6.5 基于权重的路由（金丝雀发布）

任务：
1.流量全部发送到reviews v1版本（不带五角星）
2.将90%的流量发送到reviews v1版本，另外10%的流量发送到reviews v2版本（5个黑色五角星），最后完全切换到v2版本
3.将50%的流量发送到v2版本，另外50%的流量发送到v3版本（5个红色五角星）

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612182256805-924410667.png)

service名称 端口
1、全部到v1
virtualservice -> subset -> destinationrule定义的subset名称 -> 根据标签匹配pod

```bash
kubectl apply -f networking/virtual-service-all-v1.yaml -n bookinfo
kubectl apply -f networking/destination-rule-all.yaml -n bookinfo
```

2、90%v1 10%v2 增加两个版本权重 virtualservice -> subset

```bash
kubectl apply -f networking/virtual-service-reviews-90-10.yaml -n bookinfo
```

3、50%v2 50%v3 调整权重值

```bash
kubectl apply -f networking/virtual-service-reviews-v2-v3.yaml -n bookinfo
```

## 6.6 基于请求内容的路由（A/B Test）

任务：
1.将特定用户的请求发送到reviews v2版本（5个黑色五角星），其他用户则不受影响（v3）

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612202356465-730437372.png)

```bash
kubectl apply -f networking/virtual-service-reviews-jason-v2-v3.yaml -n bookinfo
```

一个项目使用istio做灰度发布：
1、使用deployment部署这项目，并增加一个独立用于区分版本的标签：version: v1
2、添加istio灰度发布规则（virtualservice、destinationrule）
3、灰度发布准备先将新的版本部署k8s中

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612202857237-1173170742.png)

# 七、可视化监控

- 监控指标（Grafana）
- 网格可视化（Kiali）
- 调用链跟踪（Jaeger）

通过ingressgateway暴露服务

```yaml
# cat monitor-gateway.yaml
---
# 监控指标
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: grafana-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: grafana
spec:
  hosts:
  - "grafana.cnntp.cn"
  gateways:
  - grafana-gateway
  http:
  - route:
    - destination:
        host: grafana 
        port:
          number: 3000

---
# 网格可视化 Kiali 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: kiali-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: kiali
spec:
  hosts:
  - "kiali.cnntp.cn"
  gateways:
  - kiali-gateway
  http:
  - route:
    - destination:
        host: kiali 
        port:
          number: 20001 
---
# 调用链
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: tracing-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tracing 
spec:
  hosts:
  - "tracing.cnntp.cn"
  gateways:
  - tracing-gateway
  http:
  - route:
    - destination:
        host: tracing
        port:
          number: 80

# kubectl apply -f monitor-gateway.yaml -n istio-system
# kubectl get gateway -n istio-system
NAME                               AGE
grafana-gateway                    18m
ingressgateway                     7h33m
istio-multicluster-egressgateway   7h33m
kiali-gateway                      18m
tracing-gateway                    18m
```

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612210320048-636166600.png)

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612211313265-1808686812.png)



![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200612211738681-1353685367.png)

## 7.1 istio解决的问题

1. 故障排查
2. 应用容错性
3. 应用升级发布
4. 系统安全性

