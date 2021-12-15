微信公众号：[芋道源码]：SpringBoot 项目中，Spring Security 和 Shiro 该如何选择？

# Shiro

Apache Shiro是一个强大且易用的Java安全框架,能够非常清晰的处理认证、授权、管理会话以及密码加密。使用Shiro的易于理解的API,您可以快速、轻松地获得任何应用程序,从最小的移动应用程序到最大的网络和企业应用程序。

## 执行流程

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfctJZS4sOG8Dyl1CK6fLXdWBJwrT91AibhLl9VRFt4zxjFtoqh9TT7CGiasrWvzibicYW88QEj2ngAX8w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)图片

## 特点

1. 易于理解的 Java Security API；
2. 简单的身份认证（登录），支持多种数据源（LDAP，JDBC，Kerberos，ActiveDirectory 等）；
3. 对角色的简单的签权（访问控制），支持细粒度的签权；
4. 支持一级缓存，以提升应用程序的性能；
5. 内置的基于 POJO 企业会话管理，适用于 Web 以及非 Web 的环境；
6. 异构客户端会话访问；
7. 非常简单的加密 API；
8. 不跟任何的框架或者容器捆绑，可以独立运行。

# Spring Security

Spring Security 主要实现了Authentication（认证，解决who are you? ） 和 Access  Control（访问控制，也就是what are you allowed to do？，也称为Authorization）。Spring  Security在架构上将认证与授权分离，并提供了扩展点。它是一个轻量级的安全框架，它确保基于Spring的应用程序提供身份验证和授权支持。**它与Spring MVC有很好地集成** ，并配备了流行的安全算法实现捆绑在一起。

## 执行流程

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/JdLkEI9sZfctJZS4sOG8Dyl1CK6fLXdWxqyO0m0skViaclB73Fia6Vs12me56pfNMVic9bBg27OaALO4TAu1PEWyw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)图片

1. 客户端发起一个请求，进入 Security 过滤器链。
2. 当到 LogoutFilter 的时候判断是否是登出路径，如果是登出路径则到 logoutHandler ，如果登出成功则到  logoutSuccessHandler 登出成功处理，如果登出失败则由 ExceptionTranslationFilter  ；如果不是登出路径则直接进入下一个过滤器。
3. 当到 UsernamePasswordAuthenticationFilter  的时候判断是否为登录路径，如果是，则进入该过滤器进行登录操作，如果登录失败则到 AuthenticationFailureHandler  登录失败处理器处理，如果登录成功则到 AuthenticationSuccessHandler  登录成功处理器处理，如果不是登录请求则不进入该过滤器。
4. 当到 FilterSecurityInterceptor 的时候会拿到 uri ，根据 uri 去找对应的鉴权管理器，鉴权管理器做鉴权工作，鉴权成功则到 Controller 层否则到 AccessDeniedHandler 鉴权失败处理器处理。

## 特点

shiro能实现的，Spring Security 基本都能实现，依赖于Spring体系，但是好处是Spring全家桶的亲儿子，集成上更加契合，在使用上，比shiro略负责。

# 两者对比

Shiro比Spring Security更容易使用，也就是实现上简单一些，同时基本的授权认证Shiro也基本够用

Spring Security社区支持度更高，Spring社区的亲儿子，支持力度和更新维护上有优势，同时和Spring这一套的结合较好。

Shiro 功能强大、且 简单、灵活。是Apache 下的项目比较可靠，且不跟任何的框架或者容器绑定，可以独立运行。

# 我的看法

如果开发的项目是Spring这一套，用Spring  Security我觉得更合适一些，他们本身就是一套东西，顺畅，可能略微复杂一些，但是学会了就是自己的。如果开发项目比较紧张，Shiro可能更合适，容易上手，也足够用，Spring Security中有的，Shiro也基本都有，没有的部分网上也有大批的解决方案。

如果项目没有使用Spring这一套，不用考虑，直接Shiro。