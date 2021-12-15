# 错误：deprecated pixel format used, make sure you did set range correctly

视频流是正常推出去了，但是有警告信息：

```java
[swscaler @ 0x7fd84fd04c00] deprecated pixel format used, make sure you did set range correctly
[swscaler @ 0x7fd84fd04c00] deprecated pixel format used, make sure you did set range correctly
[swscaler @ 0x7fd84fd04c00] deprecated pixel format used, make sure you did set range correctly
[swscaler @ 0x7fd8500cfa00] deprecated pixel format used, make sure you did set range correctly
[swscaler @ 0x7fd84fd04c00] deprecated pixel format used, make sure you did set range correctly
```

可能导致的原因：

用的像素格式是flv不支持的，需要重新指定格式：

```java
recorder.setPixelFormat(avutil.AV_PIX_FMT_YUV420P);
```

经尝试，未果！



最后，无奈，选择过滤掉warn日志。

```java
//过滤掉warn 日志
avutil.av_log_set_level(avutil.AV_LOG_ERROR);
```

## JavaCV中FFmpegFrameGrabber调用start方法阻塞的解决办法

### 问题描述

> 目前出现阻塞的情况有如下两种：
> 1.拉历史流的时候，会发生阻塞，grabber.start()阻塞无法继续执行。
> 2.如果rtsp指令的ip乱输（或者无法建立连接），start()也会发生阻塞。

### 解决方法

------

#### 问题1

可以通过设置超时时间，如果拉不到流，触发超时时间，自动断开TCP连接。

```java
		// 设置采集器构造超时时间(单位微秒，1秒=1000000微秒)
		grabber.setOption("stimeout", "2000000");
12
```

上述方法貌似没多大作用，依然会被阻塞…

查看源码发现会发生阻塞的函数有两个：

##### 1.avformat_open_input():

打开流通道，探测一些视频格式等信息，对AVFormatContext结构体初始化。
对于这个函数阻塞的优化方法：将下面函数的seekCallback 设置为null，禁止javacv查找。

```java
		avio = avio_alloc_context(new BytePointer(av_malloc(4096)), 4096, 0, oc, readCallback, null,
					maximumSize > 0 ? seekCallback : null);
12
```

##### 2.avformat_find_stream_info()：

探测流信息（宽高码率等信息）
这个函数存在执行时间较长或者阻塞的问题，可以通过以下属性设置，来减小函数执行的时间。probesize属性限制探测时读取的最大数据量。max_analyze_duration属性限制info函数执行的时长，AV_TIME_BASE是单位秒。但是对于阻塞的问题好像并不能有效的解决，只是可以缩短函数的执行时间：

```java
		// 限制avformat_find_stream_info接口内部读取的最大数据量
		oc.probesize(5120);
		// 设置avformat_find_stream_info这个函数的持续时长，超过这个时间不结束也会结束
		oc.max_analyze_duration(5 * AV_TIME_BASE);
		// 将avformat_find_stream_info内部读取的数据包不放入AVFormatContext的缓冲区packet_buffer中
		oc.flags(AVFormatContext.AVFMT_FLAG_NOBUFFER);
123456
```

##### **3.使用inputstream进行推流时，最新版本的javacv(1.5.3),在grabber new的时候有一行注释：**

```java
/**
	 * Calls {@code FFmpegFrameGrabber(inputStream, Integer.MAX_VALUE - 8)} so that
	 * the whole input stream is seekable.
	 */
	public FFmpegFrameGrabber(InputStream inputStream) {
		this(inputStream, Integer.MAX_VALUE - 8);
	}

	/** Set maximumSize to 0 to disable seek and minimize startup time. */
	//将maximumSize设置为0以禁用查找并最小化启动时间
	public FFmpegFrameGrabber(InputStream inputStream, int maximumSize) {
		this.inputStream = inputStream;
		this.closeInputStream = true;
		this.pixelFormat = AV_PIX_FMT_NONE;
		this.sampleFormat = AV_SAMPLE_FMT_NONE;
		this.maximumSize = maximumSize;
	}
1234567891011121314151617
```

