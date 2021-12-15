[TOC]
```shell
/lib	库文件存放目录	 /etc	存放所有系统管理所需要的配置文件,比如说mysql中的配置文件,my.conf
/usr	用户的很多应用程序和文件都放在这个目录下,有点像Windows下的program files目录
/opt	正常这个文件夹是用来放安装包的
/usr/local  install安装后的程序存放的地方
/var	存放经常需要被修改的文件,比如各种日志文件
```

```bash
du -sh filename   # 查看当前filename的文件大小
du -sh *          # 查看当前文件夹下文件的大小
ctrl + c：停止进程
ctrl + l：清屏
ctrl + r：搜索历史命令
ctrl + q：退出
df –lh     查看系统分区情况
```

# 查看文件内容
```bash
cat  filename       //打印文件内容到输出终端
more  filename     //通过敲回车方式逐行查看文件的各个行内容,默认从第一行开始查看,不支持回看,输入q 退出查看
less  filename       //通过“上下左右”键查看文件的各个部分内容,支持回看,输入q 退出查看
head  -n  filename  //查看文件的前n行内容
tail  -n  filename    //查看文件的后n行内容
wc  filename        //查看文件的行数
```

![](https://www.showdoc.cc/server/api/common/visitfile/sign/1dc80df75406c7bb74a35a7b29f64da9?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/c0cf9bf4a0827925669ae33ab0fea38d?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/82481aaee99947a1d0a49631bb281f4b?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/ebc11f2ac97c93792bbd9e5021d71238?showdoc=.jpg)

# chmod
 - u：user的缩写，是英语“用户”的意思。表示所有者。
 - g：group的缩写，是英语“群组”的意思。表示群组用户。
 - o：other的缩写，是英语“其他”的意思。表示其他用户。
 - a：all的缩写，是英语“所有”的意思。表示所有用户。

 - +：加号，表示添加权限。
 - -：减号，表示去除权限。
 - =：等号，表示分配权限。

```bash
#文件file.txt的所有者增加读和运行的权限。
chmod u+rx file.txt
#文件file.txt的群组其他用户增加读的权限。
chmod g+r file.txt
#文件file.txt的其他用户移除读的权限。
chmod o-r file.txt
#文件file.txt的群组其他用户增加读的权限，其他用户移除读的权限。
chmod g+r o-r file.txt
#文件file.txt的群组其他用户和其他用户均移除读的权限。
chmod go-r file.txt
#文件file.txt的所有用户增加运行的权限。
chmod +x file.txt
#文件file.txt的所有者分配读，写和执行的权限；群组其他用户分配读的权限，不能写或执行；其他用户没有任何权限。
chmod u=rwx,g=r,o=- file.txt
```

# 文件和目录
```bash
cd ..：返回上一级目录
cd /：进入根目录
cd ~：进入用户主目录
pwd：打印当前目录juedui路径
ls：列出当前目录中的文件
ll：列出当前目录中的文件详细信息
ls -a：显示隐藏文件
tree：显示文件和目录由根目录开始的树形结构
lstree：显示文件和目录由根目录开始的树形结构
mkdir dir1：创建一个叫做 'dir1' 的目录'
mkdir dir1 dir2：同时创建两个目录
mkdir -p /tmp/dir1/dir2：创建/tmp/dir1/dir2目录树
rm -f file1：删除一个叫做 'file1' 的文件'
rmdir dir1：删除一个叫做 'dir1' 的目录'
rm -rf dir1：删除一个叫做 'dir1' 的目录并同时删除其内容
rm -rf dir1 dir2：同时删除两个目录及它们的内容
mv dir1 dir2：重命名/移动 一个目录
```

# 文件搜索
```bash
find . -name "*.txt"：列出当前目录及子目录下所有后缀为 txt 的文件
find . -type f：列出当前目录及子目录下所有一般文件
find . -ctime -20：列出当前目录及子目录下所有最近 20 天内更新过的文件
```

# 打包和压缩文件
```bash
bunzip2 file1.bz2：解压一个叫做 'file1.bz2'的文件
bzip2 file1：压缩一个叫做 'file1' 的文件
gunzip file1.gz：解压一个叫做 'file1.gz'的文件
gzip file1：压缩一个叫做 'file1'的文件
gzip -9 file1：最大程度压缩
rar a file1.rar test_file：创建一个叫做 'file1.rar' 的包
rar a file1.rar file1 file2 dir1：同时压缩 'file1', 'file2' 以及目录 'dir1'
rar x file1.rar：解压rar包
unrar x file1.rar：解压rar包
tar -cvf archive.tar file1：创建一个非压缩的 tarball
tar -cvf archive.tar file1 file2 dir1：创建一个包含了 'file1', 'file2' 以及 'dir1'的档案文件
tar -tf archive.tar：显示一个包中的内容
tar -xvf archive.tar：释放一个包
tar -xvf archive.tar -C /tmp：将压缩包释放到 /tmp目录下
tar -cvfj archive.tar.bz2 dir1：创建一个bzip2格式的压缩包
tar -jxvf archive.tar.bz2：解压一个bzip2格式的压缩包
tar -cvfz archive.tar.gz dir1：创建一个gzip格式的压缩包
tar -zxvf archive.tar.gz：解压一个gzip格式的压缩包
zip file1.zip file1：创建一个zip格式的压缩包
zip -r file1.zip file1 file2 dir1：将几个文件和目录同时压缩成一个zip格式的压缩包
unzip file1.zip：解压一个zip格式压缩包
```

# yum相关
```bash
yum install package_name：下载并安装一个软件包
yum localinstall package_name.rpm：将安装一个软件包，使用你自己的软件仓库为你解决所有依赖关系
yum update：更新当前系统中所有安装的软件包
yum update package_name：更新一个软件包
yum remove package_name：删除一个软件包
yum list ：列出当前系统中安装的所有包
yum search package_name：在仓库中搜寻软件包
yum clean packages：清理缓存目录下软件包
yum clean headers：删除所有头文件
yum clean all： 删除所有缓存的包和头文件
```

# 查看文件内容
```bash
cat file1：从第一个字节开始正向查看文件的内容
more file1：分页查看一个长文件的内容
less file1：less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。
head -2 file1：查看一个文件的前两行
tail -2 file1：查看一个文件的最后两行
tail -f file1：实时查看一个文件中的内容
```

# 文本处理
```bash
grep test *file：当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行
grep -r update /etc/acpi：查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容
grep -v test *test*：查找文件名中包含 test 的文件中不包含test 的行
```

# 系统设置
```bash
top：实时显示 process 的动态
free -m：查看内存使用量和交换区使用量
date：显示当前时间
clear：清屏
alias lx=ls：指定lx别名为ls
bind -l：列出所有按键组合
eval：重新运算求出参数的内容
ps -ef|grep mysql：查看mysql服务进程信息
```
