[在 Kubernetes 中部署高可用 Harbor 镜像仓库 ](https://mp.weixin.qq.com/s?__biz=MzU1MzY4NzQ1OA==&mid=2247490731&idx=1&sn=efb113d4d2d5b1afb60cbe73e65d3132&chksm=fbee5c66cc99d5706c0eba1bfb3c503d0c65d7f856f8bea4e27f6cc82284ba2c454d8b11cb25&mpshare=1&scene=24&srcid=1230kijHEcpwOtmvQhDEwN2s&sharer_sharetime=1609332725580&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)



## 1. Harbor 简介

### 简介

Harbor 是一个开放源代码容器镜像注册表，可通过基于角色权限的访问控制来管理镜像，还能扫描镜像中的漏洞并将映像签名为受信任。Harbor 是 CNCF  孵化项目，可提供合规性，性能和互操作性，以帮助跨 Kubernetes 和 Docker 等云原生计算平台持续，安全地管理镜像。

### 特性

- 管理：多租户、可扩展
- 安全：安全和漏洞分析、内容签名与验证



## 安装Harbor私有仓库

**注意：这里将Harbor私有仓库安装在Master节点（binghe101服务器）上，实际生产环境中建议安装在其他服务器。**

### 1.下载Harbor的离线安装版本

```bash
wget https://github.com/goharbor/harbor/releases/download/v1.10.2/harbor-offline-installer-v1.10.2.tgz
```

### 2.解压Harbor的安装包

```bash
tar -zxvf harbor-offline-installer-v1.10.2.tgz
```

解压成功后，会在服务器当前目录生成一个harbor目录。

### 3.配置Harbor

**注意：这里，我将Harbor的端口修改成了1180，如果不修改Harbor的端口，默认的端口是80。**

**（1）修改harbor.yml文件**

```bash
cd harbor
vim harbor.yml
```

修改的配置项如下所示。

```bash
hostname: 192.168.175.101
http:
  port: 1180
harbor_admin_password: binghe123
###并把https注释掉，不然在安装的时候会报错：ERROR:root:Error: The protocol is https but attribute ssl_cert is not set
#https:
  #port: 443
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
```

**（2）修改daemon.json文件**

修改/etc/docker/daemon.json文件，没有的话就创建，在/etc/docker/daemon.json文件中添加如下内容。

```bash
[root@binghe~]# cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://zz3sblpi.mirror.aliyuncs.com"],
  "insecure-registries":["192.168.175.101:1180"]
}
```

也可以在服务器上使用 **ip addr** 命令查看本机所有的IP地址段，将其配置到/etc/docker/daemon.json文件中。这里，我配置后的文件内容如下所示。

```bash
{
    "registry-mirrors": ["https://zz3sblpi.mirror.aliyuncs.com"],
    "insecure-registries":["192.168.175.0/16","172.17.0.0/16", "172.18.0.0/16", "172.16.29.0/16", "192.168.175.101:1180"]
}
```

### 4.安装并启动harbor

配置完成后，输入如下命令即可安装并启动Harbor

```bash
[root@binghe harbor]# ./install.sh 
```

### 5.登录Harbor并添加账户

安装成功后，在浏览器地址栏输入http://192.168.175.101:1180打开链接，如下图所示。

![img](https://img-blog.csdnimg.cn/20200521004111497.jpg)

输入用户名admin和密码binghe123，登录系统，如下图所示

![img](https://img-blog.csdnimg.cn/20200521004122673.jpg)

接下来，我们选择用户管理，添加一个管理员账户，为后续打包Docker镜像和上传Docker镜像做准备。添加账户的步骤如下所示。

![img](https://img-blog.csdnimg.cn/20200521004138579.jpg)

![img](https://img-blog.csdnimg.cn/20200521004149568.jpg)

此处填写的密码为Binghe123。

点击确定后，如下所示。
![img](https://img-blog.csdnimg.cn/20200521004202656.jpg)

此时，账户binghe还不是管理员，此时选中binghe账户，点击“设置为管理员”。

![img](https://img-blog.csdnimg.cn/20200521004213623.jpg)

![img](https://img-blog.csdnimg.cn/20200521004223621.jpg)

此时，binghe账户就被设置为管理员了。到此，Harbor的安装就完成了。

### 6.修改Harbor端口

**如果安装Harbor后，大家需要修改Harbor的端口，可以按照如下步骤修改Harbor的端口，这里，我以将80端口修改为1180端口为例**

**（1）修改harbor.yml文件**

```bash
cd harbor
vim harbor.yml
```

修改的配置项如下所示。

```bash
hostname: 192.168.175.101
http:
  port: 1180
harbor_admin_password: binghe123
###并把https注释掉，不然在安装的时候会报错：ERROR:root:Error: The protocol is https but attribute ssl_cert is not set
#https:
  #port: 443
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
```

**（2）修改docker-compose.yml文件**

```bash
vim docker-compose.yml
```

修改的配置项如下所示。

```bash
ports:
      - 1180:80
```

**（3）修改config.yml文件**

```bash
cd common/config/registry
vim config.yml
```

修改的配置项如下所示。

```bash
realm: http://192.168.175.101:1180/service/token
```

**（4）重启Docker**

```bash
systemctl daemon-reload
systemctl restart docker.service
```

**（5）重启Harbor**

```bash
[root@binghe harbor]# docker-compose down
Stopping harbor-log ... done
Removing nginx             ... done
Removing harbor-portal     ... done
Removing harbor-jobservice ... done
Removing harbor-core       ... done
Removing redis             ... done
Removing registry          ... done
Removing registryctl       ... done
Removing harbor-db         ... done
Removing harbor-log        ... done
Removing network harbor_harbor
 
[root@binghe harbor]# ./prepare
prepare base dir is set to /mnt/harbor
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/nginx/nginx.conf
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/registry/root.crt
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/jobservice/config.yml
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
 
[root@binghe harbor]# docker-compose up -d
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating harbor-db   ... done
Creating redis       ... done
Creating registry    ... done
Creating registryctl ... done
Creating harbor-core ... done
Creating harbor-jobservice ... done
Creating harbor-portal     ... done
Creating nginx             ... done
 
[root@binghe harbor]# docker ps -a
CONTAINER ID        IMAGE                                               COMMAND                  CREATED             STATUS                             PORTS
```