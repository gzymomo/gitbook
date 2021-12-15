## MySQL MHA部署与测试-下篇

## 1、故障测试

### 1.1 操作流程

```
# 追踪mha-manager日志
[root@db03 ~]# tail -f  /var/log/mha/app1/manager
# 关闭主库
[root@db01 ~]# /etc/init.d/mysqld stop
```

### 1.2 日志查看

```
# 确认主库宕机部分
Wed Jun  2 16:55:42 2021 - [warning] Got error on MySQL select ping: 1053 (Server shutdown in progress)
Wed Jun  2 16:55:42 2021 - [info] Executing SSH check script: exit 0
Wed Jun  2 16:55:42 2021 - [info] HealthCheck: SSH to 192.168.10.51 is reachable.
Wed Jun  2 16:55:44 2021 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '192.168.10.51' (111))
Wed Jun  2 16:55:44 2021 - [warning] Connection failed 2 time(s)..
Wed Jun  2 16:55:46 2021 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '192.168.10.51' (111))
Wed Jun  2 16:55:46 2021 - [warning] Connection failed 3 time(s)..
Wed Jun  2 16:55:48 2021 - [warning] Got error on MySQL connect: 2003 (Can't connect to MySQL server on '192.168.10.51' (111))
Wed Jun  2 16:55:48 2021 - [warning] Connection failed 4 time(s)..
Wed Jun  2 16:55:48 2021 - [warning] Master is not reachable from health checker!
Wed Jun  2 16:55:48 2021 - [warning] Master 192.168.10.51(192.168.10.51:3306) is not reachable!



# 确认各节点状态
Wed Jun  2 16:55:48 2021 - [warning] SSH is reachable.
Wed Jun  2 16:55:48 2021 - [info] Connecting to a master server failed. Reading configuration file /etc/masterha_default.cnf and /etc/mha/app1.cnf again, and trying to connect to all servers to check server status..
Wed Jun  2 16:55:48 2021 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Jun  2 16:55:48 2021 - [info] Reading application default configuration from /etc/mha/app1.cnf..
Wed Jun  2 16:55:48 2021 - [info] Reading server configuration from /etc/mha/app1.cnf..
Wed Jun  2 16:55:49 2021 - [info] GTID failover mode = 1
Wed Jun  2 16:55:49 2021 - [info] Dead Servers:
Wed Jun  2 16:55:49 2021 - [info]   192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:49 2021 - [info] Alive Servers:
Wed Jun  2 16:55:49 2021 - [info]   192.168.10.52(192.168.10.52:3306)
Wed Jun  2 16:55:49 2021 - [info]   192.168.10.53(192.168.10.53:3306)
Wed Jun  2 16:55:49 2021 - [info] Alive Slaves:
Wed Jun  2 16:55:49 2021 - [info]   192.168.10.52(192.168.10.52:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:49 2021 - [info]     GTID ON
Wed Jun  2 16:55:49 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:49 2021 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Jun  2 16:55:49 2021 - [info]   192.168.10.53(192.168.10.53:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:49 2021 - [info]     GTID ON
Wed Jun  2 16:55:49 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:49 2021 - [info]     Not candidate for the new Master (no_master is set)
Wed Jun  2 16:55:49 2021 - [info] Checking slave configurations..
Wed Jun  2 16:55:49 2021 - [info]  read_only=1 is not set on slave 192.168.10.52(192.168.10.52:3306).
Wed Jun  2 16:55:49 2021 - [info]  read_only=1 is not set on slave 192.168.10.53(192.168.10.53:3306).
Wed Jun  2 16:55:49 2021 - [info] Checking replication filtering settings..
Wed Jun  2 16:55:49 2021 - [info]  Replication filtering check ok.
Wed Jun  2 16:55:49 2021 - [info] Master is down!
Wed Jun  2 16:55:49 2021 - [info] Terminating monitoring script.
Wed Jun  2 16:55:49 2021 - [info] Got exit code 20 (Master dead).
Wed Jun  2 16:55:49 2021 - [info] MHA::MasterFailover version 0.58.
Wed Jun  2 16:55:49 2021 - [info] Starting master failover.
Wed Jun  2 16:55:49 2021 - [info]
Wed Jun  2 16:55:49 2021 - [info] * Phase 1: Configuration Check Phase..
Wed Jun  2 16:55:49 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] GTID failover mode = 1
Wed Jun  2 16:55:51 2021 - [info] Dead Servers:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info] Checking master reachability via MySQL(double check)...
Wed Jun  2 16:55:51 2021 - [info]  ok.
Wed Jun  2 16:55:51 2021 - [info] Alive Servers:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.52(192.168.10.52:3306)
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.53(192.168.10.53:3306)
Wed Jun  2 16:55:51 2021 - [info] Alive Slaves:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.52(192.168.10.52:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.53(192.168.10.53:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Not candidate for the new Master (no_master is set)
Wed Jun  2 16:55:51 2021 - [info] Starting GTID based failover.
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] ** Phase 1: Configuration Check Phase completed.
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 2: Dead Master Shutdown Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] Forcing shutdown so that applications never connect to the current master..
Wed Jun  2 16:55:51 2021 - [info] Executing master IP deactivation script:
Wed Jun  2 16:55:51 2021 - [info]   /usr/local/bin/master_ip_failover --orig_master_host=192.168.10.51 --orig_master_ip=192.168.10.51 --orig_master_port=3306 --command=stopssh --ssh_user=root

# VIP状态检查，日志补偿（此部分未发现日志缺失）
IN SCRIPT TEST====/sbin/ifconfig eth0:1 down==/sbin/ifconfig eth0:1 192.168.10.49/24===

Disabling the VIP on old master: 192.168.10.51
SIOCSIFFLAGS: Cannot assign requested address
Wed Jun  2 16:55:51 2021 - [info]  done.
Wed Jun  2 16:55:51 2021 - [warning] shutdown_script is not set. Skipping explicit shutting down of the dead master.
Wed Jun  2 16:55:51 2021 - [info] * Phase 2: Dead Master Shutdown Phase completed.
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 3: Master Recovery Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 3.1: Getting Latest Slaves Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] The latest binary log file/position on all slaves is mysql-bin.000005:196
Wed Jun  2 16:55:51 2021 - [info] Latest slaves (Slaves that received relay log files to the latest):
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.52(192.168.10.52:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.53(192.168.10.53:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Not candidate for the new Master (no_master is set)
Wed Jun  2 16:55:51 2021 - [info] The oldest binary log file/position on all slaves is mysql-bin.000005:196
Wed Jun  2 16:55:51 2021 - [info] Oldest slaves:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.52(192.168.10.52:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.53(192.168.10.53:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Not candidate for the new Master (no_master is set)
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 3.3: Determining New Master Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] Searching new master from slaves..
Wed Jun  2 16:55:51 2021 - [info]  Candidate masters from the configuration file:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.52(192.168.10.52:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Jun  2 16:55:51 2021 - [info]  Non-candidate masters:
Wed Jun  2 16:55:51 2021 - [info]   192.168.10.53(192.168.10.53:3306)  Version=8.0.20 (oldest major version between slaves) log-bin:enabled
Wed Jun  2 16:55:51 2021 - [info]     GTID ON
Wed Jun  2 16:55:51 2021 - [info]     Replicating from 192.168.10.51(192.168.10.51:3306)
Wed Jun  2 16:55:51 2021 - [info]     Not candidate for the new Master (no_master is set)
Wed Jun  2 16:55:51 2021 - [info]  Searching from candidate_master slaves which have received the latest relay log events..

# 选主结束，开始切换
Wed Jun  2 16:55:51 2021 - [info] New master is 192.168.10.52(192.168.10.52:3306)
Wed Jun  2 16:55:51 2021 - [info] Starting master failover..
Wed Jun  2 16:55:51 2021 - [info]
From:
192.168.10.51(192.168.10.51:3306) (current master)
 +--192.168.10.52(192.168.10.52:3306)
 +--192.168.10.53(192.168.10.53:3306)

To:
192.168.10.52(192.168.10.52:3306) (new master)
 +--192.168.10.53(192.168.10.53:3306)
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 3.3: New Master Recovery Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info]  Waiting all logs to be applied..
Wed Jun  2 16:55:51 2021 - [info]   done.
Wed Jun  2 16:55:51 2021 - [info] Getting new master's binlog name and position..
Wed Jun  2 16:55:51 2021 - [info]  mysql-bin.000005:196
Wed Jun  2 16:55:51 2021 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='192.168.10.52', MASTER_PORT=3306, MASTER_AUTO_POSITION=1, MASTER_USER='rep', MASTER_PASSWORD='xxx';
Wed Jun  2 16:55:51 2021 - [info] Master Recovery succeeded. File:Pos:Exec_Gtid_Set: mysql-bin.000005, 196, f4682075-c1b9-11eb-86b2-000c2934cc5a:1-21
Wed Jun  2 16:55:51 2021 - [info] Executing master IP activate script:
Wed Jun  2 16:55:51 2021 - [info]   /usr/local/bin/master_ip_failover --command=start --ssh_user=root --orig_master_host=192.168.10.51 --orig_master_ip=192.168.10.51 --orig_master_port=3306 --new_master_host=192.168.10.52 --new_master_ip=192.168.10.52 --new_master_port=3306 --new_master_user='mha'   --new_master_password=xxx
Unknown option: new_master_user
Unknown option: new_master_password

# 切换vip，处理manager配置文件（将db1踢出mha环境）
IN SCRIPT TEST====/sbin/ifconfig eth0:1 down==/sbin/ifconfig eth0:1 192.168.10.49/24===

Enabling the VIP - 192.168.10.49/24 on the new master - 192.168.10.52
Wed Jun  2 16:55:51 2021 - [info]  OK.
Wed Jun  2 16:55:51 2021 - [info] ** Finished master recovery successfully.
Wed Jun  2 16:55:51 2021 - [info] * Phase 3: Master Recovery Phase completed.
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 4: Slaves Recovery Phase..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] * Phase 4.1: Starting Slaves in parallel..
Wed Jun  2 16:55:51 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info] -- Slave recovery on host 192.168.10.53(192.168.10.53:3306) started, pid: 48882. Check tmp log /var/log/mha/app1/192.168.10.53_3306_20210602165549.log if it takes time..
Wed Jun  2 16:55:53 2021 - [info]
Wed Jun  2 16:55:53 2021 - [info] Log messages from 192.168.10.53 ...
Wed Jun  2 16:55:53 2021 - [info]
Wed Jun  2 16:55:51 2021 - [info]  Resetting slave 192.168.10.53(192.168.10.53:3306) and starting replication from the new master 192.168.10.52(192.168.10.52:3306)..
Wed Jun  2 16:55:51 2021 - [info]  Executed CHANGE MASTER.
Wed Jun  2 16:55:52 2021 - [info]  Slave started.
Wed Jun  2 16:55:52 2021 - [info]  gtid_wait(f4682075-c1b9-11eb-86b2-000c2934cc5a:1-21) completed on 192.168.10.53(192.168.10.53:3306). Executed 0 events.
Wed Jun  2 16:55:53 2021 - [info] End of log messages from 192.168.10.53.
Wed Jun  2 16:55:53 2021 - [info] -- Slave on host 192.168.10.53(192.168.10.53:3306) started.
Wed Jun  2 16:55:53 2021 - [info] All new slave servers recovered successfully.
Wed Jun  2 16:55:53 2021 - [info]
Wed Jun  2 16:55:53 2021 - [info] * Phase 5: New master cleanup phase..
Wed Jun  2 16:55:53 2021 - [info]
Wed Jun  2 16:55:53 2021 - [info] Resetting slave info on the new master..
Wed Jun  2 16:55:53 2021 - [info]  192.168.10.52: Resetting slave info succeeded.
Wed Jun  2 16:55:53 2021 - [info] Master failover to 192.168.10.52(192.168.10.52:3306) completed successfully.
Wed Jun  2 16:55:53 2021 - [info] Deleted server1 entry from /etc/mha/app1.cnf .
Wed Jun  2 16:55:53 2021 - [info]

# 切换完成
----- Failover Report -----

app1: MySQL Master failover 192.168.10.51(192.168.10.51:3306) to 192.168.10.52(192.168.10.52:3306) succeeded

Master 192.168.10.51(192.168.10.51:3306) is down!

Check MHA Manager logs at db3:/var/log/mha/app1/manager for details.

Started automated(non-interactive) failover.
Invalidated master IP address on 192.168.10.51(192.168.10.51:3306)
Selected 192.168.10.52(192.168.10.52:3306) as a new master.
192.168.10.52(192.168.10.52:3306): OK: Applying all logs succeeded.
192.168.10.52(192.168.10.52:3306): OK: Activated master IP address.
192.168.10.53(192.168.10.53:3306): OK: Slave started, replicating from 192.168.10.52(192.168.10.52:3306)
192.168.10.52(192.168.10.52:3306): Resetting slave info succeeded.
Master failover to 192.168.10.52(192.168.10.52:3306) completed successfully.
```

