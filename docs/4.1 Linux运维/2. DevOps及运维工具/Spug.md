- [开源、轻量无 Agent 自动化运维平台](https://mp.weixin.qq.com/s/qofHTpYT0T5KG4cqqvciww)

# Spug开源、轻量无 Agent 自动化运维平台

在日常运维管理的发展过程中，可视化、自动化是一个阶段的进程必备要素，所以，对于可视化运维平台的掌握与了解也非常重要，我们运维小伙伴们也在不断的探索与挖掘当中，今天，民工哥给大家安利一款可视化的自动化运维管理平台：Spug，开源、免费，功能强大。

# Spug简介

Spug面向中小型企业设计的轻量级无 Agent 的自动化运维平台，整合了主机管理、主机批量执行、主机在线终端、文件在线上传下载、应用发布部署、在线任务计划、配置中心、监控、报警等一系列功能。

- 代码仓库地址：https://github.com/openspug/spug.dev
- 官网地址：https://www.spug.dev
- 使用文档：https://www.spug.dev/docs/about-spug/
- 更新日志：https://www.spug.dev/docs/change-log/
- 常见问题：https://www.spug.dev/docs/faq/

# Spug的功能

- 批量执行: 主机命令在线批量执行
- 在线终端: 主机支持浏览器在线终端登录
- 文件管理: 主机文件在线上传下载
- 任务计划: 灵活的在线任务计划
- 发布部署: 支持自定义发布部署流程
- 配置中心: 支持 KV、文本、json 等格式的配置
- 监控中心: 支持站点、端口、进程、自定义等监控
- 报警中心: 支持短信、邮件、钉钉、微信等报警方式
- 优雅美观: 基于 Ant Design 的 UI 界面
- 开源免费: 前后端代码完全开源

# 安装环境要求

- Python 3.6+
- Django 2.2
- Node 12.14
- React 16.11

# 安装Spug

简化一切安装操作步骤，官方也建议使用docker进行安装，那么，接下来就使用docker来安装这款工具平台。本文操作基于Centos7.x操作系统。

**1. 安装docker并启动**

```bash
yum install docker -y
systemctl start docker
```

**2. 拉取镜像**

阿里云的镜像与 Docker hub 同步更新，国内用户建议使用阿里云的镜像。

```bash
$ docker pull registry.aliyuncs.com/openspug/spug
```

**3. 启动容器**

Docker镜像内部使用的 Mysql 数据库。如果需要持久化存储代码和数据，可以添加：-v 映射容器内/data路径

```bash
$ docker run -d --name=spug -p 80:80 registry.aliyuncs.com/openspug/spug

# 持久化存储启动命令：
# mydata是本地磁盘路径，/data是容器内代码和数据初始化存储的路径
$ docker run -d --name=spug -p 80:80 -v /mydata/:/data registry.aliyuncs.com/openspug/spug
```

**4. 初始化**

以下操作会创建一个用户名为 admin 密码为 spug.dev 的管理员账户，可自行替换管理员账户。

```bash
$ docker exec spug init_spug admin spug.dev

# 执行完毕后需要重启容器
$ docker restart spug
```

**5. 访问测试**

在浏览器中输入 http://localhost:80 访问,用户名：admin  密码：spug.dev

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUo81NQZP6kzaDenTYKtr3hPd4GLibDS2MNVXNLBjG8UyMWwKMeQ7XH0w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 使用介绍

登录完成后，就可以看到主界面，如下

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaU5xGuQ5d6bsAftnxKibXZ8LSicSgpiawFWsSo01YicjNInbYGkswRQu2Y1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**主机管理**

管理维护平台可操作的主机，首次添加主机时需要输入 ssh 指定用户的密码。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUTABRkAx0UTjhR62TwDHXDWg8qYYibFY53b5lnXeFVY45rOGWW0jmiaoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**批量执行**

包含维护命令模版和批量远程执行命令两部分功能，常用来执行一些临时的任务例如，批量安装/卸载某个依赖包等。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUFJnEv50QwCyS7s8Y1HlNG4M9v8iaF0AAf2IWlTVaPhu7RhjnNOOyMsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 执行任务

可以选择一到多个在主机管理中添加的主机作为执行的目标主机，命令内容可以直接写也支持从模板中读取已保存的命令。

- 模板管理

用于存储复杂、常用的命令集合，以便后期可随时使用。

**应用发布**

- 应用管理

管理和维护可发布的应用。每个应用又可以创建对应环境的发布配置，发布配置请查看发布配置文档。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaU6Ij1qdBv4hmEJ1XNso4EPYw0eCEehafZAkCeOlS4IN7mkvmJte71SA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 发布配置

配置指定应用在某环境下如何执行发布，发布支持两种方式 常规发布 和 自定义发布。

- 发布申请

创建和执行发布。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUDzATicISicoPOXv7Ce80VX6coT0iaCONZ2lyHNxVhsEWn1HhAiblq40LicA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**配置中心**

- 环境管理

管理应用的运行环境，一般包含开发环境、测试环境和生产环境，应用发布和配置管理需要用它来区分不同的环境。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUM6VyfR5q9CmloSx7fba2UG0wLcMsUciaiaWAGjdRjibibtRialnXmnNlsng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 服务管理

管理和维护应用依赖的服务配置。例如有两个应用 A 和应用 B，它们共同使用一个数据库，那么就可以把这个数据库提取出来作为单独的服务来管理。这样带来的好处是如果这个数据库配置变更了，那么只需要在服务管理里把这个数据库的配置更新即可，不必在多个应用之间切换查找更新。

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUWetSJmDV6dPAGxRmmgYemw6yMoHeNtuSWCRXJvzMicSNfgcJ3A9cB0w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 应用管理

用于维护应用的配置，应用配置包含 公共 和 私有 两种类型的配置。

- 配置管理

用户维护服务和应用在不同环境下的具体配置。

**任务调度**

维护一些周期性的任务

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUV5xmCWsy0qz5lZrNpR83vjiaTyVfFpDykrvLNs1kLd1JCNJEtBf2l4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**监控中心**

该模块提供了以下几种常用的监控模式

- 站点检测

通过 GET 请求指定的 url 匹配返回的状态码来确定站点是否异常

- 端口检测

检测指定目标主机的端口是否可以正常建立接连

- 进程检测

检测指定目标主机的某个进程是否存活

- 自定义脚本检测

在指定主机上运行自定义的脚本，通过判断返回的退出状态码来确定是否有异常

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaU2ZUfgoE4fRPb3X3QbQicGEWOmeFDDzkZQrkQ48WDkTmA4XqCtC0K92A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**报警中心**

配置与维护日常报警相关，如:报警记录、报警联系人与组

![img](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrT9SOVe6COTaltORdGkVaUmLx6NEs1cl7MQ06oeJa1tOlo9cbD4J3jhojEmLoaNv5DKQp4XrZ3hw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**系统管理**

除了页面上对普通用的管理，Spug 还提供了 manage.py user 命令可用于管理员账户的管理操作。

- 创建账户

创建账户使用 manage.py user add 命令，用法示例如下

```bash
$ cd spug/spug_api
$ source venv/bin/activate
$ python manage.py user add -u admin -p 123 -n 民工哥 -s
```

Docker 安装的可以执行如下命令

```bash
$ docker exec spug python3 /data/spug/spug_api/manage.py user add -u admin -p 123 -n 民工哥 -s
#上面的命令会创建个登录名为 admin 密码为 123 昵称为 民工哥 的管理员账户，注意最后的 -s 参数，如果携带了这个参数意味着该账户为管理员账户， 管理员账户可以不受任何限制的访问所有功能模块。
```

- 重置密码

使用 manage.py user reset 命令来重置账户密码，用法示例如下

```bash
$ cd spug/spug_api
$ source venv/bin/activate
$ python manage.py user reset -u admin -p abc
```

Docker 安装的可以执行如下命令

```bash
$ docker exec spug python3 /data/spug/spug_api/manage.py user reset -u admin -p abc
#上述操作会重置登录名为 admin 的账户的密码为 abc。
```

- 启用账户

当页面上登录连续错误数次超过3次后账户自动转为禁用状态，普通用户可以通过 系统管理 / 账户管理 在页面是启用账户即可，但管理员账户需要使用如下命令来启用

```bash
$ cd spug/spug_api
$ source venv/bin/activate
$ python manage.py user enable -u admin
```

Docker 安装的可以执行如下命令

```bash
$ docker exec spug python3 /data/spug/spug_api/manage.py user enable -u admin
```

