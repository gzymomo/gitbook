[简书：雨翼195：flv.js 中文API文档](https://www.jianshu.com/p/b58356b465c4)



# flv.js API

本文档使用类似TypeScript的定义来描述接口。

## 接口

flv.js将所有接口都以flvjs对象暴露在全局上下文window中.

flvjs 还可以通过require或ES6导入来访问对象。

方法:

- [flvjs.createPlayer()](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjscreateplayer)
- [flvjs.isSupported()](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsissupported)
- [flvjs.getFeatureList()](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsgetfeaturelist)

类:

- [flvjs.FlvPlayer](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsflvplayer)
- [flvjs.NativePlayer](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsnativeplayer)
- [flvjs.LoggingControl](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsloggingcontrol)

枚举:

- [flvjs.Events](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjsevents)
- [flvjs.ErrorTypes](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjserrortypes)
- [flvjs.ErrorDetails](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fmirrors%2Fflv.js%2Fblob%2Fmaster%2Fdocs%2Fapi.md%23flvjserrordetails)

### flvjs.createPlayer()



```php
function createPlayer(mediaDataSource: MediaDataSource, config?: Config): Player;
```

根据中指定的type字段创建一个播放器实例mediaDataSource（可选）config。

### MediaDataSource

| Field              | Type                  | Description                                             |
| ------------------ | --------------------- | ------------------------------------------------------- |
| `type`             | `string`              | 媒体类型，'flv'或'mp4'                                  |
| `isLive?`          | `boolean`             | 数据源是否为**实时流**                                  |
| `cors?`            | `boolean`             | 是否启用CORS进行http提取                                |
| `withCredentials?` | `boolean`             | 是否对Cookie进行http提取                                |
| `hasAudio?`        | `boolean`             | 流是否有音频轨道                                        |
| `hasVideo?`        | `boolean`             | 流中是否有视频轨道                                      |
| `duration?`        | `number`              | 总媒体持续时间（以毫秒为单位）                          |
| `filesize?`        | `number`              | 媒体文件的总文件大小，以字节为单位                      |
| `url?`             | `string`              | 表示媒体URL，可以以'https(s)'或'ws(s)'（WebSocket）开头 |
| `segments?`        | `Array<MediaSegment>` | 多段播放的可选字段，请参见MediaSegment                  |

如果segments存在字段，则transmuxer会将其MediaDataSource视为多部分源。

在多部分模式下，结构中的duration filesize url字段MediaDataSource将被忽略。

### MediaSegment

| Field       | Type     | Description                              |
| ----------- | -------- | ---------------------------------------- |
| `duration`  | `number` | 必填字段，指示段持续时间（以毫秒为单位） |
| `filesize?` | `number` | 可选字段，指示段文件大小（以字节为单位） |
| `url`       | `string` | 必填字段，指示段文件URL                  |

### Config

| Field                            | Type      | Default                      | Description                                                  |
| -------------------------------- | --------- | ---------------------------- | ------------------------------------------------------------ |
| `enableWorker?`                  | `boolean` | `false`                      | 启用分离的线程进行转换（暂时不稳定）                         |
| `enableStashBuffer?`             | `boolean` | `true`                       | 启用IO隐藏缓冲区。如果您需要实时（最小延迟）来进行实时流播放，则设置为false，但是如果网络抖动，则可能会停顿。 |
| `stashInitialSize?`              | `number`  | `384KB`                      | 指示IO暂存缓冲区的初始大小。默认值为384KB。指出合适的尺寸可以改善视频负载/搜索时间。 |
| `isLive?`                        | `boolean` | `false`                      | 同样要isLive在MediaDataSource，如果忽略已经在MediaDataSource结构集合。 |
| `lazyLoad?`                      | `boolean` | `true`                       | 如果有足够的数据可播放，则中止http连接。                     |
| `lazyLoadMaxDuration?`           | `number`  | `3 * 60`                     | 指示要保留多少秒的数据lazyLoad                               |
| `lazyLoadRecoverDuration?`       | `number`  | `30`                         | 指示lazyLoad恢复时间边界，以秒为单位。                       |
| `deferLoadAfterSourceOpen?`      | `boolean` | `true`                       | 在MediaSource sourceopen事件触发后加载。在Chrome上，在后台打开的标签页可能不会触发sourceopen事件，除非切换到该标签页。 |
| `autoCleanupSourceBuffer`        | `boolean` | `false`                      | 对SourceBuffer进行自动清理                                   |
| `autoCleanupMaxBackwardDuration` | `number`  | `3 * 60`                     | 当向后缓冲区持续时间超过此值（以秒为单位）时，请对SourceBuffer进行自动清理 |
| `autoCleanupMinBackwardDuration` | `number`  | `2 * 60`                     | 指示进行自动清除时为反向缓冲区保留的持续时间（以秒为单位）。 |
| `fixAudioTimestampGap`           | `boolean` | `true`                       | 当检测到较大的音频时间戳间隙时，请填充无声音频帧，以避免A / V不同步。 |
| `accurateSeek?`                  | `boolean` | `false`                      | 精确查找任何帧，不限于视频IDR帧，但可能会慢一些。可用的Chrome > 50，FireFox和Safari。 |
| `seekType?`                      | `string`  | `'range'`                    | 'range'使用范围请求进行搜索，或'param'在url中添加参数以指示请求范围。 |
| `seekParamStart?`                | `string`  | `'bstart'`                   | 指示的搜索起始参数名称 seekType = 'param'                    |
| `seekParamEnd?`                  | `string`  | `'bend'`                     | 指示的搜索结束参数名称 seekType = 'param'                    |
| `rangeLoadZeroStart?`            | `boolean` | `false`                      | Range: bytes=0-如果使用范围查找，则发送首次负载              |
| `customSeekHandler?`             | `object`  | `undefined`                  | 指示自定义搜索处理程序                                       |
| `reuseRedirectedURL?`            | `boolean` | `false`                      | 重复使用301/302重定向的url进行子序列请求，例如搜索，重新连接等。 |
| `referrerPolicy?`                | `string`  | `no-referrer-when-downgrade` | 指示使用FetchStreamLoader时的推荐人[策略](https://links.jianshu.com/go?to=https%3A%2F%2Fw3c.github.io%2Fwebappsec-referrer-policy%2F%23referrer-policy) |
| `headers?`                       | `object`  | `undefined`                  | 指示将添加到请求的其他标头                                   |

### flvjs.isSupported()

```php
function isSupported(): boolean;
```

如果基本上可以再您的浏览器上播放则返回true

### flvjs.getFeatureList()

```php
function getFeatureList(): FeatureList;
```

返回FeatureList具有以下详细信息的对象：

#### FeatureList

| Field                   | Type      | Description                                                  |
| ----------------------- | --------- | ------------------------------------------------------------ |
| `mseFlvPlayback`        | `boolean` | 与flvjs.isSupported()相同，表示您的浏览器是否可以进行基本播放。 |
| `mseLiveFlvPlayback`    | `boolean` | HTTP FLV实时流是否可以在您的浏览器上工作。                   |
| `networkStreamIO`       | `boolean` | 指示网络加载程序是否正在流式传输。                           |
| `networkLoaderName`     | `string`  | 指示网络加载程序类型名称。                                   |
| `nativeMP4H264Playback` | `boolean` | 指示您的浏览器是否本身支持H.264 MP4视频文件。                |
| `nativeWebmVP8Playback` | `boolean` | 指示您的浏览器是否本机支持WebM VP8视频文件。                 |
| `nativeWebmVP9Playback` | `boolean` | 指示您的浏览器是否本机支持WebM VP9视频文件。                 |

### flvjs.FlvPlayer

```dart
interface FlvPlayer extends Player {}
```

实现Player接口的FLV播放器。可以通过new操作进行创建

### flvjs.NativePlayer

```dart
interface NativePlayer extends Player {}
```

Player wrapper for browser's native player (HTMLVideoElement) without MediaSource src, which implements the `Player` interface. Useful for singlepart **MP4** file playback.

### Player 接口 (抽象)

```tsx
interface Player {
    constructor(mediaDataSource: MediaDataSource, config?: Config): Player;
    destroy(): void;
    on(event: string, listener: Function): void;
    off(event: string, listener: Function): void;
    attachMediaElement(mediaElement: HTMLMediaElement): void;
    detachMediaElement(): void;
    load(): void;
    unload(): void;
    play(): Promise<void>;
    pause(): void;
    type: string;
    buffered: TimeRanges;
    duration: number;
    volume: number;
    muted: boolean;
    currentTime: number;
    mediaInfo: Object;
    statisticsInfo: Object;
}
```

### flvjs.LoggingControl

A global interface which include several static getter/setter to set flv.js logcat verbose level.
 一个全局接口，其中包括几个用于设置flv.js logcat详细级别的静态getter / setter。

```tsx
interface LoggingControl {
    forceGlobalTag: boolean;
    globalTag: string;
    enableAll: boolean;
    enableDebug: boolean;
    enableVerbose: boolean;
    enableInfo: boolean;
    enableWarn: boolean;
    enableError: boolean;
    getConfig(): Object;
    applyConfig(config: Object): void;
    addLogListener(listener: Function): void;
    removeLogListener(listener: Function): void;
}
```

### flvjs.Events

一系列可以和 `Player.on()` / `Player.off()`一起使用的常数. 它们需要前缀`flvjs.Events`.

| 事件                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| ERROR               | 播放期间由于任何原因发生错误                                 |
| LOADING_COMPLETE    | 输入MediaDataSource已完全缓冲到结束                          |
| RECOVERED_EARLY_EOF | 缓冲期间发生意外的网络EOF，但已自动恢复                      |
| MEDIA_INFO          | 提供媒体的技术信息，例如视频/音频编解码器，比特率等。        |
| METADATA_ARRIVED    | 用“ onMetaData”标记提供FLV文件（流）可以包含的元数据。       |
| SCRIPTDATA_ARRIVED  | 提供FLV文件（流）可以包含的脚本数据（OnCuePoint / OnTextData）。 |
| STATISTICS_INFO     | 提供播放统计信息，例如丢帧，当前速度等。                     |

### flvjs.ErrorTypes

播放期间可能出现的错误。它们需要前缀flvjs.ErrorTypes。

| 错误          | 描述                                     |
| ------------- | ---------------------------------------- |
| NETWORK_ERROR | 与网络有关的错误                         |
| MEDIA_ERROR   | 与媒体有关的错误（格式错误，解码问题等） |
| OTHER_ERROR   | 任何其他未指定的错误                     |

### flvjs.ErrorDetails

针对网络和媒体错误提供更详细的说明。它们需要前缀flvjs.ErrorDetails。

| 错误                            | 描述                                         |
| ------------------------------- | -------------------------------------------- |
| NETWORK_EXCEPTION               | 与网络其他任何问题有关；包含一个message      |
| NETWORK_STATUS_CODE_INVALID     | 与无效的HTTP状态代码（例如403、404等）相关。 |
| NETWORK_TIMEOUT                 | 相关超时请求问题                             |
| NETWORK_UNRECOVERABLE_EARLY_EOF | 与无法恢复的意外网络EOF相关                  |
| MEDIA_MSE_ERROR                 | 与MediaSource的错误有关，例如解码问题        |
| MEDIA_FORMAT_ERROR              | 与媒体流中的任何无效参数有关                 |
| MEDIA_FORMAT_UNSUPPORTED        | flv.js不支持输入的MediaDataSource格式        |
| MEDIA_CODEC_UNSUPPORTED         | 媒体流包含不支持的视频/音频编解码器          |



