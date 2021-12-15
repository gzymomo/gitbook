## HOOK预览

MediaServer可以把内部的一些事件通过http post 第三方http服务器的方式通知出去，以下是相关的默认配置：

```
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
on_server_started=https://127.0.0.1/index/hook/on_server_started
```

如果是鉴权事件，且访问IP是`127.0.0.1`或者鉴权url参数与`admin_params`一致，那么会直接鉴权成功(不会触发鉴权web hook)。

大家也可以参考此[issue](https://github.com/zlmediakit/ZLMediaKit/issues/71)

## 详解

### 1、enable :

- 解释:

  是否开启http hook，如果选择关闭，ZLMediaKit将采取默认动作(例如不鉴权等)

### 2、timeoutSec：

- 解释:

  事件触发http post超时时间。

### 3、admin_params：

- 解释:

  超级管理员的url参数，如果访问者url参数与此一致，那么rtsp/rtmp/hls/http-flv/ws-flv播放或推流将无需鉴权。该选项用于开发者调试用。

### 4、on_flow_report：

- 解释:

  流量统计事件，播放器或推流器断开时并且耗用流量超过特定阈值时会触发此事件，阈值通过配置文件general.flowThreshold配置；此事件对回复不敏感。

- 触发请求：

  ```
  POST /index/hook/on_flow_report HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 298
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "duration" : 6,
     "params" : "token=1677193e-1244-49f2-8868-13b3fcc31b17",
     "player" : false,
     "schema" : "rtmp",
     "stream" : "obs",
     "totalBytes" : 1508161,
     "vhost" : "__defaultVhost__",
     "ip" : "192.168.0.21",
     "port" : 55345,
     "id" : "140259799100960"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型 | 参数解释                                 |
  | --------------- | -------- | ---------------------------------------- |
  | `app`           | `string` | 流应用名                                 |
  | `duration`      | `int`    | tcp链接维持时间，单位秒                  |
  | `params`        | `string` | 推流或播放url参数                        |
  | `player`        | `bool`   | true为播放器，false为推流器              |
  | `schema`        | `string` | 播放或推流的协议，可能是rtsp、rtmp、http |
  | `stream`        | `string` | 流ID                                     |
  | `totalBytes`    | `int`    | 耗费上下行流量总和，单位字节             |
  | `vhost`         | `string` | 流虚拟主机                               |
  | `ip`            | `string` | 客户端ip                                 |
  | `port`          | `int`    | 客户端端口号                             |
  | `id`            | `string` | TCP链接唯一ID                            |
  | `mediaServerId` | `string` | 服务器id,通过配置文件设置                |

- 默认回复：

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 40
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 07:09:32 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success"
  }
  ```

### 5、on_http_access：

- 解释:

  访问http文件服务器上hls之外的文件时触发。

- 触发请求：

  ```
  POST /index/hook/on_http_access HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 583
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "header.Accept" : "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
     "header.Accept-Encoding" : "gzip, deflate",
     "header.Accept-Language" : "en-US,en;q=0.5",
     "header.Cache-Control" : "max-age=0",
     "header.Connection" : "keep-alive",
     "header.Host" : "10.0.17.132",
     "header.Upgrade-Insecure-Requests" : "1",
     "header.User-Agent" : "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0",
     "id" : "140259799100960",
     "ip" : "10.0.17.132",
     "is_dir" : true,
     "params" : "",
     "path" : "/live/",
     "port" : 65073
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                    |
  | --------------- | ---------------- | --------------------------- |
  | `header.*`      | `string`         | http客户端请求header        |
  | `id`            | `string`         | TCP链接唯一ID               |
  | `ip`            | `string`         | http客户端ip                |
  | `is_dir`        | `bool`           | http 访问路径是文件还是目录 |
  | `params`        | `string`         | http url参数                |
  | `path`          | `string`         | 请求访问的文件或目录        |
  | `port`          | `unsigned short` | http客户端端口号            |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置   |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 68
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 07:27:01 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "err" : "",
     "path" : "",
     "second" : 600
  }
  ```

- 回复参数详解：

  | 参数名          | 参数类型 | 参数解释                                                     |
  | --------------- | -------- | ------------------------------------------------------------ |
  | `code`          | `int`    | 请固定返回0                                                  |
  | `err`           | `string` | 不允许访问的错误提示，允许访问请置空                         |
  | `path`          | `string` | 该客户端能访问或被禁止的顶端目录，如果为空字符串，则表述为当前目录 |
  | `second`        | `int`    | 本次授权结果的有效期，单位秒                                 |
  | `mediaServerId` | `string` | 服务器id,通过配置文件设置                                    |

### 6、on_play：

- 解释:

  播放器鉴权事件，rtsp/rtmp/http-flv/ws-flv/hls的播放都将触发此鉴权事件； 如果流不存在，那么先触发on_play事件然后触发on_stream_not_found事件。 播放rtsp流时，如果该流启动了rtsp专属鉴权(on_rtsp_realm)那么将不再触发on_play事件。

- 触发请求：

  ```
  POST /index/hook/on_play HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 189
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "id" : "140574554588960",
     "ip" : "10.0.17.132",
     "params" : "",
     "port" : 65217,
     "schema" : "rtmp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                           |
  | --------------- | ---------------- | ---------------------------------- |
  | `app`           | `string`         | 流应用名                           |
  | `id`            | `string`         | TCP链接唯一ID                      |
  | `ip`            | `string`         | 播放器ip                           |
  | `params`        | `string`         | 播放url参数                        |
  | `port`          | `unsigned short` | 播放器端口号                       |
  | `schema`        | `string`         | 播放的协议，可能是rtsp、rtmp、http |
  | `stream`        | `string`         | 流ID                               |
  | `vhost`         | `string`         | 流虚拟主机                         |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置          |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 40
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 07:41:21 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success"
  }
  ```

- 回复参数详解：

  | 参数名 | 参数类型 | 参数解释                |
  | ------ | -------- | ----------------------- |
  | `code` | `int`    | 错误代码，0代表允许播放 |
  | `msg`  | `string` | 不允许播放时的错误提示  |

### 7、on_publish：

- 解释:

  rtsp/rtmp/rtp推流鉴权事件。

- 触发请求：

  ```
  POST /index/hook/on_publish HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 231
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "id" : "140186529001776",
     "ip" : "10.0.17.132",
     "params" : "token=1677193e-1244-49f2-8868-13b3fcc31b17",
     "port" : 65284,
     "schema" : "rtmp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                     |
  | --------------- | ---------------- | ---------------------------- |
  | `app`           | `string`         | 流应用名                     |
  | `id`            | `string`         | TCP链接唯一ID                |
  | `ip`            | `string`         | 推流器ip                     |
  | `params`        | `string`         | 推流url参数                  |
  | `port`          | `unsigned short` | 推流器端口号                 |
  | `schema`        | `string`         | 推流的协议，可能是rtsp、rtmp |
  | `stream`        | `string`         | 流ID                         |
  | `vhost`         | `string`         | 流虚拟主机                   |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置    |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 89
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 07:46:43 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "enableHls" : true,
     "enableMP4" : false,
     "msg" : "success"
  }
  ```

- 回复参数详解：

  | 参数名      | 参数类型 | 参数解释                |
  | ----------- | -------- | ----------------------- |
  | `code`      | `int`    | 错误代码，0代表允许推流 |
  | `msg`       | `string` | 不允许推流时的错误提示  |
  | `enableHls` | `bool`   | 是否转换成hls协议       |
  | `enableMP4` | `bool`   | 是否允许mp4录制         |
  | `msg`       | `string` | 不允许推流时的错误提示  |

### 8、on_record_mp4:

- 解释:

  录制mp4完成后通知事件；此事件对回复不敏感。

- 触发请求：

  ```
  POST /index/hook/on_record_mp4 HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 473
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "file_name" : "15-53-02.mp4",
     "file_path" : "/root/zlmediakit/httpRoot/__defaultVhost__/record/live/obs/2019-09-20/15-53-02.mp4",
     "file_size" : 1913597,
     "folder" : "/root/zlmediakit/httpRoot/__defaultVhost__/record/live/obs/",
     "start_time" : 1568965982,
     "stream" : "obs",
     "time_len" : 11,
     "url" : "record/live/obs/2019-09-20/15-53-02.mp4",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型 | 参数解释                      |
  | --------------- | -------- | ----------------------------- |
  | `app`           | `string` | 录制的流应用名                |
  | `file_name`     | `string` | 文件名                        |
  | `file_path`     | `string` | 文件绝对路径                  |
  | `file_size`     | `int`    | 文件大小，单位字节            |
  | `folder`        | `string` | 文件所在目录路径              |
  | `start_time`    | `string` | 开始录制事件戳                |
  | `stream`        | `string` | 录制的流ID                    |
  | `time_len`      | `int`    | 录制时长，单位秒              |
  | `url`           | `string` | http/rtsp/rtmp点播相对url路径 |
  | `vhost`         | `string` | 流虚拟主机                    |
  | `mediaServerId` | `string` | 服务器id,通过配置文件设置     |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 40
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 07:53:13 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success"
  }
  ```

### 9、on_rtsp_realm：

- 解释:

  该rtsp流是否开启rtsp专用方式的鉴权事件，开启后才会触发on_rtsp_auth事件。

  需要指出的是rtsp也支持url参数鉴权，它支持两种方式鉴权。

- 触发请求：

  ```
  POST /index/hook/on_rtsp_realm HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 189
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "id" : "140553560034336",
     "ip" : "10.0.17.132",
     "params" : "",
     "port" : 65473,
     "schema" : "rtsp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                  |
  | --------------- | ---------------- | ------------------------- |
  | `app`           | `string`         | 流应用名                  |
  | `id`            | `string`         | TCP链接唯一ID             |
  | `ip`            | `string`         | rtsp播放器ip              |
  | `params`        | `string`         | 播放rtsp url参数          |
  | `port`          | `unsigned short` | rtsp播放器端口号          |
  | `schema`        | `string`         | rtsp或rtsps               |
  | `stream`        | `string`         | 流ID                      |
  | `vhost`         | `string`         | 流虚拟主机                |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置 |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 51
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:05:49 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "realm" : "zlmediakit_reaml"
  }
  ```

- 回复参数详解：

  | 参数名  | 参数类型 | 参数解释                                             |
  | ------- | -------- | ---------------------------------------------------- |
  | `code`  | `int`    | 请固定返回0                                          |
  | `realm` | `string` | 该rtsp流是否需要rtsp专有鉴权，空字符串代码不需要鉴权 |

### 10、on_rtsp_auth：

- 解释:

  rtsp专用的鉴权事件，先触发on_rtsp_realm事件然后才会触发on_rtsp_auth事件。

- 触发请求：

  ```
  POST /index/hook/on_rtsp_auth HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 274
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "id" : "140553560034336",
     "ip" : "10.0.17.132",
     "must_no_encrypt" : false,
     "params" : "",
     "port" : 65473,
     "realm" : "zlmediakit_reaml",
     "schema" : "rtsp",
     "stream" : "obs",
     "user_name" : "test",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名            | 参数类型         | 参数解释                                         |
  | ----------------- | ---------------- | ------------------------------------------------ |
  | `app`             | `string`         | 流应用名                                         |
  | `id`              | `string`         | TCP链接唯一ID                                    |
  | `ip`              | `string`         | rtsp播放器ip                                     |
  | `must_no_encrypt` | `bool`           | 请求的密码是否必须为明文(base64鉴权需要明文密码) |
  | `params`          | `string`         | rtsp url参数                                     |
  | `port`            | `unsigned short` | rtsp播放器端口号                                 |
  | `realm`           | `string`         | rtsp播放鉴权加密realm                            |
  | `schema`          | `string`         | rtsp或rtsps                                      |
  | `stream`          | `string`         | 流ID                                             |
  | `user_name`       | `string`         | 播放用户名                                       |
  | `vhost`           | `string`         | 流虚拟主机                                       |
  | `mediaServerId`   | `string`         | 服务器id,通过配置文件设置                        |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 61
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:05:49 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "encrypted" : false,
     "passwd" : "test"
  }
  ```

- 回复参数详解：

  | 参数名      | 参数类型 | 参数解释                 |
  | ----------- | -------- | ------------------------ |
  | `code`      | `int`    | 错误代码，0代表允许播放  |
  | `msg`       | `string` | 播放鉴权失败时的错误提示 |
  | `encrypted` | `bool`   | 用户密码是否已加密       |
  | `passwd`    | `string` | 用户密码                 |

### 11、on_shell_login：

- 解释:

  shell登录鉴权，ZLMediaKit提供简单的telnet调试方式

  使用`telnet 127.0.0.1 9000`能进入MediaServer进程的shell界面。

- 触发请求：

  ```
  POST /index/hook/on_shell_login HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 124
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "id" : "140227419332496",
     "ip" : "10.0.17.132",
     "passwd" : "111111",
     "port" : 49242,
     "user_name" : "xzl"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                  |
  | --------------- | ---------------- | ------------------------- |
  | `id`            | `string`         | TCP链接唯一ID             |
  | `ip`            | `string`         | telnet 终端ip             |
  | `passwd`        | `bool`           | telnet 终端登录用户密码   |
  | `port`          | `unsigned short` | telnet 终端端口号         |
  | `user_name`     | `string`         | telnet 终端登录用户名     |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置 |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 40
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:23:00 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success"
  }
  ```

- 回复参数详解：

  | 参数名 | 参数类型 | 参数解释                      |
  | ------ | -------- | ----------------------------- |
  | `code` | `int`    | 错误代码，0代表允许登录telnet |
  | `msg`  | `string` | 不允许登录telnet时的错误提示  |

### 12、on_stream_changed:

- 解释:

  rtsp/rtmp流注册或注销时触发此事件；此事件对回复不敏感。

- 触发请求：

  ```
  POST /index/hook/on_stream_changed HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 118
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "regist" : true,
     "schema" : "rtsp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型 | 参数解释                  |
  | --------------- | -------- | ------------------------- |
  | `app`           | `string` | 流应用名                  |
  | `regist`        | `bool`   | 流注册或注销              |
  | `schema`        | `string` | rtsp或rtmp                |
  | `stream`        | `string` | 流ID                      |
  | `vhost`         | `string` | 流虚拟主机                |
  | `mediaServerId` | `string` | 服务器id,通过配置文件设置 |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 40
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:27:35 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success"
  }
  ```

### 13、on_stream_none_reader：

- 解释:

  流无人观看时事件，用户可以通过此事件选择是否关闭无人看的流。

- 触发请求：

  ```
  POST /index/hook/on_stream_none_reader HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 98
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "schema" : "rtmp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

| 参数名          | 参数类型 | 参数解释                  |
| --------------- | -------- | ------------------------- |
| `app`           | `string` | 流应用名                  |
| `schema`        | `string` | rtsp或rtmp                |
| `stream`        | `string` | 流ID                      |
| `vhost`         | `string` | 流虚拟主机                |
| `mediaServerId` | `string` | 服务器id,通过配置文件设置 |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 37
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:51:23 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "close" : true,
     "code" : 0
  }
  ```

- 回复参数详解：

  | 参数名  | 参数类型 | 参数解释           |
  | ------- | -------- | ------------------ |
  | `code`  | `int`    | 请固定返回0        |
  | `close` | `bool`   | 是否关闭推流或拉流 |

### 14、on_stream_not_found：

- 解释:

  流未找到事件，用户可以在此事件触发时，立即去拉流，这样可以实现按需拉流；此事件对回复不敏感。

- 触发请求：

  ```
  POST /index/hook/on_stream_not_found HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 189
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "mediaServerId" : "your_server_id",
     "app" : "live",
     "id" : "140183261486112",
     "ip" : "10.0.17.132",
     "params" : "",
     "port" : 49614,
     "schema" : "rtsp",
     "stream" : "obs",
     "vhost" : "__defaultVhost__"
  }
  ```

- 请求参数详解：

  | 参数名          | 参数类型         | 参数解释                     |
  | --------------- | ---------------- | ---------------------------- |
  | `app`           | `string`         | 流应用名                     |
  | `id`            | `string`         | TCP链接唯一ID                |
  | `ip`            | `string`         | 播放器ip                     |
  | `params`        | `string`         | 播放url参数                  |
  | `port`          | `unsigned short` | 播放器端口号                 |
  | `schema`        | `string`         | 播放的协议，可能是rtsp、rtmp |
  | `stream`        | `string`         | 流ID                         |
  | `vhost`         | `string`         | 流虚拟主机                   |
  | `mediaServerId` | `string`         | 服务器id,通过配置文件设置    |

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 51
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:55:49 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success
  }
  ```

### 15、on_server_started

- 解释:

  服务器启动事件，可以用于监听服务器崩溃重启；此事件对回复不敏感。

- 触发请求：

  ```
  POST /index/hook/on_server_started HTTP/1.1
  Accept: */*
  Accept-Language: zh-CN,zh;q=0.8
  Connection: keep-alive
  Content-Length: 3096
  Content-Type: application/json
  Host: 127.0.0.1
  Tools: ZLMediaKit
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36
  
  {
     "api.apiDebug" : "1",
     "api.secret" : "035c73f7-bb6b-4889-a715-d9eb2d1925cc",
     "ffmpeg.bin" : "/usr/local/bin/ffmpeg",
     "ffmpeg.cmd" : "%s -re -i %s -c:a aac -strict -2 -ar 44100 -ab 48k -c:v libx264 -f flv %s",
     "ffmpeg.log" : "./ffmpeg/ffmpeg.log",
     "general.mediaServerId" : "your_server_id",
     "general.addMuteAudio" : "1",
     "general.enableVhost" : "1",
     "general.flowThreshold" : "1024",
     "general.maxStreamWaitMS" : "5000",
     "general.publishToHls" : "1",
     "general.publishToMP4" : "0",
     "general.publishToRtxp" : "1",
     "general.resetWhenRePlay" : "1",
     "general.streamNoneReaderDelayMS" : "5000",
     "general.ultraLowDelay" : "1",
     "hls.fileBufSize" : "65536",
     "hls.filePath" : "./httpRoot",
     "hls.segDur" : "2",
     "hls.segNum" : "3",
     "hls.segRetain" : "5",
     "hook.admin_params" : "secret=035c73f7-bb6b-4889-a715-d9eb2d1925cc",
     "hook.enable" : "1",
     "hook.on_flow_report" : "https://127.0.0.1/index/hook/on_flow_report",
     "hook.on_http_access" : "https://127.0.0.1/index/hook/on_http_access",
     "hook.on_play" : "https://127.0.0.1/index/hook/on_play",
     "hook.on_publish" : "https://127.0.0.1/index/hook/on_publish",
     "hook.on_record_mp4" : "https://127.0.0.1/index/hook/on_record_mp4",
     "hook.on_rtsp_auth" : "https://127.0.0.1/index/hook/on_rtsp_auth",
     "hook.on_rtsp_realm" : "https://127.0.0.1/index/hook/on_rtsp_realm",
     "hook.on_server_started" : "http://127.0.0.1/index/hook/on_server_started",
     "hook.on_shell_login" : "https://127.0.0.1/index/hook/on_shell_login",
     "hook.on_stream_changed" : "https://127.0.0.1/index/hook/on_stream_changed",
     "hook.on_stream_none_reader" : "https://127.0.0.1/index/hook/on_stream_none_reader",
     "hook.on_stream_not_found" : "https://127.0.0.1/index/hook/on_stream_not_found",
     "hook.timeoutSec" : "10",
     "http.charSet" : "utf-8",
     "http.keepAliveSecond" : "15",
     "http.maxReqCount" : "100",
     "http.maxReqSize" : "4096",
     "http.notFound" : "<html><head><title>404 Not Found</title></head><body bgcolor=\"white\"><center><h1>您访问的资源不存在</h1></center><hr><center>ZLMediaKit-4.0</center></body></html>",
     "http.port" : "80",
     "http.rootPath" : "./httpRoot",
     "http.sendBufSize" : "65536",
     "http.sslport" : "443",
     "multicast.addrMax" : "239.255.255.255",
     "multicast.addrMin" : "239.0.0.0",
     "multicast.udpTTL" : "64",
     "record.appName" : "record",
     "record.fastStart" : "0",
     "record.fileBufSize" : "65536",
     "record.filePath" : "./httpRoot",
     "record.fileRepeat" : "0",
     "record.fileSecond" : "3600",
     "record.sampleMS" : "500",
     "rtmp.handshakeSecond" : "15",
     "rtmp.keepAliveSecond" : "15",
     "rtmp.modifyStamp" : "1",
     "rtmp.port" : "1935",
     "rtp.audioMtuSize" : "600",
     "rtp.clearCount" : "10",
     "rtp.cycleMS" : "46800000",
     "rtp.maxRtpCount" : "50",
     "rtp.videoMtuSize" : "1400",
     "rtsp.authBasic" : "0",
     "rtsp.directProxy" : "1",
     "rtsp.handshakeSecond" : "15",
     "rtsp.keepAliveSecond" : "15",
     "rtsp.modifyStamp" : "0",
     "rtsp.port" : "554",
     "rtsp.sslport" : "322",
     "shell.maxReqSize" : "1024",
     "shell.port" : "9000"
  }
  ```

- 请求参数详解： 配置文件json对象

- 默认回复:

  ```
  HTTP/1.1 200 OK
  Connection: keep-alive
  Content-Length: 51
  Content-Type: application/json; charset=utf-8
  Date: Fri, Sep 20 2019 08:55:49 GMT
  Keep-Alive: timeout=10, max=100
  Server: ZLMediaKit-4.0
  
  {
     "code" : 0,
     "msg" : "success
  }
  ```