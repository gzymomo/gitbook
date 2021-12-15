# Web 应用程序报告

该报告包含有关 web 应用程序的重要安全信息。



## 介绍

该报告包含由 IBM Security AppScan Standard 执行的 Web 应用程序安全性扫描的结果。

高严重性问题：1

低严重性问题：7

报告中包含的严重性问题总数： 8

扫描中发现的严重性问题总数： 8



## 常规信息

扫描文件名称： 未命名

扫描开始时间： 2020/11/2 9:53:24

测试策略： Default

主机 ：xxxxxxxxx

端口 ：0

操作系统： 未知

**Web** 服务器： 未知

应用程序服务器： 任何



## 登陆设置

登陆方法： 记录的登录

并发登陆： 已启用

**JavaScript** 执行文件： 已禁用

会话中检测： 已启用

会话中模式：

跟踪或会话标识 **cookie**：

跟踪或会话标识参数：

登陆序列：

# 摘要

## 问题类型

![](..\等保测试\qt.png)

## 有漏洞的url

高：http://xxxxxxxxxxxxxxxxxxxxxxxx

低：http://xxxxxxxxxxxxxxxxxxxxxxxx



## 修订建议

![](..\等保测试\ad.png)

![](..\等保测试\ad1.png)



## 安全风险

![](..\等保测试\aq.png)

## 原因



![](..\等保测试\re.png)



## WASC威胁分类

![](..\等保测试\wasc.png)



# 按问题类型分类的问题



## 高：已解密的登录请求



![](..\等保测试\jm.png)

差异：

推理： AppScan 识别了不是通过 SSL 发送的密码参数。

测试请求和响应：

```html
Content-Type: application/x-www-form-urlencoded
admin_name=&admin_pwd=
```



## 低：启用了不安全的“OPTIONS”HTTP 方法 

![](..\等保测试\op.png)

差异： 

方法 从以下位置进行控制： POST 至： GET 

主体 从以下位置进行控制： admin_name=&admin_pwd= 至：

标题 已从请求除去： application/x-www-form-urlencoded

路径 从以下位置进行控制： /security/entry-ctl/login 至： /security/entry-ctl/

方法 从以下位置进行控制： GET 至： OPTIONS 

推理： Allow 头显示危险的 HTTP 选项是已允许的，这表示在服务器上启用了 WebDAV。

测试请求和响应：

```html
OPTIONS /security/entry-ctl/ HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko

Allow: GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH
```

![](..\等保测试\op1.png)

差异： 

方法 从以下位置进行控制： POST 至： GET 

主体 从以下位置进行控制： admin_name=&admin_pwd= 至：

标题 已从请求除去： application/x-www-form-urlencoded

路径 从以下位置进行控制： /security/entry-ctl/login 至： /

方法 从以下位置进行控制： GET 至： OPTIONS 

推理： Allow 头显示危险的 HTTP 选项是已允许的，这表示在服务器上启用了 WebDAV。

测试请求和响应：

![](..\等保测试\op2.png)

差异： 

方法 从以下位置进行控制： POST 至： GET 

主体 从以下位置进行控制： admin_name=&admin_pwd= 至：

标题 已从请求除去： application/x-www-form-urlencoded

路径 从以下位置进行控制： /security/entry-ctl/login 至： /security/

方法 从以下位置进行控制： GET 至： OPTIONS 

推理： Allow 头显示危险的 HTTP 选项是已允许的，这表示在服务器上启用了 WebDAV。

测试请求和响应：





## 低：缺少“Content-Security-Policy”头

![](..\等保测试\csp.png)

差异：

推理： AppScan 检测到 Content-Security-Policy 响应头缺失，这可能会更大程度得暴露于各种跨站点注

入攻击下之

测试请求和响应：



## 低：缺少“X-Content-Type-Options”头 

![](..\等保测试\xc.png)

差异：

推理： AppScan 检测到 X-Content-Type-Options 响应头缺失，这可能会更大程度得暴露于偷渡式下载攻

击之下

测试请求和响应：



## 低：缺少“X-XSS-Protection”头

![](..\等保测试\xss.png)

差异：

推理： AppScan 检测到 X-XSS-Protection 响应头缺失，这可能会造成跨站点脚本编制攻击

测试请求和响应：



## 低：自动填写未对密码字段禁用的 HTML 属性 

![](..\等保测试\ht.png)

差异：

推理： AppScan 发现密码字段没有强制禁用自动填写功能。

