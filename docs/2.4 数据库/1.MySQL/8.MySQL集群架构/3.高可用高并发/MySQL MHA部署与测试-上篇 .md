## 1、介绍

### 1.1 简介

`MHA（Master High Availability）`目前在`MySQL`高可用方面是一个相对成熟的解决方案，它由日本`DeNA`公司的`youshimaton`（现就职于`Facebook`公司）开发，是一套优秀的作为`MySQL`高可用性环境下故障切换和主从提升的高可用软件

### 1.2 主要特性

`MHA`的主要特征：

- 从`master`的监控到故障转移全部都能自动完成，故障转移也可以手动执行
- 可在秒级单位内实现故障转移
- 可将任意`slave`提升到`master`
- 具备在多个点上调用外部脚本（扩展）的技能，可以用在电源`OFF`或者`IP`地址的故障转移上
- 安装和卸载不用停止当前的`mysql`进程
- `MHA`自身不会增加服务器负担，不会降低性能，不用追加服务器
- 不依赖`Storage Engine`
- 不依赖二进制文件的格式（不论是`statement`模式还是`Row`模式）
- 针对`OS`挂掉的故障转移，检测系统是否挂掉需要`10`秒，故障转移仅需`4`秒

### 1.3 组成结构

```
manager 组件
masterha_manger             # 启动MHA
masterha_check_ssh         # 检查MHA的SSH配置状况
masterha_check_repl         # 检查MySQL复制状况，配置信息
masterha_master_monitor     # 检测master是否宕机
masterha_check_status       # 检测当前MHA运行状态
masterha_master_switch      # 控制故障转移（自动或者手动）
masterha_conf_host         # 添加或删除配置的server信息

node 组件
save_binary_logs            # 保存和复制master的二进制日志
apply_diff_relay_logs       # 识别差异的中继日志事件并将其差异的事件应用于其他的
purge_relay_logs            # 清除中继日志（不会阻塞SQL线程）
```

## 2、MHA 部署

### 2.1 环境准备

#### 2.1.1 拓扑

