- 语雀：渡渡鸟：[K8S项目交付-集群监控](https://www.yuque.com/duduniao)

- 博客园：散尽浮华：[Kubernetes容器集群管理环境 - Prometheus监控篇](https://www.cnblogs.com/kevingrace/p/11151649.html)



## 1. Prometheus

### 1.1. Prometheus介绍

Prometheus 是一款基于时序数据库的开源监控告警系统，非常适合Kubernetes集群的监控。Prometheus的基本原理是通过HTTP协议周期性抓取被监控组件的状态，任意组件只要提供对应的HTTP接口就可以接入监控。不需要任何SDK或者其他的集成过程。这样做非常适合做虚拟化环境监控系统，比如VM、Docker、Kubernetes等。输出被监控组件信息的HTTP接口被叫做exporter 。目前互联网公司常用的组件大部分都有exporter可以直接使用，比如Varnish、Haproxy、Nginx、MySQL、Linux系统信息(包括磁盘、内存、CPU、网络等等)。Promethus有以下特点：

- 支持多维数据模型：由度量名和键值对组成的时间序列数据
- 内置时间序列数据库TSDB
- 支持PromQL查询语言，可以完成非常复杂的查询和分析，对图表展示和告警非常有意义
- 支持HTTP的Pull方式采集时间序列数据
- 支持PushGateway采集瞬时任务的数据
- 支持服务发现和静态配置两种方式发现目标
- 支持接入Grafana

### 1.2. Prometheus架构

![image](https://cdn.nlark.com/yuque/0/2020/png/378176/1580906969562-aedc240e-278a-4f27-8c76-71c3eff5fec7.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

#### 1.2.1. Prometheus Server

主要负责数据采集和存储，提供PromQL查询语言的支持。包含了三个组件：

- Retrieval: 获取监控数据
- TSDB: 时间序列数据库(Time Series Database)，我们可以简单的理解为一个优化后用来处理时间序列数据的软件，并且数据中的数组是由时间进行索引的。具备以下特点：

- - 大部分时间都是顺序写入操作，很少涉及修改数据
  - 删除操作都是删除一段时间的数据，而不涉及到删除无规律数据
  - 读操作一般都是升序或者降序

- HTTP Server: 为告警和出图提供查询接口

#### 1.2.2. 指标采集

- Exporters: Prometheus的一类数据采集组件的总称。它负责从目标处搜集数据，并将其转化为Prometheus支持的格式。与传统的数据采集组件不同的是，它并不向中央服务器发送数据，而是等待中央服务器主动前来抓取
- Pushgateway: 支持临时性Job主动推送指标的中间网关

#### 1.2.3. 服务发现

- Kubernetes_sd: 支持从Kubernetes中自动发现服务和采集信息。而Zabbix监控项原型就不适合Kubernets，因为随着Pod的重启或者升级，Pod的名称是会随机变化的。
- file_sd: 通过配置文件来实现服务的自动发现

#### 1.2.4. 告警管理

通过相关的告警配置，对触发阈值的告警通过页面展示、短信和邮件通知的方式告知运维人员。

#### 1.2.5. 图形化展示

通过ProQL语句查询指标信息，并在页面展示。虽然Prometheus自带UI界面，但是大部分都是使用Grafana出图。另外第三方也可以通过 API 接口来获取监控指标。

### 1.3. 常用的几个Exporter

```
Kube-state-metrics: 收集Kubernetes对象的基本信息，但是不涉及Pod中资源占用信息
Node-exporter: 收集主机的基本信息，如CPU、内存、磁盘等
Cadvisor: 容器顾问，用于采集Docker中运行的容器信息，如CPU、内存等
Blackbox-exporter: 服务可用性探测，支持HTTP、HTTPS、TCP、ICMP等方式探测目标地址服务可用性
更多exporter参考官方：https://prometheus.io/docs/instrumenting/exporters/
```

### 1.4. 与Zabbix对比

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580909367250-d672f65e-10e4-47db-99a8-73bb88742512.png)

## 2. 交付Exporters

### 2.1. kube-state-metrics

#### 2.1.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull quay.io/coreos/kube-state-metrics:v1.5.0 # quay.io无法访问可采用以下方式
[root@hdss7-200 ~]# docker pull quay.mirrors.ustc.edu.cn/coreos/kube-state-metrics:v1.5.0
[root@hdss7-200 ~]# docker image tag quay.mirrors.ustc.edu.cn/coreos/kube-state-metrics:v1.5.0 harbor.od.com/public/kube-state-metrics:v1.5.0
[root@hdss7-200 ~]# docker image push harbor.od.com/public/kube-state-metrics:v1.5.0
```

#### 2.1.2. 准备资源配置清单

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  - nodes
  - pods
  - services
  - resourcequotas
  - replicationcontrollers
  - limitranges
  - persistentvolumeclaims
  - persistentvolumes
  - namespaces
  - endpoints
  verbs:
  - list
  - watch
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - daemonsets
  - deployments
  - replicasets
  verbs:
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - statefulsets
  verbs:
  - list
  - watch
- apiGroups:
  - batch
  resources:
  - cronjobs
  - jobs
  verbs:
  - list
  - watch
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: kube-system
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
  labels:
    grafanak8sapp: "true"
    app: kube-state-metrics
  name: kube-state-metrics
  namespace: kube-system
spec:
  selector:
    matchLabels:
      grafanak8sapp: "true"
      app: kube-state-metrics
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        grafanak8sapp: "true"
        app: kube-state-metrics
    spec:
      containers:
      - name: kube-state-metrics
        image: harbor.od.com/public/kube-state-metrics:v1.5.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http-metrics
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
      serviceAccountName: kube-state-metrics
```

#### 2.1.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/kube-state-metrics/rbac.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/kube-state-metrics/deployment.yaml
```



```bash
[root@hdss7-21 ~]# kubectl get pod -n kube-system -o wide -l app=kube-state-metrics
NAME                                  READY   STATUS    RESTARTS   AGE    IP           NODE                NOMINATED NODE   READINESS GATES
kube-state-metrics-8669f776c6-2f7gx   1/1     Running   0          100s   172.7.22.6   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# curl 172.7.22.6:8080/healthz  # 就绪性探测地址，可以额外对该地址添加存活性探针
ok
[root@hdss7-21 ~]# curl 172.7.22.6:8080/metrics  # Prometheus 取监控数据的接口
# HELP kube_configmap_info Information about configmap.
# TYPE kube_configmap_info gauge
kube_configmap_info{namespace="kube-system",configmap="extension-apiserver-authentication"} 1
kube_configmap_info{namespace="kube-system",configmap="kubernetes-dashboard-settings"} 1
......
```

### 2.2. node-exporter

#### 2.2.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull prom/node-exporter:v0.15.0
[root@hdss7-200 ~]# docker image tag prom/node-exporter:v0.15.0 harbor.od.com/public/node-exporter:v0.15.0
[root@hdss7-200 ~]# docker image push harbor.od.com/public/node-exporter:v0.15.0
```

#### 2.2.2. 准备资源配置清单

```yaml
# node-exporter采用daemonset类型控制器，部署在所有Node节点，且共享了宿主机网络名称空间
# 通过挂载宿主机的/proc和/sys目录采集宿主机的系统信息
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    daemon: "node-exporter"
    grafanak8sapp: "true"
spec:
  selector:
    matchLabels:
      daemon: "node-exporter"
      grafanak8sapp: "true"
  template:
    metadata:
      name: node-exporter
      labels:
        daemon: "node-exporter"
        grafanak8sapp: "true"
    spec:
      volumes:
      - name: proc
        hostPath: 
          path: /proc
          type: ""
      - name: sys
        hostPath:
          path: /sys
          type: ""
      containers:
      - name: node-exporter
        image: harbor.od.com/public/node-exporter:v0.15.0
        args:
        - --path.procfs=/host_proc
        - --path.sysfs=/host_sys
        ports:
        - name: node-exporter
          hostPort: 9100
          containerPort: 9100
          protocol: TCP
        volumeMounts:
        - name: sys
          readOnly: true
          mountPath: /host_sys
        - name: proc
          readOnly: true
          mountPath: /host_proc
      hostNetwork: true
```

#### 2.2.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/node-exporter/deamonset.yaml
[root@hdss7-21 ~]# kubectl get pod -n kube-system -l daemon="node-exporter" -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP          NODE                NOMINATED NODE   READINESS GATES
node-exporter-q4n5n   1/1     Running   0          34s   10.4.7.21   hdss7-21.host.com   <none>           <none>
node-exporter-zxhjg   1/1     Running   0          34s   10.4.7.22   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# curl -s 10.4.7.21:9100/metrics | head 
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
```

### 2.3. cadvisor

该exporter是通过和kubelet交互，取到Pod运行时的资源消耗情况，并将接口暴露给 Prometheus。

#### 2.3.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull google/cadvisor:v0.28.3
[root@hdss7-200 ~]# docker image tag google/cadvisor:v0.28.3 harbor.od.com/public/cadvisor:v0.28.3
[root@hdss7-200 ~]# docker image push harbor.od.com/public/cadvisor:v0.28.3
```

#### 2.3.2. 准备资源配置清单

```yaml
# cadvisor采用daemonset方式运行在node节点上，通过污点的方式排除master
# 同时将部分宿主机目录挂载到本地，如docker的数据目录
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: cadvisor
  namespace: kube-system
  labels:
    app: cadvisor
spec:
  selector:
    matchLabels:
      name: cadvisor
  template:
    metadata:
      labels:
        name: cadvisor
    spec:
      hostNetwork: true
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: cadvisor
        image: harbor.od.com/public/cadvisor:v0.28.3
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: rootfs
          mountPath: /rootfs
          readOnly: true
        - name: var-run
          mountPath: /var/run
        - name: sys
          mountPath: /sys
          readOnly: true
        - name: docker
          mountPath: /var/lib/docker
          readOnly: true
        ports:
          - name: http
            containerPort: 4194
            protocol: TCP
        readinessProbe:
          tcpSocket:
            port: 4194
          initialDelaySeconds: 5
          periodSeconds: 10
        args:
          - --housekeeping_interval=10s
          - --port=4194
      terminationGracePeriodSeconds: 30
      volumes:
      - name: rootfs
        hostPath:
          path: /
      - name: var-run
        hostPath:
          path: /var/run
      - name: sys
        hostPath:
          path: /sys
      - name: docker
        hostPath:
          path: /data/docker
```

#### 2.3.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# mount -o remount,rw /sys/fs/cgroup/  # 原本是只读，现在改为可读可写
[root@hdss7-21 ~]# ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu

[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/cadvisor/daemonset.yaml
[root@hdss7-21 ~]# kubectl get pod -n kube-system -l name=cadvisor -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP          NODE                NOMINATED NODE   READINESS GATES
cadvisor-wnxfx   1/1     Running   0          34s   10.4.7.21   hdss7-21.host.com   <none>           <none>
cadvisor-xwrvq   1/1     Running   0          34s   10.4.7.22   hdss7-22.host.com   <none>           <none>
[root@hdss7-21 ~]# curl -s 10.4.7.21:4194/metrics | head -n 1
# HELP cadvisor_version_info A metric with a constant '1' value labeled by kernel version, OS version, docker version, cadvisor version & cadvisor revision.
```

### 2.4. blackbox-exporter

#### 2.4.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull prom/blackbox-exporter:v0.15.1
[root@hdss7-200 ~]# docker image tag prom/blackbox-exporter:v0.15.1 harbor.od.com/public/blackbox-exporter:v0.15.1
[root@hdss7-200 ~]# docker image push harbor.od.com/public/blackbox-exporter:v0.15.1
```

#### 2.4.2. 准备资源配置清单

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
  namespace: kube-system
data:
  blackbox.yml: |-
    modules:
      http_2xx:
        prober: http
        timeout: 2s
        http:
          valid_http_versions: ["HTTP/1.1", "HTTP/2"]
          valid_status_codes: [200,301,302]
          method: GET
          preferred_ip_protocol: "ip4"
      tcp_connect:
        prober: tcp
        timeout: 2s
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blackbox-exporter
  namespace: kube-system
  labels:
    app: blackbox-exporter
  annotations:
    deployment.kubernetes.io/revision: 1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blackbox-exporter
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      volumes:
      - name: config
        configMap:
          name: blackbox-exporter
          defaultMode: 420
      containers:
      - name: blackbox-exporter
        image: harbor.od.com/public/blackbox-exporter:v0.15.1
        imagePullPolicy: IfNotPresent
        args:
        - --config.file=/etc/blackbox_exporter/blackbox.yml
        - --log.level=info
        - --web.listen-address=:9115
        ports:
        - name: blackbox-port
          containerPort: 9115
          protocol: TCP
        resources:
          limits:
            cpu: 200m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 50Mi
        volumeMounts:
        - name: config
          mountPath: /etc/blackbox_exporter
        readinessProbe:
          tcpSocket:
            port: 9115
          initialDelaySeconds: 5
          timeoutSeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
```



```yaml
# 没有指定targetPort是因为Pod中暴露端口名称为 blackbox-port
apiVersion: v1
kind: Service
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  selector:
    app: blackbox-exporter
  ports:
    - name: blackbox-port
      protocol: TCP
      port: 9115
```



```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: blackbox-exporter
  namespace: kube-system
spec:
  rules:
  - host: blackbox.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: blackbox-exporter
          servicePort: blackbox-port
```

#### 2.4.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/blackbox-exporter/configmap.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/blackbox-exporter/deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/blackbox-exporter/ingress.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/blackbox-exporter/service.yaml
```



```bash
[root@hdss7-11 ~]# vim /var/named/od.com.zone 
......
blackbox           A    10.4.7.10
[root@hdss7-11 ~]# systemctl restart named
```



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580962447393-6f7448b9-ca25-465e-91cf-8e4a48c6aa6e.png)



## 3. 交付Prometheus Server

### 3.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull prom/prometheus:v2.14.0
[root@hdss7-200 ~]# docker image tag prom/prometheus:v2.14.0 harbor.od.com/public/prometheus:v2.14.0
[root@hdss7-200 ~]# docker image push harbor.od.com/public/prometheus:v2.14.0
```

### 3.2. 准备资源配置清单

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - nodes/metrics
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system
```



```yaml
# Prometheus在生产环境中，一般采用一个单独的大内存node部署，采用污点让其它pod不会调度上来
# --storage.tsdb.min-block-duration 内存中缓存最新多少分钟的TSDB数据，生产中会缓存更多的数据
# --storage.tsdb.retention TSDB数据保留的时间，生产中会保留更多的数据
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "5"
  labels:
    name: prometheus
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      nodeName: hdss7-22.host.com
      securityContext:
        runAsUser: 0
      containers:
      - name: prometheus
        image: harbor.od.com/public/prometheus:v2.14.0
        command:
        - /bin/prometheus
        args:
        - --config.file=/data/etc/prometheus.yml
        - --storage.tsdb.path=/data/prom-db
        - --storage.tsdb.min-block-duration=10m
        - --storage.tsdb.retention=72h
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: /data
          name: data
        resources:
          requests:
            cpu: "1000m"
            memory: "1.5Gi"
          limits:
            cpu: "2000m"
            memory: "3Gi"
      serviceAccountName: prometheus
      volumes:
      - name: data
        nfs:
          server: hdss7-200
          path: /data/nfs-volume/prometheus
```



```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: kube-system
spec:
  ports:
  - port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app: prometheus
```



```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: prometheus
  namespace: kube-system
spec:
  rules:
  - host: prometheus.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus
          servicePort: 9090
```

### 3.3. 准备Prometheus配置

```bash
[root@hdss7-200 ~]# mkdir -p /data/nfs-volume/prometheus/{etc,prom-db}
[root@hdss7-200 ~]# cp /opt/certs/{ca.pem,client.pem,client-key.pem} /data/nfs-volume/prometheus/etc/
```



```yaml
[root@hdss7-200 ~]# vim /data/nfs-volume/prometheus/etc/prometheus.yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
- job_name: 'etcd'
  tls_config:
    ca_file: /data/etc/ca.pem
    cert_file: /data/etc/client.pem
    key_file: /data/etc/client-key.pem
  scheme: https
  static_configs:
  - targets:
    - '10.4.7.12:2379'
    - '10.4.7.21:2379'
    - '10.4.7.22:2379'
- job_name: 'kubernetes-apiservers'
  kubernetes_sd_configs:
  - role: endpoints
  scheme: https
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
    action: keep
    regex: default;kubernetes;https
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'kubernetes-kubelet'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:10255
- job_name: 'kubernetes-cadvisor'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:4194
- job_name: 'kubernetes-kube-state'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
  - source_labels: [__meta_kubernetes_pod_label_grafanak8sapp]
    regex: .*true.*
    action: keep
  - source_labels: ['__meta_kubernetes_pod_label_daemon', '__meta_kubernetes_pod_node_name']
    regex: 'node-exporter;(.*)'
    action: replace
    target_label: nodename
- job_name: 'blackbox_http_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [http_2xx]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: http
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port,  __meta_kubernetes_pod_annotation_blackbox_path]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+);(.+)
    replacement: $1:$2$3
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'blackbox_tcp_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [tcp_connect]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: tcp
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'traefik'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
    action: keep
    regex: traefik
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
```

### 3.4. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/prometheus-server/rbac.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/prometheus-server/deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/prometheus-server/service.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/prometheus-server/ingress.yaml
```



```bash
[root@hdss7-11 ~]# vim /var/named/od.com.zone 
......
prometheus         A    10.4.7.10
[root@hdss7-11 ~]# systemctl restart named
```



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580971862210-23604f68-d37d-483a-9b26-d071e26d8f30.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580976484095-eae4131e-13f4-45b3-973a-2c8b35429f6e.png?x-oss-process=image%2Fresize%2Cw_1500)

## 4. 部署Grafana

### 4.1. 安装Grafana

#### 4.1.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull grafana/grafana:5.4.2
[root@hdss7-200 ~]# docker image tag grafana/grafana:5.4.2 harbor.od.com/public/grafana:v5.4.2
[root@hdss7-200 ~]# docker image push harbor.od.com/public/grafana:v5.4.2
```

#### 4.1.2. 准备资源配置清单

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
rules:
- apiGroups:
  - "*"
  resources:
  - namespaces
  - deployments
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/cluster-service: "true"
  name: grafana
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: grafana
subjects:
- kind: User
  name: k8s-node
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
    name: grafana
  name: grafana
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      name: grafana
  template:
    metadata:
      labels:
        app: grafana
        name: grafana
    spec:
      containers:
      - name: grafana
        image: harbor.od.com/infra/grafana:v5.4.2
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: data
      imagePullSecrets:
      - name: harbor
      securityContext:
        runAsUser: 0
      volumes:
      - nfs:
          server: hdss7-200
          path: /data/nfs-volume/grafana
        name: data
```



```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: kube-system
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 3000
  selector:
    app: grafana
```



```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: kube-system
spec:
  rules:
  - host: grafana.od.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```

#### 4.1.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/grafana/rbac.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/grafana/deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/grafana/service.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/grafana/ingress.yaml
```



```bash
[root@hdss7-11 ~]# vim /var/named/od.com.zone 
......
grafana            A    10.4.7.10
[root@hdss7-11 ~]# systemctl restart named
[root@hdss7-11 ~]# dig -t A grafana.od.com +short
10.4.7.10
```



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580999434884-0cc6133a-14b8-46a3-b7d5-5092849fbfd1.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

#### 4.1.4. 安装插件

```bash
# 需要安装的插件
grafana-kubernetes-app
grafana-clock-panel
grafana-piechart-panel
briangann-gauge-panel
natel-discrete-panel
```



```bash
# 插件安装有两种方式：
# 1. 进入Container中，执行 grafana-cli plugins install $plugin_name
# 2. 手动下载插件zip包，访问 https://grafana.com/api/plugins/repo/$plugin_name 查询插件版本号 $version
#    通过 https://grafana.com/api/plugins/$plugin_name/versions/$version/download 下载zip包
#    将zip包解压到 /var/lib/grafana/plugins 下
# 插件安装完毕后，重启Grafana的Pod

# 方式一:
[root@hdss7-21 ~]# kubectl get pod -n kube-system -l name=grafana
NAME                       READY   STATUS    RESTARTS   AGE
grafana-596d8dbcd5-l2466   1/1     Running   0          3m45s
[root@hdss7-21 ~]# kubectl exec grafana-596d8dbcd5-l2466 -n kube-system -it -- /bin/bash
root@grafana-596d8dbcd5-l2466:/usr/share/grafana# grafana-cli plugins install grafana-kubernetes-app
root@grafana-596d8dbcd5-l2466:/usr/share/grafana# grafana-cli plugins install grafana-clock-panel
root@grafana-596d8dbcd5-l2466:/usr/share/grafana# grafana-cli plugins install grafana-piechart-panel
root@grafana-596d8dbcd5-l2466:/usr/share/grafana# grafana-cli plugins install briangann-gauge-panel
root@grafana-596d8dbcd5-l2466:/usr/share/grafana# grafana-cli plugins install natel-discrete-panel

# 方式二:
[root@hdss7-200 plugins]# wget -O grafana-kubernetes-app.zip https://grafana.com/api/plugins/grafana-kubernetes-app/versions/1.0.1/download
[root@hdss7-200 plugins]# ls *.zip | xargs -I {} unzip -q {}
```



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581042791976-1e6de6db-4ecc-470f-b4cc-6db739cb7b4d.png)

### 4.2. 接入Prometheus

#### 4.2.1. 接入Prometheus数据

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581044427340-1af58dce-6c6c-4049-a867-9476fb72fc15.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

#### 4.2.2. 配置kubernetes app

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581044890738-02dc64ab-fbfc-4fff-bf37-cfe263f9549f.png)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581044951163-d9e64a99-e082-4607-9a87-9dcee3a7239d.png?x-oss-process=image%2Fresize%2Cw_1500)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581045163025-bb589bc3-d01b-466b-881a-14a6300c66af.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

