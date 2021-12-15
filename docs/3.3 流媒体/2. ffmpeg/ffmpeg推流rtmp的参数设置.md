[ffmpeg推流rtmp的参数设置](https://blog.csdn.net/impingo/article/details/104163365)



# ffmpeg针对rtmp协议的参数

| 参数           | 类型       | 说明                                                         |
| -------------- | ---------- | ------------------------------------------------------------ |
| rtmp_app       | 字符串     | RTMP 流发布点，又称 APP                                      |
| rtmp_buffer    | 整数       | 客户端 buffer 大小（单位：毫秒），默认为 3 秒                |
| rtmp_conn      | 字符串     | 在 RTMP 的 Connect 命令中增加自定义 AMF 数据                 |
| rtmp_flashver  | 字符串     | 设置模拟的 flashplugin 的版本号                              |
| rtmp_live      | 整数       | 指定 RTMP 流媒体播放类型，具体如下： any：直播或点播随意 live：直播 recorded：点播 |
| rtmp_pageurl   | 字符串     | RTMP 在 Connect 命令中设置的 PageURL 字段，其为播放时所在的 Web 页面 URL |
| rtmp_playpath  | 字符串     | RTMP 流播放的 Stream 地址，或者称为密钥，或者称为发布流      |
| rtmp_subscribe | 字符串     | 直播流名称，默认设置为 rtmp_playpath 的值                    |
| rtmp_swfhash   | 二进制数据 | 解压 swf 文件后的 SHA256 的 hash 值                          |
| rtmp_swfsize   | 整数       | swf 文件解压后的大小，用于 swf 认证                          |
| rtmp_swfurl    | 字符串     | RTMP 的 Connect 命令中设置的 swfURL 播放器的 URL             |
| rtmp_swfverify | 字符串     | 设置 swf 认证时 swf 文件的 URL 地址                          |
| rtmp_tcurl     | 字符串     | RTMP 的 Connect 命令中设置的 tcURL 目标发布点地址，一般形如 rtmp://xxx.xxx.xxx/app |
| rtmp_listen    | 整数       | 开启 RTMP 服务时所监听的端口                                 |
| listen         | 整数       | 与 rtmp_listen 相同                                          |
| timeout        | 整数       | 监听 rtmp 端口时设置的超时时间，以秒为单位                   |

# ffmpeg使用示例

## 推流

使用rtmp_app、rtmp_playpath参数示例：

```bash
ffmpeg -re -i test.mp4 -c copy -f flv -rtmp_app live -rtmp_playpath steam rtmp://live.pingos.io
```

等价于

```bash
ffmpeg -re -i test.mp4 -c copy -f flv rtmp://live.pingos.io/live/stream
```

看出使用技巧了吗，其他参数的值也可以用同样的方式指定，是不是很简单！

## 给rtmp地址添加参数

一般的推流和拉流地址长这样，rtmp://xxx.xxx.xxx.xxx/app/streamname

但是很多时候我们需要服务器做一些权限验证，就要求rtmp连接时携带token，我们就可以通过以下两种方式将token带给服务器。

```bash
ffmpeg -re -i test.mp4 -c copy -f flv -rtmp_app live -rtmp_playpath "steam?token=xxx" rtmp://live.pingos.io

ffmpeg -re -i test.mp4 -c copy -f flv "rtmp://live.pingos.io/live/stream?token=xxx"
```