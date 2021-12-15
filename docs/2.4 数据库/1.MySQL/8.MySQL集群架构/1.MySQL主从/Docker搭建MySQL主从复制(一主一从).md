



## 一、Docker安装MYSQL

`说明` 系统为阿里云服务器，操作系统为**CentOS7.6**。MYSQL版本 **8.0.22**

#### 1、安装Docker

```
sudo apt-get update
sudo apt install docker.io
```

#### 2、拉取MySQL的镜像

```dockerfile
# 没有指定版本代表拉取最新版本 目前这里最新的是8.0.22。如果想指定版本可以docker pull mysql:5.7 代表下载5.7版本
docker pull mysql
```

运行完以上命令之后，镜像就已经下载下来了，可以用 **docker images** 命令查看是否已经下载成功

#### 3、第一次启动MySQL

```
docker run -p 3306:3306 --name MYSQL8 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
参数说明
-p 3306:3306:  将容器内的3306端口映射到实体机3306端口
--name MYSQL8: 给这个容器取一个容器记住的名字
-e MYSQL_ROOT_PASSWORD=123456: docker的MySQL默认的root密码是随机的，这是改一下默认的root用户密码
-d mysql:latest: 在后台运行mysql:latest镜像产生的容器
```

之后的第二次启动直接用 **docker start MYSQL8** 即可。

#### 4、连接navicat

新装了MYSQL8.0后再用navicat连接就会报2059的错误。上网查了发现是8.0之后MYSQL更改了密码的加密规则，只要在命令窗口把加密方法改回去即可。

1） **首先使用以下命令进入MySQL的Docker容器**

```sql
# MYSQL8是上面启动的时候 为该容器起的名称
docker exec -it MYSQL8 bash
```

2）**然后登录MySQL**

```sql
#这里的密码就是上面设置的密码
mysql -uroot -p123456
```

3）**最后运行以下SQL即可**

```sql
alter user 'root'@'%' identified by '123456' password expire never;
alter user 'root'@'%' identified with mysql_native_password by '123456';
flush privileges;
```

这样就可以通过navicat工具连接当前数据库了。这里顺便看下当前MYSQL的版本，通过 `select version()`;

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211720150-1013226764.jpg)

明显可以看到当前MYSQL的版本是 **8.0.22**

`注意` 我这边Master库和Slave库不在同一个服务器，所以Slave安装MYSQL的步骤和Master一样就可以了。

<font color='red'>**一定要记住Msater库和Slave库的MYSQL版本号要一致**。</font>



## 二、配置Master和Slave

这里假设主从服务器的IP如下

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211750687-275824980.jpg)

#### 1、配置Master

因为是通过Docker部署的MYSQL，所以要进入Docker内部修改MYSQL配置文件

```
# MYSQL8是上面启动的时候 为该容器起的名称
docker exec -it MYSQL8 bash
```

进入容器后，切换到 **/etc/mysql** 目录下,使用vim命令编辑 **my.cnf** 文件。

`注意` 此时用vim 命令会报 vim: command not found，因此我们需要在Docker内部安装vim工具。安装步骤推荐一篇博客：[vi: command not found](https://blog.csdn.net/weixin_39800144/article/details/79231002)

```
在my.cnf添加如下配置
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
```

添加完后保存，同时退出当前Docker容器。因为修改了配置文件,所以要重启下该MYSQL，这里重启下该Docker容器就好了。

```
#  MYSQL8是上面启动的时候 为该容器起的名称
docker restart MYSQL8
```

这个时候我们通过工具连接该MYSQL服务器，你可以通过navicat或者Sequel pro等等，连接登上后。

```
创建用户并授权
--为从库服务器 设置用户名和密码（表明从服务器的ip必须为47.00.00.02,账号为slave 密码123456
CREATE USER 'slave'@'47.00.00.02' IDENTIFIED BY '123456'; 
grant replication slave, replication client on *.* to 'slave'@'47.00.00.02'; --设置权限
flush privileges;  --权限生效
```

至此，Master配置完成。

#### 2、配置从库

和上面一样进入到 **etc/mysql** 路径，使用vim命令编辑 **my.cnf** 文件：

```
## 设置server_id,注意要唯一 和master也不能一样
server-id=101 
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin  
## 设置为只读,该项如果不设置，表示slave可读可写
read_only = 1
```

配置完成后也需要重启Docker容器。

```
#  MYSQL是上面启动的时候 为该容器起的名称
docker restart MYSQL8
```

#### 3、开启Master-Slave主从复制

上面两步Master和Slave都配置成功了，而且Master也为Slave读取Master数据专门设置了一个账号，下面就来实现同步。

```
进入Master库
```

查看Master状态

```sql
--通过该命令可以查看master数据库当前正在使用的二进制日志及当前执行二进制日志位置
show master status
```

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211815316-655570833.jpg)

