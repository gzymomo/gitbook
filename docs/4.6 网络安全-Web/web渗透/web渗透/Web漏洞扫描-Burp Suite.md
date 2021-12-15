Web漏洞扫描-Burp Suite

参考链接：博客园：[小心走火](https://home.cnblogs.com/u/nieliangcai/)：[Burp Suite使用介绍](https://www.cnblogs.com/nieliangcai/p/6692296.html)



CSDN：[lfendo](https://blog.csdn.net/u011781521)：[Kali Linux](https://blog.csdn.net/u011781521/category_6494586.html)

# 一、Burp Suite概述
安全渗透界使用最广泛的漏扫工具之一，能实现从漏洞发现到利用的完整过程。功能强大、配置较为复杂、可定制型强，支持丰富的第三方拓展插件。基于Java编写，跨平台支持，收费，不过有Free版本，功能较少。
https://portswigger.net/burp/



Burp Suite 是用于攻击web 应用程序的集成平台，包含了许多工具。Burp Suite为这些工具设计了许多接口，以加快攻击应用程序的过程。所有工具都共享一个请求，并能处理对应的HTTP 消息、持久性、认证、代理、日志、警报。



# 二、功能及特点
![](https://www.showdoc.cc/server/api/common/visitfile/sign/6daa6f75acb6454b085df43958b45c41?showdoc=.jpg)

 - Target 目标模块用于设置扫描域(target scope)、 生成站点地图(sitemap)、 生成安全分析
 - Proxy 代理模块用于拦截浏览器的http会话内容
 - Spider 爬虫模块用于自动爬取网站的每个页面内容,并生成完整的网站地图
 - Scanner 扫描模块用于自动化检测漏洞,分为主动和被动扫描
 - Intruder 入侵(渗透)模块根据上面检测到的可能存在漏洞的链接，调用攻击载荷,对目标链接进行攻击
 - 入侵模块的原理是根据访问链接中存在的参数/变量,调用本地词典、攻击载荷，对参数进行渗透测试
 - Repeater 重放模块用于实现请求重放,通过修改参数进行手工请求回应的调试
 - Sequencer 序列器模块用于检测参数的随机性,例如密码或者令牌是否可预测，以此判断关键数据是否可被伪造
 - Decoder 解码器模块用于实现对URL、HTML、 Base64、 ASCII、 二/八/十六进制、 哈希等编码转换
 - Comparer 对比模块用于对两次不同的请求和回应进行可视化对比,以此区分不同参数对结果造成的影响
 - Extender 拓展模块是burpsuite非常强悍的一一个功能，也是它跟其他Web安全评估系统最大的差别
 - 通过拓展模块，可以加载自己开发的、或者第三方模块,打造自己的burpsuite功能
 - 通过burpsuite提供的API接口，目前可以支持Java、Python、 Ruby三种语言的模块编写
 - Options 分为Project/User Options，主要对软件进行全局设置
 - Alerts 显示软件的使用日志信息

# 三、Burp Suite安装
Kali Linux:集成BurpSuite Free版本 ,不支持scanner功能
Windows :BurpSuite Pro 1. 7.30版本,支持全部功能。

启动方法:
`java -jar -Xmx1024M /burpsuite_path/BurpHe1per. jar`

# 四、Burp Suite使用
 - 代理功能（Proxy）
 - 目标功能（Target）
 - 爬虫功能（Spider）
 - 扫描功能（Scanner）



# 五、Burp Suite实践

参考链接：知乎：[信息安全发送者](https://www.zhihu.com/people/an-jie-wang)：[黑客神器 “教你手把手学会burpsuite”](https://zhuanlan.zhihu.com/p/96348391)

## 5.1 burp的功能初始化

比如说。我们在哪个模块。不小心乱设置了。或者说乱搞了。我们可以在这里重新我们乱搞的那个模块，重置就跟一开始的一模一样。所以我们在乱搞的情况下，我们可以这样重置功能。也就是模块的初始化功能 。

![img](https://pic2.zhimg.com/80/v2-f8811961b8eb2b6d0544af58e20598d1_720w.jpg)



all什么意思呢？可以理解为全部的意思，就是把全部模块重置。这就是初始化功能

![img](https://pic3.zhimg.com/80/v2-5213a9d06a8caf46d067312eb0e3346b_720w.jpg)



## 5.2 抓包，设置Proxy代理(将浏览器和Burp Suite的Options中的代理设置为一致)

抓包，要设置好代理，进入到options里面。点击add添加一个抓取的代理ip和端口

![img](https://picb.zhimg.com/80/v2-8acc3bb78d3f0a6ccc75e4f796be1199_720w.jpg)



因此浏览器上面也要设置一样的ip和端口，这样才能抓取到http的一些请求包

![img](https://pic4.zhimg.com/80/v2-09f926971f271dc12774a7d7f2dc0407_720w.jpg)



接着就是点击intercet is on开始监听，因为burp的ip和端口和浏览器设置的代理一样，所以burp通过监听去抓取请求网站资源的http数据包

![img](https://picb.zhimg.com/80/v2-73fa99dce2bfd6f9f99aaf4cd9f5973a_720w.jpg)

我这里的话，拿了dvwa的进行一方面的演示，可以看见我在登录的时候，如果burp的监听功能去抓取了登录请求的一个数据包，然后输入的账号密码也是抓取数据包中的内容一模一样。

![img](https://pic4.zhimg.com/80/v2-3002ff4ccf49016786305bc436319f5d_720w.jpg)

2）数据包其它的介绍

host:消息头用于指定出现在被访问的完整url中的主机名称

user-agent:这个消息头提供与浏览器或生产请求的，其他客户端软件有关的信息

accept：这个消息头用于告诉服务器客户端愿意接受哪些内容，如图像类型，办公文档格式等

accept-language：用于生命服务器浏览器可以支持，什么语言

accept-encoding：这个消息头用于告诉服务器，客户端愿意接受哪些内容编码

referer：这个消息头用于指示提出当前请求的原始url

cookie：提交此前服务器向客户端发送的其他参数（服务器使用set-cookie消息头来设置cookie一般用于身份验证）

Drop的意思就是将抓取的数据包给丢弃掉

![img](https://pic3.zhimg.com/80/v2-173ad1df19124396184920026a08921c_720w.jpg)



Forward的意思就是将抓取的数据包给放掉，那么网页那边也会进行一方面的继续请求。

![img](https://picb.zhimg.com/80/v2-aa45d4980ce4ace20ec675675b4f631e_720w.jpg)

## 5.3 repeater模块

burp发送http请求 ，利用repeater模块进行发送请求， 抓到包之后，全选然后发送到repeater模块下

![img](https://pic3.zhimg.com/80/v2-caae17fc401098a88ef0e3990a6bf579_720w.jpg)

然后它这里就会发亮

![img](https://picb.zhimg.com/80/v2-724872f58ee2e07527e66eb3fd771f88_720w.jpg)



然后一看，发送过来了

![img](https://pic4.zhimg.com/80/v2-18c53e1c99ac23d6afb41da177dd33d6_720w.jpg)



但是它这个GO是什么意思呢？就是把内容的http发送请求

![img](https://pic1.zhimg.com/80/v2-5317eb4869d1689217b29a48660cdfaa_720w.jpg)

然后右边就是响应的内容

![img](https://pic3.zhimg.com/80/v2-7b1719974e00375754e9aba8f3536a94_720w.jpg)



如果你要访问https你就打勾。就是443，然后它就会请求的就是https

![img](https://pic1.zhimg.com/80/v2-0a98de59161d40366742fe15734304d5_720w.jpg)

![img](https://pic2.zhimg.com/80/v2-6c71206dcaae1341868d248961711bad_720w.jpg)



然后这两个一个是后退、一个是前进。就相当于浏览器中的返回上一页

![img](https://pic4.zhimg.com/80/v2-bde62e1102e205aff2c2d5f7ad4c530d_720w.jpg)

## 5.4 scanner模块burp扫描漏洞

利用scanner模块进行扫描漏洞

它可以进行一个什么xss。sql注入的漏洞扫描

抓包之后，鼠标右键，然后发送到scanner模块

![img](https://pic2.zhimg.com/80/v2-18c3d59e9e3a13e3565f05ecc36a0f1a_720w.jpg)



然后它会弹出框框。我们点击yes

![img](https://pic4.zhimg.com/80/v2-b71c6725066f6e6edd958d07a59cd0f3_720w.jpg)



然后它这里就会亮了

![img](https://pic2.zhimg.com/80/v2-4ccb61cae1a48ec08518089aa6e73d6c_720w.jpg)



status意思就是扫描了百分之十一了。然后lssues就是扫描到多少个漏洞的意思。而request就是请求了多少次的意思

![img](https://pic3.zhimg.com/80/v2-b284a16fb0d16a283537adaabd5c582f_720w.jpg)



它支持扫描sql注入，xml注入漏洞一些，都能扫

![img](https://pic2.zhimg.com/80/v2-2b7baa9b222c4ad05dbbbe0b9aa4640c_720w.jpg)



而其它功能设置的话，它这里第一个是线程，第二个的意思是。请求一个网页如果失败了。它顶多会请求三次，最多请求三次，如果还是失败了，就那样了，而第三个的2000的意思就是时隔为2000毫秒

![img](https://pic2.zhimg.com/80/v2-ceb116ceb922be0cb6954247769df4f4_720w.jpg)

扫描完之后，lssues会提示扫描到多少个漏洞，然后我们双击点击3

![img](https://pic4.zhimg.com/80/v2-d91329cb96c8b1bae7e0bed4dbf7786a_720w.jpg)



然后就会发现一个漏洞，一个是sql注入，一个是http等等

![img](https://pic3.zhimg.com/80/v2-bf35f023e45f119726622921e73fabb0_720w.jpg)



然后这里就是响应的内容

![img](https://pic1.zhimg.com/80/v2-b6641e2e5acafc3fbe8ca7466aebbef9_720w.jpg)



我们可以看到。一开始的设置那里。请求三次，这里有三对，request是请求 而response就是响应，而图片中的红色感叹号代表高危，而响应内容中的红色一行一行的就是代表sql注入的参数， 而黑色就是低微，没什么用

![img](https://pic1.zhimg.com/80/v2-fdb53524e0d3b475b82b63d465fd8aa9_720w.jpg)

扫描的设置，什么mssql和mysql还有Oracle啊。选中之后。它就会扫描打勾了的漏洞

![img](https://pic4.zhimg.com/80/v2-a94c7b22488496d8898c9a1d5597f1e1_720w.jpg)

## 5.5 spider模块爬虫

spider模块中的options就是设置，第一个是爬行的深度，比如说目录,爬到这种五级目录的话，它就不会继续爬了，因为它这里就是设置为了5

![img](https://pic2.zhimg.com/80/v2-6ebcc33acb949dd4a9d4e784fd40e7be_720w.jpg)



他这里是参数。比如说id=1. 爬到id=50.它就不会爬

![img](https://picb.zhimg.com/80/v2-2e9bad8e774eb06d887ca34490fa1c42_720w.png)



检测这方面的文件，比如说robots文件

![img](https://picb.zhimg.com/80/v2-240cfe1e9908400b30f729a2e58fce1c_720w.jpg)



最后就是你请求过的，或者爬虫到的。它都会存到这里

![img](https://pic2.zhimg.com/80/v2-9cac363db05e5f66e875c9b76144ab2e_720w.jpg)

## 5.6 **Intruder高效暴力破解**

参考链接：博客园：[py7hon](https://home.cnblogs.com/u/pshell/)：[[Burp Suite渗透操作指南 【暴力破解】](https://www.cnblogs.com/pshell/p/7979653.html)](https://www.cnblogs.com/pshell/p/7979653.html)

[Burp Suite渗透实战操作指南-上篇](https://www.cnblogs.com/pshell/p/7979649.html)





Intruder支持多种爆破模式。分别是：单一字典爆破、多字段相同字典爆破、多字典意义对应爆破、聚合式爆破。最常用的应该是在爆破用户名和密码的时候。使用聚合方式枚举了。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160749lwtui34linl4xjij.png)
Payload Set的1、2、3分别对应要爆破的三个参数。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160749y6m3z1gofzr3rnoo.png)

### **1.1.1 字典加载**

Payload里面提供了一些fuzzing字典，但是总觉得不是特好用。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160750dpffoiz6cfboqrbi.png)
如果嫌弃自带的字典不太符合国情，你可以手动加载自定义字典列表。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160750syt4yw3j64x0sr6r.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160750ykcj40wkskwsqzw8.png)
如果爆破的结果在返回数据包中有不同的信息，我们可以通过添加匹配来显示每一个爆破的数据包中的部分类容，提高可读性。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160751zve5wlxpll6x8lav.png)
**添加方法如下：**
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160752y58m5mb3lhgt4s5g.png)
需要注意的是：Burp很多地方对中文支持不是特别好，在遍历数据的时候经常会碰到中文乱码的情况。如:

> "LoginId":"xl7dev阿西吧","NickName":"阿西吧"
> "LoginId":"xl7dev","NickName":""阿西吧"


由于响应数据的不确定因素，中文在加载字典的时候是乱码，这里为了显示好看，只匹配英文部分，可以使用正则："LoginId"\:"([a-zA-Z0-9_]+)"\,"NickName"，最终提取的都是英文字符。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160752sp466dj664rhrrln.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160753f9can4h0j55i0cni.png)


注意：Burpsuite对中文和空格支持不是很好，在设置字典路径的时候应避免路径中存在中文或者空格，字典内容中也要避免多余的空格。Porxy，repeater中的数据包中的中文乱码可通过修改字符集和编码纠正。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160753vhgotrbgfgzoowbt.png)



### **1.1.2 多种加密和编码支持**

Intruder支持多种编码和加密，常见的base64加密、md5加密爆破通过intruder将变得很容易。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160754bgfccer77ohotgf8.png)
在爆破密码或者某些参数的时候可能会遇到二次Md5加密、先URL编码再Base64解密等等情况。其实intruder中的二次、三次加密或者解密很简单。在Payload Processing中按照要加密/解密的顺序依次添加就好了。如二次Md5：
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160754ujea70a670f5j0sw.png)
第一个为123456的二次MD5
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160754pdlzi4ho4l661id4.png)



### **1.1.3 关于短信轰炸**

现在有的短信发送限制为一分钟发送一次，可以通过Burp设置延时爆破，一分钟爆破一次。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160755zt72lxvj58nnt878.png)

### **1.1.4爆破指定数字长度**

#### **1.1.4.1采用Custom iterator**

设置如下Password参数中$1$$2$$$3$$4$，Attack type为Sniper
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160756hlllrpptfccl14t3.png)
在Payloads选项卡中Payload type设置Custom iterator
Payload Options>![img](https://bbs.ichunqiu.com/static/image/smiley/default/titter.gif)osition中分别对应选择
1=>0 1 2 3 4 5 6 7 8 9
2=>0 1 2 3 4 5 6 7 8 9
3=>0 1 2 3 4 5 6 7 8 9
4=>0 1 2 3 4 5 6 7 8 9
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160756muzqbiu1lubomvuu.png)
效果如下图：
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160757qgggze66030kscc0.png)

#### **1.1.4.2 采用Brute forcer方式**

设置如下：
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160757tdccx8jmj7nwpjdx.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160758ii0mvtag03m0lam4.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160759y8vvqvdxi30uvvho.png)





#### **1.1.4.3 采用Numbers方式**

​     此方法不太稳定，可能有时候不会出现想要的结果
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160800zxdxvxd0ki9n9wvp.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160800mq2l3et2d2ceoft6.png)

由此，可以套路一波Authorization: Basic爆破。直接上图：
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160801aml8hipimkm39hlk.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160801f57bujatam27b6me.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160802pcqw69bfpdipwflq.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160802jelvvystoslmveys.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160803k8k1tkywkgwdd44a.png)

#### **1.1.5 制作Authorization: Basic字典**

除了上述方法，还可以通过一些奇葩的手段制作字典，不仅限于Authorization: Basic字典。这只是个例子，什么时候能用次方法，大家自己意淫吧。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160804ykuh9uh2gg992umz.png)
常规的对username/password进行payload测试，大家应该没有什么问题，但对于Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=这样的问题，很多朋友疑惑了. 
Auth=dXNlcjpwYXNzd29yZA==是个base64加密的字符串，解密后其组合形式时user:pwd。直接爆破时不可能的，应为字典比较独特。如果大家手里有现成的user:pwd类型字典那不用说，要是没有现成的，可以用以下方法生成：
1：随便构造一个数据包，形式入下：

POST /test/ HTTP/1.1
Host: [www.test.com](http://www.test.com/)

§username§§:§§password§
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160806hdyd71kf7cuyc71b.png)
采用Cluster bomb模式，设置三个变量：
§user§§:§§password§

设置3处payloads，
Payload 1 ------ §user§ ------为user变量的字典
Payload 2 ------ §:§ ------ 固定为冒号“:”
Payload 3 ------ §pwd§ ------ 为pwd变量的字典
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160808qjxsj1vrsjwjwf9s.png)
通过爆破发包的方式生成字典：
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160808dg6s44uy4yyn5mis.png)
点击save->Resuits table
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160808ptzpyofoso337833.png)
按照如下选项选择保存。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160809hjcgvv5id6hc5dco.png) ![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160810blsxqxos22j2qobk.png)


接下来怎么爆破Authorization: Basic就很简单了吧。
需要说明的是这个方法其实不是特别好，因为http发包线程关系，生成字典的速度不是很快，不像自己写的脚本那么方便。
如果你安装了CO2插件，生成这种字典就很容易了。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160810x8i6eiimmq4vev0u.png)


当然你也可以写个脚本生成这样的字典。全靠平时积累吧。
建议感兴趣的同学好好研究下payload sets里面的各种方法。收获一定不许小。
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160811qs6rp5x64974pkkr.png)

#### **1.1.6爆破一句话木马**

其实就是利用一句话木马传入成功的参数执行命令，根据命令返回判断是否得到了密码一句话木。

**1：抓包将提交方式由get改为post：**    
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160811cyn9d7r0ydzdbxvz.png)
**2：回车将如下代码粘贴为post的内容：**
payload=execute("response.clear:response.write(""passwordisright""):response.end")
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160812vz3fx40lbxjoxfh3.png)
**2.1：在intruder中将payload设置为爆破变量：**
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160813bx4ku33l10814l91.png)
**3：添加爆破字典**
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160813m9kaknpnr0908dbj.png)
  成功得到密码：byecho
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160814lrdzjnm75o9y7yz9.png)
下面是三种脚本的代码：
ASP：password=execute("response.clear:response.write(""passwordright""):response.end")
PHP：password=echo phpinfo();
ASPX：password=execute("response.clear:response.write(""elseHelloWorld""):response.end")
     大家可以思考小，如何才能结合之前Binghe在【python之提速千倍爆破一句话】中的高效爆破方法。地址：http://bbs.ichunqiu.com/thread-16952-1-1.html

#### **1.1.7 有限制刷票**

演示地址：http://guojigongguan.cn/2014/baobao/babyshow.shtml?BabyID=59
本例投一次票会请求2次，第一次是一个点击状态请求post，第二次才是投票
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160815r7xc66ahce3elww9.png)
第二次post页面只需要把cookie删除
Cookie: GUID=6966a72e-a577-4050-801f-0ddd43a2beb5
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160815ufb79x4pvdbxppqf.png)
Intruder设置
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160816yz1t8n433bt6bm3b.png)
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160817vrlns7talr1glhaz.png)

#### **1.1.8 关于字典**

字典这块值得好好研究下，下如何才能高效的爆破后台。之前遇到过一个大牛，只要能爆破的，很多时候都成功了，一般人去却做不到。有一套好的密码生成方法很重要。我没啥好的方法，所以这里也想个大家做些探讨：针对具体的目标网站，有哪些行之有效的字典生成方法？有哪些值得推荐的工具？
希望大家积极评论，我将整理有用的方法并及时更新。
如何选取字典

**1.OA、mail、tms、crm、mes等系统常用用户名：**
姓名拼音。
6位、8位数字编号。

**2.一般后台用户名：**
常用的用户名TOP100、TOP1000
姓名拼音TOP100、TOP1000

**3.密码字典**
最先尝试最弱面：123456、888888、666666、111111、12345678、test、admin等
其次尝试TOP100、TOP1000。
一定要搞，那就用更大的字典。

4.运营商、网络公司（思科、华为等等）的网络设备或者网站都有很多独特的用户名和密码，大家可以上网收集一下。**
个人密码生成
根据个人信息融入常规弱口令，形成带入个人特征的密码：
http://www.caimima.net/index.php
密码分析软件Oopa
地址：https://github.com/chao-mu/Oopa，首先用pip安装prettytable模块
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160817h7w1onk4ins9iw1i.png)
分析统计用的最多的前二十个密码：python oopa.py --analysis keyword xxxx.txt --top 20
![img](https://bbs.ichunqiu.com/data/attachment/forum/201612/28/160818yi9ksm77ramiezuq.png)
分析个位数密码占所有密码的百分比：python oopa.py --analysis length xxxx.txt --sort Count

## 5.7 利用burpsuit宏获取token对网站进行暴力破解

参考链接：[黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](https://www.heikeblog.com/archives/405.html)

Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。

以下是在做渗透测试时候的操作过程截图。

首先是一个登陆框。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/5c03976847554694801ce556ddae81bb)

载入burpsuite尝试爆破。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/1aa3a370bfb243999c3fb84586d8619c)

发现POST数据里面有token，token的作用主要有两个：

1.防止表单重复提交;

2.用来做身份验证;

可以防止暴力枚举，还能防止CSRF等攻击，用token作为参数去加密请求信息，每次请求都具有唯一性，csrf攻击将不成立，除非攻击者有特异功能，能预知到你的token值。

这样爆破就行不通了。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/a8ff71b7c4bb4299894b7cebb630e430)

继续挖掘，我们尝试了一下，先获取token，然后发送带token的登录数据包。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/b1c830d2d0534f06bb0e4b2d05516071)

这里我们做个改进，可以利用burpsuite宏，制作一个获取页面token的宏，在每次爆破之前，先运行一次宏，获取到新的有效token。

点击burpsuite的Project Options模块，再进入下面的Sessions子模块，在Macros下add添加一个宏。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/14e83217845d47809099515b674bcf68)

