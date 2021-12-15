- [生产环境下实现每天自动备份mysql数据库](https://blog.51cto.com/ganbing/2053583)



shell脚本+定时任务"的方式来实现自动备份mysql数据库。

# 环境

备份路径：/data/mysqlbak/

备份脚本：/data/mysqlbak/mysqlbak.sh

备份时间：每天23：59备份

备份要求：比如备份的数据只保留1周

# mysqlbak.sh脚本

```bash
#!/bin/bash
#数据库IP
dbserver='127.0.0.1'
#数据库用户名
dbuser='root'
#数据密码
dbpasswd='********'
#数据库,如有多个库用空格分开
dbname='back01'
#备份时间
backtime=`date +%Y%m%d`
#备份输出日志路径
logpath='/data/mysqlbak/'


echo "################## ${backtime} #############################" 
echo "开始备份" 
#日志记录头部
echo "" >> ${logpath}/mysqlback.log
echo "-------------------------------------------------" >> ${logpath}/mysqlback.log
echo "备份时间为${backtime},备份数据库表 ${dbname} 开始" >> ${logpath}/mysqlback.log
#正式备份数据库
for table in $dbname; do
source=`mysqldump -h ${dbserver} -u ${dbuser} -p${dbpasswd} ${table} > ${logpath}/${backtime}.sql` 2>> ${logpath}/mysqlback.log;
#备份成功以下操作
if [ "$?" == 0 ];then
cd $datapath
#为节约硬盘空间，将数据库压缩
tar zcf ${table}${backtime}.tar.gz ${backtime}.sql > /dev/null
#删除原始文件，只留压缩后文件
rm -f ${datapath}/${backtime}.sql
#删除七天前备份，也就是只保存7天内的备份
find $datapath -name "*.tar.gz" -type f -mtime +7 -exec rm -rf {} \; > /dev/null 2>&1
echo "数据库表 ${dbname} 备份成功!!" >> ${logpath}/mysqlback.log
else
#备份失败则进行以下操作
echo "数据库表 ${dbname} 备份失败!!" >> ${logpath}/mysqlback.log
fi
done
echo "完成备份"
echo "################## ${backtime} #############################"
```

为脚本加上执行权限：

```bash
#chmod +x /data/mysqlbak/mysqlbak.sh
```

**配置定时任务执行脚本**

```bash
#crontab -e

59 23 * * * /data/mysqlbak/mysqlbak.sh
```

**参数说明：**

  格式为       ：分  时  日  月  周  命令

  59 23 * * *  :每天23：59分自动执行脚本  

  M: 分钟（0-59）。每分钟用*或者 */1表示

  H：小时（0-23）。（0表示0点）

  D：天（1-31）。

  m: 月（1-12）。

  d: 一星期内的天（0~6，0为星期天）。



  提示：最好你先执行一下脚本能不能跑通，然后在写到crontab中，等执行完了，进入/data/mysqlbak/目录查看一下有没有备份文件，如果有，则表示脚本执行成功，记得不要搞错了备份的用户和密码。