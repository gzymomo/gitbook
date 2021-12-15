- [k8s+Docker部署Spring Cloud微服务](https://lengxiaobing.github.io/2019/04/29/k8s+Docker%E9%83%A8%E7%BD%B2Spring-Cloud%E5%BE%AE%E6%9C%8D%E5%8A%A1/)

# 一.Spring Cloud组件

> Spring Cloud常用的三个组件，注册中心、配置中心和网关的代码示例。

## 1.1.Eureka注册中心

**代码示例**

- pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>microservice</groupId>
    <artifactId>eureka-registration-center</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>eureka-registration-center</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
</project>
```

- application.yml

```yaml
server:
  port: 8010
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
spring:
  application:
    name: eureka-registration-center
```

- 启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaRegistrationCenterApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaRegistrationCenterApplication.class, args);
    }
}
```

## 1.2.Config配置中心

**代码示例，集成消息总线bus，GitLab存储配置文件**

- pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>microservice</groupId>
    <artifactId>config-center-bus</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>config-center-bus</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.tmatesoft.svnkit</groupId>
            <artifactId>svnkit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
</project>
```

- application.yml

```yaml
server:
  port: 8020

spring:
  application:
    name: config-center-bus
  cloud:
    # git配置方式
    config:
      server:
        git:
          # gitlab上的配置中心的项目路径
          uri: https://gitlab.com/microservice/config-center.git
          # git仓库地址下的相对地址，可以配置多个，用","分割。
          search-paths: microservice,library
          username: xxxx@xxx.com
          password: xxxxxxxxxxxx
    # 开启跟踪总线事件
    bus:
      trace:
        enabled: true
  # RabbitMQ
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

# Web管理端点的配置属性
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
# 注册到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8010/eureka/
```

- 启动类

```java
@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class ConfigCenterApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterApplication.class, args);
    }
}
```

## 1.3.Gateway网关

**代码示例，使用的Spring提供的gateway**

- pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>microservice</groupId>
    <artifactId>api-gateway</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>api-gateway</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>
</project>
```

- api-gateway-dev.yml：默认的配置文件（放在config配置中心上）

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      # 与服务注册于发现组件进行结合，通过 serviceId 转发到具体的服务实例
      discovery:
        locator:
          enabled: true
```

- bootstrap.yml：集成配置中心后的配置文件，程序启动时，从配置中心拉取api-gateway-dev.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-center-bus
      # 配置文件的后缀，例：a-dev.yml
      profile: dev
      # git的分支
      label: master

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8010/eureka/
```

# 二.服务模块改造

> 需要注意Sring Cloud和Spring Boot之间的版本对应

| Cloud Version | Boot Version |
| ------------- | ------------ |
| Greenwich.x   | 2.1.x        |
| Finchley.x    | 2.0.x        |
| Edgware.x     | 1.5.x        |
| Dalston.x     | 1.5.x        |

## 2.1.添加配置

- 修改pom.xml，添加注册中心客户端，配置中心客户端，消息总线依赖

```xml
<properties>
  <java.version>1.8</java.version>
  <spring-cloud.version>Greenwich.RC2</spring-cloud.version>
</properties>

<dependencies>
  <-- 集成注册中心客户端 -->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
    
  <-- 集成配置中心客户端 -->
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
    
  <-- 集成消息总线 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
</dependencies>

<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

## 2.2.修改配置文件

- 创建**message-service-bus-dev.yml**，添加如下信息，并上传到gitlab的配置中心

```yaml
server:
  port: 8033
spring:
  application:
    name: message-service-bus
  # RabbitMQ
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```

- 创建**bootstrap.yml**配置文件

  程序启动时会根据`name`和`profile`两个属性，拉取具体的配置文件；

  例如：下面配置会拉取`message-service-bus-dev.yml`配置文件

```yaml
server:
  port: 8030

spring:
  application:
    name: message-service-bus
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-center-bus
      profile: dev
      label: master

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8010/eureka/
```

# 三.服务Docker化

## 3.1.创建Dockerfile文件

- 在项目src文件夹同级创建Dockerfile文件

```bash
# 依赖的jre环境
FROM 192.168.3.34:80/library/openjdk:8-jre-alpine
# 创建人员和邮箱地址
MAINTAINER xxx xxxx@xx.com
# 复制jar到Docker镜像中，注意jar名称必须使用小写
COPY target/message-service-bus-1.0-SNAPSHOT.jar /message-service-bus.jar
# 映射项目端口
EXPOSE 9090
# 项目启动命令
ENTRYPOINT ["java", "-jar", "/message-service-bus.jar"]
```

## 3.2.创建build文件（可选）

- 在项目src文件夹同级创建build.sh文件，主要是方便打镜像，不需要每次都敲命令（偷懒）。

```bash
#!/usr/bin/env bash
mvn clean package

docker build -t 192.168.3.34:80/library/message-service-bus:latest .
docker push 192.168.3.34:80/library/message-service-bus:latest
```

## 3.3.创建docker-compose文件（可选）

- 在项目src文件夹同级创建docker-compose.yml文件，此处是基本的相关配置，可参考。

```yaml
# docker-compose的版本
version: '3'
# 网络设置
networks:
  default:
    external:
      name: cloud-network
