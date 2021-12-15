- 博客园：[容器编排系统之Kubectl工具的基础使用](https://www.cnblogs.com/qiuhom-1874/p/14130540.html)



# Kubernetes集群命令行工具kubectl

​	kubectl是Kubernetes集群的命令行工具，通过kubectl能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。

​	kubectl命令的语法：

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

- command：指定要对资源执行的操作，例如create、get、describe和delete。
- TYPE：指定资源类型，资源类型是大小写敏感的，开发者能够以单数、复数和缩略的形式。例如：

```bash
kubectl get pod pod1
kubectl get pods pod1
kubectl get po pod1
```

- NAME：指定资源的名称，名称也大小写敏感的。如果省略名称，则会显示所有的资源。例如：

```bash
kubectl get pods
```

- flags：指定可选的参数。例如，可用-s 或者-server参数指定Kubernetes API server的地址和端口，-n指定名称空间；等等。。

<font color='red'>注意：你从命令行指定的flags将覆盖默认值和任何相应的环境变量。优先级最高。</font>



## 1. 查看相关信息

帮助命令：

```bash
kubectl --help
```

具体查看某个操作：

```bash
kubectl get --help
```

```bash
# 获取节点和服务版本信息
kubectl get nodes
# 获取节点和服务版本信息，并查看附加信息
kubectl get nodes -o wide

# 获取pod信息，默认是default名称空间
kubectl get pod
# 获取pod信息，默认是default名称空间，并查看附加信息【如：pod的IP及在哪个节点运行】
kubectl get pod -o wide
# 获取指定名称空间的pod
kubectl get pod -n kube-system
# 获取指定名称空间中的指定pod
kubectl get pod -n kube-system podName
# 获取所有名称空间的pod
kubectl get pod -A
# 查看pod的详细信息，以yaml格式或json格式显示
kubectl get pods -o yaml
kubectl get pods -o json

# 查看pod的标签信息
kubectl get pod -A --show-labels
# 根据Selector（label query）来查询pod
kubectl get pod -A --selector="k8s-app=kube-dns"

# 查看运行pod的环境变量
kubectl exec podName env
# 查看指定pod的日志
kubectl logs -f --tail 500 -n kube-system kube-apiserver-k8s-master

# 查看所有名称空间的service信息
kubectl get svc -A
# 查看指定名称空间的service信息
kubectl get svc -n kube-system

# 查看componentstatuses信息
kubectl get cs
# 查看所有configmaps信息
kubectl get cm -A
# 查看所有serviceaccounts信息
kubectl get sa -A
# 查看所有daemonsets信息
kubectl get ds -A
# 查看所有deployments信息
kubectl get deploy -A
# 查看所有replicasets信息
kubectl get rs -A
# 查看所有statefulsets信息
kubectl get sts -A
# 查看所有jobs信息
kubectl get jobs -A
# 查看所有ingresses信息
kubectl get ing -A
# 查看有哪些名称空间
kubectl get ns

# 查看pod的描述信息
kubectl describe pod podName
kubectl describe pod -n kube-system kube-apiserver-k8s-master
# 查看指定名称空间中指定deploy的描述信息
kubectl describe deploy -n kube-system coredns

# 查看node或pod的资源使用情况
# 需要heapster 或metrics-server支持
kubectl top node
kubectl top pod

# 查看集群信息
kubectl cluster-info   或  kubectl cluster-info dump
# 查看各组件信息【172.16.1.110为master机器】
kubectl -s https://172.16.1.110:6443 get componentstatuses
```



### 1.1. get

#### 1.1.1. 基本信息查看

```bash
Usage:  kubectl get resource [-o wide|json|yaml] [-n namespace]
Man:    获取资源的相关信息，-n 指定名称空间，-o 指定输出格式
        resource可以是具体资源名称，如pod nginx-xxx；也可以是资源类型，如pod；或者all
```



```bash
[root@hdss7-21 ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-1               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
etcd-0               Healthy   {"health": "true"} 

[root@hdss7-21 ~]# kubectl get node -o wide
NAME                STATUS   ROLES         AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION          CONTAINER-RUNTIME
hdss7-21.host.com   Ready    master,node   11d   v1.15.2   10.4.7.21     <none>        CentOS Linux 7 (Core)   3.10.0-862.el7.x86_64   docker://19.3.5
hdss7-22.host.com   Ready    master,node   11d   v1.15.2   10.4.7.22     <none>        CentOS Linux 7 (Core)   3.10.0-862.el7.x86_64   docker://19.3.5

[root@hdss7-21 ~]# kubectl get svc -o wide -n kube-system
NAME                      TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
coredns                   ClusterIP   192.168.0.2       <none>        53/UDP,53/TCP,9153/TCP   4d20h   k8s-app=coredns
kubernetes-dashboard      ClusterIP   192.168.140.139   <none>        443/TCP                  3d9h    k8s-app=kubernetes-dashboard
traefik-ingress-service   ClusterIP   192.168.45.46     <none>        80/TCP,8080/TCP          3d19h   k8s-app=traefik-ingress

[root@hdss7-21 ~]# kubectl get pod nginx-ds-jdp7q -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-01-13T13:13:02Z"
  generateName: nginx-ds-
  labels:
    app: nginx-ds
......
```

#### 1.1.2. 根据标签筛选

```bash
--show-labels  显示所有标签
-l app         仅显示标签为app的资源
-l app=nginx   仅显示包含app标签，且值为nginx的资源
```



```bash
[root@hdss7-21 ~]# kubectl get pod -n app --show-labels 
NAME       READY   STATUS    RESTARTS   AGE   LABELS
pod-02     1/1     Running   0          9h    app=nginx,release=stable,version=v1.12
pod-demo   1/1     Running   9          9h    app=centos7,environment=dev,release=stable
[root@hdss7-21 ~]# kubectl get pod -n app --show-labels -l app
NAME       READY   STATUS    RESTARTS   AGE   LABELS
pod-02     1/1     Running   0          9h    app=nginx,release=stable,version=v1.12
pod-demo   1/1     Running   9          9h    app=centos7,environment=dev,release=stable
[root@hdss7-21 ~]# kubectl get pod -n app --show-labels -l app=nginx
NAME     READY   STATUS    RESTARTS   AGE   LABELS
pod-02   1/1     Running   0          9h    app=nginx,release=stable,version=v1.12
```



### 1.2. describe

```bash
Usage: kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME) [-n namespace]
Man:   描述某个资源信息
```



```bash
[root@hdss7-21 ~]# kubectl describe svc nginx-web
Name:              nginx-web
......

[root@hdss7-21 ~]# kubectl describe pod -l app=nginx-web
Name:           nginx-web-796c86d7cd-8kst5
Namespace:      default
......
```

### 1.3. 其它集群信息

```bash
[root@hdss7-21 ~]# kubectl version  # 集群版本
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:15:22Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
```



```bash
[root@hdss7-21 ~]# kubectl cluster-info  # 集群信息
Kubernetes master is running at http://localhost:8080
CoreDNS is running at http://localhost:8080/api/v1/namespaces/kube-system/services/coredns:dns/proxy
kubernetes-dashboard is running at http://localhost:8080/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```



## 2. 创建资源

### 2.1. run(弃用)

```bash
Usage:  kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [options]
Man:    通过kubectl创建一个deployment或者Job。name为deployment的名字，image为容器的镜像
        port为对容器外暴露的端口，replicas为副本数，dry-run为干运行(不创建pod)
```



```bash
[root@hdss7-21 ~]# kubectl run web-deploy --image=harbor.od.com/public/nginx:latest --replicas=2
[root@hdss7-21 ~]# kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   2/2     2            2           92s
[root@hdss7-21 ~]# kubectl get pods|grep web-deploy
web-deploy-bc78f6667-5s6rq   1/1     Running   0          119s
web-deploy-bc78f6667-h67zb   1/1     Running   0          119s
```

### 2.2. create

```bash
Uage:   kubectl create -f filename.yaml
        kubectl create resourece [options]
Man:    根据清单文件或者指定的资源参数创建资源
```



```bash
[root@hdss7-21 ~]# kubectl create namespace app     # 创建名称空间
[root@hdss7-21 ~]# kubectl get ns app
NAME   STATUS   AGE
app    Active   10s

[root@hdss7-21 ~]# kubectl create deployment app-deploy --image=harbor.od.com/public/nginx:latest -n app # 创建deployment
[root@hdss7-21 ~]# kubectl get all -n app
NAME                             READY   STATUS    RESTARTS   AGE
pod/app-deploy-5b5649fc4-plbxg   1/1     Running   0          13s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-deploy   1/1     1            1           13s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/app-deploy-5b5649fc4   1         1         1       13s
```

### 2.3. 创建service资源

```bash
Usage: Usage:  kubectl expose TYPE NAME [--port=port] [--protocol=TCP|UDP|SCTP] [--target-port=n] [--name=name] [--external-ip=external-ip-of-service] [options]
Man:    TYPE为deployment,NAME为depoly资源名称，port和target-port分别为集群和pod的端口
```



```bash
[root@hdss7-21 ~]# kubectl expose deployment app-deploy --port=80 --target-port=80 --name=app-svc -n app
[root@hdss7-21 ~]# kubectl describe svc app-svc -n app
Name:              app-svc
Namespace:         app
Labels:            app=app-deploy
Annotations:       <none>
Selector:          app=app-deploy
Type:              ClusterIP
IP:                192.168.28.124
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.7.21.8:80
```



## 3. 扩缩容

```bash
Usage:  kubectl scale --replicas=COUNT TYPE NAME [options]
Man:    对资源进行扩缩容，即修改副本数
```



```bash
[root@hdss7-21 ~]# kubectl get deploy web-deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   2/2     2            2           37m
[root@hdss7-21 ~]# kubectl scale --replicas=5 deployment web-deploy  # 扩容
[root@hdss7-21 ~]# kubectl get deploy web-deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   3/5     5            3           38m

[root@hdss7-21 ~]# kubectl scale --replicas=1 deployment web-deploy  # 缩容
[root@hdss7-21 ~]# kubectl get deploy web-deploy
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   1/1     1            1           38m
```



## 4. 删除资源

```bash
Usage:  kubectl delete ([-f FILENAME] | [-k DIRECTORY] | TYPE [(NAME | -l label | --all)]) [options]
Man:    删除指定资源
```



```bash
[root@hdss7-21 ~]# kubectl get deployment -n app
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-deploy   1/1     1            1           35m
[root@hdss7-21 ~]# kubectl delete deployment app-deploy -n app
deployment.extensions "app-deploy" deleted

[root@hdss7-21 ~]# kubectl delete ns app
namespace "app" deleted
```



## 5. 贴附到pod上

```bash
Usage:  kubectl exec (POD | TYPE/NAME) [-c CONTAINER] [flags] -- COMMAND [args...] [options]
```



```bash
[root@hdss7-21 ~]# kubectl exec nginx-web-796c86d7cd-zx2b9 -it -- /bin/bash  # 交互式
root@nginx-web-796c86d7cd-zx2b9:/# exit
exit

[root@hdss7-21 ~]# kubectl exec nginx-web-796c86d7cd-zx2b9 -- cat /etc/resolv.conf
nameserver 192.168.0.2
search default.svc.cluster.local svc.cluster.local cluster.local host.com
options ndots:5

[root@hdss7-21 ~]# kubectl exec nginx-web-796c86d7cd-zx2b9 cat /etc/resolv.conf
nameserver 192.168.0.2
search default.svc.cluster.local svc.cluster.local cluster.local host.com
options ndots:5
```



## 6. 查看资源清单文档

```bash
[root@hdss7-21 ~]# kubectl api-versions  # 查看api-version信息
apps/v1
node.k8s.io/v1beta1
v1
......
```



```bash
Usage:  kubectl explain RESOURCE [options]
Man:    查看各个字段的解释

[root@hdss7-21 ~]# kubectl explain pod.spec.containers
KIND:     Pod
VERSION:  v1

RESOURCE: containers <[]Object>
```

## 2. kubectl子命令使用分类

### 2.1 基础命令

1. create：通过文件名或标准输入创建资源
2. expose：将一个资源公开为一个新的Service
3. run：在集群中运行一个特定的镜像
4. set：在对象上设置特定的功能
5. get：显示一个或多个资源
6. explain：文档参考资料
7. edit：使用默认的编辑器编辑一个资源
8. delete：通过文件名、标准输入、资源名称或标签选择器来删除资源。



### 2.2 部署和集群管理命令

部署命令：

1. rollout：管理资源的发布
2. rolling-update：对给定的复制控制器滚动更新
3. scale：扩容或缩容Pod数量，Deployment、ReplicaSet、RC或Job
4. autoscale：创建一个字段选择扩容或缩容并设置Pod数量



集群管理命令：

1. certificate：修改证书资源
2. cluster-info：显示集群信息
3. top：显示资源（CPU/Memory/Storage）使用。需要Heapster运行
4. cordon：标记资源不可调度
5. uncordon：标记资源可调度
6. drain：驱逐节点上的应用，准备下线维护
7. taint：修改节点taint标记



故障和调试命令：

1. describe：显示特定资源或资源组的详细信息
2. logs：在一个Pod中打印一个容器日志。如果Pod只有一个容器，容器名称是可选的
3. attach：附加到一个运行的容器
4. exec：执行命令到容器
5. port-forward：转发一个或多个本地端口到一个pod
6. proxy：运行一个proxy到Kubernetes API server
7. cp：拷贝文件或目录到容器中
8. auth：检查授权



高级命令：

1. apply：通过文件名或标准输入对资源应用配置
2. patch：使用补丁修改、更新资源的字段
3. replace：通过文件名或标准输入替换一个资源
4. convert：不同的API版本之间转换配置文件



设置命令：

1. label：更新资源上的标签
2. annotate：更新资源上的注释
3. completion：用于实现kubectl工具自动补全



其他命令：

1. api-versions：打印受支持的API版本
2. config：修改kubeconfig文件（用于访问API，比如配置认证信息）
3. help：所有命令帮助
4. plugin：运行一个命令行插件
5. version：打印客户端和服务版本信息

# 操作类命令

```bash
# 创建资源
kubectl create -f xxx.yaml
# 应用资源
kubectl apply -f xxx.yaml
# 应用资源，该目录下的所有 .yaml, .yml, 或 .json 文件都会被使用
kubectl apply -f <directory>
# 创建test名称空间
kubectl create namespace test

# 删除资源
kubectl delete -f xxx.yaml
kubectl delete -f <directory>
# 删除指定的pod
kubectl delete pod podName
# 删除指定名称空间的指定pod
kubectl delete pod -n test podName
# 删除其他资源
kubectl delete svc svcName
kubectl delete deploy deployName
kubectl delete ns nsName
# 强制删除
kubectl delete pod podName -n nsName --grace-period=0 --force
kubectl delete pod podName -n nsName --grace-period=1
kubectl delete pod podName -n nsName --now

# 编辑资源
kubectl edit pod podName
```

# 进阶命令操作

```bash
# kubectl exec：进入pod启动的容器
kubectl exec -it podName -n nsName /bin/sh    #进入容器
kubectl exec -it podName -n nsName /bin/bash  #进入容器

# kubectl label：添加label值
kubectl label nodes k8s-node01 zone=north  #为指定节点添加标签
kubectl label nodes k8s-node01 zone-       #为指定节点删除标签
kubectl label pod podName -n nsName role-name=test    #为指定pod添加标签
kubectl label pod podName -n nsName role-name=dev --overwrite  #修改lable标签值
kubectl label pod podName -n nsName role-name-        #删除lable标签

# kubectl滚动升级； 通过 kubectl apply -f myapp-deployment-v1.yaml 启动deploy
kubectl apply -f myapp-deployment-v2.yaml     #通过配置文件滚动升级
kubectl set image deploy/myapp-deployment myapp="registry.cn-beijing.aliyuncs.com/google_registry/myapp:v3"   #通过命令滚动升级
kubectl rollout undo deploy/myapp-deployment 或者 kubectl rollout undo deploy myapp-deployment    #pod回滚到前一个版本
kubectl rollout undo deploy/myapp-deployment --to-revision=2  #回滚到指定历史版本

# kubectl scale：动态伸缩
kubectl scale deploy myapp-deployment --replicas=5  # 动态伸缩
kubectl scale --replicas=8 -f myapp-deployment-v2.yaml  #动态伸缩【根据资源类型和名称伸缩，其他配置「如：镜像版本不同」不生效】
```

上面滚动更新和动态伸缩涉及的deploy的yaml文件

```yaml
[root@k8s-master deploy]# cat myapp-deployment-v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 10
  # 重点关注该字段
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
        ports:
        - containerPort: 80

[root@k8s-master deploy]#
[root@k8s-master deploy]# cat myapp-deployment-v2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 10
  # 重点关注该字段
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v2
        ports:
        - containerPort: 80
```

# 按类型和名称指定资源：

```bash
# 查看一个资源类型中的多个资源
[root@k8s-master ~]# kubectl get pod -n kube-system coredns-6955765f44-c9zfh kube-proxy-28dwj
NAME                       READY   STATUS    RESTARTS   AGE
coredns-6955765f44-c9zfh   1/1     Running   8          6d7h
kube-proxy-28dwj           1/1     Running   9          6d6h
[root@k8s-master ~]#
# 查看多个资源类型
[root@k8s-master ~]# kubectl get svc,node
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   45h

NAME              STATUS   ROLES    AGE   VERSION
node/k8s-master   Ready    master   45h   v1.17.4
node/k8s-node01   Ready    <none>   45h   v1.17.4
node/k8s-node02   Ready    <none>   45h   v1.17.4
```

# 使用一个或多个文件指定资源：-f file1 -f file2 -f file<#>

```bash
# 使用YAML而不是JSON，因为YAML更容易使用，特别是对于配置文件。
kubectl get pod -f pod.yaml
```



# kubectl语法中的command操作

下表包括常见kubectl操作的简短描述和通用语法：

也可在命令行可通过kubectl -h 命令获取部分信息
或者通过以下地址查看更多详情：

```
1 https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
2 https://kubernetes.io/docs/reference/kubectl/overview/#operations
```

 

| Operation      | Syntax                                                       | Description                                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| create         | kubectl create -f FILENAME [flags]                           | 从文件或标准输入创建一个或多个资源★★★                        |
| expose         | kubectl expose (-f FILENAME                                  | TYPE NAME                                                    |
| run            | kubectl run NAME –image=image [–env=”key=value”] [–port=port] [–replicas=replicas] [–dry-run=bool] [–overrides=inline-json] [flags] | 在集群上运行指定的镜像★★★                                    |
| explain        | kubectl explain [–recursive=false] [flags]                   | 获取各种资源的文档。例如pods、nodes、services等。★★★★★       |
| get            | kubectl get (-f FILENAME                                     | TYPE [NAME                                                   |
| edit           | kubectl edit (-f FILENAME                                    | TYPE NAME                                                    |
| delete         | kubectl delete (-f FILENAME                                  | TYPE [NAME                                                   |
| rollout        | kubectl rollout SUBCOMMAND [options]                         | 对资源进行管理。有效的资源类型包括：deployments，daemonsets 和statefulsets |
| scale          | kubectl scale (-f FILENAME                                   | TYPE NAME                                                    |
| autoscale      | kubectl autoscale (-f FILENAME                               | TYPE NAME                                                    |
| cluster-info   | kubectl cluster-info [flags]                                 | 显示集群信息，显示关于集群中的主机和服务的端点信息。★★★      |
| top            | kubectl top node、kubectl top pod 需要heapster 或metrics-server支持 | 显示资源(CPU/内存/存储)使用情况★★★                           |
| cordon         | kubectl cordon NODE [options]                                | 将node标记为不可调度                                         |
| uncordon       | kubectl uncordon NODE [options]                              | 将node标记为可调度                                           |
| drain          | kubectl drain NODE [options]                                 | 排除指定node节点，为维护做准备                               |
| taint          | kubectl taint NODE NAME KEY_1=VAL_1:TAINT_EFFECT_1 … KEY_N=VAL_N:TAINT_EFFECT_N [options] | 更新一个或多个节点上的污点★★★                                |
| describe       | kubectl describe (-f FILENAME                                | TYPE [NAME_PREFIX                                            |
| logs           | kubectl logs POD [-c CONTAINER] [–follow] [flags]            | 打印pod中一个容器的日志★★★★★                                 |
| exec           | kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [– COMMAND [args…]] | 对pod中的容器执行命令或进入Pod容器★★★★★                      |
| proxy          | kubectl proxy [–port=PORT] [–www=static-dir] [–www-prefix=prefix] [–api-prefix=prefix] [flags] | 运行Kubernetes API服务的代理                                 |
| cp             | kubectl cp [options]                                         | 从宿主机复制文件和目录到一个容器；或则从容器中复制文件和目录到宿主机★★★ |
| auth           | kubectl auth [flags] [options]                               | 检查授权                                                     |
| apply          | kubectl apply -f FILENAME [flags]                            | 通过文件名中的内容或stdin将配置应用于资源★★★★★               |
| patch          | kubectl patch (-f FILENAME                                   | TYPE NAME                                                    |
| replace        | kubectl replace -f FILENAME                                  | 通过文件或stdin替换资源                                      |
| rolling-update | kubectl rolling-update OLD_CONTROLLER_NAME ([NEW_CONTROLLER_NAME] –image=NEW_CONTAINER_IMAGE | -f NEW_CONTROLLER_SPEC) [flags]                              |
| label          | kubectl label (-f FILENAME                                   | TYPE NAME                                                    |
| annotate       | kubectl annotate (-f FILENAME                                | TYPE NAME                                                    |
| api-resources  | kubectl api-resources [flags] [options]                      | 打印支持的API资源★★★                                         |
| api-versions   | kubectl api-versions [flags]                                 | 列出可用的API版本★★★                                         |
| config         | kubectl config SUBCOMMAND [flags]                            | 修改kubeconfig文件。有关详细信息，请参见各个子命令           |
| plugin         | kubectl plugin [flags] [options]                             | 提供与插件交互的实用工具                                     |
| version        | kubectl version [–client] [flags]                            | 显示在客户端和服务器上运行的Kubernetes版本★★★                |

 

# kubectl语法中的TYPE资源

下表包含常用的资源类型及其缩写别名的列表。

也可以在命令行通过kubectl api-resources得到。

| Resource Name                   | Short Names | Namespaced | Resource Kind                  |
| ------------------------------- | ----------- | ---------- | ------------------------------ |
| bindings                        |             | TRUE       | Binding                        |
| componentstatuses               | cs          | FALSE      | ComponentStatus                |
| configmaps                      | cm          | TRUE       | ConfigMap                      |
| endpoints                       | ep          | TRUE       | Endpoints                      |
| events                          | ev          | TRUE       | Event                          |
| limitranges                     | limits      | TRUE       | LimitRange                     |
| namespaces                      | ns          | FALSE      | Namespace                      |
| nodes                           | no          | FALSE      | Node                           |
| persistentvolumeclaims          | pvc         | TRUE       | PersistentVolumeClaim          |
| persistentvolumes               | pv          | FALSE      | PersistentVolume               |
| pods                            | po          | TRUE       | Pod                            |
| podtemplates                    |             | TRUE       | PodTemplate                    |
| replicationcontrollers          | rc          | TRUE       | ReplicationController          |
| resourcequotas                  | quota       | TRUE       | ResourceQuota                  |
| secrets                         |             | TRUE       | Secret                         |
| serviceaccounts                 | sa          | TRUE       | ServiceAccount                 |
| services                        | svc         | TRUE       | Service                        |
| mutatingwebhookconfigurations   |             | FALSE      | MutatingWebhookConfiguration   |
| validatingwebhookconfigurations |             | FALSE      | ValidatingWebhookConfiguration |
| customresourcedefinitions       | crd, crds   | FALSE      | CustomResourceDefinition       |
| apiservices                     |             | FALSE      | APIService                     |
| controllerrevisions             |             | TRUE       | ControllerRevision             |
| daemonsets                      | ds          | TRUE       | DaemonSet                      |
| deployments                     | deploy      | TRUE       | Deployment                     |
| replicasets                     | rs          | TRUE       | ReplicaSet                     |
| statefulsets                    | sts         | TRUE       | StatefulSet                    |
| tokenreviews                    |             | FALSE      | TokenReview                    |
| localsubjectaccessreviews       |             | TRUE       | LocalSubjectAccessReview       |
| selfsubjectaccessreviews        |             | FALSE      | SelfSubjectAccessReview        |
| selfsubjectrulesreviews         |             | FALSE      | SelfSubjectRulesReview         |
| subjectaccessreviews            |             | FALSE      | SubjectAccessReview            |
| horizontalpodautoscalers        | hpa         | TRUE       | HorizontalPodAutoscaler        |
| cronjobs                        | cj          | TRUE       | CronJob                        |
| jobs                            |             | TRUE       | Job                            |
| certificatesigningrequests      | csr         | FALSE      | CertificateSigningRequest      |
| leases                          |             | TRUE       | Lease                          |
| endpointslices                  |             | TRUE       | EndpointSlice                  |
| events                          | ev          | TRUE       | Event                          |
| ingresses                       | ing         | TRUE       | Ingress                        |
| networkpolicies                 | netpol      | TRUE       | NetworkPolicy                  |
| runtimeclasses                  |             | FALSE      | RuntimeClass                   |
| poddisruptionbudgets            | pdb         | TRUE       | PodDisruptionBudget            |
| podsecuritypolicies             | psp         | FALSE      | PodSecurityPolicy              |
| clusterrolebindings             |             | FALSE      | ClusterRoleBinding             |
| clusterroles                    |             | FALSE      | ClusterRole                    |
| rolebindings                    |             | TRUE       | RoleBinding                    |
| roles                           |             | TRUE       | Role                           |
| priorityclasses                 | pc          | FALSE      | PriorityClass                  |
| csidrivers                      |             | FALSE      | CSIDriver                      |
| csinodes                        |             | FALSE      | CSINode                        |
| storageclasses                  | sc          | FALSE      | StorageClass                   |
| volumeattachments               |             | FALSE      | VolumeAttachment               |

 

# kubectl 输出选项

## 格式化输出

所有kubectl命令的默认输出格式是人类可读的纯文本格式。

要将详细信息以特定的格式输出到终端窗口，可以将 -o 或 --output标识添加到受支持的kubectl命令中。

 

## 语法

```
kubectl [command] [TYPE] [NAME] -o <output_format>
```

根据kubectl操作，支持以下输出格式：

| Output format           | Description                                          |
| ----------------------- | ---------------------------------------------------- |
| -o custom-columns=      | 使用逗号分隔的自定义列列表打印表                     |
| -o custom-columns-file= | 使用文件中的自定义列模板打印表                       |
| -o json                 | 输出一个JSON格式的API对象                            |
| -o jsonpath=            | 打印jsonpath表达式中定义的字段                       |
| -o jsonpath-file=       | 通过文件打印jsonpath表达式定义的字段                 |
| -o name                 | 只打印资源名，不打印其他任何内容                     |
| -o wide                 | 以纯文本格式输出，包含附加信息。对于pods，包含节点名 |
| -o yaml                 | 输出一个YAML格式的API对象                            |

## 示例

wide示例

```bash
[root@k8s-master ~]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   1          28h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get pod -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
nginx-demo   1/1     Running   1          28h   10.244.3.9   k8s-node01   <none>           <none>
```

 

yaml示例

```bash
[root@k8s-master ~]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   1          28h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get pod -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: Pod
  metadata:
    annotations:
………………
```

 

json示例

```bash
[root@k8s-master ~]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   1          28h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get pod -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "annotations": {
………………
```

 

name示例

```bash
[root@k8s-master ~]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   1          28h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get pod -o name
pod/nginx-demo
```

 

custom-columns示例

```bash
[root@k8s-master ~]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   1          29h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl get pods -o custom-columns=NAME:.metadata.name,UID:.metadata.uid,imageName:.spec.containers[0].image
NAME         UID                                    imageName
nginx-demo   08121fc6-969b-4b4e-9aa4-b990a5d02148   registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
```

说明：custom-columns=key:value；其中key表示列明；value表示要显示信息，这个value信息可以通过-o json或-o yaml获取。

 

custom-columns-file示例

```bash
[root@k8s-master test]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          80s
[root@k8s-master test]#
# 要显示的列明和数据来源
[root@k8s-master test]# cat custom-col.conf
NAME          UID          imageName                containerPort
metadata.name metadata.uid spec.containers[0].image spec.containers[0].ports[0].containerPort
[root@k8s-master test]#
[root@k8s-master test]# kubectl get pod -o custom-columns-file=custom-col.conf
NAME         UID                                    imageName                                                     containerPort
nginx-demo   769dc3f4-2ffc-407c-a351-56b74ddaba4c   registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17   80
```



 

jsonpath示例

```bash
[root@k8s-master test]# kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          13m
[root@k8s-master test]#
[root@k8s-master test]# kubectl get pods -o jsonpath='{.items[0].metadata.name},{.items[0].spec.containers[0].image}'
nginx-demo,registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17
```

 

jsonpath-file示例

```bash
[root@k8s-master test]# kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          16m
[root@k8s-master test]#
# 要显示的数据来源
[root@k8s-master test]# cat custom-json.conf
{.items[0].metadata.name},{.items[0].spec.containers[0].image},{.items[0].spec.containers[0].ports[0].containerPort}
[root@k8s-master test]#
[root@k8s-master test]# kubectl get pod -o jsonpath-file=custom-json.conf
nginx-demo,registry.cn-beijing.aliyuncs.com/google_registry/nginx:1.17,80
```