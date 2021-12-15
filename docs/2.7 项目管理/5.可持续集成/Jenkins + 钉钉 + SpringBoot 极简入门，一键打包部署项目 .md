## Jenkins + 钉钉 + SpringBoot 极简入门，一键打包部署项目

> 摘要: 原创出处 http://www.iocoder.cn/Jenkins/install/ 「芋道源码」
>
> https://juejin.cn/post/6942098287533129765

# 1. 概述

目前国内绝大多数的团队，都采用 Jenkins 实现持续集成与持续发布。那么 Jenkins 是什么?在《Jenkins 用户文档中心》介绍如下：

> Jenkins 是一款开源 CI&CD 软件，用于自动化各种任务，包括构建、测试和部署软件。
>
> Jenkins 支持各种运行方式，可通过系统包、Docker 或者通过一个独立的 Java 程序。

Jenkins 官方在《Jenkins 用户文档中心》中，已经提供了较为详细的教程，并且已经提供中文翻译，非常友好哈。不过考虑到胖友可能想要更加简便的**快速入门**的教程，于是艿艿就写了本文。

# 2. 快速入门

在本小节，我们会一起来搭建一个 Jenkins 服务，并部署一个 Spring Boot 应用到远程服务器。整个步骤如下：

- 1、搭建一个 Jenkins 服务
- 2、配置 Jenkins 全局工具
- 3、创建一个 Jenkins 任务。该任务从 Git 获取的项目，并使用 Maven 构建，并将构建出来的 jar 包复制远程服务器上，最后进行 Spring Boot 应用的启动。

> 友情提示：如下必备软件，胖友需要自行安装。
>
> Jenkins 所在服务器：
>
> - JDK 8+
> - Maven 3+
> - Git
>
> 远程服务器：
>
> - JDK 8+

## 2.1 Jenkins 搭建

### 2.1 下载

打开 Jenkins 下载页面，选择想要的 Jenkins 版本。这里，我们选择 `jenkins.war` 软件包，通用所有操作系统，切版本为 `2.204.1`。

```
# 创建目录
$ mkdir -p /Users/yunai/jenkins
$ cd /Users/yunai/jenkins

# 下载
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```

### 2.1.2 启动 Jenkins 服务

执行 `nohup java -jar jenkins.war &` 命令，后台启动 Jenkins 服务。因为 `jenkins.war` 内置 Jetty 服务器，所以无需丢到 Tomcat 等等容器下，可以直接进行启动。

执行完成后，我们使用 `tail -f nohup.out` 命令，查看下启动日志。如果看到如下日志，说明启动成功：

```
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

e24a2134060f4604b45708904a4f7a25

This may also be found at: /Users/yunai/.jenkins/secrets/initialAdminPassword
```

默认情况下，Jenkins 内置一个管理员用户。其用户名为 **`admin`**，密码随机生成。这里，我们可以看到一串 **`e24a2134060f4604b45708904a4f7a25`** 就是密码。

## 2.2 Jenkins 配置

### 2.2.1 新手入门

默认情况下，Jenkins 启动在 **8080** 端口。所以，我们可以使用浏览器，访问 http://127.0.0.1:8080/ 地址，进入 Jenkins 首页。因为此时我们未完成 Jenkins 的**入门**，所以被重定向「入门」页面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NVYAxiaX6VEMyhJmjKdwBTTMsicbnfBapaYycEWfLTPLwZDlJJkFPiaIkw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

输入管理员密码，我们进入「新手入门」页面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NzHzhBSQf7ZBo7ozPSeLibXBp8YsYpze4ebB3l2I2qlbmSdMFg3c5zGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

作为一个 Jenkins 萌新，我们当然选择「安装推荐的插件」。此时，我们需要做的就是乖乖的耐心等待 Jenkins 自动下载完插件。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NSXHNlyNmPVScJ2Liamu3dsP2LIP8BmN0FbbMeDgDp8ibabumvp2Yqic1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为插件是从国外下载，所以下载速度可能比较慢，可以打开 B 站刷会视频，哈哈哈。当然，也可能出现插件下载失败的情况，点击重试即可，保持淡定。

### 2.2.2 管理员配置

安装完插件之后，会跳转到「创建第一个管理员用户」界面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NX3ic81icPKx4EUpibauOhkP2PrfAZF4KPGRdQqk3uB8ZtprtiaU7Qolpwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「保存并完成」按钮，完成管理员账号的创建。

