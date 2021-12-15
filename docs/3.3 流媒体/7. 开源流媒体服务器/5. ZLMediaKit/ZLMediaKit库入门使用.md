# 一、 ZLMediaKit 库简介

ZLMediaKit 是一个基于C++11的高性能运营级流媒体服务框架

**官方写的项目特点：**

- 基于C++11开发，避免使用裸指针，代码稳定可靠，性能优越。
- 支持多种协议(RTSP/RTMP/HLS/HTTP-FLV/Websocket-FLV/GB28181/MP4),支持协议互转。
- 使用多路复用/多线程/异步网络IO模式开发，并发性能优越，支持海量客户端连接。
- 代码经过长期大量的稳定性、性能测试，已经在线上商用验证已久。 支持linux、macos、ios、android、windows全平台。
- 支持画面秒开、极低延时(500毫秒内，最低可达100毫秒)。 提供完善的标准C API,可以作SDK用，或供其他语言调用。
- 提供完整的MediaServer服务器，可以免开发直接部署为商用服务器。 提供完善的restful api以及web hook，支持丰富的业务逻辑。 打通了视频监控协议栈与直播协议栈，对RTSP/RTMP支持都很完善。
- 全面支持H265/H264/AAC/G711/OPUS。

其功能非常多，支持RTSP、RTMP[S]、HLS、GB28181等多种流媒体格式。

# 二、Docker方式启动

```bash
docker run --name zlmediakit -id -p 1935:1935 -p 8080:80 -p 8554:554 -p 10000:10000 -p 10000:10000/udp panjjo/zlmediakit
```

