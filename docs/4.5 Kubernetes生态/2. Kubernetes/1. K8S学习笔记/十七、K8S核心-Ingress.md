# 1、Ingress

Kubernetes 提供了两种内建的云端负载均衡机制（ cloud load balancing ）用于发布公共应用， 工作于传输层的 Service 资源，它实现的是 TCP 负载均衡器”，另种是Ingress 资源，它实现的是“ HTTP(S ）负载均衡器”



<font color='red'>Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP和HTTPS。</font>

Ingress 可以提供负载均衡、SSL 和基于名称的虚拟托管。

<font color='red'>必须具有 ingress 控制器【例如 ingress-nginx】才能满足 Ingress 的要求。仅创建 Ingress 资源无效。</font>

## Ingress 是什么

Ingress 公开了从集群外部到集群内 services 的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

```
1  internet
2      |
3 [ Ingress ]
4 --|-----|--
5 [ Services ]
```

 

可以将 Ingress 配置为提供服务外部可访问的 URL、负载均衡流量、 SSL / TLS，以及提供基于名称的虚拟主机。Ingress 控制器 通常负责通过负载均衡器来实现 Ingress，尽管它也可以配置边缘路由器或其他前端来帮助处理流量。

Ingress 不会公开任意端口或协议。若将 HTTP 和 HTTPS 以外的服务公开到 Internet 时，通常使用 Service.Type=NodePort 或者 Service.Type=LoadBalancer 类型的服务。

以Nginx Ingress为例，图如下

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923212751953-1504280368.png)

 

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923212805642-433152881.png)



## Ingress示例

### 架构图

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923212820372-429939181.png)

**Ingress和Ingress Controller**

Ingress是Kubernetes API的标准资源类型之一，它其实就是基于DNS名称(host)或URL路径把请求转发至指定的Service资源的规则，用于将集群外部的请求流量转发至集群内部完成服务发布。然而，**Ingess资源自身并不能运行“流量穿透”，它仅仅是一组路由规则的集合，这些规则要想真正发挥作用还需其它功能的辅助，如监听某个套接字上，根据这些规则匹配机制路由请求流量：这种能够为Ingress资源监听套接字并转发流量的组件称之为Ingress控制器**

注意：
**不同于 Deployment 控制器等 Ingress 控制器并不直接运行为 kube-controller-rnanager的一部 ，它是Kubemetes集群的重要附件类似于 CoreDNS 需要在集群单独部署**



Ingress不是Kubernetes内置的，需要单独安装应用，来做负载均衡。



将端口号对外暴露，通过IP+端口号进行访问。

- 使用Service里面的NodePort可以实现。



NodePort缺陷：

- 在每个节点上都会起到端口，在访问时候通过任何节点，通过节点ip+暴露端口号实现访问。
- 意味着每个端口只能使用一次，一个端口对应一个应用。
- 实际访问中都是用域名，根据不同域名跳转到不同端口服务中。



## 1.1 Ingress和Pod关系

- pod和ingress通过service关联的
  - ingress作为统一入口，由service关联一组pod



![](..\..\img\ingress.png)



## 1.2 Ingress工作流程

![](..\..\img\ingress1.png)



## 1.3 使用ingress步骤

1. 部署ingress controller（选择官方nginx控制器，实现部署） 

```bash

```



2. 创建ingress规则

```bash

```



## 1.4 使用ingress对外暴露应用

1. 创建nginx应用，对外暴露端口使用NodePort

```bash
# 创建pod
kubectl create deployment web --image=nginx

# 查看
kubectl get pods

# 创建service
kubectl expose deployment web --port=80 --target-port=80 --type=NodePort
```

2. 部署ingress controller

```bash
kubectl apply -f ingress-con.yaml

# 查看ingress controller状态
kubectl get pods -n ingress-nginx
```

3. 创建ingress规则

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
meatadata:
  name: example-ingress
spec:
  rules:
  - host: example.ingredemo.com
    http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 80
