[ffmpeg和H264视频的编解码](http://www.yanzuoguang.com/article/579)



# ffmpeg和H264视频的编解码

## **背景**

做CV的人经常面对的东西之一就是用ffmpeg处理视频，本文聚焦的就是ffmpeg和H264视频编码的一些概念和命令。因为实际使用的时候大多数的人都会遇到一些比较困惑的问题，比如ffmpeg截取视频为什么做不到帧级的精确。不管怎样，本文还是属于偏工程方面的论述。

在专栏文章[使用ffmpeg命令处理音视频](http://www.yanzuoguang.com/article/579#)中，Gemfield也介绍了一些基本的ffmpeg命令，而本文还将继续介绍一些不那么基本的命令。

## **为什么视频需要编解码？**

首先要解码是因为要编码，那为什么要编码呢？因为要压缩。假设看一部电影2小时的25fps的1080p视频，假设每个像素用1个字节存储（不准确哈，就是给个数量级，RGB是3个字节，但视频用的也不是RGB信息）。那么信息量就是 1920 x 1080 x 25 x 3600 x 2 = 373248000000 个字节，约为347 GB。现在清楚了吧。为了减少这个信息量，我们需要引入压缩算法，在2个维度上进行压缩，一是帧内压缩，一是帧间压缩。这个帧间压缩就是说，帧与帧之间的变化其实也没有那么大嘛，那就只在这帧图像上保留变化的信息，这就大大的减少的了信息量。这样的帧有2种：P-Frame 和 B-Frame。

## **P-Frame 、B-Frame、I-Frame、GOP、IDR**

**P-Frame** （Predictive-Frame），利用之前的I帧或P帧，采用运动预测的方式进行帧间预测编码；

**B-Frame** （bi predictive-Frame），bi双向的意思，双向预测编码图像帧)，提供最高的压缩比，它既需要之前的图像帧(I帧或P帧)，也需要后来的图像帧(P帧)，采用运动预测的方式进行帧间双向预测编码。

所以P帧和B帧只包含变化的信息。

但是啊，有的时候，比如镜头切换等，变化的信息量反而更大，那使用P帧或者B帧反而得不偿失。怎么办呢？干脆算了，h264编码这个时候就会插入key frame（关键帧），也就是不依赖前后帧的独立的一帧图像。key frame也叫**I-Frame**，也就是intra-frame。只有这个时候需要插入key frame吗？不是的！假设一个视频从头到尾都没有这样的剧烈变化的镜头，那就只有第一帧是key frame了，那么我做seek的时候（你想快进啊，或者想从中间看视频啊），那就灾难了，比方你要seek到第1小时，那程序就得先decode 1小时的视频才能计算出你要播放的帧...卒！

所以啊，一般都是以有规律的interval来插入key frame，这个有规律的interval就叫做I-Frame interval ，或者叫做I-Frame distance，或者叫做GOP length/size（Group Of Images），这个值一般是10倍的fps（libx264默认将这个interval设置为250，另外，x264编码器在检测到大的场景变化时，会在变化开始处插入key frame) 。另外，ESPN是每10秒插入一个key frame，YouTube每2秒插入一个关键帧，Apple每3秒到每10秒插入一个key frame。

再详细一点说说**GOP**这个概念，GOP结构一般会使用2个数字来描述，比如, M=3, N=12。第一个数字3表示的是2个anchor frame(I帧 或者 P帧)之间的距离，第二个数字12表示2个key frame之间的距离（也就是GOP size或者GOP length），那么对于这个例子来说， GOP结构就是IBBPBBPBBPBBI。

IDR(instantaneous decoder refresh) frame首先是 keyframe，对于普通的keyframe（non-IDR keyframe）来说，其后的P-Frame和B-Frame可以引用此keyframe之前的帧，但是IDR就不行，IDR后的 P-Frame和B-Frame不能引用此IDR之前的帧。所以decoder遇到IDR后，就可以毫不犹豫的抛弃之前的解码序列，从新开始(refresh)。这样当遇到解码错误的时候，错误不会影响太远，将止步于IDR。

下面的伪代码展示了如何生成I-Frame和P-Frame：

```bash
if ((distance from previous keyframe) > keyint) then
    set IDR-frame
else if (1 - (bit size of P-frame) / (bit size of I-frame) < (scenecut / 100) * (distance from previous keyframe) / keyint) then
    if ((distance from previous keyframe) >= minkeyint) then
        set IDR-frame
    else
        set I-frame
else
    set P-frame
```

