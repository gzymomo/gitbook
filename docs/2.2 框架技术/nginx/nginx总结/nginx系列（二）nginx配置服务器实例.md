nginx可以直接作为静态资源文件服务器进行使用，通过配置nginx的相关配置文件，可以直接将nginx作为一个稳定快速的静态服务器来存储一些小资源，图片等文件。



# 一、配置静态资源访问

## 1.1 nginx.conf 

在nginx.conf的http节点中添加配置，参考下方格式：

```
server {
        listen       8000;
        listen       somename:8080;
        server_name  somename  alias  another.alias;

        location / {
            root   html;
            index  index.html index.htm;
        }
}
```

<font color='red'>解读server节点各参数含义</font>

- listen：代表nginx要监听的端口

- server_name:代表nginx要监听的域名

- location ：nginx拦截路径的匹配规则

- location块：location块里面表示已匹配请求需要进行的操作

## 1.2 示例

### 1.2.1 准备要访问的静态文件

两个文件夹：folder1 folder2 folder3各放两个文件一个index.html 

 ![img](https://img2020.cnblogs.com/blog/1238609/202004/1238609-20200430173513232-217657252.png)

### 1.2.2 创建一个server

```
server {
	listen       9999;
    server_name  localhost;

    location /xixi {
    	alias   /Users/qingshan/folder1;
        index  index.html;
    }

	location /haha {
    	alias   /Users/qingshan/folder2;
        index  index.html;
    }

	location /folder3 {
    	root   /Users/qingshan;
       	index  index.html;
    }
}
```

###  1.2.3 重启nginx后，即可看到如下内容

 ![img](https://img2020.cnblogs.com/blog/1238609/202004/1238609-20200430162047114-1726912757.png)

###  2.3 root与alias的区别

　　重点是理解alias与root的区别，root与alias主要区别在于nginx如何解释location后面的uri，这使两者分别以不同的方式将请求映射到服务器文件上。

- alias（别名）是一个目录别名。

例子：

```
　location /123/abc/ {
　　root /ABC;
  }
# 当请求http://qingshan.com/123/abc/logo.png时，会返回 /ABC/123/abc/logo.png文件，即用/ABC 加上 /123/abc。
```

- root（根目录）是最上层目录的定义。

例子：

```
location /123/abc/ {
	alias /ABC;
    }
# 当请求http://qingshan.com/123/abc/logo.png时，会返回 /ABC/logo.png文件，即用/ABC替换 /123/abc。
```



## 1.3 nginx配置文件解析

```bash
# 指定运行nginx的用户名
#user  nobody;
# 工作线程数，通常同cpu逻辑核心数一致
worker_processes  1;

# 错误日志路径 最小级别 [ debug | info | notice | warn | error | crit ]
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 指定进程的pid记录文件，记录当前运行的nginx的pid
#pid        logs/nginx.pid;

# 网络连接模块
events {
    # 一个工作线程支持的最大并发连接数
    worker_connections  1024;

    # keepalive超时时间，单位：秒
    keepalive_timeout 60;
}

# 设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    # 设定支持的 mime 类型
    include       mime.types;
    # 默认 mime 类型
    default_type  application/octet-stream;

    # 设定日志格式，格式名为main
    ## $remote_addr：客户端的ip地址（若使用代理服务器，则是代理服务器的ip）
    ## $remote_user：客户端的用户名（一般为“-”）
    ## $time_local：访问时间和时区
    ## $request：请求的url和请求方法
    ## $status：响应HTTP状态码
    ## $body_bytes_sent：响应body中的字节数
    ## $http_referer：客户端是从哪个url来请求的
    ## $http_user_agent：客户端用户使用的代理（一般为浏览器）
    ## $http_x_forwarded_for：客户端的ip地址（通过代理服务器记录客户端的ip地址）
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # 访问日志文件路径及日志格式
    #access_log  logs/access.log  main;

    # 指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为 on,
    # 如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime
    sendfile        on;
    #tcp_nopush     on;

    # keepalive 超时时长，单位：秒
    #keepalive_timeout  0;
    keepalive_timeout  65;

    # 打开 gzip 
    #gzip  on;

    # 以上为 nginx 的全局设置，应用于所有 Web 应用
    # 一个Web应用对应一个 server，内部配置仅针对该应用，优先级比全局的高
    server {
        // 端口号
        listen       80;
        // 域名，比如 www.test.com
        server_name  localhost;

        # 编码格式
        #charset koi8-r;

        # 访问日志文件路径
        #access_log  logs/host.access.log  main;

        # 一般路由导航到：
        location / {
            # 根目录为html
            root   html;
            # 默认页为 index.html，如果没有则是 index.htm
            index  index.html index.htm;
        }

        # 404时的展示页面
        #error_page  404              /404.html;

        # 50X时的展示页面
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # 禁止访问 .htxxx 的文件
        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    ############## HTTPS demo beign ##############
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

         # ssl 证书文件位置
    #    ssl_certificate      cert.pem;
         # ssl 证书key的位置
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;
         # 数字签名 MD5
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    ############### HTTPS demo end ###############

    ############ 反向代理 demo begin #############
    # 设定实际的服务器列表（权重默认都是1）
    #upstream myserver{
    #   server 192.168.0.1:8089 weight=7;
    #   server 192.168.0.2:8089 weight=3;
    #}

    #server {
    #    listen       80;
    #    server_name localhost;

         #反向代理的路径（和upstream绑定），location 后面设置映射的路径
    #    location / {
    #        proxy_pass http://myserver;
    #    }
    #}
    ############# 反向代理 demo end ##############

}
```





# 二、静态资源服务器

语雀：渡渡鸟

1. [压缩与解压](https://www.yuque.com/duduniao/nginx/qlupgp)
2. [浏览器缓存](https://www.yuque.com/duduniao/nginx/iuvwyf)
3. [CSRF](https://www.yuque.com/duduniao/nginx/hqyqgg)
4. [静态资源防盗链](https://www.yuque.com/duduniao/nginx/yg9fco)
5. [访问流控](https://www.yuque.com/duduniao/nginx/mnkins)
6. [访问控制](https://www.yuque.com/duduniao/nginx/kvtpgt)
7. [登陆认证](https://www.yuque.com/duduniao/nginx/xmlkno)
8. [VirtualHosts](https://www.yuque.com/duduniao/nginx/gvkbu0)



## 2.1 限流

- limit_conn_zone(限制连接数，针对客户端，即单一ip限流)
- limit_req_zone(限制请求数，针对客户端，即单一ip限流)
- ngx_http_unpstream_module（推荐，针对后台，如：有两台服务器，服务器A最大可并发处理10W条请求，服务器B最大可并发处理5W条请求，这样当12W请求按照负载均衡规则应当被分配给服务器A时，nginx为了防止A挂掉，所以将另外的2W分配给B）

### 2.1.1 limit_conn_zone

```
http {
    # 将请求客户端的IP（$binary_remote_addr）存放到perip区域，区域大小为10M，一个IP占用32Byte（32位系统）或64Byte（64位系统）左右
    # perip是区域的名字，可自定义
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    server {
        # 每个IP最大并发1条连接
        # 该语句还可直接放置到http模块下，这样下属的server都应用该配置
        # 该语句还可放置到server中的location模块中，这样仅指定的location应用该配置
        limit_conn perip 1;
        # 每个连接限速300 k/s
        limit_rate 300k; 
    }
}
```

### 2.1.2 limit_req_zone

```
http {
    # 将请求客户端的IP存放到perip区域，区域大小为10M，并限制同一IP地址的请求每秒钟只处理一次
    limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;
    server {
        # 当有大量请求爆发时，可以缓存2条请求
        # 设置了nodelay，缓存队列的请求会立即处理，若请求数 > rate+burst 时，立即返回503；如果没设置，则会按照rate排队等待处理
        # 该语句还可直接放置到http模块下，这样下属的server都应用该配置
        # 该语句还可放置到server中的location模块中，这样仅指定的location应用该配置
        limit_req zone=perip burst=2 nodelay;
    }
}
```

### 2.1.3 limit_req_zone

```
http {
    # 将请求客户端的IP存放到perip区域，区域大小为10M，并限制同一IP地址的请求每秒钟只处理一次
    limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;
    server {
        # 当有大量请求爆发时，可以缓存2条请求
        # 设置了nodelay，缓存队列的请求会立即处理，若请求数 > rate+burst 时，立即返回503；如果没设置，则会按照rate排队等待处理
        # 该语句还可直接放置到http模块下，这样下属的server都应用该配置
        # 该语句还可放置到server中的location模块中，这样仅指定的location应用该配置
        limit_req zone=perip burst=2 nodelay;
    }
}
- 测试：ab -c 1 -n 3 http://localhost/
  - 3个请求全部成功，因为正在处理的请求数1加上缓存数2，没超过限制
```

```
Server Software:        nginx/1.18.0
Server Hostname:        192.168.159.128
Server Port:            80

Document Path:          /
Document Length:        612 bytes

Concurrency Level:      1
Time taken for tests:   0.001 seconds
Complete requests:      3
Failed requests:        0
Total transferred:      2535 bytes
HTML transferred:       1836 bytes
Requests per second:    2439.02 [#/sec] (mean)
Time per request:       0.410 [ms] (mean)
Time per request:       0.410 [ms] (mean, across all concurrent requests)
Transfer rate:          2012.67 [Kbytes/sec] received
```


- 测试：ab -c 3 -n 4 http://localhost/
  - 3个请求成功，1个请求失败，因为正在处理的请求数1加上缓存数2，另外1条请求失败

```
Server Software:        nginx/1.18.0
Server Hostname:        192.168.159.128
Server Port:            80

Document Path:          /
Document Length:        612 bytes

Concurrency Level:      1
Time taken for tests:   0.002 seconds
Complete requests:      4
Failed requests:        1
   (Connect: 0, Receive: 0, Length: 1, Exceptions: 0)
Non-2xx responses:      1
Total transferred:      3223 bytes
HTML transferred:       2330 bytes
Requests per second:    2504.70 [#/sec] (mean)
Time per request:       0.399 [ms] (mean)
Time per request:       0.399 [ms] (mean, across all concurrent requests)
Transfer rate:          1970.86 [Kbytes/sec] received
```

### 2.1.4 ngx_http_upstream_module

```
upstream MyName {
    server 192.168.0.1:8080 weight=1 max_conns=10;
    server 192.168.0.2:8080 weight=1 max_conns=10;
}
```



## 2.2 压力测试工具——Ab

### 2.2.1 安装

`yum install httpd-tools -y`

### 2.2.3  测试

// 10个用户，向 http://www.test.com/ 并发发送1000条请求（总请求数=1000）
`ab -c 10 -n 1000 http://www.test.com/`

### 2.2.3 返回值

- Document Path：测试的页面路径
- Document Length：页面大小（byte）
- Concurrency Level：并发数量，即并发用户数
- Time taken for tests：测试耗费时长
- Complete requests：成功的请求数量
- Failed requests：请求失败的数量
- Write errors：错误数量
- Requests per second：每秒钟的请求数量、吞吐率
- Timer per request：每次请求所需时间、响应时间



## 2.3 GZIP

- 作用：启用gzip后，服务器将响应报文进行压缩，有效地节约了带宽，提高了响应至客户端的速度。当然，压缩会消耗nginx所在电脑的cpu
- 配置范围：http、server、location

```bash
http {
    # 启用gzip
    gzip on;
    # 允许压缩的最小字节数（即如果response header中的content-length小于该值，就不压缩）
    gzip_min_length 2k;
    # 按照原数据大小以16k为单位的4倍申请内存用作压缩缓存
    gzip_buffers 4 16k;
    # 压缩级别，级别越大，压缩率越高，占用CPU时间更长
    gzip_comp_level 5;
    # 需要被压缩的响应类型，默认值是text/html
    gzip_types text/plain application/x-javascript text/css application/xml;
    # 配置最低版本的http压缩协议（即1.0时，1.0和1.1都会启用压缩；1.1时，仅1.1时才会启用压缩）
    gzip_http_version 1.0;
    # IE6及以下禁用压缩
    gzip_disable "MSIE [1-6]\.";
}
```



# 三、代理服务器

## 3.1 代理配置

```bash
#运行用户
#user somebody;

#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志
error_log  D:/Tools/nginx-1.10.1/logs/error.log;
error_log  D:/Tools/nginx-1.10.1/logs/notice.log  notice;
error_log  D:/Tools/nginx-1.10.1/logs/info.log  info;

#PID文件，记录当前启动的nginx的进程ID
pid        D:/Tools/nginx-1.10.1/logs/nginx.pid;

#工作模式及连接数上限
events {
    worker_connections 1024;    #单个后台worker process进程的最大并发链接数
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型(邮件支持类型),类型由mime.types文件定义
    include       D:/Tools/nginx-1.10.1/conf/mime.types;
    default_type  application/octet-stream;

    #设定日志
    log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log    D:/Tools/nginx-1.10.1/logs/access.log main;
    rewrite_log     on;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  120;
    tcp_nodelay        on;

    #gzip压缩开关
    #gzip  on;

    #设定实际的服务器列表
    upstream zp_server1{
        server 127.0.0.1:8089;
    }

    #HTTP服务器
    server {
        #监听80端口，80端口是知名端口号，用于HTTP协议
        listen       80;

        #定义使用www.xx.com访问
        server_name  www.helloworld.com;

        #首页
        index index.html

        #指向webapp的目录
        root D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp;

        #编码格式
        charset utf-8;

        #代理配置参数
        proxy_connect_timeout 180;
        proxy_send_timeout 180;
        proxy_read_timeout 180;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarder-For $remote_addr;

        #反向代理的路径（和upstream绑定），location 后面设置映射的路径
        location / {
            proxy_pass http://zp_server1;
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp\views;
            #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status           on;
            access_log            on;
            auth_basic            "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }

        #禁止访问 .htxxx 文件
        location ~ /\.ht {
            deny all;
        }

        #错误处理页面（可选择性配置）
        #error_page   404              /404.html;
        #error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
    }
}
```

1. 启动 webapp，注意启动绑定的端口要和 nginx 中的 upstream 设置的端口保持一致。
2. 更改 host：在 C:\Windows\System32\drivers\etc 目录下的 host 文件中添加一条 DNS 记录
   `  127.0.0.1 www.helloworld.com  `
3. 启动前文中 startup.bat 的命令
4. 在浏览器中访问 www.helloworld.com，不出意外，已经可以访问了。



## 3.2 网站有多个 webapp 的配置

当一个网站功能越来越丰富时，往往需要将一些功能相对独立的模块剥离出来，独立维护。这样的话，通常，会有多个 webapp。

举个例子：假如 www.helloworld.com 站点有好几个 webapp，finance（金融）、product（产品）、admin（用户中心）。访问这些应用的方式通过上下文(context)来进行区分:

> www.helloworld.com/finance/
> www.helloworld.com/product/
> www.helloworld.com/admin/

我们知道，http 的默认端口号是 80，如果在一台服务器上同时启动这 3 个 webapp 应用，都用 80 端口，肯定是不成的。所以，这三个应用需要分别绑定不同的端口号。

那么，问题来了，用户在实际访问 www.helloworld.com 站点时，访问不同 webapp，总不会还带着对应的端口号去访问吧。所以，你再次需要用到反向代理来做处理。

```yml
http {
    #此处省略一些基本配置

    upstream product_server{
        server www.helloworld.com:8081;
    }

    upstream admin_server{
        server www.helloworld.com:8082;
    }

    upstream finance_server{
        server www.helloworld.com:8083;
    }

    server {
        #此处省略一些基本配置
        #默认指向product的server
        location / {
            proxy_pass http://product_server;
        }

        location /product/{
            proxy_pass http://product_server;
        }

        location /admin/ {
            proxy_pass http://admin_server;
        }

        location /finance/ {
            proxy_pass http://finance_server;
        }
    }
}
```



## 3.3 https 反向代理配置

一些对安全性要求比较高的站点，可能会使用 HTTPS（一种使用 ssl 通信标准的安全 HTTP 协议）。
使用 nginx 配置 https 需要知道几点：

- HTTPS 的固定端口号是 443，不同于 HTTP 的 80 端口
- SSL 标准需要引入安全证书，所以在 nginx.conf 中你需要指定证书和它对应的 key

其他和 http 反向代理基本一样，只是在 Server 部分配置有些不同。

```yml
  #HTTP服务器
  server {
      #监听443端口。443为知名端口号，主要用于HTTPS协议
      listen       443 ssl;

      #定义使用www.xx.com访问
      server_name  www.helloworld.com;

      #ssl证书文件位置(常见证书文件格式为：crt/pem)
      ssl_certificate      cert.pem;
      #ssl证书key位置
      ssl_certificate_key  cert.key;

      #ssl配置参数（选择性配置）
      ssl_session_cache    shared:SSL:1m;
      ssl_session_timeout  5m;
      #数字签名，此处使用MD5
      ssl_ciphers  HIGH:!aNULL:!MD5;
      ssl_prefer_server_ciphers  on;

      location / {
          root   /root;
          index  index.html index.htm;
      }
  }
```



# 四、负载均衡

假设这样一个应用场景：将应用部署在 192.168.1.11:80、192.168.1.12:80、192.168.1.13:80 三台 linux 环境的服务器上。网站域名叫 www.helloworld.com，公网 IP 为 192.168.1.11。在公网 IP 所在的服务器上部署 nginx，对所有请求做负载均衡处理。

nginx.conf 配置如下：

```bash
http {
     #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    access_log    /var/log/nginx/access.log;

    #设定负载均衡的服务器列表
    upstream load_balance_server {
        #weigth参数表示权值，权值越高被分配到的几率越大
        server 192.168.1.11:80   weight=5;
        server 192.168.1.12:80   weight=1;
        server 192.168.1.13:80   weight=6;
    }

   #HTTP服务器
   server {
        #侦听80端口
        listen       80;

        #定义使用www.xx.com访问
        server_name  www.helloworld.com;

        #对所有请求进行负载均衡请求
        location / {
            root        /root;                 #定义服务器的默认网站根目录位置
            index       index.html index.htm;  #定义首页索引文件的名称
            proxy_pass  http://load_balance_server ;#请求转向load_balance_server 定义的服务器列表

            #以下是一些反向代理的配置(可选择性配置)
            #proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_connect_timeout 90;          #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90;             #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90;             #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k;              #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
            proxy_busy_buffers_size 64k;       #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;    #设定缓存文件夹大小，大于这个值，将从upstream服务器传

            client_max_body_size 10m;          #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k;      #缓冲区代理缓冲用户端请求的最大字节数
        }
    }
}
```



## 4.1 轮询

```
upstream myserver {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;
}
```

## 4.2 加权轮询

- 性能更好的服务器权重应更高

```
upstream myserver {
  server 192.168.250.220:8080   weight=3;
  server 192.168.250.221:8080;              # default weight=1
  server 192.168.250.222:8080;              # default weight=1
}
```

## 4.3 最少连接

```
upstream myserver {
  least_conn;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;
}
```

## 4.4 加权最少连接

- 性能更好的服务器权重应更高

```
upstream myserver {
  least_conn;

  server 192.168.250.220:8080   weight=3;
  server 192.168.250.221:8080;              # default weight=1
  server 192.168.250.222:8080;              # default weight=1
}
```

## 4.5 IP Hash

- 算法：根据客户端ip进行Hash得到一个数值，然后使用该数值对服务器个数取模，得到的结果就是映射的服务器序号
- （在服务器个数不变的情况下）可保证同一ip地址的请求始终映射到同一台服务器，解决了session共享问题

```
upstream myserver {
  ip_hash;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;

}
```

## 4.6 uri Hash

- （在服务器个数不变的情况下）可保证同一uri始终映射到同一台服务器
- nginx在1.7.2之后支持uri_hash

```
upstream myserver {
  hash $request_uri;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;
}
```



# 五、缓存服务器





# 六、文件服务器

使用 Nginx 可以非常快速便捷的搭建一个简易的文件服务。

Nginx 中的配置要点：

- 将 autoindex 开启可以显示目录，默认不开启。
- 将 autoindex_exact_size 开启可以显示文件的大小。
- 将 autoindex_localtime 开启可以显示文件的修改时间。
- root 用来设置开放为文件服务的根路径。
- charset 设置为 charset utf-8,gbk;，可以避免中文乱码问题（windows 服务器下设置后，依然乱码，本人暂时没有找到解决方法）。

一个最简化的配置如下：

```yml
autoindex on;# 显示目录
autoindex_exact_size on;# 显示文件大小
autoindex_localtime on;# 显示文件时间

server {
    charset      utf-8,gbk; # windows 服务器下设置后，依然乱码，暂时无解
    listen       9050 default_server;
    listen       [::]:9050 default_server;
    server_name  _;
    root         /share/fs;
}
```



# 七、跨域解决方案

web 领域开发中，经常采用前后端分离模式。这种模式下，前端和后端分别是独立的 web 应用程序，例如：后端是 Java 程序，前端是 React 或 Vue 应用。

各自独立的 web app 在互相访问时，势必存在跨域问题。解决跨域问题一般有两种思路：

## 7.1  CORS

在后端服务器设置 HTTP 响应头，把你需要运行访问的域名加入加入 Access-Control-Allow-Origin中。

## 7.2 jsonp

把后端根据请求，构造 json 数据，并返回，前端用 jsonp 跨域。



nginx 根据第一种思路，也提供了一种解决跨域的解决方案。
举例：www.helloworld.com 网站是由一个前端 app ，一个后端 app 组成的。前端端口号为 9000， 后端端口号为 8080。
前端和后端如果使用 http 进行交互时，请求会被拒绝，因为存在跨域问题。来看看，nginx 是怎么解决的吧：
首先，在 enable-cors.conf 文件中设置 cors ：

```yml
# allow origin list
set $ACAO '*';

# set single origin
if ($http_origin ~* (www.helloworld.com)$) {
  set $ACAO $http_origin;
}

if ($cors = "trueget") {
    add_header 'Access-Control-Allow-Origin' "$http_origin";
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
}

if ($request_method = 'OPTIONS') {
  set $cors "${cors}options";
}

if ($request_method = 'GET') {
  set $cors "${cors}get";
}

if ($request_method = 'POST') {
  set $cors "${cors}post";
}
```

接下来，在你的服务器中 include enable-cors.conf 来引入跨域配置：

```yml
# ----------------------------------------------------
# 此文件为项目 nginx 配置片段
# 可以直接在 nginx config 中 include（推荐）
# 或者 copy 到现有 nginx 中，自行配置
# www.helloworld.com 域名需配合 dns hosts 进行配置
# 其中，api 开启了 cors，需配合本目录下另一份配置文件
# ----------------------------------------------------
upstream front_server{
  server www.helloworld.com:9000;
}
upstream api_server{
  server www.helloworld.com:8080;
}

server {
  listen       80;
  server_name  www.helloworld.com;

  location ~ ^/api/ {
    include enable-cors.conf;
    proxy_pass http://api_server;
    rewrite "^/api/(.*)$" /$1 break;
  }

  location ~ ^/ {
    proxy_pass http://front_server;
  }
}
```



# 八、进程数、并发数、系统优化

## 8.1 配置nginx.conf，增加并发量

```bash
# 与CPU逻辑核心数一致
worker_processes 12;
events {
    # 单个worker最大并发连接数
    worker_connection 65535;
}
```

## 8.2 调整内核参数

- 查看所有的属性值
  `ulimit -a`
- 临时设置硬限制（重启后失效）
  `ulimit -Hn 100000`
- 临时设置软限制（重启后失效）
  `ulimit -Sn 100000`
- 持久化设置（重启后仍生效）

```bash
vim /etc/security/limits.conf
 接下来是文件中需要配置的内容
    *           soft        nofile          100000
    *           hard        nofile          100000
 用户/组   软/硬限制   需要限制的项目     限制的值
```

# 九、安全配置

## 9.1 版本安全

- 隐藏HTTP Response消息头Server中的版本号
  - 隐藏前：Server: nginx/1.18.0
  - 隐藏后：Server: nginx

```
http {
    server_tokens off;
}
```

## 9.2 IP安全

- 白名单配置（适用于授权IP较少的情况），可配置在http、server、location中

```
location / {
    allow 192.168.1.1;
    deny all;
}
```

- 黑名单配置（适用于授权IP较多的情况），可配置在http、server、location中

```
location / {
    deny 192.168.1.1;
    allow all;
}
```

## 9.3 文件安全

```
location /logs {
    autoindex on;
    root/opt/nginx/;
}

location ^logs~*\.(log|txt)$ {
    add_header Content-Type text/plain;
    root/opt/nginx/;
}
```



# 十、状态监控

## 10.1 配置访问地址

```bash
location /nginxstatus {
    stub_status on;
    # 禁止将监控信息写入访问日志
    access_log off;
}
```

## 10.2 安装插件并重启

```bash
cd /usr/local/src/nginx-1.18.0
# 如果不是使用的默认路径，使用 --prefix 指定
./configure --with-http_stub_status_module
make && make install
/usr/local/nginx/sbin/nginx -s quit
/usr/local/nginx/sbin/nginx
```

## 10.3 访问地址，状态参数如下

- Active connections：活跃的连接数量
- server accepts handled requests：处理的总连接数 创建的握手数 处理的总请求数
- reading：读取客户端的Header信息的次数。这个操作仅读取头部信息，完成后立即进入writing状态
- writing：响应数据传到客户端的Header信息次数。这个操作不仅读取头部，还要等待服务响应
- waiting：开启keep-alive后等候下一次请求指令的驻留连接



# 参考链接

- [nginx配置静态资源访问](https://www.cnblogs.com/qingshan-tang/p/12763522.html)
- [微信公众号：民工哥技术之路：dunwu](https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247494790&idx=2&sn=acc3075a8b5e59824baf31a2b81b560b&chksm=e918899ade6f008cf7abb6864dbf8ca3c3e94e54c66115272a36f2df390c44ce0538ad4de1a1&scene=126&sessionid=1592266153&key=4b0291f9d497b1b0e5b13537eabdfd65f20e3711275ab33a24b1d57b7c3d58d658efa1c467b3b39845ea1832cc3275dd9265d9471b9fc4fc5008b22b4fabd2c0b8396baf2f796f35d88e133adb455b2a&ascene=1&uin=MjkxMzM3MDgyNQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A7TErMfk%2F0DnI53DaeMrKg4%3D&pass_ticket=LPvgPt7Z83HVbqEBiCUd4DpwbsYcB0xQVnfUjoxD5EVKc%2FNZ85pmeMbzepgknOxS)
- [Nginx日志的标准格式](http://www.cnblogs.com/kevingrace/p/5893499.html)
- [民工哥](https://segmentfault.com/u/jishuroad)：[Nginx + Spring Boot 实现负载均衡](https://segmentfault.com/a/1190000037594169)
- [Nginx 高性能优化配置实战总结](https://segmentfault.com/a/1190000037788252)
- 渡渡鸟：[02-Nginx事件模型](https://www.yuque.com/duduniao/nginx)



