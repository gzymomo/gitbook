# [AWK 技巧（取倒列，过滤行，匹配，不匹配，内置变量等）](https://www.cnblogs.com/kevingrace/p/8481965.html)

## **使用awk取某一行数据中的倒数第N列**：**$(NF-(n-1))**

比如取/etc/passwd文件中的第2列、倒数第1、倒数第2、倒数第4列（以冒号为分隔符）。（$NF表示倒数第一列，$(NF-1)表示倒数第二列）

```bash
[root@ipsan-node06 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
 
[root@ipsan-node06 ~]# awk -F":" '{print $2,$(NF),$(NF-1),$(NF-3)}' /etc/passwd
x /bin/bash /root 0
x /sbin/nologin /bin 1
x /sbin/nologin /sbin 2
x /sbin/nologin /var/adm 4
x /sbin/nologin /var/spool/lpd 7
x /bin/sync /sbin 0
x /sbin/shutdown /sbin 0
x /sbin/halt /sbin 0
x /sbin/nologin /var/spool/mail 12
x /sbin/nologin /root 0
```

## **linux实现将文本文件每一行中相同第一列对应的其他列进行拼接**

```bash
[root@jump-v4 ~]# sort b.txt|uniq
1    34
1    49
2    45
2    48
3    54
3    57
3    89
 
[root@jump-v4 ~]# sort b.txt|uniq|awk '{a[$1]=(a[$1]" "$2);} END{for(i in a) print i ":"a[i]}'
1: 34 49
2: 45 48
3: 54 57 89
 
命令解析：
1）首先sort test|uniq实现对test文件的去重，去掉了重复的 1 49，保留不同的行；
2）awk '{a[$1]=(a[$1]" "$2);} END{for(i in a) print i ":"a[i]}' 表示的含义是： 将每一行的第一列最为数组a的key，
   第二列作为a的value，同时碰到相同的key，就把其值进行拼接，linux的shell的字符串拼接形式为str = （str  “ ” $var），
   最后遍历数组a，其中i为数组a的每一个key，a[i]为key对应的值；
```

## **使用awk命令获取文本的某一行，某一列的技巧：**

```bash
1）打印文件的第一列(域) ： awk '{print $1}' filename
2）打印文件的前两列(域) ： awk '{print $1,$2}' filename
3）打印完第一列，然后打印第二列 ： awk '{print $1 $2}' filename
4）打印文本文件的总行数 ： awk 'END{print NR}' filename
5）打印文本第一行 ：awk 'NR==1{print}' filename
6）打印文本第二行第一列 ：sed -n "2, 1p" filename | awk 'print $1'
```

## **Awk取文件中的指定数据**

```bash
[root@jump-v4 ~]# cat a.txt
123.122.123.12 12121212
121.2332.121.11 232323
255.255.255.255 21321
123.122.123.12 12121212
123.122.123.12 1212121er2
123.122.123.12 12121212eer
123.122.123.12 12121212ere
255.255.255.255 21321
121.2332.121.11 232323
255.255.255.255 21321
 
[root@jump-v4 ~]# cat a.txt|awk '{print $1}'
123.122.123.12
121.2332.121.11
255.255.255.255
123.122.123.12
123.122.123.12
123.122.123.12
123.122.123.12
255.255.255.255
121.2332.121.11
255.255.255.255
 
[root@jump-v4 ~]# cat a.txt|awk '{print $1}'|sort|uniq -c
      2 121.2332.121.11
      5 123.122.123.12
      3 255.255.255.255
 
[root@jump-v4 ~]# cat a.txt|awk '{print $1}'|sort|uniq -c|awk '{print $2,$1}'
121.2332.121.11 2
123.122.123.12 5
255.255.255.255 3
 
[root@jump-v4 ~]# cat a.txt|awk '{print $1}'|sort|uniq -c|awk '{print $2,$1}'|sort -k2 -rn
123.122.123.12 5
255.255.255.255 3
121.2332.121.11 2
```

## **linux文件按大小来排序**

```bash
[root@cdn ~]# ls -s | sort -k 1 -n
表示对第一个字段(即文件大小)按数值大小进行排序；
如果想倒序，可以增加-r参数；
sort命令可进行排序；
-k参数表示对第几个字段进行排序；
ls -s:第一列显示的是文件大小
```

## **定时删除resin日志的脚本，每小时删除一次**

