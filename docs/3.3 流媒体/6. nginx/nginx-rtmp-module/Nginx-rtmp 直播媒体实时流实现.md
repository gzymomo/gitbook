掘金：[无脑仔的小明](https://home.cnblogs.com/u/wunaozai/)：[Nginx-rtmp 直播媒体实时流实现](https://www.cnblogs.com/wunaozai/p/9427730.html)

实际项目中，用到的架构实现流程图。

![img](https://images2018.cnblogs.com/blog/485067/201808/485067-20180805224250622-955288547.png)

```
1. 客户端A无法进行P2P穿透，请求业务服务器要进行转发。
2. 业务服务器根据客户端A，请求类型，返回对应的转发服务器地址和对应的房间号RoomID/Token等信息
3. 上述请求类型，可以是请求自建RTMP流媒体服务，购买于云厂商RTMP流媒体服务或者自定义协议媒体转发服务
4. 客户端A得到业务服务器返回的媒体服务器地址和RoomID/Token
5. 通过信令服务器或者MQTT服务器，把对应的媒体服务器地址和RoomID/Token告诉另一端客户端B
6. 客户端A和客户端B同时进入相同房间Room，客户端A进行推流，客户端B进行拉流
7. 其他媒体信息，如编解码格式，清晰度，播放，暂停，拍照等命令，通过上述信令或MQTT服务器进行命令控制
```



# 一、 编译Nginx

　　RTMP流媒体服务器，现成的开源方案有很多，有SRS，Red5，wowoza，FMS等，我这里使用的是Nginx的rtmp插件实现实时流转发。

　　下载 nginx-rtmp-module https://github.com/arut/nginx-rtmp-module

　　重新编译nginx 

```bash
--prefix=/opt/nginx --with-stream --with-http_ssl_module --with-stream_ssl_module --with-debug --add-module=../nginx-rtmp-module
```

　　文末提供Windows版本已经编译好的exe下载

# 二、配置Nginx.conf

　　基本的nginx配置，这里就不进行介绍了，需要了解的可以参考我其他博客，里面有介绍。这里只介绍rtmp段的定义。

```yaml
rtmp{
    server{
        listen 8081;
        access_log logs/rtmp_access.log;
        on_connect http://127.0.0.1:8080/v1/rtmp/on_connect;
        application rtmp {
            live on;
            notify_method get;
            on_play http://127.0.0.1:8080/v1/rtmp/on_play;
            on_publish http://127.0.0.1:8080/v1/rtmp/on_publish;
            on_done http://127.0.0.1:8080/v1/rtmp/on_done;
            on_play_done http://127.0.0.1:8080/v1/rtmp/on_play_done;
            on_publish_done http://127.0.0.1:8080/v1/rtmp/on_publish_done;
            on_record_done http://127.0.0.1:8080/v1/rtmp/on_record_done;
            on_update http://127.0.0.1:8080/v1/rtmp/on_update;
            notify_update_timeout 10s;
        }
        application vod {
            play /opt/openresty/video;
        }
    }
}
```

# 三、HTTP异步通知回调

 　Nginx-rtmp-module插件实现了针对RTMP协议的一些命令做了事件通知。这里我通过一个简单的SpringBoot项目，快速搭建一个HTTP服务来接收RTMP的回调。

```java
package com.wunaozai.rtmp.notify.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value="/v1/rtmp/")
public class RTMPNotifyController {

    @GetMapping(value="/on_connect")
    public String onConnect(HttpServletRequest request){
        debug(request, "on_connect");
        return "on_connect";
    }
    @GetMapping(value="/on_play")
    public String onPlay(HttpServletRequest request){
        debug(request, "on_play");
        return "on_play";
    }
    @GetMapping(value="/on_publish")
    public String onPublish(HttpServletRequest request){
        debug(request, "on_publish");
        return "on_publish";
    }
    @GetMapping(value="/on_done")
    public String onDone(HttpServletRequest request){
        debug(request, "on_done");
        return "on_done";
    }
    @GetMapping(value="/on_play_done")
    public String onPlayDone(HttpServletRequest request){
        debug(request, "on_play_done");
        return "on_play_done";
    }
    @GetMapping(value="/on_publish_done")
    public String onPublishDone(HttpServletRequest request){
        debug(request, "on_publish_done");
        return "on_publish_done";
    }
    @GetMapping(value="/on_record_done")
    public String onRecordDone(HttpServletRequest request){
        debug(request, "on_record_done");
        return "on_record_done";
    }
    @GetMapping(value="/on_update")
    public String onUpdate(HttpServletRequest request){
        debug(request, "on_update");
        return "on_update";
    }

    private String debug(HttpServletRequest request, String action){
        String str = action + ": " + request.getRequestURI() + " " + request.getQueryString();
        System.out.println(str);
        return str;
    }
}
```



# 四、运行效果

　　(1) 启动nginx和SpringBoot

　　(2) 以下是SpringBoot打印信息(各位可以简单分析一下这些日志的)

```java
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178269841&call=connect
on_publish: /v1/rtmp/on_publish app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=541&call=publish&name=room&type=live
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=541&call=update_publish&time=10&timestamp=3999&name=room
on_done: /v1/rtmp/on_done app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=541&call=done&name=room
on_publish_done: /v1/rtmp/on_publish_done app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=541&call=publish_done&name=room
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178305623&call=connect
on_publish: /v1/rtmp/on_publish app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=publish&name=room&type=live
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=10&timestamp=7296&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=20&timestamp=17248&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=30&timestamp=27328&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=40&timestamp=37280&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=50&timestamp=47296&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=60&timestamp=57312&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=70&timestamp=67264&name=room
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178380351&call=connect
on_play: /v1/rtmp/on_play app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=557&call=play&name=room&start=4294966296&duration=0&reset=0&pass=12345
on_play_done: /v1/rtmp/on_play_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=557&call=play_done&name=room&pass=12345
on_done: /v1/rtmp/on_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=557&call=done&name=room&pass=12345
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=80&timestamp=77344&name=room
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178388202&call=connect
on_play: /v1/rtmp/on_play app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=563&call=play&name=room&start=4294966296&duration=0&reset=0&pass=12345
on_done: /v1/rtmp/on_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=563&call=done&name=room&pass=12345
on_play_done: /v1/rtmp/on_play_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=563&call=play_done&name=room&pass=12345
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=90&timestamp=87360&name=room
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178396146&call=connect
on_play: /v1/rtmp/on_play app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=569&call=play&name=room&start=4294966296&duration=0&reset=0&pass=12345
on_done: /v1/rtmp/on_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=569&call=done&name=room&pass=12345
on_play_done: /v1/rtmp/on_play_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=569&call=play_done&name=room&pass=12345
on_connect: /v1/rtmp/on_connect app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&epoch=178403666&call=connect
on_play: /v1/rtmp/on_play app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=574&call=play&name=room&start=4294966296&duration=0&reset=0&pass=12345
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=100&timestamp=97311&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=574&call=update_play&time=10&timestamp=105504&name=room&pass=12345
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=110&timestamp=107199&name=room
on_done: /v1/rtmp/on_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=574&call=done&name=room&pass=12345
on_play_done: /v1/rtmp/on_play_done app=rtmp&flashver=&swfurl=&tcurl=rtmp://rtmp.wunaozai.com:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=574&call=play_done&name=room&pass=12345
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=120&timestamp=117344&name=room
on_update: /v1/rtmp/on_update app=rtmp&flashver=FMLE/3.0%20(compatible%3B%20FMSc/1.0)&swfurl=&tcurl=rtmp://120.24.210.62:8081/rtmp&pageurl=&addr=113.74.127.195&clientid=547&call=update_publish&time=130&timestamp=122815&name=room
```



　　(3) 客户端进行推流，这里的推流软件，我是使用这个 http://www.iavcast.com/html/ruanjian/iavcast.html 

![img](https://images2018.cnblogs.com/blog/485067/201808/485067-20180805231915378-1626592119.png)

　　(4) 移动端，我使用微信小程序里的 腾讯视频云 这个小程序里面有RTMP测试

![img](https://images2018.cnblogs.com/blog/485067/201808/485067-20180805232149241-1055414099.png)

　　(5) nginx-rtmp 日志



```java
113.74.127.195 [05/Aug/2018:16:18:08 +0800] PUBLISH "rtmp" "room" "" - 2646572 687 "" "FMLE/3.0 (compatible; FMSc/1.0)" (1m 46s)
113.74.127.195 [05/Aug/2018:16:19:49 +0800] PLAY "rtmp" "room" "pass=12345" - 413 542 "" "" (4s)
113.74.127.195 [05/Aug/2018:16:19:57 +0800] PLAY "rtmp" "room" "pass=12345" - 413 542 "" "" (4s)
113.74.127.195 [05/Aug/2018:16:20:05 +0800] PLAY "rtmp" "room" "pass=12345" - 413 542 "" "" (4s)
113.74.127.195 [05/Aug/2018:16:20:13 +0800] PLAY "rtmp" "room" "pass=12345" - 413 542 "" "" (4s)
113.74.127.195 [05/Aug/2018:16:30:39 +0800] PLAY "rtmp" "room" "pass=12345" - 413 871 "" "" (4s)
113.74.127.195 [05/Aug/2018:16:30:54 +0800] PLAY "rtmp" "room" "pass=12345" - 413 647163 "" "" (12s)
113.74.127.195 [05/Aug/2018:16:31:08 +0800] PUBLISH "rtmp" "room" "" - 4961955 409 "" "FMLE/3.0 (compatible; FMSc/1.0)" (1m 30s)
113.74.127.195 [05/Aug/2018:23:06:47 +0800] PUBLISH "rtmp" "room" "" - 425763 529 "" "FMLE/3.0 (compatible; FMSc/1.0)" (13s)
113.74.127.195 [05/Aug/2018:23:08:29 +0800] PLAY "rtmp" "room" "pass=12345" - 413 871 "" "" (4s)
113.74.127.195 [05/Aug/2018:23:08:37 +0800] PLAY "rtmp" "room" "pass=12345" - 413 871 "" "" (4s)
113.74.127.195 [05/Aug/2018:23:08:45 +0800] PLAY "rtmp" "room" "pass=12345" - 413 871 "" "" (4s)
113.74.127.195 [05/Aug/2018:23:09:05 +0800] PLAY "rtmp" "room" "pass=12345" - 413 926026 "" "" (17s)
113.74.127.195 [05/Aug/2018:23:09:30 +0800] PUBLISH "rtmp" "room" "" - 7061016 409 "" "FMLE/3.0 (compatible; FMSc/1.0)" (2m 20s)
```



# 五、RTMP鉴权方式

　　 一般商用的话，为了防止被其他人使用和安全性考虑，所以需要对RTMP进行鉴权处理。鉴权如果有特殊性的，可以通过修改nginx-rtmp-module的源代码，然后进行修改，其实就是增加个auth函数，这个函数可以查询数据库之类的，然后决定返回0成功还是-1表示失败。

　　除了上面说到的方式，还可以通过简单的方式，就是上面提到的HTTP回调。如果HTTP回调返回的HTTP状态码是2xx的，表示成功。如果是返回5xx的状态码，那么表示失败。那样的话，服务器就是断开RTMP连接。

　　就是在rtmp://rtmp.wunaozai.com/rtmp_live/room?username=username&password=password

　　至于实现，这里暂时还没有，其实就是在SpringBoot项目中对每个请求，判断一下参数即可。如果后面有机会就详细写一下，关联Redis数据库，实现房间号功能。但是可能不会写了，因为实际上不难。就是整个流程跑通还是比较多代码要写的，在博客里贴太多代码有点不好。博客最主要的还是提供思路。实际实现就应该在项目中实现了。

# 六、其他

　　这里是一些配置说明和示例

## 6.1 rtmp应用

```yaml
Application 创建一个RTMP应用，这里有点区别于http的location
Timeout 60s
stocket超时，可以配合keepalive和ping值来实现不让服务器端长期处于监听连接客户端状态，实现快速关掉socket
Ping 3m
ping_timeout 30s
RTMP ping用于检查活动连接的协议。发送一个特殊的包远程连接，在ping_timeout指定时间内期待一个回复，如果没有收到回复，连接断开
max_streams 32
设置RTMP流的最大数目，单一流数据最大限制，一般默认的32就可以了
ack_window 5000000
设置RTMP窗口的大小
chunk_size 4096
数据块大小 设置值越大CPU负载就越小
max_queue
最大队列数，一般默认即可
max_message 1M
输入数据消息的最大大小。所有输入数据消息都会保存在内存中，等待完成流媒体转发。在理论上传入的消息可以是非常大，对服务器稳定性影响较大，所以一般默认即可。
out_queue
out_cork
Buflen 5s
设置默认缓冲区长度。通常客户端发送播放前RTMP set_buflen命令并重置该设置
```

## 6.2 访问控制

```yaml
访问控制
Access
Allow/deny
允许来自指定地址或者所有地址发布/播放
Allow public 127.0.0.1
Deny publish all;
Allow play 192.168.0.0/24
Deny play all;
```



## 6.3 Exec命令

```yaml
Exec命令
Exce
exec_options on;
启动一些exec指令选项，通过一些exec事件来干预整个RTMP流
可以仔细一些外部编解码功能
Exec ffmpeg -i rtmp://localhost?src/$name -vcodec libx264 -vprofile baseline -g 10 -s 300x200 -acodec libfaac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/$name 2>> /var/log/ffmpeg-$name.log;
Exce_statc
类似exce，属于静态命令，不支持传递上下文参数
Exec_kill_signal term;
Exec_kill_signal user1;
Exec_kill_signal 3;
Exec_pull
Exec_push
Exec_publish
指定与参数外部命令要在发布事件执行。
Exec_play
指定与要在打开事件执行外部命令
Exec_play_done
指定要在打开完成事件执行外部命令
Exec_publish_done
Exec_record_done
例子
exec_play bash -c “echo $addr $pageurl >> /tmp/clients”
Exec_publish base -c “echo $addr $flashver >> /tmp/publishers”
转录
Exec_record_done ffmpeg -y -i $path -acodec libmp31ame -ar 44100 -ac 1 -vcodec libx264 $dirname/$basename.mp4
```



## 6.4 Live模式

```yaml
Live 模式
Live on
切换直播模式，即一对多广播
Meta on/copy/off
奇幻发送元数据到客户端 默认on
Interleave on/off
切换交叉模式。在该模式下，音视频会在同一个RTMPchunk流中传输。默认为off
wait_key on|off
使视频流从一个关键帧开始，默认为off
wait_video on|off
在一个视频帧发送前禁用音频。默认off
通过wait_key/wait_video进行组合以使客户端可以收到具有所有其他数据的视频关键帧。但这样会增加连接延迟。不过可以通过编解码器中调整关键帧间隔来减少延迟。
Publish_notify on
发送NetStream.Publish.Start和NetStream.Publish.Stop给用户，默认off
Drop_idle_publisher 10s
终止指定时间内闲置（没有音频、视频）的发布连接，默认为off。注意这个仅仅对于发布模式的连接起作用（发送publish命令之后）
Sync 10ms
同步音视频流。如果用户带宽不足以接收发布率，服务器会丢弃一些帧。这将导致同步问题。当时间戳差超过sync指定值，将会发送一个绝对帧来解决这个问题，默认为300ms
Play_restart off
使nginx-rtmp能够在发布启动或者停止时发送NetStream.Play.Start 和 NetStrem.Play.Stop到每个用户。如果关闭的话，那么每个用户就只能在回放的开始结束时收到该通知了。默认为on
```

## 6.5 Record模式

```yaml
Record 模式
Record off|all|audio|video|keyframes|manual
切换录制模式，流可以被记录到flv文件
Off 不录制
All 录制音频和视频
Audio
Video
Keyframes 只录制关键视频帧
Manual 不自动启动录制，使用控制接口来进行启动和停止
Record_path /tmp/rec
指定录制的flv文件存放目录
Record_suffix -%d-%b-%y-%T.flv
录制后缀strftime格式
Record_unique on|off
是否添加时间戳到录制文件，防止文件被覆盖，默认off
record_append on|off
切换文件附加模式。开启后，录制时将新数据附加到旧文件后面。默认off
record_lock on|off
锁定文件，调用系统的fcntl
record_max_size 128K
设置录制文件的最大值
Record_max_frames 2
设置每个录制文件的视频帧最大数量
Record_interval 1s/15m
在这个指令指定的时间之后重启录制。默认off设置为0表示录制中无延迟。如果record_unique为off时所有的记录都会被写到同一个文件中。否则就会以时间戳区分在不同文件
Record_notify on|off
奇幻当定义录制启动或者停止文件时发送NetStream.Record.Start和NetStream.Record.Stop状态信息onStatus到发布者。
```



## 6.6 应用

```yaml
应用
Application rtmp{
Live on;
Record all;
Record_path /var/rec;

Recorder audio{
Record audio;
Record_suffix .audio.flv;
}
Recorder chunked{
Record all;
Record_interval 15s;
Record_path /var/rec/chunked;
}
}
创建录制块。可以在单个application中创建多个记录 。
```

## 6.7 VOD媒体

```yaml
VOD媒体
Play dir|http://loc
播放指定目录或者HTTP地址的flv或者mp4文件。注意HTTP播放是要在整个文件下载完后才开始播放。同一个play可以播放多个视频地址(用于负载)。MP4格式要在编解码都被RTMP支持才可以播放。一般常见的就是H264/AAC
Application vod{
Play /var/flvs;
}
Application vod_http{
Play http://localhost/vod;
}
Play_temp_path /www
设置远程VOD文件完全下载之后复制于play_temp_path之后的路径。空值的话禁用此功能。
Play_local_path dir
在播放前设置远程存储VOD文件路径，默认/tmp
Play_local_path /tmp/videos;
Paly /tmp/videos http://localhost/videos
表示播放视频，先播放本地缓存，如果没有的话，从localhost/videos下载到本地/tmp/videos后，在进行播放
```

## 6.8 Reply模式

```yaml
Relay模式
Pull url [key=value]
创建pull中继。主要是从远程服务器拉取流媒体。并进行重新发布。
Url语法 [rtmp://]host[:port][/app[/playpath]] 如果application找不到那么将会使用本地application名，如果找不到playpath那么久用当前流名称。
参数如下(使用Key=Value方式)
app 明确application名
Name 捆绑到relay的bending流名称。如果为空，那么会使用application中所有本地流
tcUrl
pageUrl
swfUrl
flashVer
playPath
Live
Start
Stop
Static
Pull rtmp://cdn.example.com/main/ch?id=1234 name=channel;
Push url [key=value]
与pull类似，只是push推送发布流到远程服务器。
Push_reconnect 1s
在断开连接后，在push重新连接钱等待的时间，默认3秒
Session_relay on;
切换会话relay模式。在这种情况下关闭时relay销毁。
```

## 6.9 Notify模式

```yaml
Notify 模式
这个功能主要是提供HTTP回调。当发送一些连接操作是，一个HTTP请求异步发送。命令处理会被暂停挂起，知道它返回结果代码。当HTTP返回2xx成功状态码时，RTMP会话继续。3xx状态码会使RTMP重定向到另一个从HTTP返回头获取到的application，否则连接丢失。其他状态码，连接断开。目前用来做简单的鉴权。
On_connect url
设置HTTP连接回调。当客户分发连接命令时。
例子：
On_connect http://localhost/my_auth;
Location /on_connect{
If($arg_flashver != “my_secret_flashver”){
Rewrite ^.*$ fallback?permanent;
}
}
On_play url
设置HTTP播放回调。分发客户分发播放命令时。
http {
Location /redirect {
Rewrite ^.*$ newname?permanent;
}
}
Rtmp{
Application myqpp{
Live on;
On_play http://localhost/redirect;
}
}
On_publish
On_doone
On_play_done
On_publish_done
On_record_done
On_update
Notify_update_timeout
设置on_update回调时间
Notify_update_strict on|off
Notify_relay_redirect on
Notify_method get
设置HTTP方法通知，默认是application/x-www-form-urlencodeed 的POST内容类型。有时候可能会需要GET方法，在nginx的http{}部分处理调用。在这种情况下可以使用arg_*变量去访问参数。
例如如果是method为get时
Location /on_play{
If($arg_pageUrl ~* localhost){
Return 200;
}
Return 500;
}
```



## 6.10 HLS模式

```yaml
HLS 模式
Hls on|off
使application 切换HLS协议直播
Hls_path /tmp/hls;
设置HLS播放列表和分段目录。这一目录必须在nginx启动前就已经存在。
Hls_fragment 15s;
设置HLS分段长度，默认5秒，这个跟直播延迟有比较大的影响
Hls_playlist_length 20m;
设置HLS播放列表长度，默认30秒。这个跟直播缓存有关。
Hls_sync time
设置HLS时间戳同步阈值。默认2ms。这个功能防止由低分辨率RTMP(1KHz)转换到高分辨率MPEG-TS(90KHz)之后出现的噪音。
Hls_continuous on|off
切换HLS连续模式，默认off。
Hls_nested on|off
切换HLS嵌套模式。默认off。
Hls_cleanup on|off;
切换HLS清理。默认on
```

## 6.11 AccessLog日志

```yaml
AccessLog日志
Access_log off|path [format_name]
Log_format new_format ‘$remote_addr’;
Access_log logs/rtmp_access.log new_format;
Log_format 指定日志格式
创建指定的日志格式。日志格式看起来很像 nginx HTTP 日志格式。日志格式里支持的几个变量有：
* connection - 连接数。
* remote_addr - 客户端地址。
* app - application 名。
* name - 上一个流名。
* args - 上一个流播放/发布参数。
* flashver - 客户端 flash 版本。
* swfurl - 客户端 swf url。
* tcurl - 客户端 tcUrl。
* pageurl - 客户端页面 url。
* command - 客户端发送的播放/发布命令：NONE、PLAY、PUBLISH、PLAY+PUBLISH。
* bytes_sent - 发送到客户端的字节数。
* bytes_received - 从客户端接收到的字节数。
* time_local - 客户端连接结束的本地时间。
* session_time - 持续连接的秒数。
* session_readable_time - 在可读格式下的持续时间。
默认的日志格式叫做 combined。这里是这一格式的定义：
$remote_addr [$time_local] $command "$app" "$name" "$args" -
$bytes_received $bytes_sent "$pageurl" "$flashver" ($session_readable_time)
```

## 6.12 Limits限制

```yaml
Limits限制
max_connections number;
设置rtmp引擎最大连接数，默认off
```

## 6.13 hls应用

```yaml
Application hls{
Live on;
Hls on;
Hls_path /tmp/hls;
Hls_fragment 15s;
}
```



 　由于开发是在Windows上完成的，所以需要有Windows环境，这样测试起来比较方便。发现一个很好的功能： https://github.com/illuspas/nginx-rtmp-win32

 

参考资料

　　https://github.com/arut/nginx-rtmp-module

　　https://blog.csdn.net/cui918/article/details/53540397

　　https://www.cnblogs.com/zx-admin/p/5783523.html

　　https://www.cnblogs.com/lidabo/p/7099501.html

　　https://blog.csdn.net/wei389083222/article/details/78721074

Nginx-RTMP Windows二进制下载: https://files.cnblogs.com/files/wunaozai/nginx-rtmp-win32-master.rar