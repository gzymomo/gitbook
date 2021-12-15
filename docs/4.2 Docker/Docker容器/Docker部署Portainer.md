- [使用Portainer部署Docker容器实践](https://juejin.cn/post/6949464333071958029)



## 安装Portiner

安装Portiner的方式有很多种，但我一向喜欢使用最简单的方法来完成所需要做的事情，因此这里我将使用docker的方式来搭建它。

### 3.1 docker部署

docker部署的方式非常简单，只需要执行简单的运行容器命令即可，命令如下所示。

```
docker run -d \
-p 9001:9000 \
-p 8888:8000 \
--name test portainer/portainer-ce
```

命令中映射了物理机的8000端口和9000端口到容器中的8000端口以及9000端口，同时将宿主机的docker通信文件`/var/run/docker.sock`也映射到了容器中，另外为了持久化部署，还将目录` /opt/docker/portainer-ce/data`映射到了容器的`/data`目录下，命令执行完成之后，返回结果信息如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c446804d8a544266b5ba081fa5048840~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到已经成功运行了一个docker容器，接下来我需要验证服务是否正常运行，使用浏览器访问URL`http://127.0.0.1:9000/`地址，结果如下所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08f76b979fff490a927f90c40e6cf310~tplv-k3u1fbpfcp-zoom-1.image) 在上图中可以看到Portainer系统已经能够访问， 说明成功系统安装成功了。

### 3.2 节点初始化

现在我需要设置管理员的账号密码，这里我简单填写密码和确认密码之后，点击`Create user`按钮即可创建管理员账户。

管理员账户设置完成之后，需要进行初始化，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9322acfd78042e58941ddab082cfd9e~tplv-k3u1fbpfcp-zoom-1.image) 在上图中有三个选项，我选择使用Portainer管理本地docker程序，点击`Connect`按钮，即可完成初始化操作。

### 3.3 功能初探

完成初始化操作之后，就可以进入Portainer的工作界面，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc31770a289a4940af5bfaf19f7f982b~tplv-k3u1fbpfcp-zoom-1.image)

在上图找那个可以看到Portainer系统中已经有一个`local`的本地节点，我们可以点击它进入节点的管理，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cebb730a20514639b55054788e56659d~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到Portainer系统列出了`local`节点的 Stack、容器信息、镜像信息、磁盘信息、网络信息等等，这里我随意点击`Containers`区块，就可以看到容器列表，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5996a54a7a94505a2ac2d92524e2d53~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到容器列表中存在两个容器，以及容器的运行状态，也可以对这些容器进行控制。

## 四、管理节点

现在已经对本地docker可以进行控制，但是我并不满足于此，我需要对其他机器也进行控制。

### 4.1 开始添加节点

在Portainer系统中，有一个`endpoints`的菜单，在这个菜单当中可以添加多个节点，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8350a495cc4d445c81ac9936f70f7d7c~tplv-k3u1fbpfcp-zoom-1.image) 在上图中可以看到，已经有一个`local`的节点，在列表上方有一个`Add endpoint`按钮，点击按钮后就可以来到添加节点的详情页，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39ef241477d040b6ad79c3b2f814355a~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到有5个选项，这里我选择最简单的一种方式，使用`Docker API`进行控制。

### 4.2 开放API控制

这种方法需要在节点的docker启动程序中添加参数，因此我需要先登录到节点服务器中去，ssh登登录服务器的命令如下所示

```
ssh root@xxx.xxx.xxx.xxx
复制代码
```

命令执行完毕之后，返回如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a7ed4be234b451f812ae9f15816ba35~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到已经进入节点所在的服务器，接着需要编辑docker启动的配置文件，命令如下所示

```
vim /usr/lib/systemd/system/docker.service
复制代码
```

命令执行之后，就可以在vim编辑界面修改配置，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/878ef1feefc0498489f28764d0b32cf8~tplv-k3u1fbpfcp-zoom-1.image)

将开启远程访问代码加入到docker的启动命令行中，代码如下所示

```
-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
复制代码
```

将代码复制到  `/usr/bin/dockerd `程序后面，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3149ab794804f1883b0689fb1da2d26~tplv-k3u1fbpfcp-zoom-1.image)

保存配置文件之后，需要重启docker服务，重启docker的命令如下所示

```
systemctl daemon-reload  && systemctl restart docker
复制代码
```

重启docker之后，一切正常的话就完成了

### 4.3 验证端口状态

查看docker的配置信息，命令如下所示

```
docker info
复制代码
```

