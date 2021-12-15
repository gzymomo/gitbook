[ZLMediaKit支持的事件http api](https://www.jianshu.com/p/72a4fe770de7)



# ZLMediaKit支持的Http API

ZLMediaKit可以把内部的一些事件通过调用第三方http服务器api的方式通知出去，以下是相关的默认配置：



```ini
[hook]
enable=1
admin_params=secret=035c73f7-bb6b-4889-a715-d9eb2d1925cc
timeoutSec=10

on_flow_report=https://127.0.0.1/index/hook/on_flow_report
on_http_access=https://127.0.0.1/index/hook/on_http_access
on_play=https://127.0.0.1/index/hook/on_play
on_publish=https://127.0.0.1/index/hook/on_publish
on_record_mp4=https://127.0.0.1/index/hook/on_record_mp4
on_rtsp_auth=https://127.0.0.1/index/hook/on_rtsp_auth
on_rtsp_realm=https://127.0.0.1/index/hook/on_rtsp_realm
on_shell_login=https://127.0.0.1/index/hook/on_shell_login
on_stream_changed=https://127.0.0.1/index/hook/on_stream_changed
on_stream_none_reader=https://127.0.0.1/index/hook/on_stream_none_reader
on_stream_not_found=https://127.0.0.1/index/hook/on_stream_not_found
```

- **enable** :

  是否开启http hook，如果选择关闭，ZLMediaKit将采取默认动作(例如不鉴权等)

- **timeoutSec**：

  事件触发http客户端超时时间。

- **admin_params**：

  超级管理员的url参数，如果访问者参数与此一致，那么rtsp/rtmp/hls/http-flv播放或推流将无需鉴权。该选项用于开发者调试用。

- **on_flow_report**：

  流量统计事件，播放器或推流器断开时并且耗用流量超过特定阈值时会触发此事件，阈值通过配置文件general.flowThreshold配置。

- **on_http_access**：

  访问http文件服务器上hls之外的文件时触发

- **on_play**：

  播放器鉴权事件，rtsp/rtmp/http-flv/hls的播放都将触发此鉴权事件。

- **on_publish**：

  rtsp/rtmp推流鉴权事件。

- **on_record_mp4**:

  录制mp4完成后通知事件。

- **on_rtsp_auth**：

  rtsp专用的鉴权事件，先触发on_rtsp_realm事件然后才会触发on_rtsp_auth事件。

- **on_rtsp_realm**：

  该rtsp流是否开启rtsp专用方式的鉴权事件，开启后才会触发on_rtsp_auth事件。

  需要指出的是rtsp也支持url参数鉴权，它支持两种方式鉴权。

- **on_shell_login**：

  shell登录鉴权，ZLMediaKit提供简单的telnet调试方式

- **on_stream_changed**:

  rtsp/rtmp流注册或注销时触发此事件。

- **on_stream_none_reader**：

  流无人观看时事件，用户可以通过此事件选择是否关闭无人看的流。

- **on_stream_not_found**：

  流未找到事件，用户可以在此事件触发时，立即去拉流，这样可以实现按需拉流。

# API预览

## API预览

MediaServer是ZLMediaKit的主进程，目前支持以下http api接口，这些接口全部支持GET/POST方式，

```
      "/index/api/addFFmpegSource",
      "/index/api/addStreamProxy",
      "/index/api/close_stream",
      "/index/api/close_streams",
      "/index/api/delFFmpegSource",
      "/index/api/delStreamProxy",
      "/index/api/getAllSession",
      "/index/api/getApiList",
      "/index/api/getMediaList",
      "/index/api/getServerConfig",
      "/index/api/getThreadsLoad",
      "/index/api/getWorkThreadsLoad",
      "/index/api/kick_session",
      "/index/api/kick_sessions",
      "/index/api/restartServer",
      "/index/api/setServerConfig",
      "/index/api/isMediaOnline",
      "/index/api/getMediaInfo",
      "/index/api/getRtpInfo",
      "/index/api/getMp4RecordFile",
      "/index/api/startRecord",
      "/index/api/stopRecord",
      "/index/api/getRecordStatus",
      "/index/api/getSnap",
      "/index/api/openRtpServer",
      "/index/api/closeRtpServer",
      "/index/api/listRtpServer",
      "/index/api/startSendRtp",
      "/index/api/stopSendRtp"
```

其中POST方式，参数既可以使用urlencoded方式也可以使用json方式。 操作这些api一般需要提供secret参数以便鉴权，如果操作ip是127.0.0.1，那么可以无需鉴权。

## API返回结果约定

- HTTP层面统一返回200状态码，body统一为json。
- body一般为以下样式：

```
{
    "code" : -1,
    "msg" : "失败提示"
}
```

- code值代表执行结果，目前包含以下类型：

```
typedef enum {
    Exception = -400,//代码抛异常
    InvalidArgs = -300,//参数不合法
    SqlFailed = -200,//sql执行失败
    AuthFailed = -100,//鉴权失败
    OtherFailed = -1,//业务代码执行失败，
    Success = 0//执行成功
} ApiErr;
```

- 如果执行成功，那么`code == 0`,并且一般无`msg`字段。
- `code == -1`时代表业务代码执行不成功，更细的原因一般提供`result`字段，例如以下：

```
{
    "code" : -1, # 代表业务代码执行失败
    "msg" : "can not find the stream", # 失败提示
    "result" : -2 # 业务代码执行失败具体原因
}
```

- 开发者一般只要关注`code`字段和`msg`字段，如果`code != 0`时，打印显示`msg`字段即可。
- `code == 0`时代表完全成功，如果有数据返回，一般提供`data`字段返回数据。
