参考：

1. [使用Docker镜像搭建EMQ服务器搭建](https://blog.csdn.net/zdy_lyq/article/details/104366193?utm_medium=distribute.pc_relevant.none-task-blog-title-1&spm=1001.2101.3001.4242)
2. [使用docker搭建EMQ服务器](https://blog.csdn.net/nayiFuFu/article/details/81053894)
3. [一步步教你用Docker搭建MQTT服务器](https://blog.csdn.net/weixin_43676025/article/details/108401225)



# 一、Docker搭建MQTT服务器

## 1.1 拉取镜像

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/synbop/emqttd:2.3.6
```

## 1.2 运行镜像

- –name 名字
- -p 18083 服务器启动端口
- -p 1882 TCP端口
- -p 8083 WS端口
- -p 8084 WSS端口
- -p 8883 SSL端口
- -d 指定容器

```bash
docker run --name emq -p 18083:18083 -p 1883:1883 -p 8084:8084 -p 8883:8883 -p 8083:8083 -d registry.cn-hangzhou.aliyuncs.com/synbop/emqttd:2.3.6
```

## 1.3 进入emq服务页面

在浏览器输入`机器IP:18083` 就可以进入emqtt页面

初始的账户 admin, 密码 public

![emq页面](https://img-blog.csdnimg.cn/20200904113705510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzY3NjAyNQ==,size_16,color_FFFFFF,t_70#pic_center)

## 1.4 配置emq(对于V3.1.0)

为emq的用户配置权限 , emq还支持多种数据库验证, 包括 mongo, redis, pgsql 等等, 有兴趣可以自行研究

```bash
# 进入容器, 不能用 /bin/bash 进入
docker exec -it emq /bin/sh
```

- 1.首先先关闭匿名认证(默认是开启的谁都能够登录)

```bash
# 编辑配置文件
vi /opt/emqttd/etc/emq.conf
# 更改允许匿名 True -> false
allow_anonymous = false
```

- 2.建立用户和权限的 mysql 表, 可以拉一个 mysql 容器, 也可以直接在你的 ubuntu 里的 mysql 中创建

```sql
CREATE DATABASE emq charset utf8;

use eqm;

CREATE TABLE mqtt_user ( 
id int(11) unsigned NOT NULL AUTO_INCREMENT, 
username varchar(100) DEFAULT NULL, 
password varchar(100) DEFAULT NULL, 
salt varchar(20) DEFAULT NULL, 
is_superuser tinyint(1) DEFAULT 0, 
created datetime DEFAULT NULL, 
PRIMARY KEY (id), 
UNIQUE KEY mqtt_username (username) 
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE TABLE mqtt_acl ( 
id int(11) unsigned NOT NULL AUTO_INCREMENT, 
allow int(1) DEFAULT NULL COMMENT '0: deny, 1: allow', 
ipaddr varchar(60) DEFAULT NULL COMMENT 'IpAddress', 
username varchar(100) DEFAULT NULL COMMENT 'Username', 
clientid varchar(100) DEFAULT NULL COMMENT 'ClientId', 
access int(2) NOT NULL COMMENT '1: subscribe, 2: publish, 3: pubsub', 
topic varchar(100) NOT NULL DEFAULT '' COMMENT 'Topic Filter', 
PRIMARY KEY (id) 
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- 3.插入ACL规则 - [ACL规则](https://blog.csdn.net/weixin_43676025/article/details/108401225#ACL规则)

   tips: !!! 不要直接按照下面的例子设置, 先查看ACL规则了解之后在根据自己情况进行配置

```sql
INSERT INTO `mqtt_acl` (`id`, `allow`, `ipaddr`, `username`, `clientid`, `access`, `topic`) VALUES 
(1,1,NULL,'$all',NULL,2,'#'),
(2,0,NULL,'$all',NULL,1,'$SYS/#'),
(3,0,NULL,'$all',NULL,1,'eq #'),
(5,1,'127.0.0.1',NULL,NULL,2,'$SYS/#'),
(6,1,'127.0.0.1',NULL,NULL,2,'#'),
(7,1,NULL,'dashboard',NULL,1,'$SYS/#');
```

- 4.插入用户, 由此开始订阅与发布的 Client 都必须通过用户验证（sha256值请自行转换）

```sql
# 可以配置超级管理员(超级管理员会无视ACL规则对所有的topic都有订阅和推送的权限)
insert into mqtt_user (`username`, `password`) values ('admin', '03ac674216f3e15c761ee1a5e255f067953623c8b388b4459e13f978d7c846f4');
update mqtt_user set is_superuser=1 where id= 超级管理员ID ;

ps:注意 auth.mysql.password_hash(默认为sha256) 为sha256的话，新增用户时需要手动传递加密后的值，plain的话则无需加密，明码存放
```

- 5.修改emq的mysql配置文件

```
vi /opt/emqttd/etc/plugins/emq_auth_mysql.conf
auth.mysql.server = 你的mysql-IP:3306 
auth.mysql.username = root 
auth.mysql.password = xxxxxxxx 
auth.mysql.database = emq
```

- 6.重启emq

```bash
/opt/emqttd/bin/ emqx stop
/opt/emqttd/bin/ emqx start
/opt/emqttd/bin/emqttd_ctl plugins load emq_auth_mysql   #开启mysql认证插件
```

------

####  ACL规则

  **规则表字段说明：**

- allow：禁止（0），允许（1）
- ipaddr：设置 IP 地址
- username：连接客户端的用户名，此处的值如果设置为 $all 表示该规则适用于所有的用户
- clientid：连接客户端的 Client ID
- access：允许的操作：订阅（1），发布（2），订阅发布都可以（3）
- topic：控制的主题，可以使用通配符，并且可以在主题中加入占位符来匹配客户端信息，例如 `t/%c`则在匹配时主题将会替换为当前客户端的 Client ID
  %u：用户名
  %c：Client ID

  **示例**

```sql
-- 所有用户不可以订阅系统主题
INSERT INTO mqtt_acl (allow, ipaddr, username, clientid, access, topic) VALUES (0, NULL, '$all', NULL, 1, '$SYS/#');

-- 允许 10.59.1.100 上的客户端订阅系统主题
INSERT INTO mqtt_acl (allow, ipaddr, username, clientid, access, topic) VALUES (1, '10.59.1.100', NULL, NULL, 1, '$SYS/#');

-- 禁止客户端订阅 /smarthome/+/temperature 主题
INSERT INTO mqtt_acl (allow, ipaddr, username, clientid, access, topic) VALUES (0, NULL, NULL, NULL, 1, '/smarthome/+/temperature');

-- 允许客户端订阅包含自身 Client ID 的 /smarthome/${clientid}/temperature 主题
INSERT INTO mqtt_acl (allow, ipaddr, username, clientid, access, topic) VALUES (1, NULL, NULL, NULL, 1, '/smarthome/%c/temperature');
```

------

### 拓展:

emq还支持分布式集群, 共享订阅, 离线消息, 密码加盐等等的功能。