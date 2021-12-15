[低延时的H5直播方案JSMpeg](https://blog.csdn.net/xundh/article/details/94605598)



# 一、说明

## 1. JSMpeg简介

JSMpeg是一个JavaScript编写的视频解码器，支持MPEG1视频、MP2音频解码，WebGL&Canvas2D渲染、WebAudio音频输出。
JSMpeg可以通过Ajax加载静态视频、或通过WebSocket加载低延迟的流媒体（延时50ms）。
JSMpeg 在iPhone 5S上支持以30fps的速度解码720P的视频，支持绝大多数的现代浏览器，并且库非常的小巧。

## 2. JSMpeg项目地址

https://github.com/phoboslab/jsmpeg

## 3. 业务场景

- 客户端不支持Http-Flv（如iOS）的直播场景
- 客户端不能接受HLS延时过大
- 客户端不能兼容WebRTC技术

## 4. 本文需要的环境

- ffmpeg(向jsmpeg服务端推流)
- jsmpeg(流转成websocket)
- nodejs + http-server(客户端网页容器)
- 一个rtsp节目源（也可以是本地视频文件）

# 二、环境配置

## 1. 配置ffmpeg

下载可执行的ffmpeg，并在系统环境变量PATH里指向ffmpeg可执行文件的目录 。
更详细的配置过程可见本系列文章上一篇的内容。

## 2. 安装配置nodejs+http-server环境

### (1) 安装nodejs

多数Linux发行版已包含nodejs，本安装过程省略。

### (2) 安装websocket和http-server

```bash
npm install -g ws
npm install ws
npm install http-server -g
123
```

## 3. jsmpeg

### 在下面地址下载

[jsmpeg](https://codeload.github.com/phoboslab/jsmpeg/zip/master)

# 三、 运行测试

## 1. 启动jsmpeg

**打开一个命令行，进入jsmpeg目录，运行：**

```bash
node websocket-relay.js supersecret 8081 8082
1
```

**参数说明：**

- Supersecret是密码
- 8081是ffmpeg推送端口
- 8082是前端webSocket端口

## 2. 启动ffmpeg推流

**再打开一个命令行启动推流**

```bash
ffmpeg -I "rtspurl节目源地址" -q 0 -f mpegts -codec:v mpeg1video -s 1366x768 http://127.0.0.1:8081/supersecret
1
```

## 4. 启动一个http-server

**再打开一个命令行进入jsmpeg目录，输入：**

```bash
http-server
1
```

## 5. 观看视频

**用浏览器打开网址**
http://ip:8080/view-stream.html

正常就能看到视频画面了。

本测试只看单路视频，实际使用中需要再进行开发完善功能。