## 2、主动切换

故障后进行主动切换，查看主从状态和`vip`环境确认

主库已切换至`db2`

```
[root@db3 ~ ]# mysql -uroot -p123456 -e "show slave status\G;" |grep Master_Host
mysql: [Warning] Using a password on the command line interface can be insecure.
                  Master_Host: 192.168.10.52   # 主库已变更
vip`已切换至`db2
[root@db2 ~]# ip a |grep eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.10.52/24 brd 192.168.10.255 scope global noprefixroute eth0
    inet 192.168.10.49/24 brd 192.168.10.255 scope global secondary eth0:1
```

## 3、在线切换

### 3.1 只切换角色

#### 3.1.1 切换命令

```
masterha_master_switch  --conf=/etc/mha/app1.cnf --master_state=alive --new_master_host=192.168.10.51 --orig_master_is_new_slave --running_updates_limit=10000

master_ip_online_change_script is not defined. If you do not disable writes on the current master manually, applications keep writing on the current master. Is it ok to proceed? (yes/NO): yes

# 命令解析
--conf=/etc/mha/app1.cnf  # 指定配置文件
--master_state=alive      # 主库在线的情况下进行切换
--new_master_host=192.168.10.51   # 指定新主ip
--orig_master_is_new_slave  # 原主库改为从库
running_updates_limit=10000  # 网络延时(ping值)超过1w毫秒不进行切换
```

