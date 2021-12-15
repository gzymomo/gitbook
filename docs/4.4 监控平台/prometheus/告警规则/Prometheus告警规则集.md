# Prometheus 告警规则集

### 主机和硬件资源

主机和硬件资源的告警依赖 node-exporter[3] 输出的指标。例如：

#### **内存不足**

可用内存低于阈值 10% 就会触发告警。

```bash
- alert:HostOutOfMemory
    expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes *100<10
for:2m
    labels:
      severity: warning
    annotations:
      summary:Hostout of memory (instance {{ $labels.instance }})
      description:"Node memory is filling up (< 10% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

#### **主机异常的网络吞吐**

最近两分钟入站的流量超过 100m。

> `rate` 语法见这里[4]。

```bash
- alert:HostUnusualNetworkThroughputIn
    expr: sum by(instance)(rate(node_network_receive_bytes_total[2m]))/1024/1024>100
for:5m
    labels:
      severity: warning
    annotations:
      summary:Host unusual network throughput in(instance {{ $labels.instance }})
      description:"Host network interfaces are probably receiving too much data (> 100 MB/s)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

## MySQL

Mysql 的告警依赖 prometheus/mysqld_exporter[5] 输出的指标。

#### **连接数过多**

Mysql 实例的连接数最近一分钟的连接数超过最大值的 80% 触发告警

```bash
- alert:MysqlTooManyConnections(>80%)
    expr: avg by(instance)(rate(mysql_global_status_threads_connected[1m]))/ avg by(instance)(mysql_global_variables_max_connections)*100>80
for:2m
    labels:
      severity: warning
    annotations:
      summary:MySQL too many connections (>80%)(instance {{ $labels.instance }})
      description:"More than 80% of MySQL connections are in use on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

#### **慢查询**

最近一分钟慢查询数量大于 0 时触发。

```bash
- alert:MysqlSlowQueries
    expr: increase(mysql_global_status_slow_queries[1m])>0
for:2m
    labels:
      severity: warning
    annotations:
      summary:MySQL slow queries (instance {{ $labels.instance }})
      description:"MySQL server mysql has some new slow query.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

## 运行时 JVM

JVM 的运行时告警，居然只有可怜巴巴的一个。堆空间占用超过 80% 触发告警。

依赖 java-client[6] 输出的指标。

```bash
- alert:JvmMemoryFillingUp
    expr:(sum by(instance)(jvm_memory_used_bytes{area="heap"})/ sum by(instance)(jvm_memory_max_bytes{area="heap"}))*100>80
for:2m
    labels:
      severity: warning
    annotations:
      summary: JVM memory filling up (instance {{ $labels.instance }})
      description:"JVM memory is filling up (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

## Kubernetes

Kubernetes 相关的告警规则有 33 个，比较丰富。

摘个比较常见的：容器OOM告警。

```bash
- alert:KubernetesContainerOomKiller
    expr:(kube_pod_container_status_restarts_total - kube_pod_container_status_restarts_total offset 10m>=1)and ignoring (reason) min_over_time(kube_pod_container_status_last_terminated_reason{reason="OOMKilled"}[10m])==1
for:0m
    labels:
      severity: warning
    annotations:
      summary:Kubernetes container oom killer (instance {{ $labels.instance }})
      description:"Container {{ $labels.container }} in pod {{ $labels.namespace }}/{{ $labels.pod }} has been OOMKilled {{ $value }} times in the last 10 minutes.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

## SSL证书过期

通过 [7] 输出的指标，可以监控证书过期：未来 7 天 有证书过期便会触发告警。

```bash
- alert:SslCertificateExpiry(<7Days)
    expr: ssl_verified_cert_not_after{chain_no="0"}- time()<86400*7
for:0m
    labels:
      severity: warning
    annotations:
      summary: SSL certificate expiry (<7 days)(instance {{ $labels.instance }})
      description:"{{ $labels.instance }} Certificate is expiring in 7 days\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

#### 引用链接

[1] Awesome Prometheus alerts: https://awesome-prometheus-alerts.grep.to/
[2] 这里: https://awesome-prometheus-alerts.grep.to/alertmanager
[3] node-exporter: https://github.com/prometheus/node_exporter
[4] 这里: https://prometheus.io/docs/prometheus/latest/querying/functions/#rate
[5] prometheus/mysqld_exporter: https://github.com/prometheus/mysqld_exporter
[6] java-client: https://github.com/prometheus/client_java
[7] : https://github.com/ribbybibby/ssl_exporter
[8] 贡献: https://github.com/samber/awesome-prometheus-alerts#-contributing