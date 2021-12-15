[TOC]

# Nginx配置WebSocket反向代理(Tomcat+Nginx)

WebSocket 和HTTP协议不同，但是WebSocket中的握手和HTTP中的握手兼容，它使用HTTP中的Upgrade协议头将连接从HTTP升级到WebSocket。这使得WebSocket程序可以更容易的使用现已存在的基础设施。例如，WebSocket可以使用标准的HTTP端口 80 和 443，因此，现存的防火墙规则也同样适用。

一个WebSockets的应用程序会在客户端和服务端保持一个长时间工作的连接。用来将连接从HTTP升级到WebSocket的HTTP升级机制使用HTTP的Upgrade和Connection协议头。反向代理服务器在支持WebSocket方面面临着一些挑战。一项挑战是WebSocket是一个hop-by-hop协议，所以，当代理服务器拦截到一个客户端发来的Upgrade请求时，它(指服务器)需要将它自己的Upgrade请求发送给后端服务器，也包括合适的请求头。此外，由于WebSocket连接是长时间保持的，所以代理服务器需要允许这些连接处于打开状态，而不是像对待HTTP使用的短连接那样将其关闭。

NGINX 通过在客户端和后端服务器之间建立起一条隧道来支持WebSocket。为了使NGINX可以将来自客户端的Upgrade请求发送给后端服务器，Upgrade和Connection的头信息必须被显式的设置。

## 配置

其实核心配置只要这几句，但是确有很多不同的情况，需要根据情况加入支持。

```xml
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header X-real-ip $remote_addr;
proxy_set_header X-Forwarded-For $remote_addr
```

如果你不是反向代理本地，是集群，可能有跨域的问题，需要加入
```xml
# 允许跨域
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
```

nginx.conf中server部分的栗子:
```xml
    server {
        listen       80;
        #域名配置
        server_name  localhost;

        charset utf-8;
		client_max_body_size 15m;
		client_body_buffer_size 512k;
		proxy_connect_timeout 90;
		proxy_send_timeout 120;
		proxy_read_timeout 120;
		proxy_buffer_size 4k;
		proxy_buffers 4 32k;
		proxy_busy_buffers_size 64k;
		proxy_temp_file_write_size 64k;
		#websocket相关配置
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
    	proxy_set_header Connection "upgrade";
    	proxy_set_header X-real-ip $remote_addr;
		proxy_set_header X-Forwarded-For $remote_addr;
		#这里是正向代理，前端vue地址
        root    "/vdb1/xxxx/vue";
        location / {
            index  index.html index.htm index.php l.php;
           autoindex  off;
        }
        location /xxxx{
           #这里是后台Java SpringBoot地址，反向代理
           proxy_pass   http://localhost:8888/xxxx;
        }
    }
```

# nginx 请求转发webSocket连接

以下为nginx的配置文件，与普通的端口转发差不多，只需要修改或加多一两个配置信息

