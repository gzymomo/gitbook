# Helm Template

来源：微信公众号：南瓜慢说

# 1 简介

Helm作为一个优秀的包管理器，这部分我们之前已经做了介绍，文章如下：

用Helm部署Kubernetes应用，支持多环境部署与版本回滚

Kubernetes用Helm安装Ingress并踩一下使用的坑

而Helm的模板功能，一样非常强大。它可以非常方便的定义各种Kubernetes的资源模板，如Deployment、Service、Ingress、ConfigMap等。不同环境的变量放在不同文件上，渲染时指定环境变量文件即可。

# 2 初体验

使用Helm的Template功能，需要先创建一个Chart，这是Helm的基本文件组成架构。我们来创建一个Nginx的相关资源文件，命令如下：

```bash
helm create pkslow-nginx
```

命令执行完成后，就会自动创建Chart的相关文件：

![图片](https://mmbiz.qpic.cn/mmbiz_png/6cDkia4YUdoO1M7SmuPf7JC3Hz0wbREN3m6M5Oat5mU1hAolfnRyiaDqgRun2nqTWpjycicFvNpbm7QyeuOBvPvbg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

关键文件：

- 目录template：放置模板文件，想要渲染什么文件出来，就在这个目录放置对应模板；
- 文件Chart.yaml：该Chart的描述，如果只是使用Helm的模板功能，可以不用管；
- 文件values.yaml：包含变量默认值。

`templates/tests`对我们作用不大，删掉。

我们尝试不修改模板、不添加变量，直接渲染出结果文件如下：

```bash
$ helm template pkslow-nginx/ --output-dir ./result
wrote ./result/pkslow-nginx/templates/serviceaccount.yaml
wrote ./result/pkslow-nginx/templates/service.yaml
wrote ./result/pkslow-nginx/templates/deployment.yaml
```

根据一些变量和判断，helm直接帮我们生成了三种资源的文件。查看其中一个文件service.yaml，还是非常完整的，基本可以满足需要了，再根据自己的需求改改就好了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/6cDkia4YUdoO1M7SmuPf7JC3Hz0wbREN3uFVU5Ah7TZoDQHslsyqa5a7GXoUkHDWYfWeLQJJhicyOoVkicHJLEYCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 3 添加模板文件

试着添加一个模板文件configmap.yaml到templates目录，内容如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pkslow-file
  namespace: default
data:
  application.yaml: |-
    server:
      port: 8080
    pkslow:
      name: Larry
      age: 18
      webSite: www.pkslow.com
```

执行命令后渲染的结果如下：

```yaml
---
# Source: pkslow-nginx/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pkslow-file
  namespace: default
data:
  application.yaml: |-
    server:
      port: 8080
    pkslow:
      name: Larry
      age: 18
      webSite: www.pkslow.com
```

与模板并没有什么不同，那是因为我们没有在模板文件里使用变量和判断语句等。

## 3.1 模板中使用变量

我们修改模板如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pkslow-config-{{ .Values.environment }}
  namespace: default
data:
  application.yaml: |-
    server:
      port: {{ .Values.server.port }}
    pkslow:
      name: {{ .Values.pkslow.name }}
      age: {{ .Values.pkslow.age }}
    {{- if .Values.pkslow.webSite }}
      webSite: {{ .Values.pkslow.webSite }}
    {{- end }}
```

可以看见我们在模板中使用了许多双大括号的变量`{{ .Values.xxx }}`，我们需要在values.yaml文件中定义这些变量，如下：

```yaml
environment: dev
server:
  port: 80
pkslow:
  name: Larry Deng
  age: 28
  webSite: https://www.pkslow.com
```

重新执行命令`$ helm template pkslow-nginx/ --output-dir ./result`，渲染的结果如下：

```yaml
---
# Source: pkslow-nginx/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: pkslow-config-dev
  namespace: default
data:
  application.yaml: |-
    server:
      port: 80
    pkslow:
      name: Larry Deng
      age: 28
      webSite: https://www.pkslow.com
```

## 3.2 为不同环境设置不同的变量

多环境管理在Helm Template这也是非常简单的，我们创建一个values-dev.yaml的变量文件，内容如下：

```yaml
environment: dev
server:
  port: 8080
pkslow:
  name: Larry Deng
  age: 1
```

通过以下命令来指定dev环境的变量文件：

```bash
$ helm template pkslow-nginx/ --output-dir ./result -f pkslow-nginx/values-dev.yaml
```

这样渲染出来的结果就是dev的相关配置了。其它环境同理。

## 3.3 通过命令行设置变量

使用`--set`或`--set-string`，使用如下：

```bash
$ helm template pkslow-nginx/ --output-dir ./result -f pkslow-nginx/values-dev.yaml --set pkslow.webSite=www.pkslow.com
```

# 总结

代码请查看：https://github.com/LarryDpk/pkslow-samples