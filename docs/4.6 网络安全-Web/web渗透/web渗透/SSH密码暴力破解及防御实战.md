
# 一、Hydra（海德拉）
Hydra是世界顶级密码暴力破解工具，支持几乎所有协议的在线密码破解，功能强大，其密码能否被破解关键取决于破解字典是否足够强大，在网络安全渗透过程中是一款必备的测试工具。

## 1.1 指定用户破解
Examples:
```bash
hydra -1 user -P passlist. txt ftp://192.168.0.1
hydra -L userlist.txt -P defaultpw imap://192.168.0.1/PLAIN
hydra -C defaults.txt -6 pop3s://[2001 :db8::1]: 143/TLS:DIGEST-MD5
hydra -1 admin -p password ftp://[192.168.0.0/24]/
hydra -L logins .txt -P pws.txt -M targets.txt ssh
```

`root@kali:~# hydra -1 root -P pass.dic 192.168.106.134 ssh`

# 二、Medusa（美杜莎）
Medusa是一个速度快，支持大规模并行，模块化，爆破登录。可以同时对多个主机，用户或密码执行强力测试。Medusa和Hydra意义，属于在线密码破解工具。不同的是，Medusa的稳定性相较于Hydra好很多，但支持的模块要比Hydra少一些。

## 2.1 语法参数
`Medusa [-h host|-H file] [-u username|-U file] [-p password|-P file] [-C file] -M module [OPT]`

 - -h [TEXT] 目标主机名称或者IP地址
 - -H [FILE] 包含目标主机名称或者IP地址文件
 - -u [TEXT] 测试的用户名
 - -U [FILE] 包含测试的用户名文件
 - -P [TEXT] 测试的密码
 - -P [FILE] 包含测试的密码文件
 - -C [FILE] 组合条目文件
 - -0 [FILE] 日志信息文件
 - -e [n/s/ns] n代表空密码, s代表为密码与用户名相同
 - -M [TEXT] 模块执行名称
 - -m [TEXT] 传递参数到模块
 - -d 显示所有的模块名称
 - -n [NUM] 使用非默认Tcp端口
 - -S 启用SSL
 - -r [NUM] 重试间隔时间,默认为3秒
 - -t [NUM] 设定线程数量
 - -T 同时测试的主机总数
 - -L 并行化,每个用户使用一个线程
 - -f 在任何主机上找到第一个账号/密码后，停止破解
 - -F 在任何主机上找到第一个有效的用户名/密码后停止审计
 - -9 显示模块的使用信息
 - -V [NUM] 详细级别(0-6)
 - -W [NUM] 错误调试级别(0-10 )
 - -V 显示版本
 - -Z [TEXT] 继续扫描_上一-次

## 2.2 破解SSH密码
`root@kali:~# medusa -M ssh -h 192.168.106.134 -U root -P passlist. txt`

# 三、Patator
Patator，强大的命令行暴力破解器。

## 3.1 破解SSH密码
```bash
root@kali:~# patator ssh_ login --help
Patator v0.6 (http://code . google. com/p/patator/)
Usage: ssh_ login <module-options ...> [global-options ...]
Examples:
ssh_ login host=10.0.0.1 user= root password=FILEO 0=pas swords.txt -X
ignore: mesg-' Authentication failed. '
root@kali:~# patator ssh_ login host=192.168. 106.134 user=root password=FILE0 0=passlist . txt
root@kali:~# patator ssh_ login host=192. 168.106.134 user= root password=FILE0 0=passlist.txt \
-X ignore :mesg= ' Authentication failed. '
```

# 四、BrutesPray
BruteSpray是一款基于Nmap扫描输出的gnmap/XMl文件，自动调用Medusa对服务进行爆破。
```
root@kali:~# apt-get update
root@kali:~# apt-get install brutespray
```

