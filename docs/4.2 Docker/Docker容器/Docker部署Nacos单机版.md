- [使用Docker部署Nacos-Server(单机版)](https://www.jianshu.com/p/3d3e17bc629f)



# 方式一

```
仅部署nacos-server,不使用prometheus/grafana等监控组件
```

## 1.拉取镜像

```shell
docker pull nacos/nacos-server
```

## 2.挂载目录

```shell
mkdir -p /home/nacos/logs/                      #新建logs目录
mkdir -p /home/nacos/init.d/          
vim /home/nacos/init.d/custom.properties        #修改配置文件
```

添加如下参数：

```yaml
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://xx.xx.xx.x:3306/nacos_devtest_prod?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=user
db.password=pass


nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false

management.metrics.export.elastic.enabled=false

management.metrics.export.influx.enabled=false


server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i


nacos.security.ignore.urls=/,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/login,/v1/console/health/**,/v1/cs/**,/v1/ns/**,/v1/cmdb/**,/actuator/**,/v1/console/server/**
nacos.naming.distro.taskDispatchThreadCount=1
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true
nacos.naming.expireInstance=true
```

```
:wq 保存退出
```

## 3.启动容器

```shell
docker  run \
--name nacos -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /home/nacos/logs:/home/nacos/logs \
-v /home/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties \
nacos/nacos-server
```

# 方式二

```
通过docker-compose部署,包含prometheus/grafana等监控组件
```

## 1.拉取仓库

```bash
git clone --depth 1 https://github.com/nacos-group/nacos-docker.git
```

## 2.运行docker-compose

```bash
cd nacos-docker
docker-compose -f example/standalone-derby.yaml up -d 
```

