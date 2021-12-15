ffmpeg是一个源于Linux的工具软件，是FLV视频转换器，可以轻易地实现FLV向其它格式avi、asf、 mpeg的转换或者将其它格式转换为flv。

# 一、参数

## 1. 通用选项

```
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

## 2. 视频选项

```yaml
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



## 3. 高级视频选项

```yaml
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

## 4. 音频选项

```yaml
-ab bitrate 设置音频码率

-ar freq 设置音频采样率

-ac channels 设置通道 缺省为1

-an 不使能音频纪录

-acodec codec 使用codec编解码
```



## 5. 音频/视频捕获选项

```
-vd device 设置视频捕获设备。比如/dev/video0

-vc channel 设置视频捕获通道 DV1394专用

-tvstd standard 设置电视标准 NTSC PAL(SECAM)

-dv1394 设置DV1394捕获

-av device 设置音频设备 比如/dev/dsp
```



## 6. 高级选项

```
-map file:stream 设置输入流映射

-debug 打印特定调试信息

-benchmark 为基准测试加入时间

-hex 倾倒每一个输入包

-bitexact 仅使用位精确算法 用于编解码测试

-ps size 设置包大小，以bits为单位

-re 以本地帧频读数据，主要用于模拟捕获设备

-loop 循环输入流。只工作于图像流，用于ffserver测试
```

# 二、示例

## 2.1 demo1

```bash
# ffmpeg -y -i "1.avi" -title "Test" -vcodec xvid -s 368x208 -r 29.97 -b 1500 -acodec aac -ac 2 -ar 24000 -ab 128 -vol 200 -f psp -muxvb 768 "output.wmv" 

解释如下：

-y 	覆盖输出文件，即如果 output.wmv 文件已经存在的话，不经提示就覆盖掉
-i "1.avi" 	输入文件是和ffmpeg在同一目录下的1.avi文件，可以自己加路径，改名字
-title "Test" 	在PSP中显示的影片的标题
-vcodec xvid 	使用XVID编码压缩视频，不能改的
-s 368x208 	输出的分辨率为368x208，注意片源一定要是16:9的不然会变形
-r 29.97 	帧数，一般就用这个吧
-b 1500 	视频数据流量，用-b xxxx的指令则使用固定码率，数字随便改，1500以上没效果；还可以用动态码率如：-qscale 4和-qscale 6，4的质量比6高
-acodec aac 	音频编码用AAC
-ac 2 	声道数1或2
-ar 24000 	声音的采样频率，好像PSP只能支持24000Hz
-ab 128 	音频数据流量，一般选择32、64、96、128
-vol 200 	200%的音量，自己改
-muxvb 768 	好像是给PSP机器识别的码率，一般选择384、512和768，我改成1500，PSP就说文件损坏了
-f psp 	输出psp专用格式
"output.wmv" 	输出文件名，也可以加路径改文件名
```

## 2.2 demo2

```bash
# ffmpeg -ss 00:00:00 -t 00:00:03 -y -i test.mp4 -vcodec copy -acodec copy test1.mp4   #视频裁剪

解释如下：

上面的这个例子是将test.mp4视频的前3秒，重新生成一个新视频。

-ss 开始时间，如： 00:00:00，表示从0秒开始，格式也可以00:00:0

-t 时长，如： 00:00:03，表示截取3秒长的视频，格式也可以00:00:3

-y 如果文件已存在强制替换；

-i 输入，后面是空格，紧跟着就是输入视频文件；

-vcodec copy 和 -acodec copy表示所要使用的视频和音频的编码格式，这里指定为copy表示原样拷贝；
```

## 2.3 demo3

```bah
# ffmpeg -i test.mp4 -y -f mjpeg -ss 3 -t 1  test1.jpg    
      
# ffmpeg -i test.mp4 -y -f image2 -ss 3 -vframes 1 test1.jpg    

解释如下：

上面二个例子都表示，在第三秒的时候，截图。
```

## 2.4 demo4

```bash
[root@localhost test]# ffmpeg -i test.mp4 2>&1 | grep 'Duration' | cut -d ' ' -f 4 | sed s/,//   
00:00:33.73  

例子说明：

获取视频时长
```

## 2.5 demo5

```bash
# ffmpeg -i test.mp4 -ab 56 -ar 22050 -qmin 2 -qmax 16 -b 320k -r 15 -s 320x240 outputfile.flv     #mp4 转 flv  
      
# ffmpeg -i outputfile.flv -copyts -strict -2 test.mp4       #flv 转 mp4  
```

## 2.6 demo6

```bash
# ffmpeg -y -i test.mp4 -acodec copy -vf "movie=uwsgi.jpg [logo]; [in][logo] overlay=10:10:1 [out]" test2.mp4       

例子说明：

overlay=10:10:1，后三个数据表示是距离左边的距离，距离上边的距离，是否透明，1表示透明。上例我用的是jpg，当然不可能透明。
```

## 2.7 demo7 

```bash
# ffmpeg -y -i test.mp4 -acodec copy -vf "movie=uwsgi.jpg [logo]; [in][logo] overlay=enable='lte(t,1)' [out]" test2.mp4     

例子说明：

overlay=enable='lte(t,1)' ，这个参数表示，水印在前一秒显示。
```

## 2.8 demo8

```bash
# ffmpeg -i test.asf -vframes 30 -y -f gif a.gif    #把视频的前30帧转换成一个Gif  

# ffmpeg -i test2.mp4 -y -f image2 -ss 08 -t 0.001 -s 352x240 b.jpg     #在秒处接取一个352X240的图片  
```

## 2.9 demo9

```bash
# ffmpeg -i "rtmp://192.168.10.103:1935/live/111 live=1" -acodec copy -vcodec copy -f flv -y test.flv

