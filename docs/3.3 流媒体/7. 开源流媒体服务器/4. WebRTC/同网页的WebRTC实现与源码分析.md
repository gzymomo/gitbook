- [同网页的WebRTC实现与源码分析](https://www.cnblogs.com/yzy11235/p/11439933.html)

## 基本环境搭建

### 已有环境

- Mac OS 10 & Windows 10 & Ubuntu 18.04 （均实现，WebRTC支持跨平台）
- Chrome 76 & Firefox
- Webstorm IDE

### 搭建需要环境

- [Web Server for Chrome](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb)

### 下载源码

```shell
git clone https://github.com/googlecodelabs/webrtc-web
```

## getUserMedia

- 源码的Step01跑一下，浏览器获取前置摄像头就能成功，不展示具体效果了，看看源码和一些其他的应用

### 源码分析

- 源码项目所给的代码结构，多是如下图，所以常会看到`js/main.js` `css/main.css`这种src

![proj_structure](https://img2018.cnblogs.com/blog/1615451/201908/1615451-20190831175329286-2045542985.png)

- 分析源码关键调用部分

```html
<!-- core src code of index.html -->
<head>
  <title>Realtime communication with WebRTC</title>
  <link rel="stylesheet" href="css/main.css" />
</head>

<body>
  <h1>Realtime communication with WebRTC</h1>
<!-- add video and script element in this .html file -->
  <video autoplay playsinline></video>
  <script src="js/main.js"></script>
</body>
/* core src code of main.css */
body {
  font-family: sans-serif;
}

video {
  max-width: 100%;
  width: 800px;
}
```

- `html`  `css`作为标记型语言，了解其基本语法特征与调用（我是通过阅读[DOM Sripting](https://book.douban.com/subject/5436113/)的前三章后比较清楚的，阅读这部分还有一个好处是，把我不理解的简洁代码到页面奇幻效果的转化，推锅给了DOM和浏览器厂商～），上面的两个代码就不难理解，着重分析下面`js`代码

```js
'use strict';

// On this codelab, you will be streaming only video (video: true).
const mediaStreamConstraints = {
  video: true,
};

// Video element where stream will be placed.
const localVideo = document.querySelector('video');

// Local stream that will be reproduced on the video.
let localStream;

// Handles success by adding the MediaStream to the video element.
function gotLocalMediaStream(mediaStream) {
  localStream = mediaStream;
  localVideo.srcObject = mediaStream;
}

// Handles error by logging a message to the console with the error message.
function handleLocalMediaStreamError(error) {
  console.log('navigator.getUserMedia error: ', error);
}

// Initializes media stream.
navigator.mediaDevices.getUserMedia(mediaStreamConstraints)
  .then(gotLocalMediaStream).catch(handleLocalMediaStreamError);
```

- 在官方的tutorial中，对于代码第一行就有解释[ECMAScript 5 Strict Mode, JSON, and More](https://johnresig.com/blog/ecmascript-5-strict-mode-json-and-more/)，可以认为是一种语法、异常检查更严格的模式
- 对于第3行之后的代码部分，功能上可以看作
  - 一个constraint只读变量
  - `gotLocalMediaStream()` 处理视频流函数
  - `handleLocalMediaStreamError()` 异常处理函数
  - `getUserMedia()` 调用
- 看[MediaDevices.getUserMedia()](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)的API调用规则，也就大体明白了上面接近30行代码的架构

```js
navigator.mediaDevices.getUserMedia(constraints)
/* produces a MediaStream */
.then(function(stream) {
  /* use the stream */
})
.catch(function(err) {
  /* handle the error */
});
```

- 在看第14-18行，how to use the mediaStream?

  - 先看`mediaStream` 的相关[API](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)

  > The MediaStream interface represents a stream of media content. A  stream consists of several tracks such as video or audio tracks. Each  track is specified as an instance of MediaStreamTrack

  - 看代码，从整个`main.js` 文件中，我没有看出`let localStream` 有什么特殊的用途，这一行注释掉对网页也没有什么影响（也许在之后的源码中有用）

  - 但17行的代码就相当关键了（可以把这一行的代码注释看看是个什么效果～获取了媒体流，但是网页上没有视频显示）

  - 从第9行的`const localVideo = document.querySelector('video')` 说起

    - `const` 只读变量
    - [Document.querySelector()](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) 理解这个函数，需要对DOM有一些认识
    - DOM(Document Object Model)，既然是model就会有一定的逻辑表达形式，DOM文档的表示就是一棵家谱树
    - `querySelector(selectors)` 也正是基于树形数据结构，来对`document` 中的 `object` 进行深度优先的前序遍历，来获取`document` 中符合`selectors` 的`HTMLElement` 并返回

    > The matching is done using depth-first pre-order traversal of the  document's nodes starting with the first element in the document's  markup and iterating through sequential nodes by order of the number of  child nodes.

  - 17行的[HTMLMediaElement.srcObject](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/srcObject) 则是对`'video'` 流媒体的赋值，使页面显示video

### getUserMedia()++

- 在step01的demo里，前置摄像头的调用非常成功，但要刨根问底，step01中的代码并没有说明要调什么摄像头，什么类型的视频流（constraints里面只要求`video: true`）
- 在官方tutorial里面有Bonus points，回答理解这些问题来加深对`getUserMedia()` 的理解
- 由于不想把这篇博文写的太长，上面两个问题，都会在[基于浏览器的WebRTC的getUserMedia()相关补充](https://www.cnblogs.com/yzy11235/p/11439933.html)中补充说明

## RTCPeerConnection

- Let's move on to Step-02

### 源码分析

#### HTML

```html
<body>
  <h1>Realtime communication with WebRTC</h1>

  <video id="localVideo" autoplay playsinline></video>
  <video id="remoteVideo" autoplay playsinline></video>

  <div>
    <button id="startButton">Start</button>
    <button id="callButton">Call</button>
    <button id="hangupButton">Hang Up</button>
  </div>

  <script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>
  <script src="js/main.js"></script>
</body>
```

- 在HTML文档中`<head>` 以及调用`main.css` 的部分和Step-01相比几乎没有改变
- 新加入的`video` `button` `script` 其`id` & `src`命名都有很好的解释说明效果，在下文对`main.js` 的分析中，相关内容会有更清楚的解释
- 代码接近300行的样子，按页面的操作顺序，分析一下相关代码

#### 三个button

- 从三个`button` 开始，这是代码183-192行

```js
// Define and add behavior to buttons.

// Define action buttons.
const startButton = document.getElementById('startButton');
const callButton = document.getElementById('callButton');
const hangupButton = document.getElementById('hangupButton');

// Set up initial action buttons status: disable call and hangup.
callButton.disabled = true;
hangupButton.disabled = true;
```

- 上面的代码和`querySelector()`有类似功能，比较清晰
- 代码259-262行

```js
// Add click event handlers for buttons.
startButton.addEventListener('click', startAction);
callButton.addEventListener('click', callAction);
hangupButton.addEventListener('click', hangupAction);
```

- [EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)熟悉API之后，就来看看`startAction()` `callAction()` `hangupAction()` 这三个函数

#### startAction()

- 从`startAction` 开始

```js
// Handles start button action: creates local MediaStream.
function startAction() {
  startButton.disabled = true;
  navigator.mediaDevices.getUserMedia(mediaStreamConstraints)
    .then(gotLocalMediaStream).catch(handleLocalMediaStreamError);
  trace('Requesting local stream.');
}
```

- 一经页面开启`startButton` 只能click一次，之后获取`getUserMedia()`
- `mediaStreamConstraints()` 函数几乎没有变化，`gotLocalMediaStream()` & `handleLocalMediaStreamError()` 有些许变化，在19-43部分行

```js
// Define peer connections, streams and video elements.
const localVideo = document.getElementById('localVideo');
const remoteVideo = document.getElementById('remoteVideo');

let localStream;
let remoteStream;

// Define MediaStreams callbacks.

// Sets the MediaStream as the video element src.
function gotLocalMediaStream(mediaStream) {
  localVideo.srcObject = mediaStream;
  localStream = mediaStream;
  trace('Received local stream.');
  callButton.disabled = false;  // Enable call button.
}

// Handles error by logging a message to the console.
function handleLocalMediaStreamError(error) {
  trace(`navigator.getUserMedia error: ${error.toString()}.`);
}
```

- 代码的整体逻辑非常清晰，唯独新加入的`trace()` 函数比较新颖，看看

```js
// Logs an action (text) and the time when it happened on the console.
function trace(text) {
  text = text.trim();
  const now = (window.performance.now() / 1000).toFixed(3);

  console.log(now, text);
}
```

- 几个API调用[String.prototype.trim()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/Trim) [console.log()](https://developer.mozilla.org/en-US/docs/Web/API/Console/log#Difference_between_log()_and_dir()) 可以在各种浏览器的控制台查看（Chrome的是开发者工具的console），以及[performance.now()](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)，总体来说，这是一个控制台信息输出函数

#### callAction()

- 再看`callAction()` ，代码203-246行

```js
// Code from line 16-17
// Define initial start time of the call (defined as connection between peers).
let startTime = null;

// Code from line19-27
// Define peer connections
let localPeerConnection;
let remotePeerConnection;

// Handles call button action: creates peer connection.
function callAction() {
  callButton.disabled = true;	// disenable call button
  hangupButton.disabled = false;	// enable hangup button

  trace('Starting call.');
  startTime = window.performance.now();	// assign startTime with concrete time

  // Get local media stream tracks.
  const videoTracks = localStream.getVideoTracks();
  const audioTracks = localStream.getAudioTracks();
  if (videoTracks.length > 0) {
    trace(`Using video device: ${videoTracks[0].label}.`);
  }
  if (audioTracks.length > 0) {
    trace(`Using audio device: ${audioTracks[0].label}.`);
  }

  const servers = null;  // Allows for RTC server configuration.

  // Create peer connections and add behavior.
  localPeerConnection = new RTCPeerConnection(servers);
  trace('Created local peer connection object localPeerConnection.');

  localPeerConnection.addEventListener('icecandidate', handleConnection);
  localPeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);

  remotePeerConnection = new RTCPeerConnection(servers);
  trace('Created remote peer connection object remotePeerConnection.');

  remotePeerConnection.addEventListener('icecandidate', handleConnection);
  remotePeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);
  remotePeerConnection.addEventListener('addstream', gotRemoteMediaStream);

  // Add local stream to connection and create offer to connect.
  localPeerConnection.addStream(localStream);
  trace('Added local stream to localPeerConnection.');

  trace('localPeerConnection createOffer start.');
  localPeerConnection.createOffer(offerOptions)
    .then(createdOffer).catch(setSessionDescriptionError);
}
```

- 按上文代码的函数，从第28行开始，就是极其关键的`RTCPeerConnection`，解析下面所说的三个步骤，以建立连接时序展开

> Setting up a call between WebRTC peers involves three tasks:
>
> > - Create a RTCPeerConnection for each end of the call and, at each end, add the local stream from getUserMedia().
> > - Get and share network information: potential connection endpoints are known as ICE candidates.
> > - Get and share local and remote descriptions: metadata about local media in SDP format.

#### RTCPeerConnection关键部分——Local & Remote peer建立

- `getUserMedia()` 部分，不再赘述

```js
  let localPeerConnection;
  const servers = null;  // Allows for RTC server configuration. This is where you could specify STUN and TURN servers.

  // Create peer connections and add behavior.
  localPeerConnection = new RTCPeerConnection(servers);
  
  remotePeerConnection = new RTCPeerConnection(servers);
```

- 关于servers，官网tutorial给了一篇说明[WebRTC in the real world: STUN, TURN and signaling](https://www.html5rocks.com/en/tutorials/webrtc/infrastructure/)，这个我也会在随着项目系统通信搭建的深入，学习实践到servers层面再记录
- 可以认为在上述代码之后，一个RTCPeerConnection的端就实例化成功了

```js
// Add local stream to connection and create offer to connect.
  localPeerConnection.addStream(localStream);
  trace('Added local stream to localPeerConnection.');
```

- 在`addStream()` 之后，可以认为Local & Remote Peer已经全部建好（RTCPeerConnection实例化成功，media传输也可以开始进行）

#### RTCPeerConnection关键部分——ICE candidate建立

```js
  localPeerConnection.addEventListener('icecandidate', handleConnection);
  localPeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);
```

- `addEventListener()` method在button相关中已经了解，关于`'icecandidate'` Event，看[RTCPeerConnection: icecandidate event](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/icecandidate_event)，而其中的`setLocalDescription()`在下面一个section中有介绍
- 这一部分，需要对计算机网络有一些了解，以及对WebRTC signaling的过程烂熟于心，我初学是非常费解的，探索后其中内容解释在[WebRTC的RTCPeerConnection()原理探析](https://www.cnblogs.com/yzy11235/p/11439971.html)（链接中文章重原理，这篇重视基础的代码实现）
- 同样Remote Peer的建立也是类似

```js
  remotePeerConnection.addEventListener('icecandidate', handleConnection);
  remotePeerConnection.addEventListener(
    'iceconnectionstatechange', handleConnectionChange);
  remotePeerConnection.addEventListener('addstream', gotRemoteMediaStream);
```

- 继续看ICE candidate建立过程中用到的三个函数

```js
// Connects with new peer candidate.
function handleConnection(event) {
  const peerConnection = event.target;
  const iceCandidate = event.candidate;

  if (iceCandidate) {
    const newIceCandidate = new RTCIceCandidate(iceCandidate);
    const otherPeer = getOtherPeer(peerConnection);

    otherPeer.addIceCandidate(newIceCandidate)
      .then(() => {
        handleConnectionSuccess(peerConnection);
      }).catch((error) => {
        handleConnectionFailure(peerConnection, error);
      });

    trace(`${getPeerName(peerConnection)} ICE candidate:\n` +
          `${event.candidate.candidate}.`);
  }
}

// Logs changes to the connection state.
function handleConnectionChange(event) {
  const peerConnection = event.target;
  console.log('ICE state change event: ', event);
  trace(`${getPeerName(peerConnection)} ICE state: ` +
        `${peerConnection.iceConnectionState}.`);
}

// Handles remote MediaStream success by adding it as the remoteVideo src.
function gotRemoteMediaStream(event) {
  const mediaStream = event.stream;
  remoteVideo.srcObject = mediaStream;
  remoteStream = mediaStream;
  trace('Remote peer connection received remote stream.');
}
```

- 我猜测前两个函数，是针对于本机连本机的特殊应用搭建的，不具有普遍性，所以不具体分析
- `gotRemoteMediaStream()` 函数，最终将Local Peer的`addStream()` 显示
- 还有一个API值得看一下，就是[RTCPeerConnection.addIceCandidate()](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/addIceCandidate)

#### RTCPeerConnection关键部分——Get and share local and remote descriptions

- 开启一个[SDP](https://developer.mozilla.org/en-US/docs/Glossary/SDP) offer，以进行远程连接

```js
  trace('localPeerConnection createOffer start.');
  localPeerConnection.createOffer(offerOptions)
    .then(createdOffer).catch(setSessionDescriptionError);
```

- [RTCPeerConnection.createOffer()的API](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/createOffer)，看完就在源码里面找到了`offerOptions` `createdOffer()` `setSessionDescriptionError()`这三个对应内容

```js
// Set up to exchange only video.
const offerOptions = {
  offerToReceiveVideo: 1,
};

// Logs offer creation and sets peer connection session descriptions.
function createdOffer(description) {
  trace(`Offer from localPeerConnection:\n${description.sdp}`);

  trace('localPeerConnection setLocalDescription start.');
  localPeerConnection.setLocalDescription(description)
    .then(() => {		// The parameter list for a function with no parameters should be written with a pair of parentheses.
      setLocalDescriptionSuccess(localPeerConnection);
      // just logs successful info on the console
    }).catch(setSessionDescriptionError);
}

// Logs error when setting session description fails.
function setSessionDescriptionError(error) {
  trace(`Failed to create session description: ${error.toString()}.`);
}
```

- 解释一下`createOffer()`函数
- 先看`setLocalDescription` ，[API](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/setRemoteDescription)中也没有讲的特别清楚，简单的说，可以认为这个函数经过调用后，Local Peer的offer就发送成功（可参见[RTCPeerConnection.signalingState](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/signalingState)），但实际上发送的信息是什么、向谁发...等一系列问题，都是在官方教程中的源码里面未涉及的，这部分我写在了[WebRTC的RTCPeerConnection()原理探析](https://www.cnblogs.com/yzy11235/p/11439971.html)中
- Local Peer已经提供了offer，来而不往非礼也，下面就是Remote Peer的回应了

```js
  trace('remotePeerConnection setRemoteDescription start.');
  remotePeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);

  trace('remotePeerConnection createAnswer start.');
  remotePeerConnection.createAnswer()
    .then(createdAnswer)
    .catch(setSessionDescriptionError);

function createdAnswer(description) {
  trace(`Answer from remotePeerConnection:\n${description.sdp}.`);

  trace('remotePeerConnection setLocalDescription start.');
  remotePeerConnection.setLocalDescription(description)
    .then(() => {
      setLocalDescriptionSuccess(remotePeerConnection);
    }).catch(setSessionDescriptionError);
    
    trace('localPeerConnection setRemoteDescription start.');
  localPeerConnection.setRemoteDescription(description)
    .then(() => {
      setRemoteDescriptionSuccess(localPeerConnection);
    }).catch(setSessionDescriptionError);
}
```

- Remote Peer的[createAnswer()的API](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/createAnswer)以及`setRemoteDescription`的[API](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/setRemoteDescription)，Local Peer与Remote Peer之间的互相通信基本建立了
- 在Google Dev Tools Console里面截取了一张SDP的图片，感觉比较复杂，之前有做WebRTC底层优化的准备，现在觉得...可能在十分十分需要的时候才会去做QAQ

![WebRTC_SDP](https://img2018.cnblogs.com/blog/1615451/201908/1615451-20190831175652837-688914464.png)

#### hangupAction()

- 最后来看`hangup` button对应什么函数

```js
// Handles hangup action: ends up call, closes connections and resets peers.
function hangupAction() {
  localPeerConnection.close();
  remotePeerConnection.close();
  localPeerConnection = null;
  remotePeerConnection = null;
  hangupButton.disabled = true;
  callButton.disabled = false;
  trace('Ending call.');
}
```

- 非常清晰易懂，不解释

### 如何PC 2 PC

- 源码分析终于分析完了～
- 但还有一些问题，源码中的网页本地P2P通信如何改为PC 2 PC的通信？这个我记录在[原理](https://www.cnblogs.com/yzy11235/p/11439933.html)一文中

## RTCDataChannel

- RTCPeerConnection部分需要写的实在太多了，到这里，全文长度已经超过3000.orz...这部分脚步代码量略少一些，也尽量写的简洁一点，其余拓展见[补充](https://www.cnblogs.com/yzy11235/p/11439933.html)
- 还是从源码分析开始

### 源码分析

#### HTML

- HTML代码部分较RTCPeerConnection部分，增加了两个文本区

```html
  <textarea id="dataChannelSend" disabled
    placeholder="Press Start, enter some text, then press Send."></textarea>
  <textarea id="dataChannelReceive" disabled></textarea>
```

- 标记性语言，语法、效果非常容易理解，[可见](https://www.runoob.com/tags/tag-textarea.html)
- 在HTML语言中，我们也看到了和上一节类似的三个button，还是按button顺序来分析

#### 三个button

- 这次三个button的写法较上一节的有比较新奇的改变

```js
var startButton = document.querySelector('button#startButton');
var sendButton = document.querySelector('button#sendButton');
var closeButton = document.querySelector('button#closeButton');

startButton.onclick = createConnection;
sendButton.onclick = sendData;
closeButton.onclick = closeDataChannels;
```

- 首先是`querySelector()`括号里面的内容非常有范式，所以查到一个参考链接[CSS 选择器](https://www.runoob.com/cssref/css-selectors.html)，然后`onclick` [method](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onclick)也是一种非常简洁的写法
- 下面看三个button对应的功能

#### startButton

```js
var localConnection;
var remoteConnection;
var sendChannel;
var dataConstraint;
var dataChannelSend = document.querySelector('textarea#dataChannelSend');

// Offerer side 
function createConnection() {
  dataChannelSend.placeholder = '';
  var servers = null;
  pcConstraint = null;
  dataConstraint = null;
  trace('Using SCTP based data channels');
  // For SCTP, reliable and ordered delivery is true by default.
  // Add localConnection to global scope to make it visible
  // from the browser console.
  window.localConnection = localConnection =
      new RTCPeerConnection(servers, pcConstraint);	// constructor
  trace('Created local peer connection object localConnection');

  sendChannel = localConnection.createDataChannel('sendDataChannel',
      dataConstraint);
  trace('Created send data channel');

  localConnection.onicecandidate = iceCallback1;
  sendChannel.onopen = onSendChannelStateChange;
  sendChannel.onclose = onSendChannelStateChange;

  // Add remoteConnection to global scope to make it visible
  // from the browser console.
  window.remoteConnection = remoteConnection =
      new RTCPeerConnection(servers, pcConstraint);
  trace('Created remote peer connection object remoteConnection');

  remoteConnection.onicecandidate = iceCallback2;
  remoteConnection.ondatachannel = receiveChannelCallback;

  localConnection.createOffer().then(
    gotDescription1,
    onCreateSessionDescriptionError
  );
  startButton.disabled = true;
  closeButton.disabled = false;
}

function iceCallback1(event) {
  trace('local ice callback');
  if (event.candidate) {
    remoteConnection.addIceCandidate(
      event.candidate
    ).then(
      onAddIceCandidateSuccess,
      onAddIceCandidateError
    );
    trace('Local ICE candidate: \n' + event.candidate.candidate);
  }
}

function iceCallback2(event) {
  trace('remote ice callback');
  if (event.candidate) {
    localConnection.addIceCandidate(
      event.candidate
    ).then(
    	// print out info on the console
      onAddIceCandidateSuccess,
      onAddIceCandidateError
    );
    trace('Remote ICE candidate: \n ' + event.candidate.candidate);
  }
}

function onSendChannelStateChange() {
  var readyState = sendChannel.readyState;
  trace('Send channel state is: ' + readyState);
  if (readyState === 'open') {
    dataChannelSend.disabled = false;
    dataChannelSend.focus();
    sendButton.disabled = false;
    closeButton.disabled = false;
  } else {
    dataChannelSend.disabled = true;
    sendButton.disabled = true;
    closeButton.disabled = true;
  }
}
```

- 先看几个API，[RTCPeerConnection.createDataChannel()](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/createDataChannel)

> The createDataChannel() method on the RTCPeerConnection interface  creates a new channel linked with the remote peer, over which any kind  of data may be transmitted. This can be useful for back-channel content  such as images, file transfer, text chat, game update packets, and so  forth.

- [RTCPeerConnection.onicecandidate](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/onicecandidate)

> This happens whenever the local ICE agent needs to deliver a message  to the other peer through the signaling server. This lets the ICE agent  perform negotiation with the remote peer without the browser itself  needing to know any specifics about the technology being used for  signaling; simply implement this method to use whatever messaging  technology you choose to send the ICE candidate to the remote peer.

- [RTCIceCandidate.candidate](https://developer.mozilla.org/en-US/docs/Web/API/RTCIceCandidate/candidate)

> The read-only property candidate on the RTCIceCandidate interface  returns a DOMString describing the candidate in detail. Most of the  other properties of RTCIceCandidate are actually extracted from this  string.

- [RTCPeerConnection.addIceCandidate()](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/addIceCandidate)

> When a web site or app using RTCPeerConnection receives a new ICE  candidate from the remote peer over its signaling channel, it delivers  the newly-received candidate to the browser's ICE agent by calling  RTCPeerConnection.addIceCandidate(). This adds this new remote candidate to the RTCPeerConnection's remote description, which describes the  state of the remote end of the connection.

- [RTCDataChannel.onopen](https://developer.mozilla.org/en-US/docs/Web/API/RTCDataChannel/onopen)

> The RTCDataChannel.onopen property is an EventHandler which specifies a function to be called when the open event is fired; this is a simple  Event which is sent when the data channel's underlying data  transport—the link over which the RTCDataChannel's messages flow—is  established or re-established.

- [HTMLElement.focus()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)，这个功能比我想的有趣，而且我们在平时使用浏览器的时候，遇见的特别多～但是，有一点问题，就是基于DOM的Web前端开发太“高级”了，以至于我看不到底层的实现...

> The HTMLElement.focus() method sets focus on the specified element,  if it can be focused. The focused element is the element which will  receive keyboard and similar events by default.

- 之后的部分就是和RTCPeerConnection相类似的`createOffer()` `createAnswer()`部分，在学习完RTCPeerConnection之后，是非常容易的，所以不赘述

#### sendButton

```js
function sendData() {
  var data = dataChannelSend.value;
  sendChannel.send(data);
  trace('Sent Data: ' + data);
}
```

- 基于上一步建立的连接上，实现data传输功能

#### closeButton

```js
function closeDataChannels() {
  trace('Closing data channels');
  sendChannel.close();
  trace('Closed data channel with label: ' + sendChannel.label);
  receiveChannel.close();
  trace('Closed data channel with label: ' + receiveChannel.label);
  localConnection.close();
  remoteConnection.close();
  localConnection = null;
  remoteConnection = null;
  trace('Closed peer connections');
  startButton.disabled = false;
  sendButton.disabled = true;
  closeButton.disabled = true;
  dataChannelSend.value = '';
  dataChannelReceive.value = '';
  dataChannelSend.disabled = true;
  disableSendButton();
  enableStartButton();
}

function enableStartButton() {
  startButton.disabled = false;
}

function disableSendButton() {
  sendButton.disabled = true;
}
```