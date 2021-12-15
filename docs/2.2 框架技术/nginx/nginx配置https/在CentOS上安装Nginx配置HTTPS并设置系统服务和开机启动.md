[在CentOS上安装Nginx配置HTTPS并设置系统服务和开机启动](https://www.cnblogs.com/zhaifanhua/p/14535350.html)



> 常用按键：
>
> 复制操作：Shift+Ins
>
> 编辑器vim按键操作具体如下：
>
> 开启编辑：  i  或者  Insert 
>
> 退出编辑：  Esc 
>
> 退出vim： : + q 
>
> 保存vim： : + w
>
> 保存退出vim： :  + w  + q 
>
> 不保存退出vim： :  + q + ! 

⛪1、所需要的依赖包括：gcc，pcre，zlib，openssl。



```
yum -y install gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

> **GCC库**（用来编译下载下来的 nginx 源码）
>
> GCC  是 Linux 下最主要的编译工具，GCC 不仅功能非常强大，结构也非常灵活。它可以通过不同的前端模块来支持各种语言，如 Java、Fortran、Pascal、Modula-3 和 Ada。
>
> **PCRE库**（ rewrite 模块需要 pcre 库）
>
> PCRE 支持正则表达式。如果我们在配置文件 nginx.conf 中使用了正则表达式，那么在编译 Nginx 时就必须把PCRE库编译进 Nginx，因为 Nginx 的 HTTP 模块需要靠它来解析正则表达式。另外，pcre-devel  是使用PCRE做二次开发时所需要的开发库，包括头文件等，这也是编译 Nginx 所必须使用的。
>
> **ZLIB库**（ gzip 模块需要 zlib 库）
>  zlib 提供了很多压缩和解方式，用于对 HTTP 包的内容做 gzip 格式的压缩，如果我们在 nginx.conf 中配置了 gzip  on，并指定对于某些类型（content-type）的HTTP响应使用 gzip 来进行压缩以减少网络传输量，则在编译时就必须把 zlib  编译进 Nginx。zlib-devel 是二次开发所需要的库。
>
> **OpenSSL库**（SSL功能需要 openssl 库）
>  openssl 是一个安全套接字层密码库，如果服务器不只是要支持 HTTP，还需要在更安全的 SSL 协议上传输 HTTP，那么需要拥有 OpenSSL。另外，如果我们想使用 MD5、SHA1 等散列函数，那么也需要安装它。

显示以下结果表示安装完成。

[![image-20210312142614045](https://i.loli.net/2021/03/15/xRlcIsi6gWeb4GQ.png)](https://i.loli.net/2021/03/15/xRlcIsi6gWeb4GQ.png)

> 这个不是必需！不想升级可直接跳第三步，怕麻烦最好不升级，不升级也可以用，因为升级我踩过好多坑。

yum安装的 openssl ，版本比较低，我们可以通过以下命令查看 openssl 的版本。



```
openssl version -a
```

[![image-20210313215300685](https://i.loli.net/2021/03/15/FrNOhAHLZ3GE9gB.png)](https://i.loli.net/2021/03/15/FrNOhAHLZ3GE9gB.png)

> openssl 的版本是 1.0.2k，这个是2017年发布的，已经有点老了，我们下载最新版本。

🏣1、进入 [openssl官网](http://www.openssl.org/source/) 查看版本，这里我下载的是 [openssl-1.1.1j.tar.gz](https://www.openssl.org/source/openssl-1.1.1j.tar.gz)  。



```
cd /usr/local
wget https://www.openssl.org/source/openssl-1.1.1j.tar.gz
```

☕2、解压 openssl-1.1.1j.tar.gz 并进入解压目录。



```
tar -zxvf openssl-1.1.1j.tar.gz
cd openssl-1.1.1j
```

🍯3、进行重新编译 openssl 。



```
./config shared zlib
make && make install
```

> 此过程需要 3 分钟左右……

编译完成后以下图：

[![image-20210314003448904](https://i.loli.net/2021/03/15/v4JasU5h6SWdpYz.png)](https://i.loli.net/2021/03/15/v4JasU5h6SWdpYz.png)

🍼4、备份当前openssl。



```
mv /usr/bin/openssl /usr/bin/openssl.bak
mv /usr/include/openssl /usr/include/openssl.bak
```

🍶5、配置使用新版本：



```
ln -s /usr/local/bin/openssl /usr/bin/openssl
ln -s /usr/local/include/openssl /usr/include/openssl
```

🍸6、更新动态链接库数据：



```
echo "/usr/local/lib64/" >> /etc/ld.so.conf
```

重新加载动态链接库。



```
ldconfig -v
```

🍴7、重新查看版本号进行验证：



```
openssl version -a
```

[![image-20210314013041087](https://i.loli.net/2021/03/15/FtQyIWRTgch7CLr.png)](https://i.loli.net/2021/03/15/FtQyIWRTgch7CLr.png)

🍻8、为适应NGINX编译需要设置参数，需要修改openss路径，不然会出现找不到openssl目录的问题。



```
cd auto/lib/openssl
cp conf /mnt/
vim conf
```

将



```
        CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
        CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
        CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
        CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
```

修改为



```
        CORE_INCS="$CORE_INCS $OPENSSL/include"
        CORE_DEPS="$CORE_DEPS $OPENSSL/include/openssl/ssl.h"
        CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libssl.a"
        CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libcrypto.a"
```

🍱9、建立libssl.a和libcrypto.a的软连接。



```
mkdir /usr/local/ssl/lib
ln -s /usr/local/lib64/libssl.a /usr/local/ssl/lib/libssl.a
ln -s /usr/local/lib64/libcrypto.a /usr/local/ssl/lib/libcrypto.a
```

🍙10、建立软连接openssl后，/usr/local/ssl/下没有include路径，需重新指向。



```
ln -s /usr/include/ /usr/local/ssl/include
ll
```

🍥1、先进入到 usr/local 目录，创建并进入 nginx 目录。



```
cd /usr/local
mkdir nginx
cd nginx
```

🍤2、通过 wget 命令在线获取到 nginx 的安装包，这里我选择的是 nginx-1.17.7 这个版本，命令是：



```
wget https://nginx.org/download/nginx-1.17.7.tar.gz
```

> 最新版本可以直接点直通车查看：[nginx下载地址](https://nginx.org/download)

🍢3、解压 nginx-1.17.7.tar.gz。



```
tar -zxvf nginx-1.17.7.tar.gz
```

🍩4、进入刚刚解压的文件夹nginx-1.17.7进行安装。



```
cd nginx-1.17.7
```

🍚5、使用--prefix参数指定nginx安装的目录。



```
./configure --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module
```

🍢6、用make、make install安装，安装命令。



```
make && make install
```

显示以下结果表示安装完成。

[![image-20210314050029293](https://i.loli.net/2021/03/15/ShzbZVRoi1gWEw4.png)](https://i.loli.net/2021/03/15/ShzbZVRoi1gWEw4.png)

🍝7、查看nginx版本。



```
/usr/local/nginx/sbin/nginx -V
```

看到版本号为 1.17.7，各个模块也安装正常。

[![image-20210314050114769](https://i.loli.net/2021/03/15/rL2sSxP8IJ7qDAF.png)](https://i.loli.net/2021/03/15/rL2sSxP8IJ7qDAF.png)

执行完成后看看Nginx安在什么地方了。



```
whereis nginx
```

可以看到，我的nginx是安装在了 /usr/local/nginx。

[![image-20210314050157365](https://i.loli.net/2021/03/15/cB5ofPXH9jLRgQa.png)](https://i.loli.net/2021/03/15/cB5ofPXH9jLRgQa.png)

🍦8、运行命令启动nginx。



```
/usr/local/nginx/sbin/nginx
```

🍰9、用ps命令看看nginx进程跑起来没有。



```
ps -ef | grep nginx
```

跑起来了，说明安装没有问题。

[![image-20210314050237838](https://i.loli.net/2021/03/15/Z9tXGcSl6zkiUMf.png)](https://i.loli.net/2021/03/15/Z9tXGcSl6zkiUMf.png)

🍩1、现在在浏览器输入ip地址或者localhost，就能看到Nginx的欢迎界面了。

[![image-20210312152634185](https://www.cnblogs.com/zhaifanhua/p/image-20210312152634185.png)](https://www.cnblogs.com/zhaifanhua/p/image-20210312152634185.png)

到此，Nginx成功完成安装！接下来就是配置nginx让别人能访问到你的网站。

🍮1、打开文件夹 /usr/local/nginx/conf ，查看目录结构。



```
cd /usr/local/nginx/conf
ll
```

[![image-20210312160909002](https://i.loli.net/2021/03/15/DBt7GjlLxdPCFKk.png)](https://i.loli.net/2021/03/15/DBt7GjlLxdPCFKk.png)

可以看到里面有nginx.conf这个文件，使用命令 vim 打开时是这样的

[![image-20210312161955404](https://i.loli.net/2021/03/15/wRkLQuzWtTy5MHJ.png)](https://i.loli.net/2021/03/15/wRkLQuzWtTy5MHJ.png)

`后面要用到的链接`说明文件

这里有必要对nginx.conf说明一下：



```
#user  nobody;
worker_processes  1; # 工作进程：数目。根据硬件调整，通常等于cpu数量或者2倍cpu数量。

#错误日志存放路径。
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid; # nginx进程pid存放路径。

events {
    worker_connections  1024; # 工作进程的最大连接数量。
}


http {
    include       mime.types; # 指定mime类型，由mime.type来定义。
    default_type  application/octet-stream;

    # 日志格式设置
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # 用log_format指令设置日志格式后，需要用access_log来指定日志文件存放路径。
    #access_log  logs/access.log  main;

    # 指定nginx是否调用sendfile函数来输出文件，对于普通应用，必须设置on。如果用来进行下载等应用磁盘io重负载应用，可设着off，以平衡磁盘与网络io处理速度，降低系统uptime。
    sendfile        on;
    # 此选项允许或禁止使用socket的TCP_CORK的选项，此选项仅在sendfile的时候使用。
    #tcp_nopush     on;
    
    # keepalive超时时间。
    #keepalive_timeout  0;
    keepalive_timeout  65;

    # 开启gzip压缩服务。
    #gzip  on;

    # 虚拟主机。
    server {
        listen       80;  # 配置监听端口号。
        server_name  localhost;  # 配置访问域名，域名可以有多个，用空格隔开。

        #charset koi8-r;  # 字符集设置。

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        # 错误跳转页。
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        # 请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
        #location ~ \.php$ {
        
        #    root           html; # 根目录。
        #    fastcgi_pass   127.0.0.1:9000; # 请求转向定义的服务器列表。
        #    fastcgi_index  index.php; # 如果请求的Fastcgi_index URI是以 / 结束的, 该指令设置的文件会被附加到URI的后面并保存在变量$fastcig_script_name中。
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl; # 监听端口。
    #    server_name  localhost; # 域名。

    #    ssl_certificate      cert.pem; # 证书位置。
    #    ssl_certificate_key  cert.key; # 私钥位置。

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5; # 密码加密方式。
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

🍞2、在这里建议将现有的nginx.conf文件复制一份出来并重命名为nginx.conf.bak文件，方便以后配置错了恢复。



```
cp -b /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
```

🍭3、然后打开 /usr/local/nginx/conf 看看目录结构，发现已经复制好了



```
cd /usr/local/nginx/conf
ll
```

[![image-20210314050447427](https://i.loli.net/2021/03/15/5WRGESCgMvcXA48.png)](https://i.loli.net/2021/03/15/5WRGESCgMvcXA48.png)

这时后就可以安心修改nginx.conf文件了！注：上面的说明文件和conf源文件一模一样，我只是加了注释，放心复制。

🍪4、直接在本地创建一个空 nginx.conf 文件，复制上面 [说明文件](https://www.cnblogs.com/zhaifanhua/p/14535350.html#shuoming)的代码，按照我的注释配置你的网站：

> 一般情况下只需要在最后一个}之前添加server配置就可以了

[![image-20210313170838307](https://i.loli.net/2021/03/15/IcmynTujfboQLxY.png)](https://i.loli.net/2021/03/15/IcmynTujfboQLxY.png)

比如我的个人主页网站配置如下：

[![image-20210315063102695](https://i.loli.net/2021/03/15/K16NxBa5loUeYzS.png)](https://i.loli.net/2021/03/15/K16NxBa5loUeYzS.png)

下面说说我为什么这么配置（如果你的网站不需要https访问，这一步可以忽略）。

> 现在网站 https 已经是标配，这个配置还是很重要的。



```
# -------------------http-----------------------
    # -------------配置跳转https-------------
    server {
        listen 80;
        server_name *.zhaifanhua.cn;
        return 301 https://$http_host$request_uri;
    }
```

> 上面的配置是为了 http 跳转为 https ，当访问 80 端口时 nginx 自动转发为已经存在配置网址的 443 端口。比如别人访问网站 http://zhaifanhua.cn 会自动跳转为 https://zhaifanhua.cn 。
>
> 这里我设置为 *.zhaifanhua.cn 是因为我有很多站点都需要跳转为 https 协议访问，并且我的证书是泛域名解析，干脆用通配符 * 来配置一劳永逸。这样访问任何前缀的域名都可以跳转为 https 协议了。

下面就是正式配置网站目录和证书了，一下内容将参数 server_name、ssl_certificate、ssl_certificate_key、location 下的 root 修改你自己的域名和路径即可。



```
# -------------------https-----------------------
    # -------------摘繁华主页------------------
    server {
        listen					443 ssl;
        server_name				zhaifanhua.cn www.zhaifanhua.cn;
        # 证书位置
        ssl_certificate			/home/ssl/zhaifanhua.cn/full_chain.pem;
        # 私钥位置
        ssl_certificate_key		/home/ssl/zhaifanhua.cn/private.key;
        # 开启HSTS
        add_header Strict-Transport-Security "max-age=63072000" always;
        # 装订在线证书状态协议
        ssl_stapling            on;
        ssl_stapling_verify     on;
        # 配置会话缓存
        ssl_session_cache		shared:SSL:1m;
        ssl_session_timeout		5m;
        # 配置协议
        ssl_protocols			TLSv1 TLSv1.1 TLSv1.2;
        # 配置加密套件
        ssl_ciphers				ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

        # 根目录
        root                    /home/www/www;
    }
```

🍢5、修改完成后保存，然后用xftp上传到 /usr/local/nginx/conf 目录进行覆盖源文件，然后判断配置文件是否正确，有successful表示可用。



```
/usr/local/nginx/sbin/nginx -t
```

🍲6、配置正确后，重启 nginx 重新加载配置文件使配置生效：



```
/usr/local/nginx/sbin/nginx -s reload
```

🔒7、这时候访问网站已经出现了锁，说明 https 开启成功。

[![image-20210315005057299](https://i.loli.net/2021/03/15/nlrvjGMAwai1hyB.png)](https://i.loli.net/2021/03/15/nlrvjGMAwai1hyB.png)

🎃1、在系统服务目录里创建 nginx.service 文件。



```
vim /lib/systemd/system/nginx.service
```

写入内容如下：



```
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

> [Unit] 服务的说明
>  Description 描述服务
>  After 描述服务类别
>  [Service] 服务运行参数的设置
>  Type=forking 是后台运行的形式
>  ExecStart 为服务的具体运行命令
>  ExecReload 为重启命令
>  ExecStop 为停止命令
>  PrivateTmp=True 表示给服务分配独立的临时空间
>  [Install] 运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

注意：[Service]的启动、重启、停止命令全部要求使用绝对路径，保存退出。

🎆2、设置用户的权限，重新加载 nginx.service 文件



```
chmod 755 ./nginx.service
systemctl daemon-reload
```

🎏3、杀死 nginx 重启nginx



```
pkill -9 nginx
systemctl restart nginx
ps aux | grep nginx
```

[![image-20210315025038577](https://i.loli.net/2021/03/15/sZ5T7SKRbIvgFoY.png)](https://i.loli.net/2021/03/15/sZ5T7SKRbIvgFoY.png)

说明配置系统服务成功！

📷4、设置开机自启动



```
systemctl enable nginx.service
```

[![image-20210315025338229](https://i.loli.net/2021/03/15/lgCiOB6EZtu1Tho.png)](https://i.loli.net/2021/03/15/lgCiOB6EZtu1Tho.png)

说明开机自启动设置成功！



```
# 启动nginx服务
service nginx start
systemctl start nginx.service
# 查看服务当前状态
service nginx status
systemctl status nginx.service
# 重新启动服务
service nginx reload
systemctl restart nginx.service　
# 停止nginx服务
service nginx stop
systemctl stop nginx.service

# 设置开机自启动
systemctl enable nginx.service
# 停止开机自启动
systemctl disable nginx.service

# 查看所有已启动的服务
systemctl list-units --type=service
```

