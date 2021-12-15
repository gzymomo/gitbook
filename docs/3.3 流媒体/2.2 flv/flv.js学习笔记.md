- [flv.js学习笔记](https://www.jianshu.com/p/a88b8fb0233e)

# 一、io包，拿到数据

## 1.load.js有BaseLoader，以下几个loader都继承覆盖它：

 (a)fetch-stream-loader
 fetch + stream IO loader. Currently working on chrome 43+.
 fetch provides a better alternative http API to XMLHttpRequest

(b)websocket-loader
 For FLV over WebSocket live stream

(c)xhr-moz-chunked-loader
 For FireFox browser which supports `xhr.responseType = 'moz-chunked-arraybuffer'`

(d)xhr-msstream-loader
 For IE11/Edge browser by microsoft which supports `xhr.responseType = 'ms-stream'`

(e)xhr-range-loader
 Universal IO Loader, implemented by adding Range header in xhr's request header

## 2.speed-sampler.js

 Utility class to calculate realtime network I/O speed

## 3.seekType

 解析得到Loader中要用的参数
 `'range'` use range request to seek, or `'param'` add params into url to indicate request range.
 param-seek-handler.js
 range-seek-handler.js

## 4.iocontroller.js对上面几个类的综合使用

 this._selectSeekHandler();//根据seektype确定用哪个handler
 this._selectLoader();//确定用哪个Loader
 this._createLoader();

# 二、demux包

对FLV数据格式进行解析，参考[FLV.JS 代码解读--demux部分](https://zhuanlan.zhihu.com/p/24429290)
 解析完了 tag header后面分别按照不同的 tag type调用不同的解析函数。

```yaml
//flv-demuxer.js
switch (tagType) {
    case 8:  // Audio
        this._parseAudioData(chunk, dataOffset, dataSize, timestamp);
        break;
    case 9:  // Video
        this._parseVideoData(chunk, dataOffset, dataSize, timestamp, byteStart + offset);
        break;
    case 18:  // ScriptDataObject
        this._parseScriptData(chunk, dataOffset, dataSize);
        break;
}
```

## 1.amf-parser.js

 这个是用来解析上面所说的tagType=18，即ScriptDataObject。在flv-demuxer.js中，可以看到能解析出以下属性：hasAudio,hasVideo,autiodatarate,videodatarate,width,height,duration,framerate,keyframes

## 2.音频解析

 分成AAC和MP3两种格式

## 3.视频解析

 exp-golomb.js和sps-parser.js
 packetType有两种，0 表示 AVCDecoderConfigurationRecord，这个是H.264的视频信息头，包含了 sps 和  pps，AVCDecoderConfigurationRecord的格式不是flv定义的，而是264标准定义的，如果用ffmpeg去解码，这个结构可以直接放到 codec的extradata里送给ffmpeg去解释。

flv.js作者选择了自己来解析这个数据结构，也是迫不得已，因为JS环境下没有ffmpeg，解析这个结构主要是为了提取 sps和pps。虽然理论上sps允许有多个，但其实一般就一个。`let config = SPSParser.parseSPS(sps);`pps的信息没什么用，所以作者只实现了sps的分析器，说明作者下了很大功夫去学习264的标准，其中的Golomb解码还是挺复杂的，能解对不容易，我在PC和手机平台都是用ffmpeg去解析的。SPS里面包括了视频分辨率，帧率，profile level等视频重要信息。

# 三、core包部分类

## 1.features.js 检测支持度

##  2.media-info.js

 media数据

##  3.media-segment-info.js

 (a)SampleInfo
 Represents an media sample (audio / video)
 (b)MediaSegmentInfo
 // Media Segment concept is defined in Media Source Extensions spec.
 // Particularly in ISO BMFF format, an Media Segment contains a moof box followed by a mdat box.
 (c)IDRSampleList
 // Ordered list for recording video IDR frames, sorted by originalDts
 (d)MediaSegmentInfoList
 // Data structure for recording information of media segments in single track.

# 四、flv.js流程

```js
 if (flvjs.isSupported()) {
      var videoElement = document.getElementById('videoElement');
      var flvPlayer = flvjs.createPlayer({
          type: 'flv',
          url: 'http://example.com/flv/video.flv'
      });
      flvPlayer.attachMediaElement(videoElement);
      flvPlayer.load();
      flvPlayer.play();
  }
```

## 1.flv.js

 createPlayer(mediaDataSource, optionalConfig)

##  2.flv-player.js

 (a)constructor(mediaDataSource, config) {}
 (b)attachMediaElement主要设置MSEController

```js
attachMediaElement(mediaElement) {
this._mediaElement = mediaElement;
this._msectl = new MSEController();
this._msectl.on...
this._msectl.attachMediaElement(mediaElement);
}
```

(c)在load方法中，Transmuxer和MSEController建立关联

```js
load() {
this._transmuxer = new Transmuxer(this._mediaDataSource, this._config);
this._transmuxer.on(TransmuxingEvents.INIT_SEGMENT, (type, is) => {
    this._msectl.appendInitSegment(is);
});
this._transmuxer.on(TransmuxingEvents.MEDIA_SEGMENT, (type, ms) => {
    this._msectl.appendMediaSegment(ms);
this._transmuxer.on...
this._transmuxer.open();
}
```

TransmuxingEvents.INIT_SEGMENT和TransmuxingEvents.MEDIA_SEGMENT很重要。
 (d)

```js
play() {
    this._mediaElement.play();
}
```

这个简单，相当于让页面上的video标签调用play方法
 (e)core/transmuxer.js

```
this._controller = new TransmuxingController(mediaDataSource, config);
```

(f)transmuxing-worker.js
 Enable separated thread for transmuxing (unstable for now)
 多线程加载，可以先忽略

(g)transmuxing-controller.js
 **// Transmuxing (IO, Demuxing, Remuxing) controller, with multipart support**

```
// treat single part media as multipart media, which has only one segment
if (!mediaDataSource.segments) {
    mediaDataSource.segments = [{
        duration: mediaDataSource.duration,
        filesize: mediaDataSource.filesize,
        url: mediaDataSource.url
    }];
}
//上面(c)步骤中的this._transmuxer.open();会执行到this._controller.start();
start(){
this._loadSegment(0);
this._enableStatisticsReporter();
}

_loadSegment(segmentIndex, optionalFrom) {
    this._currentSegmentIndex = segmentIndex;
    let dataSource = this._mediaDataSource.segments[segmentIndex];

    let ioctl = this._ioctl = new IOController(dataSource, this._config, segmentIndex);
    ioctl.onError = this._onIOException.bind(this);
    ioctl.onSeeked = this._onIOSeeked.bind(this);
    ioctl.onComplete = this._onIOComplete.bind(this);
    ioctl.onRedirect = this._onIORedirect.bind(this);
    ioctl.onRecoveredEarlyEof = this._onIORecoveredEarlyEof.bind(this);

    if (optionalFrom) {
        this._demuxer.bindDataSource(this._ioctl);
    } else {
        ioctl.onDataArrival = this._onInitChunkArrival.bind(this);
    }

    ioctl.open(optionalFrom);
}

_onInitChunkArrival(data, byteStart) {
...
// Always create new FLVDemuxer
this._demuxer = new FLVDemuxer(probeData, this._config);

if (!this._remuxer) {
    this._remuxer = new MP4Remuxer(this._config);
}
this._demuxer.onError = this._onDemuxException.bind(this);
this._demuxer.onMediaInfo = this._onMediaInfo.bind(this);

this._remuxer.bindDataSource(this._demuxer
             .bindDataSource(this._ioctl
));

this._remuxer.onInitSegment = this._onRemuxerInitSegmentArrival.bind(this);
this._remuxer.onMediaSegment = this._onRemuxerMediaSegmentArrival.bind(this);

consumed = this._demuxer.parseChunks(data, byteStart);
}

_onRemuxerInitSegmentArrival(type, initSegment) {
    this._emitter.emit(TransmuxingEvents.INIT_SEGMENT, type, initSegment);
}
_onRemuxerMediaSegmentArrival(type, mediaSegment) {
    if (this._pendingSeekTime != null) {
        // Media segments after new-segment cross-seeking should be dropped.
        return;
    }
    this._emitter.emit(TransmuxingEvents.MEDIA_SEGMENT, type, mediaSegment);
...
}
this._remuxer.onInitSegment = this._onRemuxerInitSegmentArrival.bind(this);
this._remuxer.onMediaSegment = this._onRemuxerMediaSegmentArrival.bind(this);
```

这两句回调很重要，对应着TransmuxingEvents.INIT_SEGMENT和TransmuxingEvents.MEDIA_SEGMENT事件。
 一旦_remuxer触发了这两个事件，就表示告诉msectroller,我封装好了，可以渲染显示了。下面就去mse-controller.js中看看,然后再看`this._remuxer.onInitSegment`和`this._remuxer.onMediaSegment`

(h)mse-controller.js
 // Media Source Extensions controller
 先插播一段MSE的API，参考[使用 MediaSource 搭建流式播放器](https://zhuanlan.zhihu.com/p/26374202)
 在浏览器里，首先我们要判断是否支持 MediaSource：
 `var supportMediaSource = 'MediaSource' in window`
 然后就可以新建一个 MediaSource 对象，并且把 mediaSource 作为 objectURL 附加到 video 标签上上：

```
var mediaSource = new MediaSource()
var video = document.querySelector('video')
video.src = URL.createObjectURL(mediaSource)
```

接下来就可以监听 mediaSource 上的 sourceOpen 事件，并且设置一个回调：

```
mediaSource.addEventListener('sourceopen', sourceOpen);
function sourceOpen {
    // todo...
}
```

接下来会用到一个叫 SourceBuffer 的对象，这个对象提供了一系列接口，这里用到的是 appendBuffer 方法，可以动态地向 MediaSource 中添加视频/音频片段（对于一个 MediaSource，可以同时存在多个 SourceBuffer）

```
function sourceOpen () {
    // 这个奇怪的字符串后面再解释
    var mime = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"'

    // 新建一个 sourceBuffer
    var sourceBuffer = mediaSource.addSourceBuffer(mime);

    // 加载一段 chunk，然后 append 到 sourceBuffer 中
    fetchBuffer('/xxxx.mp4', buffer => {
        sourceBuffer.appendBuffer(buffer)
    })
}

// 以二进制格式请求某个url
function fetchBuffer (url, callback) {
    var xhr = new XMLHttpRequest;
    xhr.open('get', url);
    xhr.responseType = 'arraybuffer';
    xhr.onload = function () {
        callback(xhr.response);
    };
    xhr.send();
}
```

上面这些代码基本上就是一个最简化的流程了，加载了一段视频 chunk，然后把它『喂』到播放器中。

可以在mse-controller.js发现addSourceBuffer和appendBuffer方法

```
appendInitSegment(initSegment, deferred) {
  ...
  let sb = this._sourceBuffers[is.type] = this._mediaSource.addSourceBuffer(mimeType);
  ...
}

_doAppendSegments() {
  ...
  this._sourceBuffers[type].appendBuffer(segment.data);
  ...
  this._doAppendSegments();
}

appendMediaSegment(mediaSegment) {
  ...
  this._doAppendSegments();
}
```

(i)mp4-remuxer.js
 现在回头来看看_remuxer的onInitSegment 和onMediaSegment 。注意看_onTrackMetadataReceived方法的最后调用了this._onInitSegment。
 MP4Remuxer

```
bindDataSource(producer) {
    producer.onDataAvailable = this.remux.bind(this);
    producer.onTrackMetadata = this._onTrackMetadataReceived.bind(this);
    return this;
}

remux(audioTrack, videoTrack) {
    if (!this._onMediaSegment) {
        throw new IllegalStateException('
        MP4Remuxer: onMediaSegment callback must be specificed!');
    }
    if (!this._dtsBaseInited) {
        this._calculateDtsBase(audioTrack, videoTrack);
    }
    this._remuxVideo(videoTrack);
    this._remuxAudio(audioTrack);
}

_onTrackMetadataReceived(type, metadata) {
    let metabox = null;

    let container = 'mp4';
    let codec = metadata.codec;

    if (type === 'audio') {
        this._audioMeta = metadata;
        if (metadata.codec === 'mp3' && this._mp3UseMpegAudio) {
            // 'audio/mpeg' for MP3 audio track
            container = 'mpeg';
            codec = '';
            metabox = new Uint8Array();
        } else {
            // 'audio/mp4, codecs="codec"'
            metabox = MP4.generateInitSegment(metadata);
        }
    } else if (type === 'video') {
        this._videoMeta = metadata;
        metabox = MP4.generateInitSegment(metadata);
    } else {
        return;
    }

    // dispatch metabox (Initialization Segment)
    if (!this._onInitSegment) {
        throw new IllegalStateException('MP4Remuxer: onInitSegment callback must be specified!');
    }
    this._onInitSegment(type, {
        type: type,
        data: metabox.buffer,
        codec: codec,
        container: `${type}/${container}`,
        mediaDuration: metadata.duration  // in timescale 1000 (milliseconds)
    });
}

_remuxVideo(videoTrack) {
    …
    this._onMediaSegment('video', {
        type: 'video',
        data: this._mergeBoxes(moofbox, mdatbox).buffer,
        sampleCount: mp4Samples.length,
        info: info
    });
}
```

可以看到线索跑到了bindDataSource方法上，那么调用这个方法是在哪里呢
 。这又回到了transmuxing-controller.js

```
_onInitChunkArrival(data, byteStart) {
...
// Always create new FLVDemuxer
this._demuxer = new FLVDemuxer(probeData, this._config);

if (!this._remuxer) {
    this._remuxer = new MP4Remuxer(this._config);
}
this._demuxer.onError = this._onDemuxException.bind(this);
this._demuxer.onMediaInfo = this._onMediaInfo.bind(this);

this._remuxer.bindDataSource(this._demuxer
             .bindDataSource(this._ioctl
));
}
```

_remuxer的bindDataSource方法使用了两个回调：onDataAvailable和onTrackMetadata 。这就进一步把控制权交出去了，也就是说只要_demuxer准备好了，remuxer就会通知MSE可以渲染了。

可以在flv-demuxer.js中发现
 `this._onDataAvailable(this._audioTrack, this._videoTrack);`,不过onTrackMetadata没看到调用。

再看`this._demuxer.bindDataSource(this._ioctl)`
 flv-demuxer.js

```
   bindDataSource(loader) {
        loader.onDataArrival = this.parseChunks.bind(this);
        return this;
    }
```

显然要去io-controler.js中去找onDataArrival方法，从名字上就能猜到这是第一部分讲的那几个loader数据加载回来了。而parseChunks方法是第二部分demux包中的解析音频，视频，script数据。

现在把流程再执行一遍
 1.HTML页面执行flvPlayer.load();
 2.跳到flv-player.js中，this._transmuxer.open();
 3.跳到transmuxer.js中，this._controller.start();
 4.跳到transmuxer-controller.js中,执行`this._loadSegment(0);`在这个方法中，进一步执行`ioctl.open(optionalFrom);`
 5.跳到io-controller.js中,执行`this._loader.open(this._dataSource, Object.assign({}, this._currentRange));`
 6.有很多类型的loader，但它们都会出现一句`this._onDataArrival(chunk：ArrayBuffer, byteStart, this._receivedLength);`.显然数据就在chunk里面
 7.在flv-dumuxer.js中

```
    bindDataSource(loader) {
        loader.onDataArrival = this.parseChunks.bind(this);
        return this;
    }
```

可以看到这个回调，把数据解析交给了parseChunks方法，parseChunks把所有数据解析好之后，会执行`this._onDataAvailable(this._audioTrack, this._videoTrack);`.这又是一个回调方法，它的调用者就是_remuxer,证据就是

```
this._remuxer.bindDataSource(this._demuxer
             .bindDataSource(this._ioctl
));
```

顺便看一下这两个的数据类型

```
this._videoTrack = {type: 'video', id: 1, sequenceNumber: 0, samples: [], length: 0};
this._audioTrack = {type: 'audio', id: 2, sequenceNumber: 0, samples: [], length: 0};
```

8.去mp4-remuxer.js
 `this._onDataAvailable(this._audioTrack, this._videoTrack);`上个流程中的这句代码会跑到`remux(audioTrack, videoTrack) {}`这个方法里。然后跑到_onTrackMetadataReceived方法，然后调用了this._onInitSegment，这又是一个回调……
 9.去transmuxer-controller.js中可以找到如下代码

```
    this._remuxer.onInitSegment =this._onRemuxerInitSegmentArrival.bind(this);
    this._remuxer.onMediaSegment = this._onRemuxerMediaSegmentArrival.bind(this);

    _onRemuxerInitSegmentArrival(type, initSegment) {
        this._emitter.emit(TransmuxingEvents.INIT_SEGMENT, type, initSegment);
    }
```

10.终于TransmuxingEvents.INIT_SEGMENT事件抛出来了。在之前的load方法中，曾经注册过侦听

```
load() {
this._transmuxer = new Transmuxer(this._mediaDataSource, this._config);
this._transmuxer.on(TransmuxingEvents.INIT_SEGMENT, (type, is) => {
    this._msectl.appendInitSegment(is);
});
this._transmuxer.on(TransmuxingEvents.MEDIA_SEGMENT, (type, ms) => {
    this._msectl.appendMediaSegment(ms);
this._transmuxer.on...
this._transmuxer.open();
}
```

11.进入mse-controller.js

```
appendInitSegment(initSegment, deferred) {
  ...
  let sb = this._sourceBuffers[is.type] = this._mediaSource.addSourceBuffer(mimeType);
  ...
}

_doAppendSegments() {
  ...
  this._sourceBuffers[type].appendBuffer(segment.data);
  ...
  this._doAppendSegments();
}

appendMediaSegment(mediaSegment) {
  ...
  this._doAppendSegments();
}
```

可以看到在往_mediaSource里面塞数据，而_mediaSource早就通过attachMediaElement方法绑定了HTML中的video标签。

```
attachMediaElement(mediaElement) {
if (this._mediaSource) {
throw new IllegalStateException('MediaSource has been attached to an HTMLMediaElement!');
}
let ms = this._mediaSource = new window.MediaSource();
```