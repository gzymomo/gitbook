[HTML5里video标签支持哪些格式的视频文件及其遇到的坑](http://www.yanzuoguang.com/article/591)

# HTML5里video标签支持哪些格式的视频文件及其遇到的坑

一共支持三种格式： Ogg、MPEG4、WebM。但这三种格式对于浏览器的兼容性却各不同。

- NO: 代表不支持这款浏览器
- X.0+: 表示支持这款及版本更高的浏览器

| 格式  | IE   | FireFox | Opera | Chrome | Safari |
| ----- | ---- | ------- | ----- | ------ | ------ |
| Ogg   | No   | 3.5+    | 10.5+ | 5.0+   | No     |
| MPEG4 | 9.0+ | No      | No    | 5.0+   | 3.0+   |
| WebM  | No   | 4.0+    | 10.6+ | 6.0+   | No     |

重点：比如MP4格式，MP4只是一个容器，里面还有一个叫编码器的东西。格式虽然都是MP4但是html中只支持H.264的编码格式。所以要用软件来转码。

- MP4 = MPEG 4文件使用 H264 视频编解码器和AAC音频编解码器
- WebM = WebM 文件使用 VP8 视频编解码器和 Vorbis 音频编解码器
- Ogg = Ogg 文件使用 Theora 视频编解码器和 Vorbis音频编解码器

## video标签相关事件 、方法、属性汇总。

<video>标签的属性:

- src ：视频的属性
- poster：视频封面，没有播放时显示的图片
- preload：预加载
- autoplay：自动播放
- loop：循环播放
- controls：浏览器自带的控制条
- width：视频宽度
- height：视频高度

## html 代码

```html
<video id="media" src="http://www.sundxs.com/test.mp4" controls width="400px" heigt="400px"></video>
//audio和video都可以通过JS获取对象,JS通过id获取video和audio的对象
```

## `<video>`标签的属性

- 获取video对象

```js
Media = document.getElementById("media");
```

- Media方法和属性：

```js
HTMLVideoElement和HTMLAudioElement 均继承自HTMLMediaElement
Media.error; //null:正常
Media.error.code; //1.用户终止 2.网络错误 3.解码错误 4.URL无效
```

## 网络状态

```js
Media.currentSrc; //返回当前资源的URL
Media.src = value; //返回或设置当前资源的URL
Media.canPlayType(type); //是否能播放某种格式的资源
Media.networkState; //0.此元素未初始化 1.正常但没有使用网络 2.正在下载数据 3.没有找到资源
Media.load(); //重新加载src指定的资源
Media.buffered; //返回已缓冲区域，TimeRanges
Media.preload; //none:不预载 metadata:预载资源信息 auto:
```

## 准备状态

```js
Media.readyState;//1:HAVE_NOTHING 2:HAVE_METADATA 3.HAVE_CURRENT_DATA 4.HAVE_FUTURE_DATA 5.HAVE_ENOUGH_DATA
Media.seeking; //是否正在seeking
```

## 回放状态

```js
Media.currentTime = value; //当前播放的位置，赋值可改变位置
Media.startTime; //一般为0，如果为流媒体或者不从0开始的资源，则不为0
Media.duration; //当前资源长度 流返回无限
Media.paused; //是否暂停
Media.defaultPlaybackRate = value;//默认的回放速度，可以设置
Media.playbackRate = value;//当前播放速度，设置后马上改变
Media.played; //返回已经播放的区域，TimeRanges，关于此对象见下文
Media.seekable; //返回可以seek的区域 TimeRanges
Media.ended; //是否结束
Media.autoPlay; //是否自动播放
Media.loop; //是否循环播放
Media.play(); //播放
Media.pause(); //暂停
```

## 视频控制

```js
Media.controls;//是否有默认控制条
Media.volume = value; //音量
Media.muted = value; //静音
```

# TimeRanges(区域)对象

```js
TimeRanges.length; //区域段数
TimeRanges.start(index) //第index段区域的开始位置
TimeRanges.end(index) //第index段区域的结束位置
```

## `<video>`标签的事件

```js
var eventTester = function(e){
Media.addEventListener(e,function(){
console.log((new Date()).getTime(),e)
},false);
}
eventTester("loadstart"); //客户端开始请求数据
eventTester("progress"); //客户端正在请求数据
eventTester("suspend"); //延迟下载
eventTester("abort"); //客户端主动终止下载（不是因为错误引起）
eventTester("loadstart"); //客户端开始请求数据
eventTester("progress"); //客户端正在请求数据
eventTester("suspend"); //延迟下载
eventTester("abort"); //客户端主动终止下载（不是因为错误引起），
eventTester("error"); //请求数据时遇到错误
eventTester("stalled"); //网速失速
eventTester("play"); //play()和autoplay开始播放时触发
eventTester("pause"); //pause()触发
eventTester("loadedmetadata"); //成功获取资源长度
eventTester("loadeddata"); //
eventTester("waiting"); //等待数据，并非错误
eventTester("playing"); //开始回放
eventTester("canplay"); //可以播放，但中途可能因为加载而暂停
eventTester("canplaythrough"); //可以播放，歌曲全部加载完毕
eventTester("seeking"); //寻找中
eventTester("seeked"); //寻找完毕
eventTester("timeupdate"); //播放时间改变
eventTester("ended"); //播放结束
eventTester("ratechange"); //播放速率改变
eventTester("durationchange"); //资源长度改变
eventTester("volumechange"); //音量改变
```

我一个项目开发中要上传视频文件，视频文件为avi，rmvb格式的，需要上传后台前判断视频源的时长duration和大小size,查阅了文档知道，监听loadedmetadata事件可以拿到时长，代码如下：

```js
common.invokeUpload().then((res) => {
    if(res.type.indexOf('video') < 0){
        this.$Message.warning('文件格式错误，请重新上传！');
        return;
    }
    let url = URL.createObjectURL(res);
    let videoEl = document.createElement('video');
    videoEl.setAttribute('src',url);
    //videoEl.setAttribute('preload','metadata');
    //document.body.appendChild(videoEl);
    let duration = 0;
    let self = this;
    videoEl.addEventListener("loadedmetadata", function (_event) {
        duration = videoEl.duration;
        //console.log(duration);
        self.videoSize = (res.size / 1024 / 1024).toFixed(2);
        self.uploadMedia(res,duration);
    });
}).catch((error) => {
    this.$Message.warning('上传失败:'+ error);
})
```

然后发现avi，rmvb格式的视频源不进loadmetadata事件。所以这个获取视频时长的方法不能针对所有格式的视频，此方法适用mp4,flv。