测试请求和响应：



# 修订建议

## 高 ：发送敏感信息时，始终使用 SSL 和 POST（主体）参数。

### 该任务修复的问题类型

已解密的登录请求

### 常规

1. 确保所有登录请求都以加密方式发送到服务器。

2. 请确保敏感信息，例如：

- 用户名

- 密码

- 社会保险号码

- 信用卡号码

- 驾照号码

- 电子邮件地址

- 电话号码

- 邮政编码

一律以加密方式传给服务器。

## 低 ：将“autocomplete”属性正确设置为“off” 

### 该任务修复的问题类型

自动填写未对密码字段禁用的 HTML 属性

### 常规

如果“input”元素的“password”字段中缺失“autocomplete”属性，请进行添加并将其设置为“off“。

如果“autocomplete”属性设置为“on”，请将其更改为“off”。

例如：易受攻击站点：

```html
 <form action="AppScan.html" method="get">
     Username: <input type="text" name="firstname" /><br />
     Password: <input type="password" name="lastname" />
     <input type="submit" value="Submit" />
 <form> 
```

非易受攻击站点：

```html
 <form action="AppScan.html" method="get">
     Username: <input type="text" name="firstname" /><br />
     Password: <input type="password" name="lastname" autocomplete="off"/>
     <input type="submit" value="Submit" />
 <form>
```



## 低：将您的服务器配置为使用“Content-Security-Policy”头

### 该任务修复的问题类型

缺少“Content-Security-Policy”头

### 常规

将您的服务器配置为发送“Content-Security-Policy”头。对于 Apache，请参阅：

http://httpd.apache.org/docs/2.2/mod/mod_headers.html

对于 IIS，请参阅：

https://technet.microsoft.com/pl-pl/library/cc753133%28v=ws.10%29.aspx

对于 nginx，请参阅：

http://nginx.org/en/docs/http/ngx_http_headers_module.html



## 低：将您的服务器配置为使用“X-Content-Type-Options”头

### 该任务修复的问题类型

缺少“X-Content-Type-Options”头

### 常规

将您的服务器配置为在所有传出请求上发送值为“nosniff”的“X-Content-Type-Options”头。对于 Apache，请参阅：

http://httpd.apache.org/docs/2.2/mod/mod_headers.html

对于 IIS，请参阅：

https://technet.microsoft.com/pl-pl/library/cc753133%28v=ws.10%29.aspx

对于 nginx，请参阅：

http://nginx.org/en/docs/http/ngx_http_headers_module.html



## 低：将您的服务器配置为使用“X-XSS-Protection”头

### 该任务修复的问题类型

缺少“X-XSS-Protection”头

### 常规

将您的服务器配置为在所有传出请求上发送值为“1”（例如已启用）的“X-XSS-Protection”头。对于 Apache，请参阅：

http://httpd.apache.org/docs/2.2/mod/mod_headers.html

对于 IIS，请参阅：

https://technet.microsoft.com/pl-pl/library/cc753133%28v=ws.10%29.aspx

对于 nginx，请参阅：

http://nginx.org/en/docs/http/ngx_http_headers_module.html



## 低：禁用 WebDAV，或者禁止不需要的 HTTP 方法。

### 该任务修复的问题类型

启用了不安全的“OPTIONS”HTTP 方法

### 常规

如果服务器不需要支持 WebDAV，请务必禁用它，或禁止不必要的 HTTP 方法（动词）。



# 咨询

## 已解密的登录请求

### 测试类型：

应用程序级别测试

### 威胁分类：

传输层保护不足

### 原因：

诸如用户名、密码和信用卡号之类的敏感输入字段未经加密即进行了传递

### 安全性风险：

可能会窃取诸如用户名和密码等未经加密即发送了的用户登录信息

### 受影响产品：

CWE:

523

X-Force：

52471

### 引用：

金融隐私权：格拉斯-斯蒂格尔法案

健康保险可移植性和责任法案 (HIPAA)

萨班斯法案

加利福尼亚州 SB1386

### 技术描述：

在应用程序测试过程中，检测到将未加密的登录请求发送到服务器。由于登录过程中所使用的部分输入字段（例如：用

户名、密码、电子邮件地址、社会安全号等）是个人敏感信息，因此建议通过加密连接（例如 SSL）将其发送到服务

器。

任何以明文传给服务器的信息都可能被窃，稍后可用来电子欺骗身份或伪装用户。

