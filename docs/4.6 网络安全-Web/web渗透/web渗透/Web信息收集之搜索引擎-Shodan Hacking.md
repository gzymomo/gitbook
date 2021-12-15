Web信息收集之搜索引擎-Shodan Hacking

# 一、Shodan Hacking简介
https://www.shodan.io
Shodan（撒旦搜索引擎）是由Web工程师John Matherly（马瑟利）编写的，被称为“最可怕的搜索引擎”，可扫描一切联网的设备。除了常见的Web服务器，还能扫描防火墙、路由器、交换机、摄像头、打印机等一切联网设备。

## 1.1 ip
```
114.114.114.114
```
## 1.2 Service/protocol
```
http
http country:”DE”
http country:”DE” product:”Apache httpd”
http product:”Apache httpd”

ssh
ssh default password
ssh default password country:”JP”
```

## 1.3 Keyword
基于关键词搜索的思路是根据banner信息（设备指纹）来搜索
```
“default password” country:”TH”
FTP anon successful
```
## 1.4 Cuuntry
```
Cuuntry:cn
Cuuntry:us
Cuuntry:jp
```
## 1.5 Product
```
Product:”Microsoft IIS httpd”
Product:”nginx”
Product:”Apache httpd”
Product:MySQL
```
## 1.6 Version
```
Product:Mysql version:”5.1.73”
Product:”Microsoft IIS httpd” version:”7.5”
```
## 1.7 Hostname
```
Hostname:.org
Hostname:.edu
```
## 1.8 os
```
Os:”Windows Server 2008 R2”
Os:”Windows 7 or 8”
Os:”Linux 2.6.x”
```
## 1.9 net
`Net:110.180.13.0/24`

## 1.10 port
```
port:80
port:22
```
## 1.11 综合示例
搜索日本区开启80端口的设备:
```
country:jp port:"80"
country:jp port :"80" product: "Apache httpd"
country:jp port:"80" product: "Apache httpd" city: "Tokyo"
country:jp port:"80" product: "Apache httpd" city:"Tokyo" os:"Linux 3.x"
```

搜索日本区使用Linux2.6. x系统的设备:
```
country:jp os:"Linux 2.6.x"
country:jp os:"Linux 2.6.x" port:"80"
country:jp os:"Linux 2.6.x" port:"80" product:"Apache httpd"
```

搜索日本区使用Windows Server 系统的设备:
```
country:jp os: "Windows Server 2008 R2"
country:jp os : "Windows Server 2003" port: "445"
country:jp os : "Windows Server 2003" port: "80"
```

搜索日本区使用Microsoft IIS的设备:
`country:jp product: "Microsoft iis httpd" version:"7.5"`