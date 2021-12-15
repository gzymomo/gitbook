## K8S安装gitlab代码仓库

**注意：在Master节点（binghe101服务器上执行）**

### 1.创建k8s-ops命名空间

创建k8s-ops-namespace.yaml文件，文件内容如下所示。

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: k8s-ops
  labels:
    name: k8s-ops
```

执行如下命令创建命名空间。

```bash
kubectl apply -f k8s-ops-namespace.yaml 
```

### 2.安装gitlab-redis

创建gitlab-redis.yaml文件，文件的内容如下所示。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: k8s-ops
  labels:
    name: redis
spec:
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      name: redis
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: sameersbn/redis
        imagePullPolicy: IfNotPresent
        ports:
        - name: redis
          containerPort: 6379
        volumeMounts:
        - mountPath: /var/lib/redis
          name: data
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 10
          timeoutSeconds: 5
      volumes:
      - name: data
        hostPath:
          path: /data1/docker/xinsrv/redis

---
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: k8s-ops
  labels:
    name: redis
spec:
  ports:
    - name: redis
      port: 6379
      targetPort: redis
  selector:
    name: redis
```

首先，在命令行执行如下命令创建/data1/docker/xinsrv/redis目录。

```bash
mkdir -p /data1/docker/xinsrv/redis
```

执行如下命令安装gitlab-redis。

```bash
kubectl apply -f gitlab-redis.yaml 
```

### 3.安装gitlab-postgresql

创建gitlab-postgresql.yaml，文件内容如下所示。

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
  namespace: k8s-ops
  labels:
    name: postgresql
spec:
  selector:
    matchLabels:
      name: postgresql
  template:
    metadata:
      name: postgresql
      labels:
        name: postgresql
    spec:
      containers:
      - name: postgresql
        image: sameersbn/postgresql
        imagePullPolicy: IfNotPresent
        env:
        - name: DB_USER
          value: gitlab
        - name: DB_PASS
          value: passw0rd
        - name: DB_NAME
          value: gitlab_production
        - name: DB_EXTENSION
          value: pg_trgm
        ports:
        - name: postgres
          containerPort: 5432
        volumeMounts:
        - mountPath: /var/lib/postgresql
          name: data
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -h
            - localhost
            - -U
            - postgres
          initialDelaySeconds: 30
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -h
            - localhost
            - -U
            - postgres
          initialDelaySeconds: 5
          timeoutSeconds: 1
      volumes:
      - name: data
        hostPath:
          path: /data1/docker/xinsrv/postgresql
---
apiVersion: v1
kind: Service
metadata:
  name: postgresql
  namespace: k8s-ops
  labels:
    name: postgresql
spec:
  ports:
    - name: postgres
      port: 5432
      targetPort: postgres
  selector:
    name: postgresql
```

首先，执行如下命令创建/data1/docker/xinsrv/postgresql目录。

```bash
mkdir -p /data1/docker/xinsrv/postgresql
```

接下来，安装gitlab-postgresql，如下所示。

```bash
kubectl apply -f gitlab-postgresql.yaml
```

### 4.安装gitlab

**（1）配置用户名和密码**

首先，在命令行使用base64编码为用户名和密码进行转码，本示例中，使用的用户名为admin，密码为admin.1231

转码情况如下所示。

```bash
[root@binghe101 k8s]# echo -n 'admin' | base64 
YWRtaW4=
[root@binghe101 k8s]# echo -n 'admin.1231' | base64 
YWRtaW4uMTIzMQ==
```

转码后的用户名为：YWRtaW4= 密码为：YWRtaW4uMTIzMQ==

也可以对base64编码后的字符串解码，例如，对密码字符串解码，如下所示。

```bash
[root@binghe101 k8s]# echo 'YWRtaW4uMTIzMQ==' | base64 --decode 
admin.1231
```

接下来，创建secret-gitlab.yaml文件，主要是用户来配置GitLab的用户名和密码，文件内容如下所示。

```bash
apiVersion: v1
kind: Secret
metadata:
  namespace: k8s-ops
  name: git-user-pass
type: Opaque
data:
  username: YWRtaW4=
  password: YWRtaW4uMTIzMQ==
