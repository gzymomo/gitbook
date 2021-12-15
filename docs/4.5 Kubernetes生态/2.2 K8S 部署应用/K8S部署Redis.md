K8S部署Redis服务

- [kubernetes生产实践之redis-cluster](https://www.cnblogs.com/scofield666/p/14513024.html)

# 一、编写yaml文件

redis-controller.yaml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis
  labels:
    name: redis
spec:
  replicas: 1  #副本数为1
  selector:
    name: redis
  template:   #模板
    metadata:
      name: redis
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: redis:latest
        imagePullPolicy: IfNotPresent  #镜像拉取策略
        ports:
        - containerPort: 6379   #容器端口
```

使用如下命令创建redis的ReplicationController控制器

```bash
kubectl create -f redis-controller.yaml
```

　　

查看是否创建成功：

```bash
kubectl get rc
```

　　

![img](https://p3-tt-ipv6.byteimg.com/img/pgc-image/54d2b105c43540438e8d682400dee744~tplv-tt-shrink:640:0.image)

 

查看创建的Pod：

```bash
kubectl get pods
```

　　

![img](https://p9-tt-ipv6.byteimg.com/img/pgc-image/5de372fe7285491397cbf2d821159f1e~tplv-tt-shrink:640:0.image)



# 二、暴露为外部服务

redis-svc.yaml文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    name: redis  #选择的Pod标签
  ports:
  - port: 6379  #暴露端口号
    targetPort: 6379  #服务端口号
```

使用如下命令创建redis的Service：

```bash
kubectl create -f redis-svc.yaml
```

　　

查看是否创建成功：

```bash
kubectl get svc
```

　　

![img](https://p9-tt-ipv6.byteimg.com/img/pgc-image/a10d82bba76a4c0bac697389de6b090e~tplv-tt-shrink:640:0.image)

 

如上图所知，当前redis被分配的IP为

```
10.109.56.243
```