```bash
[root@cdn ~]# cat resin-log.sh
#!/bin/bash
cd /data/log/resin && find /data/log/resin \( -name "*jvm-app-0.log.*" -a ! -name "*.gz" \) -a -mmin +30 -exec gzip  {} \;
 
[root@cdn ~]# crontab -l
0 * * * * /bin/bash -x /root/resin-log.sh >/dev/null 2>&1
```

## **awk 获取某些列的某些行（打印或不打印第几行）**

```bash
NR==n 表示打印第n行
NR!=n 表示不打印第n行
 
1）取test.txt文件中的第1，2列，不打印第一行
[root@bz4citestap1014 app_zhibiao.sh]# cat test.txt
wang 11 aa
shi  22 bb
kevin 33 cc
grace 44 dd
hui 55 ee
[root@bz4citestap1014 app_zhibiao.sh]# awk 'NR!=1 {print $1,$2}' test.txt
shi 22
kevin 33
grace 44
hui 55
 
2）取test.txt文件中的第3列的第2行
[root@bz4citestap1014 app_zhibiao.sh]# awk 'NR==2 {print $3}' test.txt 
bb
```

## **awk中的"匹配"与"不匹配"**

```bash
~ 匹配正则
!~ 不匹配正则
== 等于
!= 不等于
 
[root@kevin~]# cat test.txt
afjdkj 80
lkdjfkja 8080
dfjj 80
jdsalfj 808080
jasj 80
jg 80
linuxidc 80
80 ajfkj
asf 80
80 linuxidc
wang bo
kevin grace
ha  80880
 
1) 打印上面test文件中第二列匹配80开头并以80结束的行
[root@kevin~]# awk '{if($2~/^80$/)print}' test.txt
afjdkj 80
dfjj 80
jasj 80
jg 80
linuxidc 80
asf 80
 
2）打印上面test文件中第二列中不匹配80开头并以80结束的行
[root@kevin~]# awk '{if($2!~/^80$/)print}' test.txt
lkdjfkja 8080
jdsalfj 808080
80 ajfkj
80 linuxidc
wang bo
kevin grace
ha  80880
 
3）打印上面test文件中第二列是"bo"的行
[root@kevin~]# cat test.txt |awk '{if($2=="bo")print}'
wang bo
```

## **AWK的内置变量（NF、NR、FNR、FS、OFS、RS、ORS）**

