[TOC]

# 1、JavaScript简介
一个完整的JavaScript实现应该由下列三个不同的部分组成：核心（ECMAScript）、文档对象模型（DOM）、浏览器对象模型（BOM）。

![](https://www.showdoc.cc/server/api/common/visitfile/sign/926abc0ab0ebe2f15c510ace2c13a499?showdoc=.jpg)

DOM：
- DOM视图：定义了跟踪不同文档（例如，应用CSS之前和之后的文档）视图的接口；
- DOM事件：定义了基于CSS为元素应用样式的接口；
- DOM遍历和范围：定义了遍历和操作文档树的接口。

BOM：
- 弹出新浏览器窗口的功能；
- 移动、缩放和关闭浏览器窗口的功能；
- 提供浏览器详细信息的navigator对象；
- 提供浏览器所加载页面的详细信息的location对象；
- 提供用户显示分辨率详细信息的screen对象；
- 对cookies的支持；

# 2、HTML中使用JavaScript
<script>定义了6个属性：
- asynv：表示应该立即下载脚本，但不应妨碍页面中的其他操作，比如下载资源或等待加载其他脚本。
`<script type="text/javascript" async src="../example.js">`
- charset：表示通过src属性指定的代码的字符集。
- defer：表示脚本可以延迟到文档被完全解析和显示之后执行。只对外部脚本文件有效。
- language：已废弃。
- src：表示包含要执行代码的外部文件。
- type：表示编写代码使用的脚本语言的内容类型。默认text/javascript。

在XHTML中的用法：
在XHTML中，用一个CDATA片段来包含JavaScript代码。在XHTML（XML）中，CDATA片段就是文档中的一个特殊区域，这个区域中可以包含不需要解析的任意格式的文本内容。
```Javascript
<script>
  <![CDATA[
    function compare(){
       ............
    }
  ]]
</script>
```

文档模式：
如果在文档开始处没有发现文档类型声明，则所有浏览器都会默认开启混杂模式。
对于标准模式，可以通过使用下面任何一种文档类型来开启：
```html
<!-- HTML 4.01 严格型 -->
<!DOCTYPE HTML PUBLIC "-//WSC/DTD HTML 4.01//EN"
"http://www.w3.org/TR/html4/strict.dtd">

<!-- HTML5 -->
<!DOCTYPE html>
```

<noscript>
用以在不支持JavaScript的浏览器中显示替代的内容。这个元素可以包含能够出现在文档<body>中的任何HTML元素————<script>元素除外。

包含在<noscript>元素中的内容只有在下列情况下才会显示出来：
- 浏览器不支持脚本，
- 浏览器支持脚本，但脚本本禁用。

# 7、函数表达式


# 11、新兴API
## 11.1 Page Visibility API
表示页面是否对用户可见。
- document.hidden:表示页面是否隐藏的布尔值。页面隐藏包括页面在后台标签页中或者浏览器最小化。
- document.visibilityState：表示下列4个可能状态的值。
  - 页面在后台标签页中或浏览器最小化。
  - 页面在前台标签页中。
  - 实际的页面已经隐藏，但用户可以看到页面的预览。
  - 页面在屏幕外执行预渲染处理。
- visibilitychange事件：当文档从可见变为不可见或从不可见变为可见时，触发该事件。

Chrome加上webkit前缀，即可使用，如：document.webkitHidden。

检查浏览器是否支持Page Visibility：
```Javascript
function isHiddenSupported(){
   return ("hidden" in document || "msHidden" in document || "webkitHidden in document");
}
```

类似的，使用同样的模式可以检测页面是否隐藏：
```Javascript
if (document.hidden || document.msHidden || document.webkitHidden){
   //页面隐藏了
}else{
   //页面未隐藏
}
```

## 11.2 Geolocation API
地理定位。通过Geolocation API，JavaScript代码能够访问到用户的当前位置信息。需要请求用户获得许可。

Geolocation API在浏览器中的实现是navigator.geolocation对象，这个对象包含3个方法。
第一个方法时：getCurrentPosition()，调用这个方法就会出发请求用户共享地理位置信息的对话框。这个方法接收3个参数：成功回调函数，可选的失败回调函数和可选的选项对象。
成功回调函数会收到一个Position对象参数，该对象有两个属性：coords和timestamp，而coords包含如下位置相关信息：
- latitude：十进制度数表示的纬度。
- longitude：十进制度数表示的经度。
- accuracy：经纬度坐标的经度，以米为单位。
- altitude：以米为单位的海拔高度，如果没有相关数据则为null。
- altitudeAccuracy：海拔高度的经度，以米为单位。
- heading：指南针的方向。0°表示正北，值为NaN表示没有检测到数据。
- speed：速度，即每秒移动多少米。

在地图上绘制用户的位置：
```Javascript
navigator.geolocation.getCurrentPosition(function(position){
   drawMapCenterdAt(position.coords.latitude,positions.coords.longitude);
});
```

## 11.3 File API
为Web开发人员提供一种安全的方式，以便在客户端访问用户计算机中的文件，并更好地对这些文件执行操作。

## 11.4 Web 计时

## 11.5 Web Workers
Web Workers规范通过让JavaScript在后台运行解决浏览器冻结用户界面的问题。

浏览器实现Web Workers规范的方式有很多种，可以使用线程，后台进程或者运行在其他处理器核心上的进程。