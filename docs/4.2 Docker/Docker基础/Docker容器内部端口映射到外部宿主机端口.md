[TOC]

博客园：散尽浮华：[Docker容器内部端口映射到外部宿主机端口 - 运维笔记](https://www.cnblogs.com/kevingrace/p/9453987.html)

Docker允许通过外部访问容器或者容器之间互联的方式来提供网络服务。
容器启动之后，容器中可以运行一些网络应用，通过-p或-P参数来指定端口映射。
注意：
 - 宿主机的一个端口只能映射到容器内部的某一个端口上，比如：8080->80之后，就不能8080->81
 - 容器内部的某个端口可以被宿主机的多个端口映射,比如：8080->80，8090->80,8099->80

# 1、启动容器时，选择一个端口映射到容器内部开放端口上
 1. -p 小写p表示docker会选择一个具体的宿主机端口映射到容器内部开放的网络端口上。
 2. -P 大写P表示docker会随机选择一个宿主机端口映射到容器内部开放的网络端口上。
比如：
```bash
[root@docker-test ~]# docker run -ti -d --name my-nginx -p 8088:80 docker.io/nginx
2218c7d88ccc917fd0aa0ec24e6d81667eb588f491d3730deb09289dcf6b8125
[root@docker-test ~]# docker run -ti -d --name my-nginx2 -P docker.io/nginx
589237ceec9d5d1de045a5395c0d4b519acf54e8c09afb07af49de1b06d71059
[root@docker-test ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                   NAMES
589237ceec9d        docker.io/nginx     "nginx -g 'daemon ..."   6 seconds ago        Up 5 seconds        0.0.0.0:32770->80/tcp   my-nginx2
2218c7d88ccc        docker.io/nginx     "nginx -g 'daemon ..."   About a minute ago   Up About a minute   0.0.0.0:8088->80/tcp    my-nginx
```
由上面可知：
容器my-nginx启动时使用了-p，选择宿主机具体的8088端口映射到容器内部的80端口上了，访问http://localhost/8088即可
容器my-nginx2启动时使用了-P，选择宿主机的一个随机端口映射到容器内部的80端口上了，这里随机端口是32770，访问http://localhost/32770 即可。

# 2、启动创建时，绑定外部的ip和端口（宿主机ip是192.168.10.214）
```bash
[root@docker-test ~]# docker run -ti -d --name my-nginx3 -p 127.0.0.1:8888:80 docker.io/nginx
debca5ec7dbb770ca307b06309b0e24b81b6bf689cb11474ec1ba187f4d7802c
[root@docker-test ~]# docker run -ti -d --name my-nginx4 -p 192.168.10.214:9999:80 docker.io/nginx
ba72a93196f7e55020105b90a51d2203f9cc4d09882e7848ff72f9c43d81852a
[root@docker-test ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                         NAMES
ba72a93196f7        docker.io/nginx     "nginx -g 'daemon ..."   2 seconds ago       Up 1 second         192.168.10.214:9999->80/tcp   my-nginx4
debca5ec7dbb        docker.io/nginx     "nginx -g 'daemon ..."   3 minutes ago       Up 3 minutes        127.0.0.1:8888->80/tcp        my-nginx3
```
由上面可知：
容器my-nginx3绑定的宿主机外部ip是127.0.0.1，端口是8888，则访问http://127.0.0.1:8888或http://localhost:8888都可以，访问http://192.168.10.214:8888就会拒绝！
容器my-nginx4绑定的宿主机外部ip是192.168.10.214，端口是9999，则访问http://192.168.10.214:9999就可以，访问http://127.0.0.1:9999或http://localhost:9999就会拒绝！

# 3、容器启动时可以指定通信协议，比如tcp、udp
```bash
[root@docker-test ~]# docker run -ti -d --name my-nginx5 -p 8099:80/tcp docker.io/nginx
c08eb29e3c0a46386319b475cc95245ccfbf106ed80b1f75d104f8f05d0d0b3e
[root@docker-test ~]# docker run -ti -d --name my-nginx6 -p 192.168.10.214:8077:80/udp docker.io/nginx
992a48cbd3ef0e568b45c164c22a00389622c2b49e77f936bc0f980718590d5b
[root@docker-test ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                 NAMES
992a48cbd3ef        docker.io/nginx     "nginx -g 'daemon ..."   3 seconds ago       Up 2 seconds        80/tcp, 192.168.10.214:8077->80/udp   my-nginx6
c08eb29e3c0a        docker.io/nginx     "nginx -g 'daemon ..."   53 seconds ago      Up 51 seconds       0.0.0.0:8099->80/tcp                  my-nginx5
```

# 4、查看容器绑定和映射的端口及Ip地址
```bash
[root@docker-test ~]# docker port my-nginx5
80/tcp -> 0.0.0.0:8099
[root@docker-test ~]# docker inspect my-nginx5|grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.6",
                    "IPAddress": "172.17.0.6",
```

# 5、容器启动绑定多IP和端口（跟多个-p）
```bash
[root@docker-test ~]# docker run -ti -d --name my-nginx8 -p 192.168.10.214:7777:80 -p 127.0.0.1:7788:80 docker.io/nginx
0e86be91026d1601b77b52c346c44a31512138cedc7f21451e996dd2e75d014d
[root@docker-test ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                 NAMES
0e86be91026d        docker.io/nginx     "nginx -g 'daemon ..."   17 seconds ago      Up 15 seconds       127.0.0.1:7788->80/tcp, 192.168.10.214:7777->80/tcp   my-nginx8
```

# 6、容器除了在启动时添加端口映射关系，还可以通过宿主机的iptables进行nat转发，将宿主机的端口映射到容器的内部端口上，这种方式适用于容器启动时没有指定端口映射的情况！
```bash
[root@docker-test ~]# docker run -ti -d --name my-nginx9 docker.io/nginx
990752e39d75b977cbff5a944247366662211ce43d16843a452a5697ddded12f
[root@docker-test ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS            NAMES
990752e39d75        docker.io/nginx     "nginx -g 'daemon ..."   2 seconds ago       Up 1 second         80/tcp           my-nginx9
```
这个时候，由于容器my-nginx9在启动时没有指定其内部的80端口映射到宿主机的端口上，所以默认是没法访问的！
现在通过宿主机的iptables进行net转发

首先获得容器的ip地址：
```bash
[root@docker-test ~]# docker inspect my-nginx9|grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.9",
                    "IPAddress": "172.17.0.9",

[root@docker-test ~]# ping 172.17.0.9
PING 172.17.0.9 (172.17.0.9) 56(84) bytes of data.
64 bytes from 172.17.0.9: icmp_seq=1 ttl=64 time=0.105 ms
64 bytes from 172.17.0.9: icmp_seq=2 ttl=64 time=0.061 ms
.....

[root@docker-test ~]# telnet 172.17.0.9 80
Trying 172.17.0.9...
Connected to 172.17.0.9.
Escape character is '^]'
```
centos7下部署iptables环境纪录（关闭默认的firewalle）
参考：http://www.cnblogs.com/kevingrace/p/5799210.html

将容器的80端口映射到dockers宿主机的9998端口:
```bash
[root@docker-test ~]# iptables -t nat -A PREROUTING -p tcp -m tcp --dport 9998 -j DNAT --to-destination 172.17.0.9:80
[root@docker-test ~]# iptables -t nat -A POSTROUTING -d 172.17.0.9/32 -p tcp -m tcp --sport 80 -j SNAT --to-source 192.16.10.214
[root@docker-test ~]# iptables -t filter -A INPUT -p tcp -m state --state NEW -m tcp --dport 9998 -j ACCEPT
```
保存以上iptables规则
```bash
[root@docker-test ~]# iptables-save > /etc/sysconfig/iptables
```
查看/etc/sysconfig/iptables文件，注意下面两行有关icmp-host-prohibited的设置一定要注释掉！否则nat转发会失败！
```bash
[root@docker-test ~]# cat /etc/sysconfig/iptables
# Generated by iptables-save v1.4.21 on Fri Aug 10 11:13:57 2018
*nat
:PREROUTING ACCEPT [32:1280]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A PREROUTING -p tcp -m tcp --dport 9998 -j DNAT --to-destination 172.17.0.9:80
-A POSTROUTING -d 172.17.0.9/32 -p tcp -m tcp --sport 80 -j SNAT --to-source 192.16.10.214
COMMIT
# Completed on Fri Aug 10 11:13:57 2018
# Generated by iptables-save v1.4.21 on Fri Aug 10 11:13:57 2018
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [50:5056]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 9998 -j ACCEPT
#-A INPUT -j REJECT --reject-with icmp-host-prohibited
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# Completed on Fri Aug 10 11:13:57 2018
```

最后重启iptbales服务
```bash
[root@docker-test ~]# systemctl restart iptables
```
查看iptables规则
```bash
[root@docker-test ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:distinct32

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

[root@docker-test ~]# iptables -L -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DNAT       tcp  --  anywhere             anywhere             tcp dpt:distinct32 to:172.17.0.9:80

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
SNAT       tcp  --  anywhere             172.17.0.9           tcp spt:http to:192.16.10.214
```
然后访问http://192.168.10.214:9998/，就能转发访问到my-nginx9容器的80端口了！！！

# 7、一次性删除所有容器，包括正在运行的容器
```bash
[root@docker-test ~]# docker rm -f `docker ps -a -q`
990752e39d75
0e86be91026d
ff2bc46a8ee4
c08eb29e3c0a
ba72a93196f7
debca5ec7dbb
589237ceec9d
2218c7d88ccc
```

# 8、Docker 端口映射到宿主机后, 外部无法访问对应宿主机端口
创建docker容器的时候,做了端口映射到宿主机, 防火墙已关闭, 但是外部始终无法访问宿主机端口?
这种情况基本就是因为宿主机没有开启ip转发功能，从而导致外部网络访问宿主机对应端口是没能转发到 Docker Container 所对应的端口上。

解决办法:
Linux 发行版默认情况下是不开启 ip 转发功能的。这是一个好的做法，因为大多数人是用不到 ip 转发的，但是如果架设一个 Linux 路由或者VPN服务我们就需要开启该服务了。

在 Linux 中开启 ip 转发的内核参数为：net.ipv4.ip_forward，查看是否开启 ip转发：
```bash
# cat /proc/sys/net/ipv4/ip_forward           // 0：未开启，1：已开启
```
打开ip转发功能, 下面两种方法都是临时打开ip转发功能!
```bash
# echo 1 > /proc/sys/net/ipv4/ip_forward
# sysctl -w net.ipv4.ip_forward=1
```
永久生效的ip转发:
```bash
# vim /etc/sysctl.conf
net.ipv4.ip_forward = 1

# sysctl -p /etc/sysctl.conf      // 立即生效
```
Linux 系统中也可以通过重启网卡来立即生效 (修改sysctl.conf文件后的生效)
```bash
# systemctl restart network              //CentOS 7
```