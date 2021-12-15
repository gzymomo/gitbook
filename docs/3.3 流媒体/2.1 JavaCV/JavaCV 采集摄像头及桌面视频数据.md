[JavaCV 采集摄像头及桌面视频数据](https://www.cnblogs.com/itqn/p/14531823.html)

javacv 封装了javacpp-presets库很多native API，简化了开发，对java程序员来说比较友好。

> 之前使用JavaCV库都是使用ffmpeg native API开发，这种方式使用起来太多坑了，还是使用JavaCV封装好的库开发方便。

### 引入依赖

前几天刚刚发布了1.5.5，这里使用最新的javacv依赖：

```xml
<properties>
  <javacpp.version>1.5.5</javacpp.version>
</properties>
<dependencies>
  <dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv</artifactId>
    <version>${javacpp.version}</version>
  </dependency>
  <dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>${javacpp.version}</version>
  </dependency>
</dependencies>
```

### OpenCVFrameGrabber采集摄像头数据

javacv的demo中有摄像头及麦克风采集音视频数据的例子（`WebcamAndMicrophoneCapture`），例子采集摄像头数据使用的就是`OpenCVFrameGrabber`，视频画面回显采用的是`CanvasFrame`。

```java
public class Sample01_Camera {

  @SuppressWarnings("resource")
  public static void main(String[] args) throws Exception {
    OpenCVFrameGrabber grabber = new OpenCVFrameGrabber(0);
    grabber.setImageWidth(1280);
    grabber.setImageHeight(720);
    grabber.start();
    CanvasFrame canvas = new CanvasFrame("摄像头");
    canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    canvas.setAlwaysOnTop(true);

    while (canvas.isDisplayable()) {
      canvas.showImage(grabber.grab());
      TimeUnit.MILLISECONDS.sleep(40);
    }
    
    grabber.stop();
  }
}
```

画面预览效果图：

![img](https://img2020.cnblogs.com/blog/2083963/202103/2083963-20210314103025488-362776866.png)

### FFmpegFrameGrabber采集摄像头数据

FFmpegFrameGrabber采集摄像头数据需要指定输入，如：`video=Integrated Camera`，这里`Integrated Camera`是摄像头的设备名。

可以通过以下方式获取摄像头设备名：

1. 打开“计算机管理”->“设备管理器”->“照相机”
2. 使用ffmpeg命令，具体查看之前我的文章 [JavaCV FFmpeg采集摄像头YUV视频数据](https://www.cnblogs.com/itqn/p/13789079.html)

采用FFmpeg查看本机设备的命令：

```shell
ffmpeg.exe -list_devices true -f dshow -i dummy  
```

FFmpegFrameGrabber的使用方式跟OpenCVFrameGrabber的方式是一样的，只不过OpenCVFrameGrabber指定的是设备索引，而FFmpegFrameGrabber指定设备输入。

```java
public class Sample02_Camera {
  @SuppressWarnings("resource")
  public static void main(String[] args) throws Exception {
    FFmpegFrameGrabber grabber = new FFmpegFrameGrabber("video=Integrated Camera");
    grabber.setImageWidth(1280);
    grabber.setImageHeight(720);
    grabber.setFormat("dshow");
    grabber.start();
    CanvasFrame canvas = new CanvasFrame("摄像头");
    canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    canvas.setAlwaysOnTop(true);

    while (canvas.isDisplayable()) {
      canvas.showImage(grabber.grab());
      TimeUnit.MILLISECONDS.sleep(40);
    }

    grabber.stop();
  }
}
```

### FFmpegFrameGrabber采集桌面数据

FFmpegFrameGrabber采集桌面采用`gdigrab`，参考雷神的博客 [FFmpeg源代码简单分析：libavdevice的gdigrab](https://blog.csdn.net/leixiaohua1020/article/details/44597955)。

```java
public class Sample03_Desktop {

  // https://blog.csdn.net/leixiaohua1020/article/details/44597955
  @SuppressWarnings("resource")
  public static void main(String[] args) throws Exception {
    FFmpegFrameGrabber grabber = new FFmpegFrameGrabber("desktop");
    grabber.setFormat("gdigrab");
    grabber.setOption("offset_x", "0");
    grabber.setOption("offset_y", "0");
    grabber.setOption("framerate", "25");
    grabber.setOption("draw_mouse", "0");
    grabber.setOption("video_size", "1920x1080");
    // 这种形式，双屏有问题
    // grabber.setImageWidth(1920);
    // grabber.setImageWidth(1080);
    grabber.start();
    CanvasFrame canvas = new CanvasFrame("摄像头");
    canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    canvas.setAlwaysOnTop(true);

    while (canvas.isDisplayable()) {
      canvas.showImage(grabber.grab());
      TimeUnit.MILLISECONDS.sleep(40);
    }

    grabber.stop();
  }
}
```

画面预览效果图：

![img](https://img2020.cnblogs.com/blog/2083963/202103/2083963-20210314111542216-1548848273.png)

这里由于我的电脑是2K屏，这里1920x1080只是截图了屏幕的一部分。