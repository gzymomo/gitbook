# [docker 安装jumpserver](https://www.cnblogs.com/smlile-you-me/p/13199996.html)

#docker 安装

```bash
mkdir /etc/docker
echo "{
  \"registry-mirrors\" : [
  \"https://registry.docker-cn.com\",
  \"https://docker.mirrors.ustc.edu.cn\",
  \"http://hub-mirror.c.163.com\",
  \"https://cr.console.aliyun.com/\"
 ]
}">>/etc/docker/daemon.json

yum -y install yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum -y install docker-ce
systemctl start docker && systemctl enable docker
```



\#生成秘钥

```bash
if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi

if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi
```



注：生成完 SECRET_KEY 和 BOOTSTRAP_TOKEN 变量后一定要确认一下，如果出现异常将会影响到后面的过程

![img](https://img2020.cnblogs.com/blog/1139005/202007/1139005-20200704233518182-1088543071.png)

 

创建jms容器中的日志及数据挂到宿机的目录

```bash
mkdir -p /jumpserver/jumpserver/data
mkdir -p /jumpserver/koko/data
mkdir -p /jumpserver/nginx/logs
mkdir -p /jumpserver/mysql/{data,logs}
```





============== **不管用以下哪种方法都要执行以上操作** 且 **创建完容器后一定要等几分钟再访问** =====================

**方法1：**

**#pull镜像和创建容器(所有数据都在容器中)**

```bash
docker run --name jms_all -d \
-p 80:80 -p 2222:2222 \
-e SECRET_KEY=$SECRET_KEY \
-e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
jumpserver/jms_all:latest
```



网页登录容器所在IP，账密：admin/admin

![img](https://img2020.cnblogs.com/blog/1139005/202006/1139005-20200627205542258-1450129765.png)

**方法2：（个人推荐使用该方法）**

1、容器中的jumpserver的数据在/opt/jumpserver/data目录中，日志在/opt/jumpserver/logs目录中，初始化数据库在/opt/jumpserver/utils目录中，配置文件在/opt/jumpserver/config.yml文件中，启动jumpserver命令为/opt/jumpserver/jms { start | restart | stop }

2、koko插件的配置文件在/opt/koko/config.yml文件中，数据在/opt/koko/data目录中

```bash
docker run --name jms_all -d \
-p 80:80 -p 2222:2222 \
-e SECRET_KEY=$SECRET_KEY \
-e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
-v /jumpserver/jumpserver/data:/opt/jumpserver/data \
-v /jumpserver/jumpserver/logs:/opt/jumpserver/logs \
-v /jumpserver/koko/data:/jumpserver/koko/data \
-v /jumpserver/nginx/logs:/var/log/nginx/ \
jumpserver/jms_all:latest
```



\#测试（其他机器连接172.16.186.131，连接用户是admin，密码是admin）

![img](https://img2020.cnblogs.com/blog/1139005/202006/1139005-20200627213945814-2112658191.png)

 

**方法3：（该方法顺序不能变）**

```bash
\#docker pull mysql
docker run --restart=always \
--name mysql5.7 -id \
-e MYSQL_DATABASE="jumpserver" \
-e MYSQL_USER="jumpserver" \
-e MYSQL_PASSWORD="AA7788aa" \
-e MYSQL_ROOT_PASSWORD="AA7788aa" \
-v /jumpserver/mysql/data:/var/lib/mysql \
-v /jumpserver/mysql/logs:/var/log/mysql/ \
-p 3306:3306 -d mysql:5.7.20
```



```bash
#docker pull redis
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo "vm.overcommit_memory=1">>/etc/sysctl.conf
echo "net.core.somaxconn= 1024">>/etc/sysctl.conf
echo "'echo never > /sys/kernel/mm/transparent_hugepage/enabled'">>/etc/rc.local
sysctl -p
docker run -p 6379:6379 --name redis -v /jumpserver/redis/data:/data -d redis redis-server --requirepass "A12345a" --appendonly yes
```



redis容器中登录方式

![img](https://img2020.cnblogs.com/blog/1139005/202006/1139005-20200628103016019-1367516191.png)

**#docker pull jms**

 ```bash
docker run --restart=always \
--name jms_all -d \
-p 80:80 -p 2222:2222 \
-e SECRET_KEY=$SECRET_KEY \
-e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
-v /jumpserver/jumpserver/data:/opt/jumpserver/data \
-v /jumpserver/jumpserver/logs:/opt/jumpserver/logs \
-v /jumpserver/koko/data:/jumpserver/koko/data \
-v /jumpserver/nginx/logs:/var/log/nginx/ \
-e DB_HOST="mysql5.7" \
-e DB_PORT=3306 \
-e DB_USER=root \
-e DB_PASSWORD=AA7788aa \
-e DB_NAME=jumpserver \
--link mysql5.7:mysql \
-e REDIS_HOST=redis \
-e REDIS_PORT=6379 \
-e REDIS_PASSWORD=A12345a \
--link redis:redis \
jumpserver/jms_all:latest
 ```


![img](https://img2020.cnblogs.com/blog/1139005/202006/1139005-20200628091725894-275852752.png)

　　　　　　　　　　　　　　　　　　　　　　**注：创建完容器后要稍等一下在去页面访问。**

 

**方法4：**（该方法适合在无互联网环境中使用且只需redis和jms_all即可）

```bash
if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi

if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi

mkdir -p /jumpserver/jumpserver/data
mkdir -p /jumpserver/koko/data
mkdir -p /jumpserver/nginx/logs
mkdir -p /jumpserver/mysql/{data,logs}

echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo "vm.overcommit_memory=1">>/etc/sysctl.conf
echo "net.core.somaxconn= 1024">>/etc/sysctl.conf
echo "'echo never > /sys/kernel/mm/transparent_hugepage/enabled'">>/etc/rc.local
sysctl -p
```



\#需在互联网中将redis和jms_all的镜像pull下来，再打成tar包发送至内网环境中，并导入进去
将镜像打包

```bash
docker save -o <new_name>.tar <ImageID>
```




导入镜像

```bash
docker load -i <new_name>.tar
```




将镜像更换名

```bash
docker tag <images_name>:<tag> <image_name>:<tag>

docker run -p 6379:6379 --name redis -v /jumpserver/redis/data:/data -d redis redis-server --requirepass "A12345a" --appendonly yes
```



```bash
docker run --restart=always \
--name jms_all -d \
-p 80:80 -p 2222:2222 \
-e SECRET_KEY=$SECRET_KEY \
-e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN \
-v /jumpserver/jumpserver/data:/opt/jumpserver/data \
-v /jumpserver/jumpserver/logs:/opt/jumpserver/logs \
-v /jumpserver/koko/data:/jumpserver/koko/data \
-v /jumpserver/nginx/logs:/var/log/nginx/ \
-v /jumpserver/mysql/data:/var/lib/mysql/ \
-v /jumpserver/mysql/logs:/var/log/mariadb/ \
-e REDIS_HOST=redis \
-e REDIS_PORT=6379 \
-e REDIS_PASSWORD=A12345a \
--link redis:redis \
jms_all:latest
```



注：如在创建jms容器时报错，需先将-v /jumpserver/mysql/logs:/var/log/mariadb/ \这条去掉，再创建，无问题后再删掉容器后再加上-v /jumpserver/mysql/logs:/var/log/mariadb/ \选项来创建jms容器

 

![img](https://img2020.cnblogs.com/blog/1139005/202006/1139005-20200628091842525-1354639315.png)

docker容器设置开机自启动：
--restart具体参数值详细信息
no - 容器退出时，不重启容器
on-failure - 只有在非0状态退出时才从新启动容器
always - 无论退出状态是如何，都重启容器
使用 on-failure 策略时指定 Docker 将尝试重新启动容器的最大次数；默认情况下Docker将尝试永远重新启动容器；

```bash
docker run --restart=on-failure:10 redis
```


如果创建容器时未指定 --restart=always ,可通过 update 命令更改；  

```bash
docker update --restart=always 容器ID
```

**如未使用--restart=always选项，在服务器或其他情况导致服务器关机/重启，再次启动容器时需先起MySQL、redis，最后起jms**