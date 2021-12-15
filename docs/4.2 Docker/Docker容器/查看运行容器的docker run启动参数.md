- [查看运行容器的docker run启动参数](https://www.cnblogs.com/aresxin/p/docker-run.html)



### 安装pip

```shell
# yum install -y python-pip
```

### 安装runlike

```shell
# pip install runlike
```

### 查看docker run参数

#### 发布一个测试容器

```shell
# docker run -d -v /data/nginx:/data/nginx -v /etc/hosts:/etc/hosts -p 8080:80 --name nginx nginx:1.18 
# netstat -lntup | grep 8080
tcp6       0      0 :::8080                 :::*                    LISTEN      5357/docker-proxy
# docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                  NAMES
76a49h8f017c   nginx:1.18   "nginx -g 'daemon of…"   2 seconds ago   Up 2 seconds   0.0.0.0:8080->80/tcp   nginx
```

#### 使用runlike查看启动参数

格式：runlike -p <容器名>|<容器ID>

```shell
# runlike -p nginx
docker run \
    --name=nginx \
    --hostname=76a49h8f017c \
    --env=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
    --env=NGINX_VERSION=1.18.3 \
    --env=NJS_VERSION=0.3.9 \
    --env='PKG_RELEASE=1~buster' \
    --volume=/data/nginx:/data/nginx \
    --volume=/etc/hosts:/etc/hosts \
    -p 8080:80 \
    --restart=no \
    --label maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>" \
    --detach=true \
    nginx:1.18 \
    nginx -g 'daemon off;'
```