```



保存后，然后执行

```bash
# 创建ingress规则
kubectl applf -f ingress-http.yaml
```



查看在那个节点：

```bash
kubectl get pods -n ingress-nginx -o wide
```



# 2、[Kubernetes Ingress-nginx使用](https://www.cnblogs.com/precipitation/p/14079472.html)

## 3.1 部署Ingress-Controller

[官方地址](https://kubernetes.github.io/ingress-nginx/deploy/)

此处部署3.0版本

在你需要部署的node节点上拉去Ingerss-Controller镜像

```bash
[root@k8s-master01 daem]# docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
```

**进入到GitHub上将mandatory.yaml复制到node节点上**

[mandatory.yaml地址](https://github.com/kubernetes/ingress-nginx/tree/nginx-0.30.0/deploy/static)

```yaml
[root@k8s-master01 ingressdeploy]# cat mandatory.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
#data:
#  whitelist-source-range: 192.168.29.102 #白名单，允许某个IP或IP段的访问
#  block-cidrs: 192.168.29.101 #黑名单拒绝访问

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      hostNetwork: true      ###修改成hostNetwork模式直接共享服务器的网络名称空间
      # wait up to five minutes for the drain of connections
      terminationGracePeriodSeconds: 300
      serviceAccountName: nginx-ingress-serviceaccount
      nodeSelector:
        kubernetes.io/os: linux
        kubernetes.io/hostname: k8s-master02
      dnsPolicy: ClusterFirstWithHostNet
      containers:
        - name: nginx-ingress-controller
          imagePullPolicy: IfNotPresent 
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 101
            runAsUser: 101
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
              #hostPort: 80
            - name: https
              containerPort: 443
              protocol: TCP
              #hostPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown

---

apiVersion: v1
kind: LimitRange
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  limits:
  - min:
      memory: 90Mi
      cpu: 100m
    type: Container
```

需修改：
hostNetwork: true ###修改成hostNetwork模式直接共享服务器的网络名称空间

执行create创建Ingress-Controller

```bash
[root@k8s-master01 ingressdeploy]# kubectl get deploy -n ingress-nginx
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress-controller   1/1     1            1           76m
```

Ingress-Controller已部署完成

## 3.2 使用Ingress规则

创建测试的web应用

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
      - name: nginxweb
        image: nginx:1.15-alpine
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
    port: 80
    protocol: TCP
    nodePort: 30001   #node节点的映射端口 可以通过外部访问
    targetPort: 80
  selector:
    app: mynginx
  sessionAffinity: None
  type: NodePort
```

创建Ingress规则

```yaml
[root@k8s-master01 daem]# cat ingress.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-daem
  annotations:
    kubernetes.io/ingress.class: "nginx" 
    #nginx.ingress.kubernetes.io/limit-connections: 10
    #nginx.ingress.kubernetes.io/limit-rate: 100K
    #nginx.ingress.kubernetes.io/limit-rps: 1
    #nginx.ingress.kubernetes.io/limit-rpm: 30
spec:
  rules:
  - host: test.nginxsvc.com
    http:
      paths:
      - backend:
          serviceName: nginx-svc
          servicePort: 80
        path: /
```

