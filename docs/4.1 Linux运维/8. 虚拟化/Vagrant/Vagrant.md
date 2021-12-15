- [效率工具 | 快速创建虚拟机，Vagrant真香！](https://www.cnblogs.com/i-code/p/14239031.html)



# 一、Vagrant概述

- `Vagrant` 是一个基于`Ruby`的工具，主要用于创建和部署虚拟化开发环境。它以来于`Oracle`的开源[`VirtualBox`](https://baike.baidu.com/item/VirtualBox)虚拟化系统，通过使用 `Chef`创建自动化虚拟环境。

- ```
  Vagrant
  ```

   主要的功能如下：

  - 建立和删除虚拟机
  - 配置虚拟机相关参数
  - 管理虚拟机运行状态
  - 自动配置和安装开发环境
  - 打包和分发虚拟机运行环境

- 因为 `Vagrant` 依赖于某种虚拟化技术，目前支持常见的 `VirtualBox`、 `VMWare`等，所以在使用`Vagrant`之前我们需要先安装`VirtualBox`或 `VMWare`，不然无法使用。推荐安装 `VirtualBox`。

- `vagrant` 可以快速，方便，全自动的构建虚拟化环境，这也是我们选择它的原因，而不是让我们像以前一样全部自己来部署。

- 它类似与 `docker` 这种，有自己的仓库，我们直接可以通过命令从仓库中拉取虚拟镜像来快速构建



# 二、下载安装

- `VirtualBox`下载地址：https://www.virtualbox.org/wiki/Downloads  ，下载好后安装直接下一步操作
- `vagrant`下载地址：https://www.vagrantup.com/downloads.html ，也是直接下一步的操作完成，需要重启电脑安装完。

> **注意：**
>
> 1. 两者软件最好都下载最新的，免得出现兼容问题，
> 2. 需要安装虚拟机，需要先开启处理器虚拟化技术，***VT-x/AMD-V硬件加速**。*



# 三、Vagrant基本命令

|         命令          |                          作用                           |
| :-------------------: | :-----------------------------------------------------: |
|    vagrant box add    |                      添加box的操作                      |
|     vagrant init      |   初始化box的操作，会生成vagrant的配置文件Vagrantfile   |
|      vagrant up       |                      启动本地环境                       |
|      vagrant ssh      |             通过 ssh 登录本地环境所在虚拟机             |
|     vagrant halt      |                      关闭本地环境                       |
|    vagrant suspend    |                      暂停本地环境                       |
|    vagrant resume     |                      恢复本地环境                       |
|    vagrant reload     | 修改了 Vagrantfile 后，使之生效（相当于先 halt，再 up） |
|    vagrant destroy    |                    彻底移除本地环境                     |
|   vagrant box list    |                显示当前已经添加的box列表                |
|  vagrant box remove   |                      删除相应的box                      |
|    vagrant package    |     打包命令，可以把当前的运行的虚拟机环境进行打包      |
|    vagrant plugin     |                    用于安装卸载插件                     |
|    vagrant status     |                  获取当前虚拟机的状态                   |
| vagrant global-status |            显示当前用户Vagrant的所有环境状态            |

# 四、安装一个虚拟机案例

- 首先我们新建一个文件夹名字 `vagrant` ，这个名字随机，就是存放要新建的虚拟机的配置的目录，之后在`vagrant` 目录中打开 `cmd`或`Power Shell` 窗口，
- 执行下面命令：

> vagrant init centos/7  --box-version 2004.01

```powershell
PS D:\vagrant> vagrant init centos/7  --box-version 2004.01
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

- 上面命令执行结束后，在之下下面 `up` 命令，这个过程会去下载我们需要的镜像，是比较漫长的过程，下载完后会直接启动，`vagrant up` 命令本来就是启动命令，这是是因为没有所以会先去下载，

```powershell
PS D:\vagrant> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'centos/7' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: 2004.01
==> default: Loading metadata for box 'centos/7'
    default: URL: https://vagrantcloud.com/centos/7
==> default: Adding box 'centos/7' (v2004.01) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/centos/boxes/7/versions/2004.01/providers/virtualbox.box
Download redirected to host: cloud.centos.org
Progress: 3% (Rate: 371k/s, Estimated time remaining: 0:18:28)
```

- 当然我们也可以直接提前将镜像文件下载好，直接使用 `vagrant box add {name}  {url}` 的命令进行本地安装，其中，`{name}` 是我们要安装的名称， `url` 是我们下载到本地的镜像路径

```powershell
PS D:\vagrant> vagrant box add centos/7 E:\迅雷下载\CentOS-7-x86_64-Vagrant-1905_01.VirtualBox.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos/7' (v0) for provider:
    box: Unpacking necessary files from: file:///E:/%D1%B8%C0%D7%CF%C2%D4%D8/CentOS-7-x86_64-Vagrant-1905_01.VirtualBox.box
    box:
==> box: Successfully added box 'centos/7' (v0) for 'virtualbox'!
```

- 如果是使用本地添加的，那么这里通过 `vagrant up` 来启动，如下：

```powershell
PS D:\vagrant> vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Checking if box 'centos/7' version '2004.01' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default:
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
==> default: Rsyncing folder: /cygdrive/d/vagrant/ => /vagrant
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```

- 启动后我们可以通过 `vagrant ssh` 开启SSH，并登陆到` centos7`

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084326411-2023407233.png)

# 五、网络IP配置

- 这是一个虚拟机，那么我们要实现与宿主机器的通信，可以采用端口转发，或者独立局域网，端口转发并不方便需要我们每个端口的配置，我们这里直接采用私有网段配置，也就是桥接的方式，
- 首先我们查看自己 `Windows` 电脑的 `IP`，其中有个网卡 ` VirtualBox Host-Only Network`。，这就是虚拟机的网卡，看到其`IP`地址段

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084326927-644746264.png)

- 直接在我们刚才 `vagrant` 的目录下的 `Vagrantfile` 文件中就行配置修改，这是我们刚才创建的虚拟机的配置文件 ，配置 `config.vm.network "private_network", ip: "192.168.56.10"`，如下所示：

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084328381-1137403466.png)

- 里面可以配置很多，我们配置私有网路，刚才看到虚拟网卡网段是 `192.168.56.1`，那么我们将这台的配置为 `192.168.56.10` ，配置好之后需要重启虚拟机，通过 `vagrant reload` ，进行重启，重启后我们可以验证其与主机是否能互通

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084329576-858258675.png)

- 到此为止，我们已经配置好了虚拟机 的网络，那么我们接下来是否能通过 `Xshell` 或 `Secure CRT` 进行远程连接呢？
- 我们需要开启远程登陆，通过 `vagrant ssh` 到虚拟机，之后找到 `/etc/ssh/sshd_config` 文件修改它，通过 `sudo vi sshd_config` ,修改里面的如下两项内容，修改后直接 `wq` 保存退出`vi`

```shell
PermitRootLogin yes 
PasswordAuthentication yes
```

- 开启后，我们再重启 `SSHD` ，通过 ` systemctl restart sshd`，这时候会让你输入`root `的密码，`root` 账号的密码默认也是 `vagrant`，你可以选择直接用 `sudo` 执行。

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084330202-785856541.png)

- 在 `xshell` 下测试是否能登录

![img](https://img2020.cnblogs.com/other/2024393/202101/2024393-20210106084331107-600665565.png)