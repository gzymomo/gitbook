原文链接：简书：合肥懒皮：[H5直播系列十 flv.js源码之IO部分](https://www.jianshu.com/p/20d1e7c90fe6)



# 一、loader.js

export class BaseLoader 有一些基本属性和回调

Loader has callbacks which have following prototypes:

- funcction onContentLengthKnown(contentLength: number): void
- funcction onURLRedirect(url: string): void
- funcction onDataArrival(chunk: ArrayBuffer, byteStart: number, receivedLength: number): void
- funcction onError(errorType: number, errorInfo: {code: number, msg: string}): void
- funcction onComplete(rangeFrom: number, rangeTo: number): void

# 二、BaseLoader的子类

参见io-controller.js，如果xhr连responseType=arraybuffer也不支持，那么就用不了这个库



```php
_selectLoader() {
    if (this._config.customLoader != null) {
        this._loaderClass = this._config.customLoader;
    } else if (this._isWebSocketURL) {
        this._loaderClass = WebSocketLoader;
    } else if (FetchStreamLoader.isSupported()) {
        this._loaderClass = FetchStreamLoader;
    } else if (MozChunkedLoader.isSupported()) {
        this._loaderClass = MozChunkedLoader;
    } else if (RangeLoader.isSupported()) {
        this._loaderClass = RangeLoader;
    } else {
        throw new RuntimeException('Your browser doesn\'t 
        support xhr with arraybuffer responseType!');
    }
}
```

1.websocket-loader.js
 class WebSocketLoader extends BaseLoader
 websocket基础知识参考[HTML5 WebSocket](https://www.jianshu.com/p/5e34b956ba89)



```kotlin
    _dispatchArrayBuffer(arraybuffer) {
        let chunk = arraybuffer;
        let byteStart = this._receivedLength;
        this._receivedLength += chunk.byteLength;

        if (this._onDataArrival) {
            this._onDataArrival(chunk, byteStart, this._receivedLength);
        }
    }
```

2.在[JS异步处理系列二 XHR Fetch](https://www.jianshu.com/p/0688f5744d22)中，介绍了Fetch的流式传输在直播/点播中很重要

> 我们可以当数据进来时就缓存下来，而且我们也不必等到数据全部读取完毕才显示内容。使响应体流式化减少了该站点的内存占用，并在网络连接很慢时为展示内容提供了更快的感知速度。而 XHR 只能缓存整个响应体，不能以小块的形式操作数据。虽然用 XHR 建立一个流是有可能的，然而这会导致 responseText 持续增长，而且你必须不断手动地从中获取数据。除此之外，当在流式传输时，Fetch APIs 还提供了访问数据的实际字节的方法，而 XHR 的 responseText 只有文本形式，这意味着在某些场景下它的作用可能非常有限。

3.fetch-stream-loader.js
 class FetchStreamLoader extends BaseLoader
 优先考虑使用fetch来加载

> fetch + stream IO loader. Currently working on chrome 43+.
>  fetch provides a better alternative http API to XMLHttpRequest



```kotlin
if (this._onDataArrival) {
    this._onDataArrival(chunk, byteStart, this._receivedLength);
}
```

4.xhr-moz-chunked-loader.js
 class MozChunkedLoader extends BaseLoader

> For FireFox browser which supports `xhr.responseType = 'moz-chunked-arraybuffer'`

如果fetch不支持，优先考虑moz-chunked-arraybuffer，这个可以参考[WebKit equivalent to Firefox's “moz-chunked-arraybuffer” xhr responseType](https://stackoverflow.com/questions/15185499/webkit-equivalent-to-firefoxs-moz-chunked-arraybuffer-xhr-responsetype)

5.xhr-range-loader.js
 class RangeLoader extends BaseLoader

> Universal IO Loader, implemented by adding Range header in xhr's request header

这里参考[XMLHttpRequest 206 Partial Content](https://stackoverflow.com/questions/15561508/xmlhttprequest-206-partial-content)



```jsx
var xhr = new XMLHttpRequest;

xhr.onreadystatechange = function () {
  if (xhr.readyState != 4) {
    return;
  }
  alert(xhr.status);
};

xhr.open('GET', 'http://fiddle.jshell.net/img/logo.png', true);
 // the bytes (incl.) you request
xhr.setRequestHeader('Range', 'bytes=100-200');
xhr.send(null);
```

**You have to make sure that the server allows range requests, though.**

在[用 FileSystem API 实现文件下载器](https://imququ.com/post/a-downloader-with-filesystem-api.html)中提到了大文件分段下载：
 **要实现并发下载，首先要合理分配任务。HTTP 协议中规定可以使用请求头的 Range 字段指定请求资源的范围。例如服务端收到「Range : bytes=10-100」这样的请求头，只需要返回资源的 10-100 字节这部分就可以了，这样的响应状态码为 206。有些服务器不支持 Range，本文继续忽略。**比如在在 Nginx 配置文件中，`add_header Access-Control-Allow-Headers Range;`

这里使用Range方式的，封装到range-seek-handler.js中。还有一种使用param方式的，也就是url后面跟?，然后加bstart和bend=xxx的，封装到param-seek-handler.js，在io-controller.js中可以看到：



```jsx
_selectSeekHandler() {
    let config = this._config;

    if (config.seekType === 'range') {
        this._seekHandler = new RangeSeekHandler(this._config.rangeLoadZeroStart);
    } else if (config.seekType === 'param') {
        let paramStart = config.seekParamStart || 'bstart';
        let paramEnd = config.seekParamEnd || 'bend';

        this._seekHandler = new ParamSeekHandler(paramStart, paramEnd);
    } else if (config.seekType === 'custom') {
        if (typeof config.customSeekHandler !== 'function') {
            throw new InvalidArgumentException('Custom seekType specified in config but invalid customSeekHandler!');
        }
        this._seekHandler = new config.customSeekHandler();
    } else {
        throw new InvalidArgumentException(`Invalid seekType in config: ${config.seekType}`);
    }
}
```

6.xhr-msstream-loader.js 目前在源码中未使用此类
 class MSStreamLoader extends BaseLoader

> For IE11/Edge browser by microsoft which supports xhr.responseType = 'ms-stream'

# 三、nginx服务器试一下Range方式

搭建服务器可以参考[H5直播系列六 flv.js demo](https://www.jianshu.com/p/9d6d81c88cf2)
 1.配置nginx.conf
 这里为了在Chrome里测试方便，直接把io-controller.js里的_selectLoader中部分代码修改：



```kotlin
else if (FetchStreamLoader.isSupported()) {
            // this._loaderClass = FetchStreamLoader;
            this._loaderClass = RangeLoader;
        } 
```

设置了`add_header Access-Control-Allow-Headers Range;`之后，是不行的，会遇到405，也就是nginx没配置OPTIONS请求。参考[flv.js/issues/159 快进播放，出现跨越问题](https://github.com/Bilibili/flv.js/issues/159)，这里理论知识没仔细看，可以参考[阮一峰 跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)。

说一下怎么解决的吧，先是参考[Nginx跨域配置，支持DELETE,PUT请求](http://to-u.xyz/2016/06/30/nginx-cors/)



```bash
location / {
    if ($request_method = 'OPTIONS') { 
        add_header Access-Control-Allow-Origin *; 
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        return 204; 
    }
```

但是没有效果哎，原因不明……然后参考[nginx静态服务器405报错](https://jsfun.info/archive/nginx静态服务器405报错/)和[nginx Cors跨域请求OPTIONS方法405 Method Not Allowed问题](https://blog.claves.me/2018/04/17/nginx-cors跨域请求options方法405-method-not-allowed问题/)



```ruby
error_page   405 =200 @405;
location @405{
            add_header Content-Length 0;
            add_header Content-Type text/plain;
            add_header Access-Control-Allow-Headers *;
            add_header Access-Control-Allow-Methods *;
            add_header Access-Control-Allow-Origin *;
            return 200;
        }
```

这样就可以了
 2.简单观察
 首先右键看一下要播放的文件jay.flv



![img](https:////upload-images.jianshu.io/upload_images/2354823-dcc9db1f95e117d0.png?imageMogr2/auto-orient/strip|imageView2/2/w/330/format/webp)

image.png



![img](https:////upload-images.jianshu.io/upload_images/2354823-749b551f60e7067c.png?imageMogr2/auto-orient/strip|imageView2/2/w/369/format/webp)

第一次XHR请求



![img](https:////upload-images.jianshu.io/upload_images/2354823-c95e04d9937c3094.png?imageMogr2/auto-orient/strip|imageView2/2/w/337/format/webp)

image.png



![img](https:////upload-images.jianshu.io/upload_images/2354823-2214222dbb8479f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/379/format/webp)

image.png



可以看到分段请求

# 四、直播流的支持度



```csharp
static supportNetworkStreamIO() {
    let ioctl = new IOController({}, createDefaultConfig());
    let loaderType = ioctl.loaderType;
    ioctl.destroy();
    return loaderType == 'fetch-stream-loader' ||
    loaderType == 'xhr-moz-chunked-loader';
}
...
features.mseLiveFlvPlayback = features.mseFlvPlayback 
&& features.networkStreamIO;
```

这样的话，连loaderType==websocket-loader也给排除了？？

# 五、io-controller.js

在上面的各种loader分析中，最后抛出数据都是给onDataArrival，在io-controller.js中，实际由_onLoaderChunkArrival来接管



```kotlin
_createLoader() {
    this._loader = new this._loaderClass(this._seekHandler, this._config);
    if (this._loader.needStashBuffer === false) {
        this._enableStash = false;
    }
    this._loader.onContentLengthKnown = this._onContentLengthKnown.bind(this);
    this._loader.onURLRedirect = this._onURL、Redirect.bind(this);
    this._loader.onDataArrival = this._onLoaderChunkArrival.bind(this);
    this._loader.onComplete = this._onLoaderComplete.bind(this);
    this._loader.onError = this._onLoaderError.bind(this);
}
```

在_onLoaderChunkArrival中这样一段：



```kotlin
this._speedSampler.addBytes(chunk.byteLength);

// adjust stash buffer size according to network speed dynamically
let KBps = this._speedSampler.lastSecondKBps;
if (KBps !== 0) {
    let normalized = this._normalizeSpeed(KBps);
    if (this._speedNormalized !== normalized) {
        this._speedNormalized = normalized;
        this._adjustStashSize(normalized);
    }
}
```

作者在 [使用flv.js做直播](https://segmentfault.com/a/1190000009695424)中回复这样解释：

> 在 flv.js 的 IOController 中，有一个用于缓存数据的 stashBuffer。buffer 大小会根据实时网速动态适应调整，以维持一个较合适的向外 dispatch buffer 的频率，减少解析频率来降低开销。

这里提到的_speedSampler，就是speed-sampler.js(Utility class to calculate realtime network I/O speed)

这个缓存功能，默认是打开的



```jsx
export const defaultConfig = {
    enableWorker: false,
    enableStashBuffer: true,
    stashInitialSize: undefined,

// default initial size: 384KB
this._stashInitialSize = 1024 * 384;  
if (config.stashInitialSize != undefined
 && config.stashInitialSize > 0) {
    // apply from config
    this._stashInitialSize = config.stashInitialSize;
}
```