## 4.1 语法参数
 - -f FILE, --file FILE 参数后跟一 个文件名，解析nmap输出的GNMAP或者XML文件
 - -o OUTPUT, --output OUTPUT 包含成功尝试的目录下
 - -s SERVICE, --service SERVICE 参数后跟-个服务名，指定要攻击的服务
 - -t THREADS, --threads THREADS参 数后跟一数值， 指定medus a线程数
 - -T HOSTS, --hosts HOSTS K参数后跟一数值,指定同时测试的主机数
 - -U USERLIST, --userlist USERLIST 参数后跟用户字典文件
 - -P PASSLIST, --passlist PASSLIST 参数后跟密码字典文件
 - -u USERNAME, --username USERNAME 参数后跟用户名,指定一个用户名进行爆破
 - -p PASSWORD， --password PASSWORD 参数后跟密码,指定一个密码进行爆破
 - -C，--continuous 成功之后继续爆破
 - -i， -- interactive 交互模式

## 4.2 nmap扫描
```bash
root@kali:~# nmap -V 192.168.106.0/24 -oX nmap 。xml
root@kali:~# nmap -A"-p22 -V 192.168.106.0/24 -oX 22. xml
root@kali:~# nmap -sP 192.168.106.0/24 -oX nmaplive. xml
root@kali:~# nmap -sV -0 192.168.106.0/24 -oX nmap. xml
```

## 4.3 字典爆破
`root@kali :~# brutespray --file 22.xml -U user1ist.txt-p passlist.txt --threads 5 --hosts 5`

# 五、MSF
Metasploit Framework（简称MSF）是一个编写、测试和使用exploit代码的完善环境。这个环境为渗透测试，Shellcode编写和漏洞研究提供了一个可靠的平台，这个框架主要是由面向对象的Perl编程语言编写的，并带有由C语言，汇编程序和Python编写的可选组件。

## 5.1 SSH模块
```bash
root@kali:~# msfconsole
msf > search ssh
```
## 5.2 SSH用户枚举
```bash
msf > use auxiliary/ scanner/ssh/ssh_ enumusers
msf auxiliary(scanner/ssh/ssh_ enumusers) > set phosts 192.168.106.134
msf auxiliary(scanner/ssh/ssh_ enumusers) > set USER_ FILE /root/userlist.txt
msf auxiliary(scanner/ssh/ssh_ enumusers) > run
```
## 5.3 SSH版本探测
```bash
msf > use auxiliary/scanner/ssh/ssh_ version
msf auxiliary( scanner/ssh/ssh_ _version) > set rhosts 192. 168.106.134
msf auxiliary( scanner/ssh/ssh_ _version) > run
```
## 5.4 SSH暴力破解
```bash
msf > use auxiliary/scanner/ssh/ ssh plogin
msf auxiliary( scanner/ssh/ssh_ login) > set rhosts 192. 168.106.134
msf auxiliary(scanner/ssh/ssh_ login) > set USER_ _FILE /root/userlist . txt
msf auxiliary(scanner/ssh/ssh_ login) > set PASS_ _FILE /root/passlist.txt
msf auxiliary(scanner/ssh/ssh_ _login) > run
```
# 六、暴力破解防御
## 6.1 useradd shell [推荐]
```bash
[root@tianyun ~]# useradd yangge -S /sbin/nologin
```
## 6.2 密码的复杂性[推荐]
字母大小写+数字+特殊字符+20位以上+定期更换

## 6.3 修改默认端口[推荐]
```bash
/etc/ssh/sshd_ config
Port 22222
```

## 6.4 限止登录的用户或组[推荐]
```
#PermitRootLogin yes
AllowUser test

[ root@tianyun ~]# man sshd_ config
AllowUsers AllowGroups DenyUsers DenyGroups
```

## 6.5 使用sudu（推荐）

## 6.6 设置允许的IP访问[可选]
```bash
/etc/hosts.allow，例如sshd:192. 168.106.167:allow
```
PAM基于IP限制
`iptables/firewalld`
只能允许从堡垒机

## 6.7 使用DenyHosts自动统计，并将其加入到/etc/hosts . deny

## 6.8 基于PAM实现登录限制[推荐]
模块: pam_ tally2. so
功能:登录统计
示例:实现防止对sshd暴力破解
```bash
[root@tianyun ~]# grep tally2 /etc/pam. d/sshd
auth
required
pam_ tally2.so deny=2 even_ deny_ root root_ unlock_ time=60 unlock_ time=6
```

## 6.9 禁用密码改用公钥方式认证
```bash
/etc/ssh/sshd_ config
Pas swordAuthentication no
```

## 6.10 保护xshel1导出会话文件[小心]