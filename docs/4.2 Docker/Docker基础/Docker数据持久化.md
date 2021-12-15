- [Docker之 数据持久化](https://www.cnblogs.com/andy6/p/10106631.html)

- [Docker 基础知识 - 使用绑定挂载(bind mounts)管理应用程序数据](https://www.cnblogs.com/ittranslator/p/13352727.html)
- [Docker 基础知识 - 使用卷(volume)管理应用程序数据](https://www.cnblogs.com/ittranslator/p/13303417.html)
- [Docker 基础知识 - 使用 tmpfs 挂载(tmpfs mounts)管理应用程序数据](https://www.cnblogs.com/ittranslator/p/13423921.html)

- [Docker 数据管理介绍](https://www.escapelife.site/posts/c2e250ea.html)

# 一、数据持久化

## 1.1 卷（volumes）

卷（volumes）是 Docker 容器生产和使用持久化数据的首选机制。[绑定挂载（bind mounts）](https://docs.docker.com/storage/bind-mounts/)依赖于主机的目录结构，卷（volumes）完全由 Docker 管理。卷与绑定挂载相比有几个优势：

- 卷比绑定挂载更容易备份或迁移。
- 您可以使用 Docker CLI 命令或 Docker API 来管理卷。
- 卷可以在 Linux 和 Windows 容器上工作。
- 卷可以更安全地在多个容器之间共享。
- 卷驱动程序允许您在远程主机或云提供商上存储卷、加密卷的内容或添加其他功能。
- 新卷的内容可以由容器预先填充。（New volumes can have their content pre-populated by a container.）

此外，与将数据持久化到容器的可写层相比，卷通常是更好的选择，因为卷不会增加使用它的容器的大小，而且卷的内容存在于给定容器的生命周期之外。

![docker-types-of-mounts-volume](https://img2020.cnblogs.com/blog/2074831/202007/2074831-20200715011755142-1123249985.png#center)

如果容器生成非持久性状态数据，请考虑使用 [tmpfs 挂载（tmpfs mount）](https://docs.docker.com/storage/tmpfs/)以避免将数据永久存储在任何位置，并通过避免写入容器的可写层来提高容器的性能。

卷使用 `rprivate` 绑定传播，并且绑定传播对于卷是不可配置的。





## 1.2 绑定挂载

绑定挂载（bind mounts）在 Docker 的早期就已经出现了。与卷相比，绑定挂载的功能有限。当您使用绑定挂载时，主机上的文件或目录将挂载到容器中。文件或目录由其在主机上的完整或相对路径引用。相反地，当您使用卷时，在主机上 Docker 的存储目录中创建一个新目录，Docker 管理该目录的内容。

该文件或目录不需要已经存在于 Docker 主机上。如果还不存在，则按需创建。绑定挂载的性能非常好，但它们依赖于主机的文件系统，该文件系统具有特定的可用目录结构。如果您正在开发新的 Docker 应用程序，请考虑改用[命名卷](https://ittranslator.cn/backend/docker/2020/07/04/docker-storage-volumes.html)。不能使用 Docker CLI 命令直接管理绑定挂载。

![docker-types-of-mounts-bind](https://img2020.cnblogs.com/blog/2074831/202007/2074831-20200721011204681-609017834.png#center)



## 1.3 `tmpfs` 挂载

[卷（volumes）](https://ittranslator.cn/backend/docker/2020/07/04/docker-storage-volumes.html) 和 [绑定挂载（bind mounts）](https://ittranslator.cn/backend/docker/2020/07/13/docker-storage-bind-mounts.html) 允许您在主机和容器之间共享文件，这样即使在容器停止后也可以持久存储数据。

如果在 Linux 上运行 Docker，那么还有第三种选择：`tmpfs` 挂载。当您创建带有 `tmpfs` 挂载的容器时，容器可以在容器的可写层之外创建文件。

与卷和绑定挂载不同，`tmpfs` 挂载是临时的，只存留在主机内存中。当容器停止时，`tmpfs` 挂载将被删除，在那里写入的文件不会被持久化。

![docker-types-of-mounts-tmpfs](https://img2020.cnblogs.com/blog/2074831/202008/2074831-20200802234239260-535898392.png#center)

这对于临时存储您不想在主机或容器可写层中持久存储的敏感文件非常有用。





## 1.4 选择 `-v` 或者 `--mount` 标记

最初，`-v` 或 `--volume` 标记用于独立容器，`--mount` 标记用于集群服务。但是，从 Docker 17.06 开始，您也可以将 `--mount` 用于独立容器。通常，`--mount` 标记表达更加明确和冗长。最大的区别是 `-v` 语法将所有选项组合在一个字段中，而 `--mount` 语法将选项分离。下面是每个标记的语法比较。

> 提示：新用户推荐使用 `--mount` 语法，有经验的用户可能更熟悉 `-v` or `--volume` 语法，但是更鼓励使用 `--mount` 语法，因为研究表明它更易于使用。

- -v或--volume: 由三个字段组成，以冒号(:)分隔。字段必须按照正确的顺序排列，且每个字段的含义不够直观明显。
  - 对于绑定挂载（bind mounts）, 第一个字段是主机上文件或目录的路径。
  - 第二个字段是容器中文件或目录挂载的路径。
  - 第三个字段是可选的，是一个逗号分隔的选项列表，比如 `ro`、`consistent`、 `delegated`、 `cached`、 `z` 和 `Z`。这些选项会在本文下面讨论。
- --mount：由多个键-值对组成，以逗号分隔，每个键-值对由一个 <key>=<value>元组组成。--mount语法比-v或--volume更冗长，但是键的顺序并不重要，标记的值也更容易理解。
  - 挂载的类型（`type`），可以是 `bind`、`volume` 或者 `tmpfs`。本主题讨论绑定挂载（bind mounts），因此类型（`type`）始终为绑定挂载（`bind`）。
  - 挂载的源（`source`），对于绑定挂载，这是 Docker 守护进程主机上的文件或目录的路径。可以用 `source` 或者 `src` 来指定。
  - 目标（`destination`），将容器中文件或目录挂载的路径作为其值。可以用 `destination`、`dst` 或者 `target` 来指定。
  - `readonly` 选项（如果存在），则会将绑定挂载以[只读形式挂载到容器](https://www.cnblogs.com/ittranslator/p/13352727.html#use-a-read-only-bind-mount)中。
  - `bind-propagation` 选项（如果存在），则更改[绑定传播](https://www.cnblogs.com/ittranslator/p/13352727.html#configure-bind-propagation)。 可能的值是 `rprivate`、 `private`、 `rshared`、 `shared`、 `rslave` 或 `slave` 之一。
  - [`consistency`](https://www.cnblogs.com/ittranslator/p/13352727.html#configure-mount-consistency-for-macos) 选项（如果存在）， 可能的值是 `consistent`、 `delegated` 或 `cached` 之一。 这个设置只适用于 Docker Desktop for Mac，在其他平台上被忽略。
  - `--mount` 标记不支持用于修改 selinux 标签的 `z` 或 `Z`选项。

## 1.5  `-v` 和 `--mount` 行为之间的差异

由于 `-v` 和 `-volume` 标记长期以来一直是 Docker 的一部分，它们的行为无法改变。这意味着 `-v` 和 `-mount` 之间有一个不同的行为。

如果您使用 `-v` 或 `-volume` 来绑定挂载 Docker 主机上还不存在的文件或目录，则 `-v` 将为您创建它。它总是作为目录创建的。

如果使用 `--mount` 绑定挂载 Docker 主机上还不存在的文件或目录，Docker 不会自动为您创建它，而是产生一个错误。



# 二、数据持久化方式

容器中数据持久化主要有两种方式：

1. 数据卷（Data Volumes）
2. 数据卷容器（Data Volumes Dontainers）

## 2.1 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，可以绕过UFS（Unix File System）。

1. 数据卷可以在容器之间共享和重用
2. 对数据卷的修改会立马生效
3. 对数据卷的更新，不会影响镜像
4. 数据卷默认会一直存在，即使容器被删除
5. 一个容器可以挂载多个数据卷

注意：数据卷的使用，类似于 Linux 下对目录或文件进行 mount。

## 2.2 创建数据卷

示例：

```bash
docker run --name nginx-data -v /mydir nginx
```

执行如下命令即可查看容器构造的详情：

```bash
docker inspect 容器ID
```

由测试可知：

1. Docker会自动生成一个目录作为挂载的目录。
2. 即使容器被删除，宿主机中的目录也不会被删除。

## 2.3 删除数据卷

数据卷是被设计来持久化数据的，因此，删除容器并不会删除数据卷。如果想要在删除容器时同时删除数据卷，可使用如下命令：

```bash
docker rm -v 容器ID
```

这样既可在删除容器的同时也将数据卷删除。

## 2.4 挂载宿主机目录作为数据卷

```bash
docker run --name nginx-data2 -v /host-dir:/container-dir nginx
```

这样既可将宿主机的/host-dir路径加载到容器的/container-dir中。

需要注意的是：

宿主机路径尽量设置绝对路径——如果使用相对路径会怎样？

1. 测试给答案

如果宿主机路径不存在，Docker会自动创建

**TIPS**

Dockerfile暂时不支持这种形式。

## 2.5 挂载宿主机文件作为数据卷

```bash
docker run --name nginx-data3 -v /文件路径:/container 路径 nginx
```

## 2.6 指定权限

默认情况下，挂载的权限是读写权限。也可使用:ro 参数指定只读权限。

示例：

```bash
docker run --name nginx-data4 -v /host-dir:/container-dir:ro nginx
```

这样，在容器中就只能读取/container-dir中的文件，而不能修改了。

## 2.7 数据卷容器

如果有数据需要在多个容器之间共享，此时可考虑使用数据卷容器。

创建数据卷容器：

```bash
docker run --name nginx-volume -v /data nginx
```

在其他容器中使用-volumes-from 来挂载nginx-volume容器中的数据卷。

```bash
docker run --name v1 --volumes-from nginx-volume nginx
docker run --name v2 --volumes-from nginx-volume nginx
```

这样：

v1、v2两个容器即可共享nginx-volume这个容器中的文件。

即使nginx-volume停止，也不会有任何影响。



# 三、数据绑定案例

## 3.1 启动带有绑定挂载的容器

考虑这样一个情况：您有一个目录 `source`，当您构建源代码时，工件被保存到另一个目录 `source/target/` 中。您希望工件在容器的 `/app/` 目录可用，并希望每次在开发主机上构建源代码时，容器能访问新的构建。使用以下命令将 `target/` 目录绑定挂载到容器的 `/app/`。在 `source` 目录中运行命令。在 Linux 或 macOS 主机上，`$(pwd)` 子命令扩展到当前工作目录。

下面的 `--mount` 和 `-v` 示例会产生相同的结果。除非在运行第一个示例之后删除了 `devtest` 容器，否则不能同时运行它们。

`--mount`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```

使用 `docker inspect devtest` 验证绑定挂载是否被正确创建。查看 `Mounts` 部分：

```bash
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
],
```

这表明挂载是一个 `bind` 挂载，它显示了正确的源和目标，也显示了挂载是可读写的，并且传播设置为 `rprivate`。

## 3.2 挂载到容器上的非空目录

如果您将其绑定挂载到容器上的一个非空目录中，则该目录的现有内容会被绑定挂载覆盖。这可能是有益的，例如当您想测试应用程序的新版本而不构建新镜像时。然而，它也可能是令人惊讶的，这种行为不同于 [docker volumes](https://ittranslator.cn/backend/docker/2020/07/04/docker-storage-volumes.html)。

这个例子被设计成极端的，仅仅使用主机上的 `/tmp/` 目录替换容器的 `/usr/` 目录的内容。在大多数情况下，这将导致容器无法正常工作。

`--mount` 和 `-v` 示例有相同的结果。

`--mount`：

```bash
$ docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

`-v`：

```bash
$ docker run -d \
  -it \
  --name broken-container \
  -v /tmp:/usr \
  nginx:latest

docker: Error response from daemon: oci runtime error: container_linux.go:262:
starting container process caused "exec: \"nginx\": executable file not found in $PATH".
```

容器被创建，但没有启动。删除它：

```bash
$ docker container rm broken-container
```

## 3.3 使用只读绑定挂载

对于一些开发应用程序，容器需要写入绑定挂载，因此更改将传播回 Docker 主机。在其他时候，容器只需要读访问。

这个示例修改了上面的示例，但是通过在容器内的挂载点之后的选项列表（默认为空）中添加 `ro`，将目录挂载为只读绑定挂载。当有多个选项时，使用逗号分隔它们。

`--mount` 和 `-v` 示例有相同的结果。

`--mount`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:ro \
  nginx:latest
```

使用 `docker inspect devtest` 验证绑定挂载是否被正确创建。查看 `Mounts` 部分：

```bash
"Mounts": [
    {
        "Type": "bind",
        "Source": "/tmp/source/target",
        "Destination": "/app",
        "Mode": "ro",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

停止容器：

```bash
$ docker container stop devtest

$ docker container rm devtest
```

## 3.4 配置绑定传播

对于绑定挂载和卷，绑定传播默认都是 `rprivate` 。只能为绑定挂载配置，而且只能在 Linux 主机上配置。绑定传播是一个高级主题，许多用户从不需要配置它。

绑定传播是指在给定绑定挂载或命名卷中创建的挂载是否可以传播到该挂载的副本。考虑一个挂载点 `/mnt`，它也挂载在 `/tmp` 上。传播设置控制 `/tmp/a` 上的挂载是否也可以在 `/mnt/a` 上使用。每个传播设置都有一个递归对应点。在递归的情况下，考虑一下 `/tmp/a` 也被挂载为 `/foo`。传播设置控制 `/mnt/a` 和/或 `/tmp/a` 是否存在。

| 传播设置 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| shared   | 原始挂载的子挂载公开给副本挂载，副本挂载的子挂载也传播给原始挂载。 |
| slave    | 类似于共享(shared)挂载，但仅在一个方向上。如果原始挂载公开子挂载，副本挂载可以看到它。但是，如果副本挂载公开子挂载，则原始挂载无法看到它。 |
| private  | 该挂载是私有的。原始挂载的子挂载不公开给副本挂载，副本挂载的子挂载也不公开给原始挂载。 |
| rshared  | 与 shared 相同，但传播也扩展到嵌套在任何原始或副本挂载点中的挂载点。 |
| rslave   | 与 slave 相同，但传播也扩展到嵌套在任何原始或副本挂载点中的挂载点。 |
| rprivate | 默认值。与 private 相同，这意味着原始或副本挂载点中的任何位置的挂载点都不会在任何方向传播。 |

当你在挂载点上设置绑定传播之前，主机文件系统需要已经支持绑定传播。

有关绑定传播的更多信息，请参见 [Linux 内核共享子树文档](https://www.kernel.org/doc/Documentation/filesystems/sharedsubtree.txt)。

下面的示例两次将 `target/` 目录挂载到容器中，第二次挂载设置了 `ro` 选项和 `rslave` 绑定传播选项。

`--mount` 和 `-v` 示例有相同的结果。

`--mount`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  --mount type=bind,source="$(pwd)"/target,target=/app2,readonly,bind-propagation=rslave \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  -v "$(pwd)"/target:/app2:ro,rslave \
  nginx:latest
```

现在，如果您创建 `/app/foo/`， `/app2/foo/` 也存在。

## 3.5 配置 selinux 标签

如果使用 `selinux` ，则可以添加 `z` 或 `Z` 选项，以修改挂载到容器中的主机文件或目录的 selinux 标签。这会影响主机上的文件或目录，并且会产生超出 Docker 范围之外的后果。

- `z` 选项表示绑定挂载内容在多个容器之间共享。
- `Z` 选项表示绑定挂载内容是私有的、非共享的。

使用这些选项时要**格外**小心。使用 `Z` 选项绑定挂载系统目录(如 `/home` 或 `/usr` )会导致您的主机无法操作，您可能需要重新手动标记主机文件。

> 重要提示：当对服务使用绑定挂载时，selinux 标签(`:Z` 和 `:Z`) 以及 `:ro` 将被忽略。详情请参阅 [moby/moby #32579](https://github.com/moby/moby/issues/32579)。

这个示例设置了 `z` 选项来指定多个容器可以共享绑定挂载的内容：

无法使用 `--mount` 标记修改 selinux 标签。

```bash
$ docker run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app:z \
  nginx:latest
```

# 四、卷（volumes）案例

与绑定挂载不同，您可以在任何容器的作用域之外创建和管理卷。

## 4.1 创建一个卷

```bash
$ docker volume create my-vol
```

卷列表：

```bash
$ docker volume ls
# 输出结果：
DRIVER              VOLUME NAME
local               my-vol
```

检查卷：

```bash
$ docker volume inspect my-vol
# 输出结果：
[
    {
        "CreatedAt": "2020-07-04T07:06:47Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

删除卷：

```bash
$ docker volume rm my-vol
```



## 4.2 启动一个带有卷的容器

如果您启动一个有尚不存在的卷的容器，Docker 将为您创建这个卷。下面的示例将卷 `myvol2` 挂载到容器中的 `/app/` 中。

下面的 `--mount` 和 `-v` 示例会产生相同的结果。除非在运行第一个示例之后删除了 `devtest` 容器和 `myvol2` 卷，否则不能同时运行它们。

`--mount`：

```bash
$ docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

使用 `docker inspect devtest` 验证卷的创建和挂载是否正确。查看 `Mounts` 部分：

```bash
"Mounts": [
    {
        "Type": "volume",
        "Name": "myvol2",
        "Source": "/var/lib/docker/volumes/myvol2/_data",
        "Destination": "/app",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```

这表明挂载是一个卷，它显示了正确的源和目标，并且挂载是可读写的。

停止容器并删除卷。注意删除卷是一个单独的步骤。

```bash
$ docker container stop devtest

$ docker container rm devtest

$ docker volume rm myvol2
```

## 4.3 启动带有卷的服务

启动服务并定义卷时，每个服务容器都使用自己的本地卷。 如果使用本地（`local`）卷驱动程序，则没有任何容器可以共享此数据，但某些卷驱动程序确实支持共享存储。Docker for AWS 和 Docker for Azure 都支持使用 Cloudstor 插件的持久存储。

下面的示例使用四个副本启动 `nginx` 服务，每个副本使用一个名为 `myvol2` 的本地卷。

```bash
$ docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```

使用 `docker service ps devtest-service` 验证服务是否正在运行：

```bash
$ docker service ps devtest-service

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
4d7oz1j85wwn        devtest-service.1   nginx:latest        moby                Running             Running 14 seconds ago
```

删除服务，该服务将停止其所有任务：

```bash
$ docker service rm devtest-service
```

删除服务不会删除该服务创建的任何卷。删除卷是一个单独的步骤。

#### 服务的语法差异

`docker service create` 命令不支持 `-v` 或 `--volume` 标记，在将卷挂载到服务的容器中时，必须使用 `--mount` 标记。

## 4.4 使用容器填充卷

如果您启动了一个创建新卷的容器，如上所述，并且该容器在要挂载的目录(例如上面的 `/app/`)中有文件或目录，那么该目录的内容将复制到新卷中。然后容器挂载并使用该卷，使用该卷的其他容器也可以访问预填充的内容。

为了说明这一点，这个例子启动了一个 `nginx` 容器，并用容器的 `/usr/share/nginx/html` 目录中的内容填充新的卷 `nginx-vol`，这个目录是 Nginx 存储默认的 HTML 内容的地方。

下面的 `--mount` 和 `-v` 示例具有相同的最终结果。

`--mount`：

```bash
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest
```

运行两个示例中的任何一个之后，运行以下命令来清理容器和卷。注意：删除卷是一个单独的步骤。

```bash
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```

## 4.5 使用只读卷

对于某些开发应用程序，容器需要写入绑定挂载，以便更改传播回 Docker 主机。在其他时候，容器只需要对数据进行读访问。记住，多个容器可以挂载相同的卷，并且可以对其中一些容器以读写方式挂载，而对其他容器以只读方式挂载。

这个示例修改了上面的示例，但是通过在容器内的挂载点之后的选项列表（默认为空）中添加 `ro`，将目录挂载为只读卷。当有多个选项时，使用逗号分隔它们。

下面 `--mount` 和 `-v` 示例有相同的结果。

`--mount`：

```bash
$ docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
```

`-v`：

```bash
$ docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html:ro \
  nginx:latest
```

使用 `docker inspect nginxtest` 验证是否正确创建了只读挂载。查看 `Mounts` 部分：

```bash
"Mounts": [
    {
        "Type": "volume",
        "Name": "nginx-vol",
        "Source": "/var/lib/docker/volumes/nginx-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "",
        "RW": false,
        "Propagation": ""
    }
],
```

停止并删除容器，再删除卷。删除卷是一个单独的步骤。

```bash
$ docker container stop nginxtest

$ docker container rm nginxtest

$ docker volume rm nginx-vol
```

# 五、在机器之间共享数据

在构建故障容错的应用程序时，您可能需要配置同一服务的多个副本，以访问相同的文件。

![volumes-shared-storage](https://img2020.cnblogs.com/blog/2074831/202007/2074831-20200715012136113-1402102667.png#center)

在开发应用程序时，有几种方法可以实现这一点。一种方法是向您的应用程序添加逻辑，在云对象存储系统（如 Amazon S3）上存储文件。另一个方法是使用支持将文件写入外部存储系统（如 NFS 或 Amazon S3）的驱动程序来创建卷。

卷驱动程序使您可以从应用程序逻辑中抽象底层存储系统。例如，如果您的服务使用带有 NFS 驱动程序的卷，那么您可以更新服务以使用其他的驱动程序（例如，将数据存储在云上），而无需更改应用程序逻辑。

## 5.1 使用卷驱动程序

当您使用 `docker volume create` 创建卷时，或者当您启动使用尚未创建的卷的容器时，可以指定一个卷驱动程序。下面的示例使用 `vieux/sshfs` 卷驱动程序，首先在创建独立卷时使用，然后在启动创建新卷的容器时使用。

### 初始设置

这个示例假定您有两个节点，第一个节点是 Docker 主机，可以使用 SSH 连接到第二个节点。

在 Docker 主机上，安装 `vieux/sshfs` 插件：

```bash
$ docker plugin install --grant-all-permissions vieux/sshfs
```

### 使用卷驱动程序创建卷

本例指定了一个 SSH 密码，但是如果两个主机配置了共享密钥，则可以省略该密码。每个卷驱动程序可能有零个或多个可配置选项，每个选项都使用 `-o` 标记指定。

```bash
$ docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```

### 启动使用卷驱动程序创建卷的容器

本例指定了一个 SSH 密码，但是如果两个主机配置了共享密钥，则可以省略该密码。每个卷驱动程序可能有零个或多个可配置选项。**如果卷驱动程序要求您传递选项，则必须使用 `--mount` 标记挂载卷，而不是使用 `-v`。**

```bash
$ docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
```

### 创建创建 NFS 卷的服务

此示例显示如何在创建服务时创建 NFS 卷。本例使用 `10.0.0.10` 作为 NFS 服务器，使用 `/var/docker-nfs` 作为 NFS 服务器上的出口目录。请注意，指定的卷驱动程序是 `local`。

#### NFSV3

```bash
$ docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```

#### NFSV4

```bash
docker service create -d \
    --name nfs-service \
    --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,"volume-opt=o=10.0.0.10,rw,nfsvers=4,async"' \
    nginx:latest
```

## 5.2 备份、还原或迁移数据卷

卷对于备份、还原和迁移非常有用。使用 `--volumes-from` 标记创建一个挂载该卷的新容器。

### 备份容器

例如，创建一个名为 `dbstore` 的新容器：

```bash
$ docker run -v /dbdata --name dbstore ubuntu /bin/bash
```

然后在下一条命令中，我们：

- 启动一个新容器并从 `dbstore` 容器挂载卷
- 挂载一个本地主机目录作为 `/backup`
- 传递一个命令，将 `/dbdata` 卷的内容压缩到目录 `/backup` 中的 `backup.tar` 文件。

```bash
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

当命令完成且容器停止时，我们留下了 `/dbdata` 卷的一个备份。

### 从备份中还原容器

使用刚刚创建的备份，您可以将其还原到同一个容器，或者其他地方创建的容器。

例如，创建一个名为 `dbstore2` 的新容器：

```bash
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```

然后在新容器的数据卷中解压备份文件：

```bash
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

您可以使用上述技术，使用您喜欢的工具自动执行备份、迁移和还原测试。

## 5.3 删除卷

当删除容器后，Docker 数据卷仍然存在。有两种类型的卷需要考虑：

- **命名卷**具有来自容器外部的特定源，例如 `awesome:/bar`。
- **匿名卷**没有特定的源，因此当容器被删除时，通知 Docker 引擎守护进程删除它们。

### 删除匿名卷

要自动删除匿名卷，请使用 `--rm` 选项。例如，这个命令创建一个匿名的 `/foo` 卷。当容器被删除时，Docker 引擎会删除 `/foo` 卷，但不会删除 `awesome` 卷。

```bash
$ docker run --rm -v /foo -v awesome:/bar busybox top
```

### 删除所有卷

要删除所有未使用的卷并释放空间，请执行以下操作：

```bash
$ docker volume prune
```

# 六、tmpfs 挂载

## 6.1 tmpfs 挂载的局限性

- 不同于卷和绑定挂载，不能在容器之间共享 `tmpfs` 挂载。
- 这个功能只有在 Linux 上运行 Docker 时才可用。

## 6.2 选择 `--tmpfs` 或 `--mount` 标记

最初，`--tmpfs` 标记用于独立容器，`--mount` 标记用于集群服务。但是从 Docker 17.06 开始，您还可以将 `--mount` 与独立容器一起使用。通常，`--mount` 标记表达更加明确和冗长。最大的区别是，`--tmpfs` 标记不支持任何可配置的选项。

- `--tmpfs`: 设置 `tmpfs` 挂载不允许您指定任何可配置选项，并且只能与独立容器一起使用。
- --mount：由多个键-值对组成，，每个键-值对由一个<key>=<value>元组组成。--mount语法比--tmpfs更冗长：
  - 挂载的类型（`type`），可以是 `bind`、`volume` 或者 `tmpfs`。本主题讨论 `tmpfs`，因此类型（`type`）始终为 `tmpfs`。
  - 目标（`destination`），将容器中 `tmpfs` 挂载设置的路径作为其值。可以用 `destination`、`dst` 或者 `target` 来指定。
  - `tmpfs-size` 和 `tmpfs-mode` 选项。请查看下文的 [指定 tmpfs 选项](https://www.cnblogs.com/ittranslator/p/13423921.html#specify-tmpfs-options)。

下面的示例尽可能同时展示 `--mount` 和 `--tmpfs` 两种语法，并且先展示 `--mount`。

### `--tmpfs` 和 `--mount` 行为之间的差异

- `--tmpfs` 标记不允许指定任何可配置选项。
- `--tmpfs` 标记不能用于集群服务。对于集群服务，您必须使用 `--mount`。

## 6.3 在容器中使用 tmpfs 挂载

要在容器中使用 `tmpfs` 挂载， 请使用 `--tmpfs` 标记, 或者使用带有 `type=tmpfs` 和 `destination` 选项的 `--mount` 标记。没有用于 `tmpfs` 挂载的源(`source`)。

下面的示例在 Nginx 容器中的 `/app` 创建一个 `tmpfs` 挂载。第一个例子使用 `--mount` 标记，第二个使用 `--tmpfs` 标记。

`--mount`：

```bash
$ docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```

`--tmpfs`：

```bash
$ docker run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

通过运行 `docker container inspect tmptest` 来验证挂载是否是 `tmpfs` 挂载，查看 `Mounts` 部分：

```bash
"Tmpfs": {
    "/app": ""
},
```

删除容器：

```bash
$ docker container stop tmptest

$ docker container rm tmptest
```

### 指定 tmpfs 选项

`tmpfs` 挂载允许两个配置选项，两个选项都不是必需的。 如果需要指定这些选项，则必须使用 `--mount` 标记，因为 `--tmpfs` 标记不支持。

| 选项         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `tmpfs-size` | tmpfs 挂载的大小（以字节为单位）。默认无限制。               |
| `tmpfs-mode` | tmpfs 的八进制文件模式。例如，`700` 或 `0770`。默认为 `1777` 或全局可写。 |

下面的示例将 `tmpfs-mode` 设置为 `1770`，因此在容器中它不是全局可读的。

```bash
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app,tmpfs-mode=1770 \
  nginx:latest
```