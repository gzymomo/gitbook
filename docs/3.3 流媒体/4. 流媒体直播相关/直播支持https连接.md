[直播支持https连接](https://blog.csdn.net/impingo/article/details/105421563)



# 描述

在浏览器上，https网页是不能引用http连接的，这也给很多人在网页上引用http-flv、hls和hls+直播地址带来困难。甚至使用srs实现https-flv的时候还需要配置反向代理。
这里介绍一种方案，直接通过修改[pingos](https://github.com/pingostack/pingos)配置快速实现https-flv、https-ts以及hls、hls+的https服务。

# 部署

```nginx
# 快速安装
git clone https://github.com/pingostack/pingos.git
cd pingos
./release.sh -i
1234
```

# 修改配置

**只需给nginx配置好https服务即可，如下：**

```nginx
listen 443 ssl;
ssl_certificate     /usr/local/pingos/cert/full_chain.pem; # 替换成你自己的公钥
ssl_certificate_key /usr/local/pingos/cert/privkey.pem; # 替换成你自己的私钥
123
```

**完整的配置模板**

```nginx
user  root;
daemon on;
master_process on;
worker_processes  1;
#worker_rlimit 4g;
#working_directory /usr/local/openresty/nginx/logs;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
error_log  logs/error.log  info;

worker_rlimit_nofile 102400;
worker_rlimit_core   2G;
working_directory    /tmp;

pid        logs/nginx.pid;

events {
    use epoll;
    worker_connections  1024;
    multi_listen unix:/tmp/http 80;
    multi_listen unix:/tmp/rtmp 1935;

    dynamic_refresh_interval 10m;
    dynamic_domain_buckets   1001;
    resolver 114.114.114.114  valid=1m;
    resolver_timeout 30s;
}

stream_zone buckets=1024 streams=4096;

dynamic_conf conf/nginx_dynamic.conf 10;
dynamic_log logs/dynamic.log info;

rtmp {
    log_format log_bandwidth '{"app":"$app","name":"$name","bitrate":$bitrate,"args":"$args","timestamp":$ntp,"ts":"$time_local","type":"$command","remote_addr":"$remote_addr","domain":"$domain"}';
    access_log logs/bandwidth.log log_bandwidth trunc=60s;

    server {
        listen 1935;
        serverid 000;
        out_queue 2048;
        server_name live.pingos.io;
        rtmp_auto_pull on;
        rtmp_auto_pull_port unix:/tmp/rtmp;

        application push {
            live on;
            push rtmp://127.0.0.1/live app=live;
        }

        application live {
#	    pull http://222.186.34.242/live/stream app=live;
           live_record off;
           live_record_path /tmp/record;

#           oclp_play http://127.0.0.1:9980/callBack stage=start,update,done;
#            recorder r1{
#                record all;
#                record_path /tmp/record;
#            }

#            exec_publish bash -c "ffmepg -i rtmp://127.0.0.1/live/$name -c copy /tmp/mp4/$name-$starttime.mp4";
#	    oclp_play http://127.0.0.1:999 stage=start args=ip=$remote_host;
#            exec_pull bash -c "ffmpeg -i rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov -c copy -f flv rtmp://127.0.0.1/live/1";
            live on;
            hls on;
            hls_path /tmp/hls;
            hls_fragment 4000ms;
#            hls_max_fragment 10000ms;
            hls_playlist_length 12000ms;
            hls_type live;

            hls2memory on;
            mpegts_cache_time 20s;

            hls2_fragment 4000ms;
            hls2_max_fragment 5000ms;
            hls2_playlist_length 12000ms;

            wait_key on;
            wait_video on;
            cache_time 1s;
            send_all on;
            low_latency off;
            fix_timestamp 2s;
# h265 codecid, default 12
            hevc_codecid  12;
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_X-Forwarded-For" "$http_X-Real-IP" "$host"';


    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #reset_server_name www.test1.com www.test2.com;
    #gzip  on;
    server {
        listen 80;
        listen 443 ssl;
        ssl_certificate     /usr/local/pingos/cert/full_chain.pem;
        ssl_certificate_key /usr/local/pingos/cert/privkey.pem;
        location /rtmp_stat {
            rtmp_stat all;
            rtmp_stat_stylesheet /stat.xsl;
        }

        location /xstat {
            rtmp_stat all;
        }

        location /sys_stat {
            sys_stat;
        }
        location ~ .mp4$ {
            root html;
            #mp4;
        }

        location /control {
            rtmp_control all;
        }
         location /flv {
             flv_live 1935 app=live;
             add_header 'Access-Control-Allow-Origin' '*';
             add_header "Access-Control-Allow-Credentials" "true";
             add_header "Access-Control-Allow-Methods" "*";
             add_header "Access-Control-Allow-Headers" "Content-Type,Access-Token";
             add_header "Access-Control-Expose-Headers" "*";
         }
         location /ts {
             ts_live 1935 app=live;
             expires -1;
             add_header 'Access-Control-Allow-Origin' '*';
             add_header "Access-Control-Allow-Credentials" "true";
             add_header "Access-Control-Allow-Methods" "*";
             add_header "Access-Control-Allow-Headers" "Content-Type,Access-Token";
             add_header "Access-Control-Expose-Headers" "*";
         }
         location /hls {
            # Serve HLS fragments
             types {
                 application/vnd.apple.mpegurl m3u8;
                 video/mp2t ts;
             }
             root /tmp;
             expires -1;
             add_header Cache-Control no-cache;
             add_header 'Access-Control-Allow-Origin' '*';
             add_header "Access-Control-Allow-Credentials" "true";
             add_header "Access-Control-Allow-Methods" "*";
             add_header "Access-Control-Allow-Headers" "Content-Type,Access-Token";
             add_header "Access-Control-Expose-Headers" "*";
         }

        location /hls2 {
             hls2_live 1935 app=live;
             add_header 'Access-Control-Allow-Origin' '*';
             add_header Cache-Control no-cache;
             add_header "Access-Control-Allow-Credentials" "true";
             add_header "Access-Control-Allow-Methods" "*";
             add_header "Access-Control-Allow-Headers" "Content-Type,Access-Token";
             add_header "Access-Control-Expose-Headers" "*";
         }
         location / {
             chunked_transfer_encoding on;
             root html/;
         }
    }
}
```