![图片](https://mmbiz.qpic.cn/mmbiz_png/Oy8CSKcrQ45uY0Ipia4W1XTx3fMA6VLibo44MYM5Vo0U8EtH446U6GCdjlLrIlz3xvT9rcKpJxmgMk4cXjDd9xtA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 2.1.2 环境准备 

需准备`Linux`环境`Mysql`1 主 2 从环境，本次部署具体使用软件版本情况如下 操作系统：`Linux CentOS 7.6`数据库：`MySQL 8.0.20`MHA：`mha4mysql-0.58`

### 2.2 环境部署

#### 2.2.1 安装操作系统

操作系统版本：`Linux CentOS 7.6`

主机名及`ip`：

`db1`：192.168.10.51，`db2`：192.168.10.52，`db3`：192.168.10.53

#### 2.2.2 安装数据库

分别在三台服务器安装`MySQL 8.0.20`数据库 配置文件如下：

```
[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/3306/data
socket=/tmp/mysql.sock

gtid-mode=on                        # gtid开关
enforce-gtid-consistency=true       # 强制GTID一致
log-slave-updates=1                 # 从库强制更新binlog日志

log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 15
log-error=/data/3306/data/mysql_error.log
server-id = 51     # server-id 三台不同 ，db2为52，db3为53
default_authentication_plugin=mysql_native_password   #修改密码加密方式

[mysql]
socket=/tmp/mysql.sock
```

#### 2.2.3 搭建主从环境

搭建主从环境，确认 1 主 2 从运行正常

### 2.3 MHA 部署

#### 2.3.1 节点互信

```
db01
rm -rf /root/.ssh
ssh-keygen
ssh-copy-id 192.168.10.51
ssh-copy-id 192.168.10.52
ssh-copy-id 192.168.10.53
db02
ssh-keygen
ssh-copy-id 192.168.10.51
ssh-copy-id 192.168.10.52
ssh-copy-id 192.168.10.53
db03
ssh-keygen
ssh-copy-id 192.168.10.51
ssh-copy-id 192.168.10.52
ssh-copy-id 192.168.10.53
```

#### 2.3.2 下载软件

```
mkdir -p /data/tools/ && cd /data/tools
wget https://qiniu.wsfnk.com/mha4mysql-node-0.58-0.el7.centos.noarch.rpm --no-check-certificate
wget https://qiniu.wsfnk.com/mha4mysql-manager-0.58-0.el7.centos.noarch.rpm --no-check-certificate
```

#### 2.3.3 创建软连接

```
ln -s /usr/local/mysql/bin/mysqlbinlog  /usr/bin/mysqlbinlog
ln -s /usr/local/mysql/bin/mysql          /usr/bin/mysql
```

#### 2.3.4 安装软件

安装`node`软件（`mha`的`manager`软件是依赖于`node`软件运行的，所以需要先安装`node`端） db1-db3

```
yum install perl-DBD-MySQL -y
rpm -ivh mha4mysql-node-0.58-0.el7.centos.noarch.rpm
```

安装`manager`db3

```
yum install -y perl-Config-Tiny epel-release perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes
yum install -y mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
```

#### 2.3.5 创建用户

在`db1`主库中创建`mha`用户，此用户用于`mha`用来确认各节点存活状态、`binlog_server`拉去`binlog`日志。(mysql 8.0 之后版本需要单独创建用户后授权。注意密码加密方式)

```
create user mha@'192.168.10.%' identified by 'mha';
grant all privileges on *.* to mha@'192.168.10.%';
```

#### 2.3.6 创建相关目录

db3

```
#创建配置文件目录
 mkdir -p /etc/mha
#创建日志目录
 mkdir -p /var/log/mha/app1
```

#### 2.3.7 manager 配置文件

db3

```
cat > /etc/mha/app1.cnf <<EOF
[server default]
manager_log=/var/log/mha/app1/manager              # MHA的工作日志设置
manager_workdir=/var/log/mha/app1                  # MHA工作目录
master_binlog_dir=/data/3306/data                  # 主库的binlog目录
user=mha                                           # 监控用户
password=mha                                       # 监控密码
ping_interval=2                                    # 心跳检测的间隔时间
repl_password=123                                  # 复制用户
repl_user=rep                                      # 复制密码
ssh_user=root                                      # ssh互信的用户

[server1]                                          # 节点信息....
hostname=192.16.10.51
port=3306

[server2]
hostname=192.168.10.52
port=3306
candidate_master=1                                 # 被选主

[server3]
no_master=1                                        # 不参与选主
hostname=192.168.10.53
port=3306
EOF
```

#### 2.3.8 检查状态

db3

```
# 检查ssl互通情况
[root@db3 ~]# vim /etc/mha/app1.cnf
[root@db3 ~]# masterha_check_ssh   --conf=/etc/mha/app1.cnf
...
Tue Jun  1 17:01:47 2021 - [info] All SSH connection tests passed successfully.


# 检查 mha配置文件
[root@db3 ~]# masterha_check_repl  --conf=/etc/mha/app1.cnf
...
MySQL Replication Health is NOT OK!
```

#### 2.3.9 启动 manager 服务

db03

```
[root@db3 ~]# nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
[1] 4406

--conf=/etc/mha/app1.cnf    # 指定配置文件
--remove_dead_master_conf   # 剔除已经死亡的节点
--ignore_last_failover      # 默认不能短时间(8小时)多次切换，此参数跳过检查
```

#### 2.3.10 查看 mha 运行状态

```
[root@db3 ~]# masterha_check_status --conf=/etc/mha/app1.cnf
app1 (pid:4406) is running(0:PING_OK), master:192.168.10.51
```

## 3、vip、故障提醒、binlog_server

```
master_ip_failover  # vip故障转移脚本
send_report         # 故障提醒脚本
```

### 3.1 vip 功能部署

vip（eth0:1）网卡日常绑定在主库的网卡上，如主库出现错误导致`mha`重新选主，也会跟随移动到新主库的网卡上。所以要求各`mha`节点网卡名称一致`vip`网段要求与各节点均在同一网段内`vip`实现脚本是根据源码 perl 脚本重新编写`master_ip_failover`

#### 3.1.1 准备脚本

```
#!/usr/bin/env perl

use strict;
use warnings FATAL => 'all';

use Getopt::Long;

my (
    $command,          $ssh_user,        $orig_master_host, $orig_master_ip,
    $orig_master_port, $new_master_host, $new_master_ip,    $new_master_port
);

my $vip = '192.168.10.49/24';
my $key = '1';
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";
my $ssh_Bcast_arp= "/sbin/arping -I eth0 -c 3 -A 192.168.10.49";

GetOptions(
    'command=s'          => \$command,
    'ssh_user=s'         => \$ssh_user,
    'orig_master_host=s' => \$orig_master_host,
    'orig_master_ip=s'   => \$orig_master_ip,
    'orig_master_port=i' => \$orig_master_port,
    'new_master_host=s'  => \$new_master_host,
    'new_master_ip=s'    => \$new_master_ip,
    'new_master_port=i'  => \$new_master_port,
);

exit &main();

sub main {

    print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

    if ( $command eq "stop" || $command eq "stopssh" ) {

        my $exit_code = 1;
        eval {
            print "Disabling the VIP on old master: $orig_master_host \n";
            &stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "start" ) {

        my $exit_code = 10;
        eval {
            print "Enabling the VIP - $vip on the new master - $new_master_host \n";
            &start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        exit 0;
    }
    else {
        &usage();
        exit 1;
    }
}

sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
     return 0  unless  ($ssh_user);
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
    print
    "Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

#### 3.1.2 修改脚本

脚本中以下部分按照要求修改

```
vim  /usr/local/bin/master_ip_failover

my $vip = '192.168.10.49/24';      # vip网段，与各节点服务器网段一致。
my $key = '1';                     # 虚拟网卡eth0:1 的 1
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";  # 网卡按照实际网卡名称填写
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";   # 网卡按照实际网卡名称填写
my $ssh_Bcast_arp= "/sbin/arping -I eth0 -c 3 -A 192.168.10.49"; # 重新声明mac地址，arp
```

#### 3.1.3 上传脚本并授权

```
\cp -a * /usr/local/bin    # 上传脚本文件到/usr/local/bin
chmod +x /usr/local/bin/*  # 给脚本增加执行权限
dos2unix /usr/local/bin/*  # 处理脚本中的中文字符
```

#### 3.1.4 修改 manager 配置文件

在配置文件中增加`vip`故障转移部分

```
vim /etc/mha/app1.cnf
master_ip_failover_script=/usr/local/bin/master_ip_failover
```

#### 3.1.5 重启 mha

```
masterha_stop  --conf=/etc/mha/app1.cnf
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
```

#### 3.1.6 在主库 db1 增加 vip

```
ifconfig eth0:1 192.168.10.49/24
```

### 3.2 故障提醒功能部署

#### 3.2.1 准备脚本

```
#!/usr/bin/perl

use strict;
use warnings FATAL => 'all';
use Mail::Sender;
use Getopt::Long;

#new_master_host and new_slave_hosts are set only when recovering master succeeded
my ( $dead_master_host, $new_master_host, $new_slave_hosts, $subject, $body );
my $smtp='smtp.qq.com';
my $mail_from='xxxxxx@qq.com';
my $mail_user='xxxxx';
my $mail_pass='xxxxxx';
#my $mail_to=['to1@qq.com','to2@qq.com'];
my $mail_to='xxxxxx@qq.com';

GetOptions(
  'orig_master_host=s' => \$dead_master_host,
  'new_master_host=s'  => \$new_master_host,
  'new_slave_hosts=s'  => \$new_slave_hosts,
  'subject=s'          => \$subject,
  'body=s'             => \$body,
);

# Do whatever you want here
mailToContacts($smtp,$mail_from,$mail_user,$mail_pass,$mail_to,$subject,$body);

sub mailToContacts {
 my ($smtp, $mail_from, $mail_user, $mail_pass, $mail_to, $subject, $msg ) = @_;
 open my $DEBUG, ">/tmp/mail.log"
  or die "Can't open the debug file:$!\n";
 my $sender = new Mail::Sender {
  ctype  => 'text/plain;charset=utf-8',
  encoding => 'utf-8',
  smtp  => $smtp,
  from  => $mail_from,
  auth  => 'LOGIN',
  TLS_allowed => '0',
  authid  => $mail_user,
  authpwd  => $mail_pass,
  to  => $mail_to,
  subject  => $subject,
  debug  => $DEBUG
 };
 $sender->MailMsg(
  {
   msg => $msg,
   debug => $DEBUG
  }
 ) or print $Mail::Sender::Error;
 return 1;
}

exit 0;
```

修改以下部分为个人邮箱内容

```
#new_master_host and new_slave_hosts are set only when recovering master succeeded
my ( $dead_master_host, $new_master_host, $new_slave_hosts, $subject, $body );
my $smtp='smtp.qq.com';                   # 发件服务器
my $mail_from='xxxxxx@qq.com';            # 发件地址
my $mail_user='xxxxx';                    # 发件用户名
my $mail_pass='xxxxxx';                   # 发件邮箱密码
my $mail_to='xxxxxx@qq.com';              # 收件地址

#my $mail_to=['to1@qq.com','to2@qq.com']; # 可设置收件邮箱群组
```

#### 3.2.2 修改 manager 配置文件

```
vim /etc/mha/app1.cnf
# 添加一行：
report_script=/usr/local/bin/send_report
```

#### 3.2.3 重启 mha

```
masterha_stop  --conf=/etc/mha/app1.cnf
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
```

### 3.3 binlog_server 搭建

`binlog_server`功能主要是实时拉去主库`binlog`进行存储，如果主库出现宕机，`mha`切换主从时，通过`binlog_server`服务器拿取`bin_log`日志对新主库进行数据补偿，实现日志补偿冗余

#### 3.3.1 创建相关目录

```
mkdir -p /data/binlog_server/   # binlog_server存储目录
chown -R mysql.mysql /data/*    # 授权
```

#### 3.3.2 开启 binlog_server 服务

```
cd  /data/binlog_server/     # 进入binlog_server存储目录
mysql -e "show slave status \G"|grep "Master_Log"  # 确认当前slave的io进程拉取的日志量

## 启动binlog日志拉取守护进程
mysqlbinlog  -R --host=192.168.10.51 --user=mha --password=mha --raw  --stop-never mysql-bin.000004 &      # 注意：拉取日志的起点,需要按照目前从库的已经获取到的二进制日志点为起点
```

#### 3.3.3 修改 manager 配置文件

增加以下内容

```
vim /etc/mha/app1.cnf
[binlog1]
no_master=1
hostname=192.168.10.53
master_binlog_dir=/data/binlog_server/
```

#### 3.3.4 重启 mha

```
masterha_stop  --conf=/etc/mha/app1.cnf
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
```

## 4、MHA 工作原理介绍

### 4.1 主要工作流程介绍

- 1、启动 MHA-manager

```
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &
```

- 2、监控

```
/usr/bin/masterha_master_monitor # 每隔ping_interval秒探测1次，连续4次还没有，说明主库宕机。
```

- 3、选主 根据各项参数选择新主
- 4、数据补偿 对新主的数据差异进行补偿。
- 5、切换 所有从库解除主从身份。stop slave ; reset slave; 重构新的主从关系。change master to
- 6、迁移 vip 将 vip 网卡（eth0:1）绑定至新主服务器。
- 7、故障提醒 发送故障提醒邮件
- 8、额外数据补偿 根据 binlog_server 服务器进行额外数据补偿
- 9、剔除故障节点 将故障的主服务器剔除 mha 环境（配置文件删除）
- 10、manager 程序“自杀

### 4.2 mha 选主策略

#### 4.2.1 选主依据

1、日志量`latest`

选取获取`binlog`最多的`slave`节点，为`latest`节点

```
[root@db3 ~]# mysql -uroot -p123456 -e "show slave status\G" |grep Master_Log
          Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 196
          Relay_Master_Log_File: mysql-bin.000005
          Exec_Master_Log_Pos: 196
```

2、备选主`pref``manager`配置文件中指定了`candidate_master=1`参数

```
[server2]
hostname=192.168.10.52
port=3306
candidate_master=1     # 备选主
```

3、不被选主 配置文件中设置`no_master=1`

```
[server3]
no_master=1    # 不被选
hostname=192.168.10.53
port=3306
```

二进制日志没开`log_bin`如果从库落后主库`100M`的日志量（可以关闭） check_slave_delay

#### 4.2.2 节点数组

| alive  | 存活     |
| :----- | :------- |
| latest | 日志最新 |
| perf   | 被选     |
| bad    | 不选     |

#### 4.2.3 选主判断

从上至下按条件依次筛选，上条不符合才会选择下一条 1、没有`pref`和`bad`，就选`latest`第一个（根据配置文件 server1-2-3） 2、即是`latest`又是`perf`，又不是`bad`3、`perf`和`latest`，都不是`bad`，选`perf`4、第`3`条也可能选`latest`，如`perf`差`100M`以上`binlog`5、如果以上条件都不满足，且只有一个`salve`可选，则选择此`slave`6、没有符合的`slave`，则选主失败，`failover`失败

### 4.3 数据补偿

#### 4.3.1 原主库 ssh 可连接

各个从节点调用：`save_binary_logs`脚本，立即保存缺失部分的`binlog`到各自节点`/var/tmp`目录

#### 4.3.2 原主库 ssh 不能连接

从节点调用`apply_diff_relay_logs`，进行`relay-log`日志差异补偿

#### 4.3.3 额外数据补偿(主库日志冗余机制)

`binlog_server`实现数据补偿