浏览器访问
添加hosts解析
192.168.29.102 test.nginxsvc.com test-tls.test.com
![img](https://img2020.cnblogs.com/blog/2005433/202012/2005433-20201203150737686-801306076.png)

```bash
[root@k8s-master01 daem]# curl -I http://test.nginxsvc.com/
HTTP/1.1 200 OK
Server: nginx/1.17.8
Date: Thu, 03 Dec 2020 06:56:49 GMT
Content-Type: text/html
Content-Length: 612
Connection: keep-alive
Vary: Accept-Encoding
Last-Modified: Sat, 11 May 2019 00:35:53 GMT
ETag: "5cd618e9-264"
Accept-Ranges: bytes
```

### 2.1 Ingress地址重写

流量重定向到目标URL

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-daem
  annotations:
    #kubernetes.io/ingress.class: "nginx" 
    nginx.ingress.kubernetes.io/permanent-redirect: https://www.baidu.com   #当访问 test.nginxsvc.com会被重写到百度上
spec:
  rules:
  - host: test.nginxsvc.com
    http:
      paths:
      - backend:
          serviceName: nginx-svc
          servicePort: 80
        path: /
```

前后端分离

```yaml
  [root@k8s-master01 daem]# cat ingress.yaml 
  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    name: ingress-daem
    annotations:
      #kubernetes.io/ingress.class: "nginx" 
      #nginx.ingress.kubernetes.io/permanent-redirect: https://www.baidu.com
      nginx.ingress.kubernetes.io/rewrite-target: /    #当访问test.nginxsvc.com/foo  会把请求打到 nginx-svc此service上
  spec:
    rules:
    - host: test.nginxsvc.com
      http:
        paths:
        - backend:
            serviceName: nginx-svc
            servicePort: 80
          path: /foo



  [root@k8s-master01 daem]# cat ingress.yaml 
  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    name: ingress-daem
    annotations:
      #kubernetes.io/ingress.class: "nginx" 
      #nginx.ingress.kubernetes.io/permanent-redirect: https://www.baidu.com
      nginx.ingress.kubernetes.io/rewrite-target: /$2
  spec:
    rules:
    - host: test.nginxsvc.com
      http:
        paths:
        - backend:
            serviceName: nginx-svc
            servicePort: 80
          path: /nginxservice(/|$)(.*)
        paths:
        - backend:
            serviceName: tomcat-svc
            servicePort: 80
          path: /tomcatservice(/|$)(.*) #当访问test.nginxsvc.com:PORT/tomcatservice -> 就会被重定向到tomcat-svc  / 资源下
```

### 2.2 配置HTTPS

```yaml
[root@k8s-master01 ~]# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.cert -subj "/CN=test-tls.test.com/O=test-tls.test.com"
Generating a 2048 bit RSA private key
..............................................+++
.........................................................................................+++
writing new private key to 'tls.key'
-----

[root@k8s-master01 ~]# kubectl create secret tls ca-cert  --key tls.key --cert tls.cert 
secret/ca-cert created
[root@k8s-master01 ~]# cat tlsingress.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
annotations:
nginx.ingress.kubernetes.io/ssl-redirect: "false"
name: test-tls
spec:
rules:
- host: test-tls.test.com
http:
paths:
- backend:
serviceName: nginx-svc
servicePort: 80
path: /
tls:                     
- hosts:
- test-tls.test.com
secretName: ca-cert
```

### 2.3 黑白名单配置

```yaml
    黑白名单

    [root@k8s-master01 ingressdeploy]# cat mandatory.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: ingress-nginx
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx

    ---

    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: nginx-configuration
      namespace: ingress-nginx
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
    data:
      #############添加如下信息#################
      whitelist-source-range: 192.168.29.102 #白名单，允许某个IP或IP段的访问
      block-cidrs: 192.168.29.101 #黑名单拒绝访问
```

### 2.4 匹配请求头

```yaml
      apiVersion: networking.k8s.io/v1beta1
      kind: Ingress
      metadata:
        annotations:
          nginx.ingress.kubernetes.io/server-snippet: |
              set $agentflag 0;

              if ($http_user_agent ~* "(Mobile)" ){
                set $agentflag 1;
              }

              if ( $agentflag = 1 ) {
                return 301 https://m.example.com;
              }
```

解释：
如果你的http_user_agent == Mobile。那么就把agentflag set成1 ，然后当agentflag == 1时，就会return 到这个域名 [https://m.example.com](https://m.example.com/)

### 2.5 速率限制

```yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: ingress-nginx
      annotations:
        kubernetes.io/ingress.class: "nginx"
        nginx.ingress.kubernetes.io/limit-rate: 100K
        nginx.ingress.kubernetes.io/limit-rps: 1
        nginx.ingress.kubernetes.io/limit-rpm: 30
    spec:
      ......
```

- nginx.ingress.kubernetes.io/limit-connections

  单个IP地址允许的并发连接数。超过此限制时返回503错误

- nginx.ingress.kubernetes.io/limit-rps:

  每秒从给定IP接受的请求数。突发限制设置为该限制乘以突发乘数，默认乘数为5。当客户机超过此限制时，将返回limit req status code default:503

- nginx.ingress.kubernetes.io/limit-rpm:

  每分钟从给定IP接受的请求数。突发限制设置为该限制乘以突发乘数，默认乘数为5。当客户机超过此限制时，将返回limit req status code default:503

- nginx.ingress.kubernetes.io/limit-burst-multiplier:

  突发大小限制速率的乘数。默认的突发乘数为5，此批注覆盖默认乘数。当客户机超过此限制时，将返回limit req status code default:503。

- nginx.ingress.kubernetes.io/limit-rate-after

  初始千字节数，此后对给定连接的响应的进一步传输将受到速率限制。此功能必须在启用代理缓冲的情况下使用

- nginx.ingress.kubernetes.io/limit-rate

  每秒允许发送到给定连接的KB数。零值禁用速率限制。此功能必须在启用代理缓冲的情况下使用

- nginx.ingress.kubernetes.io/limit-whitelist

  要从速率限制中排除的客户端IP源范围。该值是一个逗号分隔的cidr列表

**以上就是Ingress常用的相关配置，所有配置均来自官方文档：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/**

# 3、部署Ingress-Nginx

该Nginx是经过改造的，而不是传统的Nginx。

Ingress-Nginx官网地址

```
https://kubernetes.github.io/ingress-nginx/
```

 

Ingress-Nginx GitHub地址

```
https://github.com/kubernetes/ingress-nginx
```

 

本次下载版本：nginx-0.30.0

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923212905808-2085269317.png)

 

镜像下载与重命名

```bash
docker pull registry.cn-beijing.aliyuncs.com/google_registry/nginx-ingress-controller:0.30.0
docker tag 89ccad40ce8e quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
docker rmi  registry.cn-beijing.aliyuncs.com/google_registry/nginx-ingress-controller:0.30.0
```

 

ingress-nginx的yaml文件修改后并启动

```yaml
# 当前目录
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
# 获取NGINX: 0.30.0
[root@k8s-master ingress]# wget https://github.com/kubernetes/ingress-nginx/archive/nginx-0.30.0.tar.gz
[root@k8s-master ingress]# tar xf nginx-0.30.0.tar.gz
# yaml文件在下载包中的位置：ingress-nginx-nginx-0.30.0/deploy/static/mandatory.yaml
[root@k8s-master ingress]# cp -a ingress-nginx-nginx-0.30.0/deploy/static/mandatory.yaml ./
[root@k8s-master ingress]#
# yaml文件配置修改
[root@k8s-master ingress]# vim mandatory.yaml
………………
apiVersion: apps/v1
kind: DaemonSet   # 从Deployment改为DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  #replicas: 1   # 注释掉
………………
      nodeSelector:
        kubernetes.io/hostname: k8s-master   # 修改处
      # 如下几行为新加行  作用【允许在master节点运行】
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
………………
          ports:
            - name: http
              containerPort: 80
              hostPort: 80    # 添加处【可在宿主机通过该端口访问Pod】
              protocol: TCP
            - name: https
              containerPort: 443
              hostPort: 443   # 添加处【可在宿主机通过该端口访问Pod】
              protocol: TCP
………………
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl apply -f mandatory.yaml
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
daemonset.apps/nginx-ingress-controller created
limitrange/ingress-nginx created
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get ds -n ingress-nginx -o wide
NAME                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                       AGE     CONTAINERS                 IMAGES                                                                  SELECTOR
nginx-ingress-controller   1         1         1       1            1           kubernetes.io/hostname=k8s-master   9m47s   nginx-ingress-controller   quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0   app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get pod -n ingress-nginx -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
nginx-ingress-controller-rrbh9   1/1     Running   0          9m55s   10.244.0.46   k8s-master   <none>           <none>
```

## deply_service1的yaml信息

yaml文件

```yaml
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# cat deply_service1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy1
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
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip1
  namespace: default
spec:
  type: ClusterIP  # 默认类型
  selector:
    app: myapp
    release: v1
  ports:
  - name: http
    port: 80
    targetPort: 80
```

启动Deployment和Service

```bash
[root@k8s-master ingress]# kubectl apply -f deply_service1.yaml 
deployment.apps/myapp-deploy1 created
service/myapp-clusterip1 created
```

 

查看Deploy状态和信息

```bash
[root@k8s-master ingress]# kubectl get deploy -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1   3/3     3            3           28s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,release=v1
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get rs -o wide
NAME                       DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1-5695bb5658   3         3         3       30s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5695bb5658,release=v1
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get pod -o wide --show-labels
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy1-5695bb5658-n6548   1/1     Running   0          36s   10.244.2.144   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy1-5695bb5658-rqcpb   1/1     Running   0          36s   10.244.2.143   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
myapp-deploy1-5695bb5658-vv6gm   1/1     Running   0          36s   10.244.3.200   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=5695bb5658,release=v1
```

curl访问pod

```bash
[root@k8s-master ingress]# curl 10.244.2.144
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.244.2.144/hostname.html
myapp-deploy1-5695bb5658-n6548
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.244.2.143/hostname.html
myapp-deploy1-5695bb5658-rqcpb
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.244.3.200/hostname.html
myapp-deploy1-5695bb5658-vv6gm
```

 

查看Service状态和信息

```bash
[root@k8s-master ingress]# kubectl get svc -o wide
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   19d     <none>
myapp-clusterip1   ClusterIP   10.104.146.14   <none>        80/TCP    5m38s   app=myapp,release=v1
```

 

curl访问svc

```bash
[root@k8s-master ingress]# curl 10.104.146.14
Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-n6548
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-vv6gm
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.104.146.14/hostname.html
myapp-deploy1-5695bb5658-rqcpb
```

## deply_service2的yaml信息

yaml文件

```yaml
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# cat deply_service2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy2
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      release: v2
  template:
    metadata:
      labels:
        app: myapp
        release: v2
        env: test
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-clusterip2
  namespace: default
spec:
  type: ClusterIP  # 默认类型
  selector:
    app: myapp
    release: v2
  ports:
  - name: http
    port: 80
    targetPort: 80
```

启动Deployment和Service

```bash
[root@k8s-master ingress]# kubectl apply -f deply_service2.yaml 
deployment.apps/myapp-deploy2 created
service/myapp-clusterip2 created
```

 

查看Deploy状态和信息

```yaml
[root@k8s-master ingress]# kubectl get deploy myapp-deploy2 -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy2   3/3     3            3           9s    myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2   app=myapp,release=v2
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get rs  -o wide
NAME                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                                                      SELECTOR
myapp-deploy1-5695bb5658   3         3         3       7m23s   myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1   app=myapp,pod-template-hash=5695bb5658,release=v1   # 之前创建的
myapp-deploy2-54f48f879b   3         3         3       15s     myapp        registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2   app=myapp,pod-template-hash=54f48f879b,release=v2   # 当前deploy创建的
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get pod -o wide --show-labels -l "release=v2"
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES   LABELS
myapp-deploy2-54f48f879b-7pxwp   1/1     Running   0          25s   10.244.3.201   k8s-node01   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
myapp-deploy2-54f48f879b-lqlh2   1/1     Running   0          25s   10.244.2.146   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
myapp-deploy2-54f48f879b-pfvnn   1/1     Running   0          25s   10.244.2.145   k8s-node02   <none>           <none>            app=myapp,env=test,pod-template-hash=54f48f879b,release=v2
```

查看Service状态和信息

```bash
[root@k8s-master ingress]# kubectl get svc -o wide  
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP   19d    <none>
myapp-clusterip1   ClusterIP   10.104.146.14   <none>        80/TCP    8m9s   app=myapp,release=v1
myapp-clusterip2   ClusterIP   10.110.181.62   <none>        80/TCP    61s    app=myapp,release=v2
```

 

curl访问svc

```yaml
[root@k8s-master ingress]# curl 10.110.181.62
Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-lqlh2
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-7pxwp
[root@k8s-master ingress]#
[root@k8s-master ingress]# curl 10.110.181.62/hostname.html
myapp-deploy2-54f48f879b-pfvnn
```

## Ingress HTTP代理访问

yaml文件【由于自建的service在默认default名称空间，因此这里也是default名称空间】

```yaml
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# cat ingress-http.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-http
  namespace: default
spec:
  rules:
    - host: www.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip1
            servicePort: 80
    - host: blog.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip2
            servicePort: 80
```

启动ingress http并查看状态

```yaml
[root@k8s-master ingress]# kubectl apply -f ingress-http.yaml
ingress.networking.k8s.io/nginx-http created
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get ingress -o wide
NAME         HOSTS                                  ADDRESS   PORTS   AGE
nginx-http   www.zhangtest.com,blog.zhangtest.com             80      9s
```

查看nginx配置文件

```yaml
[root@k8s-master ~]# kubectl get pod -A | grep 'ingre'
ingress-nginx          nginx-ingress-controller-rrbh9               1/1     Running   0          27m
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl exec -it -n ingress-nginx nginx-ingress-controller-rrbh9 bash
bash-5.0$ cat /etc/nginx/nginx.conf
…………
##### 可见server www.zhangtest.com 和 server blog.zhangtest.com的配置
```

### 浏览器访问

hosts文件修改，添加如下信息

```
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：
# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com
```

 

浏览器访问[www.zhangtest.com](http://www.zhangtest.com/)

```
http://www.zhangtest.com/
http://www.zhangtest.com/hostname.html
```

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923213402277-142402959.png)

 

浏览器访问blog.zhangtest.com

```
http://blog.zhangtest.com/
http://blog.zhangtest.com/hostname.html
```

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923213430406-1252674577.png)

 

当然：除了用浏览器访问外，也可以在Linux使用curl访问。前提是修改/etc/hosts文件，对上面的两个域名进行解析。

## Ingress HTTPS代理访问

### SSL证书创建

```bash
[root@k8s-master cert]# pwd
/root/k8s_practice/ingress/cert
[root@k8s-master cert]# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BeiJing/O=BTC/OU=MOST/CN=zhang/emailAddress=ca@test.com"
Generating a 2048 bit RSA private key
......................................................+++
........................+++
writing new private key to 'tls.key'
-----
[root@k8s-master cert]# kubectl create secret tls tls-secret --key tls.key --cert tls.crt
secret/tls-secret created
```

### 创建ingress https

yaml文件【由于自建的service在默认default名称空间，因此这里也是default名称空间】

```yaml
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# cat ingress-https.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-https
  namespace: default
spec:
  tls:
    - hosts:
      - www.zhangtest.com
      - blog.zhangtest.com
      secretName: tls-secret
  rules:
    - host: www.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip1
            servicePort: 80
    - host: blog.zhangtest.com
      http:
        paths:
        - path: /
          backend:
            serviceName: myapp-clusterip2
            servicePort: 80
```

启动ingress https并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f ingress-https.yaml
ingress.networking.k8s.io/nginx-https created
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get ingress -o wide
NAME          HOSTS                                  ADDRESS   PORTS     AGE
nginx-https   www.zhangtest.com,blog.zhangtest.com             80, 443   8s
```

### 浏览器访问

hosts文件修改，添加如下信息

```
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：
# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com
```

 

浏览器访问[www.zhangtest.com](http://www.zhangtest.com/)

```
https://www.zhangtest.com/
https://www.zhangtest.com/hostname.html
```

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923213604360-1004242733.png)

 

浏览器访问blog.zhangtest.com

```
https://blog.zhangtest.com/
https://blog.zhangtest.com/hostname.html
```

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923213630265-647749251.png)