scenecut变量指的是场景变化的阈值，0表示当前帧和前面一帧完全一样, 100表示当前帧和前面一帧完全不一样。keyint是两个keyframe之间的最大距离，minkeyint是两个keyframe之间的最小距离。

执行下面的ffprobe命令

```text
ffprobe -v error -show_frames gemfield.mp4 
```

输入中截取如下片段：

```text
[FRAME]
media_type=video
stream_index=0
key_frame=1
pkt_pts=104000
pkt_pts_time=4.160000
pkt_dts=103000
pkt_dts_time=4.120000
best_effort_timestamp=104000
best_effort_timestamp_time=4.160000
pkt_duration=1000
pkt_duration_time=0.040000
pkt_pos=33599
pkt_size=77692
width=1280
height=720
pix_fmt=yuv420p
sample_aspect_ratio=N/A
pict_type=I
coded_picture_number=0
display_picture_number=0
interlaced_frame=0
top_field_first=0
repeat_pict=0
[/FRAME]
```

从pict_type=I可以看出这是个关键帧，然后key_frame=1 表示这是IDR frame，如果key_frame=0表示这是Non-IDR frame。

## **FFMPEG**

在继续之前，gemfield先强调三下ffmpeg的一个参数惯例：

**注意：ffmpeg所有的参数都是作用于紧跟其后的文件，因此参数的顺序相当重要！！！**

**注意：ffmpeg所有的参数都是作用于紧跟其后的文件，因此参数的顺序相当重要！！！**

**注意：ffmpeg所有的参数都是作用于紧跟其后的文件，因此参数的顺序相当重要！！！**

ffmpeg的seeking有2种方式，input seeking （使用I-Frame）和output seeking（逐帧decode）。

**1，先说图片**

比方说我要把gemfield.mp4视频的第1分05秒的一帧图像截取出来，就有2种方法，如下所示：

```bash
# input seeking
ffmpeg -ss 00:1:05 -i gemfield.mp4 -frames:v 1 out.jpg
# output seeking
ffmpeg -i gemfield.mp4 -ss 00:1:05 -frames:v 1 out1.jpg
```

-frame:v 1的意思是说在video stream上截取1帧。

好了，这两种seeking有什么区别呢？input seeking使用的是key frames，所以速度很快；而output seeking是逐帧decode，直到1分05秒，所以速度很慢。当1分05秒变为1小时1分05秒的时候，这个差距就更大了。那么这两者产生的output有什么区别呢？没有！因为不管是哪种seeking，这中间（从视频抽帧成jpg图片）必然涉及到transcoding（decoding再encoding）。

**2，再说视频**

ffmpeg截取视频的时候，照样有2种seeking了，但是此外还有2种coding模式：transcoding 和 stream copying（ffmpeg -c copy）。因为是从视频到视频，并不必然需要decoding + encoding（比方说我从原始的h264视频截取出来一小段h264视频）。

transcoding就是先decoding再encoding（输入是容器level，所以其实顺序是demuxing、decoding、filter、encoding、muxing），decoding和encoding可以加入filter（因为filter只能工作在未压缩的data上）；

而stream copying 则是不需要decoding + encoding的模式，由命令行选项-codec加上参数copy来指定（-c:v copy ）。在这种模式下，ffmpeg在video stream上就会忽略 decoding 和 encoding步骤，从而只做demuxing和muxing。通常是用来修改容器（container） level的元数据，如下图所示：

```text
_______              ______________            ________
|       |            |              |          |        |
| input |  demuxer   | encoded data |  muxer   | output |
| file  | ---------> | packets      | -------> | file   |
|_______|            |______________|          |________|
```

因为没有transcoding的过程，所以速度非常快（相比于有transcoding的过程）。

再来说说2种seeking模式，对于截取视频来说，**input seeking使用的是keyframe，output seeking使用的也是key frame！**再来说说2种模式，当使用transcoding的时候，是frame-accurate的。而当使用stream copying的方式时，这种方式就不是frame-accurate的。为什么呢？**对于ffmpeg来说，当使用-ss 和-c:v copy 时， ffmpeg将只会使用i-frames**。比方说（当output seeking + stream copying的时候）你指定起始点是719秒，而直到721秒才有个key frame，那么cut产生的视频在前2秒就只有声音，过了2秒后才会有视频（刚好到了key frame），所以一定要小心啊（注意啊，mplayer软件可能会把2秒空白期挪到后面）。

