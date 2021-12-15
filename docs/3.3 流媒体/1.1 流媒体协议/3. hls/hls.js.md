hls.js是一个JavaScript库，可实现HTTP Live Streaming客户端。它依靠HTML5视频和MediaSource扩展进行播放。 它通过将MPEG-2传输流和AAC / MP3流转换为ISO BMFF（MP4）片段来工作。如果在浏览器中可用，则可以使用Web Worker异步执行此转换。WWDC2016期间宣布，hls.js还支持HLS + fmp4 hls.js不需要任何播放器，它可以直接在标准HTML <video>元素上运行。 hls.js用ECMAScript6编写，并使用Babel在ECMAScript5中转译。



- ##### HLS（HTTP Live Streaming）

苹果推出的解决方案，将视频分成 5-10 秒的视频小分片，然后用 M3U8 索引表进行管理。由于客户端下载到的视频都是 5-10 秒的完整数据，故视频的流畅性很好，但也同样引入了很大的延迟（HLS 的一般延迟在 10-30s 左右）。相比于 FLV，HLS 在iPhone 和大部分 Android 手机浏览器上的支持非常给力，所以常用于 QQ 和微信朋友圈的 URL 分享。



**RTMP、HLS、HTTP-FLV 协议对比如下图所示：**

![img](https://img2020.cnblogs.com/blog/1074523/202003/1074523-20200324095018765-1924929212.png)

### 直播原理

**我的困惑：** 想要实现直播，需要经历怎样的过程？

**如果用一句话描述整体过程，其实就是直播时，主播端将直播流推向服务器，用户端发起请求从服务器拉视频流过来解码播放，流程如下图所示：**

![img](https://img2020.cnblogs.com/blog/1074523/202003/1074523-20200324095029970-205010266.png)

**第一部分就是视频主播端的操作：视频采集处理后推流到流媒体服务器。**

- 首先从前端采集设备中获得原始的音频、视频数据；
- 为了增强额外效果，对音频进行混音、降噪等处理，可为视频打上时间戳、添加Logo水印或增加滤镜；
- 随后对音频、视频进行编码，通过编码压缩满足其在互联网上实时传输的需求；
- 编码后就可以把各种多媒体内容（视频、音频、字幕等）盛放在同一个容器里，也就是所谓的封装，使得不同多媒体内容可同步播放，与此同时还提供了索引；
- 最后就是通过流传输协议将封装好的内容推送到流媒体服务器上；

**第二部分就是流媒体服务器：负责把从第一部分接收到的流进行处理并分发给用户。**

流媒体服务器的主要功能是对流媒体内容进行采集（接收推流）、缓存、调度和传输播放（以流式协议实现用户分发）。

> **典型的流媒体服务器：**

> - 微软的Windows Media Service（WMS）：它采用MMS协议接收、传输视频，采用Windows Media Player（WMP）作为前端播放器；
> - RealNetworks公司的Helix Server：采用RTSP/RTP协议接收、传输视频，采用Real Player作为播放前端播放器；
> - Adobe公司的Flash Media Server：采用RTMP（RTMPT/RTMPE/RTMPS）协议接收、传输视频，采用Flash Player作为前端播放器；

**第三部分就是用户：只需要拥有支持对应流媒体传输协议的播放器即可。**

这一部分其实就是我们前端需要实现的，如何在移动端的内嵌h5页面中实现直播流的播放。所以我们只需要关注后端是通过什么协议给我们返回直播流以及我们如何有效的播放就可以了~

### 客户端直播插件

**我的困惑：** 除了采用h5原生的`<video></<video>`标签，我们还能用什么插件实现视频直播，不同插件之间有什么区别？

**经过我暴风式搜索后找到三款常用并且支持实时流媒体播放的客户端插件（hls.js、video.js、vue-video-player），下面我们一个个道来。**

- ##### hls.js

hls.js是一个可以实现HTTP实时流媒体客户端的js库，主要依赖于`<video></<video>`标签和`Media Source Extensions`API。它的工作原理是将MPEG2传输流和AAC/MP3流转换成ISO BMFF (MP4)片段。由于hls.js是基于标准的`<video></<video>`标签实现播放，所以它不需要额外的播放器。

**优点：** 包比较小，很纯净，UI可以根据自己的业务自行扩展，功能可以根据需求进行封装，比较灵活，而且专业直播HLS协议流。
**缺点：** 对于常规的通用性播放器没有封装好的UI，功能上也需要自己调API实现，协议单一，只支持HLS。

- ##### video.js

video.js是一个基于h5的网络视频播放器，支持h5视频、现代流媒体格式（MP4、WebM、HLS、DASH等）以及YouTube、Vimeo，甚至连flash也支持(通过插件实现，后面会详细说明)，可在桌面端或移动端实现视频播放。

**优点：** 支持多种格式的流媒体播放，浏览器不支持时可实现优雅降级；专门有一套针对直播流的UI；插件机制强大，目前社区已有数百个皮肤和插件供下载；兼容性好，几乎支持所有桌面及移动端的浏览器。
**缺点：** 包比较大，实现hls直播的时候其实是内嵌了hls.js的代码，由于封装好UI和功能，使其不够灵活，修改UI时需要通过插件实现。

- ##### vue-video-player

vue-video-player其实就是将video.js集成到了Vue中，在Vue项目中使用起来会更方便。

### 移动端内嵌h5实现视频直播

**1、技术选型：**

- 传输协议——由于后端支持同时返回HLS协议和RTMP协议的直播流，结合考虑HLS协议的高延时问题和RTMP协议的兼容性问题，本项目决定采用向下兼容的方式实现，默认使用RTMP协议直播，当浏览器不支持时降级使用HLS协议播放。
- 直播插件——本项目基于Vue实现，并且业务逻辑为常规直播操作，无特殊需求，从开发效率、稳定性及兼容性出发，决定采用vue-video-player插件实现。

**2、vue-video-player安装与引入：**

- CDN：

```html
<link rel="stylesheet" href="path/to/video.js/dist/video-js.css"/>
<script type="text/javascript" src="path/to/video.min.js"></script>
<script type="text/javascript" src="path/to/vue.min.js"></script>
<script type="text/javascript" src="path/to/dist/vue-video-player.js"></script>
<script type="text/javascript">
  Vue.use(window.VueVideoPlayer)
</script>
```

- NPM（支持全局/按需引入）：`npm install vue-video-player --save`

  

**全局引入**

```javascript
import Vue from 'vue'
import VueVideoPlayer from 'vue-video-player'

// 引入videojs样式
import 'video.js/dist/video-js.css'
// 自定义样式引入，并为<video-player>添加对应类名即可，例如vjs-custom-skin
// import 'vue-video-player/src/custom-theme.css'

Vue.use(VueVideoPlayer, /* {
  options: 全局默认配置,
  events: 全局videojs事件
} */)
```

**按需引入**

```javascript
// 引入videojs样式
import 'video.js/dist/video-js.css'

import { videoPlayer } from 'vue-video-player'

export default {
  components: {
    videoPlayer
  }
}
```

**3、video.js插件扩展：** 当已有插件（video.js插件集合：https://videojs.com/plugins/）无法满足需求时可对已有插件进行扩展或自行开发video.js插件

```javascript
import videojs from 'video.js'

// videojs plugin
const Plugin = videojs.getPlugin('plugin')
class ExamplePlugin extends Plugin {
  // something...
}
videojs.registerPlugin('examplePlugin', ExamplePlugin)

// videojs language
videojs.addLanguage('es', {
  Pause: 'Pausa',
  // something...
})

// more videojs api...

// vue component...
```

具体实现方式可参见：https://github.com/videojs/video.js/blob/master/docs/guides/plugins.md

**4、视频直播关键代码：**
`options`：[video.js options](https://github.com/videojs/video.js/blob/master/docs/guides/options.md)
`playsinline`：设置播放器在移动设备上不全屏`[ Boolean, default: false ]`
`customEventName`：自定义状态变更时的事件名`[ String, default: 'statechanged' ]`

```html
<template>
  <video-player
        class="video-player-box"
        ref="videoPlayer"
        :options="playerOptions"
        :playsinline="true"
        customEventName="customstatechangedeventname"
        @play="onPlayerPlay($event)"
        @pause="onPlayerPause($event)"
        @ended="onPlayerEnded($event)"
        @waiting="onPlayerWaiting($event)"
        @playing="onPlayerPlaying($event)"
        @loadeddata="onPlayerLoadeddata($event)"
        @timeupdate="onPlayerTimeupdate($event)"
        @canplay="onPlayerCanplay($event)"
        @canplaythrough="onPlayerCanplaythrough($event)"
        @statechanged="playerStateChanged($event)"
        @ready="playerReadied">
  </video-player>
</template>
export default {
    data() {
      return {
        playerOptions: {
          // 是否关闭音频
          muted: true,
          // 初始语言，默认为英语，code参见：https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry
          language: 'zh-CN',
          // 播放速度，指定后Video.js将显示一个控件(vjs-playback-rate类的控件)，允许用户从选项数组中选择播放速度
          playbackRates: [0.5, 1.0, 1.5, 2.0],
          // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值，表示长宽比例
          aspectRatio: '4:3',
          // 等同于原生<video>标签中的一组<source>子标签，可实现优雅降级；type 属性规定媒体资源的 MIME 类型，标准类型可参见：https://www.iana.org/assignments/media-types/media-types.xhtml；
          sources: [{
            type: "rtmp/flv",
            src: "rtmp://58.200.131.2:1935/livetv/hunantv"
          }, {
            type: "application/x-mpegURL",
            src: "http://ivi.bupt.edu.cn/hls/cctv1hd.m3u8"
          }],
          // 兼容顺序，默认值是['html5']，这意味着html5技术是首选，其他已注册的技术将按其注册的顺序在该技术之后添加。
          techOrder: ['flash'],
          // 在视频开始播放之前显示的图像的URL（封面），这通常是一个视频帧或自定义标题屏幕，一旦用户点击“播放”，图像就会消失。
          poster: require('../assets/test.jpg'),
        }
      }
    },
    mounted() {
      console.log('this is current player instance object', this.player)
    },
    computed: {
      player() {
        return this.$refs.videoPlayer.player
      }
    },
    methods: {
      // 各个事件监听
      onPlayerPlay(player) {
        // console.log('播放器播放!', player)
      },
      onPlayerPause(player) {
        // console.log('播放器暂停!', player)
      },
      // ...（此处省略多个事件监听函数）

      // 状态监听
      playerStateChanged(playerCurrentState) {
        // console.log('播放器当前状态更新', playerCurrentState)
      },

      // 监听播放器是否就绪
      playerReadied(player) {
        console.log('播放器已就绪', player)
        // 就绪后就可以调用播放器的一些方法
      }
    }
 }
```

> **踩坑小tips：**
> 播放 HLS 协议流，需要`videojs-contrib-hls`插件，但是直接引用即可，因为在安装`vue-video-player`插件时，`videojs-contrib-hls`是一并安装的；如果需要播放RTMP协议流，需要`videojs-flash`插件，也是直接引用就可以了（flash插件需要在hls之前引用）