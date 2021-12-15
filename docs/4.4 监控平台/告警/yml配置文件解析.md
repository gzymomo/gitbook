[TOC]

# 1、prometheus.yml
```yml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 监控prometheus本身
  - job_name: '服务器Prometheus'
    static_configs:
    - targets: ['ip:9090']
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: 'Linux服务器监控'
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/home/prometheus/node_exporter.yml"
  # 监控mysql
  - job_name: 'MySql实例监控'
    static_configs:
    - targets: ['ip:9104']
  # 监控Docker
  - job_name: 'Docker实例监控'
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/home/prometheus/docker_exporter.yml"
    # 监控Redis集群
  - job_name: 'SpringBoot应用监控'
    metrics_path: '/actuator/prometheus'
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/home/prometheus/springboot_exporter.yml"
  - job_name: '64.63-Redis集群实例监控'
    static_configs:
      - targets:
        - redis://ip:7000
        - redis://ip:7001
        - redis://ip:7002
        - redis://ip:7003
        - redis://ip:7004
        - redis://ip:7005
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: ip:9121
  - job_name: 'Spark程序监控'
    static_configs:
      - targets: ['ip:9108']
  - job_name: 'Windows服务器监控'
    static_configs:
      - targets: ['ip:9182','ip:9182']
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - ip:9093
rule_files:
  - "/home/prometheus/node_down.yml"                 # 实例存活报警规则文件
  - "/home/prometheus/memory_over.yml"               # 内存报警规则文件
  - "/home/prometheus/cpu_over.yml"                  # cpu报警规则文件
```

# 2、java_springboot.yml
```yml
- targets:
  - "192.168.0.50:8085"
  labels:
    instance: ui-zy
- targets:
  - "192.168.0.50:8086"
  labels:
    instance: ui-dz
```