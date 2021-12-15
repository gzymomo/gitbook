# Ffmpeg 播放rtsp流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512080559256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1bmRo,size_16,color_FFFFFF,t_70)





# Ffmpeg 选项说明

得到帮助：
-h：													输出基本选项
-h long：												输出更多选项
-h full：												输出完整的选项列表，包括编（解）码器，分离器混合器以及滤镜等等的共享和私有选项
-h type=name 											输出命名decoder/encoder/demuxer/muxer/filter/bsf/protocol的所有选项


输出 帮助 / 信息 / 能力：
-L：													显示授权协议
-h topic：												显示帮助
-? topic：												显示帮助
-help topic：											显示帮助
--help topic：											显示帮助
-version：												显示版本信息
-buildconf：											显示构建配置
-formats：												显示所有有效的格式（包括设备）
-muxers：												显示有效的混合器
-demuxers：												显示有限的分离器
-devices：												显示有效设备
-codecs：												显示所有已支持的编码（libavcodec中的）格式
-decoders：												显示所有有效解码器
-encoders：												显示所有有效的编码器
-bsfs：													显示有效的数据流（bitstream）滤镜
-protocols：											显示支持的协议
-filters：												显示libavfilter中的滤镜
-pix_fmts：												显示有效的像素（pixel）格式
-sample_fmts：											显示有效的实例格式
-layouts：												显示信道名字和信道布局
-colors：												显示注册的颜色名
-sources device：										显示自动识别的输入设备源
-sinks device：											显示自动识别的输出设备
-hwaccels：												显示可用的硬件加速方法


全局选项（影响整个项目，而不只是一个文件）：
-loglevel loglevel：									设置日志层次
-v loglevel：											设置日志层次
-report：												生成报告
-max_alloc bytes：										设置单个分配块的最大大小
-y：													覆盖输出文件
-n：													永远不会覆盖输出文件
-ignore_unknown：										忽略未知的流类型
-filter_threads：										非复杂过滤器线程数
-filter_complex_threads：								复杂过滤器线程数
-stats：												在编码过程中输出进度报告
-max_error_rate maximum error rate：					错误率（0.0：无错误，1.0：100％错误），高于该比率ffmpeg将返回错误而不是成功
-bits_per_raw_sample number：							设置每个原始样本的位数
-vol volume：											更改音量（256=正常）


每个文件的主要选项：
-f fmt：												强制格式
-c codec：												编解码器名称
-codec codec：											编解码器名称
-pre preset：											预设名称
-map_metadata outfile[,metadata]:infile[,metadata]：	从输入文件设置输出文件的元数据信息
-t duration：											音频/视频录制或转码的持续秒数
-to time_stop：											录制或转码停止时间
-fs limit_size：										设置限制文件大小（以字节为单位）
-ss time_off：											设置开始时间偏移
-sseof time_off：										设置相对于EOF的开始时间偏移
-seek_timestamp：										使用-ss按时间戳启用/禁用查找
-timestamp time：										设置录制时间戳（now设置当前时间）
-metadata string=string：								添加元数据
-program title=string:st=number...：					添加具有指定流的项目
-target type：											指定目标文件类型（带有可选前缀"pal-","ntsc-"或"film-"的"vcd","svcd","dvd","dv"或"dv50"）
-apad：													音垫
-frames number：										设置要输出的帧数
-filter filter_graph：									设置流过滤图
-filter_script filename：								从文件中读取流过滤图描述
-reinit_filter：										输入参数更改时重新初始化过滤图
-discard：												丢弃
-disposition：											配置情况


视频选项：
-vframes number：										设置要输出的视频帧数
-r rate：												设置帧率（Hz值，分数或缩写）
-s size：												设置帧大小（WxH或缩写）
-aspect aspect：										设置长宽比（4：3、16：9或1.3333、1.7777）
-bits_per_raw_sample number：							设置每个原始样本的位数
-vn：													禁止输出视频
-vcodec codec：											强制视频编解码器（copy以复制流）
-timecode hh:mm:ss[:;.]ff：								设置初始时间码值
-pass n：												选择通行编号（1至3）
-vf filter_graph：										设置视频过滤器
-ab bitrate：											音频比特率（请使用-b：a）
-b bitrate：											视频比特率（请使用-b：v）
-dn：													禁用数据


音频选项：
-aframes number：										设置要输出的音频帧数
-aq quality：											设置音频质量（特定于编解码器）
-ar rate：												设置音频采样率（以Hz为单位）
-ac channels：											设置音频通道数
-an：													禁止输出音频
-acode codec：											强制音频编解码器（copy以复制流）
-vol volume：											更改音量（256=正常）
-af filter_graph：										设置音频过滤器


字幕选项：
-s size：												设置帧大小（WxH或缩写）
-sn：													禁用字幕
-scodec codec：											强制字幕编解码器（copy以复制流）
-stag fourcc/tag：										强制字幕标签
-fix_sub_duration：										修正字幕的持续时间
-canvas_size size：										设置画布大小（WxH或缩写）
-spre preset：											将字幕选项设置为指示的预设