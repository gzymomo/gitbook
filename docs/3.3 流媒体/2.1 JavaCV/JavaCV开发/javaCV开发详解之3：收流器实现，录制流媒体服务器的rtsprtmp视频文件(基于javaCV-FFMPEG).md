[javaCV开发详解之3：收流器实现，录制流媒体服务器的rtsp/rtmp视频文件(基于javaCV-FFMPEG)](http://www.yanzuoguang.com/article/716)



## javaCV开发详解之3：收流器实现，录制流媒体服务器的rtsp/rtmp视频文件(基于javaCV-FFMPEG)

## 前言

上一章中实现了本地推流器和本地摄像头录制功能：[http://blog.csdn.net/eguid_1/article/details/52678775](http://www.yanzuoguang.com/article/716#) 本章基于javaCV实现收流器功能和录制功能，补充：基于本功能可以实现远程流媒体服务器实时视频录制到本地

## 一、开发所依赖的包

依赖包与上一章相同，具体依赖包请查看上一章。本章使用windows环境开发，基于javaCV的基础支撑包以及ffmpeg-3.1.2-1.2.jar、ffmpeg-3.1.2-1.2-windows-x86.jar、ffmpeg-3.1.2-1.2-windows-x86_64.jar；

## 二、代码实现

本功能采用按帧实现收流器录制功能

```java
/**
 * 按帧录制视频
 *
 * @param inputFile  该地址可以是网络直播/录播地址，也可以是远程/本地文件路径
 * @param outputFile 该地址只能是文件地址，如果使用该方法推送流媒体服务器会报错，原因是没有设置编码格式
 * @throws IOException
 */
public static void frameRecord(String inputFile, String outputFile) throws IOException {
    // 获取视频源
    FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(inputFile);

    try {
        //建议在线程中使用该方法
        grabber.start();

        // 流媒体输出地址，分辨率（长，高），是否录制音频（0:不录制/1:录制）
        FFmpegFrameRecorder recorder = new FFmpegFrameRecorder(outputFile,
                                                               grabber.getImageWidth(), grabber.getImageHeight(), grabber.getAudioChannels());
        // 设置封装格式
        recorder.setFormat(grabber.getFormat());
        recorder.setFrameRate(grabber.getFrameRate());
        recorder.setOptions(grabber.getOptions());
        recorder.setMetadata(grabber.getMetadata());
        // 设置音频格式
        recorder.setAudioCodec(grabber.getAudioCodec());
        recorder.setAudioBitrate(grabber.getAudioBitrate());
        recorder.setAudioOptions(grabber.getAudioOptions());
        recorder.setAudioMetadata(grabber.getAudioMetadata());
        // 设置视频格式
        recorder.setVideoCodec(grabber.getVideoCodec());
        recorder.setVideoBitrate(grabber.getVideoBitrate());
        recorder.setVideoOptions(grabber.getVideoOptions());
        recorder.setMetadata(grabber.getMetadata());
        // 开始录制
        recorder.start();
        
        Frame frame;
        while ((frame = grabber.grabFrame()) != null) {
            recorder.record(frame);
        }
        recorder.stop();
    } finally {
        grabber.stop();
    }
}
```

## 3、测试收流器录制功能

inputFile设置为服务器播放地址，outputFile设置为本地地址，这里演示.mp4，也可以是flv等其他后缀名

```java
public static void main(String[] args) throws IOException {
    String inputFile = "https://f.video.weibocdn.com/BQjEkeC0lx07EUh2kPCg01041200Fz4U0E010.mp4?label=mp4_hd&template=852x480.25.0&trans_finger=62b30a3f061b162e421008955c73f536&ori=0&ps=1EO8O2oFB1ePdo&Expires=1594978818&ssig=Gx9UFVyfDD&KID=unistore,video";
    // Decodes-encodes
    long first = System.currentTimeMillis();
    String outputFile = "target/recorde_" + first + ".mp4";
    frameRecord(inputFile, outputFile);
}
```

rtsp地址实现:

```java
public static void main(String[] args) throws IOException {
    String inputFile = "rtsp://admin:admin@192.168.2.236:37779/cam/realmonitor?channel=1&subtype=0";
    // Decodes-encodes
    String outputFile = "recorde.mp4";
    frameRecord(inputFile, outputFile);
}
```

到这里我们已经实现了直播功能的全部基本操作：推流，录制，简单的直播系统和多人视频等已经可以实现了；突然发现，额。。。我们的直播系统貌似没有声音！！！好吧，确实是这样，直播听不到声音确实有点low。那么声音要怎么获取呢？看这里实现：[http://blog.csdn.net/eguid_1/article/details/52702385](http://www.yanzuoguang.com/article/716#)

声音会获取了，那么接下来让我们实现一下本地音视频混合推流到服务器吧：[javaCV开发详解之6：本地音频(话筒设备)和视频(摄像头)抓取、混合并推送(录制)到服务器(本地)](http://www.yanzuoguang.com/article/716#)

但是我们的系统远不止那么简单，比如监控和专业的摄像头，需要通过rtsp或者码流的形式才能获取视频流，这时我们需要一个转流器，帮助我们把他们转发到流媒体服务器，实现实时监控/视频查看