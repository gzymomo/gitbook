[TOC]

掘金：阿宝哥：[你不知道的 WebSocket](https://juejin.im/post/5f1ef215e51d453473206df6)

# 1、WebSocket
![](https://img-blog.csdn.net/20180510225115144?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21vc2hvd2dhbWU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

HTTP 协议有一个缺陷：**通信只能由客户端发起**，HTTP 协议做不到服务器主动向客户端推送信息。

![](https://img-blog.csdn.net/20180510223926952?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21vc2hvd2dhbWU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)