**将maximumSize设置为0以禁用查找并最小化启动时间;也就是grabber = new FFmpegFrameGrabber(inputStream,0);效果等同于上述序号1的设置，不修改源码来禁用avio_alloc_context()函数的seekCallback**

#### 问题2

上述设置超时时间的方法对于拉流地址(rtsp指令中的ip)输入错误时并不生效，依旧会阻塞，查看源码奈何才疏学浅不知道如何解决。变向通过检测是否能建立TCP连接，来判定是否可以正常推拉流。

如下代码：如果可以建立连接，则继续执行；否则释放资源，return null；

```java
		// 解决ip输入错误时，grabber.start();出现阻塞无法释放grabber而导致后续推流无法进行；
		Socket rtspSocket = new Socket();
		Socket rtmpSocket = new Socket();
		// 建立TCP Scoket连接，超时时间1s，如果成功继续执行，否则return
		try {
			rtspSocket.connect(new InetSocketAddress(cameraPojo.getIp(), 554), 1000);
		} catch (IOException e) {
			grabber.stop();
			grabber.close();
			rtspSocket.close();
			System.err.println("与拉流地址建立连接失败...");
			return null;
		}

		try {
			rtmpSocket.connect(new InetSocketAddress(IpUtil.IpConvert(config.getPush_ip()),
					Integer.parseInt(config.getPush_port())), 1000);
		} catch (IOException e) {
			grabber.stop();
			grabber.close();
			rtspSocket.close();
			System.err.println("与推流地址建立连接失败...");
			return null;
		}
```

## 其他解决方案

```java
avutil.av_log_set_level(avutil.AV_LOG_ERROR);
        FFmpegFrameGrabber grabber = FFmpegFrameGrabber.createDefault(streamSession.getUrl());
        // 使用tcp的方式，不然会丢包很严重
        grabber.setOption("rtsp_transport", "tcp");
        grabber.setImageWidth(640);
        grabber.setImageHeight(480);
        CountDownLatch latch = new CountDownLatch(1);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    grabber.start();
                    latch.countDown();
                } catch (FrameGrabber.Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
 
        try {
            latch.await(10, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
```

# 错误：av_write_frame() error -22 while writing video packet解决方法

## 问题分析

根据网上查阅的资料应该是：正常情况下，每一帧或者每一包过来的数据，dts和pts是累增的，也就是说，当前的帧或者pkt的dts和pts要比上一帧或者pkt的dts和pts要大的，当获取到的当前pkt的dts小于上一pkt的pts时，回报av_write_frame() error -22的错误。

## 解决方法

在推pkt之前加一个dts和pts的判断，如果是异常的pkt就丢掉，如果是正常的pkt则继续推。

```java
AVPacket pkt = null;
//用来记录dts和pts
long dts = 0;
long pts = 0;

for (int no_frame_index = 0; no_frame_index < 5 || err_index < 5;) {
    if (interrupt) {
        break;
    }
    pkt = grabber.grabPacket();
    if (pkt == null || pkt.size() <= 0 || pkt.data() == null) {
        // 空包记录次数跳过
        no_frame_index++;
        err_index++;
        continue;
    }
    // 获取到的pkt的dts，pts异常，将此包丢弃掉。
    if (pkt.dts() == avutil.AV_NOPTS_VALUE && pkt.pts() == avutil.AV_NOPTS_VALUE || pkt.pts() < dts) {
        logger.debug("异常pkt   当前pts: " + pkt.pts() + "  dts: " + pkt.dts() + "  上一包的pts： " + pts + " dts: "
                     + dts);
        err_index++;
        av_packet_unref(pkt);
        continue;
    }
    // 更新上一pkt的dts，pts
    dts = pkt.dts();
    pts = pkt.pts();
    // 推数据包
    err_index += (recorder.recordPacket(pkt) ? 0 : 1);
    // 将缓存空间的引用计数-1，并将Packet中的其他字段设为初始值。如果引用计数为0，自动的释放缓存空间。
    av_packet_unref(pkt);
}
```

