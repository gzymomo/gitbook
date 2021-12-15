- [主流相机 RTSP地址格式](https://blog.csdn.net/qq_34654240/article/details/79924390)

# 海康威视

rtsp://[username]:[password]@[ip]:[port]/[codec]/[channel]/[subtype]/av_stream
说明：
username: 用户名。例如admin。
password: 密码。例如12345。
ip: 为设备IP。例如 192.0.0.64。
port: 端口号默认为554，若为默认可不填写。
codec：有h264、MPEG-4、mpeg4这几种。
channel: 通道号，起始为1。例如通道1，则为ch1。
subtype: 码流类型，主码流为main，辅码流为sub。


例如，请求海康摄像机通道1的主码流，Url如下
主码流：
rtsp://admin:12345@192.0.0.64:554/h264/ch1/main/av_stream
rtsp://admin:12345@192.0.0.64:554/MPEG-4/ch1/main/av_stream
子码流：
rtsp://admin:12345@192.0.0.64/mpeg4/ch1/sub/av_stream
rtsp://admin:12345@192.0.0.64/h264/ch1/sub/av_stream

# 大华

rtsp://[username]:[password]@[ip]:[port]/cam/realmonitor?[channel]&[subtype]
说明:
username: 用户名。例如admin。
password: 密码。例如admin。
ip: 为设备IP。例如 10.7.8.122。
port: 端口号默认为554，若为默认可不填写。
channel: 通道号，起始为1。例如通道2，则为channel=2。
subtype: 码流类型，主码流为0（即subtype=0），辅码流为1（即subtype=1）。


例如，请求某设备的通道2的辅码流，Url如下
rtsp://admin:admin@10.12.4.84:554/cam/realmonitor?channel=2&subtype=1

# D-Link

rtsp://[username]:[password]@[ip]:[port]/[channel].sdp
说明：
username：用户名。例如admin
password：密码。例如12345，如果没有网络验证可直接写成rtsp:// [ip]:[port]/[channel].sdp
ip：为设备IP。例如192.168.0.108。
port：端口号默认为554，若为默认可不填写。
channel：通道号，起始为1。例如通道2，则为live2。


例如，请求某设备的通道2的码流，URL如下
rtsp://admin:12345@192.168.200.201:554/live2.sdp

# Axis（安讯士）

rtsp://[username]:[password]@[ip]/axis-media/media.amp?[videocodec]&[resolution]
说明：
username：用户名。例如admin
password：密码。例如12345，如果没有网络验证可省略用户名密码部分以及@字符。
ip：为设备IP。例如192.168.0.108。
videocodec：支持MPEG、h.264等，可缺省。
resolution：分辨率，如resolution=1920x1080，若采用默认分辨率，可缺省此参数。

例如，请求某设备h264编码的1280x720的码流，URL如下：
rtsp:// 192.168.200.202/axis-media/media.amp?videocodec=h264&resolution=1280x720

# 宇视相机

rtsp://{用户名}:{密码}@{ip}:{port}/video1/2/3，分别对应主/辅/三码流；

比如：

    rtsp://admin:admin@192.168.8.8:554/video1，就表示主码流；
    rtsp://admin:admin@192.168.8.8:554/video2，表示子码流；
    rtsp://admin:admin@192.168.8.8:554/video3，表示3码流；

# 凌云

rtsp://192.168.0.100/live1.sdp

# EB-01

rtsp://192.168.1.10/user=admin&password=&channel=1&stream=0.sdp

# C2

rtsp://192.168.1.10/user=admin&password=&channel=1&stream=0.sdp?

# C2S 抓拍机

rtsp://admin:password@192.168.1.10



另附某些相机rtsp地址格式：

3S: rtsp://IP地址/cam1/h264
4XEM: rtsp://IP地址/live.sdp
A-MTK: rtsp://IP地址/media/media.amp
ABS: rtsp://IP地址/mpeg4/1/media.amp
Absolutron: rtsp://IP地址/image.mpg
ACTi: rtsp://192.168.0.100:7070/
ACTi: rtsp://192.168.0.100/
Acumen: rtsp://IP地址/mpg4/rtsp.amp
Airlink101: rtsp://IP地址/mpeg4
AirLive: rtsp://IP地址/video.mp4
ALinking: rtsp://IP地址/cam1/mjpeg
ALinking: rtsp://IP地址/cam1/mpeg4
ALinking: rtsp://IP地址/cam1/h264
ALLIEDE: rtsp://IP地址:555/0/1:1/main
Asante: rtsp://IP地址/cam1/mpeg4
Asoni: rtsp://IP地址/GetData.cgi
Asoni: rtsp://IP地址/
Aviosys: rtsp://IP地址/mpeg4
Aviosys: rtsp://IP地址:8554/mpeg4
AVS: Uriel: rtsp://IP地址/mpeg4
AVTech: rtsp://IP地址/live/mpeg4
AVTech: rtsp://IP地址/live/h264
安迅士/AXIS: rtsp://IP地址/mpeg4/media.amp
安迅士/AXIS: rtsp://IP地址/安迅士/AXIS-media/media.amp
AXview: rtsp://IP地址
Basler: rtsp://192.168.100.x/mpeg4
Basler: rtsp://IP地址/h264?multicast
BiKal: IP: CCTV: rtsp://IP地址/
BiKal: IP: CCTV: rtsp://IP地址/user.pin.mp2
BlueJay: rtsp://IP地址/mpeg4
博士/Bosch: rtsp://192.168.0.1/rtsp_tunnel
博士/Bosch: rtsp://192.168.0.1/video
博士/Bosch: rtsp://192.168.0.1/?inst=2
金砖通讯/Brickcom: rtsp://192.168.1.1/channel1
佳能/Canon: rtsp://192.168.100.1/
佳能/Canon: rtsp://192.168.100.1/stream/profile1=u
佳能/Canon: rtsp://192.168.100.1/profile1=r
佳能/Canon: rtsp://192.168.100.1/profile1=u
CBC-Ganz: rtsp://IP地址/gnz_media/main
思科/Cisco: rtsp://IP地址/img/media.sav
思科/Cisco: rtsp://IP地址
Clairvoyant: MWR: rtsp://IP地址/av0_0
喜恩碧/CNB: rtsp://192.168.123.100/
喜恩碧/CNB: rtsp://192.168.123.100/mpeg4
Cohu: rtsp://IP地址/stream1
Cohu: rtsp://IP地址/cam
Compro: rtsp://IP地址/medias1
大华/Canon: rtsp://admin:admin@192.168.1.108:554/cam/realmonitor?channel=1&subtype=1
D-Link: rtsp://IP地址/play1.sdp
D-Link: rtsp://IP地址/play2.sdp
Dallmeier: rtsp://IP地址/encoder1
DoOurBest: rtsp://IP地址/: ch0_0.h264
DVTel-IOimage: rtsp://IP地址/ioImage/1
EagleVision: rtsp://IP地址/11
讯舟科技/EDIMAX: rtsp://IP地址/ipcam.sdp
讯舟科技/EDIMAX: rtsp://IP地址/ipcam_h264.sdp
ENEO: rtsp://IP地址/1/stream1
Etrovision: rtsp://IP地址/rtpvideo1.sdp
EverWorldView: rtsp://IP地址
慧友/EverFocus: rtsp://IP地址/streaming/channels/0
Fine CCTV: rtsp://IP地址/mpeg4
菲力尔/FLIR Systems: rtsp://IP地址/ch0
菲力尔/FLIR Systems: rtsp://IP地址/vis
菲力尔/FLIR Systems: rtsp://IP地址:544/wfov
福斯康姆/Foscam: rtsp://IP地址/11
FSAN: RTSP://IP地址/
Gadspot: rtsp://IP地址/video.mp4
Genie: rtsp://IP地址
Genius: rtsp://IP地址/avn=2
奇偶/GeoVision: rtsp://192.168.0.10:8554/CH001.sdp
潮流网络/Grandstream: rtsp://	192.168.1.168/
GRUNDIG: rtsp://IP地址/jpeg
GRUNDIG: rtsp://IP地址/h264
GVI: rtsp://IP地址/mpeg4
海康威视/Hikvision: rtsp://192.0.0.64:554/h264
HuntElectronics: rtsp://IP地址/video1+audio1
Ikegami: rtsp://IP地址/stream1
iLink: rtsp://IP地址
IndigoVision: rtsp://IP地址
英飞拓/Infinova: rtsp://IP地址/1.AMP
InnovativeSecurityDesigns: rtsp://IP地址/stream1
INSTEK: DIGITAL: rtsp://IP地址/
Intellinet: rtsp://IP地址/video.mp4
Intellio: rtsp://IP地址/
IONodes: rtsp://IP地址/videoinput_1/h264_1/media.stm
IPUX: rtsp://IP地址/mpeg4
IPx: rtsp://IP地址/camera.stm
IQinVision: rtsp://IP地址/now.mp4
IQinVision: rtsp://IP地址/mp4
IRLAB: rtsp://IP地址/
JVC: rtsp://IP地址/PSIA/Streaming/channels/0
JVC: rtsp://IP地址/PSIA/Streaming/channels/1
KARE: CSST-DIT: rtsp://IP地址
KTC: rtsp://IP地址/h264/
朗驰/Launch: rtsp://IP地址/0/username:password/main
朗驰/Launch: rtsp://IP地址:554/0/username:password/main
Laview: rtsp://IP地址/
LevelOne: rtsp://IP地址/access_code
LevelOne: rtsp://IP地址/channel1
LevelOne: rtsp://IP地址/live.sdp
LevelOne: rtsp://IP地址/video.mp4
LevelOne: rtsp://IP地址/h264
Linksys: rtsp://IP地址/img/video.sav
Logitech: rtsp://IP地址/HighResolutionVideo
Lorex: rtsp://IP地址/video.mp4
Lumenera: rtsp://IP地址/
LUXONVIDEO: rtsp://IP地址/user_defined
Marmitek: rtsp://IP地址/mpeg4
MaxVideo: rtsp://IP地址/0/usrnm:pwd/main
MC Electronics: rtsp://IP地址/
MeritLi-Lin: http://IP地址: /rtsph264720p
MeritLi-Lin: http://IP地址/rtsph2641080p
MeritLi-Lin: http://IP地址: /rtsph264
MeritLi-Lin: http://IP地址: /rtsph2641024p
MeritLi-Lin: rtsp://IP地址/rtsph264
MeritLi-Lin: http://IP地址: /rtspjpeg
MeritLi-Lin: http://IP地址: /rtsph264
MESSOA: rtsp://192.168.1.30:8557/h264
MESSOA: rtsp://192.168.1.30/mpeg4
MESSOA: rtsp://192.168.1.30/livestream/
MESSOA: rtsp://192.168.1.30:7070
MicroDigital: rtsp://IP地址/cam0_0
Moxa: rtsp://IP地址/multicaststream
MultiPix: rtsp://IP地址/video1
Onix: rtsp://IP地址/cam0_0
OpenEye: rtsp://IP地址/h264
松下/Panasonic: rtsp://192.168.0.253:port//nphMpeg4/g726-640x48
松下/Panasonic: rtsp://192.168.0.253/nphMpeg4/g726-640x480
松下/Panasonic: rtsp://192.168.0.253/nphMpeg4/nil-320x240
松下/Panasonic: rtsp://192.168.0.253/MediaInput/h264
松下/Panasonic: rtsp://192.168.0.253/nphMpeg4/g726-640x
松下/Panasonic: rtsp://192.168.0.253/MediaInput/mpeg4
派尔高/Pelco: rtsp://IP地址/stream1
PiXORD: rtsp://IP地址
Planet: rtsp://IP地址/ipcam.sdp
Planet: rtsp://IP地址/ipcam_h264.sdp
Planet: rtsp://IP地址/media/media.amp
PRIME: rtsp://IP地址/cam1/h264/multicast
QihanTechnology: rtsp://IP地址
Repotec: rtsp://IP地址/cam1/mpeg4?user='username'&pwd='password'
SafeSky: rtsp://IP地址
三星/Samsung: rtsp://192.168.1.200/mpeg4unicast
三星/Samsung: rtsp://192.168.1.200/mjpeg/media.smp
三星/Samsung: rtsp://192.168.1.200/h264/media.smp
Sanan: rtsp://IP地址/
三洋/Sanyo: rtsp://192.168.0.2/VideoInput/1/h264/1
ScallopImaging: rtsp://IP地址:8554/main
Sentry: rtsp://IP地址/mpeg4
SeyeonTech: -
FlexWATCH: rtsp://IP地址/cam0_1
Sharx: rtsp://IP地址/live_mpeg4.sdp
西门子/Siemens: rtsp://IP地址/img/video.asf
西门子/Siemens: rtsp://IP地址/livestream
Siqura: rtsp://IP地址/mpeg4
Siqura: rtsp://IP地址/h264
Siqura: rtsp://: IP地址/VideoInput/1/h264/1
Siqura: rtsp://: IP地址/VideoInput/1/mpeg4/1
Sitecom: rtsp://IP地址/img/video.sav
索尼/Sony: rtsp://IP地址/media/video1
Sparklan: rtsp://IP地址/mpeg4
Speco: rtsp://192.168.1.7/
StarDot: rtsp://IP地址/nph-h264.cgi
海视云威/StarVedia: rtsp://IP地址/CAM_ID.password.mp2
StoreNet: rtsp://IP地址/stream1
SuperCircuits: rtsp://IP地址/ch0_unicast_firststream
SuperCircuits: rtsp://IP地址/ch0_unicast_secondstream
SuperCircuits: rtsp://IP地址/live/mpeg4
Swann: rtsp://IP地址/mpeg4
TCLINK: rtsp://IP地址/av2
TCLINK: rtsp://IP地址/live.sdp
Topica: rtsp://IP地址:7070
Topica: rtsp://IP地址/h264/media.amp
TP-Link: rtsp://IP地址/video.mp4
TRENDnet: rtsp://IP地址/mpeg4
TRENDnet: rtsp://IP地址/play1.sdp
Truen: rtsp://IP地址/video1
优倍快网络/Ubiquiti: rtsp://IP地址/live/ch00_0
UDP Technology: rtsp://IP地址/ch0_unicast_firststream
Verint: rtsp://IP地址
Vgsion: rtsp://IP地址/11
Vicon: rtsp://{user:password}@IP地址:7070/
Vicon: rtsp://IP地址/access_name_for_stream_1_to_5
Vicon: rtsp://{user:password}@IP地址:7070/
VICON: rtsp://IP地址:8557/h264
Videolarm: rtsp://IP地址/mpeg4/1/media.amp
VisionDigi: rtsp://IP地址/
VisionhitechAmericas: rtsp://IP地址/h264&basic_auth=<base64_union_of_id&pwd>
Visionite: rtsp://IP地址
VISTA: rtsp://IP地址/cam1/h264
VISTA: rtsp://IP地址/
VITEK: rtsp://IP地址/ch0.sdp
晶睿通讯/Vivotek: rtsp://192.168.1.20/live.sdp
Weldex: rtsp://IP地址:7070/h264
雄迈/XM: rtsp://192.168.0.10:554/user=admin&password=&channel=1&stream=0.sdp?
Y-cam: rtsp://IP地址/live_mpeg4.sdp
有看头Yoosee: rtsp://IP地址:554/onvif1
Yudor: rtsp://IP地址/
Zavio: rtsp://IP地址/video.3gp
Zavio: rtsp://IP地址/video.mp4