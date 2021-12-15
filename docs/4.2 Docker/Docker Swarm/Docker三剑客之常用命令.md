# [Docker三剑客之常用命令](https://www.cnblogs.com/zhujingzhi/p/9815179.html)



常用命令：

```bash
# 创建服务
docker service create \ 
  --image nginx \
  --replicas 2 \
  nginx
 
# 更新服务
docker service update \ 
  --image nginx:alpine \
  nginx
 
# 删除服务
docker service rm nginx
 
# 减少服务实例(这比直接删除服务要好)
docker service scale nginx=0
 
# 增加服务实例
docker service scale nginx=5
 
# 查看所有服务
docker service ls
 
# 查看服务的容器状态
docker service ps nginx
 
# 查看服务的详细信息。
docker service inspect nginx
```





## 一、docker-machine

| 命令                   | 说明                                        |
| ---------------------- | ------------------------------------------- |
| docker-machine create  | 创建一个 Docker 主机（常用`-d virtualbox`） |
| docker-machine ls      | 查看所有的 Docker 主机                      |
| docker-machine ssh     | SSH 到主机上执行命令                        |
| docker-machine env     | 显示连接到某个主机需要的环境变量            |
| docker-machine inspect | 输出主机更多信息                            |
| docker-machine kill    | 停止某个主机                                |
| docker-machine restart | 重启某台主机                                |
| docker-machine rm      | 删除某台主机                                |
| docker-machine scp     | 在主机之间复制文件                          |
| docker-machine start   | 启动一个主机                                |
| docker-machine status  | 查看主机状态                                |
| docker-machine stop    | 停止一个主机                                |

## 二、docker-compose

| **命令**               | **说明**                       |
| ---------------------- | ------------------------------ |
| docker-compose build   | 建立或者重建服务               |
| docker-compose config  | 验证和查看Compose文件          |
| docker-compose create  | 创建服务                       |
| docker-compose down    | 停止和删除容器，网络，镜像和卷 |
| docker-compose events  | 从容器接收实时事件             |
| docker-compose exec    | 登录正在运行的容器执行命令     |
| docker-compose images  | 镜像列表                       |
| docker-compose kill    | 杀掉容器                       |
| docker-compose logs    | 查看容器的输出                 |
| docker-compose pause   | 暂停容器                       |
| docker-compose port    | 为端口绑定打印公共端口         |
| docker-compose ps      | 容器列表                       |
| docker-compose pull    | 下载服务镜像                   |
| docker-compose push    | 上传服务镜像                   |
| docker-compose restart | 重启容器                       |
| docker-compose rm      | 删除停止的容器                 |
| docker-compose run     | 运行一次性的命令               |
| docker-compose scale   | 设置服务的容器数量             |
| docker-compose start   | 启动服务                       |
| docker-compose stop    | 停止服务                       |
| docker-compose top     | 显示运行过程                   |
| docker-compose unpause | 暂停服务                       |
| docker-compose up      | 创建并启动容器                 |

 

## 三、docker swarm

| 命令                            | 说明                 |
| ------------------------------- | -------------------- |
| docker swarm init               | 初始化集群           |
| docker swarm join-token worker  | 查看工作节点的 token |
| docker swarm join-token manager | 查看管理节点的 token |
| docker swarm join               | 加入集群中           |

## 四、docker node

| 命令                | 说明                               |
| ------------------- | ---------------------------------- |
| docker node ls      | 查看所有集群节点                   |
| docker node rm      | 删除某个节点（`-f`强制删除）       |
| docker node inspect | 查看节点详情                       |
| docker node demote  | 节点降级，由管理节点降级为工作节点 |
| docker node promote | 节点升级，由工作节点升级为管理节点 |
| docker node update  | 更新节点                           |
| docker node ps      | 查看节点中的 Task 任务             |



## 五、docker service

| 命令                   | 说明                         |
| ---------------------- | ---------------------------- |
| docker service create  | 部署服务                     |
| docker service inspect | 查看服务详情                 |
| docker service logs    | 产看某个服务日志             |
| docker service ls      | 查看所有服务详情             |
| docker service rm      | 删除某个服务（`-f`强制删除） |
| docker service scale   | 设置某个服务个数             |
| docker service update  | 更新某个服务                 |



## 六、docker stack

| 命令                  | 说明                         |
| --------------------- | ---------------------------- |
| docker stack deploy   | 部署新的堆栈或更新现有堆栈   |
| docker stack ls       | 列出现有堆栈                 |
| docker stack ps       | 列出堆栈中的任务             |
| docker stack rm       | 删除堆栈                     |
| docker stack services | 列出堆栈中的服务             |
| docker stack down     | 移除某个堆栈（不会删除数据） |