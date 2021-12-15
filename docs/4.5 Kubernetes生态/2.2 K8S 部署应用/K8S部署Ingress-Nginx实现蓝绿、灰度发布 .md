作者：chulinx
链接：https://juejin.cn/post/6844903927318577159
来源：掘金



## 背景介绍

> 某些情况下，我们在使用Kubernetes作为业务应用的云平台，想要实现应用的蓝绿部署用来迭代应用版本，用lstio太重太复杂，而且它本身定位于流控和网格治理；Ingress-Nginx在0.21版本引入了Canary功能，可以为网关入口配置多个版本的应用程序，使用annotation来控制多个后端服务的流量分配

## Ingress-Nginx-Annotation Canary 功能介绍

> 如果想启用Canary功能，要先设置`nginx.ingress.kubernetes.io/canary: "true"`，然后可以启用以下注释来配置Canary

- `nginx.ingress.kubernetes.io/canary-weight` 请求到Canary ingress中指定的服务的请求百分比，值为0-100的整数，根据设置的值来决定大概有百分之多少的流量会分配Canary Ingress中指定的后端s服务
- `nginx.ingress.kubernetes.io/canary-by-header` 基于request header 的流量切分，适用于灰度发布或者A/B测试，当设定的hearder值为always是，请求流量会被一直分配到Canary入口，当hearder值被设置为never时，请求流量不会分配到Canary入口，对于其他hearder值，将忽略，并通过优先级将请求流量分配到其他规则
- `nginx.ingress.kubernetes.io/canary-by-header-value` 这个配置要和`nginx.ingress.kubernetes.io/canary-by-header` 一起使用，当请求中的hearder key和value 和`nginx.ingress.kubernetes.io/canary-by-header` `nginx.ingress.kubernetes.io/canary-by-header-value`匹配时，请求流量会被分配到Canary Ingress入口，对于其他任何hearder值，将忽略，并通过优先级将请求流量分配到其他规则
- `nginx.ingress.kubernetes.io/canary-by-cookie` 这个配置是基于cookie的流量切分，也适用于灰度发布或者A/B测试，当cookie值设置为always时，请求流量将被路由到Canary Ingress入口，当cookie值设置为never时，请求流量将不会路由到Canary入口，对于其他值，将忽略，并通过优先级将请求流量分配到其他规则

> 金丝雀规则按优先顺序进行如下排序：canary-by-header - > canary-by-cookie - > canary-weight

### 1.基于权重的小规模版本测试

- v1版本编排文件

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  labels:
    app: echoserverv1
  name: echoserverv1
  namespace: echoserver
spec:
  rules:
  - host: echo.chulinx.com
    http:
      paths:
      - backend:
          serviceName: echoserverv1
          servicePort: 8080
        path: /
---
kind: Service
apiVersion: v1
metadata:
  name:  echoserverv1
  namespace: echoserver
spec:
  selector:
    name:  echoserverv1
  type:  ClusterIP
  ports:
  - name:  echoserverv1
    port:  8080
    targetPort:  8080
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  echoserverv1
  namespace: echoserver
  labels:
    name:  echoserverv1
spec:
  template:
    metadata:
      labels:
        name:  echoserverv1
    spec:
      containers:
      - image:  mirrorgooglecontainers/echoserver:1.10
        name:  echoserverv1 
        ports:
        - containerPort:  8080
          name:  echoserverv1
复制代码
```

- 查看v1版本创建的资源

```
$ [K8sSj] kubectl get pod,service,ingress -n echoserver
NAME                                READY   STATUS    RESTARTS   AGE
pod/echoserverv1-657b966cb5-7grqs   1/1     Running   0          24h

NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/echoserverv1   ClusterIP   10.99.68.72   <none>        8080/TCP   24h

