简书：作者：往后余生9375：链接：https://www.jianshu.com/p/049d03705a81

# 推流命令

```bash
ffmpeg -re  -rtsp_transport tcp -i rtsp:///////  -vcodec copy  -f flv  rtmp://192.168.0.140/rtmp
```







# 压缩视频

```bash
ffmpeg -i pingcap-intro-converted.mp4 -b:v 64k -r 20 -c:v libx264 -s 640x320 -strict -2 pingcap.mp4
```

# 获取封面

```bash
ffmpeg -ss 00:00:10 -i test1.flv -f image2 -y test1.jpg
```

# 屏幕类型

```css
普屏4:3  320*240 640*480 
宽屏16:9  480*272 640*360 672*378 720*480 1024*600 1280*720 1920*1080 
```

# **ffmpeg命令参数如下:**

| 参数名称             | 输入值                               | 备注                                                         |
| -------------------- | ------------------------------------ | ------------------------------------------------------------ |
| -i                   | ffmpmg -i pingcap-xxx.mp4            | 输入您要处理的视频文件路径                                   |
| -b：v $k -bufsize $k | -b：v 64k -bufsize 64k               | 要将输出文件的视频比特率设置为64 kbit / s                    |
| -r                   | ffmpeg -i input.avi -r 24 output.avi | 要强制输出文件的帧频为24 fps                                 |
| -c:v                 | -c:v libx264                         | ffmpeg -i input -c:v libx264 -preset slow -crf 22-c:a copy output.mkv |

# 通用选项

```bash
-L license

-h 帮助

-fromats 显示可用的格式，编解码的，协议的。。。

-f fmt 强迫采用格式fmt

-I filename 输入文件

-y 覆盖输出文件

-t duration 设置纪录时间 hh:mm:ss[.xxx]格式的记录时间也支持

-ss position 搜索到指定的时间 [-]hh:mm:ss[.xxx]的格式也支持

-title string 设置标题

-author string 设置作者

-copyright string 设置版权

-comment string 设置评论

-target type 设置目标文件类型(vcd,svcd,dvd) 所有的格式选项（比特率，编解码以及缓冲区大小）自动设置 ，只需要输入如下的就可以了：
ffmpeg -i myfile.avi -target vcd /tmp/vcd.mpg

-hq 激活高质量设置

-itsoffset offset 设置以秒为基准的时间偏移，该选项影响所有后面的输入文件。该偏移被加到输入文件的时戳，定义一个正偏移意味着相应的流被延迟了 offset秒。 [-]hh:mm:ss[.xxx]的格式也支持
```

# 视频选项

```bash
-b bitrate 设置比特率，缺省200kb/s

-r fps 设置帧频 缺省25

-s size 设置帧大小 格式为WXH 缺省160X128.下面的简写也可以直接使用：
Sqcif 128X96 qcif 176X144 cif 252X288 4cif 704X576

-aspect aspect 设置横纵比 4:3 16:9 或 1.3333 1.7777

-croptop size 设置顶部切除带大小 像素单位

-cropbottom size –cropleft size –cropright size

-padtop size 设置顶部补齐的大小 像素单位

-padbottom size –padleft size –padright size –padcolor color 设置补齐条颜色(hex,6个16进制的数，红:绿:兰排列，比如 000000代表黑色)

-vn 不做视频记录

-bt tolerance 设置视频码率容忍度kbit/s

-maxrate bitrate设置最大视频码率容忍度

-minrate bitreate 设置最小视频码率容忍度

-bufsize size 设置码率控制缓冲区大小

-vcodec codec 强制使用codec编解码方式。 如果用copy表示原始编解码数据必须被拷贝。

-sameq 使用同样视频质量作为源（VBR）

-pass n 选择处理遍数（1或者2）。两遍编码非常有用。第一遍生成统计信息，第二遍生成精确的请求的码率

-passlogfile file 选择两遍的纪录文件名为file
```

# 高级选项

