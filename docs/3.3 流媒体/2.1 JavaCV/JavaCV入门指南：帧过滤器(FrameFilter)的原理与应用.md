[JavaCV入门指南：帧过滤器(FrameFilter)的原理与应用](http://www.yanzuoguang.com/article/710)

# JavaCV入门指南：帧过滤器(FrameFilter)的原理与应用

## 前言

在此之前，我们分析了FrameGrabber和FrameRecorder，对于音视频、图片和流媒体的输入输出相信大家已经基本掌握和了然于心了。那么接下来的本章，主要讲解和分析FrameFilter，让我们直接开始吧。

## FrameFilter的介绍和结构

FrameFilter就是过滤音频和视频帧，并对音频和视频进行处理的一个帧处理器，用滤镜来描述可能更为贴切一点（但是由于FrameFilter还可以处理音频，所以我们还是使用“过滤器”更合适些，虽然有可能引起歧义就是了），在采集到解码后的音视频源或者图像、音频后，对解码后的数据源进行加工的过程就是FrameFilter做的事情了。

### FrameFilter处理流程

FrameFilter的一般调用处理流程

```shell
初始化和设置解码后的数据--->start()--->循环start| push(Frame frame)--->Farme pull() |循环end--->结束时调用stop释放内存
```

结合FrameGrabber和FrameRecorder后的FrameFilter处理流程，如下图所示![1](https://img-blog.csdnimg.cn/20200624113323759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2VndWlkXzE=,size_16,color_FFFFFF,t_70)

### FrameFilter的子类

FrameFilter只有一个实现类就是FFmpegFrameFilter，所以本章主要分析FFmpegFrameFilter。

## FFmpegFrameFilter剖析

### FFmpegFrameFilter介绍

FFmpegFrameFilter本身就是FrameFilter的实现类， 结构基本相同，使用流程参考上面的流程结构图。

### FFmpegFrameFilter的初始化及参数

视频和音频混合过滤器初始化

```java
FFmpegFrameFilter(String videoFilters, String audioFilters, int imageWidth, int imageHeight, int audioChannels)
```

视频过滤器初始化

```java
FFmpegFrameFilter(String filters, int imageWidth, int imageHeight)
```

音频过滤器初始化

```java
FFmpegFrameFilter(String afilters, int audioChannels)
```

参数

- 视频需要至少三个参数：videoFilter、imageWidth和imageHeight。也就是说必须保证这三个参数（视频过滤器，图像宽度，高度）都不能为空，且图像高度和宽度大小不能太离谱。
- 音频过滤器需要至少两个参数：afilters 和 audioChannels。音频过滤器和音频通道
- 参考FFmpegFrameFilter源码：

```java
if (filters != null && imageWidth > 0 && imageHeight > 0) {
    startVideoUnsafe();
}
if (afilters != null && audioChannels > 0) {
    startAudioUnsafe();
}
```

### start方法

start()方法会对视频过滤器和音频过滤器进行判断，如果有视频过滤器和音频过滤器则就会初始化对应过滤器上下文。

### push方法

可以同时送入视频图像像素和音频采样（Frame对象可以同时包含图像像素和音频采样数据）

```java
push(Frame frame)
```

与上面一样，只不过可以设置视频图像像素格式

```java
push(Frame frame, int pixelFormat)
```

上面两个方法都是下面这个方法的语法糖，也是一样同时支持送入视频和音频， 还支持更多参数，比如视频的宽度（width）、高度（height）、图像深度（depth），图像通道（channels）、图像跨度（stride）、像素格式（pixelFormat），图像像素缓存数据

```java
pushImage(int width, int height, int depth, int channels, int stride, int pixelFormat, Buffer ... image) 
```

只送入音频采样数据，音频通道（audioChannels）, 采样率（sampleRate）, 音频编码格式（sampleFormat）和音频采样缓存

```java
pushSamples(int audioChannels, int sampleRate, int sampleFormat, Buffer ... samples)
```

补充：

- 关于图像深度（depth），图像深度是图像里用来表示一个像素点的颜色由几位构成，比如24位图像深度的1920x1080的高清图像，就表示这个高清图像它所占空间是（1920x1080x24bit/8）kb也就是：1920x1080x3 kb
- 关于图像通道（channels），channels*8=depth，也就说图像深度除8Bit就是图像通道数，图像跨度（stride）就是图像一行所占长度，一般大于等于图像的宽度。

### pull方法

拉取经过视频和音频过滤器处理后的视频图像Frame，如果同时设置了音频和视频过滤器，会同时通过Frame返回，如果只设置了其中一个，则Frame中存放的也是只有其中一个处理后的图像像素或者音频采样。

```java
pull()
```

只拉取经过视频过滤器处理后的图像像素

```java
Frame pullImage()
```

只拉取经过音频过滤器处理后的音频采样

```java
Frame pullSamples()
```

## FFmpegFrameFilter的使用

使用示例参考javaCV开发详解之13：[使用FFmpeg Filter过滤器处理音视频](http://www.yanzuoguang.com/article/710#)