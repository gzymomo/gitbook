- 冰河团队：[【Nginx】使用Nginx如何解决跨域问题？看完这篇原来很简单！！](https://www.cnblogs.com/binghe001/p/13352407.html)



## Nginx如何解决跨域？

利用Nginx的反向代理功能解决跨域问题。

Nginx作为反向代理服务器，就是把http请求转发到另一个或者一些服务器上。通过把本地一个url前缀映射到要跨域访问的web服务器上，就可以实现跨域访问。对于浏览器来说，访问的就是同源服务器上的一个url。而Nginx通过检测url前缀，把http请求转发到后面真实的物理服务器。并通过rewrite命令把前缀再去掉。这样真实的服务器就可以正确处理请求，并且并不知道这个请求是来自代理服务器的。

## Nginx解决跨域案例

使用Nginx解决跨域问题时，我们可以编译Nginx的nginx.conf配置文件，例如，将nginx.conf文件的server节点的内容编辑成如下所示。

```bash
server {
        location / {
            root   html;
            index  index.html index.htm;
            //允许cros跨域访问
            add_header 'Access-Control-Allow-Origin' '*';

        }
        //自定义本地路径
        location /apis {
            rewrite  ^.+apis/?(.*)$ /$1 break;
            include  uwsgi_params;
            proxy_pass   http://www.binghe.com;
       }
}
```

然后我把项目部署在nginx的html根目录下，在ajax调用时设置url从http://www.binghe.com/apistest/test 变为 http://www.binghe.com/apis/apistest/test然后成功解决。

假设，之前我在页面上发起的Ajax请求如下所示。

```javascript
$.ajax({
        type:"post",
        dataType: "json",
        data:{'parameter':JSON.stringify(data)},
        url:"http://www.binghe.com/apistest/test",
        async: flag,
        beforeSend: function (xhr) {
 
            xhr.setRequestHeader("Content-Type", submitType.Content_Type);
            xhr.setRequestHeader("user-id", submitType.user_id);
            xhr.setRequestHeader("role-type", submitType.role_type);
            xhr.setRequestHeader("access-token", getAccessToken().token);
        },
        success:function(result, status, xhr){
     	
        }
        ,error:function (e) {
            layerMsg('请求失败，请稍后再试')
        }
    });
```

修改成如下的请求即可解决跨域问题。

```javascript
$.ajax({
        type:"post",
        dataType: "json",
        data:{'parameter':JSON.stringify(data)},
        url:"http://www.binghe.com/apis/apistest/test",
        async: flag,
        beforeSend: function (xhr) {
 
            xhr.setRequestHeader("Content-Type", submitType.Content_Type);
            xhr.setRequestHeader("user-id", submitType.user_id);
            xhr.setRequestHeader("role-type", submitType.role_type);
            xhr.setRequestHeader("access-token", getAccessToken().token);
        },
        success:function(result, status, xhr){
     	
        }
        ,error:function (e) {
            layerMsg('请求失败，请稍后再试')
        }
    });
```

