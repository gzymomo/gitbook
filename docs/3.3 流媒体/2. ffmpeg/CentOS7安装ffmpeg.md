## [CentOs7.5安装FFmpeg](https://www.cnblogs.com/wintercloud/p/11162962.html)

## 一、FFmpeg简介

**FFmpeg**是一个自由软件，可以运行音频和视频多种格式的录影、转换、流功能，包含了libavcodec ─这是一个用于多个项目中音频和视频的解码器库，以及libavformat——一个音频与视频格式转换库

"FFmpeg"这个单词中的"FF"指的是"Fast Forward"。有些新手写信给"FFmpeg"的项目负责人，询问FF是不是代表“Fast Free”或者“Fast Fourier”等意思，"FFmpeg"的项目负责人回信说“Just for the record, the original meaning of "FF" in FFmpeg is "Fast Forward"...”

FFmpeg在Linux平台下开发，但它同样也可以在其它操作系统环境中编译运行，包括Windows、Mac OS X等。这个项目最初是由Fabrice Bellard发起的，而现在是由Michael Niedermayer在进行维护。许多FFmpeg的开发者同时也是MPlayer项目的成员，FFmpeg在MPlayer项目中是被设计为服务器版本进行开发。

2011年3月13日，FFmpeg部分开发人士决定另组libav，网址http://libav.org，同时制定了一套关于项目继续发展和维护的规则。

## 组成组件

此计划由几个组件组成：

- *ffmpeg*是一个命令行工具，用来对视频文件转换格式，也支持对电视卡实时编码
- *ffserver*是一个HTTP多媒体实时广播流服务器，支持时光平移
- *ffplay*是一个简单的播放器，基于SDL与FFmpeg库
- *libavcodec*包含了全部FFmpeg音频／视频编解码库
- *libavformat*包含demuxers和muxer库
- *libavutil*包含一些工具库
- *libpostproc*对于视频做前处理的库
- *libswscale*对于图像作缩放的库

## 参数

FFmpeg可使用众多参数，参数内容会根据ffmpeg版本而有差异，使用前建议先参考参数及编解码器的叙述。此外，参数明细可用`ffmpeg -h`显示；编解码器名称等明细可用`ffmpeg -formats`显示。

下列为较常使用的参数。

## 主要参数

- -i设置输入文件名。
- -f设置输出格式。
- -y若输出文件已存在时则覆盖文件。
- -fs超过指定的文件大小时则退出转换。
- -ss从指定时间开始转换。
- -title设置标题。
- -timestamp设置时间戳。
- -vsync增减Frame使影音同步。

## 图像参数

- -b设置图像流量，默认为200Kbit/秒。（*单位请引用下方注意事项*）
- -r设置帧率值，默认为25。
- -s设置画面的宽与高。
- -aspect设置画面的比例。
- -vn不处理图像，于仅针对声音做处理时使用。
- -vcodec设置图像图像编解码器，未设置时则使用与输入文件相同之编解码器。

## 声音参数

- -ab设置**每Channel**（最近的SVN版为所有Channel的总合）的流量。（*单位请引用下方注意事项*）
- -ar设置采样率。
- -ac设置声音的Channel数。
- -acodec设置声音编解码器，未设置时与图像相同，使用与输入文件相同之编解码器。
- -an不处理声音，于仅针对图像做处理时使用。
- -vol设置音量大小，256为标准音量。（要设置成两倍音量时则输入512，依此类推。）

## 注意事项

- 以-b及ab首选项流量时，根据使用的ffmpeg版本，须注意单位会有kbits/sec与bits/sec的不同。（可用ffmpeg -h显示说明来确认单位。）

- 以-acodec及-vcodec所指定的编解码器名称，会根据使用的ffmpeg版本而有所不同。例如使用AAC编解码器时，会有输入aac与libfaac的情况。此外，编解码器有分为仅供解码时使用与仅供编码时使用，因此一定要利用`ffmpeg -formats`确认输入的编解码器是否能运作。