```yaml
location /cultureMusicWebSocket/ {
        proxy_pass http://localhost:8082;
 
        proxy_http_version 1.1;
        proxy_read_timeout 360s;   
        proxy_redirect off;   
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

# [Nginx支持WebSocket反向代理](https://www.cnblogs.com/kevingrace/p/9512287.html)

WebSocket是目前比较成熟的技术了，WebSocket协议为创建客户端和服务器端需要实时双向通讯的webapp提供了一个选择。其为HTML5的一部分，WebSocket相较于原来开发这类app的方法来说，其能使开发更加地简单。大部分现在的浏览器都支持WebSocket，比如Firefox，IE，Chrome，Safari，Opera，并且越来越多的服务器框架现在也同样支持WebSocket。

在实际的生产环境中，要求多个WebSocket服务器必须具有高性能和高可用，那么WebSocket协议就需要一个负载均衡层，NGINX从1.3版本开始支持WebSocket，其可以作为一个反向代理和为WebSocket程序做负载均衡。

WebSocket协议与HTTP协议不同，但WebSocket握手与HTTP兼容，使用HTTP升级工具将连接从HTTP升级到WebSocket。这允许WebSocket应用程序更容易地适应现有的基础架构。例如，WebSocket应用程序可以使用标准HTTP端口80和443，从而允许使用现有的防火墙规则。

WebSocket应用程序可以在客户端和服务器之间保持长时间运行的连接，从而有助于开发实时应用程序。用于将连接从HTTP升级到WebSocket的HTTP升级机制使用Upgrade和Connection头。反向代理服务器在支持WebSocket时面临一些挑战。一个是WebSocket是一个逐跳协议，因此当代理服务器拦截客户端的升级请求时，需要向后端服务器发送自己的升级请求，包括相应的头文件。此外，由于WebSocket连接长期存在，与HTTP使用的典型短期连接相反，反向代理需要允许这些连接保持打开状态，而不是关闭它们，因为它们似乎处于空闲状态。

允许在客户机和后端服务器之间建立隧道，NGINX支持WebSocket。对于NGINX将升级请求从客户端发送到后台服务器，必须明确设置Upgrade和Connection标题。

**Nginx开启websocket代理功能的配置如下：**

```
1）编辑nginx.conf，在http区域内一定要添加下面配置：``map $http_upgrade $connection_upgrade {``  ``default upgrade;``  ``''` `close;``}` `map指令的作用：``该作用主要是根据客户端请求中$http_upgrade 的值，来构造改变$connection_upgrade的值，即根据变量$http_upgrade的值创建新的变量$connection_upgrade，``创建的规则就是{}里面的东西。其中的规则没有做匹配，因此使用默认的，即 $connection_upgrade 的值会一直是 upgrade。然后如果 $http_upgrade为空字符串的话，``那值会是 close。` `2）编辑vhosts下虚拟主机的配置文件，在location匹配配置中添加如下内容：``proxy_http_version 1.1;``proxy_set_header Upgrade $http_upgrade;``proxy_set_header Connection ``"Upgrade"``;` `示例如下：``upstream socket.kevin.com {``  ``hash` `$remote_addr consistent;``  ``server 10.0.12.108:9000;``  ``server 10.0.12.109:9000;``}` ` ``location / {``      ``proxy_pass http:``//socket``.kevin.com/;``      ``proxy_set_header Host $host:$server_port;``      ``proxy_http_version 1.1;``      ``proxy_set_header Upgrade $http_upgrade;``      ``proxy_set_header Connection ``"upgrade"``;``    ``}
```

**WebSocket 机制**
WebSocket是HTML5下一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：
1） WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样；
2）WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

传统HTTP客户端与服务器请求响应模式如下图所示：

![img](https://images2018.cnblogs.com/blog/907596/201808/907596-20180821162614243-1749361344.jpg)

WebSocket模式客户端与服务器请求响应模式如下图：

![img](https://images2018.cnblogs.com/blog/907596/201808/907596-20180821162630256-29020146.jpg)

上图对比可以看出，相对于传统HTTP每次请求-应答都需要客户端与服务端建立连接的模式，WebSocket是类似Socket的TCP长连接通讯模式。一旦WebSocket连接建立后，后续数据都以帧序列的形式传输。在客户端断开WebSocket连接或Server端中断连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

相比HTTP长连接，WebSocket有以下特点：
1）是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可以互相主动请求。而HTTP长连接基于HTTP，是传统的客户端对服务器发起请求的模式。
2）HTTP长连接中，每次数据交换除了真正的数据部分外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。Websocket协议通过第一个request建立了TCP连接之后，之后交换的数据都不需要发送 HTTP header就能交换数据，这显然和原有的HTTP协议有区别所以它需要对服务器和客户端都进行升级才能实现（主流浏览器都已支持HTML5）。此外还有 multiplexing、不同的URL可以复用同一个WebSocket连接等功能。这些都是HTTP长连接不能做到的。

总的来说：
WebSocket与Http相同点
\- 都是一样基于TCP的，都是可靠性传输协议。
\- 都是应用层协议。

WebSocket与Http不同点
\- WebSocket是双向通信协议，模拟Socket协议，可以双向发送或接受信息。HTTP是单向的。
\- WebSocket是需要浏览器和服务器握手进行建立连接的。而http是浏览器发起向服务器的连接，服务器预先并不知道这个连接。

WebSocket与Http联系
WebSocket在建立握手时，数据是通过HTTP传输的。但是建立之后，在真正传输时候是不需要HTTP协议的。

在WebSocket中，只需要服务器和浏览器通过HTTP协议进行一个握手的动作，然后单独建立一条TCP的通信通道进行数据的传送。
WebSocket连接的过程是：
1）客户端发起http请求，经过3次握手后，建立起TCP连接；http请求里存放WebSocket支持的版本号等信息，如：Upgrade、Connection、WebSocket-Version等；
2）服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据；
3）客户端收到连接成功的消息后，开始借助于TCP传输信道进行全双工通信。

下面再通过客户端和服务端交互的报文对比WebSocket通讯与传统HTTP的不同点：
1）在客户端，new WebSocket实例化一个新的WebSocket客户端对象，请求类似 ws://yourdomain:port/path 的服务端WebSocket URL，客户端WebSocket对象会自动解析并识别为WebSocket请求，并连接服务端端口，执行双方握手过程，客户端发送数据格式类似：

```
GET ``/webfin/websocket/` `HTTP``/1``.1``Host: localhost``Upgrade: websocket``Connection: Upgrade``Sec-WebSocket-Key: xqBt3ImNzJbYqRINxEFlkg==``Origin: http:``//localhost``:8080``Sec-WebSocket-Version: 13
```

可以看到，客户端发起的WebSocket连接报文类似传统HTTP报文，Upgrade：websocket参数值表明这是WebSocket类型请求，Sec-WebSocket-Key是WebSocket客户端发送的一个 base64编码的密文，要求服务端必须返回一个对应加密的Sec-WebSocket-Accept应答，否则客户端会抛出Error during WebSocket handshake错误，并关闭连接。
2）服务端收到报文后返回的数据格式类似：

```
HTTP``/1``.1 101 Switching Protocols``Upgrade: websocket``Connection: Upgrade``Sec-WebSocket-Accept: K7DJLdLooIwIG``/MOpvWFB3y3FE8``=
```

`Sec-WebSocket-Accept`的值是服务端采用与客户端一致的密钥计算出来后返回客户端的，`HTTP/1.1 101 Switching Protocols`表示服务端接受WebSocket协议的客户端连接，经过这样的请求-响应处理后，两端的WebSocket连接握手成功, 后续就可以进行TCP通讯了。

在开发方面，WebSocket API 也十分简单：只需要实例化 WebSocket，创建连接，然后服务端和客户端就可以相互发送和响应消息。在WebSocket 实现及案例分析部分可以看到详细的 WebSocket API 及代码实现。

腾讯云公网有日租类型七层负载均衡转发部分支持Websocket，目前包括英魂之刃、银汉游戏等多家企业已接入使用。当出现不兼容问题时，请修改websocket配置，websocket server不校验下图中圈出的字段：

![img](https://images2018.cnblogs.com/blog/907596/201808/907596-20180821163118288-999851871.png)

比如一个使用WebSocket应用于视频的业务思路如下：
1）使用心跳维护websocket链路，探测客户端端的网红/主播是否在线
2）设置负载均衡7层的proxy_read_timeout默认为60s
3）设置心跳为50s，即可长期保持Websocket不断开

**Nginx代理webSocket经常中断的解决方法（也就是如何保持长连接）**

现象描述：用nginx反代代理某个业务，发现平均1分钟左右，就会出现webSocket连接中断，然后查看了一下，是nginx出现的问题。
产生原因：nginx等待第一次通讯和第二次通讯的时间差，超过了它设定的最大等待时间，简单来说就是超时！

解决方法1
其实只要配置nginx.conf的对应localhost里面的这几个参数就好
proxy_connect_timeout;
proxy_read_timeout;
proxy_send_timeout;

解决方法2
发心跳包，原理就是在有效地再读时间内进行通讯，重新刷新再读时间

配置示例：

```
http {``  ``server {``    ``location / {``      ``root  html;``      ``index index.html index.htm;``      ``proxy_pass http:``//webscoket``;``      ``proxy_http_version 1.1;``      ``proxy_connect_timeout 4s;        ``#配置点1``      ``proxy_read_timeout 60s;         ``#配置点2，如果没效，可以考虑这个时间配置长一点``      ``proxy_send_timeout 12s;         ``#配置点3``      ``proxy_set_header Upgrade $http_upgrade; ``      ``proxy_set_header Connection ``"Upgrade"``; ``    ``}``  ``}``}
```

关于上面配置2的解释
这个是服务器对你等待最大的时间，也就是说当你webSocket使用nginx转发的时候，用上面的配置2来说，如果60秒内没有通讯，依然是会断开的，所以，你可以按照你的需求来设定。比如说，我设置了10分钟，那么如果我10分钟内有通讯，或者10分钟内有做心跳的话，是可以保持连接不中断的，详细看个人需求

**WebSocket与Socket的关系**
\- Socket其实并不是一个协议，而是为了方便使用TCP或UDP而抽象出来的一层，是位于应用层和传输控制层之间的一组接口。当两台主机通信时，必须通过Socket连接，Socket则利用TCP/IP协议建立TCP连接。TCP连接则更依靠于底层的IP协议，IP协议的连接则依赖于链路层等更低层次。
\- WebSocket就像HTTP一样，则是一个典型的应用层协议。
总的来说：Socket是传输控制层接口，WebSocket是应用层协议。