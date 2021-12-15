## 1、url的组成部分

以`rtsp://somedomain.com:554/live/0?token=abcdefg&field=value`为例,该url分为以下几个部分：

- `协议(scheam)` : rtsp协议,默认端口554
- `拟主机(vhost)` : somedomain.com,该字段既可以是域名也可以是ip，如果是ip则对应的虚拟主机为`__defaultVhost__`
- `服务端口号(port)` : 554,如果不指定端口号，则使用协议默认端口号
- `应用名(app)` : live
- `流ID(streamid)` : 0
- `参数(args)` : token=abcdefg&field=value

## 2、ZLMediaKit中的流媒体源

在ZLMediaKit中，流媒体源是一种可以被用于直播转发、推流转发等功能的数据对象，在本项目中被称作为`MediaSource`，目前支持两种类型的流媒体源，分别是`RtspMediaSource`以及`RtmpMediaSource`。

定位一个流媒体源，主要通过4个元素(我们后续称其为4元组)，分别是:

- `协议(scheam)`
- `虚拟主机(vhost)`
- `应用名(app)`
- `流ID(streamid)`

## 3、流媒体源对应的播放url

假定有一个`RtspMediaSource`，它的4元组分别为 `rtsp(RtspMediaSource固定为rtsp)`、`somedomain.com`、`live`、`0` 那么播放这个流媒体源的url对应为:

- `rtsp://somedomain.com/live/0`
- `rtsps://somedomain.com/live/0`
- `rtsp://127.0.0.1/live/0?vhost=somedomain.com`
- `rtsps://127.0.0.1/live/0?vhost=somedomain.com`

如果有一个`RtmpMediaSource`，它的4元组分别为 `rtmp(RtmpMediaSource固定为rtmp)`、`somedomain.com`、`live`、`0` 那么播放这个流媒体源的url对应为:

- `rtmp://somedomain.com/live/0`
- `rtmps://somedomain.com/live/0`
- `rtmp://127.0.0.1/live/0?vhost=somedomain.com`
- `rtmps://127.0.0.1/live/0?vhost=somedomain.com`

rtmp类型的流媒体源也支持`http-flv`、`websocket`直播，对应的url如下：

- `http://somedomain.com/live/0.flv`
- `https://somedomain.com/live/0.flv`
- `http://127.0.0.1/live/0.flv?vhost=somedomain.com`
- `https://127.0.0.1/live/0.flv?vhost=somedomain.com`
- `ws://somedomain.com/live/0.flv`
- `wss://somedomain.com/live/0.flv`
- `ws://127.0.0.1/live/0.flv?vhost=somedomain.com`
- `wss://127.0.0.1/live/0.flv?vhost=somedomain.com`

当然，ZLMediaKit一般会把rtsp、rtmp流媒体源互相转换，也会转换成hls/http-ts/ws-ts/http-fmp4/ws-fmp4，播放的url如下：

- HLS
  - `http://somedomain.com/live/0/hls.m3u8`
  - `https://somedomain.com/live/0/hls.m3u8`
  - `http://127.0.0.1/live/0/hls.m3u8?vhost=somedomain.com`
  - `https://127.0.0.1/live/0/hls.m3u8?vhost=somedomain.com`
- HTTP-TS/WS-TS(后缀为.live.ts,目的是为了解决与hls的冲突)
  - `http://somedomain.com/live/0.live.ts`
  - `https://somedomain.com/live/0.live.ts`
  - `http://127.0.0.1/live/0.live.ts?vhost=somedomain.com`
  - `https://127.0.0.1/live/0.live.ts?vhost=somedomain.com`
  - `ws://somedomain.com/live/0.live.ts`
  - `wss://somedomain.com/live/0.live.ts`
  - `ws://127.0.0.1/live/0.live.ts?vhost=somedomain.com`
  - `wss://127.0.0.1/live/0.live.ts?vhost=somedomain.com`
- HTTP-fMP4/WS-fMP4(后缀为.live.mp4,目的是为了解决与mp4点播的冲突)
  - `http://somedomain.com/live/0.live.mp4`
  - `https://somedomain.com/live/0.live.mp4`
  - `http://127.0.0.1/live/0.live.mp4?vhost=somedomain.com`
  - `https://127.0.0.1/live/0.live.mp4?vhost=somedomain.com`
  - `ws://somedomain.com/live/0.live.mp4`
  - `wss://somedomain.com/live/0.live.mp4`
  - `ws://127.0.0.1/live/0.live.mp4?vhost=somedomain.com`
  - `wss://127.0.0.1/live/0.live.mp4?vhost=somedomain.com`

一般而言，上述url在ZLMediaKit都有效，因为ZLMediaKit默认转换流媒体源。

## 4、点播url

ZLMediaKit的点播一般通过mp4文件来实现，推荐大家使用http mp4点播，这样是最简单，服务器也无需解复用mp4文件，当然ZLMediaKit目前也支持rtsp、rtmp、http-flv、websocket-flv的mp4点播， 对应的url跟直播url类似，不在赘述，这里只介绍区别。

- ZLMediaKit对点播限制应用名，默认为`record`

- 假如一个mp4文件放置在http根目录record文件夹(

  ```
  www/record
  ```

  )下，他的相对路径为:

  ```
  www/record/0.mp4
  ```

  ,那么点播url则为:

  - `rtsp://somedomain.com/record/0.mp4`
  - `rtmp://somedomain.com/record/0.mp4`
  - `http://somedomain.com/record/0.mp4`(这里是通用的http文件点播，服务器不用解复用文件)
  - `http://somedomain.com/record/0.mp4.flv`（这里是http-flv直播，不是http点播，服务器需要解复用文件）
  - `ws://somedomain.com/record/0.mp4.flv`
  - `http://somedomain.com/record/0.mp4.live.ts`（这里是http-ts直播，不是http点播，服务器需要解复用文件）
  - `ws://somedomain.com/record/0.mp4.live.ts`
  - `http://somedomain.com/record/0.mp4.live.mp4`（这里是http-fmp4直播，不是http点播，服务器需要解复用文件）
  - `ws://somedomain.com/record/0.mp4.live.mp4`

- 如果开启了虚拟主机，那么点播文件需要放置在 `www/somedomain.com/record/0.mp4`。

## 5、url参数

ZLMediaKit会识别url中问号后面的字符串为url参数，其格式跟http一致，其中参数`vhost`是ZLMediaKit内置支持的参数，支持指定vhost。 url参数主要用于播放、推流鉴权，在触发hook api时，会把这些参数提交给第三方业务服务器