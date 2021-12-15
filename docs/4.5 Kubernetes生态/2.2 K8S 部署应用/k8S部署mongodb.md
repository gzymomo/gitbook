[kubernetes生产实践之mongodb](https://www.cnblogs.com/scofield666/p/14525074.html)

### 简介

先看下生命周期图
 ![img](https://img2020.cnblogs.com/blog/2156744/202103/2156744-20210312170858201-2005742240.png)
 kubedb支持的mongodb版本

```
[root@qd01-stop-k8s-master001 mysql]# kubectl get mongodbversions
NAME             VERSION   DB_IMAGE                                 DEPRECATED   AGE
3.4.17-v1        3.4.17    kubedb/mongo:3.4.17-v1                                46h
3.4.22-v1        3.4.22    kubedb/mongo:3.4.22-v1                                46h
3.6.13-v1        3.6.13    kubedb/mongo:3.6.13-v1                                46h
3.6.18-percona   3.6.18    percona/percona-server-mongodb:3.6.18       46h
3.6.8-v1         3.6.8     kubedb/mongo:3.6.8-v1                                 46h
4.0.10-percona   4.0.10    percona/percona-server-mongodb:4.0.10       46h
4.0.11-v1        4.0.11    kubedb/mongo:4.0.11-v1                                46h
4.0.3-v1         4.0.3     kubedb/mongo:4.0.3-v1                                 46h
4.0.5-v3         4.0.5     kubedb/mongo:4.0.5-v3                                 46h
4.1.13-v1        4.1.13    kubedb/mongo:4.1.13-v1                                46h
4.1.4-v1         4.1.4     kubedb/mongo:4.1.4-v1                                 46h
4.1.7-v3         4.1.7     kubedb/mongo:4.1.7-v3                                 46h
4.2.3            4.2.3     kubedb/mongo:4.2.3                                        46h
4.2.7-percona    4.2.7     percona/percona-server-mongodb:4.2.7-7    46h
```

### MongoDB ReplicaSet

mongodb复制集模式架构如下图
 ![img](https://img2020.cnblogs.com/blog/2156744/202103/2156744-20210312170911239-2116243824.png)

1、定义配置文件，创建secret

```
cat mongod.conf
net:
   maxIncomingConnections: 10000

[root@qd01-stop-k8s-master001 Replication]# kubectl create secret generic -n op mg-configuration --from-file=./mongod.conf
secret/mg-configuration created

[root@qd01-stop-k8s-master001 Replication]# kubectl get secret -n op
NAME                       TYPE                                  DATA   AGE
mg-configuration      Opaque                                1      20s
```

2、定义mongod-replicaset.yaml文件

```
apiVersion: kubedb.com/v1alpha2
kind: MongoDB
metadata:
  name: mongodb-replicaset
  namespace: op
spec:
  version: "4.2.3"
  replicas: 3
  replicaSet:
    name: rs0
  configSecret:
    name: mg-configuration
  storage:
    storageClassName: "rbd"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
```

3、执行安装并查看部署结果

```
[root@qd01-stop-k8s-master001 Replication]# kubectl apply -f mongod-replicaset.yaml
mongodb.kubedb.com/mongodb-replicaset created

[root@qd01-stop-k8s-master001 Replication]# kubectl get po -n op
NAME                   READY   STATUS    RESTARTS   AGE
mongodb-replicaset-0   2/2     Running   0          24m
mongodb-replicaset-1   2/2     Running   0          22m
mongodb-replicaset-2   2/2     Running   0          20m
```

4、验证集群

```
# 获取用户名和密码
[root@qd01-stop-k8s-master001 Replication]#  kubectl get secrets -n demo mgo-replicaset-auth -o jsonpath='{.data.\username}' | base64 -d
root
[root@qd01-stop-k8s-master001 Replication]#  kubectl get secrets -n demo mgo-replicaset-auth -o jsonpath='{.data.\password}' | base64 -d
123456

# 登录mongodb-replicaset-0查看集群状态，并测试读写
[root@qd01-stop-k8s-master001 Replication]# kubectl -n op  exec -ti mongodb-replicaset-0 -- /bin/bash
root@mongodb-replicaset-0:/# mongo admin -uroot -p123456
MongoDB shell version v4.2.3
connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("cc1b6a09-f407-44d8-be3b-8b32db4a10b0") }
MongoDB server version: 4.2.3
Welcome to the MongoDB shell.
......
rs0:PRIMARY>

# 查看当前PRIMARY节点
rs0:PRIMARY> rs.isMaster().primary
mongodb-replicaset-0.mongodb-replicaset-pods.op.svc.cluster.local:27017

# 读写数据
rs0:PRIMARY> use testdb
switched to db testdb
rs0:PRIMARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
rs0:PRIMARY> db.movie.insert({"name":"scofield"});
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.movie.find().pretty()
{ "_id" : ObjectId("604b17469d22a18817a5927e"), "name" : "scofield" }
```

### MongoDB Sharding

mongodb分片模式架构如下图
 ![img](https://img2020.cnblogs.com/blog/2156744/202103/2156744-20210312170930197-321616617.png)

```
shard ：每个分片包含分片数据的子集。 从MongoDB 3.6开始，分片必须作为副本集部署。
mongos：mongos充当查询路由器，在客户端应用程序和分片群集之间提供接口。
config servers：配置服务器存储群集的元数据和配置设置。 从MongoDB 3.4开始，配置服务器必须部署为副本集（CSRS）。
```

1、编写mongodb-sharding.yaml

```
apiVersion: kubedb.com/v1alpha2
kind: MongoDB
metadata:
  name: mongo-sharding
  namespace: op
spec:
  version: 4.2.3
  shardTopology:
    configServer:
      replicas: 3
      storage:
        resources:
          requests:
            storage: 1Gi
        storageClassName: rbd
    mongos:
      replicas: 2
    shard:
      replicas: 3
      shards: 2
      storage:
        resources:
          requests:
            storage: 1Gi
        storageClassName: rbd
```

2、执行部署，并查看部署状态

```
[root@qd01-stop-k8s-master001 Sharded]# kubectl apply -f mongodb-sharding.yaml
mongodb.kubedb.com/mongo-sharding created

[root@qd01-stop-k8s-master001 Sharded]# kubectl get po -n op
NAME                         READY   STATUS    RESTARTS   AGE
mongo-sharding-configsvr-0   1/1     Running   0          27m
mongo-sharding-configsvr-1   1/1     Running   0          12m
mongo-sharding-configsvr-2   1/1     Running   0          9m54s
mongo-sharding-mongos-0      1/1     Running   0          7m27s
mongo-sharding-mongos-1      1/1     Running   0          4m23s
mongo-sharding-shard0-0      1/1     Running   0          27m
mongo-sharding-shard0-1      1/1     Running   0          25m
mongo-sharding-shard0-2      1/1     Running   0          24m
mongo-sharding-shard1-0      1/1     Running   0          27m
mongo-sharding-shard1-1      1/1     Running   0          25m
mongo-sharding-shard1-2      1/1     Running   0          22m
```

和复制集相比，分片集需要更多的资源。以保证整个集群的健壮性，而且能后提供更大容量的存储空间。

3、验证集群状态及读写

```
# 获取账号密码
kubectl get secrets -n demo mongo-sh-auth -o jsonpath='{.data.\username}' | base64 -d
root
kubectl get secrets -n demo mongo-sh-auth -o jsonpath='{.data.\password}' | base64 -d
123456

# 连接mongo
root@mongo-sharding-mongos-0:/# mongo admin -u root -p
MongoDB shell version v4.2.3
Enter password: 
connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("948eadef-87b6-4ab2-ba5a-c8f4e23689a7") }
MongoDB server version: 4.2.3
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
Server has startup warnings:
2021-03-12T08:10:48.173+0000 I  CONTROL  [main] ** WARNING: You are running this process as the root user, which is not recommended.
2021-03-12T08:10:48.173+0000 I  CONTROL  [main]
mongos>

# 查看分片集群状态
mongos> sh.status()
--- Sharding Status ---
  sharding version: {
        "_id" : 1,
        "minCompatibleVersion" : 5,
        "currentVersion" : 6,
        "clusterId" : ObjectId("604b201f7fb058e04bb03ef0")
  }
  shards:
        {  "_id" : "shard0",  "host" : "shard0/mongo-sharding-shard0-0.mongo-sharding-shard0-pods.op.svc.cluster.local:27017,mongo-sharding-shard0-1.mongo-sharding-shard0-pods.op.svc.cluster.local:27017,mongo-sharding-shard0-2.mongo-sharding-shard0-pods.op.svc.cluster.local:27017",  "state" : 1 }
        {  "_id" : "shard1",  "host" : "shard1/mongo-sharding-shard1-0.mongo-sharding-shard1-pods.op.svc.cluster.local:27017,mongo-sharding-shard1-1.mongo-sharding-shard1-pods.op.svc.cluster.local:27017,mongo-sharding-shard1-2.mongo-sharding-shard1-pods.op.svc.cluster.local:27017",  "state" : 1 }
  active mongoses:
        "4.2.3" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                shard0  1
                        { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard0 Timestamp(1, 0)

#创建分片
mongos> sh.enableSharding("test");
{
        "ok" : 1,
        "operationTime" : Timestamp(1615538870, 3),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1615538870, 3),
                "signature" : {
                        "hash" : BinData(0,"CTnuwtgOIn+ke0qqarLQIi+VXz8="),
                        "keyId" : NumberLong("6938674968410456068")
                }
        }
}
# 创建集合
mongos> sh.shardCollection("test.testcoll", {"myfield": 1});
{
        "collectionsharded" : "test.testcoll",
        "collectionUUID" : UUID("313cacbe-014c-4e0a-9112-427de2351bdd"),
        "ok" : 1,
        "operationTime" : Timestamp(1615539067, 9),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1615539067, 9),
                "signature" : {
                        "hash" : BinData(0,"1RQN94R6yHvUa0SqBHkiV2hUXNM="),
                        "keyId" : NumberLong("6938674968410456068")
                }
        }
}
# 写入数据
mongos> db.testcoll.insert({"myfield": "scofield", "agefield": "18"});
WriteResult({ "nInserted" : 1 })
mongos> db.testcoll.insert({"myfield": "amos", "otherfield": "d", "kube" : "db" });
WriteResult({ "nInserted" : 1 })

# 获取数据
mongos> db.testcoll.find();
{ "_id" : ObjectId("604b2d099446ca80f20bab7f"), "myfield" : "scofield", "agefield" : "18" }
{ "_id" : ObjectId("604b2d4d9446ca80f20bab80"), "myfield" : "amos", "otherfield" : "d", "kube" : "db" }
```