### 2.2.3 实例配置

创建完管理员账号，会要求重新登录。使用新的管理员账号登录完毕后，会跳转到「实例配置」界面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NuG9ea8iaLCsKIo6v1XEqSYFtM9yGMuvgQAU9iblw3aDXzn7CoeDJqF7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「保存并完成」按钮，完成实例的配置。此时，通过点击「重启」按钮，完成 Jenkins 的重启。之后，耐心等待 Jenkins 完成重启，我们将会进入 Jenkins 首页。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NDNWQzDlq0Vczo9WxICrcjQO2mvaf59ywUWUUwMnzUW4utBQ526lGBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.2.4 安全其它插件

虽然我们在「2.2.1 新手入门」中，已经安装 Jenkins 推荐的一些插件，但是我们还需要安装如下插件：

- Maven Integration
- Maven Info
- Publish Over SSH
- Extended Choice Parameter
- Git Parameter

从 Jenkins 首页开始，按照「Manage Jenkins -> Manage Plugins」的顺序，进入「插件管理」界面。如下图所示：

- ![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NANpgE29psibsx36RAciaI19nSuaM6D635QXIZjicpPKgrq4diaF560icoVw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)安全其它插件 - 第一步
- ![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NatNiajD4v0L7WHaeBo41PFrPdQ3Bv8eRQov97RmgwIlCbibPaFbKgWjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)安全其它插件 - 第二步

选择上述要安装的几个插件，然后点击「直接安装」按钮。之后，会进入「安装/更新 插件中」界面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NlVgal0pmQFgu6kOQ9TvlTN2uHuLQfGT5p9NIBWRDrJ1qO1xgahbjKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

之后，耐心等待插件安装完成...

### 2.2.4 JDK 配置

从 Jenkins 首页开始，按照「Manage Jenkins -> Global Tool Configuration」的顺序，进入「Global Tool Configuration」界面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NyOVsOsBYufN0Sdtl7Xsc7ypxpAGmBbtyS4rYjHRhSybc2DUA7BMibicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「新增 JDK」按钮，并取消「Install automatically」选项。之后，输入本地的 JAVA_HOME。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NpSHBNwDy0lECED2jJysK7uqdib3nyvdS6QoXL70o7oZQRCibnZVLPxVA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置完成后，点击最下面的「保存」按钮。

### 2.2.5 Maven 配置

从 Jenkins 首页开始，按照「Manage Jenkins -> Global Tool Configuration」的顺序，进入「Global Tool Configuration」界面。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NIImwgaZgVcA4qjW55PrB7nUQ2icRichL3XEq6TayWlnb5s5decqcCTKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「新增 Maven」按钮，并取消「Install automatically」选项。之后，输入本地的 MAVEN_HOME。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NzX46dEPibRCu97GV7eGKmzboF5WSCO3cBq94zQgSVb9KuFhSWypJshw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置完成后，点击最下面的「保存」按钮。

### 2.2.6 SSH 配置

因为我们通过 SSH 复制构建出来的 jar 包到远程服务器上，所以我们需要进行 SSH 配置。这里，我们使用**账号密码**的认证方式，实现 SSH 连接到远程服务器。

从 Jenkins 首页开始，按照「Manage Jenkins -> Configure System」的顺序，进入「配置」界面，然后下拉到最底部。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NBLycdwktf5XDdRib16A9eWWBUvo31zZbraNZ3ubVb4RyTTPYqzPQ9jA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「新增」按钮，并点击「高级」按钮。之后，配置远程服务器的 SSH 信息。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NXrmyBXdAQwUgtIEqOltlWJ01oiaL8gXhpFiauBaPsOXxyIu1qfaGquIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置完成后，点击最下面的「保存」按钮。

## 2.3 远程服务器配置

在远程服务器上，我们需要创建 Java 项目部署的目录。每个公司制定的目录规范不同，这里艿艿分享下自己的。固定在 `/work/projects` 目录下，创建每个项目的部署目录。并且，每个项目独立一个子目录。例如说：

```
$ pwd
/work/projects/lab-41-demo01

$ ls
ls -ls
total 18852
    4 drwxr-xr-x 2 root root     4096 Jan 13 21:21 backup
    4 drwxr-xr-x 2 root root     4096 Jan 13 21:14 build
18840 -rw-r--r-- 1 root root 19288579 Jan 13 21:21 lab-41-demo01.jar
    4 drwxr-xr-x 2 root root     4096 Jan 13 21:16 shell
```

