- [如何使用Nginx实现MySQL数据库的负载均衡？看完我懂了！！](https://www.cnblogs.com/binghe001/p/13340680.html)



# 前提条件

**注意：使用Nginx实现MySQL数据库的负载均衡，前提是要搭建MySQL的主主复制环境，关于MySQL主主复制环境的搭建，后续会在MySQL专题为大家详细阐述。这里，我们假设已经搭建好MySQL的主主复制环境，MySQL服务器的IP和端口分别如下所示。**

- 192.168.1.101 3306
- 192.168.1.102 3306

通过Nginx访问MySQL的IP和端口如下所示。

- 192.168.1.100 3306

# Nginx实现MySQL负载均衡

nginx在版本1.9.0以后支持tcp的负载均衡，具体可以参照官网关于模块[ngx_stream_core_module](http://nginx.org/en/docs/stream/ngx_stream_core_module.html#tcp_nodelay)的叙述，链接地址为：http://nginx.org/en/docs/stream/ngx_stream_core_module.html#tcp_nodelay。

nginx从1.9.0后引入模块ngx_stream_core_module，模块是没有编译的，需要用到编译，编译时需添加--with-stream配置参数，stream负载均衡官方配置样例如下所示。

```bash
worker_processes auto;
error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

stream {
    upstream backend {
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345            max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }

    upstream dns {
       server 192.168.0.1:53535;
       server dns.example.com:53;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 127.0.0.1:53 udp;
        proxy_responses 1;
        proxy_timeout 20s;
        proxy_pass dns;
    }
    
    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}
```

说到这里，使用Nginx实现MySQL的负载均衡就比较简单了。我们可以参照上面官方的配置示例来配置MySQL的负载均衡。这里，我们可以将Nginx配置成如下所示。

```bash
user  nginx;
#user root;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}

stream{
	upstream mysql{
		server 192.168.1.101:3306 weight=1;
		server 192.168.1.102:3306 weight=1;
	}
        
	server{
		listen 3306;
		server_name 192.168.1.100;
		proxy_pass mysql;
	}
}
```

配置完成后，我们就可以通过如下方式来访问MySQL数据库。

```bash
jdbc:mysql://192.168.1.100:3306/数据库名称
```

此时，Nginx会将访问MySQL的请求路由到IP地址为192.168.1.101和192.168.1.102的MySQL上。