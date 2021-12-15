

SkyWalking 是一个应用性能监控系统，特别为微服务、云原生和基于容器（Docker, Kubernetes, Mesos）体系结构而设计。除了应用指标监控以外，它还能对分布式调用链路进行追踪。类似功能的组件还有：Zipkin、**Pinpoint**、CAT等。

上几张图，看看效果，然后再一步一步搭建并使用

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202160145679-514234773.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202160203917-953663055.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202160244752-574421843.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202160334276-2025039699.png)

# 一、概念与架构

**背景**

微服务架构是通过业务来划分服务的，使用 REST  调用。对外暴露的一个接口，可能需要很多个服务协同才能完成这个接口功能，如果链路上任何一个服务出现问题或者网络超时，都会形成导致接口调用失败。为了在发生故障的时候，能够快速定位和解决问题，需要使用SkyWalking。



SkyWalking是一个开源监控平台，用于从服务和云原生基础设施收集、分析、聚合和可视化数据。SkyWalking提供了一种简单的方法来维护分布式系统的清晰视图，甚至可以跨云查看。它是一种现代APM，专门为云原生、基于容器的分布式系统设计。 

<font color='red'>SkyWalking从三个维度对应用进行监视：service（服务）, service instance（实例）, endpoint（端点）</font>

服务和实例就不多说了，端点是服务中的某个路径或者说URI

> 通过SkyWalking，用户可以了解服务与端点之间的拓扑关系，查看每个服务/服务实例/端点的指标，并设置告警规则。

SkyWalking允许用户了解服务和端点之间的拓扑关系，查看每个服务/服务实例/端点的度量，并设置警报规则。



## 1.1 什么是APM系统

APM (Application Performance Management) 即应用性能管理系统，是对企业系统即时监控以实现
对应用程序性能管理和故障管理的系统化的解决方案。应用性能管理，主要指对企业的关键业务应用进
行监测、优化，提高企业应用的可靠性和质量，保证用户得到良好的服务，降低IT总拥有成本。
APM系统是可以帮助理解系统行为、用于分析性能问题的工具，以便发生故障的时候，能够快速定位和
解决问题。

说白了就是随着微服务的的兴起，传统的单体应用拆分为不同功能的小应用，用户的一次请求会经过多个系统，不同服务之间的调用非常复杂，其中任何一个系统出错都可能影响整个请求的处理结果。为了解决这个问题，Google 推出了一个分布式链路跟踪系统 Dapper ，之后各个互联网公司都参照Dapper  的思想推出了自己的分布式链路跟踪系统，而这些系统就是分布式系统下的APM系统。

目前市面上的APM系统有很多，比如skywalking、pinpoint、zipkin等。其中

- **[Zipkin](http://zipkin.io/)**：由Twitter公司开源，开放源代码分布式的跟踪系统，用于收集服务的定时数据，以解决微服务架构中的延迟问题，包括：数据的收集、存储、查找和展现。
- **[Pinpoint](https://github.com/naver/pinpoint)**：一款对Java编写的大规模分布式系统的APM工具，由韩国人开源的分布式跟踪组件。
- **[Skywalking](http://skywalking.org/)**：国产的优秀APM组件，是一个对JAVA分布式应用程序集群的业务运行情况进行追踪、告警和分析的系统。



## 1.2 架构

整体架构包含如下三个组成部分：

1. 探针(agent)负责进行数据的收集，包含了Tracing和Metrics的数据，agent会被安装到服务所在的服务器上，以方便数据的获取。
2. 可观测性分析平台OAP(Observability Analysis  Platform)，接收探针发送的数据，并在内存中使用分析引擎（Analysis  Core)进行数据的整合运算，然后将数据存储到对应的存储介质上，比如Elasticsearch、MySQL数据库、H2数据库等。同时OAP还使用查询引擎(Query Core)提供HTTP查询接口。
3. Skywalking提供单独的UI进行数据的查看，此时UI会调用OAP提供的接口，获取对应的数据然后进行展示。

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202162519099-1509163924.png) 

<font color='red'>SkyWalking逻辑上分为四个部分：Probes（探针）, Platform backend（平台后端）, Storage（存储）, UI。</font>

这个结构就很清晰了，探针就是Agent负责采集数据并上报给服务端，服务端对数据进行处理和存储，UI负责展示

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202163020279-47413713.jpg)

### 架构图：

