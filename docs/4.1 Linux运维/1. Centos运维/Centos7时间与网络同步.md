[TOC]

# 1、安装ntpdate工具
`sudo yum -y install ntp ntpdate`

# 2、设置系统时间与网络时间同步
`sudo ntpdate cn.pool.ntp.org`

# 3、将系统时间写入硬件时间
` sudo hwclock --systohc`

# 4、查看系统时间
```bash
timedatectl
#得到
      Local time: 四 2017-09-21 13:54:09 CST
  Universal time: 四 2017-09-21 05:54:09 UTC
        RTC time: 四 2017-09-21 13:54:09
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: no
NTP synchronized: no
 RTC in local TZ: yes
      DST active: n/a
```