#### 3.1.2 注意内容

**此命令一般不会在生产环境使用，只用于测试**1、需要关闭`mha-manager`，不然切换无法执行成功。报错如下：

```
Tue Jun  8 10:03:54 2021 - [error][/usr/share/perl5/vendor_perl/MHA/MasterRotate.pm, ln143] Getting advisory lock failed on the current master. MHA Monitor runs on the current master. Stop MHA Manager/Monitor and try again.
Tue Jun  8 10:03:54 2021 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln177] Got ERROR:  at /usr/bin/masterha_master_switch line 53.
```

2、需要将原主库锁住`Flush table with read lock`，使其只读。因为在切换完主从后，`vip`尚未切换，此段时间数据还会写入到原主库，导致数据不一致。警告如下：

```
It is better to execute FLUSH NO_WRITE_TO_BINLOG TABLES on the master before switching. Is it ok to execute on 192.168.10.52(192.168.10.52:3306)? (YES/no): yes
```

3、需要手工切换`vip`

4、需要重新拉取主库`binlog`（binlog-server）

### 3.2 脚本功能实现

功能: 在线切换时，自动锁原主库，`VIP`自动切换

#### 3.2.1 准备脚本

```
vim /usr/local/bin/master_ip_online_change
#!/usr/bin/env perl

use strict;
use warnings FATAL => 'all';
use Getopt::Long;
use MHA::DBHelper;
use MHA::NodeUtil;
use Time::HiRes qw( sleep gettimeofday tv_interval );
use Data::Dumper;
my $_tstart;
my $_running_interval = 0.1;
my (
  $command,              $orig_master_is_new_slave, $orig_master_host,
  $orig_master_ip,       $orig_master_port,         $orig_master_user,
  $orig_master_password, $orig_master_ssh_user,     $new_master_host,
  $new_master_ip,        $new_master_port,          $new_master_user,
  $new_master_password,  $new_master_ssh_user,
);

###########################################################################
my $vip = "10.0.0.55";
my $key = "1";
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key $vip down";
my $ssh_Bcast_arp= "/sbin/arping -I ens33 -c 3 -A 10.0.0.55";
###########################################################################

GetOptions(
  'command=s'                => \$command,
  'orig_master_is_new_slave' => \$orig_master_is_new_slave,
  'orig_master_host=s'       => \$orig_master_host,
  'orig_master_ip=s'         => \$orig_master_ip,
  'orig_master_port=i'       => \$orig_master_port,
  'orig_master_user=s'       => \$orig_master_user,
  'orig_master_password=s'   => \$orig_master_password,
  'orig_master_ssh_user=s'   => \$orig_master_ssh_user,
  'new_master_host=s'        => \$new_master_host,
  'new_master_ip=s'          => \$new_master_ip,
  'new_master_port=i'        => \$new_master_port,
  'new_master_user=s'        => \$new_master_user,
  'new_master_password=s'    => \$new_master_password,
  'new_master_ssh_user=s'    => \$new_master_ssh_user,
);
exit &main();
sub current_time_us {
  my ( $sec, $microsec ) = gettimeofday();
  my $curdate = localtime($sec);
  return $curdate . " " . sprintf( "%06d", $microsec );
}
sub sleep_until {
  my $elapsed = tv_interval($_tstart);
  if ( $_running_interval > $elapsed ) {
    sleep( $_running_interval - $elapsed );
  }
}
sub get_threads_util {
  my $dbh                    = shift;
  my $my_connection_id       = shift;
  my $running_time_threshold = shift;
  my $type                   = shift;
  $running_time_threshold = 0 unless ($running_time_threshold);
  $type                   = 0 unless ($type);
  my @threads;
  my $sth = $dbh->prepare("SHOW PROCESSLIST");
  $sth->execute();
  while ( my $ref = $sth->fetchrow_hashref() ) {
    my $id         = $ref->{Id};
    my $user       = $ref->{User};
    my $host       = $ref->{Host};
    my $command    = $ref->{Command};
    my $state      = $ref->{State};
    my $query_time = $ref->{Time};
    my $info       = $ref->{Info};
    $info =~ s/^\s*(.*?)\s*$/$1/ if defined($info);
    next if ( $my_connection_id == $id );
    next if ( defined($query_time) && $query_time < $running_time_threshold );
    next if ( defined($command)    && $command eq "Binlog Dump" );
    next if ( defined($user)       && $user eq "system user" );
    next
      if ( defined($command)
      && $command eq "Sleep"
      && defined($query_time)
      && $query_time >= 1 );
    if ( $type >= 1 ) {
      next if ( defined($command) && $command eq "Sleep" );
      next if ( defined($command) && $command eq "Connect" );
    }
    if ( $type >= 2 ) {
      next if ( defined($info) && $info =~ m/^select/i );
      next if ( defined($info) && $info =~ m/^show/i );
    }
    push @threads, $ref;
  }
  return @threads;
}
sub main {
  if ( $command eq "stop" ) {
    ## Gracefully killing connections on the current master
    # 1. Set read_only= 1 on the new master
    # 2. DROP USER so that no app user can establish new connections
    # 3. Set read_only= 1 on the current master
    # 4. Kill current queries
    # * Any database access failure will result in script die.
    my $exit_code = 1;
    eval {
      ## Setting read_only=1 on the new master (to avoid accident)
      my $new_master_handler = new MHA::DBHelper();
      # args: hostname, port, user, password, raise_error(die_on_error)_or_not
      $new_master_handler->connect( $new_master_ip, $new_master_port,
        $new_master_user, $new_master_password, 1 );
      print current_time_us() . " Set read_only on the new master.. ";
      $new_master_handler->enable_read_only();
      if ( $new_master_handler->is_read_only() ) {
        print "ok.\n";
      }
      else {
        die "Failed!\n";
      }
      $new_master_handler->disconnect();
      # Connecting to the orig master, die if any database error happens
      my $orig_master_handler = new MHA::DBHelper();
      $orig_master_handler->connect( $orig_master_ip, $orig_master_port,
        $orig_master_user, $orig_master_password, 1 );
      ## Drop application user so that nobody can connect. Disabling per-session binlog beforehand
      $orig_master_handler->disable_log_bin_local();
      print current_time_us() . " Drpping app user on the orig master..\n";
###########################################################################
      #FIXME_xxx_drop_app_user($orig_master_handler);
###########################################################################
      ## Waiting for N * 100 milliseconds so that current connections can exit
      my $time_until_read_only = 15;
      $_tstart = [gettimeofday];
      my @threads = get_threads_util( $orig_master_handler->{dbh},
        $orig_master_handler->{connection_id} );
      while ( $time_until_read_only > 0 && $#threads >= 0 ) {
        if ( $time_until_read_only % 5 == 0 ) {
          printf
"%s Waiting all running %d threads are disconnected.. (max %d milliseconds)\n",
            current_time_us(), $#threads + 1, $time_until_read_only * 100;
          if ( $#threads < 5 ) {
            print Data::Dumper->new( [$_] )->Indent(0)->Terse(1)->Dump . "\n"
              foreach (@threads);
          }
        }
        sleep_until();
        $_tstart = [gettimeofday];
        $time_until_read_only--;
        @threads = get_threads_util( $orig_master_handler->{dbh},
          $orig_master_handler->{connection_id} );
      }
      ## Setting read_only=1 on the current master so that nobody(except SUPER) can write
      print current_time_us() . " Set read_only=1 on the orig master.. ";
      $orig_master_handler->enable_read_only();
      if ( $orig_master_handler->is_read_only() ) {
        print "ok.\n";
      }
      else {
        die "Failed!\n";
      }
      ## Waiting for M * 100 milliseconds so that current update queries can complete
      my $time_until_kill_threads = 5;
      @threads = get_threads_util( $orig_master_handler->{dbh},
        $orig_master_handler->{connection_id} );
      while ( $time_until_kill_threads > 0 && $#threads >= 0 ) {
        if ( $time_until_kill_threads % 5 == 0 ) {
          printf
"%s Waiting all running %d queries are disconnected.. (max %d milliseconds)\n",
            current_time_us(), $#threads + 1, $time_until_kill_threads * 100;
          if ( $#threads < 5 ) {
            print Data::Dumper->new( [$_] )->Indent(0)->Terse(1)->Dump . "\n"
              foreach (@threads);
          }
        }
        sleep_until();
        $_tstart = [gettimeofday];
        $time_until_kill_threads--;
        @threads = get_threads_util( $orig_master_handler->{dbh},
          $orig_master_handler->{connection_id} );
      }
###########################################################################
      print "disable the VIP on old master: $orig_master_host \n";
      &stop_vip();
###########################################################################
      ## Terminating all threads
      print current_time_us() . " Killing all application threads..\n";
      $orig_master_handler->kill_threads(@threads) if ( $#threads >= 0 );
      print current_time_us() . " done.\n";
      $orig_master_handler->enable_log_bin_local();
      $orig_master_handler->disconnect();
      ## After finishing the script, MHA executes FLUSH TABLES WITH READ LOCK
      $exit_code = 0;
    };
    if ($@) {
      warn "Got Error: $@\n";
      exit $exit_code;
    }
    exit $exit_code;
  }
  elsif ( $command eq "start" ) {
    ## Activating master ip on the new master
    # 1. Create app user with write privileges
    # 2. Moving backup script if needed
    # 3. Register new master's ip to the catalog database
    my $exit_code = 10;
    eval {
      my $new_master_handler = new MHA::DBHelper();
      # args: hostname, port, user, password, raise_error_or_not
      $new_master_handler->connect( $new_master_ip, $new_master_port,
        $new_master_user, $new_master_password, 1 );
      ## Set read_only=0 on the new master
      $new_master_handler->disable_log_bin_local();
      print current_time_us() . " Set read_only=0 on the new master.\n";
      $new_master_handler->disable_read_only();
      ## Creating an app user on the new master
      print current_time_us() . " Creating app user on the new master..\n";
###########################################################################
      #FIXME_xxx_create_app_user($new_master_handler);
###########################################################################
      $new_master_handler->enable_log_bin_local();
      $new_master_handler->disconnect();
      ## Update master ip on the catalog database, etc
###############################################################################
      print "enable the VIP: $vip on the new master: $new_master_host \n ";
      &start_vip();
###############################################################################
      $exit_code = 0;
    };
    if ($@) {
      warn "Got Error: $@\n";
      exit $exit_code;
    }
    exit $exit_code;
  }
  elsif ( $command eq "status" ) {
    # do nothing
    exit 0;
  }
  else {
    &usage();
    exit 1;
  }
}
###########################################################################
sub start_vip() {
 `ssh $new_master_ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
 `ssh $orig_master_ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}
###########################################################################
sub usage {
  print
"Usage: master_ip_online_change --command=start|stop|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
  die;
}
```

