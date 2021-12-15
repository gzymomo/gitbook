Web信息收集-目标扫描-OpenVAS

# 一、OpenVAS简述
OpenVAS（Spen Vulnerability Assessment System），即开放式漏洞评估系统，是一个用于评估目标漏洞的杰出框架，开源且功能十分强大；
它与著名的Nessus“本是同根生”，在Nessus商业后之后仍然坚持开源。号称“当前最还用的开源漏洞扫描工具”。最新版的Kali Linux不再自带OpenVAS了，需要自己部署OpenVAS漏洞检测系统。其核心部件是一个服务器，包括一套网络漏洞测试程序，可以检测远程系统和应用程序中的安全问题。

它的最常用用途是检测目标网络或主机的安全性。它的评估能力来源于数万个漏洞测试程序，这些程序都是以插件的形式存在。openvas是基于C/S（客户端/服务器），B/S（浏览器/服务器）架构进行工作，用户通过浏览器或者专用客户端程序下达扫描任务，服务器端负责授权，执行扫描操作并提供扫描结果。

官方网站：
http://www.openvas.org/
http://www.greebone.net/

# 二、部署OpenVAS

## 2.1 升级Kali Linux
```bash
apt-get update
apt-get dist-upgrage
```

## 2.2 安装OpenVAS
```bash
apt-get install openvas
openvas-setup
```

## 2.3 修改admin账户密码
`openvasmd --user=admin --new-password=test`

## 2.4 修改默认监听IP
`vim /lib/systemd/system/greenbone-security-assistant.service`

## 2.5 启动OpenVAS
`openvas-start`

## 2.6 检查安装
```bash
ss -tnlp
openvas-check-setup
```

## 2.7 登录OpenVAS
`https://192.168.106.1:9392   # IP：9392   IP为Kali IP`