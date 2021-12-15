# 百万并发下的nginx优化之道

原文地址：https://blog.51cto.com/godbliss/3006685

# 一、nginx地址重写

### 1、nginx地址重写（rewrite）介绍

​    nginx地址重写的主要功能是实现URL地址的重定向。服务器获得一个来访的URL请求，然后改写成服务器可以处理的另一个URL

语法格式：	rewrite	  旧的地址(支持正则)    新的地址    标签(可忽略)

### 2、主文件配置方式与步骤

#### ① 基本配置转发

```powershell
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf  //编辑主配置文件
server {
listen  80;
server_name	web.com.cn;
rewrite  "/a.html$"  /b.html;			//地址重写配置
[root@localhost ~]# /usr/local/nginx/sbin/nginx	-s reload		//重启nginx服务
1.2.3.4.5.6.
```

#### ② 基本正则转发

```powershell
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf  //编辑主配置文件
server {
listen  80;
server_name	web.com.cn;
rewrite  ^/  http://www.baidu.com.cn;			//地址重写配置^/指的是匹配/usr/local/nginx/html下的所有网页，访问任何网站的时候都会跳转到baidu.com.cn下
[root@localhost ~]# /usr/local/nginx/sbin/nginx	-s reload		//重启nginx服务
1.2.3.4.5.6.
```

#### ③ 高级正则地址重写

```powershell
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf  //编辑主配置文件
server {
listen  80;
server_name	web.com.cn;
rewrite  ^/(.*)$  http://www.baidu.com.cn/$1;			//地址重写配置，使用正则，()代表保留，nginx使用$1代表第一个保留字串，$2则代表第二个保留字串
[root@localhost ~]# /usr/local/nginx/sbin/nginx	-s reload		//重启nginx服务
1.2.3.4.5.6.
```

# 二、LNMP动态网站

L：linux操作系统
 N：Nginx网站服务软件
 M：Mysql、MariaDB数据库
 P：网站开发语言（PHP）

### 1、LNMP原理：

nginx:  单独部署，只能处理静态数据;
 静态数据：指的是每次打开或者访问的时候，都是看到相同的内容，不会发生变化的
 动态数据：每次运行，执行时都可以得到不同的结果

#### ① lnmp对静态数据的处理过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233722441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233801209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)

#### ② lnmp对动态数据的处理过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233629525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233850347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233858432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707233904469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ2MDAxOTMz,size_16,color_FFFFFF,t_70)

### 2、部署LNMP网站

#### ① 搭建nginx

```powershell
[root@localhost ~]# yum -y install gcc pcre-devel openssl-devel		//安装依赖包
[root@localhost ~]# tar -xf nginx-1.16.1   //解压编译包
[root@localhost ~]# cd nginx-1.16.1
[root@localhost nginx-1.16.1]#
--prefix=/usr/local/nginx \
--with-http_ssl_module
[root@localhost nginx-1.16.1]# make && make install   //编译安装
1.2.3.4.5.6.7.
```

#### ② 安装MariaDB数据库

```powershell
[root@localhost ~]# yum -y install mariadb-server mariadb mariadb-devel
1.
```

#### ③ 安装php解释器

```powershell
[root@localhost ~]# yum -y install php  php-fpm   php-mysql  
1.
```

#### ④ 启动所有服务

```powershell
[root@localhost ~]# /usr/local/nginx/sbin/nginx   //启动nginx服务
[root@localhost ~]# systemctl start mariadb   //开启mariadb服务
[root@localhost ~]# systemctl start php-fpm	//开启php-fpm服务
[root@localhost ~]# systemctl enable mariadb   //设置mariadb服务开机自启
[root@localhost ~]# systemctl enable php-fpm	//设置php-fpm服务开机自启
1.2.3.4.5.
```

# 三、配置动静分离

### 1、location语法

```powershell
localtion /test {
	deny 10.10.10.10;		//拒绝10.10.10.10
}
localtion /video {
/*允许20.20.20.20，其它的禁用*/

	allow 20.20.20.20;	
	deny all;
}
localtion / {
	allow all;   //允许所有
}
1.2.3.4.5.6.7.8.9.10.11.12.
```

### 2、修改配置文件，配置nginx动静分离

```powershell
[root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf
// location 处理静态网页
localtion / {
	root html;
	index index.html index.htm;
}

location ~\.php$ {
	root html;
	fastcgi_pass	127.0.0.1:9000;				//指定转发请求
	fastcgi_index	index.php;		//php为默认页面
	include 		fastcgi.conf;
}
[root@localhost ~]# /usr/local/nginx/sbin/nginx -s reload	//重启服务
[root@localhost ~]# iptables -F   // 清空防火墙策略
```