## 二、CentOs7.5下安装FFmpeg

### 1.[官网下载linux版本的ffmpeg源码包 ffmpeg-4.1.tar.xz](https://johnvansickle.com/ffmpeg/release-source/)

(此步骤也可以使用git clone下载源码包，本质上是一样的 )

![img](https://img2018.cnblogs.com/blog/1486162/201907/1486162-20190710094424115-701045536.png)

### 2.使用xftp将源码包ffmpeg-4.1.tar.xz上传至linux主机（usr/local/ffmpeg目录；直接使用linux命令下载到linux也可以）

```
cd /usr/local/``mkdir ffmpeg  #在usr/local目录下创建ffmpeg目录
```

　![img](https://img2018.cnblogs.com/blog/1486162/201907/1486162-20190710103349557-1317694642.png)

### 3.解压源码包

```bash
cd /usr/local/ffmpeg

tar xvJf ffmpeg-4.1.tar.xz
```

### 4.切换到ffmpeg-4.1目录、安装gcc编译器

```bash
cd ffmpeg-4.1
yum install gcc #安装gcc编译器
yum install yasm #安装yasm编译器
```

### 5.输入如下命令/usr/local/ffmpeg为自己指定的安装目录

```bash
./configure --enable-shared --prefix=/usr/local/ffmpeg
```



如果出现如下错误信息：

```bash
If you think configure made a mistake, make sure you are using the latest
version from Git.  If the latest version fails, report the problem to the
ffmpeg-user@ffmpeg.org mailing list or IRC #ffmpeg on irc.freenode.net.
Include the log file "config.log" produced by configure as this will help
solve the problem.
```

则需要先安装yasm

步骤(如已安装 则跳过此步骤)：

①wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz #下载源码包

②tar zxvf yasm-1.3.0.tar.gz #解压

③cd yasm-1.3.0 #进入目录 
④./configure #配置 
⑤make && make install #编译安装

### 6.执行make（非常非常久.......）

```bash
make
```

### 7.执行make install（安装）

### 8.修改文件/etc/ld.so.conf

```bash
vim /etc/ld.so.conf
输入以下内容
include ld.so.conf.d/*.conf``/usr/local/ffmpeg/lib/
```

输入**ldconfig**使修改生效。

### 9.查看版本

```bash
/usr/local/ffmpeg/ffmpeg-4.1/ffmpeg -version
```

![img](https://img2018.cnblogs.com/blog/1486162/201907/1486162-20190710112655139-143012466.png)

### 10.配置环境变量

```bash
# vim /etc/profile
```

在最后PATH添加环境变量：

```bash
#set ffmpeg environment PATH=$PATH:/usr/local/ffmpeg/bin export PATH
source /etc/profile #使配置生效
```

### 11.查看环境变量是否配置成功

```bash
ffmpeg -version
```

![img](https://img2018.cnblogs.com/blog/1486162/201907/1486162-20190710113208809-336235091.png)

至此安装成功

## 三、错误记录

#### 3.1 [解决ffmpeg执行报错“ffmpeg: error while loading shared libraries: libavdevice.so.58: cannot open shared ...](https://my.oschina.net/u/4257767/blog/3325852)

找下这些文件在哪里

```bash
find /usr -name 'libavdevice.so.58'
```

应该都在这个目录

```bash
/usr/local/lib/
```

![img](https://oscimg.oschina.net/oscnet/2946817047649a128cca0cd2b4dcff58240.png)

 我们export出来：

```bash
export LD_LIBRARY_PATH=/usr/local/lib/
```

然后再尝试执行

```bash
/usr/local/bin/ffmpeg
```

![img](https://oscimg.oschina.net/oscnet/5b66da0984d1005e7ac5c2aa8b8014f8a28.png)

 

 问题解决