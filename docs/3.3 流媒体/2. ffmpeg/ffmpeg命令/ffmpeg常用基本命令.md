[ffmpeg常用基本命令](http://www.yanzuoguang.com/article/590)

# ffmpeg常用基本命令(转)

## 1.分离视频音频流

```shell
ffmpeg -i input_file -vcodec copy -an output_file_video　　//分离视频流
ffmpeg -i input_file -acodec copy -vn output_file_audio　　//分离音频流
```

## 2.视频解复用

```shell
ffmpeg –i test.mp4 –vcodec copy –an –f m4v test.264
ffmpeg –i test.avi –vcodec copy –an –f m4v test.264
```

## 3.视频转码

```shell
ffmpeg –i test.mp4 –vcodec h264 –s 352*278 –an –f m4v test.264              //转码为码流原始文件
ffmpeg –i test.mp4 –vcodec h264 –bf 0 –g 25 –s 352*278 –an –f m4v test.264  //转码为码流原始文件
ffmpeg –i test.avi -vcodec mpeg4 –vtag xvid –qsame test_xvid.avi            //转码为封装文件
//-bf B帧数目控制，-g 关键帧间隔控制，-s 分辨率控制
```

## 4.视频封装

```shell
ffmpeg –i video_file –i audio_file –vcodec copy –acodec copy output_file
```

## 5.视频剪切

```shell
ffmpeg –i test.avi –r 1 –f image2 image-%3d.jpeg        //提取图片
ffmpeg -ss 0:1:30 -t 0:0:20 -i input.avi -vcodec copy -acodec copy output.avi    //剪切视频
//-r 提取图像的频率，-ss 开始时间，-t 持续时间
```

## 6.视频录制

```shell
ffmpeg –i rtsp://192.168.3.205:5555/test –vcodec copy out.avi
```

## 7.YUV序列播放

```shell
ffplay -f rawvideo -video_size 1920x1080 input.yuv
```

## 8.YUV序列转AVI

```shell
ffmpeg –s w*h –pix_fmt yuv420p –i input.yuv –vcodec mpeg4 output.avi
```

## 常用参数说明：

- **主要参数：** -i 设定输入流 -f 设定输出格式 -ss 开始时间
- **视频参数：** -b 设定视频流量，默认为200Kbit/s -r 设定帧速率，默认为25 -s 设定画面的宽与高 -aspect 设定画面的比例 -vn 不处理视频 -vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器
- **音频参数：** -ar 设定采样率 -ac 设定声音的Channel数 -acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器 -an 不处理音频

## 0. 压缩转码mp4文件

```shell
ffmpeg -i input.avi -s 640x480 output.avi
ffmpeg -i input.avi -strict -2 -s vga output.avi
```

## 1、将文件当做直播送至live

```shell
ffmpeg -re -i localFile.mp4 -c copy -f flv rtmp://server/live/streamName
```

## 2、将直播媒体保存至本地文件

```shell
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
```

## 3、将其中一个直播流，视频改用h264压缩，音频不变，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv rtmp://server/live/h264Stream
```

## 4、将其中一个直播流，视频改用h264压缩，音频改用faac压缩，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a libfaac -ar 44100 -ab 48k -c:v libx264 -vpre slow -vpre baseline -f flv rtmp://server/live/h264Stream
```

## 5、将其中一个直播流，视频不变，音频改用faac压缩，送至另外一个直播服务流

```shell
ffmpeg -i rtmp://server/live/originalStream -acodec libfaac -ar 44100 -ab 48k -vcodec copy -f flv rtmp://server/live/h264_AAC_Stream
```

## 6、将一个高清流，复制为几个不同视频清晰度的流重新发布，其中音频不变

```shell
ffmpeg -re -i rtmp://server/live/high_FMLE_stream -acodec copy -vcodec x264lib -s 640×360 -b 500k -vpre medium -vpre baseline rtmp://server/live/baseline_500k -acodec copy -vcodec x264lib -s 480×272 -b 300k -vpre medium -vpre baseline rtmp://server/live/baseline_300k -acodec copy -vcodec x264lib -s 320×200 -b 150k -vpre medium -vpre baseline rtmp://server/live/baseline_150k -acodec libfaac -vn -ab 48k rtmp://server/live/audio_only_AAC_48k
```

## 7、功能一样，只是采用-x264opts选项

```shell
ffmpeg -re -i rtmp://server/live/high_FMLE_stream -c:a copy -c:v x264lib -s 640×360 -x264opts bitrate=500:profile=baseline:preset=slow rtmp://server/live/baseline_500k -c:a copy -c:v x264lib -s 480×272 -x264opts bitrate=300:profile=baseline:preset=slow rtmp://server/live/baseline_300k -c:a copy -c:v x264lib -s 320×200 -x264opts bitrate=150:profile=baseline:preset=slow rtmp://server/live/baseline_150k -c:a libfaac -vn -b:a 48k rtmp://server/live/audio_only_AAC_48k
```

## 8、将当前摄像头及音频通过DSSHOW采集，视频h264、音频faac压缩后发布

```shell
ffmpeg -r 25 -f dshow -s 640×480 -i video=”video source name”:audio=”audio source name” -vcodec libx264 -b 600k -vpre slow -acodec libfaac -ab 128k -f flv rtmp://server/application/stream_name
```

## 9、将一个JPG图片经过h264压缩循环输出为mp4视频

```shell
ffmpeg.exe -i INPUT.jpg -an -vcodec libx264 -coder 1 -flags +loop -cmp +chroma -subq 10 -qcomp 0.6 -qmin 10 -qmax 51 -qdiff 4 -flags2 +dct8x8 -trellis 2 -partitions +parti8x8+parti4x4 -crf 24 -threads 0 -r 25 -g 25 -y OUTPUT.mp4
```

## 10、将普通流视频改用h264压缩，音频不变，送至高清流服务(新版本FMS live=1)

```shell
ffmpeg -i rtmp://server/live/originalStream -c:a copy -c:v libx264 -vpre slow -f flv “rtmp://server/live/h264Stream live=1〃
```

# 摄像头

## 1.采集usb摄像头视频命令：

```shell
ffmpeg -t 20 -f vfwcap -i 0 -r 8 -f mp4 cap1111.mp4
./ffmpeg -t 10 -f vfwcap -i 0 -r 8 -f mp4 cap.mp4
```

具体说明如下：我们采集10秒，采集设备为vfwcap类型设备，第0个vfwcap采集设备（如果系统有多个vfw的视频采集设备，可以通过-i num来选择），每秒8帧，输出方式为文件，格式为mp4。

## 2.最简单的抓屏：

```shell
ffmpeg -f gdigrab -i desktop out.mpg 
```

## 3.从屏幕的（10,20）点处开始，抓取640x480的屏幕，设定帧率为5 ：

```shell
ffmpeg -f gdigrab -framerate 5 -offset_x 10 -offset_y 20 -video_size 640x480 -i desktop out.mpg 
```

## 4.ffmpeg从视频中生成gif图片：

```shell
ffmpeg -i capx.mp4 -t 10 -s 320x240 -pix_fmt rgb24 jidu1.gif
```

## 5.ffmpeg将图片转换为视频：

[http://blog.sina.com.cn/s/blog_40d73279010113c2.html](http://www.yanzuoguang.com/article/590#)