[JavaCV本地视频流通过帧图片添加文本进行字幕合成](http://www.yanzuoguang.com/article/689)

# JavaCV本地视频流通过帧图片添加文本进行字幕合成

音视频的Java框架找了一大圈，除了JavaCV，目前找不到其他的。JavaCV封装了对底层C的调用，最终实际上执行的都是FFMPEG的函数。现在有个头疼的问题，FFMPEG的字幕合成用命令行一敲就完事了，比如想往input.mkv里合入字幕subtitles.srt，输出到output.mkv，简单的很：

```
ffmpeg -i input.mkv -vf subtitles=subtitles.srt output.mkv
```

上面的-vf参数表示内嵌字幕，也就是不用手动在播放器里设置，字幕就自己出来了。JavaCV怎么实现字幕合成呢？找了一大圈，没有。OpenCV倒是可以实现，就是比较麻烦，大概思路是：音视频帧 -> 视频帧 -> 视频帧转图片 -> 字幕文本画到图片指定位置上 -> 图片转回视频帧 -> 写视频帧 -> 写音频帧。说白了就是文本水印。可惜OpenCV不支持中文，画出来都是问号。

OpenCV就是一个画图工具而已，换，用Java自带的awt画图工具对象Graphics2D。不废话，上代码：

## 1、pom.xml添加Java CV依赖：

```xml
<dependency>
    <groupId>org.bytedeco</groupId>
    <artifactId>javacv-platform</artifactId>
    <version>1.5.2</version>
</dependency>
```

## 2、示例代码

这里用几句话模拟字幕循环输出，并展示合入字幕的时间：

```java
import org.bytedeco.ffmpeg.global.avcodec;
import org.bytedeco.javacv.*;
import org.bytedeco.javacv.Frame;
import sun.font.FontDesignMetrics;

import java.awt.*;
import java.awt.image.BufferedImage;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class SubtitleMix {

    private final static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH🇲🇲ss");

    public static void main(String[] args) throws FrameGrabber.Exception, FrameRecorder.Exception {
        // 构造测试字幕
        String[] test = {
                "世上无难事",
                "只怕有心人",
                "只要思想不滑坡",
                "办法总比困难多",
                "长江后浪推前浪",
                "前浪死在沙滩上"
        };

        // 为连续的50帧设置同一个测试字幕文本
        List<String> testStr = new ArrayList<>();
        for (int i = 0; i < 300; i++) {
            testStr.add(test[i / 50]);
        }

        // 设置源视频、加字幕后的视频文件路径
        FFmpegFrameGrabber grabber = FFmpegFrameGrabber.createDefault("E:\\BaiduNetdiskDownload\\testout.mkv");
        grabber.start();
        FFmpegFrameRecorder recorder = new FFmpegFrameRecorder("E:\\BaiduNetdiskDownload\\outtest.mkv",
                1280, 720, 2);

        // 视频相关配置，取原视频配置
        recorder.setFrameRate(grabber.getFrameRate());
        recorder.setVideoCodec(grabber.getVideoCodec());
        recorder.setVideoBitrate(grabber.getVideoBitrate());

        // 音频相关配置，取原音频配置
        recorder.setSampleRate(grabber.getSampleRate());
        recorder.setAudioCodec(avcodec.AV_CODEC_ID_MP3);

        recorder.start();
        System.out.println("准备开始推流...");
        Java2DFrameConverter converter = new Java2DFrameConverter();
        Frame frame;
        int i = 0;
        while ((frame = grabber.grab()) != null) {
            // 从视频帧中获取图片
            if (frame.image != null) {

                BufferedImage bufferedImage = converter.getBufferedImage(frame);

                // 对图片进行文本合入
                bufferedImage = addSubtitle(bufferedImage, testStr.get(i++ % 300));

                // 视频帧赋值，写入输出流
                frame.image = converter.getFrame(bufferedImage).image;
                recorder.record(frame);
            }

            // 音频帧写入输出流
            if(frame.samples != null) {
                recorder.record(frame);
            }
        }
        System.out.println("推流结束...");
        grabber.stop();
        recorder.stop();
    }

    /**
     * 图片添加文本
     *
     * @param bufImg
     * @param subTitleContent
     * @return
     */
    private static BufferedImage addSubtitle(BufferedImage bufImg, String subTitleContent) {

        // 添加字幕时的时间
        Font font = new Font("微软雅黑", Font.BOLD, 32);
        String timeContent = sdf.format(new Date());
        FontDesignMetrics metrics = FontDesignMetrics.getMetrics(font);
        Graphics2D graphics = bufImg.createGraphics();
        graphics.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
        graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER));

        //设置图片背景
        graphics.drawImage(bufImg, 0, 0, bufImg.getWidth(), bufImg.getHeight(), null);

        //设置左上方时间显示
        graphics.setColor(Color.orange);
        graphics.setFont(font);
        graphics.drawString(timeContent, 0, metrics.getAscent());

        // 计算文字长度，计算居中的x点坐标
        int textWidth = metrics.stringWidth(subTitleContent);
        int widthX = (bufImg.getWidth() - textWidth) / 2;
        graphics.setColor(Color.red);
        graphics.setFont(font);
        graphics.drawString(subTitleContent, widthX, bufImg.getHeight() - 100);
        graphics.dispose();
        return bufImg;
    }
}
```

## 打完收工，看看效果：

![1](http://www.yanzuoguang.com/upload/2020/07/1frj51ktdgi1rrqlq5in2sd33n.png)