```bash
NF   字段个数，（读取的列数）
NR   记录数（行号），从1开始，新的文件延续上面的计数，新文件不从1开始
FNR  读取文件的记录数（行号），从1开始，新的文件重新从1开始计数
FS   输入字段分隔符，默认是空格
OFS  输出字段分隔符 默认也是空格
RS   输入行分隔符，默认为换行符
ORS  输出行分隔符，默认为换行符
 
示例文件test：
[rootkevin ~]# cat test
zhong guo ren is noce!
beijing is a good city。
sheg as juf 88u kk
halt:x:7:0:halt /sbin:/sbin/halt
operator x 0:operator /root:/sbin/nologin
 
1）NF：读取记录的字段数(列数)
[rootkevin ~]# awk -F" " '{print "字段数: " NF}' test
字段数: 5
字段数: 5
字段数: 5
字段数: 2
字段数: 4
 
如上，awk在读取文件时，按行读取，每一行的字段数(列数)，赋值给内置变量NF，打印出来的就是每行的字段总数。
 
[rootkevin ~]# awk '{print $NF}' test
noce!
city。
kk
/sbin:/sbin/halt
/root:/sbin/nologin
 
如果有需求，只需要最后一列的数据，由于每一行的列数不一，最后一列无法指定固定的列数，可以使用NF来表示列数$NF表示打印出等于总列数的那一列的数据，
显而易见就是打印最后一列的数据。
 
2）NR：读取文件的行数(在某些应用场景中可以当作行号来使用)
[rootkevin ~]# awk '{print "行号为：" NR}' test
行号为：1
行号为：2
行号为：3
行号为：4
行号为：5
 
如上，打印出读取文件的行数，因为是按行读取，在应用场景中，行数可以等同于行号，用来输出对应行的行号，NR 还可以用作判断输出，如下简单例子：
[rootkevin ~]# awk '{if(NR>2)print "行号为：" NR }' test
行号为：3
行号为：4
行号为：5
 
3）FNR：读取文件的行数，但是和"NR"不同的是当读取的文件有两个或两个以上时，NR读取完一个文件，行数继续增加 而FNR重新从1开始记录
[rootkevin ~]# cp test test1
[rootkevin ~]# awk '{print "NR:"NR "FNR:"FNR}' test test1     
NR:1FNR:1
NR:2FNR:2
NR:3FNR:3
NR:4FNR:4
NR:5FNR:5
NR:6FNR:1
NR:7FNR:2
NR:8FNR:3
NR:9FNR:4
NR:10FNR:5
 
打印的两列之间加上空格
[rootkevin ~]# awk '{print "NR:"NR " " "FNR:"FNR}' test test1
NR:1 FNR:1
NR:2 FNR:2
NR:3 FNR:3
NR:4 FNR:4
NR:5 FNR:5
NR:6 FNR:1
NR:7 FNR:2
NR:8 FNR:3
NR:9 FNR:4
NR:10 FNR:5
 
由上可知，NR从一开始一直增加，FNR每读取到一个新的文件，行数重新从一开始增加。
 
有一个有趣的应用，比较两个文件A，B是否一致，以A作为参考，不一致的输出行号
[rootkevin ~]# cat A
a aa aaa 1
b bb bbb 2
c cc ccc
d dd ddd 4
e ee eee 5
 
[rootkevin ~]# cat B
a aa aaa 1
b bb bbb 2
c cc ccc 3
d dd ddd 4
e ee eee 5
 
[rootkevin ~]# awk '{if(NR==FNR){arry[NR]=$0}else{if(arry[FNR]!=$0){print FNR}}}' A B
3
 
4）FS：输入字段分割符，默认是以空格为分隔符，在日常中常常文本里面不都以空格分隔，此时就要指定分割符来格式化输入。
[rootkevin ~]# cat test2
a,b,c
1,2,3
aa,dd,ee
[rootkevin ~]# awk '{print $1}' test2            
a,b,c
1,2,3
aa,dd,ee
 
[rootkevin ~]# awk 'BEGIN{FS=","}{print $1}' test2
a
1
aa
 
使用-F参数也可以
[rootkevin ~]# awk -F"," '{print $1}' test2
a
1
aa
 
5）OFS：输出字段分割符，默认为空格，如果读进来的数据是以空格分割，为了需求可能要求输出是以"-"分割，可以使用OFS进行格式化输出。
[rootkevin ~]# cat test3
a aa aaa 1
b bb bbb 2
c cc ccc
d dd ddd 4
e ee eee 5
[rootkevin ~]# awk 'BEGIN{FS=" ";OFS="--"}{print $1,$2,$3}' test3
a--aa--aaa
b--bb--bbb
c--cc--ccc
d--dd--ddd
e--ee--eee
 
[rootkevin ~]# awk -vOFS="|" 'NF+=0' test3
a|aa|aaa|1
b|bb|bbb|2
c|cc|ccc
d|dd|ddd|4
e|ee|eee|5
 
[rootkevin ~]# cat test6
172.10.10.10
172.10.10.11
172.10.10.12
172.10.10.13
172.10.10.14
[rootkevin ~]# awk 'BEGIN{FS=".";OFS="--"}{print $1,$2,$3}' test6     
172--10--10
172--10--10
172--10--10
172--10--10
172--10--10
 
6）RS：输入行分隔符，判断输入部分的行的起始位置，默认是换行符
[rootkevin ~]# cat test4
a,b,c
d,e,f
g,h,i
j,k,l
[rootkevin ~]# awk 'BEGIN{RS=","}{print}' test4
a
b
c
d
e
f
g
h
i
j
k
l
 
[rootkevin ~]#
 
这里说明一下，RS=","将以,为分割当作一行，即a被当作一行，b也被当作一行，但是细心的会发现和d之间是没有","的为什么也当作一行了呢，
是因为输入中c后面还有一个换行符\n 即,输入应该是a,b,c\n只不过\n我们看不到,输入中，a一行，b一行，c\nd一行但是输出的时候系统会将\n视为换行符，
所以看上去c和d是两行，实际上是一行。
 
7）ORS：输出行分割符，默认的是换行符,它的机制和OFS机制一样，对输出格式有要求时，可以进行格式化输出
[rootkevin ~]# cat test5
1 22,aa:bb
haha,hehe
aa bb cc
[rootkevin ~]# awk 'BEGIN{ORS=" "}{print}' test5
1 22,aa:bb haha,hehe aa bb cc
 
[rootkevin ~]# cat test6
172.10.10.10
172.10.10.11
172.10.10.12
172.10.10.13
172.10.10.14
[rootkevin ~]# awk 'BEGIN{ORS=","}{print}' test6
172.10.10.10,172.10.10.11,172.10.10.12,172.10.10.13,172.10.10.14,
 
也可以如下实现以","隔开放在一行
[rootkevin ~]# cat test6|xargs
172.10.10.10 172.10.10.11 172.10.10.12 172.10.10.13 172.10.10.14
[rootkevin ~]# cat test6|xargs|sed 's/ /,/g'
172.10.10.10,172.10.10.11,172.10.10.12,172.10.10.13,172.10.10.14
```

