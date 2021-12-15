[javacv开发详解之1：调用本机摄像头视频（建议使用javaCV最新版本）](http://www.yanzuoguang.com/article/674)

## 前言

javacv开发包是用于支持java多媒体开发的一套开发包，可以适用于本地多媒体（音视频）调用以及音视频，图片等文件后期操作（图片修改，音视频解码剪辑等等功能），这里只使用最简单的本地摄像头调用来演示一下javacv的基础功能

### 重要：

1. 建议使用最新javaCV1.5版本，该版本已解决更早版本中已发现的大部分bug
2. javacv系列文章使用6个jar包：

- javacv.jar
- javacpp.jar
- ffmpeg.jar
- ffmpeg-系统平台.jar
- opencv.jar
- opencv-系统平台.jar

其中ffmpeg-系统平台.jar，opencv-系统平台.jar中的系统平台根据开发环境或者测试部署环境自行更改为对应的jar包，比如windows7 64位系统替换为ffmpeg-x86-x64.jar

为什么要这样做：因为ffmpeg-系统平台.jar中存放的是c/c++本地so/dll库，而ffmpeg.jar就是使用javacpp封装的对应本地库java接口的实现，而javacpp就是基于jni的一个功能性封装包，方便实现jni，javacv.jar就是对9个视觉库进行了二次封装，但是实现的功能有限，所以建议新手先熟悉openCV和ffmpeg这两个C/C++库的API后再来看javaCV思路就会很清晰了。

### 须知：

- javacv系列文章默认音视频处理使用ffmpeg，图像处理使用opencv,摄像头抓取使用opencv
- javacv官方github维护地址：[https://github.com/bytedeco/javacv](http://www.yanzuoguang.com/article/674#)

## 1、推荐使用最新的javacv1.5.3版本

(注意：从其他地方下载的依赖包请积极开心的替换为官方jar包和博主提供jar包；如果使用其他jar包版本而导致出错，不要急着找博主问为啥会报错，先把jar包替换了再试试看)

maven和gradle方式如果想要减小依赖包大小，则需要手动进行排除不需要的平台依赖即可

### （1）使用maven添加依赖

```xml
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.3</version>
</dependency>
```

### （2）使用gradle添加依赖

```gradle
dependencies {
	compile group: 'org.bytedeco', name: 'javacv-platform', version: '1.4.4'
}
```

### （3）使用本地jar包方式

最新版实在太大，需要下载全量包的请到官方<github.com/bytedeco/javacv>下载

以下都是javacv老版本，不再建议下载，建议使用maven或gradle方式构建项目。

- javaCV1.3.3版本下载（全量包，文件名:javacv-platform-1.3.3-bin.zip,大小：212M）：[http://download.csdn.net/download/eguid_1/10146035](http://www.yanzuoguang.com/article/674#)
- javaCV1.2所有jar包下载：[http://download.csdn.net/album/detail/3171](http://www.yanzuoguang.com/article/674#)

### jar包使用须知：

1. windows x64平台用到的opencv依赖：[opencv.jar](http://www.yanzuoguang.com/article/674#)；[oepncv-windows-x86_64.jar](http://www.yanzuoguang.com/article/674#)(其他平台替换为对应的jar包即可)
2. 苹果mac需要[opencv-macosx-x86_64.jar](http://www.yanzuoguang.com/article/674#)
3. linux平台需要：[opencv-linux-x86_64.jar](http://www.yanzuoguang.com/article/674#)
4. 安卓平台arm架构的需要[opencv-android-arm.jar](http://www.yanzuoguang.com/article/674#) ，基于x86的需要[opencv-android-x86.jar](http://www.yanzuoguang.com/article/674#)

## 2、为什么不需要安装opencv？

从javacv0.8开始，已经不需要本地安装opencv，直接通过引用opencv对应的系统平台的引用包即可。（比如[oepncv-windows-x86_64.jar](http://www.yanzuoguang.com/article/674#)就是典型的64位windows环境依赖包）

## 3、获取摄像头视频

最终调用的摄像头实时视频图像界面：![img](http://www.yanzuoguang.com/upload/2020/07/sj654k35b8ivfqbebu2njh87gd.png)预览本机摄像头视频图像的简单实现：

```java
package cc.eguid.javacv;

import javax.swing.JFrame;

import org.bytedeco.javacv.CanvasFrame;
import org.bytedeco.javacv.OpenCVFrameConverter;
import org.bytedeco.javacv.FrameGrabber.Exception;
import org.bytedeco.javacv.OpenCVFrameGrabber;

/**
 * 调用本地摄像头窗口视频
 * @author eguid  
 * @version 2016年6月13日  
 * @see javavcCameraTest  
 * @since  javacv1.2
    */

public class JavavcCameraTest{
    public static void main(String[] args) throws Exception, InterruptedException{

        //新建opencv抓取器，一般的电脑和移动端设备中摄像头默认序号是0，不排除其他情况
        OpenCVFrameGrabber grabber = new OpenCVFrameGrabber(0);  
        grabber.start();   //开始获取摄像头数据

        CanvasFrame canvas = new CanvasFrame("摄像头预览");//新建一个预览窗口
        canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        canvas.setAlwaysOnTop(true);

        while(true){
            //窗口是否关闭
            if(!canvas.isDisplayable()){
                grabber.close();//停止抓取
                return;
            } 
            // frame是一帧视频图像           
            Frame frame=grabber.grab(); 
            //获取摄像头图像并放到窗口上显示， 这里的Frame  
            canvas.showImage(frame);               
        }
    }
}
```

是不是很简单，原本很复杂的流媒体操作，javaCV能够帮助我们快速实现