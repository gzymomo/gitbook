# 一、信息收集概述
Web信息搜集（探测）即Web踩点，主要是掌握目标Web服务的方方面面，是实现Web渗透入侵前的准备工作。
Web踩点内容包括操作系统、服务器类型、数据库类型、Web容器、Web语言、域名信息、网站目录....
Web信息搜集涉及搜索引擎、网站扫描、域名遍历、指纹识别等工作。

# 二、Google Hacking
## 2.1 site
功能：搜索指定的域名的网页内容，可以用来搜索子域名、跟此域名相关的内容。
示例：
```
site:zhihu.com    搜索跟zhihu.com相关的网页
“web安全” site:zhihu.com   搜索zhihu.com跟web安全相关的网页
“sql注入”  site:csdn.net  在csdn.net搜索跟sql注入相关的内容
“教程” site:pan.baidu.com  在百度盘中搜索教程
```
## 2.2 filetype
功能：搜索指定文件类型
示例：
```
“web安全”  filetype:pdf   搜索跟安全书籍相关的pdf文件
namp filetype:ppt   搜索跟nmap相关的ppt文件
site:csdn.net filetype:pdf   搜索CSDN网站中的PDF文件
filetype:pdf site:www.51cto.com   搜索51CTO的PDF文件
```
## 2.3 inurl
功能：搜索url网址存在特定关键字的网页，可以用来搜寻有注入点的网站
示例：
```
inurl:.php?id=    搜索网址中有”php?id”的网页
inurl:view.php=?    搜索网址中有”view.php=”的网页
inurl:.jsp?id=   搜索网址中有”jsp?id”的网页
inutl:.asp?id=   搜索网址中有”asp?id”的网页
inurl:/admin/login.php   搜索网址中有”/admin/login.php”的网页
inurl:login   搜索网址中有”login”的网页
```
## 2.4 intitle
功能：搜索标题存在特定关键字的网页
示例：
````
intitle:后台登录    搜索网页标题是“后台登录”的相关网页
intitle:后台管理 filetype:php  搜索网页标题是“后台管理”的php页面
intitle:index of “keyword”  搜索此关键字相关的索引目录信息
intitle:index of “parent directory”  搜索根目录相关的索引目录信息
intitle:index of “password”   搜索密码相关的索引目录信息
intitle:index of “login”  搜索登录页面信息
intitle:index of “admin” 搜索后台管理页面信息
```
## 2.5 intext
功能：搜索正文存在特定关键字的网页
示例：
```
intext:Powered by Discuz   搜索Discuz论坛相关的页面
intext:Powered by wordpress   搜索wordpress制作的博客网址
intext:Powered by *CMS   搜索*CMS相关的页面
intext:Powered by xxx inurl:login  搜索此类网址的后台登录页面
```
## 2.6 实例：
搜索美女/电影等相关网站：
```
inurl:php?id= intitle:美女
inurl:php?id= intitle:美女图片 intext:powered by discuz
inurl:php?id= intitle:美女图片 intext:powered by *cms
```
搜索用Discuz搭建的论坛：
```
inurl:php?id intitle:电影 intext:powered by discuz
intext:”powered by discuz! 7.2” inurl:faq.php intitle:论坛
```
搜索使用Struts的相关网站：
```
intitle:”Struts Problen Report”
intitle:”Struts Problen Report” intext:”development mode is enabled.”
```

## 2.7 符号
```
-keyword  强制结果不要出现此关键字，例如：电影 -黑客
*keyword  模糊搜索，强制结果包含此关键字，例如：电影  一个叫*决定*
“keyword”  强制搜索结果出现此关键字，例如：书籍“web安全”
```
# 三、Google
## 3.1 怎样谷歌呢？
site:     "    "    -
 - site：只搜索某个网站的页面。
 - "   "：以整个短语作为搜索关键字，而不是拆开成每个单词。
 -  -:  排除某个关键字。

site:nytimes.com ~college "test scores" -SATs 2008..2010
 - ~ :同时搜索近义词，比如："high edution"和"university"
 - .. :显示指定年份时间段内的搜索结果。


filetype：pdf speed intitle:velocity of *swallow
 - filetype：只搜索指定类型的文档，可以用来搜搜pdf，doc，jpg等类型的文档。
 - intitle：只显示标题中包含指定关键词的搜索结果（例如：velocity）。
 - * :星号用来代替任意字符（例如：'*swallow'可以匹配'Red Rumped swallow'和'Lesser Striped swallow'等。）

author:green photossyntnesis "to buttz"
 - author：搜索Green发表的论文，而不是包含"green"这个词的论文。
 - " " ：想让结果更精确，可以在引号中输入作者的全名或者缩写。

