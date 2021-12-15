# WebRTC

WebRTC 是一整套 API，其中一部分供 Web 开发者使用，另外一部分属于要求浏览器厂商实现的接口规范。WebRTC 解决诸如客户端流媒体发送、点对点通信、视频编码等问题。桌面浏览器对WebRTC的支持较好，WebRTC 也很容易和 Native 应用集成。

使用 MSE 时，你需要自己构建视频流。使用 WebRTC 时则可以直接捕获客户端视频流。

使用 WebRTC 时，大部分情况下流量不需要依赖于服务器中转，服务器的作用主要是：

1. 在信号处理时，转发客户端的数据
2. 配合实现 NAT/防火墙 穿透
3. 在点对点通信失败时，作为中继器使用

## 1. 架构图

![](https://pic2.zhimg.com/80/v2-d912c506fa72bf665c76654390322be9_720w.jpg)

## 2  流捕获

### 2.1 捕获视频

主要是捕获客户端摄像头、麦克风。在视频监控领域用处不大，这里大概了解一下。流捕获通过 navigator.getUserMedia 调用实现：

```javascript
<script type="text/javascript">
 navigator.getUserMedia = navigator.webkitGetUserMedia || navigator.getUserMedia;
 var success = function (stream) {
   var video = document.getElementById('camrea');
   // 把MediaStream对象转换为Blob URL，提供给video播放
   video.src = URL.createObjectURL( stream );
   video.play();
 }
 var error = function ( err ) {
   console.log( err )
 }

 // 调用成功后，得到MediaStream对象
 navigator.getUserMedia( { video: true, audio: true }, success, error );
</script>
<video id="camrea" width="640" height="480"/>
```

三个调用参数分别是：

1. 约束条件，你可以指定媒体类型、分辨率、帧率
2. 成功后的回调，你可以在回调中解析出 URL 提供给 video 元素播放
3. 失败后的回调

### 2.2 捕获音频

捕获音频类似：

```javascript
navigator.getUserMedia( { audio: true }, function ( stream ) {
    var audioContext = new AudioContext();

    // 从捕获的音频流创建一个媒体源管理
    var streamSource = audioContext.createMediaStreamSource( stream );

    // 把媒体源连接到目标（默认是扬声器）
    streamSource.connect( audioContext.destination );
}, error );
```

### 2.3 MediaStream

MediaStream对象提供以下方法：

1. getAudioTracks()，音轨列表
2. getVideoTracks()，视轨列表

每个音轨、视轨都有个label属性，对应其设备名称。

### 2.4 Camera.js

Camera.js 是对 getUserMedia 的简单封装，简化了API 并提供了跨浏览器支持：

```javascript
camera.init( {
    width: 640,
    height: 480,
    fps: 30, // 帧率
    mirror: false,  // 是否显示为镜像
    targetCanvas: document.getElementById( 'webcam' ), // 默认null，如果设置了则在画布中渲染

    onFrame: function ( canvas ) {
        // 每当新的帧被捕获，调用此回调
    },

    onSuccess: function () {
        // 流成功获取后
    },

    onError: function ( error ) {
        // 如果初始化失败
    },

    onNotSupported: function () {
        // 当浏览器不支持camera.js时
    }
} );
// 暂停
camera.pause();
// 恢复
camera.start();
```

## 3 信号处理

在端点之间（Peer）发送流之前，需要进行通信协调、发送控制消息，即所谓信号处理（Signaling），信号处理牵涉到三类信息：

会话控制信息：初始化、关闭通信，报告错误
网络配置：对于其它端点来说，本机的 IP 和 port 是什么
媒体特性：本机能够处理什么音视频编码、多高的分辨率。本机发送什么样的音视频编码
WebRTC 没有对信号处理规定太多，我们可以通过 Ajax/WebSocket 通信，以 SIP、Jingle、ISUP 等协议完成信号处理。点对点连接设立后，流的传输并不需要服务器介入。信号处理的示意图如下：
![](https://pic1.zhimg.com/80/v2-a87065bc24c3a4857416703efb0f160c_720w.jpg)

### 3.1 示例代码

下面的代表片段包含了一个视频电话的信号处理过程：

```javascript
// 信号处理通道，底层传输方式和协议自定义
var signalingChannel = createSignalingChannel();
var conn;

// 信号通过此回调送达本地，可能分多次送达
signalingChannel.onmessage = function ( evt ) {
    if ( !conn ) start( false );

    var signal = JSON.parse( evt.data );
    // 会话描述协议（Session Description Protocol），用于交换媒体配置信息（分辨率、编解码能力）
    if ( signal.sdp )
    // 设置Peer的RTCSessionDescription
        conn.setRemoteDescription( new RTCSessionDescription( signal.sdp ) );
    else
    // 添加Peer的Candidate信息
        conn.addIceCandidate( new RTCIceCandidate( signal.candidate ) );
};

// 调用此方法启动WebRTC，获取本地流并显示，侦听连接上的事件并处理
function start( isCaller ) {
    conn = new RTCPeerConnection( { /**/ } );

    // 把地址/端口信息发送给其它Peer。所谓Candidate就是基于ICE框架获得的本机可用地址/端口
    conn.onicecandidate = function ( evt ) {
        signalingChannel.send( JSON.stringify( { "candidate": evt.candidate } ) );
    };

    // 当远程流到达后，在remoteView元素中显示
    conn.onaddstream = function ( evt ) {
        remoteView.src = URL.createObjectURL( evt.stream );
    };

    // 获得本地流
    navigator.getUserMedia( { "audio": true, "video": true }, function ( stream ) {
        // 在remoteView元素中显示
        localView.src = URL.createObjectURL( stream );
        // 添加本地流，Peer将接收到onaddstream事件
        conn.addStream( stream );

        if ( isCaller )
        // 获得本地的RTCSessionDescription
            conn.createOffer( gotDescription );
        else
        // 针对Peer的RTCSessionDescription生成兼容的本地SDP
            conn.createAnswer( conn.remoteDescription, gotDescription );

        function gotDescription( desc ) {
            // 设置自己的RTCSessionDescription
            conn.setLocalDescription( desc );
            // 把自己的RTCSessionDescription发送给Peer
            signalingChannel.send( JSON.stringify( { "sdp": desc } ) );
        }
    } );
}

// 通信发起方调用：
start( true );
```

## 4 流转发

主要牵涉到的接口是RTCPeerConnection，上面的例子中已经包含了此接口的用法。WebRTC在底层做很多复杂的工作，这些工作对于JavaScript来说是透明的：

1. 执行解码
2. 屏蔽丢包的影响
3. 点对点通信：WebRTC 引入流交互式连接建立（Interactive Connectivity Establishment，ICE）框架。ICE 负责建立点对点链路的建立：
   -  首先尝试直接
   -  不行的话尝试 STUN（Session Traversal Utilities for NAT）协议。此协议通过一个简单的保活机制确保NAT端口映射在会话期间有效
   -  仍然不行尝试 TURN（Traversal Using Relays around NAT）协议。此协议依赖于部署在公网上的中继服务器。只要端点可以访问TURN服务器就可以建立连接
4. 通信安全
5. 带宽适配
6. 噪声抑制
7. 动态抖动缓冲（Dynamic jitter buffering），抖动是由于网络状况的变化，缓冲用于收集、存储数据，定期发送

## 5 任意数据交换

通过 RTCDataChannel 完成，允许点对点之间任意的数据交换。RTCPeerConnection 连接创建后，不但可以传输音视频流，还可以打开多个信道（RTCDataChannel）进行任意数据的交换。RTCDataChanel 的特点是：

1. 类似于 WebSocket 的API
2. 支持带优先级的多通道
3. 超低延迟，因为不需要通过服务器中转
4. 支持可靠/不可靠传输语义。支持 SCTP、DTLS、UDP 几种传输协议
5. 内置安全传输（DTLS）
6. 内置拥塞控制
   使用 RTCDataChannel 可以很好的支持游戏、远程桌面、实时文本聊天、文件传输、去中心化网络等业务场景。

## 6  WebRTC框架

1. PeerJS ：简化 WebRTC 的点对点通信、视频、音频调用，提供云端的 PeerServer，你也可以自己搭建服务器
2. Sharefest：基于 Web 的 P2P 文件共享
3. webRTC.io：WebRTC 的一个抽象层，同时提供了客户端、服务器端 Node.js 组件。服务器端组件抽象了 STUN。类似的框架还有 SimpleWebRTC、easyrtc
4. OpenWebRTC：允许你构建能够和遵循 WebRTC 标准的浏览器进行通信的 Native 应用程序，支持Java绑定
5. NextRTC：基于 Java 实现的 WebRTC 信号处理服务器
6. Janus：这是一个 WebRTC 网关，纯服务器端组件，目前仅仅支持 Linux 环境下安装。Janus本身实现了到浏览器的 WebRTC 连接机制，支持以JSON格式交换数据，支持在服务器端应用逻辑 - 浏览器之间中继 RTP/RTCP 和消息。特殊化的功能有服务器端插件完成。官网地址：https://janus.conf.meetecho.com
7. Kurento：这是一个开源的 WebRTC 媒体服务器