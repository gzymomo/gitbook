## 推流测试

ZLMediaKit支持rtsp/rtmp/rtp推流，一般通常使用obs/ffmpeg推流测试，其中FFmpeg推流命令支持以下：

- 1、使用rtsp方式推流

```
# h264推流
ffmpeg -re -i "/path/to/test.mp4" -vcodec h264 -acodec aac -f rtsp -rtsp_transport tcp rtsp://127.0.0.1/live/test
# h265推流
ffmpeg -re -i "/path/to/test.mp4" -vcodec h265 -acodec aac -f rtsp -rtsp_transport tcp rtsp://127.0.0.1/live/test
```

- 2、使用rtmp方式推流

```
#如果未安装FFmpeg，你也可以用obs推流
ffmpeg -re -i "/path/to/test.mp4" -vcodec h264 -acodec aac -f flv rtmp://127.0.0.1/live/test
# RTMP标准不支持H265,但是国内有自行扩展的，如果你想让FFmpeg支持RTMP-H265,请按照此文章编译：https://github.com/ksvc/FFmpeg/wiki/hevcpush
```

- 3、使用rtp方式推流

```
# h264推流
ffmpeg -re -i "/path/to/test.mp4" -vcodec h264 -acodec aac -f rtp_mpegts rtp://127.0.0.1:10000
# h265推流
ffmpeg -re -i "/path/to/test.mp4" -vcodec h265 -acodec aac -f rtp_mpegts rtp://127.0.0.1:10000
```

## 观察日志

如果推流成功，会打印这种日志： ![image](https://user-images.githubusercontent.com/11495632/78963526-5568dd00-7b2a-11ea-850b-0af7d022aa2e.png)

日志中相关字符串分别代表：

```
2020-04-10 12:51:52.331 I | regist rtsp __defaultVhost__ rtp 206442D7
                                    ^           ^         ^      ^
                                  schema      vhost      app stream_id
```

## 播放地址

请按照[播放url规则](https://github.com/xiongziliang/ZLMediaKit/wiki/播放url规则)来播放上述的推流。