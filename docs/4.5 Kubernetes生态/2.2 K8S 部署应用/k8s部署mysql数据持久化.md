- [k8s部署mysql数据持久化](https://www.cnblogs.com/pluto-charon/p/14411780.html)

- [kubernetes生产实践之mysql](https://www.cnblogs.com/scofield666/p/14516923.html)

在这里我部署mysql的目的是为了后面将上一篇博客docker打包的el-admin镜像部署到k8s上，所以本文主要是部署mysql并实现持久化。

1.将我们的应用都部署到 el-admin 这个命名空间下面，创建`eladmin-namespace.yaml` 文件

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: el-admin
```

2.创建存储文件路径

```shell
[root@m ~]# mkdir -p /nfsdata/mysql
# 授权
[root@m ~]# chmod -R 777 /nfsdata/mysql
# m节点上修改文件
[root@m ~]# vi /etc/exports
/nfsdata *(rw,sync,no_root_squash)
#  m节点上重新挂载
[root@m mysql]# exportfs -r
# m节点上启动
[root@m ~]# systemctl start rpcbind && systemctl enable rpcbind
[root@m ~]# systemctl start nfs && systemctl enable nfs
# 其他节点上启动
[root@w1 ~]# systemctl start nfs
# m节点上查看
[root@m ~]# showmount -e 
Export list for m:
/nfsdata         *
```

3.编写el-admin-mysql.yaml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: el-admin-mysql-rc
  namespace: el-admin
  labels:
    name: el-admin-mysql-rc
spec:
  replicas: 1
  selector:
    name: el-admin-mysql-rc
  template:
    metadata:
      labels: 
        name: el-admin-mysql-rc
    spec:
      containers:
      - name: mysql
        image: mysql
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql          #MySQL容器的数据都是存在这个目录的，要对这个目录做数据持久化
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: el-admin-mysql-pvc       #指定pvc的名称
 
---
apiVersion: v1
kind: Service
metadata:
  name: el-admin-mysql-svc
  namespace: el-admin
  labels: 
    name: el-admin-mysql-svc
spec:
  type: NodePort
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
    name: http
    nodePort: 3306
  selector:
    name: el-admin-mysql-rc

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: el-admin-mysql-ingress
  namespace: el-admin
spec:
  rules:
  - host: eladmin.charon.com
    http:
      paths:
      - path: / 
        backend:
          serviceName: el-admin-mysql-svc
          servicePort: 3306
          
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: el-admin-mysql-pv
  namespace: el-admin
spec:
  capacity:
    storage: 2Gi 
  accessModes:
  - ReadWriteMany 
  persistentVolumeReclaimPolicy: Recycle 
  storageClassName: nfs
  nfs: 
    path: /nfsdata/mysql
    server: 192.168.189.153
  
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: el-admin-mysql-pvc
  namespace: el-admin
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs  
  resources:
    requests:
      storage: 2Gi
```

4.创建mysql的服务

```yaml
[root@m el-admin]# kubectl create -f el-admin-mysql.yaml 
```

5.查看pod，因为是在el-admin的命名空间下，所以查询pod的时候需要指定命名空间

```shell
[root@m ~]# kubectl get pods -n el-admin -o wide
NAME                      READY   STATUS    RESTARTS   AGE    IP                NODE   NOMINATED NODE   READINESS GATES
el-admin-mysql-rc-9p7wf   1/1     Running   1          151m   192.168.190.124   w1     <none>           <none>
```

从上图可以看到，mysql的pod是在w1节点上，即192.168.189.155这个节点。这个时候我们使用navicat连接mysql，肯定是连接不成功的，如下图所示：

![img](https://img2020.cnblogs.com/blog/1459011/202102/1459011-20210218144601745-674730311.png)

6.进入容器

```shell
[root@m el-admin]# kubectl exec -it el-admin-mysql-rc-9p7wf /bin/bash
# 登录mysql
root@el-admin-mysql-rc-9p7wf:/# mysql -u root -p
# 输入密码,密码为上门pod中配置的root
```

查询mysql的用户，

```sql
mysql> select host,user,plugin,authentication_string from mysql.user;
```

![img](https://img2020.cnblogs.com/blog/1459011/202102/1459011-20210218144625637-910992293.png)

为用端口为'%'用户为root的用户密码设置为root。

```sql
mysql> alter user 'root'@'%' identified with mysql_native_password by'root';
```

设置完成后，重新连接,即可连接成功。

![img](https://img2020.cnblogs.com/blog/1459011/202102/1459011-20210218144642375-975168137.png)

上面的文档里配置了ingress，ingress可以配置提供外部可访问的URL。

```shell
 # 修改windows的host文件，目录：C:\Windows\System32\drivers\etc
 # 添加内容
 192.168.189.155 eladmin.charon.com
```

使用`eladmin.charon.com`这个域名也可以连接成功

![img](https://img2020.cnblogs.com/blog/1459011/202102/1459011-20210218144703106-339485844.png)

连接成功后，在navicat里面创建一个eladmin的数据库，在k8s上的mysql挂载目录下查看新建的eladmin的数据库目录。

![img](https://img2020.cnblogs.com/blog/1459011/202102/1459011-20210218144719009-1560552968.png)

到这里，k8s部署mysql，并将数据持久化到宿主机上就完成了。

参考文件：

kubernetes部署mysql：https://www.cnblogs.com/zoulixiang/p/9910337.html