```

执行配置文件的内容，如下所示。

```bash
kubectl create -f ./secret-gitlab.yaml
```

**（2）安装GitLab**

创建gitlab.yaml文件，文件的内容如下所示。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
  namespace: k8s-ops
  labels:
    name: gitlab
spec:
  selector:
    matchLabels:
      name: gitlab
  template:
    metadata:
      name: gitlab
      labels:
        name: gitlab
    spec:
      containers:
      - name: gitlab
        image: sameersbn/gitlab:12.1.6
        imagePullPolicy: IfNotPresent
        env:
        - name: TZ
          value: Asia/Shanghai
        - name: GITLAB_TIMEZONE
          value: Beijing
        - name: GITLAB_SECRETS_DB_KEY_BASE
          value: long-and-random-alpha-numeric-string
        - name: GITLAB_SECRETS_SECRET_KEY_BASE
          value: long-and-random-alpha-numeric-string
        - name: GITLAB_SECRETS_OTP_KEY_BASE
          value: long-and-random-alpha-numeric-string
        - name: GITLAB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: git-user-pass
              key: password
        - name: GITLAB_ROOT_EMAIL
          value: 12345678@qq.com
        - name: GITLAB_HOST
          value: gitlab.binghe.com
        - name: GITLAB_PORT
          value: "80"
        - name: GITLAB_SSH_PORT
          value: "30022"
        - name: GITLAB_NOTIFY_ON_BROKEN_BUILDS
          value: "true"
        - name: GITLAB_NOTIFY_PUSHER
          value: "false"
        - name: GITLAB_BACKUP_SCHEDULE
          value: daily
        - name: GITLAB_BACKUP_TIME
          value: 01:00
        - name: DB_TYPE
          value: postgres
        - name: DB_HOST
          value: postgresql
        - name: DB_PORT
          value: "5432"
        - name: DB_USER
          value: gitlab
        - name: DB_PASS
          value: passw0rd
        - name: DB_NAME
          value: gitlab_production
        - name: REDIS_HOST
          value: redis
        - name: REDIS_PORT
          value: "6379"
        ports:
        - name: http
          containerPort: 80
        - name: ssh
          containerPort: 22
        volumeMounts:
        - mountPath: /home/git/data
          name: data
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 180
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          timeoutSeconds: 1
      volumes:
      - name: data
        hostPath:
          path: /data1/docker/xinsrv/gitlab
---
apiVersion: v1
kind: Service
metadata:
  name: gitlab
  namespace: k8s-ops
  labels:
    name: gitlab
spec:
  ports:
    - name: http
      port: 80
      nodePort: 30088
    - name: ssh
      port: 22
      targetPort: ssh
      nodePort: 30022
  type: NodePort
  selector:
    name: gitlab

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gitlab
  namespace: k8s-ops
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: gitlab.binghe.com
    http:
      paths:
      - backend:
          serviceName: gitlab
          servicePort: http
```

**注意：在配置GitLab时，监听主机时，不能使用IP地址，需要使用主机名或者域名，上述配置中，我使用的是gitlab.binghe.com主机名。**

在命令行执行如下命令创建/data1/docker/xinsrv/gitlab目录。

```bash
mkdir -p /data1/docker/xinsrv/gitlab
```

安装GitLab，如下所示。

```bash
kubectl apply -f gitlab.yaml
```

### 5.安装完成

查看k8s-ops命名空间部署情况，如下所示。

```bash
[root@binghe101 k8s]# kubectl get pod -n k8s-ops
NAME                          READY   STATUS    RESTARTS   AGE
gitlab-7b459db47c-5vk6t       0/1     Running   0          11s
postgresql-79567459d7-x52vx   1/1     Running   0          30m
redis-67f4cdc96c-h5ckz        1/1     Running   1          10h
```

也可以使用如下命令查看。

```bash
[root@binghe101 k8s]# kubectl get pod --namespace=k8s-ops
NAME                          READY   STATUS    RESTARTS   AGE
gitlab-7b459db47c-5vk6t       0/1     Running   0          36s
postgresql-79567459d7-x52vx   1/1     Running   0          30m
redis-67f4cdc96c-h5ckz        1/1     Running   1          10h
```

二者效果一样。

接下来，查看GitLab的端口映射，如下所示。

```bash
[root@binghe101 k8s]# kubectl get svc -n k8s-ops
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                     AGE
gitlab       NodePort    10.96.153.100   <none>        80:30088/TCP,22:30022/TCP   2m42s
postgresql   ClusterIP   10.96.203.119   <none>        5432/TCP                    32m
redis        ClusterIP   10.96.107.150   <none>        6379/TCP                    10h
```

此时，可以看到，可以通过Master节点（binghe101）的主机名gitlab.binghe.com和端口30088就能够访问GitLab。由于我这里使用的是虚拟机来搭建相关的环境，在本机访问虚拟机映射的gitlab.binghe.com时，需要配置本机的hosts文件，在本机的hosts文件中加入如下配置项。

```bash
192.168.175.101 gitlab.binghe.com
```

注意：在Windows操作系统中，hosts文件所在的目录如下。

```bash
C:\Windows\System32\drivers\etc
```

接下来，就可以在浏览器中通过链接：[http://gitlab.binghe.com:30088](http://gitlab.binghe.com:30088/) 来访问GitLab了，如下所示。

![img](https://img-blog.csdnimg.cn/20200521004033845.jpg)

此时，可以通过用户名root和密码admin.1231来登录GitLab了。

**注意：这里的用户名是root而不是admin，因为root是GitLab默认的超级用户。**

![img](https://img-blog.csdnimg.cn/20200521004046470.jpg)

登录后的界面如下所示。

![img](https://img-blog.csdnimg.cn/2020052100410037.jpg)

到此，K8S安装gitlab完成。