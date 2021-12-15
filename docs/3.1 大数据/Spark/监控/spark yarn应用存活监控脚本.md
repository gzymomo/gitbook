[TOC]

# 1、Spark
```bash
#!/bin/bash

list="com.xuhai.HcGPSLauncher com.xuhai.ZyStreaming com.xuhai.DzStreaming";
apps=$(/usr/etc/hadoop-3.0.0/bin/yarn application -list | grep HcGPSLauncher | awk '{if($2!="") print $2}')

for app in $list:
do
  if [ "$app" = "$apps" ];then
    echo "good"
	curl 'https://oapi.dingtalk.com/robot/send?access_token=9f33082b9db98c1cf215fe1348ad8db091777701aa435cde47d0af4a87e0460e' \
	   -H 'Content-Type: application/json' \
	   -d '{"msgtype": "text", 
			"text": {
				 "content": "alert告警：HcGPSLauncher服务已停止"
			}
		  }'
  else
    echo "bad"
  fi
done
```

# 2、spark
```bash
#!/bin/bash
apps=$(/usr/etc/hadoop-3.0.0/bin/yarn application -list | grep HcGPSLauncher | awk '{if($2!="") print $2}')
if [ -n "$apps" ];then
  echo "good"
else
  curl 'https://oapi.dingtalk.com/robot/send?access_token=9f33082b9db98c1cf215fe1348ad8db091777701aa435cde47d0af4a87e0460e' \
	   -H 'Content-Type: application/json' \
	   -d '{"msgtype": "text", 
			"text": {
				 "content": "alert告警：HcGPSLauncher服务已停止!请尽快检查程序！"
			}
		  }'
fi
```

# 3、Spark 应用监控告警和自动重启
进程监控失败重启和告警模块
监控yarn上指定的Spark应用是否存在，不存在则发出告警。

使用Python脚本查看yarn状态，指定监控应用，应用中断则通过webhook发送报警信息到钉钉群，并且自动重启。
```python
#!/usr/bin/python3.5
# -*- coding: utf-8 -*-
import os
import json
import requests

'''
Yarn应用监控：当配置的应用名不在yarn applicaition -list时，钉钉告警
'''


def yarn_list(applicatin_list):
    yarn_application_list = os.popen('yarn application -list').read()
    result = ""
    for appName in applicatin_list:
        if appName in yarn_application_list:
            print("应用:%s 正常!" % appName)
        else:
            result += ("告警--应用:%s 中断!" % appName)
            if "应用名1" == appName:
                os.system('重启命令')

    return result


def dingding_robot(data):
    # 机器人的webhooK 获取地址参考：https://open-doc.dingtalk.com/microapp/serverapi2/qf2nxq
    webhook = "https://oapi.dingtalk.com/robot/send?access_token" \
              "=你的token "
    headers = {'content-type': 'application/json'}  # 请求头
    r = requests.post(webhook, headers=headers, data=json.dumps(data))
    r.encoding = 'utf-8'
    return r.text


if __name__ == '__main__':
    applicatin_list = ["应用名1", "应用名2", "应用名3"]
    output = yarn_list(applicatin_list)
    print(output)

    if len(output) > 0:
        # 请求参数 可以写入配置文件中
        data = {
            "msgtype": "text",
            "text": {
                "content": output
            },
            "at": {
                "atMobiles": [
                    "xxxxxxx"
                ],
                "isAtAll": False
            }
        }
        res = dingding_robot(data)
        print(res)  # 打印请求结果
    else:
        print("一切正常!")

```