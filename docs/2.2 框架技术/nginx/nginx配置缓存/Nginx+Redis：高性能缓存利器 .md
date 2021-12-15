# Nginx+Redis：高性能缓存利器

来源：微信公众号：程序员修炼秘籍

## **一. OpenResty**

OpenResty是一个基于 Nginx与 Lua的高性能 Web平台，其内部集成了大量精良的 Lua库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态Web 应用、Web 服务和动态网关。

接入层缓存技术就是使用OpenResty的技术用Lua语言进行二次开发。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7lh9BLlqg99ibtKIx5A5jmba8C4fHN5f4tpL8sW0dJ250pvvVHf0akITQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **二.Nginx +redis**

下图左边是常用的架构，http请求经过nginx负载均衡转发到tomcat，tomcat再从redis读取数据，整个链路过程是串行的，当tomcat挂掉或者tomcat线程数被消耗完，就无法正常返回数据。

使用OpenResty的lua-resty-redis模块使nginx具备直接访问redis的能力，不占用tomcat线程，Tomcat暂时挂掉仍可正常处理请求，减少响应时长，提高系统并发能力。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## **三.压缩减少带宽**

数据大于1K，nginx压缩再保存到redis:

- 提高redis的读取速度
- 减少带宽的占用

压缩会消耗cpu时间，小于1K的数据不压缩tps更高。

OpenResty并没有提供redis连接池的实现，需要自己用lua实现redis的连接池，在网上已有实现的例子`http://wiki.jikexueyuan.com/project/openresty/redis/out_package.html`，直接参照使用。

Redis的value值用json格式保存`{length:xxx,content:yyy}`,content是压缩后的页面内容，length是content压缩前的大小，length字段是为了在读取redis时，根据length的大小来判断是否要解压缩content的数据。

使用lua-zlib库进行压缩。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7lzwTcrb52qHdaVXoQLODYj4ZTfcaoIkUOtPKTmlzREEjT9dBcYyhJQg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **四. 定时更新**

按下图第1和第2步定时执行，nginx lua定时器定时请求tomcat页面的url，返回的页面html保存在redis。

缓存有效期可设置长些，比如1个小时，可保证1个小时内tomcat挂掉，仍可使用缓存数据返回，缓存的定时更新时间可设置短些，比如1分钟，保证缓存快速更新

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7lPI4k2JVvuYPgXjHQ5X92kp545lfOlsEtXBHaR2oDKxia1o1Aic6ibiancw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **五.请求转发**

浏览器打开页面:

- nginx先从redis获取页面html
- redis不存在数据时，从tomcat获取页面，同时更新redis
- 返回页面HTML给浏览器

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7lIxJiaA0kAPC9bFI3Rz34N4cTgicEWCKtmxcnpcibCPHLU2gvwNjVNqqSw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **六. 单进程定时更新**

Nginx的所有worker进程都可以处理前端请求转发到redis,只有nginx worker 0才运行定时任务定时更新redis,lua脚本中通过`ngx.worker.id()`获取worker进程编号。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7lfp2Q8tF24Sibo67L6oAm37icOP5Usdv4jdeVLibYw40YKPiaXLgvTEws3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **七 . 可配置化**

通过管理后台配置需要缓存的URL,可配置缓存URL、缓存有效期、定时更新时间,比如`modify?url=index&&expire=3600000&&intervaltime=300000&sign=xxxx`,sign的值是管理后台secretkey对`modify?url=index&&expire=3600000&&intervaltime=300000`签名运算得到的，nginx端用相同的secretkey对`modify?url=index&&expire=3600000&&intervaltime=300000`签名运算，得到的值与sign的值相同则鉴权通过,允许修改nginx的配置。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkoUkFxliaJ25LlyAjrFI7l0NHl0lPRoAWU3exwqbxqUtthuFhL1QzXicGlN0ica603jTic6pcnFv4VA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)