- `lab-41-demo01` 目录，会放置一个项目的所有。

- 在每个**子**目录下，固定分成如下文件/目录：

- - `lab-41-demo01.jar`：项目的 `jar` 包。
  - `build` 目录：Jenkins 构建完项目后的**新** `jar` 包，会上传到 `build` 目录下，避免对**原** `jar` 包覆盖，导致无法正常关闭 Java 服务。
  - `backup` 目录：对历史 `jar` 包的备份目录。每次使用新的 `jar` 启动服务时，会将老的 `jar` 移到 `backup` 目录下备份。
  - `shell` 目录：脚本目录。目前只有 `deploy.sh` 脚本，我们来一起瞅瞅。

整个 `deploy.sh` 脚本，有接近 200 行不到，所以我们先来整体看看。后续，胖友可以点击 传送门 ，进行完整查看。核心代码如下：

```
#!/bin/bash
set -e

# 基础
# export JAVA_HOME=/work/programs/jdk/jdk1.8.0_181
# export PATH=PATH=$PATH:$JAVA_HOME/bin
# export CLASSPATH=$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

DATE=$(date +%Y%m%d%H%M)
# 基础路径
BASE_PATH=/work/projects/lab-41-demo01
# 编译后 jar 的地址。部署时，Jenkins 会上传 jar 包到该目录下
SOURCE_PATH=$BASE_PATH/build
# 服务名称。同时约定部署服务的 jar 包名字也为它。
SERVER_NAME=lab-41-demo01
# 环境
PROFILES_ACTIVE=prod
# 健康检查 URL
HEALTH_CHECK_URL=http://127.0.0.1:8078/actuator/health/

# heapError 存放路径
HEAP_ERROR_PATH=$BASE_PATH/heapError
# JVM 参数
JAVA_OPS="-Xms1024m -Xmx1024m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=$HEAP_ERROR_PATH"
# JavaAgent 参数。可用于配置 SkyWalking 等链路追踪
JAVA_AGENT=

# 备份
function backup() {
    // ... 省略代码
}

# 最新构建代码 移动到项目环境
function transfer() {
    // ... 省略代码
}

# 停止
function stop() {
    // ... 省略代码
}

# 启动
function start() {
    // ... 省略代码
}

# 健康检查
function healthCheck() {
    // ... 省略代码
}

# 部署
function deploy() {
    cd $BASE_PATH
    # 备份原 jar
    backup
    # 停止 Java 服务
    stop
    # 部署新 jar
    transfer
    # 启动 Java 服务
    start
    # 健康检查
    healthCheck
}

deploy
```

- 在开头，我们定义了一堆变量，胖友可以根据其上的注释，进行理解。
- 在结尾，我们可以看到对 `#deploy()` 方法，进行项目的部署。整个步骤，分为 5 步，分别对应 5 个方法。我们逐个方法来看看。

**① backup**

`#backup()` 方法，将**原** `jar` 包备份到 `backup` 目录下。代码如下：

```
function backup() {
    # 如果不存在，则无需备份
    if [ ! -f "$BASE_PATH/$SERVER_NAME.jar" ]; then
        echo "[backup] $BASE_PATH/$SERVER_NAME.jar 不存在，跳过备份"
    # 如果存在，则备份到 backup 目录下，使用时间作为后缀
    else
        echo "[backup] 开始备份 $SERVER_NAME ..."
        cp $BASE_PATH/$SERVER_NAME.jar $BASE_PATH/backup/$SERVER_NAME-$DATE.jar
        echo "[backup] 备份 $SERVER_NAME 完成"
    fi
}
```

**② stop**

`#stop()` 方法，将原 `jar` 包对应的 Java 进程，**进行**优雅关闭。代码如下：