# Ingress-Nginx实现BasicAuth认证

官网地址：

```
https://kubernetes.github.io/ingress-nginx/examples/auth/basic/
```

 

准备工作

```bash
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# yum install -y httpd
[root@k8s-master ingress]# htpasswd -c auth foo
New password: #输入密码
Re-type new password: #重复输入的密码
Adding password for user foo   ##### 此时会生成一个 auth文件
[root@k8s-master ingress]# kubectl create secret generic basic-auth --from-file=auth
secret/basic-auth created
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get secret basic-auth -o yaml
apiVersion: v1
data:
  auth: Zm9vOiRhcHIxJFpaSUJUMDZOJDVNZ3hxdkpFNWVRTi9NdnZCcVpHaC4K
kind: Secret
metadata:
  creationTimestamp: "2020-08-17T09:42:04Z"
  name: basic-auth
  namespace: default
  resourceVersion: "775573"
  selfLink: /api/v1/namespaces/default/secrets/basic-auth
  uid: eef0853b-a52b-4684-922a-817e4cd9e9ca
type: Opaque
```

ingress yaml文件

```yaml
[root@k8s-master ingress]# pwd
/root/k8s_practice/ingress
[root@k8s-master ingress]# cat nginx_basicauth.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-with-auth
  annotations:
    # type of authentication
    nginx.ingress.kubernetes.io/auth-type: basic
    # name of the secret that contains the user/password definitions
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    # message to display with an appropriate context why the authentication is required
    nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'
spec:
  rules:
  - host: auth.zhangtest.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp-clusterip1
          servicePort: 80
```

启动ingress并查看状态

```bash
[root@k8s-master ingress]# kubectl apply -f nginx_rewrite.yaml
ingress.networking.k8s.io/rewrite created
[root@k8s-master ingress]#
[root@k8s-master ingress]# kubectl get ingress -o wide
NAME                HOSTS                   ADDRESS          PORTS     AGE
rewrite             rewrite.zhangtest.com                    80        13s
```

## 浏览器访问

hosts文件修改，添加如下信息

```
文件位置：C:\WINDOWS\System32\drivers\etc\hosts
添加信息如下：
# K8S ingress学习
10.0.0.110  www.zhangtest.com  blog.zhangtest.com auth.zhangtest.com  rewrite.zhangtest.com
```

 

浏览器访问rewrite.zhangtest.com

```
http://rewrite.zhangtest.com/
```

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923214020402-954120757.png)

之后，可见重定向到了[https://www.baidu.com 百度页面](https://www.baidu.xn--com-vy2fl66hqi5b8tb/)