此外，若干隐私权法规指出，用户凭证之类的敏感信息一律以加密方式传给网站



## 启用了不安全的**“OPTIONS”HTTP** 方法

### 测试类型：

基础结构测试

### 威胁分类：

内容电子欺骗

### 原因：

Web 服务器或应用程序服务器是以不安全的方式配置的

### 安全性风险：

可能会在 Web 服务器上上载、修改或删除 Web 页面、脚本和文件

### 受影响产品：

CWE:

74

X-Force：

52655

引用：

WASC 威胁分类：内容电子欺骗

### 技术描述：

似乎 Web 服务器配置成允许下列其中一个（或多个）HTTP 方法（动词）：

\- DELETE

\- SEARCH

\- COPY

\- MOVE

\- PROPFIND

\- PROPPATCH

\- MKCOL

\- LOCK

\- UNLOCK

\- PUT

这些方法可能表示在服务器上启用了 WebDAV，可能允许未授权的用户对其进行利用。



## 缺少**“Content-Security-Policy”**头

### 测试类型：

应用程序级别测试

### 威胁分类：

信息泄露

### 原因：

Web 应用程序编程或配置不安全

### 安全性风险：

可能会收集有关 Web 应用程序的敏感信息，如用户名、密码、机器名和/或敏感文件位置

可能会劝说初级用户提供诸如用户名、密码、信用卡号、社会保险号等敏感信息

### 受影响产品：

CWE:

200

### 引用：

有用 HTTP 头的列表

内容安全策略的简介

### 技术描述：

“Content-Security-Policy”头设计用于修改浏览器渲染页面的方式，并因此排除各种跨站点注入，包括跨站点脚本编

制。以不会阻止 web 站点的正确操作的方式正确地设置头值就非常的重要。例如，如果头设置为阻止内联 JavaScript

的执行，那么 web 站点不得在其页面中使用内联 JavaScript。



## 缺少**“X-Content-Type-Options”**头

### 测试类型：

应用程序级别测试

### 威胁分类：

信息泄露

### 原因：

Web 应用程序编程或配置不安全

### 安全性风险：

可能会收集有关 Web 应用程序的敏感信息，如用户名、密码、机器名和/或敏感文件位置

可能会劝说初级用户提供诸如用户名、密码、信用卡号、社会保险号等敏感信息

### 受影响产品：

CWE:

200

### 引用：

有用 HTTP 头的列表

减小 MIME 类型安全性风险

### 技术描述：

“X-Content-Type-Options”头（具有“nosniff”值）可防止 IE 和 Chrome 忽略响应的内容类型。该操作可能防止在用户浏

览器中执行不受信任的内容（例如用户上载的内容）（例如在恶意命名之后）。



## 缺少**“X-XSS-Protection”**头

### 测试类型：

应用程序级别测试

### 威胁分类：

信息泄露

### 原因：

Web 应用程序编程或配置不安全

### 安全性风险：

可能会收集有关 Web 应用程序的敏感信息，如用户名、密码、机器名和/或敏感文件位置

可能会劝说初级用户提供诸如用户名、密码、信用卡号、社会保险号等敏感信息

### 受影响产品：

CWE:

200

### 引用：

有用 HTTP 头的列表

IE XSS 过滤器

### 技术描述：

“X-XSS-Protection”头强制将跨站点脚本编制过滤器加入“启用”方式，即使用户已禁用时也是如此。该过滤器被构建到

最新的 web 浏览器中（IE 8+，Chrome 4+），通常在缺省情况下已启用。虽然它并非设计为第一个选择而且仅能防御

跨站点脚本编制，但它充当额外的保护层。



## 自动填写未对密码字段禁用的 **HTML** 属性

### 测试类型：

应用程序级别测试

### 威胁分类：

信息泄露

### 原因：

Web 应用程序编程或配置不安全

### 安全性风险：

可能会绕开 Web 应用程序的认证机制

### 受影响产品：

CWE:

522

X-Force：

85989

### 技术描述：

“autocomplete”属性已在 HTML5 标准中进行规范。W3C 的站点声明该属性有两种状态：“on”和“off”，完全忽略时等同

于设置为“on”。

该页面易受攻击，因为“input”元素的“password”字段中的“autocomplete”属性没有设置为“off”。

这可能会使未授权用户（具有授权客户机的本地访问权）能够自动填写用户名和密码字段，并因此登录站点。