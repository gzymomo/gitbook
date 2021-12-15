- [基于Nginx实现灰度发布与AB测试](https://www.cnblogs.com/wzh2010/)

# 背景

单位的云办公相关系统没有成熟的平滑发布方案，导致每一次发布都是直接发布，dll文件或配置文件的变更会引起站点的重启。 

云办公系统的常驻用户有10000+，即使短短半分多钟，也会收到一堆投诉。基于此，我们梳理了一套平滑发布的方案。

# 实施方案

1、跟nginx代理服务器约定了一个健康检查的接口

2、通过接口返回的http状态码来让ngx是否分流用户请求（这个我们单位的技术部那边有标准的做法）

3、根据提供的这个服务健康检查的接口：nginx判断只要某个实例的接口返回5xx的状态码，即把该实例下线（nginx不会把流量转发到该实例）

 ![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnjaWjzOgLicdbjM5LEUtaicbME9B84CO8NOybAfOhV5CVq6f4tqkzJGyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#  发布流程

目的主要是为了发布的时候能够平滑发布，所以QA与开发人员在发布得时候按照如下步骤操作：

1、打开系统的nginx列表管理页面：[/publish/ngxconfig]

2、下架某一个实例（假设系统集群有A、B、C个实例），比如A实例

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnL4IqsDwll4YXNLN0P4P96WMYosooAa2BmMxk87vc3KIIicQoDCUqpzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、查看是否下架成功：这个就是我们跟nginx约定的健康检查接口，正常在线状态下是200的statu，切离线后，这个接口返回的是401的statu。

在线情况：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnYENs4fYsT4QGIZSlx0CwHg4QgfbRP9VoUCspW4L7Mf8tHzDMjMMCMQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 

离线情况：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnsb2lJmRLOkeHWuVXT9TEytjAftFFqKSEjXkmUb349Kmecu5ujNXLiaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4、观察监控站点，直至该实例下的Req、Connnectiuon流量都消失

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHngavUSvTBoiamvuGZyjia6852CcEaYbfmBbAqEQK4oZu7E3hKbUaO8xpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

5、在该实例下进行版本发布

6、打开Fidller，host到待发布的实例，然后判断是否发布成功（发布dll、配置文件时，IIS站点会短暂重启）

7、QA同学走查灰度的A实例服务器，保证它正常运行，如此循环，直到所有服务器都发布。

# 进一步AB测试的优化

平滑发布做完之后，确实给我带来很大的便利，不用每次发布都发公告，不重要的或者非功能性的内容发布了就是了。

但是用久了，客户量上去之后，又遇到一个问题，那就是每一次业务大变更，大型发布都是直接发布到生产，这样可能存在风险。设计师设计的功能，用户不一定完全接受，一旦上线新版本，收到一大堆的吐槽，都是用户呀，如果能在小范围人群内进行灰度试用，完成平稳的过度和使用反馈之后，优化后再上到生产会更好一点。

所以这边需要思考和设计一套统一的技术方案，未来无论云办公还是其他的业务系统，都能通过灰度发布在可指定的小范围内先进行体验和功能验证。

基于上面的平滑，我们在Nginx反向代理服务器上动心思，让nginx来帮我们做ABTesting的方案。

以下是我们尝试的几种方案：

## 1、Nginx反向代理：来路IP策略 

流程图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnyc1Wd4lkkX1nnSkcWPawP5dpj2CcnaGym4CSH6WKQPZHDPwOV3x2sg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 步骤： 

1、进入云办公系统，进入Nginx反代服务器

2、Nginx读取来路IP的AB名单

3、根据IP AB名单进行流量转发(名单A走特定实例，名单B走云办公原有集群实例)

```yaml
server {
listen 80;
server_name officecloud.com;
access_log officecloud.com/logs main;
ip_list 192.168.254.4,192.168.254.170

set $group default;
if ($remote_addr in iplist) {
set $group ACluster;
}
location / { 
proxy_pass http://$group;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
index index.html index.htm;
}
}
```

#### 优缺点：

1、配置简单，原资源平台的灰度升级就是根据IP名单来划分设计升级的

2、外部计算机很多都是非固定IP，这个适合在公司内网实现，比如只是配置公司内网的IP。

## 2、Nginx反向代理：$.Cookies策略 

#### 流程图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHnZ9ALvibtKd795iaRZW56xmoLJUUVSNAE7bGz9Gvx91IuIBzY2DQsMTFQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 步骤 

1、进入云办公系统，进入Nginx反代服务器

2、Nginx读取Http请求的Cokie的version信息（也可以是别的key）

3、根据Key的版本来进行流量转发(比如Version1.1走特定集群，Version1.0走通用集群实例) 

```
server {
listen 80;
server_name officecloud.com;
access_log officecloud.com/logs main;
ip_list 192.168.254.4,192.168.254.170

set $group default;
if ($http_cookie ~* "version=V1.0"){
set default;
}
if ($http_cookie ~* "version=V1.1"){
set $group ACluster;
}

location / { 
proxy_pass http://$group;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
index index.html index.htm;
}
}
```

#### 优缺点：

1、配置简单，根据Nginx的 $COOKIE_version 属性来判断

2、相对稳定，对需要开放名单的用户，在Cookie头部加入特定的版本即可，应用只要少许的开发量

3、首次访问静态页面可能不会产生cookie

备注：这是团队内认为最好的Nginx代理方案，同理，User-Agent和Header都可以做此种类型的判断，但是Header需要侵入底层HttpRequest去业务添加，不建议。

### 3、AB集群+业务代理方式 

#### 流程图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/fEsWkVrSk56iaRI6pZY4rSxjCcIPFSIHntyyiaIGicG6BkuZrJibN1NF3hAmKRuvaIFoYaR4ZVUaAvP9n2TMyVaiaow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

步骤：

1、进入云办公系统，两种方式进入系统，一种是登录页登录：~/login ，一种是default页面带uckey登录：~/default?usertoken=#usertoken#

2、登录的时候和usertoken传入的时候进去 路由代理模块,进行用户信息校验，根据不同的人员和部门(人员和部门配置归属AB名单)分流到两个不同的AB集群

3、根据转发跳到具体的实例集群域名下（可以配置AB集群拥有不同域名,更容易区分）

#### 优缺点 ：

1、与Nginx剥离，不用依赖公司的通用平台和技术部的实现

2、需要申请AB集群，AB集群拥有不同的域名。

3、如果是前后端分离情况下，需要保证静态站点和服务站点均申请AB集群

4、所有入口需要统一做代理，有一定的开发量

目前手上2个系统已经根据该方案实现了。