#### 4.2.3. 导入Kubernetes面板

当前的Kubernetes面板存在一些问题，可以通过导入更加合适的面板来实现图表展示。
附件:[📎GrafanaDashboard.zip](https://www.yuque.com/attachments/yuque/0/2020/zip/378176/1581083729746-0aed89e5-d47d-492f-a3d9-07218765c292.zip) [📎GrafanaDashboard.zip](https://www.yuque.com/attachments/yuque/0/2020/zip/378176/1581083773205-8b1baeec-ccbe-4822-bf97-9df934dd8000.zip)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581083924617-c918ebca-1fc4-4881-b37c-4a491790d687.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581084000383-619d4815-290f-497e-ba88-61d5ef802c7d.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



## 5. 部署alertmanager

### 5.1. 准备镜像

```bash
[root@hdss7-200 ~]# docker pull docker.io/prom/alertmanager:v0.14.0
[root@hdss7-200 ~]# docker image tag prom/alertmanager:v0.14.0 harbor.od.com/public/alertmanager:v0.14.0
[root@hdss7-200 ~]# docker push harbor.od.com/public/alertmanager:v0.14.0
# 新版本容器启动后可能会报错：
# couldn't deduce an advertise address: no private IP found, explicit advertise addr not provided
```

### 5.2. 准备资源配置清单

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: kube-system
data:
  config.yml: |-
    global:
      # 在没有报警的情况下声明为已解决的时间
      resolve_timeout: 5m
      # 配置邮件发送信息
      smtp_smarthost: 'smtp.163.com:25'
      smtp_from: 'linux_hy_1992@163.com'
      smtp_auth_username: 'linux_hy_1992@163.com'
      smtp_auth_password: 'linux1992'
      smtp_require_tls: false
    # 所有报警信息进入后的根路由，用来设置报警的分发策略
    route:
      # 这里的标签列表是接收到报警信息后的重新分组标签，例如，接收到的报警信息里面有许多具有 cluster=A 和 alertname=LatncyHigh 这样的标签的报警信息将会批量被聚合到一个分组里面
      group_by: ['alertname', 'cluster']
      # 当一个新的报警分组被创建后，需要等待至少group_wait时间来初始化通知，这种方式可以确保您能有足够的时间为同一分组来获取多个警报，然后一起触发这个报警信息。
      group_wait: 30s
      # 当第一个报警发送后，等待'group_interval'时间来发送新的一组报警信息。
      group_interval: 5m
      # 如果一个报警信息已经发送成功了，等待'repeat_interval'时间来重新发送他们
      repeat_interval: 5m
      # 默认的receiver：如果一个报警没有被一个route匹配，则发送给默认的接收器
      receiver: default
    receivers:
    - name: 'default'
      email_configs:
      - to: '1659775014@qq.com'
        send_resolved: true
```



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertmanager
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      containers:
      - name: alertmanager
        image: harbor.od.com/public/alertmanager:v0.14.0
        args:
          - "--config.file=/etc/alertmanager/config.yml"
          - "--storage.path=/alertmanager"
        ports:
        - name: alertmanager
          containerPort: 9093
        volumeMounts:
        - name: alertmanager-cm
          mountPath: /etc/alertmanager
      volumes:
      - name: alertmanager-cm
        configMap:
          name: alertmanager-config
```



```yaml
# Prometheus调用alert采用service name。不走ingress域名
apiVersion: v1
kind: Service
metadata:
  name: alertmanager
  namespace: kube-system
spec:
  selector: 
    app: alertmanager
  ports:
    - port: 80
      targetPort: 9093
```

### 5.3. 应用资源配置清单

```bash
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/alertmanager/configmap.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/alertmanager/deployment.yaml
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/devops/prometheus/alertmanager/service.yaml
```

### 5.4. 添加告警规则

```yaml
[root@hdss7-200 ~]# cat /data/nfs-volume/prometheus/etc/rules.yml # 配置中prometheus目录下
groups:
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu{mode!='idle'}[5m]))) by (instance) > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} CPU usage above 85% (current value: {{ $value }}%)"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal - node_memory_MemAvailable)/node_memory_MemTotal > 0.85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "{{ $labels.instance }} MEM usage above 85% (current value: {{ $value }}%)"
  - alert: OutOfInodes
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/"} / node_filesystem_size{fstype="overlay",mountpoint ="/"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of inodes (instance {{ $labels.instance }})"
      description: "Disk is almost running out of available inodes (< 10% left) (current value: {{ $value }})"
  - alert: OutOfDiskSpace
    expr: node_filesystem_free{fstype="overlay",mountpoint ="/rootfs"} / node_filesystem_size{fstype="overlay",mountpoint ="/rootfs"} * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Out of disk space (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 10% left) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputIn
    expr: sum by (instance) (irate(node_network_receive_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput in (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably receiving too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualNetworkThroughputOut
    expr: sum by (instance) (irate(node_network_transmit_bytes[2m])) / 1024 / 1024 > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual network throughput out (instance {{ $labels.instance }})"
      description: "Host network interfaces are probably sending too much data (> 100 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadRate
    expr: sum by (instance) (irate(node_disk_bytes_read[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read rate (instance {{ $labels.instance }})"
      description: "Disk is probably reading too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskWriteRate
    expr: sum by (instance) (irate(node_disk_bytes_written[2m])) / 1024 / 1024 > 50
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write rate (instance {{ $labels.instance }})"
      description: "Disk is probably writing too much data (> 50 MB/s) (current value: {{ $value }})"
  - alert: UnusualDiskReadLatency
    expr: rate(node_disk_read_time_ms[1m]) / rate(node_disk_reads_completed[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk read latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (read operations > 100ms) (current value: {{ $value }})"
  - alert: UnusualDiskWriteLatency
    expr: rate(node_disk_write_time_ms[1m]) / rate(node_disk_writes_completedl[1m]) > 100
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Unusual disk write latency (instance {{ $labels.instance }})"
      description: "Disk latency is growing (write operations > 100ms) (current value: {{ $value }})"
- name: http_status
  rules:
  - alert: ProbeFailed
    expr: probe_success == 0
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Probe failed (instance {{ $labels.instance }})"
      description: "Probe failed (current value: {{ $value }})"
  - alert: StatusCode
    expr: probe_http_status_code <= 199 OR probe_http_status_code >= 400
    for: 1m
    labels:
      severity: error
    annotations:
      summary: "Status Code (instance {{ $labels.instance }})"
      description: "HTTP status code is not 200-399 (current value: {{ $value }})"
  - alert: SslCertificateWillExpireSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate will expire soon (instance {{ $labels.instance }})"
      description: "SSL certificate expires in 30 days (current value: {{ $value }})"
  - alert: SslCertificateHasExpired
    expr: probe_ssl_earliest_cert_expiry - time()  <= 0
    for: 5m
    labels:
      severity: error
    annotations:
      summary: "SSL certificate has expired (instance {{ $labels.instance }})"
      description: "SSL certificate has expired already (current value: {{ $value }})"
  - alert: BlackboxSlowPing
    expr: probe_icmp_duration_seconds > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow ping (instance {{ $labels.instance }})"
      description: "Blackbox ping took more than 2s (current value: {{ $value }})"
  - alert: BlackboxSlowRequests
    expr: probe_http_duration_seconds > 2 
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Blackbox slow requests (instance {{ $labels.instance }})"
      description: "Blackbox request took more than 2s (current value: {{ $value }})"
  - alert: PodCpuUsagePercent
    expr: sum(sum(label_replace(irate(container_cpu_usage_seconds_total[1m]),"pod","$1","container_label_io_kubernetes_pod_name", "(.*)"))by(pod) / on(pod) group_right kube_pod_container_resource_limits_cpu_cores *100 )by(container,namespace,node,pod,severity) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod cpu usage percent has exceeded 80% (current value: {{ $value }}%)"
```



```yaml
[root@hdss7-200 ~]# vim /data/nfs-volume/prometheus/etc/prometheus.yml # 在末尾追加，关联告警规则
......
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager"]
rule_files:
 - "/data/etc/rules.yml"
```



```bash
# 重载配置文件，即reload
[root@hdss7-21 ~]# kubectl exec prometheus-78f57bbb58-6tcmq -it -n kube-system -- kill -HUP 1
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581131137945-f1584b95-4bb5-4216-a4e8-1f95a9eed763.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

## 6. Prometheus的使用

### 6.1. Prometheus配置文件解析

```yaml
# 官方文档： https://prometheus.io/docs/prometheus/latest/configuration/configuration/
[root@hdss7-200 ~]# vim /data/nfs-volume/prometheus/etc/prometheus.yml
global:
  scrape_interval:     15s  # 数据抓取周期,默认1m
  evaluation_interval: 15s  # 估算规则周期,默认1m
scrape_configs:             # 抓取指标的方式，一个job就是一类指标的获取方式
- job_name: 'etcd'          # 指定etcd的指标获取方式，没指定scrape_interval会使用全局配置
  tls_config:
    ca_file: /data/etc/ca.pem
    cert_file: /data/etc/client.pem
    key_file: /data/etc/client-key.pem
  scheme: https             # 默认是http方式获取
  static_configs:
  - targets:
    - '10.4.7.12:2379'
    - '10.4.7.21:2379'
    - '10.4.7.22:2379'
- job_name: 'kubernetes-apiservers'  
  kubernetes_sd_configs:
  - role: endpoints  # 目标资源类型，支持node、endpoints、pod、service、ingress等
  scheme: https      # tls,bearer_token_file都是与apiserver通信时使用
  tls_config:
    ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
  relabel_configs: # 对目标标签修改时使用
  - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
    action: keep   # action支持: 
    # keep,drop,replace,labelmap,labelkeep,labeldrop,hashmod
    regex: default;kubernetes;https
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'kubernetes-kubelet'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:10255
- job_name: 'kubernetes-cadvisor'
  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
  - source_labels: [__meta_kubernetes_node_name]
    regex: (.+)
    target_label: __address__
    replacement: ${1}:4194
- job_name: 'kubernetes-kube-state'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
  - source_labels: [__meta_kubernetes_pod_label_grafanak8sapp]
    regex: .*true.*
    action: keep
  - source_labels: ['__meta_kubernetes_pod_label_daemon', '__meta_kubernetes_pod_node_name']
    regex: 'node-exporter;(.*)'
    action: replace
    target_label: nodename
- job_name: 'blackbox_http_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [http_2xx]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: http
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port,  __meta_kubernetes_pod_annotation_blackbox_path]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+);(.+)
    replacement: $1:$2$3
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'blackbox_tcp_pod_probe'
  metrics_path: /probe
  kubernetes_sd_configs:
  - role: pod
  params:
    module: [tcp_connect]
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_blackbox_scheme]
    action: keep
    regex: tcp
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_blackbox_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __param_target
  - action: replace
    target_label: __address__
    replacement: blackbox-exporter.kube-system:9115
  - source_labels: [__param_target]
    target_label: instance
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
- job_name: 'traefik'
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scheme]
    action: keep
    regex: traefik
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
  - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
    action: replace
    regex: ([^:]+)(?::\d+)?;(\d+)
    replacement: $1:$2
    target_label: __address__
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
  - source_labels: [__meta_kubernetes_namespace]
    action: replace
    target_label: kubernetes_namespace
  - source_labels: [__meta_kubernetes_pod_name]
    action: replace
    target_label: kubernetes_pod_name
