- [Linux 下自动化工具 Parallel SSH 中文使用指南](https://www.escapelife.site/posts/8c0f83d.html)



![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmZlhgVEZ5FJw1GQpMLhcUJUqHVgOb2FDAx3DP0J6nsrkVChzfyPqgUA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

parallel-ssh 是为小规模自动化而设计的异步并行的 SSH 库!

parallel-ssh 是为小规模自动化而设计的异步并行的 [SSH](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247511398&idx=1&sn=a307015167c24ce6fe4801aba434912a&chksm=e918c87ade6f416c117c9bcbfd6c9600700ae89b115f5703e7be4bab6478b1b53a56b48c9c5e&scene=21#wechat_redirect) 库，包括 pssh、pscp、prsync、pslurp 和 pnuke工具，其源代码使用  Python语言编写开发的。该项目最初位于Google Code上，是由Brent  N.Chun编写和维护的，但是由于工作繁忙，Brent于2009年10月将维护工作移交给了Andrew McNabb管理。到了  2012年的时候，由于Google Code的已关闭，该项目一度被废弃，现在也只能在 Google Code 的归档中找到当时的版本了。

但是需要注意的是，之前的版本是不支持 Python3 的，但是 Github 上面有人 Fork 了一份，自己进行了改造使其支持 Python3  以上的版本了。与此同时，还有一个组织专门针对 parallel-ssh  进行了开发和维护，今天看了下很久都没有更新了。有需要的，自己可以自行查阅。

- https://github.com/lilydjwg/pssh

- https://github.com/ParallelSSH/parallel-ssh

- 可扩展性

- - 支持扩展到百台，甚至上千台主机使用

- 易于使用

- - 只需两行代码，即可在任意数量的主机上运行命令

- 执行高效

- - 号称是最快的 Python SSH 库可用

- 资源使用

- - 相比于其他 Python SSH 库，其消耗资源最少

## 安装

```
# Mac系统安装
$ brew install pssh

# CentOS系统安装
$ yum install pssh

# Ubuntu系统安装
$ apt install pssh

# PIP安装
$ pip insall pssh
```

源代码编译安装(2.3.1)

```
# 官方地址: https://code.google.com/archive/p/parallel-ssh/source/default/source
$ tar zxvf pssh-2.3.1.tar.gz
$ cd pssh-2.3.1
$ python setup.py install
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmYknjPWIIp6JxRZonI9jyic2gSlYvkk7a75qd7nqewmy3eccs5cnUJsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```
# 工具对应的子命令子命令
$ ls -lh /usr/local/Cellar/pssh/2.3.1_1/bin/
pnuke -> ../libexec/bin/pnuke
prsync -> ../libexec/bin/prsync
pscp -> ../libexec/bin/pscp
pslurp -> ../libexec/bin/pslurp
pssh -> ../libexec/bin/pssh
pssh-askpass -> ../libexec/bin/pssh-askpass
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmHrN6iafsI5UZnE2WIkgp0AwZlCic0Hsiazs2KrdOPS3CQP5hbjyxeC84g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1. pssh

通过 ssh 协议在多台主机上并行地运行命令

##### 命令参数使用

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

##### 适用范例

```
# Usage: pssh [OPTIONS] command [...]

# 在两个主机上运行命令并在每个服务器上打印其输出
$ pssh -i -H "host1 host2" hostname -i

# 运行命令并将输出保存到单独的文件中
$ pssh -H host1 -H host2 -o path/to/output_dir hostname -i

# 在多个主机上运行命令并在新行分隔的文件中指定
$ pssh -i -h path/to/hosts_file hostname -i

# 以root运行命令(要求输入root用户密码)
$ pssh -i -h path/to/hosts_file -A -l root_username hostname -i

# 运行带有额外SSH参数的命令
$ pssh -i -h path/to/hosts_file -x "-O VisualHostKey=yes" hostname -i

# 运行并行连接数量限制为10的命令
$ pssh -i -h path/to/hosts_file -p 10 'cd dir; ./script.sh; exit'
```

## 2. pscp

通过 ssh 协议把文件并行地复制到多台主机上

##### 命令参数使用

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmMjK7w6JKGAKLySqJx40o743sa95TdhPwmIk60ttySVq6q92oVc1DuA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 适用范例

```
# Usage: pscp [OPTIONS] local remote

# 将本地文件复制到远程机器上
$ pscp -h hosts.txt -l root foo.txt /home/irb2/foo.txt
[1] 23:00:08 [SUCCESS] 172.18.10.25
[2] 09:52:28 [SUCCESS] 172.18.10.24
```

## 3. prsync

通过 rsync 协议把文件高效地并行复制到多台主机上

##### 命令参数使用

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmRUVkNLGYjVOos88z5PxeZnnbYKkhHjOGDNuSmfibVtic78v0khNwpS1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 适用范例

```
# Usage: prsync [OPTIONS] local remote

# 使用rsync协议进行本地文件复制操作
$ prsync -r -h hosts.txt -l root foo /home/irb2/foo
```

## 4. pslurp

通过 ssh 协议把文件并行地从多个远程主机复制到中心主机上

##### 命令参数使用

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmVlUv56qNr7pwFfpvkPDSJHicPugqV3eZtTwtS0YOCV0zDSOxDCfL3Pw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 适用范例

```
# Usage: pslurp [OPTIONS] remote local

# 将远程主机上面的文件复制到本地
$ pslurp -h hosts.txt -l root -L /tmp/outdir /home/irb2/foo.txt foo.txt
```

## 5. pnuke

通过 ssh 协议并行地在多个远程主机上杀死进程

##### 命令参数使用

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrM12CQQVCLPtqWyvdhIGxmsJbkjY9iarAXc0GnjTeO7kQvurRxhBspYfpDvVbES7VYF49jvfqeAibw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 适用范例

```
# Usage: pnuke [OPTIONS] pattern

# 结束远程主机上面的进程任务
$ pnuke -h hosts.txt -l root java
```