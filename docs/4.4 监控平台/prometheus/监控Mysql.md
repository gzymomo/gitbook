- 码农沉思录：[Prometheus + Granafa 构建高大上的MySQL监控平台](https://mp.weixin.qq.com/s?__biz=MzAxNjM2MTk0Ng==&mid=2247494672&idx=2&sn=69d18a3bf8c5e347fab40e18d02c5c12&chksm=9bf75ca5ac80d5b39023b2991b178025cfdac6db92597ab881532a0fb8e162c586520c378aa4&mpshare=1&scene=24&srcid=1222pvdFkP3pespvABV0eIOp&sharer_sharetime=1608612421478&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)



# 一、prometheus监控体系之MySQL监控

Prometheus+Granafa监控体系监控MySQL需使用MySQL-Exporter。

## 1.1 mysqld_exporter源码安装部署

1. 安装exporter

```bash
[root@controller2 opt]# wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.10.0/mysqld_exporter-0.10.0.linux-amd64.tar.gz
[root@controller2 opt]# tar -xf mysqld_exporter-0.10.0.linux-amd64.tar.gz 
```

2. 添加mysql 账户：

```mysql
GRANT SELECT, PROCESS, SUPER, REPLICATION CLIENT, RELOAD ON *.* TO 'exporter'@'%' IDENTIFIED BY 'localhost';
flush privileges;
```

3. 编辑配置文件：

```bash
[root@controller2 mysqld_exporter-0.10.0.linux-amd64]# cat /opt/mysqld_exporter-0.10.0.linux-amd64/.my.cnf 
[client]
user=exporter
password=123456
```

4. 设置配置文件：

```bash
[root@controller2 mysqld_exporter-0.10.0.linux-amd64]# cat /etc/systemd/system/mysql_exporter.service 
[Unit]
Description=mysql Monitoring System
Documentation=mysql Monitoring System

[Service]
ExecStart=/opt/mysqld_exporter-0.10.0.linux-amd64/mysqld_exporter \
-collect.info_schema.processlist \
-collect.info_schema.innodb_tablespaces \
-collect.info_schema.innodb_metrics  \
-collect.perf_schema.tableiowaits \
-collect.perf_schema.indexiowaits \
-collect.perf_schema.tablelocks \
-collect.engine_innodb_status \
-collect.perf_schema.file_events \
-collect.info_schema.processlist \
-collect.binlog_size \
-collect.info_schema.clientstats \
-collect.perf_schema.eventswaits \
-config.my-cnf=/opt/mysqld_exporter-0.10.0.linux-amd64/.my.cnf

[Install]
WantedBy=multi-user.target
```

5. 添加配置到prometheus server

```bash
- job_name: 'mysql'
  static_configs:
   - targets: ['192.168.1.11:9104','192.168.1.12:9104']
```

6. 测试看有没有返回数值：

```bash
curl http://192.168.1.12:9104/metrics
```

正常我们通过mysql_up可以查询倒mysql监控是否已经生效，是否起起来

```bash
#HELP mysql_up Whether the MySQL server is up.
#TYPE mysql_up gauge
mysql_up 1
```

## 1.2 mysqld_exporter通过docker安装部署





# 二、mysql监控告警规则

```yaml
    groups:
    - name: MySQL-rules
      rules:
      - alert: MySQL Status 
        expr: up == 0
        for: 5s 
        labels:
          severity: warning
        annotations:
          summary: "{{$labels.instance}}: MySQL has stop !!!"
          description: "检测MySQL数据库运行状态"

      - alert: MySQL Slave IO Thread Status
        expr: mysql_slave_status_slave_io_running == 0
        for: 5s 
        labels:
          severity: warning
        annotations: 
          summary: "{{$labels.instance}}: MySQL Slave IO Thread has stop !!!"
          description: "检测MySQL主从IO线程运行状态"

      - alert: MySQL Slave SQL Thread Status 
        expr: mysql_slave_status_slave_sql_running == 0
        for: 5s 
        labels:
          severity: warning
        annotations: 
          summary: "{{$labels.instance}}: MySQL Slave SQL Thread has stop !!!"
          description: "检测MySQL主从SQL线程运行状态"

      - alert: MySQL Slave Delay Status 
        expr: mysql_slave_status_sql_delay == 30
        for: 5s 
        labels:
          severity: warning
        annotations: 
          summary: "{{$labels.instance}}: MySQL Slave Delay has more than 30s !!!"
          description: "检测MySQL主从延时状态"

      - alert: Mysql_Too_Many_Connections
        expr: rate(mysql_global_status_threads_connected[5m]) > 200
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "{{$labels.instance}}: 连接数过多"
          description: "{{$labels.instance}}: 连接数过多，请处理 ,(current value is: {{ $value }})"  

      - alert: Mysql_Too_Many_slow_queries
        expr: rate(mysql_global_status_slow_queries[5m]) > 3
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "{{$labels.instance}}: 慢查询有点多，请检查处理"
          description: "{{$labels.instance}}: Mysql slow_queries is more than 3 per second ,(current value is: {{ $value }})"
```

添加规则到prometheus：

```yaml
rule_files:
  - "rules/*.yml" 
```

然后打开prometheus的Web UI界面，找到监控告警栏目，即可看到监控告警是否成功！