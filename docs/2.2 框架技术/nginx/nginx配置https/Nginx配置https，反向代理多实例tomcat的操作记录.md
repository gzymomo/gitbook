- [centos7.x下环境搭建(五)—nginx搭建https服务](https://www.cnblogs.com/fozero/p/10968550.html)
- [nginx配置ssl证书实现https](https://www.cnblogs.com/zhoudawei/p/9257276.html)
- [Nginx配置SSL证书](https://www.cnblogs.com/zyh-s/p/13253531.html)



# 一、https证书获取

十大免费SSL证书：https://blog.csdn.net/ithomer/article/details/78075006

如果用的是阿里云或腾讯云，他们都提供了免费版的ssl证书，直接下载就可以了，这里我使用的是阿里云提供的免费证书。

腾讯云免费证书是crt格式，使用的时候搞了半天老是出问题，遂立马转阿里，搞的SSL证书是pem格式，立马就能使用了。



# 二、修改nginx配置

1、在nginx安装目录下创建cert目录并将.pem和.key的证书拷贝到该目录下
.crt文件：是证书文件，crt是pem文件的扩展名。
.key文件：证书的私钥文件

2、nginx.conf配置

如果我们需要配置多站点的话，我们在根目录下创建一个vhost目录，并新建.conf结尾的文件，加入以下内容

```yaml
server {
    listen              443 ssl;
    server_name         test.test.cn;
    ssl_certificate     /etc/nginx/cert/2283621_test.cn.pem;
    ssl_certificate_key /etc/nginx/cert/2283621_test.cn.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
         location / {
           proxy_set_header Host $host;
           proxy_set_header X-Real-Ip $remote_addr;
           proxy_set_header X-Forwarded-For $remote_addr;
           proxy_pass http://localhost:7001;
        }
 }
```

然后修改nginx.conf配置文件，并在http节点里面最后一行添加如下配置

```yaml
include /etc/nginx/vhost/*.conf;
```

这样在配置多站点的时候，我们就直接可以在vhost下新建一个.conf结尾的文件就行了

3、转发80端口到https

配置完https之后，默认端口是443，但如果使用http请求的话，并不会跳转到https，这时候我们需要将80端口转发到https上面来
添加80端口监听`listen 80`

```
server {
	listen       80;
    listen              443 ssl;
    server_name         love.diankr.cn;
    ssl_certificate     /etc/nginx/cert/2283621_test.cn.pem;
    ssl_certificate_key /etc/nginx/cert/2283621_test.cn.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
         location / {
           proxy_set_header Host $host;
           proxy_set_header X-Real-Ip $remote_addr;
           proxy_set_header X-Forwarded-For $remote_addr;
           proxy_pass http://localhost:7001;
        }
 }
```

这样在访问http://开头的地址的时候会自动跳转到https地址下

# 三、nginx错误日志

```
 tail -f -n 20  /var/log/nginx/error.log
```

## 问题总结

## 3.1 nginx: [warn] the "ssl" directive is deprecated的解决方法

这是一个warn警告，nginx提示ssl这个指令已经不建议使用，要使用listen ... ssl替代。网上查找nginx更新日志里面，也有提到：
`Change: the “ssl” directive is deprecated; the “ssl” parameter of the “listen” directive should be used instead.`
ssl不建议作为一个指令使用，而是应该listen指令的一个参数。

解决
如果使用listen 443 ssl，删除ssl on就行了。

