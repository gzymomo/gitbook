[Kubernetes K8S之存储Secret详解](https://www.cnblogs.com/zhanglianghhh/p/13743024.html)



# Secret概述

Secret解决了密码、token、秘钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod Spec中。Secret可以以Volume或者环境变量的方式使用。

用户可以创建 secret，同时系统也创建了一些 secret。

要使用 secret，pod 需要引用 secret。Pod 可以用两种方式使用 secret：作为 volume 中的文件被挂载到 pod 中的一个或者多个容器里，或者当 kubelet 为 pod 拉取镜像时使用。

 

## Secret类型

- Service Account：用来访问Kubernetes API，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中。
- Opaque：base64编码格式的Secret，用来存储密码、秘钥等。
- kubernetes.io/dockerconfigjson：用来存储私有docker registry的认证信息。

 

# Service Account

通过kube-proxy查看

```
[root@k8s-master ~]# kubectl get pod -A | grep 'kube-proxy'
kube-system            kube-proxy-6bfh7                             1/1     Running   12         7d3h
kube-system            kube-proxy-6vfkf                             1/1     Running   11         7d3h
kube-system            kube-proxy-bvl9n                             1/1     Running   11         7d3h
[root@k8s-master ~]#
[root@k8s-master ~]# kubectl exec -it -n kube-system kube-proxy-6bfh7 -- /bin/sh
# ls -l /run/secrets/kubernetes.io/serviceaccount
total 0
lrwxrwxrwx 1 root root 13 Jun  8 13:39 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root 16 Jun  8 13:39 namespace -> ..data/namespace
lrwxrwxrwx 1 root root 12 Jun  8 13:39 token -> ..data/token
```

 

# Opaque Secret

## 创建secret

手动加密，基于base64加密

```bash
[root@k8s-master ~]# echo -n 'admin' | base64
YWRtaW4=
[root@k8s-master ~]# echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

 

yaml文件

```yaml
 1 [root@k8s-master secret]# pwd
 2 /root/k8s_practice/secret
 3 [root@k8s-master secret]# cat secret.yaml 
 4 apiVersion: v1
 5 kind: Secret
 6 metadata:
 7   name: mysecret
 8 type: Opaque
 9 data:
10   username: YWRtaW4=
11   password: MWYyZDFlMmU2N2Rm
```

 

或者通过如下命令行创建【secret名称故意设置不一样，以方便查看对比】，生成secret后会自动加密，而非明文存储。

```bash
kubectl create secret generic db-user-pass --from-literal=username=admin --from-literal=password=1f2d1e2e67df
```

 

生成secret，并查看状态

```
[root@k8s-master secret]# kubectl apply -f secret.yaml
secret/mysecret created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get secret   ### 查看默认名称空间的secret简要信息
NAME                  TYPE                                  DATA   AGE
basic-auth            Opaque                                1      2d12h
default-token-v48g4   kubernetes.io/service-account-token   3      27d
mysecret              Opaque                                2      23s  ### 可见已创建
tls-secret            kubernetes.io/tls                     2      3d2h
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get secret mysecret -o yaml     ### 查看mysecret详细信息
apiVersion: v1
data:
  password: MWYyZDFlMmU2N2Rm
  username: YWRtaW4=
kind: Secret
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"password":"MWYyZDFlMmU2N2Rm","username":"YWRtaW4="},"kind":"Secret","metadata":{"annotations":{},"name":"mysecret","namespace":"default"},"type":"Opaque"}
  creationTimestamp: "2020-06-08T14:08:59Z"
  name: mysecret
  namespace: default
  resourceVersion: "987419"
  selfLink: /api/v1/namespaces/default/secrets/mysecret
  uid: 27b58929-71c4-495b-99a5-0d411910a529
type: Opaque
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl describe secret mysecret     ### 查看描述信息
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:
Type:         Opaque

Data
====
password:  12 bytes
username:  5 bytes
```

 

## 将Secret挂载到Volume中

yaml文件

```
[root@k8s-master secret]# pwd
/root/k8s_practice/secret
[root@k8s-master secret]# cat pod_secret_volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-volume
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret
```

 

启动pod并查看状态

```
[root@k8s-master secret]# kubectl apply -f pod_secret_volume.yaml
pod/pod-secret-volume created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get pod -o wide
NAME                READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-secret-volume   1/1     Running   0          16s   10.244.2.159   k8s-node02   <none>           <none>
```

 

查看secret信息

```
[root@k8s-master secret]# kubectl exec -it pod-secret-volume -- /bin/sh
/ # ls /etc/secret
password  username
/ #
/ # cat /etc/secret/username
admin/ #
/ #
/ # cat /etc/secret/password
1f2d1e2e67df/ #
```

由上可见，在pod中的secret信息实际已经被解密。

 

## 将Secret导入到环境变量中

yaml文件

```
[root@k8s-master secret]# pwd
/root/k8s_practice/secret
[root@k8s-master secret]# cat pod_secret_env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-env
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
  restartPolicy: Never
