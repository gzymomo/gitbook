- [基于BuildKit优化Dockerfile的构建](https://os.51cto.com/art/202104/660131.htm)



## Docker镜像构建神器 BuildKit

Docker通过读取Dockerfile中的指令自动构建镜像，Dockerfile是一个文本文件，其中依次包含构建给定镜像所需的所有命令。

![图片](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEbpic9Pk3r0iayT7CN5Kw3hQIbRUFCTYG3XUpM1zVdWYvt0aibNj2Gfe94QUGvLLmgx3Ccib05w5pribUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的解释摘自Docker的官方文档并总结了Dockerfile的用途。Dockerfile的使用非常重要，因为它是我们的蓝图，是我们添加到Docker镜像中的层的记录。

本文，我们将学习如何利用BuildKit功能，这是Docker v18.09上引入的一组增强功能。集成BuildKit将为我们提供更好的性能，存储管理和安全性。

### 本文目标

- 减少构建时间；
- 缩小镜像尺寸；
- 获得可维护性；
- 获得可重复性；
- 了解多阶段Dockerfile;
- 了解BuildKit功能。

### 先决条件

- Docker概念知识
- 已安装Docker（当前使用v19.03）
- 一个Java应用程序（在本文中，我使用了一个Jenkins Maven示例应用程序）

让我们开始吧！

### 简单的Dockerfile示例

以下是一个包含Java应用程序的未优化Dockerfile的示例。我们将逐步进行一些优化。

```
FROM debian
COPY . /app
RUN apt-get update
RUN apt-get -y install openjdk-11-jdk ssh emacs
CMD [“java”, “-jar”, “/app/target/my-app-1.0-SNAPSHOT.jar”]
```

在这里，我们可能会问自己：构建需要多长时间？为了回答这个问题，让我们在本地开发环境上创建该Dockerfile，并让Docker构建镜像。

```
# enter your Java app folder
cd simple-java-maven-app-master
# create a Dockerfile
vim Dockerfile
# write content, save and exit
docker pull debian:latest # pull the source image
time docker build --no-cache -t docker-class . # overwrite previous layers
# notice the build time
0,21s user 0,23s system 0% cpu 1:55,17 total
```

此时，我们的构建需要1m55s。

如果我们仅启用BuildKit而没有其他更改，会有什么不同吗？

### 启用BuildKit

BuildKit可以通过两种方法启用：

在调用Docker build命令时设置DOCKER_BUILDKIT = 1环境变量，例如：

```
time DOCKER_BUILDKIT=1 docker build --no-cache -t docker-class
```

将Docker BuildKit设置为默认开启，需要在/etc/docker/daemon.json进行如下设置，然后重启：

```
{ "features": { "buildkit": true } }
```

BuildKit最初的效果

```
DOCKER_BUILDKIT=1 docker build --no-cache -t docker-class .
0,54s user 0,93s system 1% cpu 1:43,00 total
```

此时，我们的构建需要1m43s。在相同的硬件上，构建花费的时间比以前少了约12秒。这意味着构建几乎无需费力即可节约10％左右的时间。

现在让我们看看是否可以采取一些额外的步骤来进一步改善。

### 从最小到最频繁变化的顺序

因为顺序对于缓存很重要，所以我们将COPY命令移到更靠近Dockerfile末尾的位置。

```
FROM debian
RUN apt-get update
RUN apt-get -y install openjdk-11-jdk ssh emacs
RUN COPY . /app
CMD [“java”, “-jar”, “/app/target/my-app-1.0-SNAPSHOT.jar”]
```

### 避免使用“COPY .”

选择更具体的COPY参数，以避免缓存中断。仅复制所需内容。

```
FROM debian
RUN apt-get update
RUN apt-get -y install openjdk-11-jdk ssh vim
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### apt-get update 和install命令一起使用

这样可以防止使用过时的程序包缓存。

```
FROM debian
RUN apt-get update && \
    apt-get -y install openjdk-11-jdk ssh vim
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 删除不必要的依赖

在开始时，不要安装调试和编辑工具，以后可以在需要时安装它们。

```
FROM debian
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
    openjdk-11-jdk
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 删除程序包管理器缓存

你的镜像不需要此缓存数据。借此机会释放一些空间。

```
FROM debian
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
    openjdk-11-jdk && \
    rm -rf /var/lib/apt/lists/*
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 尽可能使用官方镜像

使用官方镜像有很多理由，例如减少镜像维护时间和减小镜像尺寸，以及预先配置镜像以供容器使用。

```
FROM openjdk
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 使用特定标签

请勿使用latest标签。

```
FROM openjdk:8
COPY target/my-app-1.0-SNAPSHOT.jar /app
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 寻找最小的镜像

以下是openjdk镜像列表。选择最适合自己的最轻的那个镜像。

```
REPOSITORY TAG标签 SIZE大小
openjdk 8 634MB
openjdk 8-jre 443MB
openjdk 8-jre-slim 204MB
openjdk 8-jre-alpine 83MB
```

### 在一致的环境中从源构建

如果你不需要整个JDK，则可以使用Maven Docker镜像作为构建基础。

```
FROM maven:3.6-jdk-8-alpine
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn -e -B package
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 在单独的步骤中获取依赖项

可以缓存–用于获取依赖项的Dockerfile命令。缓存此步骤将加快构建速度。

```
FROM maven:3.6-jdk-8-alpine
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package
CMD [“java”, “-jar”, “/app/my-app-1.0-SNAPSHOT.jar”]
```

### 多阶段构建：删除构建依赖项

为什么要使用多阶段构建？

- 将构建与运行时环境分开

- DRY方式

- 

- - 具有开发，测试等环境的不同详细信息

- 线性化依赖关系

- 具有特定于平台的阶段

```
FROM maven:3.6-jdk-8-alpine AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn -e -B dependency:resolve
COPY src ./src
RUN mvn -e -B package

FROM openjdk:8-jre-alpine
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]
```

如果你此时构建我们的应用程序，

```
time DOCKER_BUILDKIT=1 docker build --no-cache -t docker-class .
0,41s user 0,54s system 2% cpu 35,656 total
```

你会注意到我们的应用程序构建需要大约35.66秒的时间。这是一个令人愉快的进步。

下面，我们将介绍其他场景的功能。

### 多阶段构建：不同的镜像风格

下面的Dockerfile显示了基于Debian和基于Alpine的镜像的不同阶段。

```
FROM maven:3.6-jdk-8-alpine AS builder
…
FROM openjdk:8-jre-jessie AS release-jessie
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]

FROM openjdk:8-jre-alpine AS release-alpine
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]
```

要构建特定的镜像，我们可以使用–target参数：

```
time docker build --no-cache --target release-jessie .
```

### 不同的镜像风格（DRY /全局ARG）

```
ARG flavor=alpine
FROM maven:3.6-jdk-8-alpine AS builder
…
FROM openjdk:8-jre-$flavor AS release
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]
```

ARG命令可以指定要构建的镜像。在上面的例子中，我们指定alpine为默认的镜像，但我们也可以在docker build命令中，通过–build-arg flavor=参数指定镜像。

```
time docker build --no-cache --target release --build-arg flavor=jessie .
```

### 并发

并发在构建Docker镜像时很重要，因为它会充分利用可用的CPU线程。在线性Dockerfile中，所有阶段均按顺序执行。通过多阶段构建，我们可以让较小的依赖阶段准备就绪，以供主阶段使用它们。

BuildKit甚至带来了另一个性能上的好处。如果在以后的构建中不使用该阶段，则在结束时将直接跳过这些阶段，而不是对其进行处理和丢弃。

下面是一个示例Dockerfile，其中网站的资产是在一个assets阶段中构建的：

```
FROM maven:3.6-jdk-8-alpine AS builder
…
FROM tiborvass/whalesay AS assets
RUN whalesay “Hello DockerCon!” > out/assets.html

FROM openjdk:8-jre-alpine AS release
COPY --from=builder /app/my-app-1.0-SNAPSHOT.jar /
COPY --from=assets /out /assets
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]
```

这是另一个Dockerfile，其中分别编译了C和C ++库，并在builder以后使用该阶段。

```
FROM maven:3.6-jdk-8-alpine AS builder-base
…

FROM gcc:8-alpine AS builder-someClib
…
RUN git clone … ./configure --prefix=/out && make && make install

FROM g++:8-alpine AS builder-some CPPlib
…
RUN git clone … && cmake …

FROM builder-base AS builder
COPY --from=builder-someClib /out /
COPY --from=builder-someCpplib /out /
```

### BuildKit应用程序缓存

BuildKit具有程序包管理器缓存的特殊功能。以下是一些缓存文件夹位置的示例：

包管理器	路径

```
apt /var/lib/apt/lists
go ~/.cache/go-build
go-modules $GOPATH/pkg/mod
npm ~/.npm
pip ~/.cache/pip
```

我们可以将此Dockerfile与上面介绍的在一致的环境中从源代码构建中介绍的Dockerfile进行比较。这个较早的Dockerfile没有特殊的缓存处理。我们可以使用–mount=type=cache来做到这一点。

```
FROM maven:3.6-jdk-8-alpine AS builder
WORKDIR /app
RUN --mount=target=. --mount=type=cache,target /root/.m2 \
    && mvn package -DoutputDirectory=/

FROM openjdk:8-jre-alpine
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar /
CMD [“java”, “-jar”, “/my-app-1.0-SNAPSHOT.jar”]
```

### BuildKit的安全功能

BuildKit具有安全功能，下面的示例中，我们使用了–mount=type=secret隐藏了一些机密文件，例如~/.aws/credentials。

```
FROM <baseimage>
RUN …
RUN --mount=type=secret,id=aws,target=/root/.aws/credentials,required \
./fetch-assets-from-s3.sh
RUN ./build-scripts.sh
```

要构建此Dockerfile，需要使用–secret参数：

```
docker build --secret id=aws,src=~/.aws/credentials
```

还有为了提高安全性，避免使用诸如COPY ./keys/private.pem /root .ssh/private.pem之类的命令，我们可以使用BuildKit中的ssh解决此问题：

```
FROM alpine
RUN apk add --no-cache openssh-client
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
ARG REPO_REF=19ba7bcd9976ef8a9bd086187df19ba7bcd997f2
RUN --mount=type=ssh,required git clone git@github.com:org/repo /work && cd /work && git checkout -b $REPO_REF
```

要构建此Dockerfile，你需要在ssh-agent中加载到你的SSH私钥。

```
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa # this is the SSH key default location
docker build --ssh=default .
```