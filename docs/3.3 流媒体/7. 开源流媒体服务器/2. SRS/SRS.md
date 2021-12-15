#  SRS(Simple Realtime Server)

[SRS](http://ossrs.net/)(Simple RTMP Server) 是国人写的一款非常优秀的开源流媒体服务器软件，可用于直播/录播/视频客服等多种场景，其定位是运营级的互联网直播服务器集群。是一个流媒体集群，支持RTMP/HLS/WebRTC/SRT/GB28181，高效、稳定、易用，简单而快乐。

## 软件定位

- **运营级** ：商业运营追求极高的稳定性、良好的系统对接、错误排查和处理机制。譬如日志文件格式、reload、系统 HTTP 接口、提供 init.d 脚本、转发、转码和边缘回多源站，都是根据 CDN 运营经验作为判断这些功能作为核心的依据。
- **互联网** ：互联网最大的特征是变化，唯一不变的就是不断变化的客户要求，唯一不变的是基础结构的概念完整性和简洁性。互联网还意味着参与性，听取用户的需求和变更，持续改进和维护。
- **直播服务器** ：直播和点播这两种截然不同的业务类型，导致架构和目标完全不一致，从运营的设备组，到应对的挑战都完全不同。两种都支持只能说明没有重心或者低估了代价。
- **集群** ：FMS(AMS) 的集群还是很不错的，虽然运营容错很差。SRS 支持完善的直播集群，Vhost 分为源站和边缘，容错支持多源站切换、测速、可追溯日志等。
- **概念完整性** ：虽然代码甚至结构都在变化，但是结构的概念完整性是一直追求的目标。SRS 服务器、P2P、ARM 监控产业、MIPS 路由器，服务器监控管理、ARM 智能手机，SRS 的规模不再是一个服务器而已。

## 软件应用

- 搭建大规模 CDN 集群，可以在 CDN 内部的源站和边缘部署 SRS。
- 小型业务快速搭建几台流媒体集群，譬如学校、企业等，需要分发的流不多，同时 CDN 覆盖不如自己部署几个节点，可以用 SRS 搭建自己的小集群。
- SRS 作为源站，CDN 作为加速边缘集群。比如推流到 CDN 后 CDN 转推到源站，播放时 CDN 会从源站取流。这样可以同时使用多个 CDN。同时还可以在源站做 DRM 和 DVR，输出 HLS，更重要的是如果直接推 CDN，一般 CDN 之间不是互通的，一个 CDN 出现故障无法快速切换到其他 CDN。
- 编码器可以集成 SRS 支持拉流。一般编码器支持推 RTMP/UDP 流，如果集成 SRS 后，可以支持多种拉流。
- 协议转换网关，比如可以推送 FLV 到 SRS 转成 RTMP 协议，或者拉 RTSP 转 RTMP，还有拉 HLS 转 RTMP。SRS 只要能接入流，就能输出能输出的协议。
- 学习流媒体可以用 SRS。SRS 提供了大量的协议的文档、wiki 和文档对应的代码、详细的 issues、流媒体常见的功能实现，以及新流媒体技术的尝试等。

# SRS特点

1、简单，足够稳定。

2、高性能，高并发，SRS是单线程、事件/st-线程驱动。最大可支持6k客户端。官网性能介绍： [性能测试](https://github.com/ossrs/srs/wiki/v1_CN_Performance)

3、可以作为rtmp源服务器，也可作为节点对接CND，从其他rtmp服务器上推/拉流。

4、支持Vhost 及defaultVhost配置。

5、核心功能是分发RTMP，主要定位就是分发RTMP低延时流媒体，同时支持分发HLS流。

6、服务Reload 机制，即在不中断服务时应用配置的修改。达到不中断服务调整码率，添加或调整频道。

7、cache 一个GOP ，达到播放器能快速播放的效果。(gop_cache配置项)

8、可监听多个端口，支持长时间推拉流。

9、forward配置项，可在服务器间转发流。

10、支持转码，可以对推送到SRS的RTMP流进行转码，然后输出到其他RTMP服务器。可对指定的流配置是否转码。内置了FFMPEG.同时会提供FFMPEG的部分功能：输出纯音频、加文字水印、剪切视频、添加图片LOGO等。

11、支持http回调，提供了客户端连接接口、关闭连接接口、流发布、流停止、流播放、停止播放等接口，方便再封装的应用跟踪流信息。内置也有一个http服务器，可直接调用api接口。

12、内置流带宽测试工具、完善的日志跟踪规则。

13、脚本管理，提供init.d系统脚本，也可通过调用api 控制服务状态。

14、采集端支持：设备、本地文件，RTSP摄像头、rtmp等。官方意思是，能拉任意的流，只要FFMPEG支持，不是h264/aac都没有关系，FFMPEG能转码。SRS的接入方式可以是“推流到SRS”和“SRS主动拉流”。

15、支持将RTMP流录制成flv文件。FLV文件的命名规则是随机生成流名称，上层应用可通过http-callback 管理流信息。

16、SRS日志很完善，支持打印到console和file，支持设置level，支持连接级别的日志，支持可追溯日志。

# 软件对比

与其他媒体软件对比。

### Stream Delivery

| FEATURE     | SRS        | NGINX  | CRTMPD | FMS    | WOWZA  |
| ----------- | ---------- | ------ | ------ | ------ | ------ |
| RTMP        | Stable     | Stable | Stable | Stable | Stable |
| HLS         | Stable     | Stable | X      | Stable | Stable |
| HDS         | Experiment | X      | X      | Stabl  | Stable |
| HTTP FLV    | Stable     | X      | X      | X      | X      |
| HLS(aonly)  | Stable     | X      | X      | Stable | Stable |
| HTTP Server | Stable     | Stable | X      | X      | Stable |

### Cluster

| FEATURE     | SRS    | NGINX | CRTMPD | FMS    | WOWZA  |
| ----------- | ------ | ----- | ------ | ------ | ------ |
| RTMP Edge   | Stable | X     | X      | Stable | X      |
| RTMP Backup | Stable | X     | X      | X      | X      |
| VHOST       | Stable | X     | X      | Stable | Stable |
| Reload      | Stable | X     | X      | X      | X      |
| Forward     | Stable | X     | X      | X      | X      |
| ATC         | Stable | X     | X      | X      | X      |

### Stream Service

| FEATURE        | SRS    | NGINX  | CRTMPD | FMS    | WOWZA  |
| -------------- | ------ | ------ | ------ | ------ | ------ |
| DVR            | Stable | Stable | X      | X      | Stable |
| Transcode      | Stable | X      | X      | X      | Stable |
| HTTP API       | Stable | Stable | X      | X      | Stable |
| HTTP hooks     | Stable | X      | X      | X      | X      |
| GopCache       | Stable | X      | X      | Stable | X      |
| Security       | Stable | Stable | X      | X      | Stable |
| Token Traverse | Stable | X      | X      | Stable | X      |



# Docker安装

```
docker run -p 1935:1935 -p 1985:1985 -p 8080:8080 ossrs/srs:3
```

After SRS is running, you can:

- Publish stream to SRS by `ffmpeg -re -i doc/source.200kbps.768x320.flv -c copy -f flv rtmp://127.0.0.1/live/livestream`
- Play stream from SRS by `ffmpeg -f flv -i rtmp://127.0.0.1/live/livestream -f flv -y /dev/null`
- Access the console by http://127.0.0.1:1985/console

The env of docker is bellow:

- config file: /usr/local/srs/conf/srs.conf
- log file: /usr/local/srs/objs/srs.log

To overwrite the config by `/path/of/yours.conf` and gather log to `/path/of/yours.log`:

```
docker run -p 1935:1935 -p 1985:1985 -p 8080:8080 \
    -v /path/of/yours.conf:/usr/local/srs/conf/srs.conf \
    -v /path/of/yours.log:/usr/local/srs/objs/srs.log \
    ossrs/srs:3
```