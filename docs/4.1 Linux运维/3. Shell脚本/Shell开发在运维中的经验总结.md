原文地址：https://blog.csdn.net/cpongo2ppp1/article/details/90172429

# 一、规范类Shell编写经验总结

介绍并参考一些shell编写规范，编写时严格遵守这些规范，不仅使编写人受益，同时也能提高使用者的执行效率。



## 1.1 脚本开头部分

脚本开头部分应有脚本功能说明、参数使用说明、作者姓名、创建/修改日期、版本信息，格式为:

```bash
#######################################
# 该脚本的中文功能描述
# 输入参数说明
# 参数1： .
# 参数2： .
# 作者	日期    		版本
# 张三    20210502  	 V1.0
# 修改时间	20210603    修改者  李四
# 修改内容： 修改内容描述
#######################################
```

## 1.2 脚本格式

脚本编写时，注意格式对齐，如所有的循环或者判断语句前后的语句进行对齐，以及case的选取完全，如：

```bash
case `uname` in
 AIX | HP-UX | Linux)
  if [ -d "$HDIR" ]
  then
    echo "$HDIR is OK"
  else
    mkdir -p "$HDIR"
  fi
  *) echo "Uneupported Operation System for this Script...EXITING"
    exit 1-2
esac
```



## 1.3 执行过程中若遇到使用了未定义的变量或命令返回值为非零，直接报错退出

脚本开头执行时，执行如下命令，在执行过程中若遇到使用了未定义的变量或命令返回值为非零，将直接报错退出：

```bash
set -eu
或
set -euo pipefail
```

## 1.4 参数放在单引号、双引号中

建议将命令行的每个参数放在单引号、双引号中，特别是rm、mv等可能对生产现有数据造成修改的操作，建议使用垃圾箱策略：rm操作转意为mv操作，制定文件保存目录，以防回退，并定期清理：

```bash
rm -r "$APPPATH/20200101"
rm -r `/app/data/20200101`
mv "$LOGPATH/20200101.tar" "$LOGBAKPATH"
```

## 1.5 通配符的使用

命令行中参数需要使用‘*’、‘？’通配符的，应依据最精确匹配原则，如能确定文件、目录名称的前缀、后缀、扩展名及其他可识别关键字的，须在参数中包含该信息，如能确定文件、目录的长度应使用‘？’通配符，不得使用‘*’，推荐的使用方式：

```bash
rm $APPPATH/app20????????.log
```

不推荐使用的方式：

```bash
rm $APPPATH/app*.log
rm $APPPATH/*.log
rm $APPPATH/app8
```

禁止使用的方式：

```bash
rm $APPPATH/*
rm $APPPATH/*.*
rm *
rm *.*
```

## 1.6 数值类型转换需确认

给数值型变量的赋值后，需由手段保证变量的值为数值型，避免在后续的处理中出现异常：

```bash
count=`cat app.log | wc -l`
isNumber $count
isNumber()
{
 if["$1" -eq "$1"];then true
 else
  echo "[$1] isn't number"
  exit 1
 fi
}
```

## 1.7 判断条件变量处于双引号中

在判断条件中使用的变量，必须包含在双引号中，如：

```bash
if[ "$i" -lt "$max"]
then
  break
fi
if[ -f "$conf"]
then
  break
fi
```

禁止使用的方式：

```bash
if [$i -lt $max]
then
  break
fi

if[-f $conf]
then
  break
fi
```

## 1.8 文件打包备份使用相当路径

对文件进行打包备份时，必须使用相对路径进行打包，如：

```bash
cd `/app/data/log` && tar cvf `/backup/20200101.tar` `20200101`
```

严禁将全路径打入tar包， 如：

```bash
tar cvf `/backup/20200101.tar` `/app/data/log/20200101`
```

## 1.9 文件打包压缩后使用管道进行处理

对于打包后还需进行压缩的文件，建议使用管道进行处理，如：

```bash
cd `/app/data/log` && tar cvf - '20200101' | compress -c 20200101.tar.Z
```

不建议两部分分开执行：

```bash
cd `/app/data/log` && tar cvf - '20200101'
compress -c 20200101.tar.Z
```



