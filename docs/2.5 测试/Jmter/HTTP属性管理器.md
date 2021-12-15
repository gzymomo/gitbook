# [HTTP属性管理器](https://www.cnblogs.com/imyalost/p/7062440.html)

jmeter是一个开源灵活的接口和性能测试工具，当然也能利用jmeter进行接口自动化测试。在我们利用它进行测试过程中，最常用的sampler大概就是Http Request，

使用这个sampler时，一般都需要使用配置元件里的http属性管理器，其作用就是用于尽可能的模拟浏览器的行为，在http协议层上定制发送给被测应用的http请求。

jmeter提供以下五种http属性管理器：

**HTTP Cache Manager：Cache管理器**

**HTTP Cookie Manager：cookie管理器**

**HTTP Header Manager：信息头管理器**

**HTTP Authorzation Manager：授权管理器**

**HTTP Request Defaults：请求默认值**

 

**1、HTTP Cache Manager**

![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170622102758366-1424618413.png)

**Clear cache each iteration?**（每次迭代清空缓存）:如果选择该项，则该属性管理器下的所有Sampler每次执行时都会清除缓存；

**Use Cache-Control/Expires header when processing GET requests**:在处理GET请求时使用缓存/过期信息头；

**Max Number of elements in cache**（缓存中的最大元素数）:默认数值为5000，当然可以根据需要自行修改；

**PS**：如果Test Plan中某个Sampler请求的元素是被缓存的元素，则Test Plan在运行过程中会直接从Cache中读取元素，这样得到的返回值就会是空。

在这种情况下，如果为该Sampler设置了断言检查响应体中的指定内容是否存在，该断言就会失败！

为test plan增加该属性管理器后，test plan运行过程中会使用Last-Modified、ETag和Expired等决定是否从Cache中获取对应元素。

**Cache**：一般指的是浏览器的缓存

**Last-Modified**：文件在服务端最后被修改的时间

**ETag**：在HTTP协议规格说明中定义为：被请求变量的实体标记

**Expired**：给出的日期/时间之后；一般结合Last-Modified一起使用，用于控制请求文件的有效时间

