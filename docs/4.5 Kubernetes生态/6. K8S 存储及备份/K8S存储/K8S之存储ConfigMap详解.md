[Kubernetes K8S之存储ConfigMap详解](https://www.cnblogs.com/zhanglianghhh/p/13818190.html)

# ConfigMap概述

ConfigMap 是一种 API 对象，用来将非机密性的数据保存到健值对中。使用时可以用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap 将环境配置信息和容器镜像解耦，便于应用配置的修改。当你需要储存机密信息时可以使用 Secret 对象。

备注：ConfigMap 并不提供保密或者加密功能。如果你想存储的数据是机密的，请使用 Secret；或者使用其他第三方工具来保证数据的私密性，而不是用 ConfigMap。

 

# ConfigMap创建方式

## 通过目录创建

配置文件目录

```bash
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# ll /root/k8s_practice/storage/configmap    # 配置文件存在哪个目录下
total 8
-rw-r--r-- 1 root root 159 Jun  7 14:52 game.properties
-rw-r--r-- 1 root root  83 Jun  7 14:53 ui.properties
[root@k8s-master storage]#
[root@k8s-master storage]# cat configmap/game.properties    # 涉及文件1
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAs
secret.code.allowed=true
secret.code.lives=30

[root@k8s-master storage]#
[root@k8s-master storage]# cat configmap/ui.properties   # 涉及文件2
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
```

创建ConfigMap并查看状态

```bash
[root@k8s-master storage]# kubectl create configmap game-config --from-file=/root/k8s_practice/storage/configmap
configmap/game-config created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get configmap
NAME          DATA   AGE
game-config   2      14s
```

查看ConfigMap有哪些数据

```bash
[root@k8s-master storage]# kubectl get configmap -o yaml   ##### 查看方式1
apiVersion: v1
items:
- apiVersion: v1
  data:
    game.properties: |+   ##### 本段最后有一行空格，+ 表示保留字符串行末尾的换行
      enemies=aliens
      lives=3
      enemies.cheat=true
      enemies.cheat.level=noGoodRotten
      secret.code.passphrase=UUDDLRLRBABAs
      secret.code.allowed=true
      secret.code.lives=30

    ui.properties: |
      color.good=purple
      color.bad=yellow
      allow.textmode=true
      how.nice.to.look=fairlyNice
  kind: ConfigMap
  metadata:
    creationTimestamp: "2020-06-07T06:57:28Z"
    name: game-config
    namespace: default
    resourceVersion: "889177"
    selfLink: /api/v1/namespaces/default/configmaps/game-config
    uid: 6952ac85-ded0-4c5e-89fd-b0c6f0546ecf
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl describe configmap game-config   ##### 查看方式2
Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAs
secret.code.allowed=true
secret.code.lives=30


ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
```

## 通过文件创建

配置文件位置

```bash
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat /root/k8s_practice/storage/configmap/game.properties
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAs
secret.code.allowed=true
secret.code.lives=30
```

创建ConfigMap并查看状态

```bash
[root@k8s-master storage]# kubectl create configmap game-config-2 --from-file=/root/k8s_practice/storage/configmap/game.properties
configmap/game-config-2 created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get configmap game-config-2
NAME            DATA   AGE
game-config-2   1      29s
```

查看ConfigMap有哪些数据

```bash
[root@k8s-master storage]# kubectl get configmap game-config-2 -o yaml   ##### 查看方式1
apiVersion: v1
data:
  game.properties: |+   ##### 本段最后有一行空格，+ 表示保留字符串行末尾的换行
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAs
    secret.code.allowed=true
    secret.code.lives=30

kind: ConfigMap
metadata:
  creationTimestamp: "2020-06-07T07:05:47Z"
  name: game-config-2
  namespace: default
  resourceVersion: "890437"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-2
  uid: 02d99802-c23f-45ad-b4e1-dea9bcb166d8
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl describe configmap game-config-2    ##### 查看方式2
Name:         game-config-2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAs
secret.code.allowed=true
secret.code.lives=30


Events:  <none>
```

## 通过命令行创建

创建ConfigMap并查看状态

```bash
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# kubectl create configmap special-config --from-literal=special.how=very --from-literal="special.type=charm"
configmap/special-config created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get configmap special-config
NAME             DATA   AGE
special-config   2      23s
```

查看ConfigMap有哪些数据

```bash
[root@k8s-master storage]# kubectl get configmap special-config -o yaml    ##### 查看方式1
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
metadata:
  creationTimestamp: "2020-06-07T09:32:04Z"
  name: special-config
  namespace: default
  resourceVersion: "912702"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 76698e78-1380-4826-b5ac-d9c81f746eac
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl describe configmap special-config    ##### 查看方式2
Name:         special-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
special.how:
----
very
special.type:
----
charm
Events:  <none>
```

## 通过yaml文件创建

yaml文件

```bash
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-demo
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "3"
  ui_properties_file_name: 'user-interface.properties'
  #
  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true	
```

创建ConfigMap并查看状态

```bash
[root@k8s-master storage]# kubectl apply -f configmap.yaml
configmap/configmap-demo created
[root@k8s-master storage]# kubectl get configmap configmap-demo
NAME             DATA   AGE
configmap-demo   4      2m59s
```

查看ConfigMap有哪些数据

```yaml
[root@k8s-master storage]# kubectl get configmap configmap-demo -o yaml    ##### 查看方式1
apiVersion: v1
data:
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  player_initial_lives: "3"
  ui_properties_file_name: user-interface.properties
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"game.properties":"enemy.types=aliens,monsters\nplayer.maximum-lives=5\n","player_initial_lives":"3","ui_properties_file_name":"user-interface.properties","user-interface.properties":"color.good=purple\ncolor.bad=yellow\nallow.textmode=true\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"configmap-demo","namespace":"default"}}
  creationTimestamp: "2020-06-07T11:36:46Z"
  name: configmap-demo
  namespace: default
  resourceVersion: "931685"
  selfLink: /api/v1/namespaces/default/configmaps/configmap-demo
  uid: fdad7000-87bd-4b72-be98-40dd8fe6400a
[root@k8s-master storage]#
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl describe configmap configmap-demo     ##### 查看方式2
Name:         configmap-demo
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","data":{"game.properties":"enemy.types=aliens,monsters\nplayer.maximum-lives=5\n","player_initial_lives":"3","ui_proper...

Data
====
game.properties:
----
enemy.types=aliens,monsters
player.maximum-lives=5

player_initial_lives:
----
3
ui_properties_file_name:
----
user-interface.properties
user-interface.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true

Events:  <none>
```

# Pod中使用ConfigMap

如何在Pod中使用上述的ConfigMap信息。

## 当前存在的ConfigMap

```bash
[root@k8s-master storage]# kubectl get configmap
NAME             DATA   AGE
configmap-demo   4      30m
game-config      2      5h9m
game-config-2    1      5h1m
special-config   2      5m48s
```

## 使用ConfigMap来替代环境变量

yaml文件

```bash
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat pod_configmap_env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap-env
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    command: ["/bin/sh", "-c", "env"]
    ### 引用方式1
    env:
    - name: SPECAIL_HOW_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config   ### 这个name的值来自 ConfigMap
          key: special.how       ### 这个key的值为需要取值的键
    - name: SPECAIL_TPYE_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.type
    ### 引用方式2
    envFrom:
    - configMapRef:
        name: game-config-2   ### 这个name的值来自 ConfigMap
    restartPolicy: Never
```

启动pod并查看状态

```bash
[root@k8s-master storage]# kubectl apply -f pod_configmap_env.yaml
pod/pod-configmap-env created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get pod -o wide
NAME                READY   STATUS      RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-configmap-env   0/1     Completed   0          6s    10.244.2.147   k8s-node02   <none>           <none>
```

查看打印日志

```bash
[root@k8s-master storage]# kubectl logs pod-configmap-env
MYAPP_SVC_PORT_80_TCP_ADDR=10.98.57.156
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
MYAPP_SVC_PORT_80_TCP_PORT=80
HOSTNAME=pod-configmap-env
SHLVL=1
MYAPP_SVC_PORT_80_TCP_PROTO=tcp
HOME=/root
SPECAIL_HOW_KEY=very  ### 来自ConfigMap
game.properties=enemies=aliens  ### 来自ConfigMap
lives=3  ### 来自ConfigMap
enemies.cheat=true  ### 来自ConfigMap
enemies.cheat.level=noGoodRotten  ### 来自ConfigMap
secret.code.passphrase=UUDDLRLRBABAs  ### 来自ConfigMap
secret.code.allowed=true  ### 来自ConfigMap
secret.code.lives=30  ### 来自ConfigMap


SPECAIL_TPYE_KEY=charm  ### 来自ConfigMap
MYAPP_SVC_PORT_80_TCP=tcp://10.98.57.156:80
NGINX_VERSION=1.12.2
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
MYAPP_SVC_SERVICE_HOST=10.98.57.156
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
PWD=/
KUBERNETES_SERVICE_HOST=10.96.0.1
MYAPP_SVC_SERVICE_PORT=80
MYAPP_SVC_PORT=tcp://10.98.57.156:80
```

## 使用ConfigMap设置命令行参数

yaml文件

```yaml
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat pod_configmap_cmd.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap-cmd
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    command: ["/bin/sh", "-c", "echo \"===$(SPECAIL_HOW_KEY)===$(SPECAIL_TPYE_KEY)===\""]
    env:
    - name: SPECAIL_HOW_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.how
    - name: SPECAIL_TPYE_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: special.type
  restartPolicy: Never
```

启动pod并查看状态

```bash
[root@k8s-master storage]# kubectl apply -f pod_configmap_cmd.yaml
pod/pod-configmap-cmd created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get pod -o wide
NAME                READY   STATUS      RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-configmap-cmd   0/1     Completed   0          5s    10.244.4.125   k8s-node01   <none>           <none>
```

查看打印日志

```bash
[root@k8s-master storage]# kubectl logs pod-configmap-cmd
===very===charm===
```

## 通过数据卷插件使用ConfigMap【推荐】

在数据卷里面使用ConfigMap，最基本的就是将文件填入数据卷，在这个文件中，键就是文件名【第一层级的键】，键值就是文件内容。

yaml文件

```yaml
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat pod_configmap_volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap-volume
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    #command: ["/bin/sh", "-c", "ls -l /etc/config/"]
    command: ["/bin/sh", "-c", "sleep 600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: configmap-demo
  restartPolicy: Never
```

启动pod并查看状态

```bash
[root@k8s-master storage]# kubectl apply -f pod_configmap_volume.yaml
pod/pod-configmap-volume created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
pod-configmap-volume   1/1     Running   0          5s    10.244.2.153   k8s-node02   <none>           <none>
```

进入pod并查看

```bash
[root@k8s-master storage]# kubectl exec -it pod-configmap-volume sh
/ # ls /etc/config
game.properties            player_initial_lives       ui_properties_file_name    user-interface.properties
/ #
/ #
/ #
/ # cat /etc/config/player_initial_lives
3/ #
/ #
/ #
/ # cat /etc/config/ui_properties_file_name
user-interface.properties/ #
/ #
/ #
/ # cat /etc/config/game.properties
enemy.types=aliens,monsters
player.maximum-lives=5
/ #
/ #
/ # cat /etc/config/user-interface.properties
color.good=purple
color.bad=yellow
allow.textmode=true
```

# ConfigMap热更新

## 准备工作

yaml文件

```yaml
[root@k8s-master storage]# pwd
/root/k8s_practice/storage
[root@k8s-master storage]# cat pod_configmap_hot.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-config
  namespace: default
data:
  log_level: INFO
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 2
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
        - containerPort: 80
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: log-config
```

应用yaml文件并查看状态

```bash
[root@k8s-master storage]# kubectl apply -f pod_configmap_hot.yaml
configmap/log-config created
deployment.apps/myapp-deploy created
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get configmap log-config
NAME         DATA   AGE
log-config   1      21s
[root@k8s-master storage]#
[root@k8s-master storage]# kubectl get pod -o wide
NAME                           READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
myapp-deploy-58ff9c997-drhwk   1/1     Running   0          30s   10.244.2.154   k8s-node02   <none>           <none>
myapp-deploy-58ff9c997-n68j2   1/1     Running   0          30s   10.244.4.126   k8s-node01   <none>           <none>
```

查看ConfigMap信息

```yaml
[root@k8s-master storage]# kubectl get configmap log-config -o yaml
apiVersion: v1
data:
  log_level: INFO
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"log_level":"INFO"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"log-config","namespace":"default"}}
  creationTimestamp: "2020-06-07T16:08:11Z"
  name: log-config
  namespace: default
  resourceVersion: "971348"
  selfLink: /api/v1/namespaces/default/configmaps/log-config
  uid: 7e78e1d7-12de-4601-9915-cefbc96ca305
```

查看pod中的ConfigMap信息

```bash
[root@k8s-master storage]# kubectl exec -it myapp-deploy-58ff9c997-drhwk -- cat /etc/config/log_level
INFO
```

## 热更新

修改ConfigMap

```bash
[root@k8s-master storage]# kubectl edit configmap log-config     ### 将 INFO 改为了 DEBUG
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  log_level: DEBUG
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"log_level":"DEBUG"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"log-config","namespace":"default"}}
  creationTimestamp: "2020-06-07T16:08:11Z"
  name: log-config
  namespace: default
  resourceVersion: "971348"
  selfLink: /api/v1/namespaces/default/configmaps/log-config
  uid: 7e78e1d7-12de-4601-9915-cefbc96ca305
```

查看ConfigMap信息

```yaml
[root@k8s-master storage]# kubectl get configmap log-config -o yaml
apiVersion: v1
data:
  log_level: DEBUG
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"log_level":"DEBUG"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"log-config","namespace":"default"}}
  creationTimestamp: "2020-06-07T16:08:11Z"
  name: log-config
  namespace: default
  resourceVersion: "972893"
  selfLink: /api/v1/namespaces/default/configmaps/log-config
  uid: 7e78e1d7-12de-4601-9915-cefbc96ca305
```

稍后10秒左右，再次查看pod中的ConfigMap信息

```bash
[root@k8s-master storage]# kubectl exec -it myapp-deploy-58ff9c997-drhwk -- cat /etc/config/log_level
DEBUG
```

由此可见，完成了一次热更新