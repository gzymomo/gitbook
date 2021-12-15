# prometheus监控mongodb

# 1 mongodb_exporter部署

## 1.1 安装

```bash
wget
https://github.com/percona/mongodb_exporter/releases/download/v0.11.2/mongodb_exporter-0.11.2.linux-amd64.tar.gz
tar xf mongodb_exporter-0.11.2.linux-amd64.tar.gz -C /opt/
cd /opt &&mv mongodb_exporter-0.11.2.linux-amd64.tar.gz mongodb_exporter
```

## 1.2 启动

```bash
#无密码
/opt/mongodb_exporter/mongodb_exporter  --mongodb.uri=mongodb://172.16.0.9:27017
#有密码
/opt/mongodb_exporter/mongodb_exporter  --mongodb.uri=mongodb://user:password@172.16.0.9:27017
cat <<EOF >/usr/lib/systemd/system/mongodb_exporter.service
[Unit]
Description=MongoDB Exporter
User=root

[Service]
Type=simple
Restart=always
ExecStart=/opt/mongodb_exporter/mongodb_exporter  -- 
mongodb.uri=mongodb://172.16.0.9:27017

[Install]
WantedBy=multi-user.target
EOF
systemctl start mongodb_exporter&&systemctl enable mongodb_exporter
```

mongodb_exporter暴露的endpoint端口默认为9216

```bash
curl http://172.16.0.9:9216/metrics
```

# 2 配置prometheus连接mongodb_exporter

```yml
- job_name: 'elasticsearch_exporter'
    scrape_interval: 10s
    metrics_path: "/metrics"
    static_configs:
    - targets: ['172.16.0.9:9216']
```

# 3 Grafana dashboard

导入mongodb监控模板，dashboard Id：`2583` `12079`

# 4 添加mongodb监控告警

```yml
groups:
- name: mongodb
rules:
 - alert: MongodbDown
    expr: mongodb_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MongoDB Down (instance {{ $labels.instance }})
      description: "MongoDB instance is down\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  - alert: MongodbReplicationLag
    expr: mongodb_mongod_replset_member_optime_date{state="PRIMARY"} - ON (set) mongodb_mongod_replset_member_optime_date{state="SECONDARY"} > 10
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MongoDB replication lag (instance {{ $labels.instance }})
      description: "Mongodb replication lag is more than 10s\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  - alert: MongodbReplicationHeadroom
    expr: (avg(mongodb_mongod_replset_oplog_tail_timestamp - mongodb_mongod_replset_oplog_head_timestamp) - (avg(mongodb_mongod_replset_member_optime_date{state="PRIMARY"}) - avg(mongodb_mongod_replset_member_optime_date{state="SECONDARY"}))) <= 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MongoDB replication headroom (instance {{ $labels.instance }})
      description: "MongoDB replication headroom is <= 0\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
  - alert: MongodbTooManyConnections
    expr: avg by(instance) (rate(mongodb_connections{state="current"}[1m])) / avg by(instance) (sum (mongodb_connections) by (instance)) * 100 > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: MongoDB too many connections (instance {{ $labels.instance }})
      description: "Too many connections (> 80%)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```