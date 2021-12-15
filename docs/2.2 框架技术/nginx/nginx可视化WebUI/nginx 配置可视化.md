### `nginxWebUI`

- `nginx`网页配置工具
- QQ技术交流群: `1106758598`
- 官网地址: `http://www.nginxwebui.cn`
- 源码地址：`https://git.chihiro.org.cn/chihiro/nginxWebUI`



### 功能说明

本项目可以使用WebUI配置nginx的各项功能, 包括http协议转发, tcp协议转发, 反向代理, 负载均衡, 			ssl证书自动申请、续签、配置等, 最终生成nginx.conf文件并覆盖nginx的默认配置文件, 完成nginx的最终功能配置.

本项目可管理多个nginx服务器集群, 随时一键切换到对应服务器上进行nginx配置, 			也可以一键将某台服务器配置同步到其他服务器, 方便集群管理

nginx本身功能复杂, 本项目并不能涵盖nginx所有功能, 只能配置常用功能, 			更高级的功能配置仍然需要在最终生成的nginx.conf中进行手动编写。

部署此项目后, 配置nginx再也不用上网各种搜索, 再也不用手动申请和配置ssl证书, 			只需要在本项目中进行增删改查就可方便的配置nginx。

### 技术说明

本项目是基于springBoot的web系统, 数据库使用sqlite, 因此服务器上不需要安装任何数据库

本系统通过Let's encrypt申请证书, 使用acme.sh脚本进行自动化申请和续签, 			开启续签的证书将在每天凌晨2点进行续签, 只有超过60天的证书才会进行续签. 只支持在linux下签发证书.

添加tcp/ip转发配置支持时, 			一些低版本的nginx可能需要重新编译，通过添加–with-stream参数指定安装stream模块才能使用, 但在ubuntu 			18.04下, 官方软件库中的nginx已经带有stream模块, 不需要重新编译. 本系统如果配置了tcp转发项的话, 			会自动引入ngx_stream_module.so的配置项, 如果没有开启则不引入, 最大限度优化ngnix配置文件.

### jar安装说明

以Ubuntu操作系统为例

注意：本项目需要在root用户下运行系统命令，极容易被黑客利用，请一定修改密码为复杂密码

1.安装java运行环境和nginx

ubuntu:

​				apt install openjdk-8-jdk
sudo apt install nginx 		

centos:

​				yum install java-1.8.0-openjdk
yum install nginx 		

2.下载最新版发行包jar

wget http://file.nginxwebui.cn/nginxWebUI-2.4.5.jar

有新版本只需要修改路径中的版本即可

3.启动程序

nohup java -jar -Xmx64m nginxWebUI-2.4.5.jar --server.port=8080 --project.home=/home/nginxWebUI/ > /dev/null &

参数说明(都是非必填)

-Xmx64m 最大分配内存数

--server.port 占用端口, 默认以8080端口启动

--project.home 项目配置文件目录，存放数据库文件，证书文件，日志等, 默认为/home/nginxWebUI/

--spring.database.type=mysql 使用其他数据库，不填为使用本地sqlite，选项包括mysql和postgresql

--spring.datasource.url=jdbc:mysql://ip:port/nginxwebui 数据库url

--spring.datasource.username=root 数据库用户

--spring.datasource.password=pass 数据库密码

注意命令最后加一个&号, 表示项目后台运行

### docker安装说明

本项目制作了docker镜像, 同时包含nginx和nginxWebUI在内, 一体化管理与运行nginx.

1.安装docker容器环境

ubuntu:

apt install docker.io

centos:

yum install docker

2.下载镜像:

docker pull cym1102/nginxwebui:latest

启动容器:

docker run -itd -v  /home/nginxWebUI:/home/nginxWebUI -e BOOT_OPTIONS="--server.port=8080"  --privileged=true --net=host cym1102/nginxwebui:latest /bin/bash 		

注意:

启动容器时请使用--net=host参数, 直接映射本机端口, 因为内部nginx可能使用任意一个端口, 			所以必须映射本机所有端口.

容器需要映射路径/home/nginxWebUI:/home/nginxWebUI, 此路径下存放项目所有数据文件, 			包括数据库, nginx配置文件, 日志, 证书等, 升级镜像时, 此目录可保证项目数据不丢失. 请注意备份.

-e BOOT_OPTIONS 参数可填充java启动参数, 可以靠此项参数修改端口号, "--server.port 占用端口", 不填默认以8080端口启动

日志默认存放在/home/nginxWebUI/log/nginxWebUI.log

### 编译说明

1.使用maven编译打包

mvn clean package

2.使用docker构建镜像

docker build -t nginxwebui:2.4.5 .

### 找回密码

如果忘记了登录密码，可按如下教程找回密码

\1. 安装sqlite3命令

apt install sqlite3

\2. 读取sqlite.db文件

sqlite3 /home/nginxWebUI/sqlite.db

\3. 查找admin表

select * from admin;

\4. 退出sqlite3

.quit