- [omnidb数据库web管理工具](https://www.cnblogs.com/xiao987334176/p/12698961.html)



# 一、概述

OmniDB是一个基于浏览器的工具，它简化了专注于交互性的数据库管理，旨在实现在Web端强大的数据库管理功能且是轻量级的，目前支持PostgreSQL、Oracle、MySQL / MariaDB，未来应该会支持Firebird、 SQLite、Microsoft SQL Server、IBM DB2等数据库

让我们一起看看它的一些特点：

1、Web工具：

可以从任何平台访问，使用浏览器作为媒介

2、响应式界面：

单个页面使用所有功能

3、统一工作空间：

在单个工作空间中管理的不同功能

4、简化编辑：

轻松添加和删除连接

5、安全性：

具有加密个人信息的多用户支持

6、交互式表格：

所有功能都使用交互式表格，允许以块为单位进行复制和粘贴

7、智能SQL编辑器：

上下文SQL代码智能提示

8、多主题SQL编辑器：

您可以选择许多可用的颜色主题

9、选项卡式SQL编辑器：

轻松添加，重命名或删除编辑器选项卡



# 二、安装

## 环境说明

操作系统：centos 7.6

ip地址：192.168.128.134

 

## 安装

打开官网

[https://omnidb.org](https://omnidb.org/)

点击下载，选择server

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414162655333-1846084412.png)

 

下载centos7的版本并安装

```
wget https://omnidb.org/dist/2.17.0/omnidb-server_2.17.0-centos7-amd64.rpm
rpm -ivh omnidb-server_2.17.0-centos7-amd64.rpm
```

 

启动

```
nohup omnidb-server -H 0.0.0.0 -p 8090 &
```

参数解释：

-H 允许连接的ip地址

-p 指定运行端口

 

## 访问页面

http://192.168.128.134:8090/

**默认的用户名和密码都是admin**

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163014240-278027218.png)

 

点击Connections-->New Connection

 ![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163129757-1944530837.png)

 

 

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163429059-1766482215.png)

 

Technology：数据库类型，这里选择mysql

Server：mysql连接地址

Port：mysql端口

User：mysql用户名

 

设置完成后，点击保存数据。

然后刷新页面，这里会出现一个mysql，点击MySQL

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163644571-447388399.png)

 

会要求输入密码

 ![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163713587-1253019600.png)

 

 验证成功后，会显示mysql的版本，以及数据库列表

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163809138-1691216032.png)

 

 打开其中一个表，点击Query Data

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414163906252-1566123917.png)

 

 效果如下：

![img](https://img2020.cnblogs.com/blog/1341090/202004/1341090-20200414164017046-1374411664.png)

 

 

如果需要docker方式部署，请参考链接：

https://blog.csdn.net/Aria_Miazzy/article/details/85161820

 

本文参考链接：

https://www.kutu66.com//GitHub/article_139004

https://www.hsedo.com/webnews/1598