- [LVS+Keepalive双机热备](https://www.cnblogs.com/xiao987334176/p/13094763.html)



# 一、概述

本实验基于CentOS7.6  操作系统，总共5台设备，两台做后端web服务器，两台做lvs和keepalived，一台做客户机，实验以LVS(DR)+Keepalived和LVS(NAT)+Keepalived两种模式来做双机热备份，实验环境拓扑如下图所示：

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200611172139716-1035486064.png)

 

 

从架构可以看出来，从用户的角度来说，会直接访问192.168.31.200，也就是说不管系统如何设计要保证此ip的可用性，

从架构师的角度考虑：

  用户访问192.168.31.200，这是一个虚拟iP，用户不关心内部如何协调。

我们使用 192.168.31.54 作为主机 Master机器，然后使用keepalived 技术配置 HA(high avilable)配置高可用行，也就是说如果 分发的机器Master宕机了，keepalive会自动转到 backup机器，

这就是HA配置，保证master即使宕机了，也不影响转发；

master机器负责把用户的请求转发到  真实的机器， web1和web2,他们会按照一定的轮训机制，访问，如果web1宕机，master会自动转发到web2;

我们在两台负载均衡的机器上面，配置keepalived保证分发机器的高可用行HA；

## 环境说明

系统的 IP配置如下：

| 服务器名        | 主机名 | IP地址         | 虚拟设备名 | 虚拟IP         |
| --------------- | ------ | -------------- | ---------- | -------------- |
| Director Server | lvs1   | 192.168.31.54  | enp0s3:0   | 192.168.31.200 |
| Backup Server   | lvs2   | 192.168.31.51  | enp0s3:0   | 192.168.31.200 |
| Real Server1    | web1   | 192.168.31.113 | lo:0       | 192.168.31.200 |
| Real Server2    | web2   | 192.168.31.150 | lo:0       | 192.168.31.200 |

 

请确保4台服务器关闭了防火墙

```
systemctl stop firewalld
setenforce 0
```

 

# 二、web服务器配置

## 安装nginx

**登录主机web1和web2**

安装nginx

```
# 安装Nginx源
yum -y install epel-release
# 安装Nginx
yum -y install nginx
# 启动Nginx服务
systemctl start nginx
# 加入开机自启
systemctl enable nginx
# 备份原有默认页面
mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html_bak
```

 

**登录主机web1**，修改默认页面

```
vi /usr/share/nginx/html/index.html
```

清空内容，完整内容如下：

```
This is Server 111
```

 

**登录主机web2**，修改默认页面

```
vi /usr/share/nginx/html/index.html
```

清空内容，完整内容如下：

```
This is Server 222
```

 

## 访问页面

访问web1

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200611175601415-1328222622.png)

 

 

访问web2

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200611175619652-250688665.png)

# 三、LVS+Keepalive

## 安装

**登录主机lvs1和lvs2**

安装keepalived和lvs管理工具

```
yum -y install keepalived* ipvsadm
```

加载内核模块

```
modprobe ip_vs
```

备份配置文件

```
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf_bak
```

 

## 配置

**登录主机lvs1**

```
vi /etc/keepalived/keepalived.conf
```

内容如下：

```
global_defs {
    router_id LVS_TEST    #服务器名字
}

vrrp_instance VI_1 {
    state MASTER    #配置主备，备用机此配置项为BACKUP
    interface enp0s3    #指定接口
    virtual_router_id 51    #指定路由ID，主备必须一样
    priority 101    #设置优先级，主略高于备份
    advert_int 1    #设置检查时间
    authentication {
        auth_type PASS    #设置验证加密方式
        auth_type 1234    #设置验证密码
    }
    virtual_ipaddress {
        192.168.31.200
    }
}

virtual_server 192.168.31.200 80 {
    delay_loop 3    #健康检查时间
    lb_algo rr    #LVS调度算法
    lb_kind DR   #LVS工作模式
    !persistence 60    #是否保持连接，！不保持
    protocol TCP    #服务采用TCP协议
    real_server 192.168.31.113 80 {
        weight 1    #权重
        TCP_CHECK {    #TCP检查
            connect_port 80   #检查端口80
            connect_timeout 3    #超时时间3秒
            nb_get_retry 3    #重试次数3次
            delay_before_retry 4    #重试间隔4秒
        }
    }
    real_server 192.168.31.150 80 {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 4
        }
    }
}
```

请根据实际情况修改红色部分

说明：

interface enp0s3 表示本地网卡为enp0s3

192.168.31.200 表示虚拟ip

 

重启keepalived服务

```
systemctl restart keepalived
systemctl enable keepalived
```

 

验证虚拟IP是否生效

```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:04:34:88 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.54/24 brd 192.168.31.255 scope global noprefixroute dynamic enp0s3
       valid_lft 31218sec preferred_lft 31218sec
    inet 192.168.31.200/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a66d:efde:f19f:9882/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

可以发现虚拟ip已经有了

 

**登录主机lvs2**

```
vi /etc/keepalived/keepalived.conf
```

LVS2的配置同LVS1，只需将配置文件中下面两处修改即可

内容如下：

```
global_defs {
    router_id LVS_TEST    #服务器名字
}

vrrp_instance VI_1 {
    state BACKUP    #配置主备，备用机此配置项为BACKUP
    interface enp0s3    #指定接口
    virtual_router_id 51    #指定路由ID，主备必须一样
    priority 100    #设置优先级，主略高于备份
    advert_int 1    #设置检查时间
    authentication {
        auth_type PASS    #设置验证加密方式
        auth_type 1234    #设置验证密码
    }
    virtual_ipaddress {
        192.168.31.200
    }
}

