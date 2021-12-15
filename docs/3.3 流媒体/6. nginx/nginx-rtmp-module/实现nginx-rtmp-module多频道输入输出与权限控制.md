[实现nginx-rtmp-module多频道输入输出与权限控制](http://www.ptbird.cn/nginx-rtmp-multi-channel.html)

# 一、权限控制方面

文档中主要有两个部分需要注意：

- **live配置的publish_notify部分**
  - https://github.com/arut/nginx-rtmp-module/wiki/Directives#publish_notify
- **publish_notify中Notify的配置部分**
  - https://github.com/arut/nginx-rtmp-module/wiki/Directives#notify

## 1、live的publish_notify

所谓的publish_notify是涉及publish_notify默认是off的，主要涉及推送的过程中一些事件。

开启publish_notify即可进行Notify的配置操作。

```bash
publish_notify on;
```



## 2、Notify的配置

Notify的配置相关是涉及直播的事件并执行回调代码。

比如：推流链接、直播开启、直播结束状态，然后异步调用http的链接，进行一些逻辑的处理。

主要的配置参数有下面这些：

- on_connect
- on_play
- on_publish
- on_done
- on_play_done
- on_publish_done
- on_record_done
- on_update
- ......

从上面的配置参数可以看出，能够触发连接、直播、输出、直播结束等等，从而能够进行权限验证、

比如，当触发推流的时候，通过 on_publish http://www.example.com/uri 进行权限控制，接收相关参数并进行控制，如果用户不存在，则不允许推流。

# 二、多频道输入输出

这里的多频道输入输出意思是:多个人直播，每个人有不同的输出地址。



## 1、直播推流端

多个人有不同的推流和直播地址，就涉及了直播参数，而实际上，各大平台直播的时候，除了地址，都有一个直播密钥或者是直播码。

以OBS举例，**串流类型选择自定义流媒体服务器，然后会出现一个URL和流密钥**。

![rtmp1.jpg](http://www.ptbird.cn/usr/uploads/2017/03/1478188502.jpg)

而流密钥就是实现多频道输入输出的重点。



## 2、rtmp Publish配置

既然需要进行权限控制，就要使用publish，首先进行权限的验证，证明有推流权限。

所以rtmp的配置如下（最后会给出一个完整的配置示例）：

因为只是探讨权限控制，因此hls之类的不需要关心，我也注释了。

```bash
    #设置直播的application名称是 myapp
    application myapp{ 
         live on; #live on表示开启直播模式        
         publish_notify on;
         on_publish http://tp5.ptbird-ubuntu/on_publish.html;
         #hls on;
         #hls_path /tmp/hls;
         #hls_fragment 2s;
         #hls_playlist_length 6s;
    }
```

关键配置是：**on_publish http://tp5.ptbird-ubuntu/on_publish.html;**

后面的 **http://tp5.ptbird-ubuntu/on_publish.html** 是假设在web服务器上的处理程序，网上有的将这个逻辑假设在本机nginx，我是建议不要混在一起，直接在别的能够连接的web服务器上部署即可。

## 3、权限验证URL

一个示例的配置如下所示，这是一个模板配置示例。

![rtmp2.jpg](http://www.ptbird.cn/usr/uploads/2017/03/2805500523.jpg)

流密钥的格式是：test?pass=123456，可以看出这个非常像url的get参数配置，test就是用户的name，而pass就是密码。

为什么没有name=test

- 因为name是rtmp on_publish的默认参数，name是不能更改的。

绝对不能使用GET['name']，而应该使用POST

- 一开始我总是使用get去获取参数，但是发现一直无法成功，也没办法验证 - -
- 后来网上查了查，发现不能使用get获取，虽然流密钥的格式像是get类型，但是必须使用POST获取参数。

自定义参数

- 除了name不能更改之外，其他的都是可以自定义参数的
- 比如pass=123456&check=123456这样的