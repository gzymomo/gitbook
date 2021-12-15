## Nginx配置

Nginx的配置这块和普通的一样就可以了，只要在nginx/html 目录新增文件即可。然后通过Nginx的IP加上文件的路径即可下载，比如在nginx/html目录创建一个test目录，然后在test目录在创建一个xuwujing.txt和xuwujing.zip的文件，最在浏览器输入 http://localhost:8080/test/xuwujing.zip，即可进行下载。

示例图:

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/4/30/171cb5f10a0da48e?imageView2/0/w/1280/h/960/ignore-error/1)



### 静态文件下载

上述的配置可以简单满足一些要求，但是有时候我们想通过nginx进行下载其他的格式的文件时候，比如下载一张图片，但是访问这个url浏览器会自动展现这张图片，那么这时我们就可以通过增加配置，并且让浏览器下载该图片。 例如，我们在访问test目录的静态文件，那么我们在nginx/conf中添加如下配置即可!

```
      location /test {
           add_header Content-Disposition "attachment;";
        }     
复制代码
```

示例图:

未加配置的时候:

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/4/30/171cb5f10c43b62f?imageView2/0/w/1280/h/960/ignore-error/1)



添加配置的时候:

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/4/30/171cb5f10cbc6996?imageView2/0/w/1280/h/960/ignore-error/1)



### 指定文件存放路径

Nginx的文件路径默认在安装的nginx/html 目录下，如果我们想改变这路径，可以将location 的root  路径进行更改，比如更改到opt目录下 :

```
 location / {
           root   /opt/nginx/nginx-1.8.0/html;
           index  index.html index.htm;
}
复制代码
```

### nginx/conf 配置

那么nginx/conf的配置如下

```
worker_processes  1;
 
events {
    worker_connections  1024;
}
 
 
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    sendfile        on;

    keepalive_timeout  65;
 

 
    server {
        listen       8080;
        server_name  localhost;
 
   
        
             location / {
            root   /opt/nginx/nginx-1.8.0/html;
            index  index.html index.htm;
 
        }
        
        
        location /test {
           add_header Content-Disposition "attachment;";
        }
        
     
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```


作者：虚无境
链接：https://juejin.cn/post/6844904145451761672