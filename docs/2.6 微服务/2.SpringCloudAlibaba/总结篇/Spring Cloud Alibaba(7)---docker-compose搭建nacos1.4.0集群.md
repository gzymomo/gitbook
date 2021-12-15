[Spring Cloud Alibaba(7)---docker-compose搭建nacos1.4.0集群](https://www.cnblogs.com/qdhxhz/p/14705388.html)

##   一、项目概述

#### 1、技术选型

项目总体技术选型

```
CentOS 7.6 + Nacos 1.4.0 + MYSQL 8.0.22 + docker-compose 1.24.1 + docker 1.13.1
```

#### 2、服务器配置

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210426163907506-98021780.jpg)

因为自己只有两台阿里云服务器，所以这里Nacos集群数就两个。Mysql主从之前就搭建好了，这里就不描述搭建的过程。

#### 3、流程图

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210426163918905-1947530710.jpg)

有关 **微服务和Nginx集群** 也不再这篇讲述。这篇就是搭建好Nacos集群。

#### 4、集群方式

其实集群的方式有两种,一种是单机集群，一种是多机集群

```
单机集群: 在同一台服务器上，启动多个Nacos,组成集群。他们的Ip地址是一样的，只是端口号不一样(192.168.1.1:8848,192.168.1.1:8849,192.168.1.1:8850)
多机集群: 在不同服务器上，每台服务器启动一个nacos，组成集群。他们的Ip地址是不一样的，但端口号可以一样(192.168.1.1:8848,192.168.1.2:8848,192.168.1.3:8848)
```

我们这边采用的是第二种方式(多机集群),其实第一种可以理解成伪集群，第二种才是真集群。

#### 5、和官方docker-compose搭建nacos集群差异

