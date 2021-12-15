[JavaCV入门指南：调用opencv原生API和JavaCV是如何封装了opencv的图像处理操作？](http://www.yanzuoguang.com/article/706)

## JavaCV入门指南：调用opencv原生API和JavaCV是如何封装了opencv的图像处理操作？

## 一、前言

通过第二章的JavaCV入门指南：调用FFmpeg原生API和JavaCV是如何封装了FFmpeg的音视频操作大家已经知道javacv的答题结构如图：![1](https://img-blog.csdnimg.cn/20200622134123999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VndWlkXzE=,size_16,color_FFFFFF,t_70)

## 二、javaCPP直接调用opencv的API

我们通过[《实时视频添加文字水印并截取视频图像保存成图片》](http://www.yanzuoguang.com/article/706#)作为实例来阐述，实例地址：

[openCV图像处理之1：实时视频添加文字水印并截取视频图像保存成图片，实现文字水印的字体、位置、大小、粗度、翻转、平滑等操作](http://www.yanzuoguang.com/article/706#)

这部分内容主要是如何调用opencv的API，本系列作为JavaCV入门指南不会讲解opencv的具体用法，如果想要深入学习opencv部分，除了博主的opencv系列之外，还可以参考opencv的官方文档及实例。

## 三、JavaCV是如何封装了opencv的音视频及图像处理操作？

与ffmpeg相似的是，JavaCV把opencv的操作也抽象成了“读取媒体文件或地址，循环抓取图像，停止”的流程，其中不同的是，opencv可以直接循环读取设备列表：详细请参考：

[opencv图像处理3：使用opencv原生方法遍历摄像头设备及调用（方便多摄像头遍历及调用，相比javacv更快的摄像头读取速度和效率，方便读取后的图像处理）](http://www.yanzuoguang.com/article/706#)

JavaCV把opencv的操作分成了两大块：OpenCVFrameGrabber和OpenCVFrameRecorder。其中OpenCVFrameGrabber用来读取设备、视频流媒体和图片文件等，而OpenCVFrameRecorder则用来录制文件。

### 1、OpenCVFrameGrabber读取设备、媒体文件及流

OpenCVFrameGrabber其实内部封装了opencv的VideoCapture操作，支持设备、视频文件、图片文件和流媒体地址（rtsp/rtmp等）。

可以通过 ImageMode设置色彩模式，支持ImageMode.COLOR（色彩图）和ImageMode.GRAY（灰度图）

支持的格式请参考：[http://mp4ra.org/#/codecs](http://www.yanzuoguang.com/article/706#)和[https://www.fourcc.org/codecs.php](http://www.yanzuoguang.com/article/706#)这两个列表。

注意：opencv并不支持音频读取和录制等操作，只支持视频文件、视频流媒体、图像采集设备的画面抓取。另外需要注意的是，读取非动态图片，只能读取一帧。

通过OpenCVFrameRecorder的grab()抓取到的图像是Frame，其实javaCV内部通过OpenCVFrameConverter把opencv的Mat转换为了Frame，也即是说，可以通过OpenCVFrameConverter实现Mat和Frame的互转。

### 2、OpenCVFrameRecorder录制媒体文件

opencv的录制支持的视频编码fourCC的编码集请参考：[http://mp4ra.org/#/codecs](http://www.yanzuoguang.com/article/706#)和[https://www.fourcc.org/codecs.php](http://www.yanzuoguang.com/article/706#)这两个列表。

OpenCVFrameRecorder主要封装了opencv的VideoWriter模块，用来实现视频流媒体的录制操作，支持的格式同样参考：[http://mp4ra.org/#/codecs](http://www.yanzuoguang.com/article/706#)和[https://www.fourcc.org/codecs.php](http://www.yanzuoguang.com/article/706#)这两个列表。

通过循环recordFrame就可以录制视频流媒体，当然如果是进行图像处理操作，得到的是mat，就可以通过OpenCVFrameConverter把Mat转换成Frame即可进行record()录制操作。

### 3、OpenCVFrameConverter进行Mat、 IplImage和Frame的互转

由于我们使用opencv需要进行图像处理等操作，处理完得到的是Mat或者IplImage，读取和录制却是Frame，所以需要使用OpenCVFrameConverter提供的转换操作来完成两个对象间的转换操作。

- IplImage与Frame互转

```java
//把frame转换成IplImage
IplImage convertToIplImage(Frame frame)
//把IplImage转换成frame
 Frame convert(IplImage img) 
```

- Mat与Frame互转

```java
//frame转换成Mat
Mat convertToMat(Frame frame)
//mat转换成frame
Frame convert(Mat mat)
```