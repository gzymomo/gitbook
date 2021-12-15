- [K8s 部署 Prometheus + Grafana](https://www.cnblogs.com/lb477/p/14950213.html)             

## 一、简介[#](https://www.cnblogs.com/lb477/p/14950213.html#一、简介)

### 1. Prometheus[#](https://www.cnblogs.com/lb477/p/14950213.html#1-prometheus)

- 一款开源的**监控&报警&时间序列数据库**的组合，起始是由 SoundCloud 公司开发的
- 基本原理是通过 HTTP 协议周期性抓取被监控组件的状态，这样做的好处是任意组件只要提供 HTTP 接口就可以接入监控系统，不需要任何 SDK 或者其他的集成过程。这样做非常适合虚拟化环境比如 VM 或者 Docker
- 输出被监控组件信息的 HTTP 接口被叫做 exporter 。目前互联网公司常用的组件大部分都有 exporter 可以直接使用，比如 Varnish、Haproxy、Nginx、MySQL、Linux 系统信息（包括磁盘、内存、CPU、网络等），具体支持的源看：[https://github.com/prometheus](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fprometheus)
- 特点：
  - 一个多维数据模型（时间序列由指标名称定义和设置键/值尺寸）
  - 非常高效的存储，平均一个采样数据占 ~3.5bytes 左右，320 万的时间序列，每 30 秒采样，保持 60 天，消耗磁盘大概 228G
  - 一种灵活的查询语言
  - 不依赖分布式存储，单个服务器节点
  - 时间集合通过 HTTP 上的 PULL 模型进行
  - 通过中间网关支持推送时间
  - 通过服务发现或静态配置发现目标
  - 多种模式的图形和仪表板支持

### 2. Grafana[#](https://www.cnblogs.com/lb477/p/14950213.html#2-grafana)

- 一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知
- 特点：
  - 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，如热图、折线图、图表等多种展示方式
  - 数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch 和 KairosDB 等
  - 通知提醒：以可视方式定义最重要指标的警报规则，Grafana 将不断计算并发送通知，在数据达到阈值时通过 Slack、PagerDuty 等获得通知
  - 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源
  - 注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记
  - 过滤器：Ad-hoc 过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询

### 3. 效果展示[#](https://www.cnblogs.com/lb477/p/14950213.html#3-效果展示)

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629151254987-19319508.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629151254987-19319508.png)

## 二、部署[#](https://www.cnblogs.com/lb477/p/14950213.html#二、部署)

```bash
$ kubectl create ns ns-monitor
$ kubectl create -f ...
$ kubectl get all -n ns-monitor
NAME                              READY   STATUS    RESTARTS   AGE
pod/node-exporter-rcbss           1/1     Running   0          4h41m
pod/grafana-5567c66c9d-49b5w      1/1     Running   0          4h25m
pod/prometheus-5ccc8db98f-lkwf5   1/1     Running   0          3h12m

NAME                            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/node-exporter-service   NodePort   10.43.75.152    <none>        9100:31672/TCP   4h41m
service/grafana-service         NodePort   10.43.26.238    <none>        3000:32534/TCP   4h25m
service/prometheus-service      NodePort   10.43.174.110   <none>        9090:31396/TCP   3h12m
```

> grafana 和 prometheus 没有配置`nodePort`，端口随机生成

### 1. node-exporter[#](https://www.cnblogs.com/lb477/p/14950213.html#1-node-exporter)

- 用于采集 k8s 集群中各个节点的物理指标，如 Memory、CPU 等。可以直接在每个物理节点直接安装

```yml
kind: DaemonSet
apiVersion: apps/v1
metadata: 
  labels:
    app: node-exporter
  name: node-exporter
  namespace: ns-monitor
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:v0.16.0
          ports:
            - containerPort: 9100
              protocol: TCP
              name:	http
      hostNetwork: true  # 获得Node的物理指标信息
      hostPID: true  # 获得Node的物理指标信息
#      tolerations:  # Master节点
#        - effect: NoSchedule
#          operator: Exists

---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: node-exporter
  name: node-exporter-service
  namespace: ns-monitor
spec:
  ports:
    - name:	http
      port: 9100
      nodePort: 31672
      protocol: TCP
  type: NodePort
  selector:
    app: node-exporter
```

http://192.168.11.210:31672/metrics

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629152431380-1891906640.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629152431380-1891906640.png)

### 2. Prometheus[#](https://www.cnblogs.com/lb477/p/14950213.html#2-prometheus)

