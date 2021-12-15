[Spring Cloud Gateway+Spring Security OAuth2集成统一认证授权平台下实现注销使JWT失效方案](https://www.cnblogs.com/haoxianrui/p/13740264.html)



# 一. 前言

在上一篇文章介绍 [youlai-mall](https://github.com/hxrui/youlai-mall) 项目中，通过整合Spring Cloud Gateway、Spring Security  OAuth2、JWT等技术实现了微服务下统一认证授权平台的搭建。最后在文末留下一个值得思考问题，就是如何在注销、修改密码、修改权限场景下让JWT失效？所以在这篇文章来对方案和实现进行补充。想亲身体验的小伙伴们可以了解下 [youlai-mall](https://github.com/hxrui/youlai-mall) 项目和Spring Cloud实战系列往期文章。

**[youlai-mall项目地址](https://github.com/hxrui/youlai-mall)**

**往期系列文章**

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

# 二. 解决方案

JWT最大的一个优势在于它是**无状态**的，自身包含了认证鉴权所需要的所有信息，服务器端无需对其存储，从而给服务器减少了存储开销。

但是无状态引出的问题也是可想而知的，它无法作废未过期的JWT。举例说明注销场景下，就传统的cookie/session认证机制，只需要把存在服务器端的session删掉就OK了。但是JWT呢，它是不存在服务器端的啊，好的那我删存在客户端的JWT行了吧。额，社会本就复杂别再欺骗自己了好么，被你在客户端删掉的JWT还是可以通过服务器端认证的。

[![img](https://i.loli.net/2020/09/27/Vd7KRprwuJz16CE.png)](https://i.loli.net/2020/09/27/Vd7KRprwuJz16CE.png)

首先明确一点JWT失效的唯一途径就是等过期，就是说不借助外力的情况下，无法达到某些场景下需要主动使JWT失效的目的。而外力则是在服务器端存储着JWT的状态，在请求资源时添加判断逻辑，这与JWT特性无状态是相互矛盾的存在。但是，你要知道如果你选择走上了JWT这条路，那就没得选了。如果你有好的方式，希望你来打我脸。

以下就JWT在某些场景需要失效的简单方案整理如下：

**1. 白名单方式**

认证通过时，把JWT缓存到Redis，注销时，从缓存移除JWT。请求资源添加判断JWT在缓存中是否存在，不存在拒绝访问。这种方式和cookie/session机制中的会话失效删除session基本一致。

**2. 黑名单方式**

注销登录时，缓存JWT至Redis，且缓存有效时间设置为JWT的有效期，请求资源时判断是否存在缓存的黑名单中，存在则拒绝访问。

白名单和黑名单的实现逻辑差不多，黑名单不需每次登录都将JWT缓存，仅仅在某些特殊场景下需要缓存JWT，给服务器带来的压力要远远小于白名单的方式。

# 三. 黑名单方式实现

以下演示在退出登录时通过添加至黑名单的方式实现JWT失效

逻辑很明确，在调用退出登录接口时将JWT缓存到Redis的黑名单中，然后在网关做判定请求头的JWT是否在黑名单内做对应的处理。

### 1. 认证中心(youlai-auth)退出登录接口

登出接口/oauth/logout的主要逻辑把JWT添加至Redis黑名单缓存中，但没必要把整个JWT字符串都存储下来，JWT的载体中有个jti(JWT ID)字段声明为JWT提供了唯一的标识符。JWT解析的结构如下：

[![img](https://i.loli.net/2020/09/19/8SuirOcdvGt3ACm.png)](https://i.loli.net/2020/09/19/8SuirOcdvGt3ACm.png)

既然有这么个字段能作为JWT的唯一标识，从JWT解析出jti之后将其存储到黑名单中作为判别依据，相较于存储完整的JWT字符串减少了存储开销。另外我们只需保证JWT在其有效期内用户登出后失效就可以了，JWT有效期过了黑名单也就没有存在的必要，所以我们这里还需要设置黑名单的过期时间，不然黑名单的数量会无休止的越来越多，这是我们不想看到的。



```
@Api(tags = "认证中心")
@RestController
@RequestMapping("/oauth")
@AllArgsConstructor
public class AuthController {

    private RedisTemplate redisTemplate;

    @DeleteMapping("/logout")
    public Result logout(HttpServletRequest request) {
        String payload = request.getHeader(AuthConstants.JWT_PAYLOAD_KEY);
        JSONObject jsonObject = JSONUtil.parseObj(payload);

        String jti = jsonObject.getStr("jti"); // JWT唯一标识
        long exp = jsonObject.getLong("exp"); // JWT过期时间戳(单位:秒)

        long currentTimeSeconds = System.currentTimeMillis() / 1000;

        if (exp < currentTimeSeconds) { // token已过期
            return Result.custom(ResultCode.INVALID_TOKEN_OR_EXPIRED);
        }
        redisTemplate.opsForValue().set(AuthConstants.TOKEN_BLACKLIST_PREFIX + jti, null, (exp - currentTimeSeconds), TimeUnit.SECONDS);
        return Result.success();
    }
}
```

### 2. 网关(youlai-gateway)的全局过滤器

从请求头提取JWT，解析出唯一标识jti，然后判断该标识是否存在黑名单列表里，如果是直接返回响应token失效的提示信息。



```
/**
 * 全局过滤器 黑名单token过滤
 */
@Component
@Slf4j
@AllArgsConstructor
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    private RedisTemplate redisTemplate;

    @SneakyThrows
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst(AuthConstants.JWT_TOKEN_HEADER);
        if (StrUtil.isBlank(token)) {
            return chain.filter(exchange);
        }
        token = token.replace(AuthConstants.JWT_TOKEN_PREFIX, Strings.EMPTY);
        JWSObject jwsObject = JWSObject.parse(token);
        String payload = jwsObject.getPayload().toString();

        // 黑名单token(登出、修改密码)校验
        JSONObject jsonObject = JSONUtil.parseObj(payload);
        String jti = jsonObject.getStr("jti"); // JWT唯一标识

        Boolean isBlack = redisTemplate.hasKey(AuthConstants.TOKEN_BLACKLIST_PREFIX + jti);
        if (isBlack) {
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.OK);
            response.getHeaders().set(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
            response.getHeaders().set("Access-Control-Allow-Origin", "*");
            response.getHeaders().set("Cache-Control", "no-cache");
            String body = JSONUtil.toJsonStr(Result.custom(ResultCode.INVALID_TOKEN_OR_EXPIRED));
            DataBuffer buffer = response.bufferFactory().wrap(body.getBytes(Charset.forName("UTF-8")));
            return response.writeWith(Mono.just(buffer));
        }

        ServerHttpRequest request = exchange.getRequest().mutate()
                .header(AuthConstants.JWT_PAYLOAD_KEY, payload)
                .build();
        exchange = exchange.mutate().request(request).build();
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 3. 注销后JWT失效测试

测试流程涉及到以下3个接口

[![img](https://i.loli.net/2020/09/26/gQm39Z8KjY4yAG1.png)](https://i.loli.net/2020/09/26/gQm39Z8KjY4yAG1.png)

**1. 登录访问资源**

- http://localhost:9999/youlai-auth/oauth/token
- http://localhost:9999/youlai-admin/users/me

[![img](https://i.loli.net/2020/09/26/vTf1B5lVsNoZGjJ.png)](https://i.loli.net/2020/09/26/vTf1B5lVsNoZGjJ.png)

**2. 退出登录再次访问资源**

- http://localhost:9999/youlai-auth/oauth/logout
- http://localhost:9999/youlai-admin/users/me

退出成功查看redis缓存黑名单列表

[![img](https://i.loli.net/2020/09/26/sBtTDa4ZWJObFMU.png)](https://i.loli.net/2020/09/26/sBtTDa4ZWJObFMU.png)

再次访问登录用户信息如下：

[![img](https://i.loli.net/2020/09/26/xBwAYrSEJP5ivhQ.png)](https://i.loli.net/2020/09/26/xBwAYrSEJP5ivhQ.png)

可以看到退出登录后再次使用原JWT请求提示“token无效或已过期”

**3. youlai-mall项目退出登录演示**

上面报“token无效或已过期”的响应码是"A0230"，这个对应的是Java开发手册【泰山版】的错误码

[![img](https://i.loli.net/2020/09/27/ruOvYfkbFdWaq3n.png)](https://i.loli.net/2020/09/27/ruOvYfkbFdWaq3n.png)

打开之前搭建好的前端管理平台[youlai-mall-admin-web](https://github.com/hxrui/youlai-mall-admin-web)，修改src/util/request.js文件中的无效token的响应码为“A0230”，这样在token无效的情况下提示重新登录

[![img](https://i.loli.net/2020/09/27/QpPlOqoD1nre7ZR.png)](https://i.loli.net/2020/09/27/QpPlOqoD1nre7ZR.png)

演示通过第三方接口调试工具调用注销接口让JWT失效，然后再次刷新页面请求资源会因为JWT的失效而跳转到登录页。

[![img](https://i.loli.net/2020/09/27/FHqm5M2b7iUyDzL.gif)](https://i.loli.net/2020/09/27/FHqm5M2b7iUyDzL.gif)

# 四. 总结

JWT是JSON风格轻量级的授权和身份认证规范,可实现无状态、分布式应用的统一认证鉴权。但是事物往往具有两面性，有利必有弊，因为JWT的无状态，自生成后不借助外界条件唯一失效的方式就是过期。然而借助的外界的条件后JWT便有状态了的，也就是没有所谓严格意义上的无状态，其实也不必纠结于此，因为瑕不掩瑜。在白名单和黑名单的实现方式，这里选择了后者状态性更小的黑名单方式。还是文中提到过的一句话，如果你有更好的实现方式，欢迎留言告知，不胜感激！

本篇是暂阶段的Spring Cloud实战的最终章了，也就是说基于Spring Boot +Spring Cloud+  Element-UI搭建的前后端分离基础权限框架已经搭建完成。后面计划写使用此基础框架整合uni-app跨平台前端框架开发一套商城小程序，希望大家给个关注或star，感谢感谢~

**本篇完整代码下载地址**：

[youlai-mall](https://github.com/hxrui/youlai-mall)

[youlai-mall-admin-web](https://github.com/hxrui/youlai-mall-admin-web)