然后在弹出来的界面中，选择获取token值的页面，点OK。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/ba2a8640bf224ee0b6121d02de6f1888)

进入宏编辑界面，输入宏的名字，然后点击右边的Configure item。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/f7898c51df834644a955de815656759d)

宏设置页面，指定参数的位置，点击add添加参数。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/690e355772e740bfbb465a6444794dbb)

输入参数名称，选择token的位置，在configure Macro Item、Macro Editor全部点击确定，完成宏的录制。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/727ff66245514d26a48c82f518f63cab)

可以在Macros列表看到刚才录制的宏。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/d05013f4719748c6ad5fbf927b2784bf)

在会话处理规则处，点击add，添加一条新规则。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/acb167f3937f47688997e7a93d754230)

填写规则描述，添加一个动作，在请求之前运行一个宏。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/deb86fd7993d4572b727c1533b312c28)

在弹出来的界面中，选择刚才录制的宏，选择值更新指定的参数，输入请求包的token名。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/8d6cac47c47a4d03a15a79d018b2a557)

编辑完成后，回到会话操作规则编辑界面，选择Scope，设置这个宏的作用域。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/37f96a8d99b947718e6d235d61e4f3c8)

完成上述操作后，会话处理规则列表中有了新增的规则。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p3.pstatp.com/large/pgc-image/66565cba1f1c43a9af1978a3d64dfd81)

我们现在来验证一下规则是否生效，再使用Intrude模块进行爆破，发现每次提交请求后，token的值都会自动更新。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/c25a628347a04c94bc9b9b8a798b3fc8)

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/36abba4d253d4a7387803fda9e4f5ba3)

爆破成功。

![黑客进阶教程——利用burpsuit宏获取token对网站进行暴力破解](http://p1.pstatp.com/large/pgc-image/ff576f800ce6435f9887f7a8ccf5a4ab)