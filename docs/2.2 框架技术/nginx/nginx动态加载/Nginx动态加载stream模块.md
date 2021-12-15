Ngix1.9.11开始增加加载动态模块支持，从此不再需要替换nginx文件即可增加第三方扩展。目前官方只有几个模块支持动态加载，第三方模块需要升级支持才可编译成模块。
我们测试下通过nginx动态加载模块，添加stream模块实现tcp 反向代理功能。

##### 查看支持

```bash
./configure --help | grep dynamic
  --with-http_xslt_module=dynamic    enable dynamic ngx_http_xslt_module
  --with-http_image_filter_module=dynamic
 enable dynamic ngx_http_image_filter_module
  --with-http_geoip_module=dynamic   enable dynamic ngx_http_geoip_module
  --with-http_perl_module=dynamic    enable dynamic ngx_http_perl_module
  --with-mail=dynamic                enable dynamic POP3/IMAP4/SMTP proxy module
  --with-stream=dynamic              enable dynamic TCP/UDP proxy module
  --with-stream_geoip_module=dynamic enable dynamic ngx_stream_geoip_module
  --add-dynamic-module=PATH          enable dynamic external module
  --with-compat                      dynamic modules compatibility
```

##### 动态加载stream模块

```bash
nginx -V
nginx version: nginx/1.12.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.0.2l  25 May 2017
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-ipv6 --with-http_sub_module --with-openssl=/home/packet/lnmp1.4-full/src/openssl-1.0.2l
```

在原始参数后面添加 –with-stream=dynamic

```
./configure  --user=www --group=www --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-http_v2_module --with-http_gzip_static_module --with-ipv6 --with-http_sub_module --with-openssl=/home/packet/lnmp1.4-full/src/openssl-1.0.2l --with-stream=dynamic
```

**make编译 ，切记不要install**
当前目录objs中会存在 编译好的 ngx_stream_module.so

```bash
cp ngx_stream_module.so    nginx_worker_dir/modules/
```

##### 更改配置

```bash
load_module  modules/ngx_stream_module.so;

stream {
    upstream proxy_name {
        server 127.0.0.1:9553;
    }
    server {
        listen 8088;
        proxy_pass proxy_name;
    }
}
```

reload nginx 测试stream是否生效。