```

 

启动pod并查看状态

```
[root@k8s-master secret]# kubectl apply -f pod_secret_env.yaml
pod/pod-secret-env created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-secret-env   1/1     Running   0          6s    10.244.2.160   k8s-node02   <none>           <none>
```

 

查看secret信息

```
[root@k8s-master secret]# kubectl exec -it pod-secret-env -- /bin/sh
/ # env
………………
HOME=/root
SECRET_PASSWORD=1f2d1e2e67df    ### secret信息
MYAPP_SVC_PORT_80_TCP=tcp://10.98.57.156:80
TERM=xterm
NGINX_VERSION=1.12.2
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
MYAPP_SVC_SERVICE_HOST=10.98.57.156
SECRET_USERNAME=admin    ### secret信息
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
………………
```

由上可见，在pod中的secret信息实际已经被解密。

 

# docker-registry Secret

## harbor镜像仓库

首先使用harbor搭建镜像仓库，搭建部署过程参考：「[Harbor企业级私有Docker镜像仓库部署](https://www.cnblogs.com/zhanglianghhh/p/13205786.html)」

**harbor部分配置文件信息**

```
[root@k8s-master harbor]# pwd
/root/App/harbor
[root@k8s-master harbor]# vim harbor.yml
# Configuration file of Harbor
hostname: 172.16.1.110

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 5000

# https related config
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /etc/harbor/cert/httpd.crt
  private_key: /etc/harbor/cert/httpd.key
harbor_admin_password: Harbor12345
```

 

**启动harbor后客户端http设置**

集群所有机器都要操作

```
[root@k8s-master ~]# vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "insecure-registries": ["172.16.1.110:5000"]
}
[root@k8s-master ~]#
[root@k8s-master ~]# systemctl restart docker   # 重启docker服务
```

 添加了 “insecure-registries”: [“172.16.1.110:5000”] 这行，其中172.16.1.110为内网IP地址。该文件必须符合 json 规范，否则 Docker 将不能启动。

如果在Harbor所在的机器重启了docker服务，记得要重新启动Harbor。

 

**创建「私有」仓库**

**![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200928084254635-1852476315.png)**

 

 

**镜像上传**

```bash
docker pull registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
docker tag registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1 172.16.1.110:5000/k8s-secret/myapp:v1
# 登录
docker login 172.16.1.110:5000 -u admin -p Harbor12345
# 上传
docker push 172.16.1.110:5000/k8s-secret/myapp:v1
```

 

![img](https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200928084324114-1106422618.png)

 

**退出登录**

之后在操作机上退出harbor登录，便于后面演示

```
### 退出harbor登录
[root@k8s-node02 ~]# docker logout 172.16.1.110:5000
Removing login credentials for 172.16.1.110:5000
### 拉取失败，需要先登录。表明完成准备工作
[root@k8s-master secret]# docker pull 172.16.1.110:5000/k8s-secret/myapp:v1
Error response from daemon: pull access denied for 172.16.1.110:5000/k8s-secret/myapp, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
```

 

## pod直接下载镜像

在yaml文件中指定image后，直接启动pod

```bash
[root@k8s-master secret]# pwd
/root/k8s_practice/secret
[root@k8s-master secret]# cat pod_secret_registry.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-registry
spec:
  containers:
  - name: myapp
    image: 172.16.1.110:5000/k8s-secret/myapp:v1
