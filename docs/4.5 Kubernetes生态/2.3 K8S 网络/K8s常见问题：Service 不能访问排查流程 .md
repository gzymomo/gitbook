在学习Kubernetes过程中，经常会遇到Service无法访问，这篇文章总结了可能导致的情况，希望能帮助你找到问题所在。



为了完成本次演练，先运行部署一个应用：

```
[root@k8s-master ~]# kubectl create deployment web --image=nginx --replicas=3
[root@k8s-master ~]# kubectl expose deployment web --port=80 --type=NodePort
```

确保Pod运行：

```
[root@k8s-master ~]# kubectl get pods,svc
NAME READY STATUS RESTARTS AGE
pod/web-96d5df5c8-6rtxj 1/1     Running 0          110s
pod/web-96d5df5c8-dfbkv 1/1     Running 0          110s
pod/web-96d5df5c8-jghbs 1/1     Running 0          110s

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/kubernetes ClusterIP 10.96.0.1      <none> 443/TCP 3d5h
service/web NodePort 10.104.0.64    <none> 80:32035/TCP 22s
```

## **问题1：无法通过 Service 名称访问**

如果你是访问的Service名称，需要确保CoreDNS服务已经部署：

```
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME              READY     STATUS RESTARTS AGE
coredns-7f89b7bc75-745s4 1/1     Running 0          3d5h
coredns-7f89b7bc75-fgdfm 1/1     Running 0          3d5h
```

确认CoreDNS已部署，如果状态不是Running，请检查容器日志进一步查找问题。

接下来创建一个临时Pod测试下DNS解析是否正常：



```
root@k8s-master ~]# kubectl run -it --rm --image=busybox:1.28.4 -- sh


If you don't see a command prompt, try pressing enter.

/ # nslookup web
Server: 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name: web
Address 1: 10.104.0.64 web.default.svc.cluster.local
```



如果解析失败，可以尝试限定命名空间：

```
/ # nslookup web.default
Server: 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name: web.default
Address 1: 10.104.0.64 web.default.svc.cluster.local
```



如果解析成功，需要调整应用使用跨命名空间的名称访问Service。

如果仍然解析失败，尝试使用完全限定的名称：

```
/ # nslookup web.default.svc.cluster.local
Server: 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name: web.default.svc.cluster.local
Address 1: 10.104.0.64 web.default.svc.cluster.local
```



说明：其中“default”表示正在操作的命名空间，“svc”表示是一个Service，“cluster.local”是集群域。

再集群中的Node尝试指定DNS IP（你的可能不同，可以通过kubectl get svc -n kube-system查看）解析下：

```
[root@k8s-node1 ~]# nslookup web.default.svc.cluster.local 10.96.0.10
Server: 10.96.0.10
Address: 10.96.0.10#53

Name: web.default.svc.cluster.local
Address: 10.104.0.64
```

如果你能使用完全限定的名称查找，但不能使用相对名称，则需要检查 `/etc/resolv.conf` 文件是否正确：

```
/ # cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

说明：

- nameserver：行必须指定CoreDNS Service，它通过在kubelet设置 --cluster-dns 参加自动配置。
- search ：行必须包含一个适当的后缀，以便查找 Service 名称。在本例中，它在本地 Namespace（default.svc.cluster.local）、所有 Namespace 中的 Service（svc.cluster.local）以及集群（cluster.local）中查找服务。
- options ：行必须设置足够高的 ndots，以便 DNS 客户端库优先搜索路径。在默认情况下，Kubernetes 将这个值设置为 5。

### **问题2：无法通过 Service IP访问**

假设可以通过Service名称访问（CoreDNS正常工作），那么接下来要测试的 Service 是否工作正常。从集群中的一个节点，访问 Service 的 IP：

```
[root@k8s-master ~]# curl -I 10.104.0.64:80
HTTP/1.1 200 OK
```

如果 Service 是正常的，你应该看到正确的状态码。如果没有，有很多可能出错的地方，请继续。

## **思路1：Service 端口配置是否正确？**

检查 Service 配置和使用的端口是否正确：

```
[root@k8s-master ~]# kubectl get svc web -o yaml
...
spec:
  clusterIP: 10.104.0.64
  clusterIPs:
  - 10.104.0.64
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32035
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
...
```



- spec.ports[]：访问ClusterIP带的端口
- targetPort ：目标端口，是容器中服务提供的端口
- spec.nodePort ：集群外部访问端口，http://NodeIP:32035

## **思路2：Service 是否正确关联到Pod？**

检查 Service 关联的 Pod 是否正确：

```
[root@k8s-master ~]# kubectl get pods -l app=web
NAME READY STATUS RESTARTS AGE
web-96d5df5c8-6rtxj 1/1     Running 0          38m
web-96d5df5c8-dfbkv 1/1     Running 0          38m
web-96d5df5c8-jghbs 1/1     Running 0          38m
```

-l app=hostnames 参数是一个标签选择器。

在 Kubernetes 系统中有一个控制循环，它评估每个 Service 的选择器，并将结果保存到 Endpoints 对象中。

```
[root@k8s-master ~]# kubectl get endpoints web
NAME   ENDPOINTS                                             AGE
web    10.244.169.135:80,10.244.169.136:80,10.244.36.73:80   38m
```

结果所示， endpoints  控制器已经为 Service 找到了 Pods。但并不说明关联的Pod就是正确的，还需要进一步确认Service 的 spec.selector 字段是否与Deployment中的 metadata.labels 字段值一致。

```
[root@k8s-master ~]# kubectl get svc web -o yaml
...
  selector:
    app: web
```

```
[root@k8s-master ~]# kubectl get deployment web -o yaml
...
selector:
    matchLabels:
      app: web
