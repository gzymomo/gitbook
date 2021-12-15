[TOC]

[思否：民工哥](https://segmentfault.com/a/1190000022589937)

# 一、ping 基本使用详解
在网络中 ping 是一个十分强大的 TCP/IP 工具。它的作用主要为：

1. 用来检测网络的连通情况和分析网络速度
2. 根据域名得到服务器 IP
3. 根据 ping 返回的 TTL 值来判断对方所使用的操作系统及数据包经过路由器数量。

我们通常会用它来直接 ping ip 地址，来测试网络的连通情况。
![](https://segmentfault.com/img/remote/1460000022589941)

类如这种，直接 ping ip 地址或网关，ping 通会显示出以上数据，有朋友可能会问，`bytes=32；time<1ms；TTL=128` 这些是什么意思。

- bytes 值：数据包大小，也就是字节。
- time 值：响应时间，这个时间越小，说明你连接这个地址速度越快。
- TTL 值：Time To Live, 表示 DNS 记录在 DNS 服务器上存在的时间，它是 IP 协议包的一个值，告诉路由器该数据包何时需要被丢弃。可以通过 Ping 返回的 TTL 值大小，粗略地判断目标系统类型是 Windows 系列还是 UNIX/Linux 系列。

默认情况下，Linux 系统的 TTL 值为 64 或 255，WindowsNT/2000/XP 系统的 TTL 值为 128，Windows98 系统的 TTL 值为 32，UNIX 主机的 TTL 值为 255。

因此一般 TTL 值：

- 100~130ms 之间，Windows 系统 ；
- 240~255ms 之间，UNIX/Linux 系统。

![](https://segmentfault.com/img/remote/1460000022589942)