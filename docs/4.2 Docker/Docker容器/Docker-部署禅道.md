- [docker方式部署禅道](https://www.cnblogs.com/xiao987334176/p/14205606.html)



# 一、概述

使用docker方式部署禅道简单，快速，不容易出错。比起编译安装要方便很多。

 

# 二、部署

## 环境说明

操作系统：centos 7.6

ip地址：10.212.82.65

docker版本：19.03.8

配置：2核4g

 

关于docker安装，请参考链接：

https://www.cnblogs.com/xiao987334176/p/11771657.html

 

## 下载镜像

访问dockerhub链接：https://hub.docker.com/r/easysoft/zentao/tags

最新版本为：12.5.2

 

下载docker镜像

```
docker pull easysoft/zentao:12.5.2
```

 

创建持久化目录

```
mkdir -p /data/zentao/pms /data/zentao/mysql/data
```

 

## 启动禅道

```
docker run --name zentao -p 80:80 -v /data/zentao/pms:/www/zentaopms -v /data/zentao/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=abcd@1234 -d easysoft/zentao:12.5.2
```

说明：指定mysql初始密码为abcd@1234

 

查看日志

```
...
 * Stopping MariaDB database server mysqld
   ...done.
 * Starting MariaDB database server mysqld
   ...done.
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

这里出现了一个mysql错误，可以先忽略。因为此时禅道还没有初始化。

 

## 访问页面

注意：10.212.82.65是服务器ip

```
http://10.212.82.65/
```

效果如下，点击开始安装

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112059066-1047911311.png)

 

 然后点击下一步，输入数据库密码：abcd@1234

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112209998-856467139.png)

 

 输入账号信息

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112341591-818248734.png)

 

 

点击登录

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112355800-1541576001.png)

 

 

输入用户名和密码，都是admin

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112429851-827394036.png)

 

 

设置新密码

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112517334-643750570.png)

 

 

选择流程

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112534841-983058196.png)

 

 

首页效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202012/1341090-20201229112548015-450703284.png)

 

本文参考链接：

https://www.zentao.net/book/zentaopmshelp/40.html