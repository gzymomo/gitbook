[JavaCV入门指南：帧抓取器(FrameGrabber)的原理与应用](http://www.yanzuoguang.com/article/708)

# JavaCV入门指南：帧抓取器(FrameGrabber)的原理与应用

## 前言

上一章大体讲解了javaCV的结构，本章就具体的FrameGrabber实现方式展开探索。

## FrameGrabber(帧抓取器/采集器)介绍

用于采集/抓取视频图像和音频采样。封装了检索流信息，自动猜测视频解码格式，音视频解码等具体API，并把解码完的像素数据（可配置像素格式）或音频数据保存到Frame中返回等功能。

## FrameGrabber的结构

FrameGrabber本身是个抽象类，抽象了所有抓取器的通用方法和一些共用属性。FrameGrabber的子类/实现类包含以下几个：

- FFmpegFrameGrabber
- OpenCVFrameGrabber
- IPCameraFrameGrabber
- VideoInputFrameGrabber(仅支持Windows)
- FlyCapture2FrameGrabber
- DC1394FrameGrabber
- RealSenseFrameGrabber
- OpenKinectFrameGrabber
- OpenKinect2FrameGrabber
- PS3EyeFrameGrabber

## 几种FrameGrabber子类实现介绍

### FFmpegFrameGrabber

强大到离谱的音视频处理/计算机视觉/图像处理库，视觉和图像处理这块没有opencv强大；

可以支持网络摄像机：udp、rtp、rtsp和rtmp等，支持本机设备，比如屏幕画面、摄像头、音频话筒设备等等、还支持视频文件、网络流（flv、rtmp、http文件流、hls、等等等）

### OpenCVFrameGrabber

Intel开源的opencv计算机视觉/图像处理库，也支持读取摄像头、流媒体等，但是一般常用于读取图片，处理图像，流媒体音视频这块功能没有ffmpeg强大；

也支持rtsp视频流（不支持音频），本地摄像机设备画面采集、视频文件、网络文件、图片等等。

### IPCameraFrameGrabber

基于openCV调用“网络摄像机”。可以直接用openCV调用，所以一般也不需要用这个抓取器，例如这个网络摄像机地址：[http://IP:port/videostream.cgi?user=admin&pwd=12345&.mjpg](http://www.yanzuoguang.com/article/708#)；

### VideoInputFrameGrabber

只支持windows，用来读取通过USB连接的摄像头/相机。videoinput库已经被内置到opencv中了，另外videoinput调用摄像机的原理是dshow方式，FFmpegFrameGrabber和OpenCVFrameGrabber都支持dshow方式读取摄像机/相机，所以这个库一般用不到；

### FlyCapture2FrameGrabber

通过USB3.1,USB2.0、GigE和FireWire（1394）四种方式连接的相机，比videoinput要慢一些；

### DC1394FrameGrabber

另一个支持MAC的FireWire（1394）外接摄像头/相机，这种接口现在已经不常见了；

### RealSenseFrameGrabber

支持Intel的3D实感摄像头，基于openCV；

### OpenKinectFrameGrabber

支持Xbox Kinect v1体感摄像头，通过这个采集器可支持mac\linux\windows上使用Kinect v1；

### OpenKinect2FrameGrabber

用来支持Xbox Kinect v2（次世代版）体感摄像头；

### PS3EyeFrameGrabber

没错就是那个sony的PS3，用来支持PS3的体感摄像头。

## FrameGrabber及其实现类的一般使用流程

- new初始化及初始化设置
- start（用于启动一些初始化和解析操作）-
- 循环调用grab()获取音视频
- stop（销毁所有对象，回收内存）

1. 先通过各种new实现类进行初始化以及一些设置（比如设置分辨率、帧率等等）
2. 然后调用start()，start中调用了一系列的解析操作，所以会很耗时，尤其是当读取流媒体时，根据当前网络状态情况阻塞时间会长一些。
3. 调用start()方法时会等待（阻塞）一会儿就可以获取到源的信息（如果时视频源，这时就可以获取到视频源的相关信息了，比如格式，编码，分辨率和码率等等属性）,我们一般在这一步的时候把一些需要用到的属性存起来，硫代Recorder或Filter或者进行其他图像处理等操作时使用。
4. 通过for循环不断的调用grab()获取视频帧或者音频帧，FrameGrabber中有几种grab：

- Frame grab() : 通用获取解码后的视频图像像素和音频采样，一般的FrameGrabber子类实现只有这个，特殊的子类实现会多几种获取方法。
- Frame grabFrame() : FFmpegFrameGrabber特有，等同于上面的grab()
- Frame grabImage() : FFmpegFrameGrabber特有，只获取解码后的视频图像像素
- Frame grabSamples() : FFmpegFrameGrabber特有，只获取音频采样
- Frame grabKeyFrame() : FFmpegFrameGrabber特有，只获取关键帧
- Avpacket grabPacket() : FFmpegFrameGrabber特有，获取解复用的音/视频帧，也就是grabPacket()可以获取没有经过解码的音/视频帧。
- void grab() 后还需要再调用getVideoImage()、getIRImage()和getDepthImage() : 这是OpenKinectFrameGrabber特有的来获取体感摄像头图像的方法
- grabDepth()、grabIR()和grabVideo() : 这是OpenKinect2FrameGrabber和RealSenseFrameGrabber特有的调用体感摄像头的方法
- grabBufferedImage() : 是IPCameraFrameGrabber用于调用网络摄像机图像的方法
- grab_RGB4()和grab_raw() : 是PS3EyeFrameGrabber用于调用体感摄像头图像的方法

## FrameGrabber 的使用

### 1、采集/抓取的源：

- 读取流媒体文件（视频、图片、音乐和字幕等等文件），例如：[收流器实现，录制流媒体服务器的rtsp/rtmp视频文件(基于javaCV-FFMPEG)](http://www.yanzuoguang.com/article/708#)
- 拉流（拉取rtsp/rtmp/hls/flv等等流媒体），例如：[收流器实现，录制流媒体服务器的rtsp/rtmp视频文件(基于javaCV-FFMPEG)](http://www.yanzuoguang.com/article/708#)
- 采集摄像头图像（支持linux/mac/windows/安卓等等多种设备摄像机采集），例如：[opencv方式调用本机摄像头视频](http://www.yanzuoguang.com/article/708#)；[基于dshow调用windows摄像头视频和音频，想要获取windows屏幕画面首选gdigrab](http://www.yanzuoguang.com/article/708#)
- 采集设备音频采样（支持linux/mac/windows/安卓等等多种设备音频采集）
- 抓取设备桌面屏幕画面（支持linux/mac/windows/安卓等等多种设备桌面屏幕画面抓取，支持鼠标绘制），例如：[基于gdigrab的windows屏幕画面抓取/采集（基于javacv的屏幕截屏、录屏功能）](http://www.yanzuoguang.com/article/708#)；[基于avfoundation的苹果Mac和ios获取屏幕画面及录屏/截屏以及摄像头画面和音频采样获取实现](http://www.yanzuoguang.com/article/708#)
- 其他设备/虚拟设备采集（比如支持Xbox Kinect2\PS3 Eye\RealSense体感摄像头）
- 通过USB等接口外接的设备（比如USB3.1,USB2.0、GigE和FireWire（1394）等方式外接的摄像头等）
- 读取视频裸流，可以通过FFmpegFrameGrabber的构造函数提供构造好的InputStream，可以把裸流通过inputstream传进FrameGrabber中，就可以通过FrameGrabber进行解复用和解码等操作。

### 2、一些初始化设置

除了可以设置源的编码格式、像素格式和音频格式，音频编码等采集属性。还可以设置比如视频/桌面屏幕分辨率，帧率，音频采样率等等属性。示例参考：

- 音频重采样[https://eguid.blog.csdn.net/article/details/106813330](http://www.yanzuoguang.com/article/708#)
- 视频帧像素格式转换[https://eguid.blog.csdn.net/article/details/106823269](http://www.yanzuoguang.com/article/708#)

### 3、start后就能够读取到源的信息

比如视频的格式、编码、分辨率、帧率、时长和码率等等信息，音频可以读取到音频的帧率、采样率、比特率

### 4、抓取/采集到的数据（图像、音频）

调用start之后视频支持采集到解复用（未解码）后的视频帧，视频帧解码后的图像像素数据。

音频支持解码后的pcm采样等等。