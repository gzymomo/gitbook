- [Grafana邮件报警](https://www.cnblogs.com/xiao987334176/p/11944661.html)



# 一、概述

报警是Grafana的一项革命性功能，它让Grafana从一个数据可视化工具变成一个真正的任务监控工具。报警规则可以使用现有的图表控制面板设置，阈值可以通过拖拉右边的线控制，非常简单。Grafana服务器会不断评估设置的规则，在规则条件符合的时候发送出通知。



# 二、配置

Grafana版本必须是4.0+才支持报警功能。

首先编辑配置文件

```
cd /etc/grafana/
cp grafana.ini grafana.ini.bak
vi grafana.ini
```

 

这里以 阿里云邮箱为列：

smtp内容如下：

```yaml
#################################### SMTP / Emailing ##########################
[smtp]
enabled = true
host = smtp.mxhichina.com:465
user = monitor@xx.com
password = 123456
from_address = monitor@xx.com
from_name = Grafana
skip_verify = true
ehlo_identity = xx.com
```

 

配置完成以后重启服务使其生效

```
service grafana-server restart
```

 

后台配置

![img](https://img2018.cnblogs.com/i1/1341090/201911/1341090-20191127185718644-2063442253.png)

 

 

添加邮件报警

![img](https://img2018.cnblogs.com/i1/1341090/201911/1341090-20191127185914769-1358539444.png)

 

 

# 三、测试

点击测试

![img](https://img2018.cnblogs.com/i1/1341090/201911/1341090-20191127190114974-808584631.png)

 

出现以下提示，表示成功！

![img](https://img2018.cnblogs.com/i1/1341090/201911/1341090-20191127190158776-510166657.png)

 

查看邮件

![img](https://img2018.cnblogs.com/i1/1341090/201911/1341090-20191127190409745-1265283618.png)

 

本文参考链接：

https://cloud.tencent.com/developer/article/1096717