命令执行之后，返回的信息如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c32aa25aa6048138d12299615f205a2~tplv-k3u1fbpfcp-zoom-1.image) 在上图中可以看到docker给了一个warning的警告提示，告知我开启远程访问会存在安全风险，这里暂时不理会它，不过出现这个提示说明确实是开启了远程访问的功能

另外可以查看通过开放端口，来验证开启是否成功，命令如下所示

```
netstat -ntl
复制代码
```

命令执行完毕之后，会返回当前主机的端口开放情况，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/210392915adc424a80433c96b0c0640b~tplv-k3u1fbpfcp-zoom-1.image) 在上图中可以看到`2375`端口已经被开启成功， 说明节点本身开启docker是OK了；

但是Portainer通过ip访问此节点的时候，要考虑网络中的防火墙是否会屏蔽此端口，这里可以使用`nmap`工具来探测节点的端口是否可以被访问，现在我回到Portainer系统的命令终端，并使用nmap工具进行探测，命令如下所示

```
nmap -p 2375 xxx.xxx.xxx.xxx
复制代码
```

命令执行之后，会返回2375是否处于开启的情况，执行结果如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4a74c3d08964d3ba13db6520a63979a~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到节点的`2375`端口是开启的，并且可以进行连接。

### 4.4 完成添加节点

接下来回到浏览器窗口，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc8e7520b93c49c793fdc1ac4786de54~tplv-k3u1fbpfcp-zoom-1.image)

在上图所示的网页中，将节点的IP地址和端口通过URL形式填写进去，然后点击`Add endpodint`按钮，即可将节点增加进去，添加成功会有相应的提示，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4423f1f8fef4fc4b793aa118eec3077~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到Portainer系统提示添加节点已经成功，并且节点列表可以看到此节点了。

## 五、部署容器

添加节点完成之后，我准备在远程节点中部署我的容器；

### 5.1 部署单个容器

回到Portainer主页，在主页可以看到刚才添加的节点信息，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87d381c4095241dda880996d92acfd5a~tplv-k3u1fbpfcp-zoom-1.image)

在上图中选择刚才添加的节点，然后进入容器菜单选项，可以看到此节点的容器列表，，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48272ebdbfd64fb78dec1ec6abaf557c~tplv-k3u1fbpfcp-zoom-1.image)

在上图所示页面的列表上方有一个`Add container`按钮，点击此按钮后就会调整到添加容器详情页

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2374591770044249ae901b32aa668627~tplv-k3u1fbpfcp-zoom-1.image) 在上图所示的页面中，需要将docker镜像地址填写进去，这里我随意选举了一个nginx镜像，并且将主机的8888端口映射到了容器的80端口，提交这些信息之后，Portainer系统会告知你容器运行是否成功，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8743bded30924ad8adbf1023e4668e2c~tplv-k3u1fbpfcp-zoom-1.image) 在上图中可以看到容器已经运行成功，并且跳转到了容器列表中，接下来我们可以访问此节点对应的8888端口，来验证服务是否可用.

打开浏览器，然后在地址栏中填入URL`http://xxx.xxx.xxx.xxx:8888/`，访问之后返回的结果如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ea2aa499d2c40419d54f7e5f0dd08b7~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到`nginx`服务已经成功运行了；

### 5.2 部署 docker-compose

除了在容器列表页部署容器之外，Portainer系统还支持使用docker-compose的方式进行部署，在Portainer系统中叫做`stacks`,在菜单栏中选择此项，可以进入docker-compose服务的列表，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfb18c2784894fc39ec78574b0060621~tplv-k3u1fbpfcp-zoom-1.image)

在列表的上方有一个`Add stack`按钮，点击此按钮，就可以添加`docker-compose`服务，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3378be2274db40eabcb38105be1a95ce~tplv-k3u1fbpfcp-zoom-1.image)

在上图所示的页面中，会要求我填写docker-compose的信息，这里我准备了一个Redis服务的`docker-compose`的配置，配置代码如下所示

```
version: '3.5'
services:
  redis:
    image: "redis:latest"
    container_name: redis_test
    command: redis-server
    ports:
      - "16379:16379"
复制代码
```

降配置填到页面的后，进行提交Portainer就会在对应节点部署刚才的`docker-compose`服务，如下图所示 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ca4e562695345a6ba2890440ee7ed3e~tplv-k3u1fbpfcp-zoom-1.image) 部署成功之后，可以在stacks列表中看到刚才部署的服务，你还可以点击列表中的服务名称，进入详情页进行查看和修改，如下图所示

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0a6a05ad0484307a3bcb63322238bf4~tplv-k3u1fbpfcp-zoom-1.image)

在上图中可以看到此服务具体运行了什么容器，也可以终止或删除该容器。


作者：汤青松
链接：https://juejin.cn/post/6949464333071958029
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。