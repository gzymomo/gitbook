- [Grafana微信报警](https://www.cnblogs.com/xiao987334176/p/12101506.html)



# 一、概述

由于grafana的多数据源特性，结合alertmanager实现微信报警。

 

# 二、注册企业微信

访问链接：

https://work.weixin.qq.com/wework_admin/register_wx

这里直接使用自己的微信，即可完成注册。不需要进行企业认证，也可以使用。

 

## 添加应用

点击应用管理-->创建应用

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226144527898-945341074.png)

 

 添加成功后，就可以看到 Agentld和Secret

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226144731596-1838922521.png)

 

 

点击右上角我的企业，就会看到企业id

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226144857573-1223249484.png)

 

 

点击通信录，查看成员详情

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226145005732-1881809766.png)

 

 

那么这4个信息，就是接下来要使用的了。

 

# 二、alertmanager

alertmanager为prometheus一个单独的报警模块，具有分组、抑制、静默等功能。

github地址：

https://github.com/prometheus/alertmanager

 

## 安装

登录到prometheus服务器

```
tar zxvf alertmanager-0.19.0.linux-amd64.tar.gz -C /data
mv /data/alertmanager-0.19.0.linux-amd64 /data/alertmanager
```

 

## 配置

```
cd /data/alertmanager/
vim grafana.yml
```

内容如下：

```yaml
global:
  resolve_timeout: 5m

templates:
- '/usr/local/alertmanager/wechat.tmpl'

route:
  group_by: ['alertname']
  group_wait: 5s
  #同一组内警报，等待group_interval时间后，再继续等待repeat_interval时间
  group_interval: 1m
  #当group_interval时间到后，再等待repeat_interval时间后，才进行报警
  repeat_interval: 10m
  receiver: 'wechat'
receivers:
- name: 'wechat'
  wechat_configs:
  - corp_id: 'wwbba17dd372e'
    agent_id: '1000005'
    api_secret: '-CJ9QLEFxLzx7wPgoK9Dt-NWYOLuy-RuX3I'
    to_user: 'yangguangda'
    send_resolved: true
```

 

corp_id：企业id

agent_id：应用Agentld

api_secret：应用Secret

to_user：通讯录人员

 

报警再次发送时间为group_interval+repeat_interval，也就是先等待group_interval，再等待repeat_interval。

注意：企业号新建应用的须设置相应的可见范围及人员，否则无法发送信息。



## 报警模板

```
cd /data/alertmanager
vim wechat.tmpl
```

内容如下：

```
{{ define "grafana.default.message" }}{{ range .Alerts }}
{{ .StartsAt.Format "2006-01-02 15:03:04" }}
{{ range .Annotations.SortedPairs }}{{ .Name }} = {{ .Value }}
{{ end }}{{ end }}{{ end }}

{{ define "wechat.default.message" }}
{{ if eq .Status "firing"}}[Warning]:{{ template "grafana.default.message" . }}{{ end }}
{{ if eq .Status "resolved" }}[Resolved]:{{ template "grafana.default.message" . }}{{ end }}
{{ end }}
```

其中：
Status 只有两个状态firing、resolved，通过这个参数是否发送warning和resolved报警信息。

模板的语法还需查官网进行深入学习。

注意： prometheus 默认时区为UTC且无法改变时区，官方建议在用户的web ui 中重新设置时区，因此我们的报警时间应该+8:00



## 启动

```
cd /data/alertmanager
nohup /data/alertmanager/alertmanager --config.file=/data/alertmanager/grafana.yml --storage.path=/data/alertmanager/data/ --log.level=debug &
```

 

启动后，可通过ip:9093 访问alertmanager界面。

 

# 三、grafana设置

添加报警渠道

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226150019261-543126802.png)

 

 

其中include image 没有作用；
Disable Resolve Message 没有勾选，但不发送报警取消信息；

我是在alertmanager 模板中判断若Status没有firing（则为resolved），则发送报警解决信息。

 

在dashboard中设置alert

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226150130386-307999297.png)

 

 当报警时会发送给alertmanager。

 

微信报警如下

时间为UTC时区，而不是CST时区，因此我们需要自行+8:00

![img](https://img2018.cnblogs.com/i1/1341090/201912/1341090-20191226150308583-1317558217.png)

 

 

**注意：只有企业微信才能收到报警信息，普通微信是收不到的。**

**这个是腾讯故意设置的，为了工作和生活分开。**

**所以，你需要其他人接收报警信息，那么他们也需要下载企业微信才可以。**

 

本文参考链接：

https://blog.csdn.net/yanggd1987/article/details/95204976