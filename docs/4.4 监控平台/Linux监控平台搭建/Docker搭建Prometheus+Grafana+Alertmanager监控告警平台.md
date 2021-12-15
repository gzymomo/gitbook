# 一、概述

在之前的文章《企业级监控平台如何选择？》中，我们了解到目前社区活跃度比较广，互联网大厂倾向的监控平台Promethes，今天我们就实战搭建一个企业级的监控平台，用来监控服务器、MySQL数据库、Redis、Docker等。



需要搭建的环境有：

1. Docker-用来部署各个软件的基础环境
2. Prometheus-时序数据库
3. Grafana-大屏风格的Web可视化监控方案
4. AlertManager-配置告警选项
5. webhook-实现钉钉告警



# 二、Centos部署Docker环境

## 2.1 方式一：yum方式安装

```bash
# 更新yum源
yum update
# 安装所需环境
yum install -y yum-utils device-mapper-persistent-data lvm2
# 配置yum仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装Docker
yum install docker-ce
# 启动Docker
systemctl start docker
systemctl enable docker
```



## 2.2 方式二：（推荐安装方式）

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

## 2.3 离线安装

1. 下载docker的安装文件：https://download.docker.com/linux/static/stable/x86_64/

![image-20210201092144600](http://lovebetterworld.com/image-20210201092144600.png)

2. 将下载后的tgz文件传至服务器，通过FTP工具上传即可
3. 解压`tar -zxvf docker-19.03.8-ce.tgz`
4. 将解压出来的docker文件复制到 /usr/bin/ 目录下：`cp docker/* /usr/bin/`
5. 进入**/etc/systemd/system/**目录,并创建**docker.service**文件

```bash
[root@localhost java]# cd /etc/systemd/system/
[root@localhost system]# touch docker.service
```

6. 打开**docker.service**文件,将以下内容复制

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=192.168.200.128
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

7. 给docker.service文件添加执行权限：`chmod 777 /etc/systemd/system/docker.service `

8. 重新加载配置文件：`systemctl daemon-reload `

9. 启动Docker `systemctl start docker`

10. 设置开机启动：`systemctl enable docker.service`

11. 查看Docker状态：`systemctl status docker`

    ![image-20210201092621496](http://lovebetterworld.com/image-20210201092621496.png)

如出现如图界面，则表示安装成功！



# 三、Docker部署监控平台

## 3.1 Docker部署Grafana

```
docker run -d --restart=always --name=grafana -p 3000:3000 grafana/grafana
```

- -d：后台启动
- --restart=always：重启
- -p：端口映射
- grafana/grafana：后面没有指定版本号，默认拉取latest版本，如需其他版本，指定相应版本即可



搭建完成后，在浏览器访问：http://localhost:3000

默认账号密码：admin，admin

第一次登陆后，会提示修改密码，按照要求修改密码即可。

![image-20210201093843084](http://lovebetterworld.com/image-20210201093843084.png)



## 3.2 Docker部署Prometheus

1. 创建一个文件夹用来存放Prometheus的配置文件

```bash
mkdir -p /var/project/prometheus
```

2. 编写prometheus.yml配置文件

```yaml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: 'prometheus监控'  
    static_configs:
    - targets: ['127.0.0.1:9090']
```

3. 启动Prometheus

```bash
docker run -d --restart=always --name=prometheus -p 9090:9090 -v /etc/localtime:/etc/localtime:ro  -v /var/project/prometheus:/etc/prometheus prom/prometheus
```

- -d：后台启动
- --restart=always：重启
- -p：端口映射
- -v /etc/localtime:/etc/localtime:ro：同步本地时间和容器时间一致
- -v /var/project/prometheus:/etc/prometheus：将容器内的prometheus配置文件路径映射在容器外部
- prom/prometheus：后面没有指定版本号，默认拉取latest版本，如需其他版本，指定相应版本即可

4. 浏览器访问验证：http://localhost:9090

![image-20210201094448559](http://lovebetterworld.com/image-20210201094448559.png)



出现如图所示界面，则表示安装成功！

## 3.3 Docker部署Webhook

此处我使用的告警方式是钉钉群主告警，故需添加自定义钉钉机器人实现告警。

1. 创建一个自定义的钉钉机器人，创建一个钉钉群，找到只能群助手

![image-20210201095444854](http://lovebetterworld.com/image-20210201095444854.png)



2. 添加一个机器人

![image-20210201095508577](http://lovebetterworld.com/image-20210201095508577.png)



3. 选择自定义Webhook机器人

![image-20210201095533268](http://lovebetterworld.com/image-20210201095533268.png)



4. 添加相关信息

![image-20210201095700510](http://lovebetterworld.com/image-20210201095700510.png)

5. 完成后，出现如下界面，复制Webhook。

![image-20210201095905616](http://lovebetterworld.com/image-20210201095905616.png)

6. Docker部署Webhook钉钉

```bash
docker run -d --restart=always -p 8060:8060 --name webhook timonwong/prometheus-webhook --ding.profile="webhook1=https://oapi.dingtalk.com/robot/send?access_token={替换成自己的dingding token}
```

- -d：后台启动
- --restart=always：重启
- -p：端口映射
- --ding.profile：配置webhook



## 3.4 Docker部署AlertManager

1. 创建一个文件夹用来存放AlertManager的配置文件

```bash
mkdir -p /var/project/alertmanager
```

2. 编辑alertmanager.yml配置文件

```bash
vi alertmanager.yml
```

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
  - url: http://127.0.0.1:8060/dingtalk/webhook1/send  
    send_resolved: true
```

3. 启动AlertManager

```bash
docker run -d --restart=always --name alertmanager -p 9093:9093 -v /var/project/prometheus:/etc/alertmanager prom/alertmanager:latest
```

- -d：后台启动

- --restart=always：重启

- -p：端口映射

- -v /var/project/prometheus:/etc/alertmanager：将容器内的配置文件路径映射在容器外部

  ![image-20210201094912579](http://lovebetterworld.com/image-20210201094912579.png)



# 二、配置文件

## 2.1 prometheus.yml

```yaml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 监控prometheus本身
  - job_name: '服务器Prometheus'  
    static_configs:
    - targets: ['127.0.0.1:9090']
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: 'Linux服务器监控'  
    file_sd_configs:
    - refresh_interval: 1m
      files: 
      - "/etc/prometheus/node_exporter.yml"
  # 监控mysql
  - job_name: 'MySql实例监控'  
    static_configs:
    - targets: ['127.0.0.1:9104']
  # 监控Docker
  #- job_name: 'Docker实例监控'
  #  file_sd_configs:
  #  - refresh_interval: 1m
  #    files: 
  #    - "/var/project/prometheus/docker_exporter.yml"
    #监控Java，SpringBoot应用
  - job_name: 'SpringBoot应用监控'
    metrics_path: '/actuator/prometheus'
    file_sd_configs:
    - refresh_interval: 1m
      files: 
      - "/etc/prometheus/springboot_exporter.yml"
    #监控Redis集群 
  - job_name: 'Redis集群实例监控'
    static_configs:
      - targets:
        - redis://127.0.0.1:7000
        - redis://127.0.0.1:7001
        - redis://127.0.0.1:7002
        - redis://127.0.0.1:7003
        - redis://127.0.0.1:7004
        - redis://127.0.0.1:7005
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9121
  - job_name: http-blackbox
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets: ['www.baidu.com']
        labels:
          instance: 百度
          group: 'web'
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: 127.0.0.1:9115

# Alertting告警平台 
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 127.0.0.1:9093
rule_files:
  - "/etc/prometheus/rules/node_down.yml"                 # 实例存活报警规则文件
  - "/etc/prometheus/rules/memory_over.yml"               # 内存报警规则文件
  - "/etc/prometheus/rules/cpu_over.yml"                  # cpu报警规则文件
  - "/etc/prometheus/rules/black_exporter.yml"            # 黑盒监测
```

## 2.2 black_exporter.yml

```yml
groups:
- name: 网站URL告警规则
  rules:
  - alert:  网站URL存活告警
  # 这里我后续想了下还是换成 联通性为离线好点
    expr: probe_http_status_code{job="http-blackbox"}>=399 or probe_success{job="http-blackbox"}==0
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "{{ $labels.instance }} of job {{ $labels.job }}  {{ $value }}  网页无法连接"
```

## 2.3 blackbox-exporter：config.yml

```yml
modules:
  http_2xx: # http 监测模块
    prober: http
  http_post_2xx: # http post 监测模块
    prober: http
    http:
      method: POST
  tcp_connect:  # tcp 监测模块
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
```



## 2.4 alertmanager.yml

```yml
global:
  resolve_timeout: 5m #在没有报警的情况下声明为已解决的时间
route: # route用来设置报警的分发策略
  receiver: webhook # 设置默认接收人
  group_wait: 60m # 组告警等待时间。也就是告警产生后等待10s，如果有同组告警一起发出
  group_interval: 60m # 两组告警的间隔时间
  repeat_interval: 60m # 重复告警的间隔时间，减少相同邮件的发送频率
  group_by: [alertname]
  routes:
  - receiver: webhook
    group_wait: 10s
receivers:
- name: webhook
  webhook_configs:
  - url: http://10.19.64.63:8060/dingtalk/webhook1/send  
    send_resolved: true
```

## 2.5 node_exporter.yml

```yml
- targets:
  - "127.0.0.1:9100"
  labels:
    instance: 服务器A
- targets:
  - "127.0.0.1:9101"
  labels:
    instance: 服务器B
```

## 2.6 springboot_exporter.yml

```yml
- targets:
  - "127.0.0.1:8085"
  labels:
    instance: 服务A
- targets:
  - "127.0.0.1:8086"
  labels:
    instance: 服务B
```

# 三、rules

## 3.1 black_exporter.yml

```yml
groups:
- name: 网站URL告警规则
  rules:
  - alert:  网站URL存活告警
  # 这里我后续想了下还是换成 联通性为离线好点
    expr: probe_http_status_code{job="http-blackbox"}>=399 or probe_success{job="http-blackbox"}==0
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "{{ $labels.instance }} of job {{ $labels.job }}  {{ $value }}  网页无法连接"
```

## 3.2 cpu_over.yml

```yml
groups:
- name: CPU报警规则
  rules:
  - alert: CPU使用率告警
    expr: 100 - (avg by (instance)(irate(node_cpu_seconds_total{mode="idle"}[1m]) )) * 100 > 90
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "服务器: CPU使用超过90%！(当前值: {{ $value }}%)"
```

### 3.3 memory_over.yml

```yml
groups:
- name: 内存报警规则
  rules:
  - alert: 内存使用率告警
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes )) / node_memory_MemTotal_bytes * 100 > 90
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "服务器: 内存使用超过90%！(当前值: {{ $value }}%)"
```

## 3.4 node_down.yml

```yml
groups:
- name: 实例存活告警规则
  rules:
  - alert: 实例存活告警
    expr: up == 0
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "{{ $labels.instance }} of job {{ $labels.job }} 已经停止1分钟."
```

