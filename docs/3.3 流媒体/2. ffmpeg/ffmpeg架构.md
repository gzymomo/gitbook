# 简介

FFMPEG是特别强大的专门用于处理音视频的开源库。你既可以使用它的API对音视频进行处理，也可以使用它提供的工具，如 ffmpeg, ffplay, ffprobe，来编辑你的音视频文件。

# 目录以及作用

- libavcodec： 提供了一系列编码器的实现。
- libavformat： 实现在流协议，容器格式及其本IO访问。
- libavutil： 包括了hash器，解码器和各利工具函数。
- libavfilter： 提供了各种音视频过滤器。
- libavdevice： 提供了访问捕获设备和回放设备的接口。
- libswresample： 实现了混音和重采样。
- libswscale： 实现了色彩转换和缩放工能

# 基本概念

- 音/视频流

在音视频领域，我们把一路音/视频称为一路**流**。如我们小时候经常使用VCD看港片，在里边可以选择粤语或国语声音，其实就是CD视频文件中存放了两路音频流，用户可以选择其中一路进行播放。

- 容器

我们一般把 MP4､ FLV、MOV等文件格式称之为**容器**。也就是在这些常用格式文件中，可以存放多路音视频文件。以 MP4 为例，就可以存放一路视频流，多路音频流，多路字幕流。

- channel

channel是音频中的概念，称之为声道。在一路音频流中，可以有单声道，双声道或立体声。

# 播放器架构

