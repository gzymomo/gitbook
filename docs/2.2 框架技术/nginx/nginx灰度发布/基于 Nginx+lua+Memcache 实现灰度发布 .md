- [nginx+lua+memcache实现灰度发布](https://www.cnblogs.com/wenbiao/p/3227998.html)



# 一、灰度发布

灰度发布是指在黑与白之间，能够平滑过渡的一种发布方式。AB  test就是一种灰度发布方式，让一部分用户继续用A，一部分用户开始用B，如果用户对B没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到B上面 来。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。

 

这里的用于WEB系统新代码的测试发布，让一部分（IP）用户访问新版本，一部分用户仍然访问正常版本，其原理如图：

 

 ![img](https://images0.cnblogs.com/blog/548340/201307/31154355-226569803ea346d38747baddcdb4dc4c.jpg)

执行过程：

1、   当用户请求到达前端代理服务Nginx，内嵌的lua模块解析Nginx配置文件中的lua脚本代码；

2、   Lua变量获得客户端IP地址，去查询memcached缓存内是否有该键值，如果有返回值执行@client_test，否则执行@client。

3、   Location @client_test把请求转发给部署了new版代码的服务器，location @client把请求转发给部署了normal版代码的服务器，服务器返回结果。整个过程完成。