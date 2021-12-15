# **实现思路：**

# 跨平台的HLS\DASH方案

**简介：**

HLS是Apple首先提出的流媒体分发协议，目前在苹果家族的整个产品都得到了比较好的支持，后来谷歌在Chrome浏览器和移动端浏览器也进行了原生支持，所以目前无论你是在PC还是移动端的浏览器基本都原生支持HLS协议进行播放视频，算是一个在移动端比较好的跨平台方案，同时微信内嵌的浏览器也都是原生支持的。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvdDBuT3BZYjV0NDJrZm9ycmJZVmhvZHlJdWlhNXh2UkswMmlhWlM5Tk1vVDd4Z1dneXRtWXp4NncvNjQw?x-oss-process=image/format,png)

## **缺点**：

延时比较大，由于HLS协议本身的切片原理，基本延迟都在10秒+，这对于一些低延时场景非常不友好，虽然HLS也在努力优化，但是想达到秒级延迟还是不现实的。

## **微信小程序演示效果：**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvb3hIWjc0NXRpYnZXYnkzS0dMSU5qVE11NVBkcnY5UFJMTWpYQVZQYlZGZ0tGb0RtVmliekFrZlEvNjQw?x-oss-process=image/format,png)

# 基于HTML5 Video和Audio的MSE方案

MSE即Media Source Extensions是一个W3C草案，其中桌面对MSE的支持比较好，移动端支持缓慢。MSE扩展了HTML5的Video和Audio标签能力，允许你通过JS来从服务端拉流提供到HTML5的Video和Audio标签进行播放。

MSE在各个浏览器的支持情况如下，目前看在PC端的浏览器支持比较友好，但在移动端浏览器支持这个接口目前还处于刚开始。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvQTJHdmZWVXBpY2ZwMDZIcXRZbHJXTGVOeHRqRVk2TmRMc25DaWMzMlBINWJTajFSTFhNaWNtQXZ3LzY0MA?x-oss-process=image/format,png)

MSE目前支持的视频封装格式是MP4，支持的视频编码是H.264和MPEG4,支持的音频编码是AAC和MP3，目前编码层的东西摄像机都支持比较友好，问题不大。封装格式的处理目前要么就是从服务端拉裸流过来，在Web前端合成MP4片段进行播放，要么在服务端提前转封装好直接喂给MSE接口，同时由于RTMP协议在CDN场景的大量使用，所以Web前端应该还支持解析FLV然后转成MP4片段，于是就产生了以下技术细类：

## **3.1方案：HTTP+FLV**

**简介：**

服务端经摄像头拉流转成FLV，然后客户端过来拉流即可，拉过来的流解封装下FLV然后转成MP4片段，再喂给MSE即可。这个事情现在有个开源项目就是bilibili的flv.js已经帮你实现了，你直接利用这个开源项目简单改改，就基本支持起来了，我们已经在用。

### **缺点：**

目前移动端的微信或者Chrome76我已经测试过，开始支持了，但是IOS的Safari浏览器没有支持，所以这个方案暂时在移动端完全支持起来有困难。

**演示：**

1. PC Web展示效果：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRva2FiU0VzSGtiaWFxcTRPblJ2dk5HUGpadGFoR01jWDlENVk5SE9jdWRBdDhwVjZlN3lXTTBFQS82NDA?x-oss-process=image/format,png)

2. 手机微信7.0.4和Android Chrome76演示效果：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvNFBrcDFDRTd0RndFcVVxOE1EQ0RpYUNFMWNKZDFDc013djIwaWI2ZUJ6eVlaY3Y4VnY4V2o1Q3cvNjQw?x-oss-process=image/format,png)

## **3.2方案：WebSocket+FLV**

**简介：**

方案和3.1目前差不多，就是将拉流协议换成Web的原生WebSocket协议而已，拉过来的FLV码流还是可以靠flv.js来进行转封装为Mp4片段，喂给MSE即可，相应的MP4片段也可以在服务端生成，然后直接用WebSocket拉过来即可，也就是3.3方案。

