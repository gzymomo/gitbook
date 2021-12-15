# 一、Nginx概述

Nginx 是一款轻量级的 Web 服务器 、反向代理服务器及电子邮件（IMAP/POP3）代理服务器。

主要的优点是：

1. 支持高并发连接，尤其是静态界面，官方测试Nginx能够支撑5万并发连接
2. 内存占用极低
3. 配置简单，使用灵活，可以基于自身需要增强其功能，同时支持自定义模块的开发
4. 使用灵活：可以根据需要，配置不同的负载均衡模式，URL地址重写等功能
5. 稳定性高，在进行反向代理时，宕机的概率很低
6. 支持热部署，应用启动重载非常迅速



> 反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。



## 1.1 nginx在架构体系中的作用

- 网关 （面向客户的总入口，可以简单的理解为用户请求和服务器响应的关口，即面向用户的总入口，网关可以拦截客户端所有请求，对该请求进行权限控制、负载均衡、日志管理、接口调用监控等，因此无论使用什么架构体系，都可以使用`Nginx`作为最外层的网关）
- 虚拟主机（为不同域名 / ip / 端口提供服务。如：VPS虚拟服务器）
- 路由（正向代理 / 反向代理）
- 静态服务器
- 负载集群（提供负载均衡）



## 1.2 nginx架构设计