其实官方对 docker-compose搭建nacos集群 有提供项目拿来即用。官方地址:[Nacos Docker](https://github.com/nacos-group/nacos-docker)

如果你只想在一台服务器上部署集群，那么跟着上面的教程，非常方便的就可以搭建单机集群，甚至mysql和nginx 官方提供的 docker-compose.yaml 都一并构建好了。

**我这边和官方提供的主要区别在于**

```
1、我们这边是多机集群，所以每台服务器上都需要一个 docker-compose.yaml，而且每台服务器只会启动一个nacos。
2、有关mysql和nginx 我这边是不需要通过 docker-compose.yaml生成对于容器，而是独立出来重新搭建，在docker-compose.yaml配置中只是添加连接Mysql配置数据就可以了。
```

#### 5、项目目录

因为是通过docker-compose搭建nacos集群，所以这里只需要我们编写好docker-compose.yml 文件就好。它其实就相当于一个脚本，下面是项目目录，具体文件我会放在

Github上，文章下方会提供地址。

```
nacos-docker
├── init.d
│   └── custom.properties
├── nacos-1
│   └── docker-compose-nacos-1.yml
└── nacos-2
    └── docker-compose-nacos-2.yml
目录说明
init.d/custom.properties - 官方提供的自选功能配置文件，Nacos节点均包含此目录
nacos-1/docker-compose-nacos-1.yml - 第一个Nacos节点的Docker-compose配置文件
nacos-2/docker-compose-nacos-2.yml - 第二个Nacos节点的Docker-compose配置文件
```



##  二、配置文件详解

通过三面目录可以看出，一共就三个文件，这里详解展示下文件具体配置

#### 1、custom.properties

这个是每个nacos公用的，跟官方保持一致即可

```yml
#spring.security.enabled=false
#management.security=false
#security.basic.enabled=false
#nacos.security.ignore.urls=/**
#management.metrics.export.elastic.host=http://localhost:9200
# metrics for prometheus
management.endpoints.web.exposure.include=*

# metrics for elastic search
#management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200

# metrics for influx
#management.metrics.export.influx.enabled=false
#management.metrics.export.influx.db=springboot
#management.metrics.export.influx.uri=http://localhost:8086
#management.metrics.export.influx.auto-create-db=true
#management.metrics.export.influx.consistency=one
#management.metrics.export.influx.compressed=true
```

#### 2、docker-compose-nacos-1.yml

这里将docker-compose-nacos-1.yml在 47.19.203.55上启动。

```yml
version: '3' 
services:
  # nacos-server服务注册与发现，配置中心服务    
  docker-nacos-server:
    image: nacos/nacos-server:1.4.0
    container_name: nacos-server-1
    ports:
      - "8848:8848"
      - "9555:9555"
    networks: 
      - nacos_net
    restart: on-failure
    privileged: true
    environment:
      PREFER_HOST_MODE: ip #如果支持主机名可以使用hostname,否则使用ip，默认也是ip
      SPRING_DATASOURCE_PLATFORM: mysql #数据源平台 仅支持mysql或不保存empty
      NACOS_SERVER_IP: 47.19.203.55 #多网卡情况下，指定ip或网卡
      NACOS_SERVERS: 47.19.203.55:8848 118.11.224.65:8848 #集群中其它节点[ip1:port ip2:port ip3:port]
      MYSQL_SERVICE_HOST: 47.19.203.55
      MYSQL_SERVICE_PORT: 3306
      MYSQL_SERVICE_USER: root
      MYSQL_SERVICE_PASSWORD: root
      MYSQL_SERVICE_DB_NAME: nacos
      MYSQL_SERVICE_DB_PARAM: characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&allowPublicKeyRetrieval=true&serverTimezone=GMT%2B8
      #JVM调优参数
      JVM_XMS:  128m
      JVM_XMX:  128m
      JVM_XMN:  128m
      #JVM_MS:   #-XX:MetaspaceSize default :128m
      #JVM_MMS:  #-XX:MaxMetaspaceSize default :320m
      #NACOS_DEBUG: n #是否开启远程debug，y/n，默认n
      #TOMCAT_ACCESSLOG_ENABLED: true #是否开始tomcat访问日志的记录，默认false
    volumes:
      - ./cluster-logs/nacos1:/home/nacos/logs #日志输出目录
      - ../init.d/custom.properties:/home/nacos/init.d/custom.properties #../init.d/custom.properties内包含很多自定义配置，可按需配置

networks:
  nacos_net:
    driver: bridge
```

#### 3、docker-compose-nacos-2.yml

这里将docker-compose-nacos-2.yml在 118.11.224.65上启动。

```yml
version: '3' 
services:
  # nacos-server服务注册与发现，配置中心服务    
  docker-nacos-server:
    image: nacos/nacos-server:1.4.0
    container_name: nacos-server-1
    ports:
      - "8848:8848"
      - "9555:9555"
    networks: 
      - nacos_net
    restart: on-failure
    privileged: true
    environment:
      PREFER_HOST_MODE: ip #如果支持主机名可以使用hostname,否则使用ip，默认也是ip
      SPRING_DATASOURCE_PLATFORM: mysql #数据源平台 仅支持mysql或不保存empty
      NACOS_SERVER_IP: 118.11.224.65 #多网卡情况下，指定ip或网卡
      NACOS_SERVERS: 47.19.203.55:8848 118.11.224.65:8848 #集群中其它节点[ip1:port ip2:port ip3:port]
      MYSQL_SERVICE_HOST: 47.19.203.55
      MYSQL_SERVICE_PORT: 3306
      MYSQL_SERVICE_USER: root
      MYSQL_SERVICE_PASSWORD: root
      MYSQL_SERVICE_DB_NAME: nacos
      MYSQL_SERVICE_DB_PARAM: characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&allowPublicKeyRetrieval=true&serverTimezone=GMT%2B8
      #JVM调优参数
      JVM_XMS:  128m
      JVM_XMX:  128m
      JVM_XMN:  128m
      #JVM_MS:   #-XX:MetaspaceSize default :128m
      #JVM_MMS:  #-XX:MaxMetaspaceSize default :320m
      #NACOS_DEBUG: n #是否开启远程debug，y/n，默认n
      #TOMCAT_ACCESSLOG_ENABLED: true #是否开始tomcat访问日志的记录，默认false
    volumes:
      - ./cluster-logs/nacos1:/home/nacos/logs #日志输出目录
      - ../init.d/custom.properties:/home/nacos/init.d/custom.properties #../init.d/custom.properties内包含很多自定义配置，可按需配置

networks:
  nacos_net:
    driver: bridge
```

#### 4、总结配置

1、这里需要注意的是,默认 **JVM_XMS** 和 **JVM_XMX** 需要 **2g**，对我来讲太大了，所以这里改了小点，这样就不会内存溢出。

2、从Nacos 1.3.1版本开始,数据库存储已经升级到8.0,并且它向下兼容，所以我们不需要对于mysql8配置做特别处理了。

3、这里只需要配置master的mysql,不需要配置slave的mysql。这个官方也做了很好的解释。[移除数据库主从镜像配置](https://github.com/nacos-group/nacos-docker/wiki/移除数据库主从镜像配置)

后续所有镜像都会移除主从镜像相关属性,具体移除和替换属性如下:

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210426163940369-154162015.jpg)

##  三、部署到服务器 

#### 1、上传文件服务器

我先在本地把上面文件创建好并压缩，然后将压缩打包到服务器中

```
scp  nacos-docker.zip root@47.19.203.55:
scp  nacos-docker.zip root@118.11.224.65:
```

两台服务器，都上传同一份文件，只是启动的docker-compose.yml不是同一个

```
#解压
unzip nacos-docker.zip
#移动到指定位置
cp -r nacos-docker /usr/local/nacos
```

#### 2、启动容器

分别在各主机上进入各自对应的nacos目录中，启动容器，命令如下：

```
47.19.203.55服务器
$ cd nacos-docker/nacos-1
$ docker-compose -f docker-compose-nacos-1.yml up -d
118.11.224.65服务器
$ cd nacos-docker/nacos-2
$ docker-compose -f docker-compose-nacos-2.yml up -d
```

启动后访问（记得放开8848端口）

```
47.19.203.55:8848/nacos
118.11.224.65:8848/nacos
```

如果访问失败，那么对日志中查看错误日志

```
tail -f cluster-logs/nacos*/nacos.log
```



##  四、测试

#### 1、访问客户端

访问 下面 任意一个。

```
47.19.203.55:8848/nacos
118.11.224.65:8848/nacos
```

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210426165213494-1687836401.jpg)

从图中看出，集群已经配置成功。而这个47.19.203.55:8848其实是主节点。

#### 2、微服务配置

这里微服务配置也需要稍微改动下

```yml
spring:
  cloud:
    nacos:
      config:
        server-addr: 47.19.203.55:8848,118.11.224.65:8848 #Nacos配置中心地址
        file-extension: yaml #文件拓展格式
      discovery: #服务中心地址
        server-addr: 47.19.203.55:8848,118.11.224.65:8848
```



`Github地址`:[nacos-docker](https://github.com/yudiandemingzi/nacos-docker)



###  参考

[1、官方Nacos Docker](https://github.com/nacos-group/nacos-docker)

[2、Nacos高可用集群解决方案-Docker版本，基于Nacos 1.0.1](https://www.cnblogs.com/hellxz/p/nacos-cluster-docker.html)

[3、docker搭建nacos server集群](https://gitee.com/ysdxhsw/docker-compose-nacos-mysql/tree/master)