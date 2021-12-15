[JAVA 中通过 JavaCV 实现跨平台视频 / 图像处理 - 调用摄像头](http://www.yanzuoguang.com/article/701)

# JAVA 中通过 JavaCV 实现跨平台视频 / 图像处理 - 调用摄像头

我对源代码加了一点注释，也补充了一些资料

## 一、简介

　　JavaCV 使用来自计算机视觉领域 (OpenCV, FFmpeg, libdc1394, PGR FlyCapture, OpenKinect, librealsense, CL PS3 Eye Driver, videoInput, ARToolKitPlus, flandmark, Leptonica, and Tesseract) 领域的研究人员常用库的 JavaCPP 预设的封装。提供实用程序类，使其功能更易于在 Java 平台上使用，包括 Android。

## 二、案例 1：调用摄像头

（1）使用 IDEA 新建 Maven 项目，然后在 pom.xml 中引入下列依赖（因为要用到 opencv 来实现，所以需要引入 opencv-platform 包，此包会自动引入各大平台的依赖 jar（内含 dll））：

```xml
<!-- https://mvnrepository.com/artifact/org.bytedeco/javacv-platform -->
        <dependency>
            <groupId>org.bytedeco</groupId>
            <artifactId>javacv-platform</artifactId>
            <version>1.4.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.bytedeco.javacpp-presets/opencv-platform -->
        <dependency>
            <groupId>org.bytedeco.javacpp-presets</groupId>
            <artifactId>opencv-platform</artifactId>
            <version>3.4.1-1.4.1</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

（2）测试类

```java
package com.jiading;

import org.bytedeco.javacv.*;
import org.junit.Test;

import javax.swing.*;

public class camera {
    @Test
    public void testCamera() throws FrameGrabber.Exception, InterruptedException {
        OpenCVFrameGrabber grabber = new OpenCVFrameGrabber(0);//抓取视频帧
        grabber.start();//开始获取摄像头数据
        CanvasFrame canvas = new CanvasFrame("摄像头");//新建一个窗口
        canvas.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);//设置窗口关闭时的动作
        canvas.setAlwaysOnTop(true);//窗口永远在其他窗口上方
        while(true){
            if(!canvas.isDisplayable()){//如果窗口关闭的话
                grabber.stop();//停止采集
                System.exit(-1);
            }
            Frame frame=grabber.grab();//抓取页面
            canvas.showImage(frame);//获取摄像头图像并放到窗口上显示， 这里的Frame frame=grabber.grab(); frame是一帧视频图像
            Thread.sleep(10);//切换刷新率可以决定视频是否连贯
        }
    }
    @Test
    public void testCamera1() throws FrameGrabber.Exception, InterruptedException {
        VideoInputFrameGrabber grabber=VideoInputFrameGrabber.createDefault(0);
        grabber.start();
        CanvasFrame canvas=new CanvasFrame("摄像头");
        canvas.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        canvas.setAlwaysOnTop(true);
        while(true){
            if(!canvas.isDisplayable()){
                grabber.stop();
                System.exit(-1);
            }
            Frame frame=grabber.grab();
            canvas.showImage(frame);
            Thread.sleep(20);
        }
    }
}
```

这里用到了一个WindowConstants接口，[这篇文章](http://www.yanzuoguang.com/article/701#)对其进行了分析:

```java
package javax.swing;

public interface WindowConstants
{   
    // The do-nothing default window close operation.
    public static final int DO_NOTHING_ON_CLOSE = 0;  
    // The hide-window default window close operation.
    public static final int HIDE_ON_CLOSE = 1;  
    // The dispose-window default window close operation.
    public static final int DISPOSE_ON_CLOSE = 2; 
    // The exit application default window close operation.
    public static final int EXIT_ON_CLOSE = 3; 

}
```

| 常量名              | 代表值 | 含义                                            |
| ------------------- | ------ | ----------------------------------------------- |
| DO_NOTHING_ON_CLOSE | 0      | 关闭窗体时，不执行任何操作                      |
| HIDE_ON_CLOSE       | 1      | 关闭窗体时，窗体自动隐藏                        |
| DISPOSE_ON_CLOSE    | 2      | 关闭窗体时，自动隐藏并释放窗体                  |
| EXIT_ON_CLOSE       | 3      | 关闭窗体时，退出应用程序（Application非Applet） |

　原来这些常量并非是JFrame所特有的，而是在一个特定的接口中定义的。当然，在WindowConstants接口中也有相关说明： The setDefaultCloseOperation and getDefaultCloseOperation methods provided by **JFrame**, **JInternalFrame**, and **JDialog**. 也就是说，JFrame、JInternalFrame和JDialog也实现了WindowConstants接口。

> 　　JFrame中重新定义了 public static final int EXIT_ON_CLOSE = 3; ，因此上面代码中的 JFrame.EXIT_ON_CLOSE 无论无何是合理的。

（3）两种方法的测试效果：

![1](http://www.yanzuoguang.com/upload/2020/07/qj2vjaeh6sjj4rr8puk4oalg6m.png)

其实这两种方法唯一的区别就是grabber的类不一样，一个是OpenCVFrameGrabber，另外一个是VideoInputFrameGrabber，其他所有的代码一模一样。从下图我们也可以看出来，这两个类都是继承自同一个类，所以功能很像

![2](http://www.yanzuoguang.com/upload/2020/07/q5949avhhiisbrml3npehp1kd2.png)![3](http://www.yanzuoguang.com/upload/2020/07/0h7cqjc4kegr5rikm31tit62jc.png)

但是第二种方法在我的电脑上报错了:Platform "linux-x86_64" not supported by class org.bytedeco.javacpp.videoInputLib.考虑到原博主的版本可能并不是最新的，不一定现在的版本就不支持