根据自身环境修改以下内容（与 vip 脚本设置一致）

```
my $vip = '192.168.10.49/24';      # vip网段，与各节点服务器网段一致。
my $key = '1';                     # 虚拟网卡eth0:1 的 1
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";  # 网卡按照实际网卡名称填写
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";   # 网卡按照实际网卡名称填写
my $ssh_Bcast_arp= "/sbin/arping -I eth0 -c 3 -A 192.168.10.49"; # 重新声明mac地址，arp
```

#### 3.2.2 mha 配置文件

修改`mha-manager`配置文件，增加一下内容

```
vim /etc/mha/app1.cnf
master_ip_online_change_script=/usr/local/bin/master_ip_online_change
```

#### 3.2.3 关停 mha 服务

```
masterha_stop  --conf=/etc/mha/app1.cnf
```

#### 3.2.4 检查 repl

```
masterha_check_repl   --conf=/etc/mha/app1.cnf
```

#### 3.2.5 在线切换

```
masterha_master_switch  --conf=/etc/mha/app1.cnf --master_state=alive --new_master_host=192.168.10.51 --orig_master_is_new_slave --running_updates_limit=10000

From:
192.168.10.52(192.168.10.52:3306) (current master)
 +--192.168.10.51(192.168.10.51:3306)
 +--192.168.10.53(192.168.10.53:3306)

To:
192.168.10.51(192.168.10.51:3306) (new master)
 +--192.168.10.53(192.168.10.53:3306)
 +--192.168.10.52(192.168.10.52:3306)

Tue Jun  8 10:44:10 2021 - [info] Switching master to 192.168.10.51(192.168.10.51:3306) completed successfully.  # 切换成功
```

