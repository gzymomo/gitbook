**1、RTMP：**

   实时消息传输协议，Real Time Messaging Protocol，是 Adobe Systems 公司为 Flash 播放器和服务器之间音频、视频和数据传输开发的开放协议。协议基于 TCP，是一个协议族，包括 RTMP 基本协议及 RTMPT/RTMPS/RTMPE 等多种变种。RTMP 是一种设计用来进行实时数据通信的网络协议，主要用来在 Flash/AIR 平台和支持RTMP协议的流媒体/交互服务器之间进行音视频和数据通信。这种方式的实时性比较强，基本能保证延迟在1-2s内，是现在国内直播主要采用的方式之一；不过使用这种协议，就必须安装flash，而H5、IOS、Android并不能原生支持flash，因此这种协议能流行多久，就不得而知了，毕竟移动端才是现在的主流。

**2、HLS：**

   hls是Apple推出的直播协议，是通过视频流切片成文件片段来直播的。客户端首先会请求一个m3u8文件，里面会有不同码率的流，或者直接是ts文件列表，通过给出的ts文件地址去依次播放。在直播的时候，客户端会不断请求m3u8文件，检查ts列表是否有新的ts切片。这种方式的实时性较差，不过优势是H5、IOS、Android都原生支持。

**3、HTTP-FLV：**

   HTTP-FLV就是对RTMP协议的封装，相比于RTMP，它是一个开放的协议。因此他具备了RTMP的实时性和RTMP不具备的开发性，而且随着flv.js出现（感谢B站），使得浏览器在不依赖flash的情况下，播放flv视频，从而兼容了移动端，所以现在很多直播平台，尤其是手机直播平台，都会选择它。



# 一、安装加载nginx-rtmp-module模块的nginx



## **1、到[nginx.org](http://www.baidu.com/link?url=7l7tucTMDAAOTOLIpq30mcdwr6Xp5TnHiiole1ikCxG) 下载稳定版本的nginx**

```bash
wget http://nginx.org/download/nginx-1.12.1.tar.gz

tar -xvf nginx-1.12.1.tar.gz
```



##  **2、到 https://github.com/arut/nginx-rtmp-module 下载rtmp模块**

```bash
git clone https://github.com/arut/nginx-rtmp-module.git
```



解压nginx的tar包；nginx 和trmp模块在同一目录

```
nginx-1.12.2  nginx-1.12.2.tar.gz  nginx-rtmp-module
```

## **3、到nginx解压目录配置编译参数**

```bash
# 注意nginx-rtmp-module的安装路径
./configure --prefix=/usr/local/nginx --add-module=/opt/nginx-rtmp-module --conf-path=/usr/local/nginx/nginx.conf
# 经测试发现，通过--conf-path指定的nginx.conf配置文件并不好使。
# 故采用另一种方式启动nginx
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
# 经测试，上述命令更好使。通过-c 来指定配置文件。
```

## **4、make && make install 安装**

```bash
make && make install
```



## 5、问题记录

```bash
错误./configure: error: the HTTP rewrite module requires the PCRE library.

解决：安装pcre-devel：yum -y install pcre-devel

错误：./configure: error: the HTTP cache module requires md5 functions 

from OpenSSL library. You can either disable the module by using 
–without-http-cache option, or install the OpenSSL library into the system, 
or build the OpenSSL library statically from the source with nginx by using 
–with-http_ssl_module –with-openssl= options.

解决：yum -y install openssl openssl-devel

错误：./configure: error: the HTTP XSLT module requires the libxml2/libxslt    缺少libxml2

解决：yum -y install libxml2 libxml2-devel && yum -y install libxslt-devel

错误信息：./configure: error: the HTTP image filter module requires the GD library. You can either do not enable the module or install the libraries.

解决方法：http_image_filter_module是nginx提供的集成图片处理模块，需要gd-devel的支持   yum -y install gd-devel

错误信息：./configure: error: perl module ExtUtils::Embed is required 缺少ExtUtils

解决方法：yum -y install perl-devel perl-ExtUtils-Embed

错误信息：./configure: error: the GeoIP module requires the GeoIP library. You can either do not enable the module or install the library. 缺少GeoIP

解决方法：yum -y install GeoIP GeoIP-devel GeoIP-data

错误信息：./configure: error: the Google perftools module requires the Google perftools
library. You can either do not enable the module or install the library.

解决方法：yum -y install gperftools
```

# 二、nginx视频播放配置

## 1、点播视频服务器的配置

通过上一步nginx服务器已经搭建完成，然后我们就可以开启一个视频点播的服务了。打开配置文件nginx.conf，添加RTMP的配置。

```yaml
worker_processes  1;

events {
    worker_connections  1024;
}
rtmp {                #RTMP服务
    server {
        listen 1935;  #//服务端口 
	chunk_size 4096;   #//数据传输块的大小
        
	application vod {
		play /opt/video/vod; #//视频文件存放位置。
	}
    }
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```

## 2、直播视频服务器的配置

在点播服务器配置文件的基础之上添加直播服务器的配置。一共2个位置，第一处就是给RTMP服务添加一个application这个名字可以任意起，也可以起多个名字，由于是直播我就叫做它live吧，如果打算弄多个频道的直播就可以live_cctv1、live_cctv2名字任意。第二处就是添加两个location字段，字段的内容请直接看文件吧。

```yaml
worker_processes  1;

events {
    worker_connections  1024;
}

rtmp {
    server { 
        listen 1935;
	chunk_size 4096;
        
	application vod {
		play /opt/video/vod;
	}

	application live{ #第一处添加的直播字段
		live on;
	}
    }

}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
	
	location /stat {     #第二处添加的location字段。
            rtmp_stat all;
	    rtmp_stat_stylesheet stat.xsl;
	}

	location /stat.xsl { #第二处添加的location字段。
		root /usr/local/nginx/nginx-rtmp-module/;
	}

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}


```

