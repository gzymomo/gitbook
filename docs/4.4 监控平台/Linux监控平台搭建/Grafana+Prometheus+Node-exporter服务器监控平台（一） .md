- [Docker系列——Grafana+Prometheus+Node-exporter服务器监控平台（一）](https://www.cnblogs.com/hong-fithing/p/14695803.html)
- [Docker系列——Grafana+Prometheus+Node-exporter服务器告警中心（二）](https://www.cnblogs.com/hong-fithing/p/14797242.html)
- [Docker系列Grafana+Prometheus+Node-exporter微信推送（三）](https://www.cnblogs.com/hong-fithing/p/14820253.html)



# 一、Node-exporter

## 1.1 Node-exporter简介

在配置环境前，可能会有疑问，为什么需要？所以就先来讲下其作用。

在Prometheus的架构设计中，Prometheus  Server并不直接服务监控特定的目标，其主要任务负责数据的收集，存储并且对外提供数据查询支持。因此为了能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。

Node-exporter 与之前博文中的 Jmeter  是一个作用，就是用于机器系统数据收集。我们在展示数据时，数据肯定是有来源的，监控系统，监控的数据有：CPU、内存、磁盘、I/O等信息。所以就需要部署Node-exporter，我们一起来看下部署过程。

## 1.2 环境部署

### 1.2.1 拉取镜像

使用命令 docker pull prom/node-exporter 拉取镜像

使用命令如下命令启动即可：

```
docker run -d -p 9100:9100 \
 -v "/proc:/host/proc:ro" \
 -v "/sys:/host/sys:ro" \
 -v "/:/rootfs:ro" \
 --net="host" \
 prom/node-exporter
```

我们可以查看端口，是否有启动成功，命令 `netstat -anpt`

或者使用命令 `docker ps -a` 查看容器是否启用成功
 确认启动成功之后，可以通过url访问查看，服务器ip+9100；这里需要注意的是，服务器端口有安全策略，需要手动开启，开启后才能正常访问，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516191743642-502651600.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516191743642-502651600.png)

我们可以查看到具体的监听信息，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516191809234-546277592.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516191809234-546277592.png)

# 二、Prometheus

Prometheus（普罗米修斯）是一套开源的监控&报警&时间序列数据库的组合，起始是由SoundCloud公司开发的。随着发展，越来越多公司和组织接受采用Prometheus，社会也十分活跃，他们便将它独立成开源项目，并且有公司来运作。Google  SRE的书内也曾提到跟他们BorgMon监控系统相似的实现是Prometheus。现在最常见的Kubernetes容器管理系统中，通常会搭配Prometheus进行监控。

Prometheus基本原理是通过HTTP协议周期性抓取被监控组件的状态，这样做的好处是任意组件只要提供HTTP接口就可以接入监控系统，不需要任何SDK或者其他的集成过程。这样做非常适合虚拟化环境比如VM或者Docker 。

Prometheus应该是为数不多的适合Docker、Mesos、Kubernetes环境的监控系统之一。

输出被监控组件信息的HTTP接口被叫做exporter 。目前互联网公司常用的组件大部分都有exporter可以直接使用，比如Varnish、Haproxy、Nginx、MySQL、Linux 系统信息 (包括磁盘、内存、CPU、网络等等)。

与其他监控系统相比，Prometheus的主要特点是：

- 易于管理
- 监控服务的内部运行状态
- 强大的数据模型
- 强大的查询语言PromQL
- 高效
- 可扩展
- 易于集成
- 可视化
- 开放性

## 2.1 Prometheus架构

架构图如下所示：

[![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521193734422-462637317.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521193734422-462637317.png)

从架构图中，可以看出有几大核心组件，一起来看下。

### 2.1.1 Prometheus Server

Prometheus Server是Prometheus组件中的核心部分，负责实现对监控数据的获取，存储以及查询。 Prometheus  Server可以通过静态配置管理监控目标，也可以配合使用Service  Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。其次Prometheus  Server需要对采集到的监控数据进行存储，Prometheus  Server本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。最后Prometheus  Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。

Prometheus Server内置的Express Browser UI，通过这个UI可以直接通过PromQL实现数据的查询以及可视化。

Prometheus Server的联邦集群能力可以使其从其他的Prometheus Server实例中获取数据，因此在大规模监控的情况下，可以通过联邦集群以及功能分区的方式对Prometheus Server进行扩展。

### 2.1.2 Exporters

Exporter将监控数据采集的端点通过HTTP服务的形式暴露给Prometheus Server，Prometheus Server通过访问该Exporter提供的Endpoint端点，即可获取到需要采集的监控数据。

一般来说可以将Exporter分为2类：

- 直接采集：这一类Exporter直接内置了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
- 间接采集：间接采集，原有监控目标并不直接支持Prometheus，因此我们需要通过Prometheus提供的Client  Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

### 2.1.3 AlertManager

在Prometheus  Server中支持基于PromQL创建告警规则，如果满足PromQL定义的规则，则会产生一条告警，而告警的后续处理流程则由AlertManager进行管理。在AlertManager中我们可以与邮件，Slack等等内置的通知方式进行集成，也可以通过Webhook自定义告警处理方式。AlertManager即Prometheus体系中的告警处理中心。

### 2.1.4 PushGateway

由于Prometheus数据采集基于Pull模型进行设计，因此在网络环境的配置上必须要让Prometheus  Server能够直接与Exporter进行通信。  当这种网络需求无法直接满足时，就可以利用PushGateway来进行中转。可以通过PushGateway将内部网络的监控数据主动Push到Gateway当中。而Prometheus Server则可以采用同样Pull的方式从PushGateway中获取到监控数据。

## 2.2 环境部署

### 2.2.1 拉取镜像

使用命令 `docker pull prom/prometheus` 即可。

• 添加配置文件
 使用命令 `mkdir -p /opt/prometheus && cd /opt/prometheus` 新建文件夹，并进入到该目录下。新建配置文件的操作命令 `vim prometheus.yml` ，添加如下内容：

```
global:
  scrape_interval:     60s
  evaluation_interval: 60s
             
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['自己ip:9090']
        labels:
          instance: prometheus
                                                                       
  - job_name: linux
    static_configs:
      - targets: ['自己ip:9100']
        labels:
          instance: ip
```

• 启动服务
 使用如下命令，将配置文件挂载到容器中：



```
docker run  -d \
  -p 9090:9090 \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml  \
  prom/prometheus
```

启动后，可以检查是否启动成功，查看端口或者查看容器运行情况。

我们可以通过web页面查看，服务运行状态，访问地址：服务器ip+端口/targets，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193027928-1664347179.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193027928-1664347179.png)

# 三、Grafana配置

在之前的博文中，已经详细介绍过Grafana了，所以这里就不细说了，详细可以看之前的博文，[Docker系列——InfluxDB+Grafana+Jmeter性能监控平台搭建（三）](https://www.cnblogs.com/hong-fithing/p/14578602.html)

## 3.1 添加数据源

在使用过程中，稍有区别就是需要添加新的数据源，因为我们现在使用的是 Prometheus ，操作如下：

访问grafana：http://ip:3000/login  ，添加即可，操作如下：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193051277-1612222615.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193051277-1612222615.png)

在这里可以配置数据源名称，配置数据源的地址，地址填启动的prometheus地址即可。

## 3.2 引用模板

接下来就是添加模板，模板直接按id导入： `8919` ，可以配置监控面板的名称和引用源，添加即可。引用模板就是让数据展示的更美观。其他面板，也可以在Grafana官网去搜索，里面还有其他的模板数据。
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193124848-150623723.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210516193124848-150623723.png)

## 3.3 数据展示

配置完成后，查看面板数据如下：
 [![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195453174-284422602.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195453174-284422602.png)

[![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195548923-435817565.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195548923-435817565.png)

[![img](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195408172-674201206.png)](https://img2020.cnblogs.com/blog/1242227/202105/1242227-20210521195408172-674201206.png)

我们从图中看出，监控面板的数据很齐全，各种指标都有展示。