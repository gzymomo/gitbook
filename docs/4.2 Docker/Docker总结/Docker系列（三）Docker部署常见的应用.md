

Docker的使用，其实没有多么复杂，当你掌握了Docker的基础知识后，了解了Docker的相关命令的使用，那么关于Docker的使用就可以直接上手了，在此篇文章中介绍一些常见服务的部署。



Docker 启动容器时，若在本地没有找到相应的镜像，先回去你配置的仓库寻找对应的镜像，如果在你配置的仓库也没有，那么会去Docker Hub上寻找镜像。许多时候，因为我们已经明确知道我们所要的镜像是哪个，所以可以省去docker pull的命令，通过docker run自动去拉取相应的镜像。



# 一、Docker部署MySQL

```bash
docker run -p 3306:3306 --name mysql -v /var/project/mysql/conf:/etc/mysql/conf.d -v /var/project/mysql/data:/var/lib/mysql -v /etc/localtime:/etc/localtime -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

参数解析：

- -p：端口映射
- --name：容器名称
- -v：将容器内部的目录挂载至宿主机
- -e：设定初始化root账户的密码
- -d：后台启动



在服务器上执行如上命令，即可成功部署一个5.7版本的MySQL服务，通过其他工具，如Navicat，DBeaver，heidiSql，DataGrip等数据库连接工具连接至该数据库。



也可直接进入到该容器内部，连接数据库，示例操作如下：

1. 首先进入容器：`docker exec -it mysql /bin/sh`

![image-20210202151230695](http://lovebetterworld.com/image-20210202151230695.png)

2. 连接数据库`mysql -u root -p`，然后回车，根据提示输入密码



若需要退出，则输入`exit`即可。



# 二、Docker部署Redis

```bash
docker run -p 6379:6379 --name redis -v /var/project/redis/redis.conf:/etc/redis/redis.conf -v /var/project/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes
```

- -p：端口映射
- --name：容器名称
- -v：将容器内部的目录挂载至宿主机
- -d：后台启动
- --appendonly yes：Redis配置数据持久化，开启AOF模式



在服务器上执行如上命令，即可成功部署Redis服务，通过其他工具，如RedisDesktopManager工具连接至该数据库，即可成功访问Redis。



# 三、Docker部署nginx

```bash
docker run -d -p 80:80 --name nginx -v /var/project/nginx/html:/usr/share/nginx/html -v /var/project/nginx/conf:/etc/nginx -v /var/project/nginx/logs:/var/log/nginx nginx
```

- -p：端口映射
- --name：容器名称
- -v：将容器内部的目录挂载至宿主机
- -d：后台启动



在服务器上执行如上命令，即可成功部署nginx服务。但是，如果直接这样启动容器后，会报错提示`no such file or direcotory`，那是因为我们宿主机/var/project/nginx/conf目录下，没有nginx的配置文件，我们需要手动添加一个nginx的配置文件。

```bash
vi nginx.conf
```

```yaml
#Nginx的worker进程运行用户以及用户组
#user  nobody;

#Nginx开启的进程数
worker_processes  1;

#定义全局错误日志定义类型，[debug|info|notice|warn|crit]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#指定进程ID存储文件位置
#pid        logs/nginx.pid;

#事件配置
events {

    #use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];
    #epoll模型是Linux内核中的高性能网络I/O模型，如果在mac上面，就用kqueue模型。
    use kqueue;

    #每个进程可以处理的最大连接数，理论上每台nginx服务器的最大连接数为worker_processes*worker_connections。理论值：worker_rlimit_nofile/worker_processes
    worker_connections  1024;
}

#http参数
http {
    #文件扩展名与文件类型映射表
    include       mime.types;
    #默认文件类型
    default_type  application/octet-stream;

    #日志相关定义
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #连接日志的路径，指定的日志格式放在最后。
    #access_log  logs/access.log  main;

    #开启高效传输模式
    sendfile        on;

    #防止网络阻塞
    #tcp_nopush     on;

    #客户端连接超时时间，单位是秒
    keepalive_timeout  65;

    #开启gzip压缩输出
    #gzip  on;

    #虚拟主机基本设置
    server {
        #监听的端口号
        listen       80;
        #访问域名
        server_name  localhost;

        #编码格式，如果网页格式与当前配置的不同的话将会被自动转码
        #charset koi8-r;

        #虚拟主机访问日志定义
        #access_log  logs/host.access.log  main;

        #对URL进行匹配
        location / {
            #访问路径，可相对也可绝对路径
            root   html;
            #首页文件，匹配顺序按照配置顺序匹配
            index  index.html index.htm;
        }

        #错误信息返回页面
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        #禁止访问.ht页面
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    #HTTPS虚拟主机定义
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```

在添加一份mime.types文件，然后重启nginx容器即可。

```bash
docker start nginx
```



# 四、Docker部署代码质量监测工具SonarQube

```bash
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=123456 --name postgres postgres:10

docker run -d -p 9900:9000 -e "SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.0.50:5432/sonar" -e "SONARQUBE_JDBC_USERNAME=postgres" -e "SONARQUBE_JDBC_PASSWORD=123456" --name sonarqube sonarqube
```



# 五、Docker部署Minio文件服务器

```bash
docker run -d -p 9000:9000 --name minio -e MINIO_ACCESS_KEY=admin -e MINIO_SECRET_KEY=123456 -v /var/project/minio/data:/data -v /var/project/minio/config:/root/.minio minio/minio server /data
```



# 六、Docker部署halo博客系统

```bash
docker run -d --name halo -p 7081:8090  -v /var/project/halo:/root/.halo ruibaby/halo
```





# 总结

了解了上述Docker部署服务的命令后，你会发现使用Docker安装一个软件太简单了，无非就是执行好一条docker run命令而已，而公共的参数也就是常见的那些，如-d，-p，-v，-e等等，对于其他更多的软件应用，都是如上道理。

Docker部署应用一般是比较容易的，重要的是这些应用的使用，调优等，也可实现将一些常用的服务自己构建镜像，然后上传至自己的私服或者Docker Hub上。



# Docker思维导图总结

思维导图下载链接：
[Docker思维导图下载](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/Docker)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218164558449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70#pic_center)