## **AWK对文件的"某一列进行去重"的做法   (命令: awk '{a[$n]=$0}END{for(i in a)print a[i]}' filename)**

```bash
命令：awk '{a[$n]=$0}END{for(i in a)print a[i]}' filename
解释：对filename文件的第n列进行去重
 
举例：
1）对kevin.txt文件的第一列进行去重
[root@bobo tmp]# cat kevin.txt
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:30:40
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:31:14
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:31:13
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:31:13
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:31:12
FFM_理财平台系统 FFM_scial 2019-11-09 11:34:37
ASI_账管服务整合 ASI-OPsmart 2019-11-09 13:12:34
ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53
IID_智惠存 IID_wmpayment 2019-11-12 15:38:53
SIX_安全基础工具 SIX_microservice_config 2019-11-11 19:34:45
DDI_茹能 DDI_from 2019-11-13 21:09:13
FFM_理财平台系统 FFM_scial 2019-11-13 21:27:08
SCC_信贷系统 SCC-index 2019-11-12 21:29:59
GGA_账务管理中心 GGA_IFPmar 2019-11-13 22:01:48
UPI_智能用户平台 UPI_CMSO 2019-11-13 22:23:26
UPI_智能用户平台 UPI_CMSO 2019-11-13 22:51:13
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-13 10:10:16
FFM_理财平台系统 FFM_scial 2019-11-08 17:17:04
MPB_手机银行APP MPB_bizzManagement 2019-11-08 18:49:27
SIX_安全基础工具 SIX_microservice_config 2019-11-12 15:50:57
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22
CIM_渠道内部管理系统 CIM_cimservice 2019-11-13 17:06:27
CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26
ASI_账管服务整合 ASI-OPsmart 2019-11-13 19:34:07
 
[root@bobo tmp]# awk '{a[$1]=$0}END{for(i in a)print a[i]}' kevin.txt
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22
ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53
CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26
ASI_账管服务整合 ASI-OPsmart 2019-11-13 19:34:07
FFM_理财平台系统 FFM_scial 2019-11-08 17:17:04
SIX_安全基础工具 SIX_microservice_config 2019-11-12 15:50:57
ABB-仓库系统 ABB-pay-ce 2019-11-08 23:31:12
UPI_智能用户平台 UPI_CMSO 2019-11-13 22:51:13
DDI_茹能 DDI_from 2019-11-13 21:09:13
MPB_手机银行APP MPB_bizzManagement 2019-11-08 18:49:27
GGA_账务管理中心 GGA_IFPmar 2019-11-13 22:01:48
SCC_信贷系统 SCC-index 2019-11-12 21:29:59
 
2）对test.txt文件的第三列进行去重
[root@bobo tmp]# cat test.txt
1 anhui wangbo 90
2 henan hexin  78
3 shenzhen wangbo 89
4 shanghai zhoumen 98
5 liuzhou  hexin  96
6 xinhuang wangbo 77
7 suzhou  zhupin  95
8 ningbo  niuping 100
9 chongqing wangbo 93
10 meizhou  lishuyan 98
 
[root@bobo tmp]# awk '{a[$3]=$0}END{for(i in a)print a[i]}' test.txt
8 ningbo  niuping 100
9 chongqing wangbo 93
4 shanghai zhoumen 98
7 suzhou  zhupin  95
5 liuzhou  hexin  96
10 meizhou  lishuyan 98
```

## **AWK 将列转为行的做法**

