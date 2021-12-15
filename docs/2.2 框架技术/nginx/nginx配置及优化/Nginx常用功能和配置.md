[TOC]

# 1、nginx常用功能和配置
## 1.1 限流
- limit_conn_zone(限制连接数，针对客户端，即单一ip限流)
- limit_req_zone(限制请求数，针对客户端，即单一ip限流)
- ngx_http_unpstream_module（推荐，针对后台，如：有两台服务器，服务器A最大可并发处理10W条请求，服务器B最大可并发处理5W条请求，这样当12W请求按照负载均衡规则应当被分配给服务器A时，nginx为了防止A挂掉，所以将另外的2W分配给B）

## 1.2 压力测试工具——Ab
### 1.2.1安装
`yum install httpd-tools -y`
### 1.2.2 测试
// 10个用户，向 http://www.test.com/ 并发发送1000条请求（总请求数=1000）
`ab -c 10 -n 1000 http://www.test.com/`

### 1.2.3 返回值
- Document Path：测试的页面路径
- Document Length：页面大小（byte）
- Concurrency Level：并发数量，即并发用户数
- Time taken for tests：测试耗费时长
- Complete requests：成功的请求数量
- Failed requests：请求失败的数量
- Write errors：错误数量
- Requests per second：每秒钟的请求数量、吞吐率
- Timer per request：每次请求所需时间、响应时间

## 1.3 limit_conn_zone
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
## 1.4 limit_req_zone
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

## 1.5 limit_req_zone
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

## 1.5 ngx_http_upstream_module
```
upstream MyName {
    server 192.168.0.1:8080 weight=1 max_conns=10;
    server 192.168.0.2:8080 weight=1 max_conns=10;
}
```

# 2、安全配置
## 2.1 版本安全
- 隐藏HTTP Response消息头Server中的版本号
  - 隐藏前：Server: nginx/1.18.0
  - 隐藏后：Server: nginx

```
http {
    server_tokens off;
}
```
## 2.2 IP安全
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
## 2.3 文件安全

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

# 3、进程数、并发数、系统优化
## 3.1 配置nginx.conf，增加并发量

```
# 与CPU逻辑核心数一致
worker_processes 12;
events {
    # 单个worker最大并发连接数
    worker_connection 65535;
}
```
## 3.2 调整内核参数
- 查看所有的属性值
`ulimit -a`
- 临时设置硬限制（重启后失效）
`ulimit -Hn 100000`
- 临时设置软限制（重启后失效）
`ulimit -Sn 100000`
- 持久化设置（重启后仍生效）

```
vim /etc/security/limits.conf
 接下来是文件中需要配置的内容
    *           soft        nofile          100000
    *           hard        nofile          100000
 用户/组   软/硬限制   需要限制的项目     限制的值
```

# 4、GZIP
- 作用：启用gzip后，服务器将响应报文进行压缩，有效地节约了带宽，提高了响应至客户端的速度。当然，压缩会消耗nginx所在电脑的cpu
- 配置范围：http、server、location

```
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

# 5、状态监控
## 5.1 配置访问地址
```
location /nginxstatus {
    stub_status on;
    // 禁止将监控信息写入访问日志
    access_log off;
}
```

## 5.2 安装插件并重启
```
cd /usr/local/src/nginx-1.18.0
# 如果不是使用的默认路径，使用 --prefix 指定
./configure --with-http_stub_status_module
make && make install
/usr/local/nginx/sbin/nginx -s quit
/usr/local/nginx/sbin/nginx
```
## 5.3 访问地址，状态参数如下
- Active connections：活跃的连接数量
- server accepts handled requests：处理的总连接数 创建的握手数 处理的总请求数
- reading：读取客户端的Header信息的次数。这个操作仅读取头部信息，完成后立即进入writing状态
- writing：响应数据传到客户端的Header信息次数。这个操作不仅读取头部，还要等待服务响应
- waiting：开启keep-alive后等候下一次请求指令的驻留连接

# 6、负载均衡
## 6.1 轮询
```
upstream myserver {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;
}
```
## 6.2 加权轮询
- 性能更好的服务器权重应更高

```
upstream myserver {
  server 192.168.250.220:8080   weight=3;
  server 192.168.250.221:8080;              # default weight=1
  server 192.168.250.222:8080;              # default weight=1
}
```

## 6.3 最少连接
```
upstream myserver {
  least_conn;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080;
  server 192.168.250.221:8080;
  server 192.168.250.222:8080;
}
```

## 6.4 加权最少连接
- 性能更好的服务器权重应更高
```
upstream myserver {
  least_conn;

  server 192.168.250.220:8080   weight=3;
  server 192.168.250.221:8080;              # default weight=1
  server 192.168.250.222:8080;              # default weight=1
}
```
## 6.5 IP Hash
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

## 6.6 uri Hash
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

# 7、access日志切割
## 7.1 新建Shell脚本
### 脚本文件路径随意

`vim /usr/local/nginx/nginx_log.sh`
- 将以下内容添加到脚本中

```
#! /bin/bash
# 设置日志文件存放目录（nginx安装目录为/usr/local/nginx）
LOG_HOME="/usr/local/nginx/logs"

# 备份Log名称
LOG_PATH_BAK="$(date -d yesterday +%Y%m%d%H%M)".access.log

# 重命名日志文件
mv ${LOG_HOME}/access.log ${LOG_HOME}/${LOG_PATH_BAK}.log

# 向nginx主进程发信号重新打开日志
kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
```

## 7.2 创建crontab定时作业

`crontab -e`

- 将以下内容添加到作业中去（任取一个）
  - corn表达式生成器：http://cron.qqe2.com/

```
# 以每分钟切割一次为例
*/1 * * * * sh /usr/local/nginx/nginx_log.sh

# 以每天切割一次为例
0 0 0 1/1 * ? sh /usr/local/nginx/nginx_log.sh
```

# 8、动静分离
- 概念：将动态请求和静态请求分开
- 实现方式：
  - （推荐）将静态文件存放在专门的服务器上，使用单独的域名
  - 另一种是将动态和静态文件放在一起，使用nginx区分
## 8.1 以实现方式1为例
- 前提：将静态文件存放在代理服务器中
- 在ngnix中创建文件目录（如/usr/local/nginx/static），将所有静态文件发布到该目录中
- 在nginx.conf http server 中配置动静分离
```
server {
    location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css)$
    {
        root /usr/local/nginx/static;
        # 缓存30天
        expires 30d;
    }
}
```
在实际的后台服务器中发布的程序中，使用静态文件时，路径指向设置为静态文件服务器（这里是代理服务器）。

# 9、nginx转发配置

```yaml
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;

    keepalive_timeout  65;

    server {
        listen  8888;
	    client_max_body_size 50m;        
        charset utf-8;

        location / {
		   proxy_pass  http://ip:8868/;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header Host $host:50;
	       proxy_set_header Upgrade $http_upgrade;
	       proxy_set_header Connection  "upgrade";
	    }

        location /arcgis_js_api/ {
           add_header 'Access-Control-Allow-Origin' '*';
           add_header 'Access-Control-Allow-Credentials' 'true';
           add_header 'Access-Control-Allow-Headers' 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
           add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
           root /usr/local/nginx-zydz/html;
           expires  7d;
        }
    }
}

```

