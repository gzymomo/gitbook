- [Docker系列——Grafana+Prometheus+Node-exporter服务器监控平台（一）](https://www.cnblogs.com/hong-fithing/p/14695803.html)
- [Docker系列——Grafana+Prometheus+Node-exporter服务器告警中心（二）](https://www.cnblogs.com/hong-fithing/p/14797242.html)
- [Docker系列Grafana+Prometheus+Node-exporter微信推送（三）](https://www.cnblogs.com/hong-fithing/p/14820253.html)



在前一篇博文中介绍，服务器监控已经部署成功。如果每天都需要人去盯着服务情况，那也不太现实。既然监控平台已经部署好了，是不是可以自动触发报警呢？

在上一篇Prometheus架构中有讲到，核心组件之一：AlertManager，AlertManager即Prometheus体系中的告警处理中心。所以实现告警功能，可以使用该组件，具体如何实现，我们来看。

# 一、AlertManager配置

## 1.1 服务部署

### 1.1.1 拉取镜像

使用命令 `docker pull prom/alertmanager:latest`

使用如下命令：

```
docker run -d --name alertmanager -p 9093:9093 \
prom/alertmanager:latest
```

启动服务后，通过地址访问，http://:9093可以看到默认提供的 UI 页面，不过现在是没有任何告警信息的，因为我们还没有配置报警规则来触发报警。
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527132532142-999938380.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527132532142-999938380.png)

## 1.2 配置文件

AlertManager 默认配置文件为 alertmanager.yml，在容器内路径为 `/etc/alertmanager/alertmanager.yml`，默认配置如下：

```
global:
  resolve_timeout: 5m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

简单介绍一下主要配置的作用：
 • global: 全局配置，包括报警解决后的超时时间、SMTP 相关配置、各种渠道通知的 API 地址等等。
 • route: 用来设置报警的分发策略，它是一个树状结构，按照深度优先从左向右的顺序进行匹配。
 • receivers: 配置告警消息接受者信息，例如常用的 email、wechat、slack、webhook 等消息通知方式。
 • inhibit_rules: 抑制规则配置，当存在与另一组匹配的警报（源）时，抑制规则将禁用与一组匹配的警报（目标）。



## 1.3 告警规则

alertmanager配置好后，我们来添加告警规则，就是符合规则，才会推送消息，规则配置如下：

```
mkdir -p /root/prometheus/rules && cd /root/prometheus/rules/
vim host.rules
```

添加如下信息：

```
groups:
- name: node-up
  rules:
  - alert: node-up
    expr: up{job="linux"} == 0
    for: 15s
    labels:
      severity: 1
      team: node
    annotations:
      summary: "{{ $labels.instance }} 已停止运行超过 15s！"
```

说明一下：该 `rules` 目的是监测 `node` 是否存活，expr 为 PromQL 表达式验证特定节点 `job="linux"` 是否活着，for 表示报警状态为 Pending 后等待 15s 变成 Firing 状态，一旦变成 Firing 状态则将报警发送到  AlertManager，labels 和 annotations 对该 alert 添加更多的标识说明信息，所有添加的标签注解信息，以及  prometheus.yml 中该 job 已添加 label 都会自动添加到邮件内容中 ，[参考规则](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/#rule)。

## 1.4 prometheus.yml配置文件

修改prometheus.yml配置文件，添加 rules 规则文件，已有内容不变，配置文件中添加如下内容：

```
alerting:
  alertmanagers:
    - static_configs:
      - targets: 
        - 'ip:9093'
rule_files:
  - "/etc/prometheus/rules/*.rules"
```

注意: 这里rule_files为容器内路径，需要将本地host.rules文件挂载到容器内指定路径，修改 Prometheus 启动命令如下，并重启服务。

```
docker run  -d --name prometheus \
  -p 9090:9090 \
  -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \
  -v /root/prometheus/rules/:/etc/prometheus/rules/  \
prom/prometheus
```

通过ip+端口访问，查看rules，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527165113048-1743607434.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527165113048-1743607434.png)

这里说明一下 Prometheus Alert 告警状态有三种状态：Inactive、Pending、Firing。

- Inactive：非活动状态，表示正在监控，但是还未有任何警报触发。
- Pending：表示这个警报必须被触发。由于警报可以被分组、压抑/抑制或静默/静音，所以等待验证，一旦所有的验证都通过，则将转到 Firing 状态。
- Firing：将警报发送到 AlertManager，它将按照配置将警报的发送给所有接收者。一旦警报解除，则将状态转到 Inactive，如此循环。

说了这么多，就用推送邮件来实践下结果。

# 二、邮件推送

## 2.1 邮件配置

我们来配置一下使用 Email 方式通知报警信息，这里以 QQ 邮箱为例，新建目录 `mkdir -p /root/prometheus/alertmanager/ && cd /root/prometheus/alertmanager/`

使用vim命令 `vim alertmanager.yml`，配置文件中添加如下内容：

```
global:
  resolve_timeout: 5m
  smtp_from: '11111111@qq.com'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '11111111@qq.com'
  smtp_auth_password: 'XXXXXXXXX'
  smtp_hello: 'qq.com'
  smtp_require_tls: false
route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 5s
  repeat_interval: 5m
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: '222222222@foxmail.com'
    send_resolved: true
    #insecure_skip_verify: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

其中几个关键的配置说明一下：

- smtp_smarthost: 这里为 QQ 邮箱 SMTP 服务地址，官方地址为 smtp.qq.com 端口为 465 或 587，同时要设置开启 POP3/SMTP 服务。
- smtp_auth_password: 这里为第三方登录 QQ 邮箱的授权码，非 QQ 账户登录密码，否则会报错，获取方式在 QQ 邮箱服务端设置开启 POP3/SMTP 服务时会提示。
- smtp_require_tls: 是否使用 tls，根据环境不同，来选择开启和关闭。如果提示报错 email.loginAuth  failed: 530 Must issue a STARTTLS command first，那么就需要设置为  true。着重说明一下，如果开启了 tls，提示报错 starttls failed: x509: certificate signed by  unknown authority，需要在 email_configs 下配置 insecure_skip_verify: true 来跳过  tls 验证。

## 2.2 重启AlertManager

修改 AlertManager 启动命令，将本地alertmanager.yml文件挂载到容器内指定位置，是配置生效，命令如下所示：



```
docker run -d --name alertmanager -p 9093:9093 \
-v /root/prometheus/alertmanager/:/etc/alertmanager/ \
prom/alertmanager:latest
```

## 2.3 触发报警

之前我们定义的 rule 规则为监测 job="linux" Node 是否活着，那么就可以停掉node-exporter服务来间接起到 Node Down 的作用，从而达到报警条件，触发报警规则。

使用命令 `docker stop 容器id`，停止服务后，等待 60s 之后可以看到 Prometheus  target 里面 linux 状态为 unhealthy 状态，等待 60s 后，alert 页面由绿色 node-up (0 active) Inactive 状态变成了黄色 node-up (1 active) Pending 状态，继续等待 60s 后状态变成红色 Firing  状态，向 AlertManager 发送报警信息，此时 AlertManager 则按照配置规则向接受者发送邮件告警。

停掉服务后，我们来看状态的变化，首先是Inactive状态，AlertManager也没有报警信息，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170318065-298108447.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170318065-298108447.png)

等待60s后，再次查看服务状态，变成了Pending状态，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170352854-1301816517.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170352854-1301816517.png)

