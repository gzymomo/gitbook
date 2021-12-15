



在集体挺进HTML5的时代，来讨论Adobe Flash相关的话题似乎有点过时，但现如今还是有很多的视频网站采用的是Flash播放器，播放的文件也依然还有很多是FLV格式，而且仅从一个文件格式的角度去了解和分析FLV应该也还说的过去的。FLV(Flash Video)是Adobe的一个免费开放的音视频格式。

整体上，FLV分为`Header`和`Body`两大块。

- Header： 记录FLV的类型，版本，当前文件类型等信息，这些信息可以让我们对当前FLV文件有个概括的了解。

- Body： FLV的Body是Flv的数据区域，这些是FLV的具体内容，因为FLV中的内容有多种，并可同时存在，因此，Body也不是一整块的数据，而是由更细分的块来组成，这个细分的块叫Tag。



![img](https:////upload-images.jianshu.io/upload_images/2354823-2b99f282403fe635.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)



先来一张图，这是《东风破》——周杰伦（[下载](http://ovjkwgfx6.bkt.clouddn.com/dongfengpo.flv)）的一个MV视频。我使用的是Binary Viewer的二进制查看工具。

![img](https:////upload-images.jianshu.io/upload_images/2354823-4f356b034806f4e5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

# 一、Header

头部分由一下几部分组成，Signature(3 Byte)+Version(1 Byte)+Flags(1 Bypte)+DataOffset(4 Byte)，共9字节。

##  1.signature

 46 4C 56 正是FLV这三个字符的ASCII编码，这个是固定标识，表示它是FLV文件。

##  2.version

 版本号0x01

##  3.Flags

 0x05，对应二进制00000101，前面一个1表示有音频数据，后面一个1表示有视频数据。

##  4.DataOffset

 此4字节共同组成一个无符号32位整数（使用大头序），表示文件从FLV Header开始到Flv Body的字节数，当前版本固定为9（0x00，0x00，0x00，0x09）

# 二、Body

## 1.Previous Tag Size

 这个比较好理解，就是前一个Tag的大小，这里同样存的是无符号32位整型数值。因为第一个Previous Tag Size是紧接着FLV Header的，因此，其值也是固定为0（0x00，0x00，0x00，0x00）。

## 2.TAG

 FLV中的TAG不止一种，当前版本共有3种类型组成：音频（audio），视频（video），脚本数据（script data），这三种类型会在Tag内进行标志区分。其中：Audio Tag是音频数据，Video Tag是视频数据，**Script Data存放的是关于FLV视频和音频的一些参数信息（亦称为Metadata Tag），通常该Tag会在FLV File Header后面作为第一个Tag出现，并且一个文件仅有一个Script Data Tag。**

为了在Tag内存放不同的数据，并且能够方便区分，每个Tag被定义了Tag Header和Tag Data两部分，他们的结构如下：

```undefined
-------------------------
|       Tag Header      |
-------------------------
|       Tag  Data       |
-------------------------
 
 
 
                -------------------------
                |       Tag Header      |
                -------------------------
                 /                    \
--------------------------------------------------------
| 08 | 00 | 00 | 18 | 00 | 00 | 00 | 00 | 00 | 00 | 00 |
--------------------------------------------------------
```

Tag Header由11字节组成：

- 第1字节type：标志当前Tag的类型，音频（0x08），视频（0x09），Script Data（0x12），除此之外，其他值非法；
- 第2-4字节tag data size：表示一个无符号24位整型数值，表示当前Tag Data的大小；
- 第5-7字节Timestreamp：无符号24位整型数值（UI24），当前Tag的时间戳（单位为ms），第一个Tag的时间戳总为0；
- 第8字节TimestampExtended：为时间戳的扩展字节，当前24位不够用时，该字节作为最高位，将时间戳扩展为32位无符号整数（UI32）
- 第9-11字节stream id：UI24类型，表示Stream ID，总是0

看一下上述实例，第一个TAG：

```bash
type=0x12=18，是一个Script Data。
tag data size=0x000125=293。长度为293。
timestreamp=0x000000。这里是scripts，所以为0
TimestampExtended =0x00。
stream id =0x000000
```

这里来找一下第一个TAG在哪里结束，Tag Header本身是11个字节，第2-4字节tag data size现在已经知道是293字节，合计第一个TAB长度是11+293=304也就是16进制的130。那么下一个TAG的Previous Tag Size应该就是0x 00 00 01 30，见下图红线

![img](https:////upload-images.jianshu.io/upload_images/2354823-87d4c84040556003.png?imageMogr2/auto-orient/strip|imageView2/2/w/853/format/webp)

可以在图片上数一下，第一行的12 00 01是3个，中间共18行，每行16字节，最后划红线00 00 01那行有13字节，合计是3+18*16+13=304，确认无误。

## 3.TAG DATA

 Tag Data由Tag Header标志后，就被分成音频，视频，Script Data三类

## 4.Script Data

 脚本Tag一般只有一个，是flv的第一个Tag，用于存放flv的信息，比如duration、audiodatarate、creator、width等。

首先介绍下脚本的数据类型。**所有数据都是以数据类型+（数据长度）+数据的格式出现的，数据类型占1byte，数据长度看数据类型是否存在，后面才是数据。**

![img](https:////upload-images.jianshu.io/upload_images/2354823-e1c3d8c78598b0f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/697/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/2354823-b6d7aaa2137a6bda.png?imageMogr2/auto-orient/strip|imageView2/2/w/690/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/2354823-c40c2c0325f6a29a.png?imageMogr2/auto-orient/strip|imageView2/2/w/691/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/2354823-4a5efb113cbb5857.png?imageMogr2/auto-orient/strip|imageView2/2/w/696/format/webp)

string类型会先用uint16 标识出数据长度

![img](https:////upload-images.jianshu.io/upload_images/2354823-1f008e13cfd0a414.png?imageMogr2/auto-orient/strip|imageView2/2/w/704/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/2354823-c7b38bdfcdd6400e.png?imageMogr2/auto-orient/strip|imageView2/2/w/716/format/webp)

一般来说，该Tag Data结构包含两个AMF包。AMF（Action Message Format）是Adobe设计的一种通用数据封装格式，在Adobe的很多产品中应用，简单来说，AMF将不同类型的数据用统一的格式来描述。第一个AMF包封装字符串类型数据，用来装入一个“onMetaData”标志，这个标志与Adobe的一些API调用有，在此不细述。第二个AMF包封装一个数组类型，这个数组中包含了音视频信息项的名称和值。具体说明如下，大家可以参照图片上的数据进行理解。

### (1)先看第一个AMF包，从02 00 0A 6F往后读

 **第一个域是Name。Name又是SCRIPTDATAVALUE类型**
 type = 0x 02 对照上表是SCRIPTDATASTRING
 **SCRIPTDATASTRING类型会先用uint16标识出数据长度**
 size = 0x 00 0A ，说明长度为10
 value=onMeta Data= 0x 6F 6E 4D 65 74 61 44 61 74 61对应的ASCII码正是onMetaData，见下图红线。

![img](https:////upload-images.jianshu.io/upload_images/2354823-ced5eaa30119ffcb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

### (2)然后看第二个AMF

 type = 0x 08 对照上表是数组，后面Length UI32，即4个字节为数组的个数
 size = 0x 00 00 00 0D = 13，说明数组长度为13，后面有13个SCRIPTDATAOBJECTPROPERTY。
 **对照上图，SCRIPTDATAOBJECTPROPERTY由PropertyName(SCRIPTDATASTRING类型)和PropertyData(SCRIPTDATAVAULE类型)组成**

### (3)第一个键值对

 **PropertyName 是SCRIPTDATASTRING类型**
 string length = 0x 00 08 说明长度为8
 string data= 0x 64 75 72 61 74 69 6F 6E，正是ASCII码duration

**PropertyData是一个SCRIPTDATAVAULE类型。用的是UI8标识type**
 type = 0x 00 是个double数值(8字节)
 value = 0x 40 73 A7 85 1E B8 51 EC

### (4)第二个键值对

 **PropertyName 是SCRIPTDATASTRING类型**
 string length = 0x 00 05 说明长度为5
 string data = 0x 77 69 64 74 68，正是ASCII码width

**PropertyData是一个SCRIPTDATAVAULE类型。用的是UI8标识type**
 type = 0x 00 是个double数值(8字节)
 value = 0x 40 76 00 00 00 00 00 00

### (5)第三个键值对

 PropertyName
 string length = 0x 00 06 长度为6
 string data = 68 65 69 64 74 68 即width

PropertyData
 type = 0x 00
 value = 0x 40 76 00 00 00 00 00 00

后面的属性同理

## 5.video tag



![img](https:////upload-images.jianshu.io/upload_images/2354823-7c04438f22c2ca25.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

type=0x09=9。这里应该是一个video。
 size=0x000030=48。长度为48。
 timestreamp=0x000000。
 TimestampExtended =0x00。
 stream id =0x000000

### (1)接着StreamID字段之后的就是VideoTAagHeader



![img](https:////upload-images.jianshu.io/upload_images/2354823-73a1bc703f8d5b2d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

特殊情况
 视频的格式(CodecID)是AVC（H.264）的话，VideoTagHeader会多出4个字节的信息，AVCPacketType 和CompositionTime。

**AVCPacketType 占1个字节**

|  值  |                             类型                             |
| :--: | :----------------------------------------------------------: |
|  0   |      AVCDecoderConfigurationRecord(AVC sequence header)      |
|  1   |                           AVC NALU                           |
|  2   | AVC end of sequence (lower level NALU sequence ender is not required or supported) |

AVCDecoderConfigurationRecord.包含着是H.264解码相关比较重要的sps和pps信息，再给AVC解码器送数据流之前一定要把sps和pps信息送出，否则的话解码器不能正常解码。而且在解码器stop之后再次start之前，如seek、快进快退状态切换等，都需要重新送一遍sps和pps的信息.AVCDecoderConfigurationRecord在FLV文件中一般情况也是出现1次，也就是第一个video tag.

**CompositionTime 占3个字节**

|       条件        |           值            |
| :---------------: | :---------------------: |
| AVCPacketType ==1 | Composition time offset |
| AVCPacketType !=1 |            0            |

### (2)第一个字节是0x 17，即

 FrameType= 0x01
 CodecID = 0x07
 因为codecID=7,所以后面有AVCPacketType1个字节=0，CompositionTime3个字节也是0。

### (3)我们看第一个video tag，也就是前面那张图。我们看到AVCPacketType =0。而后面三个字节也是0。说明这个tag记录的是AVCDecoderConfigurationRecord。包含sps和pps数据。

![img](https:////upload-images.jianshu.io/upload_images/2354823-ddd86e034c7362c7.png?imageMogr2/auto-orient/strip|imageView2/2/w/608/format/webp)


 下面为了复制截图方便，引用了[FLV 实例分析](https://blog.csdn.net/y_z_hyangmo/article/details/79219996)中的例子

![img](https:////upload-images.jianshu.io/upload_images/2354823-26246b518a02ab9a.png?imageMogr2/auto-orient/strip|imageView2/2/w/808/format/webp)


 configurationVersion = 0x01
 AVCProfileIndication = 0x4D (77) Main Profile
 profile_compatibiltity = 0x40
 AVCLevelIndication = 0x1F (31)
 第五六字节是0xFFE1 ,写成二进制格式 ‘1111 1111 1110 0001’b
 对应到AVCDecoderConfigurationRecord的语法定义
 lengthSizeMinusOne = ‘11’b (3) 也就是NALUintLength字段会是4个字节
 numOfSequenceParameterSets = ‘00001’b 有一个Sps结构
 接下来16bits 是 sequenceParameterSetLength = 0x0019 (25 bytes)
 下图选中部分就是Sps了

![img](https:////upload-images.jianshu.io/upload_images/2354823-a623d5f8eb7c7383.png?imageMogr2/auto-orient/strip|imageView2/2/w/834/format/webp)


 再往下：
 numOfPictureParamterSets = 0x01
 pictureParameterSetLength = 0x0004;
 下图选中的就是pps了

![img](https:////upload-images.jianshu.io/upload_images/2354823-c0acd07cd732251b.png?imageMogr2/auto-orient/strip|imageView2/2/w/799/format/webp)


 再后面四个字节是PreviousTagSize= 0x00 00 00 38 (56) 等于这个Tag的 DataSize + 11 == （45） + 11。



## 6.audio tag

 再下来又是一个新的FLVTAG了。
 11个字节的头部先取出来
 TagType = 8 (音频)
 DataSize =0x000009 ( 9bytes)

### (1)AudioTagHeader结构 0xAF

 SoundFormat(4bits) = 0x0A (10 == AAC)
 SoundRate(2bits) = ‘11’b (3 == 44kHz)
 SoundSize(1bit) =’1’b (1 == 16-bit)
 SoundType(1bit) = ‘1’b (1= Stereo)
 注： 虽然这里SoundRate, SoundSize SoundType 都是 1 。但是这些都是定值，AAC格式的时候，不看这里的值，可以忽略掉。具体的真是值应该从后面的数据从获取
 AACPacketType == 0x00 (AAC seqence header)
 所以AACPacketType后面的就是AudioSpecificConfig了 0x13 90 56
 AudioSpecificConfig.audioObjectType(5 bits) =  2 (AAC LC)
 AudioSpecificConfig.samplingFrequencyIndex（4 bits） = 7
 AudioSpecificConfig.channelConfiguration （4 bits）= 2
 AudioSpecificConfig.GASpecificConfig.frameLengthFlag （1 bit)  = 0
 AudioSpecificConfig.GASpecificConfig.dependsOnCoreCoder ： (1 bit)  = 0
 AudioSpecificConfig.GASpecificConfig.extensionFlag ： (1 bit)  = 1
 剩下的四个字节就是extensionflag3的相关内容，这块还没有研究过。
 到这一个audioTag结束。
 后面的0x00000014是这个AudioTag的长度 等于 20 = 9 + 11。

## 7.再后面又是一个新的TAG



![img](https:////upload-images.jianshu.io/upload_images/2354823-74c8c70f01582377.png?imageMogr2/auto-orient/strip|imageView2/2/w/821/format/webp)



从这前11个字节知道的是这是一个视频Tag. DataSize = 0x00099F （2463）timeStamp == 0；
 然后再往后看，是一个VideoTagHeader结构，可以得到的信息如下
 FrameType = 1 (是一个Key  Frame)
 CodecID= 7 （AVC）
 AVCPacketType = 1 ；是一个普通的AVC NALUint
 CompositionTime =  0x000043 (67)
 那么这个Key Frame包含多少个NALUint呢，我们再来一步步看下去吧。记得前面我们分析的吗？NALUint数据的开头的NALUintLength字段。由之前的分析可知。占四个字节，
 NALUintLength = 0x00000222 (546 Bytes) 说明第一NALUint的长度是546字节



![img](https:////upload-images.jianshu.io/upload_images/2354823-6d4368acdc40d39c.png?imageMogr2/auto-orient/strip|imageView2/2/w/839/format/webp)

选中的546字节

接着是第二个NALUintLength = 0x00 00 05 F3 (1523 bytes) ，图片太长，详见原文
 接下来又是一个NALUintLength = 0x00 00 01 0B （267 bytes） ，图略
 下来又是一个NALUintLenth = 0x00 00 00 32 （50 bytes） ，图略
 接下来还是一个NALUintLength = 0x00 00 00 34 (52 byte)，图略

好了，到这里我们第一个KeyFrame的所有NALUint都已经取出来了 ，TagDataSize (2463 Bytes)= 1 (FrameType + CodecID) + 1 (AVCPacketType) + 3 (CompositionTime) + 4 + 546 +4 + 1523 + 4 + 267 + 4 + 50 + 4 + 52  等式成立。



