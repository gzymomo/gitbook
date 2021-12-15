# 基于javacv的视频转码

## 目标

将所有格式的视频转码为mp4格式

## 依赖

```xml
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv</artifactId>
    <version>1.5.3</version>
</dependency>
```

```java
import java.io.File;

import org.bytedeco.ffmpeg.global.avcodec;
import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.FFmpegFrameRecorder;
import org.bytedeco.javacv.Frame;

public class VideoUtils {
	
	public static String convert(File file) {
		FFmpegFrameGrabber frameGrabber = new FFmpegFrameGrabber(file.getAbsolutePath());
		String fileName = null;
	
		Frame captured_frame = null;
	
		FFmpegFrameRecorder recorder = null;
		
		try {
			frameGrabber.start();
			fileName = file.getAbsolutePath().replace(file.getAbsolutePath().substring(file.getAbsolutePath().lastIndexOf(".")), "_recode.mp4");
			recorder = new FFmpegFrameRecorder(fileName, frameGrabber.getImageWidth(), frameGrabber.getImageHeight(), frameGrabber.getAudioChannels());
			recorder.setVideoCodec(avcodec.AV_CODEC_ID_H264);
			recorder.setFormat("mp4");
			recorder.setFrameRate(frameGrabber.getFrameRate());
			recorder.setVideoBitrate(frameGrabber.getVideoBitrate());
			recorder.setAudioBitrate(192000);
			recorder.setAudioOptions(frameGrabber.getAudioOptions());
			recorder.setAudioQuality(0);
			recorder.setSampleRate(44100);
			recorder.setAudioCodec(avcodec.AV_CODEC_ID_AAC);
			recorder.start();
			while (true) {
				try {
					captured_frame = frameGrabber.grabFrame();
					
					if (captured_frame == null) {
						System.out.println("!!! Failed cvQueryFrame");
						break;
					}
					recorder.record(captured_frame);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
			recorder.stop();
			recorder.release();
			frameGrabber.stop();
			frameGrabber.release();
			recorder.close();
			frameGrabber.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
//		file.delete();
		return fileName;
	}

	public static void main(String[] args) throws java.lang.Exception {
		String filePath = "D:/1590473414299_20200526.3gp";
		System.out.println("开始。。。");
	    String convert = convert(new File(filePath));
	    System.out.println(convert);
	}

}
```

