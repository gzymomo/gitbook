- [centos7 离线安装gcc pcre-devel openssl-devel zlib-devel](https://www.cnblogs.com/chouc/p/7447039.html)



1、先进入http://vault.centos.org/ 选择自己的centos 版本（查看命令：cat /etc/redhat-release）

2、进入 http://vault.centos.org/**/os/x86_64/Packages/  **表示就不多解释了哈！

3、下载以下的文件  下载的时候一定要注意文件后缀里面有32位和64位的（i686 为 32，x86_64为32位）

```bash
autogen-libopts-5.18-5.el7.x86_64.rpm
cpp-4.8.2-16.el7.x86_64.rpm
gcc-4.8.2-16.el7.x86_64.rpm
glibc-devel-2.17-55.el7.x86_64.rpm
glibc-headers-2.17-55.el7.x86_64.rpm
kernel-headers-3.10.0-123.el7.x86_64.rpm
keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm
krb5-devel-1.11.3-49.el7.x86_64.rpm
libcom_err-devel-1.42.9-4.el7.x86_64.rpm
libmpc-1.0.1-3.el7.x86_64.rpm
libselinux-devel-2.2.2-6.el7.x86_64.rpm
libsepol-devel-2.1.9-3.el7.x86_64.rpm
libverto-devel-0.2.5-4.el7.x86_64.rpm
mpfr-3.1.1-4.el7.x86_64.rpm
ntp-4.2.6p5-18.el7.centos.x86_64.rpm
ntpdate-4.2.6p5-18.el7.centos.x86_64.rpm
openssl098e-0.9.8e-29.el7.centos.x86_64.rpm
openssl-1.0.1e-34.el7.x86_64.rpm
openssl-devel-1.0.1e-34.el7.x86_64.rpm
openssl-libs-1.0.1e-34.el7.x86_64.rpm
pcre-devel-8.32-12.el7.x86_64.rpm
pkgconfig-0.27.1-4.el7.x86_64.rpm
tcl-8.5.13-4.el7.x86_64.rpm
zlib-1.2.7-13.el7.x86_64.rpm
zlib-devel-1.2.7-13.el7.x86_64.rpm
```

4、scp 到 centos home目录下

5、rpm -Uvh ./*.rpm --nodeps --force