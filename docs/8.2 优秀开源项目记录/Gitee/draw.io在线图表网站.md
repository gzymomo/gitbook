Gitee地址：https://gitee.com/mirrors/drawio-desktop

官网：https://www.diagrams.net/index.html



### 软件简介

draw.io 是在线图表网站 [diagrams.net](https://www.diagrams.net/) 的代码实现。[diagrams.net](https://www.diagrams.net/) 是一个用于构建图表应用程序的开源技术栈，也是世界上使用最广泛的基于浏览器的终端用户图表软件。

![img](https://static.oschina.net/uploads/space/2021/0319/142955_JSLz_2744687.png)

**构建**

draw.io 由两部分组成。主要部分是客户端的 JavaScript 代码。用户可以使用 Ant build.xml 文件中默认的  "all" 任务来创建 minified JavaScript，该任务通过在仓库的 etc/build 文件夹中运行 ant 来执行。

注意，如果只使用客户端代码，会导致缺少 Gliffy 和 .vsdx 导入器、嵌入支持、图标搜索和发布到 Imgur。如果用户想用  Java 服务器端代码和客户端 JavaScript 构建完整的项目，请调用 Ant build.xml 文件中的 "war" 任务，并将生成的 .war 文件部署到 servlet 引擎中。

**运行**

运行 diagrams.net 的一种方法是 fork 该项目，将主分支发布到 GitHub  页面上，页面站点将拥有完整的编辑器功能（不含集成）。另一种方法是使用推荐的 Docker 项目或下载 draw.io Desktop。客户端和  servlets 的完整 packaged.war 可以在release 页面获得。

**支持的浏览器**

diagrams.net 目前支持 IE 11、Chrome 70+、Firefox 70+、Safari 11+、Opera 50+、原生安卓浏览器7x+、当前及以前主要 iO S版本的默认浏览器（如11.2.x和10.3.x）和 Edge 79+。