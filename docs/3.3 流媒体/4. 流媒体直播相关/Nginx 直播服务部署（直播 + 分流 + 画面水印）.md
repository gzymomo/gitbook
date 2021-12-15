- [Nginx 直播服务部署（直播 + 分流 + 画面水印）](https://juejin.cn/post/6844904089361317901?utm_source=gold_browser_extension%3Futm_source%3Dgold_browser_extension)



需求：将直播流分流到两个云厂商的直播云，一个有水印，一个无水印。使用hls播放

**拓扑示意图：**

![图片](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEbaiabcvjqE4EpeGKY4BqMmT2btDt3GM6wGbgy6voTgffxicdCsEU4L338ShW7GTgicdlwky7kTf07Mw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当前拓扑示意图

![图片](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEbaiabcvjqE4EpeGKY4BqMmTs0o25iaibibbYWwQhJGEaAtfoibbI4Wlr3D9zibJeVialicbtEkHtTRicRqphw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 一、docker-nginx-rtmp-ffmpeg

基于docker-nginx-rtmp进行配置部署，这篇文章的意义是实现直播分流及直播画面水印.



- Nginx 1.16.1（从源代码编译的稳定版本）

- nginx-rtmp-module 1.2.1（从源代码编译）

- ffmpeg 4.2.1（从源代码编译）

- 已配置好的nginx.conf

- - 只支持1920*1080

  - 实现两路分流

  - - 本机
    - 直播云（例：阿某云、腾讯云、ucloud）

  - 实现直播水印效果

  - - 水印图片存放位置（容器内）：/opt/images/logo.png

# 二、部署安装

```bash
拉取docker镜像并运行
docker pull ar414/nginx-rtmp-ffmpeg
docker run -it -d -p 1935:1935 -p 8080:80 --rm ar414/nginx-rtmp-ffmpeg
```



**推流地址（Stream live content to）：**

```
rtmp://<server ip>:1935/stream/$STREAM_NAME
```



**SSL证书：**

将证书复制到容器内，并在容器内修改nginx.conf配置文件，然后重新commit（操作容器内的文件都需要重新commit才会生效）

```
listen 443 ssl;
ssl_certificate     /opt/certs/example.com.crt;
ssl_certificate_key /opt/certs/example.com.key;
```



## OBS配置

- Stream Type: `Custom Streaming Server`

- URL: `rtmp://<server ip>:1935/stream`

- Stream Key：ar414

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEbaiabcvjqE4EpeGKY4BqMmTTxwnHm3rkyljHm9S7taf43LQlYApIfOguhDDKU0VqB9aslkOTt8tfg/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**观看测试**

HLS播放测试工具：player.alicdn.com/aliplayer/s… （如果配置了证书则使用https）

**HLS播放地址**

RTMP测试工具：PotPlayer

**RTMP播放地址**

- - 无水印：

    rtmp://<server ip>:1935/stream/ar414

    ![图片](https://mmbiz.qpic.cn/mmbiz_png/QFzRdz9libEbaiabcvjqE4EpeGKY4BqMmTiaHag23xFGnLudeVjR5qYKTFPHjgcMmMK57EqbtQNKcsIflib1o3dBkQ/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  - 有水印：

    需要分流到其他服务器上



:page_facing_up:配置文件简解（分流、水印及水印位置）

**完整配置文件：**

> https://github.com/ar414-com/nginx-rtmp-ffmpeg-conf/blob/master/nginx.conf



**9、RTMP配置**



```
rtmp {
    server {
        listen 1935; #端口
        chunk_size 4000;
        #RTMP 直播流配置
        application stream {
            live on;
            #添加水印及分流，这次方便测试直接分流到当前服务器hls
            #实际生产一般都分流到直播云（腾讯云、阿某云、ucloud）
            #只需把需要分流的地址替换即可
            #有水印：rtmp://localhost:1935/hls/$name_wm
            #无水印：rtmp://localhost:1935/hls/$name
            exec ffmpeg -i rtmp://localhost:1935/stream/$name -i /opt/images/ar414.png
              -filter_complex "overlay=10:10,split=1[ar414]"
              -map '[ar414]' -map 0:a -s 1920x1080 -c:v libx264 -c:a aac -g 30 -r 30 -tune zerolatency -preset veryfast -crf 23 -f flv rtmp://localhost:1935/hls/$name_wm
              -c:a libfdk_aac -b:a 128k -c:v libx264 -b:v 2500k -f flv -g 30 -r 30 -s 1920x1080 -preset superfast -profile:v baseline rtmp://localhost:1935/hls/$name;
        }

        application hls {
            live on;
            hls on;
            hls_fragment 5;
            hls_path /opt/data/hls;
        }
    }
}
```

如果需要推多个直播云则复制多个 exec ffmpeg即可 如下：

```bash
application stream {
    live on;
    #分流至本机hls
    exec ffmpeg -i rtmp://localhost:1935/stream/$name -i /opt/images/ar414.png
      -filter_complex "overlay=10:10,split=1[ar414]"
      -map '[ar414]' -map 0:a -s 1920x1080 -c:v libx264 -c:a aac -g 30 -r 30 -tune zerolatency -preset veryfast -crf 23 -f flv rtmp://localhost:1935/hls/$name_wm
      -c:a libfdk_aac -b:a 128k -c:v libx264 -b:v 2500k -f flv -g 30 -r 30 -s 1920x1080 -preset superfast -profile:v baseline rtmp://localhost:1935/hls/$name;
    
    #分流至腾讯云
    exec ffmpeg -i rtmp://localhost:1935/stream/$name -i /opt/images/ar414.png
      -filter_complex "overlay=10:10,split=1[ar414]"
      -map '[ar414]' -map 0:a -s 1920x1080 -c:v libx264 -c:a aac -g 30 -r 30 -tune zerolatency -preset veryfast -crf 23 -f flv rtmp://live-push.tencent.com/stream/$name_wm
      -c:a libfdk_aac -b:a 128k -c:v libx264 -b:v 2500k -f flv -g 30 -r 30 -s 1920x1080 -preset superfast -profile:v baseline rtmp://live-push.tencent.com/stream/$name;

    #分流至阿某云
    exec ffmpeg -i rtmp://localhost:1935/stream/$name -i /opt/images/ar414.png
      -filter_complex "overlay=10:10,split=1[ar414]"
      -map '[ar414]' -map 0:a -s 1920x1080 -c:v libx264 -c:a aac -g 30 -r 30 -tune zerolatency -preset veryfast -crf 23 -f flv rtmp://live-push.aliyun.com/stream/$name_wm
      -c:a libfdk_aac -b:a 128k -c:v libx264 -b:v 2500k -f flv -g 30 -r 30 -s 1920x1080 -preset superfast -profile:v baseline rtmp://live-push.aliyun.com/stream/$name;
}
```



**水印位置**

**水印位置**

| 水印图片位置 | overlay值                                 |
| ------------ | ----------------------------------------- |
| 左上角       | 10:10                                     |
| 右上角       | main_w-overlay_w-10:10                    |
| 左下角       | 10:main_h-overlay_h-10                    |
| 右下角       | main_w-overlay_w-10 : main_h-overlay_h-10 |

**overlay参数**

| 参数      | 说明                                 |
| --------- | ------------------------------------ |
| main_w    | 视频单帧图像宽度（当前配置文件1920） |
| main_h    | 视频单帧图像高度（当前配置文件1080） |
| overlay_w | 水印图片的宽度                       |
| overlay_h | 水印图片的高度                       |

> 原文连接：
>
> https://juejin.im/post/5e6ba67651882549652d67ec?utm_source=gold_browser_extension