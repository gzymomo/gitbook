[Shell脚本控制docker容器启动顺序](https://www.cnblogs.com/songsongsun/p/14486371.html)



## 1.遇到的问题

在分布式项目部署的过程中,经常要求服务器重启之后,应用(包括数据库)能够自动恢复使用.虽然使用`docker update --restart=always containerid`能够让容器自动随docker启动,但是并不能保证是在数据库启动之后启动,如果数据库未启动,那么将导致应用启动失败;网上还有一种解决方法是通过docker-compose容器编排来控制启动顺序,这个博主研究的比较少.



## 2.解决思路

使用Shell脚本来控制,思路大致如下

1. 探测数据库端口来检验数据库是否启动成功.
2. 数据库启动成功后,探测配置中心及服务注册中心的端口来检验其是否启动成功.
3. 当数据库及配置中心都启动之后,再启动其他微服务应用.



## 3.端口探测

端口探测使用的命令是

```
nc -w 1 host port </dev/null && echo "200"
```

host:目标主机的ip

port:服务监听的端口

如果服务启动了 这条命令会返回 200,未启动则返回空.



## 4.Shell脚本

直接贴代码了,使用的配置中心是nacos

```sh
#!/bin/bash
#chkconfig: 2345 80 90
#description:autoStartMaintenanceService.sh
#
#前提:
#1.docker必须能开机自启
#2.docker能够正常启动运维服务
#3.此脚本须运行微服务所在的机器上
#
##需要修改的配置-----开始
##数据库所在的机器IP
DATABASE_HOST=192.169.1.52
##数据库监听的端口
DATABASE_PORT=3306
##微服务所在机器IP
LOCAL_HOST=192.169.1.46
##微服务访问端口
Maintenance_Port=8180
##NACOS所在机器的ip
NACOS_HOST=192.169.1.82
##NACOS的监听端口
NACOS_PORT=8848
##微服务容器名称(NAMES列)
Maintenance_Container_Name="umc-maintenance"
##该脚本生成的日志路径
Log_Path=/home/test/log
##需要修改的配置-----结束
##
##循环延时时间(s)秒
LOOP_TIME=5
at_time=""
at_date=""

getAtTime() {
    at_time="$(date +%Y-%m-%d-%H:%M:%S) --- "
    at_date=$(date +%Y-%m-%d)
}

autoStartWebService() {
    ##如果日志路径不存在则创建
    if [ ! -d "$Log_Path" ]; then
        mkdir -p $Log_Path
    fi

    while true; do
        ##判断数据库是否启动
        req_message=$(nc -w 1 ${DATABASE_HOST} ${DATABASE_PORT} </dev/null && echo "200")
        if [ -n "$req_message" ]; then
            getAtTime
            echo "$at_time Database is running" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
            waitNacosStarting
        else
            getAtTime
            echo "$at_time Database is not running and please wait for Database starting" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
            sleep $LOOP_TIME
        fi
    done
}
##判断Nacos是否启动
waitNacosStarting() {
    req_message=$(nc -w 1 ${NACOS_HOST} ${NACOS_PORT} </dev/null && echo "200")
    if test $((req_message)) -eq 200; then
        getAtTime
        echo "$at_time Nacos is running" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
        startMaintenanceService
        sleep $LOOP_TIME
    else
        getAtTime
        echo "$at_time Nacos is not running and please wait for nacos starting" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
        sleep $LOOP_TIME
    fi
}

##启动微服务
startMaintenanceService() {
    req_message=$(nc -w 1 ${LOCAL_HOST} ${Maintenance_Port} </dev/null && echo "200")
    if test $((req_message)) -eq 200; then
        getAtTime
        echo "$at_time Maintenance service is running" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
    else
        container_id=$(docker ps -a | grep $Maintenance_Container_Name | grep -v grep | awk '{print $1}')
        getAtTime
        echo "$at_time Maintenance service container id is ${container_id}" >>${Log_Path}/"$at_date"_autoStartMaintenanceService.log
        docker start ${container_id}
    fi

}

autoStartWebService
```



## 5.Shell输入输出重定向

写这个脚本的时候,也让博主对Shell输入输出重定向更加熟悉

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

| 命令             | 说明                                             |
| ---------------- | ------------------------------------------------ |
| command > file   | 将输出重定向到 file且会覆盖file                  |
| command < file   | 将输入重定向到 file                              |
| command >> file  | 将输出以追加的方式重定向到file                   |
| command 2> file  | 将错误输出到file且会覆盖file                     |
| command 2>> file | 将错误以追加的方式重定向到file                   |
| << tag           | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |

如果希望将 stdout 和 stderr 合并后重定向到 file(即将正确信息和错误信息都输出到file)，可以这样写：

```sh
command > file 2>&1
或者
command >> file 2>&1
```



### /dev/null文件

**/dev/null**是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到**禁止输出**的效果

`command > /dev/null 2>&1` 可以屏蔽stdout和stderr