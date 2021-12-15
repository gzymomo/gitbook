掘金：[无脑仔的小明](https://home.cnblogs.com/u/wunaozai/)：[物联网架构成长之路(42)-直播流媒体入门(RTMP篇)](https://www.cnblogs.com/wunaozai/p/11772392.html)

#  一、安装RTMP流媒体服务器

　　以前其实我是利用Nginx-RTMP-module搭建过RTMP流媒体服务器，并实现了鉴权功能。参考https://www.cnblogs.com/wunaozai/p/9427730.html，看看发布时间，已经是一年多以前的事情了，感概时间过得好快啊。
　　先在Nginx官网【http://nginx.org/en/download.html】下载源码包，然后在github【https://github.com/arut/nginx-rtmp-module】下载插件包。

```bash
#下载
wget http://nginx.org/download/nginx-1.16.1.tar.gz
git clone https://github.com/arut/nginx-rtmp-module
#解压
tar -zxvf nginx-1.16.1.tar.gz
#编译，在编译检查configure过程中如果出现缺少包的，请通过apt-get获取即可
cd nginx-1.16.1
./configure --prefix=/opt/nginx --with-stream --with-http_ssl_module --with-stream_ssl_module --with-debug --add-module=../nginx-rtmp-module
make
make install
```

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171143042-1174382071.png)

　　基本的Nginx配置，如下，配置了RTMP实时流转发和HLS实时流转发两种。更多配置参考官方文档【https://github.com/arut/nginx-rtmp-module/blob/master/README.md】

```yaml
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        access_log logs/rtmp_access.log;
        application rtmp {
            live on;
        }
    }
}
```



# 二、测试推送RTMP流到服务器，并客户端拉取流数据

##  1、推流

```bash
ffmpeg -re -i 003.mov -c copy -f flv rtmp://172.16.23.202/rtmp/room
```


![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171237338-1808758035.png)

## 2、拉流：

```bash
ffplay -i rtmp://172.16.23.202/rtmp/room
```

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171250916-1980293382.png)
　　也可以使用微信小程序【腾讯视频云】里面有推流和拉流工具
![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171305896-319048683.png) ![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171322875-1077000739.png)
　　拉流测试也可以使用基于Flash的js库。这种库，还是有很多的。ckplayer【www.ckplayer.com】GrindPlayer【https://github.com/kutu/GrindPlayer】videojs【https://videojs.com】这些库基本都差不多，都是支持RTMP和HLS，RTMP使用Flash来播放。
　　浏览器播放RTMP直播流（使用ckplayer）【http://www.ckplayer.com/down/】

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>ckplayer</title>
        <style type="text/css">body{margin:0;padding:0px;font-family:"Microsoft YaHei",YaHei,"微软雅黑",SimHei,"黑体";font-size:14px}</style>
    </head>
    <body>
        <div id="video" style="width: 600px; height: 400px;"></div>
        <script type="text/javascript" src="../ckplayer/ckplayer.js"></script>
        <script type="text/javascript">
            var videoObject = {
                container: '#video', //容器的ID或className
                variable: 'player',//播放函数名称
                autoplay:false,
                live:true,
                video: 'rtmp://172.16.23.202/rtmp/room'
            };
            var player = new ckplayer(videoObject);
        </script>
    </body>
</html>
```

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171357714-2119835121.png)

 

## 3、 测试推送RTMP流到服务器，服务器提供HLS直播流

　　nginx.conf 配置文件

```yaml
http{
...
    server {
        listen 1936;
        location /html {
            root /tmp;
        }
        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no_cache;
        }

        location /dash {
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        access_log logs/rtmp_access.log;
        application rtmp {
            live on;
        }
        application hls{
            live on;
            hls on;
            hls_path /tmp/hls;
#hls_fragment 5s;
        }
        application dash{
            live on;
            dash on;
            dash_path /tmp/dash;
        }
    }
}
```

## 4、服务器缓存TS文件

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171449537-1918399951.png)

　　浏览器播放HLS直播流（这里使用videojs，10秒左右延时）



```html
<!DOCTYPE html>
<html lang="cn">
    <head>
        <meta chartset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Test</title>
        <link href="https://cdn.bootcss.com/video.js/7.6.5/video-js.min.css" rel="stylesheet">
        <script src="https://cdn.bootcss.com/video.js/7.6.5/video.min.js"></script>
    </head>

    <body>
        <h1>Live</h1>
        <video id="example-video" width="960" height="540" class="video-js vjs-default-skin" controls poster="001.png">
            <!--<source src="http://ivi.bupt.edu.cn/hls/lytv.m3u8"></source>-->
            <source src="http://172.16.23.202:1936/hls/room.m3u8"></source>
        </video>

        <script type="text/javascript">
            var player = videojs('example-video')
            player.play();
        </script>
    </body>
</html>
```

　　浏览器运行界面

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171532738-817921883.png)

```bash
ffplay -i http://172.16.23.202:1936/hls/room.m3u8
```

　　运行效果图

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171557396-198653437.png)



# 三、RTSP转RTMP协议

## 1、ffmpeg 将rtsp流媒体转换成RTMP流

```bash
ffmpeg -i "rtsp://172.16.20.197:5454/test.rtsp" -vcodec copy -acodec copy -f flv -an "rtmp://172.16.23.202:1935/rtmp/room"
ffmpeg -i "rtsp://172.16.20.197:5454/test.rtsp" -vcodec copy -acodec copy -f flv -an "rtmp://172.16.23.202:1935/hls/room"
```

　　如下图所示，FFServer先提供RTSP流。然后利用FFMpeg将RTSP流转换为RTMP流，并推送至Nginx-RTMP流媒体服务器。

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171643573-1072817230.png)

　　然后浏览器可以利用上面的js库，播放RTMP流，或者播放HLS流。

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171656787-1450262629.png)

 

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171711550-1790772151.png)

 

# 四、本章小结

　　最后总结一下，就是使用CKPlayer播放器，如果对时延不敏感的使用HLS，如果要求高的使用RTMP。虽然Flash慢慢被淘汰，但是在微信自带浏览器中还是有比较好的支持。而且各大厂商提供的直播流服务，基本都是以RTMP为主流。

　　RTSP和RTMP简单推流拉流示意图

![img](https://img2018.cnblogs.com/blog/485067/201910/485067-20191031171006023-562500722.jpg)

 

# 五、参考资料：

　　阿里云提供的播放器： https://player.alicdn.com/aliplayer/setting/setting.html
　　ffmpeg常用功能： https://www.jianshu.com/p/c141fc7881e7
　　Nginx-rtmp 直播实时流搭建：https://www.cnblogs.com/wunaozai/p/9427730.html
　　IPCamera RTSP格式： https://blog.csdn.net/hk121/article/details/83858480
　　EasyDarwin提供PC工具：https://pan.baidu.com/s/1-7lZ3KM4wPl87OLx2tWjTQ
　　很不错的一个项目：https://gitee.com/Tinywan/html5-dash-hls-rtmp
　　FFmpeg 处理RTMP流媒体命令：https://blog.csdn.net/leixiaohua1020/article/details/12029543