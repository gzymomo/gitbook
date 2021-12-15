官网：https://pypi.org/project/webssh/

在linux机器上安装python环境，并且使用命令pip3 install webssh,装上这个模块

我们就可以在浏览器web页面登录我们的linux机器

## 功能

- 支持SSH密码验证，包括空密码。
- 支持SSH公钥认证，包括DSA RSA ECDSA Ed25519密钥。
- 支持加密密钥。
- 支持两要素身份验证（基于时间的一次性密码）
- 支持全屏终端。
- 终端窗口可调整大小。
- 自动检测ssh服务器的默认编码。
- 现代浏览器支持Chrome，Firefox，Safari，Edge，Opera。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpGibgkzvpuDibiawp3MAUSWw2nay0icADAceaEib88Sqkv3g8mPFUjAZm4YVSaIrzep1iaTCYlXYKpGGVA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 安装

```
pip3 install webssh
```

## 运行服务

```
# 直接运行wssh，使用默认8888端口
wssh

# 通过绑定IP地址和端口启动
wssh --address='192.168.83.129' --port=8888
wssh --address='0.0.0.0' --port=8888

# 通过绑定IP地址和端口启动，只允许本地地址访问
wssh --address='127.0.0.1' --port=8888
```

## 启动服务效果

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpGibgkzvpuDibiawp3MAUSWw2YJ4cBJ7xsQj9qTZTXYNqn5xLSkxZfytTuEXQymFYSMalEnylgpGKpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 使用

打开浏览器，输入 http://192.168.83.129:8888

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpGibgkzvpuDibiawp3MAUSWw2zs0QdZ3oBTTPVCKxN5qqEQpRgDz341UqPPdpic9e3cDEnqwl8Bjexaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击Connect

![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpGibgkzvpuDibiawp3MAUSWw2GghIB2giceMbBk81JVckbKcvicia2R4ZH4At2HSwj6JiaBRpZOcu4oBgMA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**注意**

服务启动后，可以通过 http://192.168.83.129:8888/ 或 http://localhost:8888 来访问。

页面会要求输入要登录的机器名，端口，用户和密码，然后就可以SSH到指定机器了。

若要使用root用户登录Webssh,必须修改配置文件 vim /etc/ssh/sshd_config

注释掉 #PermitRootLogin without-password 添加PermitRootLogin yes

```
# Authentication:
LoginGraceTime 120

#PermitRootLogin prohibit-password
PermitRootLogin yes
StrictModes yes
```

然后重启服务即可。