![image-20210207140356655](http://lovebetterworld.com/image-20210207140356655.png)



（1）核心模块；

核心模块是Nginx服务器正常运行必不可少的模块，如同操作系统的内核。它提供了Nginx最基本的核心服务。像进程管理、权限控制、错误日志记录等；

（2）标准HTTP模块；

标准HTTP模块支持标准的HTTP的功能，如：端口配置，网页编码设置，HTTP响应头设置等；

（3）可选HTTP模块；

可选HTTP模块主要用于扩展标准的HTTP功能，让Nginx能处理一些特殊的服务，如：解析GeoIP请求，SSL支持等；

（4）邮件服务模块；

邮件服务模块主要用于支持Nginx的邮件服务；

（5）第三方模块；

第三方模块是为了扩展Nginx服务器应用，完成开发者想要的功能，如：Lua支持，JSON支持等；

> 模块化设计使得Nginx方便开发和扩展，功能很强大



## 1.3 nginx请求处理过程

在请求的处理阶段也会经历诸多的过程，`Nginx`将各功能模块组织成一条链，当有请求到达的时候，请求依次经过这条链上的部分或者全部模块，进行处理，每个模块实现特定的功能。

一个 HTTP Request 的处理过程：

- 初始化 HTTP Request
- 处理请求头、处理请求体
- 如果有的话，调用与此请求（URL 或者 Location）关联的 handler
- 依次调用各 phase handler 进行处理
- 输出内容依次经过 filter 模块处理



![image-20210207140500639](http://lovebetterworld.com/image-20210207140500639.png)





# 二、nginx事件模型

## 2.1 nginx的多进程模型

Nginx 在启动后，会有一个 `master`进程和多个 `worker`进程。

`master`进程主要用来管理`worker`进程，包括接收来自外界的信号，向各 worker 进程发送信号，监控 worker 进程的运行状态以及启动 worker 进程。

`worker`进程是用来处理来自客户端的请求事件。多个 worker 进程之间是对等的，它们同等竞争来自客户端的请求，各进程互相独立，一个请求只能在一个 worker 进程中处理。worker 进程的个数是可以设置的，一般会设置与机器 CPU 核数一致，这里面的原因与事件处理模型有关

Nginx 的进程模型，可由下图来表示：

![image-20210207140530548](http://lovebetterworld.com/image-20210207140530548.png)



这种设计带来以下优点：

**1） 利用多核系统的并发处理能力**

现代操作系统已经支持多核 CPU 架构，这使得多个进程可以分别占用不同的 CPU 核心来工作。Nginx 中所有的 worker 工作进程都是完全平等的。这提高了网络性能、降低了请求的时延。

**2） 负载均衡**

多个 worker 工作进程通过进程间通信来实现负载均衡，即一个请求到来时更容易被分配到负载较轻的 worker 工作进程中处理。这也在一定程度上提高了网络性能、降低了请求的时延。

**3） 管理进程会负责监控工作进程的状态，并负责管理其行为**

管理进程不会占用多少系统资源，它只是用来启动、停止、监控或使用其他行为来控制工作进程。首先，这提高了系统的可靠性，当 worker 进程出现问题时，管理进程可以启动新的工作进程来避免系统性能的下降。其次，管理进程支持 Nginx 服务运行中的程序升级、配置项修改等操作，这种设计使得动态可扩展性、动态定制性较容易实现。



## 2.2 多线程的方式

Apache 的 worker模式和event模式就是多线程方式，当收到客户端的请求时，由服务器的工作进程生成一个线程来处理。

该方式的优势是大大增强了服务器处理并发请求的能力，但是socket本身是阻塞状态的，当有10万并发连接访问时，就需要10万的线程来维护这些请求，线程的上下文切换非常平台，会引发很严重的性能问题。



## 2.3 异步方式

- 同步和异步

同步是指在发送方发出消息后，需要等待接收到接收方发回的响应，或者通过回调函数来接收到对方响应信息。

异步是指在发送方发出请求后，接收方不返回消息或者不等待返回消息。

- 阻塞和非阻塞

在网络通讯中，阻塞和非阻塞主要是指Socket的阻塞和非阻塞方式，而socket的实质是IO操作。

阻塞是指在IO操作返回结果之前，当前的线程处于被挂起状态，直到调用结果返回后才能处理其它新的请求。

非阻塞指在IO操作返回结果之前，当前的线程会继续处理其它的请求。

- 同步阻塞方式(常用,简单)

发送方向接收方发送请求后，一直处理等待响应状态。接收方处理请求时进行IO操作，如果该操作没有立刻返回结果，将继续等待，直到返回结果后，才响应发送方，期间不能处理其它请求。

- 异步非阻塞(常用)

发送方向接收方发送请求后，继续进行其它工作。接收方处理请求进行IO操作，如果没有立刻返回结果，将不再等待，而是处理其它请求。



# 三、nginx的进程

## 3.1 进程的分类

### 3.1.1. master process

Nginx启动时运行的主要进程，主要功能如下

- 读取Nginx配置文件
- 建立、绑定、关闭socket.
- 按照配置生成、管理进程.
- 接收外界指令，如重启、退出等
- 编译和处理Perl脚本
- 开启日志记录

### 3.1.2. worker processes

- 接收并处理客户端请求
- 访问和调用缓存数据
- 接收主程序的指令

### 3.1.3. cache loader

在开启缓存服务器功能下，在Nginx主进程启动一段时间后(默认1分钟)，由主进程生成cache loader，在缓存索引建立完成后将自动退出。

### 3.1.4. Cache Manager

在开启缓存服务器功能下，在Nginx主进程的整个生命周期内，管理缓存索引，主要对索引是否过期进程判断。

## 3.2. 进程之间的交互

### 3.2.1. 主进程与工作进程的交互

主进程生成工作进程的时候，会将工作进程添加到工作进程列表中，并建立一个主进程到工作进程的单向管道。当主进程需要与工作进程进行交互时，主进程通过管道向工作进程发出指令，工作进程读取管道中的数据。

### 3.2.2. 工作进程与工作进程的交互

主进程在生成工作进程时，会将工作进程添加到工作进程列表中，并将该进程的ID等信息通过管道传递给其他进程。

当工作进程需要与其他工作进程通讯时，会从主进程给予的工作进程信息中找到对方进程的ID，然后建立与对方进程的管道，进行信息传递。



# 四、nginx的安装部署

## 4.1 方式一：二进制安装

Nginx 是 C语言 开发，建议在 Linux 上运行，当然，也可以安装 Windows 版本。

### 1. 基础环境配置

**一. gcc 安装**

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```bash
yum install gcc-c++
```

**二. PCRE pcre-devel 安装**

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

```bash
yum install -y pcre pcre-devel
```

**三. zlib 安装**

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```bash
yum install -y zlib zlib-devel
```

**四. OpenSSL 安装**

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。 nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```shell
yum install -y openssl openssl-devel
```

### 2. 官网下载安装包

1.直接下载`.tar.gz`安装包，地址：https://nginx.org/en/download.html

![nginx.png](http://www.linuxidc.com/upload/2016_09/160905180451092.png)

2.使用`wget`命令下载（推荐）。

```bash
wget -c https://nginx.org/download/nginx-1.16.1.tar.gz
```

![nginx-wget.png](http://www.linuxidc.com/upload/2016_09/160905180451091.png)

### 3. 解压

依然是直接命令：

```bash
tar -zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
```

### 4. 配置

1. 使用默认配置

```bash
./configure
```

2. 自定义配置（不推荐，除非需要自定义一些动态的module，比如流媒体或其他方面的使用）

```bash
./configure \
--prefix=/usr/local/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/conf/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

> 注：将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录

### 5. 编译安装

```bash
make
make install
```

查找安装路径：

```bash
whereis nginx
```

### 6. 启动、停止nginx

```bash
cd /usr/local/nginx/sbin/
# 启动命令
./nginx 

# 停止命令
./nginx -s stop

# 终止命令
./nginx -s quit

# 重新加载
./nginx -s reload
```

> `./nginx -s quit`:此方式停止步骤是待nginx进程处理任务完毕进行停止。 `./nginx -s stop`:此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。

查询nginx进程：

```bash
ps aux | grep nginx
```

### 7. 重启 nginx

1. 先停止再启动（推荐）： 对 nginx 进行重启相当于先停止再启动，即先执行停止命令再执行启动命令。如下：

```bash
cd /usr/local/nginx/sbin/
# 先停止在启动
./nginx -s quit
./nginx
```

2. .重新加载配置文件： 当 ngin x的配置文件 nginx.conf 修改后，要想让配置生效需要重启 nginx，使用`-s reload`不用先停止 ngin x再启动 nginx 即可将配置信息在 nginx 中生效，如下： 


```bash
cd /usr/local/nginx/sbin/
# 检查配置文件是否有误
./nginx -t
# 重新加载配置文件
./nginx -s reload
```

### 8. 开机启动

即在`rc.local`增加启动代码就可以了。

```bash
vi /etc/rc.local
```

增加一行 `/usr/local/nginx/sbin/nginx` 设置执行权限：

```bash
chmod 755 rc.local
```

![nginx-rclocal.png](http://www.linuxidc.com/upload/2016_09/160905180451095.png)

到这里，nginx就安装完毕了，启动、停止、重启操作也都完成了。



## 4.2 方式二：Docker方式安装

只需要执行一条命令即可：

```bash
docker run -d -p 80:80 --name nginx-8095 -v /var/project/nginx/html:/usr/share/nginx/html -v /etc/nginx/conf:/etc/nginx -v /var/project/logs/nginx:/var/log/nginx nginx
```

修改完配置文件，只需重启Docker 容器即可

```
docker restart nginx
```



# 参考资源文档：

- [nginx配置静态资源访问](https://www.cnblogs.com/qingshan-tang/p/12763522.html)
- [微信公众号：民工哥技术之路：dunwu](https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247494790&idx=2&sn=acc3075a8b5e59824baf31a2b81b560b&chksm=e918899ade6f008cf7abb6864dbf8ca3c3e94e54c66115272a36f2df390c44ce0538ad4de1a1&scene=126&sessionid=1592266153&key=4b0291f9d497b1b0e5b13537eabdfd65f20e3711275ab33a24b1d57b7c3d58d658efa1c467b3b39845ea1832cc3275dd9265d9471b9fc4fc5008b22b4fabd2c0b8396baf2f796f35d88e133adb455b2a&ascene=1&uin=MjkxMzM3MDgyNQ%3D%3D&devicetype=Windows+10+x64&version=62090070&lang=zh_CN&exportkey=A7TErMfk%2F0DnI53DaeMrKg4%3D&pass_ticket=LPvgPt7Z83HVbqEBiCUd4DpwbsYcB0xQVnfUjoxD5EVKc%2FNZ85pmeMbzepgknOxS)
- [Nginx日志的标准格式](http://www.cnblogs.com/kevingrace/p/5893499.html)
- [民工哥](https://segmentfault.com/u/jishuroad)：[Nginx + Spring Boot 实现负载均衡](https://segmentfault.com/a/1190000037594169)
- [Nginx 高性能优化配置实战总结](https://segmentfault.com/a/1190000037788252)
- 渡渡鸟：[02-Nginx事件模型](https://www.yuque.com/duduniao/nginx/ileqg3)

