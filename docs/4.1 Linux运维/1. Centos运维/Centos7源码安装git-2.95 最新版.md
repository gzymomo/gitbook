[centos源码安装git-2.95 最新版](https://www.cnblogs.com/faberbeta/p/git003.html)

到 git官网下载git 源码安装包，git官网地址：https://www.git-scm.com/

选择[Tarballs](https://www.kernel.org/pub/software/scm/git/)系列的安装包，官网git下载：https://mirrors.edge.kernel.org/pub/software/scm/git/

选择最新的tar.gz结尾的安装包

如下安装脚本

vi ./install_git.sh

```bash
#!/bin/bash
yum remove git -y
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker -y
cd /usr/local/src/
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
tar zxvf git-2.9.5.tar.gz
cd git-2.9.5
./configure --prefix=/usr/local/git/
make
make install
ln -sf /usr/local/git/bin/git /bin/
ln -sf /usr/local/git/bin/git-upload-pack /bin/
ln -sf /usr/local/git/bin/git-cvsserver /bin/
ln -sf /usr/local/git/bin/gitk /bin/
ln -sf /usr/local/git/bin/git-receive-pack /bin/
ln -sf /usr/local/git/bin/git-shell /bin/
ln -sf /usr/local/git/bin/git-upload-archive /bin/
#echo 'PATH=$PATH:/usr/local/git/bin' >> /etc/profile
#source /etc/profile
```

:wq保存，之后执行bash ./install_git.sh

[root@localhost ~]git --version

git version 2.9.5