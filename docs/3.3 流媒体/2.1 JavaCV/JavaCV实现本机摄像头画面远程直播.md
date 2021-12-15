[悄摸直播 —— JavaCV实现本机摄像头画面远程直播](https://www.cnblogs.com/scywkl/p/12101437.html)



# 目录

- 前言
- 需要的jar包和依赖
- 需要实现的模块（附带源码教程）
- 项目效果展示

# 前言

最近想用Java实现一个类似于远程直播的功能

像这样：（功能示意图）![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224142117598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)

# 需要的jar包和依赖

Maven依赖:

```xml
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
```

# 需要实现的模块（附带源码教程）：

**[推流器 —— 视频获取，转流推流](https://www.cnblogs.com/scywkl/p/12092428.html)**
**[播流器 —— 播流，展示](https://www.cnblogs.com/scywkl/p/12092457.html)**
**[服务器 —— 搭建](https://www.cnblogs.com/scywkl/p/12092475.html)**



# 一、[ 推流器的实现（获取笔记本摄像头画面，转流推流到rtmp服务器）](https://www.cnblogs.com/scywkl/p/12092428.html)



## 推流器

### 一、功能说明

获取pc端的摄像头流数据 + 展示直播效果 + 推流到rtmp服务器

### 二、代码实现

```java
/**
	 * 推流器
	 * @param devicePath   摄像头的地址。可以是摄像头rtsp地址，也可以是设备号码，本机摄像头是0
	 * @param outputPath   接收路径
	 * @param v_rs         帧率
	 * @throws Exception
	 * @throws org.bytedeco.javacv.FrameRecorder.Exception
	 * @throws InterruptedException
	 */
	public static void recordPush(String outputPath,int v_rs) throws Exception, org.bytedeco.javacv.FrameRecorder.Exception, InterruptedException {
		
		Loader.load(opencv_objdetect.class);
		
		//创建采集器
		OpenCVFrameGrabber grabber = new OpenCVFrameGrabber(0);  //本地摄像头默认为0
		
		//开启采集器
		try {
			grabber.start();  
		} catch (Exception e) {
			try {
				grabber.restart();  //一次重启尝试
			} catch (Exception e2) {
				throw e;
			}
		}
		
		OpenCVFrameConverter.ToIplImage converter = new OpenCVFrameConverter.ToIplImage();  //转换器  
		Frame grabframe = grabber.grab();  //获取一帧
		IplImage grabbedImage = null;
		if (grabframe!=null) {
			grabbedImage = converter.convert(grabframe); //将这一帧转换为IplImage
		}
		
		//创建录制器
		FrameRecorder recorder;
		recorder = FrameRecorder.createDefault(outputPath, 1280, 720);   //输出路径，画面高，画面宽
		recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264);  //设置编码格式
		recorder.setFormat("flv");   
		recorder.setFrameRate(v_rs);
		recorder.setGopSize(v_rs);
		
		//开启录制器
		try {
			recorder.start();
		} catch (java.lang.Exception e) {
			System.out.println("recorder开启失败");
			System.out.println(recorder);
			try {
				if (recorder != null) {  //尝试重启录制器
					recorder.stop();
					recorder.start();
				}
			} catch (java.lang.Exception e1) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
		//直播效果展示窗口
		CanvasFrame frame = new CanvasFrame("直播效果",CanvasFrame.getDefaultGamma() / grabber.getGamma());
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		frame.setAlwaysOnTop(true);
		
		//推流
		while(frame.isVisible() && (grabframe=grabber.grab()) != null) {
			frame.showImage(grabframe);   //展示直播效果
			grabbedImage = converter.convert(grabframe);
			Frame rotatedFrame = converter.convert(grabbedImage);
			
			if (rotatedFrame != null) {
				recorder.record(rotatedFrame);  
			}
			
			Thread.sleep(50);  //50毫秒/帧
		}
	}
```

### 三、测试推流器

```java
public static void main(String[] args) throws Exception, org.bytedeco.javacv.FrameRecorder.Exception, InterruptedException {
		//设置rtmp服务器地址
		String outputPath = "rtmp://192.168.1.48:1935/live/stream";
		
		recordPush(outputPath, 25);
	}
```

如果出现“直播效果”的swing窗口，并能够播放摄像头画面，则说明推流器成功。



# 二、[播流器实现（拉取rtmp服务器中的数据流，播放直播画面）](https://www.cnblogs.com/scywkl/p/12092457.html)



## 播流器

#### 一、功能说明

从rtmp服务器中获取视频流数据 + 展示直播画面

#### 二、代码实现

```java
/**
	 * 播流器
	 * @param inputPath  rtmp服务器地址
	 * @throws Exception
	 * @throws org.bytedeco.javacv.FrameRecorder.Exception 
	 */
	public static void pullStream(String inputPath) throws Exception, org.bytedeco.javacv.FrameRecorder.Exception {
		//创建+设置采集器
		FFmpegFrameGrabber grabber = FFmpegFrameGrabber.createDefault(inputPath);
        grabber.setOption("rtsp_transport", "tcp"); 
        grabber.setImageWidth(960);
        grabber.setImageHeight(540);
        
        //开启采集器
        grabber.start();
        
        //直播播放窗口
        CanvasFrame canvasFrame = new CanvasFrame("悄摸直播——来自"+inputPath);
        canvasFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        canvasFrame.setAlwaysOnTop(true);
        OpenCVFrameConverter.ToMat converter = new OpenCVFrameConverter.ToMat();
        
        //播流
        while (true){
            Frame frame = grabber.grabImage();  //拉流
            opencv_core.Mat mat = converter.convertToMat(frame);
            canvasFrame.showImage(frame);   //播放
        }
	}
```

#### 三、测试播流器

```java
public static void main(String[] args) throws Exception, org.bytedeco.javacv.FrameRecorder.Exception {
		//rtmp服务器地址
		String inputPath = "rtmp://192.168.1.48/live/stream";
		pullStream(inputPath);
	}
```

如果出现“悄摸直播——来自XXX”的swing窗口，并能正常播放直播画面，则播流器成功。

# 三、[搭建rtmp服务器（smart_rtmpd - rtmp服务器搭建）](https://www.cnblogs.com/scywkl/p/12092475.html)

## 搭建rtmp服务器

#### 一、素材

rtmp服务器：[smart_rtmpd](http://www.qiyicc.com/download/rtmpd.rar)
ffmpeg工具：[ffmpeg.exe](https://ffmpeg.zeranoe.com/builds/)

#### 二、搭建

###### 1.下载smart_rtmpd软件包，解压rtmpd.rar，进一步解压smart_rtmpd_win.rar得到smart_rtmpd.exe文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224154252793.png)

###### 2.运行smart_rtmpd.exe文件，直接启动。（如果对rtmp服务器有特殊配置需求，可以在smart_rtmpd.exe同级目录下的config.xml文件中修改）

**启动成功示例：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224154638870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)
smart_rtmpd会在启动时获取本机所有网络的ip地址，自动生成rtmp服务器地址。
具体样式：rtmp://192.168.1.48/live/stream
*192.168.1.48就是本机的网络IP地址*

如果想更改rtmp服务器的地址，可以在config.xml文件中修改：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224155300519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)

#### 三、推流&播流测试

###### 1.安装ffmpeg工具

*安装包请看“素材”*

###### 2.推流测试

cmd打开小黑窗，输入命令：

> ffmpeg.exe -re -stream_loop -1 -i F:\55427366\48\55427366_48_0.flv -c copy -f flv rtmp://192.168.1.48/live/stream

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224160433915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)
推流成功：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224160554258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)

###### 3.播流测试

找到一个播放器，打开rtmp服务器地址：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224160744734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)
播流成功：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191224160916251.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDIxNzYxMg==,size_16,color_FFFFFF,t_70)