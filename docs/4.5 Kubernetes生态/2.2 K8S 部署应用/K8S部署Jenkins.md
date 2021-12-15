## 安装Jenkins（一般的做法）

### 1.安装nfs（之前安装过的话，可以省略此步）

使用 nfs 最大的问题就是写权限，可以使用 kubernetes 的 securityContext/runAsUser 指定 jenkins 容器中运行 jenkins 的用户 uid，以此来指定 nfs 目录的权限，让 jenkins 容器可写；也可以不限制，让所有用户都可以写。这里为了简单，就让所有用户可写了。

如果之前已经安装过nfs，则这一步可以省略。找一台主机，安装 nfs，这里，我以在Master节点（binghe101服务器）上安装nfs为例。

在命令行输入如下命令安装并启动nfs。

```bash
yum install nfs-utils -y
systemctl start nfs-server
systemctl enable nfs-server
```

### 2.创建nfs共享目录

在Master节点（binghe101服务器）上创建 `/opt/nfs/jenkins-data`目录作为nfs的共享目录，如下所示。

```bash
mkdir -p /opt/nfs/jenkins-data
```

接下来，编辑/etc/exports文件，如下所示。

```bash
vim /etc/exports
```

在/etc/exports文件文件中添加如下一行配置。

```bash
/opt/nfs/jenkins-data 192.168.175.0/24(rw,all_squash)
```

这里的 ip 使用 kubernetes node 节点的 ip 范围，后面的 `all_squash` 选项会将所有访问的用户都映射成 nfsnobody 用户，不管你是什么用户访问，最终都会压缩成 nfsnobody，所以只要将 `/opt/nfs/jenkins-data` 的属主改为 nfsnobody，那么无论什么用户来访问都具有写权限。

这个选项在很多机器上由于用户 uid 不规范导致启动进程的用户不同，但是同时要对一个共享目录具有写权限时很有效。

接下来，为 `/opt/nfs/jenkins-data`目录授权，并重新加载nfs，如下所示。

```bash
chown -R 1000 /opt/nfs/jenkins-data/
systemctl reload nfs-server
```

在K8S集群中任意一个节点上使用如下命令进行验证：

```bash
showmount -e NFS_IP
```

如果能够看到 /opt/nfs/jenkins-data 就表示 ok 了。

具体如下所示。

```bash
[root@binghe101 ~]# showmount -e 192.168.175.101
Export list for 192.168.175.101:
/opt/nfs/jenkins-data 192.168.175.0/24

[root@binghe102 ~]# showmount -e 192.168.175.101
Export list for 192.168.175.101:
/opt/nfs/jenkins-data 192.168.175.0/24
```

### 3.创建PV

Jenkins 其实只要加载对应的目录就可以读取之前的数据，但是由于 deployment 无法定义存储卷，因此我们只能使用 StatefulSet。

首先创建 pv，pv 是给 StatefulSet 使用的，每次 StatefulSet 启动都会通过 volumeClaimTemplates 这个模板去创建 pvc，因此必须得有 pv，才能供 pvc 绑定。

创建jenkins-pv.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins
spec:
  nfs:
    path: /opt/nfs/jenkins-data
    server: 192.168.175.101
  accessModes: ["ReadWriteOnce"]
  capacity:
    storage: 1Ti
```

我这里给了 1T存储空间，可以根据实际配置。

执行如下命令创建pv。

```bash
kubectl apply -f jenkins-pv.yaml 
```

### 4.创建serviceAccount

创建service account，因为 jenkins 后面需要能够动态创建 slave，因此它必须具备一些权限。

创建jenkins-service-account.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkins
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create", "delete", "get", "list", "patch", "update", "watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
```

上述配置中，创建了一个 RoleBinding 和一个 ServiceAccount，并且将 RoleBinding 的权限绑定到这个用户上。所以，jenkins 容器必须使用这个 ServiceAccount 运行才行，不然 RoleBinding 的权限它将不具备。

RoleBinding 的权限很容易就看懂了，因为 jenkins 需要创建和删除 slave，所以才需要上面这些权限。至于 secrets 权限，则是 https 证书。

执行如下命令创建serviceAccount。

```bash
kubectl apply -f jenkins-service-account.yaml 
```

### 5.安装Jenkins

创建jenkins-statefulset.yaml文件，文件内容如下所示。

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jenkins
  labels:
    name: jenkins
spec:
  selector:
    matchLabels:
      name: jenkins
  serviceName: jenkins
  replicas: 1
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      name: jenkins
      labels:
        name: jenkins
    spec:
      terminationGracePeriodSeconds: 10
      serviceAccountName: jenkins
      containers:
        - name: jenkins
          image: docker.io/jenkins/jenkins:lts
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
            - containerPort: 32100
          resources:
            limits:
              cpu: 4
              memory: 4Gi
            requests:
              cpu: 4
              memory: 4Gi
          env:
            - name: LIMITS_MEMORY
              valueFrom:
                resourceFieldRef:
                  resource: limits.memory
                  divisor: 1Mi
            - name: JAVA_OPTS
              # value: -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
              value: -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
          livenessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12 # ~2 minutes
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12 # ~2 minutes
  # pvc 模板，对应之前的 pv
  volumeClaimTemplates:
    - metadata:
        name: jenkins-home
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Ti
```

jenkins 部署时需要注意它的副本数，你的副本数有多少就要有多少个 pv，同样，存储会有多倍消耗。这里我只使用了一个副本，因此前面也只创建了一个 pv。

使用如下命令安装Jenkins。

```bash
kubectl apply -f jenkins-statefulset.yaml 
```

### 6.创建Service

创建jenkins-service.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  # type: LoadBalancer
  selector:
    name: jenkins
  # ensure the client ip is propagated to avoid the invalid crumb issue when using LoadBalancer (k8s >=1.7)
  #externalTrafficPolicy: Local
  ports:
    - name: http
      port: 80
      nodePort: 31888
      targetPort: 8080
      protocol: TCP
    - name: jenkins-agent
      port: 32100
      nodePort: 32100
      targetPort: 32100
      protocol: TCP
  type: NodePort
```

使用如下命令安装Service。

```bash
kubectl apply -f jenkins-service.yaml 
```

### 7.安装 ingress

jenkins 的 web 界面需要从集群外访问，这里我们选择的是使用 ingress。创建jenkins-ingress.yaml文件，文件内容如下所示。

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins
spec:
  rules:
    - http:
        paths:
          - path: /
            backend:
              serviceName: jenkins
              servicePort: 31888
      host: jekins.binghe.com
```

**这里，需要注意的是host必须配置为域名或者主机名，否则会报错，如下所示。**

```bash
The Ingress "jenkins" is invalid: spec.rules[0].host: Invalid value: "192.168.175.101": must be a DNS name, not an IP address
```

使用如下命令安装ingress。

```bash
kubectl apply -f jenkins-ingress.yaml 
```

最后，由于我这里使用的是虚拟机来搭建相关的环境，在本机访问虚拟机映射的jekins.binghe.com时，需要配置本机的hosts文件，在本机的hosts文件中加入如下配置项。

```bash
192.168.175.101 jekins.binghe.com
```

注意：在Windows操作系统中，hosts文件所在的目录如下。

```bash
C:\Windows\System32\drivers\etc
```

接下来，就可以在浏览器中通过链接：[http://jekins.binghe.com:31888](http://jekins.binghe.com:31888/) 来访问Jekins了。