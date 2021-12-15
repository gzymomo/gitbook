- [docker封装redis镜像](https://www.cnblogs.com/xiao987334176/p/11984713.html)



# 一、概述

默认的镜像启动时，是没有`redis.conf`的，如果需要加配置，需要自己定义配置文件。



# 二、封装镜像

创建目录

```
# dockerfile目录
mkdir -p /opt/dockerfile/redis
# 持久化目录
mkdir -p /data/redis
```

 

/opt/dockerfile/redis目录结构如下：

```
./
├── dockerfile
├── redis.conf
└── run.sh
```

 

## dockerfile

```
FROM redis:3.2.12
COPY redis.conf /usr/local/etc/redis/redis.conf
CMD [ "redis-server", "/usr/local/etc/redis/redis.conf"]
```

 

## redis.conf

```bash
dir /data
pidfile /data/redis.pid
logfile "/data/redis.log"
repl-disable-tcp-nodelay yes
no-appendfsync-on-rewrite yes
maxmemory 2048m
maxmemory-policy allkeys-lru
requirepass 123456
```

注意：调整maxmemory参数。我这里的服务器内存是4g，所以调整为2g



## run.sh

```bash
#!/bin/bash
docker run -d -it --name redis_prod --restart=always -p 6379:6379 -v /data/redis:/data redis_prod:3.2.12
```

 

## 生成镜像

```
cd /opt/dockerfile/redis
docker build -t redis_prod:3.2.12 .
```

 

## 启动镜像

```
bash run.sh
```



# 三、测试

```
# docker exec -it redis_prod /bin/bash
# redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> info
# Server
redis_version:3.2.12
...
```