# 错误：JavaCV异常：avformat_write_header error() error -40: Could not write header to ‘null‘

在使用javaCV推拉流时，出现如下报错：

```log
org.bytedeco.javacv.FrameRecorder$Exception: avformat_write_header error() error -40: Could not write header to 'null'
	at org.bytedeco.javacv.FFmpegFrameRecorder.startUnsafe(FFmpegFrameRecorder.java:952)
	at org.bytedeco.javacv.FFmpegFrameRecorder.start(FFmpegFrameRecorder.java:431)
	at org.bytedeco.javacv.FFmpegFrameRecorder.start(FFmpegFrameRecorder.java:426)
	at com.banmajio.push.RtmpPush.push(RtmpPush.java:138)
	at com.banmajio.push.RealPlay.play(RealPlay.java:59)
	at com.banmajio.controller.HcSDKController.openstream(HcSDKController.java:44)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:105)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:879)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:793)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1040)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:943)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:660)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:373)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1590)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657
```

## 解决思路

首先确保视频时h264编码的，javacv貌似不支持h265编码的视频数据。
检验方式：

### 在FFmpegFrameGrabber.start()之前设置FFmpeg日志级别

```java
avutil.av_log_set_level(avutil.AV_LOG_ERROR);
FFmpegLogCallback.set();
```

其中**AV_LOG_ERROR**可以根据需要设置DEBUG,INFO日志级别获取更详细的信息。

### 观察控制台打印信息确认视频编码格式

```log
Debug: [mpeg @ 0000000026017a40] After avformat_find_stream_info() pos: 2730320 bytes read:2731008 seeks:0 frames:2

Info: Input #0, mpeg, from 'java.io.BufferedInputStream@259bb828':

Info:   Duration: 
Info: N/A
Info: , start: 
Info: 7943.742044
Info: , bitrate: 
Info: 64 kb/s
Info: 

Info:     Stream #0:0
Info: [0x1e0]
Debug: , 1, 1/90000
Info: : Video: hevc (Main), 1 reference frame, yuvj420p(pc, bt709), 1920x1080, 0/1
Info: , 
Info: 90k tbr, 
Info: 90k tbn
Info: 

Info:     Stream #0:1
Info: [0x1c0]
Debug: , 1, 1/90000
Info: : Audio: pcm_alaw, 8000 Hz, mono, s16, 64 kb/s
Info: 
1234567891011121314151617181920212223242526
```

如上所示，Input为avformat_find_stream_info()函数探测出来的流信息。**Info: : Video: hevc (Main), 1 reference frame, yuvj420p(pc, bt709), 1920x1080, 0/1** hevc(Main)说明该视频流为h265编码。故而报错。

# JavaCV1.5.3版本FFmpegFrameGrabber初始化的时候加载时间长的解决方法

## 问题描述

> **最新推出的JavaCV1,5,3版本使用的时候发现FFmpegFrameGrabber，FFmpegFrameRecorder在new的时候会加载很长时间，差不多3s左右。**

## 问题分析

查看源码，发现是因为FFmpegFrameGrabber和FFmpegFrameRecorder类中有一个静态代码块，new的时候会去加载一些资源，所以会导致耗时。但是老版本也有这个操作却不会出现耗时的现象，具体原图不太清楚。FFmpegFrameGrabber类的初始化加载如下图：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200424091744899.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDc3NzUxMA==,size_16,color_FFFFFF,t_70#pic_center)

## 解决方法

在服务启动的时候手动执行该静态方法，使接口调用时已经加载过这些资源，从而解决new的时候耗时的问题。操作方法如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200424092039990.png)

## 解决方法

如果是本地视频文件，请更换h264编码的视频。
如果是海康设备，请在nvr中更改视频编码为h264

# Java examples

![推拉流](img\推拉流.jpg)



