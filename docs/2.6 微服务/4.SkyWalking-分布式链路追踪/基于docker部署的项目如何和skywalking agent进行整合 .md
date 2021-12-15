基于docker部署的项目如何和skywalking agent进行整合：https://juejin.cn/post/6945389939396476959

### skywalking简介

skywalking是一款开源的应用性能监控系统，包括指标监控，分布式追踪，分布式系统性能诊断

### skywalking官方中文翻译文档

[skyapm.github.io/document-cn…](https://skyapm.github.io/document-cn-translation-of-skywalking/)

### 如何快速搭建skywalking

[github.com/apache/skyw…](https://github.com/apache/skywalking-docker)

### 项目如何集成skywalking

> 1、下载skywalking agent

[archive.apache.org/dist/skywal…](https://archive.apache.org/dist/skywalking/)

解压后的目录形如下 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5eeef4ea34524fb19783bf27f2e93175~tplv-k3u1fbpfcp-zoom-1.image)

> 2、为我们项目配置skywalking探针

形如下

```java
java -javaagent:D:apache-skywalking-apm-es7-8.4.0/apache-skywalking-apm-bin-es7/agentskywalking-agent.jar -Dskywalking.agent.service_name=当前项目在skywalking显示的名称 -Dskywalking.collector.backend_service=xxxx:11800 -jar spring-demo-0.0.1-SNAPSHOT.jar

复制代码
```

官方其实也提供了文档，告诉我们如何配置，如下图 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c1bacf4f7b849b08812cde062b06262~tplv-k3u1fbpfcp-zoom-1.image) 更详细配置信息，可以查看如下链接 [github.com/apache/skyw…](https://github.com/apache/skywalking/blob/master/docs/en/setup/service-agent/java-agent/README.md)

通过以上几步就项目就可以和skywalking整合了。然而有些小伙伴反馈在docker环境中，就不懂要怎么使用skywalking的agent进行埋点了。那下面就介绍一下，基于docker部署的项目如何和skywalking agent进行整合

### 思考点：docker中的项目中要如何才能使用到skywalking agent？

道理可能大家都懂，就是把skywalking agent与项目都塞到到同个docker容器中，基于这个理论，就衍生出一下2种方案

> 方案一：把skywalking agent的整个agent文件夹都集成进行要埋点的项目中

形如下图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f6d750c79be4b4393b44c67af8cfb7c~tplv-k3u1fbpfcp-zoom-1.image) 然后修改一下项目的dockerfile文件，修改后的内容如下

```yaml
FROM adoptopenjdk/openjdk8
VOLUME /tmp
COPY localtime /etc/localtime
RUN echo "Asia/Shanghai" > /etc/timezone
COPY target/spring-demo-*.jar app.jar
COPY agent /usr/local/agent
ENTRYPOINT [ "sh", "-c", "java  -javaagent:/usr/local/agent/skywalking-agent.jar -Dskywalking.agent.service_name=spring-demo -Dskywalking.collector.backend_service=192.168.1.2:11800 -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
复制代码
```

核心的主要以下这两句

```yaml
COPY agent /usr/local/agent
ENTRYPOINT [ "sh", "-c", "java  -javaagent:/usr/local/agent/skywalking-agent.jar -Dskywalking.agent.service_name=spring-demo -Dskywalking.collector.backend_service=192.168.1.2:11800 -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
复制代码
```

把项目中的agent文件夹拷贝进行容器中的/usr/local/agent文件夹中，然后就后面操作就跟在普通环境使用skwalking agent的操作一样了

整合后如下图 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb19def53ace40979e93ef600e46ed15~tplv-k3u1fbpfcp-zoom-1.image)

> 方案二：在我们构建基础镜像时，把skywalking agent也加进去

比如我们构建java运行的jdk基础镜像时，加入skywalking agent ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53b88154b49a4b2e9ba368424656b40e~tplv-k3u1fbpfcp-zoom-1.image) 其dockerfile内容形如下

```yaml
FROM adoptopenjdk/openjdk8
VOLUME /tmp
#ENV JAVA_OPTS="-Dcom.sun.management.jmxremote.port=39083 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
ENV JAVA_OPTS=""
ENV SKYWALKING_AGENT_SERVICE_NAME=""
ENV SKYWALKING_COLLECTOR_BACKEND_SERVICE=""
COPY localtime /etc/localtime
COPY agent /usr/local/agent
RUN echo "Asia/Shanghai" > /etc/timezone

ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -javaagent:/usr/local/agent/skywalking-agent.jar -Dskywalking.agent.service_name=$SKYWALKING_AGENT_SERVICE_NAME -Dskywalking.collector.backend_service=$SKYWALKING_COLLECTOR_BACKEND_SERVICE -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
复制代码
```

然后通过docker build -t 镜像名 . 或者通过docker-compose build 把基础镜像构建出来

本例构建出来的基础镜像为openjdk8-trace-agent。

这边有几个参数说明下：SKYWALKING_AGENT_SERVICE_NAME和SKYWALKING_COLLECTOR_BACKEND_SERVICE是作为环境变量，可以在docker-compose.yml文件或者k8s文件中指定具体环境变量值。以在docker-compose.yml为例

配置形如下

```yaml
version: '3.1'
services:
  spring-demo:
    restart: always
    image: 192.168.1.3:5002/demo/spring-demo:dev
    container_name: spring-demo
    network_mode: bridge
    ports:
     - "8085:8080"
    environment:
     - SKYWALKING_AGENT_SERVICE_NAME=spring-demo-test
     - SKYWALKING_COLLECTOR_BACKEND_SERVICE=192.168.1.2:11800
复制代码
```

其次

```yaml
ONBUILD COPY app.jar app.jar
复制代码
```

我们在maven构建时，把业务的jar统一命名成app.jar，因此第一个app.jar 是我们业务项目的jar，第二个jar是运行在docker容器的jar。这样我们在业务的dockerfile中，只需这么写就行

```yaml
FROM 192.168.1.3:5002/dev/openjdk8-trace-agent
复制代码
```

整合后示例如下图 ![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6e8d270db79441490f51056b7fedba5~tplv-k3u1fbpfcp-zoom-1.image)

## 


作者：linyb极客之路
链接：https://juejin.cn/post/6945389939396476959