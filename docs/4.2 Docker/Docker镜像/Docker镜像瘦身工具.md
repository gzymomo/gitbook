- [docker-slim](https://github.com/docker-slim/docker-slim)



# Docker-Slim-减少镜像大小神器

## 前言

docker镜像的大小,一直是一个绕不开的问题,微服务使用过程中,大量镜像需要传输,一个还好,但一般需要几十个进行传输,导致网络不好的地方很是老火,而且镜像服务器整个公司使用下来,硬盘也是妥妥压力大,最近在网上看到一个可以将镜像最小化的应用.

## Docker-Slim

不修改镜像内容前提下,缩小30倍镜像。
 其官方通过静态分析跟动态分析来实现镜像的缩小。

### 静态分析

通过docker镜像自带镜像历史信息,获取生成镜像的dockerfile文件及相关的配置信息。

### 动态分析

通过内核工具ptrace(跟踪系统调用)、pevent(跟踪文件或目录的变化)、fanotify(跟踪进程)解析出镜像中必要的文件和文件依赖，将对应文件组织成新镜像。