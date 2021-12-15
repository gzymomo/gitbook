[TOC]

- [恕我直言，你可能真没用过这些 IDEA 插件！](https://www.cnblogs.com/coding-farmer/p/13468038.html#)

# 一、IDEA配置
## 1.1 代码智能提示，忽略大小写
File -> Settings -> Editor -> Code Completion里把Case sensitive completion设置为None就可以了

## 1.2 IDEA 注释模板设置
- [IDEA 注释模板设置](https://www.cnblogs.com/youngyajun/p/11588730.html)


# 二、IDEA插件
## 2.1 Background Image Plus
用于修改编辑器背景图片的插件。使用方法：按照下图的提示，选择自己喜欢的图片即可.

![](https://img2018.cnblogs.com/blog/1654189/201909/1654189-20190927193438099-1847272681.png)

## 2.2 Codota—代码智能提示

Codota 这个插件用于智能代码补全，它基于数百万Java程序，能够根据程序上下文提示补全代码。相比于IDEA自带的智能提示来说，Codota 的提示更加全面一些,如下图所示。

我们创建线程池现在变成下面这样：
![](https://mmbiz.qpic.cn/mmbiz_gif/iaIdQfEric9TyGFJPg3XcS7wGbNuRukbGLIl2XmAdSt1Pxicticiaa5LkT0I4ITFLLgRibJ8icKBddHf7slE7IKFMOLuA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

上面只是为了演示这个插件的强大，实际上创建线程池不推荐使用这种方式， 推荐使用 ThreadPoolExecutor 构造函数创建线程池。我下面要介绍的一个阿里巴巴的插件-Alibaba Java Code Guidelines 就检测出来了这个问题，所以，Executors下面用波浪线标记了出来。

除了，在写代码的时候智能提示之外。你还可以直接选中代码然后搜索相关代码示例。
可以使用快捷键： ctrl + shift + o ， 快速查询相关使用案例，同时可以通过添加关键字进行过滤，查找到更加精确的代码样例

![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TyGFJPg3XcS7wGbNuRukbGLSiaQg87aKtep6KmyV35wKc7X4MyLarpSDicaZjSegWCSOtpb97w0TXDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Codota 还有一个在线网站，在这个网站上你可以根据代码关键字搜索相关代码示例，非常不错！我在工作中经常会用到，说实话确实给我带来了很大便利。网站地址：https://www.codota.com/code ，比如我们搜索 Files.readAllLines相关的代码，搜索出来的结果如下图所示：
![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TyGFJPg3XcS7wGbNuRukbGLd4LT6rSp0qlbxhtrx1sflJwz4hq1C94qEBugEll4DdNrtzWSpc1RVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当你不知道某个类如何使用时，可以直接使用快捷键：ctrl + shift + y ， 然后输入关键字，会查询到很多【开源框架】中使用该类的经典案例。不用脱离 IDE，没有广告，没有废话，只有经典的代码样例，你说爽不爽？

![](https://segmentfault.com/img/remote/1460000022552128/view)

## 2.3 Statistic—项目信息统计
有了这个插件之后你可以非常直观地看到你的项目中所有类型的文件的信息比如数量、大小等等，可以帮助你更好地了解你们的项目。
![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TyGFJPg3XcS7wGbNuRukbGLD5c46byGvlCAXq8BKNE498BLjicA6hVoC67lPToWTibxf422fHwGcj2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

你还可以使用它看所有类的总行数、有效代码行数、注释行数、以及有效代码比重等等这些东西。
![](https://mmbiz.qpic.cn/mmbiz_png/iaIdQfEric9TyGFJPg3XcS7wGbNuRukbGLWPMMFsHwic2ciaDVVgWPSic2HxkHYtVia5Nqj7Wia2Q6aGqCuzMzs3bJEsA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.4 RestfulToolkit

- 根据接口搜索
- 提供接口可以测试



Windows：   Ctrl + \
1. 根据接口进行搜索
2. 侧边栏找到RestServices
提供了所有Controller里的接口，还有默认的测试数据。



## 2.5 Material Theme UI
Material Theme UI 在主题下载量排行榜中高居第一。安装主题后（在页面底部就会有进入主题的快捷入口），选择自己喜欢的主题进行微调就可以啦，如果懒得做配置，按照下图勾选相应设置就和我的一样了：

![](https://segmentfault.com/img/bVbGMZE)

## 2.6 Rainbow Brackets
翻译过来叫【彩虹括号】，该插件除了可以实现多彩的括号匹配外，我使用更多的是其【区域代码高亮】功能 ，这样可以清晰定位区域代码内容

Mac 快捷键：cmd + 鼠标右键;
Windows 快捷键：ctrl + 鼠标右键

![](https://segmentfault.com/img/bVbGMZF)

你也可以使用非选中部分暗淡效果
快捷键：alt + 鼠标右键
![](https://segmentfault.com/img/remote/1460000022552126)

## 2.7 CodeGlance

装该插件后，IDE右侧会出现一个mini 视图，比如看 ConcurrentHashMap 源码，那么长的内容，可以通过该插件快速的拖动到大概位置，方便很多

![](https://segmentfault.com/img/bVbGMZT)

## 2.8 SonarLint

SonarLint 是一个免费的IDE扩展，允许您在编写代码时修复错误和漏洞！与拼写检查器一样，SonarLint会动态地突出显示代码问题，并提供明确的修复指导，以便在代码提交之前修复这些问题。在流行的IDEs（Eclipse, IntelliJ, Visual Studio, VS Code）和流行的编程语言，SonarLint 帮助所有开发人员编写更好、更安全的代码！

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2dpZi9TdmRrbGliVFRzU0c4SFhpYlUzaElUZmh5azNqNWhSRDQzWTFHVjBpYnBaOWc4bVhwSnBsTFNJQUhrWlNvQWJyMm5xMVRoU2xRT2hwSmlhRXpRRU1RaHo4MmcvNjQw?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**区别：**

这两款插件的侧重点不同：

- `AlibabaJavaCodingGuidelines` 插件比较关心的是代码规范，编码风格上的，例如，命名规范，注释，代码行数等
- `SonarLint` 插件比较关心代码正确性，存在的问题，风险，漏洞等，例如，重复代码，空指针，安全漏洞等

使用 `AlibabaJavaCodingGuidelines` 插件来规范代码，使用 `SonarLint` 来提前发现代码问题，配合起来提高工程整体的代码质量，并且能够在编码阶段规避风险，提高程序的健壮性。

## 2.9 SequenceDiagram

 SequenceDiagram 插件可以根据代码调用链路自动生成时序图，这对梳理工作中的业务代码有很大的帮助，堪称神器，暴赞！



![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X2dpZi9TdmRrbGliVFRzU0c4SFhpYlUzaElUZmh5azNqNWhSRDQzUGgwYVBCTEY3OWtpY2hnZm10bE9pYnJNUGlja2Vxd2ljdDBVSkIzOXVsbUg4bTg5M1VMV0htNzcyQS82NDA?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**TIP：**双击顶部的类名可以跳转到对应类的源码中，双击调用的方法名可以直接跳入指定方法的源码中