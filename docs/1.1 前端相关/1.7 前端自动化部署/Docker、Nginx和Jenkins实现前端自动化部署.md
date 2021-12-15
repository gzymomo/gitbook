- [使用Docker、Nginx和Jenkins实现前端自动化部署](https://blog.51cto.com/u_14928332/2878403)



前期准备

- 基于CentOS 7系统云服务器一台。
- 基于Vue-CLI的项目部署在GitLab之上。

# 部署目标

搭建Docker+Nginx+Jenkins环境，用于实现前端自动化部署的流程。具体的实现效果为开发人员在本地开发，push提交代码到指定分支，自动触发Jenkins进行持续集成和自动化部署。可以设置在部署完成后通过邮件通知，部署的成功与否，成功后会将打包后的文件上传到服务器，通过nginx反向代理展现页面，失败则会打印相关的错误日志。

> 友情提示：尽量选择阿里云或者腾讯云服务器，其他服务器部署时可能会出现Jenkins无法正常启动！

Dcoker环境的搭建连接云服务器

可以选择阿里云或者腾讯云提供的在线终端（有时会卡），但是推荐使用本地电脑进行连接。在终端输入连接命令：

```
ssh root@你的服务器公网地址
```




 之后输入云服务器密码，命令显示结果如下：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/7b6e7cf408d8aec96fc85080c0f7133b.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 Docker有两个分支版本：Docker CE和Docker EE，即社区版和企业版。本教程基于CentOS 7安装Docker CE。

安装Docker环境

1、安装Docker的依赖库。

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```




 2、添加Docker CE的软件源信息。

```
sudo yum-config-manager --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo
```




 3、安装Docker CE。

```
sudo yum install docker-ce
```




 4、启动Docker服务。

```
sudo systemctl enable docker // 设置开机自启
sudo systemctl start docker //  启动docker
```



Docker安装Docker Compose

Docker Compose是用于定义和运行多容器Docker应用程序的工具。通过Compose，您可以使用YML文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从YML文件配置中创建并启动所有服务。下载docker-compose：

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```




 安装完成后提升权限：

```
sudo chmod +x /usr/local/bin/docker-compose
```




 输入docker-compose -v显示如下页面：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/b020b4cbbb7c2f0f274e0249cf7af73e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

Docker安装Nginx和Jenkins服务安装Nginx和Jenkins

Docker镜像拉取Nginx和Jenkins环境命令如下：

```
docker pull nginx
docker pull jenkins/jenkins:lts
```




 安装完成后执行docker images可以清晰的看到当前Docker下存在的镜像。

```
docker images
```



![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/a3407775732ee73b66797e1df1839765.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

Nginx和Jenkins目录编写

为了便于管理，在Docker下我们将Nginx和Jenkins聚集到一个文件目录之中。目录结构如下：

```
+ compose
- docker-compose.yml  // docker-compose执行文件
+ nginx 
+ conf.d
- nginx.conf        // Nginx配置
+ jenkins
- jenkins_home       // Jenkins挂载卷
+ webserver 
-static              //存放前端打包后的dist文件
```




 Web  server目录属于后期生成暂不讨论，需要手动创建的是Compose，Nginx和Jenkins目录及其下属文件，其中最主要的是docker-compose.yml文件和nginx.conf文件的配置。以上文件夹建议放在根目录下面，可以放在home文件夹之下也可以单独创建一个新的文件夹。

docker-compose.yml文件配置

```
version: '3'
services:                                      # 集合
docker_jenkins:
user: root                                 # 为了避免一些权限问题 在这我使用了root
restart: always                            # 重启方式
image: jenkins/jenkins:lts                 # 指定服务所使用的镜像 在这里我选择了 LTS (长期支持)
container_name: jenkins                    # 容器名称
ports:                                     # 对外暴露的端口定义
  - 8080:8080
  - 50000:50000
volumes:                                   # 卷挂载路径
  - /home/jenkins/jenkins_home/:/var/jenkins_home  # 这是我们一开始创建的目录挂载到容器内的jenkins_home目录  - /var/run/docker.sock:/var/run/docker.sock
  - /usr/bin/docker:/usr/bin/docker                # 这是为了我们可以在容器内使用docker命令
  - /usr/local/bin/docker-compose:/usr/local/bin/docker-compose
docker_nginx:
restart: always
image: nginx
container_name: nginx
ports:
  - 8090:80
  - 80:80
  - 433:433
volumes:
  - /home/nginx/conf.d/:/etc/nginx/conf.d  - /home/webserver/static/jenkins/dist/dist:/usr/share/nginx/html
```



nginx.conf文件配置

```
server{
listen  80;
root /usr/share/nginx/html;index index.html index.htm;
}
```




 上述两个文件配置完成之后，需要进入/home/compose目录下面输入以下命令，进行环境的启动：

```
docker-compose up -d
```




 输入docker ps -a 查看容器的情况：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/48bd1af5062c3ea96ae83005ab81a6b2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 状态显示up，后面的端口号显示如上为正常状态。在浏览器输入你云服务器的公网IP加上8080的端口号就可以显示如下页面：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/f0cea2f1672dbfeef8e42417386cc1e5.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 注意点：

- 在此步骤之前，切记一定要开放云服务器的80端口安全组（可以参考提供的一键开通功能），但是除此之外建议手动添加8080端口的安全组。
- 80端口：是为HTTP（HyperText Transport Protocol）即超文本传输协议开放的端口。
- 8080端口：是被用于WWW代理服务的，可以实现网页浏览。


 上图所需要的密码在docker-compose.yml中的volumes中的/home/jenkins/jenkins_home/secrets/initAdminPassword中。可以通过以下命令获得：

```
cat /home/jenkins/jenkins_home/secrets/initialAdminPassword
```



安装Jenkins插件

进入页面之后，选择推荐安装。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/ae87bdf8a2872a296742563bc6474cb9.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 安装完成之后，选择左侧Manage Jenkins选项。如下图所示：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/bf130040ec1def94469ff27d697d37bd.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 Jenkins中Manage Plugins搜索以下插件GitLab、Publish Over SSH、Nodejs并安装。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/27195a602a15c01de1503bb98e9b4c58.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 安装完成后配置Nodejs环境和SSH参数 在首页选择global tool Configuration>NodeJS选择自动安装和对应的Nodejs版本号，选择成功后点击保存。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/6f0c20f1fa5f75954481b6a1bad9df9f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/27e702084b77cb87fd6c8e4addf85537.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 配置SSH信息，Manage Jenkins>configure System填写服务器的相关信息：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/89098bddfe04705a3237645f7fff0b30.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/c65da578c84f6d2211d62049763bf439.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

关联Jenkins和GitLab生成密钥

在根目录下执行一下命令：

```
ssh-keygen -t rsa
```




 一般默认两次回车，如下图所示：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/33c790baee014dcdd49f46e85c7c4f71.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 使用cd ~/.ssh查看生成的文件。将生成的密钥id_rsa复制粘贴到Jenkins中的凭证。如图所示：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/d1978d66f317613a0881ca957d4c9506.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/197a3b6117e5b6f1d069d2661973e608.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/3f9dee77bfcb38eb24742f5eea816bdb.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 登陆GitLab，在GitLab中配置id_rsa.pub公钥：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/7da872b17e22ebb219cb026e815aad2f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

新建项目

准备完毕后，开始新建一个任务，选择新建item>freestyle project构建一个自由风格的项目。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/26bd2b8f8163ca0836eaa46f1cae4fae.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

源码管理

新建完成后，在源码管理中配置Git信息，credentials选择我们刚刚添加的凭证。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/75522bde12d9f1e8e35b19a8ade5c6d4.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

构建触发器

在构建触发器中选择我们触发构建的时机，你可以选择队友的钩子，比如push代码的时候，Merge Request的时候：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/d46aceac6a06b39452f429d0be64e442.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 点击高级选项找到secret token>Generate生成一个token值：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/06473e2e2e0da59b84deacd0fa5dc5fc.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 配置成功后，还需要到GitLab中增加对应的钩子。记下上图的webhookURL（红线框出）和secret token值，到GitLab中进行配置。

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/6ddb0cd7d5b3ced3c7d4a8582c520b6e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

构建环境及构建配置

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/28fdb7d5802041cbe903c752e9be5d85.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/c6b60c3783336a7599ba097812f9c2ec.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 


 完成上述配置后，Jenkins就和GitLab关联起来，在本地push文件时，就会自动构建，访问云服务器的公网IP地址就可以访问修改完成后的项目，同样也可以在Jenkins上手动构建，如图所示：

![用Docker、Nginx和Jenkins实现前端自动化部署](https://s4.51cto.com/images/blog/202106/07/40789ddfa12a47f6447564653ca084b7.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

 