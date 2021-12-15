[TOC]

# 1、fast
fast 是 Netflix 提供的一项服务，它不仅可以通过命令行来使用，而且可以直接在 Web 端使用：fast.com。

通过以下命令来安装这个工具：
`$ npm install --global fast-cli`

不管是网页端还是命令行，它都提供了最基本的网络下载测速。命令行下最简单的使用方法如下：
```bash
$ fast
    93 Mbps ↓
```

从以上结果可以看出，直接使用 fast 命令的话，将只返回网络下载速度。如果你也想获取网络的上传速度，则需要使用 -u 选项。
```bash
$ fast -u
    ⠧ 81 Mbps ↓ / 8.3 Mbps ↑
```

# 2、speedtest
speedtest 是一个更加知名的工具。它是用 Python 写成的，可以使用 apt 或 pip 命令来安装。你可以在命令行下使用，也可以直接将其导入到你的 Python 项目。

安装方式：
```bash
$ sudo apt install speedtest-cli
或者
$ sudo pip3 install speedtest-cli
```
使用的时候，可以直接运行 speedtest 命令即可：
```bash
$ speedtest
Retrieving speedtest.net configuration...
Testing from Tencent cloud computing (140.143.139.14)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Henan CMCC 5G (Zhengzhou) [9.69 km]: 28.288 ms
Testing download speed................................................................................
Download: 56.20 Mbit/s
Testing upload speed......................................................................................................
Upload: 1.03 Mbit/s
```
从运行结果可以看出，speedtest 命令将直接提供上传/下载速率，测试的过程也是挺快的。你可以编写一个脚本来调用这个命令，然后定期进行网络测试，并在结果保存在一个文件或数据库，这样你就可以实时跟踪你的网络状态。

# 3、iPerf
iperf 是一个网络性能测试工具，它可以测试 TCP 和 UDP 带宽质量，可以测量最大 TCP 带宽，具有多种参数和 UDP 特性，可以报告带宽，延迟抖动和数据包丢失。利用 iperf 这一特性，可以用来测试一些网络设备如路由器，防火墙，交换机等的性能。

Debian 系的发行版可以使用如下命令安装 iPerf ：
`$ sudo apt install iperf`
这个工具不仅仅在 Linux 系统下可以用，在 Mac 和 Windows 系统同样可以使用。

如果你想测试网络带宽，则需要两台电脑。这两台电脑需要处于同样的网络，一台作为服务机，另一台作为客户机，并且二者必须都要安装 iPerf 。

可以通过如下命令获取服务器的 IP 地址：
```bash
$ ip addr show | grep inet.*brd
    inet 192.168.242.128/24 brd 192.168.242.255 scope global dynamic noprefixroute ens33
```
我们知道，在局域网里，我们的 ipv4 地址一般是以 192.168 开头的。运行以上命令之后，我们需要记下服务机的地址，后面会用到。

之后，我们再在服务机上启动 iperf 工具：
` $ iperf -s `
然后，我们就可以等待客户机的接入了。客户机可以使用以下命令来连上服务机：
` $ iperf -c 192.168.242.128 `
通过几秒钟的测试，它就会返回网络传输速率及带宽。