![img](https://img2020.cnblogs.com/blog/1341090/202008/1341090-20200819165510536-432360122.png)



## 1.3 功能

skywalking提供了在很多不同的场景下用于观察和监控分布式系统的方式。
首先，像传统的方法，skywalking为java,c#,Node.js等提供了自动探针代理.同时，它为Go,C++提供了手工探针。
随着本地服务越来越多，需要越来越多的语言，掌控代码的风险也在增加，Skywalking可以使用网状服务探针收集数据，以了解整个分布式系统。
通常，skywalking提供了观察service,service instance,endpoint的能力。

- service: 一个服务
- Service Instance: 服务的实例(1个服务会启动多个节点)
- Endpoint: 一个服务中的其中一个接口



# 二、下载与安装

SkyWalking有两中版本，ES版本和非ES版。如果我们决定采用ElasticSearch作为存储，那么就下载es版本。 

https://skywalking.apache.org/downloads/

https://archive.apache.org/dist/skywalking/

## 2.1 Linux安装

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202163641739-1296053361.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202164208326-2131290371.png)

- agent目录将来要拷贝到各服务所在机器上用作探针

- bin目录是服务启动脚本

- config目录是配置文件

- oap-libs目录是oap服务运行所需的jar包

- webapp目录是web服务运行所需的jar包

接下来，要选择存储了，支持的存储有：

- H2
- ElasticSearch 6, 7
- MySQL
- TiDB
- **InfluxDB**

作为监控系统，首先排除H2和MySQL，这里推荐InfluxDB，它本身就是时序数据库，非常适合这种场景。

但是InfluxDB我不是很熟悉，所以这里先用ElasticSearch7

https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/backend-storage.md

### 2.1.1 安装ElasticSearch

https://www.elastic.co/guide/en/elasticsearch/reference/7.10/targz.html 

```bash
# 启动
./bin/elasticsearch -d -p pid
# 停止
pkill -F pid
```

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202170103327-1062492306.png)

ElasticSearch7.x需要Java 11以上的版本，但是如果你设置了环境变量JAVA_HOME的话，它会用你自己的Java版本

通常，启动过程中会报以下三个错误：

```bash
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
```

解决方法：

在 /etc/security/limits.conf 文件中追加以下内容：

```bash
* soft nofile 65536
* hard nofile 65536
* soft nproc  4096
* hard nproc  4096
```

可通过以下四个命令查看修改结果：

```bash
ulimit -Hn
ulimit -Sn
ulimit -Hu
ulimit -Su
```

修改 /etc/sysctl.conf 文件，追加以下内容：

```bash
vm.max_map_count=262144
```

修改es配置文件 elasticsearch.yml 取消注释，保留一个节点

```bash
cluster.initial_master_nodes: ["node-1"]
```

为了能够ip:port方式访问，还需修改网络配置

```bash
network.host: 0.0.0.0
```

修改完是这样的：

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202171507677-55826230.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202171617529-1017295018.png)

至此，ElasticSearch算是启动成功了

接下来，在 config/application.yml 中配置es地址即可

```yaml
storage:
  selector: ${SW_STORAGE:elasticsearch7}
  elasticsearch7:
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:192.168.100.19:9200}
```

### 2.1.2 安装Agent

https://github.com/apache/skywalking/blob/v8.2.0/docs/en/setup/service-agent/java-agent/README.md

将agent目录拷贝至各服务所在的机器上

```bash
scp -r ./agent chengjs@192.168.100.12:~/
```

这里，我将它拷贝至各个服务目录下

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202172252276-1789031720.png)

plugins是探针用到各种插件，SkyWalking插件都是即插即用的，可以把optional-plugins中的插件放到plugins中 

修改 agent/config/agent.config 配置文件，也可以通过命令行参数指定

主要是配置服务名称和后端服务地址

```bash
agent.service_name=${SW_AGENT_NAME:user-center}
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:192.168.100.17:11800}
```

当然，也可以通过环境变量或系统属性的方式来设置，例如：

```bash
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800
```

最后，在服务启动的时候用命令行参数 -javaagent 来指定探针

```bash
java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -jar yourApp.jar
```

例如：

```bash
java -javaagent:./agent/skywalking-agent.jar -Dspring.profiles.active=dev -Xms512m -Xmx1024m -jar demo-0.0.1-SNAPSHOT.jar
```



## 2.2 Windows安装

windows
 JDK8

**开发工具:**
 idea



# 三、启动服务

修改 webapp/webapp.yml 文件，更改端口号及后端服务地址

```yaml
server:
  port: 8080

collector:
  path: /graphql
  ribbon:
    ReadTimeout: 10000
    # Point to all backend's restHost:restPort, split by ,
    listOfServers: 127.0.0.1:12800
```

启动服务

```bash
bin/startup.sh
```

或者分别依次启动

```bash
bin/oapService.sh
bin/webappService.sh
```

查看logs目录下的日志文件，看是否启动成功

浏览器访问 http://127.0.0.1:8080

# 四、告警

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202175500721-1876340234.png) 

编辑 alarm-settings.yml 设置告警规则和通知

