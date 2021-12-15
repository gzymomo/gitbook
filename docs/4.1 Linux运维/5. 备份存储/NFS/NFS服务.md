##  介绍

NFS就是Network File System的缩写，它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。

NFS服务器可以让PC将网络中的NFS服务器共享的目录挂载到本地端的文件系统中，而在本地端的系统中来看，那个远程主机的目录就好像是自己的一个磁盘分区一样，在使用上相当便利；

NFS一般用来存储共享视频，图片等静态数据；NFS 协议默认是不加密的，它不像 Samba，它不提供用户身份鉴别。服务端通过限定客户端的 IP 地址和端口来限制访问。



## 原理

NFS在文件传送或信息传送的过过程中，依赖于RPC协议。RPC，远程过程调用（Remote Procedure Call）,是使客户端能够执行其他系统中程序的一种机制。NFS本身是没有提供信息传输的协议和功能的，但NFS却能让我们通过网络进行资料的分享，就是因为NFS使用了RPC提供的传输协议，可以说NFS就是使用PRC的一个程序

[![nfs](https://gitee.com/owen2016/pic-hub/raw/master/pics/20201206224432.png)](https://gitee.com/owen2016/pic-hub/raw/master/pics/20201206224432.png)

1. 首先服务器端启动RPC服务，并开启111端口
2. 服务器端启动NFS服务，并向RPC注册端口信息
3. 客户端启动RPC（portmap服务），向服务端的RPC(portmap)服务请求服务端的NFS端口
4. 服务端的RPC(portmap)服务反馈NFS端口信息给客户端。
5. 客户端通过获取的NFS端口来建立和服务端的NFS连接并进行数据的传输

**注意：** 在启动NFS SERVER之前，首先要启动RPC服务（即portmap服务，下同）否则NFS SERVER就无法向RPC服务区注册，另外，如果RPC服务重新启动，原来已经注册好的NFS端口数据就会全部丢失。因此此时RPC服务管理的NFS程序也要重新启动以重新向RPC注册。

特别注意：一般修改NFS配置文档后，是不需要重启NFS的，直接在命令执行systemctl reload nfs或exportfs –rv即可使修改的/etc/exports生效

## 适用场景

- NFS 最好是部署在局域网 ，不要在公网上 ；
- 适合在中小型企业使用；大型网站不会用 NFS 的， 用的都是 分布式存储

## 安装

### NFS服务端

安装nfs-kernel-server：
`$ sudo apt install nfs-kernel-server`

### NFS客户端

安装 nfs-common：
`$ sudo apt install nfs-common`

## 配置

### 服务端配置

1. 创建共享目录

   `sudo mkdir -p /var/nfs/sharedir`

2. 配置 /etc/exports （NFS挂载目录及权限由/etc/exports文件定义）

   ```shell
   格式： 共享目录的路径 允许访问的NFS客户端（共享权限参数)
   
   参数：
   - ro 只读
   - rw 读写
   - root_squash 当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户
   - no_root_squash 当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员
   - all_squash 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户
   - sync 同时将数据写入到内存与硬盘中，保证不丢失数据
   - async 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据
   
   请注意，NFS客户端地址与权限之间没有空格
   ```

   - 比如要将`/var/nfs/sharedir`目录共享, 则在该文件末尾添加下列语句：
     /home/lin/NFSshare 192.168.66.*(rw,sync,no_root_squash)

     `/var/nfs/sharedir *(rw,sync,no_root_squash,no_subtree_check)`

   - 要限制客户端IP, 添加下列语句：

     `/var/nfs/sharedir 122.111.222.111(rw,sync,no_subtree_check)`

   - 共享目录为/public , 允许访问的客户端为192.168.245.0/24网络用户，权限为只读

     `/public 192.168.245.0/24(ro)`

3. 重新加载 nfs-kernel-server 配置：

   `$ sudo systemctl reload nfs-kernel-server`

### 客户端配置

1. 使用`showmount` 命令查看nfs服务器共享信息。

   输出格式为“共享的目录名称 允许使用客户端地址”。
   showmount命令参数：

   - -a ：显示目前主机与客户端的 NFS 联机分享的状态；
   - -e ：显示/etc/exports 所分享的目录数据。

   ```shell
   user@k8s-node-01:~$ showmount -e 192.168.249.5
   Export list for 192.168.249.5:
   /var/nfs/sharedir *
   ```

2. 在客户端创建目录，并挂载共享目录

   `$ sudo mkdir -p /mnt/nfs_sharedir`

3. 挂载远程共享目录：

   `$ sudo mount your_nfs_server_ip:/var/nfs/sharedir /mnt/nfs_sharedir`

   e.g. `sudo mount 192.168.249.5:/var/nfs/sharedir /mnt/nfs_sharedir`

   - 编辑fstab文件, 使系统每次启动时都能自动挂载

     `$ sudo vim /etc/fstab`

   添加如下:

   `your_nfs_server_Ip:/var/nfs/sharedir /mnt/nfs_sharedir nfs defaults 0 0`

4. 检查客户端挂载状态

   [![df](https://gitee.com/owen2016/pic-hub/raw/master/pics/20201206224528.png)](https://gitee.com/owen2016/pic-hub/raw/master/pics/20201206224528.png)

5. 卸载远程挂载

   `$ sudo umount /mnt/nfs_sharedir`