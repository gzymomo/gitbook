[TOC]

[掘金：clouding：浏览器播放rtsp视频流解决方案](https://juejin.im/post/5d183a71f265da1b6e65b8ff)
[利用JAVACV解析RTSP流，通过WEBSOCKET将视频帧传输到WEB前端显示成视频](https://www.freesion.com/article/4840533481/)
[CSDN：zctel：javacv](https://blog.csdn.net/u013947963/category_9570094.html)
[CSDN：斑马jio：JavaCV转封装rtsp到rtmp（无需转码，低资源消耗）](https://blog.csdn.net/weixin_40777510/article/details/103764198?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7)
[博客园：之乏：流媒体](https://www.cnblogs.com/zhifa/tag/%E6%B5%81%E5%AA%92%E4%BD%93/)
[博客园：断点实验室：ffmpeg播放器实现详解 - 视频显示](https://www.cnblogs.com/breakpointlab/p/13309393.html)
[Gitee：chengoengvb：RtspWebSocket](https://gitee.com/yzfar/RtspWebSocket)

![](https://pic2.zhimg.com/v2-9a7f1464868ad15496c732877f89d11e_1200x500.jpg)


# 方案一：服务器端用 websocket 接受 rtsp ，然后，推送至客户端
![](https://img-blog.csdnimg.cn/20190613174326863.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fqcm0wOTI1,size_16,color_FFFFFF,t_70)

此方案，客户端因为直接转成了mp4，所以H5的video标签直接可以显示。

参考地址：https://github.com/Streamedian/html5_rtsp_player

## 实现步骤：
1. 服务器安装streamedian服务器
2. 客户端通过video标签播放
```html
<video id="test_video" controls autoplay></video>
<script src="free.player.1.8.4.js"></script>
<script>

    if (window.Streamedian) {
        var errHandler = function(err){
            console.log('err', err.message);
        };

        var infHandler = function(inf) {
            console.log('info', inf)
        };

        var playerOptions = {
            socket: "ws://localhost:8088/ws/",
            redirectNativeMediaErrors : true,
            bufferDuration: 30,
            errorHandler: errHandler,
            infoHandler: infHandler
        };

        var html5Player  = document.getElementById("test_video");
        html5Player.src = "rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov";

        var player = Streamedian.player('test_video', playerOptions);

        window.onbeforeunload = function(){
            player && player.destroy();
            player = null;
            Request = null;
        }
    }
</script>
```

# 方案二：使用 ffmpeg + nginx 把 rtsp 转成了 hls 协议，客户端使用 videojs 播放
参考地址：
> https://videojs.com/getting-started/

HLS (HTTP Live Streaming) 直播 是有苹果提出的一个基于http的协议。其原理是把整个流切分成一个个的小视频文件，然后通过一个m3u8的文件列表来管理这些视频文件。

HTTP Live Streaming 并不是一个真正实时的流媒体系统，这是因为对应于媒体分段的大小和持续时间有一定潜在的时间延时。在客户端，至少在一个分段媒体文件被完全下载后才能够开始播放，而通常要求下载完两个媒体文件之后才开始播放以保证不同分段音视频之间的无缝连接。

此外，在客户端开始下载之前，必须等待服务器端的编码器和流分割器至少生成一个TS文件，这也会带来潜在的时延。

服务器软件将接收到的流每缓存一定时间后包装为一个新的TS文件，然后更新m3u8文件。m3u8文件中只保留最新的几个片段的索引，以保证观众任何时候连接进来都会看到较新的内容，实现近似直播的效果。
这种方式的理论最小延时为一个ts文件的时长，一般为2-3个ts文件的时长。

## 实现步骤
1. 安装ffmpeg工具
2. ffmpeg转码
```bash
ffmpeg -i "rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov" -c copy -f hls -hls_time 2.0 -hls_list_size 0 -hls_wrap 15 "D:/Program Files/html/hls/test.m3u8"
```

ffmpeg 关于hls方面的指令说明：

- -hls_time n: 设置每片的长度，默认值为2。单位为秒
- -hls_list_size n:设置播放列表保存的最多条目，设置为0会保存有所片信息，默认值为5
- -hls_wrap n:设置多少片之后开始覆盖，如果设置为0则不会覆盖，默认值为0.这个选项能够避免在磁盘上存储过多的片，而且能够限制写入磁盘的最多的片的数量
- -hls_start_number n:设置播放列表中sequence number的值为number，默认值为0

3. video 播放

```html
<html>
<head>
<title>video</title>
<!-- 引入css -->
<link rel="stylesheet" type="text/css" href="./videojs/video-js.min.css" />

</head>
<body>

<div class="videoBox">
    <video id="my_video_1" class="video-js vjs-default-skin" controls>
        <source src="http://localhost:8088/hls/test.m3u8" type="application/x-mpegURL">
    </video>
</div>

</body>
</html>
<script type="text/javascript" src="./videojs/video.min.js"></script>
<script type="text/javascript" src="./videojs/videojs-contrib-hls.min.js"></script>
<script>
videojs.options.flash.swf = "./videojs/video-js.swf"
    var player = videojs('my_video_1', {"autoplay":true});
    player.play();
</script>
```


# 方案三：用 ffmpeg 把 rtsp 转成 rtmp 通过 nginx代理出去，其中核心处用到了 nginx 的 nginx-rtmp-module 模块，在客户端则使用著名的网页播放器 jwplayer 来播放 rtmp 流即可，但是 jwplayer 播放器好像是付费的，此方案也是搭建流媒体服务器的通用方案之一。

rtmp是adobe开发的协议，一般使用adobe media server 可以方便的搭建起来；随着开源时代的到来，有大神开发了nginx的rtmp插件，也可以直接使用nginx实现rtmp
rtmp方式的最大的优点在于低延时，经过测试延时普遍在1-3秒，可以说很实时了；缺点在于它是adobe开发的，rtmp的播放严重依赖flash，而由于flash本身的安全，现代浏览器大多禁用flash。

## 实现步骤：
1. 安装ffmpeg工具
2. 安装nginx 注意：linux系统需要安装 nginx-rtmp-module 模块，Windows系统安装包含rtmp的(如nginx 1.7.11.3 Gryphon)
3. 更改nginx配置

```xml
rtmp{
    server{
    listen 1935;

        application live{
          live on;
          record off;
        }
        application hls{
          live on;
          hls on;
          hls_path nginx-rtmp-module/hls;
          hls_cleanup off;
        }
    }
}
```

4. ffmpeg转码

```bash
ffmpeg -i "rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov" -f flv -r 25 -s 1080*720 -an "rtmp://127.0.0.1:1935/hls/mystream"
```

5. video 播放

```html
<html>
<head>
<title>video</title>
<!-- 引入css -->
<link rel="stylesheet" type="text/css" href="./videojs/video-js.min.css" />

</head>
<body>
<video id="test_video" class="video-js vjs-default-skin vjs-big-play-centered" controls autoplay>
    <source src='rtmp://127.0.0.1:1935/hls/mystream' type='rtmp/flv'/>
</video>

</body>
</html>
<!-- 引入js -->
<script type="text/javascript" src="./videojs/video.min.js"></script>
<script type="text/javascript" src="./videojs/videojs-flash.js"></script>

<script>
videojs.options.flash.swf = "./videojs/video-js.swf"
    var player = videojs('test_video', {"autoplay":true});
    player.play();
</script>
```
注意：使用谷歌浏览器播放时，需要开启flash允许

# 四：video-source资源
> 视频播放测试服务（spring boot+javacv+websocket）
> spring boot+opencv+websocket:https://www.freesion.com/article/2770277090/

> Web下无插件播放rtsp视频流的方案总结_网络_经验之外:https://blog.csdn.net/ajrm0925/article/details/91879551?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2

> FFmpeg转封装rtsp到rtmp（无需转码，低资源消耗）_Java_banmajio的博客-CSDN博客:https://blog.csdn.net/weixin_40777510/article/details/103764198?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7

> easyCV/pom.xml at master · eguid/easyCV · GitHub:https://github.com/eguid/easyCV/blob/master/pom.xml
> javacv rtsp rtmp_Java：https://blog.csdn.net/zb313982521/article/details/88110368

> JAVA中通过JavaCV实现跨平台视频/图像处理-调用摄像头：https://blog.csdn.net/weixin_33881041/article/details/85992437

> javacv 通过rtsp 获取视频流 设置帧率：https://www.cnblogs.com/svenwu/p/9663038.html

> javaCV开发详解之8：转封装在rtsp转rtmp流中的应用（无须转码，更低的资源消耗）：https://www.cnblogs.com/eguid/p/10195557.html

> RTSP实例解析：https://www.cnblogs.com/qq78292959/archive/2010/08/12/2077039.html

> JAVACV集成海康摄像头RTSP流时出现的丢包与无法解析的问题：https://bbs.csdn.net/topics/392237412?page=1java使用opencv拉取rtsp流：https://blog.csdn.net/sinat_21184471/article/details/93978622

> WebSocket插件：https://www.bbsmax.com/A/RnJW3xRRJq/sockjs+stomp的websocket插件

> sockjs+stomp的websocket插件：https://www.bbsmax.com/A/pRdBPLN9Jn/

> SpringBoot使用Websocket：https://blog.csdn.net/weixin_43835717/article/details/94066791

> SpringBoot2.0集成WebSocket，实现后台向前端推送信息：https://blog.csdn.net/moshowgame/article/details/80275084

> 【Websoket】实时推送图像数据，前端实时显示：https://blog.csdn.net/qq_20038925/article/details/70919307

> WebSocket传输图片：https://www.cnblogs.com/jzblogs/p/5613988.html

> JavaScript进行WebSocket字节流通讯示例：https://blog.coloc.cc/2018/11/javascriptwebsocket.html

> JavaScript进行WebSocket字节流通讯示例 ：https://www.cnblogs.com/coloc/p/8127268.html

> spring boot 使用WebSocket与前端进行byte字节数组交互：https://blog.csdn.net/m0_37992075/article/details/83587624

> spring boot集成javacv + websocket实现实时视频推流回放（延时1-2秒）：https://blog.csdn.net/qq_23945685/article/details/104074262