继续等待 60s，变成了Firing状态，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170431648-53912866.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170431648-53912866.png)

并且AlertManager 有报警信息，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527173029262-1777313371.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527173029262-1777313371.png)

查看自己的邮件，收到了邮件推送，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170745980-1450183992.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527170745980-1450183992.png)

服务一直处于停止状态，会一直推送消息，5分钟一次，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527171034126-1417000951.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527171034126-1417000951.png)

说到这里，对有些时间节点有点不理解，这里有几个地方需要解释一下：
 • 每次停止/恢复服务后，60s 之后才会发现 Alert 状态变化，是因为 prometheus.yml中 global ->  scrape_interval: 60s 配置决定的，如果觉得等待 60s 时间太长，可以修改小一些，可以全局修改，也可以局部修改。例如局部修改 linux 等待时间为 5s。
 • Alert 状态变化时会等待 15s 才发生改变，是因为host.rules中配置了for: 15s状态变化等待时间。
 • 报警触发后，每隔 5m 会自动发送报警邮件(服务未恢复正常期间)，是因为alertmanager.yml中route -> repeat_interval: 5m配置决定的。

# 三、邮件自定义

在刚才的邮件内容中，基本信息有，但不直观，那可不可以自定义模板内容呢？答案是有的，我们继续来看。

