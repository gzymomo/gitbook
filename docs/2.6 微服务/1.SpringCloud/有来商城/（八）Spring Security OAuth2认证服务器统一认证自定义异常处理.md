[Spring Security OAuth2认证服务器统一认证自定义异常处理](https://www.cnblogs.com/haoxianrui/p/14028366.html)



[**本文完整代码下载点击**](https://github.com/hxrui/youlai-mall.git)

[![img](https://i.loli.net/2020/11/23/UEzdVXqR5wsm7Hv.gif)](https://i.loli.net/2020/11/23/UEzdVXqR5wsm7Hv.gif)

# 一. 前言

相信了解过我或者看过我之前的系列文章应该多少知道点我写这些文章包括创建 [**有来商城youlai-mall**](https://github.com/hxrui/youlai-mall.git)  这个项目的目的，想给那些真的想提升自己或者迷茫的人（包括自己--一个工作6年觉得一无是处的菜鸟）提供一块上升的基石。项目是真的从无到有（往期文章佐证），且使用当前主流的开发模式（微服务+前后端分离），最新主流的技术栈（Spring Boot+ Spring Cloud +Spring Cloud Alibaba +  Vue），最流行的统一安全认证授权（OAuth2+JWT）,好了玩笑开完了大家别当真，总之有兴趣一起的小伙伴欢迎加入~

接下来说下这篇文章的原因，之前我是没想过应用到项目中的OAuth2+JWT这套组合拳这么受大家关注，期间一直有童鞋问怎么自定义Spring Security  OAuth2的异常处理、JWT怎么续期、JWT退出等场景下如何失效等问题，所以最近有点时间想把这套统一认证授权完善掉，本篇就以如何自定义Spring Security OAuth2异常处理展开。

**往期文章链接：**

> 后端

1. [Spring Cloud实战 | 第一篇：Windows搭建Nacos服务 ](https://www.cnblogs.com/haoxianrui/p/13581881.html)
2. [Spring Cloud实战 | 第二篇：Spring Cloud整合Nacos实现注册中心](https://www.cnblogs.com/haoxianrui/p/13584204.html)
3. [Spring Cloud实战 | 第三篇：Spring Cloud整合Nacos实现配置中心](https://www.cnblogs.com/haoxianrui/p/13585125.html)
4. [Spring Cloud实战 | 第四篇：Spring Cloud整合Gateway实现API网关](https://www.cnblogs.com/haoxianrui/p/13608650.html)
5. [Spring Cloud实战 | 第五篇：Spring Cloud整合OpenFeign实现微服务之间的调用](https://www.cnblogs.com/haoxianrui/p/13615592.html)
6. [Spring Cloud实战 | 第六篇：Spring Cloud Gateway+Spring Security OAuth2+JWT实现微服务统一认证授权](https://www.cnblogs.com/haoxianrui/p/13719356.html)
7. [Spring Cloud实战 | 最七篇：Spring Cloud Gateway+Spring Security OAuth2集成统一认证授权平台下实现注销使JWT失效方案](https://www.cnblogs.com/haoxianrui/p/13740264.html)
8. [Spring Cloud实战 | 最八篇：Spring Cloud +Spring Security OAuth2+ Vue前后端分离模式下无感知刷新实现JWT续期](https://www.cnblogs.com/haoxianrui/p/14022632.html)
9. [Spring Cloud实战 | 最九篇：Spring Security OAuth2认证服务器统一认证自定义异常处理](https://www.cnblogs.com/haoxianrui/p/14028366.html)

> 管理前端

1. [vue-element-admin实战 | 第一篇： 移除mock接入后台，搭建有来商城youlai-mall前后端分离管理平台](https://www.cnblogs.com/haoxianrui/p/13624548.html)
2. [vue-element-admin实战 | 第二篇： 最小改动接入后台实现根据权限动态加载菜单](https://www.cnblogs.com/haoxianrui/p/13676619.html)

> 微信小程序

1. [vue+uniapp商城实战 | 第一篇：【有来小店】微信小程序快速开发接入Spring Cloud OAuth2认证中心完成授权登录](https://www.cnblogs.com/haoxianrui/p/13882310.html)

# 二. 自定义异常实现代码

直接需要答案的本节走起，添加和修改三个文件即可，异常分析，[**点击下载完整工程代码**](https://github.com/hxrui/youlai-mall.git)

**1. 在youlai-auth认证服务器模块添加全局异常处理器AuthExceptionHandler**



```
package com.youlai.auth.exception;

import com.youlai.common.core.result.Result;
import com.youlai.common.core.result.ResultCode;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.authentication.InternalAuthenticationServiceException;
import org.springframework.security.oauth2.common.exceptions.InvalidGrantException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
@Slf4j
public class AuthExceptionHandler {

    /**
     * 用户名和密码错误
     *
     * @param e
     * @return
     */
    @ExceptionHandler(InvalidGrantException.class)
    public Result handleInvalidGrantException(InvalidGrantException e) {
        return Result.custom(ResultCode.USERNAME_OR_PASSWORD_ERROR);
    }

    /**
     * 账户异常(禁用、锁定、过期)
     *
     * @param e
     * @return
     */
    @ExceptionHandler({InternalAuthenticationServiceException.class})
    public Result handleInternalAuthenticationServiceException(InternalAuthenticationServiceException e) {
        return Result.error(e.getMessage());
    }
}
```

**2. 重写ClientCredentialsTokenEndpointFilter实现客户端自定义异常处理**



```
package com.youlai.auth.filter;

import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.client.ClientCredentialsTokenEndpointFilter;
import org.springframework.security.web.AuthenticationEntryPoint;

/**
 * 重写filter实现客户端自定义异常处理
 */
public class CustomClientCredentialsTokenEndpointFilter extends ClientCredentialsTokenEndpointFilter {

    private AuthorizationServerSecurityConfigurer configurer;
    private AuthenticationEntryPoint authenticationEntryPoint;


    public CustomClientCredentialsTokenEndpointFilter(AuthorizationServerSecurityConfigurer configurer) {
        this.configurer = configurer;
    }

    @Override
    public void setAuthenticationEntryPoint(AuthenticationEntryPoint authenticationEntryPoint) {
        super.setAuthenticationEntryPoint(null);
        this.authenticationEntryPoint = authenticationEntryPoint;
    }

    @Override
    protected AuthenticationManager getAuthenticationManager() {
        return configurer.and().getSharedObject(AuthenticationManager.class);
    }

    @Override
    public void afterPropertiesSet() {
        setAuthenticationFailureHandler((request, response, e) -> authenticationEntryPoint.commence(request, response, e));
        setAuthenticationSuccessHandler((request, response, authentication) -> {
        });
    }
}
```

**3. AuthorizationServerConfig认证服务器配置修改**



```
/**
 * 授权服务配置
 */
@Configuration
@EnableAuthorizationServer
@AllArgsConstructor
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    ......

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        /*security.allowFormAuthenticationForClients();*/
        CustomClientCredentialsTokenEndpointFilter endpointFilter = new CustomClientCredentialsTokenEndpointFilter(security);
        endpointFilter.afterPropertiesSet();
        endpointFilter.setAuthenticationEntryPoint(authenticationEntryPoint());
        security.addTokenEndpointAuthenticationFilter(endpointFilter);

        security.authenticationEntryPoint(authenticationEntryPoint())
                .tokenKeyAccess("isAuthenticated()")
                .checkTokenAccess("permitAll()");
    }

    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint() {
        return (request, response, e) -> {
            response.setStatus(HttpStatus.HTTP_OK);
            response.setHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_UTF8_VALUE);
            response.setHeader("Access-Control-Allow-Origin", "*");
            response.setHeader("Cache-Control", "no-cache");
            Result result = Result.custom(ResultCode.CLIENT_AUTHENTICATION_FAILED);
            response.getWriter().print(JSONUtil.toJsonStr(result));
            response.getWriter().flush();
        };
    }
    ......

}
```

# 三. 异常处理分析

其实你搜一下有关Spring Security  OAuth2如何自定义异常处理，网上会很多差不多解决方案提供参考，但是照搬过来试用，一点效果没有？！咋回事么？其实不能武断的说人家方案不行，最多的可能是Spring Security  OAuth2版本不一致，本篇项目使用的是2.3.4版本，目前截止写这篇文章最新一版的是2020.5.28发布的2.5.0版本，后续项目会升级，如果有差异我会修改本篇文章，总之给大家提供一个解决思路，可行不可行我是不希望大家不能在我里浪费时间。

好了正文开始了~ Spring Security OAuth2认证服务器异常目前我知道的有3类：

1. 用户名或密码错误
2. 账户状态异常
3. 客户端认证异常

有知道其他的欢迎留言补充~，以下就这3类异常逐一分析

在异常处理之前先看下UserDetailsServiceImpl#loadUserByUsername方法抛出的异常信息，如下图：
 [![img](https://i.loli.net/2020/11/23/nTmtIWgYZVuHikL.png)](https://i.loli.net/2020/11/23/nTmtIWgYZVuHikL.png)

### 1. 用户名或密码错误

- **异常分析**



```
org.springframework.security.oauth2.common.exceptions.InvalidGrantException: 用户名或密码错误
```

[![img](https://i.loli.net/2020/11/23/kfEgXiRASZandYK.png)](https://i.loli.net/2020/11/23/kfEgXiRASZandYK.png)

通过异常堆栈信息定位到最终抛出异常的方法是ResourceOwnerPasswordTokenGranter#getOAuth2Authentication，异常类型是InvalidGrantException,其实到这个异常类型中间经过几道转换UsernameNotFoundException->BadCredentialsException->InvalidGrantException

- **处理方法**

添加全局异常处理器捕获（定位标识：AuthExceptionHandler）



```
/**
 * 用户名和密码异常
 *
 * @param e
 * @return
 */
@ExceptionHandler(InvalidGrantException.class)
public Result handleInvalidGrantException(InvalidGrantException e) {
    return Result.error(ResultCode.USERNAME_OR_PASSWORD_ERROR);
}
```

- **结果验证**

验证成功，已按照自定义异常格式返回

[![img](https://i.loli.net/2020/11/23/lNwvqVATuCKMoWG.png)](https://i.loli.net/2020/11/23/lNwvqVATuCKMoWG.png)

### 2. 账户状态异常

- **异常分析**

首先我们需要把数据库youlai的表sys_user的字段status设置为0，表示不可用状态，然后输入正确的用户名和密码，看看跑出来的原生异常信息，可惜的是这个异常没有打印堆栈信息，不过没关系，我们断点调试下，最终定位到ProviderManager#authenticate方法抛出的异常，异常类型是InternalAuthenticationServiceException。

[![img](https://i.loli.net/2020/11/23/bGKlOI6LkAHUzhY.png)](https://i.loli.net/2020/11/23/bGKlOI6LkAHUzhY.png)

- **处理方法**

添加全局异常处理器捕获（定位标识：AuthExceptionHandler）



```
/**
 * 账户异常(禁用、锁定、过期)
 *
 * @param e
 * @return
 */
@ExceptionHandler({InternalAuthenticationServiceException.class})
public Result handleInternalAuthenticationServiceException(InternalAuthenticationServiceException e) {
    return Result.error(e.getMessage());
}
```

- **结果验证**

验证成功，已按照自定义异常格式返回

[![img](https://i.loli.net/2020/11/23/rJCilthgFsfa6LA.png)](https://i.loli.net/2020/11/23/rJCilthgFsfa6LA.png)

### 3. 客户端认证异常

- **异常分析**

之前两种异常方式都可以通过全局异常处理器捕获，且@RestControllerAdvice只能捕获Controller的异常。

客户端认证的异常则是发生在过滤器filter上，此时还没进入DispatcherServlet请求处理流程，便无法通过全局异常处理器捕获。

先看下客户端认证异常出现的位置，首先把客户端ID改成错的。

[![img](https://i.loli.net/2020/11/23/foebrqUjKABRZth.png)](https://i.loli.net/2020/11/23/foebrqUjKABRZth.png)

然后执行“登录”操作，返回错误信息如下：



```
{"error":"invalid_client","error_description":"Bad client credentials"}
```

一眼望去，这显然不是我们想要的格式。

那怎么做才能捕获这个异常转换成自定义数据格式返回呢？显然全局异常处理器无法实现，那必须转换下思路了。

首先客户端的认证是交由ClientCredentialsTokenEndpointFilter来完成的，其中有后置添加失败处理方法，最后把异常交给OAuth2AuthenticationEntryPoint这个所谓认证入口处理。

[![img](https://i.loli.net/2020/11/23/MA1b2sQBhN9iZO4.png)](https://i.loli.net/2020/11/23/MA1b2sQBhN9iZO4.png)

认证入口OAuth2AuthenticationEntryPoint#commence方法中转给父类AbstractOAuth2SecurityExceptionHandler#doHandle方法。

[![img](https://i.loli.net/2020/11/23/sShFNOJL31pRjHX.png)](https://i.loli.net/2020/11/23/sShFNOJL31pRjHX.png)

最后异常定格在AbstractOAuth2SecurityExceptionHandler#doHandle方法上，如下图：

[![img](https://i.loli.net/2020/11/23/ekopFEbYsXftxrn.png)](https://i.loli.net/2020/11/23/ekopFEbYsXftxrn.png)

其中this.enhanceResponse是调用OAuth2AuthenticationEntryPoint#enhanceResponse方法得到响应结果数据。

- **处理方法**

上面我们得知客户端的认证失败异常是过滤器ClientCredentialsTokenEndpointFilter转交给OAuth2AuthenticationEntryPoint得到响应结果的，既然这样我们就可以重写ClientCredentialsTokenEndpointFilter然后使用自定义的AuthenticationEntryPoint替换原生的OAuth2AuthenticationEntryPoint，在自定义AuthenticationEntryPoint处理得到我们想要的异常数据。

自定义AuthenticationEntryPoint设置异常响应数据格式

[![img](https://i.loli.net/2020/11/23/GI8MwAD7qRhfBox.png)](https://i.loli.net/2020/11/23/GI8MwAD7qRhfBox.png)

重写ClientCredentialsTokenEndpointFilter替换AuthenticationEntryPoint

[![img](https://i.loli.net/2020/11/23/2RFrmZbicIxXqyH.png)](https://i.loli.net/2020/11/23/2RFrmZbicIxXqyH.png)

认证服务器配置添加自定义过滤器

[![img](https://i.loli.net/2020/11/23/8FihgWN2Yc4TepA.png)](https://i.loli.net/2020/11/23/8FihgWN2Yc4TepA.png)

- **结果验证**

验证成功，已按照自定义异常格式返回

[![img](https://i.loli.net/2020/11/23/7dr52nSkYyA3Xze.png)](https://i.loli.net/2020/11/23/7dr52nSkYyA3Xze.png)

# 四. 总结

至此，认证服务器的自定义异常处理已全部处理完毕，资源服务器异常处理说明在这篇文章 [Spring Cloud实战 | 第六篇：Spring Cloud Gateway+Spring Security OAuth2+JWT实现微服务统一认证授权](https://www.cnblogs.com/haoxianrui/p/13719356.html)，这就宣告 [**youlai-mall**](https://github.com/hxrui/youlai-mall.git) 的统一认证授权模块基本达到完善的一个标准， 后面继续回到业务功能的开发,所以觉得对你有帮助的给个关注（持续更新）或者给个star，灰常感谢! 最重要的如果你真的对这个项目有兴趣想一起开发学习的像文章开始说的那样请联系我哈~