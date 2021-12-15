[怎么搭建hls低延时直播（lowlatency hls）](https://blog.csdn.net/impingo/article/details/102558792)



# hls延时现状

hls延时太大是众所周知的情况，网上也有非常多的资料分析为何产生那么大的延时。然而降低延时的方案却非常少。

## 出身草根的HLS+

网上有人提出过hls+的解决方案，大概思路就是为每个播放端维护一个虚拟session。
第一 保证第一个切片以最近的I帧开始，第二 保证切片足够小（小于等于1s）。
实际测试下来延时大概能够缩小到4s左右。例如[这篇文章提到的解决方案](http://www.voidcn.com/article/p-uxbnbvbr-qr.html)。

这种解决方案虽然看起来是比较野路子一些，但是确实目前实现成本最低（基于http 1.0 或者 1.1 没有对m3u8做任何改动）并且可维护性更高的一种方案。它的优点正如[这篇文章](http://www.voidcn.com/article/p-uxbnbvbr-qr.html)提到的：
（1）大幅降低传统HLS协议的延时；

（2）HLS+与RTMP/FLV等协议共用同一个流媒体服务器集群，CDN网络可省一套设备组（众多机房和设备建设）；

（3）HLS+与RTMP/FLV/HLS等使用同一个域名，用户配置简单，使用简单，域名CNAME简单；

（4）数据统一，系统给出的在线人数，带宽，计费，防盗链，鉴权认证等，全部统一实现，无需单独区分协议；

（5）合并回源，访问RTMP/FLV/HLS+只需回一路RTMP，并且是访问时才回源，大幅降低了客户源站带宽与回源压力；

（6）快速排错，HLS+使用观止云BMS可追溯日志，可以毫秒级抓取到每个客户端链接数据，实现快速排错；

（7）快速启动，通过GOP优化能做到无感官差异的快速启动；

（8）避免404，HLS是切片文件分发，会有一定概率的404，而HLS+为流式回源杜绝了此情况；

（9）避免每次回源取片的时间，HLS每个切片从源站到边缘都得回源，而HLS+使用长连接回源。

# 使用开源项目pingo搭建hls+服务

**项目地址：https://github.com/im-pingo/nginx-rtmp-module**

pingo项目是基于nginx-rtmp-module实现的支持hls hls+ http-ts http-flv rtmp多种直播协议的流媒体服务器，并且当前已经有数家公司在使用。其他协议的配置可以参考我之前的文章[分布式直播系统（二）【搭建单点rtmp\http-flv\hls流媒体服务器】](https://blog.csdn.net/impingo/article/details/99131594)，里面详细介绍了各个协议的配置方法并且有详细的配置模板。
这里只提供hls+的配置模板。

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

#pid        logs/nginx.pid;

events {
    use epoll;
    worker_connections  1024;
    multi_listen unix:/tmp/http 80;
}

stream_zone buckets=1024 streams=4096;

rtmp {
    log_format log_json '{$remote_addr, [$time_local]}';
    access_log logs/rtmp.log trunc=2s;
    server {
        listen 1935;
        serverid 000;
        out_queue 2048;
   
        application live {
            rtmp_auto_pull on;
            rtmp_auto_pull_port unix:/tmp/rtmp;
            live on;
            hls on;
            hls_path /tmp/hls;
            hls_fragment 1300ms;
            hls_max_fragment 1800ms;
            hls_playlist_length 3900ms;

            hls2memory on;
            mpegts_cache_time 20s;

            hls2_fragment 1000ms;
            hls2_max_fragment 1300ms;
            hls2_playlist_length 3000ms;

            wait_key on;
            wait_video on;
            cache_time 3s;
            low_latency off;
            fix_timestamp 0s;
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

         location /live {
            flv_live 1935;
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
         }

        location /hls2 {
             hls2_live 1935 app=live;
             add_header 'Access-Control-Allow-Origin' '*';
         }
         location / {
             chunked_transfer_encoding on;
             root html/;
         }
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119
```

使用上面的配置模板测试推流：
rtmp推流地址：rtmp://i p / l i v e / {ip}/live/*i**p*/*l**i**v**e*/{stream}
hls+ (http短连接)播放地址：http://i p / h l s 2 / {ip}/hls2/*i**p*/*h**l**s*2/{stream}
http-flv播放地址：http://i p / l i v e / {ip}/live/*i**p*/*l**i**v**e*/{stream}
rtmp播放地址：rtmp://i p / l i v e / {ip}/live/*i**p*/*l**i**v**e*/{stream}

# hls+ 测试效果

以上面的配置模板为例，推流端设置为2s gop大小，最终测试延时大概在4s~5s。
hls+与http-flv、rtmp和传统hls直播延时的对比情况如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015150034466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)