## 3.1 自定义模板

自定义一个邮件模板，在/root/prometheus/alertmanager/目录下，vim email.tmpl配置如下：



```
{{ define "email.from" }}1111111111@qq.com{{ end }}
{{ define "email.to" }}222222222222@foxmail.com{{ end }}
{{ define "email.html" }}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} 级 <br>
告警类型: {{ .Labels.alertname }} <br>
故障主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }} <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ .StartsAt.Format "2006-01-02 08:08:08" }} <br>
=========end==========<br>
{{ end }}
{{ end }}
```

简单说明一下，上边模板文件配置了 email.from、email.to、email.to.html 三种模板变量，可以在  alertmanager.yml 文件中直接配置引用。这里 email.to.html 就是要发送的邮件内容，支持 Html 和 Text  格式，这里为了显示好看，采用 Html 格式简单显示信息。下边 {{ range .Alerts }} 是个循环语法，用于循环获取匹配的  Alerts 的信息，下边的告警信息跟上边默认邮件显示信息一样，只是提取了部分核心值来展示。

## 3.2 修改alertmanager.yml

由于已经定义了变量，所以我们在alertmanager配置文件中可以引用变量，并且引用我们自定义的模板，引用模板需要增加 templates ，配置如下：



```
global:
  resolve_timeout: 5m
  smtp_from: '{{ template "email.from" . }}'
  smtp_smarthost: 'smtp.qq.com:465'
  smtp_auth_username: '{{ template "email.from" . }}'
  smtp_auth_password: 'XXXXXXXXXXXXXXXXX'
  smtp_hello: 'qq.com'
  smtp_require_tls: false
templates:
  - '/etc/alertmanager/*.tmpl'
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
    html: '{{ template "email.html" . }}'
    send_resolved: true
    #insecure_skip_verify: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

## 3.3 重启Alertmanager

修改 AlertManager 启动命令，将本地email.tmpl文件挂载到容器内指定位置并重启。由于我的配置是跟alertmanager的配置文件在同一个目录下，所以不用重新挂载，重启容器即可。
 我们将node服务停止，再次查收邮件，查看下效果，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172028784-537436645.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172028784-537436645.png)

# 四、模板优化

## 4.1 时间格式

我们从上图可以看出，邮件内容格式已经改变，但时间却显示的有点离谱，原因是时间格式问题，修改邮件模板，针对时间配置格式，如下所示：



```
{{ define "email.from" }}11111111111@qq.com{{ end }}
{{ define "email.to" }}2222222222222@foxmail.com{{ end }}
{{ define "email.html" }}
{{ range .Alerts }}
=========start==========<br>
告警程序: prometheus_alert <br>
告警级别: {{ .Labels.severity }} 级 <br>
告警类型: {{ .Labels.alertname }} <br>
故障主机: {{ .Labels.instance }} <br>
告警主题: {{ .Annotations.summary }} <br>
告警详情: {{ .Annotations.description }} <br>
触发时间: {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }} <br>
=========end==========<br>
{{ end }}
{{ end }}
```

保存后再次重启alertmanager服务，重新操作一遍之前的动作，查看最终的邮件效果，如下所示：

[![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172622941-1805257306.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172622941-1805257306.png)

## 4.2 邮件标题

配置时间格式后，我们看效果图，时间是修正了的。

还可以自定义邮件标题，修改alertmanager.yml配置文件，增加参数：headers即可，如下所示：

```
receivers:
- name: 'email'
  email_configs:
  - to: '{{ template "email.to" . }}'
    html: '{{ template "email.html" . }}'
    send_resolved: true
    headers: { Subject: "{{ .CommonAnnotations.summary }}" }
```

配置好重启alertmanager服务，再次触发告警邮件，收到的内容如下：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172755680-1426377777.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210527172755680-1426377777.png)