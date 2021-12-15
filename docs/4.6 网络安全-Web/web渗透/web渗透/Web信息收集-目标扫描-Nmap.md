Web信息收集-目标扫描-Nmap

# 一、Nmap简介
Nmap是安全渗透领域最强大的开源端口扫描器，能跨平台支持运行。
官网：Https://nmap.org/
官网：http://sectools.org/

## Centos安装nmap

```bash
rpm -vhU https://nmap.org/dist/nmap-7.80-1.x86_64.rpm
```



# 二、扫描示例
## 主机发现
`nmap -sn 192.168.106/24`

## 端口扫描案例

参考链接：博客园：[nmap](https://home.cnblogs.com/u/nmap/)：[nmap命令-----基础用法](https://www.cnblogs.com/nmap/p/6232207.html)



`nmap-sS -p1-1024 192.168.106.1`



### 端口扫描

B机器使用nmap去扫描A机器，扫描之前，A机器先查看自己上面有哪些端口在被占用

A机器上查看本地ipv4的监听端口



netstat参数解释：

- -l (listen) 仅列出 Listen (监听) 的服务

- -t (tcp) 仅显示tcp相关内容

- -n (numeric) 直接显示ip地址以及端口，不解析为服务名或者主机名

- -p (pid) 显示出socket所属的进程PID 以及进程名字

- --inet 显示ipv4相关协议的监听

查看IPV4端口上的tcp的监听：

```bash
[root@A ~]# netstat   -lntp    --inet
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name  
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      2157/sshd          
tcp        0      0 127.0.0.1:631               0.0.0.0:*                   LISTEN      1930/cupsd         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      2365/master        
tcp        0      0 0.0.0.0:13306               0.0.0.0:*                   LISTEN      21699/mysqld       
tcp        0      0 0.0.0.0:873                 0.0.0.0:*                   LISTEN      2640/rsync         
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      21505/rpcbind    
```

过滤掉监控在127.0.0.1的端口：

```bash
[root@A ~]# netstat   -lntp    --inet | grep -v 127.0.0.1
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name  
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      2157/sshd          
tcp        0      0 0.0.0.0:13306               0.0.0.0:*                   LISTEN      21699/mysqld       
tcp        0      0 0.0.0.0:873                 0.0.0.0:*                   LISTEN      2640/rsync         
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      21505/rpcbind  
```

### 扫描TCP端口

B机器上使用nmap扫描A机器所有端口（-p后面也可以跟空格）

下面表示扫描A机器的1到65535所有在监听的tcp端口。

`nmap 10.0.1.161  -p1-65535`



指定端口范围使用-p参数，如果不指定要扫描的端口，Nmap默认扫描从1到1024再加上nmap-services列出的端口

nmap-services是一个包含大约2200个著名的服务的数据库，Nmap通过查询该数据库可以报告那些端口可能对应于什么服务器，但不一定正确。

所以正确扫描一个机器开放端口的方法是上面命令。-p1-65535

注意，nmap有自己的库，存放一些已知的服务和对应端口号，假如有的服务不在nmap-services，可能nmap就不会去扫描，这就是明明一些端口已经是处于监听状态，nmap默认没扫描出来的原因，需要加入-p参数让其扫描所有端口。

虽然直接使用nmap 10.0.1.161也可以扫描出开放的端口，但是使用-p1-65535 能显示出最多的端口

区别在于不加-p 时，显示的都是已知协议的端口，对于未知协议的端口没显示。



```bash
[root@B ~]# nmap  10.0.1.161  -p1-65535
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:11 CST
Nmap scan report for 10.0.1.161
Host is up (0.00017s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
111/tcp   open  rpcbind
873/tcp   open  rsync
13306/tcp open  unknown
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 2.49 seconds
```



如果不加-p1-65535，对于未知服务的端口（A机器的13306端口）就没法扫描到：

```bash
[root@B ~]# nmap  10.0.1.161
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:12 CST
Nmap scan report for 10.0.1.161
Host is up (0.000089s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 0.43 seconds
```



### 扫描一个IP的多个端口

连续的端口可以使用横线连起来，端口之间可以使用逗号隔开

A机器上再启动两个tcp的监听，分别占用7777和8888端口，用于测试，加入&符号可以放入后台

```bash
[root@A ~]# nc -l 7777&
[1] 21779
[root@A ~]# nc -l 8888&
[2] 21780
```

```bash
[root@B ~]# nmap  10.0.1.161   -p20-200,7777,8888
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:32 CST
Nmap scan report for 10.0.1.161
Host is up (0.00038s latency).
Not shown: 179 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
111/tcp  open  rpcbind
7777/tcp open  cbt
8888/tcp open  sun-answerbook
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 0.17 seconds
```



### 扫描udp端口

先查看哪些ipv4的监听，使用grep -v排除回环接口上的监听

```bash
[root@A ~]# netstat -lnup --inet |grep -v 127.0.0.1
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name  
udp        0      0 0.0.0.0:111                 0.0.0.0:*                               21505/rpcbind      
udp        0      0 0.0.0.0:631                 0.0.0.0:*                               1930/cupsd         
udp        0      0 10.0.1.161:123              0.0.0.0:*                               2261/ntpd          
udp        0      0 0.0.0.0:123                 0.0.0.0:*                               2261/ntpd          
udp        0      0 0.0.0.0:904                 0.0.0.0:*                               21505/rpcbind    
```

- -sU：表示udp scan ， udp端口扫描

- -Pn：不对目标进行ping探测（不判断主机是否在线）（直接扫描端口）

对于udp端口扫描比较慢，扫描完6万多个端口需要20分钟左右

```bash
[root@B ~]# nmap  -sU  10.0.1.161  -Pn
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:16 CST
Stats: 0:12:54 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 75.19% done; ETC: 10:33 (0:04:16 remaining)
Stats: 0:12:55 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 75.29% done; ETC: 10:33 (0:04:15 remaining)
Nmap scan report for 10.0.1.161
Host is up (0.0011s latency).
Not shown: 997 closed ports
PORT    STATE         SERVICE
111/udp open          rpcbind
123/udp open          ntp
631/udp open|filtered ipp
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 1081.27 seconds
```



### 扫描多个IP用法

 中间用空格分开

```bash
[root@B ~]# nmap 10.0.1.161  10.0.1.162
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:18 CST
Nmap scan report for 10.0.1.161
Host is up (0.000060s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap scan report for 10.0.1.162
Host is up (0.0000070s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 2 IP addresses (2 hosts up) scanned in 0.26 seconds
```

也可以采用下面方式逗号隔开

```bash
[root@B ~]# nmap 10.0.1.161,162
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:19 CST
Nmap scan report for 10.0.1.161
Host is up (0.00025s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap scan report for 10.0.1.162
Host is up (0.0000080s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 2 IP addresses (2 hosts up) scanned in 0.81 seconds
```

### 扫描连续的ip地址

```bash
[root@B ~]# nmap 10.0.1.161-162
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:20 CST
Nmap scan report for 10.0.1.161
Host is up (0.00011s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap scan report for 10.0.1.162
Host is up (0.0000030s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 2 IP addresses (2 hosts up) scanned in 0.25 seconds
```

### 扫描一个子网网段所有IP

```bash
[root@B ~]# nmap  10.0.3.0/24
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:21 CST
Nmap scan report for 10.0.3.1
Host is up (0.020s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
23/tcp   open  telnet
6666/tcp open  irc
8888/tcp open  sun-answerbook
 
Nmap scan report for 10.0.3.2
Host is up (0.012s latency).
Not shown: 997 closed ports
PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp filtered ssh
23/tcp open     telnet
 
Nmap scan report for 10.0.3.3
Host is up (0.018s latency).
Not shown: 997 closed ports
PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp filtered ssh
23/tcp open     telnet
 
Nmap done: 256 IP addresses (3 hosts up) scanned in 14.91 seconds
```

### 扫描文件里的IP

如果你有一个ip地址列表，将这个保存为一个txt文件，和namp在同一目录下,扫描这个txt内的所有主机，用法如下

```bash
[root@B ~]# cat ip.txt
10.0.1.161
10.0.1.162
[root@B ~]# nmap -iL ip.txt
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:23 CST
Nmap scan report for 10.0.1.161
Host is up (0.00030s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap scan report for 10.0.1.162
Host is up (0.0000070s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 2 IP addresses (2 hosts up) scanned in 0.68 seconds
```

### 扫描地址段是排除某个IP地址

```bash
[root@B ~]# nmap 10.0.1.161-162  --exclude 10.0.1.162
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:24 CST
Nmap scan report for 10.0.1.161
Host is up (0.0022s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 0.53 seconds
```

### 扫描时排除多个IP地址

排除连续的，可以使用横线连接起来

```bash
[root@B ~]# nmap 10.0.1.161-163   --exclude 10.0.1.162-163
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:25 CST
Nmap scan report for 10.0.1.161
Host is up (0.00023s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
873/tcp open  rsync
MAC Address: 00:0C:29:56:DE:46 (VMware)
 
Nmap done: 1 IP address (1 host up) scanned in 0.56 seconds
```

排除分散的，使用逗号隔开

```bash
[root@B ~]# nmap 10.0.1.161-163 --exclude 10.0.1.161,10.0.1.163
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:27 CST
Nmap scan report for 10.0.1.162
Host is up (0.0000030s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```

### 扫描多个地址时排除文件里的IP地址

把10.0.1.161和10.0.1.163添加到一个文件里，文件名可以随意取

下面扫描10.0.1.161到10.0.1.163 这3个IP地址，排除10.0.1.161和10.0.1.163这两个IP

```bash
[root@B ~]# cat ex.txt
10.0.1.161
10.0.1.163
[root@B ~]# nmap 10.0.1.161-163  --excludefile ex.txt
 
Starting Nmap 5.51 ( http://nmap.org ) at 2016-12-29 10:29 CST
Nmap scan report for 10.0.1.162
Host is up (0.0000050s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
111/tcp open  rpcbind
 
Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```





## 系统扫描
`nmap -O 192.168.106.1`

### osscan-guess 猜测匹配操作系统

`nmap -O --osscan-guess 192.168.1.134`

通过Nmap准确的检测到远程操作系统是比较困难的需要使用到Nmap的猜测功能选项,`–osscan-guess`猜测认为最接近目标的匹配操作系统类型。



## 版本扫描
`nmap -sV 192.168.106.1`
## 综合扫描
`nmap -A 192.168.106.1`

## 脚本扫描

nmap官方脚本文档: https://nmap.org/nsedoc/

`nmap --script 类别`

```bash
/usr/shar/nmap/scripts#
nmap --script=default 192.168.106.1
nmap --script=auth 192.168.106.1
nmap --script=brute 192.168.106.1
nmap --script=vuln 192.168.106.1
nmap --script=broadcast 192.168.106.1
nmap --script=smb-brute.nse 192.168.106.1
nmap --script=smb-check-vulns.nse --script-args=unsafe=1 192.168.106.1
nmap --script=smb-vuln-conficker.nse --script-args=unsafe=1 192.168.106.1
nmap -p3306 --script=mysql-empty-password.nse 192.168.106.1
```



![一款强大的安全扫描器nmap：不老的神器](https://image.3001.net/2017/07/7822ed61bd955f648b1fae0408c09fa02.png)

左侧列出了脚本的分类点击分类 可以看到每一个分类下有很多具体的脚本供我们使用。`nmap --script=类别`这里的类别可以填写下面14大分类中的其中之一也可以填写分类里面的具体漏洞扫描脚本。nmap脚本分类:

\- auth: 负责处理鉴权证书绕开鉴权的脚本   

- broadcast: 在局域网内探查更多服务开启状况如dhcp/dns/sqlserver等服务   
- brute: 提供暴力破解方式针对常见的应用如http/snmp等   
- default: 使用-sC或-A选项扫描时候默认的脚本提供基本脚本扫描能力   
- discovery: 对网络进行更多的信息如SMB枚举、SNMP查询等   
- dos: 用于进行拒绝服务攻击   
- exploit: 利用已知的漏洞入侵系统   
- external: 利用第三方的数据库或资源例如进行whois解析   
- fuzzer: 模糊测试的脚本发送异常的包到目标机探测出潜在漏洞  
- intrusive: 入侵性的脚本此类脚本可能引发对方的IDS/IPS的记录或屏蔽 
- malware: 探测目标机是否感染了病毒、开启了后门等信息   
- safe: 此类与intrusive相反属于安全性脚本   
- version: 负责增强服务与版本扫描Version Detection功能的脚本   
- vuln: 负责检查目标机是否有常见的漏洞Vulnerability如是否有MS08_067



### 使用具体脚本进行扫描

```bash
nmap --script 具体的脚本 www.baidu.com
```

### 常用脚本使用案例

#### 扫描服务器的常见漏洞

```
nmap --script vuln <target>
```

#### 检查FTP是否开启匿名登陆

```
nmap --script ftp-anon <target>
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 1170     924            31 Mar 28  2001 .banner
| d--x--x--x   2 root     root         1024 Jan 14  2002 bin
| d--x--x--x   2 root     root         1024 Aug 10  1999 etc
| drwxr-srwt   2 1170     924          2048 Jul 19 18:48 incoming [NSE: writeable]
| d--x--x--x   2 root     root         1024 Jan 14  2002 lib
| drwxr-sr-x   2 1170     924          1024 Aug  5  2004 pub
|_Only 6 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
```

#### 对MySQL进行暴破解

```bash
nmap --script=mysql-brute <target>
3306/tcp open  mysql
| mysql-brute:
|   Accounts
|     root:root - Valid credentials
```

![一款强大的安全扫描器nmap：不老的神器](https://image.3001.net/2017/07/5bb8e360090ff1da26b303ecf7d5e69e2.png)

可以看出已经暴力成功破解了MySQL,在368秒内进行45061次猜测平均TPS为146.5。

对MySQL进行暴力破解

```bash
nmap -p 1433 --script ms-sql-brute --script-args userdb=customuser.txt,passdb=custompass.txt <host>
| ms-sql-brute:
|   [192.168.100.128\TEST]
|     No credentials found
|     Warnings:
|       sa: AccountLockedOut
|   [192.168.100.128\PROD]
|     Credentials found:
|       webshop_reader:secret => Login Success
|       testuser:secret1234 => PasswordMustChange
|_      lordvader:secret1234 => Login Success
```

#### 对Oracle数据库进行暴破解

```
nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=ORCL <host>
PORT     STATE  SERVICE REASON
1521/tcp open  oracle  syn-ack
| oracle-brute:
|   Accounts
|     system:powell => Account locked
|     haxxor:haxxor => Valid credentials
|   Statistics
|_    Perfomed 157 guesses in 8 seconds, average tps: 19
```

#### 对pgSQL的暴力破解

```
nmap -p 5432 --script pgsql-brute <host>
5432/tcp open  pgsql
| pgsql-brute:
|   root:<empty> => Valid credentials
|_  test:test => Valid credentials
```

#### 对SSH进行暴力破解

```bash
nmap -p 22 --script ssh-brute --script-args userdb=users.lst,passdb=pass.lst --script-args ssh-brute.timeout=4s <target>
22/ssh open  ssh
| ssh-brute:
|  Accounts
|    username:password
|  Statistics
|_   Performed 32 guesses in 25 seconds.
```





## 路由跟踪扫描

路由器追踪功能能够帮网络管理员了解网络通行情况同时也是网络管理人员很好的辅助工具通过路由器追踪可以轻松的查处从我们电脑所在地到目标地之间所经常的网络节点并可以看到通过各个节点所花费的时间

```bash
nmap -traceroute www.baidu.com
```

![一款强大的安全扫描器nmap：不老的神器](https://image.3001.net/2017/07/f2a28889bb2e9cf9998df5be554618052.png)







# 三、Nmap图形化界面-Zenmap

## 3.1 Intense scan
`Nmap -T4 -A -v 192.168.106.1`
 - -T 设置速度登记，1到5级，数字越大，速度越快
 - -A 综合扫描
 - -v 输出扫描过程

## 3.2 Intense scan plus UDP
`Nmap -sS -sU -T4 -A -v 192.168.106.1`
 - -sS TCP全连接扫描
 - -sU UDP扫描

## 3.3 Intense scan,all TCP ports
`Nmap -p 65535 -T4 -A -v 192.168.106.1`
 - -p 指定端口范围，默认扫描1000个端口

## 3.4 Intense scan no ping
`Nmap -T4 -A -v -Pn 192.168.106.1/24`
 - -Pn 不做ping扫描，例如针对防火墙等安全产品

## 3.5 ping scan
`Nmap -sn 192.168.106.1/24`
`Nmap -sn -T4 -v 192.168.106.1/24`
 - -sn 只做ping扫描，不做端口扫描

## 3.6 quick scan
`Nmap -T4 -F 192.168.106.1`
 - -F fast模式，只扫描常见服务端口，比默认端口（1000个）还少

## 3.7 quick scan plus
`Nmap -sV -T4 -O -F --version-light 192.168.106.1`
 - -sV 扫描系统和服务版本
 - -O 扫描操作系统版本

## 3.8 Quick traceroute
`Nmap -sn --traceroute www.baidu.com`

## 3.9 Regular scan
`Nmap www.baidu.com`

## 3.10 slow comprehensive scan
`Nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389 -PU40125 -PY -g 53 --script “default or (discovery and safe)” www.baidu.com`