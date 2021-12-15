- [Mysql 中的SSL 连接](https://www.cnblogs.com/plluoye/p/11182945.html)
- [Mysql使用SSL连接](https://www.cnblogs.com/maggieq8324/p/11414889.html)
- [Mysql启用SSL以及JDBC连接Mysql配置](https://blog.csdn.net/qq_24832959/article/details/103239240)
- [JDBC通过SSL方式连接MySQL](https://blog.csdn.net/loiterer_y/article/details/106646667)



# 一、使用 OpenSSL 创建 SSL 证书和私钥

- 根据自己的操作系统下载Win(xx)OpenSSL安装
- 新建一个目录用于存放生成的证书和私钥

```bash
//生成一个 CA 私钥
openssl genrsa 2048 > cert/ca-key.pem
//使用私钥生成一个新的数字证书,执行这个命令时, 会需要填写一些问题, 随便填写就可以了
openssl req -sha1 -new -x509 -nodes -days 3650 -key ./cert/ca-key.pem > cert/ca-cert.pem
//创建服务器端RSA 私钥和数字证书,这个命令会生成一个新的私钥(server-key.pem), 同时会使用这个新私钥来生成一个证书请求文件(server-req.pem).
//这个命令同样需要回答几个问题, 随便填写即可. 不过需要注意的是, A challenge password 这一项需要为空
openssl req -sha1 -newkey rsa:2048 -days 3650 -nodes -keyout cert/server-key.pem > cert/server-req.pem
//将生成的私钥转换为 RSA 私钥文件格式
openssl rsa -in cert/server-key.pem -out cert/server-key.pem
//使用原先生成的 CA 证书来生成一个服务器端的数字证书
openssl x509 -sha1 -req -in cert/server-req.pem -days 3650 -CA cert/ca-cert.pem -CAkey cert/ca-key.pem -set_serial 01 > cert/server-cert.pem
//创建客户端的 RSA 私钥和数字证书
openssl req -sha1 -newkey rsa:2048 -days 3650 -nodes -keyout cert/client-key.pem > cert/client-req.pem
//将生成的私钥转换为 RSA 私钥文件格式
openssl rsa -in cert/client-key.pem -out cert/client-key.pem
//为客户端创建一个数字证书
openssl x509 -sha1 -req -in cert/client-req.pem -days 3650 -CA cert/ca-cert.pem -CAkey cert/ca-key.pem -set_serial 01 > cert/client-cert.pem
```

## 1.1 SSL 配置

在前面的步骤中, 我们已经生成了8个文件, 分别是:

- ca-cert.pem: CA 证书, 用于生成服务器端/客户端的数字证书.
- ca-key.pem: CA 私钥, 用于生成服务器端/客户端的数字证书.
- server-key.pem: 服务器端的 RSA 私钥
- server-req.pem: 服务器端的证书请求文件, 用于生成服务器端的数字证书.
- server-cert.pem: 服务器端的数字证书.
- client-key.pem: 客户端的 RSA 私钥
- client-req.pem: 客户端的证书请求文件, 用于生成客户端的数字证书.
- client-cert.pem: 客户端的数字证书.



# 二、查看数据库是否支持 SSL

首先在 MySQL 上执行如下命令, 查询是否 MySQL 支持 SSL:

```mysql
mysql> SHOW VARIABLES LIKE 'have_ssl';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| have_ssl      | YES   |
+---------------+-------+
1 row in set (0.02 sec)
```

当 have_ssl 为 YES 时, 表示此时 MySQL 服务已经支持 SSL 了. 如果是 DESABLE, 则需要在启动 MySQL 服务时, 使能 SSL 功能.



## 2.1 设置MySQL配置文件，开启SSL验证

vi my.cnf

```yaml
# 在mysqld下面添加如下配置
[mysqld]
require_secure_transport = ON
```

## 2.2 配置MySQL的SSL

接下来我们就需要分别配置服务器端和客户端：

- 服务器端配置
  服务器端需要用到三个文件, 分别是: CA 证书, 服务器端的 RSA 私钥, 服务器端的数字证书, 我们需要在 [mysqld] 配置域下添加如下内容:

```bash
[mysqld]
ssl-ca=/etc/mysql/ca-cert.pem
ssl-cert=/etc/mysql/server-cert.pem
ssl-key=/etc/mysql/server-key.pem
```

接着我们还可以更改 bind-address, 使 MySQL 服务可以接收来自所有 ip 地址的客户端, 即:

```bash
bind-address = *
```

当配置好后, 我们需要重启 MySQL 服务
最后一步, 我们添加一个需要使用 SSL 才可以登录的帐号, 来验证一下我们所配置的 SSL 是否生效:

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'ssl_test'@'%' IDENTIFIED BY 'ssl_test' REQUIRE SSL;
FLUSH PRIVILEGES;
```

- 查看用户是否使用ssl

```mysql
SELECT ssl_type From mysql.user Where user="ssler"
```

ssl_type为空字符串，表示该用户不强制要求使用ssl连接。

- 配置用户必须使用ssl连接

```mysql
ALTER USER 'ssler'@'%' REQUIRE SSL;
FLUSH PRIVILEGES
```

此时再执行`SELECT ssl_type From mysql.user Where user="ssler"`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200609171648616.png)
`ANY`表示必须使用ssl连接。



当配置好后, 使用 root 登录 MySQL

```mysql
mysql --ssl-ca="D:/Program Files/OpenSSL-Win64/bin/cert/ca-cert.pem" --ssl-cert="D:/Program Files/OpenSSL-Win64/bin/cert/client-cert.pem" --ssl-key="D:/Program Files/OpenSSL-Win64/bin/cert/client-key.pem"  -u coisini -p
```

当连接成功后, 我们执行 show variables like '%ssl%' 语句会有如下输出:

```mysql
mysql> show variables like '%ssl%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| have_openssl  | YES             |
| have_ssl      | YES             |
| ssl_ca        | ca.pem          |
| ssl_capath    |                 |
| ssl_cert      | server-cert.pem |
| ssl_cipher    |                 |
| ssl_crl       |                 |
| ssl_crlpath   |                 |
| ssl_key       | server-key.pem  |
+---------------+-----------------+
9 rows in set (0.01 sec)
```



# 三、JAVA-JDBC配置

使用jdk自带的keytool导入mysql的客户端证书到密钥仓库，并生成密钥文件。

根据上文查到的ca.pem，将其复制到目标主机上，然后执行下述指令



- 使用该命令生成java使用SSL连接所需的文件：

```bash
keytool -importcert -alias MySQLCACert -file "D:\Program Files\OpenSSL-Win64\bin\cert\ca-cert.pem" -keystore truststore -storepass 密码
```

- 通过指令验证证书是否导入

```bash
$ keytool -list -keystore mysql.ks
输入密钥库口令:
密钥库类型: jks
密钥库提供方: SUN

您的密钥库包含 1 个条目

mysql, 2020-6-9, trustedCertEntry,
证书指纹 (SHA1): 6B:EE:FE:B4:74:89:A3:88:6C:49:22:44:6D:FB:88:DE:18:6A:7A:F6
```



- 将生成的文件配置系统环境变量

```bash
名：JAVA_OPTS 
值：-Djavax.net.ssl.trustStore="上一步中生成文件的本地路径" -Djavax.net.ssl.trustStorePassword="密码"
```

- JDBC配置连接

```bash
##jdbc.properties:
yxaq.dz=jdbc:mysql://127.0.0.1:3306/yxaqgl?verifyServerCertificate=true&useSSL=true&requireSSL=true
```