**PS**：上面提到的几个字段，都是HTTP协议里面的报文首部的字段，感兴趣的请自行查阅相关内容，或可参考这篇博客：[浏览器缓存详解](http://blog.csdn.net/eroswang/article/details/8302191)

 

**2、HTTP Cookie Manager**

**![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170622140904679-1200186696.png)**

**Clear cookie each iteration?**（每次迭代时清除自己会话区域的所有cookie）；

**Implementation**：实现方式；

**Cookie Policy**：cookie的管理策略，建议选择compatibility,兼容性强；

**PS**：对于JMeter来说，一个test plan只能有一个cookie管理器。因为当多个magager存在时，JMeter没有方法来指定使用那个manager；

同时，一个cookie manager中的存储的cookie也不能被其他cookie manager所引用，所以同一个计划中不建议使用多个cookie manager；

如果你想让JMeter的cookie manager支持跨域，  修改JMeter.property :CookieManager.check.cookies=false；

**HTTP cookie Manager管理cookie有两种方法：**

①、它可以像浏览器一样存储和发送cookie，如果发送一个带cookie的http请求，cookie manager会自动存储该请求的cookies，并且后面如果发送同源站点的http请求时，

都可以用这个cookies；每个线程都有自己的“cookie存储区域”，所以当测试一个使用cookie来管理session信息的web站点时，每个JMeter线程都有自己的session；

**PS**：以这种自动收集的方式收集到的cookie不会在cookie manager中进行展示，但是运行后通过查看结果树可以查看到cookie信息，接受到的cookie会被自动存储在线程变量中，

但是从Jmeter2.3.2版本后，默认不再存储，如果你想要manager自动存储收集到 的cookie，你需要修改JMeter.property:CookieManager.save.cookies=true；

存储的时候，cookie的key会以“COOKIE_”为前缀命名（默认情况），如果你想自定义这个前缀，修改JMeter.property:CookieManager.name.prefix= ；

②、除了上面说的自动收集，还可以手动添加cookie，点击界面下方的Add按钮，然后输入cookie的相关信息；

**PS：**一般不建议手动添加，可以将cookie通过浏览器插件（比如Firefox的firebug）导出cookie，然后点击界面下方的load按钮，载入刚才导出的cookie文件即可。

**关于Cookie：**

cookie一般分为2种：**持久cookie**（Permanent cookie）和**会话cookie**（Session cookie）：

持久cookie：保存在客户端本地硬盘上，在浏览器被关闭后仍然存在；

会话cookie：通常保存在浏览器进程的会话中，一旦浏览器会话结束或关闭，cookie就不再存在。

 

**3、HTTP Header Manager**

![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170623105600023-508092848.png)

通常Jmeter向服务器发送http请求的时候，后端需要一些验证信息，比如说web服务器需要带过去cookie给服务器进行验证，一般就是放在请求头（header）中，或者请求传参

需要定义参数格式等；因此对于此类请求，在Jmeter中可以通过HTTP信息头管理器，在添加http请求之前，添加一个HTTP信息头管理器，发请求头中的数据通过键值对的形式放到

HTTP信息头管理器中，在往后端请求的时候就可以模拟web携带header信息。

**PS**：可以点击添加、删除按钮等来新增和删减信息头的数据，也可通过载入按钮来将信息头数据加载进去（信息头数据较多时推荐使用）。

 

**4、HTTP Authorzation Manager**

![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170623111634554-1881451286.png)

该属性管理器用于设置自动对一些需要验证的页面进行验证和登录；

**基础URL：**需要使用认证页面的基础URL，如上图，当取样器访问它时，jmeter会使用定义的username和password进行认证和登录；

**用户名：**用于认证和登录的用户名；

**密码：**用于认证和登录的口令；

**域：**身份认证页面的域名；

Realm：Realm字串；

**Mechanism：**机制；jmeter的http授权管理器目前提供2种认证机制：BASIC_DIGEST和KERBEROS：

​      **BASIC_DIGEST：**HTTP协议并没有定义相关的安全认证方面的标准，而BASIC_DIGEST是一套基于http服务端的认证机制，保护相关资源避免被非法用户访问，

​               如果你要访问被保护的资源，则必需要提供合法的用户名和密码。它和HTTPS没有任何关系（前者为用户认证机制，后者为信息通道加密）。

​      **KERBEROS：**一个基于共享秘钥对称加密的安全网络认证系统，其避免了密码在网上传输，将密码作为对称加密的秘钥，通过能否解密来验证用户身份；

 

**5、HTTP Request Defaults**

![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170623143527070-1702817586.png)

![img](https://images2015.cnblogs.com/blog/983980/201706/983980-20170623144703726-10841135.png)

HTTP请求默认值，这个属性管理器用于设置其作用范围内的所有HTTP Request默认值，包括：

**服务器请求或IP：**请求发送的目标服务器名称或地址；

**端口：**目标服务器的端口号，默认80；

**协议：**箱目标服务器发送请求所采用的协议，HTTP或HTTPS，默认HTTP；

**Content encoding** ：内容的编码方式，默认值为iso8859；

**路径**：目标URL路径（不包括服务器地址和端口）；

**同请求一起发送参数** ： 对于带参数的URL ，jmeter提供了一个简单的对参数化的方法：用户可以将URL中所有参数设置在本表中，表中的每一行是一个参数值对；

**从HTML文件获取所有有内含的资源**：该选项被选中时，jmeter在发出HTTP请求并获得响应的HTML文件内容后，还对该HTML进行Parse 并获取HTML中包含的

所有资源（图片、flash等），默认不选中；如果用户只希望获取页面中的特定资源，可以在下方的Embedded URLs must match 文本框中填入需要下载的特定资源表达式，

这样，只有能匹配指定正则表达式的URL指向资源会被下载。

**注意事项：**

①、一个测试计划中可以有多个Defaults组件，多个Defaults组件的默认值会叠加；

②、两个default中都定义的"Server Name or IP"，显示在发送请求时只能使用一个；

参考博客：http://www.cnblogs.com/puresoul/p/4853276.html

​    http://blog.chinaunix.net/uid-29578485-id-5604160.html