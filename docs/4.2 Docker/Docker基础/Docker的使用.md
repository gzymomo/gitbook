[TOC]

![](https://img2020.cnblogs.com/blog/1519601/202006/1519601-20200614140529730-20742309.png)

# 一、Docker安装使用

## Docker设置镜像源
`vi /etc/dockerdaemon.json`
```yml
{
 "registry-mirrors":["https://6kx4zyno.mirror.aliyuncs.com"]
}
```


# Docker 将镜像推送至Docker Hub仓库中
```shell
docker  tag  镜像id       要推入仓库的用户名/要推入的仓库名:新定义的tag 
docker tag b6a497c707d0 aishiyushijiepingxing/jdk:1.8

docker push      要推入仓库的用户名/要推入的仓库名:镜像标签
docker push aishiyushijiepingxing/jdk:1.8
```
## Docker查看磁盘大小
```shell
docker system df  # （类似于Linux上的df命令，用于查看Docker的磁盘使用情况。）
```
## 通过docker方式启动注册中心
```shell
docker run -p 3306:3306 --name service-register -v /usr/local/mysqldata/register-service/conf:/etc/mysql/conf.d -v /usr/local/mysqldata/register-service/logs:/logs -v /usr/local/mysqldata/register-service/data:/var/lib/mysql -v /etc/localtime:/etc/localtime -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
## 查看容器内的日志

```shell
docker logs -f jenkins
```
## 启动Jenkins
```shell
docker run -d -p 8080:8080 -v ~/jenkins:/var/jenkins_home --name jenkins jenkins
# 给jenkins用户增加权限：chown -R 1000:1000 ~/jenkins
```
## 启动Service服务
```shell
docker run -e "--spring.profiles.active=dev" -d --net=host -v /logs/yjgl-dev:/logs gly/service-yjgl:0.0.1
```
## 查看容器端口
```shell
docker port containername<containerid>
```
## 查看容器日志
```shell
docker logs containername<containerid>
```
## 查看指定容器内部运行进程
```shell
docker top containername<containerid>
```
## 为镜像添加新的标签
```shell
docker tag
```
## 查询容器ip
```shell
docker inspect containername<containerid>
```
## 提交容器副本
```shell
docker commit [-m -a] container-id image-name
-m：提交的描述信息
-a：指定镜像作者
container-id：容器ID
image-name：你要命名的镜像名
```
## 可视化工具
```shell
docker run -p 9000:9000 --name prtainer -v /var/run/docker.sock:/var/run/docker.sock -d portainer/portainer
```
## 保存镜像到本地：
```shell
docker save -o jdk.tar imageId
```
## 打包镜像导入目标服务器，并使用docker导入:
```shell
docker load -i images.tar
```
## Docker使用gzip压缩导出/导入镜像
```shell
docker save <myimage>:<tag> | gzip > <myimage>_<tag>.tar.gz
gunzip -c <myimage>_<tag>.tar.gz | docker load
```
## Docker进入容器内部
```shell
sudo docker exec -it 7b04cf9f4ce8 /bin/sh
docker exec -it imageId /bin/bash
```
## Docker创建镜像
```shell
docker build -t jdk:8 .
```
## Docker安装SonarQube
```shell
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=123456 --name postgres postgres:10
docker run -d -p 9900:9000 -e "SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.0.50:5432/sonar" -e "SONARQUBE_JDBC_USERNAME=postgres" -e "SONARQUBE_JDBC_PASSWORD=123456" --name sonarqube sonarqube
```
## Docker启动Nginx
```shell
docker run -d -p 80:80 --name nginx -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/conf:/etc/nginx -v /home/nginx/logs:/var/log/nginx nginx
```
## Docker查看容器使用资源情况--docker stats
docker stats (不带任何参数选项)
[CONTAINER]：以短格式显示容器的 ID。
[CPU %]：CPU 的使用情况。
[MEM USAGE / LIMIT]：当前使用的内存和最大可以使用的内存。
[MEM %]：以百分比的形式显示内存使用情况。
[NET I/O]：网络 I/O 数据。
[BLOCK I/O]：磁盘 I/O 数据。 
[PIDS]：PID 号。
```shell
docker stats --no-stream  (只返回当前的状态)
docker stats   --no-stream   容器ID/Name (只输出指定的容器)
```
## Docker安装Grafana
```shell
docker search grafana
```
对'/var/lib/grafana/plugins'没有权限创建目录，那么就赋予权限：
```shell
chmod 777 /data/grafana
docker run -d -p 3000:3000 --name=grafana -v /data/grafana:/var/lib/grafana grafana/grafana
```
## Docker启动Promethus
```shell
docker run -d -p 9090:9090 --name=prometheus -v /root/software/prometheus/prometheus-config.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

## Docker安装Mysql
```shell
docker run -p 3307:3306 --name mysql -v /root/logs/mysql/conf:/etc/mysql/conf.d -v /root/logs/mysql/logs:/logs -v /root/logs/mysql/data:/var/lib/mysql -v /etc/localtime:/etc/localtime -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 --lower_case_table_names=1
```

## Docker安装jenkins
```bash
docker run -d -p 9001:8080 \
-v /usr/local/jenkins/workspace/:/root/.jenkins/workspace \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/git:/usr/bin/git \
-v /opt/jdk1.8:/usr/local/jdk1.8 \
-v /opt/maven:/usr/local/maven3 --name jenkins jenkins
```