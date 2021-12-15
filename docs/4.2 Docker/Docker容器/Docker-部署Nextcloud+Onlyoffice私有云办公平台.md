- [使用docker安装Nextcloud+Onlyoffice ](https://www.cnblogs.com/cooper-73/p/13083161.html)
- [私有云办公平台大规模集群/企业级集群/小型工作室集群解决方案：NextCloud集群部署方案--NextCloud集群架构设计](https://blog.csdn.net/Aria_Miazzy/article/details/85028421)







# 一、安装内容

mysql、nextcloud、onlyoffice

 

# 二、镜像准备

```
~]# docker pull mysql
~]# docker pull nextcloud~]# docker pull onlyoffice
```

 

# 三、安装

## 3.1 安装mysql

```mysql
~]# docker run -p 3306:3306 --name fno_mysql -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/logs:/logs -v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
***********04d8c735c3b6133fb3af83d321bc72*************
~]# docker ps |grep mysql
757******bbb4        mysql               "docker-entrypoint.s…"   32 seconds ago      Up 31 seconds       0.0.0.0:3306->3306/tcp, 33060/tcp   fno_mysql
~]# docker exec -it 757******bbb4 /bin/bash
root@75767208bbb4:/# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.20 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant all PRIVILEGES on *.* to root@'%' WITH GRANT OPTION;  #授权root登录
Query OK, 0 rows affected (0.01 sec)
#修改root账号的密码验证插件类型为mysql_native_password这是mysql8之后的问题：
mysql> ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.02 sec)

mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

mysql>
```

 

## 3.2 安装nextcloud

```bash
~]# docker run -d \
>     -v /root/nextcloud/html:/var/www/html \
>     -v /root/nextcloud/apps:/var/www/html/custom_apps \
>     -v /root/nextcloud/config:/var/www/html/config \
>     -v /root/nextcloud/nextcloud/data:/var/www/html/data \
>     -v /root/nextcloud/themes:/var/www/html/themes \
>     -p 8080:80 \
>     nextcloud:17-apache
b3***************7d6c84328856c37233aca*******88a66c
~]#
```

 

访问[http://安装主机ip:8080/](http://10.1.234.68:8080/) 页面：

![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200610093624425-1748681963.png)

 

A: 创建管理员账号/密码；

B: 配置数据库；

C: 点击安装完成；

等待稍许分钟会安装完成：

访问 [http://安装主机ip:8080/](http://10.1.234.68:8080/)apps/files/ ，打开如下页面：

![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200610094510156-120616530.png)

 

## 3.3 安装onlyoffice

```bash
docker run -i -t -d -p 6060:80 --restart=always \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice  \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data  \
    -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
    -v /app/onlyoffice/DocumentServer/db:/var/lib/postgresql  onlyoffice/documentserver
```

访问 http:// ip:6060,打开如下页面即安装成功。


![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200610115049708-628939062.png)

 

##  3.4 下载onlyoffice插件并配置nextcloud

 

应用 》office&text 》右上角搜索onlyoffice ，点击下载并启用。

 ![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200611191147771-1379234859.png)

 

管理 》onlyoffice ，配置onlyofiice服务地址，点击保存，如下图即配置成功。

 

![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200611191500577-1154863231.png)

 

现在回到主页面，点击如图加号，可以看到已成功加载office组件。

![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200611191549767-1878626707.png)



至此，nextcloud+onlyoffice安装完成。

![img](https://img2020.cnblogs.com/blog/1495654/202006/1495654-20200611192506252-1409895707.png)

 