自动补齐需要依赖工具 bash-complete，如果没有，则需要手动安装，命令如下：

`[root@docker ~]# yum -y install bash-completion`

安装成功后，得到文件为  /usr/share/bash-completion/bash_completion。

前面已经安装了 bash_completion，执行如下命令：

`[root@docker ~]# source /usr/share/bash-completion/bash_completion`

再次尝试，发现可以正常列出docker的子命令，示例如下：
```bash
[root@docker ~]# docker  （docker + 空格 + 连续按2次Tab键）
attach    container  engine    history   inspect   logs      port     restart   search    stats    top      volume
build     context    events    image     kill      network   ps       rm        secret    stop     trust    wait
builder   cp         exec      images    load      node      pull     rmi       service   swarm    unpause
commit    create     export    import    login     pause     push     run       stack     system   update
config    diff       help      info      logout    plugin    rename   save      start     tag      version
```

