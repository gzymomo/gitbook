# 一、官方方式安装Docker Compose

Linux 上我们可以从 Github 上下载它的二进制包来使用，最新发行的版本地址：https://github.com/docker/compose/releases。

运行以下命令以下载 Docker Compose 的当前稳定版本：

```bash
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

要安装其他版本的 Compose，请替换 1.24.1。

将可执行权限应用于二进制文件：

```bash
chmod +x /usr/local/bin/docker-compose
```

创建软链：

```bash
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```bash
docker-compose --version
```

# 二、通过pip方式安装Docker Compose

安装pip

```bash
yum -y install epel-release
yum -y install python-pip
```

查看版本

```bash
pip --version
```

更新pip

```bash
pip install --upgrade pip
```

安装docker-compose

```bash
pip install docker-compose
```

查看docker compose的版本

```bash
docker-compose version
```



# 三、离线安装

访问https://github.com/docker/compose/releases，下载  docker-compose-Linux-x86_64，我是复制链接地址，在迅雷中下载的，下载后，将docker-compose-Linux-x86_64重命名为docker-compose
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200104121440853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l0YW5nZGlnbA==,size_16,color_FFFFFF,t_70)
 通过ssh工具MobaXterm，将刚才下载的docker-compose文件上传到centos7的/usr/local/bin/目录下
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200104121805207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3l0YW5nZGlnbA==,size_16,color_FFFFFF,t_70)
 如上图，输入以下命令 添加可执行权限和查看docker compose版本

```bash
# 添加可执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 查看docker-compose版本
docker-compose -v
```