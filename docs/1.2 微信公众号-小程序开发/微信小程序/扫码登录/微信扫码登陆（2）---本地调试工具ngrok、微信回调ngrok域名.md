[微信扫码登陆（2）---本地调试工具ngrok、微信回调ngrok域名](https://www.cnblogs.com/qdhxhz/p/9678137.html)

## 一、本地调试工具ngrok

####   1、什么是ngrok？ 

 简单总结下：内网穿透利器，使用反向代理原理，达到从外网访问防火墙内部的服务。

####   2、ngrok作用

举例：用户微信扫码授权成功后，会带上用户信息回调对应的域名，但在本地电脑开发，微信没法回调，所以需要配置个地址映射，就是微信服务器

​     可以通过这个地址访问当前开发电脑的地址，这样我们就可以很好的进行本地代码调试了。

参考Sunny-Ngrok说的: 为什么使用Sunny-Ngrok？

  (1)提供免费内网穿透服务，免费服务器支持绑定自定义域名

  (2)管理内网服务器，内网web进行演示

  (3)快速开发微信程序和第三方支付平台调试

  (4)本地WEB外网访问、本地开发微信、TCP端口转发

  (5)本站新增FRP服务器，基于 FRP 实现https、udp转发

  (6)无需任何配置，下载客户端之后直接一条命令让外网访问您的内网不再是距离

####   3、为什么ngrock能做到让域名映射到本地

我从表面意思来理解，微信既然通过域名回调到你本地，说明该域名和本地（127.0.0.1）是一个1对1的关系

它这里做了两点：

（1） 域名是唯一的，比如我注册的：http://jincou.vipgz1.idcfengye.com 就是唯一的，只要我注册了，其它人就不能注册。

（2）该域名启动是唯一的，就是只要我在本地启动该域名对应ngrok，那么其它电脑就不能启动该域名的ngrok了。

所以保障唯一，就能映射到本地。

####    4、ngrock工具启动

 我这里采用的是Sunny-Ngrok进行搭建，具体文档说明可以参考官方教程：https://www.ngrok.cc/

 **（1）我在官网买了个服务，自己注册了唯一域名，该域名对应本地端口号为：8080**

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180919220358979-1038470310.png)

 **（2）启动ngrok**

```
./sunny clientid 隧道id
```

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180919221930994-1703986069.png)

代表启动成功！

 **（3）测试**

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180919222228675-935806120.png)

成功！

 

## 二、微信回调ngrok域名

####   1、项目开发前说明

  用户扫码授权之后回调ngrok域名的时候，并没有用sunny-ngrok，而是用另一个ngrok，不是sunny-ngrok不好用，是我自己的公众号主体是个人，无法获取完成微信认证，

所以就无法获取回调Url，自己也用申请了公众平台测试账号，进行调试，发现Token是认证通过了，授权回调页面域名也填了，但是在调登陆二维码的时候一直报：

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180920213510944-227524219.png)

我在想应该是公众平台测试账号没有这个权限吧。

先说明：如果你想用ngrok验证微信是否会回调ngrok域名到本地，那么是可以用公众平台测试账号的，因为token验证的时候会回调接口的。

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180920213616249-225477797.png)

至于Token如何验证，可以看该链接教程：[微信token验证失败Java解决办法](https://blog.csdn.net/chmod_r_755/article/details/75554735)

####   2、ngrok工具启动

因为本人没有微信公众号通过认证，所以用人家的appid和回调域名，所以这里用的是另一个ngrok工具，其实和上面大同小异。

首先，下载ngrok工具到你的计算机中，使用方法是：

  (1) 先在命令行窗口下切换到ngrok工具的位置下。

  (2) 然后输入命令：./ngrok -config=ngrok.cfg -subdomain example 8080

说明：

 example — 可以自己设置，如helloworld,但这个一定要配置过的，如上面sunny-ngrok

   8080 — 你服务器的端口号，如tomcat的服务器为8083，就改为8083

启动成功，所实现的功能就和上面一样了

 ![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180920213943426-919380054.png)

####   3、本地接口代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Controller
@RequestMapping("/api/v1/wechat")
public class WechatController {


    /**
     * 微信开放平台二维码连接
     */
    private final static String OPEN_QRCODE_URL = "https://open.weixin.qq.com/connect/qrconnect?appid=%s&redirect_uri=%s&response_type=code&scope=snsapi_login&state=%s#wechat_redirect";

    /**
     * 开放平台回调url
     * 注意：test16web.tunnel.qydev.com 域名地址要和在微信端 回调域名配置 地址一直，否则会报回调地址参数错误
     * 
     * http://test16web.tunnel.qydev.com 映射 127.0.0.1:8080,所以可以回调下面接口
     */
    private final static String OPEN_REDIRECT_URL = "http://test16web.tunnel.qydev.com/api/v1/wechat/user/code";

    /**
     * 微信审核通过后的appid
     */
    private final static String OPEN_APPID = "wx025575eac69a2d5b";

    /**
     * 拼装微信扫一扫登录url
     */
    @GetMapping("login_url")
    @ResponseBody
    public JsonData loginUrl(@RequestParam(value = "access_page", required = true) String accessPage) throws UnsupportedEncodingException {

        //官方文档说明需要进行编码
        String callbackUrl = URLEncoder.encode(OPEN_REDIRECT_URL, "GBK"); //进行编码

        //格式化，返回拼接后的url，去调微信的二维码
        String qrcodeUrl = String.format(OPEN_QRCODE_URL, OPEN_APPID, callbackUrl, accessPage);

        return JsonData.buildSuccess(qrcodeUrl);

    }

    /**
     * 用户授权成功，获取微信回调的code
     */
    @GetMapping("/user/code")
    public void wechatUserCallback(@RequestParam(value = "code",required = true) String code){

        System.out.println(code);

    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 用户扫一扫二维码授权成功后如图：

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180920215144984-644714656.png)

 再看idea后台，惊喜的发现，确实带上code来回调本地代码来，nice！

![img](https://img2018.cnblogs.com/blog/1090617/201809/1090617-20180920215353686-1365267422.png)

 

下一遍就该写，如果通过code获取access_token用户信息了。

 

### 参考

   [微信开发地址映射工具下载及使用](https://mp.weixin.qq.com/s?__biz=MzUyMDg1MDE2MA==&mid=2247484072&idx=1&sn=cc47e4d0d4ed992c7a5ae08fbabfd3c1&source=41#wechat_redirect)