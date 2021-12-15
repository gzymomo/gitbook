- [正确的在 docker 中启动 springboot](https://shanhy.blog.csdn.net/article/details/110051774)



使用 docker 构建 springboot 应用

# 1、直接启动

直接在 `ENTRYPOINT` 中写命令

```java
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","${JAVA_OPTS}","-jar","/app.jar"]
```



# 2、通过脚本启动

编写一个 sh 启动脚本，然后在 `ENTRYPOINT` 中执行它

```java
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY run.sh .
COPY target/*.jar app.jar
ENTRYPOINT ["run.sh"]
```

然后对应的 run.sh 脚本内容如下：

```java
#!/bin/sh
其他命令XXX1
其他命令XXX2
其他命令XXX3
exec java $JAVA_OPTS -jar /app.jar
```

运行如上两种不同 `ENTRYPOINT` 方式的容器，然后进入 docker 内的命令行，使用 `ps -ef` 查看进程的 PID，你会发现：
1、第一种方式构建的镜像，容器启动后 java 进程的 pid 为 1
2、第二种方式构建的镜像，容器启动后 java 进程的 pid 一定不是 1，pid 为 1 的进程是我们的 run.sh 脚本



**当 docker 容器被正常关闭时，只有 init（pid 1）进程能收到中断信号，如果容器的 pid 1 进程是 sh 进程，它不具备转发结束信号到它的子进程的能力（而我们第二种在 run.sh 中启动的 java 进程是 pid 为 1 的 run.sh 进程的子进程），所以我们真正的 java 程序得不到中断信号，springboot 也就不能正常的回收资源（比如正在执行中的线程不能正常的执行结束），也就不能实现优雅关闭（会导致我们程序被暴力终止无法善始善终）。**



## 解决

根据上文内容，得出解决 “run.sh” 方式启动服务的思路：
1、让pid 1 进程具备转发终止信号
2、或者将 java 程序配成 pid 1 进程

**方法1：**
在 docker 镜像中配置 tini 作为 init（pid 1）进程，因为 tini 具备转发信号的能力（[tini官方地址](https://github.com/krallin/tini)）
编辑 Dockerfile 文件，添加如下内容：

```bash
# Add Tini
ENV TINI_VERSION v0.19.0 
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /sbin/tini
RUN chmod +x /sbin/tini
```

修改 `ENTRYPOINT`

```bash
ENTRYPOINT [ "/sbin/tini", "--", "run.sh"]
```

> 像 jenkins、rancher 等中间件也使用了这样的方式

**方法2：**
在 run.sh 中，使用 exec 启动 java 子进程。

> 注：使用exec command方式，会用command进程替换当前shell进程，并且保持PID不变。执行完毕，直接退出，不回到之前的shell环境。

这里就不需要修改 Dockerfile 文件了，我们需要修改启动脚本（run.sh），内容如下：

```bash
#!/bin/sh
其他命令XXX1
其他命令XXX2
其他命令XXX3
exec java $JAVA_OPTS -jar /app.jar
```

至此，问题解决，可以构建镜像后运行容器，通过 ps -ef 查看 pid，这样以来当我们正常停止容器时，容器内部的 springboot 服务程序在收到信号后，会执行相关的资源回收操作（比如很多 destroy 方法等）。