# 服务
services:
  # 服务名，自定义
  message-service:
    # 服务镜像
    image: message-service:latest
    # 服务内配置文件中的参数配置
    command:
      - "--mysql.address=127.0.0.1:3306"
    # 该服务依赖的服务
    links:
      - user-service
    # 暴露的端口
    ports:
      - 8080:8080
```

# 四.k8s集群部署服务

> 首先，考虑一下系统有哪些模块？模块是否需要对外暴露端口？集群内模块之间如何调用？
>
> 本文的k8s集群采用的是一主二从。

现在，开始一一解决这些问题。

首先，Sring Cloud的三个组件：

- **Erueka**：需要搭建集群，这里在k8s集群的两个节点上各放一个（测试项目资源有限），采用指点k8s节点的方式部署，对外暴露端口，方便Erueka之间相互通讯，以及其他服务进行注册。

  **注**：此方法无法做到快速扩容，只能保证固定初始化时的几个节点，如果想做到快速扩容，需要使用Erueka的`DNS`方式。

- **Config**：k8s集群内部运行，不对外暴露端口，保证配置文件的安全；在k8s集群内部，与其他服务之间相互通讯，可以使用service名称加端口的方式。

- **Gateway**：对外暴露端口，实现转发，作为整个系统对外访问的入口。在k8s集群内部，与其他服务之间相互通讯，可以使用service名称加端口的方式。

  - **注意**：在k8s集群中，要在网关处设置跨域，不要在具体服务中设置，没有效果。

接着，服务模块的部署：

- **后端服务**：如果需要在k8s集群外访问的话，可以考虑暴露端口；也可以不暴露端口，通过网关访问。在k8s集群内部，与其他服务之间相互通讯，可以使用service名称加端口的方式。

- **前端服务**：可以考虑暴露端口，进行直接访问；也可以不暴露端口，通过网关访问。如果后端服务暴露端口可以直接调用，否则，需要通过网关调用后端服务。

  **注意**：通过网关访问，需要考虑静态资源加载的问题。可以通过两种方式解决：

  - 将静态资源放在网关下；
  - 服务模块各自的静态资源各自放在一个请求路径下，通过网关进行转发。

下面，开始进行部署。

## 4.1.Erueka的k8s配置文件

- application.yml：注册中心服务的配置文件

```yaml
spring:
  application:
    name: eureka-registration-center
server:
  port: ${server.port}
eureka:
  instance:
    hostname: ${spring.application.name}
    appname: ${spring.application.name}
    prefer-ip-address: false
    lease-expiration-duration-in-seconds: 90
  server:
    # 设为false，关闭自我保护
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 5000
  client:
    register-with-eureka: true
#    fetch-registry: false
    service-url:
      # 根据集群数量确定连接个数，示例为两个节点
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/,http://${eureka-node.hostname}:${eureka-node.port}/eureka/
```

- eureka-registration-center-1.yaml：注册中心k8s的配置文件

  使用指定k8s节点的方式部署，集群模式需要配置多个文件，并修改端口、ip等相应的一些配置。这里只写了一个示例。

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: eureka-registration-center-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: eureka-registration-center-1
  template:
    metadata:
      labels:
        app: eureka-registration-center-1
    spec:
      # 指定使用的节点主机名
      nodeName: k8s-node1
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      # 使用的容器镜像
      containers:
        - name: eureka-registration-center
          image: 192.168.3.34:80/library/eureka-registration-center:V1.0.0
          # 参数，集群模式：不同节点的配置都不相同，示例为两个节点
          env:
            - name: server.port
              value: "8761"
            - name: eureka-node.hostname
              value: "192.168.3.32"
            - name: eureka-node.port
              value: "8762"
          ports:
            - name: http
              containerPort: 8761
              hostPort: 8761
          # 心跳检测，需要在程序中写一个 /health 的 controller接口
          livenessProbe:
            failureThreshold: 2
            httpGet:
              path: /health
              port: 8761
              scheme: HTTP
            initialDelaySeconds: 20
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 2
            httpGet:
              path: /health
              port: 8761
              scheme: HTTP
            initialDelaySeconds: 20
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
---
apiVersion: v1
kind: Service
metadata:
  name: eureka-registration-center-1
  labels:
    app: eureka-registration-center-1
spec:
  ports:
    - port: 8761
      name: eureka-registration-center-1
      targetPort: 8761
      nodePort: 8761
  selector:
    app: eureka-registration-center-1
```

## 4.2.Config的k8s配置文件

- application.yml：配置中心服务的配置文件

```yaml
server:
  port: ${server.port}

spring:
  application:
    name: config-center
  cloud:
    # git配置方式
    config:
      server:
        git:
          uri: ${git.url}
          # git仓库地址下的相对地址，可以配置多个，用,分割。
          search-paths: ${git.paths}
          username: ${git.username}
          password: ${git.password}

eureka:
  client:
    serviceUrl:
      defaultZone: ${eureka.url}
```

- config-center.yaml：配置中心的k8s配置文件

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: config-center
  name: config-center
