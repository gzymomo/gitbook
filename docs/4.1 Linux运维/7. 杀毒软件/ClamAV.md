### 一、介绍

`Clam AntiVirus` 是一款 UNIX 下开源的 (GPL) 反病毒工具包，专为邮件网关上的电子邮件扫描而设计。该工具包提供了包含灵活且可伸缩的监控程序、命令行扫描程序以及用于自动更新数据库的高级工具在内的大量实用程序。该工具包的核心在于可用于各类场合的反病毒引擎共享库。
 **主要使用ClamAV开源杀毒引擎检测木马、病毒、恶意软件和其他恶意的威胁**

#### 1.1、高性能

`ClamAV`包括一个多线程扫描程序守护程序，用于按需文件扫描和自动签名更新的命令行实用程序。

#### 1.2、格式支持

```
ClamAV`支持多种文件格式，文件和存档解包以及多种签名语言。`PDF、JS、XLS、DOCX、PPT等
```

### 二、ClamAV安装

#### 2.1、安装

`CentOS` 上安装，`clamav`包需要EPEL存储库

```bash
yum install -y epel-release
yum install -y clamav
```

#### 2.2、更新

为防止蠕虫传播，必须经常检查更新，`ClamAV`用户需要经常执行`freshclam`，检查间隔为30分钟。由于`ClamAV`用户数量过大，托管病毒数据库文件的服务器很容易过载。如果直接执行`freshclam`从公网更新会很慢，可以通过搭建私有镜像源进行内网分发
 **默认更新**

```bash
cat /etc/cron.d/clamav-update  
##每三个小时执行更新
0  */3 * * * root /usr/share/clamav/freshclam-sleep
##更新病毒库
freshclam 
##病毒库文件
/var/lib/clamav/daily.cvd
/var/lib/clamav/main.cvd
```

**Private Local Mirrors**
 为解决内网多客户端上`ClamAV`的更新，占用带宽和无法访问等问题，可以通过搭建本地镜像仓库进行分发，通过修改`freshclam.conf`配置，直接从内网服务器进行下载病毒数据库文件。

### 三、ClamAV扫描病毒

`Clamscan` 可以扫描文件、用户目录或者整个系统



```bash
##扫描文件
clamscan targetfile
##递归扫描home目录，并且记录日志
clamscan -r -i /home  -l  /var/log/clamscan.log
##递归扫描home目录，将病毒文件删除，并且记录日志
clamscan -r -i /home  --remove  -l /var/log/clamscan.log
##建议##扫描指定目录，然后将感染文件移动到指定目录，并记录日志
clamscan -r -i /home  --move=/opt/infected  -l /var/log/clamscan.log
```

![img](https:////upload-images.jianshu.io/upload_images/9860291-781214c99ff72960.png?imageMogr2/auto-orient/strip|imageView2/2/w/549/format/webp)

- 扫描完成后，会将扫描详细信息列出

### 四、周期自动扫描病毒



```bash
##每天凌晨11点进行文件扫描
crontab -e
0 23 * * * root  /usr/local/bin/clamscan.sh
##配置扫描文件
vim /usr/local/clamscan.sh
clamscan -r -i /home  --move=/opt/infected  -l /var/log/clamscan.log
```

### 五、ClamAV与业务系统整合

![img](https:////upload-images.jianshu.io/upload_images/9860291-3b704af75df129be.png?imageMogr2/auto-orient/strip|imageView2/2/w/907/format/webp)

image.png

> 业务系统可以直接调用`clamav-scanner`服务来扫描上传的文件。

**方案**

- 在业务系统安装`clamav-REST`服务
- 部署`clamav-scanner server`
- 部署clamav更新服务器，或者直接上网更新
- 部署clamav病毒库更新服务器
- 部署clamav查杀文件所产生的日志服务器（可以直接放在服务端本地）

客户端上传文件，业务系统调用`clamav-rest`接口，让clamd主程序对文件进行扫描，并记录日志

  [clamav-rest](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsolita%2Fclamav-rest)
 [clamav-java](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsolita%2Fclamav-java)



