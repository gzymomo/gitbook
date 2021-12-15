[Github地址](https://github.com/ErosZy/WXInlinePlayer)



通过软解FLV的方式实现了WXInlinePlayer，其用的第三方技术和平台API如下：

1. [OpenH264](https://github.com/cisco/openh264)/[TinyH264](https://github.com/udevbe/tinyh264)/[de265](https://github.com/strukturag/libde265)；
2. [emscripten](https://github.com/emscripten-core/emscripten)
3. [WebGL](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API)
4. [Web Audio Api](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API)

同时我们也编写了WebAssembly版本的FLV Demuxer，你可以在[lib/codec](https://github.com/qiaozi-tech/WXInlinePlayer/tree/master/lib/codec)找到相关代码。

## 特性

1. FLV H264/H265 点播/直播全支持
2. 自由选择解码依赖，在实际gzip中，Tinyh264只需 ~180k，OpenH264 ~260k，de265 ~210k （[如何选择解码依赖](https://github.com/qiaozi-tech/WXInlinePlayer#如何选择解码依赖)）
3. 专为移动端性能优化，内存和CPU占用稳定
4. 直播延迟优化，比MSE的原生Video实现低1-2s（[如何降低卡顿和延迟](https://github.com/qiaozi-tech/WXInlinePlayer#如何降低卡顿和延迟)）
5. 音频/视频独立支持
6. 微信WebView自动播放
7. 无音频动画自动播放
8. 良好的移动端WebView兼容性

## 兼容性

兼容测试使用BrowserStack服务提供的相关机型，仅供参考：

- Android 5+
- iOS 10+ （含Safari及WebView）
- Chrome 25+
- Firefox 57+
- Edge 15+
- Safari 10.1+