## **3.3方案：WebSocket+MP4**

**缺点：**

缺点就是要在服务端提前生成好MP4片段，转封装这块工作服务端需要处理好。

## **3.4方案：WebSocket+RTSP+H.264数据**

**简介：**

因为现在视频监控类设备目前支持最好的拉流协议基本就是RTSP协议，基本都进行了标准支持，因为视频监控领域有一个国际标准就是ONVIF标准，这个标准使用的拉流协议就是RTSP，所以视频监控不支持RTSP，就无法支持ONVIF，在国际就没有市场。

所以要是Web能直接通过RTSP拉流，那就非常友好，想做到这点比较难，因为Web的W3C标准就不支持RTSP协议，曲线救国的方案就是将RTSP协议放到Websocket协议里面进行透传，然后在服务端做一个Websocket到RTSP协议的代理转换协议，这样就可以在Web支持RTSP协议了，对于视频监控领域用户比较友好，一看就是熟悉的味道，相同的道理也可以在Web前端支持RTMP协议，基本的原理如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvNUxnZk13blhybFhEaWE5YmliV1RSU1BraWE0YmlhcVAzdTlMSGlhaWJHYWpNblNzdjhJUGJzVW1McWFRLzY0MA?x-oss-process=image/format,png)

### **缺点：**

需要服务端做相应的协议转换代理，拉过来的码流Web还是要进行相应的转成MP4片段，这些都是不小的开发工作量；

这个也有相应的开源项目，其中Web这边有个html5_rtsp_player开源项目，实现了RTSP客户端功能，你可以利用此框架直接播放RTSP直播流。此播放器把RTP协议下的H264/AAC再转换为ISO BMFF供video元素使用。Streamedian支持Chrome 23+, FireFox 42+, Edge 13+，以及Android 5.0+。不支持iOS和IE。

### **演示效果如下：**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvcFNXRFp1dloxUWQwRWQ1ZklwU2pKaWM3MUpOVzRwSnd3MURmSnNrV0JIOU94SFdpY1gwTTBIRkEvNjQw?x-oss-process=image/format,png)

 

# WebRTC方案

## **简介：**

WebRTC是一整套API，其中一部分供Web开发者使用，另外一部分属于要求浏览器厂商实现的接口规范。WebRTC解决诸如客户端流媒体发送、点对点通信、视频编码等问题。桌面浏览器对WebRTC的支持较好，WebRTC也很容易和Native应用集成。

WebRTC实现了浏览器P2P的实时通信，其中可以通过调用相应的Web API采集视频进行推流，如果放到视频监控，我们可以把这一段在嵌入式摄像头上实现，将摄像机的编码视频数据采集出来，然后直接发送出去即用摄像头模拟P2P的推流端，另外一端在Web浏览器上用相应接口解码和渲染。我们在自家摄像头预研过这套方案，目前看是可以的。延时非常小，播放非常稳定，同时WebRTC目前在跨平台方面支持比较好。

## **演示效果：**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvd2NQdWx5d2ljcDNSY2wyR1N6TFBZcUVzS1RpY2ljWGxFemJpYnd2cWJLc0E5Umo1NGxDSU5CVDIyUS82NDA?x-oss-process=image/format,png)

# **WebSocket HTTP + WebGL Canvas2D+ FFmpeg+WebAssembly**

## 简介：

WebAssembly 是一种新的编码方式，可以在现代的网络浏览器中运行 － 它是一种低级的类汇编语言，具有紧凑的二进制格式，并为其他语言提供一个编译目标，以便它们可以在 Web 上运行。它也被设计为可以与 JavaScript 共存，允许两者一起工作。近几年已经被各主流浏览器所广泛支持，支持情况：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvVWQ4Tm9RVE5ja0EzQTNHYWZiVVFMWGFpYnpaaWFnZmZKamVSbVU3R2tqb1psaWFtVEtaSzNucGljdy82NDA?x-oss-process=image/format,png)

