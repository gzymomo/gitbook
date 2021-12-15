[TOC]

# 1、node.js简介
Node.js 就是**运行在服务端的 JavaScript**。

Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。

Node.js是一个**事件驱动I/O服务端JavaScript环境**，**基于Google的V8引擎**，V8引擎执行Javascript的速度非常快，性能非常好。

## 1.1 node.js应用组成
Node.js 应用是由哪几部分组成的：

1. 引入 required 模块：我们可以使用 require 指令来载入 Node.js 模块。
` var http = require("http"); `

2. 创建服务器：服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。

```javascript
var http = require('http');

http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```

3. 接收请求与响应请求 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。

![](https://www.runoob.com/wp-content/uploads/2014/03/node-hello.gif)