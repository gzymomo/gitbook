在Docker容器创建好之后，可能会发现容器时间跟宿主机时间不一致，这就需要同步它们的时间，让容器时间跟宿主机时间保持一致。如下：

```bash
宿主机时间
[root@slave-1 ~]# date
Fri May 12 11:20:30 CST 2017
 
容器时间
[root@slave-1 ~]# docker exec -ti 87986863838b /bin/bash
root@87986863838b:/# date                                                                                                                    
Fri May 12 03:20:33 UTC 2017
 
发现两者之间的时间相差了八个小时！
宿主机采用了CST时区，CST应该是指（China Shanghai Time，东八区时间）
容器采用了UTC时区，UTC应该是指（Coordinated Universal Time，标准时间）
 
统一两者的时区有下面几种方法
1）共享主机的localtime
创建容器的时候指定启动参数，挂载localtime文件到容器内，保证两者所采用的时区是一致的。
# docker run -ti -d --name my-nginx -v /etc/localtime:/etc/localtime:ro  docker.io/nginx  /bin/bash
 
2)复制主机的localtime
[root@slave-1 ~]# docker cp /etc/localtime 87986863838b:/etc/
 
然后再登陆容器，查看时间，发现已经跟宿主机时间同步了
[root@slave-1 ~]# docker exec -ti 87986863838b /bin/bash
root@87986863838b:/# date                                                                                                                    
Fri May 12 11:26:19 CST 2017
 
3）创建dockerfile文件的时候，自定义该镜像的时间格式及时区。在dockerfile文件里添加下面内容：
......
FROM tomcat
ENV CATALINA_HOME /usr/local/tomcat
.......
#设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
......
 
保存后，利用docker build命令生成镜像使用即可,使用dockerfile创建的镜像的容器改变了容器的时区，这样不仅保证了容器时间与宿主机时间一致（假如宿主机也是CST）,并且像上面使用tomcat作为父镜像的话，JVM的时区也是CST,这样tomcat的日志信息的时间也是和宿主机一致的，像上面那两种方式只是保证了宿主机时间与容器时间一致，JVM的时区并没有改变，tomcat日志的打印时间依旧是UTC。
```

 ![img](https://img2018.cnblogs.com/blog/1034798/201902/1034798-20190217123454006-10231913.png)