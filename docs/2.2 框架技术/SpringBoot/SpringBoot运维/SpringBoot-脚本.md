[TOC]

```java
SpringBoot：
nohup java -jar zlkj-data-server-1.0-SNAPSHOT.jar --spring.profiles.active=prod > /usr/local/program/jgp.log &

Java：
nohup java -jar JT808.jar > /usr/local/program/JT808.log &
```

# 1、启动
编写启动脚本startup.sh：
```bash
#!/bin/bash
echo Starting application
nohup java -jar springboot_helloword-0.0.1-SNAPSHOT.jar &
```

# 2、关闭
编写关闭脚本stop.sh：
```bash
#!/bin/bash
PID=$(ps -ef | grep springboot.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    echo '服务已经启动！！！'
else
    echo '停止服务！'
    kill $PID
    nohup java -jar springboot.jar --spring.profiles.active=prod > /root/logs/springboot.log &
    echo '查看日志'
    tail -f /root/logs/springboot.log
fi
```

# 3、重启
编写重启脚本restart.sh：
```java
#!/bin/bash
echo Stopping application
source ./stop.sh
echo Starting application
source ./startup.sh
```

# 4、脚本授权
```bash
chmod +x startup.sh
```

# 5、脚本案例
启动项目：
```shell
nohup java -jar -Dserver.port=8761 -Dspring.profiles.active=test jgp-service-register-1.0-SNAPSHOT.jar  > /logs/jgp.log 2>&1 &
```
```shell
	nohup java -jar -Dserver.port=8025 -Dspring.profiles.active=prod data-server-dz.jar  > /logs/data-server.log 2>&1 &
Dockerfile设置虚拟机参数：
	ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","-Xmx1024m","-Xms1024m","/swapping.jar"]
		ENTRYPOINT 配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。
```
