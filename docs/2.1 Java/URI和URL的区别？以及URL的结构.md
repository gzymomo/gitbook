[TOC]

URI = Universal Resource Identifier
URL = Universal Resource Locator

- 统一资源标识符(Uniform Resource Identifier, URI)：是一个用于标识某一互联网资源名称的字符串。
- 统一资源定位符(Uniform Resource Locator, URL)：是一个用于标识和定位某一互联网资源名称的字符串。

URL 其实是 URI 的一个子集，即 URL 是靠标识定位地址的一个 URI。

# URL的构成

URL（Uniform Resource Locator,统一资源定位符），用于定位网络上的资源，每一个信息资源都有统一的且在网上唯一的地址。
Url一般有以下部分组成：
> scheme://host:port/path?query#fragment

- Scheme: 通信协议，一般为http、https等；
- Host: 服务器的域名主机名或ip地址；
- Port: 端口号，此项为可选项，默认为80；
- Path: 目录，由“/”隔开的字符串，表示的是主机上的目录或文件地址；
- Query: 查询，此项为可选项，可以给动态网页传递参数，用“&”隔开，每个参数的名和值用“=”隔开；
- Fragment: 信息片段，字符串，用于指定网络资源中的某片断；