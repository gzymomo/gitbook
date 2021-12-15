可以使用下面的命令来查看docker容器和镜像磁盘占用情况：

```
docker system df
```

可以看到类似如下的输出，包括镜像(Images)、容器（Containers）、数据卷（Local Volumes）、构建缓存（Build Cache）：

```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          5         5         5.158GB   0B (0%)
Containers      6         6         7.601MB   0B (0%)
Local Volumes   4         3         46.64GB   207MB (0%)
Build Cache     34        0         1.609MB   1.609MB
```

可以看到以上4种类型里面Local Volumes占用的磁盘空间最大。如果还想查看更详细的报告，则使用如下命令。

```
docker system df -v
```

可以看到很多输出，其中关于Local Volumes的是：

```
VOLUME NAME                                                        LINKS     SIZE
641d4976908910dca270a2bf5edf33408daf7474a0f27c850b6580b5936b6dd0   1         40.1GB
ovpn-data                                                          1         33.51kB
267b52c5eab8c6b8e0f0d1b02f8c68bdaffba5ea80a334a6d20e67d22759ef48   1         6.325GB
f4a3866ef41e3972e087883f8fa460ad947b787f2eafb6545c759a822fb6e30d   0         207MB
```

为了腾出空间，第一个想到的简单粗暴的办法是将所有停止的容器删除，命令如下。

```
docker system prune -a 
```

但是使用这个命令还是要谨慎，记得把需要使用的docker容器都先启动起来，不然那些没有启动的容器就会被这条命令删除了。基于安全的考虑，这个命令默认不会删除那些未被任何容器引用的数据卷，如果需要同时删除这些数据卷，你需要显式的指定 --volumns。
 所以如果想强制删除包括容器、网络、镜像、数据卷，可以使用如下命令。

```
docker system prune --all --force --volumns
```

第二个方法是把docker存储数据的路径改到磁盘空间更大的其他地方。如果是Mac用户，可以在图形化的Docker Desktop的设置里面修改Disk image location设置。

我尝试过第二种办法，把Disk image location改到外接的SSD上，并且尝试把之前的数据先同步过去。后面发现一个很大的问题，就是在mysql容器中导入数据会非常缓慢，这大概就是外接SSD在docker容器中的写入瓶颈。
 假如你只是想运行几个容器，而不是本地存储数据库数据，那么将docker数据存储到SSD是一个不错的办法。