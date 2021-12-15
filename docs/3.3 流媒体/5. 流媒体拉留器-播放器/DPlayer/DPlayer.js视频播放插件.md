其他类似[工具](http://www.fly63.com/tool): DPlayer, videos.[js](http://www.fly63.com/tag/js), ckplayer, [vue](http://www.fly63.com/tag/vue)-DPlayer, [vue](http://www.fly63.com/tag/vue)-video-player

官方文档：http://dplayer.js.org

**DPlayer** 是一个支持弹幕的 [html](http://www.fly63.com/tag/html)5 视频播放器。支持 Bilibili 视频和 danmaku，实时视频（HTTP Live Streaming，M3U8格式）以及 FLV 格式。 

### 用法

[html](http://www.fly63.com/tag/html)

```html
<div id="player1" class="dplayer"></div>
<!-- ... -->
<script src="dist/DPlayer.min.js"></script>
```

### 选项

```javascript
var dp = new DPlayer({
    element: document.getElementById('player1'),                       // 可选，player元素
    autoplay: false,                                                   // 可选，自动播放视频，不支持移动浏览器
    theme: '#FADFA3',                                                  // 可选，主题颜色，默认: #b7daff
    loop: true,                                                        // 可选，循环播放音乐，默认：true
    lang: 'zh',                                                        // 可选，语言，`zh'用于中文，`en'用于英语，默认：Navigator 
                                                                          language
    screenshot: true,                                                  // 可选，启用截图功能，默认值：false，注意：如果设置为
                                                                          true，视频和视频截图必须启用跨域
    hotkey: true,                                                      // 可选，绑定热键，包括左右键和空格，默认值：true
    preload: 'auto',                                                   // 可选，预加载的方式可以是'none''metadata''auto'，默认
                                                                          值：'auto'
    video: {                                                           // 必需，视频信息
        url: '若能绽放光芒.mp4',                                         // 必填，视频网址
        pic: '若能绽放光芒.png'                                          // 可选，视频截图
    },
    danmaku: {                                                         // 可选，显示弹幕，忽略此选项以隐藏弹幕
        id: '9E2E3368B56CDBB4',                                        // 必需，弹幕 id，注意：它必须是唯一的，不能在你的新播放器
                                                                          中使用这些： `https://api.prprpr.me/dplayer/list`
        api: 'https://api.prprpr.me/dplayer/',                             // 必需，弹幕 api
        token: 'tokendemo',                                            // 可选，api 的弹幕令牌
        maximum: 1000,                                                 // 可选，最大数量的弹幕
        addition: ['https://api.prprpr.me/dplayer/bilibili?aid=4157142']   // 可选的，额外的弹幕，参见：`Bilibili弹幕支持`
    }
});
```

### 事件绑定

dp.on（event, handler）事件：

```
play: DPlayer 开始播放时触发 
pause: DPlayer 暂停时触发 
canplay: 在有足够的数据可以播放时触发 
playing: DPlayer 播放时定期触发 
ended: DPlayer 结束时触发 error: 发生错误时触发
```

### HLS支持（实时视频，M3U8格式）

它需要 hls.[js](http://www.fly63.com/tag/js) 库，并且应该在 DPlayer.min.[js](http://www.fly63.com/tag/js) 之前加载。实时弹幕尚不支持。

```html
<div id="player1" class="dplayer"></div>
<!-- ... -->
<script src="plugin/hls.min.js"></script>
<script src="dist/DPlayer.min.js"></script>

<script>
var dp = new DPlayer({
// ...
video: {
url: 'xxx.m3u8'
// ...
}
});
</script>
```

### FLV支持

它需要 flv.[js](http://www.fly63.com/tag/js) 库，并且应该在 DPlayer.min.[js](http://www.fly63.com/tag/js) 之前加载。

```html
<div id="player1" class="dplayer"></div>
<!-- ... -->
<script src="plugin/flv.min.js"></script>
<script src="dist/DPlayer.min.js"></script>

<script>
var dp = new DPlayer({
// ...
video: {
url: 'xxx.flv'
// ...
}
});
</script>
```

### 使用bundler模块

```html
var DPlayer = require('DPlayer'); 
var dp = new DPlayer(option);
```

