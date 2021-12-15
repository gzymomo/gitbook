- [docker部署mysql集群](https://www.cnblogs.com/lmhcblog/p/15082967.html)

# docker部署mysql集群

# 1.0 安装环境

## 1.1 安装Centos7

- Docker官方建议在Ubuntu中安装，因为Docker是基于Ubuntu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的。在很多版本的CentOS中是不支持更新最新的一些补丁包的。
- 如果docker安装在centos上面建议用Centos7版本,在CentOS6.x的版本中，安装前需要安装其他很多的环境而且Docker很多补丁不支持更新。

## 1.2 安装Docker

```shell
# 更新原有安装包
yum -y update
# 安装依赖
 sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 安装docker
sudo yum install docker-ce
```

### 1.21 docker常用命令

- linux 的 service 和 systemctl 命令大致区别
- 启用可以用service docker start也可以用systemctl start docker其他重启停止也可以用systemctl
- CentOS 7.x 开始，CentOS 开始使用 systemd 服务来代替 daemon，原来管理系统启动和管理系统服务的相关命令全部由 systemctl命 令来代替。
- service命令是Redhat Linux兼容的发行版中用来控制系统服务的实用工具，它以启动、停止、重新启动和关闭系统服务，还可以显示所有系统服务的当前状态。
- service启动缺点
- 一是启动时间长。init 进程是串行启动，只有前一个进程启动完，才会启动下一个进程。
- 二是启动脚本复杂。init 进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况，这往往使得脚本变得很长
- systemctl 优缺点
- Systemd 的优点是功能强大，使用方便，缺点是体系庞大，非常复杂。事实上，现在还有很多人反对使用 Systemd，理由就是它过于复杂，与操作系统的其他部分强耦合，违反 “keep simple, keep stupid” 的Unix 哲学。

```shell
# 查看docker版本
docker -v
# 启动 
service docker start
# 停止
service dockerstop
# 重启
service docker restart
# 开机启动
systemctl enable docker
# 重启docker
systemctl restart  docker
```

### 1.21 在线安装docker镜像

```shell
# 搜索java镜像
docker search java
#拉取java镜像
docker pull java
```

- docker仓库是部署在国外服务器上面的,所以如果在国内拉取镜像那将是一个非常漫长的过程,因此我们可以用一些国内的镜像仓库,比如阿里云的又或者加速器DaoCloud

### 1.22 配置Docker加速器

1. 配置阿里云镜像加速器
   1. 配置阿里云镜像加速器需要注册账号
   2. https://cr.console.aliyun.com/#/imageList)
   3. 注册之后点击左下方镜像加速器会生成一个专属加速网址
   4. 将生成的专属网址,加入/etc/docker/daemon.json即可

```shell
# 修改docker配置文件
vi /etc/docker/daemon.json
```

1. 配置Daocloud加速器
   1. 官网:https://www.daocloud.io/mirror

```shell
#配置加速器命令（复制粘贴执行即可）Ps:此命令仅克用于Linux操作系统
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

- 注意:在设置完成后,可能有一个坑存在的,执行命令设置后,它会在docker配置文件中添加一个地址,但是地址后面是多了一个,号的,需要手动删除

![img](https://www.41it.cn/wp-content/uploads/2021/07/Snipaste_2021-03-03_22-33-42.png)

- 删除配置文件中多余的,号

```shell
# 修改docker配置文件
 vi /etc/docker/daemon.json
```

### 1.23 导出和导出镜像

```shell
# 导出镜像
docker save 镜像名>导出路径
docker sava tomcat > /home/tomcat.tar.gz
# 导入镜像 
docker load<镜像文件路径
docker load < /home/mysql.tar.gz
#查看docker已有镜像
docker images
# 删除镜像
docker rmi 镜像名
docker rmi reids
#修改镜像名
docker tag 原镜像名 修改后镜像名
docker tag docker.io/percona/percona-xtradb-cluster pxc
```

### 1.24 容器相关命令

```shell
# 创建并且启动一个容器
# -it表示启动容器后开启一个交互的界面 --name 给容器起一个名字不取就没有可通过id辨别 bash代表启动程序后运行什么样的成员bash=bash命令行
docker run -it --name myTomcat tomcat bash

# 开启容器并且映射端口 -p 8088:8080代表将容器8080端口映射到宿主机8088上面 可以映射多个端口 
docker run -it --name myTomcat  -p 8088:8080 -p 8089:3306 tomcat bash

#  开启容器并且映射目录或者文件
# -v宿主机目录映射到容器中/home/data:/mysqlData冒号之前是宿主机的目录集将目录/home/data映射到/mysqlData
# --privileged这个是代表容器操作映射目录使用的是最高权限,即可读可写可执行
docker run -it --name myTomcat -v /home/data:/mysqlData --privileged tomcat  bash

# 三条命令合一
docker run -it -p 8088:8080 -p 8089:3306  -v /home/data:/mysqlData --privileged --name myTomcat tomcat bash

# 停止容器 myTomcat是容器名字没有可以通过容器id识别
docker pause myTomcat
# 恢复容器
docker unpauser myTomcat
# 彻底停止容器
docker stop myTomcat
# 重新启动容器
docker start -i myTomcat
# 退出交互页面开启容器-it执行的(同时会彻底关闭容器)
exit
# 删除容器
docker rm myTomcat
# 进入容器
docker exec -it 容器名 bash 
# 重命名容器名
docker rename 原容器名称 新容器名称
```

PS：

以上仅仅是Docker基础命令

Docker后面还有

容器数据卷

DockerFile(制作镜像使用)

Docker Compose (多容器管理)

Docker Swarm (docker集群)

# 2.0 基于Docker部署Mysql集群

### 2.01 单节点数据库的弊端

- 1. 大型互联网程序用户群体庞大,所以架构必须要特殊设计
  2. 单节点的数据库无法满足性能上的要求
  3. 单节点设计,无冗余设计,一旦数据库宕机,整个系统面临崩溃,无法满足高可用

### 2.02 常见Mysql数据库集群方案

- 常见的mysql集群有两种：
- 1. Replication
     1. 速度快，但仅能保证弱一致性，适用于保存价值不高的数据，比如日志、帖子、新闻等。
     2. 采用master-slave结构，在master写入会同步到slave，能从slave读出；但在slave写入无法同步到master。
     3. 采用异步复制，master写入成功就向客户端返回成功，但是同步slave可能失败，会造成无法从slave读出的结果。
- 1. PXC (Percona XtraDB Cluster)
     1. 速度慢，但能保证强一致性，适用于保存价值较高的数据，比如订单、客户、支付等。
     2. 数据同步是双向的，在任一节点写入数据，都会同步到其他所有节点，在任何节点上都能同时读写。
     3. 采用同步复制，向任一节点写入数据，只有所有节点都同步成功后，才会向客户端返回成功。事务在所有节点要么同时提交，要么不提交。

![img](https://www.41it.cn/wp-content/uploads/2021/07/mysql%E9%9B%86%E7%BE%A4%E7%9A%84%E5%B8%B8%E8%A7%81%E4%B8%A4%E7%A7%8D%E6%96%B9%E6%A1%88.png)

- 1. PXC既然能保障强一致性,那么当一个节点宕机了,其他节点无法向这个节点,写入数据,那是不是意味着整个,mysql集群就挂掉了呢？
  2. 其实如果说不介入中间件的情况下,确实是这样的,一旦一个节点宕机,那么其他节点都将无法写入数据,就相当于整个集群,挂了
  3. 但是采用PXC做数据库集群,肯定会采用中间件比如支持tcp协议Haproxy和Nginx
  4. 因为每个PXC节点都是可以读写的，所以SQL语句无论读写，发送哪个节点都可以执行。有一个节点挂掉也不怕，因为Haproxy有心跳检测，节点宕机，就不向这个节点发送SQL语句。当然一个Haproxy还存在宕机的问题，所以可以配置双机热备的Haproxy方案

  ## 2.1 Docker安装PXC

```shell
# docker拉去安装PXC 
# 这边使用的nysql是5.7.21版本的,我现在部署的项目使用的数据库就是这个版本,用mysql8就需要修改一些配置
# 需要注意的是，如果你使用的是mysql8,pxc是需要设置一个ssl秘钥的,否则启用第二个节点会爆错
docker pull percona/percona-xtradb-cluster:5.7.21
# docker本地安装PXC
doker load < 镜像压缩文件路径
```

### 2.12 设置PXC内部网络

```shell
# docker自带网段172.17.0....多个网段一次类推
docker network create network1
docker network create network2
docker network create network3
# 创建自定义ip 网段--subnet=172.19.0.0/24指定网段为172.19.0.0子网掩码24位
docker network create --subnet=172.19.0.0/24 network1
# 查看已经创建网段
docker network inspect 网段名
# 删除已有网段
docker network rm 网段名
```

### 2.13 创建Docker数据卷

- pxc在容器中使用是无法直接使用映射目录的,所以为它创建docker容器卷

```shell
# 创建docker卷
docker volume create --name 卷名
# 查看数据卷具体位置
docker inspect 卷名
# 删除数据卷
docker volume rm 卷名
```

### 2.14 创建PXC容器

```shell
# 创建第1个MySQL节点
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -v v1:/var/lib/mysql  --privileged --name=pxcnode1 --net=network1 --ip 172.19.0.2 pxc
# 创建第2个MySQL节点
# 需要注意,只有当第一个节点的mysql完全启动后(在本地连接成功后)才可以创建第二个,第三个
docker run -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=pxcnode1 -v v2:/var/lib/mysql   --privileged --name=pxcnode2 --net=network1 --ip 172.19.0.3 pxc
# 创建第3个MySQL节点
docker run -d -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=pxcnode1 -v v3:/var/lib/mysql   --privileged --name=pxcnode3 --net=network1 --ip 172.19.0.4 pxc
# 创建第4个MySQL节点
docker run -d -p 3309:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=pxcnode1 -v v4:/var/lib/mysql   --privileged --name=pxcnode4 --net=network1 --ip 172.19.0.5 pxc
# 创建第5个MySQL节点并映射数据库热备数据卷用于后面热备数据
docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 -e CLUSTER_NAME=PXC -e XTRABACKUP_PASSWORD=123456 -e CLUSTER_JOIN=pxcnode1 -v v5:/var/lib/mysql -v backupv:/data  --privileged --name=pxcnode5 --net=network1 --ip 172.19.0.6 pxc
```

![img](https://www.41it.cn/wp-content/uploads/2021/07/%E5%88%9B%E5%BB%BApxc%E5%AE%B9%E5%99%A8%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A32.png)

- PXC容器全部启动成功

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/pxc%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/Snipaste_2021-03-07_01-12-32.png)

- 容器启动失败

- 当容器启动失败后,通过log查看具体错误,并修改。
   ![img](https://www.41it.cn/wp-content/uploads/2021/07/pxc%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5.png)

- 当你在其中一个数据库中添加表或者数据,其他四个数据库也会同步添加

- 当你把其中一个节点关闭后,数据库将无法插入数据,（强一致性）

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/Snipaste_2021-03-07_01-22-52.png)

## 2.2 数据库负载均衡

- 复制均衡的中间件,常用的有4个

  1. Haproxy
  2. Nginx
  3. Apache
  4. LVS

- Haproxy,Nginx,Apache,LVS对比

- Nginx最近几年才支持tcp协议

- Apache不支持tcp无法使用

- LVS其实性能最好的,但是无法在虚拟机使用

- Haproxy不支持插件,但是对比其他几个,个人感觉这个比价合适

- 不使用负载均衡的弊端

  1. 所有请求都发送到一个数据库,而其他数据库,只做同步,此时数据库1负载时很高的,而数据库2,3,4却很空闲
  2. 当数据库1承受不住压力宕机了,其他节点也就无法使用
  3. 当数据库2意外宕机了,而正常处理请求的数据库1也无法在写入数据

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/%E4%B8%8D%E4%BD%BF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png)

- 使用负载均衡的优点

  1. 使用Haproxy,Haproxy会将应用程序发送的请求均匀的分发到每一个pxc节点上面,使每个节负载比较低
  2. Haproxy有心跳检测，节点宕机，就不向这个节点发送请求,即使其中一个节点意外宕机了,也不会使整个服务挂掉

​	![img](https://www.41it.cn/wp-content/uploads/2021/07/%E4%BD%BF%E7%94%A8%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png)

### 2.21 安装Haproxy

```shell
#拉去Haproxy镜像
docker pull  docker.io/haproxy:2.0
```

### 2.22  创建Haproxy配置文件

```shell
mkdir /home/soft/haproxy
vi /home/soft/haproxy/haproxy.cfg
```

- haproxy.cfg

  ```shell
  global
  	#工作目录
  	chroot /usr/local/etc/haproxy
  	#日志文件，使用rsyslog服务中local5日志设备（/var/log/local5），等级info
  	log 127.0.0.1 local5 info
  	#守护进程运行
  	daemon
  
  defaults
  	log	global
  	mode	http
  	#日志格式
  	option	httplog
  	#日志中不记录负载均衡的心跳检测记录
  	option	dontlognull
      #连接超时（毫秒）
  	timeout connect 5000
      #客户端超时（毫秒）
  	timeout client  50000
  	#服务器超时（毫秒）
      timeout server  50000
  
  #监控界面
  listen  admin_stats
  	#监控界面的访问的IP和端口
  	bind  0.0.0.0:18081
  	#访问协议
      mode        http
  	#URI相对地址
      stats uri   /dbs
  	#统计报告格式
      stats realm     Global\ statistics
  	#登陆帐户信息
      stats auth  admin:123456
  #数据库负载均衡
  listen  proxy-mysql
  	#访问的IP和端口
  	bind  0.0.0.0:3306
      #网络协议
  	mode  tcp
  	#负载均衡算法（轮询算法）
  	#轮询算法：roundrobin
  	#权重算法：static-rr
  	#最少连接算法：leastconn
  	#请求源IP算法：source
      balance  roundrobin
  	#日志格式
      option  tcplog
  	#在MySQL中创建一个没有权限的haproxy用户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
  	#server  LmhcBlogDB1 172.19.0.2:3306 check weight 1 maxconn 2000
  	#LmhcBlogDB1名字随意 172.19.0.2:3306容器ip端口	 check发送心跳监测  weight 1权重采用权重算法才会生效	maxconn 2000最大连接数	 
      option  mysql-check user haproxy
      server  LmhcBlogDB1 172.19.0.2:3306 check weight 1 maxconn 2000
      server  LmhcBlogDB2 172.19.0.3:3306 check weight 1 maxconn 2000
  	server  LmhcBlogDB3 172.19.0.4:3306 check weight 1 maxconn 2000
  	server  LmhcBlogDB4 172.19.0.5:3306 check weight 1 maxconn 2000
  	server  LmhcBlogDB5 172.19.0.6:3306 check weight 1 maxconn 2000
  	#使用keepalive检测死链
      option  tcpka
  ```

### 2.23 创建Haproxy容器

```shell
# 创建Haproxy容器
# 注意：不同版本的haproxy映射的目录是不一样的,如果容器创建成功后没有启动或者处于创建状态先去看他的log,haproxy的log会给你一个可以执行的目录
docker run -it -d -p 5000:18081 -p 5001:3306 -v /home/soft/haproxy:/usr/local/etc/haproxy --name lmhcblogh1	 --privileged --net=network1 --ip 172.19.0.7 124f5f3f731b 

#  进入容器 指明配置文件位置
docker exec -it lmhcblogh1 bash
haproxy -f /usr/local/etc/haproxy/haproxy.cfg
/usr/local/etc/haproxy

# Haproxy 后台登录
宿主机ip:5000/dbs
-- 创建haproxy账号创建这个账号是无法连接,haproxy要用这个账号去向pxc发送心跳监测
create user 'haproxy'@'%' IDENTIFIED BY '';
```

- HaproxyWeb界面

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/haproxyweb%E7%95%8C%E9%9D%A2.png)

### 2.24 Haproxy双机热备

- 单节点Haproxy是不具备高可用的 当Haproxy出现故障宕机，应用将无法在于数据库做交互，

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/%E5%8D%95%E8%8A%82%E7%82%B9Haproxy.png)

- 利用Keepalived实现双击热备

  1. Keepalived执行方式类似于线程是抢占式的，两个Keepalived抢一个虚拟Haproxy ip，其中一个Keepalived抢到虚拟ip另一个则在等待状态，两个Keepalived之间是有心跳监测的

  

- 安装Keepalived

```shell
# 进入Haproxy容器
docker exec -it lmhcblogh1 bash

# Keepalived只能安装在Haproxy容器中
# Haproxy 以Ubuntu创建出来的 Ubuntu命令以apt-get
# 更新apt-get
apt-get update
# 安装Keepalived 
apt-get install keepalived
# 安装vim编辑器用来编写keepalived配置文件
apt-get install vim
# 进入Keepalived配置文件
 vim /etc/keepalived/keepalived.conf
```

- Keepalived配置文件

  ```shell
  vrrp_instance  VI_1 {
      state  MASTER
      interface  eth0
      virtual_router_id  51
      priority  100
      advert_int  1
      authentication {
          auth_type  PASS
          auth_pass  123456
      }
      virtual_ipaddress {
          172.19.0.200
      }
  }
  ```

- 启动Keepalived

  ```shell
  service keepalived start
  ```

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/keepalived%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

- 查看Keepalived IP172.19.0..200与宿主机的之间通信是否有问题

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/%E6%9F%A5%E7%9C%8B%E9%80%9A%E4%BF%A1%E6%98%AF%E5%90%A6%E6%AD%A3%E5%B8%B8.png)

### 2.25 数据库热备份

- 常见的数据库备份方案

  1. lvm,LVM采用的是快照方式备份,备份的是分区里面存储的数据,可以兼容任何数据库,备份时加锁,只可读不可写
  2. XtraBackup,
     1. XtraBackup备份不加锁
     2. 不会打断正在执行事务
     3. 压缩备份
     4. 可增量备份（只备份变化数据）全量备份（首次备份采用全量后续采用增量）

- 安装 XtraBackup

  ```shell
  # 在前面创建pxcnode5容器是已经映射了用于备份的数据库backupv
  # pxc容器安装XtraBackup 
  # 更新atp-get
  apt-get update
  # 安装XtraBackup 
  apt-get install percona-xtrabackup-24
  # 备份数据user=数据库用户吗 password=数据库密码 备份路径/date/backup/full
  innobackupex --user=root --password=lmhcblog2020 /date/backup/full
  ```

  ![img](https://www.41it.cn/wp-content/uploads/2021/07/%E5%A4%87%E4%BB%BD%E6%88%90%E5%8A%9F.png)

### 2.26 数据还原

- 删除原有pxc容器删除数据卷并重新创建pxc容器清除数据然后还原数据

  ```shell
  #删除数据
  rm -rf /var/lib/mysql/*
  #清空事务
  innobackupex --user=root --password=abc123456 --apply-back /data/backup/full/2021-03-20-11_08-05-06/
  #还原数据
  innobackupex --user=root --password=abc123456 --copy-back  /data/backup/full/2021-03-20-11_08-05-06/
  ```

### 错误汇总

- WARNING: IPv4 forwarding is disabled. Networking will not work.

![img](https://www.41it.cn/wp-content/uploads/2021/07/ipv4notwork.png)

```shell
# 解决命令
echo "net.ipv4.ip_forward=1" >>/usr/lib/sysctl.d/00-system.conf
# 重启network 和docker 
 systemctl restart network && systemctl restart docker
```