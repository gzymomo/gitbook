[FFmpeg Protocols Documentation](https://ffmpeg.org/ffmpeg-protocols.html)

# 实时流协议(rtsp)



RTSP在技术上不是libavformat中的协议处理程序，而是一个demuxer和muxer。demuxer支持普通的RTSP（通过RTP传输数据；这是Apple和Microsoft等公司使用的）和Real RTSP（通过RDT传输数据）。

RTSP url所需的语法为：

```bash
rtsp://hostname[:port]/path
```

选项可以在ffmpeg/ffplay命令行上设置，也可以通过AVOptions在代码中设置，或者在avformat_open_输入中设置。



## 支持以下选项(options)

### initial_pause

初始暂停

如果设置为1，请不要立即开始播放流。默认值为0。

### rtsp_transport

设置RTSP传输协议。

它接受以下值：

- “udp”

使用UDP作为较低的传输协议。

- “tcp”

使用TCP（RTSP控制通道内的交错）作为较低的传输协议。

- “udp_multicast”

使用UDP多播作为较低的传输协议。

- “http”

使用HTTP隧道作为较低的传输协议，这对于传递代理很有用。

可以指定多个较低的传输协议，在这种情况下，一次尝试一个协议（如果一个协议的设置失败，则尝试下一个协议）。对于muxer，只支持“tcp”和“udp”选项。

### rtsp_flags

可接受以下值：

- filter_src：只接受来自协商的对等地址和端口的数据包。
- listen：充当服务器，监听传入的连接。
- prefer_tcp：如果TCP可用作RTSP RTP传输，请先尝试使用TCP进行RTP传输。

默认值为“无”。



### allowed_media_types

允许的媒体类型

将媒体类型设置为从服务器接受。

接受以下标志：

- ‘video’
- ‘audio’
- ‘data’

默认情况下，它接受所有媒体类型。



### min_port

设置最小本地UDP端口。默认值为5000。

### max_port

设置最大本地UDP端口。默认值为65000。

### timeout

设置等待传入连接的最大超时（秒）。

值-1表示无穷大（默认值）。此选项意味着将rtsp_标志设置为“listen”。

### reorder_queue_size

设置用于处理重新排序的数据包的缓冲区的数据包数。

### stimeout

以微秒为单位设置套接字TCP I/O超时。

### user-agent

重写用户代理头。如果未指定，则默认为libavformat标识符字符串。



当通过UDP接收数据时，demuxer会尝试对接收到的数据包进行重新排序（因为它们可能会无序到达，或者数据包可能会完全丢失）。这可以通过将最大解复用延迟设置为零（通过AVFormatContext的max_delay字段）来禁用。



当用ffplay观看多比特率的实时RTSP流时，可以用-vst n和-ast n分别选择视频和音频要显示的流，并且可以通过按v和a来动态切换。

## 示例：examples

通过UDP观看流，最大重新排序延迟为0.5秒：

```bash
ffplay -max_delay 500000 -rtsp_transport udp rtsp://server/video.mp4
```

观察一个stream tunneled over HTTP：

```bash
ffplay -rtsp_transport http rtsp://server/video.mp4
```

将流实时发送到RTSP服务器，供其他人观看：

```bash
ffmpeg -re -i input -f rtsp -muxdelay 0.1 rtsp://server/live.sdp
```

实时接收流：

```bash
ffmpeg -rtsp_flags listen -i rtsp://ownaddress/live.sdp output
```

### Java代码示例

```java
/**
  * 读流器
  */
private FFmpegFrameGrabber grabber;

 try {
     grabber = new FFmpegFrameGrabber(url);
     if ("rtsp".equals(url.substring(0, 4))) {
         //rw_timeout：等待（网络）读/写操作完成的最长时间（微秒）。
         //1秒= 1000000 微妙
         grabber.setOption("rtsp_transport", "tcp");
         grabber.setOption("stimeout", "5000000");
     } else {
         grabber.setOption("timeout", "5000000");
     }
     grabber.start();
```



# 实时消息传递协议（rtmp）

实时消息传递协议（RTMP）用于在TCP/IP网络上传输多媒体内容。

所需的语法是：

```bash
rtmp://[username:password@]server[:port][/app][/instance][/playpath]
```



## 可接受的参数为：

### username

An optional username (mostly for publishing).

### password

An optional password (mostly for publishing).

### server

The address of the RTMP server.

### port

The number of the TCP port to use (by default is 1935).

### app

It is the name of the application to access. It usually corresponds to the path where the application is installed on the RTMP server (e.g. /ondemand/, /flash/live/, etc.). You can override the value parsed from the URI through the `rtmp_app` option, too.

### playpath

It is the path or name of the resource to play with reference to the application specified in app, may be prefixed by "mp4:". You can override the value parsed from the URI through the `rtmp_playpath` option, too.

### listen

Act as a server, listening for an incoming connection.

### timeout

Maximum time to wait for the incoming connection. Implies listen.

Additionally, the following parameters can be set via command line options (or in code via `AVOption`s):

### rtmp_app

Name of application to connect on the RTMP server. This option overrides the parameter specified in the URI.

### rtmp_buffer

Set the client buffer time in milliseconds. The default is 3000.

### rtmp_conn

Extra arbitrary AMF connection parameters, parsed from a string, e.g. like `B:1 S:authMe O:1 NN:code:1.23 NS:flag:ok O:0`. Each value is prefixed by a single character denoting the type, B for Boolean, N for number, S for string, O for object, or Z for null, followed by a colon. For Booleans the data must be either 0 or 1 for FALSE or TRUE, respectively. Likewise for Objects the data must be 0 or 1 to end or begin an object, respectively. Data items in subobjects may be named, by prefixing the type with ’N’ and specifying the name before the value (i.e. `NB:myFlag:1`). This option may be used multiple times to construct arbitrary AMF sequences.

### rtmp_flashver

Version of the Flash plugin used to run the SWF player. The default is LNX 9,0,124,2. (When publishing, the default is FMLE/3.0 (compatible; <libavformat version>).)

### rtmp_flush_interval

Number of packets flushed in the same request (RTMPT only). The default is 10.

### rtmp_live

Specify that the media is a live stream. No resuming or seeking in live streams is possible. The default value is `any`, which means the subscriber first tries to play the live stream specified in the playpath. If a live stream of that name is not found, it plays the recorded stream. The other possible values are `live` and `recorded`.

### rtmp_pageurl

URL of the web page in which the media was embedded. By default no value will be sent.

### rtmp_playpath

Stream identifier to play or to publish. This option overrides the parameter specified in the URI.

### rtmp_subscribe

Name of live stream to subscribe to. By default no value will be sent. It is only sent if the option is specified or if rtmp_live is set to live.

### rtmp_swfhash

SHA256 hash of the decompressed SWF file (32 bytes).

### rtmp_swfsize

Size of the decompressed SWF file, required for SWFVerification.

### rtmp_swfurl

URL of the SWF player for the media. By default no value will be sent.

### rtmp_swfverify

URL to player swf file, compute hash/size automatically.

### rtmp_tcurl

URL of the target stream. Defaults to proto://host[:port]/app.



## examples

例如，使用ffplay从RTMP服务器“myserver”中读取应用程序“vod”中名为“sample”的多媒体资源：

```bash
ffplay rtmp://myserver/vod/sample
```

要发布到受密码保护的服务器，请分别传递播放路径和应用程序名称：

```bash
ffmpeg -re -i <input> -f flv -rtmp_playpath some/long/path -rtmp_app long/app/name rtmp://username:password@myserver/
```



# 传输控制协议(tcp)

TCP url所需的语法为：

```bash
tcp://hostname:port[?options]
```

options包含格式为key=val的&-分隔选项列表。



下面是支持的选项列表。

## **listen=**1|0

监听传入的连接。默认值为0。

## timeout=microseconds

设置引发错误超时，以微秒表示。此选项仅与读取模式相关：如果在超过此时间间隔的时间内没有数据到达，则引发错误。

## listen_timeout=milliseconds

设置侦听超时，以毫秒为单位。

## recv_buffer_size=bytes

设置接收缓冲区大小（以字节表示）。

## send_buffer_size=bytes

设置发送缓冲区大小（以字节表示）。

## tcp_nodelay=1|0

将TCP_NODELAY设置为禁用Nagle的算法。默认值为0。

## tcp_mss=bytes

设置传出TCP数据包的最大段大小，以字节表示。



以下示例显示如何使用ffmpeg设置侦听TCP连接，然后使用ffplay访问该连接：

```bash
ffmpeg -i input -f format tcp://hostname:port?listen
ffplay tcp://hostname:port
```

# hls

将符合Apple HTTP Live Streaming的分段流视为统一流。描述片段的M3U8播放列表可以是远程HTTP资源或本地文件，可以使用标准文件协议进行访问。嵌套协议通过在hls URI方案名后面指定“+proto”来声明，其中proto是“file”或“http”。

```bash
hls+http://host/path/to/remote/resource.m3u8
hls+file://path/to/local/resource.m3u8
```

不鼓励使用此协议-hls demuxer应该也能正常工作（如果不能，请报告问题），并且更完整。要使用hls demuxer，只需使用指向m3u8文件的直接url。

