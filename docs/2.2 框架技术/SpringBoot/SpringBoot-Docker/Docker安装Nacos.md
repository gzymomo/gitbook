[TOC]

# 1、 安装镜像
`$ docker pull nacos/nacos-server`

# 2、启动
配置文件/opt/nacos/init.d/custom.properties内容如下：
` management.endpoints.web.exposure.include=*   `
使用standalone模式并开放8848端口，并映射配置文件和日志目录，数据库默认使用 Derby:
```bash
$ docker run -d -p 8848:8848 -e MODE=standalone -v /opt/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties -v /opt/nacos/logs:/home/nacos/logs --restart always --name nacos nacos/nacos-server
```
# 3、测试访问
直接访问 http://127.0.0.1:8848/nacos， 使用账号：nacos，密码：nacos 直接登录。nacos是默认账号和密码，登录成功后可以修改密码。

# 4、docker-compose启动
docker-compose文件 standalone-derby.yaml
```yml
version: "2"
services:
  nacos:
    image: nacos/nacos-server:latest
    container_name: nacos
    environment:
    - MODE=standalone
    volumes:
    - /opt/nacos/logs:/home/nacos/logs
    -  /opt/nacos/init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
    - "8848:8848"
```
## 4.1 启动
`docker-compose -f standalone-derby.yaml up`

## 4.2 关闭
`docker-compose -f standalone-derby.yaml stop `

## 4.3 移除
` docker-compose -f standalone-derby.yaml rm `

## 4.4 关闭且移除
` docker-compose -f standalone-derby.yaml down `
