[TOC]

# 1、Docker安全防护工具
 - 安全防护工具之: ClamAV
 - 安全防护工具之: Clair
 - 安全防护工具之: Anchore

# 2、Docker开源工具
## 2.1 watchtower：自动更新 Docker 容器
Watchtower 监视运行容器并监视这些容器最初启动时的镜像有没有变动。当 Watchtower 检测到一个镜像已经有变动时，它会使用新镜像自动重新启动相应的容器。

Watchtower 文档
 > https://github.com/v2tec/watchtower/blob/master/README.md

GitHub 地址：
 > https://github.com/v2tec/watchtower

## 2.2 docker-gc：容器和镜像的垃圾回收
Docker-gc 工具通过删除不需要的容器和镜像来帮你清理 Docker 主机。它会删除存在超过一个小时的所有容器。此外，它还删除不属于任何留置容器的镜像。
docker-gc 文档：
 > https://github.com/spotify/docker-gc/blob/master/README.md

GitHub 地址：
 > https://github.com/spotify/docker-gc

## 2.3 docker-slim：面向容器的神奇减肥药
docker-slim 工具使用静态和动态分析方法来为你臃肿的镜像瘦身。要使用 docker-slim，可以从 Github 下载 Linux 或者 Mac 的二进制安装包。成功下载之后，将它加入到你的系统变量 PATH 中。
docker-slim 文档：
 > https://github.com/docker-slim/docker-slim/blob/master/README.md

GitHub 地址：
 > https://github.com/docker-slim/docker-slim

## 2.4 rocker：突破 Dockerfile 的限制
Rocker（https://github.com/grammarly/rocker）为 Dockerfile 指令集增加了新的指令。Grammarly 为了解决他们遇到的 Dockerfile 格式的问题，创建了 Rocker。

## 2.5 ctop：容器的类顶层接口
提供多个容器的实时指标视图。
ctop 文档：
> https://github.com/bcicen/ctop/blob/master/README.md

GitHub 地址：
> https://github.com/bcicen/ctop