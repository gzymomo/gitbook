相关参考文档：

[Centos7 安装 clamav](https://www.cnblogs.com/lexihc/p/11339921.html)

[Centos7下杀毒软件clamav的安装和使用](https://www.cnblogs.com/ghl1024/p/9018212.html)

————————————————————————————————————————————————————————



### 环境

```
CentOS: 7.x
```

### 下载

```bash
下载地址 ：http://www.clamav.net/downloads，使用目前最新版本为：clamav-0.101.3
使用 wget 下载
wget https://clamav-site.s3.amazonaws.com/production/release_files/files/000/000/484/original/clamav-0.101.3.tar.gz?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIY6OSGQFGUNJQ7GQ%2F20190812%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20190812T053848Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=965dd9d950d9337bf792a5af2b0fb009dd08fc2533f2b262504a778739439b64
```

### 安装

1. 解压

```
tar -zxvf clamav-0.101.3.tar.gz
```

1. 安装依赖

```
yum install gcc gcc-c++ openssl openssl-devel  -y
```

1. 编译安装

```
cd clamav-0.101.3
./configure --prefix=/usr/local/clamav
make && make install
```

1. 添加用户

```
groupadd clamav
useradd -g clamav clamav
```

### 配置

1. 创建日志目录和病毒库目录

```
mkdir /usr/local/clamav/logs
mkdir /usr/local/clamav/updata
```

1. 创建日志文件

```
touch /usr/local/clamav/logs/clamd.log
touch /usr/local/clamav/logs/freshclam.log
```

1. 文件授权

```
chown clamav:clamav /usr/local/clamav/logs/clamd.log
chown clamav:clamav /usr/local/clamav/logs/freshclam.log
chown clamav:clamav /usr/local/clamav/updata
chown -R clamav.clamav /usr/local/clamav/
```

1. 修改配置文件

```
    cp  /usr/local/clamav/etc/clamd.conf.sample /usr/local/clamav/etc/clamd.conf
    cp  /usr/local/clamav/etc/freshclam.conf.sample /usr/local/clamav/etc/freshclam.conf
    vim /usr/local/clamav/etc/clamd.conf
    // 注释掉第8行,如下
    #Example　　
    #添加以下内容
    LogFile /usr/local/clamav/logs/clamd.log
    PidFile /usr/local/clamav/updata/clamd.pid
    DatabaseDirectory /usr/local/clamav/updata
    vim /usr/local/clamav/etc/freshclam.conf
    // 注释掉第8行,如下
    #Example　　
    #添加以下内容
    DatabaseDirectory /usr/local/clamav/updata
    UpdateLogFile /usr/local/clamav/logs/freshclam.log
    PidFile /usr/local/clamav/updata/freshclam.pid
```

### 执行

创建软链接：

> ln -s /usr/local/clamav/bin/clamscan /usr/local/sbin/clamscan

1. 更新病毒库

```
/usr/local/clamav/bin/freshclam
```

1. 启动

```bash
# 启动
systemctl start clamav-freshclam.service    
    
# 开机启动
systemctl enable clamav-freshclam.service 
    
# 查看状态
systemctl status clamav-freshclam.service
```

1. 扫描杀毒

```
参数：
    -r 递归扫描子目录
    -i 只显示发现的病毒文件
    –no-summary 不显示统计信息

用法：
    # 帮助
    /usr/local/clamav/bin/clamscan --help     
    
    # 默认扫描当前目录下的文件，并显示扫描结果统计信息            
    /usr/local/clamav/bin/clamscan
    
    #扫描当前目录下的所有目录和文件，并显示结果统计信息
    /usr/local/clamav/bin/clamscan -r
    
    #扫描data目录下的所有目录和文件，并显示结果统计信息　                 
    /usr/local/clamav/bin/clamscan -r /data　 
    
    #扫描data目录下的所有目录和文件，只显示有问题的扫描结果            
    /usr/local/clamav/bin/clamscan -r --bell -i /data  
    
    #扫描data目录下的所有目录和文件，不显示统计信息  
    /usr/local/clamav/bin/clamscan --no-summary -ri /data
```

1. 定时更新和杀毒

   crontab定时任务，让服务器每天定时更新和定时杀毒，保存杀毒日志。

```
echo "

 10 2  * * *  /usr/local/clamav/bin/freshclam --quiet

 20 3  * * *  /usr/local/clamav/bin/clamscan  -r /home  --remove -l /var/log/clamscan.log

 ">>crontab
```

安装配置脚本