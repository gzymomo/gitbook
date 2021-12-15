- [使用HTML5地理位置定位到城市的方法及注意事项](http://www.yanzuoguang.com/article/794)



# 使用HTML5地理位置定位到城市的方法及注意事项

## 介绍

本文将简述一下如何通过HTML5和百度地图开放平台提供的API来实现对浏览器的定位，同时记录了遇到的问题和解决方案。实现效果为显示出用户所在的省市，即： XXX省 XXX市。

## 实现思路

利用HTML5 提供的API获取到用户的经纬度信息，再将用户的经纬度信息传到百度地图开放平台，百度地图开放平台根据提供的坐标信息返回当前的省市。

## 兼容性及依赖

兼容性：Internet Explorer 9+, Firefox, Chrome, Safari 和 Opera 都支持Geolocation（地理定位）.

依赖：不依赖于任何库和框架

## HTML5 API

使用HTML5 Geolocation API的getCurrentPosition() 方法能够获取用户的经纬度信息。

getCurrentPosition() 常用参数有两个，一个是成功时执行，一个时错误处理。如果getCurrentPosition()运行成功，则向第一个参数中规定的函数返回一个coordinates对象，用于提供位置信息。

coordinates对象属性如下：

| 属性                    | 描述                   |
| ----------------------- | ---------------------- |
| coords.latitude         | 十进制数的纬度         |
| coords.longitude        | 十进制数的经度         |
| coords.accuracy         | 位置精度               |
| coords.altitude         | 海拔，海平面以上以米计 |
| coords.altitudeAccuracy | 位置的海拔精度         |
| coords.heading          | 方向，从正北开始以度计 |
| coords.speed            | 速度，以米/每秒计      |
| timestamp               | 响应的日期/时间        |

其中，latitude、longitude 以及 accuracy 属性 是固定返回的属性，其他属性在可用状态下才会一起返回。

如果getCurrentPosition()运行失败，则向第二个参数中规定的函数返回一个error对象，用于提供错误信息。

| 属性    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| code    | 1: PERMISSION_DENIED,用户不允许地理定位 2: POSITION_UNAVAILABLE,无法获取当前位置 3:TIMEOUT,操作超时 |
| message | 返回相应的错误信息                                           |

下例是一个简单的地理定位实例，可返回用户位置的经度和纬度:

```js
function getLocation(){
    if (navigator.geolocation){
        navigator.geolocation.getCurrentPosition(showPosition, showError);
    }
    else{
        console.log("该浏览器不支持获取地理位置。");
    }
}

function showPosition(position){
    console.log("纬度: " + position.coords.latitude + " 经度: " + position.coords.longitude);    
}

function showError(error) {
    console.log(error);
}
```

参考资料：

[菜鸟教程](http://www.yanzuoguang.com/article/794#)

## 百度地图API

#### 命名空间

百度地图API使用BMap作为命名空间，所有类均在该命名空间之下，比如：BMap.Map、BMap.Control、BMap.Overlay。

#### 使用方法

首先，我们需要引入百度地图API文件：

```javascript
//参数v表示您加载API的版本，使用JavaScript APIv1.4及以前版本可使用此方式引用。
<script src="http://api.map.baidu.com/api?v=1.4"></script> 
//使用JavaScript APIv2.0请先申请密钥ak，按此方式引用。
<script src="http://api.map.baidu.com/api?v=2.0&ak=您的密钥"></script> 
```

然后，我们再创建一个地理点坐标：

```javascript
var point = new BMap.Point(116.404, 39.915);
```

| 构造函数                        | 描述                                 |
| ------------------------------- | ------------------------------------ |
| Point(lng: Number, lat: Number) | 以指定的经度和纬度创建一个地理点坐标 |

最后，我们可以通过创建好的点坐标获取到用户的地址解析：

```javascript
var myGeo = new BMap.Geocoder();
myGeo.getLocation(point, function (result) {
    console.log(result.addressComponents.province + ' ' + result.addressComponents.city);
}
```

| 构造函数   | 描述                     |
| ---------- | ------------------------ |
| Geocoder() | 创建一个地址解析器的实例 |

| 方法                                                         | 返回值 | 描述                                                         |
| ------------------------------------------------------------ | ------ | ------------------------------------------------------------ |
| getPoint(address: String, callback: Function, city: String)  | none   | 对指定的地址进行解析。如果地址定位成功，则以地址所在的坐标点Point为参数调用回调函数。否则，回调函数的参数为null。city为地址所在的城市名，例如“北京市” |
| getLocation(point: Point, callback: Function, options: LocationOptions) | none   | 对指定的坐标点进行反地址解析。如果解析成功，则回调函数的参数为GeocoderResult对象，否则回调函数的参数为null |

参考资料：

1. [百度地图 JavaScript API](http://www.yanzuoguang.com/article/794#)
2. [百度地图 JavaScript API起步](http://www.yanzuoguang.com/article/794#)
3. [百度地图 JavaScript API类参考](http://www.yanzuoguang.com/article/794#)

## 完整代码示例

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <p id="test"></p>
    </body>
    <script src="https://api.map.baidu.com/api?v=2.0&ak=您的密钥" type="text/javascript"></script>
    <script>
        function getLocation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(showPosition, showError);
            }
            else {
                console.log("该浏览器不支持获取地理位置。");
            }
        }

        function showPosition(position) {
            var point = new BMap.Point(position.coords.longitude, position.coords.latitude);
            var myGeo = new BMap.Geocoder();
            myGeo.getLocation(point, function (result) {
                alert(result.addressComponents.province + ' ' + result.addressComponents.city);
            })
        }

        function showError(error) {
            console.log(error);
        }
        getLocation();
    </script>
</html>
```

## 浏览器安全机制

按照上面的步骤去做，理论上是可以实现我们的功能。但事实并非如此，不信你可以起个服务验证一下看看。

通过验证，你会发现在Chrome 浏览器中使用`http://localhost:8080` 或者 `http://127.0.0.1:8080` 可以正常获取到浏览器的地理位置，但通过IP或者域名的形式，如：`http://172.21.3.82:8080` 和 `http://b.cunzhang.com`进行访问时却获取不到。

为什么呢？打开控制台，你会发现有以下错误信息： `Only secure origins are allowed (see: https://goo.gl/Y0ZkNV).`

“只有在安全来源的情况才才被允许”。错误信息里还包含了一个提示链接，我们不妨打开这个链接（[https://goo.gl/Y0ZkNV](http://www.yanzuoguang.com/article/794#)）看看。原来，为了保障用户的安全，Chrome浏览器认为只有安全的来源才能开启定位服务。那什么样才算是安全的来源呢？在打开链接的页面上有这么一段话：

> “Secure origins” are origins that match at least one of the following (scheme, host, port) patterns:
>
> - (https, *, *)
> - (wss, *, *)
> - (*, localhost, *)
> - (*, 127/8, *)
> - (*, ::1/128, *)
> - (file, *, —)
> - (chrome-extension, *, —)
>
> This list may be incomplete, and may need to be changed. Please discuss!

大概意思是说只有包含上述列表中的`scheme`、`host`或者`port`才会被认为是安全的来源，现在这个列表还不够完整，后续可能还会有变动，有待讨论。

这就可以解释了为什么在`http://localhost:8080` 和 `http://127.0.0.1:8080`访问下可以获取到浏览器的地理位置，而在`http://172.21.3.82:8080` 和 `http://b.cunzhang.com` 确获取不到了。如果需要在域名访问的基础上实现地位位置的定位，那我们只能把`http`协议升级为`https`了。

## nginx搭建https服务

从`http`升级为`https`要先获取一张证书。 证书是一个二进制文件，里面包含经过认证的网站公钥和一些元数据，要从经销商购买。由于我司已经升级到了`https`，就不需要我瞎折腾了，感兴趣的可以参考阮一峰老师的这篇文章《[HTTPS 升级指南](http://www.yanzuoguang.com/article/794#)》。虽然我司对外开放的网站都已经全面升级为`https`，但是内网的测试环境还是没有升级到`https`，下面将简述一下如何通过`nginx` 来搭建一个`https`服务。

#### SSL 证书

要设置安全服务器，使用公共钥创建一对公私钥对。大多数情况下，发送证书请求（包括自己的公钥），你的公司证明材料以及费用到一个证书颁发机构(CA)。CA验证证书请求及您的身份，然后将证书返回给您的安全服务器。 但是内网实现一个服务器端和客户端传输内容的加密，可以自己给自己颁发证书，只需要忽略掉浏览器不信任的警报即可！

#### 制作CA证书

XXXX.key CA私钥：

```shell
openssl genrsa -des3 -out XXXX.key 2048
```

XXXX.crt CA根证书（公钥）：

```shell
openssl req -new -x509 -days 365 -key XXXX.key -out XXXX.crt
```

在配置你的公钥过程中会让你填写一些信息，这时候随便填一下就可以了。

#### 虚拟主机配置文件

```nginx
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
    upstream sslfpm {
        server 127.0.0.1:9000 weight=10 max_fails=3 fail_timeout=20s;
    }
    
    server {
        listen      443;
        server_name  b.cunzhang.com;
        #为一个server开启ssl支持
        ssl                  on;
        #为虚拟主机指定pem格式的证书文件
        ssl_certificate      /etc/ssl/cunzhang_test.crt;
        #为虚拟主机指定私钥文件
        ssl_certificate_key  /etc/ssl/cunzhang_test.key;
        #客户端能够重复使用存储在缓存中的会话参数时间
        ssl_session_timeout  5m;
        #指定使用的ssl协议
        ssl_protocols  SSLv3 TLSv1;
        #指定许可的密码描述
        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        #SSLv3和TLSv1协议的服务器密码需求优先级高于客户端密码
        ssl_prefer_server_ciphers  on;

        location / {
            root  /etc/ssl/;
            autoindex on;
            autoindex_exact_size    off;
            autoindex_localtime on;
        }

        # redirect server error pages to the static page /50x.html
        #
        error_page  500 502 503 504  /50x.html;
        error_page  404 /404.html;

        location = /50x.html {
            root  /usr/share/nginx/www;
        }
        location = /404.html {
            root  /usr/share/nginx/www;
        }

        # proxy the PHP scripts to fpm
        location ~ \.php$ {
            access_log  /etc/ssl/ssl.access.log ;
            error_log /etc/ssl/ssl.error.log;
            root /etc/ssl/;
            fastcgi_param HTTPS  on;
            include /usr/local/etc/nginx/fastcgi_params;
            fastcgi_pass    sslfpm;
        }
    }
}
```

虚拟主机文件配置之后还有记得在hosts给你的域名配置好ip地址，这样就可以通过`https`访问到你的网页实现定位功能了。