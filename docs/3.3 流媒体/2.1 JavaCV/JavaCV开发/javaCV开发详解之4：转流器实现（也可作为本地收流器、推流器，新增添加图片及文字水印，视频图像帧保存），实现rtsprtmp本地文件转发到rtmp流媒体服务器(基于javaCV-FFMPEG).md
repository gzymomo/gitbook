[javaCV开发详解之4：转流器实现（也可作为本地收流器、推流器，新增添加图片及文字水印，视频图像帧保存），实现rtsp/rtmp/本地文件转发到rtmp流媒体服务器(基于javaCV-FFMPEG)](http://www.yanzuoguang.com/article/717)



# javaCV开发详解之4：转流器实现（也可作为本地收流器、推流器，新增添加图片及文字水印，视频图像帧保存），实现rtsp/rtmp/本地文件转发到rtmp流媒体服务器(基于javaCV-FFMPEG)

## 前言

上一章中实现了本地推流器和本地摄像头录制功能：[http://blog.csdn.net/eguid_1/article/details/52678775](http://www.yanzuoguang.com/article/717#) 本章基于javaCV实现转流器和收流器功能，测试采用监控rtsp地址转发至rtmp服务器地址。

更好的转流实现参照这章：[javaCV开发详解之8：转封装在rtsp转rtmp流中的应用（无须转码，更低的资源消耗，更好的性能，更低延迟）](http://www.yanzuoguang.com/article/717#)

1. 新增openCV保存图片功能。
2. 补充：

- 作为转流器可以轻松实现rtsp/rtmp/本地文件/本地摄像头推送到rtmp流媒体服务器；
- 作为收流器可以用来把流媒体服务器视频流录制到本地文件。
- 关于默认接收/推送rtsp流丢帧问题，由于ffmpeg默认采用udp方式，所以可以通过更改为tcp的方式来实现丢帧补偿，解决方式如下：
  1. FFmpeg命令方式：增加一个配置命令 -rtsp_transport tcp
  2. javacv方式：FFmpegFrameGrabber.java中533行 AVDictionary options= new AVDictionary(null);后面增加一个配置av_dict_set(options, "rtsp_transport", "tcp", 0); 即可
  3. ffmpeg原生方式：同上

## 一、所依赖的包

具体依赖包请查看javacv开发详解之1。本章使用windows环境开发，基于javaCV的基础支撑包以及ffmpeg-3.1.2-1.2.jar、ffmpeg-3.1.2-1.2-windows-x86.jar、ffmpeg-3.1.2-1.2-windows-x86_64.jar；

补充：

1. 如果想要给视频添加水印，需要从视频中取出图像帧，给图像帧添加文字、图片水印即可
2. 在此之前我们需要取到BufferedImage，通过这个我们就可以用java的方式添加水印
3. 如何用java添加水印：[http://blog.csdn.net/eguid_1/article/details/52973508](http://www.yanzuoguang.com/article/717#)
4. 如何从grabber中获取BufferedImage：

```java
//获取BufferedImage可以给图像帧添加水印
Java2DFrameConverter javaconverter=new Java2DFrameConverter();
BufferedImage buffImg=javaconverter.convert(grabber.grab());
```

获取到了BufferedImage我们就可以给视频帧添加文字或者图片水印了

## 二、代码实现

本功能实现按帧取流和转发服务

```java
/**
	 * 转流器
	 * @param inputFile
	 * @param outputFile
	 * @throws Exception
	 * @throws org.bytedeco.javacv.FrameRecorder.Exception
	 * @throws InterruptedException
	 */
public static void recordPush(String inputFile,String outputFile,int v_rs) throws Exception,org.bytedeco.javacv.FrameRecorder.Exception,InterruptedException{
    FrameGrabber grabber =new FFmpegFrameGrabber(inputFile);
    try {
        grabber.start();
    } catch (Exception e) {
        throw e;
    }
    //一个opencv视频帧转换器
    OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();
    Frame grabframe =grabber.grab();
    IplImage grabbedImage =null;
    if(grabframe!=null){
        System.out.println("取到第一帧");
        grabbedImage = converter.convert(grabframe);
    }else{
        System.out.println("没有取到第一帧");
    }
    // 如果想要保存图片,可以用如下函数来保存图片
    // opencv_imgcodecs.cvSaveImage("hello.jpg", grabbedImage);

    FrameRecorder recorder;
    try {
        recorder = FrameRecorder.createDefault(outputFile, 1280, 720);
    } catch (org.bytedeco.javacv.FrameRecorder.Exception e) {
        throw e;
    }

    recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264); // avcodec.AV_CODEC_ID_H264
    recorder.setFormat("flv");
    recorder.setFrameRate(v_rs);
    recorder.setGopSize(v_rs);

    try {
        recorder.start();
    } catch (org.bytedeco.javacv.FrameRecorder.Exception e) {
        System.out.println("录制器启动失败");
        throw e;
    }

    CanvasFrame frame = new CanvasFrame("camera", CanvasFrame.getDefaultGamma() / grabber.getGamma());
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setAlwaysOnTop(true);

    while (frame.isVisible() && (grabframe=grabber.grab()) != null) {
        EnumSet<Type> videoOrAudio=frame.getTypes();
        if(videoOrAudio.contains(Type.VIDEO)) {
            frame.showImage(grabframe);
        }
        if(rotatedFrame!=null){
            recorder.record(rotatedFrame);
        }
    }
    recorder.close();
    grabber.close();
}
```

## 三、测试转流服务

```java
public static void main(String[] args) throws FrameRecorder.Exception, FrameGrabber.Exception,InterruptedException{
    String inputFile = "rtsp://admin:admin@192.168.2.236:37779/cam/realmonitor?channel=1&subtype=0";
    String outputFile="rtmp://192.168.30.21/live/pushFlow";
    recordPush(inputFile, outputFile,25);
}
```

## 四、优化后的代码

```java
/**
     * 转流器
     *
     * @param inputFile
     * @param outputFile
     * @param convert    是否转码
     * @throws Exception
     * @throws org.bytedeco.javacv.FrameRecorder.Exception
     * @throws InterruptedException
     */
public static void recordPush(String inputFile, String outputFile, boolean convert) throws IOException {
    FrameGrabber grabber = new FFmpegFrameGrabber(inputFile);
    grabber.start();


    FFmpegFrameRecorder recorder = new FFmpegFrameRecorder(outputFile, grabber.getImageWidth(), grabber.getImageHeight(), grabber.getAudioChannels());
    recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264); // avcodec.AV_CODEC_ID_H264
    recorder.setFormat("flv");
    recorder.setFrameRate(grabber.getFrameRate());
    recorder.setGopSize((int) grabber.getFrameRate());
    recorder.start();

    CanvasFrame frame = new CanvasFrame("camera", CanvasFrame.getDefaultGamma() / grabber.getGamma());
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setAlwaysOnTop(true);

    Frame grabFrame;
    Frame firstFrame = null;
    while (frame.isVisible() && (grabFrame = grabber.grab()) != null) {
        EnumSet<Frame.Type> videoOrAudio = grabFrame.getTypes();
        if (videoOrAudio.contains(Frame.Type.VIDEO)) {
            if (convert) {
                //一个opencv视频帧转换器
                OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();//转换器
                IplImage grabbedImage = converter.convert(grabFrame);//抓取一帧视频并将其转换为图像，至于用这个图像用来做什么？加水印，人脸识别等等自行添加
                Frame rotatedFrame = converter.convert(grabbedImage);
                grabFrame = rotatedFrame;

                if (firstFrame == null) {
                    firstFrame = grabFrame;
                    System.out.println("取到第一帧");
                    // 如果想要保存图片,可以用如下函数来保存图片
                    // opencv_imgcodecs.cvSaveImage("hello.jpg", grabbedImage);
                }
            }

            frame.showImage(grabFrame);
        }
        recorder.record(grabFrame);
    }
    recorder.close();
    grabber.close();
}
```