NAME                              HOSTS              ADDRESS   PORTS   AGE
ingress.extensions/echoserverv1   echo.chulinx.com             80      24h
复制代码
```

- 访问v1的服务,可以看到10个请求都是访问到一个pod上也就是v1版本的服务

```
$ [K8sSj] for i in `seq 10`;do curl -s echo.chulinx.com|grep Hostname;done
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
复制代码
```

- 创建v2版本的服务

> 我们开启canary功能，将v2版本的权重设置为50%,这个百分比并不能精确的将请求平均分配到两个版本的服务，而是在50%上下浮动

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "50"
  labels:
    app: echoserverv2
  name: echoserverv2
  namespace: echoserver
spec:
  rules:
  - host: echo.chulinx.com
    http:
      paths:
      - backend:
          serviceName: echoserverv2
          servicePort: 8080
        path: /
---
kind: Service
apiVersion: v1
metadata:
  name:  echoserverv2
  namespace: echoserver
spec:
  selector:
    name:  echoserverv2
  type:  ClusterIP
  ports:
  - name:  echoserverv2
    port:  8080
    targetPort:  8080
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  echoserverv2
  namespace: echoserver
  labels:
    name:  echoserverv2
spec:
  template:
    metadata:
      labels:
        name:  echoserverv2
    spec:
      containers:
      - image:  mirrorgooglecontainers/echoserver:1.10
        name:  echoserverv2 
        ports:
        - containerPort:  8080
          name:  echoserverv2
复制代码
```

- 再次查看创建的资源

```
$ [K8sSj] kubectl get pod,service,ingress -n echoserver
NAME                                READY   STATUS    RESTARTS   AGE
pod/echoserverv1-657b966cb5-7grqs   1/1     Running   0          24h
pod/echoserverv2-856bb5758-f9tqn    1/1     Running   0          4s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/echoserverv1   ClusterIP   10.99.68.72      <none>        8080/TCP   24h
service/echoserverv2   ClusterIP   10.111.103.170   <none>        8080/TCP   4s

NAME                              HOSTS              ADDRESS   PORTS   AGE
ingress.extensions/echoserverv1   echo.chulinx.com             80      24h
ingress.extensions/echoserverv2   echo.chulinx.com             80      4s
复制代码
```

- 访问测试

> 可以看到请求有4个落到v2版本，6个落到v1版本，理论上来说，请求说越多，落到v2版本的请求数越接近设置的权重50%

```
$ [K8sSj] for i in `seq 10`;do curl -s echo.chulinx.com|grep Hostname;done
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
复制代码
```

### 2.基于header的A/B测试

- 更改v2版本的编排文件

> 增加header`nginx.ingress.kubernetes.io/canary-by-header: "v2"`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "50"
    nginx.ingress.kubernetes.io/canary-by-header: "v2"
  labels:
    app: echoserverv2
  name: echoserverv2
  namespace: echoserver
spec:
  rules:
  - host: echo.chulinx.com
    http:
      paths:
      - backend:
          serviceName: echoserverv2
          servicePort: 8080
        path: /
---
kind: Service
apiVersion: v1
metadata:
  name:  echoserverv2
  namespace: echoserver
spec:
  selector:
    name:  echoserverv2
  type:  ClusterIP
  ports:
  - name:  echoserverv2
    port:  8080
    targetPort:  8080
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  echoserverv2
  namespace: echoserver
  labels:
    name:  echoserverv2
spec:
  template:
    metadata:
      labels:
        name:  echoserverv2
    spec:
      containers:
      - image:  mirrorgooglecontainers/echoserver:1.10
        name:  echoserverv2 
        ports:
        - containerPort:  8080
          name:  echoserverv2