alerting:                             # Alertmanager配置
  alertmanagers:
    - static_configs:
        - targets: ["alertmanager"]
rule_files:                           # 引用外部的告警或者监控规则，类似于include
 - "/data/etc/rules.yml"
```



### 6.2. Pod接入Exporter

当前实验部署的是通用的Exporter，其中Kube-state-metrics是通过Kubernetes API采集信息，Node-exporter用于收集主机信息，这两项与Pod无关，部署完毕后直接使用即可。

根据Prometheus配置文件，可以看出Pod监控信息获取是通过标签(注释)选择器来实现的，给资源添加对应的标签或者注释来实现数据的监控。

#### 6.2.1. Traefik接入

```yaml
# 在traefik的daemonset.yaml的spec.template.metadata 加入注释，然后重启Pod
annotations:
  prometheus_io_scheme: traefik
  prometheus_io_path: /metrics
  prometheus_io_port: "8080"
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581849270044-01e944fe-b0e8-46ce-aae6-d398e130907f.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

#### 6.2.2. 接入Blackbox监控

```yaml
# 在对应pod的注释中添加，以下分别是TCP探测和HTTP探测，Prometheus中没有定义其它协议的探测
annotations:
  blackbox_port: "20880"
  blackbox_scheme: tcp
  
annotations:
  blackbox_port: "8080"
  blackbox_scheme: http
  blackbox_path: /hello?name=health
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581850563162-19d160e3-212b-4487-836f-eb835245263b.png?x-oss-process=image%2Fresize%2Cw_1500)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581850697500-9539a55e-2036-4fc5-887f-a9c15c5b758f.png)

#### 6.2.3. Pod接入监控

```
# 在对应pod的注释中添加，该信息是jmx_javaagent-0.3.1.jar收集的，开的端口是12346。true是字符串！
annotations:
  prometheus_io_scrape: "true"
  prometheus_io_port: "12346"
  prometheus_io_path: /
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581851405918-5c1df614-7d9b-48f9-be0f-f81052279c30.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

### 6.3. 测试Alertmanager

- 当停掉dubbo-demo-service的Pod后，blackbox的HTTP会探测失败，然后触发告警：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581852629025-0bfd432e-d62a-4b8b-8335-70970f290d8c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

- 在此启动dubbo-demo-service的Pod后，告警恢复

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1581852948272-0d495961-57c1-4e8a-b105-c9e9145952ea.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

