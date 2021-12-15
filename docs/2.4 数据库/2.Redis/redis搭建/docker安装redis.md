# Docker安装redis

## 1、获取redis镜像

  执行命令：docker pull redis，不加版本号是获取最新版本，也可以加上版本号获取指定版本

 **![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405111615229-1727448413.png)**

 

## 2、查看本地镜像

![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405111816049-1493073957.png)

 

##  3、创建本地配置文件redis.conf，从[官网下载](http://download.redis.io/redis-stable/redis.conf)



```
在/usr/local目录下创建docker目录
mkdir /usr/local/docker
cd /usr/local/docker
再在docker目录下创建redis目录
mkdir redis&&cd redis
创建配置文件，并将官网redis.conf文件配置复制下来进行修改
touch redis.conf
创建数据存储目录data
mkidr data
```

![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405113759717-103929108.png)

修改启动默认配置(从上至下依次)：

bind 127.0.0.1 #注释掉这部分，这是限制redis只能本地访问

protected-mode no #默认yes，开启保护模式，限制为本地访问

daemonize no#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败

databases 16 #数据库个数（可选），我修改了这个只是查看是否生效。。

dir ./ #输入本地redis数据库存放文件夹（可选）

appendonly yes #redis持久化（可选）

requirepass 密码 #配置redis访问密码

## 4、创建并启动redis容器

docker run -p 6379:6379 --name redis -v /usr/local/docker/redis/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes

## 5、查看redis容器

执行命令：docker container ls -a

![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405123422159-1265425755.png)

 

 执行命令：docker ps查看运行的容器

![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405123517777-1783416610.png)

 

##  5、通过 redis-cli 连接测试使用 redis 服务

  执行命令：docker exec -it redis /bin/bash  进入docker终端，在终端中输入：redis-cli

![img](https://img2020.cnblogs.com/blog/395813/202004/395813-20200405123842578-889277769.png)

# Docker 部署 redis教程，附带部分小建议，防止踩坑

redis运行最主要的问题就是要把这个配置文件给挂载出来，那么我们在运行之前，就要提前 在 /data/redis/conf 目录下（这只是我的目录，你可以任意目录，记得替换掉启动参数里的路径）新建一个 redis.conf ， 这里同样给出一份配置（配置可以去参考redis的详细配置）



```
# 这里要设置成no，因为我们容器本身就已经是-d启动了，如果设置成yes，会无法启动起redis
daemonize no
# 这样设置可以让外界连接到redis，如果不想对公网暴露，可以设置成bind 127.0.0.1 ，但也有坑，坑见下文 
bind 0.0.0.0
# 写入文件，不开启，一重启数据就没了
appendonly yes
# 运行5个连接存活，防止出现长时间不连，需要重连的情况
tcp-keepalive 5
# 原则上必须填写，你要是对公网开放还没有密码，那就是裸奔
requirepass 你的密码
```

 运行后，此redis实例可以使用了，测试是否成功，执行以下命令，直接连接名称为redis-test的容器并执行 redis-cli命令



```
docker exec -it redis-test redis-cli
```

 成功，则看到连接到redis



```
127.0.0.1:6379>
```

 输入 auth 你的密码 获取权限, OK则没有问题



```
127.0.0.1:6379>auth 你的密码
OK
```

 同样的，可以测试从本地连接了。

## 五、启动一个最简单的redis实例，无密码



```
$ docker run -itd --name redis-test -p 6379:6379 redis
```

## 六、不踩坑姿势

1. 如果要对外网关闭，只对内网开放，你以为的：bind 127.0.0.1 就可以？

   - 常见错误：容器内设置bind 127.0.0.1 仅仅是对容器绑定，那会造成宿主机无法访问

   容器是不识别宿主机的local IP的，所以你想绑定bind 192.X.X.X 也同样不可行

   - 解决思路：
     1. 打通宿主机和容器的网络，可在启动的时候使用**--net=host**，直接让容器使用宿主机的IP和host
     2. 在iptables层（或者阿里云的安全组类似的）进行端口的控制，决定暴露给谁使
     3. 密码强度增加，端口更换成其他的，也可以解决不少安全性，这样开放就开放减低了被扫描的可能性

2. 数据没有保存出来，想直接抓aof数据

   1. 挂载出来即可，和conf同理