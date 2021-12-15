# 一、简介

Clam AntiVirus是一个类UNIX系统上使用的反病毒软件包。ClamAV是**免费**而且开放源代码的防毒软件，软件与病毒码的更新皆由社群免费发布。目前ClamAV主要是使用在由Linux、FreeBSD等Unix-like系统架设的邮件服务器上，提供电子邮件的病毒扫描服务。ClamAV是一个在命令行下查毒软件，因为它不将杀毒作为主要功能，默认只能查出您计算机内的病毒，但是无法清除。

**# 免费，重点是免费！！！**

# 二、获取

## 1.下载

clamav官网：http://www.clamav.net/downloads 。 点击这里下载，推荐下载最新版本，这里使用的是：最新稳定版0.101.2 。

或者wget直接下载源码包。
wget http://www.clamav.net/downloads/production/clamav-0.101.2.tar.gz

## 2.解压

```shell
tar -zxvf clamav-0.101.2.tar.gz
```

# 三、安装

## 1.编译安装

进入到clamav目录下，看到configure文件后使用以下命令。
过程中存在一定的依赖关系，需要提前准备好相关的文件。离线状态下，安装软件过程中，总是存在各种依赖关系需要解决，着实费劲。提前做好yum源，是最优的解决办法。否则，就要逐步解决其对应的依赖关系。

```shell
cd clamav-0.101.2

./configure --prefix=/usr/local/clamav

make && make install
```

## 2.添加用户组和组成员

```shell
groupadd clamav
useradd -g clamav clamav
```

# 四、配置

## 1.创建日志目录和病毒库目录

```shell
mkdir -p /usr/local/clamav/logs
mkdir -p /usr/local/clamav/updata
```

## 2. 创建日志文件

```shell
touch /usr/local/clamav/logs/clamd.log
touch /usr/local/clamav/logs/freshclam.log
```

## 3. 文件授权

修改文件的属主

```shell
chown clamav:clamav /usr/local/clamav/logs/clamd.log
chown clamav:clamav /usr/local/clamav/logs/freshclam.log
chown clamav:clamav /usr/local/clamav/updata
```

## 4. 修改配置文件

```shell
cp  /usr/local/clamav/etc/clamd.conf.sample /usr/local/clamav/etc/clamd.conf
cp /usr/local/clamav/etc/freshclam.conf.sample /usr/local/clamav/etc/freshclam.conf
```

编辑这两个配置文件内容
复制的两个新文件，几乎是全文注释，只保留了一个醒目的Example，所以也要注释掉Example，增加以下内容。见下文：

文件1：clamd.conf

```shell
vim /usr/local/clamav/etc/clamd.conf

#Example　　//注释掉这一行
#添加以下内容
LogFile /usr/local/clamav/logs/clamd.log
PidFile /usr/local/clamav/updata/clamd.pid
DatabaseDirectory /usr/local/clamav/updata
```

文件2：freshclam.conf

```shell
vim /usr/local/clamav/etc/freshclam.conf

#Example　　//注释掉这一行
#添加以下内容
DatabaseDirectory /usr/local/clamav/updata
UpdateLogFile /usr/local/clamav/logs/freshclam.log
PidFile /usr/local/clamav/updata/freshclam.pid
```

# 五、执行

## 1. 更新病毒库

离线环境是无法更新的，所以需要在线环境中更新。更新好了以后，将核心文件：daily.cvd 、main.cvd 、 Makefile 、 [Makefile.am](http://makefile.am/) 、 Makefile.in等复制到病毒库文件夹下。一般为：
…/clamav-0.101.2/database
如果条件可以（可以连接互联网的情况下），使用命令直接更新病毒库。
更新病毒库的命令：

```shell
/usr/local/clamav/bin/freshclam
```

## 2.扫描

```shell
参数：
-r 递归扫描子目录
-i 只显示发现的病毒文件
–no-summary 不显示统计信息

用法：
--帮助
/usr/local/clamav/bin/clamscan --help     

--默认扫描当前目录下的文件，并显示扫描结果统计信息            
/usr/local/clamav/bin/clamscan

--扫描当前目录下的所有目录和文件，并显示结果统计信息　　 　                
/usr/local/clamav/bin/clamscan -r　

--扫描data目录下的所有目录和文件，并显示结果统计信息　                 
/usr/local/clamav/bin/clamscan -r /data　 

--扫描data目录下的所有目录和文件，只显示有问题的扫描结果            
/usr/local/clamav/bin/clamscan -r --bell -i /data  

--扫描data目录下的所有目录和文件，不显示统计信息  
/usr/local/clamav/bin/clamscan --no-summary -ri /data 
```

## 3.杀毒

所谓的杀毒，就是将扫描出来的文件删除，就完成了杀毒工作。
（以下命令均扫描当前目录下的文件和目录）

```shell
/usr/local/clamav/bin/clamscan -r --remove #删除扫描过程中的发现的病毒文件
/usr/local/clamav/bin/clamscan -r --bell -i #扫描过程中发现病毒发出警报声
/usr/local/clamav/bin/clamscan -r --move [路径] #扫描并将发现的病毒文件移动至对应的路径下
/usr/local/clamav/bin/clamscan -r --infected -i #扫描显示发现的病毒文件，一般文件后面会显示FOUND
```

使用第一条命令的时候要特别注意，如果病毒感染力系统的核心区域，该命令直接会删除系统核心文件，严重者会导致系统崩溃，所以要慎重，最好是先确认好病毒的位置后，再执行删除命令。

## 4. 自动定时更新和杀毒

一般使用计划任务，让服务器每天定时更新和定时杀毒，保存杀毒日志。设置crontab定时任务。

```shell
1  3  * * *          /usr/local/clamav/bin/freshclam --quiet
20 3  * * *          /usr/local/clamav/bin/clamscan  -r /home  --remove -l /var/log/cla
```

## 5. 案例

使用命令扫描

```shell
/usr/local/clamav/bin/clamscan -r
```

结果如下：

```shell
----------- SCAN SUMMARY -----------     #扫描摘要
Known viruses: 6377069                   #已知病毒：6377069
Engine version: 0.99.2                   #引擎版本：0.92.2
Scanned directories: 18186               #扫描目录：18186
Scanned files: 80762                     #扫描文件：80762
Infected files: 46                        #感染档案：46
Total errors: 4253                       #总误差：4253
Data scanned: 4717.23 MB                 #数据扫描：4717.23兆字节
Data read: 9475.00 MB (ratio 0.50:1)     #数据读取：9475MB（比0.50∶1）
Time: 1939.667 sec (32 m 19 s)           #时间：1939.667秒（32分19秒）
--------------------- 
```

# 六、小结

## 1、离线环境下的使用

①在一台在线的服务器上，按照上述步骤，安装好clamav软件；
②使用以下命令获取最新的病毒库；

```shell
/usr/local/clamav/bin/freshclam
```

③打开对应位置，将病毒库传输到本地；

```shell
cd /usr/local/clamav/update/
```

④将病毒库上传到离线环境中，放置到对应的文件夹底下；

⑤完成以上步骤就可以在离线服务器上扫描了。

## 2、离线安装clamav时的依赖关系

依赖关系主要是gcc和g++，提前安装好就行。

```shell
yum -y install gcc
yum -y install gcc-c++
```

离线环境中，可以将两个安装包上传到系统中，然后分别编译安装。

```shell
./configure

make && make install
```

可以本地制作一个yum源，使用yum解决依赖关系吧！时间充足的话，可以慢慢琢磨。