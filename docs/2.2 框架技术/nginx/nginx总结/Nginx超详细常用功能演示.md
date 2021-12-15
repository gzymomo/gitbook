- 原文地址：Code综艺圈-博客园：[Nginx实战部署常用功能演示(超详细版)](https://www.cnblogs.com/zoe-zyq/p/14843709.html)

# 1. 配置文件解读

Nginx和Redis一样，只需简单的文件配置，就能轻松实现吊炸天的功能，所以先来了解一下配置文件内容，不用太急着知道怎么用，接下来在功能实操的时候还会用到。

**nginx.conf**文件是经常需要配置的，我这里安装完成之后，该配置文件的路径见下图：

![image-20210603214246888](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603214246888.png)

文件主要内容如下：

```
#指定用户，可以不进行设置
#user  nobody;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;   
#错误日志存放目录,可以根据后面的日志级别指定到不同目录
error_log  /var/log/nginx/error.log info;
#进程pid存放位置
pid        /var/run/nginx.pid;

events {
    # 单个后台进程的最大并发数
    worker_connections  1024; 
}

http {
    #文件扩展名与类型映射表，指定为当前目录下的 mime.types
    include       mime.types;
    #默认文件类型
    default_type  application/octet-stream;  
    #设置日志显示格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #nginx访问日志存放位置
    access_log  /var/log/nginx/access.log  main;   

    #开启高效传输模式
    sendfile        on;   
    #tcp_nopush     on;    
    #保持连接的时间，也叫超时时间
    keepalive_timeout  65;  
    #开启gzip压缩
    #gzip  on;  
    #server的配置可以单独为一个子配置文件,避免单个配置文件过大
    server {
        #配置监听端口
        listen       80;  
        #配置域名
        server_name  localhost;  
        #charset koi8-r;     
        #access_log  /var/log/nginx/host.access.log  main;
        location / {
            #指定默认目录
            root   html;
            #默认访问页面
            index  index.html index.htm;    
        }
        # 指定http code 配置404页面
        #error_page  404              /404.html;   

        # redirect server error pages to the static page /50x.html
        #错误状态码的显示页面，配置后需要重启
        error_page   500 502 503 504  /50x.html;   
        location = /50x.html {
            root   html;
        }
   }
} 
```

在上面配置文件中，有几个点需要注意：

- **http配置块中可以配置多个server块**，而每个server块就相当于一个虚拟主机(后续会说到)；
- **在server块中可以同时包含多个location块**。
- 在http配置块中可以使用 include 目录/*.conf; 指定子配置文件的位置，然后自动加载配置内容进来，避免单文件配置过大。

# 2. 常用命令

这里演示没有配置环境变量，所以需要进入nginx的安装目录(/usr/local/nginx/sbin)中进行操作，进入可以执行以下命令：

## 2.1 开启nginx

```
./nginx #启动
```

## 2.2 停止nginx

```
# 方式1
./nginx -s stop # 立即停止
# 方式2
./nginx -s quit # 进程完成当前工作后在停止
# 方式3
killall nginx # 直接杀死进程
```

## 2.3 重新加载配置文件

```
./nginx -s reload
```

## 2.4 查看nginx的启动情况

```
ps aux|grep nginx
```

## 2.5 查看端口号占用情况

```
netstat -tlnp # 查看整体端口占用情况
netstat -tlnp|grep 端口号  # 查看指定端口的占用情况
```

# 3. 常用功能实战

## 3.1 反向代理

经常有小伙伴要用google搜索资料，被无情的拒绝了，所以只能百度；如果非要用google进行搜索咋弄？**翻墙(需要配置相关信息)**，其实本质是**本机电脑借助代理服务器转到对应目标服务器(小伙伴机器和代理服务器在同一个LAN内)**，然后就可以间接获取到信息啦，这种形式就叫**正向代理**。如下图：

![image-20210603214302972](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603214302972.png)

**反向代理**与正向代理刚好相反，**反向代理和目标服务器在同一个LAN内**，小伙伴直接访问反向代理服务器地址，由反向代理将请求转发给目标服务服务器，然后将结果返回给小伙伴。如下图：

![image-20210603215617714](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603215617714.png)

**案例演示：**

新建一个API项目，然后部署到云服务器上，通过nginx进行反向代理，隐藏项目的真实地址，为了运行API项目，这里需要安装.NetCore3.1的运行环境(不是开发就不用安装SDK啦)；

```
#第一步，注册 Microsoft 密钥和存储库。安装必需的依赖项。
rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
#第二步，安装 .NET Core3.1 运行时，不是开发环境，就不需要安装sdk
yum install aspnetcore-runtime-3.1
```

然后执行`dotnet --version` 命令，如果显示对应版本就可以继续部署程序啦；

创建一个TestWebAPI项目，将编译之后的项目文件通过Xftp拷贝到云服务器上，然后将其启动，如下：

![image-20210603215626840](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603215626840.png)

运行之后，由于阿里云云服务器的安全组没有对外开放5000端口，所以外网是访问不了的，但可以在服务器内通过curl命令测试站点是否启动，如下：

![image-20210603215633142](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603215633142.png)

我这个服务器，80端口是对外开放的，可以访问到的，如下：

![image-20210603220139868](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220139868.png)

所以现在我们通过nginx能访问的80端口，反向代理到我们内部开启的测试项目，即5000那个端口。nginx配置如下：

![image-20210603220146984](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220146984.png)

重启nginx之后，就可以访问啦，如下：

![image-20210603220154617](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220154617.png)

**关键知识：**

- 在Server块中指定对外的端口和server_name(域名或IP)；

- 配置对应Server块中的location块；配置location可以进行正则匹配，语法如下：

  ```
  location [ = | ~ | ~* |^~] uri{
  
  } # 匹配的路径
  ```

  **=**：表示uri不包含正则表达式，要求请求字符串与uri严格匹配，只要匹配成功立即处理该请求，不在继续寻求匹配的规则；

  **~**：用于表示uri中包含正则表达式，区分大小写；

  **~***：用于表示uri中包含正则表达式，不区分大小写；

  **^~**：表示uri不包含正则表达式，找到请求字符串与uri匹配度最高的location后，然后立即处理请求。

  例：

  ![image-20210603220202977](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220202977.png)

  实操如下：

  ![image-20210603220208506](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220208506.png)

- 在location中使用**proxy_pass**配置需要转发到的目标服务器地址；

**nginx反向代理好处：**

- 屏蔽目标服务器的真实地址，相对安全性较好；
- nginx的性能好，便于配置负载均衡和动静分离功能，合理利用服务器资源。
- 统一入口，当做负载均衡时，不管目标服务器怎么扩展部署，调用者只访问代理服务器入口即可。

## 3.2 负载均衡

系统的高可用是比较重要的，所以站点会通常以集群的方式进行部署， 但为了让请求均匀分配到各服务器上，则就要用到负载均衡策略啦，不管是软件的方式还是硬件的方式都可以实现(这里就不详细列举啦)，大概模式如下图：

![image-20210603220218191](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220218191.png)

**案例演示**

案例采用一个nginx做为反向代理，并通过简单的配置实现负载均衡功能；由于设备有限，目标服务器采用端口不同的形式进行模拟，端口分别是5000和6000，然后在原来的**项目中增加一个获取端口的接口**，用于便于案例演示，代码如下：

![image-20210603220224248](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220224248.png)

然后将编译完成之后的项目文件通过xFtp拷贝到云服务器上，然后用以不同端口的形式分别在不同终端启动，命令如下：

![image-20210603220234937](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220234937.png)

另外打开一个终端，如上图一样启动项目，只是配置端口为5000打开，这样项目就启动了两个(集群)，接下来就通过配置nginx来实现负载均衡的功能。如下图：

![image-20210603220247172](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220247172.png)

**nginx负载均衡策略**

如上演示，默认情况下，**nginx的负载均衡策略为轮询**，在实际应用场景中可以根据需要配置其他策略，如下：

- **轮询**：默认就是，指每个请求按照请求顺序逐一分配到不同到目标服务器，如果目标服务器有宕机的，还能自动剔除。

- **权重(weight)**：通过配置权重来实现请求分配，目标服务器配置的权重越高，被分配的请求越多。

  ```
  # 其他不变，只是在每个目标服务器后面增加权重即可
  upstream testloadbalance {
        server 127.0.0.1:5000 weight=5;
        server 127.0.0.1:6000 weight=10;
    }
  ```

  按照上面配置重启nginx，多次请求测试，请求会更多的转发到6000上面。

- **ip_hash**：每个请求有对应的ip，通过对ip进行hash计算，根据这个结果就能访问到指定的目标服务器；这种方式可以保证对应客户端固定访问到对应的目标服务器；

  ```
  # 其他不变，只是增加一个策略进行
  upstream testloadbalance {
        ip_hash; # 指定策略为通过ip进行hash之后转发
        server 127.0.0.1:5000;
        server 127.0.0.1:6000;
    }
  ```

- **fair**：按目标服务器的响应时间来分配请求，响应时间短的优先被分配。

  关于这种模式需要额外安装nginx-upstream-fair，然后配置一下策略即可，安装就不具体演示，点击上面连接进入看说明；配置内容如下：

  ```
  # 其他不变，只是增加一个策略进行
  upstream testloadbalance {
        fair; # 指定策略为fair
        server 127.0.0.1:5000;
        server 127.0.0.1:6000;
    }
  ```

负载均衡的功能的配置是不是很简单~~~，动动手感觉就是舒坦。

## 3.3 动静分离

前后端分离开发的模式最近几年是火的不行，在部署方面，为了提高系统性能和用户体验，也会将动静分离部署，即将静态资源(如html、js、css、图片等)单独部署一个站点，关于WebAPI获取和处理信息单独部署为一个站点。本质还是通过location匹配规则，将匹配的请求进行不同的处理即可。

**环境准备**

在nginx安装目录下创建一个static目录，用于存放相关静态资源：

![image-20210603220257328](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220257328.png)

结构如下：

![image-20210603220304134](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220304134.png)

**动静分离配置**

![image-20210603220314564](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220314564.png)

**重启nginx(或重新加载配置文件)，然后访问看效果**：

![image-20210603220626303](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220626303.png)

动静分离思想就是这样直观，小伙伴可以根据自己的需要，定义location的匹配规则即可。

# 4. 其他功能

除了以上常用的功能，可能还有一些小功能也会常用到哦，比如**根据http状态码配置指定页面、访问权限控制、适配PC或移动端等**，以下挑几个平时比较常用的演示一把，如下：

## 根据状态码配置指定页面

就拿平时常见的404举例，默认可能就是简单的页面提示，如下：

![image-20210603220743656](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220743656.png)

但是对于很多企业都喜欢做自己个性化的页面，还有一些用来做公益广告等等；nginx配置很简单，如下：

![image-20210603220752164](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220752164.png)

其他http状态码也可以通过上面的方式进行自定义页面展示。

## 访问权限控制

为了系统安全，会针对请求增加访问权限控制，比如使用黑白名单的方式来进行控制，将访问IP加入到白名单就可以访问，加入到黑名单就不可以访问啦，如下：

![image-20210603220637602](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220637602.png)

上图是拒绝指定IP，如果是允许指定IP，可进行如下配置，如下：

```
location /weatherforecast/ {
 proxy_pass http://testloadbalance;
 # 这个ip是百度输入ip查看到的,也可以通过nginx日志可以看
 allow 223.88.45.26; 
}
```

注：**如果在同一location块中同时配置deny和allow，配置在最前面的会覆盖下面的**，如下：

```
location /weatherforecast/ {
  proxy_pass http://testloadbalance;
  # deny all 放在前面，就所有不能访问，deny all 会覆盖下面配置
  #deny all; 
  allow  223.88.45.26;
  # deny all 放在后面，被上面allow进行覆盖
  deny all; 
}
```

## 适配PC或移动端

现在的移动端好多都是采用H5的形式进行开发，或者是混合模式，所以也需要针对移动端部署对应的站点，那用nginx如何自动适配PC还是移动端页面呢？

**准备环境**

在nginx安装目录中创建pcandmobile目录，如下：

![image-20210603220647230](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220647230.png)

目录里面内容如下：

![image-20210603220652544](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220652544.png)

两个index.html中的就只有一个h1标签，分别显示“PC端页面”和“移动端页面” 文字。

**nginx配置**

```
location / {
root pcandmobile/pc; # 默认在pc目录中找页面
# 当请求头中User-Agent中匹配如下内容时，就去mobile目录找页面
if ($http_user_agent ~* '(Android|webOS|iPhone|iPod|BlackBerry)') {
    root pcandmobile/mobile;
}
index index.html;
}
```

运行效果如下：

![image-20210603220700947](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220700947.png)

**本质就是判断请求头中User-Agent，只要匹配到移动端，就去找指定移动页面就行啦。**

# 5.跨域

**跨域是因为浏览器同源策略的保护，不能直接执行或请求其他站点的脚本和数据**；一般我们认为的同源就是指**协议、域名、端口都相同**，否则就不是同源。

现在前后端分离开发已经很普遍了，跨域问题肯定少不了，但解决的方式也很多，比如JsonP、后端添加相关请求头等；很多时候，不想改动代码，如果用到nginx做代理服务器，那就可以轻松配置解决跨域问题。

## 5.1 环境准备

需要准备两个项目，一个前端项目发布在80端口上，一个API项目发布在5000端口上，这里需要在阿里云的安全组中将这两个端口开放；

- **API项目环境(对外是5000端口)**

  API接口还是使用上次演示的项目，很简单，过程我就不再重复上图啦，直接来两张重要的；

  ![image-20210603220711781](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210603220711781.png)

  配置nginx反向代理，然后运行看效果：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa337f2d32a84a779c0cae01f0bed7bc~tplv-k3u1fbpfcp-zoom-1.image)

- **前端环境(对外是80端口)**

  前端页面(kuayu.html)

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>跨域测试</title>
  	<script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
  	<script type="text/javascript">
  		$(document).ready(function(){
  		  // 点击按钮 请求数据，
  		  $("#b01").click(function(){
  		  		// 请求数据
  		  		htmlobj=$.ajax({url:"http://47.113.204.41:5000/weatherforecast/getport",async:false});
  		  		// 将请求的数据显示在div中
  		  		$("#myDiv").html(htmlobj.responseText);
  		  });
  		});
  	</script>
  </head>
  <body>
  	<h2>获取结果</h2>
  	<div id="myDiv">结果显示</div>
  	<button id="b01" type="button">GetPort</button>
  </body>
  </html>
  ```

  将kuayu.html通过xFtp拷贝到服务器上，对应的static目录是自己创建的，如下图：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eb87e09b7c54681af1c3b842327f535~tplv-k3u1fbpfcp-zoom-1.image)

- **nginx配置及运行测试**

  配置nginx，在原有配置文件中再新增一个server块，如下配置：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b005af6b33b44d1b4a74ea9fc97ebd0~tplv-k3u1fbpfcp-zoom-1.image)

  运行测试：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93a7da19ed414a868d5bd6002764b908~tplv-k3u1fbpfcp-zoom-1.image)

  跨域问题出现了，现在前后端都不想改代码，要干架吗？nginx说：和谐，一定要和谐~~~

## 5.2 配置跨域及运行

在API的server中进行跨域配置，如下：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cca72fe959274919a17d6671778d010d~tplv-k3u1fbpfcp-zoom-1.image)

重启nginx之后，再测试，看看搞定了没？

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9bb84d820bc4799a39957726504b30e~tplv-k3u1fbpfcp-zoom-1.image)

# 6. 配置SSL证书

现在的站点几乎都是https了，所以这个功能必须要实操一把；为了更符合真实线上场景，我特意准备了域名和证书，真真实实在阿里云服务器上演示； 这里需要登录到阿里云上购买域名，然后根据域名申请免费的SSL证书，最后进行配置使用，详情如下：

## 6.1 准备域名

这里我注册了一个域名为：**codezyq.cn**；下面先说说注册域名的流程：

- 登录阿里云，找到域名注册入口

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04d68166d1664473a1055bdbe86fd30f~tplv-k3u1fbpfcp-zoom-1.image)

- 进入到一个页面，然后输入需要注册的域名

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/742db4d4ef534cca8400dcd95a03dcdf~tplv-k3u1fbpfcp-zoom-1.image)

- 然后就出现搜索结果，如果被注册就会提示，可以选择其他类型或更换域名

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/669093ab82d0421293b5e09fcb98537d~tplv-k3u1fbpfcp-zoom-1.image)

- 买了域名之后，需要进行实名认证，个人上传身份证就完事啦，一会就实名完成；  完成之后就需要配置域名解析，即解析到自己的云服务器上，后续通过域名才可以访问；

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c09c3c8d2e742658d3069d06046faf4~tplv-k3u1fbpfcp-zoom-1.image)

- 测试是否能解析成功，直接在自己电脑上ping一下域名，看看是否解析到指定IP即可，简单直接；这样域名就可以用啦~~~

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/519271d14d9d4ef79a8f08aad4c97577~tplv-k3u1fbpfcp-zoom-1.image)

## 6.2 准备证书

免费证书这块的申请需要用到买的域名，大概步骤如下：

- 领取免费证书数量(20个)

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5fbb79cf20a4fb0a88f9600dca4402e~tplv-k3u1fbpfcp-zoom-1.image)

- 进入SSL证书(应用安全页面)进行证书创建

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53a55829a2ef408eb44d5b492780d832~tplv-k3u1fbpfcp-zoom-1.image)

- 进行证书申请，即绑定域名

  填写申请信息，如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/812d6c94c99a480190c7ab38c9cb648d~tplv-k3u1fbpfcp-zoom-1.image)

  这里一般填完申请信息，后续验证那块直接过也能签发，列表状态如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd28a942ed64b0b9a9d9eba83235675~tplv-k3u1fbpfcp-zoom-1.image)

- 签发完成之后，下载对应服务器的证书即可，点击下载选择

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b90678335bb4401185bb3201b8850fef~tplv-k3u1fbpfcp-zoom-1.image)

  直接下载下来一会配置使用。

## 6.3 nginx配置证书

- **检查环境**

  **检查端口是否可访问**

  https需要443端口，所以在阿里云中将443端口加入到安全组中，如下图：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cae340d32394bdca0798bca9d61ae28~tplv-k3u1fbpfcp-zoom-1.image)

  另外还需要查看云服务器的防火墙有没有开放对应的端口，如下命令：

  ```bash
  # 查看防火墙端口开放情况
  firewall-cmd --list-all
  public
    target: default
    icmp-block-inversion: no
    interfaces: 
    sources: 
    services: dhcpv6-client ssh
    # 显示的结果没有开放443端口
    ports: 80/tcp 22/tcp 5000/tcp 3306/tcp 6379/tcp  
  
  # 将443端口加入到里面
  firewall-cmd --zone=public --add-port=443/tcp --permanent
  # 重新加载
  firewall-cmd --reload
  # 再看防火墙端口开放情况
  firewall-cmd --list-all
  public
    target: default
    icmp-block-inversion: no
    interfaces: 
    sources: 
    services: dhcpv6-client ssh
    # 443端口加入进来了
    ports: 80/tcp 22/tcp 5000/tcp 3306/tcp 6379/tcp 8080/tcp 443/tcp 
  ```

  **检查nginx中是否包含http_ssl_module模块**

  在配置证书之前需要检查一下nginx中是否已经包含http_ssl_module：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bdc293952154968b23c2df0c90d891f~tplv-k3u1fbpfcp-zoom-1.image)

  如果没有就算配置了也不能用，如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aa5ca36210b46c8977e2d3594df9789~tplv-k3u1fbpfcp-zoom-1.image)

- **为nginx加上http_ssl_module**

  需要下载一个源码进行引入，这里还是使用版本1.18.0，具体步骤如下：

  第一步，先准备环境，比如支持编译、openssl等，执行以下命令：

  ```bash
  # 安装对应的包
  yum -y install gcc openssl openssl-devel pcre-devel zlib zlib-devel
  ```

  第二步，下载对应版本源码到usr/local/src中，并进行解压，执行以下命令

  ```bash
  # 下载指定版本nginx包
  wget http://nginx.org/download/nginx-1.18.0.tar.gz
  # 解压
  tar -zxvf nginx-1.18.0.tar.gz
  ```

  第三步，进入解压目录中，配置http_ssl_module，命令如下：

  ```bash
  # 进入解压目录
  cd nginx-1.18.0
  # 加入http_ssl_module
  ./configure --prefix=/usr/local/nginx --with-http_ssl_module
  ```

  第四步，编译，在解压目录中直接执行make命令即可

  ```bash
  # 执行make命令，在当前目录就会添加新目录objs
  make
  # 如果是新安装nginx，执行以下命令
  # make&make install
  ```

  如果没报错，就编译出最新的啦； 这里我在实操的时候，先执行的./configure  配置命令，然后再执行第一步的命令，所以导致make的时候老是不成功，这里解决方案是在添加http_ssl_module模块时，同时指定了openssl源码路径(直接下载即可)，然后再执行make命令就成功了。命令如下：

  ```bash
  # 指定openssl源码路径 需要下载
  ./configure --prefix=/usr/local/nginx --with-openssl=/usr/src/openssl-1.1.1d --with-http_ssl_module
  # 再执行make命令编译
  make
  ```

  第五步，将编译出来新的nginx文件替换原有的nginx文件，操作如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a99353b1196a446ca50a5c505cfdea14~tplv-k3u1fbpfcp-zoom-1.image)

- 在nginx配置SSL的支持

  还记得获取下载证书的时候吗，下载界面那有一个帮助操作，点击就有nginx配置SSL证书的详细步骤，如下图：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de1c964483f548ff8282ca4f8b1ff482~tplv-k3u1fbpfcp-zoom-1.image)

  这里演示配置的内容如下(新增一个server块，专门配置SSL的)：

  ```bash
  server {
  	   # https 监听的是443端口
         listen       443 ssl;
         # 指定准备好的域名
         server_name  codezyq.cn;
  	   # 指定证书路径，这里需要把准备好的证书放到此目录
         ssl_certificate      /usr/local/nginx/myssl/codezyq.cn.pem;
         ssl_certificate_key  /usr/local/nginx/myssl/codezyq.cn.key;
         ssl_session_cache    shared:SSL:1m;
         # 超时时间
         ssl_session_timeout 5m;
         # 表示使用的加密套件的类型
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
         # 表示使用的TLS协议的类型
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
         ssl_prefer_server_ciphers on;
         location /www/ {
             root   static;
             index  index.html index.htm;
         }
     }
  ```

- 重启nginx，然后进行https访问对应的连接就可以啦

  **注：网站如果没有备案，会被拦截，会导致不能访问，正规站点都是要备案的**。

# 7.  防盗链配置

**盗链通俗一点的理解就是别人的网站使用自己服务器的资源**，比如常见的在一个网站引入其他站点服务器的图片等资源，这样被盗链的服务器带宽等资源就会额外被消耗，在一定程度上会损害被盗链方的利益，所以防盗链是必要的；这里通过nginx简单实现静态资源防盗链的案例，原理很简单，就是判断一下请求源是否被允许。

## 7.1 准备一个html和图片

将准备的html和图片放在创建的static目录下，如下图：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05a41399e30348ffa4e1a7d71e182c45~tplv-k3u1fbpfcp-zoom-1.image)

**anti-stealing-link.html内容如下**

```html
<!DOCTYPE html>
  <html>
  <head>
  	<title>anti-stealing-link  test</title>
  </head>
  <body>
  	<h2>使用云服务器上的图片</h2>
      <!-- 访问服务器图片 -->
  	<img src="http://47.113.204.41/img/test.jpg" alt="">
  </body>
</html>
```

## 7.2 正常配置nginx

```bash
server {
        listen       80;
        server_name 47.113.204.41;
		# 针对html访问的匹配规则
        location /www/ {
            root static;
            index index.html index.htm;
        }
		# 针对图片访问的匹配规则
        location /img/ {
            root static;
        }
        charset utf-8;
    }
```

运行起来看效果：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90a1856cea8f453a85cd205b087335f9~tplv-k3u1fbpfcp-zoom-1.image)

现在有个需求，只能是云服务器的html才能使用图片，其他引用源都认为是盗链，直接返回403或重写到固定图片。

## 7.3 轻松配置nginx防盗链

针对img配置如下：

```bash
server {
        listen       80;
        server_name 47.113.204.41;
		# 针对html访问的匹配规则
        location /www/ {
            root static;
            index index.html index.htm;
        }
		# 针对图片访问的匹配规则
        location /img/ {
            root static;
            #对源站点的验证，验证IP是否是47.113.204.41
            #可以输入域名，多个空格隔开
            valid_referers 47.113.204.41;
            #非法引入会进入下方判断
            if ($invalid_referer) {
               #这里返回403，也可以rewrite到其他图片
               return 403;
            }
        }
        charset utf-8;
    }
```

重启nginx,清除缓存再试：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9134717a6404296bc07a5571b2a0767~tplv-k3u1fbpfcp-zoom-1.image)

防盗链是不是很简单，也可以通过代码的形式，比如在过滤器或管道中也可以实现，如果没有特殊需求，nginx稍微一配置就能实现，岂不美哉~~~

# 8. 隐藏版本信息

之前在项目中做渗透测试时，其中有一项问题就是不希望在响应头中体现服务器相关版本，避免在某些版本出现漏洞时，攻击者可以特意针对此版本进行恶意攻击，从而影响系统服务可用性。

现有情况：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c3cb20d55c84be69d21bbd4aa1a7ae1~tplv-k3u1fbpfcp-zoom-1.image)

页面找不到时：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73c94cf0109541fb95dd5d9a5325bed5~tplv-k3u1fbpfcp-zoom-1.image)

看看nginx是如何关闭，如下配置：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/999aa22f9c424e97b1ff42542a78a76b~tplv-k3u1fbpfcp-zoom-1.image)

看效果：

正常访问

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65854b341da8486093175330da57c3fd~tplv-k3u1fbpfcp-zoom-1.image)

找不到报错，版本也没有啦

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f878f1f5950447aaba29f637e694c2a~tplv-k3u1fbpfcp-zoom-1.image)

# 9. 高可用配置

尽管nginx性能再强，但服务器和网络有很多因素是不可控的，如：网络抖动、网络不通、服务器宕机等情况，都会导致服务不可用(可理解为单点故障)，这样的系统体验肯定得不到好评；如果**当一个服务器宕机或服务挂掉时，另外一台服务器自动代替宕机服务器提供服务，保证系统的正常运行**，这就实现了高可用；这里nginx实现高可用的方式和Redis的主从模式很相似，只是nginx使用的是keepalived来实现高可用集群。

## 8.1 keepalived简介

**keepalived**实现高可用的关键思路还是主、备节点的来回切换；

- 首先需要**配置一个VIP(虚拟IP)**，用于提供访问的接口地址，刚开始是在主节点上的；
- 当主节点发生故障时，VIP就漂移到备节点，由备节点提供服务；
- 如果主节点恢复，会通知备节点健康状态，VIP就会漂移到主节点；

由上可见，在keepalive实现高可用时，肯定要有机制选择主备节点，主备之间肯定要互相通知，不然咋知道节点之间的健康状态，所以就使用了VRRP协议，目的就是为了解决静态路由的单点故障。

**VRRP协议，全称Virtual Router Redundancy Protocol(虚拟路由冗余协议)，利用IP多播的方式实现通信**；通过**竞选协议机制**(根据配置的优先级来竞选)来将路由任务交给某台VRRP路由器，保证服务的连续性；

理论先了解这么多，先来把keepalived安装上，

**方式一**

执行以下命令可直接安装：

```bash
# 安装，这种直接安装完成了，修改配置文件启动即可
yum install-y keepalived
# 启动
systemctl start keepalived.service
```

这种方式可能会**遇到启动keepalived的时候报错，原因可能是服务配置文件(/usr/lib/systemd/system/keepalived.service)指定的keepalived相关目录找不到**；  如果文件目录都正常，还报错，我折腾了好久，后来用源码方式进行安装就正常啦~~~

**方式二**

建议使用源码的形式进行安装，大概步骤如下：

第一步，环境准备

```bash
# 安装对应的依赖包
yum -y install gcc openssl openssl-devel pcre-devel zlib zlib-devel
```

第二步，下载并解压源码包

```bash
# 在/usr/local/src目录下执行，下载到该目录
wget https://www.keepalived.org/software/keepalived-2.0.18.tar.gz
# 解压
tar -zxvf keepalived-2.0.18.tar.gz
```

第三步，安装

```bash
# 进入到解压出来的目录
cd keepalived-2.0.18
# 编译并安装
./configure && make && make install
```

第四步，创建启动文件，即将编译出来的文件拷贝到对应的目录下

```bash
cp  -a /usr/local/etc/keepalived   /etc/init.d/
cp  -a /usr/local/etc/sysconfig/keepalived    /etc/sysconfig/
cp  -a /usr/local/sbin/keepalived    /usr/sbin/
```

第五步，创建配置文件

```bash
# 先创建配置文件存放的目录
mkdir /etc/keepalived
# 再将创建好的配置文件通过xFtp传到此目录，也可以直接在这里创建
```

配置文件名为**keepalived.conf**，内容如下：

```bash
! Configuration File for keepalived
global_defs {
	   # 每台机器的唯一标识
       router_id 31
}
# vrrp实例，可以配置多个
vrrp_instance VI_1 {
	# 标识为主节点，如果是被节点，就为BACKUP
    state MASTER
    # 网卡名称，通过ip addr 命令可以看到对应网卡，需要哪个就配置哪个就行
    interface enp0s8
    # 路由ID，主、备节点的id要一样
    virtual_router_id 3
    # 优先级，主节点的优先级要大于备节点的优先级
    priority 200
    # 主备心跳检测，间隔时间为1s
    advert_int 1 
    # 认证方式，主备节点要一致
    authentication {
       auth_type PASS
       auth_pass 123456
    }
    virtual_ipaddress {
       # 虚拟IP
       192.168.30.108
    }
}
```

第六步，启动

```bash
# 启动
systemctl start keepalived.service
# 查看虚拟IP情况
ip addr 
```

查看效果如下，虚拟ip正常在master节点上：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de84ec6c60c943a6ad20a5c912dbb993~tplv-k3u1fbpfcp-zoom-1.image)

安装完keepalived和nginx之后就可以进行主备演示啦~~~

## 8.2 主备演示

这里用了两台虚拟机，结构如下：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0147f5d9126d42d09e234ed8561f4ebc~tplv-k3u1fbpfcp-zoom-1.image)

- 第一步nginx配置准备

  这里用到的就是nginx默认配置，基本没咋改，如果要了解配置文件详情，点击([Nginx超详细常用功能演示，够用啦~~~](http://mp.weixin.qq.com/s?__biz=MzU1MzYwMjQ5MQ==&mid=2247485151&idx=1&sn=9859ab32f3343346675e2394a48962fd&chksm=fbf11a0bcc86931d02f7f4a2100a7be1a3c3b21e0edbf1d153a8d405fd858e4fe37c6bd72c7d#rd))有详细说明。两台虚拟机配置的nginx.conf内容如下：

  ```bash
  #user  nobody;
  worker_processes  1;
  events {
      worker_connections  1024;
  }
  http {
      include       mime.types;
      default_type  application/octet-stream;
      sendfile        on;
      keepalive_timeout  65;
      #gzip  on;
      server {
      	# 需要防火墙开放80端口
          listen       80;
          server_name  localhost;
          location / {
              root   html;
              index  index.html index.htm;
          }
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   html;
          }
      }
  }
  ```

- 第二步准备html

  两台虚拟机中html，用的是nginx默认的index.html(路径：/usr/local/nginx/html)，为了方便演示，在index.html中增加了个**105和106的显示标注**，105机器内容如下，106机器上的html只是把内容中的105改成106即可：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab0967f8f4564352a49b28b8d67b5427~tplv-k3u1fbpfcp-zoom-1.image)

- 第三步准备两台虚拟机上keepalived配置文件(路径：/etc/keepalived)

  **105机器上keepalived.conf内容如下**：

  ```bash
  ! Configuration File for keepalived
  global_defs {
  	# 每台机器不一样
      router_id 31
  }
  #检测nginx服务是否在运行
  vrrp_script chk_nginx {
     #使用脚本检测
     script "/usr/local/src/chk_nginx.sh"
     #脚本执行间隔，每2s检测一次
     interval 2
     #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
     weight -5
     #连续2次检测失败才确定是真失败
     fall 2
     #检测到1次成功就算成功
     rise 1                    
  }
  
  vrrp_instance VI_1 {
  	# vrrp实例，可以配置多个
      state MASTER
      # 网卡名称，通过ip addr 命令可以看到对应网卡，需要哪个就配置哪个就行
      interface enp0s8
      # 路由ID，主、备节点的id要一样
      virtual_router_id 3
      # 优先级，主节点的优先级要大于备节点的优先级
      priority 200
      # 主备心跳检测，间隔时间为1s
      advert_int 1
      # 认证方式，主备节点要一致
      authentication {
            auth_type PASS
            auth_pass 123456
      }
      #执行监控的服务。
      track_script { 
      	#引用VRRP脚本，即在 vrrp_script 部分指定的名字。
          chk_nginx                    
       }
      virtual_ipaddress {
      	# 虚拟IP
          192.168.30.108
      }
  }
  ```

  在/usr/local/src/中准备chk_nginx.sh，keepalived和nginx没有直接关系的，只有通过检查nginx的运行状态来进行高可用服务切换，chk_nginx.sh内容如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f4941c761044029b754b911743a6365~tplv-k3u1fbpfcp-zoom-1.image)

  然后给这个文件增加执行权限，命令如下：

  ```bash
  chmod +x chk_nginx.sh
  ```

  106虚拟机上的步骤和检测脚本chk_nginx.sh都一样，只是keepalived.conf内容稍微有点变动。

  **106机器上keepalived.conf内容如下**：

  ```bash
  ! Configuration File for keepalived
  global_defs {
      # 每台机器上不一样
      router_id 32
  }
  #检测nginx服务是否在运行
  vrrp_script chk_nginx {
     #使用脚本检测
     script "/usr/local/src/chk_nginx.sh"
     #脚本执行间隔，每2s检测一次
     interval 2
     #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
     weight -5
     #连续2次检测失败才确定是真失败
     fall 2
     #检测到1次成功就算成功
     rise 1                    
  } 
  vrrp_instance VI_1 {
  	# 配置为备节点
      state BACKUP
      # ip addr 查看对应的网卡名称
      interface enp0s8
      virtual_router_id 3
      # 优先级比主节点低
      priority 100
      advert_int 1 
      authentication {
          auth_type PASS
          auth_pass 123456
      }
      #执行监控的服务
      track_script {
          #引用VRRP脚本，即在 vrrp_script 部分指定的名字。
          chk_nginx                    
      }
      virtual_ipaddress {
          # 虚拟IP
          192.168.30.108
      }
  }
  ```

- 第四步分别启动两台虚拟机上的nginx和keepalived，命令如下：

  ```bash
  # 启动nginx
  cd /usr/local/nginx/sbin/
  ./nginx
  # 启动keepalived
  systemctl start keepalived.service
  # 查看keepalived状态,是否运行
  systemctl status keepalived.service
  ```

  两台虚拟机都要执行

- 第五步测试，效果如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5f971b4a7ad48d892d209fd8aa8233a~tplv-k3u1fbpfcp-zoom-1.image)

  直接访问虚拟IP就能访问到主节点的服务啦；现在测试当主节点挂掉时，还会不会正常访问服务，在105机器上执行如下命令：

  ```bash
  # 模拟宕机，停止keepalived
  systemctl stop keepalived.service
  # ip addr 查看虚拟IP已经漂移到备节点上了，在备节点用ip addr 查看
  ```

  备节点显示，虚拟IP已经漂移过来啦~

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/909b1ad7e9e04b9abaa3d0d5b62ad7ad~tplv-k3u1fbpfcp-zoom-1.image)

  再用虚拟IP访问，效果如下：

  ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/484a867956e14a22aad3807d2c1d29a4~tplv-k3u1fbpfcp-zoom-1.image)

  看见已经切换到106机器上提供服务啦，这样就实现高可用啦；

  那主节点重新恢复，虚拟IP会不会恢复回来继续提供服务呢？

  ```bash
  # 重启keepalived
  systemctl restart keepalived.service
  #ip addr 查看虚拟IP情况，恢复过来啦
  ```

  继续用虚拟IP访问服务，又回到主服务啦，如果没有,那可能是浏览器缓存，因为这里用静态html演示，清掉缓存再访问。

## 8.3 多主多备演示

对于上面的主备模式，只有主节点提供服务，如果主节点不发生故障，备节点服务器就有点资源浪费啦，这个时候多主多备不仅能合理利用资源，还得提供备用服务(根据实际需要配置)，形成真正集群提供服务。

这里就演示一下双主双备的配置，思路就是在keepalived增加多个vrrp实例，105机器在vrrp实例VI_1中作为主节点，106机器作为备节点，在vrrp实例VI_2中，105机器作为备节点，106机器作为主节点，这样就形成了互为主备的模式，资源就能很好的利用啦；其他逻辑不变，只是分别在105、106机器上加上的keepalived.conf中加上VI_2实例即可，如下：

**105机器上keepalived.conf**内容如下：

```bash
global_defs {
       router_id 31
}
#检测nginx服务是否在运行
vrrp_script chk_nginx {
   #使用脚本检测
   script "/usr/local/src/chk_nginx.sh"
   #脚本执行间隔，每2s检测一次
   interval 2
   #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
   weight -5
   #连续2次检测失败才确定是真失败
   fall 2
   #检测到1次成功就算成功
   rise 1                   
}

vrrp_instance VI_1 {
       state MASTER
       interface enp0s8
       virtual_router_id 3
       priority 200
       advert_int 1 
       authentication {
               auth_type PASS
               auth_pass 123456
       }
       track_script {                      
                chk_nginx                    
        }
       virtual_ipaddress {
               192.168.30.108
       }
}
vrrp_instance VI_2 {
		# VI_1是MASTER,这里就是备节点
       state BACKUP
       interface enp0s8
       # 修改路由编号
       virtual_router_id 5
       # 备节点优先级稍低
       priority 100
       advert_int 1
       authentication {
               auth_type PASS
               auth_pass 123456
       }
       track_script {                      
                chk_nginx                    
        }
       virtual_ipaddress {
       		   # 虚拟IP
               192.168.30.109
       }
}
```

**106机器上keepalived.conf**内容如下：

```bash
global_defs {
       router_id 32
}
#检测nginx服务是否在运行
vrrp_script chk_nginx {
   #使用脚本检测
   script "/usr/local/src/chk_nginx.sh"
   #脚本执行间隔，每2s检测一次
   interval 2
   #脚本结果导致的优先级变更，检测失败（脚本返回非0）则优先级 -5
   weight -5
   #连续2次检测失败才确定是真失败
   fall 2
   #检测到1次成功就算成功
   rise 1                   
}
vrrp_instance VI_1 {
       state BACKUP
       interface enp0s8
       virtual_router_id 3
       priority 100
       advert_int 1 
       authentication {
               auth_type PASS
               auth_pass 123456
       }
        track_script {                      
                chk_nginx                    
        }
       virtual_ipaddress {
               192.168.30.108
       }
}
vrrp_instance VI_2 {
	   # 这里是主节点
       state MASTER
       interface enp0s8
       # 这里和105机器上的VI_2中id一致
       virtual_router_id 5
       priority 200
       advert_int 1
       authentication {
               auth_type PASS
               auth_pass 123456
       }
       track_script {                      
                chk_nginx                    
        }
       virtual_ipaddress {
       			# 虚拟IP
               192.168.30.109
       }
}
```

分别重启两台机器上的keepalived，执行命令如下：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16e00c4ffb9c40308a8f8049e93179ec~tplv-k3u1fbpfcp-zoom-1.image)

然后分别访问虚拟ip 192.168.30.108 和192.168.30.109，都能提供对应的服务啦，这里不截图了。