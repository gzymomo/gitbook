# **一 公众号配置服务器**

微信官方提供了非常完善的接入文档，如果想了解文档的具体内容，直接浏览器搜索**微信开发文档**就可以了。但是为了方便开发，一般不会直接去根据微信开发文档进行开发，github上有许多开源项目对微信开发文档进行了封装，这里我使用`mica-weixin`开发包进行演示，`mica-weixin`是`jfinal-weixin`的boot版本。

配置服务器信息很简单，具体流程就是**微信服务发送请求一个请求给业务服务器，业务服务器验证请求后给微信服务一个响应**。

## 1.1 搭建业务服务

本地搭建一个`spring-boot-weixin`的项目，使用内网穿透工具进行穿透，使其可以与外网进行通信。

### 1.1.1 引入`mica-weixin`依赖

```xml
<dependency>
    <groupId>net.dreamlu</groupId>
    <artifactId>mica-weixin</artifactId>
    <version>2.0.1</version>
</dependency>
```

1.1.2 配置公众号信息

`mica-weixin`通过配置文件进行公众号信息的配置，如果你想通过数据库配置公众号信息，可以参考我以前写过的一篇文章[jfinal-weixin自定义配置支持多公众号](https://mp.weixin.qq.com/s?__biz=MzU5NjA3MjQ5MA==&mid=2247484188&idx=1&sn=4d84845ca7dacf1aaff0076e19c2e583&chksm=fe690259c91e8b4f36f6aef4ad368349e4992fb81f2e3568bfa9dabd82427aaafa3e3f7c4113&token=894770771&lang=zh_CN&scene=21#wechat_redirect)。

```yaml
dream:
  weixin:
    wx-configs:
    - appId: xxxxxx
      appSecret: xxxxxx
      token: javatrip
      encodingAesKey: xxxxxx
```



`appId`和`appSecret`可在公众号后台进行查看，具体位置在菜单**开发—>基本配置**中，其中`appSecret`要妥善保管，现在公众号已经不支持查看`appSecret`了，如果你忘了`appSecret`，只能进行重置。

### 1.1.3 开发消息校验接口

`mica-weixin`已经为我们提供好了消息校验接口，只需要继承`DreamMsgControllerAdapter`就可以了。

```java
@WxMsgController("/weixin/wx")
public class WeiXinMsgController extends DreamMsgControllerAdapter {
    @Override
    protected void processInFollowEvent(InFollowEvent inFollowEvent) {
    }

    @Override
    protected void processInTextMsg(InTextMsg inTextMsg) {
    }

    @Override
    protected void processInMenuEvent(InMenuEvent inMenuEvent) {
    }
}
```

同时，需要开启缓存，由于`mica-weixin`的将`access_token`等信息放在了缓存中。在启动类上加`@EnableCaching`就开启了。

```java
@SpringBootApplication
@EnableCaching
public class WeixinApplication {
    public static void main(String[] args) {
        SpringApplication.run(WeixinApplication.class, args);
    }
}
```

### 1.1.4 公众号后台配置服务器信息

使用内网穿透工具穿透内网地址，然后在公众号后台菜单**开发—>基本配置**中填写服务器配置信息。

![img](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVcvkeVVdmicj2ln1l7o4libDictQlYvzyzGOtuTy465hjEDxCiaS9sJWw7hiaYYXia5qumrswmnh6ZBNDiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

填写完成后点击启用，这样就完成了微信服务器和业务服务器的关系配置。开启开发者配置后，自动回复、自定义菜单等功能都不能正常使用了。这时候就需要去调用对应的接口实现这些功能。

![img](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVcvkeVVdmicj2ln1l7o4libDicRhukC8PUpymf0pqOOX2kvS367dL1F4UnMdQPcO4MibYg6x9rFRWSUzg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# **二 实现各种消息接口**

## 2.1 关注消息

在一步中，自定义类`WeiXinMsgController`中需要重写三个父类中的方法，其中`processInFollowEvent()`就是关注和取消关注的方法，取消关注后用户虽然不能收到消息，但是后台可以接收到用户取消关注的事件。

```java
@Override
protected void processInFollowEvent(InFollowEvent inFollowEvent) {

    OutTextMsg defaultMsg = new OutTextMsg(inFollowEvent);
    // 关注
    if(InFollowEvent.EVENT_INFOLLOW_SUBSCRIBE.equals(inFollowEvent.getEvent())){
        // 可将关注用户录入db，此处可以获取到用户openid
        String openId = inFollowEvent.getFromUserName();
        // 查询db，根据响应消息类型封装消息体
        if("文本消息"){
            OutTextMsg otm = new OutTextMsg(inFollowEvent);
            otm.setContent("消息内容");
            render(otm);
            return;
        }else if("图片消息"){
            OutImageMsg oim = new OutImageMsg(inFollowEvent);
            // 这里需要调用微信提供的素材接口，将图片上传至素材库。
            oim.setMediaId("图片素材id");
            render(oim);
            return;
        }else if("图文消息"){
            OutNewsMsg onm = new OutNewsMsg(inFollowEvent);
            onm.addNews("标题","简介","图片地址","图文链接");
            render(onm);
            return;
        }else if("视频消息"){
            OutVideoMsg ovm = new OutVideoMsg(inFollowEvent);
            ovm.setTitle("标题");
            ovm.setDescription("简介");
            ovm.setMediaId("视频素材id");
            render(ovm);
            return;
        }else{
            defaultMsg.setContent("感谢关注");
        }
    }
    // 取消关注
    if(InFollowEvent.EVENT_INFOLLOW_UNSUBSCRIBE.equals(inFollowEvent.getEvent())){
        log.info("用户取消关注了");
        // 此处可以将取消关注的用户更新db
    }
}
```

## 2.2 关键词消息

响应内容跟关注消息一样，查询db去匹配关键词，然会根据消息内容封装对应的消息体进行返回，如果没匹配到关键词则回复统一的消息内容。`processInTextMsg()`方法就是用来回复关键词消息的。

```java
@Override
protected void processInTextMsg(InTextMsg inTextMsg) {

    String content = inTextMsg.getContent();
    // 根据用户发送的content去查询db中的响应内容
    if("文本消息"){
        OutTextMsg otm = new OutTextMsg(inTextMsg);
        otm.setContent("消息内容");
        render(otm);
        return;
    }else if("图片消息"){
        OutImageMsg oim = new OutImageMsg(inTextMsg);
        // 这里需要调用微信提供的素材接口，将图片上传至素材库。
        oim.setMediaId("图片素材id");
        render(oim);
        return;
    }else if("图文消息"){
        OutNewsMsg onm = new OutNewsMsg(inTextMsg);
        onm.addNews("标题","简介","图片地址","图文链接");
        render(onm);
        return;
    }else if("视频消息"){
        OutVideoMsg ovm = new OutVideoMsg(inTextMsg);
        ovm.setTitle("标题");
        ovm.setDescription("简介");
        ovm.setMediaId("视频素材id");
        render(ovm);
        return;
    }else{
        OutTextMsg otm = new OutTextMsg(inTextMsg);
        otm.setContent("暂未查到关键词...");
    }
}
```

## 2.3 菜单消息

点击菜单后也是一样，通过`processInMenuEvent()`方法进行响应内容的回复。

```java
@Override
protected void processInMenuEvent(InMenuEvent inMenuEvent) {
    String eventKey = inMenuEvent.getEventKey();
    // 根据用户发送的content去查询db中的响应内容
    if("文本消息"){
        OutTextMsg otm = new OutTextMsg(inMenuEvent);
        otm.setContent("消息内容");
        render(otm);
        return;
    }else if("图片消息"){
        OutImageMsg oim = new OutImageMsg(inMenuEvent);
        // 这里需要调用微信提供的素材接口，将图片上传至素材库。
        oim.setMediaId("图片素材id");
        render(oim);
        return;
    }else if("图文消息"){
        OutNewsMsg onm = new OutNewsMsg(inMenuEvent);
        onm.addNews("标题","简介","图片地址","图文链接");
        render(onm);
        return;
    }else if("视频消息"){
        OutVideoMsg ovm = new OutVideoMsg(inMenuEvent);
        ovm.setTitle("标题");
        ovm.setDescription("简介");
        ovm.setMediaId("视频素材id");
        render(ovm);
        return;
    }else{
        OutTextMsg otm = new OutTextMsg(inMenuEvent);
        otm.setContent("无效链接，请重试...");
    }
}
```

# 三 接口API调用

目前，微信提供的接口对订阅号的限制比较大，未认证的订阅号基本上只有接收消息的几个功能接口。

调用接口的时候需要传递`token`，获取token需要在微信后台中配置业务服务器的白名单。如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/lgiaG5BicLkVcvkeVVdmicj2ln1l7o4libDic3JBMqFUzUquwYoTU6czfibGNHnHFcSROgwA3A0FrgaAxca2Tn2GY5JA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果需要配置多个白名单ip，使用回车键将多个ip分隔开。

`mica-weixin`提供了所有的接口封装，具体可参考它的官方文档，如果要获取微信菜单，可以这样写：

```java
@WxApi("weixin/api")
public class WeiXinApiController {
    @GetMapping("menu")
    @ResponseBody
    public String getMenu(){
        ApiResult menu = MenuApi.getMenu();
        return menu.getJson();
    }
}
```

`@WxApi`这个是它的自定义注解，其实就是包含了`@RequestMapping`和`@Controller`。

# **四 其他事项**

## 4.1 多公众号配置

`mica-weixin`提供了多公众号配置的功能，使用`ThreadLocal`和`appid`进行绑定。只需要简单配置即可实现多公众号配置。

```yaml
dream:
  weixin:
    wx-configs:
      - appId: xxxxxx
        appSecret: xxxxxx
        token: javatrip
        encodingAesKey: xxxxxx
      - appId: xxxxxx
        appSecret: xxxxxx
        token: javatrip
        encodingAesKey: xxxxxx
```

## 4.2 redis配置

`access_token`的有效期是2小时，并且该接口有调用次数限制，`mica-weixin`将`access_token`存储在redis中，避免每次调用接口都去获取`access-token`，因此项目需要配置redis。

```
spring:
  redis:
    host: localhost
    port: 6379
```

## 4.3 手动选择ThreadLocal

如果想要开发微信公众号的后台管理功能，多公众号的时候就需要手动去指定当前线程使用哪个公众号信息。如下：

```
ApiConfigKit.setThreadLocalAppId(appid);
```

至此，SpringBoot开发微信公众号就算完成了，由于订阅号开放的接口太少了，好多功能不能正常演示。还有`mica-weixin`也许不是最好的选择，如果想试着开发微信公众号，可以在github上找一下开发包。至于我为什么会使用`mica-weixin`，是因为我曾用过一段时间的`jfinal`框架，与之配套的微信开发包就是`jfinal-weixin`，也就是jfinal版的`mica-weixin`。