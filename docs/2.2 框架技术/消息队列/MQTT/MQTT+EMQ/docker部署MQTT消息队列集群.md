[docker部署MQTT消息队列集群](https://www.cnblogs.com/lfl17718347843/p/13404325.html)



环境：三台节点
192.168.200.100 master1
192.168.200.110 master2
192.168.200.120 master3

# 1.每台节点下载docker-ce源

```bash
wget https://download.docker.com/linux/centos/docker-ce.repo
```

# 2.每台节点安装docker-ce

```bash
yum -y install docker-ce
```

# 3.启动并加入开机自启

```bash
systemctl start docker && systemctl enable docker
```

# 4.master1节点安装MQTT服务

```bash
docker run -it --network host --name emqtt-master1-1
-p 1883:1883
-p 18083:18083
-p 8083:8083
-p 8883:8883
-p 8080:8080
-e EMQX_NAME="master1"
-e EMQX_HOST=192.168.200.100
-e EMQX_LISTENER__TCP_EXTERNAL=1883
-e EMQX_WAIT_TIME=30
-e EMQX_CLUSTER__DISCOVERY="static"
-e EMQX_JOIN_CLUSTER="master1@192.168.200.100"
-e EMQX_CLUSTER__STATIC__SEEDS="master1@192.168.200.100,master2@192.168.200.110,master3@192.168.200.120"
emqx/emqx:v3.2.2
```

# 5.master2节点安装MQTT服务

```bash
docker run -it --network host --name emqtt-master2-1
-p 1883:1883
-p 18083:18083
-p 8083:8083
-p 8883:8883
-p 8080:8080
-e EMQX_NAME="master2"
-e EMQX_HOST=192.168.200.110
-e EMQX_LISTENER__TCP_EXTERNAL=1883
-e EMQX_WAIT_TIME=30
-e EMQX_CLUSTER__DISCOVERY="static"
-e EMQX_JOIN_CLUSTER="master2@192.168.200.110"
-e EMQX_CLUSTER__STATIC__SEEDS="master1@192.168.200.100,master2@192.168.200.110,master3@192.168.200.120"
emqx/emqx:v3.2.2
```



# 6.master3节点安装MQTT服务

```bash
docker run -it --network host --name emqtt-master3-1
-p 1883:1883
-p 18083:18083
-p 8083:8083
-p 8883:8883
-p 8080:8080
-e EMQX_NAME="master2"
-e EMQX_HOST=192.168.200.120
-e EMQX_LISTENER__TCP_EXTERNAL=1883
-e EMQX_WAIT_TIME=30
-e EMQX_CLUSTER__DISCOVERY="static"
-e EMQX_JOIN_CLUSTER="master3@192.168.200.120"
-e EMQX_CLUSTER__STATIC__SEEDS="master1@192.168.200.100,master2@192.168.200.110,master3@192.168.200.120"
emqx/emqx:v3.2.2
```



# 7.查看MQTT容器运行及状态

![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730155625190-216927172.png)

# 8.运行状态

```bash
docker stats![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730160032737-537833100.png)
```



# 9.登陆MQTTweb界面端口是18083

master1服务器ip地址:18083
默认账号：admin
默认密码：public
![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730160811544-668758424.png)

# 10.右上角搜索栏

代表刚才部署的三个集群
![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730160910867-2134959859.png)

# 11.运行状态

绿色代表正在运行
![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730161558387-439108870.png)

# 12.测试

![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730161901506-277665459.png)
![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730162214704-952793813.png)

# 13.参数说明

![img](https://img2020.cnblogs.com/blog/1805318/202007/1805318-20200730162603776-154888417.png)