[TOC]

编写SpringBoot应用运维脚本除了基本的Shell语法要相对熟练之外，还需要解决两个比较重要的问题：
 - 正确获取目标应用程序的进程ID，也就是获取Process ID（下面称PID）的问题。
 - kill命令的正确使用姿势。
 - 命令nohup的正确使用方式。

# 一、获取PID
一般而言，如果通过应用名称能够成功获取PID，则可以确定应用进程正在运行，否则应用进程不处于运行状态。应用进程的运行状态是基于PID判断的，因此在应用进程管理脚本中会多次调用获取PID的命令。通常情况下会使用grep命令去查找PID，例如下面的命令是查询Redis服务的PID：
```bash
ps -ef |grep redis |grep -v grep |awk '{print $2}'
```
其实这是一个复合命令，每个|后面都是一个完整独立的命令，其中：
 - ps -ef是ps命令加上-ef参数，ps命令主要用于查看进程的相关状态，-e代表显示所有进程，而-f代表完整输出显示进程之间的父子关系，例如下面是笔者的虚拟机中的CentOS 7执行ps -ef后的结果：
[![](https://img2018.cnblogs.com/blog/1412331/202003/1412331-20200301220428894-69054240.png)](https://img2018.cnblogs.com/blog/1412331/202003/1412331-20200301220428894-69054240.png)
 - grep XXX其实就是grep对应的目标参数，用于搜索目标参数的结果，复合命令中会从前一个命令的结果中进行搜索。
 - grep -v grep就是grep命令执行时候忽略grep自身的进程。
 - awk '{print $2}'就是对处理的结果取出第二列。
 - ps -ef |grep redis |grep -v grep |awk '{print $2}'复合命令执行过程就是：
 - <1>通过ps -ef获取系统进程状态。
 - <2>通过grep redis从<1>中的结果搜索redis关键字，得出redis进程信息。
 - <3>通过grep -v grep从<2>中的结果过滤掉grep自身的进程。
 - <4>通过awk '{print $2}'从<3>中的结果获取第二列。

在Shell脚本中，可以使用这种方式获取PID：
```bash
PID=`ps -ef |grep redis-server |grep -v grep |awk '{print $2}'`
echo $PID
```
使用eval简化这个过程：
```bash
PID_CMD="ps -ef |grep docker |grep -v grep |awk '{print \$2}'"
PID=$(eval $PID_CMD)
echo $PID
```


# 二、Kill命令

kill命令的一般形式是kill -N PID，本质功能是向对应PID的进程发送一个信号，然后对应的进程需要对这个信号作出响应，信号的编号就是N，这个N的可选值如下（系统是CentOS 7）：
![](https://img2018.cnblogs.com/blog/1412331/202003/1412331-20200301220446316-1291905981.png)
其中开发者常见的就是9) SIGKILL和15) SIGTERM，它们的一般描述如下：

| 信号编号  | 信号名称  | 描述  |功能| 影响  |
| ------------ | ------------ | ------------ | ------------ | ------------ |
| 15  |  SIGTERM | Termination (ANSI)  |系统向对应的进程发送一个SIGTERM信号   | 进程立即停止，或者释放资源后停止，或者由于等待IO继续处于运行状态，也就是一般会有一个阻塞过程，或者换一个角度来说就是进程可以阻塞、处理或者忽略SIGTERM信号  |
| 9  | SIGKILL  | Kill(can't be caught or ignored) (POSIX)  |系统向对应的进程发送一个SIGKILL信号   | SIGKILL信号不能被忽略，一般表现为进程立即停止（当然也有额外的情况）  |

不带-N参数的kill命令默认就是kill -15。一般而言，kill -9 PID是进程的必杀手段，但是它很有可能影响进程结束前释放资源的过程或者中止I/O操作造成数据异常丢失等问题。

# 三、nohup命令
如果希望在退出账号或者关闭终端后应用进程不退出，可以使用nohup命令运行对应的进程。
> nohup就是no hang up的缩写，翻译过来就是"不挂起"的意思，nohup的作用就是不挂起地运行命令。

nohup命令的格式是：nohup Command [Arg...] [&]，功能是：基于命令Command和可选的附加参数Arg运行命令，忽略所有kill命令中的挂断信号SIGHUP，&符号表示命令需要在后台运行。

这里注意一点，操作系统中有三种常用的标准流：
 - 0：标准输入流STDIN
 - 1：标准输出流STDOUT
 - 2：标准错误流STDERR

直接运行nohup Command &的话，所有的标准输出流和错误输出流都会输出到当前目录nohup.out文件，时间长了有可能导致占用大量磁盘空间，所以一般需要把标准输出流STDOUT和标准错误流STDERR重定向到其他文件，例如nohup Command 1>server.log 2>server.log &。但是由于标准错误流STDERR没有缓冲区，所以这样做会导致server.log会被打开两次，导致标准输出和错误输出的内容会相互竞争和覆盖，因此一般会把标准错误流STDERR重定向到已经打开的标准输出流STDOUT中，也就是经常见到的2>&1，而标准输出流STDOUT可以省略>前面的1，所以：

```bash
nohup Command 1>server.log 2>server.log &修改为nohup Command >server.log 2>&1 &
```

然而，更多时候部署Java应用的时候，应用会专门把日志打印到磁盘特定的目录中便于ELK收集，如笔者前公司的运维规定日志必须打印在/data/log-center/${serverName}目录下，那么这个时候必须把nohup的标准输出流STDOUT和标准错误流STDERR完全忽略。一个比较可行的做法就是把这两个标准流全部重定向到"黑洞/dev/null"中。例如：

```bash
nohup Command >/dev/null 2>&1 &
```

# 四、编写SpringBoot应用运维脚本
SpringBoot应用本质就是一个Java应用，但是会有可能添加特定的SpringBoot允许的参数。

## 4.1全局变量
提取可复用的全局变量。先是定义JDK的位置JDK_HOME：
```bash
JDK_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/bin/java"
```
定义应用的位置APP_LOCATION：
```bash
APP_LOCATION="/data/shell/app.jar"
```
定义应用名称APP_NAME（主要用于搜索和展示）：
```bash
APP_NAME="app"
```
定义获取PID的命令临时变量PID_CMD，用于后面获取PID的临时变量：
```bash
PID_CMD="ps -ef |grep $APP_LOCATION |grep -v grep |awk '{print \$2}'"
// PID = $(eval $PID_CMD)
```
定义虚拟机属性VM_OPTS：
```bash
VM_OPTS="-Xms2048m -Xmx2048m"
```
定义SpringBoot属性SPB_OPTS（一般用于配置启动端口、应用Profile或者注册中心地址等等）：
```bash
SPB_OPTS="--spring.profiles.active=dev"
```

## 4.2编写核心方法
例如脚本的文件是server.sh，那么最后需要使用sh server.sh Command执行，其中Command列表如下：
 - start：启动服务。
 - info：打印信息，主要是共享变量的内容。
 - status：打印服务状态，用于判断服务是否正在运行。
 - stop：停止服务进程。
 - restart：重启服务。
 - help：帮助指南。

通过case关键字和命令执行时输入的第一个参数确定具体的调用方法。
```bash
start() {
 echo "start: start server"
}

stop() {
 echo "stop: shutdown server"
}

restart() {
 echo "restart: restart server"
}

status() {
 echo "status: display status of server"
}

info() {
 echo "help: help info"
}

help() {
   echo "start: start server"
   echo "stop: shutdown server"
   echo "restart: restart server"
   echo "status: display status of server"
   echo "info: display info of server"
   echo "help: help info"
}

case $1 in
start)
    start
    ;;
stop)
    stop
    ;;
restart)
    restart
    ;;
status)
    status
    ;;
info)
    info
    ;;
help)
    help
    ;;
*)
    help
    ;;
esac
exit $?
```
测试：
```bash
[root@localhost shell]# sh server.sh 
start: start server
stop: shutdown server
restart: restart server
status: display status of server
info: display info of server
help: help info
......
[root@localhost shell]# sh c.sh start
start: start server
```
编写对应的方法实现。
## 4.3Info方法
info()主要用于打印当前服务的环境变量和服务的信息等等。
```bash
info() {
  echo "=============================info=============================="
  echo "APP_LOCATION: $APP_LOCATION"
  echo "APP_NAME: $APP_NAME"
  echo "JDK_HOME: $JDK_HOME"
  echo "VM_OPTS: $VM_OPTS"
  echo "SPB_OPTS: $SPB_OPTS"
  echo "=============================info=============================="
}
```
## 4.4status方法
status()方法主要用于展示服务的运行状态。
```bash
status() {
  echo "=============================status==============================" 
  PID=$(eval $PID_CMD)
  if [[ -n $PID ]]; then
       echo "$APP_NAME is running,PID is $PID"
  else
       echo "$APP_NAME is not running!!!"
  fi
  echo "=============================status=============================="
}
```
## 4.5start方法
start()方法主要用于启动服务，需要用到JDK和nohup等相关命令。
```bash
start() {
 echo "=============================start=============================="
 PID=$(eval $PID_CMD)
 if [[ -n $PID ]]; then
    echo "$APP_NAME is already running,PID is $PID"
 else
    nohup $JDK_HOME $VM_OPTS -jar $APP_LOCATION $SPB_OPTS >/dev/null 2>\$1 &
    echo "nohup $JDK_HOME $VM_OPTS -jar $APP_LOCATION $SPB_OPTS >/dev/null 2>\$1 &"
    PID=$(eval $PID_CMD)
    if [[ -n $PID ]]; then
       echo "Start $APP_NAME successfully,PID is $PID"
    else
       echo "Failed to start $APP_NAME !!!"
    fi
 fi  
 echo "=============================start=============================="
}
```
 - 先判断应用是否已经运行，如果已经能获取到应用进程PID，那么直接返回。
 - 使用nohup命令结合java -jar命令启动应用程序jar包，基于PID判断是否启动成功。
## 4.6stop方法
stop()方法用于终止应用程序进程，这里为了相对安全和优雅地kill掉进程，先采用kill -15方式，确定kill -15无法杀掉进程，再使用kill -9。
```bash
stop() {
 echo "=============================stop=============================="
 PID=$(eval $PID_CMD)
 if [[ -n $PID ]]; then
    kill -15 $PID
    sleep 5
    PID=$(eval $PID_CMD)
    if [[ -n $PID ]]; then
      echo "Stop $APP_NAME failed by kill -15 $PID,begin to kill -9 $PID"
      kill -9 $PID
      sleep 2
      echo "Stop $APP_NAME successfully by kill -9 $PID"
    else 
      echo "Stop $APP_NAME successfully by kill -15 $PID"
    fi 
 else
    echo "$APP_NAME is not running!!!"
 fi
 echo "=============================stop=============================="
}
```
## 4.7restart方法
先stop()，再start()。
```bash
restart() {
  echo "=============================restart=============================="
  stop
  start
  echo "=============================restart=============================="
}
```
## 4.8测试
基于SpringBoot依赖只引入spring-boot-starter-web最简依赖，打了一个Jar包app.jar放在虚拟机的/data/shell目录下，同时上传脚本server.sh到/data/shell目录下：
```bash
/data/shell
  - app.jar
  - server.sh
```
![](https://img2018.cnblogs.com/blog/1412331/202003/1412331-20200301220459668-962053040.png)

某一次测试结果如下：
```bash
[root@localhost shell]# sh server.sh info
=============================info==============================
APP_LOCATION: /data/shell/app.jar
APP_NAME: app
JDK_HOME: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/bin/java
VM_OPTS: -Xms2048m -Xmx2048m
SPB_OPTS: --spring.profiles.active=dev
=============================info==============================
......
[root@localhost shell]# sh server.sh start
=============================start==============================
app is already running,PID is 26950
=============================start==============================
......
[root@localhost shell]# sh server.sh stop
=============================stop==============================
Stop app successfully by kill -15 
=============================stop==============================
......
[root@localhost shell]# sh server.sh restart
=============================restart==============================
=============================stop==============================
app is not running!!!
=============================stop==============================
=============================start==============================
Start app successfully,PID is 27559
=============================start==============================
=============================restart==============================
......
[root@localhost shell]# curl http://localhost:9091/ping -s
[root@localhost shell]# pong
```
## 4.9完整的server.sh内容
```bash
#!/bin/bash
JDK_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64/bin/java"
VM_OPTS="-Xms2048m -Xmx2048m"
SPB_OPTS="--spring.profiles.active=dev"
APP_LOCATION="/data/shell/app.jar"
APP_NAME="app"
PID_CMD="ps -ef |grep $APP_NAME |grep -v grep |awk '{print \$2}'"

start() {
 echo "=============================start=============================="
 PID=$(eval $PID_CMD)
 if [[ -n $PID ]]; then
    echo "$APP_NAME is already running,PID is $PID"
 else
    nohup $JDK_HOME $VM_OPTS -jar $APP_LOCATION $SPB_OPTS >/dev/null 2>\$1 &
    echo "nohup $JDK_HOME $VM_OPTS -jar $APP_LOCATION $SPB_OPTS >/dev/null 2>\$1 &"
    PID=$(eval $PID_CMD)
    if [[ -n $PID ]]; then
       echo "Start $APP_NAME successfully,PID is $PID"
    else
       echo "Failed to start $APP_NAME !!!"
    fi
 fi  
 echo "=============================start=============================="
}

stop() {
 echo "=============================stop=============================="
 PID=$(eval $PID_CMD)
 if [[ -n $PID ]]; then
    kill -15 $PID
    sleep 5
    PID=$(eval $PID_CMD)
    if [[ -n $PID ]]; then
      echo "Stop $APP_NAME failed by kill -15 $PID,begin to kill -9 $PID"
      kill -9 $PID
      sleep 2
      echo "Stop $APP_NAME successfully by kill -9 $PID"
    else 
      echo "Stop $APP_NAME successfully by kill -15 $PID"
    fi 
 else
    echo "$APP_NAME is not running!!!"
 fi
 echo "=============================stop=============================="
}

restart() {
  echo "=============================restart=============================="
  stop
  start
  echo "=============================restart=============================="
}

status() {
  echo "=============================status==============================" 
  PID=$(eval $PID_CMD)
  if [[ -n $PID ]]; then
       echo "$APP_NAME is running,PID is $PID"
  else
       echo "$APP_NAME is not running!!!"
  fi
  echo "=============================status=============================="
}

info() {
  echo "=============================info=============================="
  echo "APP_LOCATION: $APP_LOCATION"
  echo "APP_NAME: $APP_NAME"
  echo "JDK_HOME: $JDK_HOME"
  echo "VM_OPTS: $VM_OPTS"
  echo "SPB_OPTS: $SPB_OPTS"
  echo "=============================info=============================="
}

help() {
   echo "start: start server"
   echo "stop: shutdown server"
   echo "restart: restart server"
   echo "status: display status of server"
   echo "info: display info of server"
   echo "help: help info"
}

case $1 in
start)
    start
    ;;
stop)
    stop
    ;;
restart)
    restart
    ;;
status)
    status
    ;;
info)
    info
    ;;
help)
    help
    ;;
*)
    help
    ;;
esac
exit $?
```