```bash
1）如下，将a1.txt文件中的列转为行，并用逗号隔开
[root@bobo tmp]# cat a1.txt              
1
2
3
4
5
 
[root@bobo tmp]# awk '{printf "%s,",$1}' a1.txt
1,2,3,4,5,
 
上面列转为行后，去掉最后一个逗号
[root@bobo tmp]# awk '{printf "%s,",$1}' a1.txt|sed 's/.$//'
1,2,3,4,5
[root@bobo tmp]# awk '{printf "%s,",$1}' a1.txt | awk '{sub(/.$/,"")}1'
1,2,3,4,5
# awk '{printf "%s,",$1}' a1.txt | awk '{printf $0"\b \n"}'
1,2,3,4,5
 
也就是说，shell去掉最后一个字符，有下面三种方式实现：
sed 's/.$//'
awk '{sub(/.$/,"")}1'
awk '{printf $0"\b \n"}' ufile
 
 
2）如下，将a2.txt文件中的列转为行，并用冒号隔开。
[root@bobo tmp]# cat a2.txt
a 1
b 2
c 3
d 4
 
注意下面实现的几种效果。
可以用$1,$2，...$n，也可以使用$0表示文件全部列转为行。
每列转为行后，行与行之间的隔开形式
[root@bobo tmp]# awk '{printf "%s,",$1$2}' a2.txt 
a1,b2,c3,d4,
 
[root@bobo tmp]# awk '{printf "%s,",$1,$2}' a2.txt 
a,b,c,d,
 
[root@bobo tmp]# awk '{printf "%s,",$1" "$2" "}' a2.txt
a 1 ,b 2 ,c 3 ,d 4 ,
 
[root@bobo tmp]# awk '{printf "%s,",$0}' a2.txt          
a 1,b 2,c 3,d 4,
 
[root@bobo tmp]# awk '{printf "%s,",$0}' a2.txt |sed 's/.$//'
a 1,b 2,c 3,d 4
 
3）将下面test.list文件中的列转为行，并逗号隔开
[root@bobo tmp]# cat test.list
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22
ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53
CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26
 
下面两种方式实现效果一样
[root@bobo tmp]# awk '{printf "%s,",$1" "$2" "$3" "$4}' test.list
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22,ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53,CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26,
 
[root@bobo tmp]# awk '{printf "%s,",$0}' test.list               
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22,ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53,CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26,
 
去掉最后一个逗号
[root@bobo tmp]# awk '{printf "%s,",$0}' test.list|sed 's/.$//'
PMS_项目信息管理系统 PMS_PMSConsole 2019-11-08 21:27:22,ICS_智能客服系统 ICS_basic-pass 2019-11-11 15:01:53,CIM_渠道内部管理系统 CIM_cimbizz 2019-11-13 17:06:26
```

## **shell将多行转为一行（或将多行中的某一列转化为行）的做法**

