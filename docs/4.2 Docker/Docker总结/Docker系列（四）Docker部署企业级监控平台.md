# 一、概述

在之前的文章《企业级监控平台如何选择？》中，我们了解到目前社区活跃度比较广，互联网大厂倾向的监控平台Promethes，今天我们就实战搭建一个企业级的监控平台，用来监控Linux服务器、MySQL数据库、Redis、Docker等。



需要搭建的环境有：

1. Docker：用来部署各个软件的基础环境
2. Prometheus：时序数据库
3. Grafana：大屏风格的Web可视化监控方案
4. AlertManager：配置告警选项
5. webhook：实现钉钉告警



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

## 3.5 Grafana配置Prometheus数据源

访问Grafana，添加Prometheus的数据源：

![image-20210202165340741](http://lovebetterworld.com/image-20210202165340741.png)

选择Prometheus。

![image-20210202165422307](http://lovebetterworld.com/image-20210202165422307.png)



添加HTTP URL地址，然后点击下方绿色按钮，若无报错则表示添加数据源成功。

![image-20210202165512846](http://lovebetterworld.com/image-20210202165512846.png)



# 四、服务器的监控

## 4.1 启动node-exporter

```bash
docker run -d --restart=always --net="host" --pid="host" --name=node-exporter -p 9100:9100 prom/node-exporter
```

- -d：后台启动
- --restart=always：重启策略
- --net="host"：使用本地host网络
- --pid="host"：进程相关
- --name：容器名称
- -p 9100:9100：端口映射

Linux服务器的监控，通过node-exporter来获取数据，在需要被监控的服务器上，启动node-exporter应用。



验证是否启动成功，通过curl命令：

```bash
curl http://127.0.0.1:9100/metrics
```

![image-20210202164855131](http://lovebetterworld.com/image-20210202164855131.png)



## 4.2 修改prometheus配置文件

在Prometheus的配置文件prometheus.yml中添加服务器的监控锚点：

```yaml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: '服务器监控'  
    static_configs:
    - targets: ['172.21.0.15:9100']
```

保存后重启prometheus：

```bash
docker restart prometheus
```

查看是否修改成功，通过访问prometheus的Web端进行查看：

访问地址：http://ip:9090/targets



![image-20210202165136201](http://lovebetterworld.com/image-20210202165136201.png)

可以看到服务器的监控成功。

## 4.3 Grafana添加模板

添加模板，选择Import的方式：

![image-20210202165743383](http://lovebetterworld.com/image-20210202165743383.png)



进入Grafana官网Dashboards，寻找有些的模板。

Grafana地址：https://grafana.com/grafana/dashboards

根据收集器筛选或使用其他条件进行过滤，找到对应的模板，点进去查看模板号。

![image-20210203085205771](http://lovebetterworld.com/image-20210203085205771.png)

复制对应的模板号，

![image-20210203085322449](http://lovebetterworld.com/image-20210203085322449.png)



将模板号填入Grafana中：

![image-20210203085405577](http://lovebetterworld.com/image-20210203085405577.png)

配置数据源：

![image-20210203085443327](http://lovebetterworld.com/image-20210203085443327.png)

点击下方红色按钮，然后就能看到监控面板：

![image-20210203085528237](http://lovebetterworld.com/image-20210203085528237.png)



# 五、MySQL数据库的监控

## 5.1 新建mysql用户并赋予权限

```bash
# 新建exporter用户并赋予相应权限
GRANT REPLICATION CLIENT, PROCESS ON  *.*  to 'exporter'@'%' identified by '123456';
GRANT SELECT ON performance_schema.* TO 'exporter'@'%';
# 刷新权限
flush privileges;
```

## 5.2 启动mysqld-exporter

```bash
docker run -d  --restart=always  --name mysqld-exporter -p 9104:9104   -e DATA_SOURCE_NAME="exporter:123456@(192.168.1.82:3306)/"   prom/mysqld-exporter
```

参数解释同上面服务器监控说明。

## 5.3 修改prometheus配置文件

修改添加prometheus.yml的配置文件

```yaml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: '服务器监控'  
    static_configs:
    - targets: ['172.21.0.15:9100']
  - job_name: 'MySQL数据库监控'  
    static_configs:
    - targets: ['172.21.0.15:9104']
```



## 5.4 添加Grafana模板

在Grafana官网找到合适的Dashboards，此处以某个Dashboards为例：12826

![image-20210203091423327](http://lovebetterworld.com/image-20210203091423327.png)



# 六、Redis集群的监控

## 6.1 启动redis-exporter

```bash
docker run -d --name redis_exporter -p 9121:9121 oliver006/redis_exporter --redis.addr redis://172.16.11.51:6379 --redis.password '123456'
```

参数解释：

1. –redis.addr 指定redis地址，由于这里使用docker起的服务，所以不能使用127.0.0.1地址。
2. –redis.password redis认证密码，如果没有密码，该参数不需要



## 6.2 修改prometheus配置文件

修改追加内容至prometheus.yml

```yaml
# 全局配置
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  # 通过node_exporter将监控数据传给prometheus，如果要监控多台服务器，只要在每个服务器上安装node_exporter，指定不同多ip地址就好了
  - job_name: '服务器监控'  
    static_configs:
    - targets: ['172.21.0.15:9100']
  - job_name: 'MySQL数据库监控'  
    static_configs:
    - targets: ['172.21.0.15:9104']
  # 监控redis集群，只要能连接到一个集群的一个节点，自然就能查询其他节点的指标了。
  - job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://172.18.11.139:7000
        - redis://172.18.11.139:7001
        - redis://172.18.11.140:7002
        - redis://172.18.11.140:7003
        - redis://172.18.11.141:7004
        - redis://172.18.11.141:7005
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.18.11.139:9121
```

## 6.3 添加Grafana模板

在Grafana官网找到合适的Dashboards，此处以某个Dashboards为例：4074

![image-20210203091715590](http://lovebetterworld.com/image-20210203091715590.png)

# 七、AlertManager告警配置

## 7.1 服务器存活，CPU，内存告警配置

修改并追加内容至prometheus：

```yaml
# prometheus配置文件中追加如下内容
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - ip:9093
# 指定告警规则文件路径
rule_files:
  - "/home/prometheus/node_down.yml"                 # 实例存活报警规则文件
  - "/home/prometheus/memory_over.yml"               # 内存报警规则文件
  - "/home/prometheus/cpu_over.yml"                  # cpu报警规则文件
```



附上三个告警文件中的具体内容：

- node_down.yml

```yaml
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
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

- memory_over.yml

```yaml
groups:
- name: 内存报警规则
  rules:
  - alert: 内存使用率告警
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes )) / node_memory_MemTotal_bytes * 100 > 80
    for: 1m
    labels:
      user: prometheus
      severity: warning
    annotations:
      description: "服务器: 内存使用超过80%！(当前值: {{ $value }}%)"
```

- cpu_over.yml

```yaml
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

配置成功后，重启prometheus。

```bash
docker restart prometheus
```

验证是否配置成功：

![image-20210203092728424](http://lovebetterworld.com/image-20210203092728424.png)

Prometheus Alert 告警状态有三种状态：Inactive、Pending、Firing。

- Inactive：非活动状态，表示正在监控，但是还未有任何警报触发。
- Pending：表示这个警报必须被触发。由于警报可以被分组、压抑/抑制或静默/静音，所以等待验证，一旦所有的验证都通过，则将转到 Firing 状态。
- Firing：将警报发送到 AlertManager，它将按照配置将警报的发送给所有接收者。一旦警报解除，则将状态转到 Inactive，如此循环。



## 7.2 AlertManager 配置自定义邮件模板

AlertManager 也是支持自定义邮件模板配置的，首先新建一个模板文件 email.tmpl。

```bash
$ mkdir -p /root/prometheus/alertmanager-tmpl && cd /root/prometheus/alertmanager-tmpl
$ vim email.tmpl
{{ define "email.from" }}xxxxxxxx@qq.com{{ end }}
{{ define "email.to" }}xxxxxxxx@qq.com{{ end }}
{{ define "email.to.html" }}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} 级 <br>
告警类型: {{ .Labels.alertname }} <br>
故障主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }} <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2019-08-04 16:58:15" }} <br>
=========end==========<br>
{{ end }}
{{ end }}
```

简单说明一下，上边模板文件配置了 email.from、email.to、email.to.html 三种模板变量，可以在  alertmanager.yml 文件中直接配置引用。这里 email.to.html 就是要发送的邮件内容，支持 Html 和 Text  格式，这里为了显示好看，采用 Html 格式简单显示信息。下边 {{ range .Alerts }} 是个循环语法，用于循环获取匹配的 Alerts 的信息，下边的告警信息跟上边默认邮件显示信息一样，只是提取了部分核心值来展示。然后，需要增加 alertmanager.yml 文件 templates 配置如下：

```yml
global:
  resolve_timeout: 5m
  smtp_from: '{{ template "email.from" . }}'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '{{ template "email.from" . }}'
  smtp_auth_password: 'xxxxxxxxxxxxxxx'
  smtp_require_tls: false
  smtp_hello: 'qq.com'
templates:
  - '/etc/alertmanager-tmpl/email.tmpl'
route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: '{{ template "email.to" . }}'
    html: '{{ template "email.to.html" . }}'
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

上边模板中由于配置了 {{ .Annotations.description }} 变量，而之前 node.rules 中并没有配置该变量，会导致获取不到值，所以这里我们修改一下 node.rules 并重启 Promethues 服务。

```yml
 $ vim /root/prometheus/rules/node-up.rules

groups:
- name: node-up
  rules:
  - alert: node-up
    expr: up{job="node-exporter"} == 0
    for: 15s
    labels:
      severity: 1
      team: node
    annotations:
      summary: "{{ $labels.instance }} 已停止运行!"
      description: "{{ $labels.instance }} 检测到异常停止！请重点关注！！！"
```

# 八、踩坑点

## 8.1 钉钉群组收不到告警信息

![image-20210203093206899](http://lovebetterworld.com/image-20210203093206899.png)

钉钉机器人的安全设置，我增加了IP限制，导致多次访问不成功，由于必须添加安全设置，所以添加自定义关键字的限制，添加了如下关键字：alert、promethues、alertmanager、webhook。可自行决定添加几个。



# Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)

