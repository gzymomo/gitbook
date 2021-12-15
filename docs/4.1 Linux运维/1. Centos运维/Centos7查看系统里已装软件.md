[TOC]

# 1、查看linux服务器装了哪些软件
```bash
--last
Orders the package listing by install time such that the latest packages are at the top.

[root@localhost rpm]# rpm -qa --last
lsof-4.87-6.el7.x86_64                        Mon 27 Apr 2020 01:06:33 PM CST
mysql-community-server-5.7.27-1.el7.x86_64    Mon 27 Apr 2020 11:27:23 AM CST
net-tools-2.0-0.25.20131004git.el7.x86_64     Mon 27 Apr 2020 11:27:03 AM CST
mysql-community-client-5.7.27-1.el7.x86_64    Mon 27 Apr 2020 11:27:03 AM CST
mysql-community-libs-5.7.27-1.el7.x86_64      Mon 27 Apr 2020 11:27:00 AM CST
mysql-community-common-5.7.27-1.el7.x86_64    Mon 27 Apr 2020 11:27:00 AM CST
mysql80-community-release-el7-3.noarch        Fri 24 Apr 2020 05:33:14 PM CST
lrzsz-0.12.20-36.el7.x86_64                   Fri 24 Apr 2020 05:32:56 PM CST
yum-3.4.3-163.el7.centos.noarch               Fri 24 Apr 2020 09:24:24 AM CST
vim-enhanced-7.4.629-6.el7.x86_64             Thu 23 Apr 2020 10:37:44 AM CST
vim-common-7.4.629-6.el7.x86_64               Thu 23 Apr 2020 10:37:44 AM CST
vim-filesystem-7.4.629-6.el7.x86_64           Thu 23 Apr 2020 10:37:42 AM CST
perl-5.16.3-294.el7_6.x86_64                  Thu 23 Apr 2020 10:37:42 AM CST
gpm-libs-1.20.7-6.el7.x86_64                  Thu 23 Apr 2020 10:37:42 AM CST
perl-Pod-Simple-3.28-4.el7.noarch             Thu 23 Apr 2020 10:37:41 AM CST
perl-Getopt-Long-2.40-3.el7.noarch            Thu 23 Apr 2020 10:37:41 AM CST
...
```

# 2、查看一个已安装的rpm包的额外信息
## 2.1 查询一个已经安装的包
```bash
[root@localhost rpm]# rpm -q mysql-community-server
mysql-community-server-5.7.27-1.el7.x86_64
[root@localhost rpm]# rpm -q mysql-community-server-5.7.27 
mysql-community-server-5.7.27-1.el7.x86_64

#如果查不到，会打印相应信息
[root@localhost rpm]# rpm -q mysql-community-server-5.7.27xx 
package mysql-community-server-5.7.27xx is not installed
```

## 2.2 查看配置文件信息
```bash
Package Query Options:
-c, --configfiles
List only configuration files (implies -l).

[root@localhost rpm]# rpm -q mysql-community-server -c
/etc/logrotate.d/mysql
/etc/my.cnf
```

## 2.3 查看文档信息，包括man帮助文档
```bash
-d, --docfiles
List only documentation files (implies -l).

[root@localhost rpm]# rpm -q mysql-community-server -d
/usr/share/doc/mysql-community-server-5.7.27/COPYING
...
/usr/share/man/man8/mysqld.8.gz
```

## 2.4 列出内部的全部文件
```bash
--filesbypkg
List all the files in each selected package.

[root@localhost rpm]# rpm -q mysql-community-server --filesbypkg
mysql-community-server    /etc/logrotate.d/mysql
mysql-community-server    /etc/my.cnf
mysql-community-server    /etc/my.cnf.d
```

## 2.5 查看包的信息，包括安装时间
```bash
-i, --info
Display package information, including name, version, and description. This uses the --queryformat if one was specified.

[root@localhost rpm]# rpm -q mysql-community-server -i
Name        : mysql-community-server
Version     : 5.7.27
Release     : 1.el7
Architecture: x86_64
Install Date: Mon 27 Apr 2020 11:27:23 AM CST
...
```

# 3、查看指定包，要依赖的东西
```bash
-R, --requires
List capabilities on which this package depends.

[root@localhost rpm]# rpm -q mysql-community-server -R
/bin/bash
/bin/sh
/bin/sh
/bin/sh
/bin/sh
/usr/bin/perl
config(mysql-community-server) = 5.7.27-1.el7
coreutils
grep
ld-linux-x86-64.so.2()(64bit)
ld-linux-x86-64.so.2(GLIBC_2.3)(64bit)
...
```

# 4、查看指定包的一些安装卸载过程中的脚本
```bash
--scripts
List the package specific scriptlet(s) that are used as part of the installation and uninstallation processes.

[root@localhost rpm]# rpm -q mysql-community-server --scripts
preinstall scriptlet (using /bin/sh):
/usr/sbin/groupadd -g 27 -o -r mysql >/dev/null 2>&1 || :
/usr/sbin/useradd -M -N -g mysql -o -r -d /var/lib/mysql -s /bin/false \
    -c "MySQL Server" -u 27 mysql >/dev/null 2>&1 || :
postinstall scriptlet (using /bin/sh):
[ -e /var/log/mysqld.log ] || install -m0640 -omysql -gmysql /dev/null /var/log/mysqld.log >/dev/null 2>&1 || :

if [ $1 -eq 1 ] ; then 
        # Initial installation 
        systemctl preset mysqld.service >/dev/null 2>&1 || : 
fi 

/usr/bin/systemctl enable mysqld >/dev/null 2>&1 || :
preuninstall scriptlet (using /bin/sh):

if [ $1 -eq 0 ] ; then 
        # Package removal, not upgrade 
        systemctl --no-reload disable mysqld.service > /dev/null 2>&1 || : 
        systemctl stop mysqld.service > /dev/null 2>&1 || : 
fi
postuninstall scriptlet (using /bin/sh):

systemctl daemon-reload >/dev/null 2>&1 || : 
if [ $1 -ge 1 ] ; then 
        # Package upgrade, not uninstall 
        systemctl try-restart mysqld.service >/dev/null 2>&1 || : 
fi
```