![Shell开发在运维中的经验总结](http://p1.pstatp.com/large/pgc-image/25b26074ad9c4c12afc90f04dcee3fb9)

## 1.10 ps筛选进程，指定用户名称

使用ps命令筛选进程时，如能确定进程所属用户，必须在参数中指定用户名称，如其输出作为kill命令的输入，则必须指定进程所属用户，如：

```bash
kill -9 `ps -fu $USERNAME | grep java | grep AdminServer`
```

# 二、易错类Shell编写编写经验总结

## 2.1 更新文件使用>不用cp

使用>修改和回退文件时，保留原文件的属组和权限，避免使用cp时权限属组被修改。

```bash
cat a.txt.bak >a.txt,避免使用cp a.txt.bak a.txt
sed -e `s/aaa/ccc/g` a.txt >a.txt.tmp cat a.txt.tmp >a.txt
```

## 2.2 使用kill前确认

关键字用-w 精确匹配字段；

kill前后都保留现场, 两次ps -ef|grep -w 关键字|grep -v grep >>/tmp/kill_进程名_.backup；

删除前要校验，获取进程号是否唯一，避免多杀或误杀的情况。

```bash
pid=`ps -ef|grep -w 关键字|grep -v grep | awk '{print $2}'`
if[ $pid -gt 1 -a $pid -lt 999999999];
then
  kill
else
  return "$pid"
fi
```

## 2.3  使用rm前确认

删除前备份删除对象信息，避免使用变量，直接使用文件和目录名；

如果必须使用时，删除前，建议检查避免误删，删除目录和文件信息保留：

```bash
文件信息备份：ls -lt /目录名（全路径）/* >> /tmp/delete_目录名_info.bak
目录信息备份：find/目录名（全路径）/xargs ls -lt >> /tmp/delete_目录名_info.bak
删除文件前检查避免误删： test -f a.log && rm -f /test/a.log
```

建议禁用find遍历根目录进行查找，同时删除前进行确认，避免多删或误删的情况。

## 2.4 For循环的坑

for循环的in条件按空格来区分，避免进入不正确或死循环。

```bash
for i in `ls *.txt`;do ...; done
# 错误写法，因为for...in语句是按照空格来分词的，包含空格的文件名会被拆成多个词。
# 正确写法
for i in *.txt;do...;done
```

## 2.5 while循环的禁忌

如果还想使用循环中的变量，不要while结合管道使用。

```bash
variable = old_value
cat file | while read line
do
	variable = new_value
done
echo $variable
=======================
输出将会是：old_value
# 有问题的写法，因为管道中的语句都会放到一个新的子shell中执行，所以子shell中定义的变量无法传递出来，循环结束变量释放，循环外无法引用。
# 如果给While传递的是非文件数据，while读取一次后就丢弃了，如果是文件数据，可供多次读取，但只要一次读取了全部内容也无法再次读取。
# 所以可以采用从文件中读取
while read line
do
...
done < /path/filename
# 或采用进程替换：
while read line
do
...
done <<(command_list)
```

## 2.6 慎用cp

这句话基本上正确，但同样有空格分词的问题。所以应当用双引号：

```bash
cp "$file" "$target"
```

但是如果凑巧文件名以 - 开头，这个文件名会被 cp 当作命令行选项来处理。

可以试试下面这个： 

```bash
cp -- "$file" "$target"
```

但也可能再碰上一个不支持 -- 选项的系统，所以最好用下面的方法： 

```bash
for i in ./*.txt; do cp "$i /target
```

## 2.7 慎用cd

避免使用cd到操作目录再操作的方式，可能导致进入目录失败，误删除，如：

```bash
cd /app/logbak
rm 20??????.tar
```

建议如下：

```bash
rm `/app/logbak/20200101.tar`
mv `~/log/20200101.tar ~/logbak/`
```

## 2.8 用[[ ]]代替[ ]

```bash
[$var="var"]
```

当$var为空时，上面的命令就变成了[ ="bar" ]

类似地，当$var包含空格时：

[ space words here = "var" ]两者都会出错。所以应当用双引号将变量括起来：

[ "$var" = var ] 几乎完美了。

但是，当$var以 - 开头时依然会有问题。在较新的bash中你可以用下面的方法来代替，[[ ]]关键字能正确处理空白、空格、带横线等问题。


![Shell开发在运维中的经验总结](http://p1.pstatp.com/large/pgc-image/2ecdef6a37d94357bcb6eefc4e8087a4)

另注意，[[适用于字符串，如果是数值，要用如：(( $var > 8 ))



## 2.9 管道操作中不要同时读写文件

```bash
cat file | sed s/old/new/ > file
```

你不能在同一条管道操作中同时读写一个文件。根据管道的实现方式，file要么被截断成0字节，要么会无限增长直到填满整个硬盘。如果想改变原文件的内容，只能先将输出写到临时文件中再用mv命令。

```bash
sed `s/old/new/g` file > tmpfile && mv tmpfile file
```

## 2.10 cd的易错问题

cd 有可能会出错，导致要执行的命令就会在你预想不到的目录里执行了。所以一定要记得判断cd的返回值。

```bash
cd /dir && test
```

如果你要根据cd的返回值执行多条命令，可以用 ||。

```bash
cd /dir || exit 1;test
```

关于目录的一点题外话，假设你要在shell程序中频繁变换工作目录，如下面的代码：

```bash
find ... -type d | while read subdir;do cd "%subdir" && ... && cd -done
```

不如这样写：

```bash
find ... -type d | while read subdir;do(cd "$subdir" && ... && ...)done
&&...)done&&...)done&&...)done
cd -done
```

括号会强制启动一个子shell，这样在这个子shell中改变工作目录不会影响父shell（执行这个脚本的shell），就可以省掉cd - 的麻烦。



# 三、应用类Shell编写经验总结

目前行里自动化工具越来越多，无论是应用的MAOP或系统的SMDB，自动化实现都还是日常运维脚本的调用，结合日常运维的一些经验，脚本中就更需要考虑周全和控制风险。这里介绍一些结合运维场景的脚本应用，希望规避以前犯过的错，重点在控制风险。

## 3.1 支持交互式脚本的应用

很多脚本中需要进行交互，在规避风险的同时，需要通过自动化工具发布来支持交互，可以使用expect，示例如下：

```bash
#!/usr/bin/expect -f
## 设置登录响应时间为5秒，如果密码过长，需要设置更大的时间
## 如果也没无手动输入，则采用send命令方式后台交互密码输入
set timeout 5
###########################################
## 把接收逇参数赋值与变量，变量设置格式set ##
## 与bash脚本不同，变量接收是从0开始	   ##
## 接收格式"[lindex $argv n]","
```



![Shell开发在运维中的经验总结](http://p9.pstatp.com/large/pgc-image/b4c591968e654146b47fbe5d58b517a2)

也可以使用curl工具来替代简单的交互：

#FTP SFTP下载

```bash
curl-u ftpuser:ftppassword -O "sftp://ftp_ip:ftp_port/pathfile"
```

#FTP SFTP上传

```bash
curl-u ftpuser:ftppassword --ftp-create-dirs-T upfile "sftp://ftp_ip:ftp_port/filepath/upfile"
```

## 3.2 脚本规范执行和日志追溯

直接执行的脚本很危险，要提示用户如何使用脚本，并记录日志以便跟踪。

示例如下：
![Shell开发在运维中的经验总结](http://p3.pstatp.com/large/pgc-image/a182d878f1df417c93ec0014747eacb6)

## 3.3 脚本的并发锁控制

避免多人同时执行或并发同时执行的异常问题，建议增加锁机制，示例如下：

![Shell开发在运维中的经验总结](http://p1.pstatp.com/large/pgc-image/f7dd2031492249df9f97bf2e680da5a2)

## 3.4 控制脚本不退出的风险

周期频繁执行的脚本，需要防止脚本hang住不退出，导致后续脚本再次执行。

![Shell开发在运维中的经验总结](http://p1.pstatp.com/large/pgc-image/af6f60fa9d35469280004b7a4abe4f76)

## 3.5 避免集中发布脚本造成的风险

使用ftp、sftp传输、下载文件，或者集中访问存储端口时，尽量增加发布对象散列，避免集中操作造成存储端口拥堵，跨防火墙流量超限报警等影响。

```bash
# 散列代码片段，不同脚本20秒内随机打散
sleeptime =`expr $RANDOM%20`
sleep${sleeptime}
if[ $? -ne 0];then
  sleep 20
fi
```

## 3.6 避免文件无限增长的风险

向一个文件中追加数据时，一定要设置阀值，必要时清空，避免文件无限增大：

![Shell开发在运维中的经验总结](http://p1.pstatp.com/large/pgc-image/3babf587e2c84dea8599f6909c6f9f21)

目录增加清理过期文件策略，避免产生的文件越来越多，造成文件节点用尽:

```bash
# 删除7天前的gzip文件
find /home/sysadmin/psinfo -mtime +7 -type f -name "*_psaux.txt.gz" -exec rm -f {} \;
```

目录中的文件过多，会报参数太长错误无法删除，建议放在循环中遍历删除：

```bash
cd /backup_databak/bak_tar
# 此目录下文件数量巨大，已无法直接删除
for i in`ls /backup_databak/bak_tar/*.tar`
do
  test -f "${i}" && rm -f "${i}"
#每次循环遍历，确认后删除
done
```

# 四、总结

鉴于以上脚本，我们可以从中汲取一些经验,规避一些风险：

通过增加日志记录输出和脚本执行的方法说明，并自动交互和传递参数，避免执行脚本的操作风险；利用文件锁机制和运维中一些规避风险的方法，使得脚本自动执行起来更便捷更安全。

1. 通过规范类脚本的定义，标准常量定义、清晰的注释、函数和变量大小写用法，细节中可以看出严谨，即使只有几行，也能体现出一名优秀脚本开发人员的素质。

2. 通过易错类脚本中的“坑”，使得 shell面向过程的编写更得心应手，让脚本规范的同时，逻辑也更严谨清晰，避免了错误,也提高了脚本的开发效率。

3. 通过运维场景的脚本应用，规避各种开发和执行过程中的风险，使得shell脚本不仅能支持自动化发布，更可以全面智能化的为运维服务。
