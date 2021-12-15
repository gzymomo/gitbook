# PrettyZoo 

基于 Apache Curator + JavaFX 实现的 ZooKeeper GUI 客户端。

- 可同时管理多个 ZooKeeper 连接
- ZooKeeper 节点数据实时同步
- 支持 ZooKeeper 节点搜索，高亮
- 支持简单的 ACL，以及 ACL 语法检查
- 支持 SSH Tunnel
- 支持配置导入和导出



# 安装 PrettyZoo

PrettyZoo 提供了操作系统 windows 和 macOS 的客户端，可访问 https://github.com/vran-dev/PrettyZoo/releases 地址下载。下载完成，点击安装即可。

# 快速体验

PrettyZoo 的使用非常简单，本小节主要提供界面的演示。

## 3.1 连接 ZooKeeper Server

![图片](https://mmbiz.qpic.cn/mmbiz_gif/JdLkEI9sZfdtxZOdW2HnWdIPOxfwoguU53gChfIsgCvXJXawjRibZ1jMBZsZObR9Acg5jn7Rb8jl7DEfSFGvwVQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)连接 ZooKeeper Server

## 3.2 搜索 ZooKeeper 节点

![图片](https://mmbiz.qpic.cn/mmbiz_gif/JdLkEI9sZfdtxZOdW2HnWdIPOxfwoguUBzWbiaq4WtmxPI2FM3pDiblfvzXPXlEicWyuoiave1LW4gQhlQPv2QX4pw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)搜索 ZooKeeper 节点

## 3.3 添加 ZooKeeper 节点

![图片](https://mmbiz.qpic.cn/mmbiz_gif/JdLkEI9sZfdtxZOdW2HnWdIPOxfwoguUf7BgyrjL3UAPpMwfib9r780HCcZLWyTlDjzF5NZhL3clsic9EEVvwe8A/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)添加 ZooKeeper 节点

## 3.4 删除 ZooKeeper 节点

![图片](https://mmbiz.qpic.cn/mmbiz_gif/JdLkEI9sZfdtxZOdW2HnWdIPOxfwoguUtPpCRlIWTgnDYhnib5pUejyfL7xqbpicDD8W9WJWPk1seWEoOiaIpDyqw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

删除 ZooKeeper 节点

目前艿艿使用的是 `v0.3.1` 版本，删除暂时没有二次确认功能，所以操作一定要小心。