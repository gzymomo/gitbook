# 一、Docker部署postgres

先拉取镜像，这里选择版本10，更多版本请查看【[Postgres Docker](https://hub.docker.com/_/postgres)】。

```bash
docker pull postgres:10
```

通过以下命令启动一个实例：

```bash
docker run -itd \
    --name pkslow-postgres \
    -e POSTGRES_DB=pkslow \
    -e POSTGRES_USER=pkslow \
    -e POSTGRES_PASSWORD=pkslow \
    -e PGDATA=/var/lib/postgresql/data/pgdata \
    -v /custom/mount:/var/lib/postgresql/data \
    -p 5432:5432 \
    postgres:10
```

# 二、postgres优秀客户端

[DBeaver](https://dbeaver.io/)支持多种数据库，如PostgreSQL/MySQL/DB2/Oracle等，支持`Mac/Win/Linux`，还支持中文，比较全面。

[![img](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/10/docker-install-postgres.DBeaver.png)](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/10/docker-install-postgres.DBeaver.png)

[pgAdmin](https://www.pgadmin.org/)基于`Web`的客户端工具。

[![img](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/10/docker-install-postgres.pgAdmin.png)](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/10/docker-install-postgres.pgAdmin.png)

