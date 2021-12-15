[TOC]

[Shell脚本实战：日志关键字监控+自动告警](https://segmentfault.com/a/1190000022014331)

用于Linux服务器监控程序日志，如出现关键字异常则触发相应的动作或告警操作，通知到邮件联系人。

# 一、安装邮件服务
## 1.1 解压
`tar -jxf mailx-12.4.tar.bz2 -C /usr/src/ && cd /usr/src/mailx-12.4/`

## 1.2 编译
`make install UCBINSTALL=/usr/bin/install`

## 1.3 配置
```bash
vi /etc/nail.rc，在文件的最后添加
set from=
set smtp=
set smtp-auth-user=
set smtp-auth-password=
set smtp-auth=login
```
![](https://segmentfault.com/img/remote/1460000022014339)

## 1.4 测试发送邮件
echo '邮件正文'|mailx -v -s "邮件标题" 邮箱地址
![](https://segmentfault.com/img/remote/1460000022014340)

至此邮件功能测试正常，接下来是关键字监控脚本的编写工作，通过日志关键字脚本触发邮件告警脚本实现日志监控。

# 二、关键字监控脚本开发
CheckLogs.sh 日志监控程序，SendMail.sh 发邮件脚本,该脚本可自定义，conf 配置文件目录,chklist为配置文件。

![](https://segmentfault.com/img/remote/1460000022014341)

初次执行CheckLogs.sh脚本会读取日志文件并记录当前读取的行数
![](https://segmentfault.com/img/remote/1460000022014341)

后续运行脚本则读取更新日志，例如当前日志更新了9行数据，则脚本从最新的9行数据获取关键字
![](https://segmentfault.com/img/bVbEw6T)

测试插入关键字error：
![](https://segmentfault.com/img/bVbEw6U)

继续执行脚本可以看到已捕获关键字并触发告警
![](https://segmentfault.com/img/remote/1460000022014342)

测试多关键字
![](https://segmentfault.com/img/remote/1460000022014343)

再次执行可以看到已捕获最新更新的日志信息
![](https://segmentfault.com/img/remote/1460000022014345)

测试频率控制
![](https://segmentfault.com/img/remote/1460000022014344)

# 三、完整脚本
```bash
#!/bin/bash
basedir=$(cd `dirname "$0"`;pwd)
logsdir=$basedir/logs
tmpsdir=$basedir/tmps
confdir=$basedir/conf
chkfile=$confdir/chklist
logfile=$logsdir/log.log_$(date +%F)

#创建各种目录
mkdir -p $logsdir $tmpsdir $confdir

#创建配置文件
if test ! -e "$chkfile";then
    echo "#日志文件,关键字(多关键字|隔开),重试次数,最大执行次数,启动命令,停止命令" >$chkfile
fi

#生成日志函数
do_writelog() {
    case $1 in
    i|I)
        shift
        echo "$(date +%Y-%m-%d) $(date +%H:%M:%S)|INFO|$@" >>$logfile
        ;;
    e|E)
        shift
        echo "$(date +%Y-%m-%d) $(date +%H:%M:%S)|ERROR|$@" >>$logfile
        ;;
    w|W)
        shift
        echo "$(date +%Y-%m-%d) $(date +%H:%M:%S)|WARNING|$@" >>$logfile
        ;;
    *)
        echo "$(date +%Y-%m-%d) $(date +%H:%M:%S)|DEBUG|$@" >>$logfile
        esac
}

#日志处理部分代码
cat $chkfile|egrep -v "^($|#)"|while read i;do
(
    app_name=$(echo "$i"|awk -F, '{print $1}')
    if test -z "$app_name";then
        do_writelog e "应用名称为空,退出执行"
        exit 0
    fi
    log_filename=$(echo "$i"|awk -F, '{print $2}')
    if test ! -e "$log_filename";then
        do_writelog e "日志文件($log_filename)不存在,退出执行"
        exit 0
    fi
    log_md5sum=$(echo -n "$log_filename"|md5sum|awk '{print $1}')
    log_gjz=$(echo "$i"|awk -F, '{print $3}')
    if test -z "$log_gjz";then
        do_writelog i "日志文件($log_filename),关键字为空,退出执行"
        exit 0
    fi
    log_retry=$(echo "$i"|awk -F, '{print $4}')
    expr $log_retry + 0 &>/dev/null
    if [ $? -ne 0 ];then
        log_retry=0
    fi
    log_max=$(echo "$i"|awk -F, '{print $5}')
    expr $log_max + 0 &>/dev/null
    if [ $? -ne 0 ];then
        log_max=3
    fi
    start_command=$(echo "$i"|awk -F, '{print $6}')
    stops_command=$(echo "$i"|awk -F, '{print $7}')
    open_sendmail=$(echo "$i"|awk -F, '{print $8}')
    mail_scripts=$(echo "$i"|awk -F, '{print $9}')
    mail_contacts=$(echo "$i"|awk -F, '{print $10}')
    if [ $open_sendmail -eq 1 ];then
        if test -z "$mail_scripts";then
            do_writelog i "应用($app_name),触发动作脚本为空,退出执行"
            exit 0
        fi
        if test -z "$mail_contacts";then
            do_writelog i "应用($app_name),联系人为空,退出执行"
            exit 0
        fi
    fi
    if test ! -e "$tmpsdir/$log_md5sum";then
        log_new_count=$(wc -l $log_filename|awk '{print $1}')
        echo "$log_new_count" >$tmpsdir/$log_md5sum
        do_writelog i "日志文件($log_filename),初始化读取日志行数:$log_new_count,退出执行"
    else
        log_old_count=$(cat $tmpsdir/$log_md5sum)
        expr $log_old_count + 0 &>/dev/null
        if [ $? -ne 0 ];then
            do_writelog e "日志文件($log_filename),读取历史行数失败,退出执行"
            exit 0
        fi
        log_new_count=$(wc -l $log_filename|awk '{print $1}')
        if [ $log_new_count -eq $log_old_count ];then
            do_writelog i "日志文件($log_filename),未更新,退出执行"
            exit 0
        elif [ $log_new_count -lt $log_old_count ];then
            do_writelog i "日志文件($log_filename),跨日更新日志行数:$log_new_count,退出执行"
            echo "$log_new_count" >$tmpsdir/$log_md5sum
        else
           log_upd_count=$(expr $log_new_count - $log_old_count)
           do_writelog i "日志文件($log_filename),历史行数:$log_old_count,最新行数:$log_new_count,更新行数:$log_upd_count" 
           #读取更新的日志
           log_content=$(tail -n +`expr $log_old_count + 1` $log_filename|head -n +$log_upd_count)
           oldIFS=$IFS
           IFS="|"
           count=0
           for i in $log_gjz;do
               if [ $(echo "$log_content"|grep -c -w "$i") -ge 1 ];then
                   let count+=1  
               fi
               if [ $count -gt 0 ];then
                   break
               fi
           done
           IFS=$oldIFS
           if [ $count -gt 0 ];then
               echo "0" >>$tmpsdir/${log_md5sum}.retry
           else
              do_writelog i "日志文件($log_filename),未获取到关键字,退出执行"
              echo "$log_new_count" >$tmpsdir/$log_md5sum
              exit 0
           fi
           if [[ $(wc -l $tmpsdir/${log_md5sum}.retry|awk '{print $1}') -gt $log_retry ]];then
               echo "0" >>$tmpsdir/${log_md5sum}_$(date +%F)
               if [ $(wc -l $tmpsdir/${log_md5sum}_$(date +%F)|awk '{print $1}') -le $log_max ];then
                   $stops_command 1>>$logfile 2>&1
                   $start_command 1>>$logfile 2>&1
                   do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,停止命令:$stops_command,启动命令:$start_command,重启服务"
                   if [ $open_sendmail -eq 1 ];then
                       if test -n "$mail_scripts";then
                           $mail_scripts "应用[$app_name]故障" "日志文件($log_filename),日志内容:$log_content,找到关键字:$i" "$mail_contacts" 1>>$logfile 2>&1
                           if [ $? -ne 0 ];then
                               do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,触发告警失败"
                           fi
                           do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,触发告警通知联系人:[$mail_contacts]"
                       fi
                   else
                       do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,不触发告警"
                   fi
               else
                   do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,重启服务超出当天限制次数:$log_max,退出执行"
               fi
               rm -f $tmpsdir/${log_md5sum}.retry &>/dev/null
           else
               do_writelog i "日志文件($log_filename),日志内容:$log_content,找到关键字:$i,重试检测:$(wc -l $tmpsdir/${log_md5sum}.retry|awk '{print $1}')"
           fi
           echo "$log_new_count" >$tmpsdir/$log_md5sum
           do_writelog i "日志文件($log_filename),更新记次文件完成" 
        fi
    fi
)&
done
```