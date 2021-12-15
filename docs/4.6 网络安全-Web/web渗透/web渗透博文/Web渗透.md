前端漏洞：钓鱼、暗链、xss、点击劫持、csrf、url跳转
后端漏洞：sql注入、命令注入、文件上传、文件包含、暴力破解

![](https://img-blog.csdn.net/20180713125937232?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDE5MDEz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

例子：
![](https://img-blog.csdn.net/20180713130024316?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDE5MDEz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
报文：
![](https://img-blog.csdn.net/20180713130315361?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI5NDE5MDEz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
HTTP请求-其他请求方式
 - HEAD 与Get请求类似，不同在与服务器只返回HTTP头部信息，没有页面内容。
 - PUT 上传指定URL的描述。
 - DELETE 删除指定资源。
 - OPTIONS 返回服务器支持的HTTP方法。

HTTP请求-Referer
 - HTTP Referer：告知服务器该请求的来源（浏览器自动加上）
 - 统计流量：CNZZ、百度统计。
 - 判断来源合法性：防止盗链、防止CSRF漏洞。