```bash
常习惯于使用xargs将多行转为一行，示例如下：
1）将ip.list文件中的每行ip转化到一行里面，并使用逗号隔开：
[root@localhost ~]# cat ip.list
192.168.10.10
192.168.10.11
192.168.10.12
192.168.10.13
192.168.10.14
192.168.10.15
192.168.10.16
192.168.10.17
192.168.10.18
192.168.10.19
192.168.10.20
 
使用xargs命令就会将管道符|前面输出内容放在一行，并默认使用空格隔开
[root@localhost ~]# cat ip.list|xargs
192.168.10.10 192.168.10.11 192.168.10.12 192.168.10.13 192.168.10.14 192.168.10.15 192.168.10.16 192.168.10.17 192.168.10.18 192.168.10.19 192.168.10.20
 
再结合sed将空格替换为逗号
[root@localhost ~]# cat ip.list|xargs|sed -i 's/ /,/g'
sed: no input files
xargs: echo: terminated by signal 13
 
注意：
sed使用-i参数时，后面必须要跟具体的文件名，-i参数表示替换效果已在文件中生效！
如果不使用-i参数，则表示替换效果仅仅在当前终端展示里生效，并不会在文件中生效！
[root@localhost ~]# cat ip.list|xargs|sed 's/ /,/g'  
192.168.10.10,192.168.10.11,192.168.10.12,192.168.10.13,192.168.10.14,192.168.10.15,192.168.10.16,192.168.10.17,192.168.10.18,192.168.10.19,192.168.10.20
 
注意：
这里替换结果不能直接重定向到原来的文件ip.list中，因为前面cat命令正在读，这里如果将替换结果重定向到ip.list文件中，会造成ip.list文件为空！
应该重定向到别的一个文件中，然后再mv到原来的ip.list文件
[root@localhost ~]# cat ip.list|xargs|sed 's/ /,/g' > ip.list_tmp
[root@localhost ~]# mv ip.list_tmp ip.list
mv: overwrite ‘ip.list’? y
[root@localhost ~]# cat ip.list
192.168.10.10,192.168.10.11,192.168.10.12,192.168.10.13,192.168.10.14,192.168.10.15,192.168.10.16,192.168.10.17,192.168.10.18,192.168.10.19,192.168.10.20
 
2）将test.txt文件中的内容放在一行，并使用<<<<<<隔开
[root@localhost ~]# cat test.txt
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
 
[root@localhost ~]# cat test.txt|xargs|sed 's/ /<<<<<</g'          
root:x:0:0:root:/root:/bin/bash<<<<<<bin:x:1:1:bin:/bin:/sbin/nologin<<<<<<daemon:x:2:2:daemon:/sbin:/sbin/nologin<<<<<<adm:x:3:4:adm:/var/adm:/sbin/nologin
 
3）将kevin.txt文件中的第二列内容放在一行，并使用分开隔开
[root@localhost ~]# cat kevin.txt
wangbo 90 abc
zhangkai 93 ccs
liuru 88 ffn
mamin 95 efe
huomei 85 cbs
haoke 91 mmn
 
[root@localhost ~]# cat kevin.txt|awk '{print $2}'|xargs
90 93 88 95 85 91
[root@localhost ~]# cat kevin.txt|awk '{print $2}'|xargs|sed 's/ /;/g'
90;93;88;95;85;91
 
再看看下面的转化
[root@localhost ~]# cat kevin.txt
wangbo 90 abc
zhangkai 93 ccs
liuru 88 ffn
mamin 95 efe
huomei 85 cbs
haoke 91 mmn
 
[root@localhost ~]# cat kevin.txt|awk '{print $1":"$2}'
wangbo:90
zhangkai:93
liuru:88
mamin:95
huomei:85
haoke:91
 
[root@localhost ~]# cat kevin.txt|awk '{print $1":"$2}'|xargs
wangbo:90 zhangkai:93 liuru:88 mamin:95 huomei:85 haoke:91
 
[root@localhost ~]# cat kevin.txt|awk '{print $1":"$2}'|xargs|sed 's/ /,/g'
wangbo:90,zhangkai:93,liuru:88,mamin:95,huomei:85,haoke:91
```

## **shell去掉最后一个字符的做法**

```bash
删除最后的那个字符，下面三种方法可以实现：
1）sed 's/.$//'
2）awk '{sub(/.$/,"")}1'
3）awk '{printf $0"\b \n"}' ufile
 
举例如下：
1）删除test.txt 文件中所有行的最后一个字符
[root@bobo tmp]# cat test.txt
www.kevin.com/
www.haha.com//uh
www.hehe.com/a
 
[root@bobo tmp]# cat test.txt|sed 's/.$//'
www.kevin.com
www.haha.com//u
www.hehe.com/
 
[root@bobo tmp]# cat test.txt|awk '{sub(/.$/,"")}1'
www.kevin.com
www.haha.com//u
www.hehe.com/
 
[root@bobo tmp]# awk '{printf $0"\b \n"}' test.txt
www.kevin.com
www.haha.com//u
www.hehe.com/
 
2）删除bo.txt 文件中所有行的最后一个字符
[root@bobo tmp]# cat bo.txt
192.168.10.154
192.168.10.159
192.168.10.160
model_C
model_D
stop_time_out=120
start_time_out=400
 
[root@bobo tmp]# cat bo.txt|sed 's/.$//'
192.168.10.15
192.168.10.15
192.168.10.16
model_
model_
stop_time_out=12
start_time_out=40
 
[root@bobo tmp]# cat bo.txt|awk '{sub(/.$/,"")}1'
192.168.10.15
192.168.10.15
192.168.10.16
model_
model_
stop_time_out=12
start_time_out=40
 
[root@bobo tmp]# awk '{printf $0"\b \n"}' bo.txt
192.168.10.15
192.168.10.15
192.168.10.16
model_
model_
stop_time_out=12
start_time_out=40
```

