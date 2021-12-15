个人学习工作笔记总结(包含Java相关，数据库相关，运维相关，docker，Kubernetes，流媒体相关，项目管理相关，代码审查相关，安全渗透相关，开发工具，框架技术等等内容)。

​														 	[![爱是与世界平行/lovebetterworld](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/widgets/widget_card.svg?colors=393222,ebdfc1,fffae5,d8ca9f,393222,a28b40)](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld)

## 介绍

​	个人读书，学习，阅读，工作的笔记库，收藏来自各大博文网站，书籍，小道系统的学习笔记，文章汇总等资源，或总结一些个人学习过程的知识点等。

​	部分内容准备迁移至语雀，为了更方便的实现多端阅读。

​	语雀地址：https://www.yuque.com/lovebetterworld

## 阅读说明

推荐使用Typora阅读本笔记，里面笔记全部为MarkDown格式。

1. 克隆项目到本地

   `git clone https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld.git`

2. 通过Typora打开文件夹浏览

![image-20210926090240534](https://gitee.com/er-huomeng/l-img/raw/master/typora/image-20210926090240534.png)

## 流媒体

### 直播

- [如何降低直播延时](https://blog.csdn.net/impingo/article/details/104096040)
- [直播延时讲解](https://blog.csdn.net/impingo/article/details/104079647)
- [直播支持https连接](https://blog.csdn.net/impingo/article/details/105421563)
- [直播系统开发过程中，如何选择流媒体协议？](https://cloud.tencent.com/developer/article/1534015)
- [如何将安防摄像头接入互联网直播服务器](https://blog.csdn.net/impingo/article/details/102907201)

### ffmpeg

- [CentOS7安装ffmpeg](https://www.cnblogs.com/wangrong1/p/11951856.html)

- [ffmpeg架构]()

- [ffmpeg推流rtmp的参数设置](https://blog.csdn.net/impingo/article/details/104163365)

- [FFmpeg Protocols Documentation](https://ffmpeg.org/ffmpeg-protocols.html)

  【ffmpeg命令】

- [ffmpeg命令](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/ffmpeg/ffmpeg%E5%91%BD%E4%BB%A4)

  【ffmpeg官方文档详解】

  - [ffmpeg官方文档详解](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/ffmpeg/ffmpeg%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E4%B8%AD%E6%96%87)

  【架构图】

  [FFmpeg源代码结构图 - 解码](http://blog.csdn.net/leixiaohua1020/article/details/44220151)

  [FFmpeg源代码结构图 - 编码](http://blog.csdn.net/leixiaohua1020/article/details/44226355)

  【通用】

  [FFmpeg 源代码简单分析：av_register_all()](http://blog.csdn.net/leixiaohua1020/article/details/12677129)

  [FFmpeg 源代码简单分析：avcodec_register_all()](http://blog.csdn.net/leixiaohua1020/article/details/12677265)

  [FFmpeg 源代码简单分析：内存的分配和释放（av_malloc()、av_free()等）](http://blog.csdn.net/leixiaohua1020/article/details/41176777)

  [FFmpeg 源代码简单分析：常见结构体的初始化和销毁（AVFormatContext，AVFrame等）](http://blog.csdn.net/leixiaohua1020/article/details/41181155)

  [FFmpeg 源代码简单分析：avio_open2()](http://blog.csdn.net/leixiaohua1020/article/details/41199947)

  [FFmpeg 源代码简单分析：av_find_decoder()和av_find_encoder()](http://blog.csdn.net/leixiaohua1020/article/details/44084557)

  [FFmpeg 源代码简单分析：avcodec_open2()](http://blog.csdn.net/leixiaohua1020/article/details/44117891)

  [FFmpeg 源代码简单分析：avcodec_close()](http://blog.csdn.net/leixiaohua1020/article/details/44206699)

  【解码】

  [图解FFMPEG打开媒体的函数avformat_open_input](http://blog.csdn.net/leixiaohua1020/article/details/8661601)

  [FFmpeg 源代码简单分析：avformat_open_input()](http://blog.csdn.net/leixiaohua1020/article/details/44064715)

  [FFmpeg 源代码简单分析：avformat_find_stream_info()](http://blog.csdn.net/leixiaohua1020/article/details/44084321)

  [FFmpeg 源代码简单分析：av_read_frame()](http://blog.csdn.net/leixiaohua1020/article/details/12678577)

  [FFmpeg 源代码简单分析：avcodec_decode_video2()](http://blog.csdn.net/leixiaohua1020/article/details/12679719)

  [FFmpeg 源代码简单分析：avformat_close_input()](http://blog.csdn.net/leixiaohua1020/article/details/44110683)

  【编码】

  [FFmpeg 源代码简单分析：avformat_alloc_output_context2()](http://blog.csdn.net/leixiaohua1020/article/details/41198929)

  [FFmpeg 源代码简单分析：avformat_write_header()](http://blog.csdn.net/leixiaohua1020/article/details/44116215)

  [FFmpeg 源代码简单分析：avcodec_encode_video()](http://blog.csdn.net/leixiaohua1020/article/details/44206485)

  [FFmpeg 源代码简单分析：av_write_frame()](http://blog.csdn.net/leixiaohua1020/article/details/44199673)

  [FFmpeg 源代码简单分析：av_write_trailer()](http://blog.csdn.net/leixiaohua1020/article/details/44201645)

  【其它】

  [FFmpeg源代码简单分析：日志输出系统（av_log()等）](http://blog.csdn.net/leixiaohua1020/article/details/44243155)

  [FFmpeg源代码简单分析：结构体成员管理系统-AVClass](http://blog.csdn.net/leixiaohua1020/article/details/44268323)

  [FFmpeg源代码简单分析：结构体成员管理系统-AVOption](http://blog.csdn.net/leixiaohua1020/article/details/44279329)

  [FFmpeg源代码简单分析：libswscale的sws_getContext()](http://blog.csdn.net/leixiaohua1020/article/details/44305697)

  [FFmpeg源代码简单分析：libswscale的sws_scale()](http://blog.csdn.net/leixiaohua1020/article/details/44346687)

  [FFmpeg源代码简单分析：libavdevice的avdevice_register_all()](http://blog.csdn.net/leixiaohua1020/article/details/41211121)

  [FFmpeg源代码简单分析：libavdevice的gdigrab](http://blog.csdn.net/leixiaohua1020/article/details/44597955)

  【脚本】

  [FFmpeg源代码简单分析：makefile](http://blog.csdn.net/leixiaohua1020/article/details/44556525)

  [FFmpeg源代码简单分析：configure](http://blog.csdn.net/leixiaohua1020/article/details/44587465)

  【H.264】

  [FFmpeg的H.264解码器源代码简单分析：概述](http://blog.csdn.net/leixiaohua1020/article/details/44864509)

### flv

- [Flv.js全面解析](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/flv)
- [Flv文档使用随记](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/flv)
- [FLV文件格式](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/flv)
- [Flv.js源码-IO部分](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/flv)
- [Flv.js源码-flv-demuxer.js](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/flv)

### MSE

- [Media Source Extensions](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/MSE)

### WebRTC

- [WebRTC](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/WebRTC)
- [WebRTC直播](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/WebRTC)
- [关于视频会议系统（WebRTC）的反思](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/WebRTC)

### hls

- [怎么搭建hls低延时直播（lowlatency hls）](https://blog.csdn.net/impingo/article/details/102558792)

### JavaCV

- [使用JavaCV实现海康rtsp转rtmp实现无插件web端直播（无需转码，低资源消耗）](https://blog.csdn.net/weixin_40777510/article/details/103764198?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7)

### rtmp

- [FFmpeg RTMP推HEVC/H265流](https://blog.csdn.net/smallhujiu/article/details/81703434)
- [分布式直播系统（四）【nginx-rtmp流媒体直播服务器单集群实现方式】](https://blog.csdn.net/impingo/article/details/100379853)

### rtsp

- [掘金：clouding：浏览器播放rtsp视频流解决方案](https://juejin.im/post/5d183a71f265da1b6e65b8ff)
  [利用JAVACV解析RTSP流，通过WEBSOCKET将视频帧传输到WEB前端显示成视频](https://www.freesion.com/article/4840533481/)
  [CSDN：zctel：javacv](https://blog.csdn.net/u013947963/category_9570094.html)
  [CSDN：斑马jio：JavaCV转封装rtsp到rtmp（无需转码，低资源消耗）](https://blog.csdn.net/weixin_40777510/article/details/103764198?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-7)
  [博客园：之乏：流媒体](https://www.cnblogs.com/zhifa/tag/%E6%B5%81%E5%AA%92%E4%BD%93/)
  [博客园：断点实验室：ffmpeg播放器实现详解 - 视频显示](https://www.cnblogs.com/breakpointlab/p/13309393.html)
  [Gitee：chengoengvb：RtspWebSocket](https://gitee.com/yzfar/RtspWebSocket)

### video

- [video标签在不同平台上的事件表现差异分析](https://segmentfault.com/a/1190000023519979)

### nginx-rtmp-module

- [Nginx-rtmp 直播媒体实时流实现](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-rtmp-module)
- [nginx搭建RTMP视频点播、直播、HLS服务器](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-rtmp-module)
- [rtmp-nginx-module实现直播状态、观看人数控制](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-rtmp-module)
- [实现nginx-rtmp-module多频道输入输出与权限控制](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-rtmp-module)
- [直播流媒体入门(RTMP篇)](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-rtmp-module)

### nginx-http-flv-module

- [nginx-http-flv-module](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/nginx-http-flv-module)

个人总结的思维导图：

- [流媒体](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE)
- [流媒体，flv.js，MSE](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93/%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE)

其他博文：

- [Nginx-rtmp rtmp、http-flv、http-ts、hls、hls+ 配置说明](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93)

- [知乎：chapin：基于 H5 的直播协议和视频监控方案](https://zhuanlan.zhihu.com/p/100519553?utm_source=wechat_timeline)

- [前端 Video 播放器 | 多图预警](https://juejin.im/post/5f0e52fe518825742109d9ee)

- [分布式直播系统（三）【Nginx-rtmp rtmp、http-flv、http-ts、hls、hls+ 配置说明】](https://blog.csdn.net/impingo/article/details/99703528)

- [流媒体相关介绍](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93)

- [在HTML5上开发音视频应用的五种思路](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93)

- [流媒体资源](https://gitee.com/AiShiYuShiJiePingXing/lovebetterworld/tree/master/%E6%B5%81%E5%AA%92%E4%BD%93)
