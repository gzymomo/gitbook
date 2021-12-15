# 一、FMP4简介

比较常用的视频封装格式有 WebM 和 fMP4。

WebM 和 WebP 是两个姊妹项目，都是由 Google 赞助的。由于 WebM 是基于 Matroska 的容器格式，天生是流式的，很适合用在流媒体领域里。



MP4 是由一系列的 Boxes 组成的。普通的 MP4 的是嵌套结构的，客户端必须要从头加载一个 MP4 文件，才能够完整播放，不能从中间一段开始播放。

而 fMP4 由一系列的片段组成，如果服务器支持 byte-range 请求，那么，这些片段可以独立的进行请求到客户端进行播放，而不需要加载整个文件。

为了更加形象的说明这一点，下面我介绍几个常用的分析 MP4 文件的工具。

gpac，原名 mp4box，是一个媒体开发框架，在其源码下有大量的媒体分析工具，可以使用testapps；

- mp4box.js，是 mp4box 的 Javascript 版本；
- bento4，一个专门用于 MP4 的分析工具；
- mp4parser，在线 MP4 文件分析工具。

## 1.1 fragment mp4 VS non-fragment mp4

下面是一个 fragment mp4 文件通过 mp4parser（[Online MPEG4 Parser](http://link.zhihu.com/?target=http%3A//mp4parser.com) ）分析后的截图 ▽

![img](https://pic2.zhimg.com/v2-fdf146b51fc8a9f62fe59c17063b407d_b.png)

下面是一个 non-fragment mp4 文件通过 mp4parser 分析后的截图 ▽

![img](https://pic1.zhimg.com/v2-00a18c3bcdc89b9f754213d1bc450e54_b.png)

我们可以看到 non-fragment mp4 的最顶层 box 类型非常少，而 fragment mp4 是由一段一段的 moof+mdat 组成的，它们已经包含了足够的 metadata 信息与数据, 可以直接 seek 到这个位置开始播放。也就是说 fMP4 是一个流式的封装格式，这样更适合在网络中进行流式传输，而不需要依赖文件头的metadata。

Apple在WWDC 2016 大会上宣布会在 iOS 10、tvOS、macO S的 HLS 中支持 fMP4，可见fMP4 的前景非常的好。

值得一提的是，fMP4、CMAF、ISOBMFF 其实都是类似的东西。



# 二、FMP4流媒体直播服务器

[rtmpmate](http://studease.cn/rtmpmate.html)是一款集流媒体直播、websocket聊天室、http静态文件服务等功能于一身的高性能服务器。该服务器使用Golang编写，支持跨平台。

服务器：http://studease.cn/rtmpmate.html

播放器：http://studease.cn/playease.html

聊天室：http://studease.cn/chatease.html

## 支持的协议、标准和功能

\> 支持rtmp、rtsp（tcp）、http/ws-flv、http/ws-fmp4、mpeg-dash、hls等多种直播协议和标准；

\> 支持flv录制（可远程控制），及实时回放功能；

\> 支持websocket弹幕聊天功能；

\> 支持直播流的发布、播放、录制，以及聊天室的连接、控制等事件推送；

\> 支持origin-edge集群、代理，及负载均衡；

\> 支持跨域保护；

\> 支持定制开发（额外付费）。

 

## 服务器配置详解

### 1. upstream

​    所有协议内部均可配置多个上游服务器，以name分类，分类为服务器内部/协议间可见。例如，rtmp里面配置了name=web的upstream，与http里面配置的同名服务器会被划分为同一类，也就是实际上定义了两台name=web的服务器（虽然在不同的地方定义），在使用时按照权重（weight）实行负载均衡。

\> name：服务器名称/分类。

\> address：IP地址。

\> port：端口号。

\> flag：标识（"normal" "backup" "down"），默认="normal"。

\> weight：权重，默认=5。

\> timeout：超时时间（单位=秒），默认=5。

\> max_fails：最大失败次数，达到限制数后flag设置为"down"，默认=0（无限制）。

 

### 2. rtmp server

\> listen：监听端口，默认=1935。

\> timeout：连接最大闲置时间，默认=65。

\> root：--

\> cors：--

\> chunk_size：rtmp协议层参数，最小值=128，默认=4096。

\> cache：发送缓冲区帧数，默认=512。

\> target：--

\> location：路由模块。

\>> pattern：路由路径，默认="/"。

\>> handler：--

\>> on_xxx：事件推送接口。

\>> dvr：录像机。

\>> proxy：代理/集群。

 

#### **2.1 dvr**

​    要启用http-flv功能，须有一个id="DVR_FLV"的flv录像机；要启用mpeg-dash、hls功能，须有一个id="DVR_FMP4"的fmp4录像机。启用最大值限定（max_xxx），并成功触发条件时，DVR_FLV会将后续数据写入新文件（注意启用unique，否则会被覆盖），DVR_FMP4会生成一个切片。

\> id：特殊场景下用于访问该录像机（例如：远程控制）。

\> name：录像机名称（"DVR_FLV" "DVR_FMP4"）。

\> mode：录像模式（"audio" "video" "all" "autoAudio" "autoVideo" "auto" "keyframe" "advanced" "manual" "off"），常用="all"。

\> directory：存储目录，可用变量"$(app)" "$(instance)"，支持格式化时间。

\> file：文件名（name="DVR_FLV"）、文件夹名（name="DVR_FMP4"），可用变量"%s"（streamName）。

\> unique：文件名是否唯一（仅name="DVR_FLV"），示例：sample-1554107911.flv。

\> append：是否为追加模式。

\> max_size：最大文件大小，单位=byte。

\> max_time：最大录像时长，单位=ms。

\> max_frames：最大帧数。

\> on_record：录像开始的回调（仅DVR_FLV）。

\> on_record_done：录像结束的回调（仅DVR_FLV）。

 

### 3. http server

\> listen：监听端口，默认=80。

\> timeout：连接最大闲置时间，默认=65。

\> root：根目录。

\> cors：可访问的域，多个域以","分隔，"*"表示所有域，默认=""。

\> server_name：域名/IP正则匹配pattern，用于将二级域名重定向到子文件夹。

\> location：路由模块。

\>> pattern：路由路径，默认="/"。

\>> handler：注册的回调函数名（"http_file" "rtmp_control" "chat"）。

 

#### **3.1 handler http_file**

​    提供静态文件分发。要启用http-flv、mpeg-dash、hls等功能，需开启http服务的http_file handler功能。

\> cache：发送缓冲区帧数，默认=512。

 

#### **3.2 handler rtmp_control**

​    远程控制rtmp行为。

**3.2.1 record**

\> start：http://192.168.4.248/rtmp/control/record/start?dvr=dvr1&app=live&inst=_definst_&name=sample

\> stop：http://192.168.4.248/rtmp/control/record/stop?dvr=dvr1&app=live&inst=_definst_&name=sample

 

#### **3.3 handler chat**

\> protocol：websocket握手阶段Sec-Websocket-Protocol响应值。

\> cache：发送缓冲区消息数，默认=32。

\> query：消息存储器，目前只支持sqlite3。

\> on_connect：连接事件推送接口。

\> on_control：权限管理事件推送接口。

\> visitor：游客模式。

\> group：频道内分组。

\>> maximum：最大分组数，0表示无限制。

\>> capacity：组容量，0表示无限制。

\> user：在线人数推送服务。

\>> period：推送周期，单位=秒。