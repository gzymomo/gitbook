[JavaCV入门指南：帧录制器/推流器(FrameRecorder)的原理与应用](http://www.yanzuoguang.com/article/709)



# JavaCV入门指南：帧录制器/推流器(FrameRecorder)的原理与应用

\## 前言 上一章大体讲解了FrameGrabber（抓取器/采集器），本章就FrameRecorder展开探索。

## FrameRecorder(录制器/推流器)介绍

用于音视频/图片的封装、编码、推流和录制保存等操作。把从FrameGrabber或者FrameFilter获取的Frame中的数据取出并进行编码、封装、推流发送等操作流程。为了方便理解和阅读，下文开始我们统一把FrameRecorder简称为：**录制器**。

### FrameRecorder的结构和分析

FrameRecorder与FrameGrabber类似，也是个抽象类，抽象了所有录制器的通用方法和一些共用属性。FrameRecorder只有两个子类实现：**FFmpegFrameRecorder**和**OpenCVFrameRecorder**

### 两个FrameRecorder实现类的介绍

- FFmpegFrameRecorder：使用ffmpeg作为音视频编码/图像、封装、推流、录制库。除了支持音视频录制推流之外，还可以支持gif/apng动态图录制，对于音视频这块的功能ffmpeg依然还是十分强大的保障。
- OpenCVFrameRecorder：使用opencv的videowriter录制视频，不支持像素格式转换，根据opencv的官方介绍，如果在ffmpeg可用的情况下，opencv的视频功能可能也是基于ffmpeg的，macos下基于avfunctation。

总得来说，音视频这块还是首选ffmpeg的录制器。但是凡事都有例外，由于javacv的包实在太大，开发的时候下载依赖都要半天。为了减少程序的体积大小，如果只需要进行图像处理/图像那个识别的话只使用opencv的情况就能解决问题，那就opencvFrameGrabber和OpenCVFrameRecorder配套使用吧，毕竟可以省很多空间。

同样的，如果只是做音视频流媒体，那么能使用ffmpeg解决就不要用其他库了，能节省一点空间是一点，确实javacv整个完整包太大了，快要1G了。。。这些都是题外话了，还是回归正题吧。

### FrameRecorder的结构和流程

FrameRecorder的整个代码结构和运作流程很简单

```shell
初始化和设置参数--->start()--->循环record(Frame frame)--->close()
```

close()包含stop()和release()两个操作，这个要注意。由于只有两个实现类，就直接分开单独分析了

## FFmpegFrameRecorder结构和分析

FFmpegFrameRecorder是比较复杂的，我们主要它来实现像素格式转换、视频编码和录制推流。我们把流程分为转封装流程和转码流程

1. 转封装流程：

```shell
FFmpegFrameRecorder初始化-->start()-->循环recordPacket(AVPacket)-->close()
```

这里的AVPacket是未解码的解复用视频帧。转封装的示例：转封装在rtsp转rtmp流中的应用 2. 转码编码流程：

```
FFmpegFrameRecorder初始化-->start()-->循环record(Frame)/recordImage()/recordSamples()-->close()
```

转码的示例：

- [转流器实现](http://www.yanzuoguang.com/article/709#)
- [视频文件转gif动态图片实现](http://www.yanzuoguang.com/article/709#)
- [视频转apng动态图片实现](http://www.yanzuoguang.com/article/709#)

### FFmpegFrameRecorder初始化及参数说明

FFmpegFrameRecorder初始化支持文件地址、流媒体推流地址、OutputStream流的形式和imageWidth(视频宽度)、imageHeight（图像高度）、audioChannels（音频通道，1-单声道，2-立体声）

FFmpegFrameRecorder参数较多，不一一列举，直接看代码中的参数说明：

```java
FFmpegFrameRecorder recorder=new FFmpegFrameRecorder(output, width,height,0);
recorder.setVideoCodecName(videoCodecName);//优先级高于videoCodec
recorder.setVideoCodec(videoCodec);//只有在videoCodecName没有设置或者没有找到videoCodecName的情况下才会使用videoCodec
//recorder.setFormat(format);//只支持flv，mp4，3gp和avi四种格式，flv:AV_CODEC_ID_FLV1;mp4:AV_CODEC_ID_MPEG4;3gp:AV_CODEC_ID_H263;avi:AV_CODEC_ID_HUFFYUV;
recorder.setPixelFormat(pixelFormat);// 只有pixelFormat，width，height三个参数任意一个不为空才会进行像素格式转换
recorder.setImageScalingFlags(imageScalingFlags);//缩放，像素格式转换时使用，但是并不是像素格式转换的判断参数之一
recorder.setGopSize(gopSize);//gop间隔
recorder.setFrameRate(frameRate);//帧率
recorder.setVideoBitrate(videoBitrate);
recorder.setVideoQuality(videoQuality);
//下面这三个参数任意一个会触发音频编码
recorder.setSampleFormat(sampleFormat);//音频采样格式,使用avutil中的像素格式常量，例如：avutil.AV_SAMPLE_FMT_NONE
recorder.setAudioChannels(audioChannels);
recorder.setSampleRate(sampleRate);
recorder.start();
```

### FFmpegFrameRecorder的start

start中其实做了很多事情：一堆初始化操作、打开网络流、查找编码器、把format字符转换成对应的videoCodec和videoFormat等等。

### FFmpegFrameRecorder中的各种record

- record(Frame frame)：编码方式录制，把已经解码的图像像素编码成对应的编码和格式推流出去。
- record(Frame frame, int pixelFormat)：多出一个pixelFormat像素格式参数就是因为它会触发像素格式转换，如果像素格式与编码器要求的像素格式不同的时候，就会进行转换像素格式，在转换的时候同时也会转换width、height、imageScalingFlags。像素格式转换参考示例：[javaCV开发详解之15：视频帧像素格式转换](http://www.yanzuoguang.com/article/709#)
- recordImage(int width, int height, int depth, int channels, int stride, int pixelFormat, Buffer ... image)：其实上面两个都是调用的这个方法。Buffer[] image是从Frame frame里的image参数获取的，所以很多小伙伴问我FrameGrabber解码出来的图像像素在哪里？看，它在frame.image里。
- recordSamples(Buffer ... samples)：用来编码录制音频的，它调用了下面的方法，Buffer ... samples是从Frame的frame.samples来，所以如上所述，很多同学来问我FrameGrabber解码出来的音频采样存哪里去了？看，它还在Frame里面。
- recordSamples(int sampleRate, int audioChannels, Buffer ... samples)：用来编码录制音频的，它会触发重采样，当samples_channels（音频通道）、samples_format（音频采样格式）和samples_rate（采样率）任意参数不为空的时候就会触发重采样。音频重采样示例：[javaCV开发详解之14：音频重采样](http://www.yanzuoguang.com/article/709#)
- recordPacket(AVPacket pkt):转封装操作，用来放未经过解码的解复用视频帧。

### FFmpegFrameRecorder中的stop和close

非常需要的注意的是，当我们在录制文件的时候，一定要保证stop()方法被调用，因为stop里面包含了写出文件头的操作，如果没有调用stop就会导致录制的文件损坏，无法被识别或无法播放。close()方法中包含stop()和release()方法

## OpenCVFrameRecorder结构分析

OpenCVFrameRecorder的代码很简单，不到120行的代码，主要是基于opencv的videoWriter封装，流程与FrameRecorder的相同：

```shell
初始化和设置参数--->start()--->循环record(Frame frame)--->close()
```

在start中会初始化opencv的VideoWriter，VideoWriter是opencv中用来保存视频的模块，是比较简单的，可以对照参考opencv官方文档介绍：[https://docs.opencv.org/master/dd/d9e/classcv_1_1VideoWriter.html](http://www.yanzuoguang.com/article/709#)。

### OpenCVFrameRecorder的初始化和参数设置

OpenCVFrameRecorder初始化参数只有三个：filename（文件名称或者文件保存地址），imageWidth（图像宽度）, imageHeight（图像高度）。但是OpenCVFrameRecorder的有作用的参数只有六个，其中pixelFormat和videoCodec这两个比较特殊。

- pixelFormat：该参数并不像ffmpeg那样表示像素格式，这里只表示原生和灰度图，其中pixelFormat=1表示原生，pixelFormat=0表示灰度图。
- videoCodec：这个参数，在opencv中对应的是fourCCCodec，所以编码这块的设置也要按照opencv的独特方式来设置，有关opencv的视频编码fourCC的编码集请参考：[http://mp4ra.org/#/codecs](http://www.yanzuoguang.com/article/709#)和[https://www.fourcc.org/codecs.php](http://www.yanzuoguang.com/article/709#)这两个列表，由于列表数据较多，这里就不展示了。
- filename（文件名称或者文件保存地址），imageWidth（图像宽度）, imageHeight（图像高度）和frameRate（帧率）这四个参数就不过多赘述了。

### OpenCVFrameRecorder开始录制start

调用start时其实就是初始化了opencv的VideoWriter。这个比较简单，传了上面那六个参数。

### OpenCVFrameRecorder录制record

在循环中把Frame放进record中就可以一直录制，内部是通过OpenCVFrameConverter把Frame转换成了Mat调用VideoWriter进行录制。

### OpenCVFrameRecorder销毁stop/close

不管是stop还是close，其实都是调用的release()方法， release则是调用了opencv的VideoWriter的release()，结构很简单。