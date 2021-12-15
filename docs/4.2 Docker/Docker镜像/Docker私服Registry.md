[TOC]

# 1、Docker推送本地仓库至阿里云私服
 - Docker登录一下私服账号：(阿里私服)
`docker login --username=爱是与世界平行l registry.cn-qingdao.aliyuncs.com`

```shell
docker tag [imageId] registry.cn-qingdao.aliyuncs.com/loveworld/jdk:1.8
docker push registry.cn-qingdao.aliyuncs.com/loveworld/jdk:1.8
```

# 2、Docker私服Jdk1.8的使用
 1. 首先从docker私服pull下来jdk:1.8
`docker pull registry.cn-qingdao.aliyuncs.com/loveworld/jdk:1.8`
 2. 然后将pull下来的镜像改变其tag
`docker tag registry.cn-qingdao.aliyuncs.com/loveworld/jdk:1.8 jdk:1.8`
 3. 删除原镜像
`docker rmi registry.cn-qingdao.aliyuncs.com/loveworld/jdk:1.8`

# 3、Docker SpringBoot启动脚本
```bash
#!/bin/sh
pid=$(docker ps -a|grep "yt_traffic_manage" | awk '{print $1}')
if [ -n "$pid" ]; then
	docker stop $pid
    docker rm $pid
fi

ima=$(docker images | grep "yt_traffic_manage" | awk  '{if($3!="")  print  $3}')
if [ -n "$ima" ]; then 
    docker rmi $ima 
fi

docker pull registry.cn-qingdao.aliyuncs.com/loveworld/java/yt_traffic_manage:0.1
docker run -e "--spring.profiles.active=prod" -d --name yt_traffic_manage --net=host -v /root/logs/yt_traffic_manage:/opt/logs registry.cn-qingdao.aliyuncs.com/loveworld/java/yt_traffic_manage:0.1

tail -f /root/logs/yt_traffic_manage/jgp.log
```