# shell——系统安全检测（centos）

```bash
#!/bin/sh"
echo "#######################################「OS系统信息」##########################################"
OS_TYPE=`uname`
OS_Number=`dmidecode -t system |grep 'Serial Number'|awk '{print $3}'|awk -F, '{print $1}'`
OS_VERSION=`cat /etc/redhat-release`
OS_IPADDR=`ifconfig ens160|grep "inet" |awk '{print $2}' | sed -n '1p'`
OS_KERNER=`uname -a|awk '{print $3}'`
OS_NOWTIME=`date +%F_%T`
OS_RUN_TIME=`uptime |awk '{print $3,$4}'|awk -F, '{print $1}'`
OS_LASTREBOOT_TIME=`who -b|awk '{print $2,$3}'`
OS_HOSTNAME=`hostname`
echo " 主机类型: $OS_TYPE"
echo " 主机序列号: $OS_Number"
echo " 系统版本: $OS_VERSION"
echo " 系统IP地址: $OS_IPADDR"
echo " 内核版本: $OS_KERNER"
echo " 系统时间: $OS_NOWTIME"
echo " 运行时间: $OS_RUN_TIME"
echo " 最后重启时间: $OS_LASTREBOOT_TIME"
echo " 主机名称: $OS_HOSTNAME"
echo " SELinux：` /usr/sbin/sestatus | grep 'SELinux status:' | awk '{print $3}'`"
echo " 语言环境：`echo $LANG`"
echo "#######################################「OS资源信息」##########################################"
OS_CPU_PRO=`cat /proc/cpuinfo |grep "processor" | wc -l`
OS_CPU_COR=`cat /proc/cpuinfo| grep "cpu cores"| uniq |awk {'print $4'}`
OS_CPU_TYPE=`grep "model name" /proc/cpuinfo | awk -F ': ' '{print $2}' | sort | uniq`
echo " CPU总个数: $OS_CPU_PRO"
echo " CPU总核数: $OS_CPU_COR"
echo " CPU型 号： $OS_CPU_TYPE"
OS_SWAP_S=`free|grep Swap|awk {'print $2'}`
OS_PARTS=(`df -T|sed 1d|egrep -v "tmpfs|sr0"|awk {'print $3'}`)
OS_MEM_TAL=`free -m|grep Mem|awk '{print $2}'`
OS_MEM_FREE=`free -m|grep Mem|awk '{print $7}'`
echo " 内存总量: ${OS_MEM_TAL}MB"
echo " 内存余量: ${OS_MEM_FREE}MB"
OS_DISKS=0
OS_SWAP=`free|grep Swap|awk {'print $2'}`
OS_PARTS=(`df -T|sed 1d|egrep -v "tmpfs|sr0"|awk {'print $3'}`)
for ((i=0;i<`echo ${#OS_PARTS[*]}`;i++))
do
OS_DISKS=`expr $OS_DISKS + ${OS_PARTS[$i]}`
done
((OS_DISKS=\($OS_DISKS+$OS_SWAP\)/1024/1024))
echo " 磁盘总量: ${OS_DISKS}GB"
OS_DISKS=0
OS_SWAP=`free|grep Swap|awk '{print $4}'`
OS_PARTS=(`df -T|sed 1d|egrep -v "tmpfs|sr0"|awk '{print $5}'`)
for ((i=0;i<`echo ${#OS_PARTS[*]}`;i++))
do
OS_DISKS=`expr $OS_DISKS + ${OS_PARTS[$i]}`
done
((freetotal=\($OS_DISKS+$OS_SWAP\)/1024/1024))
echo " 磁盘余量： ${freetotal}GB"
echo "#######################################「OS网络监测」##########################################"

echo `ip a | grep eno | awk "NR==2" | awk '{print $NF,":",$2}'`
echo "网关：`ip route | awk 'NR==1'| awk '{print $3}'`"
echo "DNS: `cat /etc/resolv.conf | grep "nameserver" | awk '{print $2}'`"
ping -c 4 www.baidu.com > /dev/null
if [ $? -eq 0 ];then
echo "网络连接状态：正常"
else
echo "网络连接状态：失败"
fi
echo
echo "#######################################「OS安全检查」##########################################"

echo "用户登陆信息：`last | grep "still logged in" | awk '{print $1}'| sort | uniq`"
md5sum -c --quiet /etc/passwd > /dev/null 2&>1
if [ $? -eq 0 ];then
echo "文件未被篡改"
else
echo "文件被篡改"
fi
```

