允许特定客户端 ip 地址连接

```mysql
mysql> GRANT ALL ON *.* to root@'192.168.1.4' IDENTIFIED BY 'your-root-password'; 

mysql> FLUSH PRIVILEGES;
```

配成识别ip网段

```mysql
GRANT ALL ON *.* to root@'119.143.12..%' IDENTIFIED BY 'password'; 
```

%表示通配的意思