```
function stop() {
    echo "[stop] 开始停止 $BASE_PATH/$SERVER_NAME"
    PID=$(ps -ef | grep $BASE_PATH/$SERVER_NAME | grep -v "grep" | awk '{print $2}')
    # 如果 Java 服务启动中，则进行关闭
    if [ -n "$PID" ]; then
        # 正常关闭
        echo "[stop] $BASE_PATH/$SERVER_NAME 运行中，开始 kill [$PID]"
        kill -15 $PID
        # 等待最大 60 秒，直到关闭完成。
        for ((i = 0; i < 60; i++))
            do  
                sleep 1
                PID=$(ps -ef | grep $BASE_PATH/$SERVER_NAME | grep -v "grep" | awk '{print $2}')
                if [ -n "$PID" ]; then
                    echo -e ".\c"
                else
                    echo '[stop] 停止 $BASE_PATH/$SERVER_NAME 成功'
                    break
                fi
      done
        
        # 如果正常关闭失败，那么进行强制 kill -9 进行关闭
        if [ -n "$PID" ]; then
            echo "[stop] $BASE_PATH/$SERVER_NAME 失败，强制 kill -9 $PID"
            kill -9 $PID
        fi
    # 如果 Java 服务未启动，则无需关闭
    else
        echo "[stop] $BASE_PATH/$SERVER_NAME 未启动，无需停止"
    fi
}
```

- 首先，获得到 Java 服务对应的 PID 进程编号。

- 然后，先 `kill -15` 对应进程，尝试正常关闭 Java 服务。考虑到整个关闭是一个**过程**，所以我们需要等待一段时间，直到 Java 服务正常关闭。

  > 友情提示：这里定义的 60 秒，如果胖友需要更短或者更长，可以自行修改。当然，也可以做成变量，嘿嘿。

- 最后，万一 Java 服务无法正常关闭，则 `kill -9` 对应进程，强制关闭 Java 服务。

  > 友情提示：如果胖友不希望强制关闭 Java 服务，可以考虑此处进行 `exit 1` ，异常退出部署，然后人工介入，进行问题的排查。

**③ transfer**

`#transfer()` 方法，将 `build` 目录的**新** `jar` 包，“覆盖”到**老**的 `jar` 包上。代码如下：

```
function transfer() {
    echo "[transfer] 开始转移 $SERVER_NAME.jar"

    # 删除原 jar 包
    if [ ! -f "$BASE_PATH/$SERVER_NAME.jar" ]; then
        echo "[transfer] $BASE_PATH/$SERVER_NAME.jar 不存在，跳过删除"
    else
        echo "[transfer] 移除 $BASE_PATH/$SERVER_NAME.jar 完成"
        rm $BASE_PATH/$SERVER_NAME.jar
    fi

    # 复制新 jar 包
    echo "[transfer] 从 $SOURCE_PATH 中获取 $SERVER_NAME.jar 并迁移至 $BASE_PATH ...."
    cp $SOURCE_PATH/$SERVER_NAME.jar $BASE_PATH

    echo "[transfer] 转移 $SERVER_NAME.jar 完成"
}
```

- 这里，我们并不是直接覆盖，因为 `cp` 命令覆盖时，系统会提示是否尽心覆盖，需要手动输入 `y` 指令，显然无法满足我们自动化部署的需要。

**④ start**

`#start()` 方法，使用**新**的 `jar` 包，启动 Java 服务。代码如下：

```
function start() {
    # 开启启动前，打印启动参数
    echo "[start] 开始启动 $BASE_PATH/$SERVER_NAME"
    echo "[start] JAVA_OPS: $JAVA_OPS"
    echo "[start] JAVA_AGENT: $JAVA_AGENT"
    echo "[start] PROFILES: $PROFILES_ACTIVE"
    
    # 开始启动
    BUILD_ID=dontKillMe nohup java -server $JAVA_OPS $JAVA_AGENT -jar $BASE_PATH/$SERVER_NAME.jar --spring.profiles.active=$PROFILES_ACTIVE &
    echo "[start] 启动 $BASE_PATH/$SERVER_NAME 完成"
}
```

- 比较简单，核心就是通过 `java -jar` 命令，通过 `jar` 包来启动 Java 服务。

**④ healthCheck**

`#healthCheck()` 方法，通过健康检查 URL ，判断 Java 服务是否启动成功。代码如下：