```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
  - apiGroups: [""]  # "" indicates the core API group
    resources:
      - nodes
      - nodes/proxy
      - services
      - endpoints
      - pods
    verbs:
      - get
      - watch
      - list
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - watch
      - list
  - nonResourceURLs: ["/metrics"]
    verbs:
      - get

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: ns-monitor
  labels:
    app: prometheus

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: ns-monitor
roleRef:
  kind: ClusterRole
  name: prometheus
  apiGroup: rbac.authorization.k8s.io

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-conf
  namespace: ns-monitor
  labels:
    app: prometheus
data:
  prometheus.yml: |-
    # my global config
    global:
      scrape_interval:     15s  # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 15s  # Evaluate rules every 15 seconds. The default is every 1 minute.
      # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: 'prometheus'
        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.
        static_configs:
          - targets: ['localhost:9090']
          
      - job_name: 'grafana'
        static_configs:
          - targets:
              - 'grafana-service.ns-monitor:3000'

      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        # Default to scraping over https. If required, just disable this or change to
        # `http`.
        scheme: https
        # This TLS & bearer token file config is used to connect to the actual scrape
        # endpoints for cluster components. This is separate to discovery auth
        # configuration because discovery & scraping are two separate concerns in
        # Prometheus. The discovery auth config is automatic if Prometheus runs inside
        # the cluster. Otherwise, more config options have to be provided within the
        # <kubernetes_sd_config>.
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          # If your node certificates are self-signed or use a different CA to the
          # master CA, then disable certificate verification below. Note that
          # certificate verification is an integral part of a secure infrastructure
          # so this should only be disabled in a controlled environment. You can
          # disable certificate verification by uncommenting the line below.
          #
          # insecure_skip_verify: true
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        # Keep only the default/kubernetes service endpoints for the https port. This
        # will add targets for each API server which Kubernetes adds an endpoint to
        # the default/kubernetes service.
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
          
      # Scrape config for nodes (kubelet).
      #
      # Rather than connecting directly to the node, the scrape is proxied though the
      # Kubernetes apiserver.  This means it will work if Prometheus is running out of
      # cluster, or can't connect to nodes for some other reason (e.g. because of
      # firewalling).
      - job_name: 'kubernetes-nodes'
        # Default to scraping over https. If required, just disable this or change to
        # `http`.
        scheme: https
        # This TLS & bearer token file config is used to connect to the actual scrape
        # endpoints for cluster components. This is separate to discovery auth
        # configuration because discovery & scraping are two separate concerns in
        # Prometheus. The discovery auth config is automatic if Prometheus runs inside
        # the cluster. Otherwise, more config options have to be provided within the
        # <kubernetes_sd_config>.
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics

      # Scrape config for Kubelet cAdvisor.
      #
      # This is required for Kubernetes 1.7.3 and later, where cAdvisor metrics
      # (those whose names begin with 'container_') have been removed from the
      # Kubelet metrics endpoint.  This job scrapes the cAdvisor endpoint to
      # retrieve those metrics.
      #
      # In Kubernetes 1.7.0-1.7.2, these metrics are only exposed on the cAdvisor
      # HTTP endpoint; use "replacement: /api/v1/nodes/${1}:4194/proxy/metrics"
      # in that case (and ensure cAdvisor's HTTP server hasn't been disabled with
      # the --cadvisor-port=0 Kubelet flag).
      #
      # This job is not necessary and should be removed in Kubernetes 1.6 and
      # earlier versions, or it will cause the metrics to be scraped twice.
      - job_name: 'kubernetes-cadvisor'
        # Default to scraping over https. If required, just disable this or change to
        # `http`.
        scheme: https
        # This TLS & bearer token file config is used to connect to the actual scrape
        # endpoints for cluster components. This is separate to discovery auth
        # configuration because discovery & scraping are two separate concerns in
        # Prometheus. The discovery auth config is automatic if Prometheus runs inside
        # the cluster. Otherwise, more config options have to be provided within the
        # <kubernetes_sd_config>.
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

      # Scrape config for service endpoints.
      #
      # The relabeling allows the actual service scrape endpoint to be configured
      # via the following annotations:
      #
      # * `prometheus.io/scrape`: Only scrape services that have a value of `true`
      # * `prometheus.io/scheme`: If the metrics endpoint is secured then you will need
      # to set this to `https` & most likely set the `tls_config` of the scrape config.
      # * `prometheus.io/path`: If the metrics path is not `/metrics` override this.
      # * `prometheus.io/port`: If the metrics are exposed on a different port to the
      # service then set this appropriately.
      - job_name: 'kubernetes-service-endpoints'
        kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
          action: replace
          target_label: __scheme__
          regex: (https?)
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: kubernetes_name

      # Example scrape config for probing services via the Blackbox Exporter.
      #
      # The relabeling allows the actual service scrape endpoint to be configured
      # via the following annotations:
      #
      # * `prometheus.io/probe`: Only probe services that have a value of `true`
      - job_name: 'kubernetes-services'
        metrics_path: /probe
        params:
          module: [http_2xx]
        kubernetes_sd_configs:
        - role: service
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
          action: keep
          regex: true
        - source_labels: [__address__]
          target_label: __param_target
        - target_label: __address__
          replacement: blackbox-exporter.example.com:9115
        - source_labels: [__param_target]
          target_label: instance
        - action: labelmap
          regex: __meta_kubernetes_service_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_service_name]
          target_label: kubernetes_name

      # Example scrape config for probing ingresses via the Blackbox Exporter.
      #
      # The relabeling allows the actual ingress scrape endpoint to be configured
      # via the following annotations:
      #
      # * `prometheus.io/probe`: Only probe services that have a value of `true`
      - job_name: 'kubernetes-ingresses'
        metrics_path: /probe
        params:
          module: [http_2xx]
        kubernetes_sd_configs:
          - role: ingress
        relabel_configs:
          - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
            regex: (.+);(.+);(.+)
            replacement: ${1}://${2}${3}
            target_label: __param_target
          - target_label: __address__
            replacement: blackbox-exporter.example.com:9115
          - source_labels: [__param_target]
            target_label: instance
          - action: labelmap
            regex: __meta_kubernetes_ingress_label_(.+)
          - source_labels: [__meta_kubernetes_namespace]
            target_label: kubernetes_namespace
          - source_labels: [__meta_kubernetes_ingress_name]
            target_label: kubernetes_name

      # Example scrape config for pods
      #
      # The relabeling allows the actual pod scrape endpoint to be configured via the
      # following annotations:
      #
      # * `prometheus.io/scrape`: Only scrape pods that have a value of `true`
      # * `prometheus.io/path`: If the metrics path is not `/metrics` override this.
      # * `prometheus.io/port`: Scrape the pod on the indicated port instead of the
      # pod's declared ports (default is a port-free target if none are declared).
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

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
  namespace: ns-monitor
  labels:
    app: prometheus
data:
  cpu-usage.rule: |
    groups:
      - name: NodeCPUUsage
        rules:
          - alert: NodeCPUUsage
            expr: (100 - (avg by (instance) (irate(node_cpu{name="node-exporter",mode="idle"}[5m])) * 100)) > 75
            for: 2m
            labels:
              severity: "page"
            annotations:
              summary: "{{$labels.instance}}: High CPU usage detected"
              description: "{{$labels.instance}}: CPU usage is above 75% (current value is: {{ $value }})"

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: "prometheus-data-pv"
  labels:
    name: prometheus-data-pv
    release: stable
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /nfs/prometheus/data
    server: 192.168.11.210

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-data-pvc
  namespace: ns-monitor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      name: prometheus-data-pv
      release: stable

---
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: ns-monitor
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus
      securityContext:
        runAsUser: 0
      containers:
        - name: prometheus
          image: prom/prometheus:latest
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - mountPath: /prometheus
              name: prometheus-data-volume
            - mountPath: /etc/prometheus/prometheus.yml
              name: prometheus-conf-volume
              subPath: prometheus.yml
            - mountPath: /etc/prometheus/rules
              name: prometheus-rules-volume
          ports:
            - containerPort: 9090
              protocol: TCP
      volumes:
        - name: prometheus-data-volume
          persistentVolumeClaim:
            claimName: prometheus-data-pvc
        - name: prometheus-conf-volume
          configMap:
            name: prometheus-conf
        - name: prometheus-rules-volume
          configMap:
            name: prometheus-rules
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---
kind: Service
apiVersion: v1
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  labels:
    app: prometheus
  name: prometheus-service
  namespace: ns-monitor
spec:
  ports:
    - port: 9090
      targetPort: 9090
  selector:
    app: prometheus
  type: NodePort
```