#### 3.2.6 确认 vip

```
[root@db1 ~]# ip a |grep eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.10.51/24 brd 192.168.10.255 scope global noprefixroute eth0
    inet 192.168.10.49/24 brd 192.168.10.255 scope global secondary eth0:1
```

#### 3.2.7 重构 binlog-server

1、查询当前`salve`当前拿取的`binlog`日志

```
[root@db3 ~]# mysql -uroot -p123456 -e "show slave status \G"|grep "Master_Log"
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 196
        Relay_Master_Log_File: mysql-bin.000006
          Exec_Master_Log_Pos: 196
```

2、关停当前`binlog-server`服务，清除`binlog-server`目录里拉取的日志文件

```
[root@db03 bin]# ps -ef |grep mysqlbinlog
root      28144  16272  0 17:50 pts/1    00:00:00 mysqlbinlog -R --host=192.168.10.52 --user=mha --password=x x --raw --stop-never mysql-bin.000005
root      28529  16272  0 18:03 pts/1    00:00:00 grep --color=auto mysqlbinlog
[root@db03 bin]# kill -9 28144
[root@db03 bin]# cd /data/binlog_server/
[root@db03 binlog_server]# ll
total 4
-rw-r----- 1 root root 194 Apr  1 17:50 mysql-bin.000005
[root@db03 binlog_server]# rm -rf *
[1] 28534
```

3、进入日志目录，重新启动`binlog-server`守护进程

```
[root@db03 bin]# cd /data/binlog_server/
[root@db03 binlog_server]# mysqlbinlog  -R --host=192.168.10.51 --user=mha --password=mha --raw  --stop-never mysql-bin.000009 &
```

#### 3.2.8 重新启动 mha

```
[root@db03 bin]# nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover  < /dev/null> /var/log/mha/app1/manager.log 2>&1 &

[root@db03 binlog_server]# masterha_check_status   --conf=/etc/mha/app1.cnf
app1 (pid:28535) is running(0:PING_OK), master:192.168.10.51
```