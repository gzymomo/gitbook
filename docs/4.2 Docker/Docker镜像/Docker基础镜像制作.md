# 一、docker通过dockerfile构建JDK最小镜像

## 1.1 下载JRE

下载地址：https://www.java.com/en/download/manual.jsp

## 1.2 解压JRE,删除相关不需要文件

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

## 1.3 重新打包

重新打包所有文件(不打包也可以，在Dockerfile里ADD这个目录即可，当前精简完jre目录大小是107M，压缩后是41M)

## 1.4 创建Dockerfile

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

## 1.5 docker构建镜像

构建：

```bash
docker build -t fds/java8:1.0 .
```

## 1.6 测试运行

```javascript
# docker run -it voole/java8:1.0
/opt # java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

# 二、Docker导出导入镜像

## 2.1 保存镜像到本地

```bash
docker save -o jdk.tar imageId
```

## 2.2 将打包镜像导入目标服务器，并使用docker导入

```bash
docker load -i images.tar
```