## 1. Service

### 1.1. Service 介绍

Pod控制器实现了对一组pod的版本、副本数和配置的控制，但是Pod本身被创建和消除会导致IP地址不断变化，Service为一组pod提供了一个固定的VIP，所有通过该VIP地址的请求都会被转发到后端的pod上。Service是工作在四层网络之上的，支持三种工作模式。

在 Kubernetes v1.0 版本，代理完全在 userspace。在 Kubernetes v1.1 版本，新增了 iptables 代理，但并不是默认的运行模式。 从 Kubernetes v1.2 起，默认就是 iptables 代理。在Kubernetes v1.8.0-beta.0中，添加了ipvs代理。

#### 1.1.1.userspace模式

客户端请求到内核空间后，由iptables规则将其转发到用户空间的 kube-proxy，kube-proxy 将其调度到各个node上的目标Pod。这种模式需要多次切换用户空间和内核空间，性能很差，生产环境不再使用。

![image](https://cdn.nlark.com/yuque/0/2020/jpeg/378176/1579700610515-71593fa8-5f55-487b-8451-8c20dc242afa.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

#### 1.1.2. iptables模式

iptables模式中，客户端的请求进入内核态后，直接由iptables将请求转发到各个pod上，这种模式中随着iptables规则的增加，性能越来越差，而且iptables规则查看和管理难度很大，生产环境中尽量少用。

![image](https://cdn.nlark.com/yuque/0/2020/jpeg/378176/1579700849322-90dffe35-45ab-4167-9c30-ff70bb581a2b.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

#### 1.1.3. ipvs模式

ipvs模式中，客户端的请求进入内核态后，直接由ipvs规则将请求转发到各个pod上，这种模式相当于内嵌一套lvs集群，性能比iptables强大，更重要的是支持lvs中调度算法，生产环境中ipvs模式有先考虑使用。

![image](https://cdn.nlark.com/yuque/0/2020/png/378176/1579701028672-4566118f-c449-4c8e-964c-982f5d180cdd.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_20%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

### 1.2. 模板

```
apiVersion: v1
kind: Service
metadata
    name        <string>            # 在一个名称空间不能重复
    namespace   <string>            # 指定名称空间，默认defalut
    labels      <map[string]string> # 标签
    annotations <map[string]string> # 注释
spec
    selector                    <map[string]string> # 仅支持key/value方式定义选择器
    type                        <string>            # service暴露方式
        # ClusterIP: 通过集群的VIP方式，这种方式下仅能在集群内部访问，默认值
        # NodePort: 通过暴露node端口来将流量路由到clusterIP，从而实现从集群外部访问service功能，且需要宿主机开启监听端口，容易出现端口冲突
        # LoadBalancer: 使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到NodePort服务和ClusterIP服务
        # ExternalName: 通过返回 CNAME 和它的值，可以将服务映射到 externalName 字段的内容
    clusterIP                   <string>            # 定义集群VIP地址
        # 当指定确定IP地址时，会以该IP地址为准
        # 当指定为None时，为 headless service
    ports                       <[]Object>          # 指定端口映射
        name                    <string>            # 端口名称
        port                    <integer> -required-# 指定service端口
        targetPort              <string>            # 指定pod端口
        nodePort                <integer>           # 指定node端口，仅在NodePort类型使用.3000-29999之间
    sessionAffinity             <string>            # 是否启用粘性会话，
```

### 1.3. 案例

#### 1.3.1. 不指定VIP的service

```bash
[root@hdss7-200 service]# vim /data/k8s-yaml/base_resource/service/slb-s1.yaml
apiVersion: v1
kind: Service
metadata:
  name: slb-s1
  namespace: app
spec:
  selector:
    app: nginx
    release: stable
    partition: website
    tier: slb
  ports:
  - name: http
    port: 80
    targetPort: 80
```



```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/service/slb-s1.yaml
[root@hdss7-21 ~]# kubectl get svc -n app -o wide
NAME     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
slb-s1   ClusterIP   192.168.219.15   <none>        80/TCP    11h   app=nginx,partition=website,release=stable,tier=slb
[root@hdss7-21 ~]# ipvsadm -ln # NAT模式下采用nq算法的负载均衡策略(部署kube-proxy是指定的)
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn      
TCP  192.168.219.15:80 nq
  -> 172.7.21.7:80                Masq    1      0          0         
  -> 172.7.21.8:80                Masq    1      0          0         
  -> 172.7.21.9:80                Masq    1      0          0         
  -> 172.7.22.7:80                Masq    1      0          0         
  -> 172.7.22.8:80                Masq    1      0          0         
[root@hdss7-21 ~]# kubectl describe svc slb-s1 -n app # service通过endpoint对象管控pod
Name:              slb-s1
Namespace:         app
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"slb-s1","namespace":"app"},"spec":{"ports":[{"name":"http","port"...
Selector:          app=nginx,partition=website,release=stable,tier=slb
Type:              ClusterIP
IP:                192.168.219.15
Port:              http  80/TCP
TargetPort:        80/TCP
Endpoints:         172.7.21.7:80,172.7.21.8:80,172.7.21.9:80 + 2 more...
Session Affinity:  None
Events:            <none>
```



```bash
# 测试域名解析！
[root@pod-demo /]# curl -s http://slb-s1.app.svc.cluster.local/info  # 全域名
2020-01-23T03:17:16+00:00|172.7.22.8|nginx:v1.14
[root@pod-demo /]# curl -s http://slb-s1/info  # 短域名
2020-01-23T03:17:53+00:00|172.7.22.8|nginx:v1.14
```



```bash
# 测试负载均衡！因为是nq算法，所有用单进程curl看不出效果，采用多进程同时访问进行测试
[root@hdss7-21 ~]# vim /tmp/test.sh 
test() {
    for i in {1..1000}
    do
        curl -s http://192.168.219.15/info ; sleep 0.005
    done
}

for j in {a..k}
do
    test > /tmp/res.$j 2>&1 &
done

wait
awk -F'|' '{ret[$2]+=1}END{for(i in ret) print i,ret[i]}' /tmp/res.*|sort -nk 2
rm -f /tmp/res.*
[root@hdss7-21 ~]# bash /tmp/test.sh
172.7.21.7 87
172.7.21.8 624
172.7.21.9 2049
172.7.22.7 3388
172.7.22.8 4852
```

#### 1.3.2. 指定VIP的service

```bash
# 大部分情况下，不需要指定vip，因为内部pod通过域名的方式访问service
[root@hdss7-200 service]# vim /data/k8s-yaml/base_resource/service/slb-s2.yaml 
apiVersion: v1
kind: Service
metadata:
  name: slb-s2
  namespace: app
spec:
  selector:
    app: nginx
    release: stable
    partition: website
    tier: slb
  clusterIP: 192.168.10.100
  ports:
  - name: http
    port: 80
    targetPort: 80
```



```bash
[root@hdss7-21 ~]# kubectl get svc slb-s2 -n app
NAME     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
slb-s2   ClusterIP   192.168.10.100   <none>        80/TCP    14s
[root@hdss7-21 ~]# kubectl describe svc slb-s2 -n app
Name:              slb-s2
Namespace:         app
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"slb-s2","namespace":"app"},"spec":{"clusterIP":"192.168.10.100","...
Selector:          app=nginx,partition=website,release=stable,tier=slb
Type:              ClusterIP
IP:                192.168.10.100
Port:              http  80/TCP
TargetPort:        80/TCP
Endpoints:         172.7.21.7:80,172.7.21.8:80,172.7.21.9:80 + 2 more...
Session Affinity:  None
Events:            <none>
```

#### 1.3.3. 使用NodePort的service

```bash
[root@hdss7-200 service]# vim /data/k8s-yaml/base_resource/service/slb-s3.yaml
apiVersion: v1
kind: Service
metadata:
  name: slb-s3
  namespace: app
spec:
  selector:
    app: nginx
    release: stable
    partition: website
    tier: slb
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 3080
```



```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/service/slb-s3.yaml
[root@hdss7-21 ~]# kubectl get svc slb-s3 -n app  
NAME     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)       AGE
slb-s3   NodePort   192.168.1.125   <none>        80:3080/TCP   39s
[root@hdss7-21 ~]# netstat -lntp|grep 3080  # 在宿主机上开启3080端口，实现了集群外访问,注意避免端口冲突
tcp6       0      0 :::3080                 :::*                    LISTEN      1023/kube-proxy

[root@hdss7-200 ~]# curl -s 10.4.7.21:3080/info  # 集群外访问测试
2020-01-23T03:33:23+00:00|172.7.22.8|nginx:v1.14
```

#### 1.3.4. Headless service

```bash
# 直接将域名解析到pod上
[root@hdss7-200 service]# vim /data/k8s-yaml/base_resource/service/slb-s4.yaml 
apiVersion: v1
kind: Service
metadata:
  name: slb-s4
  namespace: app
spec:
  selector:
    app: nginx
    release: stable
    partition: website
    tier: slb
  clusterIP: None
  ports:
  - name: http
    port: 80
    targetPort: 80
```



```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/service/slb-s4.yaml
[root@hdss7-21 ~]# kubectl get svc slb-s4 -n app
NAME     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
slb-s4   ClusterIP   None         <none>        80/TCP    16s
[root@hdss7-21 ~]# dig -t A slb-s4.app.svc.cluster.local @192.168.0.2 +short # DNS解析
172.7.21.9
172.7.22.8
172.7.22.7
172.7.21.8
172.7.21.7
[root@hdss7-21 ~]# kubectl exec pod-demo -n app -- /bin/bash -c "curl -s http://slb-s4/info"
2020-01-23T05:34:59+00:00|172.7.21.9|nginx:v1.14
[root@hdss7-21 ~]# kubectl exec pod-demo -n app -- /bin/bash -c "curl -s http://slb-s4/info"
2020-01-23T05:35:00+00:00|172.7.21.7|nginx:v1.14
```



## 2. Ingress

### 2.1. Ingress/IngressController

Service虽说是为集群内部pod访问一组微服务提供固定接入点，其实NodePort、LoadBalancer方式可以通过在Node上配置端口映射实现外部集群访问内部service。这种方式存在以下问题：

- 仅支持L4调度，不支持域名和Loctation方式分发流量
- 对于SSL会话卸载，需要在pod内部完成，比较复杂
- 当业务数量增加时，容易出现端口冲突



Ingress 是 Kubernetes 的一种 API 对象，是实现HTTP/HTTPS、基于路径和域名进行流量转发的一组规则，实现了将集群外部七层流量转发到集群内部。IngressController是将Ingress规则实现的一种负载均衡器Pod，如Nginx、Traefik、HAProxy等。



IngressController有两种部署方式：一种是采用DaemonSet方式部署，一种是Deployment方式部署。为了降低复杂度，一般都采用DaemonSet方式部署。如下图所示：

- ingress-controller 采用DaemonSet方式部署在各个node节点之上
- 集群外部部署一个七层负载均衡，如Nginx。Nginx负责做流量转发和SSL会话卸载
- ingress-controller根据ingress规则将流量调度到指定的service上
- 各service再将流量转发到各个pod上

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1579786389744-e22dc04b-9f1a-4e15-820c-d26f893d2d70.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

### 2.2. 安装IngressController

参考： https://www.yuque.com/duduniao/ww8pmw/tr3hch#x7V9z

### 2.3. 模板

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata
    name        <string>            # 在一个名称空间不能重复
    namespace   <string>            # 指定名称空间，默认defalut
    labels      <map[string]string> # 标签
    annotations <map[string]string> # 注释
spec
    backend                     <Object>                # 后端pod对应的服务，仅在集群中仅单个service暴露时使用
        serviceName             <string> -required-     # 服务名
        servicePort             <string> -required-     # 服务接入端口
    rules                       <[]Object>              # 流量转发规则
        host                    <string>                # 基于server name进行转发
        http                    <Object>                # 基于path路径转发
            paths               <[]Object> -required-   # 指定转发的path路径和后端service
                path            <string>                # 指定path        
                backend         <Object> -required-     # 后端pod对应的服务
                    serviceName <string> -required-     # 服务名
                    servicePort <string> -required-     # 服务接入端口
```

### 2.4. 案例

#### 2.4.1. http请求

在内网中，采用http方式通信是主流，内网中客户端访问k8s集群内部，可采用http协议，降低性能损失。通过公网访问k8s内部集群服务时，一律采用https协议，并在前端负载均衡器中卸载SSL会话，这样就避免在每个node上配置证书。

```bash
[root@hdss7-200 ingress]# vim /data/k8s-yaml/base_resource/ingress/api-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ops-api-ingress
  namespace: app
  labels:
    tier: ops
    role: api
spec:
  rules:
  - host: monitor.api.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: slb-s1
          servicePort: 80
```



```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/base_resource/ingress/api-ingress.yaml
[root@hdss7-21 ~]# kubectl get ingress -n app
NAME              HOSTS                ADDRESS   PORTS   AGE
ops-api-ingress   monitor.api.od.com             80      9s
[root@hdss7-21 ~]# kubectl describe ingress ops-api-ingress -n app
Name:             ops-api-ingress
Namespace:        app
Address:          
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                Path  Backends
  ----                ----  --------
  monitor.api.od.com  
                      /   slb-s1:80 (172.7.21.7:80,172.7.21.8:80,172.7.21.9:80 + 2 more...)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{},"labels":{"role":"api","tier":"ops"},"name":"ops-api-ingress","namespace":"app"},"spec":{"rules":[{"host":"monitor.api.od.com","http":{"paths":[{"backend":{"serviceName":"slb-s1","servicePort":80},"path":"/"}]}}]}}

Events:  <none>
```



```bash
# 配置集群外部DNS服务器，解析monitor.api.od.com域名
[root@hdss7-11 ~]# vim /var/named/od.com.zone 
$ORIGIN od.com.
$TTL 600        ; 10 minutes
@               IN SOA  dns.od.com. dnsadmin.od.com. (
                                2020011304 ; serial
                                10800      ; refresh (3 hours)
                                900        ; retry (15 minutes)
                                604800     ; expire (1 week)
                                86400      ; minimum (1 day)
                                )
                                NS   dns.od.com.
$TTL 60 ; 1 minute
dns                A    10.4.7.11
harbor             A    10.4.7.200
k8s-yaml           A    10.4.7.200
traefik            A    10.4.7.10
dashboard          A    10.4.7.10
monitor.api        A    10.4.7.10
[root@hdss7-11 ~]# systemctl restart named
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1579790190633-0682f8db-f559-4080-91e8-9b95a9d3342a.png)

#### 2.4.2. https请求

参考dashboard安装部署: https://www.yuque.com/duduniao/ww8pmw/tr3hch#B15Nw