- [Docker 封装java镜像](https://www.cnblogs.com/xiao987334176/p/11771881.html)



# 一、jdk镜像

在docker中跑java应用，需要有jdk环境支持才行。

获取jdk镜像，有2种方式。

1. 自己制作jdk镜像

2. 使用Docker Hub 现成的jdk镜像。

 

首先说明一下，自己制作jdk镜像。如果基础镜像采用centos，ubuntu，那么制作出来的镜像会特别大。

Alpine只有5M，可以通过作为基础镜像，来制作镜像。但是会有2个问题：

1. 直接调用java命令会报错。
2.  时区不是中国时区。

基于2个问题，我采用的是2种方式。

https://hub.docker.com/r/mayan31370/openjdk-alpine-with-chinese-timezone/tags

 

这个镜像，已经帮你解决了，上面2个问题。而且，镜像本身，也做了优化。只有68M左右，非常小。

 

# 二、封装java镜像

有了jdk镜像后，封装java就简单多了。

## 创建目录

创建应用目录，文件如下：

```
.
├── Dockerfile
└── RMS.jar
```

 

Dockerfile

```
FROM mayan31370/openjdk-alpine-with-chinese-timezone:8-jdk
ADD RMS.jar .
EXPOSE 8080
ENTRYPOINT [ "java", "-jar", "RMS.jar" ]
```

注意：这个jar启动，会监听8080端口。

 

RMS.jar是已经打包好的java应用。

 

## 生成镜像

```
docker build -t rms .
```

 

## 启动镜像

```
docker run -it -p 8080:8080 rms /bin/bash
```

输出：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)

2019-10-31 16:10:02.517 [main] INFO  com.iaicmt.rms.RmsApplication - Starting RmsApplication v1.0-SNAPSHOT on e6c7908e56ab with PID 1 (/RMS.jar started by root in /)
...
2019-10-31 16:10:13.321 [main] INFO  o.s.j.e.a.AnnotationMBeanExporter - Registering beans for JMX exposure on startup
2019-10-31 16:10:13.549 [main] INFO  o.s.b.c.e.u.UndertowEmbeddedServletContainer - Undertow started on port(s) 8080 (http)
2019-10-31 16:10:13.570 [main] INFO  com.iaicmt.rms.RmsApplication - Started RmsApplication in 12.483 seconds (JVM running for 14.623)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

## 访问页面

```
# curl 127.0.0.1:8080
{"timestamp":1572509686431,"status":200,"error":"","message":"Null","path":"/"}
```