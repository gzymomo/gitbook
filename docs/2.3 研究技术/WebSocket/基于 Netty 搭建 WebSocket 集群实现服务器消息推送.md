- [基于 Netty 搭建 WebSocket 集群实现服务器消息推送](https://blog.csdn.net/qq_37217713/article/details/120278819)

# 一、背景

我们在实际的工作开发中，必然会遇到一些需要网页与服务器端保持连接（起码看上去是保持连接）的需求，比如类似微信网页版的聊天类应用，比如需要频繁更新页面数据（实时数据例如天气，电流电压，pm2.5等等这样类似的数据）的监控系统页面或股票看盘页面，比如服务器读取MySQL或者redis或者第三方的数据主动推送给浏览器客户端等等业务场景。我们通常采用如下几种技术：

**（1）短轮询**：前端老师利用ajax定期向服务器发起http请求，无论数据是否更新立马返回数据。这样存在的缺点就是，一方面如果后端数据木有更新，那么这一次http请求就是无用的，另一方面高并发情况下，短链接的频繁创建销毁，以及客户端数量过大造成过多无用的http请求，都会对服务器和带宽造成压力，短轮询只适用于客户端连接少，并发量不高的场景；

 **（2）长轮询**：利用comet不断向服务器发起请求，服务器将请求暂时挂起，直到有新的数据的时候才返回，相对短轮询减少了请求次数得到了一定的优化，但是在高并发的场景下依然不适用；

 **（3）SSE**：服务端推送（Server Send Event），在客户端发起一次请求后会保持该连接，服务器端基于该连接持续向客户端发送数据，从HTML5开始加入。

 **（4）Websocket**：这是也是一种保持长连接的技术，并且是双向的，从HTML5开始加入，并非完全基于HTTP，适合于频繁和较大流量的双向通讯场景，是服务器推送消息功能的最佳实践。而实现websocket的最佳方式，就是**netty**。

网上的很多netty搭建websocket的博文都不够全面，有很多问题都木有解决方式，我通过实际工作中的经验，把常遇到的问题总结了方法，下文会说到！

# 二、什么是websocekt呢？

websocket是一种在单个TCP连接上进行全双工通信的协议。也就是一种保持长连接的技术，并且是双向的。

websocket协议本身是构建在http协议之上的升级协议，客户端首先向服务器端去建立连接，这个连接本身就是http协议只是在头信息中包含了一些websocket协议的相关信息，一旦http连接建立之后，服务器端读到这些websocket协议的相关信息就将此协议升级成websocket协议。WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

 **简单理解，就是一种通讯协议，重点是websocket的实现方式–netty。**

# 三、什么是netty呢?

## 3.1 socket

（1）**网络编程本质**就是说两个设备之间信息的发送与接收，通过操作相应API调度计算机硬件资源，并且利用管道（网线）进行数据交互的过程。相关技术点像ISO七层模型，TCP三次握手/四次挥手等网络编程的基础不再赘述。

（2）**而socket是**对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket发起系统调用操作系统内核，我们才能使用TCP/IP协议。

（3）**我们经常说的I/O**，在计算机中指Input/Output,即输入输出，实质上IO分为两种，一种是磁盘IO，磁盘上的数据读取到用户空间，那么这次数据转移操作其实就是一次I/O操作，也就是一次文件I/O。一种是网络IO，当一个客户端和服务端之间相互通信,交互我们称之为网络io(网络通讯)

## 3.2 Java IO模型

有BIO（同步阻塞IO）、NIO（同步非阻塞IO）、AIO（异步IO），netty就是一个NIO的高性能的框架。

  （１）BIO：同步阻塞IO模型，适用于连接数目比较少且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，服务器实现模式为一个连接一个线程，即客户端有连 接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造  成不必要的线程开销，可以通过线程池机制改善(实现多个客户连接服务器)。
 

![img](https://img-blog.csdnimg.cn/img_convert/108c1746a996500a33dc0d988edec88b.png)

（２）NIO：同步非阻塞IO模型，适用于连接数据多且连接比较短的架构，如聊天服务器，弹幕系统，服务器间通讯等；服务器实现模式为一个线程处理多个连接（一个请求一个线程），包含Selector 、Channel 、Buffer三大组件。
 

![img](https://img-blog.csdnimg.cn/img_convert/540a1a41210ef9c69ac69209d14d3832.png)

①**selector选择器**：用一个线程处理多个客户端的连接，就会使用到selector选择器，selector用于监听多个通道上是否有事件发生（比如连接请求，数据到达等），如果有事件发生，便获取事件然后针对于每个事件进行相应的处理，因此可以使用单个线程就可以监听多个客户端通道。

 ②**channel通道**：Channel管道和Java  IO中的Stream(流)是差不多一个等级的。只不过Stream是单向的，譬如：InputStream,  OutputStream.而Channel是双向的，同时进行读写数据，而流只能读或者写。可以实现异步读写数据，可以从缓冲区读数据，也可以写数据到缓冲区。

 ③**buffer缓冲区**：Buffer本质上是一个可以读写的内存块，可以理解成容器对象，底层是有一个数组，通过buffer实现非阻塞机制，该对象提供了一组方法，可以轻松的使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel提供了从文件，网络读取数据的通道，但是读取和写入的数据必须经过buffer。

 （３）AIO：异步IO模型，使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分  调用OS参与并发操作，编程比较复杂。在Linux底层用epoll(一种轮询模型)，aio多包了一层封装，aio的api更好用。Windows上的aio是自己实现的，不是轮询模型是事件模型，完成端口实现的，要比linxu上的aio效率高。

![img](https://img-blog.csdnimg.cn/img_convert/9bd085b0d16e3b98c75c62bf48096be5.png)

## 3.3 netty

### 3.3.1 概念

netty是一个开源异步的事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。

### 3.3.2 三大特点

①高并发：Netty 是一款基于 NIO（Nonblocking  IO，非阻塞IO）开发异步事件驱动的高性能网络通信框架，nio使用了select模型（多路复用器技术），从而使得系统在单线程的情况下可以同时处理多个客户端请求。Netty使用了Reactor模型，Reactor模型有三种多线程模型，netty是在主从 Reactor 多线程模型上做了一定的改进。

Netty有两个线程组，一个作为bossGroup线程组,负责客户端接收,一个workerGroup线程组负责工作线程的工作(与客户端的IO操作和任务操作等等)，Netty 的所有 IO 操作都是异步非阻塞的，通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO  操作结果。他的并发性能得到了很大提高。

 ②传输快：Netty  的传输依赖于零拷贝特性，实现了更高效率的传输。零拷贝要求内核（kernel）直接将数据从磁盘文件拷贝到Socket缓冲区（套接字），而无须通过应用程序。零拷贝减少不必要的内存拷贝，不仅提高了应用程序的性能，而且减少了内核态和用户态上下文切换。

 ③封装好：Netty 封装了 NIO 操作的很多细节，提供了易于使用调用接口。

### 3.3.3 主从Reactor架构图

![img](https://img-blog.csdnimg.cn/img_convert/86a3dd52c0c0fbe05ae16de949646b25.png)


 说明：①Reactor响应式编程（事件驱动模型）：一般有一个主循环和一个任务队列，所有事件只管往队列里塞，主循环则从队列里取出并处理。

如果不依赖于多路复用处理多个任务就会需要多线程(与连接数对等) ,但是依赖于多路复用，这个循环就可以在单线程的情况下处理多个连接。无论是哪个连接发生了什么事件，都会被主循环从队列取出并处理(可能用回调函数处理等) ,也就是说程序的走向由事件驱动.

 ②mainReactor：主Reactor负责 单线程就可以接受所有客户端连接

 ③subReactor：子Reactor负责 多线程处理客户端的读写IO事件

 ④ThreadPool：线程池负责 处理业务耗时的操作

### 3.3.4 应用场景

①现在物联网的应用无处不在，大量的项目都牵涉到应用传感器和服务器端的数据通信，Netty作为基础通信组件进行网络编程。

  ②现在互联网系统讲究的都是高并发、分布式、微服务，各类消息满天飞，Netty在这类架构里面的应用可谓是如鱼得水，如果你对当前的各种应用服务器不爽，那么完全可以基于Netty来实现自己的HTTP服务器，FTP服务器，UDP服务器，RPC服务器，WebSocket服务器，Redis的Proxy服务器，MySQL的Proxy服务器等等。

 现在非常多的开源软件都是基于netty开发的，例如阿里分布式服务框架 Dubbo 的 RPC 框架，淘宝的消息中间件 RocketMQ；

 ③游戏行业：无论是手游服务端还是大型的网络游戏，Java 语言得到了越来越广泛的应用。Netty 作为高性能的基础通信组件，它本身提供了 TCP/UDP 和 HTTP 协议栈。地图服务器之间可以方便的通过 Netty 进行高性能的通信。

 ④大数据：开源集群运算框架 Spark；分布式计算框架 Storm；

# 四、架构设计

## 4.1 系统设计架构图

![img](https://img-blog.csdnimg.cn/img_convert/5f90ac3520d96d2d8635d87646f34f2f.png)

## 4.2 架构中存在的六大经典问题

**第一个问题**：客户端和服务端单独通信，怎么实现？

 **第二个问题**：单机中websocekt主动向所有客户端推送消息如何实现？在集群中如何实现？

 **第三个问题**：单机如何统计同时在线的客户数量？websocket集群如何统计在线的客户数量呢？

 **第四个问题**：由于客户端和websocket服务器集群中的某个节点建立长连接是随机的，如何解决服务端向某个或某些部分客户端推送消息？

 **第五个问题**：websocket服务端周期性向客户端推送消息，单机或集群中如何实现？

 **第六个问题**：websocket集群中一个客户端向其他客户端主动发送消息，如何实现？
 福利来啦！！以上所有问题，已在代码中全部解决并实践！！！

## 4.3 引入pom依赖和yml配置

（1）pom依赖

```go
<dependencies>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-all</artifactId>
      <version>4.1.36.Final</version>
    </dependency>
  </dependencies>
```

（2）yml配置

```go
websocket:
  port: 7000 #端口
  url: /msg #访问url
```

(3) 客户端和服务端交互的消息体

```go
package com.wander.netty.websocket.yeelight;
import lombok.Data;
import java.io.Serializable;

@Data
public class MessageRequest implements Serializable {

    private Long unionId;

    private Integer current = 1;

    private Integer size = 10;
}
```

## 4.4 Websocket 初始化器

```go
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket初始化器
 **/
@Slf4j
@Component
public class WebsocketInitialization {

    @Resource
    private WebsocketChannelInitializer websocketChannelInitializer;

    @Value("${websocket.port}")
    private Integer port;

    @Async
    public void init() throws InterruptedException {

        //bossGroup连接线程组，主要负责接受客户端连接，一般一个线程足矣
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        //workerGroup工作线程组，主要负责网络IO读写
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {

            //启动辅助类
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            //bootstrap绑定两个线程组
            serverBootstrap.group(bossGroup, workerGroup);
            //设置通道为NioChannel
            serverBootstrap.channel(NioServerSocketChannel.class);
            //可以对入站\出站事件进行日志记录，从而方便我们进行问题排查。
            serverBootstrap.handler(new LoggingHandler(LogLevel.INFO));
            //设置自定义的通道初始化器，用于入站操作
            serverBootstrap.childHandler(websocketChannelInitializer);
            //启动服务器,本质是Java程序发起系统调用，然后内核底层起了一个处于监听状态的服务，生成一个文件描述符FD
            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            //异步
            channelFuture.channel().closeFuture().sync();

        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

}
```

## 4.5 websocket 通道初始化器

```go
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket通道初始化器
 **/
@Component
public class WebsocketChannelInitializer extends ChannelInitializer<SocketChannel> {

    @Autowired
    private WebSocketHandler webSocketHandler;

    @Value("${websocket.url}")
    private String websocketUrl;

    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {

        //获取pipeline通道
        ChannelPipeline pipeline = socketChannel.pipeline();
        //因为基于http协议，使用http的编码和解码器
        pipeline.addLast(new HttpServerCodec());
        //是以块方式写，添加ChunkedWriteHandler处理器
        pipeline.addLast(new ChunkedWriteHandler());
        /*
          说明
          1. http数据在传输过程中是分段, HttpObjectAggregator ，就是可以将多个段聚合
          2. 这就就是为什么，当浏览器发送大量数据时，就会发出多次http请求
        */
        pipeline.addLast(new HttpObjectAggregator(8192));
        /* 说明
          1. 对应websocket ，它的数据是以 帧(frame) 形式传递
          2. 可以看到WebSocketFrame 下面有六个子类
          3. 浏览器请求时 ws://localhost:7000/msg 表示请求的uri
          4. WebSocketServerProtocolHandler 核心功能是将 http协议升级为 ws协议 , 保持长连接
          5. 是通过一个 状态码 101
        */
        pipeline.addLast(new WebSocketServerProtocolHandler(websocketUrl));
        //自定义的handler ，处理业务逻辑
        pipeline.addLast(webSocketHandler);
    }
}
```

## 4.6 websocket 入站处理器

```go
/**
 * @Author WDYin
 * @Date 2021/6/10
 * @Description websocket处理器
 **/
@Slf4j
@Component
@ChannelHandler.Sharable//保证处理器，在整个生命周期中就是以单例的形式存在，方便统计客户端的在线数量
public class WebSocketHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    //通道map，存储channel，用于群发消息，以及统计客户端的在线数量，解决问题问题三，如果是集群环境使用redis的hash数据类型存储即可
    private static Map<String, Channel> channelMap = new ConcurrentHashMap<>();
    //任务map，存储future，用于停止队列任务
    private static Map<String, Future> futureMap = new ConcurrentHashMap<>();
    //存储channel的id和用户主键的映射，客户端保证用户主键传入的是唯一值，解决问题四，如果是集群中需要换成redis的hash数据类型存储即可
    private static Map<String, Long> clientMap = new ConcurrentHashMap<>();

    /**
     * 客户端发送给服务端的消息
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        try {
            //接受客户端发送的消息
            MessageRequest messageRequest = JSON.parseObject(msg.text(), MessageRequest.class);

            //每个channel都有id，asLongText是全局channel唯一id
            String key = ctx.channel().id().asLongText();
            //存储channel的id和用户的主键
            clientMap.put(key, messageRequest.getUnionId());
            log.info("接受客户端的消息......" + ctx.channel().remoteAddress() + "-参数[" + messageRequest.getUnionId() + "]");

            if (!channelMap.containsKey(key)) {
                //使用channel中的任务队列，做周期循环推送客户端消息，解决问题二和问题五
                Future future = ctx.channel().eventLoop().scheduleAtFixedRate(new WebsocketRunnable(ctx, messageRequest), 0, 10, TimeUnit.SECONDS);
                //存储客户端和服务的通信的Chanel
                channelMap.put(key, ctx.channel());
                //存储每个channel中的future，保证每个channel中有一个定时任务在执行
                futureMap.put(key, future);
            } else {
                //每次客户端和服务的主动通信，和服务端周期向客户端推送消息互不影响 解决问题一
                ctx.channel().writeAndFlush(new TextWebSocketFrame(Thread.currentThread().getName() + "服务器时间" + LocalDateTime.now() + "wdy"));
            }

        } catch (Exception e) {

            log.error("websocket服务器推送消息发生错误：", e);

        }
    }

    /**
     * 客户端连接时候的操作
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        log.info("一个客户端连接......" + ctx.channel().remoteAddress() + Thread.currentThread().getName());
    }

    /**
     * 客户端掉线时的操作
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {

        String key = ctx.channel().id().asLongText();
        //移除通信过的channel
        channelMap.remove(key);
        //移除和用户绑定的channel
        clientMap.remove(key);
        //关闭掉线客户端的future
        Optional.ofNullable(futureMap.get(key)).ifPresent(future -> {
            future.cancel(true);
            futureMap.remove(key);
        });
        log.info("一个客户端移除......" + ctx.channel().remoteAddress());
        ctx.close(); //关闭连接
    }

    /**
     * 发生异常时执行的操作
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        String key = ctx.channel().id().asLongText();
        //移除通信过的channel
        channelMap.remove(key);
        //移除和用户绑定的channel
        clientMap.remove(key);
        //移除定时任务
        Optional.ofNullable(futureMap.get(key)).ifPresent(future -> {
            future.cancel(true);
            futureMap.remove(key);
        });
        //关闭长连接
        ctx.close();
        log.info("异常发生 " + cause.getMessage());
    }

    public static Map<String, Channel> getChannelMap() {
        return channelMap;
    }

    public static Map<String, Future> getFutureMap() {
        return futureMap;
    }

    public static Map<String, Long> getClientMap() {
        return clientMap;
    }
}
```

## 4.7 channel中任务队列的线程任务

```go
/**
 * @Author WDYin
 * @Date 2021/8/10
 * @Description websocket程序
 **/
@Slf4j
@Component
public class WebsocketApplication {

    @Resource
    private WebsocketInitialization websocketInitialization;

    @PostConstruct
    public void start() {
        try {
            log.info(Thread.currentThread().getName() + ":websocket启动中......");
            websocketInitialization.init();
            log.info(Thread.currentThread().getName() + ":websocket启动成功！！！");
        } catch (Exception e) {
            log.error("websocket发生错误：",e);
        }
    }
}
```

## 4.8 websocket启动程序

```go
/**
 * @Author WDYin
 * @Date 2021/8/10
 * @Description websocket程序
 **/
@Slf4j
@Component
public class WebsocketApplication {

    @Resource
    private WebsocketInitialization websocketInitialization;

    @PostConstruct
    public void start() {
        try {
            log.info(Thread.currentThread().getName() + ":websocket启动中......");
            websocketInitialization.init();
            log.info(Thread.currentThread().getName() + ":websocket启动成功！！！");
        } catch (Exception e) {
            log.error("websocket发生错误：",e);
        }
    }
}
```

## 4.9 问题六解决方案

```go
/**
 * @Author WDYin
 * @Date 2021/9/12
 * @Description
 **/
@RequestMapping("index")
@Controller
public class WebsocketController {

    /**
     *
     * @param id 用户主键
     * @param idList 要把消息发送给其他用户的主键
     */
    @RequestMapping("hello1")
    private void hello(Long id, List<Long> idList){
        //获取所有连接的客户端,如果是集群环境使用redis的hash数据类型存储即可
        Map<String, Channel> channelMap = WebSocketHandler.getChannelMap();
        //获取与用户主键绑定的channel,如果是集群环境使用redis的hash数据类型存储即可
        Map<String, Long> clientMap = WebSocketHandler.getClientMap();
        //解决问题六,websocket集群中一个客户端向其他客户端主动发送消息，如何实现？
        clientMap.forEach((k,v)->{
            if (idList.contains(v)){
                Channel channel = channelMap.get(k);
                channel.eventLoop().execute(() -> channel.writeAndFlush(new TextWebSocketFrame(Thread.currentThread().getName()+"服务器时间" + LocalDateTime.now() + "wdy")));
            }
        });
    }
}
```

## 4.10 前端代码

```go
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    var socket;
    //判断当前浏览器是否支持websocket
    if(window.WebSocket) {
        socket = new WebSocket("ws://localhost:7000/msg");
        //相当于channelReado, ev 收到服务器端回送的消息
        socket.onmessage = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + ev.data;
        }

        //相当于连接开启(感知到连接开启)
        socket.onopen = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value = "连接开启了.."
        }

        //相当于连接关闭(感知到连接关闭)
        socket.onclose = function (ev) {

            var rt = document.getElementById("responseText");
            rt.value = rt.value + "\n" + "连接关闭了.."
        }
    } else {
        alert("当前浏览器不支持websocket")
    }

    //发送消息到服务器
    function send(websocketMessage) {
        if(!window.socket) { //先判断socket是否创建好
            return;
        }
        if(socket.readyState == WebSocket.OPEN) {
            //通过socket 发送消息
            socket.send(websocketMessage)
        } else {
            alert("连接没有开启");
        }
    }
</script>
    <form onsubmit="return false">
        <textarea name="websocketMessage" style="height: 300px; width: 300px"></textarea>
        <input type="button" value="发生消息" onclick="send(this.form.websocketMessage.value)">
        <textarea id="responseText" style="height: 300px; width: 300px"></textarea>
        <input type="button" value="清空内容" onclick="document.getElementById('responseText').value=''">
    </form>
</body>
</html>
```

## 4.11 wesocket在nginx中配置

nginx.conf中的配置

```go
#第一步：
upstream websocket-router {
        server 192.168.1.31:7000 max_fails=10 weight=1 fail_timeout=5s;
        keepalive 1000;
}
#第二步：
server {
        listen      80; #监听80端口
        server_name websocket.wdy.com; #域名配置

        ssl_session_cache    shared:SSL:1m; 
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;

        location / {
            client_max_body_size 100M;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade; #支持wss
            proxy_set_header Connection "upgrade"; #支持wssi
            proxy_pass http://websocket-router; #代理路由
            root   html;
            index  index.html index.htm;
        }
    }
```

## 4.12 效果图 

springboot启动后，用浏览器打开前端页面：
 

![img](https://img-blog.csdnimg.cn/img_convert/e40e66e94369137f6303bbe72db86ea7.png)

本文主要讲述了网络编程相关的IO模型以及NIO框架 netty 如何搭建websocket。

