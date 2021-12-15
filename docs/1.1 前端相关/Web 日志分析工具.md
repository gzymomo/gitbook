[一款开源且具有交互视图界面的实时 Web 日志分析工具！](https://juejin.cn/post/6968343739819360286)

**前言**

在 Linux 操作系统下，分析日志文件是一件非常头疼的事情，它记录了很多日志，对于大多数的新手及系统管理员不知该如何下手进行分析，除非你在分析日志方面有足够的经验积累，那就是 Linux 系统高手了。

而在 Linux 操作系统中也有许多分析日志工具，GoAccess 是允许用户轻松分析 Web 服务器日志的工具之一，在这篇文章中我们将详细介绍 GoAcccess 工具。

**什么是 GoAccess？**

GoAccess 是一个开源的`实时 Web 日志分析器`和`交互式查看器`，可以在 *nix 系统中的终端运行或通过浏览器进行访问，它需要的依赖少，采用 C 语言编写，只需 ncurses，`支持 Apache、Nginx 和 Lighttpd 日志`，为需要动态可视化服务器报告的系统管理员提供了`高效的`、`具有价值的 HTTP 统计信息`。

**为什么要用 GoAccess？**

GoAccess 可解析指定的 Web 日志文件并将数据输出至终端和浏览器，基于终端的快速日志分析器，其主要还是实时快速分析并查看 Web 服务器上的统计信息，无需使用浏览器，`默认是在终端输出，能够将完整的实时 HTML 报告以及 JSON 和 CSV 报告。`

GoAccess 支持任何自定义日志格式，Apache/Nginx中的组合日志格式：XLF/ELF，Apache 中的通用日志格式：CLF，但并不限于此。

**GoAccess 的功能**

- **完全实时：** 所有面板和指标时间安排在终端输出以每 200 ms 更新一次，在 HTML输出上每秒更新一次的频率；
- **支持几乎所有 Web 日志格式：** GoAccess 允许任何自定义日志格式字符串。预定义的选项包括Apache，Nginx，Amazon S3，Elastic Load Balancing，CloudFront等
- **支持跟踪应用程序响应时间：** 跟踪处理请求所需的时间，当网站运行缓慢时，其效果非常实用；
- **支持增量日志处理：** 可通过磁盘 B + Tree 数据库增量处理日志；
- **所需配置最少：** 可以仅对访问日志文件运行它，选择日志格式后让 GoAccess 解析访问日志并向您进行显示统计信息；
- **访问者：** 按小时或日期确定运行最慢请求的点击数、访问者、带宽和指标等；
- **每个虚拟主机的指标：** 具有一个面板，显示哪个虚拟主机正在消耗大多数 Web 服务器资源；
- **可自定义配色：** 可根据自己的颜色进行调整，通过终端或简单的在 HTML 输出上应用样式表；
- **仅一个依赖：** 用 C 语言编写，运行它，只需将 ncurses 作为依赖项即可；
- **对大型数据集的支持：** 为大型数据集提供了一个磁盘 B + Tree 存储，无法容纳所有内存；
- **Docker支持：** 能够从上游构建GoAccess的Docker映像。

**GoAccess 默认所支持的 Web 日志格式**

- Amazon CloudFront：亚马逊 CloudFront Web 分布式系统
- AWSS3：亚马逊简单存储服务 (S3)
- AWSELB：AWS 弹性负载平衡
- COMBINED：组合日志格式(XLF/ELF) Apache | Nginx
- COMMON：通用日志格式(CLF) Apache
- Google Cloud Storage：谷歌云存储
- Apache virtual hosts：Apache 虚拟主机
- Squid Native Format：Squid 原生格式
- W3C：W3C (IIS)格式

**GoAccess 日期格式**

- **time-format：** 参数`time-format`变量后需要跟一个空格，指定日志格式日期。该日期包含`常规字符`和`特殊格式说明符`的任意组合。以`百分比（％）符号`开头。可参考：`man strftime`，`%T`或`%H:%M:%S`。

> 注意：以毫秒为单位的时间戳，则`%f`必须将其用作时间格式。

- **date-format：** 参数`date-format`变量后需要跟一个空格，指定日志格式日期。该日期包含`常规字符`和`特殊格式说明符`的任意组合。以`百分比（％）符号`开头。可参考：`man strftime`。

> 注意：时间戳以微秒为单位，则`%f`必须用作日期格式。

- **日志格式：** 日志格式变量后需要跟一个`空格`或`\t制表符分隔符`，指定日志格式字符串。

**特殊字符所代表的含义**

- **%x：** 与时间格式和日期格式变量匹配的日期和时间字段。当时间戳而不是将日期和时间放在两个单独的变量中时，使用此方法；
- **%t：** 与时间格式变量匹配的时间字段；
- **%d：** 匹配日期格式变量的日期字段；
- **%v：** 根据规范名称设置的服务器名称（服务器块或虚拟主机）；
- **%e：** 请求文档时，由 HTTP 验证决定的用户 ID；
- **%h：** 主机（客户端IP地址，IPv4 或 IPv6）
- **%r：** 客户端的请求行。这就请求的特定分隔符（单引号，双引号等）是可解析的。否则需使用特殊的格式说明符，例如：`%m`，`%U`，`%q`和`%H`解析各个字段，可使用`%r`获取完整的请求，也可使用`%m`，`%U`，`%q`和`%H`组合你的请求，但不能同时使用；
- **%m：** 请求方法；
- **%U：** 请求URL路径，如果查询字符串在`%U`中，无需使用`%q`。如果`URL路径`不包含任何查询字符串，则使用`%q`，查询字符串将附加到请求中；
- **%q：** 查询字符串；
- **%H：** 请求协议；
- **%s：** 服务器发送回客户端的状态代码；
- **%b：** 返回给客户端对象的大小；
- **%R：** HTTP 请求的 "Referer" 值；
- **%u：** HTTP 请求的 "UserAgent" 值；
- **%D：** 处理请求所花费的时间（以微秒为单位）；
- **%T：** 处理请求所花费的时间（以毫秒为单位）；
- **%L ：** 处理请求所花费的时间（以十进制数毫秒为单位）；
- **%^：** 忽略此字段；
- **%~：** 向前移动日志字符串，直到找到非空格（！isspace）字符；
- **~h：** X-Forwarded-For（XFF）字段中的主机（客户端IP地址，IPv4或IPv6）。

**GoAccess 三个存储选项**

- **默认哈希表**：内存存储提供了更好的性能，其缺点是将数据集的大小限制在可用物理内存的数量。默认情况下，GoAccess 将使用内存中的哈希表。数据集如果放在内存中，执行会很好。因为它具有很好的内存使用和相当好的性能；
- **Tokyo Cabinet 磁盘B+树**：使用此存储方法主要针对无法在内存中容纳所有内容的大型数据集。B+树数据库比任何哈希数据库都慢，因为它的数据必须提交到磁盘。从而使用 SSD 可以极大地提高性能。如果需要数据持久性以及接下来要快速加载的统计数据，可使用该存储方法；
- **Tokyo Cabinet 内存哈希表**：它是默认哈希表的替代方案，使用泛型类型，针对内存和速度而言，它的性能是平均的；

**安装 GoAccess**

**源码安装**

```
# wget https://tar.goaccess.io/goaccess-1.3.tar.gz
# tar -xzvf goaccess-1.3.tar.gz
# cd goaccess-1.3/
# ./configure --enable-utf8 --enable-geoip=legacy
# make
# make install
复制代码
```

**Debian / Ubuntu 系统**

```
# apt-get install goaccess
复制代码
```

获取最新 GoAccess 包，请使用 GoAccess 官方存储库，如下：

```
$ echo "deb https://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
$ wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install goaccess
复制代码
```

**注意：**

- 要获得磁盘上的支持（`Trusty +` 或 `Wheezy +`）可执行：

```
$ sudo apt-get install goaccess-tcb
复制代码
```

- `.deb`官方库的软件包可以通过`HTTPS`获取，这里可能需要安装，可执行：

```
$ apt-transport-https
复制代码
```

**RHEL / CentOS 系统**

```
# yum install goaccess
复制代码
```

**OS X / Homebrew**

```
# brew install goaccess
复制代码
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8c404cd38d2463ca2cb361ee9e4a9f9~tplv-k3u1fbpfcp-zoom-1.image)

**使用 GoAccess**

**不同方式的输出格式：**

输出至终端并生成交互式报告：

```
# goaccess access.log
复制代码
```

生成 HTML 报告：

```
# goaccess access.log -a -o report.html
复制代码
```

生成 JSON 报告：

```
# goaccess access.log -a -d -o report.json
复制代码
```

生成 CSV 文件：

```
# goaccess access.log --no-csv-summary -o report.csv
复制代码
```

GoAccess 为实时过滤和解析提供了巨大的灵活性。如果要从 goaccess 启动以来通过监视日志来快速诊断问题：

```
# tail -f access.log | goaccess-
复制代码
```

如果你要进行筛选，同时保持打开的管道保持实时分析，我们可以利用的`tail -f`和匹配模式的工具，如`grep`、`awk`、`sed`等来进行实现

```
# tail -f access.log | grep -i --line-buffered 'firefox' | goaccess --log-format=COMBINED -
复制代码
```

从文件的开头进行解析，保持管道处于打开状态并应用过滤器

```
# tail -f -n +0 access.log | grep -i --line-buffered 'firefox' | goaccess -o report.html --real-time-html -
复制代码
```

**多日志文件输出格式：**

将多个日志文件传递到命令行：

```
# goaccess access.log access.log.1
复制代码
```

读取常规文件时从管道中解析文件：

```
# cat access.log.2 | goaccess access.log access.log.1-
复制代码
```

> 注意：单破折号附加到命令行以使GoAccess知道它应该从管道读取，在Mac OS X上，请使用 gunzip -c 代替 zcat。

**实时 HTML 输出格式：**

生成实时 HTML 报告的过程与创建静态报告的过程类似，只需加个参数选项：`--real-time-html`使其实现实时的效果。

```
# goaccess access.log -o /usr/share/nginx/html/site/report.html --real-time-html
复制代码
```

除上述三种操作使用外，还可以与日期、虚拟主机、文件，状态代码和启动、服务器一起相结合使用，更多细节请参考其`man手册页`或`帮助`。

```
# man goaccess
或
# goaccess --help
复制代码
```

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f78564dc6574117a09321031db943f8~tplv-k3u1fbpfcp-zoom-1.image)

**Matters needing attention**

每个活动面板共有`366`个项目，或实时`HTML`报告中的`50`个项目，可使用`max-items`自定义项目数量。但是，只有`CSV`和`JSON`输出允许的最大数量大于每个面板`366`个项目的默认值。

使用`磁盘B + Tree`两次分析同一日志文件`--keep-db-files`并`--load-from-disk`在每次运行时使用和时，GoAccess 将每个条目计数两次。

匹配是请求访问日志中的内容，10个请求 = 10个匹配。具有`相同IP`，`日期`和`用户代理`的`HTTP请求`被视为唯一访问。

**Problems during installation**

安装过程中，难免会出现一些问题，具体可参考如下链接：

1、[www.cnblogs.com/zkfopen/p/1…](https://www.cnblogs.com/zkfopen/p/10126959.html)
 2、[www.cnblogs.com/jshp/p/1014…](https://www.cnblogs.com/jshp/p/10143170.html)

**Reference**

1、[github.com/allinurl/go…](https://github.com/allinurl/goaccess)
 2、[goaccess.io/features](https://goaccess.io/features)
 3、[goaccess.io/get-started](https://goaccess.io/get-started)
 4、[goaccess.io/man](https://goaccess.io/man)
 5、[goaccess.io/download](https://goaccess.io/download)

**总结**

通过本篇文章介绍了什么是 GoAccess、为什么要用 GoAccess、GoAccess 的功能、GoAccess 默认所支持的 Web 日志格式、GoAccess 日期格式、GoAccess 特殊字符所代表的含义、GoAccess 三个存储选项、安装以及结合不同场景使用GoAccess，希望大家在今后的工作中能运用起来并通过该工具来解决日常 Web 服务器一些日志的相关问题。