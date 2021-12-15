[surging 基于流媒体服务如何集群分流](https://www.cnblogs.com/fanliang11/p/14749259.html)

# 微服务特点

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210509224537675-793749450.png)

 

 

6个典型特点

1.高内聚，低耦合：程序模块的可重用性、移植性大大增强，针对于接口和业务模块的不同部署就能做到本地和远程调用；

2.单一职责：只负责某一业务功能；

3.独立：每个服务可以独立的开发、测试、构建、部署；

4.灵活性：小而灵活可以满足开发不同类型的业务模块。

5.进程隔离：可以通过隔离，发生熔断，雪崩而不会影响其它服务调用和运行

6.多样性： 针对于协议的扩展可以支持多样化业务场景的解决方案。

# 部件和功能

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210509225956451-67661614.png)

通过上图发现，surging 可以支持以下功能：

1.可以支持web,移动端,物联网和流媒体等业务场景

2.可以支持rtmp ,http-flv,thrift,mqtt,ws,http,grpc 多种协议以满足不同业务的场景的需要

3,可以支持服务之间调用的链路追踪。

4.可以支持扫描模块引擎，cli 工具，任务调度服务，日志，swagger 文档，服务注册，服务发现，协议主机，缓存中间件，事件总线等功能。

# 实例

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210510041615931-638617077.png)

 

 

 

下面以上图流媒体集群分流的例子，看看是如何研发运行的。

 以部署两台服务提供者为例，服务A，通过下载[surging](https://github.com/microsurging/SurgingVista)企业版（非商业用户不能用于商业，只能用于学习），通过以下配置来修改surgingSettings.json文件

```json
 {
 "Surging": {
 
    "Port": "${Surging_Server_Port}|81",
     
    "Ports": {
      "HttpPort": "${HttpPort}|28",
      "WSPort": "${WSPort}|96",
      "MQTTPort": "${MQTTPort}|97",
      "GrpcPort": "${GrpcPort}|95"
    }，
  "LiveStream": {
    "RtmpPort": 77, //rtmp 端口
    "HttpFlvPort": 8081, //HttpFlv 端口
    "EnableLog": true, //是否启用log
    "EnableHttpFlv": true,
    "RouteTemplate": "live1", //直播服务路由规则名称，可以根据规则设置，比如集群节点2，可以设置live2, 集群节点3，可以设置live3
    "ClusterNode": 2 //集群节点数里，会根据routetemplate 转推流
  }
}
```

 

服务B，通过以下配置来修改surgingSettings.json文件

```json
 {
 "Surging": {
 
    "Port": "${Surging_Server_Port}|82",
     
    "Ports": {
      "HttpPort": "${HttpPort}|281",
      "WSPort": "${WSPort}|961",
      "MQTTPort": "${MQTTPort}|971",
      "GrpcPort": "${GrpcPort}|951"
    }，
  "LiveStream": {
    "RtmpPort": 76, //rtmp 端口
    "HttpFlvPort": 8080, //HttpFlv 端口
    "EnableLog": true, //是否启用log
    "EnableHttpFlv": true,
    "RouteTemplate": "live1", //直播服务路由规则名称，可以根据规则设置，比如集群节点2，可以设置live2, 集群节点3，可以设置live3
    "ClusterNode": 2 //集群节点数里，会根据routetemplate 转推流
  }
}
```

然后可以通过`ffmpeg或者obs进行推流，以ffmpeg 工具为例，可以输入以下命令`

```
ffmpeg -re -i D:/大H包.HDTC1280高清国语中字版.mp4 -c copy -f flv rtmp://127.0.0.1:76/live1/livestream2
```

通过以下代码我们创建httpflv 的html 文件，url配置为：http://127.0.0.1:8080/live1/livestream2 

```html
<!DOCTYPE html>
<html>
 
<head>
    <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
    <title>flv.js demo</title>
 
    <style>
        .mainContainer {
            display: block;
            width: 1024px;
            margin-left: auto;
            margin-right: auto;
        }
 
        .urlInput {
            display: block;
            width: 100%;
            margin-left: auto;
            margin-right: auto;
            margin-top: 8px;
            margin-bottom: 8px;
        }
 
        .centeredVideo {
            display: block;
            width: 100%;
            height: 576px;
            margin-left: auto;
            margin-right: auto;
            margin-bottom: auto;
        }
 
        .controls {
            display: block;
            width: 100%;
            text-align: left;
            margin-left: auto;
            margin-right: auto;
            margin-top: 8px;
            margin-bottom: 10px;
        }
 
        .logcatBox {
            border-color: #CCCCCC;
            font-size: 11px;
            font-family: Menlo, Consolas, monospace;
            display: block;
            width: 100%;
            text-align: left;
            margin-left: auto;
            margin-right: auto;
        }
    </style>
</head>
 
<body>
     
    <div class="mainContainer">
        <video name="videoElement" class="centeredVideo" id="videoElement" controls width="1024" height="576" autoplay>
            Your browser is too old which doesn't support HTML5 video.
        </video>
 
    </div>
 
    <script src="https://cdn.bootcdn.net/ajax/libs/flv.js/1.5.0/flv.min.js"></script>
     
    <script>
        (function () {
            if (flvjs.isSupported()) {
                startVideo()
            }
 
            function startVideo() {
                var videoElement = document.getElementById('videoElement');
                var flvPlayer = flvjs.createPlayer({
                    type: 'flv',
                    isLive: true,
                    hasAudio: true,
                    hasVideo: true,
                    enableStashBuffer: false,
                    url: 'http://127.0.0.1:8080/live1/livestream2'
                });
                flvPlayer.attachMediaElement(videoElement);
                flvPlayer.load();
                window.setTimeout(function () { flvPlayer.play(); }, 500);
 
            }
 
            videoElement.addEventListener('click', function () {
                alert('是否支持点播视频：' + flvjs.getFeatureList().mseFlvPlayback + ' 是否支持httpflv直播流：' + flvjs.getFeatureList().mseLiveFlvPlayback)
            })
 
            function destoryVideo() {
                flvPlayer.pause();
                flvPlayer.unload();
                flvPlayer.detachMediaElement();
                flvPlayer.destroy();
                flvPlayer = null;
            }
 
            function reloadVideo() {
                destoryVideo()
                startVideo()
            }
        }) (document)
    </script>
     
</body>
 
</html>
```

然后就可以通过打开创建的html文件，运行效果如下图

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210509233612417-1042942011.png)

 

 通过potplay 配置服务A,链接为：rtmp://127.0.0.1:77/live1/livestream2，然后如下图所示：

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210509233921675-1021772211.png)

 

 可以通过potplay 配置服务B,链接为：rtmp://127.0.0.1:76/live1/livestream2，然后如下图所示：

![img](https://img2020.cnblogs.com/blog/192878/202105/192878-20210509234012825-752379305.png)