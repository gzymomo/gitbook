- [k8s微服务的性能测试监控平台搭建](https://www.cnblogs.com/congyiwei/p/14272703.html)
- [容器编排系统K8s之Prometheus监控系统+Grafana部署](https://www.cnblogs.com/qiuhom-1874/p/14287942.html)
- [Prometheus+Grafana+Alertmanager搭建全方位的监控告警系统](https://www.cnblogs.com/yrxing/p/14431089.html)
- [k8s监控体系搭建prometheus+grafana+alertmanager无坑版](https://blog.51cto.com/luoguoling/2966209)



K8S的监控我们所有的操作都要在master下进行。



# 一、部署Grafana

## 1.1 修改配置文件

```yaml
spec:
      containers:
      - name: grafana　　　　#镜像版本号
        image: grafana/grafana:7.2.1
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_USER　　　　　　# 登录账号
          value: admin
        - name: GF_SECURITY_ADMIN_PASSWORD　　　　　　#登录密码
          value: admin123
        volumeMounts:
        - mountPath: /var/lib/grafana/abc
          name: storage
      volumes:
      - name: storage
        nfs:　　　　　　#master的 ipv4地址
          server: 192.0.0.1
          path: /root/nfs-share
```

## 1.2 部署Grafana

创建garafana pod

```bash
kubectl create -f /root/k8s/node_exporter.yaml
```



# 二、部署mysqld_exporter

由于mysql_exporter是对mysql数据库进行监控，我们需要把mysql_exporter和mysql数据库打包在一个pod中，所以要对项目原有的mysql yaml文件进行update。

```yaml
- name: mysql-exporter
        env:
        - name: DATA_SOURCE_NAME　　　　　　# 数据库账号:密码@（地址：端口）
          value: root:123@(127.0.0.1:3306)/
        image: prom/mysqld-exporter
        imagePullPolicy: Always
        name: mysql-exporter
        ports:
        - containerPort: 9104
          protocol: TCP
      volumes:
      - name: mysql-data
        nfs:　　　　　　#修改为master的ipv4地址
          server: 192.168.19.133
          path: /root/nfs-share
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    nodePort: 30306
    targetPort: 3306
    name: mysql
  - port: 9104
    protocol: TCP
    targetPort: 9104
    nodePort: 30304
    name: mysql-exporter
  selector:
    name: mysql
```

上面的为新增内容（有部分会与当前已有的重复），新增后重建pod。

如果有多个节点请在replicas ： 后面增加节点数



# 三、部署node_exporter

```yaml
apiVersion: apps/v1# DaemonSet 方式会在所有绑定master的节点下安装
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    k8s-app: node-exporter
spec:
  selector:
    matchLabels:
      k8s-app: node-exporter
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter
```

创建node_exporter pod：

```bash
kubectl create -f /root/k8s/node_exporter.yaml
```

# 四、部署Prometheus

## 4.1 修改configmap.yaml文件

```yaml
- job_name: k8s-nodes
      static_configs:
      - targets:
        - 192.168.1.180:31672　　　　　# master 节点ip
        - 192.168.1.181:31672        # node1 节点ip
        - 192.168.1.182:31672        # node2 节点ip
    - job_name: mysql
      static_configs:
      - targets:
        - 192.168.1.180:30304　　　　　# master 节点ip
```



# 五、新增节点监控操作

如果集群中新增一个节点，此时我们的监控已经完成，我们应该如何去操作

1.当一个新的节点新增到集群中，node_exporter会自动在新的节点下创建一个pod，所以这里不需要额外操作

2.需要对Prometheus的配置文件进行uodate：

　　修改配置文件configmap.yaml：

```yaml
- job_name: k8s-nodes
      static_configs:
      - targets:
        - 192.168.1.180:31672
        - 192.168.1.181:31672
        - 192.168.1.182:31672
        # 增加新的节点地址
        - 192.168.1.183:31672
```

　然后执行下面的操作：

```
kubectl replace -f configmap.yaml #替换配置文件
kubectl delete -f prometheus.deploy.yml#删除服务
kubectl create -f prometheus.deploy.yml #重建服务
```