![img](https://box.kancloud.cn/1cfb47f91903b586906422f7ecf2af34_776x579.png)

# 渲染过程

![img](https://box.kancloud.cn/338042756690cf9fd4841c197dafb33e_2352x308.png)

# ffmpeg处理音视流程

![img](https://box.kancloud.cn/77b6ab2557f384ca80c4c30c1d852a00_2314x866.png)

将编码的数据包传送给解码器（除非为数据流选择了流拷贝）。

解码器产生未压缩的帧（原始视频/ PCM音频/ ...），可以通过滤波进一步处理。

在过滤之后，帧被传递到编码器，编码器并输出编码的数据包。 最后，这些传递给复用器，将编码的数据包写入输出文件。

默认情况下，ffmpeg只包含输入文件中每种类型（视频，音频，字幕）的一个流，并将其添加到每个输出文件中。 它根据以下标准挑选每一个的“最佳”：对于视频，它是具有最高分辨率的流，对于音频，它是具有最多channel的流，对于字幕，是第一个字幕流。 在相同类型的几个流相等的情况下，选择具有最低索引的流。

您可以通过使用`-vn/-an/-sn/-dn`选项来禁用某些默认设置。 要进行全面的手动控制，请使用`-map`选项，该选项禁用刚描述的默认设置

# 下载编译与安装

**mac需要这个才能录制声音**

```
brew cask install soundflower
```

还要配置下的去常见错误那看

**下载编译**

```
git clone https://git.ffmpeg.org/ffmpeg.git
./configure --help
```

![img](https://box.kancloud.cn/b9306f798f14600ce22d8cbe11cca117_2998x408.png)
缺少的要去安装

```
make && make install
```

这些安装包的版本都是Homebrew的方案(formulas)，安装程序会自动将FFmpeg的依赖库安装好。你可以输入`brew info ffmpeg`查看额外的安装选项，如：如果想要添加`libfdk_aac`或`libvpx`两个库(这两个库是高度推荐安装的)，可以输入以下包含额外推荐选项的命令:

```
brew install ffmpeg --with-fdk-aac --with-ffplay --with-freetype --with-libass --with-libquvi --with-libvorbis --with-libvpx --with-opus --with-x265
```

如果你想通过Homebrew安装FFmpeg的最新Git版本，在第一条安装命令后面添加`--HEAD`，如：

```
brew install ffmpeg --HEAD
```

或者

```
brew install ffmpeg --with-chromaprint --with-fdk-aac --with-libass --with-libsoxr --with-libssh --with-tesseract --with-libvidstab --with-opencore-amr --with-openh264 --with-openjpeg --with-openssl --with-rtmpdump --with-rubberband --with-sdl2 --with-snappy --with-tools --with-webp --with-x265 --with-xz --with-zeromq --with-zimg --with-fontconfig --with-freetype --with-frei0r --with-game-music-emu --with-libbluray --with-libbs2b --with-libcaca --with-libgsm --with-libmodplug --with-libvorbis --with-libvpx --with-opus --with-speex --with-theora --with-two-lame --with-wavpack
```

**如果需要'libfdk_aac'可以**

```
brew install fdk-aac
```

或者

```
git clone git://github.com/mstorsjo/fdk-aac
cd fdk-aac
autoreconf -i
./configure
make install
```

或者

```
brew install ffmpeg --with-fdk-aac
```

**mac编译需要这些**

```
brew install automake fdk-aac git lam libass libtool libvorbis libvpx \ opus sdl shtool texi2html theora wget x264 xvid yasm
```

在执行各自的 configure 创建编译配置文件时，最好都强制带上 --enable-static 和 --enable-shared 参数以确保生成静态库和动态库。另外因为是在 Mac OS X 环境下编译，因此在各自编译完后，都要执行 sudo make install，安装到默认的 `/usr/local` 目录下相应位置（Mac OS X 下不推荐 /usr），因此不要在 configure 时指定 --prefix，就用默认的 /usr/local 目录前缀即可。完成编译安装后，FFmpeg 的头文件将会复制到 /usr/local/include 下面相应位置，静态库及动态库会被复制到 `/usr/local/lib` 目录下，FFmpeg 的可执行程序（ffmpeg、ffprobe、ffserver）会被复制到 `/usr/local/bin` 目录下，这样 FFmpeg 的开发环境就构建好了

# 声音

- 音调: 就是音频, 男生->女生->儿童
- 音量: 振动的幅度
- 音色: 它与材质有很大的关系,本质是谐波

声波波形图

![img](https://box.kancloud.cn/26162cb264944191a445c9c9f163f1c2_675x470.png)

音色

![img](https://box.kancloud.cn/f7de15ceb1ec524b4ad42482cbb102f8_787x387.png)

听觉范围

![img](https://box.kancloud.cn/4cbf07e7871f453305e9f660ecb79737_913x100.png)

模拟信号转数字信号
音频量化过程
![img](https://box.kancloud.cn/5cd40810a1659da51fcc945251b9c6f7_544x540.png)

**量化基本概念**

- 采样大小: 一个采样用多少bit存放.常用的是16bit
- 采样率: 采样率8k, 16k, 32k, 44.1k, 48k
- 声道数: 单声道,双声道,多声道

**码率计算**
要计算一个PCM音频流的码率是一件很轻松的事情,`采样率*采样大小*声道数`
例如 : 采样率为44.1KHz,采样大小为16bit,双声道的PCM编码的WAV文件,它的码率为`44.1k*16*2=1411.2kb/s`

## 音频压缩

- 消除冗余数据
- 哈夫曼无损编码

## 音频的冗余信息

- 压缩的主要方法是去除采集到的音频冗余信息,所谓冗余信息包括人耳听觉范围外的音频信号以及被掩蔽掉的音频信号
- 信号的掩蔽可以分为频域掩蔽和时域掩蔽

## 音频编码过程

![img](https://box.kancloud.cn/36a082957c77842d3d5b59bedfe88bc8_991x238.png)

## 常见的音频编码器

- 常见的音频编码器包括OPUS,AAC,Vorbis,Speex,iLBC,AMR,G.711等
  OPUS>AAC>Vorbis

**AAC规格**

- `AAC LC`: 低复杂度,码流128k
- `AAC HE`: `AAC LC + SBR(分屏复用)`
- `AAC HE V2`: `AAC LC + SBR + PS(双声道)`

**AAC格式**

- ADIF这种格式只能从头开始解码,常用在磁盘文件中
- ADTS这种格式每一帧都有一个同步字,可以在音频流的任何位置开始解码,它类似于数据流格式

AAC编码库那个好?

```
Libfdk_AAC` > `ffmpeg AAC` > `libfaac` > `libvo_aacenc
```

# 视频

**H264基本概念**

- I帧: 关键帧,采用帧内压缩技术
- P帧: 向前参考帧,压缩时只参考前一个帧,属于帧间压缩技术
- B帧: 双向参考帧,压缩时即参考前一帧也参考后一帧,帧间压缩技术

**GOF**
一组帧,一个I帧和另一个I帧,或者一个P帧和另一个P帧

![img](https://box.kancloud.cn/6c600ed16bf05dfffa3ee7dbdeb72eca_778x301.png)

**SPS与PPS**

- SPS: 序列参数集,存放帧数,参考帧数目,解码图像尺寸,帧场编码模式选择标识等
- PPS: 图像参数集,存放熵编码模式选择标识,片组数目,初始化量化参数和去方法滤波系数调整标识等

## 视频花屏卡顿原因

- 如果GOP分组中的P帧丢失会造成解码端的图像发生错误
- 为了避免花屏问题的发生,一般如果发现P帧或者I帧丢失,就不显示本GOP内的所有帧,直到下一个I帧来后重新刷新图像

花屏是丢了数据
卡顿是为了不花屏把那一组数据丢了

## 常见哪些视频编码器

- `x264/x265`
- openH264
- vp8/vp9

**H264压缩技术**

- 帧内预测压缩,解决的是空域数据冗余问题
- 帧间预测压缩,解决的是时域数据冗余问题
- 整数离散余弦变换(DCT),将空间上的相关性变为频域上无关的数据然后进行量化
- CABAC压缩,无损压缩

**宏块划分与分组**

H264宏块划分

![img](https://box.kancloud.cn/849a6bdc2ec0c143ea4fb2171bf4cd91_753x293.png)

![img](https://box.kancloud.cn/2fbba9da5e6587711161d57fd998ea0d_533x358.png)

**子块划分**

![img](https://box.kancloud.cn/7598435173a9df52c108a34e25e30ef7_958x261.png)

**帧分组**
![img](https://box.kancloud.cn/900356b3d068a63108dc5e84556b23ee_515x189.png)

## 视频压缩

**组内宏块查找**

![img](https://box.kancloud.cn/cf27e3ca6f29baa2fd74011bb31d0306_488x210.png)

**运动估算**

![img](https://box.kancloud.cn/1254156f0c6fc6573e7c591b299ed5ab_859x222.png)

**运动矢量与补偿压缩**
![img](https://box.kancloud.cn/fbcad8c93764f81f03a2584b087f46c1_598x378.png)

**帧内预测**

![img](https://box.kancloud.cn/50f4746022bcbb09c4cd430aef49d3f0_1044x277.png)

计算帧内预测残差值

**DCT压缩**

**vlc压缩**

**CABAC压缩**

![img](https://box.kancloud.cn/d6ae414c262b516b6a116c4acaed84f6_693x344.png)

## H264结构图

![img](https://box.kancloud.cn/8a4e408e5249f34c213dcbd462946c61_644x413.png)

## H264编码分层

- NAL层: 视频数据网络抽象层
- VCL层: 视频数据编码层

**码流基本概念**

- SODB: 原始数据比特流,长度不一定是8的倍数,他是由vcl层产生的
- RBSP: 算法是SODB最后一位补1,不按字节对齐则补0
- EBSP: 需到两个连续的0x00就增加一个0x03
- NALU: `NAL Header(1B)+EBSP`

**切片和宏块的关系**

![img](https://box.kancloud.cn/dc806e0bfd101ab7e52289eb0b5df43e_751x389.png)

**H264切片**

![img](https://box.kancloud.cn/4e71bf54ea59a52419ae53dc7133d51b_572x409.png)

## YUV

**图像除了RGB还有YUV**

- YUV是电视系统采用的一种颜色编码方法
- Y表示明亮度,也就是灰阶值,它是基础信号
- U和V表示的则是色度,UV是作用是描述影响色彩以及饱和度,它们用于指定像素的颜色

**常见格式**

- YUV4:2:0 (YCbCr 4:2:0)
- YUV4:2:2(YCbcR 4:2:2)
- YUV4:4:4 (YCbCr 4:4:4)

**YUV存储格式**

- planar(平面)
- packed(打包)