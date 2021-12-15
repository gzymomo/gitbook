[JavaCV入门指南：FrameConverter转换工具类及CanvasFrame图像预览工具类（javaCV教程完结篇）](http://www.yanzuoguang.com/article/711)



# JavaCV入门指南：FrameConverter转换工具类及CanvasFrame图像预览工具类（javaCV教程完结篇）

## 前言

再此章之前，我们已经详细介绍和剖析了javacv的结构和ffmpeg和opencv的封装调用方式，以及javacv中重要的FrameGrabber和FrameRecorder的原理和用法，本章是javacv入门指南的最后一章，主要介绍转换工具和图像预览工具类。

## FrameConverter介绍

FrameConverter封装了常用的转换操作，比如opencv与Frame的互转、java图像与Frame的互转以及安卓平台的Bitmap图像与Frame互转操作。

### FrameConverter的子类

- AndroidFrameConverter
- Java2DFrameConverter
- JavaFXFrameConverter
- LeptonicaFrameConverter
- OpenCVFrameConverter

由于JavaCV的Frame完全是仿照ffmpeg的AVFrame格式设计的，所有AVFrame和Frame不存在互转，它们的数据格式基本是互通的，直接赋值即可。

## AndroidFrameConverter互转操作

专门用于安卓平台的转换操作，用于将Bitmap和Frame进行互转，以及提供了额外的yuv转bgr操作。

```java
//Frame转换为Bitmap
Bitmap convert(Frame frame)

//bitmap转换为frame
Frame convert(Bitmap bitmap)

//yuv4:2:0像素转换为BGR像素
/**
* Convert YUV 4:2:0 SP (NV21) data to BGR, as received, for example,
* via {@link Camera.PreviewCallback#onPreviewFrame(byte[],Camera)}.
*/
public Frame convert(byte[] data, int width, int height)
```

## Java2DFrameConverter互转操作

提供了Frame和java图像BufferedImage的互转操作。

```java
//Frame转BufferedImage图像
public BufferedImage getBufferedImage(Frame frame)

// 伽马值，用来调节图像的灰度曲线，与显示设备有关
BufferedImage getBufferedImage(Frame frame, double gamma)
BufferedImage getBufferedImage(Frame frame, double gamma, boolean flipChannels, ColorSpace cs)

// BufferedImage图像转Frame
Frame getFrame(BufferedImage image)
Frame getFrame(BufferedImage image, double gamma)
Frame getFrame(BufferedImage image, double gamma, boolean flipChannels)
```

## JavaFXFrameConverter互转操作

提供了JavaFX的图像Image和Frame的转换操作。

```java
//把javaFX的图像Image转换为javacv的Frame
Frame convert(Image f)

//把Frame转换为javaFX的Image图像对象
Image convert(Frame frame) 
```

## LeptonicaFrameConverter互转操作

用于Leptonica和tesserac的PIX和Frame的互转，Leptonica是图像识别库google tesserac ocr的依赖库，也即是说该工具类一般是用于tesserac的图像PIX对象与Frame互转操作。

```java
//Frame转tesserac的PIX图像
PIX convert(Frame frame)

//tesserac的PIX图像转Frame
 Frame convert(PIX pix)
```

## OpenCVFrameConverter互转操作

主要用于opencv的IplImage/Mat和Frame的互转操作。 IplImage与Frame互转

```java
//把frame转换成IplImage
IplImage convertToIplImage(Frame frame)

//把IplImage转换成frame
 Frame convert(IplImage img) 
```

Mat与Frame互转

```java
//frame转换成Mat
Mat convertToMat(Frame frame)

//mat转换成frame
Frame convert(Mat mat)
```

## CanvasFrame介绍

CanvasFrame是用于预览Frame图像的工具类，但是这个工具类的gama值通常是有问题的，所以显示的图像可能会偏色，但是不影响最终图像的色彩。

### CanvasFrame的原理

CanvasFrame内部是使用的swing的Canvas画板操作，使用canvas画板绘制图像。

### CanvasFrame的使用

```java
CanvasFrame canvas = new CanvasFrame("转换apng中屏幕预览");// 新建一个窗口
canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
canvas.setAlwaysOnTop(true)；

 //显示画面，这个操作用来显示Frame，一般Frame从各个FrameGrabber中获取或者从各个converter转换类中而来。
canvas.showImage(frame);
```