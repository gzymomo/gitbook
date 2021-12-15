MSE格式demo，云游戏示例，硬解：

```html
<!DOCTYPE html PUBLIC"-//W3C//DTD XHTML 1.0 Strict//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <title>wzjy-ydn-sdk</title>
</head>
<body>
<style>
    body {
        margin: 0;
    }

    .fabButton {
        position: fixed;
        z-index: 999;
        bottom: 20px;
        right: 20px;
        background-color: #CCCCCC;
        width: 30px;
        height: 30px;
        border-radius: 100px;
    }

    .fabButton div {
        margin: 5px;
        background-color: #999999;
        width: 20px;
        height: 20px;
        border-radius: 100px;
    }

    .buttonView {
        z-index: 998;
        width: 30%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        position: fixed;
        right: 0;
        top: 0;
        z-index: 100;
    }

    .show {
        display: block;
    }

    .hide {
        display: none;
    }

    .buttonView button {
        margin-top: 5px;
    }
</style>

<div id="container" style="position:absolute;width:100%;height:100%">
    <video id="container_video" width="1920" height="1080" muted></video>
    <img id="container_cursor" class="hide" style="position:absolute"/>
</div>

<div class="fabButton" onclick="toggleButtonView()">
    <div></div>
</div>
<div class="buttonView show" id="buttonViewId">
    <div>
        <button type="primary" size="mini" onclick="setWindowSize(1024,768)">设置分辨率1024</button>
        <button type="primary" size="mini" onclick="setWindowSize(1920,1080)">设置分辨率1080P</button>
        <button type="primary" size="mini" onclick="setWindowSize(2560,1440)">设置分辨率2K</button>
        <button type="primary" size="mini" onclick="setWindowSize(4096,2160)">设置分辨率4K</button>
    </div>
    <div>
        鼠标模式：
        <button type="primary" size="mini" id="mouseMode" onclick="toggleMouseMode()">正常</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouseMove(0,1)" style="margin-left: 35px">上</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouseMove(-1,0)">左</button>
        <button type="primary" size="mini" onclick="mouseMove(0,-1)">下</button>
        <button type="primary" size="mini" onclick="mouseMove(1,0)">右</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="moveToAnyWhere(45,45)">移到垃圾桶</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="moveToAnyWhere(666,144)">移到地址栏</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="moveToAnyWhere(697,315)">移到可滚动处</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="moveToAnyWhere(749,700)">移到3</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouse_left_double_click()">双击左键</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouse_left_single_click()">单击左键</button>
        <button type="primary" size="mini" onclick="mouse_right_single_click()">单击右键</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouse_wheel(0,1)" style="margin-left: 75px">滚轮向上</button>
    </div>
    <div>
        <button type="primary" size="mini" onclick="mouse_wheel(1,0)">滚轮向左</button>
        <button type="primary" size="mini" onclick="mouse_wheel(0,-1)">滚轮向下</button>
        <button type="primary" size="mini" onclick="mouse_wheel(-1,0)">滚轮向右</button>
    </div>
    <div>
        <!--        <button type="primary" size="mini" onclick="keyboard_win_click()">WIN</button>-->
        <!--        <button type="primary" size="mini" onclick="keyboard_win_d_click()">WIN+D</button>-->
        <button type="primary" size="mini" onclick="keyboard_ecs_click()">Esc</button>
        <button type="primary" size="mini" onclick="keyboard_ctrl_s_click()">Ctrl+S</button>
    </div>
</div>

<script>
    function toggleButtonView() {
        var classList = document.getElementById('buttonViewId').classList
        if (classList.contains('hide')) {
            classList.remove('hide')
            classList.add('show')
        } else {
            classList.remove('show')
            classList.add('hide')
        }
    }

    function setWindowSize(width, height) {
        sendRequest('setWindowSize', {width, height});
    }

    //默认0是正常，1是游戏模式
    var mouseMode = 0
    //鼠标渲染的位置
    var mouse_x = 0
    var mouse_y = 0

    //切换鼠标模式
    function toggleMouseMode() {
        if (mouseMode === 0) {
            mouseMode = 1
            document.getElementById('mouseMode').textContent = '游戏'
        } else {
            mouseMode = 0
            document.getElementById('mouseMode').textContent = '正常'
        }
    }

    //鼠标通用
    function mouseMove(x, y) {
        mouse_x += x
        mouse_y -= y
        if (mouseMode === 0) {
            //正常
            sendRequest('mouseMove', {x: mouse_x, y: mouse_y})
        } else if (mouseMode === 1) {
            //游戏
            sendRequest('mouseMove', {m: 1, x, y})
        }
        container_cursor.style.left = mouse_x + 'px'
        container_cursor.style.top = mouse_y + 'px'
    }

    //移到
    function moveToAnyWhere(x, y) {
        mouse_x = x
        mouse_y = y
        sendRequest('mouseMove', {m: 0, x, y})
        container_cursor.style.left = mouse_x + 'px'
        container_cursor.style.top = mouse_y + 'px'
    }

    // u不传为按下，传1为弹起
    // r不传为左键，传1为右键

    //双击左键
    function mouse_left_double_click() {
        sendRequest('mouseClick')
        sendRequest('mouseClick', {u: 1})
        sendRequest('mouseClick')
        sendRequest('mouseClick', {u: 1})
    }

    //单击左键
    function mouse_left_single_click() {
        sendRequest('mouseClick')
        sendRequest('mouseClick', {u: 1})
    }

    //单击右键
    function mouse_right_single_click() {
        sendRequest('mouseClick', {r: 1})
        sendRequest('mouseClick', {u: 1, r: 1})
    }

    //滚轮上下左右
    function mouse_wheel(wheel_x, wheel_y) {
        sendRequest('mouseWheel', {wx: wheel_x, wy: wheel_y});
    }

    // //暂屏蔽发送window键
    // //window键
    // function keyboard_win_click() {
    //     sendRequest('sendKeyCodes', [91])
    // };
    //
    // //window+d组合键
    // function keyboard_win_d_click() {
    //     sendRequest('sendKeyCodes', [91, 68])
    // };

    //ecs键
    function keyboard_ecs_click() {
        sendRequest('sendKeyCodes', [27])
    };

    //ctrl+s
    function keyboard_ctrl_s_click() {
        sendRequest('sendKeyCodes', [17, 83])
    };

    // sdk开始
    let container_video = document.getElementById('container_video')
    let container_cursor = document.getElementById('container_cursor')

    // 视频实例
    const mp4Instance = {
        mediaSource: new window.MediaSource(),
        mediaPlayer: container_video,
        sourceBuffer: null,
        create: function () {
            mp4Instance.mediaPlayer.src = window.URL.createObjectURL(mp4Instance.mediaSource);

            mp4Instance.mediaSource.addEventListener('sourceopen', function () {
                mp4Instance.sourceBuffer = mp4Instance.mediaSource.addSourceBuffer('video/mp4; codecs="avc1.640030"');
            }, false);
        },
        playVideo: function (data) {
            try {
                mp4Instance.sourceBuffer.appendBuffer(data);
                if (mp4Instance.mediaPlayer.paused) {
                    mp4Instance.mediaPlayer.play();
                }
            } catch (err) {
                // console.log(err)
            }
        }
    }
    // 初始化 视频实例
    mp4Instance.create()
    // 音频实例
    const audioInstance = {
        audioContext: new AudioContext,
        audioNextTime: 0,
        playAudio: function (data) {
            audioInstance.audioContext.decodeAudioData(new Uint8Array(data.data).buffer, function (e) {
                try {
                    let bufferSource = audioInstance.audioContext.createBufferSource();
                    const currentTime = audioInstance.audioContext.currentTime;
                    const t1 = audioInstance.audioNextTime - currentTime;
                    const t2 = currentTime + Math.max(t1, 0);

                    let containerSize = data.containerSize;
                    if ((e.duration - containerSize) < 0) {
                        containerSize = e.duration;
                    }
                    bufferSource.buffer = e;
                    bufferSource.connect(audioInstance.audioContext.destination);
                    audioInstance.audioNextTime = t2 + (containerSize - data.sampleRate);
                    bufferSource.start(t2, data.sampleRate, containerSize - data.sampleRate);
                } catch (err) {
                    // console.log(err)
                }
            })
        },
    }
    var websocket = null;

    function initWebSocket() {
        let socketAddress = "ws://s11111.ydn.shakuai.com.cn:18080?q=20201211-3ddbd560-9b29-4abc-ac06-185bdf39ef68";
        // let socketAddress = "ws://127.0.0.1:18080?q=20201211-3ddbd560-9b29-4abc-ac06-185bdf39ef68";
        if (websocket) {
            websocket.close();
            websocket = null;
        }
        return new Promise((resolve, reject) => {
            websocket = new WebSocket(socketAddress);
            websocket.onmessage = (e) => {
                try {
                    let msg = JSON.parse(e.data);
                    if (typeof msg.type != 'undefined') {
                        websocketOnmessage(msg);
                    } else {
                    }
                } catch (e) {
                }
            };
        });
    }

    //发送消息
    function sendRequest(type, data) {
        if (websocket.readyState == 1) {
            websocket.send(JSON.stringify({type, data}));
        }
    }

    //收到消息
    function websocketOnmessage(message) {
        if (message.type === 'video') {
            mp4Instance.playVideo(new Uint8Array(message.message))
        } else if (message.type === 'audio') {
            message.message.data = new Uint8Array(message.message.data)
            audioInstance.playAudio(message.message)
        } else if (message.type === 'cursor') {
            let classList = container_cursor.classList
            if (message.message) {
                classList.remove('hide')
                classList.add('show')
                container_cursor.src = message.message
            } else {
                classList.remove('show')
                classList.add('hide')
            }
        }
    }

    //连接websocket
    async function connectWebSocket() {
        if (!websocket) {
            await initWebSocket().catch(err => {
                console.error("err " + err);
                return;
            });
        }
    }

    connectWebSocket()
</script>
</body>
</html>
```