virtual_server 192.168.31.200 80 {
    delay_loop 3    #健康检查时间
    lb_algo rr    #LVS调度算法
    lb_kind DR   #LVS工作模式
    !persistence 60    #是否保持连接，！不保持
    protocol TCP    #服务采用TCP协议
    real_server 192.168.31.113 80 {
        weight 1    #权重
        TCP_CHECK {    #TCP检查
            connect_port 80   #检查端口80
            connect_timeout 3    #超时时间3秒
            nb_get_retry 3    #重试次数3次
            delay_before_retry 4    #重试间隔4秒
        }
    }
    real_server 192.168.31.150 80 {
        weight 1
        TCP_CHECK {
            connect_port 80
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 4
        }
    }
}
```

 

重启keepalived服务

```
systemctl restart keepalived
systemctl enable keepalived
```

 

查看虚拟IP

```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d4:0e:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.51/24 brd 192.168.31.255 scope global noprefixroute dynamic enp0s3
       valid_lft 31079sec preferred_lft 31079sec
    inet6 fe80::8cdb:32f0:4bbd:9294/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    inet6 fe80::a66d:efde:f19f:9882/64 scope link tentative noprefixroute dadfailed 
       valid_lft forever preferred_lft forever
```

注意：此时是没有的。因为master和bakcup只能同时存在一个。只有当master异常时，bakcup才会出现虚拟ip。

 

**登录主机lvs1**，关掉主服务器的keepalived服务

```
systemctl stop keepalived
```

 

**登录主机lvs2**，验证备份的keepalived是否生效

```
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d4:0e:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.51/24 brd 192.168.31.255 scope global noprefixroute dynamic enp0s3
       valid_lft 30713sec preferred_lft 30713sec
    inet 192.168.31.200/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::8cdb:32f0:4bbd:9294/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    inet6 fe80::a66d:efde:f19f:9882/64 scope link tentative noprefixroute dadfailed 
       valid_lft forever preferred_lft forever
```

 

# 四、web服务器绑定VIP

 2台web服务器为lo:0绑定VIP地址、抑制ARP广播

**登录主机web1，web2**，编写以下脚本文件realserver.sh

```
vi /etc/init.d/realserver.sh 
```

内容如下：

```
#!/bin/bash
#description: Config realserver

VIP=192.168.31.200

#/etc/rc.d/init.d/functions

case "$1" in
start)
       /sbin/ifconfig lo:0 $VIP netmask 255.255.255.255 broadcast $VIP
       /sbin/route add -host $VIP dev lo:0
       echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
       sysctl -p >/dev/null 2>&1
       echo "RealServer Start OK"
       ;;
stop)
       /sbin/ifconfig lo:0 down
       /sbin/route del $VIP >/dev/null 2>&1
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
       echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
       echo "RealServer Stoped"
       ;;
*)
       echo "Usage: $0 {start|stop}"
       exit 1
esac

exit 0
```

请根据实际情况，修改VIP地址。

 

**登录主机web1，web2，**分别执行脚本

```
bash /etc/init.d/realserver.sh
```

 

执行完了之后，验证一下 使用ifconfig 你会发现 回环地址的网卡会多出一个lo:0的 网卡；

```
# ifconfig lo:0
lo:0: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 192.168.31.200  netmask 255.255.255.255
        loop  txqueuelen 1000  (Local Loopback)
```

 

登录lvs1，查看VIP是否成功映射

```
# ipvsadm -L
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  MiWiFi-R3P-srv:http rr
  -> 192.168.31.113:http          Route   1      0          0         
  -> 192.168.31.150:http          Route   1      0          0  
```

 

#  五、测试可用性

## 测试keepalived的监控检测

访问虚拟ip

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200612092551622-1430394738.png)

 

可以看到，现在是访问的web1 

 

登录主机web1，关闭nginx进程。

```
nginx -s stop
```

 

刷新页面

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200612092654508-1208735492.png)

 

 可以发现切换到web2了。

 

登录主机web1，启动Nginx

```
nginx
```

 

刷新页面

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200612092913091-473828793.png)

 

 发现切换到web1了。

 

## 测试 keepalived 的HA特性

登录主机lvs1，此时的vip是在master节点上的。关闭keepalived服务

```
systemctl stop keepalived
```

 

查看vip，发现已经没有了

```
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:04:34:88 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.54/24 brd 192.168.31.255 scope global noprefixroute dynamic enp0s3
       valid_lft 42290sec preferred_lft 42290sec
    inet6 fe80::a66d:efde:f19f:9882/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

 

登录主机lvs2，查看vip，发现已经出现了。

```
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:d4:0e:fb brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.51/24 brd 192.168.31.255 scope global noprefixroute dynamic enp0s3
       valid_lft 42248sec preferred_lft 42248sec
    inet 192.168.31.200/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::8cdb:32f0:4bbd:9294/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    inet6 fe80::a66d:efde:f19f:9882/64 scope link tentative noprefixroute dadfailed 
       valid_lft forever preferred_lft forever
```

 

此时，刷新页面，依然访问正常。

![img](https://img2020.cnblogs.com/blog/1341090/202006/1341090-20200612093253948-1533756832.png)

 

 说明keepalived的HA特性是正常的。

 

 

本文参考链接：

https://blog.csdn.net/weixin_42342456/article/details/86100090

https://www.cnblogs.com/aspirant/p/6740556.html