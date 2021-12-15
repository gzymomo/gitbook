[TOC]

[[认证授权：一键登录的背后过程](https://www.cnblogs.com/cwsheng/p/13511864.html)]

　以QQ登录简书为例，来查看整个过程。

　1、进入QQ登录页面：

　　可以通过F12查看源码知道：点击简书登录界面QQ图标是打开到了 **/users/auth/qq_connect** 页面，如下图1；但我们点击后查看到的界面却是QQ登录界面,如下图2

　　![图1](https://img2020.cnblogs.com/blog/374428/202008/374428-20200816163833203-712882987.png)

![img](https://img2020.cnblogs.com/blog/374428/202008/374428-20200816164643852-897845717.png)

　　**值得注意的是**，我们已经跳转到了QQ的服务器地址了：https://**graph.qq.com**/oauth2.0/show?which=Login&display=pc&**client_id**=100410602&**redirect_uri**=http%3A%2F%2Fwww.jianshu.com%2Fusers%2Fauth%2Fqq_connect%2Fcallback&**response_type**=code&**state**=%257B%257D

　　通过发现里面有几个特别的参数：

| **参数名 **                    | **参数值**                                                   |
| ------------------------------ | ------------------------------------------------------------ |
| **client_id（客户端id）**      | **100410602（这是来源哪里呢？）**                            |
| **redirect_uri（重定向地址）** | http%3A%2F%2Fwww.jianshu.com%2Fusers%2Fauth%2Fqq_connect%2Fcallback**（是简书的一个回调地址）** |
| **response_type（相应类型）**  | **code（代表什么意思呢？）**                                 |
| **state（状态）**              | **%257B%257D**                                               |

　　通过观察该页面主要包含两部分：

　　　a）账号密码输入框

　　　b）授权简书可访问该账号的权限内容：获取昵称、头像、性别 

　2、输入账号同意授权登录后，我们就直接回到了简书的主页面中，此时已登录用户。如图

　　　对于用户来说页面在登录后就调整到了主页，但在程序过程中却经历了好几个步骤：

　　　a）登录用户名、密码校验

　　　**b）获取授权码，返回设置的回调地址**

　　　**c）根据授权码获取access_token**

　　　**d）根据access_token获取OpenID**

　　　**e）根据OpenId获取用户信息**

　　　**f）返回跳转到简书主页**

　　　![img](https://img2020.cnblogs.com/blog/374428/202008/374428-20200816182755723-1760030091.png)

　那么这整个过程在程序中是怎么完成的呢?接下来我们用一张图来介绍完整过程。

## 流程回顾

　　根据上面流程绘制了如下认证流程图：

　![img](https://img2020.cnblogs.com/blog/374428/202008/374428-20200816205450643-954873583.png) 

 　在[QQ互联开发网站](https://wiki.connect.qq.com/)中，我们可以了解到QQ是OAuth方式实现的，那么现在可能大家就有些疑问:

　　a）OAuth是什么？

　　　 **OAuth（开放授权）是一个开放标准，允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方网站或分享他们数据的所有内容。**

　　b）该过程中简书服务器、QQ认证服务器、QQApi服务器到底是什么，有什么关系？

　　c）过程中的授权码、Token、openId又是什么呢？

　　d）……

　　带着这些问题，我们将一步步去学习，解决这些问题。已到达完整的了解整个过程　　

## 总结

　　通过以上分析主要步骤包含：

　　1、在简书登录界面点击QQ登录图标

　　2、简书后台(users/auth/qq_connect)重定向到QQ用户登录界面；需要携带参数（response_type+client_id+redirect_uri+state ）

　　3、输入QQ点击登录授权，校验QQ用户，生成授权码；返回简书回调地址

　　4、简书服务器根据获取的授权码获取获取Access Token

　　5、根据Access Token获取对应用户身份的OpenID

　　6、根据OpenID，调用OpenApi接口