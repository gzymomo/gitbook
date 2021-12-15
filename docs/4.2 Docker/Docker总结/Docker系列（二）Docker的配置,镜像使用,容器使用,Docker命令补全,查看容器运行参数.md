# 	一、Docker配置

## 1.1 Docker镜像加速

在我们使用docker的过程中，可能会需要下载一些基础镜像等，默认会从Docker Hub上进行下载，下载速度可能不是很乐观，此时我们通过修改docker配置文件，实现docker镜像加速。



此处，我们使用阿里云的docker镜像加速方式：

阿里云容器镜像服务地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

使用支付宝或钉钉登录阿里云，然后访问阿里云控制台找到容器镜像服务，在容器镜像服务中找到镜像加速器，在这里面会有一个加速器地址，复制该地址。

![image-20210202132447147](http://lovebetterworld.com/image-20210202132447147.png)

然后根据下面的说明执行操作：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["此处是复制过来的镜像地址"]
}
EOF
# 重新加载配置文件
sudo systemctl daemon-reload
# 重启Docker服务
sudo systemctl restart docker
```



## 1.2 Docker的其他基础配置

如果还需对Docker的其他配置进行修改，可继续在daemon.json文件中添加内容，如下示例：

```bash
vi /etc/docker/daemon.json
```

修改添加内容如下：

```yaml
{
  "registry-mirrors": ["https://bk6kzfqm.mirror.aliyuncs.com"], #docker镜像加速
  "insecure-registries": ["192.168.0.241"], #docker仓库
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```



## 1.3 Docker远程端口开放

如果需要远程操作Docker或使用其他API操作Docker，则需要开启Docker的2375端口，开启方式如下：

1. 查看docker状态

```bash
[root@VM-0-15-centos ~]# systemctl status docker
```

2. 找到docker.server配置文件的地址，然后修改docker.service

![image-20210202133153933](http://lovebetterworld.com/image-20210202133153933.png)

```bash
vi /usr/lib/systemd/system/docker.service
```

3. 修改ExecStart行为下面内容：

```bash
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock \
```

4. 重新加载docker配置

```bash
systemctl daemon-reload // 1，加载docker守护线程
systemctl restart docker // 2，重启docker
```



# 二、Docker镜像使用

Docker的镜像是其核心概念，镜像中包含了程序运行所需的相应环境，学习Docker镜像，我们不光要了解常见的Docker镜像，如何查找Docker镜像，还要自己能够制作所需的镜像。

## 2.1 镜像查找

Docker也有一个仓库Docker Hub，其类似Github一样的一个仓库，只不过Docker Hub上包含的是丰富的镜像，对于这些镜像的使用，要尽量使用官方封装的镜像。

Docker Hub官网地址：https://registry.hub.docker.com/search?q=&type=image

![image-20210202134025769](http://lovebetterworld.com/image-20210202134025769.png)

在上方搜索到所需的镜像后，可以看到右侧会有相应的说明，以redis官方镜像为例，点进去后可以看到该镜像的描述。

![image-20210202134145464](http://lovebetterworld.com/image-20210202134145464.png)

执行右上角的命令

```bash
docker pull redis
```

默认拉取的是latest版本的镜像仓库，要想使用其他版本，则找到上图中的Tags，点进去查看其他版本。

根据自己的需要拉取相应版本：

![image-20210202134356341](http://lovebetterworld.com/image-20210202134356341.png)



上述方式是通过Docker Hub查找镜像，我们也可以直接通过命令行方式进行查找。

```bash
docker search [OPTIONS] TERM
```

示例：

![image-20210202134541396](http://lovebetterworld.com/image-20210202134541396.png)



## 2.2 下载镜像

找到我们想要的基础镜像后，通过docker pull命令来拉取镜像

```bash
docker pull [OPTIONS] NAME[:TAG]
```

如果不指定TAG，则默认拉取的为latest版本的镜像。

示例：

```bash
docker pull mysql:5.6
```

拉取5.6版本的mysql，拉取成功后，如图所示。

![image-20210202134951483](http://lovebetterworld.com/image-20210202134951483.png)





## 2.3 查看所有镜像

查看本地已经拥有的镜像

```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

示例：

![image-20210202134704558](http://lovebetterworld.com/image-20210202134704558.png)



## 2.4 删除镜像

如果下载错了某个版本的基础镜像，想要删除，通过如下命令：

```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

- 如果不指定tag，则会默认删除latest的版本。
- 删除镜像，可以通过REPOSITORY:TAG的方式，也可通过指定IMAGE ID的方式。



示例：删除5.6版本的MySQL

```bash
docker rmi mysql:5.6
```

![image-20210202135241516](http://lovebetterworld.com/image-20210202135241516.png)



## 2.5 增加镜像标签

修改或增加镜像的标签，通过如下命令：

```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

示例：

![image-20210202135519274](http://lovebetterworld.com/image-20210202135519274.png)



## 2.6 将本地镜像推送至Docker Hub

如果想将自己制作的基础镜像推送至Docker Hub，然后在任何地方进行拉取镜像，则首先需要去注册Docker Hub账号，注册成功后，在服务器上登录。

```bash
docker login -u 用户名 -p 密码
```

然后推送镜像至Docker Hub：

```bash
docker push [OPTIONS] NAME[:TAG]
```



## 2.7 通过dockerfile构建JDK最小镜像

### 1. 下载JRE

下载地址：https://www.java.com/en/download/manual.jsp

### 2. 解压JRE,删除相关不需要文件

```bash
#解压
tar xvcf jre-8u161-linux-x64.tar.gz
#进入目录
cd jre1.8.0_161/
#删除文本文件
rm -rf COPYRIGHT LICENSE README release THIRDPARTYLICENSEREADME-JAVAFX.txtTHIRDPARTYLICENSEREADME.txt Welcome.html
#删除其他无用文件
rm -rf     lib/plugin.jar \
           lib/ext/jfxrt.jar \
           bin/javaws \
           lib/javaws.jar \
           lib/desktop \
           plugin \
           lib/deploy* \
           lib/*javafx* \
           lib/*jfx* \
           lib/amd64/libdecora_sse.so \
           lib/amd64/libprism_*.so \
           lib/amd64/libfxplugins.so \
           lib/amd64/libglass.so \
           lib/amd64/libgstreamer-lite.so \
           lib/amd64/libjavafx*.so \
           lib/amd64/libjfx*.so
```

### 3. 重新打包

重新打包所有文件(不打包也可以，在Dockerfile里ADD这个目录即可，当前精简完jre目录大小是107M，压缩后是41M)

### 4. 创建Dockerfile

创建Dockerfile（ps：Dockerfile文件要和jre8.tar.gz在一个路径下，如果不一样，下面的ADD里面的路径要相应变化）

```javascript
# using alpine-glibc instead of alpine  is mainly because JDK relies on glibc
FROM docker.io/jeanblanchard/alpine-glibc
# A streamlined jre
ADD jre8.tar.gz /usr/java/jdk/
# set env
ENV JAVA_HOME /usr/java/jdk
ENV PATH ${PATH}:${JAVA_HOME}/bin
# run container with base path:/opt
WORKDIR /opt
```

### 5. docker构建镜像

构建：

```bash
docker build -t fds/java8:1.0 .
```

### 6. 测试运行

```javascript
# docker run -it voole/java8:1.0
/opt # java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

## 2.8 Docker保存镜像到本地

```bash
docker save -o jdk.tar imageId
```

利用管道打包压缩一步到位

```bash
docker save REPOSITORY:TAG | gzip > images.tar.gz 
```

## 2.9 Docker将打包镜像导入目标服务器，并使用docker导入

```bash
docker load -i images.tar
```

## 2.10 Docker将容器保存成镜像

```bash
sudo docker commit -a 'package images' CONTAINER-ID  REPOSITORY:TAG
```



# 三、Docker容器使用

当我们下载好想要的镜像后，接下来就可以启动容器，即通过使用基础镜像部署相应的服务了。

## 3.1 查看所有容器

```bash
docker ps [OPTIONS]
```

![image-20210202141151360](http://lovebetterworld.com/image-20210202141151360.png)

一般经常使用的话，查看所有容器，包括已经停止的容器，通过如下命令：

```bash
docker ps -a
```

若只查看正在运行的容器，则去掉-a参数即可。

```bash
docker ps
```

![image-20210202141258175](http://lovebetterworld.com/image-20210202141258175.png)



## 3.2 停止容器

若想要停止正在运行的容器，则可以通过如下方式：

```bash
docker stop/ kill CONTAINER
```

![image-20210202141427491](http://lovebetterworld.com/image-20210202141427491.png)

可以根据容器ID停止，或根据容器的NAMES停止：

![image-20210202141523220](http://lovebetterworld.com/image-20210202141523220.png)



## 3.3 启动/重启容器

对于处于退出状态的容器，可以通过start/restart来重启容器：

![image-20210202141638153](http://lovebetterworld.com/image-20210202141638153.png)

```bash
docker start/restart CONTAINER
```

可以根据容器ID启动，或根据容器的NAMES启动：

示例：

```bash
docker start node-exporter
或
docker start f9461795ffea
```



## 3.4 删除容器

对于不需要的容器，通过rm命令删除容器，对于要删除的容器，首先必须将其停止掉。

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

可以根据容器ID删除，或根据容器的NAMES删除。

```bash
docker rm NAMES
或
docker rm IMAGE ID
```

## 3.5 查看容器的日志

```bash
docker logs [OPTIONS] CONTAINER
```

动态实时查看容器日志：

```bash
docker logs -f [OPTIONS] CONTAINER
```

## 3.6 执行容器命令

```bash
docker exec CONTAINER COMMAND
```

通常此命令使用挺频繁，比如对于运行中的容器，排查一些问题或者情况时，除了查看容器的运行日志，我还会去容器内容查看相关文件。

示例：

```bash
docker exec -it IMAGED ID /bin/bash
或
docker exec -it IMAGED ID /bin/sh
```

对于有些容器，我进入容器需要时通过root用户，则通过如下命令：

```bash
docker exec -it -u root IMAGED ID /bin/bash
或
docker exec -it -u root IMAGED ID /bin/sh
```

## 3.7 重命名容器

```bash
docker rename CONTAINER CONTAINER_NEW
```

## 3.8 查看容器的详细信息

```bash
docker inspect IMAGE ID
或
docker inspect NAMES
```

能够查看容器的所有描述信息。

# 四、Docker其他技巧

## 4.1 Docker命令自动补全

自动补齐需要依赖工具 bash-complete，如果没有，则需要手动安装，命令如下：

`[root@docker ~]# yum -y install bash-completion`

安装成功后，得到文件为  /usr/share/bash-completion/bash_completion。

前面已经安装了 bash_completion，执行如下命令：

`[root@docker ~]# source /usr/share/bash-completion/bash_completion`

再次尝试，发现可以正常列出docker的子命令，示例如下：

```bash
[root@docker ~]# docker  （docker + 空格 + 连续按2次Tab键）
attach    container  engine    history   inspect   logs      port     restart   search    stats    top      volume
build     context    events    image     kill      network   ps       rm        secret    stop     trust    wait
builder   cp         exec      images    load      node      pull     rmi       service   swarm    unpause
commit    create     export    import    login     pause     push     run       stack     system   update
config    diff       help      info      logout    plugin    rename   save      start     tag      version
```



## 4.2 查看运行中的容器的启动参数

### 1. 安装pip

```shell
# yum install -y python-pip
```

### 2. 安装runlike

```shell
# pip install runlike
```

### 3. 查看docker run参数

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

# 五、Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)

