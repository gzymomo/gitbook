- [Node+OBS直播服务器搭建总结](https://juejin.cn/post/6999177199030894599)

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ec13d58a02c485b85bc85f0b151dc26~tplv-k3u1fbpfcp-zoom-crop-mark:1304:1304:1304:734.image)

## 直播流媒体协议

先来了解一下基本的直播流媒体协议。

http-flv,rtpm

| 协议/特点 | 开发者                           | 原理                                                         | 优点                                                         | 缺点                                                   |
| --------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| http-flv  | Abode                            | 通过服务器把flv下载到本地缓存，然后通过NetConnection本地连接播放 | 节省服务器消耗                                               | 保密性差                                               |
| rtmp      | Abode                            | 通过NetConnection连接到服务器，实时播放服务器的flv           | 保密性好                                                     | 消耗服务器资源                                         |
| rtsp      | 哥伦比亚大学、网景和RealNetworks | 控制具有实时特性的数据的发送，依赖传输协议                   | 实时效果非常好                                               | 实现复杂                                               |
| hls       | 苹果公司                         | 包括一个m3u(8)的索引文件，TS媒体分片文件和key加密串文件，不将TS切片文件存到磁盘，而是存在内存当中 | 极大减少了磁盘的I/O次数，延长了服务器磁盘的使用寿命，极大提高了服务器运行的稳定性 | 会生成大量的文件，存储或处理这些文件会造成大量资源浪费 |

## 拉流与推流

**推流**，指的是把采集阶段封包好的内容传输到服务器的过程。

**拉流**, 指服务器已有直播内容，用指定地址进行拉取的过程。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/031d9f95fc8f4d4ca8351ff4993a620d~tplv-k3u1fbpfcp-watermark.image)

## Node服务搭建

- 安装依赖包

这次使用`node-media-server`包，来搭建，获取[更多请访问](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fnode-media-server)。

```sh
mkdir live
cd live
npm init -y
npm i node-media-server
复制代码
```

引入包，编写配置文件

```js
// server.js
const nodeMediaServer = require('node-media-server');
const config = {
    rtmp: {
        port: 3001,
        chunk_size: 6000,
        gop_cache: true,
        ping: 30,
        ping_timeout: 60
    },
    http: {
        port: 3002,
        allow_origin: "*"
    }
}

const nms = new nodeMediaServer(config);

nms.run();
复制代码
```

启动以后会输入一下内容

```cmd
D:\live>node server.js
2021/8/22 14:52:19 9588 [INFO] Node Media Server v2.3.8
2021/8/22 14:52:19 9588 [INFO] Node Media Rtmp Server started on port: 3001
2021/8/22 14:52:19 9588 [INFO] Node Media Http Server started on port: 3002
2021/8/22 14:52:19 9588 [INFO] Node Media WebSocket Server started on port: 3002
复制代码
```

如果打印出以上内容，说明一个rtmp的直播服务器就已经搭建成功了。

- 拉推流地址

AppName就是App名称;StreamName就是流名称。

推流地址：

url: `rtmp://localhost/live` key: STREAM_NAME

拉流地址:

*rtmp*: `rtmp://localhost:port/live/STREAM_NAME`
 *http-flv*: `http://localhost:3002/live/STREAM_NAME.flv`
 *HLS*: `http://localhost:3002/live/STREAM_NAME/index.m3u8`
 *DASH*: `http://localhost:3002/live/STREAM_NAME/index.mpd`
 *websocket-flv*: `ws://localhost:3002/live/STREAM_NAME.flv`

这里主要使用的推流地址是：`rtmp://localhost/xqlive/demo`,拉流地址是`http://localhost:3002/xqlive/demo.flv`。

## 前端播放页面

这里主要是使用[flv.js](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FBilibili%2Fflv.js)进行播放。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>云直播</title>
    <style>
        #live {
            margin: 0 auto;
            display: block;
            min-width: 300px;
            max-width: 600px;
            width: 100%;
        }
    </style>
</head>

<body>
    <video id="live" playsinline controls src="" poster="./img/poster.jpg"></video>
    <script src="js/flv.min.js"></script>
    <script>
        if (flvjs.isSupported()) {
            let ve = document.getElementById('live');
            let flvPlayer = flvjs.createPlayer({
                type: 'flv',
                url: 'http://localhost:3002/xqlive/demo.flv'
            });
            flvPlayer.attachMediaElement(ve);
            flvPlayer.load();
            flvPlayer.play();
        }
    </script>
</body>

</html>
复制代码
```

看一下效果

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/054547c11a484da2924b22dc69a47bea~tplv-k3u1fbpfcp-watermark.image)

## OBS推流配置

这里使用OBS进行推流直播。

[OBS下载地址](https://link.juejin.cn?target=https%3A%2F%2Fobsproject.com%2F)

下载好后安装然后打开主页面，找到文件=》设置=》推流

然后填写好地址与密钥就好了。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1fb0db8abcd4288a1bb04a28366b472~tplv-k3u1fbpfcp-watermark.image)

接着选择媒体源开始推流。

- 推流界面

下面是我选择的一段小视频进行推流直播。

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be4622f4e60a4ab8b392911b13e9cf8c~tplv-k3u1fbpfcp-watermark.image)

- 播放界面

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbcb38db9c334ddd9cfb13fb4e24c8c7~tplv-k3u1fbpfcp-watermark.image)

除了媒体源，你还可以选择直播显示器桌面，直播文字，直播图片，以及开启摄像头直播你自己，都可以。

如果你要部署到线上的话，要保障你的服务器带宽至少在10MB左右，不然就会很卡的。