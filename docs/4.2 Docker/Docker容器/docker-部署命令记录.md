[TOC]

# 一、代码质量监测工具SonarQube
```bash
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=123456 --name postgres postgres:10

docker run -d -p 9900:9000 -e "SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.0.50:5432/sonar" -e "SONARQUBE_JDBC_USERNAME=postgres" -e "SONARQUBE_JDBC_PASSWORD=123456" --name sonarqube sonarqube
```

# 二、Mysql数据库

```bash
docker run -p 3308:3306 --name gly-test -v /home/logs/mysql/test/conf:/etc/mysql/conf.d -v /home/logs/mysql/test/logs:/logs -v /home/logs/mysql/test/data:/var/lib/mysql -v /etc/localtime:/etc/localtime -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 --lower_case_table_names=1
```

# 三、Nginx
```bash
docker run -d -p 8095:80 --name nginx-8095 -v /var/project/nginx/html:/usr/share/nginx/html -v /etc/nginx/conf:/etc/nginx -v /var/project/logs/nginx:/var/log/nginx nginx
```

# 四、Node_exporter
```bash
docker run -d -p 9101:9100 --net="bridge" --pid="host" --name=node-exporter -v "/:/host:ro,rslave" quay.io/prometheus/node-exporter --path.rootfs /host
```

# 五、Minio
```bash
docker run -d -p 9000:9000 --name minio -e MINIO_ACCESS_KEY=zlkjminio -e MINIO_SECRET_KEY=zlkjminio -v /var/project/minio/data:/data -v /var/project/minio/config:/root/.minio minio/minio server /data
```

# 六、redis

```bash
docker run --name redis -p 6379:6379 -v /var/project/redis/data:/data -v /var/project/redis/conf:/usr/local/etc/redis/redis.conf -d redis redis-server --appendonly yes

# 命令说明
--name redis         　　　　　　　　 　   启动后容器名为 my-redis
-p 6379:6379           　　　　　　　　　　  将容器的 6379 端口映射到宿主机的 6379 端口
-v /usr/local/workspace/redis/data:/data           将容器  /usr/local/workspace/redis/data 日志目录挂载到宿主机的 /data
-v /home/redis/conf:/usr/local/etc/redis/redis.conf    将容器  /usr/local/etc/redis/redis.conf 配置目录挂载到宿主机的 /conf
redis-server --appendonly yes　　　          在容器执行redis-server启动命令，并打开redis持久化配置
redis-server --requirepass 123456              配置redis的登录密码
```



# 七、halo

```bash
docker run -d --name halo -p 7081:8090  -v /var/project/halo:/root/.halo ruibaby/halo
```

# 八、docker搭建yapi

## 1.启动 MongoDB

```bash
# docker run -d --name mongo-yapi mongo
```

## 2.获取yapi镜像

```bash
# docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
```

## 3.初始化数据库索引及管理员账号

```bash
# docker run -it --rm \
  --link mongo-yapi:mongo \
  --entrypoint npm \
  --workdir /api/vendors \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  run install-server
```

## 4.启动yapi服务

```bash
# docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 3000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js
```

## 5.访问

访问 http://localhost:3000 登录账号 admin@admin.com，密码 ymfe.org

## 6.其他命令

```bash
#启动停止
# docker stop yapi
# docker start yapi
# 开机自启动
# chmod +x /etc/rc.d/rc.local
# systemctl daemon-reload
# sudo service docker restart
# docker start mongo-yapi
# docker start yapi
```