```
function healthCheck() {
    # 如果配置健康检查，则进行健康检查
    if [ -n "$HEALTH_CHECK_URL" ]; then
        # 健康检查最大 60 秒，直到健康检查通过
        echo "[healthCheck] 开始通过 $HEALTH_CHECK_URL 地址，进行健康检查";
        for ((i = 0; i < 60; i++))
            do
                # 请求健康检查地址，只获取状态码。
                result=`curl -I -m 10 -o /dev/null -s -w %{http_code} $HEALTH_CHECK_URL || echo "000"`
                # 如果状态码为 200，则说明健康检查通过
                if [ "$result" == "200" ]; then
                    echo "[healthCheck] 健康检查通过";
                    break
                # 如果状态码非 200，则说明未通过。sleep 1 秒后，继续重试
                else
                    echo -e ".\c"
                    sleep 1
                fi
            done
        
        # 健康检查未通过，则异常退出 shell 脚本，不继续部署。
        if [ ! "$result" == "200" ]; then
            echo "[healthCheck] 健康检查不通过，可能部署失败。查看日志，自行判断是否启动成功";
            tail -n 10 nohup.out
            exit 1;
        # 健康检查通过，打印最后 10 行日志，可能部署的人想看下日志。
        else
            tail -n 10 nohup.out
        fi
    # 如果未配置健康检查，则 slepp 60 秒，人工看日志是否部署成功。
    else
        echo "[healthCheck] HEALTH_CHECK_URL 未配置，开始 sleep 60 秒";
        sleep 60
        echo "[healthCheck] sleep 60 秒完成，查看日志，自行判断是否启动成功";
        tail -n 50 nohup.out
    fi
}
```

- 和 Java 服务的**关闭**一样，Java 服务的**启动**也是一个**过程**。这里，我们提供了两种策略：1）通过健康检查 URL，自动判断应用是否启动成功。2）未配置健康检查 URL 的情况下，我们通过 sleep 60 秒，然后查看日志，人工判断是否启动成功。
- 健康检查的 URL，我们通过 Spring Boot Actuator 提供的 `health` 端点，判断请求返回的状态码是否为 200 。如果是，则说明应用健康，启动完成。对 Spring Boot Actuator  不了解的胖友，可以后续看看 《芋道 Spring Boot 监控端点 Actuator 入门》的「4. health  端点」小节。现在，胖友暂时可以只需要知道这个设定就好。
- 当然，为了满足胖友的“好奇心”，艿艿最后还是打印了 N 行日志，满足胖友希望看到启动日志的诉求，哈哈哈。

嘿嘿，虽然有 5 个步骤，200 行不到的 Shell 代码，实际还是比较简单，嘻嘻。

## 2.4 Jenkins 部署任务配置

从 Jenkins 首页开始，点击「新建Item」按钮，进入 Jenkins 任务创建界面。输入任务名，并选择构建一个 Maven 项目。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NsE0NnFXJheE2OLRJUntmIBtwq5UYeyeibhFqO6pficBA5nP9MDIFNngg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击「确认」按钮后，进行该任务的配置界面。配置项比较多，如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NTKtO4CtUuY5I9bId3dQPUaVTbU3VmrsZuLG7ibmQwzaLwibr6dItmNJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.4.1 详细配置

下面，我们一个一个配置项，逐个来看看哈。

**① General**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0Nfldq6OSjcMMErUBf3V43rvAokf0DqRQm5pCSo2j2d7TpI0pCAGyxvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)General

- 比较简单，只需要配置下描述即可。

**② Maven Info Plugin Configuration**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NTlVciaz1WHXUObW6aNmWVHNRN4EnZ1vL9g3SqQpPI9dI0rCRdN0pHZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Maven Info Plugin Configuration

- Discard old builds 配置项：设置保留的构建。因为我们会不断的重新构建项目，如果不进行设置，Jenkins 所在服务器的磁盘可能会不够用噢。
- This project is parameterized 配置项：参数化构建。这里，我们使用 Git Parameter 插件，创建了参数名为 `BRANCH`，值为 Git 项目的 Branch/Tag。如此，我们在后续的项目构建中，可以选择构建的 Git 项目的分支/标签啦。

**③ 源码管理**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NGB74KQZfDKNFnt8nO4E4rY1ngbPZoq03fDzRv0AtSdNhBCA1sLPF5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)源码管理

- 选择 Git，从而选择 Git 仓库。
- Repositories 配置项：设置使用的 Git 仓库。这里可以直接使用 https://github.com/YunaiV/SpringBoot-Labs 仓库，艿艿已经准备好了示例项目。
- Branches to build	配置项：设置使用的 Git 分支/标签。这里，我们使用「② Maven Info Plugin Configuration」配置的构建参数 `BRANCH` 。

**④ 构建触发器**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NZPIeEcGesnWbQg4XDFvlib57o2RVIRPvmVZVgn8yp3wfZL46xSagPMA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)构建触发器

- 暂时不需要配置，可无视哈。

**⑤ 构建环境**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NOVCfaicibf1lCzHDZFJ30Qo70gPAPxTYiaLnoibTTN05rVrwvZ3Mcr3bLg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)构建环境