## 3、实时回看视频服务器的配置

```yaml
worker_processes  1;

events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1935;
	chunk_size 4096;
        
	application vod {
		play /opt/video/vod;
	}

        application live {
        live on;
	    hls on; #这个参数把直播服务器改造成实时回放服务器。
	    wait_key on; #对视频切片进行保护，这样就不会产生马赛克了。
	    hls_path /opt/video/hls; #切片视频文件存放位置。
	    hls_fragment 10s;     #每个视频切片的时长。
	    hls_playlist_length 60s;  #总共可以回看的事件，这里设置的是1分钟。
	    hls_continuous on; #连续模式。
	    hls_cleanup on;    #对多余的切片进行删除。
	    hls_nested on;     #嵌套模式。
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
	location /stat {
            rtmp_stat all;
	    rtmp_stat_stylesheet stat.xsl;
	}

	location /stat.xsl {
		root /usr/local/nginx/nginx-rtmp-module/;
	}
	
	location /live {  #这里也是需要添加的字段。
		types {  
			application/vnd.apple.mpegurl m3u8;  
			video/mp2t ts;  
		}
		alias /opt/video/hls;   
		expires -1;
		add_header Cache-Control no-cache;  
	}  

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## 4、**配置nginx rtmp直播功能nginx.conf**

```yaml
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on; #内核直接发送文件到客户端
    tcp_nopush          on; #够一定数据量再发送
    tcp_nodelay         on; #同上
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
#        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        location / {
            resolver 8.8.8.8;
            #proxy_pass $scheme://$http_host$request_uri;
            proxy_buffers   256 4k;                         
            proxy_max_temp_file_size 0k; 
        }
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/server.crt";
        ssl_certificate_key "/etc/pki/nginx/private/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    server {
    listen 8080;
        #配置RTMP状态一览HTTP页面=========================================
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
            root /opt/rtmp/nginx-rtmp-module/;
        }
        #配置RTMP状态一览界面结束==========================
　　　　　　　　　　#HTTP协议访问直播流文件配置
        location /hls {  #添加视频流存放地址。
                types {
                    application/vnd.apple.mpegurl m3u8;
                    video/mp2t ts;
                }
                #访问权限开启，否则访问这个地址会报403
                autoindex on;
                alias /usr/share/nginx/html/hls;#视频流存放地址，与下面的hls_path相对应，这里root和alias的区别可自行百度
                expires -1;
                add_header Cache-Control no-cache;
                #防止跨域问题
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Allow-Credentials' 'true';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';  
            }
    }

}
#点播/直播功能实现配置rtmp协议
rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application vod {
            play /usr/share/nginx/html/vod/flvs/;#点播媒体存放目录
        }
        application live {
            live on;
        }　　　　　　#HLS直播配置
        application hls {
            live on;
            hls on;
            hls_path /usr/share/nginx/html/hls;#视频流存放地址
            hls_fragment 5s;
            hls_playlist_length 15s;
            hls_continuous on; #连续模式。
            hls_cleanup on;    #对多余的切片进行删除。
            hls_nested on;     #嵌套模式。
        }
    }
}
```

# 三、实践使用的nginx.conf

## 1、ffmpeg拉流命令

```bash
ffmpeg -re  -rtsp_transport tcp -i rtsp:////////  -vcodec copy  -f flv  rtmp://192.168.0.140/rtmp
```

## 2、nginx配置文件

```yaml
#user  nobody;

error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

worker_processes 1;

events {
  worker_connections 1024;
}

rtmp {
  server {
    listen 1935;
	access_log logs/rtmp_access.log;
    chunk_size 4096;
	
	application rtmp {
            live on;
            notify_method get;
            on_play http://192.168.0.141:7081/rtmp/on_play;
            on_publish http://192.168.0.141:7081/rtmp/on_publish;
            on_done http://192.168.0.141:7081/rtmp/on_done;
            on_play_done http://192.168.0.141:7081/rtmp/on_play_done;
            on_publish_done http://192.168.0.141:7081/rtmp/on_publish_done;
            on_record_done http://192.168.0.141:7081/rtmp/on_record_done;
            on_update http://192.168.0.141:7081/rtmp/on_update;
            notify_update_timeout 30s;
        }

    application vod {
      play /opt/video/vod;
    }

    application live { #第一处添加的直播字段
      live on;
    }

    #hls配置
    application hls {
      live on;
      hls on;
      hls_path /opt/video/hls;
    }
  }

}

http {
  include mime.types;
  default_type application/octet-stream;
  sendfile on;
  keepalive_timeout 65;
  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;

  gzip  on;
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  output_buffers 1 32k;
  postpone_output 1460;
  client_header_timeout 3m;
  client_body_timeout 3m;
  send_timeout 3m;
  tcp_nopush on;
  tcp_nodelay on;

  server {
    listen 80;
    server_name localhost;

    charset utf-8;

    location /stat { #第二处添加的location字段。
      rtmp_stat all;
      rtmp_stat_stylesheet stat.xsl;
    }
    location /stat.xsl { #第二处添加的location字段。
      root /usr/local/nginx/nginx-rtmp-module;
    }
    location / {
      root html;
      index index.html index.htm;
    }
    #配置hls
    location /hls {
      types {
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
      }
      root /opt;
      add_header Cache-Control no-cache;
    }

    location ~ \.mp4$ {
      root   /opt/video/mp4/;
    }


    location ~ \.m4v$ {
      root   /opt/video/m4v/;
    }

    location ~ \.flv$ {
      root   /opt/video/flv/;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
      root html;
    }
  }
}
```