```

## **思路3：Pod 是否正常工作？**

检查Pod是否正常工作，绕过Service，直接访问Pod IP：

```
[root@k8s-master ~]# kubectl get pods -o wide
NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
web-96d5df5c8-6rtxj 1/1     Running 0          50m   10.244.169.135   k8s-node2 <none>           <none>
web-96d5df5c8-dfbkv 1/1     Running 0          50m   10.244.36.73     k8s-node1 <none>           <none>
web-96d5df5c8-jghbs 1/1     Running 0          50m   10.244.169.136   k8s-node2 <none>
```

```
[root@k8s-master ~]# curl -I 10.244.169.135:80
HTTP/1.1 200 OK
```

> **注：** 使用的是 `Pod` 端口（80），而不是 `Service` 端口（80

如果不能正常响应，说明容器中服务有问题， 这个时候可以用kubectl logs查看日志或者使用 kubectl exec 直接进入 Pod检查服务。

除了本身服务问题外，还有可能是CNI网络组件部署问题，现象是：curl访问10次，可能只有两三次能访问，能访问时候正好Pod是在当前节点，这并没有走跨主机网络。

如果是这种现象，检查网络组件运行状态和容器日志：



```
[root@k8s-master ~]# kubectl get pods -n kube-system
NAME READY STATUS RESTARTS AGE
calico-kube-controllers-97769f7c7-2cvnw 1/1     Running 0          3d6h
calico-node-c7jjd 1/1     Running 0          3d6h
calico-node-cc9jt 1/1     Running 0          3d6h
calico-node-xqhwh 1/1     Running 0          3d6h
```

## **思路4：ku****be-proxy 组件正常工作吗？**

如果到了这里，你的 Service 正在运行，也有 Endpoints， Pod 也正在服务。

接下来就该检查负责 Service 的组件kube-proxy是否正常工作。

确认 kube-proxy 运行状态：

```
[root@k8s-node1 ~]# ps -ef |grep kube-proxy
root 5842   2246  0 04:56 pts/1    00:00:00 grep --color=auto kube-proxy
root 46540  46510  0 Mar12 ? 00:00:52 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --hostname-override=k8s-node1
```

如果有进程存在，下一步确认它有没有工作中有错误，比如连接主节点失败。

要做到这一点，必须查看日志。查看日志方式取决于K8s部署方式，如果是kubeadm部署：

```
kubectl logs kube-proxy-mplxk -n kube-system
```

如果是二进制方式部署：

```
journalctl -u kube-proxy
```

### **思路5：kube-proxy 是否在写 iptables 规则？**

kube-proxy 的主要负载 Services 的 负载均衡 规则生成，默认情况下使用iptables实现，检查一下这些规则是否已经被写好了。

```
[root@k8s-node1 ~]# iptables-save |grep web
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/web" -m tcp --dport 32035 -j KUBE-MARK-MASQ
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/web" -m tcp --dport 32035 -j KUBE-SVC-LOLE4ISW44XBNF3G
-A KUBE-SEP-AHKVJ4M3GZYQHEVU -s 10.244.36.73/32 -m comment --comment "default/web" -j KUBE-MARK-MASQ
-A KUBE-SEP-AHKVJ4M3GZYQHEVU -p tcp -m comment --comment "default/web" -m tcp -j DNAT --to-destination 10.244.36.73:80
-A KUBE-SEP-OUZ3ZV3ZECH37O4J -s 10.244.169.136/32 -m comment --comment "default/web" -j KUBE-MARK-MASQ
-A KUBE-SEP-OUZ3ZV3ZECH37O4J -p tcp -m comment --comment "default/web" -m tcp -j DNAT --to-destination 10.244.169.136:80
-A KUBE-SEP-PO5YJF2LZTBQLFX5 -s 10.244.169.135/32 -m comment --comment "default/web" -j KUBE-MARK-MASQ
-A KUBE-SEP-PO5YJF2LZTBQLFX5 -p tcp -m comment --comment "default/web" -m tcp -j DNAT --to-destination 10.244.169.135:80
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.104.0.64/32 -p tcp -m comment --comment "default/web cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.104.0.64/32 -p tcp -m comment --comment "default/web cluster IP" -m tcp --dport 80 -j KUBE-SVC-LOLE4ISW44XBNF3G
-A KUBE-SVC-LOLE4ISW44XBNF3G -m comment --comment "default/web" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-PO5YJF2LZTBQLFX5
-A KUBE-SVC-LOLE4ISW44XBNF3G -m comment --comment "default/web" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-OUZ3ZV3ZECH37O4J
-A KUBE-SVC-LOLE4ISW44XBNF3G -m comment --comment "default/web" -j KUBE-SEP-AHKVJ4M3GZYQHEVU
```

#### 如果你已经讲代理模式改为IPVS了，可以通过这种方式查看：

```
[root@k8s-node1 ~]# ipvsadm -ln
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port Forward Weight ActiveConn InActConn
...
TCP 10.104.0.64:80 rr
  -> 10.244.169.135:80 Masq 1 0 0
  -> 10.244.36.73:80 Masq 1 0 0
  -> 10.244.169.136:80 Masq 1 0 0...
```

正常会得到上面结果，如果没有对应规则，说明kube-proxy组件没工作或者与当前操作系统不兼容导致生成规则失败。

**附：Service工作流程图**

![图片](https://mmbiz.qpic.cn/mmbiz_png/d5patQGz8KfsHwAu2s5L8Nr1QpFzvKH3KKAqaCNr1KEEgHIhUwmAFTlMHWt5LEIVoEZAbEBJdsru1FFxuPBaDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)