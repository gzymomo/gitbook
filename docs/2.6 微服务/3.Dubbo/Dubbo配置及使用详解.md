- [Dubbo配置及使用详解](https://blog.51cto.com/u_15083739/2922473)



## **配置原则**

在服务提供者配置访问参数。因为服务提供者更了解服务的各种参数。

## **关闭检查**

dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认  check=true。通过 check="false"关闭检查， 比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

例 1：关闭某个服务的启动时检查

> <dubbo:reference interface="com.foo.BarService" check="false" />

例 2：关闭注册中心启动时检查

> <dubbo:registry check="false" />

默认启动服务时检查注册中心存在并已运行。注册中心不启动会报错。

分享给大家视频讲解教程，可直接点击观看，视频内容涵盖：

- 从基础开始手把手详细讲解了RPC概念，PRC在分布式应用的重要作用。
- Dubbo分布式服务框架的应用入门基础。
- 传统应用到分布式以及微服务的转变思想。
- Dubbo协议的特点。
- Dubbo分布式服务的详细开发流程、Dubbo服务的实施部署，Zookeeper的服务管理等。

> **在线观看：https://www.bilibili.com/video/BV1Sk4y197eD**
>
> **资料下载：http://www.bjpowernode.com/javavideo/129.html**

## **重试次数**

消费者访问提供者，如果访问失败，则切换重试访问其它服务器，但重试会带来更长延迟。访问时间变长，用户的体验较差。多次重新访问服务器有可能访问成功。可通过 retries="2" 来设置重试次数(不含第一次)。

![深入解析Dubbo如何配置及使用](https://p1-tt.byteimg.com/origin/pgc-image/c14a761b57ec42a6b6407de933b6ceed?from=pc)

## **超时时间**

由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间。

timeout：调用远程服务超时时间(毫秒)

**dubbo 消费端**

![深入解析Dubbo如何配置及使用](https://p1-tt.byteimg.com/origin/pgc-image/f19da7c7ec72472caf7db0c5dba88090?from=pc)

**dubbo 服务端**

![深入解析Dubbo如何配置及使用](https://p6-tt.byteimg.com/origin/pgc-image/51d0479af56b40049250c1bd84fa4edb?from=pc)

## **版本号**

每个接口都应定义版本号，为后续不兼容升级提供可能。当一个接口有不同的实现，项目早期使用的一个实现类， 之后创建接口的新的实现类。区分不同的接口实现使用 version。

复制 zk-node-shop-userservice 为*zk-node-shop-multi-userservice*

复制 UserServiceImpl.java

![深入解析Dubbo如何配置及使用](https://p1-tt.byteimg.com/origin/pgc-image/30c39090626f444b9f5953bc1581d039.png?from=pc)

UserServiceImpl2 中的地址信息都加入 2 的内容，用来区别原始的数据。

dubbo 配置文件 userservice-provider.xml

增加版本 version 标志

![深入解析Dubbo如何配置及使用](https://p3-tt.byteimg.com/origin/pgc-image/4cc7bc32046247a4950f1ec75b6189e1?from=pc)

复制 zk-node-shop-web 项目为 zk-node-shop-multi-web

（3） ShopService 接口

![深入解析Dubbo如何配置及使用](https://p6-tt.byteimg.com/origin/pgc-image/b79e98e1198d4a7b8707ea61fb1a75be?from=pc)

（4） ShopServiceImpl 接口实现类

![深入解析Dubbo如何配置及使用](https://p6-tt.byteimg.com/origin/pgc-image/dbbbb64380a344a3a9f259e367fbae4a?from=pc)

（5） ShopController 类中添加方法

![深入解析Dubbo如何配置及使用](https://p6-tt.byteimg.com/origin/pgc-image/6cb7a580a09b45cda7cfa3f2b5aafb1c?from=pc)

（6） 修改消费者配置文件

![深入解析Dubbo如何配置及使用](https://p3-tt.byteimg.com/origin/pgc-image/dad1150173df41979886855c7b55a2ca?from=pc)

## **测试应用**

1.先启动 zookeeper

2.启动 tomcat

3.访问服务

比较订单中的地址 ，查看用户信息的地址是不同的内容