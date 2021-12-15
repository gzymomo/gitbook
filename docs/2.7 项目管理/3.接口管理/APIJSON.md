[APIJSON 自动化接口和文档 ORM 库](https://www.oschina.net/p/api-json)

APIJSON是一种专为API而生的 JSON网络传输协议 以及 基于这套协议实现的ORM库。
为 简单的增删改查、复杂的查询、简单的事务操作 提供了完全自动化的API。
能大幅降低开发和沟通成本，简化开发流程，缩短开发周期。
适合中小型前后端分离的项目，尤其是互联网创业项目和企业自用项目。

通过自动化API，前端可以定制任何数据、任何结构！
大部分HTTP请求后端再也不用写接口了，更不用写文档了！
前端再也不用和后端沟通接口或文档问题了！再也不会被文档各种错误坑了！
后端再也不用为了兼容旧接口写新版接口和文档了！再也不会被前端随时随地没完没了地烦了！

# 一、特点功能

## 1.1 在线解析

- 自动生成接口文档，清晰可读永远最新
- 自动校验与格式化，支持高亮和收展
- 自动生成各种语言代码，一键下载
- 自动管理与测试接口用例，一键共享
- 自动给请求JSON加注释，一键切换

## 1.2 对于前端

- 不用再向后端催接口、求文档
- 数据和结构完全定制，要啥有啥
- 看请求知结果，所求即所得
- 可一次获取任何数据、任何结构
- 能去除重复数据，节省流量提高速度

## 1.3 对于后端

- 提供通用接口，大部分API不用再写
- 自动生成文档，不用再编写和维护
- 自动校验权限、自动管理版本、自动防SQL注入
- 开放API无需划分版本，始终保持兼容
- 支持增删改查、模糊搜索、正则匹配、远程函数等

 

[![img](https://static.oschina.net/uploads/img/202009/10112721_SSmG.jpg)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_Auto_get.jpg)

多表关联查询、结构自由组合、多个测试账号、一键共享测试用例

[![img](https://static.oschina.net/uploads/img/202009/10112738_eYLF.jpg)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_Auto_code.jpg)

自动生成封装请求JSON的Android与iOS代码、一键自动生成JavaBean或解析Response的代码

[![img](https://static.oschina.net/uploads/img/202009/10112858_ttWQ.jpg)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_Auto_doc.jpg)

自动保存请求记录、自动生成接口文档，可添加常用请求、快捷查看一键恢复

[![img](https://static.oschina.net/uploads/img/202009/10112922_LDYO.jpg)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_Auto_test.jpg)

一键自动接口回归测试，不需要写任何代码(注解、注释等全都不要)

[![img](https://static.oschina.net/uploads/img/202009/10112943_1k1W.jpg)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_Auto_summary.jpg)

一图胜前言 - 部分基础功能概览
[以下Gif图看起来比较卡，实际在手机上App运行很流畅]
 [![img](https://static.oschina.net/uploads/img/202009/10114412_RwLw.gif)](https://raw.githubusercontent.com/TommyLemon/StaticResources/master/APIJSON_App_Moment_Comment.gif)

 

# 二、为什么要用APIJSON？

[前后端 关于接口的 开发、文档、联调 等 10 大痛点解析](https://gitee.com/TommyLemon/APIJSON/wikis)

# 三、常见问题

## 3.1 如何定制业务逻辑？

在后端编写 远程函数，可以拿到 session、version、当前 JSON 对象、参数名称 等，然后对查到的数据自定义处理
https://github.com/APIJSON/APIJSON/issues/101

## 3.2 如何控制权限？

在 Access 表配置校验规则，默认不允许访问，需要对 每张表、每种角色、每种操作 做相应的配置，粒度细分到 Row 级
https://github.com/APIJSON/APIJSON/issues/12

## 3.3 如何校验参数？

在 Request 表配置校验规则 structure，提供 NECESSARY、TYPE、VERIFY 等通用方法，可通过 远程函数 来完全自定义
[https://github.com/APIJSON/APIJSON/wiki#%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86](https://github.com/APIJSON/APIJSON/wiki#实现原理)


**其它问题见 Closed Issues**
https://github.com/APIJSON/APIJSON/issues?q=is%3Aissue+is%3Aclosed

# 四、快速上手

## 4.1 后端部署

可以跳过这个步骤，直接用APIJSON服务器IP地址 apijson.cn:8080 来测试接口。
见 [APIJSON后端部署 - Java](https://gitee.com/TommyLemon/APIJSON/tree/master/APIJSON-Java-Server)

## 4.2 前端部署

可以跳过这个步骤，直接使用 [APIAuto-自动化接口管理工具](https://gitee.com/TommyLemon/APIAuto) 或 下载客户端App。
见 [Android](https://gitee.com/TommyLemon/APIJSON/tree/master/APIJSON-Android) 或 [iOS](https://gitee.com/TommyLemon/APIJSON/tree/master/APIJSON-iOS) 或 [JavaScript](https://gitee.com/TommyLemon/APIJSON/tree/master/APIJSON-JavaScript)