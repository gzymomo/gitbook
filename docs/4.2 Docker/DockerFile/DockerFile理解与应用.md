[Dockerfile详解与最佳实践](https://www.cnblogs.com/spec-dog/p/11570394.html)

[《容器高手实战： Dockerfile最佳实践》](https://www.cnblogs.com/bindot/p/dockerfile.html)



# DockerFile是什么？

Dockerfile是一个文本文件，包含了一条条指令，每条指令对应构建一层镜像，Docker基于它来构建一个完整镜像。

DockerFile是用来构建Docker镜像的构建文件，一般分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令，’#’ 为 Dockerfile 中的注释。

# 一、DockerFile解析

1）基础结构由大写的保留字指令+参数构成。
2）指令从上到下顺序执行。
3）#表示注解。
4）每条指令都会创建新的镜像层，并对镜像进行提交。


## 1.1 理解构建上下文（build context）

Docker镜像通过`docker build`指令构建，该指令执行时当前的工作目录就是docker构建的上下文，即build context，上下文中的文件及目录都会作为构建上下文内容发送给Docker Daemon。

```bash
$ docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context
```

如上 –no-cache 表示镜像构建时不使用缓存，-f 指定Dockerfile文件位置， context 指定build context目录。

将一些非必要的文件包含到build context中，会导致build context过大，从而导致镜像过大，会增加镜像构建、推送及拉取的时间，以及容器运行时的大小。

执行docker build时会显示build context的大小，

```bash
Sending build context to Docker daemon  187.8MB
```

**最佳实践建议**

1. 使用.dockerignore来排除不需要加入到build context中的文件，类似于.gitignore
2. 不要安装不必要的包，所有包含的东西都是镜像必须的，非必须的不要包含。
3. 解耦应用，如果应用有分层，解耦应用到多个容器，便于横向扩展，如web应用程序栈包含web服务应用，数据库，缓存等。
4. 最少化镜像层数：只有RUN、COPY、ADD指令会创建镜像层，其它指令创建临时的中间镜像，不会增大镜像构建的大小
5. 如果可能，尽可能使用多阶段构建，只复制你需要的组件到最终镜像，这使得你可以在中间构建阶段包含工具与debug信息，同时又不会增大最终镜像的大小。
6. 排序多行参数：将参数按字母排序，有利于避免包重复，及后续的维护与提高易读性


## 1.2 FROM

**作用**
FROM指定基础镜像，每一个定制镜像，必须以一个现有镜像为基础。因此一个Dockerfile中FROM是必须的指令，并且必须是第一条。使用格式，

```bash
FROM <image>:<tag>
# 注释以#开头。基础镜像的tag可不指定，默认使用latest
# 示例：FROM mysql:5.7
```

**最佳实践建议**

1. 如果不想以任何镜像为基础，则可以使用`FROM scratch`
2. 尽量使用官方镜像作为基础镜像
3. 推荐使用Alpine镜像，因为它足够轻量级（小于5MB），但麻雀虽小五脏俱全，基本具有Linux的基础功能


## 1.3 **RUN**

**作用**
用来执行命令行命令，是最常用的指令之一。使用格式，

```bash
# shell格式，跟直接在命令行输入命令一行
RUN <命令>
# 示例：RUN mkdir -p /usr/src/redis

# exec格式，类似于函数调用
RUN ["可执行文件", "参数1", "参数2"]
```

RUN指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，可以在构建指令中指定–no-cache参数，如：`docker build --no-cache`

**最佳实践建议**

1. 将比较长的复杂的指令通过 \ 分为多行，让Dockerfile文件可读性、可理解性、可维护性更高，将多个指令通过 && 连接，减少镜像的层数
2. 确保每一层只添加必需的东西，任何无关的东西都应该清理掉，如所有下载、展开的文件，apt 缓存文件等，以尽可能减少镜像各层的大小
3. 将`RUN apt-get update` 与 `RUN apt-get install` 组合成一条RUN指令（将apt-get update单独作为一条指令会因为缓存问题导致后续的apt-get install 指令失败）

比如先按如下Dockerfile创建了一个镜像

```bash
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
```

一段时间后，再按以下Dockerfile创建另一个镜像

```bash
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx
```

因为RUN指令创建的镜像层会被缓存，所以下面镜像的`RUN apt-get update`并不会执行，直接使用了前面构建的镜像层，这样，curl、nginx就可能安装已经过时的版本。

因此 在 `apt-get update` 之后立即接 `&& apt-get install -y` ，这叫做“ cache busting”（缓存破坏），也可以通过指定包的版本，来达到同样的目的，这叫“ version pinning” （版本指定）示例：

```bash
RUN apt-get update && apt-get install -y \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    #删除apt 缓存减少镜像层的大小
    && rm -rf /var/lib/apt/lists/*
```

1. 使用管道（pipes）。一些RUN指令依赖于从一个指令管道输出到另一个，如

```bash
RUN wget -O - https://some.site | wc -l > /number
```

Docker使用/bin/sh -c 解释器来执行这些指令，只会评估管道最后一条命令的退出码来确定是否成功，如上例中只要wc -l成功了就算wget失败，也会认为是成功的。
如果要使管道命令的任何一步报错都导致指令失败，则可通过加 `set -o pipefile &&` 来实现，如

```bash
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```

不是所有的shell都支持`-o pipefail`选项，如果不支持的话可以使用如下形式，显式地指定一个支持的shell

```bash
RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
```



## 1.4 COPY | ADD

**作用**
COPY从构建上下文的目录中复制文件/目录到镜像层的目标路径。使用格式，

```bash
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

同RUN一样，也有两种格式。源文件可以多个，甚至可以是通配符，目标路径是容器的绝对路径，可以是相对工作目录（WORKDIR指定）的相对路径，目标路径不存在时会自动创建。使用`--chown=<user>:<group>`来改变文件的所属用户与组。
ADD与COPY的使用格式与性质差不多，但功能更丰富，如源路径可以是URL（下载后放到目标路径下，文件权限为600），也可以为tar压缩包，压缩格式为gzip，bzip2及xz的情况下，ADD 指令将会自动解压缩这个压缩文件到目标路径去

 

**最佳实践建议**

1. 如果在Dockerfile中有多处需要使用不同的文件，分别使用COPY，而不是一次性COPY所有的，这可以保证每一步的构建缓存只会在对应文件改变时，才会失效。比如

```bash
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

如果把`COPY . /tmp/` 放在RUN上面，将使RUN层镜像缓存失效的场景更多——因为 . 目录（当前目录）中任何一个文件的改变都会导致缓存失效。

2. 因为镜像大小的原因， 使用ADD来获取远程包是非常不推荐的，应该使用curl或wget，这种方式可以在不再需要使用时删除对应文件，而不需要增加额外的层，如，应避免如下用法

```bash
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

而应使用

```bash
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

3. 如果不需要使用ADD的自动解压特性，尽量使用COPY（语义更清晰）



## 1.5 CMD

**作用**
CMD指定容器的启动命令。容器实质就是进程，进程就需要启动命令及参数，CMD指令就是用于指定默认的容器主进程的启动命令的。使用格式

```bash
# shell格式
CMD <命令>
# exec格式
CMD ["可执行文件", "参数1", "参数2"...]
# 参数列表格式，在指定了ENTRYPOINT指令后，用CMD来指定具体的参数
CMD ["参数1", "参数2"...]
```

在容器运行时可以指定新的命令来覆盖Dockerfile中设置的这个默认命令

**最佳实践建议**

1. 服务类镜像建议：`CMD ["apache2","-DFOREGROUND"]`，`CMD ["nginx", "-g", "daemon off;"]` 容器进程都应以前台运行，不能以后台服务的形式运行，否则启动就退出了。
2. 其它镜像，建议给一个交互式的shell，如bash，python，perl等：`CMD ["python"]`, `CMD ["php", "-a"]`



## 1.6 ENTRYPOINT

**作用**
ENTRYPOINT的目的和CMD一样，都是在指定容器启动时要运行的程序及参数。ENTRYPOINT在运行时也可以替代，不过比CMD要略显繁琐，需要通过docker run的参数 –entrypoint 来指定。如果指定了ENTRYPOINT，则CMD将只是提供参数，传递给ENTRYPOINT。使用ENTRYPOINT可以在容器运行时直接为默认启动程序添加参数。与RUN指令格式一样，ENTRYPOINT也分为exec格式和shell格式。

**最佳实践建议**

1. ENTRYPOINT可用来指定镜像的主命令，允许镜像能像命令一样运行，可以使用CMD来作为默认的标志（参数），如

   ```bash
   ENTRYPOINT ["s3cmd"]
   CMD ["--help"]
   ```

直接run时，相当于执行了`s3cmd --help`。也可以使用shell脚本，在脚本中做一些预处理的工作，如

```bash
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```



## 1.7 LABEL

**作用**
为镜像添加label以方便组织镜像，记录licensce信息，帮助自动化实现等等。字符串中包含空格需要转义或包含在引号中， 如

```bash
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL com.example.release-date="2019-09-12"
LABEL com.example.version.is-production=""

# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2019-09-12"

# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2019-09-12"
```

## 1.8 ENV

**作用**
ENV设置环境变量，无论是后面的其它指令，如 RUN（使用 $环境变量key 的形式） ，还是运行时的应用，都可以直接使用这里定义的环境变量。使用格式有两种，

```bash
#只能设置一个key value
ENV <key> <value>
#可以设置多个，value中如果包含空格可以使用\来进行转义，也可以通过""括起来；也可以用反斜线来续行
ENV <key1>=<value1> <key2>=<value2>...
```

除了RUN，还有这些指令可以引用环境变量：ADD 、 COPY 、 ENV 、 EXPOSE 、 LABEL 、 USER 、 WORKDIR 、 VOLUME 、STOPSIGNAL 、 ONBUILD

**最佳实践建议**

1. 定义环境变量，更新PATH环境变量，如要使 CMD [“nginx”] 运行，可设置环境变量 `ENV PATH /usr/local/nginx/bin:$PATH`
2. ENV也可以用于定义常量，便于维护



## 1.9 ARG

**作用**
ARG设置构建参数，即docker build命令时传入的参数。和ENV的效果差不多，都是设置环境变量，不同的是，ARG设置的是构建环境的环境变量，在容器运行时是不会存在这些环境变量的。
Dockerfile中的ARG指令是定义参数名称，以及默认值（可选）。该默认值可以在执行构建命令docker build时用 –build-arg <参数名>=<值> 来覆盖。使用格式，

~~~bash
ARG <参数名>[=<默认值>]
```

**最佳实践建议**
1. 不要使用ARG来保存密码之类的信息，因为通过docker history还是可以看到docker build执行时的所有值
2. 使用ARG，对于使用CI系统（持续集成），用同样的构建流程构建不同的 Dockerfile 的时候比较有帮助，避免构建命令必须根据每个 Dockerfile 的内容修改

## 10. WORKDIR

**作用**
WORKDIR用于指定工作目录（或当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，会自动创建。使用格式，
```shell
WORKDIR <工作目录路径>
```

**最佳实践建议**
1. WORKDIR应该使用绝对路径，显得更为清楚、可靠
2. 使用WORKDIR，避免使用`RUN cd … && do-something`，可读性差，难以维护

## 11. VOLUME

**作用**
VOLUME用于定义匿名卷。容器运行时应该尽量保持容器存储层不发生写操作，应该将数据写入存储卷。VOLUME就是为了防止运行时用户忘记将动态文件所保存的目录挂载为卷，我们事先在Dockerfile中指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。使用格式，
```shell
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
~~~

如 `VOLUME /data`， 任何向/data目录写入的数据都会写入匿名卷。可以运行容器时覆盖这个挂载设置 `docker run -d -v host-path:/data xxxx` 

**最佳实践建议**

1. VOLUME应该被用来暴露所有的数据存储，配置存储，或者被容器创建的文件、目录
2. 如果数据动态变化，强烈建议使用VOLUME



## 1.10 EXPOSE

**作用**
EXPOSE指令是声明运行时容器提供的服务端口，也只是一个声明，在容器运行时并不会因为这个声明应用就一定会开启这个端口的服务,容器启动时，还是需要通过 `-p host-port:container-port`来实现映射。EXPOSE主要是帮助镜像使用者了解这个镜像服务的监听端口，以方便进行映射配置，另一个用处是在运行时如果是使用随机端口映射，也就是通过 `docker run -P`的形式时，会自动随机映射EXPOSE声明的端口。使用格式，

~~~bash
EXPOSE <端口1> [<端口2>...]
```

**最佳实践建议**
1. 应该使用常用的惯用的端口，如nginx 80，mongoDB 27017

## 13. USER

**作用**
USER指令和WORKDIR相似，都是改变环境状态并影响以后的层。WORKDIR是改变工作目录， USER则是改变之后的层在执行RUN , CMD以及ENTRYPOINT这类命令时的身份。USER帮助你切换到指定的用户，这个用户必
须是事先建立好的，否则无法切换。使用格式
```shell
USER <用户名>[:<用户组>]
~~~

**最佳实践建议**

1. 如果一个服务不需要权限也能运行，则使用USER来切换到非root用户，如`RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres`
2. 避免使用sudo，因为可能存在一些不可预见的TTY与信号转发行为导致问题，如果实在需要，考虑使用“gosu”。为了减少镜像层数，应避免不断切换USER
   使用gosu示例

```bash
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/
releases/download/1.7/gosu-amd64" \
&& chmod +x /usr/local/bin/gosu \
&& gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```

## 1.11 HEALTHCHECK

**作用**
HEALTHCHECK用于检查容器的健康状态，Docker可通过健康状态来决定是否对容器进行重新调度。使用格式

```
HEALTHCHECK [选项] CMD <命令>
```

支持的选项为

- –interval=<间隔> ：两次健康检查的间隔，默认为30秒
- –timeout=<时长> ：执行健康检查命令的超时时间，如果超时，则本次健康检查就被视为失败，默认30秒
- –retries=<次数> ：当连续失败指定的次数后，将容器状态置为unhealthy ，默认3次

 

命令的返回值决定了该次健康检查的成功与否—— 0 ：成功；1 ：失败；2 ：保留（不要使用这个值），如：

~~~bash
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib
/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
    CMD curl -fs http://localhost/ || exit 1
```
可以使用docker ps 或docker inspect来查看容器的健康状态。

**最佳实践建议**
1. 如果基础镜像有健康检查指令，想要屏蔽掉其健康检查，可以使用`HEALTHCHECK NONE`
2. 对一些可能造成假死（进程还在， 但提供不了服务了）的服务建议提供健康检查，以便及时重新调度恢复服务


## 15. ONBUILD

**作用**
ONBUILD后跟的指令，只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。使用格式
```shell
ONBUILD <其它指令>
~~~

它后面跟的是其它指令，比如 RUN , COPY 等，这些指令在当前镜像构建时并不会被执行。
ONBUILD命令在本镜像的子镜像中执行，把ONBUILD想象为父镜像为子镜像声明的一条指令，Docker会在子镜像所有命令之前执行ONBUILD指令。

**最佳实践建议**

1. 当在ONBUILD指令中使用ADD或COPY时要注意，如果build context中没有指定的资源，可能导致灾难性的错误。



# 二、用缓存镜像提高效率

Docker在构建镜像时会复用缓存中已经存在的镜像，如果明确不使用缓存，则可加参数`docker build --no-cache=true`
使用缓存镜像的规则

1. 从一个已存在于缓存的父镜像开始构建，则会将当前镜像的下一行指令与所有继承于那个父镜像的子镜像比较，如果其中没有一个是使用相同的指令构建的，则缓存失效
2. 大部分情况下，将Dockerfile中的指令与其中一个子镜像简单比较就够了，但是某些指令需要更多的检查与说明：对于ADD，COPY指令，文件内容会被检查，会计算每一个文件的checksum，checksum中不会考虑最后修改及最后访问时间，在缓存中查找时，checksum会与已经存在的镜像进行比较，如果文件中有修改，则缓存失效。除了ADD，COPY命令，缓存检查不会查看容器中的文件来决定缓存匹配，如处理`RUN apt-get -y update`命令时，容器中文件的更新不会进行检查来确定缓存是否命中， 这种情况下， 只会检查指令字符串本身是否匹配。
3. 一旦缓存失效，所有后续的指令都会产生新的镜像，不会再使用缓存。



# 三、其它镜像构建方式

## 3.1 通过标准输入来生成Dockerfile构建，

通过标准输入来生成Dockerfile构建，不会发送build context（从stdin读取build context，只包含Dockerfile），适用于一次性构建，不需要写Dockerfile

```bash
# 将会构建一个名称与tag均为none的镜像
echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
#或
docker build - <<EOF
FROM busybox
RUN echo "hello world"
EOF

# 构建一个命名的镜像
docker build -t myimage:latest - <<EOF
FROM busybox
RUN echo "hello world"
EOF
```

连字符 - 作为文件名告诉Docker从stdin读取Dockerfile

## 3.2 使用stdin来生成Dockerfile， 

使用stdin来生成Dockerfile， 但是使用当前目录作为build context

```bash
# build an image using the current directory as context, and a Dockerfile passed through stdin
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt .
RUN cat /somefile.txt
EOF
```

## 3.3 使用远程git仓库构建镜像

使用远程git仓库构建镜像，从stdin生成Dockerfile

```bash
docker build -t myimage:latest -f - https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c .
EOF
```