```

 

启动pod并查看状态

```bash
[root@k8s-master secret]# kubectl apply -f pod_secret_registry.yaml
pod/pod-secret-registry created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get pod -o wide     ### 可见镜像下载失败
NAME                  READY   STATUS             RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-secret-registry   0/1     ImagePullBackOff   0          7s    10.244.2.161   k8s-node02   <none>           <none>
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl describe pod pod-secret-registry     ### 查看pod详情
Name:         pod-secret-registry
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Mon, 08 Jun 2020 23:59:07 +0800
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"pod-secret-registry","namespace":"default"},"spec":{"containers":[{"i...
Status:       Pending
IP:           10.244.2.161
IPs:
  IP:  10.244.2.161
Containers:
  myapp:
    Container ID:
    Image:          172.16.1.110:5000/k8s-secret/myapp:v1
    Image ID:
………………
Events:
  Type     Reason     Age                From                 Message
  ----     ------     ----               ----                 -------
  Normal   Scheduled  23s                default-scheduler    Successfully assigned default/pod-secret-registry to k8s-node02
  Normal   BackOff    19s (x2 over 20s)  kubelet, k8s-node02  Back-off pulling image "172.16.1.110:5000/k8s-secret/myapp:v1"
  Warning  Failed     19s (x2 over 20s)  kubelet, k8s-node02  Error: ImagePullBackOff
  Normal   Pulling    9s (x2 over 21s)   kubelet, k8s-node02  Pulling image "172.16.1.110:5000/k8s-secret/myapp:v1"
  Warning  Failed     9s (x2 over 21s)   kubelet, k8s-node02  Failed to pull image "172.16.1.110:5000/k8s-secret/myapp:v1": rpc error: code = Unknown desc = Error response from daemon: pull access denied for 172.16.1.110:5000/k8s-secret/myapp, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed     9s (x2 over 21s)   kubelet, k8s-node02  Error: ErrImagePull
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl delete -f pod_secret_registry.yaml
```

可见拉取私有镜像失败。

 

## pod通过Secret下载镜像

通过命令行创建Secret，并查看其描述信息

```bash
[root@k8s-master secret]# kubectl create secret docker-registry myregistrysecret --docker-server='172.16.1.110:5000' --docker-username='admin' --docker-password='Harbor12345'
secret/myregistrysecret created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
basic-auth            Opaque                                1      2d14h
default-token-v48g4   kubernetes.io/service-account-token   3      27d
myregistrysecret      kubernetes.io/dockerconfigjson        1      8s    # 刚刚创建的
mysecret              Opaque                                2      118m
tls-secret            kubernetes.io/tls                     2      3d4h
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get secret myregistrysecret -o yaml     ### 查看详细信息
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyIxMC4wLjAuMTEwOjUwMDAiOnsidXNlcm5hbWUiOiJhZG1pbiIsInBhc3N3b3JkIjoiSGFyYm9yMTIzNDUiLCJhdXRoIjoiWVdSdGFXNDZTR0Z5WW05eU1USXpORFU9In19fQ==
kind: Secret
metadata:
  creationTimestamp: "2020-06-08T16:07:32Z"
  name: myregistrysecret
  namespace: default
  resourceVersion: "1004582"
  selfLink: /api/v1/namespaces/default/secrets/myregistrysecret
  uid: b95f4386-64bc-4ba3-b43a-08afb1c1eb9d
type: kubernetes.io/dockerconfigjson
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl describe secret myregistrysecret     ### 查看描述信息
Name:         myregistrysecret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  109 bytes
```

 

修改之前的yaml文件

```yaml
[root@k8s-master secret]# cat pod_secret_registry.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-registry
spec:
  containers:
  - name: myapp
    image: 172.16.1.110:5000/k8s-secret/myapp:v1
  imagePullSecrets:
  - name: myregistrysecret
```

 

启动pod并查看状态

```bash
[root@k8s-master secret]# kubectl apply -f pod_secret_registry.yaml
pod/pod-secret-registry created
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl get pod -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-secret-registry   1/1     Running   0          8s    10.244.2.162   k8s-node02   <none>           <none>
[root@k8s-master secret]#
[root@k8s-master secret]# kubectl describe pod pod-secret-registry
Name:         pod-secret-registry
Namespace:    default
Priority:     0
Node:         k8s-node02/172.16.1.112
Start Time:   Tue, 09 Jun 2020 00:22:40 +0800
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"pod-secret-registry","namespace":"default"},"spec":{"containers":[{"i...
Status:       Running
IP:           10.244.2.162
IPs:
  IP:  10.244.2.162
Containers:
  myapp:
    Container ID:   docker://ef4d42f1f1616a44c2a6c0a5a71333b27f46dfe76eb392962813a28d69150c00
    Image:          172.16.1.110:5000/k8s-secret/myapp:v1
    Image ID:       docker-pullable://172.16.1.110:5000/k8s-secret/myapp@sha256:9eeca44ba2d410e54fccc54cbe9c021802aa8b9836a0bcf3d3229354e4c8870e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 09 Jun 2020 00:22:41 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-v48g4 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-v48g4:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-v48g4
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                 Message
  ----    ------     ----  ----                 -------
  Normal  Scheduled  22s   default-scheduler    Successfully assigned default/pod-secret-registry to k8s-node02
  Normal  Pulling    22s   kubelet, k8s-node02  Pulling image "172.16.1.110:5000/k8s-secret/myapp:v1"
  Normal  Pulled     22s   kubelet, k8s-node02  Successfully pulled image "172.16.1.110:5000/k8s-secret/myapp:v1"
  Normal  Created    22s   kubelet, k8s-node02  Created container myapp
  Normal  Started    21s   kubelet, k8s-node02  Started container myapp
```

由上可见，通过secret认证后pod拉取私有镜像是可以的。