记住File和Position，后面Slave库会在这个文件这个位置进行同步数据。此时一定不要操作Master库，否则将会引起Master状态的变化，File和Position字段也将会进行变化。

```
进入Slave库
```

执行SQL

```sql
change master to
master_host='47.00.00.01',
master_user='slave',
master_password='123456',
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=156 ;
命令说明
master_host ：Master库的地址，指的是容器的独立ip,可以通过:
master_port ：Master的端口号，指的是容器的端口号(默认3306)
master_user ：用于数据同步的用户
master_password ：用于同步的用户的密码
master_log_file ：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos ：从哪个 Position 开始读，即上文中提到的 Position 字段的值
```

`使用start slave`命令开启主从复制过程

```sql
start slave;  -- 顺便提供下其它命令 stop slave 停止slave。reset slave重启slave。 reset master重启master。
```

启动之后我们来看下有没有成功。

`show slave status`命令

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211831382-1150139349.jpg)

从这张图很明显看出，对于Slave的两个线程都成功了，那就说明整个MYSQL主从搭建成功了。如果有一个为NO,那就需要到后面看错误日志，是什么原因出错了，解决下就好了。

```
Slave_IO_Running: 从服务器中I/O线程的运行状态，YES为运行正常
Slave_SQL_Running: 从服务器中SQL线程的运行状态，YES为运行正常
```

## 三、测试

这里简单做一个测试

#### 1、只在Mater 创建一张User表

现在 `只在Mater 创建一张User表`,如果现在Slave也同样生成这张User表，那就说明成功了。

```sql
CREATE TABLE `user` (
  `id` bigint NOT NULL COMMENT '主键',
  `user_name` varchar(11) NOT NULL COMMENT '用户名',
  `password` varchar(11) NOT NULL COMMENT '密码',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='用户表';
```

实际测试结果是成功的。

```
注意
```

这里如果我们先手动在Slave创建这张User表，然后再到Master创建User表那就出事情了。我们按照上面的步骤创建完后，去Slave通过 `show slave status` 查看

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211844949-1647951371.jpg)

发现 SQL线程都变NO了。原因很简单,错误日志也说明了(Error 'Table 'user' already exists')。因为你在Master创建User表的SQL会记录到bin-log日志中，然后Slave

去读取这个操作，然后写入Slave中的时后发现这个SQL执行失败,因为你Slave已经存在该User表，然后这整个主从复制就卡在这里了。这是个很严重的问题。

所以`一旦搭建主从复制成功,只要在Master做更新事件(update、insert、delete),不要在从数据做，否则会出现数据不一致甚至同步失败。`

#### 2、Master插入一条数据

```sql
INSERT INTO `user` (`id`, `user_name`, `password`, `create_time`)
VALUES
	(0, '张三', '123456', '2020-11-25 04:29:43');
```

再去Slave数据库查看

![img](https://img2020.cnblogs.com/blog/1090617/202011/1090617-20201126211904680-1861769724.jpg)

发现从数据库已经写入成功了。

`总结`：在搭建的过程可能还有其它的问题出现 你只要在Slave服务器，通过**show slave status**，如果两个IO是否为YES就代表是否成功，如果有为NO的，

后面有字段说明是什么原因导致的，你再根据相关错误信息去查询下解决方案，那就可以了。