```bash
-g gop_size 设置图像组大小

-intra 仅适用帧内编码

-qscale q 使用固定的视频量化标度(VBR)

-qmin q 最小视频量化标度(VBR)

-qmax q 最大视频量化标度(VBR)

-qdiff q 量化标度间最大偏差 (VBR)

-qblur blur 视频量化标度柔化(VBR)

-qcomp compression 视频量化标度压缩(VBR)

-rc_init_cplx complexity 一遍编码的初始复杂度

-b_qfactor factor 在p和b帧间的qp因子

-i_qfactor factor 在p和i帧间的qp因子

-b_qoffset offset 在p和b帧间的qp偏差

-i_qoffset offset 在p和i帧间的qp偏差

-rc_eq equation 设置码率控制方程 默认tex^qComp

-rc_override override 特定间隔下的速率控制重载

-me method 设置运动估计的方法 可用方法有 zero phods log x1 epzs(缺省) full

-dct_algo algo 设置dct的算法 可用的有 0 FF_DCT_AUTO 缺省的DCT 1 FF_DCT_FASTINT 2 FF_DCT_INT 3 FF_DCT_MMX 4 FF_DCT_MLIB 5 FF_DCT_ALTIVEC

-idct_algo algo 设置idct算法。可用的有 0 FF_IDCT_AUTO 缺省的IDCT 1 FF_IDCT_INT 2 FF_IDCT_SIMPLE 3 FF_IDCT_SIMPLEMMX 4 FF_IDCT_LIBMPEG2MMX 5 FF_IDCT_PS2 6 FF_IDCT_MLIB 7 FF_IDCT_ARM 8 FF_IDCT_ALTIVEC 9 FF_IDCT_SH4 10 FF_IDCT_SIMPLEARM

-er n 设置错误残留为n 1 FF_ER_CAREFULL 缺省 2 FF_ER_COMPLIANT 3 FF_ER_AGGRESSIVE 4 FF_ER_VERY_AGGRESSIVE

-ec bit_mask 设置错误掩蔽为bit_mask,该值为如下值的位掩码 1 FF_EC_GUESS_MVS (default=enabled) 2 FF_EC_DEBLOCK (default=enabled)

-bf frames 使用frames B 帧，支持mpeg1,mpeg2,mpeg4

-mbd mode 宏块决策 0 FF_MB_DECISION_SIMPLE 使用mb_cmp 1 FF_MB_DECISION_BITS 2 FF_MB_DECISION_RD

-4mv 使用4个运动矢量 仅用于mpeg4

-part 使用数据划分 仅用于mpeg4

-bug param 绕过没有被自动监测到编码器的问题

-strict strictness 跟标准的严格性

-aic 使能高级帧内编码 h263+

-umv 使能无限运动矢量 h263+

-deinterlace 不采用交织方法

-interlace 强迫交织法编码 仅对mpeg2和mpeg4有效。当你的输入是交织的并且你想要保持交织以最小图像损失的时候采用该选项。可选的方法是不交织，但是损失更大

-psnr 计算压缩帧的psnr

-vstats 输出视频编码统计到vstats_hhmmss.log

-vhook module 插入视频处理模块 module 包括了模块名和参数，用空格分开
```

# 音频选项

```bash
-ab bitrate 设置音频码率

-ar freq 设置音频采样率

-ac channels 设置通道 缺省为1

-an 不使用音频，去掉音频

-acodec codec 使用codec编解码
```

# 音视频捕获选项

```bash
-vd device 设置视频捕获设备。比如/dev/video0

-vc channel 设置视频捕获通道 DV1394专用

-tvstd standard 设置电视标准 NTSC PAL(SECAM)

-dv1394 设置DV1394捕获

-av device 设置音频设备 比如/dev/dsp
```

# 高级选项

```bash
-map file:stream 设置输入流映射

-debug 打印特定调试信息

-benchmark 为基准测试加入时间

-hex 倾倒每一个输入包

-bitexact 仅使用位精确算法 用于编解码测试

-ps size 设置包大小，以bits为单位

-re 以本地帧频读数据，主要用于模拟捕获设备

-loop 循环输入流。只工作于图像流，用于ffserver测试
```

## 1. ffmpeg视频剪切

```
$ ffmpeg -i ./in.mp4  -vcodec copy -acodec copy -ss 00:00:20 -to 00:05:30 ./out.mp4
```

-ss为开始时间，-to为结束时间。

## 2. 设置视频大小

```
$ ffmpeg -i ./sea.mp4 -fs 19M output.mp4
```

-fs需要设置的大小，例如19M、1024K，其实就是剪切了前19M、1024K的视频内容。

## 3. 删除视频中的音频

```
$ ffmpeg  -i in.mp4  -map 0:0  -vcodec copy -acodec copy out.mp4
```

通过ffprobe命令，可以查看所有的通道，例子中的0:0就是视频通道。

## 4. 设置分辨率

```
$ ffmpeg -i video_1920.mp4 -vf scale=640:360 video_640.mp4 -hide_banner
```

高分辨率向低分辨率的转化。

## 5.设置视频的宽高比

```
$ ffmpeg -i video_320x180.mp4 -vf scale=320:240,setdar=4:3 video_320x240.mp4 -hide_banner
```

## 6. 视频倒放，无音频

```
$ ffmpeg -i in.mp4 -filter_complex [0:v]reverse[v] -map [v] -preset superfast out.mp4
```

## 7.视频倒放，音频不变

```
$ ffmpeg -i in.mp4 -vf reverse out.mp4
```

## 8.音频倒放，视频不变

```
$ ffmpeg -i in.mp4 -map 0 -c:v copy -af "areverse" out.mp4
```

## 9.音视频同时倒放

```
$ ffmpeg -i in.mp4 -vf reverse -af areverse -preset superfast out.mp4
```

## 10.抽取音频

```
$ ffmpeg -i 3.mp4 -vn -y -acodec copy 3.aac
$ ffmpeg -i 3.mp4 -vn -y -acodec copy 3.m4a
```

## 11.提取视频或者叫做删除音频

```
ffmpeg -i Life.of.Pi.has.subtitles.mkv -vcodec copy –an  videoNoAudioSubtitle.mp4
ffmpeg -i output.mp4 -c:v copy -an  input-no-audio.mp4
```

## 12.为无声的视频添加音频

```
ffmpeg -i ../out/4in1.mp4  -i ./3.aac  -vcodec copy -acodec copy output.mp4
```