http://192.168.11.210:31396/targets

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629152315174-557678604.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629152315174-557678604.png)

### 3. Grafana[#](https://www.cnblogs.com/lb477/p/14950213.html#3-grafana)

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: "grafana-data-pv"
  labels:
    name: grafana-data-pv
    release: stable
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /nfs/grafana/data
    server: 192.168.11.210

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-data-pvc
  namespace: ns-monitor
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      name: grafana-data-pv
      release: stable

---
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    app: grafana
  name: grafana
  namespace: ns-monitor
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        runAsUser: 0
      containers:
        - name: grafana
          image: grafana/grafana:latest
          imagePullPolicy: IfNotPresent
          env:
            - name: GF_AUTH_BASIC_ENABLED
              value: "true"
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "false"
          readinessProbe:
            httpGet:
              path: /login
              port: 3000
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-data-volume
          ports:
            - containerPort: 3000
              protocol: TCP
      volumes:
        - name: grafana-data-volume
          persistentVolumeClaim:
            claimName: grafana-data-pvc

---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: grafana
  name: grafana-service
  namespace: ns-monitor
spec:
  ports:
    - port: 3000
      targetPort: 3000
  selector:
    app: grafana
  type: NodePort
```

http://192.168.11.210:32534

***配置数据源***

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629150352669-1051319872.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210629150352669-1051319872.png)

***Import dashboard from file（非必须）***

https://files.cnblogs.com/files/lb477/Kubernetes-Pod-Resources.json