- [Nginx性能优化功能- Gzip压缩(大幅度提高页面加载速度)](https://www.cnblogs.com/kevingrace/p/10018914.html)



# 一、Gzip介绍

Nginx开启Gzip压缩功能， 可以使网站的css、js 、xml、html 文件在传输时进行压缩，提高访问速度, 进而优化Nginx性能! Web网站上的图片，视频等其它多媒体文件以及大文件，因为压缩效果不好，所以对于图片没有必要支压缩，如果想要优化，可以图片的生命周期设置长一点，让客户端来缓存。 

开启Gzip功能后，Nginx服务器会根据配置的策略对发送的内容, 如css、js、xml、html等静态资源进行压缩, 使得这些内容大小减少，在用户接收到返回内容之前对其进行处理，以压缩后的数据展现给客户。这样不仅可以节约大量的出口带宽，提高传输效率，还能提升用户快的感知体验, 一举两得; 尽管会消耗一定的cpu资源，但是为了给用户更好的体验还是值得的。

经过Gzip压缩后页面大小可以变为原来的30%甚至更小，这样，用户浏览页面的时候速度会快得多。Gzip 的压缩页面需要浏览器和服务器双方都支持，实际上就是服务器端压缩，传到浏览器后浏览器解压并解析。浏览器那里不需要我们担心，因为目前的巨大多数浏览器 都支持解析Gzip过的页面。



## 1.1 Gzip压缩作用

Nginx实现资源压缩的原理是通过ngx_http_gzip_module模块拦截请求，并对需要做gzip的类型做gzip，ngx_http_gzip_module是Nginx默认集成的，**不需要重新编译，直接开启即可**。

将响应报⽂发送⾄客户端之前可以启⽤压缩功能，这能够有效地节约带宽，并提⾼响应⾄客户端的速度。Gzip压缩可以配置http,server和location模块下。

## 1.2 nginx中开启gzip实例

```yaml
server{
    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_types application/javascript text/css text/xml;
    gzip_disable "MSIE [1-6]\."; #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;
}
```

**直接gzip on：在nginx的配置中就可以开启gzip压缩**



## 1.3 什么样的资源不适合开启gzip压缩？

二进制资源：例如图片/mp3这样的二进制文件,不必压缩；因为压缩率比较小, 比如100->80字节,而且压缩也是耗费CPU资源的。



# 二、nginx开启Gzip压缩参数说明

Nginx开启Gzip压缩参数说明：

```yaml
gzip on;                 #决定是否开启gzip模块，on表示开启，off表示关闭；
gzip_min_length 1k;      #设置允许压缩的页面最小字节(从header头的Content-Length中获取) ，当返回内容大于此值时才会使用gzip进行压缩,以K为单位,当值为0时，所有页面都进行压缩。建议大于1k
gzip_buffers 4 16k;      #设置gzip申请内存的大小,其作用是按块大小的倍数申请内存空间,param2:int(k) 后面单位是k。这里设置以16k为单位,按照原始数据大小以16k为单位的4倍申请内存
gzip_http_version 1.1;   #识别http协议的版本,早起浏览器可能不支持gzip自解压,用户会看到乱码
gzip_comp_level 2;       #设置gzip压缩等级，等级越底压缩速度越快文件压缩比越小，反之速度越慢文件压缩比越大；等级1-9，最小的压缩最快 但是消耗cpu
gzip_types text/plain application/x-javascript text/css application/xml;    #设置需要压缩的MIME类型,非设置值不进行压缩，即匹配压缩类型
gzip_vary on;            #启用应答头"Vary: Accept-Encoding"
 
gzip_proxied off;
nginx做为反向代理时启用,off(关闭所有代理结果的数据的压缩),expired(启用压缩,如果header头中包括"Expires"头信息),no-cache(启用压缩,header头中包含"Cache-Control:no-cache"),
no-store(启用压缩,header头中包含"Cache-Control:no-store"),private(启用压缩,header头中包含"Cache-Control:private"),no_last_modefied(启用压缩,header头中不包含
  "Last-Modified"),no_etag(启用压缩,如果header头中不包含"Etag"头信息),auth(启用压缩,如果header头中包含"Authorization"头信息)
 
gzip_disable msie6;
(IE5.5和IE6 SP1使用msie6参数来禁止gzip压缩 )指定哪些不需要gzip压缩的浏览器(将和User-Agents进行匹配),依赖于PCRE库
 
######################################################################################################
#如下：修改nginx配置文件 /usr/local/nginx/conf/nginx.conf
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf        #将以下配置放到nginx.conf的http{ ... }区域中
 
#修改配置为
gzip on;                     #开启gzip压缩功能
gzip_min_length 10k;         #设置允许压缩的页面最小字节数; 这里表示如果文件小于10个字节，就不用压缩，因为没有意义，本来就很小.
gzip_buffers 4 16k;          #设置压缩缓冲区大小，此处设置为4个16K内存作为压缩结果流缓存
gzip_http_version 1.1;       #压缩版本
gzip_comp_level 2;           #设置压缩比率，最小为1，处理速度快，传输速度慢；9为最大压缩比，处理速度慢，传输速度快; 这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6
gzip types text/css text/xml application/javascript;      #制定压缩的类型,线上配置时尽可能配置多的压缩类型!
gzip_disable "MSIE [1-6]\.";       #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
gzip vary on;    #选择支持vary header；改选项可以让前端的缓存服务器缓存经过gzip压缩的页面; 这个可以不写，表示在传送数据时，给客户端说明我使用了gzip压缩
```

# 三、线上Nginx开启Gzip压缩常用配置

```yaml
[root@external-lb02 ~]# cat /data/nginx/conf/nginx.conf
........
http {
.......
    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 9;
    gzip_types       text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;
 
}
```

如果不开启Gzip压缩功能(即注释掉Gzip的相关配置), 查看某个图片大小

```yaml
[root@external-lb02 ~]#  ll  -h /data/web//www/test.bmp
-rw-r--r-- 1 root root 453K 3月  14 18:43 /data/web//www/test.bmp
```

如下可知, 文件没有被压缩,文件传输大小还是400多K

![img](https://img2018.cnblogs.com/blog/907596/201811/907596-20181126100243840-1974865114.png)

如果开启Nginx的Gzip压缩功能(即打开Gzip的相关配置), 然后再次访问test.bmp图片, 发现压缩后的该图片文件传输大小只有200多K !

![img](https://img2018.cnblogs.com/blog/907596/201811/907596-20181126100420593-1993957123.png)

通过上面测试对比, 发现Nginx开启Gzip压缩功能后, 定义的gzip type的文件在传输时的大小明显变小, 这样这会大大提高nginx访问性能. 

直接用curl测试命令:

```yaml
[root@fvtlb02 ~]# curl -I -H "Accept-Encoding: gzip, deflate" "http://fvtvfc-web.kevin.com/service-worker.js"
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Mon, 26 Nov 2018 02:19:16 GMT
Content-Type: application/javascript; charset=utf-8
Connection: keep-alive
Vary: Accept-Encoding
Last-Modified: Sun, 25 Nov 2018 22:28:15 GMT
Vary: Accept-Encoding
ETag: W/"5bfb21ff-40be"
Content-Encoding: gzip
 
如上,response header头信息中出现"Conten_Encoding: gzip" , 就说明Nginx已开启了压缩 (在浏览器访问, 通过F12看请求的响应头部 也是一样)
```

Nginx的Gzip压缩功能虽然好用，但是下面两类文件资源不太建议启用此压缩功能。
**1) 图片类型资源 (还有视频文件)**
原因：图片如jpg、png文件本身就会有压缩，所以就算开启gzip后，压缩前和压缩后大小没有多大区别，所以开启了反而会白白的浪费资源。（可以试试将一张jpg图片压缩为zip，观察大小并没有多大的变化。虽然zip和gzip算法不一样，但是可以看出压缩图片的价值并不大）

**2) 大文件资源**
原因：会消耗大量的cpu资源，且不一定有明显的效果。