https://github.com/apache/skywalking/blob/v8.2.0/docs/en/setup/backend/backend-alarm.md

重点说下告警通知

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202180721755-1194205245.png)

![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202175909064-1593166227.png) 

为了使用钉钉机器人通知，接下来，新建一个项目



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.wt.monitor</groupId>
    <artifactId>skywalking-alarm</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>skywalking-alarm</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>alibaba-dingtalk-service-sdk</artifactId>
            <version>1.0.1</version>
        </dependency>

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.15</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



可选依赖（不建议引入）

```xml
<dependency
    <groupId>org.apache.skywalking</groupId>
    <artifactId>server-core</artifactId>
    <version>8.2.0</version>
</dependency>
```

定义告警消息实体类

```java
package com.wt.monitor.skywalking.alarm.domain;

import lombok.Data;

import java.io.Serializable;

/**
 * @author ChengJianSheng
 * @date 2020/12/1
 */
@Data
public class AlarmMessageDTO implements Serializable {

    private int scopeId;

    private String scope;

    /**
     * Target scope entity name
     */
    private String name;

    private String id0;

    private String id1;

    private String ruleName;

    /**
     * Alarm text message
     */
    private String alarmMessage;

    /**
     * Alarm time measured in milliseconds
     */
    private long startTime;

}
```



发送钉钉机器人消息

```java
package com.wt.monitor.skywalking.alarm.service;

import com.dingtalk.api.DefaultDingTalkClient;
import com.dingtalk.api.DingTalkClient;
import com.dingtalk.api.request.OapiRobotSendRequest;
import com.taobao.api.ApiException;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.codec.binary.Base64;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq
 * @author ChengJianSheng
 * @data 2020/12/1
 */
@Slf4j
@Service
public class DingTalkAlarmService {

    @Value("${dingtalk.webhook}")
    private String webhook;
    @Value("${dingtalk.secret}")
    private String secret;

    public void sendMessage(String content) {
        try {
            Long timestamp = System.currentTimeMillis();
            String stringToSign = timestamp + "\n" + secret;
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256"));
            byte[] signData = mac.doFinal(stringToSign.getBytes("UTF-8"));
            String sign = URLEncoder.encode(new String(Base64.encodeBase64(signData)),"UTF-8");

            String serverUrl = webhook + "&timestamp=" + timestamp + "&sign=" + sign;
            DingTalkClient client = new DefaultDingTalkClient(serverUrl);
            OapiRobotSendRequest request = new OapiRobotSendRequest();
            request.setMsgtype("text");
            OapiRobotSendRequest.Text text = new OapiRobotSendRequest.Text();
            text.setContent(content);
            request.setText(text);

            client.execute(request);
        } catch (ApiException e) {
            e.printStackTrace();
            log.error(e.getMessage(), e);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            log.error(e.getMessage(), e);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            log.error(e.getMessage(), e);
        } catch (InvalidKeyException e) {
            e.printStackTrace();
            log.error(e.getMessage(), e);
        }
    }
}
```



AlarmController.java



```java
package com.wt.monitor.skywalking.alarm.controller;

import com.alibaba.fastjson.JSON;
import com.wt.monitor.skywalking.alarm.domain.AlarmMessageDTO;
import com.wt.monitor.skywalking.alarm.service.DingTalkAlarmService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.text.MessageFormat;
import java.util.List;

/**
 * @author ChengJianSheng
 * @date 2020/12/1
 */
@Slf4j
@RestController
@RequestMapping("/skywalking")
public class AlarmController {

    @Autowired
    private DingTalkAlarmService dingTalkAlarmService;

    @PostMapping("/alarm")
    public void alarm(@RequestBody List<AlarmMessageDTO> alarmMessageDTOList) {
       log.info("收到告警信息: {}", JSON.toJSONString(alarmMessageDTOList));
       if (null != alarmMessageDTOList) {
           alarmMessageDTOList.forEach(e->dingTalkAlarmService.sendMessage(MessageFormat.format("-----来自SkyWalking的告警-----\n【名称】: {0}\n【消息】: {1}\n", e.getName(), e.getAlarmMessage())));
       }
    }
}
```



![img](https://img2020.cnblogs.com/blog/874963/202012/874963-20201202181238297-1332650779.png)

# 附件文档

https://skywalking.apache.org/

https://skywalking.apache.org/zh/ 

https://github.com/apache/skywalking/tree/v8.2.0/docs

https://archive.apache.org/dist/

https://www.elastic.co/guide/en/elasticsearch/reference/master/index.html 

- [主流微服务全链路监控系统之战](https://mp.weixin.qq.com/s/LaJHdCiZ-P1C1bKdbprNZA)
- [为你的应用加上skywalking（链路监控）](https://www.cnblogs.com/coolops/p/13750164.html)

