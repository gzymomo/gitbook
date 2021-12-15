- [Nginx日志运维笔记](https://www.cnblogs.com/kevingrace/p/8483089.html)

在分析服务器运行情况和业务数据时，nginx日志是非常可靠的数据来源，而掌握常用的nginx日志分析命令的应用技巧则有着事半功倍的作用，可以快速进行定位和统计。

## Nginx日志的标准格式

- （可参考：http://www.cnblogs.com/kevingrace/p/5893499.html）

```yaml
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' 
             '$status $body_bytes_sent "$http_referer" ' 
             '"$http_user_agent" $request_time';
```

记录的形式如下： 

```yaml
192.168.28.22 - - [28/Feb/2018:04:01:11 +0800] "GET /UserRecommend.php HTTP/1.1" 200 870 "http://wwww.kevin.www/grace/index.html"
"Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30)" 320
```

### **日志格式说明：**

- $remote_addr       远程请求使用的IP地址
- $remote_user       远端登录名
- $time_local         时间，用普通日志时间格式(标准英语格式)
- $request           请求的第一行
- $status            状态
- $body_bytes_sent   请求返回的字节数，包括请求头的数据
- $http_referer       请求头Referer的内容
- $http_user_agent   请求头User-Agent的内容
- $request_time      处理完请求所花时间，以秒为单位



### **Apache日志的标准格式**

```yaml
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %T " combined 
    CustomLog log/access_log combined
```

记录的形式如下：

```yaml
192.168.28.23 - frank [28/Feb/2018:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html"
"Mozilla/4.08 [en] (Win98; I ;Nav)"
```

### **日志格式说明：**

%h   请求使用的IP地址
%l   远端登录名(由identd而来，如果支持的话)，除非IdentityCheck设为"On"，否则将得到一个"-"。
%u   远程用户名(根据验证信息而来；如果返回status(%s)为401，可能是假的)
%t   时间，用普通日志时间格式(标准英语格式)
%r   请求的第一行
%s   状态。对于内部重定向的请求，这个状态指的是原始请求的状态，---%>s则指的是最后请求的状态。
%b   以CLF格式显示的除HTTP头以外传送的字节数，也就是当没有字节传送时显示'-'而不是0。
\"%{Referer}i\"   发送到服务器的请求头Referer的内容。
\"%{User-Agent}i\"   发送到服务器的请求头User-Agent的内容。
%T   处理完请求所花时间，以秒为单位。
%I    接收的字节数，包括请求头的数据，并且不能为零。要使用这个指令你必须启用mod_logio模块。
%O   发送的字节数，包括请求头的数据，并且不能为零。要使用这个指令你必须启用mod_logio模块。

### **Nginx 日志字段解释**

| **说明**                       | **字段名**              | **示例**                                                     |
| ------------------------------ | ----------------------- | ------------------------------------------------------------ |
| 主机头                         | $host                   | 域名 kevin.bo.com                                            |
| 服务器ip                       | $server_addr            | 192.168.10.109                                               |
| 端口                           | $server_port            | 80                                                           |
| 客户ip                         | $remote_addr            | 172.17.12.18                                                 |
| 客户                           | $remote_user            | -                                                            |
| 时间                           | $time_iso8601           | 2018-11-04T10:13:40+09:00                                    |
| 状态码                         | $status                 | 204                                                          |
| 发送主体大小                   | $body_bytes_sent        | 0                                                            |
| 发送总大小                     | $bytes_sent             | 140                                                          |
| 请求总大小                     | $request_length         | 578                                                          |
| 请求主体大小                   | $request_body           | -                                                            |
| 请求时间                       | $request_time           | 0.001                                                        |
| 请求方式                       | $request_method         | GET                                                          |
| uri                            | $uri                    | /rest/quickreload/latest/18747370                            |
| 变量                           | $args                   | since=1559180602998&_=1559181197999                          |
| 协议                           | $server_protocol        | HTTP/1.1                                                     |
| cookie                         | $cookie_nid             | -                                                            |
| 记录从哪个页面链接访问过来     | $http_referer           | http://kevin.bo.com/pages/viewpage.action?pageId=18747370    |
| 客户端信息                     | $http_user_agent        | Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36 |
| 客户端真实ip（经过反向代理）   | $http_x_forwarded_for   | -                                                            |
| 编码                           | $http_accept_encoding   | gzip, deflate                                                |
| 当前通过一个连接获得的请求数量 | $connection_requests    | 1                                                            |
| 后端ip                         | $upstream_addr          | 192.168.10.33:8090                                           |
| 后端状态码                     | $upstream_status        | 204                                                          |
| 后端响应时间                   | $upstream_status        | 0.001                                                        |
| 后台缓存                       | $upstream_cache_status  | -                                                            |
| 后端接口状态                   | $upstream_http_x_status | -                                                            |

### **配置文件**

```yaml
log_format main '$host\t$server_addr\t$server_port\t$remote_addr\t'
'$remote_user\t$time_iso8601\t$status\t'
'$body_bytes_sent\t$bytes_sent\t$request_length\t'
'$request_body\t$request_time\t$request_method\t'
'$uri\t$args\t$server_protocol\t$cookie_nid\t'
'$http_referer\t$http_user_agent\t$http_x_forwarded_for\t'
'$http_accept_encoding\t$connection_requests\t$upstream_addr\t'
'$upstream_status\t$upstream_response_time\t$upstream_cache_status\t$upstream_http_x_status'; 
```



## **Nginx日志切割**

```bash
#!/bin/sh  
# 设置日志文件备份文件名   
#logfilename=`date +%Y%m%d` 
logfilename=`date +\%Y\%m\%d -d "1 days ago"` 
# 设置日志文件原始路径   
logfilepath=/opt/nginx/logs/ 
# 设置日志备份文件路径 
backupfilepath=/opt/data/logs/nginx/ 
   
LOG_FILE='access error log_mm log_db' 
for j in $LOG_FILE 
do 
        cd ${logfilepath} 
        tar zcvf ${backupfilepath}$j/${logfilename}.tar.gz $j.log 
        rm -rf $j.log 
done 
   
kill -USR1 `cat  /opt/nginx/nginx.pid 
```

## **apache日志切割**

```bash
#!/bin/bash 
# 获取昨天的日期 
logfilename=`date -d yesterday +%Y_%m_%d` 
today=`date +%Y.%m.%d` 
# 设置日志文件原始路径   
logfilepath=/opt/apache2/logs/ 
# 设置日志备份文件路径 
backupfilepath=/opt/data/logs/apache/ 
   
echo "get access log:" 
# 打包压缩访问日志文件 
cd ${logfilepath} 
tar zcvf ${backupfilepath}access/${logfilename}.tar.gz access_${logfilename}.log 
rm -rf access_${logfilename}.log 
   
echo "get error log:" 
# 打包压缩错误日志文件 
cd ${logfilepath} 
tar zcvf ${backupfilepath}error/${logfilename}.tar.gz error_${logfilename}.log 
rm -rf error_${logfilename}.log 
   
echo "done @"${today}
```

## **日志定时清理的脚本**

```bash
#!/bin/sh 
####################### clear logs ######################### 
### nginx ### 
#clear nginx access log(by hour .log) 2 days ago  
/usr/bin/find /opt/data/logs/nginx/access -mtime +2 -name "access.log*" -exec rm -rf {} \; 
   
#clear nginx (access,error,log_mm,log_db) log(by day tar.gz) 10 days ago 
NGINX='access error log_mm log_db' 
for i in $NGINX 
do 
        /usr/bin/find /opt/data/logs/nginx/$i -mtime +10 -name "*tar.gz" -exec rm -rf {} \; 
done 
   
### apache ###  
#clear apache (access,error) log(by day tar.gz) 10 days ago 
APACHE='access error' 
for j in $APACHE 
do 
        /usr/bin/find /opt/data/logs/apache/$j -mtime +10 -name "*tar.gz" -exec rm -rf {} \; 
done 
   
### other log ### 
#clear (txt/mq,txt/auto,txt/man) log(by day .log) 10 days ago 
OTHER='txt/mq txt/auto txt/man' 
for k in $OTHER 
do 
        /usr/bin/find /opt/data/logs/$k -mtime +10 -name "*log" -exec rm -rf {} \; 
done
```

## **在分析nginx日志时常用命令总结**

### **1. 利用grep ,wc命令统计某个请求或字符串出现的次数**

```bash
比如统计GET /app/kevinContent接口在某天的调用次数，则可以使用如下命令：
[root@Fastdfs_storage_s1 ~]# cat /usr/local/nginx/logs/access.log | grep 'GET /app/kevinContent' | wc -l
 
其中cat用来读取日志内容，grep进行匹配的文本搜索，wc则进行最终的统计。
当然只用grep也能实现上述功能：
[root@Fastdfs_storage_s1 ~]# grep 'GET /app/kevinContent'  /usr/local/nginx/logs/access.log -c
```

### **2. 统计所有接口的调用次数并显示出现次数最多的前二十的URL**

```bash
[root@Fastdfs_storage_s1 ~]# cat /usr/local/nginx/logs/access.log|awk '{split($7,b,"?");COUNT[b[1]]++;}END{for(a in COUNT) print  COUNT[a], a}'|
sort -k1 -nr|head -n20
 
2722 /
10 /group1/M00/00/00/wKgKylqT3OCAUrqYAAAwK2jUNaY262.png
9 /group1/M00/00/00/wKgKylqUxBOAFo8hAAKHUIZ3K9s443.jpg
6 /group1/M00/00/00/wKgKylqUrceAGkPOAAAwK2jUNaY843.png
4 /group1/M00/00/00/wKgKylqTsFCAdeEuAAKHUIZ3K9s287.png
3 /group2/M00/00/00/wKgKy1qUtu2Acai1AAKHUIZ3K9s555.jpg
2 /favicon.ico
1 /group2/M00/00/00/wKgKy1qT3P-Ae-vQAAKHUIZ3K9s459.png
1 /group2/M00/00/00/wKgKy1qT3P-Ae-vQAAKHUIZ3K9s459.jpg
1 /group1/M00/00/00/wKgKylqUyMuAdkLwAAAwK2jUNaY176.png
1 /group1/M00/00/00/wKgKylqUtuyAA5xrAAKHUIZ3K9s226.jpg
1 /group1/M00/00/00/wKgKylqUscKAa4NXAAKHUIZ3K9s530.jpg
1 /group1/M00/00/00/wKgKylqTsFCAdeEuAAKHUIZ3K9s287.jpg
1 /group1/M00/00/00/wKgKylqT4ESAHdNjAAKHUIZ3K9s730.jpg
1 /group1/M00/00/00/wKgKylqT3-6AbEeUAAKHUIZ3K9s742.png
 
解释说明：
这里awk是按照空格把每一行日志拆分成若干项，其中$7对应的就是URL，当然具体对应的内容和使用nginx时设置的日志格式有关。
这样就可以通过拆分提取出IP，URL，状态码等信息。split是awk的内置函数，在此的意思是按照“？”将URL进行分割得到一个数组，并赋值给b。
COUNT[b[1]]++表示相同的接口数目加1。sort用来排序，-k1nr表示要把进行排序的第一列作为数字看待，并且结果倒序排列。
head -n20意为取排名前二十的结果。
```

### **3. 统计报错的接口** 

```bash
统计nginx日志中报错较多的接口，对于分析服务器的运行情况很有帮助，也可以有针对性的修复bug和性能优化。
[root@Fastdfs_storage_s1 ~]# cat /usr/local/nginx/logs/access.log|awk '{if($9==500) print $0}'|
awk '{split($7,b,"?");COUNT[b[1]]++;}END{for(a in COUNT) print  COUNT[a], a}'|sort -k 1 -nr|head -n10
 
先用awk’{if(9==500)print0}’过滤出500错误的日志，然后在此基础上做统计，其思路同2类似！
```

### **4. 统计HTTP响应状态码**

```bash
通过统计响应状态码可以看出服务器的响应情况，比如499较多时可以判断出服务器响应缓慢，再结合3可以找出响应慢的接口，
这样就能有针对性进行性能分析和优化。
 
[root@Fastdfs_storage_s1 ~]# cat /usr/local/nginx/logs/access.log |awk '{counts[$(9)]+=1}; END {for(code in counts) print code, counts[code]}'
| sort -k 2 -nr
 
200 2733
304 20
404 11
```

### **5. 统计服务器并发量**

```bash
[root@Fastdfs_storage_s1 ~]# cat /usr/local/nginx/logs/access.log |grep '10.15.19.138'| awk '{COUNT[$4]++}END{for( a in COUNT) print a,COUNT[a]}'
|sort -k 2 -nr|head -n20
 
nginx转发请求时可以记录响应请求的服务器IP,先通过grep过滤出某个服务器所有的请求，然后统计各个时间点的并发请求响应的数量即可得到某个服务器的并发量。
$4对应的是响应时间。当然，如果把grep的内容更换成某个接口也就可以统计出该接口对应的并发量了。
```

### **6. grep多条件与或操作**

```bash
有时候我们需要在nginx日志通过多个条件来查找某些特定请求，比如我需要找个某个用户浏览文章的请求，则可以需要同时匹配两个条件：
浏览文章接口GET /app/kevinContent和userId=59h7hrrn。
 
grep对应的与操作命令如下：
[root@Fastdfs_storage_s1 ~]# grep -E "GET /app/kevinContent.*userId=59h7hrrn" /usr/local/nginx/logs/access.log
 
grep与命令格式： grep -E “a.*b” file，ab条件同时成立
而grep或命令的格式为：grep -E “a|b” file ，ab两个条件有一个成立即可。
```

### **7. grep打印匹配的前后几行** 

```bash
有时候我们需要查找某个特定请求的前后几行的请求，以观察用户的关联操作情况。grep提供了一下几条命令：
# grep -C 5 'parttern' inputfile    //打印匹配行的前后5行。
# grep -A 5 'parttern' inputfile    //打印匹配行的后5行
# grep -B 5 'parttern' inputfile    //打印匹配行的前5行
 
grep -An  或grep -A n
grep -Bn  或grep -B n
grep -Cn  或grep -C n
 
如下，打印出access.log日志文件中匹配/app/kevinContent关键字符所在行的前后各10行
[root@Fastdfs_storage_s1 ~]# grep -C 10 'GET /app/kevinContent' /usr/local/nginx/logs/access.log
```

