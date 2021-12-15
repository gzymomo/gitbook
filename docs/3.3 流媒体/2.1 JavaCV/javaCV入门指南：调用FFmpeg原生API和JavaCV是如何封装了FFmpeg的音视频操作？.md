[JavaCV入门指南：调用FFmpeg原生API和JavaCV是如何封装了FFmpeg的音视频操作？](http://www.yanzuoguang.com/article/705)



# 介绍

本章将正式开始javaCV之旅，先看一下官方文档里的介绍:

JavaCV是计算机视觉领域的开发人员（OpenCV、FFmpeg、libdc1394、PGR FlyCapture、OpenKinect、li.lsense、CL PS3 Eye Driver、videoInput、ARToolKitPlus、flandmark、Leptonica和Tesseract）常用库的JavaCPP预置的包装器，并提供实用的程序类使它们的功能更容易在Java平台上使用，包括Android。

JavaCV还提供了硬件加速的全屏图像显示（CanvasFrame和GLCanvasFrame）、在多核（并行）上并行执行代码的简便方法、照相机和投影机的用户友好的几何和颜色校准（GeometricCalibrator，ProCamometricCalibrato）r，ProCamColorCalibrator），特征点的检测和匹配（ObjectFinder），一组用于实现投影仪-照相机系统的直接图像对齐的类（主要是GNImageAligner、ProjectiveTransformer、ProjectiveColorTransformer、ProCamTransformer和ReflectanceInitializer），一个blob分析包（BLUB），以及JavaCV类中的各种功能。其中一些类还具有OpenCL和OpenGL的对应类，它们的名称以CL结尾或以GL开始，即：JavaCVCL、GLCanvasFrame等。

## 一、什么是JavaCPP

大家知道FFmpeg是C语言中著名的音视频库（注意，不是c++。使用c++调用ffmpeg库的性能损失与Java方式调用损耗相差并不大）。

JavaCV利用JavaCPP在FFmpeg和Java之间构建了桥梁，我们通过这个桥梁可以方便的调用FFmpeg，当然这并不是没有损失的，性能损失暂且不提，最主要问题在于调用ffmpeg之于jvm是native方法，所以通过ffmpeg创建的结构体实例与常量、方法等等都是使用堆外内存，都需要像C那样手动的释放这些资源（jvm并不会帮你回收这部分），以此来保证不会发生内存溢出/泄露等风险。

Javapp在Java内部提供了对本地C++的高效访问，这与一些C/C++编译器与汇编语言交互的方式不同。不需要发明新的语言，比如SWIG、SIP、C++、CLI、Cython或Rython。相反，类似于CPpyy为Python所做的努力，它利用了Java和C++之间的语法和语义相似性。在引擎盖下，它使用JNI，因此除了Java、SE和RoboVM（指令）之外，它还适用于Java SE的所有实现...

详细描述请参考：[https://github.com/bytedeco/javacpp](http://www.yanzuoguang.com/article/705#)

## 二、javaCPP直接调用FFmpeg的API

我们通过《视频拉流解码成YUVJ420P，并保存为jpg图片》作为实例来阐述，实例地址：[https://blog.csdn.net/eguid_1/article/details/81369055](http://www.yanzuoguang.com/article/705#)

这部分内容主要是如何调用FFmpeg的API，本系列作为JavaCV入门不会讲解FFmpeg的具体用法，如果想要深入学习FFmpeg部分，可以选择通过查看FFmpeg的API手册ffmpeg.org，或者访问雷霄骅的博客详细学习FFmpeg的使用。

## 三、JavaCV是如何封装了FFmpeg的音视频操作？

JavaCV通过JavaCPP调用了FFmpeg，并且对FFmpeg复杂的操作进行了封装，把视频处理分成了两大类：“帧抓取器”（FrameGrabber）和“帧录制器”（又叫“帧推流器”，FrameRecorder）以及用于存放音视频帧的Frame（FrameFilter暂且不提）。

整体结构如下：

```shell
视频源---->帧抓取器（FrameGabber） ---->抓取视频帧（Frame）---->帧录制器（FrameRecorder）---->推流/录制---->流媒体服务/录像文件
```

### 1、帧抓取器（FrameGrabber）

封装了FFmpeg的检索流信息，自动猜测视频解码格式，音视频解码等具体API，并把解码完的像素数据（可配置像素格式）或音频数据保存到Frame中返回。

帧抓取器详细剖析：[帧抓取器(FrameGrabber)的原理与应用](http://www.yanzuoguang.com/article/705#)

### 2、帧录制器/推流器（FrameRecorder）

封装了FFmpeg的音视频编码操作和封装操作，把传参过来的Frame中的数据取出并进行编码、封装、发送等操作流程。

帧录制器/推流器详细剖析：[帧录制器/推流器（FrameRecorder）的原理与应用](http://www.yanzuoguang.com/article/705#)

### 3、过滤器（FrameFilter）

FrameFilter的实现类其实只有FFmpegFrameFilter，因为只有ffmpeg支持音视频的过滤器操作，主要封装了简单的ffmpeg filter操作。

过滤器详细剖析：[帧过滤器(FrameFilter)的原理与应用](http://www.yanzuoguang.com/article/705#)

### 4、Frame

用于存放解码后的视频图像像素和音频采样数据（如果没有配置FrameGrabber的像素格式和音频格式，那么默认解码后的视频格式是yuv420j，音频则是pcm采样数据）。

里面包含解码后的图像像素数据，大小（分辨率）、音频采样数据，音频采样率，音频通道（单声道、立体声等等）等等数据

Frame里面的一个字段opaque引用AVFrame、AVPacket、Mat等数据，也即是说，如果你是解码后获取的Frame，里面存放的属性找不到你需要的，可以从opaque属性中取需要的AVFrame原生属性。

例如：

```java
// 获取视频解码后的图像像素，也就是说这时的Frame中的opaque存放的是AVFrame
Frame frame = grabber.grabImage();
// 把Frame直接强制转换为AVFrame
AVFrame avframe=(AVFrame)frame.opaque;
 long lastPts=avframe.pts();
 System.err.println("显示时间："+lastPts);
```

## 补充：

FFmpeg中两个重要的结构体：AVPacket和AVFrame。

AVPacket是ffmpeg中存放解复用（未解码）的音视频帧的结构体，视频只有可能是一帧，大小不定（分为关键帧/I帧、P帧和B帧三种视频帧）。AVPacket属性除了包含音视频帧以外，还包含：

- pts: 显示时间
- dts: 解码时间
- duration: 持续时长
- stream_index: 表示音视频流的通道，音频和视频一般是分开的，通过stream_index来区分是视频帧还是音频帧
- flags: AV_PKT_FLAG_KEY（值是1）-关键帧，2-损坏数据，4-丢弃数据
- pos: 在流媒体中的位置
- size

某些情况下，使用AVPacket直接推流（不经过转码）的过程称之为：转封装。

AVFrame是ffmpeg中存放解码后的音视频图像像素或音频采样数据的结构体，大部分属性是与Frame相同的，多了像素格式、pts、dts和音频布局等等属性。

AVPakcet和AVFrame的使用流程如下图所示：![1](http://www.yanzuoguang.com/upload/2020/07/7n72a5t1rog5mq6mcmhmulm2k8.png)