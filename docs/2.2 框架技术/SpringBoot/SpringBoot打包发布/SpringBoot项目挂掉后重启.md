[作者：ykz200](http://blog.bhusk.com/articles/2019/05/07/1557208217068)



1、启动一台 Eureka：
 ` java -jar clouddo-eureka-0.0.1-SNAPSHOT.jar`

2、启动脚本：sh ./eureka.sh ，脚本文件和 jar 包放在同一个目录下。脚本内容：

```bash
#!/bin/sh
while :
do
run=$(ps -ef |grep "clouddo-eureka-0.0.1-SNAPSHOT" |grep -v "grep")
if [ "$run" ] ; then
echo "The service is alive!"
else
echo "The service was shutdown!"
echo "Starting service ..."
nohup java -jar $PWD/clouddo-eureka-0.0.1-SNAPSHOT.jar&
echo "The service was started!"
fi
sleep 10
done
```

注意：这里是在 while 死循环下，每隔 10 秒检测一次我们的 eureka 进程，如果进程存在则打印 The service is alive!，如果进程失败就执行重启命令。当然，我们也可以指定这段 shell 开机运行，这样就可以省去很多事情。
 3、杀掉服务：
 将服务 kill 掉  kill -9 xxx
 4、服务自动重启
 xxx      3829  9.4 14.3 2635272 294356 ?      Sl   11:04   0:45 Java -jar clouddo-blackdir-0.0.1-SNAPSHOT.jar