spec:
  ports:
    - port: 8030
      protocol: TCP
      targetPort: 8030
  selector:
    app: config-center
  type: ClusterIP
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: config-center-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: config-center
    spec:
      containers:
        - name: config-center
          image: 192.168.3.34:80/library/config-center:V1.0.0
          # 参数
          env:
            # gitlab的配置
            - name: git.url
              value: "http://192.168.3.34:81/root/config-file.git"
            - name: git.paths
              value: "collection,microservice,visualization"
            - name: git.username
              value: "root"
            - name: git.password
              value: "root"
            # eureka配置
            - name: eureka.url
              value: "http://192.168.3.31:8761/eureka/,http://192.168.3.32:8762/eureka/"
          ports:
            - containerPort: 8030
```

## 4.3.服务的k8s配置文件

- bootstrap.yaml：服务程序的配置文件，启动时拉取存放在配置中心的配置。

  **注意**：

  - 服务从配置获取配置文件，需要使用uri方式
  - uri的地址：使用k8s的服务名称加端口号，例如：http://config-center:8030

```yaml
server:
  port: ${server.port}

spring:
  application:
    name: message-service-bus
  cloud:
    config:
      profile: ${config.profile}
      label: master
      # k8s集群内需要使用uri方式
      uri: ${config.url}
      # 失败重试机制
      fail-fast: true
      retry:
        initial-interval: 2000
        max-interval: 10000
        multiplier: 2
        max-attempts: 10

eureka:
  client:
    serviceUrl:
      defaultZone: ${eureka.url}
```

- message-service-bus.yaml：服务程序的k8s配置文件

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: message-service-bus
  name: message-service-bus
spec:
  ports:
    - port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    app: message-service-bus
  type: ClusterIP
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: message-service-bus-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: message-service-bus
    spec:
      containers:
        - name: message-service-bus
          image: 192.168.3.34:80/library/message-service-bus:V1.0.0
          env:
            - name: server.port
              value: "8080"
            - name: config.url
              value: "http://config-center:8030"
            - name: config.profile
              value: "test"
            - name: eureka.url
              value: "http://192.168.3.31:8761/eureka/,http://192.168.3.32:8762/eureka/"
          ports:
            - containerPort: 8080
```

## 4.4.Gateway的k8s配置文件

- bootstrap.yml：网关服务的配置文件

```yaml
server:
  port: ${server.port}

spring:
  application:
    name: api-gateway
  cloud:
    config:
      uri: ${config.url}
      profile: ${config.profile}
      label: master
      fail-fast: true
      retry:
        initial-interval: 2000
        max-interval: 10000
        multiplier: 2
        max-attempts: 10

eureka:
  client:
    serviceUrl:
      defaultZone: ${eureka.url}
```

- api-gateway-dev.yml：网关转发的具体配置，放在gitlab上。

```yaml
server:
  port: 8888

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 后端示例程序
        - id: dvis-service
          uri: http://dvis-sys-service:8080
          predicates:
            - Path=/dvis-service/**
          filters:
            - StripPrefix=1
```

- api-gateway.yaml：网关的k8s启动文件，使用NodePort模式，对外暴露端口，作为系统对外访问的入口。

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: api-gateway
  name: api-gateway
spec:
  ports:
    - port: 8888
      protocol: TCP
      targetPort: 8888
      nodePort: 8888
  selector:
    app: api-gateway
  type: NodePort
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: api-gateway-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
        - name: api-gateway
          image: 192.168.3.34:80/library/api-gateway:V1.0.0
          env:
            - name: server.port
              value: "8888"
            - name: config.url
              value: "http://config-center:8030"
            - name: config.profile
              value: "dev"
            - name: eureka.url
              value: "http://192.168.3.31:8761/eureka/,http://192.168.3.32:8762/eureka/"
          ports:
            - containerPort: 8888
```

## 4.5.k8s访问外部服务

- k8s访问集群外的服务最好的方式是采用Endpoint方式(可以看作是将k8s集群之外的服务抽象为内部服务)，也可以直接使用外部服务的ip。

  **首先创建一个没有 selector的service，其不会创建相关的 Endpoints 对象**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-test
spec:
  # 自定义暴露的Ip，可以不设置，直接用服务名称调用
  clusterIP: 10.106.96.101
  ports:
  - port: 3306
    targetPort: 3306
    protocol: TCP
```

​	**接着创建一个Endpoints，手动将 Service映射到指定的 Endpoints**

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  # 名称必须和Service相同
  name: mysql-test
subsets:
  - addresses:
    # 多个ip可以再次列出
    - ip: 192.168.3.141
    - ip: 192.168.3.142
    ports:
    # 多个端口的话可以在此处列出，将上述两个ip的端口，映射到内部
    - port: 3306
      protocol: TCP
```

application.yml：服务使用service名称加端口访问

```yaml
spring:
  application:
    name: dvis-sys-service
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    # 此处使用k8s的服务名称，或者使用自定义暴露的clusterIP
    url: jdbc:mysql://mysql-test:3306/local_mysql?useUnicode=true
    username: root
    password: root
```

