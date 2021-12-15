[Spring Cloud +Spring Security OAuth2+ Vue前后端分离模式下无感知刷新实现JWT续期](https://www.cnblogs.com/haoxianrui/p/14022632.html)



# 一. 前言

记得上一篇Spring Cloud的文章关于如何使JWT失效进行了理论结合代码实践的说明，想当然的以为那篇会是基于Spring Cloud统一认证架构系列的最终篇。但关于JWT另外还有一个热议的话题是**JWT续期？**。

本篇就个人觉得比较好的JWT续期方案以及落地和大家分享一下，算是抛转引玉，大家有好的方案欢迎留言哈。

> 后端

1. [Spring Cloud实战 | 第一篇：Windows搭建Nacos服务 ](https://www.cnblogs.com/haoxianrui/p/13581881.html)
2. [Spring Cloud实战 | 第二篇：Spring Cloud整合Nacos实现注册中心](https://www.cnblogs.com/haoxianrui/p/13584204.html)
3. [Spring Cloud实战 | 第三篇：Spring Cloud整合Nacos实现配置中心](https://www.cnblogs.com/haoxianrui/p/13585125.html)
4. [Spring Cloud实战 | 第四篇：Spring Cloud整合Gateway实现API网关](https://www.cnblogs.com/haoxianrui/p/13608650.html)
5. [Spring Cloud实战 | 第五篇：Spring Cloud整合OpenFeign实现微服务之间的调用](https://www.cnblogs.com/haoxianrui/p/13615592.html)
6. [Spring Cloud实战 | 第六篇：Spring Cloud Gateway+Spring Security OAuth2+JWT实现微服务统一认证授权](https://www.cnblogs.com/haoxianrui/p/13719356.html)
7. [Spring Cloud实战 | 最七篇：Spring Cloud Gateway+Spring Security OAuth2集成统一认证授权平台下实现注销使JWT失效方案](https://www.cnblogs.com/haoxianrui/p/13740264.html)
8. [Spring Cloud实战 | 最八篇：Spring Cloud +Spring Security OAuth2+ Vue前后端分离模式下无感知刷新实现JWT续期](https://www.cnblogs.com/haoxianrui/p/13740264.html)

> 管理前端

1. [vue-element-admin实战 | 第一篇： 移除mock接入后台，搭建有来商城youlai-mall前后端分离管理平台](https://www.cnblogs.com/haoxianrui/p/13624548.html)
2. [vue-element-admin实战 | 第二篇： 最小改动接入后台实现根据权限动态加载菜单](https://www.cnblogs.com/haoxianrui/p/13676619.html)

> 微信小程序

1. [vue+uniapp商城实战 | 第一篇：【有来小店】微信小程序快速开发接入Spring Cloud OAuth2认证中心完成授权登录](https://www.cnblogs.com/haoxianrui/p/13882310.html)

# 二. 方案

**理论背景：** 在 [**有来商城**](https://github.com/hxrui/youlai-mall.git) 微服务项目  OAuth2实现微服务的统一认证的背景下，前端调用/oauth/token接口认证，在认证成功会返回两个令牌access_token和refresh_token，出于安全考虑access_token时效相较refresh_token短很多（access_token默认12小时,refresh_token默认30天）。当access_token过期或者将要过期时，需要拿refresh_token去刷新获取新的access_token返回给客户端，但是为了客户良好的体验需要做到无感知刷新。

**方案一：**

浏览器起一个定时轮询任务，每次在access_token过期之前刷新。

**方案二：**

请求时返回access_token过期的异常时，浏览器发出一次使用refresh_token换取access_token的请求，获取到新的access_token之后，重试因access_token过期而失败的请求。

**方案比较：**

第一种方案实现简单，但在access_token过期之前刷新，那些旧access_token依然能够有效访问，如果使用黑名单的方式限制这些就的access_token无疑是在浪费资源。

第二种方案是在access_token已经失效的情况下才去刷新便不会有上面的问题，但是它会多出来一次请求，而且实现起来考虑的问题相较下比较多，例如在token刷新阶段后面来的请求如何处理，等获取到新的access_token之后怎么重新重试这些请求。

总结：第一种方案实现简单；第二种方案更为严谨，过期续期不会造成已被刷掉的access_token还有效；总之两者都是可行方案，本篇就第二种方案如何通过前后端的配合实现无感知刷新token实现JWT续期展开说明。

# 三. 实现

直接进入主题，如何通过代码实现在access_token过期时使用refresh_token刷新续期，本篇涉及代码基于Spring Cloud后端[**youlai-mall**](https://github.com/hxrui/youlai-mall.git) 和 Vue前端 [**youlai-mall-admin**](https://github.com/hxrui/youlai-mall-admin.git)，需要的小伙伴可以下载到本地参考下，如果对你有帮助，也希望给个star,感谢~

## 后端

后端部分这里唯一工作是在网关youlai-gateway鉴定access_token过期时抛出一个自定义异常提供给前端判定，如下图所示：

[![img](https://i.loli.net/2020/11/22/Y2jLFHxa6ORSloe.png)](https://i.loli.net/2020/11/22/Y2jLFHxa6ORSloe.png)

小伙伴们在这里也许会有疑问，网关这里如何判断JWT是否已过期？先不急，下文会说明，先看实现之后再说原理。

## 前端

### 1. OAuth2客户端设置

设置OAuth2客户端支持刷新模式,只有这样才能使用refresh_token刷新换取新的access_token。以及为了方便我们测试分别设置access_token和refresh_token的过期时间，因为默认的12小时和30天我们吃不消的；除此之外，还必须满足t(refresh_token) > 60s + t(access_token)的条件，  refresh_token的时效大于access_token时效我们可以理解，那这个60s是怎么回事，别急还是先看实现，原因下文会说明。
 [![img](https://i.loli.net/2020/11/22/qBsFPIlw6Sp5CnG.png)](https://i.loli.net/2020/11/22/qBsFPIlw6Sp5CnG.png)

### 2. 添加刷新令牌方法

设置了支持客户端刷新模式之后，在前端添加一个refreshToken方法，调用的接口和登录认证是同一个接口/oauth/token,只是参数授权方式grant_type的值由password切换到refresh_token，即密码模式切换到刷新模式，这个方法作用是在刷新token之后将新的token写入到localStorage覆盖旧的token。
 [![img](https://i.loli.net/2020/11/22/g7WfsvialEXhDIo.png)](https://i.loli.net/2020/11/22/g7WfsvialEXhDIo.png)

### 3. 请求响应拦截添加令牌过期处理

在判断响应结果是token过期时，执行刷新令牌方法覆盖本地的token。

在刷新期间需做到两点，一是避免重复刷新，二是请求重试，为了满足以上两点添加了两个关键变量：

- **refreshing**----刷新标识

在第一次access_token过期请求失败时，调用刷新token请求时开启此标识，标识当前正在刷新中，避免后续请求因token失效重复刷新。

- **waitQueue**----请求等待队列

当执行刷新token期间时，需要把后来的请求先缓存到等待队列，在刷新token成功时，重新执行等待队列的请求即可。

修改请求响应封装request.js的代码如下，关键部分使用注释说明，完整工程 [**youlai-mall-admin**](https://github.com/hxrui/youlai-mall-admin.git)



```
let refreshing = false,// 正在刷新标识，避免重复刷新
  waitQueue = [] // 请求等待队列

service.interceptors.response.use(
  response => {
    const {code, msg, data} = response.data
    if (code !== '00000') {
      if (code === 'A0230') { // access_token过期 使用refresh_token刷新换取access_token
        const config = response.config
        if (refreshing == false) {
          refreshing = true
          const refreshToken = getRefreshToken()
          return store.dispatch('user/refreshToken', refreshToken).then((token) => {
            config.headers['Authorization'] = 'Bearer ' + token
            config.baseURL = '' // 请求重试时，url已包含baseURL
            waitQueue.forEach(callback => callback(token)) // 已刷新token，所有队列中的请求重试
            waitQueue = []
            return service(config)
          }).catch(() => { // refresh_token也过期，直接跳转登录页面重新登录
            MessageBox.confirm('当前页面已失效，请重新登录', '确认退出', {
              confirmButtonText: '重新登录',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              store.dispatch('user/resetToken').then(() => {
                location.reload()
              })
            })
          }).finally(() => {
            refreshing = false
          })
        } else {
          // 正在刷新token，返回未执行resolve的Promise,刷新token执行回调
          return new Promise((resolve => {
            waitQueue.push((token) => {
              config.headers['Authorization'] = 'Bearer ' + token
              config.baseURL = '' // 请求重试时，url已包含baseURL
              resolve(service(config))
            })
          }))
        }
      } else {
        Message({
          message: msg || '系统出错',
          type: 'error',
          duration: 5 * 1000
        })
      }
    }
    return {code, msg, data}
  },
  error => {
    return Promise.reject(error)
  }
)
```

# 四. 测试

完成上面前后端代码调整之后，接下来进入测试，还记得上面设置access_token时效为1s、refresh_token为120s吧。这里access_token设置为1s，但是时效确是61s，至于原因下文细说。这里把测试根据时间分为3个阶段：

1. **0~61s：双token都没过期，正常请求过程。**

[![img](https://i.loli.net/2020/11/22/EPU5rkCgAq3JYRw.png)](https://i.loli.net/2020/11/22/EPU5rkCgAq3JYRw.png)

1. **61s~120s：access_token过期，再次请求会执行一次刷新请求。**

[![img](https://i.loli.net/2020/11/22/TvXE28hmV5RI4b1.png)](https://i.loli.net/2020/11/22/TvXE28hmV5RI4b1.png)
 [![img](https://i.loli.net/2020/11/22/gnOrV9EtIwTisZ4.png)](https://i.loli.net/2020/11/22/gnOrV9EtIwTisZ4.png)

1. **120s+： refresh_token过期，神仙都救不了，重新登录。**

[![img](https://i.loli.net/2020/11/22/BmtkUp1TfEbghIy.png)](https://i.loli.net/2020/11/22/BmtkUp1TfEbghIy.png)

# 五. 问题

**声明：** 问题基于[**youlai-mall**](https://github.com/hxrui/youlai-mall.git)项目使用的nimbus-jose-jwt这个JWT库，依赖spring-security-oauth2-jose这个jar包。

**1. 如何判定JWT过期？**

JWT的是否过期判断最终落点是在JwtTimestampValidator#validate方法上

**2.为什么access_token比设定多了60s时效？**

开挂？有后台？向天再借60s？

刚开始在不知情的情况下以为自己哪里配置错了，设置5s过期，等个1min多钟。后来确实没办法决心去调试下源码，最后找到JWT验证过期的方法JwtTimestampValidator#validate

[![img](https://i.loli.net/2020/11/22/Fur6ghT4xCEAcGB.png)](https://i.loli.net/2020/11/22/Fur6ghT4xCEAcGB.png)

基本上满足 `Instant.now(this.clock).minus(this.clockSkew).isAfter(expiry)` 就说明JWT过期了

now - 60s > expiry   **=转换=>**   now > expiry + 60s

按正常理解当前时间大于过期时间就可判定为过期，但这里却在过期时间加了个时钟偏移60s，活生生的延长了一分钟，至于为什么？没找到说明文档，注释也没说明，知道的小伙伴欢迎下方留言~

# 六. 总结

本篇讲述 [**youlai-mall**](https://github.com/hxrui/youlai-mall.git)  项目中如何通过前后端配合利用双token刷新实现JWT续期的功能需求，后端抛出token过期异常，前端捕获之后调用刷新token请求，成功则完成续期，失败（一般指refresh_token也过期了）则需要重新登录。在代码的实现过程中了解到在资源服务器（youlai-gateway）如何判断JWT是否过期、axios如何进行请求重试等一些问题。

最后说一下自己的项目，[**youlai-mall**](https://github.com/hxrui/youlai-mall.git) 集成当前主流开发模式微服务加前后端分离，当前最新主流技术栈 Spring Cloud + Spring Cloud Alibaba + Vue , 以及最流行统一认证授权Spring Cloud Gateway + Spring Security OAuth2 +  JWT。所以觉得本文对你有所帮助的话给个关注（持续更新中...），或者对该项目感兴趣的小伙伴给个star，也期待你的加入和建议，还是老样子有问题随时联系我~（微信号：haoxianrui）。

| 项目名称   | 地址                                                         |
| ---------- | ------------------------------------------------------------ |
| 后台       | [youlai-mall](https://github.com/hxrui/youlai-mall)          |
| 管理前端   | [youlai-mall-admin](https://github.com/hxrui/youlai-mall-admin) |
| 微信小程序 | [youlai-mall-weapp](https://github.com/hxrui/youlai-mall-weapp) |