相关内容参考地址：

- [使用flv.js快速搭建html5网页直播](https://blog.csdn.net/impingo/article/details/103077380)
- [B站视频开源代码flv.js的使用部署心得（代码案例应用）](https://blog.csdn.net/hj7jay/article/details/54906612)
- [使用flv.js做直播](https://www.cnblogs.com/burro/p/10149566.html)
- [**Flv.js** B 站 HTML5 播放器内核](https://www.oschina.net/p/flv-js)
- [FLV文件格式详解](http://blog.useasp.net/archive/2016/02/28/The-flv-video-file-format-specification-version-10-1.aspx)
- [flv格式详解+实例剖析](https://www.jianshu.com/p/7ffaec7b3be6)
- [FLV 实例分析](https://blog.csdn.net/y_z_hyangmo/article/details/79219996)
- [H5直播系列八 FLV文件格式](https://www.jianshu.com/p/1df4bc217dbd)
- [js对flv提取h264、aac音视频流](https://www.cnblogs.com/suyuanli/p/14196729.html)



Flv.js 是 HTML5 Flash 视频（FLV）播放器，纯原生 JavaScript 开发，没有用到 Flash。由 bilibili 网站开源。

Github地址：https://github.com/Bilibili/flv.js/

# 常见直播协议

- RTMP: 底层基于TCP，在浏览器端依赖Flash。
- HTTP-FLV: 基于HTTP流式IO传输FLV，依赖浏览器支持播放FLV。
- WebSocket-FLV: 基于WebSocket传输FLV，依赖浏览器支持播放FLV。WebSocket建立在HTTP之上，建立WebSocket连接前还要先建立HTTP连接。
- HLS: Http Live Streaming，苹果提出基于HTTP的流媒体传输协议。HTML5可以直接打开播放。
- RTP: 基于UDP，延迟1秒，浏览器不支持。



# **Flv.js概览**

一个实现了在 HTML5 视频中播放 FLV 格式视频的 JavaScript 库。它的工作原理是将 FLV 文件流转码复用成 ISO BMFF（MP4 碎片）片段，然后通过 [Media Source Extensions](https://w3c.github.io/media-source/) 将 MP4 片段喂进浏览器。

flv.js只做了一件事，在获取到FLV格式的音视频数据后通过原生的JS去解码FLV数据，再通过[Media Source Extensions](https://w3c.github.io/media-source/) API 喂给原生HTML5 Video标签。(HTML5 原生仅支持播放 mp4/webm 格式，不支持 FLV)

flash性能问题是长期以来被全世界人所诟病的，尤其是明年起chrome彻底抛弃flash，越来越多有直播需求的人产生焦虑。这就加速了html5播放器的发展，也使得人们对html5非插件式的播放器更加渴望。而flv.js就是这么一款可以利用html5的video标签将http-flv直播流实时播放的一个js版的播放器。



flv.js 为什么要绕一圈，从服务器获取FLV再解码转换后再喂给Video标签呢？原因如下：

1. 兼容目前的直播方案：目前大多数直播方案的音视频服务都是采用FLV容器格式传输音视频数据。
2. FLV容器格式相比于MP4格式更加简单，解析起来更快更方便。



HTML5 原生仅支持播放 mp4/webm 格式，flv.js 实现了在 HTML5 上播放 FLV 格式视频。



flv.js在获取到FLV格式的音视频数据后将 FLV 文件流转码复用成 ISO BMFF（MP4 碎片）片段，再通过Media Source Extensions API 传递给原生HTML5 Video标签进行播放。

flv.js 是使用 ECMAScript 6 编写的，然后通过 [Babel Compiler](https://babeljs.io/) 编译成 ECMAScript 5，使用 [Browserify](http://browserify.org/) 打包。



## Flv.js结构图

![](F:/lovebetterworld/流媒体/flv/flv.png)

flv.js 中的demux就是一套 FLV 媒体数据格式的解析器。



## 架构图

 

[![img](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706001317581-1716943373.png)](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706001317581-1716943373.png)



# **功能**

- FLV 容器，具有 H.264 + AAC 编解码器播放功能
- 多部分分段视频播放
- HTTP FLV 低延迟实时流播放
- FLV 通过 WebSocket 实时流播放
- 兼容 Chrome, FireFox, Safari 10, IE11 和 Edge
- 十分低开销，并且通过你的浏览器进行硬件加速



# API文档相关

```javascript
flvjs.isSupported()
// 查看当前浏览器是否支持flv.js，返回类型为布尔值

flvjs.createPlayer(mediaDataSource: MediaDataSource, config?: Config)

/* 创建一个Player实例，它接收一个MediaDataSource(必选), 一个Config(可选)，如：
var flvPlayer = flvjs.createPlayer({
   type: 'flv',
   url: 'http://example.com/flv/video.flv'
}); */
```

### MediaDataSource的字段列表如下，

[![img](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003330312-325000885.png)](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003330312-325000885.png)

这里说下最后一个segments字段（其余字段都很简单），它接收一个数组，类型为MediaSegment，MediaSegment的字段列表如下，

[![img](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003400574-247659967.png)](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003400574-247659967.png)

如果segments字段存在，transmuxer将把此MediaDataSource视为多片段源。在多片段模式下，将忽略MediaDataSource结构中的duration filesize url字段。
什么个意思呢，用白话说就是如果指定了segments字段那么之前指定的duration filesize url字段就不再生效了，将标志这是一个多片段合成一个的视频，进度条的总时长就等于各片段相加的和，所以每个片段的duration filesize一定要指定准确。



### Config

[![img](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003658119-2029801664.png)](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003658119-2029801664.png)



 

### **flvjs.getFeatureList()**

// 返回一些功能特性列表，比如是否支持FLV直播流、H264 MP4 视频文件等等，如下

[![img](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003816978-699660997.png)](https://img2020.cnblogs.com/blog/1516215/202007/1516215-20200706003816978-699660997.png)

 

# Flv.js兼容性

理论上只要是支持Media Source Extensions和ECMAScript 5的浏览器都是兼容flv.js的，浏览器对MSE的兼容情况如下图，从图中可以看出，flv.js的兼容性还是非常不错的。 需要指出的是iPhone版的Safari是不支持MSE的，所以在iPhone上只有hls是最理想的选择，而庆幸的是PC版和android版的浏览器大多都是支持MSE的，也就是说可以利用http-flv直播实现延时较低的效果。

![img](https://pic4.zhimg.com/80/v2-d22799434b9bd312bd6e7688d4c17690_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-dfe0429c69f0035aa33f25e648d6eb23_720w.jpg)

**如果你对兼容性要求非常高的话，HLS会是非常好的选择，而并非所有浏览器版本都支持HLS播放，但是你可以利用另外一个JS播放器项目（video.js）实现全平台的hls直播。**

**各个浏览器对HLS的支持情况：**

![img](https://picb.zhimg.com/80/v2-a3fbc74314471f2e9bc2da48ce7a4654_720w.jpg)

**所以，你可以将flv.js和video.js配合使用，针对不同平台实现最优的方案。**



# 直播服务器搭建

关于直播服务器搭建的流程，我在以前的博客里写了很多，感兴趣的可以参考[分布式直播系统（二）【搭建单点rtmp\http-flv\hls流媒体服务器】](https://blog.csdn.net/impingo/article/details/99131594)

当然也可以使用我的一键部署脚本安装：
https://github.com/im-pingo/pingos

```nginx
# 快速安装
git clone https://github.com/im-pingo/pingos.git

cd pingos

./release.sh -i

# 启动服务
cd /usr/local/pingos/
./sbin/nginx
12345678910
```

## 推流

### ffmpeg推流

```bash
ffmpeg -re -i 文件.mp4 -vcodec copy -acodec copy -f flv rtmp://ip地址/live/01
1
```

### OBS推流

Open Broadcaster Software（简称OBS）是一款直播流媒体内容制作软件。同时程序和其源代码都是免费的。

支持 OS X、Windows、Linux操作系统。适用于多种直播场景。满足大部分直播行为的操作需求（发布桌面、发布摄像头、麦克风、扬声器等等）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191115111036758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)

## flv.js搭建过程

### 下载链接

为了方便使用，我已经将flv.js编译打包好了，直接下载解压到你的网站目录引用即可。

1. **可以使用百度云盘下载**
   链接: https://pan.baidu.com/s/1ihTo15nsgfLqXKa0vyFt-w
   提取码: gd55
2. **也可以从github下载**

```bash
git clone https://github.com/im-pingo/h5player.git
```

将h5player复制到你的网站目录，h5player/flv目录下有个index.html文件，这里是js播放器接口的调用示例，你可以直接利用这个页面演示。

### flv.js Demo演示

Demo地址：http://player.pingos.io/flv

我已经搭建了一个完整的演示界面，你可以快速体验一把。

1. 输入http-flv的直播地址
2. 点击load按钮即可播放。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191115102729932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)

#### 播放器主要参数

| 参数              | 描述                 |
| ----------------- | -------------------- |
| enableStashBuffer | 是否开启播放器端缓存 |
| stashInitialSize  | 播放器端缓存         |
| isLive            | 是否为直播流         |
| hasAudio          | 是否播放声音         |
| hasVideo          | 是否播放画面         |

## 源码分析

- createPlayer 接收两个参数：媒体资源 mediaDataSource 和配置 optionalConfig，根据媒体类型( type 属性)创建一个播放器

- isSupported 实际上是 Features.supportMSEH264Playback()

- getFeatureList 实际上是 Features.getFeatureList()



### flv.js-flvjs 对象入口

它做了以下几件事：

1. 激活了 polyfill

2. 组合播放器实例以及相关方法：FlvPlayer 和 NativePlayer，createPlayer、isSupported、getFeatureList。`这里个人建议使用 Object.assign 完成组合`
   2.1 createPlayer 接收两个参数：媒体资源 mediaDataSource 和配置 optionalConfig，根据媒体类型( type 属性)创建一个播放器
   2.2 isSupported 实际上是 Features.supportMSEH264Playback()
   2.3 getFeatureList 实际上是 Features.getFeatureList()

3. 关联相关事件和错误、调试工具

4. 挂载到window对象导出

polyfill.js-Polyfill类-es6的polyfill

1. 引入了 Object.setPrototypeOf、Object.assign、promise 的 polyfill，包装在 install 函数里
2. 这一块完全可以交给 polyfill 库或者从 MDN 引，不是 flv.js 的重点

### features.js-Features 类-MSE 特征检测

**重点文件，可以知道浏览器目前支持 MSE 的哪些功能**



supportMSEH264Playback 判断全局上是否有 [MediaSource](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FMediaSource) 这个对象，并且需要支持 `video/mp4; codecs="avc1.42E01E,mp4a.40.2"`这种类型。

- flv.js 是将 flv 格式转换成 "avc1.42E01E,mp4a.40.2" 格式了。
- MDN 的 MediaSource 示例也给我们展示了如何通过 MediaSource 的方法和事件加载一个 mp4 文件。



supportNetworkStreamIO 通过创建一个 IOController 来判断加载器是否支持流。 （只能是 fetch-stream-loader 类型或 xhr-moz-chunked-loader 类型）。



supportNativeMediaPlayback 通过创建一个 [video 元素](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FHTMLVideoElement),利用它的 canPlayType 方法判断是否支持某种 mime 的数据

getFeatureList 获取支持的特性列表，分别是：

- mseFlvPlayback MSE是否支持
- networkStreamIO 数据流是否支持
- networkLoaderName 数据加载器名称
- mseLiveFlvPlayback MSE 流视频是否支持
- nativeMP4H264Playback 原生 MP4 格式是否支持
- nativeWebmVP8Playback 原生 [Webm](https://link.jianshu.com/?t=https%3A%2F%2Fzh.wikipedia.org%2Fzh-cn%2FWebM) VP8 格式是否支持
- nativeWebmVP9Playback 原生 Webm VP9 格式是否支持

### logging-control.js-LoggingControl类-调试控制器

这里涉及到繁琐的参数设置，并且使用 get 和 set 控制了读写过程，不具体介绍每个方法，主要是介绍用途和事件的使用。

1. 组合了 EventEmitter，采用发布-订阅模式管理调试，getConfig 方法可以获得所有调试选项，applyConfig 方法可以接受一个 config 对象来配置调试选项。
2. forceGlobalTag 是否开启强制全局标签和 globalTag 全局标签在 set 中使用了 _notifyChange 方法发布变化。
3. enableAll/enableDebug/enableVerbose/enableInfo/enableWarn/enableError 这六个方法是是否允许特定模式的 console，刚好对应原生的调试 API，同上会在 set 中发布了变化
4. _notifyChange 关键是利用 emitter 触发一个 change 事件，参数是所有调试配置。
   4.1 这里利用 listenerCount 方法让多个事件只触发一次。
5. registerListener(listener) 和 removeListener(listener) 是让 emitter 注册或移除事件监听。
   5.1 代码初始化的时候 new 了一个 EventEmitter 注入到 LoggingControl 中。

### exception.js

这个文件里有四个类，用来描述代码运行中的三类错误，其中 RuntimeException 是基类。

1. RuntimeException类-运行时错误，基类，拥有 _message 私有属性和 message、name 两个只读属性，以及一个 toString 方法用来描述完整的错误信息。

2. IllegalStateException类-无效状态，name 只读属性重写为 'IllegalStateException'

3. InvalidArgumentException类-无效参数，name 只读属性重写为 'InvalidArgumentException'

4. NotImplementedException-未实现功能，name 只读属性重写为 'NotImplementedException'