- 暂时不需要配置，可无视哈。

**⑥ Pre Steps**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NOAU290NlZZ3Yw3Io8qGbYaAyTUE0hpiaoQ2zmx5hnFVt5HkKLBI2aAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Pre Steps

- 暂时不需要配置，可无视哈。

**⑦ Build**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NPrqM1TN3FypYaIGuN6erYibyVIicWsY36zfnHRp594E2TVhTnrgaQyYQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Build

- Root POM 配置项：设置根 `pom.xml` 配置文件。一般情况下，设置 `pom.xml` 即可。

- Goals and options	配置项：设置 Maven 构建命令。

- - 这里，因为我们只想构建 lab-41/lab-41-demo01 **子 Maven 模块**，所以使用 `-pl lab-41/lab-41-demo01` 参数。其它 Maven 参数，不了解的话，自己搜索哈。
  - 如果胖友要构建整个项目，可以考虑使用 `clean package -Dmaven.test.skip=true` 命令。

**⑧ Post Steps**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NpMo4Eia1QeYcpFTUWYnZ8EjRb9EC40ZQuFK1vfawygYb3pSvib7D1rIg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Post Steps

- 暂时不需要配置，可无视哈。

**⑧ 构建设置**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NoPt4pd5y0F3hPj1YenWibNBBdV21E7ZAqaM3P7dyRdzrbCEX4KVgqQA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)构建设置

- 暂时不需要配置，可无视哈。

**⑨ 构建后操作**

点击「增加构建后操作步骤」按钮，选择「Send build artifacts over SSH」选项，配置将 Maven 构建出来的 `jar` 包，通过 SSH 发送到远程服务器，并执行相应脚本，进行启动 Java 服务。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0Nbsqo8icJAT6OibXB6nwISiaHRaamRoJmOq7OCYN4rFsWRT13LBDcEEE8g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- Name 配置项：选择部署的远程服务器。这里，我们选择「2.3 远程服务器配置」的服务器。

- Transfer Set Sources files 配置项：设置传输的文件。这里，我们输入 `lab-41/lab-41-demo01/target/*.jar` 地址，表示传输的是 lab-41/lab-41-demo01 **子 Maven 模块**构建出来的 `jar` 包。

- - 使用 `lab-41/lab-41-demo01` 在开头的原因是，因为我们构建的是 lab-41/lab-41-demo01 **子 Maven 模块**。
  - 使用 `target` 在中间的原因是，Maven 构建的结果，在 `target` 目录下。
  - 使用 `*.jar` 在结尾的原因是，我们只传输 Maven 构建出来的 `jar` 包。
  - 如果胖友是使用 `clean install -Dmaven.test.skip=true` 命令时，则此处配置 `target/*.jar` 即可。

- Remove prefix 配置项：设置传输的文件，需要移除的前缀。这里，我们输入了 `lab-41/lab-41-demo01/target/` 地址，表示传输到远程时，文件名仅为 `*.jar` 的名字。

- Remote directory 配置项：传输到远程服务器的目录。这里，我们输入了 `/work/projects/lab-41-demo01/build` 地址，表示传输到远程服务器的 `lab-41-demo01` 项目的 `build` 目录。

- Exec command	 配置项：设置传输完文件后，执行的 Shell 命令。这里，我们输入了 `cd /work/projects/lab-41-demo01/shell && ./deploy.sh` 命令，表示执行部署脚本，进行启动 Java 服务。。

- Exec in pty	配置项：**必须勾选上**，表示模拟一个终端执行脚本。😎 咳咳咳，如果不勾选上它，执行命令会超时。具体的原因，艿艿也没找到原因，反正这么用就完事了。

  > 友情提示：这个配置项，需要点击其所在表单的「高级」按钮，才能展示出来。

- Add Server 按钮：如果要部署到更多的远程服务器，部署多个节点，可以点击，进行配置噢。

之后，点击下面的「高级」按钮，将「Fail the build if an error occurs」选项选择上。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NU8jCZwl7FhscrsfC8OS5icCmMA4r9D0htoA1wUTiajoIz4iaiaXDfhvelA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 通过这个选项，我们在执行 Shell 命令发生错误时，可以将部署任务的每一次部署，标记为 Failure 失败。具体我们会在「2.4.3 失败部署示例」小节中，进行演示。

至此，我们已经完成了 Jenkins 部署任务的配置，点击最下面「保存」按钮，进行配置的保存。美滋滋~