以截取一段4秒长的视频为例（选取00:01:01，也就是起始点为61秒，是因为此处最近的关键帧位于58.56秒和64.56秒）：

```text
#1, use stream copying & input seeking
ffmpeg -ss 00:01:01 -i gemfield.mp4 -t 4 -c copy cut1.mp4

#2 use stream copying & output seeking
ffmpeg -i gemfield.mp4 -ss 00:01:01  -t 4 -c:v copy cut2.mp4

#3 use transcoding & input seeking
ffmpeg -ss 00:01:01 -i gemfield.mp4  -t 4 -c:v libx264 cut3.mp4

#4 use transcoding & output seeking
ffmpeg -i gemfield.mp4 -ss 00:01:01 -t 4 -c:v libx264 cut4.mp4
```

这#1、#2、#3、#4分别表现是什么呢？

\#1，当为Input seeking + stream copying的时候，我们想截取的是[61, 65)的片段，实际截取的是**[58.56, 65)**的片段，是的，ffmpeg往前移动到了一个I-Frame；

\#2，当为output seeking + stream copying的时候，我们想截取的是[61, 65)的片段，实际截取的是**[64.56, 65)** 的画面，**再加上 (4 - 0.44)秒的空白片段**，生成长度为4秒的视频。播放器在播放的时候，这个空白片段怎么播放是由播放器自定义的。总之，画面的有效信息只有后面的关键帧开始的一小段信息；

\#3，当为input seeking + transcoding 的时候，我们想截取的是[61, 65)的片段，实际截取的是**[61, 65)**的片段，是的，frame-accurate；

\#4，当为output seeking + transcoding 的时候，我们想截取的是[61, 65)的片段，实际截取的是**[61, 65)**的片段，是的，frame-accurate。

可以看到，#3和#4是一样的。

**3，最后给出一些命令**

**a, 怎么得到一个视频的总的帧数呢？**

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -count_frames -select_streams v:0 -show_entries stream=nb_frames -of default=nokey=1:noprint_wrappers=1 gemfield.mp4
2399
```

gemfield.mp4有2399帧。

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -count_frames -select_streams v:0 -show_entries stream=nb_read_frames -of default=nokey=1:noprint_wrappers=1 gemfield.mp4
2398
```

gemfield.mp4有2398帧。

**b, 怎么得到一个视频的key frame的帧数呢？**

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -count_frames -select_streams v:0 -show_entries stream=nb_read_frames -of default=nokey=1:noprint_wrappers=1 -skip_frame nokey gemfield.mp4
21
```

gemfield.mp4有21个关键帧。

**c, 怎么得到一个视频的key frame所在的时间呢？**

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -skip_frame nokey -select_streams v:0 -show_entries frame=pkt_pts_time -of csv=print_section=0 gemfield.mp4
4.160000
8.640000
13.760000
18.080000
24.080000
26.080000
28.120000
......
```

**d, 看一个视频中关键帧的分布情况**

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -show_frames gemfield.mp4 | grep pict_type
pict_type=I
pict_type=P
pict_type=B
pict_type=B
pict_type=B
pict_type=B
pict_type=P
pict_type=P
pict_type=B
......
```

**e, 看一个视频中关键帧所在的帧数**

```text
gemfield@ThinkPad-X1C:~$ ffprobe -v error -select_streams v -show_frames -show_entries frame=pict_type -of csv gemfield.mp4 | grep -n I | cut -d ':' -f 1
1
113
241
349
499
549
600
......
```

**f, 重新设置key frame interval**

```text
gemfield@ThinkPad-X1C:~$ ffmpeg -i gemfield.mp4 -vcodec libx264 -x264-params keyint=1:scenecut=0 -acodec copy out.mp4
......
#看看视频的大小变化
gemfield@ThinkPad-X1C:~$ ls -lh gemfield.mp4 out.mp4
-rw-rw-r-- 1 gemfield gemfield 13M 4月   3 11:49 gemfield.mp4
-rw-rw-r-- 1 gemfield gemfield 97M 4月  25 21:10 out.mp4

#看看波特率的变化
gemfield@ThinkPad-X1C:~$ ffprobe -v error -select_streams v:0 -show_entries stream=bit_rate -of default=noprint_wrappers=1:nokey=1 gemfield.mp4
1033337
gemfield@ThinkPad-X1C:~$ ffprobe -v error -select_streams v:0 -show_entries stream=bit_rate -of default=noprint_wrappers=1:nokey=1 out.mp4 
7985842
```