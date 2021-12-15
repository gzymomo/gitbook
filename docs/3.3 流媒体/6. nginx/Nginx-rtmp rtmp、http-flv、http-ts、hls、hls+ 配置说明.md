[分布式直播系统（三）【Nginx-rtmp rtmp、http-flv、http-ts、hls、hls+ 配置说明】](https://blog.csdn.net/impingo/article/details/99703528)

# 全局配置

| 配置项                     | 参数类型 | 默认值                | 描述                                                         |
| -------------------------- | -------- | --------------------- | ------------------------------------------------------------ |
| server_names_hash_max_size | 整型数   | 512                   | 服务器server_name的最大长度                                  |
| listen                     | 整型数   | 1935                  | rtmp服务监听端口                                             |
| server_name                | 字符串   | 空                    | 作用同http的server_name，允许不同域名使用不同配置            |
| server                     | 配置区块 | 空                    | 作用同http中的server，允许同时配置多个server区块             |
| application                | 配置区块 | 空                    | 作用类似http配置中的location，在rtmp中被称作挂载点           |
| so_keepalive               | 布尔类型 | off                   | tcp socket 选项                                              |
| timeout                    | 时间     | 60000ms               | 连接超时时间，如果tcp连接建立长时间不发数据，服务器会断开这条tcp连接 |
| ping                       | 时间     | 60000ms               | rtmp服务的ping功能，rtmp连接建立后多久开始ping功能           |
| ping_timeout               | 时间     | 30000ms               | rtmp服务多久没有收到ping回复超时                             |
| cache_time                 | 时间     | 0ms                   | 服务器端缓存多少时长的音视频数据，如果设置成0ms则没有秒开效果 |
| low_latency                | 布尔类型 | off                   | 只有在cache_time大于0时有效，当设置为on 服务器在收到新关键帧时会从新关键帧处下发数据 |
| send_all                   | 布尔类型 | off                   | 只有在cache_time大于0时有效，当设置为on 服务器会将缓冲区中的数据一次性下发给客户端 |
| fix_timestamp              | 布尔类型 | off                   | 只有在cache_time大于0时有效，当设置为on 如果推流上来的时间戳出现回滚或者跳跃过大的情况则自动修复 |
| zero_start                 | 布尔类型 | off                   | 只有在cache_time大于0时有效，当设置为on 播放端收到的码流中的时间戳值从0开始 |
| max_streams                | 整型     | 32                    | rtmp通道内最大传输的流个数，rtmp协议允许一个通道内传输多条流 |
| chunk_size                 | 整型     | 4096（单位：字节）    | rtmp传输时一个碎片的长度，值越小cpu消耗越大，值越大内存和延时消耗越大，建议使用默认值 |
| max_message                | 整型     | 1048576（单位：字节） | 单个rtmp消息的最大长度，注意：如果传输大码率视频一个关键帧长度可能大于1MB，所以要根据真实场景调整该值 |
| out_queue                  | 整型     | 2048                  | 发送缓冲区允许缓存的最大帧数（包括视频和音频）               |
| merge_frame                | 整型     | 1048576               | 单次发送的最大帧数（无需修改此配置）                         |
| out_cork                   | 整型     | out_queue/8           | 发送缓冲区中帧数到达该值后就开始丢帧，优先丢B帧、P帧         |
| serverid                   | 字符型   | default               | 该配置起到绑定不同域名的作用，当使用不同的域名访问同一个server区块下的服务时需要使用此配置 |
| hevc_codecid               | 整型     | 12                    | 由于adobe已经停止升级rtmp和flv协议，所以原生的rtmp和flv是没有支持h265标准的，此配置是对rtmp和flv进行扩展，将编码id 12设置为h265编码，从而使服务器支持h265编码 |

# rtmp服务配置

| 配置项              | 参数类型 | 默认值 | 描述                                                         |
| ------------------- | -------- | ------ | ------------------------------------------------------------ |
| live                | 布尔类型 | off    | rtmp服务开关，必须配置成on才能开启rtmp服务                   |
| sync                | 时间     | 300ms  | 当掉包导致时间不连续时，超过多少时间则重新发送音视频头，以使播放器重新开始计算时间 |
| wait_key            | 布尔类型 | off    | 是否一定要等到关键帧才开始给播放端下发数据                   |
| wait_video          | 布尔类型 | off    | 是否一定要等到有视频帧才开始给播放端下发数据                 |
| publish_notify      | 布尔类型 | off    | 是否在收到rtmp的publish消息后给推流端回复消息                |
| play_restart        | 布尔类型 | off    | 是否在收到rtmp的play/stop消息后给播放端回复消息              |
| idle_streams        | 布尔类型 | on     | 当播放端请求的流不存在时是否依旧保持连接不断                 |
| drop_idle_publisher | 时间     | 0s     | 当没有播放端观看某一路流时，多久断开推流端的连接，默认0s代表永不主动断开 |

# http-flv配置

开启http-flv配置的方式如下面的配置方法：
当location与application的名字不同时，需要其后跟 app=xxx 来把location与application绑定起来。如果location与application的名字相同，则无需app=xxx参数绑定。

rtmp推流地址：rtmp://ip/live0/stream-name
对应的http-flv播放地址：http://ip/live0/stream-name 和 http://ip/flv0/stream-name

rtmp推流地址：rtmp://ip/live1/stream-name
对应的http-flv播放地址：http://ip/live1/stream-name 和 http://ip/flv1/stream-name

通过这个配置你可以实现自定义的location名字，无需跟application名保持一致。

```nginx
rtmp {
    server {
        listen 1935;
        application live0 {
            live on;
		}
		application live1 {
		    live on;
		}
    }
}

http {
    server {
        listen 80;
        location flv0 {
            flv_live 1935 app=live0;
        }
        location flv1 {
            flv_live 1935 app=live1;
        }
        location live0 {
            flv_live 1935;
        }
        location live1 {
            flv_live 1935;
        }
    }
}
1234567891011121314151617181920212223242526272829
```

# http-ts服务配置

http-ts的配置方式和http-flv的配置原理完全相同。
rtmp推流地址：rtmp://ip/live0/stream-name
对应的http-ts播放地址：http://ip/live0/stream-name 和 http://ip/ts0/stream-name

rtmp推流地址：rtmp://ip/live1/stream-name
对应的http-ts播放地址：http://ip/live1/stream-name 和 http://ip/ts1/stream-name

```nginx
rtmp {
    server {
        listen 1935;
        application live0 {
            live on;
		}
		application live1 {
		    live on;
		}
    }
}

http {
    server {
        listen 80;
        location ts0 {
            ts_live 1935 app=live0;
        }
        location ts1 {
            ts_live 1935 app=live1;
        }
        location live0 {
            ts_live 1935;
        }
        location live1 {
            ts_live 1935;
        }
    }
}
1234567891011121314151617181920212223242526272829
```

# hls服务配置

| 配置项                | 参数类型     | 默认值          | 描述                                                         |
| --------------------- | ------------ | --------------- | ------------------------------------------------------------ |
| hls                   | 布尔类型     | off             | hls开关，on为开                                              |
| hls_fragment          | 时间         | 5000ms          | 单个ts文件切片的大小                                         |
| hls_max_fragment      | 时间         | hls_fragment*10 | 单个ts文件切片的最大时长。hls切片的规则是尽量以hls_fragment为准，并且让每个ts文件尽量以关键帧开头，所以如果码流中迟迟没有关键帧，这个ts切片就会过大，hls_max_fragment就是为了防止切片过大存在的，当切片时长超过这个值就强制截断。 |
| hls_path              | 字符串       | 空              | ts切片和m3u8文件存放路径                                     |
| hls_playlist_length   | 时间         | 30000ms         | hls_playlist_length/hls_fragment 等于m3u8文件中记录的ts文件条数 |
| hls_sync              | 时间         | 2ms             | 在打包音频时，服务器会通过采样率计算音频时间戳，然后使用计算出来的时间戳进行打包。如果计算出来的时间戳和推送端推送过来的时间戳相差超过该值则使用推送来的时间戳打包。 |
| hls_continuous        | 布尔类型     | on              | 如果hls_path目录下存在上次推流的残余文件，那么这次重新推流是否继续上次的残余文件更新m3u8，如果你非常在意推流端闪断后hls服务刷新带来的影响，请保持这个选项打开。 |
| hls_nested            | 布尔类型     | off             | off时，服务器会将所有流生成的m3u8文件和ts文件放在hls_path指定的一级目录下。on时，服务器会在hls_path一级目录下生成一个以流名命名的目录，在流名命名的目录下生成m3u8文件和ts文件，而且m3u8文件会以index.m3u8命名。 |
| hls_fragment_naming   | 可选的字符串 | sequential      | 此选项用来设置ts文件的命名规则，sequential：使用单调递增的序列号命名，timestamp：使用码流里的时间戳命名，system：使用系统时间戳命名 |
| hls_fragment_slicing  | 可选的字符串 | plain           | 此选项用来配置切片规则，plain：当前正在生成的切片时长如果大于等于hls_max_fragment配置的时间则马上切片，否则要同时满足两个条件才会生成下一个切片，第一、ts文件时长大于等于hls_fragment配置的时间，第二、有新的关键帧产生。aligned：hls_max_fragment的影响，其余规则和plain配置下的切片规则一致 |
| hls_type              | 可选的字符串 | live            | 此选项用来配置hls服务的类型，live：直播模式，m3u8文件实时删除旧的ts文件记录，同时追加新生产的ts文件记录。event：该模式下服务器不会删除m3u8文件中任何数据只会追加新的ts文件记录 |
| hls_max_audio_delay   | 时间         | 300ms           | 服务器在打包ts的音频数据的时候不会将独立的一帧音频内容打包成ts，而是会将n多分音频帧合在一起作为一个PES打包进ts文件中，这个值就是用来控制缓存多久的内容后封装为PES，然后打包进ts文件。 |
| hls_audio_buffer_size | 整型数       | 1048576         | 同理hls_max_audio_delay，这个参数用来设置缓存音频的buffer大小 |
| hls_cleanup           | 布尔类型     | on              | 这个参数用来控制是否实时删除过期的ts文件                     |
| hls_variant           | 字符串       | 无              | 此选项用来实现多码率的hls流，例如低码率的流名叫 xxx_low 并且码率是320000bps，高码率的叫 xxx_high 码率为9600000bps，则需要配置两条配置 hls_variant _low BANDWIDTH=320000; 和 hls_variant _high BANDWIDTH=960000; 那么在请求hls流的时候只需要请求http://ip/location/xxx.m3u8即可让播放器实现自动切片码率的功能 |
| hls_base_url          | 字符串       | 空              | 如果你希望让播放器下载ts文件的时候使用与m3u8不同的域名可以通过这个来修改，例如不配置此项m3u8中ts记录为 xxx.ts，加了此项配置hls_base_url http://pingos.me/path; m3u8文件中ts记录则会变为 http://pingos.me/path/xxx.ts |

# 录像服务

| 配置项                   | 参数类型 | 默认值    | 描述                                                       |
| ------------------------ | -------- | --------- | ---------------------------------------------------------- |
| live_record              | 布尔类型 | off       | 录像功能开关，开启后将在指定目录存储直播内容文件（ts格式） |
| live_record_path         | 字符串   | 空        | 录像根目录，将在该目录下生成ts文件                         |
| live_record_interval     | 时间     | 600000ms  | 录制过程中重新打开索引文件和ts文件的时间间隔               |
| live_record_min_fragment | 时间     | 8000ms    | 录制过程中索引文件里记录的最小分片大小                     |
| live_record_max_fragment | 时间     | 12000ms   | 录制过程中索引文件里记录的最大分片大小                     |
| live_record_buffer       | 整型数   | 1024*1024 | 录制过程中数据缓冲大小                                     |

