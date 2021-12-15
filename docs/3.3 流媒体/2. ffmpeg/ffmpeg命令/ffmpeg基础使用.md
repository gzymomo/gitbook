- [ffmpeg基础使用](https://www.jianshu.com/p/ddafe46827b7)
-  [ffmpeg常用命令](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fnewchenxf%2Farticle%2Fdetails%2F51384360)
-  [ffmpeg参数中文详细解释](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fleixiaohua1020%2Farticle%2Fdetails%2F12751349)
-  [[总结\]FFMPEG视音频编解码零基础学习方法](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fleixiaohua1020%2Farticle%2Fdetails%2F15811977)



# 常用命令

主要参数：

```
-i 设定输入流 
-f 设定输出格式 
-ss 开始时间 
```

视频参数：

```
-b 设定视频流量(码率)，默认为200Kbit/s 
-r 设定帧速率，默认为25 
-s 设定画面的宽与高 
-aspect 设定画面的比例 
-vn 不处理视频 
-vcodec 设定视频编解码器，未设定时则使用与输入流相同的编解码器 
```

音频参数：

```
-ar 设定采样率 
-ac 设定声音的Channel数 
-acodec 设定声音编解码器，未设定时则使用与输入流相同的编解码器 
-an 不处理音频
```

## 1. 视频格式转换

**（其实格式转换说法不太准确，但大家都这么叫，准确的说，应该是视频容器转换）**
 比如一个avi文件，想转为mp4，或者一个mp4想转为ts。
 `ffmpeg -i input.avi output.mp4`
 `ffmpeg -i input.mp4 output.ts`
 插个号外：某天我在知乎上看到一段视频，想转给微信好友看，怎么操作呢。这里参考[如何全自动下载知乎上的视频到本地(注意不要滥用)](https://links.jianshu.com/go?to=https%3A%2F%2Fbaijiahao.baidu.com%2Fs%3Fid%3D1595252673329058399%26wfr%3Dspider%26for%3Dpc)，先打开要观看的视频页面，再F12清空，然后开始播放视频，就能看到类似`https://vdn.vzuu.com/SD/49c84c7c-c61a-11e8-8bad-0242ac112a0a.mp4?auth_key=1539832492-0-0-c61c22f39c&expiration=1539832492&disable_local_cache=1`这样的字符串，然后用`ffmpeg -i "https://vdn.vzuu.com/SD/49c8..." output.mp4`即可下载。弄到电脑上，用电脑QQ发送到手机QQ上，在手机QQ上点击选择保存到手机上。然后在微信里选照片就能看到这个视频了(注意视频文件不要超过20M，另外最开始用的不是电脑QQ，而是百度网盘，发现不行……)。

## 2. 提取音频

比如我有一个“晓松奇谈”，可是我不想看到他的脸，我只想听声音， 地铁上可以听，咋办？
 `ffmpeg -i 晓松奇谈.mp4 -acodec copy -vn output.aac`
 上面的命令，默认mp4的audio codec是aac，如果不是会出错，咱可以暴力一点，不管什么音频，都转为最常见的aac。
 `ffmpeg -i 晓松奇谈.mp4 -acodec aac -vn output.aac`

> (-vn 不处理视频 )

## 3. 提取视频

我目测有些IT员工，特别是做嵌入式的，比如机顶盒，想debug一下，没有音频的情况下，播放一个视频几天几夜会不会crash，这时候你需要一个纯视频文件，可以这么干。
 `ffmpeg -i input.mp4 -vcodec copy -an output.mp4`

> -an 不处理音频

## 4. 视频剪切

经常要测试视频，但是只需要测几秒钟，可是视频却有几个G，咋办？切啊！
 下面的命令，就可以从时间为00:00:15开始，截取5秒钟的视频。
 `ffmpeg -ss 00:00:15 -t 00:00:05 -i input.mp4 -vcodec copy -acodec copy output.mp4`
 -ss表示开始切割的时间，-t表示要切多少。上面就是从开始，切5秒钟出来。

摘自[使用 MediaSource 搭建流式播放器](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F26374202)

> 注意一个问题，ffmpeg  在切割视频的时候无法做到时间绝对准确，因为视频编码中关键帧（I帧）和跟随它的B帧、P帧是无法分割开的，否则就需要进行重新帧内编码，会让视频体积增大。所以，如果切割的位置刚好在两个关键帧中间，那么 ffmpeg 会向前/向后切割，所以最后切割出的 chunk 长度总是会大于等于应有的长度。

## 5. 码率控制

码率控制对于在线视频比较重要。因为在线视频需要考虑其能提供的带宽。

那么，什么是码率？很简单： `bitrate = file size / duration`

比如一个文件20.8M，时长1分钟，那么，码率就是：
 `biterate = 20.8M bit/60s = 20.8*1024*1024*8 bit/60s= 2831Kbps`
 一般音频的码率只有固定几种，比如是128Kbps， 那么，video的就是
 `video biterate = 2831Kbps -128Kbps = 2703Kbps。`

说完背景了。好了，来说ffmpeg如何控制码率。 ffmpg控制码率有3种选择，`-minrate -b:v -maxrate`

- -b:v主要是控制平均码率。 比如一个视频源的码率太高了，有10Mbps，文件太大，想把文件弄小一点，但是又不破坏分辨率。 `ffmpeg -i input.mp4 -b:v 2000k output.mp4`上面把码率从原码率转成2Mbps码率，这样其实也间接让文件变小了。目测接近一半。
- 不过，ffmpeg官方wiki比较建议，设置b:v时，同时加上 -bufsize
   -bufsize 用于设置码率控制缓冲器的大小，设置的好处是，让整体的码率更趋近于希望的值，减少波动。（简单来说，比如1 2的平均值是1.5， 1.49 1.51 也是1.5, 当然是第二种比较好） `ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k output.mp4`
- -minrate -maxrate就简单了，在线视频有时候，希望码率波动，不要超过一个阈值，可以设置maxrate。
   `ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k -maxrate 2500k output.mp4`

## 6. 视频编码格式转换

比如一个视频的编码是MPEG4，想用H264编码，咋办？
 `ffmpeg -i input.mp4 -vcodec h264 output.mp4`
 相反也一样
 `ffmpeg -i input.mp4 -vcodec mpeg4 output.mp4`

当然了，如果ffmpeg当时编译时，添加了外部的x265或者X264，那也可以用外部的编码器来编码。（不知道什么是X265，可以Google一下，简单的说，就是她不包含在ffmpeg的源码里，是独立的一个开源代码，用于编码HEVC，ffmpeg编码时可以调用它。当然了，ffmpeg自己也有编码器）
 `ffmpeg -i input.mp4 -c:v libx265 output.mp4`
 `ffmpeg -i input.mp4 -c:v libx264 output.mp4`

## 7. 只提取视频ES数据

这个可能做开发的人会用到，顺便提一下吧。
 `ffmpeg –i input.mp4 –vcodec copy –an –f m4v output.h264`

## 8. 过滤器的使用

这个我在另一篇博客提到了，具体参考[ffmpeg filter过滤器 基础实例及全面解析](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fnewchenxf%2Farticle%2Fdetails%2F51364105)

### 8.1 将输入的1920x1080缩小到960x540输出:

`ffmpeg -i input.mp4 -vf scale=960:540 output.mp4`
 //ps: 如果540不写，写成-1，即scale=960:-1, 那也是可以的，ffmpeg会通知缩放滤镜在输出时保持原始的宽高比。

### 8.2 为视频添加logo

比如，我有这么一个图片

![img]()

image.png


 想要贴到一个视频上，那可以用如下命令：
`./ffmpeg -i input.mp4 -i iQIYI_logo.png -filter_complex overlay output.mp4`
 结果如下所示：

![img]()

image.png



## 9. 抓取视频的一些帧，存为jpeg图片

比如，一个视频，我想提取一些帧，存为图片，咋办？
 `ffmpeg -i input.mp4 -r 1 -q:v 2 -f image2 pic-%03d.jpeg`

> -r 表示每一秒几帧
>  -q:v表示存储jpeg的图像质量，一般2是高质量。

如此，ffmpeg会把input.mp4，每隔一秒，存一张图片下来。假设有60s，那会有60张。60张？什么？这么多？不要不要。。。。。不要咋办？？ 可以设置开始的时间，和你想要截取的时间呀。
 `ffmpeg -i input.mp4 -ss 00:00:20 -t 10 -r 1 -q:v 2 -f image2 pic-%03d.jpeg`

> -ss 表示开始时间
>  -t表示共要多少时间。

如此，ffmpeg会从input.mp4的第20s时间开始，往下10s，即20~30s这10秒钟之间，每隔1s就抓一帧，总共会抓10帧。

## 10.输出YUV420原始数据

对于一下做底层编解码的人来说，有时候常要提取视频的YUV原始数据。 怎么坐？很简答： `ffmpeg -i input.mp4 output.yuv`怎么样，是不是太简单啦？！！！哈哈(如果你想问yuv的数据，如何播放，我不会告诉你，RawPlayer挺好用的！！)

那如果我只想要抽取某一帧YUV呢？ 简单，你先用上面的方法，先抽出jpeg图片，然后把jpeg转为YUV。 比如： 你先抽取10帧图片。 `ffmpeg -i input.mp4 -ss 00:00:20 -t 10 -r 1 -q:v 2 -f image2 pic-%03d.jpeg`
 结果：

```
-rw-rw-r-- 1 chenxf chenxf    296254  7月 20 16:08 pic-001.jpeg
-rw-rw-r-- 1 chenxf chenxf    300975  7月 20 16:08 pic-002.jpeg
-rw-rw-r-- 1 chenxf chenxf    310130  7月 20 16:08 pic-003.jpeg
-rw-rw-r-- 1 chenxf chenxf    268694  7月 20 16:08 pic-004.jpeg
-rw-rw-r-- 1 chenxf chenxf    301056  7月 20 16:08 pic-005.jpeg
-rw-rw-r-- 1 chenxf chenxf    293927  7月 20 16:08 pic-006.jpeg
-rw-rw-r-- 1 chenxf chenxf    340295  7月 20 16:08 pic-007.jpeg
-rw-rw-r-- 1 chenxf chenxf    430787  7月 20 16:08 pic-008.jpeg
-rw-rw-r-- 1 chenxf chenxf    404552  7月 20 16:08 pic-009.jpeg
-rw-rw-r-- 1 chenxf chenxf    412691  7月 20 16:08 pic-010.jpeg
```

然后，你就随便挑一张，转为YUV: `ffmpeg -i pic-001.jpeg -s 1440x1440 -pix_fmt yuv420p xxx3.yuv`如果-s参数不写，则输出大小与输入一样。当然了，YUV还有yuv422p啥的，你在-pix_fmt 换成yuv422p就行啦！

## 11.H264编码profile & level控制

举3个例子吧

```
ffmpeg -i input.mp4 -profile:v baseline -level 3.0 output.mp4
ffmpeg -i input.mp4 -profile:v main -level 4.2 output.mp4
ffmpeg -i input.mp4 -profile:v high -level 5.1 output.mp4
```

如果ffmpeg编译时加了external的libx264，那就这么写：
 `ffmpeg -i input.mp4 -c:v libx264 -x264-params "profile=high:level=3.0" output.mp4`
 从压缩比例来说，baseline< main <  high，对于带宽比较局限的在线视频，可能会选择high，但有些时候，做个小视频，希望所有的设备基本都能解码（有些低端设备或早期的设备只能解码baseline），那就牺牲文件大小吧，用baseline。自己取舍吧！

## 12.旋转视频

在手机上录的视频，在电脑放，是颠倒的，需要旋转90度。使用格式工厂失败了……
 参考[ffmpeg视频的翻转vflip、hflip，旋转rotate、transpose](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fyu540135101%2Farticle%2Fdetails%2F94481549)
 使用`ffmpeg -i 3.mp4 -vf rotate=PI/2 rotate8.mp4`画面确实旋转过来了，但是尺寸不对，变成横屏后，两侧的画面看不到了。改用`ffmpeg -i 3.mp4 -vf transpose=1 rotate8.mp4`解决了问题

# 三、[小丸工具箱](https://links.jianshu.com/go?to=https%3A%2F%2Fmaruko.appinn.me%2Findex.html%23)

小丸工具箱是一款用于处理音视频等多媒体文件的软件。是一款x264、ffmpeg等命令行程序的图形界面。它的目标是让视频压制变得简单、轻松。

主要功能：

- 高质量的H264+AAC视频压制
- ASS/SRT字幕内嵌到视频
- AAC/WAV/FLAC/ALAC音频转换
- MP4/MKV/FLV的无损抽取和封装

> 参考自[小丸FAQ](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnbeining.com%2F2013%2F11%2Fhow-do-i-report-an-error%2F)：小丸工具箱是一个x264(taro编译版,现在是7mod)、MP4Box、ffmpeg、MediaInfo等软件的GUI。工具箱只是一个调用其他程序的程序，自己没有压制功能！只是把平常需要命令行完成的工作图形化了！其实一切转换软件都是这个意思。

# 四、[fluent-ffmpeg](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Ffluent-ffmpeg-extended)

参考自
 [[FFmpeg探秘\]Ep.(1) 什么是FFmpeg?](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fread%2Fcv859)
 [[FFmpeg探秘\]Ep.(2) 从node-fluent-ffmpeg开始](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.bilibili.com%2Fread%2Fcv1073)
 [NODEJS基于FFMPEG视频推流测试](https://links.jianshu.com/go?to=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000012049916)

该nodejs包封装了ffmpeg的命令行调用部分,加强了代码的可读性,若熟悉ffmpeg 命令行使用手册,亦可不使用该包。

```
npm install --save fluent-ffmpeg
//使用js编码的用户,可以忽略下条命令
npm install --save @types/fluent-ffmpeg 
```

![img]()

image.png

# 五、使用ffmpeg推RTMP直播流

1.安装nignx环境
 弄个WINDOWS版本的Nginx吧，参照[Linux&Windows搭建基于nginx的视频点播服务器](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.csdn.net%2Fakeron%2Farticle%2Fdetails%2F54974034)，使用了[nginx-rtmp-win32](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Filluspas%2Fnginx-rtmp-win32)做了本地点播测试。具体步骤参照原文，为了节约时间，最好去[CSDN下载作者那个DEMO](https://links.jianshu.com/go?to=http%3A%2F%2Fdownload.csdn.net%2Fdetail%2Fakeron%2F9752215)

2.参考[手把手教你搭建Nginx-rtmp流媒体服务器+使用ffmpeg推流](https://www.jianshu.com/p/06c2025edcd3)
 在上述下载的demo中，看一下conf/nginx.conf配置文件：

```
rtmp {
    server {
        listen 1935;

        application live {
            live on;
        }
        
        application vod {
            play video;
        }
        
        application hls {
            live on;
            hls on;  
            hls_path temp/hls;  
            hls_fragment 8s;  
        }
    }
}
```

其中rtmp就是rtmp服务器模块，端口是1935，application我理解为一个路径。可以通过访问`rtmp://localhost/live`来访问live这个资源。live on  表示这是实时的传输，这不同于点播，点播就好比我在某视频网站上想看一个视频，无论我什么时候去点击，它会从头开始播放。而实时传输（直播），就是好比看电视，我在19:20去打开电视（打开直播路），视频不会从头开始播放，而是从当前(19:20)的视频数据开始播放。

然后在nginx.exe路径下命令行运行nginx -s reload重新加载配置。

3.使用ffmpeg推流
 参考[使用FFmpeg在B站直播的姿势](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F23951969)
 `ffmpeg -re -i 1.mp4 -vcodec copy -f flv rtmp://localhost/live`
 或者

```
ffmpeg -re -i 1.mp4 -vcodec copy -acodec copy
 -b:v 800k -b:a 32k -f flv rtmp://localhost/live
```

-re : **表示使用文件的原始帧率进行读取，因为ffmpeg读取视频帧的速度很快，如果不使用这个参数，ffmpeg可以在很短时间就把video.mp4中的视频帧全部读取完并进行推流，这样就无法体现出视频播放的效果了。**官方文档中对这个参数的解释是：

> -re (input)
>  Read input at native frame rate. Mainly used to simulate a grab device,  or live input stream (e.g. when reading from a file). Should not be used with actual grab devices or live input streams (where it can cause  packet loss). By default ffmpeg attempts to read the input(s) as fast as possible. This option will slow down the reading of the input(s) to the native frame rate of the input(s). It is useful for real-time output  (e.g. live streaming).

-vcodec copy : -vcodec表示使用的视频编解码器 ，前缀v表示video。后面紧跟的copy 表示复制使用源文件的视频编解码器，比如原文件的编解码器(codec)是h264，则这里就使用h264。

-acodec copy : -acodec表示使用的音频编解码器，前缀a表示audio。后面的copy 表示使用源文件的音频编解码器。

-b:v 800k : -b:v表示视频的比特率(bitrate) ，为800k。

-b:a 32k : 表示音频的比特率为32k。

-f flv : -f表示format ，就是强制输出格式为flv，这一步其实也叫封装(mux)，封装要做的事就是把视频和音频混合在一起，进行同步。紧跟在后面的`rtmp://localhost/live` 表示输出的"文件名"，这个文件名可以是一个本地的文件，也可以指定为rtmp流媒体地址。指定为rtmp流媒体地址后，则ffmpeg就可以进行推流。

4.可以使用VLC或ffplay进行播放了