- [WebRTC 系列之音频会话管理](https://www.cnblogs.com/wangyiyunxin/p/14416300.html)

- [WebRTC 音视频同步原理与实现](https://www.cnblogs.com/VideoCloudTech/p/14500271.html)



WebRTC（Web Real-Time Communication）是一个支持网页浏览器进行实时语音对话或视频对话的 API。W3C 和 IETF 在2021年1月26日共同宣布 WebRTC 1.0 定稿，促使 WebRTC  从事实上的互联网通信标准成为了官方标准，其在不同场景的应用将得到更为广泛的普及。

WebRTC  提供了视频会议的核心技术，包括音视频的采集、编解码、网络传输、显示等功能，并且还支持跨平台：Windows，Mac，iOS，Android。本文主要介绍 WebRTC 其 iOS 平台的音频会话 AVAudioSession。关于 WebRTC 往期相关技术分享，在文末有集合，也欢迎持续关注~

# 概念介绍

iOS 音频会话 AVAudioSession 是每一个进行 iOS 音频开发的开发者必须了解的基本概念。音频会话在操作系统 iOS、tvOS、watchOS 中是一项托管服务，系统通过音频会话在应用程序内、应用程序间和设备间管理音频行为。

我们可以使用音频会话来与系统交流，计划如何在应用程序中使用音频。此时，音频会话充当应用程序与操作系统之间的中介，进而充当基础音频硬件之间的中介。我们可以使用它向操作系统传达应用程序音频的性质，而无需详细说明特定行为或与音频硬件的必要交互。将这些细节的管理委派给音频会话，可以确保对用户的音频体验进行最佳管理。

下图出自[《Audio Session Programming Guide》](https://developer.apple.com/library/archive/documentation/Audio/Conceptual/AudioSessionProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007875-CH1-SW1)，从图中可以看到 AVAudioSession 就是用来管理多个 APP 对音频硬件设备的资源使用。

![AVAudioSession](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219151708450-1360060738.png)

# 音频会话的能力

iOS 的音频会话能力主要分为以下几种情况：

- 配置音频会话
- 激活音频会话
- 响应中断
- 响应路由更改
- 配置设备硬件
- 保护用户隐私

具体的详细说明可以参考官网：[《Audio Session Programming Guide》](https://netease-we.feishu.cn/docs/doccnnoxrJ2Oj6VNggPXFV4i93e#//apple_ref/doc/uid/TP40007875-CH1-SW1)，这里就不再全盘细说，本文将主要分析配置音频会话、配置设备硬件两方面的技术细节以及分享实际开发过程中踩过的坑。

# 配置音频会话

AVAudioSession 的 Category、CategoryOption、Mode 配合使用，不同的应用类型或者使用场景需要搭配不同的组合。

Category 主要有以下7种类型，其主要描述以及特点在表中详细介绍：

![Category 的7种类型](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219151750352-1548709257.png)

CategoryOption 主要有以下7种类型：

![CategoryOption 的7种类型](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219151827649-1693016703.png)

下图详细列出了 Category、CategoryOption、Mode 配合使用情况：

![Category、CategoryOption、Mode 配合使用情况](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219151853089-2020853749.png)

## 分类表达音频角色

表达音频行为的主要机制是使用音频会话类别。通过设置类别，我们可以指示应用程序是使用音频输入还是音频输出，例如是否需要麦克风的采集、是否需要扬声器的播放等等。下文主要介绍音频会话类别 Category 在不同场景的应用，针对不同的 Category 的区别，可以对应上文表一详细查看。

### 游戏应用的场景

大多数游戏都需要用户交互发生在游戏中。用户调出另一个应用程序或锁定屏幕时，他们不希望该应用程序继续播放。设计游戏时，可以使用  AVAudioSessionCategoryAmbient 或 AVAudioSessionCategorySoloAmbient 类别。

### 用户控制的播放和录制应用程序的场景

录制应用程序和播放应用程序具有相似的准则。这些类型的应用程序的使用  AVAudioSessionCategoryRecord，AVAudioSessionCategoryPlayAndRecord 或  AVAudioSessionCategoryPlayback 类别。

### VoIP 和聊天应用程序的场景

VoIP 和聊天应用程序要求输入和输出路由均可用。这些类型的应用程序使用 AVAudioSessionCategoryPlayAndRecord 类别，并且不会与其他应用程序混合使用。

### 计量应用的场景

计量应用程序需要应用到输入和输出路径的系统提供的信号处理量最少。设置 AVAudioSessionCategoryPlayAndRecord 类别和测量模式以最小化信号处理。此外，此类型的应用程序不能与其他应用程序混合使用。

### 播放音频类似浏览器的应用程序场景

社交媒体或其他类似浏览器的应用程序经常播放短视频。他们使用 AVAudioSessionCategoryPlayback 类别，并且不服从铃声开关。这些应用程序也不会与其他应用程序混合使用。

### 导航和健身应用程序的场景

导航和锻炼应用程序使用 AVAudioSessionCategoryPlayback 或  AVAudioSessionCategoryPlayAndRecord  类别。这些应用的音频通常包含简短的语音提示。播放时，这些提示会中断口语音频（例如播客或有声书），并与其他音频（例如从“音乐”应用中播放）混音。

### 合作音乐应用的场景

合作音乐应用程序旨在播放其他应用程序时播放。这些类型的应用程序使用 AVAudioSessionCategoryPlayback 或 AVAudioSessionCategoryPlayAndRecord 类别，并与其他应用程序混合使用。

## 网易云信使用情况

表中为网易云信在不同的使用场景下，Category、Options、Mode 的配置情况：

![网易云信在不同的使用场景下 Category、Options、Mode 配置情况](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219152004053-461683382.png)

针对上表的内容，我们也整理了一些常见的问题：

- 问题一：相信了解这块的小伙伴们肯定会有疑问，为什么 NERTC 实时音视频 SDK 默认不带 DefaultToSpeaker 选项吗？
- 答：其实原来我们使用听筒和扬声器切换都是使用 AVAudioSessionCategoryOptions  带AVAudioSessionCategoryOptionDefaultToSpeaker 和  不带AVAudioSessionCategoryOptionDefaultToSpeaker 操作的；当 APP  为扬声器时，AVAudioSessionCategoryOptions 带  AVAudioSessionCategoryOptionDefaultToSpeaker；而 callkit  本身切换听筒和扬声器应该使用的输出路由的变更 overrideOutputAudioPort: 操作的，因此切到听筒  AVAudioSessionPortOverrideNone 时，由于 category 和 categoryOptions  都没有变化，因此无法切换。最后 SDK 内部 AVAudioSessionCategoryOptions 将不再携带  AVAudioSessionCategoryOptionDefaultToSpeaker；同时扬声器和听筒切换都改为  overrideOutputAudioPort: 操作。
- 问题二：AVAudioSessionPortOverrideSpeaker 和AVAudioSessionCategoryOptionDefaultToSpeaker 之间的区别是什么呢？
- 答：区别在于，AVAudioSessionPortOverride 通过调用  overrideOutputAudioPort:，设置的时间要比使用 category 选项  AVAudioSessionCategoryOptionDefaultToSpeaker  更短暂。调用overrideOutputAudioPort:和设置 AVAudioSessionPortOverride 到  AVAudioSessionPortOverrideSpeaker  是暂时压倒一切的输出将其路由到扬声器的方式。遵循后进制胜规则，任何路线更改或中断都将导致音频被路由回到其正常路线。考虑使用overrideOutputAudioPort:  可能用于实现免提电话按钮的方式，在该按钮上您希望能够在扬声器（AVAudioSessionPortOverrideSpeaker）和正常输出路线（AVAudioSessionPortOverrideNone）之间切换。AVAudioSessionCategoryOptionDefaultToSpeaker 修改 AVAudioSessionCategoryPlayAndRecord  类别的路由行为，以便在不使用其他附件（例如耳机）的情况下，音频将始终路由到扬声器，而不是接收器。使用时AVAudioSessionCategoryOptionDefaultToSpeaker，将尊重用户的手势。例如，插入耳机将导致路由更改为耳机麦克风/耳机，拔出耳机将导致路由更改为内置麦克风/扬声器（与内置麦克风/接收器相反）被设置。
- 问题三：为什么 NERTC 实时音视频 SDK 会有2种模式？
- 答：因为 VoIP 情况下会使用AVAudioSessionModeVoiceChat模式，在 RemoteIO 情况下，会使用AVAudioSessionModeDerfault模式。
- 问题四：在AVAudioSessionCategoryPlayAndRecord类别下，Mode 有哪些隐藏含义？
- 答：设置AVAudioSessionModeVoiceChat模式将启用AVAudioSession类别选项AVAudioSessionCategoryOptionAllowBluetooth，从而进一步修改类别的行为，AVAudioSessionCategoryPlayAndRecord以允许将配对的蓝牙免提配置文件（HFP）设备用于输入和输出。设置AVAudioSessionModeVideoChat模式将AVAudioSessionCategoryPlayAndRecord通过设置AVAudioSessionCategoryOptionAllowBluetooth选项和AVAudioSessionCategoryOptionDefaultToSpeaker选项,进一步修改类别的行为。

## 网易云音乐使用情况

正常音乐软件使用 AVAudioSessionCategoryPlayback  类别，在云音乐新上线的“一起听”功能模块会使用实时音视频，因此使用 AVAudioSessionCategoryPlayAndRecord  类别。因为音乐软件对音质要求比较高，所以在蓝牙情况下，会使用 A2DP 模式，使用  AVAudioSessionCategoryOptionAllowBluetoothA2DP。

# 配置设备硬件

使用音频会话属性，可以在运行时针对设备硬件优化应用程序的音频行为。这样做可以使我们的代码适应正在运行设备的特性，以及用户在应用程序运行时所做的更改（例如插入耳机或将设备对接）。

我们在配置设备硬件 ，可以通过 AVAudioSession 实现相应的属性配置：

- 为采样率和 I / O 缓冲区持续时间指定首选的硬件设置。
- 查询许多硬件特性，例如输入和输出延迟，输入和输出通道数，硬件采样率，硬件音量设置以及音频输入的可用性。

想要配置设备硬件，首先需要了解清楚硬件的详细情况，具体可以看下面的2张图，自行测试和查阅文档得出。

**iPhone 硬件详情：**

![iPhone 硬件详情](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219152034206-930678805.png)

**iPad 硬件详情：**

![iPad 硬件详情](https://img2020.cnblogs.com/blog/855570/202102/855570-20210219152050474-86563522.png)

# 问题排查

## 音频可用性问题

音频可用性问题通常指音频的采集和播放是否正常工作，那么会有哪些因素会影响音频的可用性呢？

- 设备权限：无麦克风权限、没有配置音频后台权限。
- 被其他声音抢占，如微信通话、打电话中断、siri 中断等。
- 用户行为：接口调用 mute，修改 AVAudioSession 的 Category 等。
- 机器故障：硬件启动失败。

为了应对音频问题的排查，我们新增了音频事件上报和音频回路检测功能。

## iOS 音频事件上报类型

这里我们罗列几种常见的 iOS 音频事件上报的具体类型：

- 音频输入设备变更事件：上报设备名，如【 麦克风 (BuiltInMic)、普通耳机 (HeadsetMic)、蓝牙耳机 (BluetoothHFP)（一般指HFP）】
  - 示例： {"InputDeviceChange":"BuiltInMic"}
- 音频输出设备变更事件：上报设备名，如【 扬声器 (BuiltInSpeaker)、听筒 (BuiltInReceiver)、普通耳机  (Headphones)、蓝牙耳机 （BluetoothHFP）、蓝牙耳机 （BluetoothLE）、蓝牙耳机  （BluetoothA2DP）】
  - 示例：{"OutputDeviceChange":"BuiltInSpeaker"}
- 音频采集采样率变更事件：上报当前采集采样率
  - 示例：{"RecordSampleRateChange":"48000"}
- 音频播放采样率变更事件：上报当前播放采样率
  - 示例：{"PlayoutSampleRateChange":"48000"}
- 音频设备异常状态变更事件：上报当前异常状态
  - 示例：{"InitRecordingErr\StartRecordingErr\StopRecordingErr...":"-108"}
- 音频系统音量变更事件：上报当前系统音量
  - 示例：{"SystemVolumeChange":"70"}
- 音频播放故障检测变更事件：上报当前播放故障次数
  - 示例：{"PlayoutGlitch":"3"}
- 音频会话相关变更事件有以下8种情况：
  - 音频打断开始事件  audioInterruptionBegin 0
  - 音频打断结束事件  audioInterruptionEnd 1
  - 音频媒体服务丢失事件  audioMediaServicesWereLost  2
  - 音频媒体服务重置事件 audioMediaServicesWereReset  3
  - 沉默辅助音频提示通知开始事件 audioSilenceSecondaryAudioHintBegin  4
  - 沉默辅助音频提示通知结束事件 audioSilenceSecondaryAudioHintEnd 5
    - 示例：{"audioInterruptionBegin\audioInterruptionEnd...":"0"}
  - AVAudioSession 相关的 Category 6
    - 示例:  {"CategoryChange":"AVAudioSessionCategoryPlayAndRecord"}
  - AVAudioSession 相关的 CategoryOption 7
    - 示例:{"CategoryOptionChange":"37"}
  - AVAudioSession 相关的 Mode 8
    - 示例:{"ModeChange":"AVAudioSessionModeVoiceChat"}

## 音频回路检测

我们在使用过程中需要实时观察音频的回路工作状态，我们重点分析以下两种情况：

- 当我们需要准确了解音频回路的实际工作状态，那么我们可以通过采集音频和播放音频对应的采样率以及单位时间内的音频采样 Samples 偏差，同时也能了解到它向NetEQ（即：音频 Buffer） 索要音频播放数据的节奏。
- 当播放线程出现卡顿的时候，我们需要实时获取音频播放线程状态以及分析解码、混音等各个阶段的耗时，排查其他环节对于播放线程的影响。

# 未来展望

产品以及系统也在不断迭代升级，例如：

1. 麦克风的位置选择（Built-in 不可控，因为要完美处理回声问题）和麦克风的极性模式设置（极性模式定义了其对声音相对于声源方向的灵敏度）。
2. 从 iOS 14 和 iPadOS 14 开始，我们现在可以使用支持的设备上的内置麦克风来捕获立体声音频，从而获得非常有沉浸式的录音体验。

我们期望未来可以结合这些优势能力，在音频会话技术上深度挖掘，以提供更好的音频服务。

# 总结

本文介绍了基于 WebRTC 实现 iOS 音频会话的实现以及管理。一个完整的系统，需要有预警各种异常、处理各种异常情况的能力。由于移动端设备的复杂性，移动端音频预警，自行恢复机制是一个比较核心的技术。

熟练掌握 AVAudioSession 的 Category、CategoryOption、Mode 的各个含义和了解 iPhone、iPad的硬件构造，对于理解 iOS 音频至关重要，上文如有不正确之处，欢迎指出，也欢迎交流。

## 系列文章

- [基于 WebRTC 实现自定义编码分辨率发送](http://mp.weixin.qq.com/s?__biz=MzI1NTMwNDg3MQ==&mid=2247486179&idx=1&sn=d50a9d66d6761a3d38295e9bb6cd6cdd&chksm=ea36bf2bdd41363d5f4063e5c3409736ef0570f59dc3d27836c5c973e31575bb9dfcd547fbb4&scene=21#wechat_redirect)
- [WebRTC 系列之视频辅流](http://mp.weixin.qq.com/s?__biz=MzI1NTMwNDg3MQ==&mid=2247485640&idx=1&sn=d87ede70f10d32999d30907c1bb2448a&chksm=ea36bd00dd4134166479726e238aaf378a4f5281e5a543bcdc58cf3abc01d3f3f86405b5aacf&scene=21#wechat_redirect)
- [WebRTC系列之音频的那些事](http://mp.weixin.qq.com/s?__biz=MzI1NTMwNDg3MQ==&mid=2247484072&idx=1&sn=bb6390a6b1a8052de0e4c06ef88972ba&chksm=ea36b760dd413e7601e5584741c62c44d3e52e072177ae0d55afb7f6d25fa0dedd240ec8fa9a&scene=21#wechat_redirect)
- [从入门到进阶｜如何基于WebRTC搭建一个视频会议](http://mp.weixin.qq.com/s?__biz=MzI1NTMwNDg3MQ==&mid=2247483663&idx=1&sn=ae4655258aefee3cdf6a2e1ae2e3b652&chksm=ea36b4c7dd413dd189e130fa2f3966dd3d75972694f90800e8a15e3ceeed23930287118325be&scene=21#wechat_redirect)