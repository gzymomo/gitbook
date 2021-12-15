prometheus.yml

```yml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: '腾讯云服务器监控'  
    static_configs:
    - targets: ['172.21.0.15:9100']
  # 监控mysql
  - job_name: 'MySql实例监控'  
    static_configs:
    - targets: ['172.21.0.15:9104']
  # 监控Docker
#  - job_name: '腾讯云服务器Docker监控'
#    file_sd_configs:
#    - refresh_interval: 1m
#      files: 
#      - "/var/minio/prometheus/docker_exporter.yml"
#  - job_name: '腾讯云服务器SpringBoot监控'
#    metrics_path: '/actuator/prometheus'
#    static_configs:
#    - targets: ['172.21.0.15:7081']
#alerting:
#  alertmanagers:
#  - static_configs:
#    - targets:
#      - 172.21.0.15:9093
#rule_files:
#  - "/var/minio/prometheus/node_down.yml"                 # 实例存活报警规则文件
#  - "/var/minio/prometheus/memory_over.yml"               # 内存报警规则文件
#  - "/var/minio/prometheus/cpu_over.yml"                  # cpu报警规则文件
```



alertmanager.yml

```yaml
global:
  resolve_timeout: 5m
route:
  receiver: webhook
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 5m
  group_by: [alertname]
  routes:
  - receiver: webhook
    group_wait: 10s
receivers:
- name: webhook
  webhook_configs:
  - url: http://172.21.0.15:8060/dingtalk/webhook1/send  
    send_resolved: true
```

