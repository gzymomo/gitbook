nginx配置文件详细解析：

# 一、nginx的hls配置实例

```yaml
rtmp
{
 
  server
  {
     listen 1935;
     chunk_size 8192;
 
   #vod config
 
     application vod
     {
        play /var/vod/flv;
     }
 
   #live config
 
     application live
     {
        live on;
        max_connections 1024;
        allow play all;
        
        record_path /var/live;
 
        recorder audio
        {
           record audio;
           record_suffix -%d-%b-%y-%T.flv;
        } 
        recorder chunked
        {
           record all;
          #record_max_size 5120K;
           record_interval 15s;
           record_path /var/live/chunked;
        }
     }
 
   #hls
 
      application hls
      {
        live on;
        hls on;
        hls_path /var/hls;
        hls_playlist_length 30s;
        hls_sync 100ms;        
        meta on;
        recorder chunked
        {
           record all;
          #record_max_size 5120K;
           record_interval 15s;
           record_path /var/hls/Chunked;
        }
        recorder audio 
        {
           record audio;
           record_suffix -%d-%b-%y-%T.flv;
        }
      }
 
   }
}
```

# 二、配置详解

> 配置指令的解释基于nginx官方的2013年nginx-rtmp-model文档



## 1、rtmp{}

是一个用于保存所有rtmp配置的块

在这里就是rtmp直播录播配置的根

## 2、server{}

server块里面放服务器实例，比如配置里的三个application（application在第6个解释）

## 3、listen

listen比较好理解，监听某个端口，让nginx监听并接收rtmp连接

## 4、chunk_size

接收网络流的块大小，接触过NIO的应该比较清楚，基于块比基于流效率要高，chunk_size默认值是4096，至少128，数字越大服务器负载越高，服务器调优这里较为重要

## 5、注释：#

nginx配置文件里使用‘#’作为注释

## 6、application

见名知意，创建一个应用/实例，后面接上应用实例类型，如上配置，创建了三个应用，比如上面配置的三个服务器应用实例：（1）rtmp录播（vod），（2）rtmp直播(live)，（3）hls直播(hls)

**重要：rtmp模块的配置与nginx的http模块是两种不同的配置指令，两者不要混淆**

## 7、vod(录播)配置详解

play  录播的播放目录

## 8、live（直播）配置详解

### （1）live  on/off

直播模式，一对多广播

### （2）max_connections

最大连接数，rtmp直播流协议属于长连接协议，服务器能开多少连接要把握好，hls协议是基于http的协议，所以理论上要比rtmp协议并发量要高很多

### （3）allow play/publish all/ip地址

允许来自指定的地址/所有地址播放和发布

比如上面配置使用allow play all允许所有地址播放实时流，如果设置成allow play 127.0.0.1 就是只允许本地播放；

再举个例子：allow publish 127.0.0.1就是允许本机发布实时流，其他地址都不能发布。

### （4）record_path

用来指定录制文件的目录，默认是/tmp

### （5）record  off/all/video/audio/keyframes/maual

record off：什么都不录制

record all:录制所有

record video:只录制视频

record audio:只录制音频

record keyframes:只录制关键帧

record maual:通过接口控制录制的启动停止

可以进行组合，比如：record video keyframes:就是只录制视频关键帧

### （6）record suffix

录制文件的文件名后缀，默认是.flv

比如上面的配置 record suffix -%d-%b-%y-%T.flv，录制文件生成文件名就是这样（举例）：应用名-24-Jul-04-17:07:45.flv

### （7）record_max_size

上面配置：record_size 5120k，录制文件的最大值是5M

### （8）record_interval

配置里的：record_interval 15s，就是录制文件的间隔，间隔15秒开始下一个片段的录制；

设置成 record_interval 0就是录制视频文件没有间隔；

设置成record-interval off就会把所有视频流全都写到一个文件里去。

**重要：想要把流媒体保存文件，这个可以用来做文件分片，可以按天或者按小时生成新的文件，很实用的功能**

## 9、hls直播配置详解

### （1）hls on 

这个参考 live on就行了，很简单，就是开不开启hls，hls off就是不开启

### （2）hls_path

就是录制视频文件的目录/路径

### （3）hls_playlist_length

hls播放列表长度，默认30分钟，这里设置成30秒：hls_playlist_length 30s

### （4）hls_sync

设置hls时间戳同步阈值，通俗一点就是强制的音/视频同步，可以防止音画不同步的现象，默认是2ms，

### （5）meta on

切换发送视频元数据到客户端，默认就是meta on，如果想要用修改后的视频得用meta off了

# 三、常用的nginx rtmp指令

## 1、recorder块

创建录制块，可以在application块中添加多个recorder记录，recorder块中可以使用所有录制指令，recorder块继承application块中的录制指令。（所有record开头的都是录制指令）。




## live指令补充：

### 2、wait_key 

wait_key指令是否等待视频从一个关键帧开始（录播想要实现视频的进度随意控制就需要视频中存在关键帧），

默认是off：不开启，设置为on：开启。

### 3、drop_idle_publisher

终止指定时间内空闲的发布连接，默认是不开启，如

### drop_idle_publisher 10s; *推荐开启该功能*



### 4、sync

同步音频和视频，默认是 sync 300ms。

这是很常见的问题，如果用户的带宽不足就会自动丢掉一些帧，这时就会产生同步问题；

sync指令作用就是隔一段时间发送一个时间戳来同步音频和视频。

### 5、meta

发送元数据到客户端，默认是 meta on；meta off就是不发送元数据，一般不需要更改。

（音视频的元数据包含一些视频的基本信息，比如标题，文件大小，关键帧，视频长度等等信息）



## record指令补充：

### 1、record_unique

是否添加时间戳到录制文件，默认是 record_unique off：不添加

不开启的效果就是同样的录制文件每次录制的时候都会被覆盖重写，当然可以通过给文件名添加时间后缀的形式避免，其实两个效果是一样的，一个可以控制格式，一个不能控制格式的问题

### 2、record_append

切换文件的附加模式，默认是record_append off

如果设置成on开启的话录制的时候就会在老文件后面追加视频，不开启的效果就是上面讲到的覆盖重写

### 3、record_lock

录制的时候锁定文件，默认是off；

不锁定的话，客户端可以实时播放录制的视频，可以达到跟直播一样的效果了

### 4、record_max_frames

录制文件的最大视频帧数量，比如record_max_frames 20 就是这个录制文件最大20个视频帧（视频帧的作用上面已经解释过了，用来控制进度拖放的）