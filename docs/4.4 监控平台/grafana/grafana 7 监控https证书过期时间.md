- [grafana 7 监控https证书过期时间](https://www.cnblogs.com/fsckzy/p/14172262.html)

某个 https 证书突然过期，导致某个业务出现问题。理论上来说这个问题不应该存在，证书到期时间是固定的，更新也不费时间，但这个问题还是存在。

使用 Grafana 7 中[new table visualization](https://grafana.com/blog/2020/05/18/grafana-v7.0-released-new-plugin-architecture-visualizations-transformations-native-trace-support-and-more/#ux-enhancements-and-unified-data-model)功能，使用[Prometheus](http://grafana.com/oss/prometheus)监视证书的到期日期，并使用[Grafana](http://grafana.com/oss/grafana)进行

展示。

这就是它的样子，所有证书一目了然：证书到期之前的剩余时间，HTTP状态码和连接时间等等

![img](https://img2020.cnblogs.com/blog/891189/202012/891189-20201222121037400-1516025860.png)

## 导出和获取指标

使用 blackbox_exporter 收集此数据需要的一切指标，blackbox exporter 可以监控 http/https 页面，ip、端口等。

在这里我们使用它监控 [host:port] ，可以获取SSL证书信息，并从中自动捕获到期日期，并使用`probe_ssl_earliest_cert_expiry`指标计算剩余时间。

以下是配置，非常简单：

1. [blackbox.yml](https://github.com/prometheus/blackbox_exporter/blob/master/blackbox.yml)

```yaml
modules:
  http_2xx:
    prober: http
    http:
            preferred_ip_protocol: "ip4"
            tls_config:
                    insecure_skip_verify: true
```

1. prometheus.yml 添加一个 job

```yaml
- job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
              - https://xx.com
              - https://baidu.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <blackboxexporter_IP>:9115  # Blackbox exporter scraping address
```

## 展示所有指标

现在已经收集到了指标，接下来就要在 Grafana 展示这些内容了。使用https://grafana.com/grafana/dashboards/13230 这个模板。

下面是几个尽量简单的例子：

![img](https://img2020.cnblogs.com/blog/891189/202012/891189-20201222121054543-1736285888.png)

剩余时间：

```sql
probe_ssl_earliest_cert_expiry-time()
```

HTTP 状态码：

```shell
probe_http_status_code
```

所有 HTTP **duration**  查询：

```shell
probe_http_duration_seconds{phase="resolve"}
probe_http_duration_seconds{phase="connect"}
probe_http_duration_seconds{phase="tls"}
probe_http_duration_seconds{phase="processing"}
probe_http_duration_seconds{phase="transfer"}
```

## 转换功能

我们还利用了Grafana 7的 Transform 功能：在 instance 字段上使用 **Outer join** 将所有查询的结果一起显示在一行上。

![img](https://img2020.cnblogs.com/blog/891189/202012/891189-20201222121108270-353925882.png)

该**组织字段**转换也被用来过滤显示在我们的面板上

![img](https://img2020.cnblogs.com/blog/891189/202012/891189-20201222121118638-678098040.png)

## 告警

虽然我们在 Grafana 做了展示，但是我们不会经常去看的，所以还是要在 Prometheus 设置告警，这样在证书过期时会收到通知。

```yaml
- name: ssl_expiry
  rules:
  - alert: Ssl Cert Will Expire in 30 days
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 30
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate will expire soon on (instance {{ $labels.instance }})"
      description: "SSL certificate expires in 30 days\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

当然你也可以修改为其他时间，比如 10 天，那表达式就改为：

```shell
expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 10
```

别忘了同步修改警报名称和描述