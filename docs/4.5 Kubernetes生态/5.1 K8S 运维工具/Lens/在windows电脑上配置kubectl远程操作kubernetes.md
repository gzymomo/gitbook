- [在windows电脑上配置kubectl远程操作kubernetes](https://blog.csdn.net/boling_cavalry/article/details/90577769)



Kubernetes集群经常部署在Linux环境，而本机环境经常是Windows，除了ssh登录到kubernetes所在机器进行操作，也可以在本机配置kubectl，来远程操作服务器上的kubernetes。

# 环境信息

- kubernetes：1.14.0
- kubectl：1.7.0
- kubernetes所在Linux服务器：CentOS7.4
- 本地环境：win10专业版64位



# 操作步骤

1. 下载windows版的kubectl可执行文件，地址是：https://storage.googleapis.com/kubernetes-release/release/v1.7.0/bin/windows/amd64/kubectl.exe
2. 进入在当前windows用户的home目录，我用的账号是Administrator，所以进入目录C:\Users\Administrator，在里面创建文件夹.kube，（建议用命令行创建，因为名字中带点，在桌面上输入名字不会成功）创建之后如下图：

![](https://img-blog.csdnimg.cn/20190526171500975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aW5jaGVuLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

3. 登录到可以执行kubectl的Linux服务器，去目录~/.kube/,将里面的config文件下载下来，放到上一步创建的.kube目录下；
4. 回到windows电脑，打开控制台，进入kubectl.exe文件所在目录，即可通过kubectl对kubernetes环境进行操作，如下图：

![](https://img-blog.csdnimg.cn/20190526172326818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly94aW5jaGVuLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

至此，windows环境下已经可以远程操作kubernetes环境了；




