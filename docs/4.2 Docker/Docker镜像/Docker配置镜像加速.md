- 创建文件夹

```bash
sudo mkdir -p /etc/docker 
```

- 编辑配置文件

```bash
vi /etc/docker/daemon.json
```

- 将如下配置置入配置文件中

```yaml
{
 "registry-mirrors": [
  "https://paucfus3.mirror.aliyuncs.com",
  "https://hub-mirror.c.163.com",
    "https://registry.aliyuncs.com",
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

- 重启docker

```bash
systemctl daemon-reload && systemctl restart docker
```

