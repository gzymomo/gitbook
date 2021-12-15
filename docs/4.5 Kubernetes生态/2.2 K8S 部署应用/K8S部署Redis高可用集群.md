[Kubernetes 上部署 Redis 高可用集群](https://blog.51cto.com/u_11451275/2842835)



### 在 Kubernetes 中部署 Redis 集群

在Kubernetes中部署Redis集群面临挑战，因为每个 Redis 实例都依赖于一个配置文件，该文件可以跟踪其他集群实例及其角色。为此，我们需要结合使用Kubernetes StatefulSets和PersistentVolumes。

### 克隆部署文件

```html
git clone https://github.com/llmgo/redis-sts.git1.
```

### 创建 statefulset 类型资源

```html
[root@node01 redis-sts]# cat redis-sts.yml 
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-cluster
data:
  update-node.sh: |
    #!/bin/sh
    REDIS_NODES="/data/nodes.conf"
    sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" ${REDIS_NODES}
    exec "$@"
  redis.conf: |+
    cluster-enabled yes
    cluster-require-full-coverage no
    cluster-node-timeout 15000
    cluster-config-file /data/nodes.conf
    cluster-migration-barrier 1
    appendonly yes
    protected-mode no
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:5.0.5-alpine
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
      volumes:
      - name: conf
        configMap:
          name: redis-cluster
          defaultMode: 0755
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
      storageClassName: standard
$ kubectl apply -f redis-sts.yml
configmap/redis-cluster created
statefulset.apps/redis-cluster created
$ kubectl get pods -l app=redis-cluster
NAME              READY   STATUS    RESTARTS   AGE
redis-cluster-0   1/1     Running   0          53s
redis-cluster-1   1/1     Running   0          49s
redis-cluster-2   1/1     Running   0          46s
redis-cluster-3   1/1     Running   0          42s
redis-cluster-4   1/1     Running   0          38s
redis-cluster-5   1/1     Running   0          34s1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.45.46.47.48.49.50.51.52.53.54.55.56.57.58.59.60.61.62.63.64.65.66.67.68.69.70.71.72.73.74.75.76.77.78.79.80.81.82.
```

### 创建 service

```html
[root@node01 redis-sts]# cat redis-svc.yml   
---  
apiVersion: v1  
kind: Service  
metadata:  
  name: redis-cluster  
spec:  
  type: ClusterIP  
  clusterIP: 10.96.0.100  
  ports:  
  - port: 6379  
    targetPort: 6379  
    name: client  
  - port: 16379  
    targetPort: 16379  
    name: gossip  
  selector:  
    app: redis-cluster  
$ kubectl apply -f redis-svc.yml  
service/redis-cluster created  
$ kubectl get svc redis-cluster  
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)              AGE  
redis-cluster   ClusterIP   10.96.0.1001.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.
```

### 初始化 redis cluster

> “
>
> 下一步是形成Redis集群。为此，我们运行以下命令并键入yes以接受配置。前三个节点成为主节点，后三个节点成为从节点。

```html
$ kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')  
  
>>> Performing hash slots allocation on 6 nodes...  
Master[0] -> Slots 0 - 5460  
Master[1] -> Slots 5461 - 10922  
Master[2] -> Slots 10923 - 16383  
Adding replica 10.244.2.11:6379 to 10.244.9.19:6379  
Adding replica 10.244.9.20:6379 to 10.244.6.10:6379  
Adding replica 10.244.8.15:6379 to 10.244.7.8:6379  
M: 00721c43db194c8f2cacbafd01fd2be6a2fede28 10.244.9.19:6379  
   slots:[0-5460] (5461 slots) master  
M: 9c36053912dec8cb20a599bda202a654f241484f 10.244.6.10:6379  
   slots:[5461-10922] (5462 slots) master  
M: 2850f24ea6367de58fb50e632fc56fe4ba5ef016 10.244.7.8:6379  
   slots:[10923-16383] (5461 slots) master  
S: 554a58762e3dce23ca5a75886d0ccebd2d582502 10.244.8.15:6379  
   replicates 2850f24ea6367de58fb50e632fc56fe4ba5ef016  
S: 20028fd0b79045489824eda71fac9898f17af896 10.244.2.11:6379  
   replicates 00721c43db194c8f2cacbafd01fd2be6a2fede28  
S: 87e8987e314e4e5d4736e5818651abc1ed6ddcd9 10.244.9.20:6379  
   replicates 9c36053912dec8cb20a599bda202a654f241484f  
Can I set the above configuration? (type 'yes' to accept): yes  
>>> Nodes configuration updated  
>>> Assign a different config epoch to each node  
>>> Sending CLUSTER MEET messages to join the cluster  
Waiting for the cluster to join  
...  
>>> Performing Cluster Check (using node 10.244.9.19:6379)  
M: 00721c43db194c8f2cacbafd01fd2be6a2fede28 10.244.9.19:6379  
   slots:[0-5460] (5461 slots) master  
   1 additional replica(s)  
M: 9c36053912dec8cb20a599bda202a654f241484f 10.244.6.10:6379  
   slots:[5461-10922] (5462 slots) master  
   1 additional replica(s)  
S: 87e8987e314e4e5d4736e5818651abc1ed6ddcd9 10.244.9.20:6379  
   slots: (0 slots) slave  
   replicates 9c36053912dec8cb20a599bda202a654f241484f  
S: 554a58762e3dce23ca5a75886d0ccebd2d582502 10.244.8.15:6379  
   slots: (0 slots) slave  
   replicates 2850f24ea6367de58fb50e632fc56fe4ba5ef016  
S: 20028fd0b79045489824eda71fac9898f17af896 10.244.2.11:6379  
   slots: (0 slots) slave  
   replicates 00721c43db194c8f2cacbafd01fd2be6a2fede28  
M: 2850f24ea6367de58fb50e632fc56fe4ba5ef016 10.244.7.8:6379  
   slots:[10923-16383] (5461 slots) master  
   1 additional replica(s)  
[OK] All nodes agree about slots configuration.  
>>> Check for open slots...  
>>> Check slots coverage...  
[OK] All 16384 slots covered.1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.45.46.47.48.49.50.
```

### 验证集群

```html
[root@node01 redis-sts]# kubectl exec -it redis-cluster-0 -- redis-cli cluster info  
cluster_state:ok  
cluster_slots_assigned:16384  
cluster_slots_ok:16384  
cluster_slots_pfail:0  
cluster_slots_fail:0  
cluster_known_nodes:6  
cluster_size:3  
cluster_current_epoch:6  
cluster_my_epoch:1  
cluster_stats_messages_ping_sent:16  
cluster_stats_messages_pong_sent:22  
cluster_stats_messages_sent:38  
cluster_stats_messages_ping_received:17  
cluster_stats_messages_pong_received:16  
cluster_stats_messages_meet_received:5  
cluster_stats_messages_received:38  
[root@node01 redis-sts]# for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli role; echo; done  
redis-cluster-0  
master  
14  
10.244.2.11  
6379  
14  
  
redis-cluster-1  
master  
28  
10.244.9.20  
6379  
28  
  
redis-cluster-2  
master  
28  
10.244.8.15  
6379  
28  
  
redis-cluster-3  
slave  
10.244.7.8  
6379  
connected  
28  
  
redis-cluster-4  
slave  
10.244.9.19  
6379  
connected  
14  
  
redis-cluster-5  
slave  
10.244.6.10  
6379  
connected  
281.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.45.46.47.48.49.50.51.52.53.54.55.56.57.58.59.
```

### 测试集群

我们想使用集群，然后模拟节点的故障。对于前一项任务，我们将部署一个简单的 Python 应用程序，而对于后者，我们将删除一个节点并观察集群行为。

### 部署点击计数器应用

我们将一个简单的应用程序部署到集群中，并在其前面放置一个负载平衡器。此应用程序的目的是在将计数器值作为 HTTP 响应返回之前，增加计数器并将其存储在 Redis 集群中。

```html
$ kubectl apply -f app-deployment-service.yml  
service/hit-counter-lb created  
deployment.apps/hit-counter-app created1.2.3.
```

在此过程中，如果我们继续加载页面，计数器将继续增加，并且在删除Pod之后，我们看到没有数据丢失。

```html
$  curl `kubectl get svc hit-counter-lb -o json|jq -r .spec.clusterIP`  
I have been hit 20 times since deployment.  
$  curl `kubectl get svc hit-counter-lb -o json|jq -r .spec.clusterIP`  
I have been hit 21 times since deployment.  
$ curl `kubectl get svc hit-counter-lb -o json|jq -r .spec.clusterIP`  
I have been hit 22 times since deployment.  
$ kubectl delete pods redis-cluster-0  
pod "redis-cluster-0" deleted  
$ kubectl delete pods redis-cluster-1  
pod "redis-cluster-1" deleted  
$  curl `kubectl get svc hit-counter-lb -o json|jq -r .spec.clusterIP`  
I have been hit 23 times since deployment.1.2.3.4.5.6.7.8.9.10.11.12.
```

### 参考文档

1. [redis-cluster集群配置1] http://llmgo.cn/post/redis-003/
2. [redis-cluster集群配置2] http://www.yunweipai.com/36338.html
3. [Redis集群 cluster-slot操作指令] https://blog.csdn.net/shuyun123456789/article/details/60868718