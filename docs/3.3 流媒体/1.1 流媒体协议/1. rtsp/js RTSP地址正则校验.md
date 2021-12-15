- [js RTSP地址正则校验](https://blog.csdn.net/liona_koukou/article/details/103762500)



需求，前端需要严格校验RTSP地址，上面主流RTSP我都试过了可以通过

rtsp://admin:12345@192.0.0.64:554/h264/ch1/main/av_stream

rtsp://admin:12345@192.0.0.64/h264/ch1/sub/av_stream

rtsp://192.168.200.202/axis-media/media.amp?videocodec=h264&resolution=1280x720

rtsp://admin:password@192.168.1.10

rtsp://192.168.0.100/live1.sdp

rtsp://192.168.0.100:554

..........

类似以下这种都判断为不正确的RTSP地址：

rtsp://admin:12345@192.0.0.64:

rtsp://admin:12345@192.0.0.64:aaa

rtsp://admin:12345@192.0.0.64wewrerwerw

放代码：

```js
function isRTSP(str) {
        const reg= /^rtsp:\/\/([a-z]{0,10}:.{0,10}@)?(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$/;
	const reg1= /^rtsp:\/\/([a-z]{0,10}:.{0,10}@)?(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5]):[0-9]{1,5}/;	
	const reg2= /^rtsp:\/\/([a-z]{0,10}:.{0,10}@)?(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\//;
	return (reg.test(str) || reg1.test(str) || reg2.test(str));
}
```