### 2.4.2 成功部署示例

点击「保存」按钮之后，会进入我们配置的部署任务的详情页。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NQWL3ooLtF0aNraaK0XEPeYDM2nwRNUQ7wAcWxClAtibuo8Wscj9FCgg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击左边「Build with Parameters」菜单，进行参数化构建，进行项目的部署。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NwHBnILyJdCucG1GRsMibens1JbxxHnTwhjcrN8WLjBV4mdzvI6tficuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为我们的 https://github.com/YunaiV/SpringBoot-Labs 仓库只有一个 master 分支，所以我们在 `BRANCH` 参数仅有一个选择。点击「开始构建」按钮，开始部署。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NiaUuXsSKNemITpib4GBrm6edI81giancx2PzfgibnN2GEZ9zJrvdG4W36g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击 Build History 中的红圈部分，查看本次构建。之后，点击「 控制台输出」菜单，查看整个构建过程，会看到很多日志。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NcAYgODLr4aM5qkDTNXv9ZkvU0h5dNPPro1VZZyENMNpeGc6huNSFrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

耐心等待，最终看到如下日志，说明部署成功。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NQibhd6MpJqNADx28ib537iadLhRbhShpX3bm4JKhpF2gefV7SsybAURJw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时，我们回到 Jenkins 首页，可以看到该部署任务执行结果是**成功**的。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NibGG8R1E76HcYxC74EQpFyt7vNH6licYzCMm4kkdM3eZSkJVvnty1eUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.4.3 失败部署示例

为了模拟失败部署，我们修改下远程服务器的 `deploy.sh` 脚本，将健康检查 URL 修改如下，用于达到健康检查失败的结果。代码如下：

```
# 健康检查 URL
HEALTH_CHECK_URL=http://127.0.0.1:8078/actuator/health/1
```

- 这样，因为修改后的健康检查 URL 是不存在的，所以健康检查最终肯定是不通过的。如此，`deploy.sh` 脚本最终会 `exit 1`，表示执行错误。

修改完成后，重新进行部署任务的构建，最终会构建失败。控制台输出日志如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0Nk1LwVglEDHhwghFT9p4S118wVRsqemyLJE6ibrU5icaHjvvVnD22sMlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时，我们回到 Jenkins 首页，可以看到该部署任务执行结果是**失败**的。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NllE0XmOMQWHsraa87sorVYhypr3x0w0jgicAQQwroRuSqpwkzYf98Tg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.5 小结

至此，我们已经完成了一个 Jenkins 部署任务的配置的学习。虽然说，整个过程可能有点冗长，加上艿艿是一个啰嗦的人，但是作为一个“配置工程师”，一回生二回熟。

因为本文主要以 Jenkins 为主，所以并未介绍 Java 项目打包等等内容。胖友可以马上去看看《芋道 Spring Boot 持续交付 Jenkins  入门》文章，艿艿会在该文中，分享更多 Spring Boot 如何打包，整个部署过程中如何更好的和 Nginx  健康检查做好结合。一定一定一定要看，哈哈哈。

# 3. 邮件通知

Jenkins 支持配置邮件通知，在构建成功（Success）、构建失败（Fail）、构建不稳定（Unstable）时，发送相应的邮件。

下面，我们在「2. 快速入门」的基础上，配置邮件通知的功能。

## 3.1 全局的邮箱配置

**①** 从 Jenkins 首页开始，按照「Manage Jenkins -> Configure System」的顺序，进入「配置」界面，然后下拉到「**邮件通知**」位置，配置**发送通知的邮箱**。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NPugYogIJQzaNg3mpVKhO9ZSUVMw2nFbHKfd4bzYLIXJzaeNkohkQSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**②** 不过要注意，一些邮箱服务，要求使用 SMTP 认证的用户名和发件邮箱保持一致，例如说网易 163 邮箱。因此，我们上拉到「Jenkins Location」位置，配置**发件邮箱**。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NnHrWX7emHqN5dekAibpKoqYDNXfwFiaQ70A2Sx5KtIEVRiaKtYpYH9nNA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**③** 配置完成后，点击「**邮件通知**」位置的「Test configuration」按钮，测试发送邮箱的功能。如果测试通过，在界面上我们会看到 `"Email was successfully sent"` 提示。同时，我们打开收件邮件，会看到测试邮件，如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NIjbicmhOQPR8tIxskZwyGMSeshVh5ZOVsV4VXZSygmYRLWk6j6iaPk7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

