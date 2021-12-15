参考
 [w3c media-source](https://w3c.github.io/media-source/)
 [Media Source 系列 - 使用 Media Source Extensions 播放视频](https://www.jackpu.com/media-source-xi-lie/)
 [全面进阶 H5 直播](https://www.villainhr.com/page/2017/03/31/全面进阶 H5 直播)
 [无 Flash 时代，让直播拥抱 H5（MSE篇）](https://www.villainhr.com/page/2017/10/10/无 Flash 时代，让直播拥抱 H5（MSE篇）)
 [使用 MediaSource 搭建流式播放器](https://zhuanlan.zhihu.com/p/26374202)

简书：合肥懒皮：[H5直播系列二 MSE(Media Source Extensions)](https://www.jianshu.com/p/1bfe4470349b)



**MSE**：Media Source Extension，媒体源扩展是为了让HTML5支持流媒体操作（在用户层面就是播放）的一个标准。在计算机领域，标准一词和技术、解决方案都是等价的。

# 一、MSE 意义

## [粗识 HTML5 video 标签和MSE媒体源扩展](https://blog.csdn.net/dong_daxia/article/details/80335849)

以往用户在浏览网页内容尤其是视频内容时，需要使用像Adobe Flash或是微软的Silverlight这样的插件，播放视音频内容即使是电脑小白也知道，需要媒体播放器的支持，前面提到的插件就是起到媒体播放器的作用。但是使用插件这样的方式是很不便捷且很不安全的，一些不法分子会在这些插件上动手脚。因此W3C的最新的HTML5标准中，定义了一系列新的元素来避免使用插件，其中就包含了<video>标签这一大名鼎鼎的元素。

正是使用了<video>标签，支持HTML5的浏览器得以实现无插件就原生支持播放媒体内容，但是对媒体内容的格式有所限制。说到媒体内容，就自然地需要谈到媒体的封装格式和编码格式，这里总结一下，原视频文件通过编码来压缩文件大小，再通过封装将压缩视音频、字幕组合到一个容器内。



>  **我们可以把<video>标签看做拥有解封装和解码功能的浏览器自带播放器。随着视频点播、直播等视频业务的发展，视频通过流媒体传输协议（目前常用的有两种，MPEG-DASH和Apple的HLS）从服务器端分发给客户端，媒体内容进一步包含在一层传输协议中，这样<video>就无法识别了。以HLS为例，将源文件内容分散地封装到了一个个TS文件中。**
>
> 
>
> **仅靠<video>标签无法识别这样的TS文件，那么就引入了MSE拓展来帮助浏览器识别并处理TS文件，将其变回原来可识别的媒体容器格式，这样<video>就可以识别并播放原来的文件了。那么支持HTML5的浏览器就相当于内置了一个能够解析流协议的播放器。**



[hls.js](https://github.com/video-dev/hls.js)

> hls实际会先通过 ajax（loader 是可以完成自定义的） 请求 m3u8文件，然后会读取到文件的分片列表，以及视频的编码格式，时长等。随后会按照顺序(非 seek )去对分片进行请求，这些也是通过 ajax 请求二进制的文件，然后借助 [Media Source Extensions](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource) 将 buffer 内容进行合流，然后组成一个可播的媒体资源文件。

它定义了一个MediaSource对象来给HTMLMediaElement提供媒体数据的源。MediaSource对象拥有一个或多个SourceBuffer对象。浏览器应用通过添加数据片段（data segments）给SourceBuffer对象，然后根据系统性能和其他因素来适应不同质量的后续data。SourceBuffer对象里的数据是被组建成需要被编码和播放的音频、视频和文字信息的轨道缓冲（track buffer）格式的。被用于扩展的二进制流格式结构如下图所示：
![clipboard.png](https://segmentfault.com/img/remote/1460000011245417)









## [为什么国内大部分视频厂商不对PC开放HTML5?](https://www.zhihu.com/question/55247875)

视频源存在兼容性问题。原生的 HTML5 <video> 元素在 Windows PC 上仅支持 mp4 （H.264 编码）、webm、ogg 等格式视频的播放。而由于历史遗留问题（HTML5 视频标准最终被广泛支持以前，Flash 在 Web 视频播放方面有着统治地位），视频网站的视频源和转码设置，很多都高清源都是适用于 Flash 播放的 FLV 格式，只有少量低清晰度视频是 mp4 格式，webm 和 ogg 更是听都没听说过。比如优酷只有高清和标清才有 MP4 源，超清、1080P 等，基本都是 FLV 和 HLS（M3U8）的视频源（在 Windows PC 上支持 M3U8 比支持 FLV 更复杂，我们不做过多赘述）。而腾讯视频，因为转型 MP4 比较早，视频源几乎全部都是 MP4 和 HLS，所以现在可以在 Mac OS X 上率先支持 PC Web 端的 HTML5 播放器（Safari 下 HLS、Chrome 下 MP4）。



但是 HTML5 是不是就真的没办法播放 FLV 等格式视频了呢？不是。解决方案是 MSE，Media Source Extensions，就是说，HTML5 <video> 不仅可以直接播放上面支持的 mp4、m3u8、webm、ogg 格式，还可以支持由 JS 处理过后的视频流，这样我们就可以用 JS 把一些不支持的视频流格式，转化为支持的格式（如 H.264 的 mp4）。B 站开源的 flv.js 就是这个技术的一个典型实现。B 站的 PC HTML5 播放器，就是**用 MSE 技术，将 FLV 源用 JS 实时转码成 HTML5 支持的视频流编码格式**（其实就一个文件头的差异（这里文件头改成容器。感谢评论区谦谦的指教，是容器的差异，容器不只是文件头）），提供给 HTML5 播放器播放。



一些人问我为什么不直接采用 MP4 格式，并表示对 FLV 格式的厌恶。这个问题一方面是历史遗留问题，由于视频网站前期完全依赖 Flash 播放而选择 FLV 格式；另一方面，如果仔细研究过 FLV/MP4 封装格式，你会发现 FLV 格式非常简洁，而 MP4 内部 box 种类繁杂，结构复杂固实而又有太多冗余数据。FLV 天生具备流式特征适合网络流传输，而 MP4 这种使用最广泛的存储格式，设计却并不一定优雅。



## [Media Source Extensions](https://algate.coding.me/2018/01/04/Media Source Extensions/)

我们已经可以在 Web 应用程序上无插件地播放视频和音频了。但是，现有架构过于简单，只能满足一次播放整个曲目的需要，无法实现拆分/合并数个缓冲文件。流媒体直到现在还在使用 Flash 进行服务，以及通过 RTMP 协议进行视频串流的 Flash 媒体服务器。



MSE 使我们可以把通常的单个媒体文件的 src 值替换成引用 MediaSource 对象（一个包含即将播放的媒体文件的准备状态等信息的容器），以及**引用多个 SourceBuffer 对象（代表多个组成整个串流的不同媒体块）的元素。MSE 让我们能够根据内容获取的大小和频率，或是内存占用详情（例如什么时候缓存被回收），进行更加精准地控制**。 它是基于它可扩展的 API 建立自适应比特率流客户端（例如DASH 或 HLS 的客户端）的基础。



Download ---》 Response.arrayBuffer(适用fetch/xhr等异步获取流媒体数据) ---》 SourceBuffer(添加到MediaSource的buffer中) ---》 <vedio/> or <autio/>



# Media Source Extensions API

参考链接：[Media Source Extensions API](https://developer.mozilla.org/zh-CN/docs/Web/API/Media_Source_Extensions_API)

## MSE 标准

媒体源扩展（MSE）实现后，情况就不一样了。MSE 使我们可以把通常的单个媒体文件的 `src` 值替换成引用 `MediaSource` 对象（一个包含即将播放的媒体文件的准备状态等信息的容器），以及引用多个 `SourceBuffer` 对象（代表多个组成整个串流的不同媒体块）的元素。MSE 让我们能够根据内容获取的大小和频率，或是内存占用详情（例如什么时候缓存被回收），进行更加精准地控制。 它是基于它可扩展的 API 建立自适应比特率流客户端（例如DASH 或 HLS 的客户端）的基础。



在现代浏览器中创造能兼容 MSE 的媒体（assets）非常费时费力，还要消耗大量计算机资源和能源。此外，还须使用外部实用程序将内容转换成合适的格式。虽然浏览器支持兼容 MSE 的各种媒体容器，但采用 H.264 视频编码、AAC 音频编码和 MP4 容器的格式是非常常见的，且一定兼容。MSE 同时还提供了一个 API，用于运行时检测容器和编解码是否受支持。

如果没有精确的控制时间、媒体质量和内存释放等需求，使用 [`video`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video) 和 [`source`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/source) 是一个更加简单但够用的方案。

## DASH

DASH（Dynamic Adaptive Streaming over HTTP ）是一个规范了自适应内容应当如何被获取的协议。它实际上是建立在 MSE 顶部的一个层，用来构建自适应比特率串流客户端。虽然已经有一个类似的协议了（例如 HTTP 串流直播（HLS）），但 DASH 有最好的跨平台兼容性。



DASH 将大量逻辑从网络协议中移出到客户端应用程序逻辑中，使用更简单的 HTTP 协议获取文件。 这样就可以用一个简单的静态文件服务器来支持 DASH，这对CDN也很友好。这与之前的流传输解决方案形成鲜明对比，那些流解决方案需要昂贵的许可证来获得非标准的客户端/服务器协议才能实现。

DASH 的两个最常见的用例涉及“点播”或“直播”观看内容。点播功能让开发者有时间把媒体文件转码出多种不同的分辨率质量。

实时处理内容会引入由转码和播发带来的延迟。因此 DASH 并不适用于类似 [WebRTC](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) 的即时通讯。但它可以支持比 WebRTC 更多的客户端连接。



## 媒体源扩展接口

[`MediaSource`](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaSource)

代表了将由 [`HTMLMediaElement`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement) 对象播放的媒体资源。

[`SourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/SourceBuffer)

代表了一个经由 `MediaSource` 对象被传入 [`HTMLMediaElement`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement) 的媒体块。

[`SourceBufferList`](https://developer.mozilla.org/zh-CN/docs/Web/API/SourceBufferList)

列出多个 `SourceBuffer` 对象的简单的容器列表。

[`VideoPlaybackQuality`](https://developer.mozilla.org/zh-CN/docs/Web/API/VideoPlaybackQuality)

包含了有关正被 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video) 元素播放的视频的质量信息，例如被丢弃或损坏的帧的数量。由 [`HTMLVideoElement.getVideoPlaybackQuality()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLVideoElement/getVideoPlaybackQuality) 方法返回。

[`TrackDefault`](https://developer.mozilla.org/zh-CN/docs/Web/API/TrackDefault)

为在媒体块的[初始化段（initialization segments）](http://w3c.github.io/media-source/#init-segment)中没有包含类型、标签和语言信息的轨道，提供一个包含这些信息的 [`SourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/SourceBuffer)。

[`TrackDefaultList`](https://developer.mozilla.org/zh-CN/docs/Web/API/TrackDefaultList)

列出多个 `TrackDefault` 对象的简单的容器列表。



## 其他接口的扩展

[`URL.createObjectURL()`](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)

​	创建一个指向一个 `MediaSource` 对象的 URL。要求此 URL 可以被指定为一个用来播放媒体流的 HTML 媒体元素的 `src` 的值。

[`HTMLMediaElement.seekable`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement/seekable)

​	当一个 `MediaSource` 对象被 HTML 媒体元素播放时，此属性将返回一个包含用户能够在什么时间范围内进行调整的对象 [`TimeRanges`](https://developer.mozilla.org/zh-CN/docs/Web/API/TimeRanges)。

[`HTMLVideoElement.getVideoPlaybackQuality()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLVideoElement/getVideoPlaybackQuality)

​	针对正在播放的视频，返回一个 [`VideoPlaybackQuality`](https://developer.mozilla.org/zh-CN/docs/Web/API/VideoPlaybackQuality) 对象。

[`AudioTrack.sourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioTrack/sourceBuffer), [`VideoTrack.sourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/VideoTrack/sourceBuffer), [`TextTrack.sourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/TextTrack/sourceBuffer)

​	返回创建了相关轨道的 [`SourceBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/API/SourceBuffer)。



# Demo

## [WebSocket+MSE——HTML5 直播技术解析](https://zhuanlan.zhihu.com/p/27248188)

