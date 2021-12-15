- [编写 Dockerfile 最佳实践](https://mp.weixin.qq.com/s/Vq64iXB3fPD9J9ju4XirxA)

# 1、减少镜像层

一次RUN指令形成新的一层，尽量Shell命令都写在一行，减少镜像层。
例如：

```bash
FROM centos:7
MAINTAINER www.aliangedu.cn
RUN yum install epel-release -y 
RUN yum install -y gcc gcc-c++ make -y
RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz
RUN tar zxf php-5.6.36.tar.gz
RUN cd php-5.6.36
RUN ./configure --prefix=/usr/local/php 
RUN make -j 4 
RUN make install
EXPOSE 9000
CMD ["php-fpm"]
```

应该写成：

```bash
FROM centos:7
MAINTAINER www.aliangedu.cn
RUN yum install epel-release -y && \
    yum install -y gcc gcc-c++ make

RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz && \
    tar zxf php-5.6.36.tar.gz && \
    cd php-5.6.36 && \
    ./configure --prefix=/usr/local/php && \
    make -j 4 && make install
EXPOSE 9000
CMD ["php-fpm"]
```

结果：**12层 -> 6层**

# 2、优化镜像大小：清理无用数据

一次RUN形成新的一层，如果没有在同一层删除，无论文件是否最后删除，都会带到下一层，所以要在每一层清理对应的残留数据，减小镜像大小。

```bash
FROM centos:7
MAINTAINER www.aliangedu.cn
RUN yum install epel-release -y && \
    yum install -y gcc gcc-c++ make gd-devel libxml2-devel \
    libcurl-devel libjpeg-devel libpng-devel openssl-devel \
    libmcrypt-devel libxslt-devel libtidy-devel autoconf \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*

RUN wget http://docs.php.net/distributions/php-5.6.36.tar.gz && \
    tar zxf php-5.6.36.tar.gz && \
    cd php-5.6.36 && \
    ./configure --prefix=/usr/local/php \
    make -j 4 && make install && \
    cd / && rm -rf php*
```

至少能节省几十M，甚至几百M。

# 3、减少网络传输时间

最好在内部有一个存放软件包的地方，类似于上述的PHP官方下载地址：http://docs.php.net/distributions/php-5.6.36.tar.gz，如果用到maven构建这样的操作，同时也更改为私有maven仓库，减少网络传输时间，提高镜像构建速度。

# 4、多阶段进行镜像构建

- 如果运行一个项目，根据咱们上面的做法，是直接把代码拷贝到基础镜像里，如果是一个需要预先代码编译的项目呢？例如JAVA语言，如何代码编译、部署在一起完成呢！
- 上面做法需要事先在一个Dockerfile构建一个基础镜像，包括项目运行时环境及依赖库，再写一个Dockerfile将项目拷贝到运行环境中，有点略显复杂了。
- 像JAVA这类语言如果代码编译是在Dockerfile里操作，还需要把源代码构建进去，但实际运行时只需要构建出的包，这种把源代码放进去有一定安全风险，并且也增加了镜像体积。
  为了解决上述问题，Docker 17.05开始支持多阶段构建（multi-stage builds），可以简化Dockerfile，减少镜像大小。

例如，构建JAVA项目镜像：

```bash
# git clone https://github.com/lizhenliang/tomcat-java-demo
# cd tomcat-java-demo
# vi Dockerfile
FROM maven AS build
ADD ./pom.xml pom.xml
ADD ./src src/
RUN mvn clean package

FROM lizhenliang/tomcat
RUN rm -rf /usr/local/tomcat/webapps/ROOT
COPY --from=build target/*.war /usr/local/tomcat/webapps/ROOT.war
# docker build -t demo:v1 .
# docker container run -d -v demo:v1
```

首先，第一个FROM 后边多了个 AS 关键字，可以给这个阶段起个名字。
然后，第二部分FROM用的我们上面构建的Tomcat镜像，COPY关键字增加了—from参数，用于拷贝某个阶段的文件到当前阶段。这样一个Dockerfile就都搞定了。

# 5、选择小的基础镜像

- 选择原则一：追求镜像小，可使用Alpine镜像，Alpine是一个轻量级的Linux发行版，镜像仅有5.6MB，构建出的镜像也很小，但其采用MuslLibc，相比Glibc兼容性市面主流技术较差，需进一步测试。
- 选择原则二：追求稳定及使用习惯，使用CentOS镜像。
- 选择原则三：追求稳定及软件包新版，使用Ubuntu/Debian镜像，软件包新版发布上线快。