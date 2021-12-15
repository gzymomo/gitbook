简书：[合肥懒皮](https://www.jianshu.com/u/50e1d98d51ac)：[H5直播系列九 flv.js源码之flv-demuxer.js](https://www.jianshu.com/p/f788033f9b2b)

# 一、解析头



```rust
// function parseChunks(chunk: ArrayBuffer, byteStart: number): number;
parseChunks(chunk, byteStart) {
    if (!this._onError || !this._onMediaInfo || 
    !this._onTrackMetadata || !this._onDataAvailable) {
        throw new IllegalStateException('Flv: onError & onMediaInfo
        & onTrackMetadata & onDataAvailable callback must be specified');
    }

    let offset = 0;
    let le = this._littleEndian;

    if (byteStart === 0) {  // buffer with FLV header
        if (chunk.byteLength > 13) {
            let probeData = FLVDemuxer.probe(chunk);
            offset = probeData.dataOffset;
        } else {
            return 0;
        }
    }
    ...
```

这里判断 > 13，其实就是FLV Header的9字节，加上Body中第一个Previous Tag Size的4字节。



```kotlin
static probe(buffer) {
    let data = new Uint8Array(buffer);
    let mismatch = {match: false};

    if (data[0] !== 0x46 || data[1] !== 0x4C ||
    data[2] !== 0x56 || data[3] !== 0x01) {
        return mismatch;
    }

    let hasAudio = ((data[4] & 4) >>> 2) !== 0;
    let hasVideo = (data[4] & 1) !== 0;
```

这里判断开头是不是46 4C 56 01，又去判断hasAudio,hasVideo，对照FLV Header很容易理解

> 一、Header
>  头部分由一下几部分组成，Signature(3 Byte)+Version(1 Byte)+Flags(1 Bypte)+DataOffset(4 Byte)，共9字节。
>  1.signature
>  46 4C 56 正是FLV这三个字符的ASCII编码，这个是固定标识，表示它是FLV文件。
>  2.version
>  版本号0x01
>  3.Flags
>  0x05，对应二进制00000101，前面一个1表示有音频数据，后面一个1表示有视频数据。
>  4.DataOffset
>  此4字节共同组成一个无符号32位整数（使用大头序），表示文件从FLV Header开始到Flv Body的字节数，当前版本固定为9（0x00，0x00，0x00，0x09）

parseChunks 后面的代码就是在不断解析 tag，flv把一段媒体数据称为 TAG，每个tag有不同的type，实际上真正用到的只有三种type，8、9、18 分别对应，音频、视频和Script Data。



```jsx
 if (tagType !== 8 && tagType !== 9 && tagType !== 18) {
    Log.w(this.TAG, `Unsupported tag type ${tagType}, skipped`);
    // consume the whole tag (skip it)
    offset += 11 + dataSize + 4;
    continue;
}
```

Tag Header由11字节组成，Previous Tag Size有4字节

> Tag Header由11字节组成：
>  第1字节type：标志当前Tag的类型，音频（0x08），视频（0x09），Script Data（0x12），除此之外，其他值非法；
>  第2-4字节tag data size：表示一个无符号24位整型数值，表示当前Tag Data的大小；
>  第5-7字节Timestreamp：无符号24位整型数值（UI24），当前Tag的时间戳（单位为ms），第一个Tag的时间戳总为0；
>  第8字节TimestampExtended：为时间戳的扩展字节，当前24位不够用时，该字节作为最高位，将时间戳扩展为32位无符号整数（UI32）
>  第9-11字节stream id：UI24类型，表示Stream ID，总是0

因为TimestampExtended的存在（24位若不够，扩展到32位），出现了下面这段代码：



```bash
let ts2 = v.getUint8(4);
let ts1 = v.getUint8(5);
let ts0 = v.getUint8(6);
let ts3 = v.getUint8(7);
let timestamp = ts0 | (ts1 << 8) | (ts2 << 16) | (ts3 << 24);
```

先取三个字节按照Big-Endian转换成整数再在高位放上第四个字节。

解析完了 tag header后面分别按照不同的 tag type调用不同的解析函数。



```kotlin
switch (tagType) {
    case 8:  // Audio
        this._parseAudioData(chunk, dataOffset, dataSize, timestamp);
        break;
    case 9:  // Video
        this._parseVideoData(chunk, dataOffset, dataSize, timestamp, byteStart + offset);
        break;
    case 18:  // ScriptDataObject
        this._parseScriptData(chunk, dataOffset, dataSize);
        break;
}
```

# 二、_parseVideoData



```kotlin
if (codecId !== 7) {
    this._onError(DemuxErrors.CODEC_UNSUPPORTED, 
    `Flv: Unsupported codec in video frame: ${codecId}`);
    return;
}

this._parseAVCVideoPacket(arrayBuffer, dataOffset + 1, 
dataSize - 1, tagTimestamp, tagPosition, frameType);
```

![img](https:////upload-images.jianshu.io/upload_images/2354823-4615b9ab19c6b69e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

image.png



codecID=7是AVC，也就是H264。简单粗暴地讲，不是H264的，直接抛出错误，表示不支持。后面必然要解析pps,sps了。

视频的格式(CodecID)是AVC（H.264）的话，VideoTagHeader会多出4个字节的信息，AVCPacketType 和CompositionTime。AVCPacketType 占1个字节，CompositionTime 占3个字节



```jsx
let le = this._littleEndian;
let v = new DataView(arrayBuffer, dataOffset, dataSize);

let packetType = v.getUint8(0);
let cts_unsigned = v.getUint32(0, !le) & 0x00FFFFFF;
let cts = (cts_unsigned << 8) >> 8;  // convert to 24-bit signed int
```

1.CompositionTime
 解释下 CTS的概念，CompositionTime，我们前面在tag header里拿到过一个 timestamp，这个在视频里对应于DTS，就是解码时间戳，而CTS实际上是一个offset，表示 PTS相对于DTS的偏移量，就是 PTS和DTS的差值。

这里有个坑，参考adobe的文档，这是CTS是个有符号的24位整数，SI24，就是说它有可能是个负数。因为负数的24位整型到32位负数转换的时候要手工处理高位的符号位和补码问题。



![img](https:////upload-images.jianshu.io/upload_images/2354823-ef4866ada5f4e85e.png?imageMogr2/auto-orient/strip|imageView2/2/w/436/format/webp)

image.png

1. 



```kotlin
if (packetType === 0) {  // AVCDecoderConfigurationRecord
    this._parseAVCDecoderConfigurationRecord(
    arrayBuffer, dataOffset + 4, dataSize - 4);
} else if (packetType === 1) {  // One or more Nalus
    this._parseAVCVideoData(arrayBuffer, dataOffset + 4,
    dataSize - 4, tagTimestamp, tagPosition, frameType, cts);
} else if (packetType === 2) {
    // empty, AVC end of sequence
} else {
    this._onError(DemuxErrors.FORMAT_ERROR,
    `Flv: Invalid video packet type ${packetType}`);
    return;
}
```

packetType有两种，0 表示 AVCDecoderConfigurationRecord，这个是H.264的视频信息头，包含了 sps 和 pps，AVCDecoderConfigurationRecord的格式不是flv定义的，而是264标准定义的，如果用ffmpeg去解码，这个结构可以直接放到 codec的extradata里送给ffmpeg去解释。

flv.js作者选择了自己来解析这个数据结构，也是迫不得已，因为JS环境下没有ffmpeg，解析这个结构主要是为了提取 sps和pps。虽然理论上sps允许有多个，但其实一般就一个。

pps的信息没什么用，所以作者只实现了sps的分析器，说明作者下了很大功夫去学习264的标准，其中的Golomb解码还是挺复杂的，能解对不容易，我在PC和手机平台都是用ffmpeg去解析的。SPS里面包括了视频分辨率，帧率，profile level等视频重要信息。



```go
for (let i = 0; i < ppsCount; i++) {
    let len = v.getUint16(offset, !le);  // pictureParameterSetLength
    offset += 2;

    if (len === 0) {
        continue;
    }

    // pps is useless for extracting video information
    offset += len;
}
```



```jsx
for (let i = 0; i < spsCount; i++) {
    let len = v.getUint16(offset, !le);  // sequenceParameterSetLength
    offset += 2;

    if (len === 0) {
        continue;
    }

    // Notice: Nalu without startcode header (00 00 00 01)
    let sps = new Uint8Array(arrayBuffer, dataOffset + offset, len);
    offset += len;

    let config = SPSParser.parseSPS(sps);
    if (i !== 0) {
        // ignore other sps's config
        continue;
    }
```

packetTtype 为 1 表示 NALU，NALU= network abstract layer unit，这是H.264的概念，网络抽象层数据单元，其实简单理解就是一帧视频数据。

NALU的头有两种标准，一种是用 00 00 00 01四个字节开头这叫 start code，另一个叫mp4风格以Big-endian的四字节size开头，flv用了后一种，而我们在H.264的裸流里常见的是前一种。



作者：合肥懒皮
链接：https://www.jianshu.com/p/f788033f9b2b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。