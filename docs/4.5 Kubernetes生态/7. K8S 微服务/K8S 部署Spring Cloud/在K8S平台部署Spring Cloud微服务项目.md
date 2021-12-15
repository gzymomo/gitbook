- [在K8S平台部署Spring Cloud微服务项目](https://cloud.tencent.com/developer/article/1602058)

# 1.熟悉Spring Cloud微服务项目

代码分支说明：

- dev1交付代码
- dev2 编写Dockerfile构建镜像
- dev3 K8S资源编排
- dev4 微服务链路监控
- master 最终上线

# 2.在K8S中部署Spring Cloud微服务项目的逻辑架构

![image-20210910141714477](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141714477.png)

整体逻辑架构图

![image-20210910141741734](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141741734.png)

服务暴露的关系图

# 3.准备环境

一套k8s集群，单master或者多master都可以

| IP             | 角色          | 配置 |
| :------------- | :------------ | :--- |
| 192.168.73.138 | master        | 2C4G |
| 192.168.73.139 | node1         | 2C4G |
| 192.168.73.140 | node2         | 2C4G |
| 192.168.73.137 | harbor、mysql | 1C2G |

- CoreDNS
- Ingress Controller （可参考之前的k8s搭建文件）
- 准备mariadb[数据库](https://cloud.tencent.com/solution/database?from=10680)

在k8s外部的192.168.73.137上安装mariadb

```javascript
[root@localhost ~]# yum -y install mariadb mariadb-server
[root@localhost ~]# systemctl start mariadb && systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@localhost ~]# mysql_secure_installation ###初始化数据库，除了设置密码为123456，其他都回车即可
参考文章：https://www.jianshu.com/p/85ad52c88399
```

# 4.源代码构建

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的[项目管理](https://cloud.tencent.com/product/coding-pm?from=10680)工具软件

## 安装jdk和maven环境

```javascript
[root@k8s-master ~]# yum install java-1.8.0-openjdk maven -y
[root@k8s-master ~]# java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
[root@k8s-master ~]# mvn -version
Apache Maven 3.0.5 (Red Hat 3.0.5-17)
Maven home: /usr/share/maven
Java version: 1.8.0_242, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.el7.x86_64", arch: "amd64", family: "unix"
```

## 设置maven的国内镜像源

/etc/maven/settings.xml

```javascript
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

## 拉取代码

```javascript
[root@k8s-master ~]# cd /opt
[root@k8s-master opt]# git clone https://github.com/lizhenliang/simple-microservice.git
[root@k8s-master opt]# cd simple-microservice/
[root@k8s-master simple-microservice]# ls
basic-common  eureka-service   k8s      lombok.config  pom.xml         product-service  stock-service
db            gateway-service  LICENSE  order-service  portal-service  README.md
```

## 代码编译构建测试

```javascript
[root@k8s-master simple-microservice]# mvn clean package -Dmaven.test.skip=true
..............
[INFO] Reactor Summary:
[INFO]
[INFO] simple-microservice ............................... SUCCESS [0.164s]
[INFO] basic-common ...................................... SUCCESS [0.008s]
[INFO] basic-common-core ................................. SUCCESS [4.749s]
[INFO] gateway-service ................................... SUCCESS [5:54.575s]
[INFO] eureka-service .................................... SUCCESS [29.727s]
[INFO] product-service ................................... SUCCESS [0.005s]
[INFO] product-service-api ............................... SUCCESS [0.987s]
[INFO] stock-service ..................................... SUCCESS [0.003s]
[INFO] stock-service-api ................................. SUCCESS [0.808s]
[INFO] product-service-biz ............................... SUCCESS [11.105s]
[INFO] stock-service-biz ................................. SUCCESS [1.290s]
[INFO] order-service ..................................... SUCCESS [0.002s]
[INFO] order-service-api ................................. SUCCESS [1.319s]
[INFO] order-service-biz ................................. SUCCESS [2.531s]
[INFO] basic-common-bom .................................. SUCCESS [0.002s]
[INFO] portal-service .................................... SUCCESS [3.707s]
..............
```

# 5.构建项目镜像推送到镜像仓库

该项目的每一个服务下都有一个Dockerfile文件，可以通过Dockerfile来对项目镜像进行构建,我们可以进入网关服务gateway-service目录，查看一下该文件，本demo项目中已经写好，可以直接进行镜像构建，然后打包推送到镜像仓库

```javascript
[root@k8s-master simple-microservice]# cd gateway-service/
[root@k8s-master gateway-service]# ls
Dockerfile  pinpoint  pom.xml  src  target
[root@k8s-master gateway-service]# cat Dockerfile
FROM lizhenliang/java:8-jdk-alpine
LABEL maintainer www.ctnrs.com
ENV JAVA_ARGS="-Dfile.encoding=UTF8 -Duser.timezone=GMT+08"
COPY ./target/gateway-service.jar ./
COPY pinpoint /pinpoint
EXPOSE 9999
CMD java -jar -javaagent:/pinpoint/pinpoint-bootstrap-1.8.3.jar -Dpinpoint.agentId=$(echo $HOSTNAME | awk -F- '{print "gateway-"$NF}') -Dpinpoint.applicationName=ms-gateway $JAVA_ARGS $JAVA_OPTS /gateway-service.jar
```

返回上一层到，进入k8s目录，可以看到一个构建镜像推送到镜像仓库的脚本，循环的对每一个项目进行镜像打包，然后推送到harbor镜像仓库，仓库地址需要修改成自己地址，然后去harbor镜像仓库中创建一个microservice项目

```javascript
[root@k8s-master gateway-service]# cd ..
[root@k8s-master simple-microservice]# cd k8s/
[root@k8s-master k8s]# ls
docker_build.sh  eureka.yaml  gateway.yaml  order.yaml  portal.yaml  product.yaml  stock.yaml
[root@k8s-master k8s]# cat docker_build.sh
#!/bin/bash
kubectl create ns ms
docker_registry=192.168.73.137
kubectl create ns ms
kubectl create secret docker-registry registry-pull-secret --docker-server=$docker_registry --docker-username=admin --docker-password=Harbor12345 --docker-email=admin@ctnrs.com -n ms
service_list="eureka-service gateway-service order-service product-service stock-service portal-service"
service_list=${1:-${service_list}}
work_dir=$(dirname $PWD)
current_dir=$PWD

cd $work_dir
mvn clean package -Dmaven.test.skip=true

for service in $service_list; do
   cd $work_dir/$service
   if ls |grep biz &>/dev/null; then
      cd ${service}-biz
   fi
   service=${service%-*}
   image_name=$docker_registry/microservice/${service}:$(date +%F-%H-%M-%S)
   docker build -t ${image_name} .
   docker push ${image_name}
done
[root@k8s-master k8s]# ./docker_build.sh
```

# 6.K8S服务编排

- 网关服务gateway，使用Deployment进行pod创建，对外使用ingress暴露服务

```javascript
[root@k8s-master k8s]# cat gateway.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gateway
  namespace: ms
spec:
  rules:
    - host: gateway.ctnrs.com
      http:
        paths:
        - path: /
          backend:
            serviceName: gateway
            servicePort: 9999
---
apiVersion: v1
kind: Service
metadata:
  name: gateway
  namespace: ms
spec:
  ports:
  - port: 9999
    name: gateway
  selector:
    project: ms
    app: gateway
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      project: ms
      app: gateway
  template:
    metadata:
      labels:
        project: ms
        app: gateway
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: gateway
        image: 192.168.73.137/microservice/gateway:2020-03-08-16-40-54
        imagePullPolicy: Always
        ports:
          - protocol: TCP
            containerPort: 9999
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 9999
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 9999
          initialDelaySeconds: 60
          periodSeconds: 10
```

- 前端portal

```javascript
[root@k8s-master k8s]# cat product.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: portal
  namespace: ms
spec:
  rules:
    - host: portal.ctnrs.com
      http:
        paths:
        - path: /
          backend:
            serviceName: portal
            servicePort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: portal
  namespace: ms
spec:
  ports:
  - port: 8080
    name: portal
  selector:
    project: ms
    app: portal
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portal
  namespace: ms
spec:
  replicas: 1
  selector:
    matchLabels:
      project: ms
      app: portal
  template:
    metadata:
      labels:
        project: ms
        app: portal
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: portal
        image: 192.168.73.137/microservice/portal:2020-03-08-16-41-31
        imagePullPolicy: Always
        ports:
          - protocol: TCP
            containerPort: 8080
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
```

- 库存服务stock的yaml

```javascript
[root@k8s-master k8s]# cat stock.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stock
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      project: ms
      app: stock
  template:
    metadata:
      labels:
        project: ms
        app: stock
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: stock
        image: 192.168.73.137/microservice/stock:2020-03-08-16-41-23
        imagePullPolicy: Always
        ports:
          - protocol: TCP
            containerPort: 8030
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 8030
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8030
          initialDelaySeconds: 60
          periodSeconds: 10
```

- 订单服务order的yaml文件

```javascript
[root@k8s-master k8s]# cat order.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      project: ms
      app: order
  template:
    metadata:
      labels:
        project: ms
        app: order
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: order
        image: 192.168.73.137/microservice/order:2020-03-08-16-41-04
        imagePullPolicy: Always
        ports:
          - protocol: TCP
            containerPort: 8020
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 8020
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8020
          initialDelaySeconds: 60
          periodSeconds: 10
```

- 商品服务product的yaml文件

```javascript
[root@k8s-master k8s]# cat product.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
  namespace: ms
spec:
  replicas: 2
  selector:
    matchLabels:
      project: ms
      app: product
  template:
    metadata:
      labels:
        project: ms
        app: product
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: product
        image: 192.168.73.137/microservice/product:2020-03-08-16-41-14
        imagePullPolicy: Always
        ports:
          - protocol: TCP
            containerPort: 8010
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 8010
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8010
          initialDelaySeconds: 60
          periodSeconds: 10
```

- 注册中心eureka的yaml文件

采用的是有状态控制器StatefulSet进行部署，使用ingress控制器把service无头服务暴露出去进行访问

```javascript
[root@k8s-master k8s]# cat eureka.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: eureka
  namespace: ms
spec:
  rules:
    - host: eureka.ctnrs.com
      http:
        paths:
        - path: /
          backend:
            serviceName: eureka
            servicePort: 8888
---
apiVersion: v1
kind: Service
metadata:
  name: eureka
  namespace: ms
spec:
  clusterIP: None
  ports:
  - port: 8888
    name: eureka
  selector:
    project: ms
    app: eureka

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: eureka
  namespace: ms
spec:
  replicas: 3
  selector:
    matchLabels:
      project: ms
      app: eureka
  serviceName: "eureka"
  template:
    metadata:
      labels:
        project: ms
        app: eureka
    spec:
      imagePullSecrets:
      - name: registry-pull-secret
      containers:
      - name: eureka
        image: 192.168.73.137/microservice/eureka:2020-03-08-16-40-31
        ports:
          - protocol: TCP
            containerPort: 8888
        env:
          - name: JAVA_OPTS
            value: "-Xmx1g"
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
        resources:
          requests:
            cpu: 0.5
            memory: 256Mi
          limits:
            cpu: 1
            memory: 1Gi
        readinessProbe:
          tcpSocket:
            port: 8888
          initialDelaySeconds: 60
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 8888
          initialDelaySeconds: 60
          periodSeconds: 10
```

# 7.在K8S中部署Eureka集群（注册中心）

使用准备好yaml文件进行手动部署

> ❝注意修改，yaml中的镜像地址 ❞

```javascript
[root@k8s-master simple-microservice]# cd k8s/
[root@k8s-master k8s]# kubectl apply -f eureka.yaml
ingress.extensions/eureka created
service/eureka created
statefulset.apps/eureka created
[root@k8s-master k8s]# kubectl get pod,svc,ingress -n ms -o wide
NAME           READY   STATUS    RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
pod/eureka-0   1/1     Running   1          3h51m   172.17.10.4   192.168.73.140   <none>           <none>
pod/eureka-1   1/1     Running   0          3h47m   172.17.10.5   192.168.73.140   <none>           <none>
pod/eureka-2   1/1     Running   0          149m    172.17.42.3   192.168.73.135   <none>           <none>

NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/eureka   ClusterIP   None         <none>        8888/TCP   3h51m   app=eureka,project=ms

NAME                        HOSTS              ADDRESS   PORTS   AGE
ingress.extensions/eureka   eureka.ctnrs.com             80      3h51m
```

在本机的host文件中对eureka.ctnrs.com域名进行解析，然后通过浏览即可访问

![image-20210910141805025](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141805025.png)

注册中心

# 8. 导入数据库文件到MySQL数据库

在数据库服务器上操作：

```javascript
[root@localhost ~]# scp -r root@192.168.73.138:/opt/simple-microservice/db .
[root@localhost ~]# mysql -uroot -p
Enter password:
mysql>use mysql;
MariaDB [mysql]> update user set host = '%' where user = 'root'  and host='localhost';
Query OK, 1 row affected (0.07 sec)
Rows matched: 1  Changed: 1  Warnings: 0
MariaDB [mysql]> select host, user from user;
+-----------+------+
| host      | user |
+-----------+------+
| %         | root |
| 127.0.0.1 | root |
| ::1       | root |
+-----------+------+
3 rows in set (0.00 sec)
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)
MariaDB [(none)]> create database tb_order;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create database tb_product;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create database tb_stock;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| tb_order           |
| tb_product         |
| tb_stock           |
+--------------------+
6 rows in set (0.01 sec)

MariaDB [(none)]> use tb_order;
Database changed
MariaDB [tb_order]> source ~/db/order.sql;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

MariaDB [tb_order]> use tb_product;
Database changed
MariaDB [tb_product]> source ~/db/product.sql;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

MariaDB [tb_product]> use tb_stock;
Database changed
MariaDB [tb_stock]> source ~/db/stock.sql;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

MariaDB [tb_stock]>
```

# 9. 部署网关（gateway）

> ❝注意修改yaml中的镜像地址 ❞

```javascript
[root@k8s-master k8s]# kubectl create -f gateway.yaml
ingress.extensions/gateway created
service/gateway created
deployment.apps/gateway created

[root@k8s-master k8s]# kubectl get pod -n ms -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
eureka-0                   1/1     Running   1          4h28m   172.17.10.4   192.168.73.140   <none>           <none>
eureka-1                   1/1     Running   2          4h24m   172.17.10.5   192.168.73.140   <none>           <none>
eureka-2                   1/1     Running   0          3h6m    172.17.42.3   192.168.73.135   <none>           <none>
gateway-7877fc8867-gsb59   1/1     Running   0          9m32s   172.17.42.2   192.168.73.135   <none>           <none>
gateway-7877fc8867-rbxqz   1/1     Running   0          9m32s   172.17.5.2    192.168.73.138   <none>           <none>
```

# 10. 部署业务程序（product、stock、order）

这里面需要注意修改三个业务程序配置文件中的数据库地址和yaml中的镜像地址

```javascript
[root@k8s-master k8s]# vim ../product-service/product-service-biz/src/main/resources/application-fat.yml
spring:
  datasource:
    url: jdbc:mysql://192.168.73.137:3306/tb_product?characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka-0.eureka.ms:8888/eureka,http://eureka-1.eureka.ms:8888/eureka,http://eureka-2.eureka.ms:8888/eureka
[root@k8s-master k8s]# kubectl create -f product.yaml
deployment.apps/product created
[root@k8s-master k8s]# vim ../stock-service/stock-service-biz/src/main/resources/application-fat.yml
spring:
  datasource:
    url: jdbc:mysql://192.168.73.137:3306/tb_stock?characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka-0.eureka.ms:8888/eureka,http://eureka-1.eureka.ms:8888/eureka,http://eureka-2.eureka.ms:8888/eureka
[root@k8s-master k8s]# kubectl create -f stock.yaml
deployment.apps/stock created
[root@k8s-master k8s]# vim ../order-service/order-service-biz/src/main/resources/application-fat.yml
spring:
  datasource:
    url: jdbc:mysql://192.168.73.137:3306/tb_order?characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver

eureka:
  instance:
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka-0.eureka.ms:8888/eureka,http://eureka-1.eureka.ms:8888/eureka,http://eureka-2.eureka.ms:8888/eureka
[root@k8s-master k8s]# kubectl create -f order.yaml
deployment.apps/order created
```

- 检查pod状态

只有当所有的pod全部都是running状态才是正常，当出现其他状态的时候，需要使用r如下指令排查，适当较少pod和增加健康检查时间

- `kubectl get pod -o yaml` # 查看 Pod 配置是否正确
- `kubectl describe pod` # 查看 Pod 详细事件信息
- `kubectl logs [-c ]` # 查看容器日志

```javascript
[root@k8s-master k8s]# kubectl get pod -n ms -o wide
NAME                       READY   STATUS              RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
eureka-0                   1/1     Running             1          5h11m   172.17.10.4   192.168.73.140   <none>           <none>
eureka-1                   1/1     Running             2          7s      172.17.10.5          192.168.73.140   <none>
eureka-2                   0/1     Running             1          3m45s   172.17.42.2   192.168.73.135   <none>           <none>
gateway-7d9c68f9b9-597b8   1/1     Running             0          7m28s   172.17.42.3   192.168.73.135   <none>           <none>
product-5bb494596b-5m75d   1/1     Running             0          16s     172.17.10.3    192.168.73.140   <none>           <none>
product-845889bccb-9z982   1/1     Running             10         36m     172.17.42.4   192.168.73.135   <none>           <none>
stock-7fdb87cf6c-wsp4r     1/1     Running             9          32m     172.17.42.5   192.168.73.135   <none>           <none>
```

# 11. 部署前端（portal）

> ❝注意修改yaml中的镜像地址 ❞

```javascript
[root@k8s-master k8s]# kubectl create -f portal.yaml
ingress.extensions/portal created
service/portal created
deployment.apps/portal created
[root@k8s-master k8s]# kubectl get pod -n ms -o wide
NAME                       READY   STATUS              RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
eureka-0                   1/1     Running             1          5h11m   172.17.10.4   192.168.73.140   <none>           <none>
eureka-1                   1/1     Running             2          7s      172.17.10.5          192.168.73.140   <none>
eureka-2                   0/1     Running             1          3m45s   172.17.42.2   192.168.73.135   <none>           <none>
gateway-7d9c68f9b9-597b8   1/1     Running             0          7m28s   172.17.42.3   192.168.73.135   <none>           <none>
product-5bb494596b-5m75d   1/1     Running             0          16s     172.17.10.3    192.168.73.140   <none>           <none>
product-845889bccb-9z982   1/1     Running             10         36m     172.17.42.4   192.168.73.135   <none>           <none>
stock-7fdb87cf6c-wsp4r     1/1     Running             9          32m     172.17.42.5   192.168.73.135   <none>           <none>
portal-cc6844699-4wr9j     1/1     Running            1           46m     172.17.10.2   192.168.73.140   <none>           <none>
```

# 12. 访问前端验证功能

在本机的host文件中对portal.ctnrs.com域名进行解析，然后通过浏览即可访问

![image-20210910141841130](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141841130.png)

注册中心

![image-20210910141846896](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141846896.png)

前端

![image-20210910141852473](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141852473.png)

商品服务

![](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141852473.png)

商品服务

![image-20210910141906984](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210910141906984.png)

订单服务温馨