🔥 测试通过后，记得点击「保存」按钮啊！

## 3.2 部署任务的邮箱配置

在「2.4 Jenkins 部署任务配置」的基础上，我们修改邮件配置。一共有两处配置项，我们逐个来看看哈。

**① 构建设置**

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NSS8D3cqakXZybicl0mLo0CjIOKhgiax5HcT5rMjlprvGWMHUk4Ym8VqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)构建设置

- 这里主要配置构建**失败**、构建**不稳定**情况下的邮箱通知配置。一般情况下，按照图中配置即可。

**② 构建后操作**

点击「增加构建后操作步骤」按钮，选择「Editable Email Notification」选项，配置构建**成功**情况下的邮箱通知配置。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NjDNe8ictXE3YSjoGia7DrgQXc0dsicOvMsTOP0waKoUJTwDvTvwtPNacA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- Project Recipient List 配置项：配置收件人列表，每行一个。😈 这里，我们配置了一个邮箱，嘿嘿。

- 其它配置项，我们可以暂时不配置，后面胖友有需要，可以自行配置。另外，上面的 `$DEFAULT_RECIPIENTS`、`$DEFAULT_REPLYTO` 等变量，是由 Extended E-mail Notification 插件提供，胖友可以按照「Manage Jenkins -> Configure System」的顺序，进入「配置」界面，然后下拉到「**Extended E-mail Notification**」位置，进行配置。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NgXQcicibwIDlZ79TmCZl2iauQmDshH1BELGY0lQC6Hqv3ByXLlW52guMw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- - 这里我们先不进行使用配置，使用默认配置即可，嘿嘿。

**③** 简单测试

🔥 测试通过后，记得点击「保存」按钮啊！然后，可以进行一次部署测试，看看邮件通知的效果。

如下是一次构建**成功**的示例，如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NXm8pxTMpanoz1l0fJB5IhwCugwRAbXPWW7Jrgmu3wUetTHNcZpDCtQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如下是一次构建**失败**的示例，如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NniaoLPEf4f9WCd66DRqDdODKYn13vqPN7NrWY0xABrTX8AlsLmyc1ag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 4. 钉钉通知

Jenkins 支持配置**钉钉**通知，在构建成功（Success）、构建失败（Fail）、构建不稳定（Unstable）时，发送相应的邮件。

下面，我们在「2. 快速入门」的基础上，配置邮件通知的功能。

## 4.1 安装插件

默认情况下，提供钉钉通知的 Jenkins 插件 Dingding Notification Plugin 并未安全，所以我们需要进行安装。安装的步骤很简单，和2.2.4 安全其它插件小节是一样的，胖友自己去安装下噢。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0Ngiaia9nzzabhV3P84viasOoiaLHjjiaf248ldFrf9DBhORMbMQ6TEmneBDA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一如既往，安装插件比较慢，我们先接着往下看。

## 4.2 创建钉钉群机器人

参考《芋道 Prometheus + Grafana + Alertmanager 极简入门》文章的「7.5.5.1 创建钉钉群机器人」小节，创建一个钉钉群机器人。

不过有一点要**注意**，Jenkins 目前配置的钉钉通知方式，暂时不支持钉钉群机器人的**“加密”**安全方式，所以胖友请使用**“IP地址（段）”**的安全设置，自己的外网 IP 可以去 http://ip138.com/ 看。

## 4.3 部署任务的钉钉配置

在「2.4 Jenkins 部署任务配置」的基础上，我们修改钉钉配置。仅仅修改一处配置项即可，哇哈哈。

**① 构建后操作**

点击「增加构建后操作步骤」按钮，选择「钉钉通知器配置」选项，配置构建**成功**情况下的钉钉通知配置。如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NSOicPanGUibry0erURG3N6ibk9G5PIg1dto1zAicHiaDuB1ft2Q0N8mBgTQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 具体选择哪些选项，胖友可以根据自己的需要哈。

**②** 简单测试

🔥 配置完成后，记得点击「保存」按钮啊！然后，可以进行一次部署测试，看看钉钉通知的效果。

如下是一次构建**成功**的示例，如下图所示：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdPwMocfbmDXezzWAufWT0NicRfKicnktqHibO94SJbhCZIVcibYx63nKFR5MD1TricY1xdyJDTgVicN6pg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们目前主要采用钉钉通知，而不是邮件通知，嘿嘿。