它的大概原理：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvNWhQQ3VBczBhMDVhUWliZHdzRGZQZm1sdEFxZ2hpYTNaS1I4em5Oc2xWRFhVMFRoelh5bDZjU2cvNjQw?x-oss-process=image/format,png)

利用这种技术可以将C/C++库进行前端移植，比如WebAssembly 技术可以帮我们把 FFmpeg 运行在浏览器里，其实就是通过 Emscripten 工具把我们按需定制、裁剪后的 FFmpeg 编译成 Wasm 文件，加载进网页，与 JavaScript 代码进行交互。

这样Wasm 用于从 JavaScript 接收WebSocket或者HTTP-FLV 直播流数据，并对这些数据利用FFmpeg进行解码，然后通过回调的方式把解码后的 YUV 视频数据和 PCM 音频数据传送回 JavaScript，并最终通过 WebGL 在 Canvas 上绘制视频画面，同时通过 Web Audio API 播放音频。

## **缺点：**

前端消耗性能还是比较大，Web前端播放H265的1080P视频还是比较吃力的，同时想在前端播放多路视频基本是不现实的，所以这个应用场景还是局限在特殊的应用场景，不能通用。

我们当时的实践是利用这种技术在Web界面播放鱼眼摄像头视频，这种摄像头视频是几路合成的，如果借助原始Web接口是无法播放的，所以播放器解码必须是我们自己C++写的，然后借助WebAssembly技术允许js接口调用解码播放。

## **效果演示：**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9MTUREaWFWaWF3M0tLdlVnTGFyMHhKUjF6OXc0dnlBRXRvZmR6eHFEa0JTTlBoeHA1bUZhTGRWOTJFUmVtdWdmQ3Z2WmhYNWtCdmxHTXVIZ0lCVVlpYVczQS82NDA?x-oss-process=image/format,png)

------

# 总结：

目前在web浏览器上想播放音视频主要的技术大类就是上面四种：

1. 插件化的技术虽然可以实现各个浏览器的播放音视频，但是即将淘汰；

2. HLS/DASH浏览器虽然原生支持，跨平台比较好，但是延时太大，对于低延时领域不适用；

3. 基于MSE的技术，虽然可以大大降低延时，适用直播和点播，但是苹果系列产品目前还未完全支持，只能部分使用；

4. 基于WebRTC技术，非常适用于实时视频，但是开发量比较大，对于视频监控等低延时交互领域有点杀鸡焉用牛刀的感觉，而且技术处于快速发展阶段，很多还不成熟，对于小公司有一定的门槛；

5. 基于WebAssembly将C/C++音视频解码库前端移植化，这个在一些特殊应用场景可以使用比如浏览器要播放H265音视频，但是弊端是前端跑起来比较重，渲染不了几路视频，性能优化还有待提升，但是是可以突破浏览器和操作系统接口的隔阂。

所以目前来看想在Web上做音视频操作，浏览器的原生支持还远远不够，相比较开发APP还是缺乏一定的灵活性，不仅有一定的限制而且需要兼容处理的事情非常多，想一招解决你的需求还是有困难，所以还是需要上述几种技术综合搭配使用来解决Web播放音视频需求。不过后面相信浏览器会在这方面突破，毕竟5G要来了，浏览器厂商会加紧布局。

 

------

本篇文章参考网址和项目：

https://github.com/ty6815 

https://github.com/gwuhaolin/blog

https://github.com/Streamedian/html5_rtsp_player

https://github.com/bilibili/flv.js

https://mp.weixin.qq.com/s/EC8Yd74HEoIO2QxJe8-iNQ

https://blog.csdn.net/vn9PLgZvnPs1522s82g/article/details/96405736

https://www.jianshu.com/p/1bfe4470349b