例子说明：

将rtmp流，以文件的形势保存到本地
```

## 2.10 demo10

```bash
# ffmpeg -i test.mp3  -i test1.mp3 -filter_complex amix=inputs=2 -f mp3 c.mp3    #合并二个音频

# ffmpeg -i test.mp3 -i test1.mp3 -filter_complex amix=inputs=2:duration=first:dropout_transition=2 -f mp3 a.mp3

例子说明：

合并二个音频，以第一个音频的时长为新音频时长
```

## 2.11 demo11

```
a.txt 格式如下file '/path/to/file1'
file '/path/to/file2'
file '/path/to/file3'
```

```bash
[root@iZ94zz3wqciZ live]# ffmpeg -f concat -i a.txt -c copy a.flv    #视频合成
ffmpeg version 2.2.1 Copyright (c) 2000-2014 the FFmpeg developers
  built on Aug 11 2016 14:59:59 with gcc 4.4.7 (GCC) 20120313 (Red Hat 4.4.7-17)
  configuration: --prefix=/usr/local/ffmpeg/
  libavutil      52. 66.100 / 52. 66.100
  libavcodec     55. 52.102 / 55. 52.102
  libavformat    55. 33.100 / 55. 33.100
  libavdevice    55. 10.100 / 55. 10.100
  libavfilter     4.  2.100 /  4.  2.100
  libswscale      2.  5.102 /  2.  5.102
  libswresample   0. 18.100 /  0. 18.100
[flv @ 0x1e607e0] Broken FLV file, which says no streams present, this might fail
[concat @ 0x1e57900] Estimating duration from bitrate, this may be inaccurate
Input #0, concat, from 'a.txt':
  Duration: 00:00:00.03, start: 0.000000, bitrate: 51 kb/s
    Stream #0:0: Video: flv1, yuv420p, 320x240, 320 kb/s, 15.17 fps, 15 tbr, 1k tbn, 1k tbc
    Stream #0:1: Audio: mp3, 22050 Hz, stereo, s16p, 64 kb/s
Output #0, flv, to 'a.flv':
  Metadata:
    encoder         : Lavf55.33.100
    Stream #0:0: Video: flv1 ([2][0][0][0] / 0x0002), yuv420p, 320x240, q=2-31, 320 kb/s, 15.17 fps, 1k tbn, 1k tbc
    Stream #0:1: Audio: mp3 ([2][0][0][0] / 0x0002), 22050 Hz, stereo, 64 kb/s
Stream mapping:
  Stream #0:0 -> #0:0 (copy)
  Stream #0:1 -> #0:1 (copy)
Press [q] to stop, [?] for help
[flv @ 0x1e607e0] Broken FLV file, which says no streams present, this might fail
[flv @ 0x1f0d420] Broken FLV file, which says no streams present, this might fail
frame= 5049 fps=0.0 q=-1.0 Lsize=   16532kB time=00:05:36.59 bitrate= 402.3kbits/s    
video:13621kB audio:2631kB subtitle:0 data:0 global headers:0kB muxing overhead 1.726711%
```

## 2.12 demo12

```bash
[root@iZ94zz3wqciZ live]# ffmpeg -i "concat:1.flv|2.flv|3.flv" -c copy 4.flv      #视频合成
ffmpeg version 2.2.1 Copyright (c) 2000-2014 the FFmpeg developers
  built on Aug 11 2016 14:59:59 with gcc 4.4.7 (GCC) 20120313 (Red Hat 4.4.7-17)
  configuration: --prefix=/usr/local/ffmpeg/
  libavutil      52. 66.100 / 52. 66.100
  libavcodec     55. 52.102 / 55. 52.102
  libavformat    55. 33.100 / 55. 33.100
  libavdevice    55. 10.100 / 55. 10.100
  libavfilter     4.  2.100 /  4.  2.100
  libswscale      2.  5.102 /  2.  5.102
  libswresample   0. 18.100 /  0. 18.100
[flv @ 0x248b900] Broken FLV file, which says no streams present, this might fail
Input #0, flv, from 'concat:1.flv|2.flv|3.flv':
  Duration: 00:00:05.59, start: 0.000000, bitrate: 24214 kb/s
    Stream #0:0: Video: flv1, yuv420p, 320x240, 320 kb/s, 15 tbr, 1k tbn, 1k tbc
    Stream #0:1: Audio: mp3, 22050 Hz, stereo, s16p, 64 kb/s
Output #0, flv, to '4.flv':
  Metadata:
    encoder         : Lavf55.33.100
    Stream #0:0: Video: flv1 ([2][0][0][0] / 0x0002), yuv420p, 320x240, q=2-31, 320 kb/s, 1k tbn, 1k tbc
    Stream #0:1: Audio: mp3 ([2][0][0][0] / 0x0002), 22050 Hz, stereo, 64 kb/s
Stream mapping:
  Stream #0:0 -> #0:0 (copy)
  Stream #0:1 -> #0:1 (copy)
Press [q] to stop, [?] for help
frame=  833 fps=0.0 q=-1.0 Lsize=    2898kB time=00:00:55.56 bitrate= 427.3kbits/s    
video:2418kB audio:434kB subtitle:0 data:0 global headers:0kB muxing overhead 1.633255%
```

## 2.13 demo13

```bash
# ffmpeg -i 1.m2ts -vcodec hevc_nvenc -preset llhp -rc:v constqp -qp 18 -level 4.1 xxxx.mp4 

hevc_nvenc 是硬件编码，速度上有优势。几乎是软编码的20~30倍。缺点是同码率下，效果稍差，容量大20%内，如果静态画面多容量会突破天际（一般电影不会很多静态吧，除了写真和风景）。

这里 -QP 相当于 CRF参数。
```

