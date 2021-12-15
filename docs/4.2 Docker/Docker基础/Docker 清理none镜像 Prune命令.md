- [Docker 清理none镜像 Prune命令](https://blog.csdn.net/gxf212/article/details/89676307)



# 如何清理none对象

> Docker 采用保守的方法来清理未使用的对象（通常称为“垃圾回收”），例如镜像、容器、卷和网络：
> 除非明确要求 Docker 这样做，否则通常不会删除这些对象。这可能会导致 Docker 使用额外的磁盘空间。
> 对于每种类型的对象，Docker 都提供了一条 prune 命令。
> 另外，可以使用 docker system prune一次清理多种类型的对象。本主题讲解如何使用这些 prune 修剪命令

## 修剪镜像

- 清理none镜像(虚悬镜像)

> 命令: docker image prune
>  默认情况下，docker image prune 命令只会清理 虚无镜像（没被标记且没被其它任何镜像引用的镜像）

```bash
root@instance-o70no2nw:~# docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

- 清理无容器使用的镜像

> 命令: docker image prune -a

默认情况下，系统会提示是否继续。要绕过提示，请使用 -f 或 --force 标志。
 可以使用 --filter 标志使用过滤表达式来限制修剪哪些镜像。例如，只考虑 24 小时前创建的镜像：

```bash
$ docker image prune -a --filter "until=24h"
```

## 修剪容器

停止容器后不会自动删除这个容器，除非在启动容器的时候指定了 –rm 标志。使用 docker ps -a 命令查看 Docker 主机上包含停止的容器在内的所有容器。你可能会对存在这么多容器感到惊讶，尤其是在开发环境。停止状态的容器的可写层仍然占用磁盘空间。要清理掉这些，可以使用 docker container prune 命令：

```java
$ docker container prune

WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示是否继续。要绕过提示，请使用 -f 或 --force 标志。

默认情况下，所有停止状态的容器会被删除。可以使用 --filter 标志来限制范围。例如，下面的命令只会删除 24 小时之前创建的停止状态的容器：

## 修剪卷

卷可以被一个或多个容器使用，并占用 Docker 主机上的空间。卷永远不会被自动删除，因为这么做会破坏数据。

```java
$ docker volume prune

WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
```

## 修剪网络

Docker 网络不会占用太多磁盘空间，但是它们会创建 iptables 规则，桥接网络设备和路由表条目。要清理这些东西，可以使用 docker network prune 来清理没有被容器未使用的网络。

```java
$ docker network prune
```

## 修剪一切

docker system prune 命令是修剪镜像、容器和网络的快捷方式。在 Docker 17.06.0 及以前版本中，还好修剪卷。在 Docker 17.06.1 及更高版本中必须为 docker system prune 命令明确指定 --volumes 标志才会修剪卷。

```bash
$ docker system prune

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] y
```

如果使用 Docker 17.06.1 或更高版本，同时也想修剪卷，使用 --volumes 标志。

```bash
$ docker system prune --volumes

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all volumes not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] y
```