复制代码
```

- 更新访问测试

> 测试了header 为`v2:always` `v2:never` `v2:true`这三个hearder值，可以看到当hearder为`v2:always`时，流量会全部流入v2，当`v2:never`时，流量会全部流入v1，当`v2:true`时，也就是非`always/never`,流量会按照配置的权重流入对应版本的服务

```
$ [K8sSj] kubectl apply -f appv2.yml
ingress.extensions/echoserverv2 configured
service/echoserverv2 unchanged
deployment.extensions/echoserverv2 unchanged

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:always" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:never" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:true" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
复制代码
```

- 自定义header-value

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "50"
    nginx.ingress.kubernetes.io/canary-by-header: "v2"
    nginx.ingress.kubernetes.io/canary-by-header-value: "true"
  labels:
    app: echoserverv2
  name: echoserverv2
  namespace: echoserver
spec:
  rules:
  - host: echo.chulinx.com
    http:
      paths:
      - backend:
          serviceName: echoserverv2
          servicePort: 8080
        path: /
---
kind: Service
apiVersion: v1
metadata:
  name:  echoserverv2
  namespace: echoserver
spec:
  selector:
    name:  echoserverv2
  type:  ClusterIP
  ports:
  - name:  echoserverv2
    port:  8080
    targetPort:  8080
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  echoserverv2
  namespace: echoserver
  labels:
    name:  echoserverv2
spec:
  template:
    metadata:
      labels:
        name:  echoserverv2
    spec:
      containers:
      - image:  mirrorgooglecontainers/echoserver:1.10
        name:  echoserverv2 
        ports:
        - containerPort:  8080
          name:  echoserverv2
复制代码
```

- 更新测试

> 可以看到只有header为`v2:never`时，请求流量才会流入v2版本，其他值流量都会按照权重设置流入不通版本的服务

```
$ [K8sSj] kubectl apply -f appv2.yml
ingress.extensions/echoserverv2 configured
service/echoserverv2 unchanged
deployment.extensions/echoserverv2 unchanged

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:true" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:always" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn

$ [K8sSj] for i in `seq 10`;do curl -s -H "v2:never" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
复制代码
```

### 3.基于cookie的流控

> cookie其实和header原理大致相同，也是ingress自动cookie值，客户访问如果cookie匹配，流量就会流入与之匹配的后端服务

- 更新v2版本的编排文件

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "50"
    nginx.ingress.kubernetes.io/canary-by-header: "v2"
    nginx.ingress.kubernetes.io/canary-by-header-value: "true"
    nginx.ingress.kubernetes.io/canary-by-cookie: "user_from_shanghai"
  labels:
    app: echoserverv2
  name: echoserverv2
  namespace: echoserver
spec:
  rules:
  - host: echo.chulinx.com
    http:
      paths:
      - backend:
          serviceName: echoserverv2
          servicePort: 8080
        path: /
---
kind: Service
apiVersion: v1
metadata:
  name:  echoserverv2
  namespace: echoserver
spec:
  selector:
    name:  echoserverv2
  type:  ClusterIP
  ports:
  - name:  echoserverv2
    port:  8080
    targetPort:  8080
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name:  echoserverv2
  namespace: echoserver
  labels:
    name:  echoserverv2
spec:
  template:
    metadata:
      labels:
        name:  echoserverv2
    spec:
      containers:
      - image:  mirrorgooglecontainers/echoserver:1.10
        name:  echoserverv2 
        ports:
        - containerPort:  8080
          name:  echoserverv2
复制代码
```

- 访问测试

> 可以看和header的访问效果是一样的，只不过cookie不能自定义value

```
$ [K8sSj] kubectl apply -f appv2.yml
ingress.extensions/echoserverv2 configured
service/echoserverv2 unchanged
deployment.extensions/echoserverv2 unchanged

$ [K8sSj] for i in `seq 10`;do curl -s --cookie "user_from_shanghai" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn

# zlx @ zlxdeMacBook-Pro in ~/Desktop/unicom/k8syml/nginx-ingress-canary-deployment [16:01:52]
$ [K8sSj] for i in `seq 10`;do curl -s --cookie "user_from_shanghai:always" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv1-657b966cb5-7grqs
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn

# zlx @ zlxdeMacBook-Pro in ~/Desktop/unicom/k8syml/nginx-ingress-canary-deployment [16:02:25]
$ [K8sSj] for i in `seq 10`;do curl -s --cookie "user_from_shanghai=always" echo.chulinx.com|grep Hostname;done
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
Hostname: echoserverv2-856bb5758-f9tqn
复制代码
```

## 总结

> 灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以对新版本进行测试、发现和调整问题，以保证其影响度,以上内容通过实例详细介绍了Ingress-Nginx的实战Canary Annotation，可以借助Ingress-Nginx轻松实现蓝绿发布和金丝雀发布

### 其他

#### 关于蓝绿发布、金丝雀发布、和A/B测试

- 蓝绿发布

> 蓝绿部署中，一共有两套系统：一套是正在提供服务系统，标记为“绿色”；另一套是准备发布的系统，标记为“蓝色”。两套系统都是功能完善的，并且正在运行的系统，只是系统版本和对外服务情况不同。 最初，没有任何系统，没有蓝绿之分。 然后，第一套系统开发完成，直接上线，这个过程只有一个系统，也没有蓝绿之分。 后来，开发了新版本，要用新版本替换线上的旧版本，在线上的系统之外，搭建了一个使用新版本代码的全新系统。 这时候，一共有两套系统在运行，正在对外提供服务的老系统是绿色系统，新部署的系统是蓝色系统。 蓝色系统不对外提供服务，用来做啥？ 用来做发布前测试，测试过程中发现任何问题，可以直接在蓝色系统上修改，不干扰用户正在使用的系统。（注意，两套系统没有耦合的时候才能百分百保证不干扰） 蓝色系统经过反复的测试、修改、验证，确定达到上线标准之后，直接将用户切换到蓝色系统： 切换后的一段时间内，依旧是蓝绿两套系统并存，但是用户访问的已经是蓝色系统。这段时间内观察蓝色系统（新系统）工作状态，如果出现问题，直接切换回绿色系统。 当确信对外提供服务的蓝色系统工作正常，不对外提供服务的绿色系统已经不再需要的时候，蓝色系统正式成为对外提供服务系统，成为新的绿色系统。 原先的绿色系统可以销毁，将资源释放出来，用于部署下一个蓝色系统。 蓝绿部署只是上线策略中的一种，它不是可以应对所有情况的万能方案。 蓝绿部署能够简单快捷实施的前提假设是目标系统是非常内聚的，如果目标系统相当复杂，那么如何切换、两套系统的数据是否需要以及如何同步等，都需要仔细考虑。

- 金丝雀发布

> 金丝雀发布（Canary）也是一种发布策略，和国内常说的灰度发布是同一类策略。蓝绿部署是准备两套系统，在两套系统之间进行切换，金丝雀策略是只有一套系统，逐渐替换这套系统 譬如说，目标系统是一组无状态的Web服务器，但是数量非常多，假设有一万台。 这时候，蓝绿部署就不能用了，因为你不可能申请一万台服务器专门用来部署蓝色系统（在蓝绿部署的定义中，蓝色的系统要能够承接所有访问）。 可以想到的一个方法是： 只准备几台服务器，在上面部署新版本的系统并测试验证。测试通过之后，担心出现意外，还不敢立即更新所有的服务器。 先将线上的一万台服务器中的10台更新为最新的系统，然后观察验证。确认没有异常之后，再将剩余的所有服务器更新。 这个方法就是金丝雀发布。 实际操作中还可以做更多控制，譬如说，给最初更新的10台服务器设置较低的权重、控制发送给这10台服务器的请求数，然后逐渐提高权重、增加请求数。 这个控制叫做“流量切分”，既可以用于金丝雀发布，也可以用于后面的A/B测试。 蓝绿部署和金丝雀发布是两种发布策略，都不是万能的。有时候两者都可以使用，有时候只能用其中一种。

- A/B测试

> 首先需要明确的是，A/B测试和蓝绿部署以及金丝雀，完全是两回事。 蓝绿部署和金丝雀是发布策略，目标是确保新上线的系统稳定，关注的是新系统的BUG、隐患。 A/B测试是效果测试，同一时间有多个版本的服务对外服务，这些服务都是经过足够测试，达到了上线标准的服务，有差异但是没有新旧之分（它们上线时可能采用了蓝绿部署的方式）。 A/B测试关注的是不同版本的服务的实际效果，譬如说转化率、订单情况等。 A/B测试时，线上同时运行多个版本的服务，这些服务通常会有一些体验上的差异，譬如说页面样式、颜色、操作流程不同。相关人员通过分析各个版本服务的实际效果，选出效果最好的版本。 在A/B测试中，需要能够控制流量的分配，譬如说，为A版本分配10%的流量，为B版本分配10%的流量，为C版本分配80%的流量。


