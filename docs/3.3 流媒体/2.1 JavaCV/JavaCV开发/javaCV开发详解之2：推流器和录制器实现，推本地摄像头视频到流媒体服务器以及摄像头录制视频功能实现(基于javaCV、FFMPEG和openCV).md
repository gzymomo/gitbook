[javaCV开发详解之2：推流器和录制器实现，推本地摄像头视频到流媒体服务器以及摄像头录制视频功能实现(基于javaCV、FFMPEG和openCV)](http://www.yanzuoguang.com/article/712)

# javaCV开发详解之2：推流器和录制器实现，推本地摄像头视频到流媒体服务器以及摄像头录制视频功能实现(基于javaCV、FFMPEG和openCV)

## 前言：

上一章简单的介绍了javacv并且演示了如何获取本机摄像头：[http://blog.csdn.net/eguid_1/article/details/51659578](http://www.yanzuoguang.com/article/712#)．本章将在上一章的基础上，增加视频推流到流媒体服务器和视频录制的功能；

功能：实现边播放边录制/推流，停止预览即停止录制/推流

## 一、开发所依赖的包

- javacv.jar
- javacpp.jar
- ffmpeg.jar
- ffmpeg-系统平台.jar
- opencv.jar
- opencv-系统平台.jar

其中ffmpeg-系统平台.jar，opencv-系统平台.jar中的系统平台根据开发环境或者测试部署环境自行更改为对应的jar包，比如windows7 64位系统替换为ffmpeg-x86-x64.jar

为什么要这样做：因为ffmpeg-系统平台.jar中存放的是c/c++本地so/dll库，而ffmpeg.jar就是使用javacpp封装的对应本地库java接口的实现，而javacpp就是基于jni的一个功能性封装包，方便实现jni，javacv.jar就是对9个视觉库进行了二次封装，但是实现的功能有限，所以建议新手先熟悉openCV和ffmpeg这两个C/C++库的API后再来看javaCV思路就会很清晰了。

## 二、代码实现

本功能采用按帧录制/推流，通过关闭播放窗口停止视频录制/推流。

注：本章代码中的opencv转换器是未来方便演示如何获取图片，长时间运行该代码会导致内存溢出的原因是没有及时释放IplImage资源，所以大家推流时应当去除转换代码，直接推流即可。

```java
/**
* 按帧录制本机摄像头视频（边预览边录制，停止预览即停止录制）
* 
* @author eguid
* @param outputFile -录制的文件路径，也可以是rtsp或者rtmp等流媒体服务器发布地址
* @param frameRate - 视频帧率
* @throws Exception
* @throws InterruptedException
* @throws org.bytedeco.javacv.FrameRecorder.Exception
*/
public static void recordCamera(String outputFile, double frameRate){
    //另一种方式获取摄像头，opencv抓取器方式获取摄像头请参考第一章，FrameGrabber会自己去找可以打开的摄像头的抓取器。
    FrameGrabber grabber = FrameGrabber.createDefault(0);//本机摄像头默认0
    grabber.start();//开启抓取器

    OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();//转换器
    IplImage grabbedImage = converter.convert(grabber.grab());//抓取一帧视频并将其转换为图像，至于用这个图像用来做什么？加水印，人脸识别等等自行添加
    int width = grabbedImage.width();
    int height = grabbedImage.height();

    FrameRecorder recorder = FrameRecorder.createDefault(outputFile, width, height);
    recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264); // avcodec.AV_CODEC_ID_H264，编码
    recorder.setFormat("flv");//封装格式，如果是推送到rtmp就必须是flv封装格式
    recorder.setFrameRate(frameRate);

    recorder.start();//开启录制器
    long startTime=0;
    long videoTS=0;
    CanvasFrame frame = new CanvasFrame("camera", CanvasFrame.getDefaultGamma() / grabber.getGamma());
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setAlwaysOnTop(true);
    Frame rotatedFrame=converter.convert(grabbedImage);//不知道为什么这里不做转换就不能推到rtmp
    while (frame.isVisible() && (grabbedImage = converter.convert(grabber.grab())) != null) {

        rotatedFrame = converter.convert(grabbedImage);
        frame.showImage(rotatedFrame);
        if (startTime == 0) {
            startTime = System.currentTimeMillis();
        }
        videoTS = 1000 * (System.currentTimeMillis() - startTime);
        recorder.setTimestamp(videoTS);
        recorder.record(rotatedFrame);
        Thread.sleep(40);
    }
    frame.dispose();//关闭窗口
    recorder.close();//关闭推流录制器，close包含release和stop操作
    grabber.close();//关闭抓取器

}
```

总的来说，我们已经实现了基本的推流器功能，那么需要注意的就是转换那里，不清楚为什么不做转换就不能推送到rtmp流媒体服务器，如果哪位有更好的方案希望可以联系博主，感谢！

3、测试录制功能和推流功能

```java
public static void main(String[] args) throws Exception, InterruptedException, org.bytedeco.javacv.FrameRecorder.Exception {
    recordCamera("output.mp4",25);
}
public static void main(String[] args) throws Exception, InterruptedException, org.bytedeco.javacv.FrameRecorder.Exception {
    recordCamera("rtmp://192.168.30.21/live/record1",25);
}
```

看到了摄像头窗口就说明已经开始录制，点击右上角关闭按钮即停止录制视频，在录制的时候刷新项目目录发现新生成了一个output.mp4文件，可以正常播放这个视频文件

1. 针对内存泄露优化如下:

```java
public void recordCamera1(String outputFile, double frameRate) throws IOException, InterruptedException {
    //另一种方式获取摄像头，opencv抓取器方式获取摄像头请参考第一章，FrameGrabber会自己去找可以打开的摄像头的抓取器。
    FrameGrabber grabber = FrameGrabber.createDefault(0);//本机摄像头默认0
    grabber.start();//开启抓取器

    FrameRecorder recorder = FrameRecorder.createDefault(outputFile, grabber.getImageWidth(), grabber.getImageHeight());
    recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264); // avcodec.AV_CODEC_ID_H264，编码
    recorder.setFormat("flv");//封装格式，如果是推送到rtmp就必须是flv封装格式
    recorder.setFrameRate(frameRate);

    recorder.start();//开启录制器
    CanvasFrame convertFrame = new CanvasFrame("camera", CanvasFrame.getDefaultGamma() / grabber.getGamma());
    convertFrame.setDefaultCloseOperation(JFrame.HIDE_ON_CLOSE);
    convertFrame.setAlwaysOnTop(true);

    Frame frame;
    long startTime = System.currentTimeMillis(); // 视频开始时间
    long videoTS = 0;
    while (convertFrame.isVisible() && (frame = grabber.grab()) != null) {
        OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();//转换器
        IplImage grabbedImage = converter.convert(frame);//抓取一帧视频并将其转换为图像，至于用这个图像用来做什么？加水印，人脸识别等等自行添加
        Frame rotatedFrame = converter.convert(grabbedImage);
        convertFrame.showImage(rotatedFrame);
        videoTS = 1000 * (System.currentTimeMillis() - startTime);
        recorder.setTimestamp(videoTS);
        recorder.record(rotatedFrame);
        Thread.sleep((int) (1000 / frameRate));
    }
    // 用于记录视频的时间，方便去对比生成视频时间的差高难度
    long end = System.currentTimeMillis();
    System.out.println("视频时长:" + (end - startTime));

    convertFrame.dispose();//关闭窗口
    recorder.close();//关闭推流录制器，close包含release和stop操作
    grabber.close();//关闭抓取器
}
```