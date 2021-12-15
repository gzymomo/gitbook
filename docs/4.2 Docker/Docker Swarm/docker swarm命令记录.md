# 1. service创建

```bash
docker service create \
--with-registry-auth \
--mode global \
--name jmdiservice \
--config source=jmdiservice-application.properties,target=/root/application.properties \
--mount type=bind,source=/opt/lib,destination=/root/lib \
--env JAVA_OPTS="-Xms1024m -Xmx1024m" \
--publish 20036:20036 \
-td xx.xx.xx.xx:5000/zwx/jmdiservice:1123　　
```

## 命令详解：

--with-registry-auth：将registry身份验证详细信息发送给集群代理。

--mode global：全局模式，在这种模式下，每个节点上仅运行一个副本。
　　　另一种是复制模式(--replicas 3)，这种模式会部署期望数量的服务副本，并尽可能均匀地将各个副本分布在整个集群中。

--name：服务名称。（避免使用符号，容易解析错误）

--config：指定要向服务公开的配置。（添加配置参考docker config create）

--mount：将文件系统挂载附加到服务。（需要保留/读取的容器外信息）

--env：设置环境变量。

--publish：将端口发布为节点端口。（默认把需要发布的端口映射到本地）

-td：分配伪TTY，并后台运行。

**注意：**镜像地址、名称、标签一定填写正确

## 服务更新：

```bash
> docker service update --args "ping www.baidu.com" redis 　　# 添加参数
> docker service scale redis=4　　　　# 为服务扩(缩)容scale
> docker service update --image redis:3.0.7 redis　　# 更新服务的镜像版本
> docker service update --rollback redis　　# 回滚服务
```

# 2. 删除docker swarm及node节点

```bash
docker node rm -f [hostname]

docker swarm leave -f
```



# 3. docker service命令详解

[Docker系列之：docker service命令详解](https://www.cnblogs.com/yulibostu/articles/10642267.html)

```bash
[root@centos181001 ~]# docker service create --help

Usage:    docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]

Create a new service

Options:
      --config config                      Specify configurations to expose to the service
      --constraint list                    Placement constraints
      --container-label list               Container labels
                                            容器标签

      --credential-spec credential-spec    Credential spec for managed service account (Windows only)
  -d, --detach                             Exit immediately instead of waiting for the service to converge
                                            立即退出而不是等待服务收敛

      --dns list                           Set custom DNS servers
                                            指定DNS

      --dns-option list                    Set DNS options
                                            设置DNS选项

      --dns-search list                    Set custom DNS search domains
                                            设置DNS搜索域

      --endpoint-mode string               Endpoint mode (vip or dnsrr) (default "vip")
                                            端点模式 (vip or dnsrr) (default "vip")

      --entrypoint command                 Overwrite the default ENTRYPOINT of the image
                                            覆盖镜像的默认ENTRYPOINT

  -e, --env list                           Set environment variables
                                            设置环境变量

      --env-file list                      Read in a file of environment variables
                                            从配置文件读取环境变量

      --generic-resource list              User defined resources
      --group list                         Set one or more supplementary user groups for the container
      --health-cmd string                  Command to run to check health
                                            健康检查命令

      --health-interval duration           Time between running the check (ms|s|m|h)
                                            健康检查间隔 (ms|s|m|h)

      --health-retries int                 Consecutive failures needed to report unhealthy
                                            报告不健康需要连续失败次数

      --health-start-period duration       Start period for the container to initialize before counting retries towards unstable (ms|s|m|h)
                                            在重试计数到不稳定之前，开始容器初始化的时间段(ms|s|m|h)

      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)
                                            允许一次健康检查最长运行时间 (ms|s|m|h)

      --host list                          Set one or more custom host-to-IP mappings (host:ip)
                                            设置一个或多个自定义主机到IP映射 (host:ip)

      --hostname string                    Container hostname
                                            容器名称

      --init                               Use an init inside each service container to forward signals and reap processes
                                            在每个服务容器中使用init来转发信号并收集进程

      --isolation string                   Service container isolation mode
                                            服务容器隔离模式

  -l, --label list                         Service labels
                                            服务标签
      --limit-cpu decimal                  Limit CPUs
                                            CPU限制

      --limit-memory bytes                 Limit Memory
                                            内存限制

      --log-driver string                  Logging driver for service
      --log-opt list                       Logging driver options
      --mode string                        Service mode (replicated or global) (default "replicated")
      --mount mount                        Attach a filesystem mount to the service
      --name string                        Service name
                                            服务名称

      --network network                    Network attachments
                                            网络

      --no-healthcheck                     Disable any container-specified HEALTHCHECK
      --no-resolve-image                   Do not query the registry to resolve image digest and supported platforms
      --placement-pref pref                Add a placement preference
  -p, --publish port                       Publish a port as a node port
                                            发布端口

  -q, --quiet                              Suppress progress output
                                            简化输出

      --read-only                          Mount the container's root filesystem as read only
                                            将容器的根文件系统挂载为只读

      --replicas uint                      Number of tasks
                                            同时运行的副本数

      --reserve-cpu decimal                Reserve CPUs
                                            为本服务需要预留的CPU资源

      --reserve-memory bytes               Reserve Memory
                                            为本服务需要预留的内存资源

      --restart-condition string           Restart when condition is met ("none"|"on-failure"|"any") (default "any")
                                            满足条件时重新启动("none"|"on-failure"|"any") (default "any")

      --restart-delay duration             Delay between restart attempts (ns|us|ms|s|m|h) (default 5s)
                                            重启尝试之间的延迟 (ns|us|ms|s|m|h) (default 5s)

      --restart-max-attempts uint          Maximum number of restarts before giving up
                                            放弃前的最大重启次数

      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h) (default 0s)
                                            任务回滚之间的延迟(ns|us|ms|s|m|h) (default 0s)

      --rollback-failure-action string     Action on rollback failure ("pause"|"continue") (default "pause")
                                            回滚失败的操作("pause"|"continue") (default "pause")

      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback (default 0)
                                            回滚期间容忍的失败率(default 0)

      --rollback-monitor duration          Duration after each task rollback to monitor for failure (ns|us|ms|s|m|h) (default 5s)
                                            每次任务回滚后监视失败的持续时间 (ns|us|ms|s|m|h) (default 5s)

      --rollback-order string              Rollback order ("start-first"|"stop-first") (default "stop-first")
                                            回滚选项("start-first"|"stop-first") (default "stop-first")

      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0 to roll back all at once) (default 1)
                                            同时回滚的最大任务数（0表示一次回滚）（默认值为1）

      --secret secret                      Specify secrets to expose to the service
                                            指定要公开给服务的秘钥

      --stop-grace-period duration         Time to wait before force killing a container (ns|us|ms|s|m|h) (default 10s)
                                            在强行杀死容器之前等待的时间(ns|us|ms|s|m|h) (default 10s)

      --stop-signal string                 Signal to stop the container
                                            发出信号停止容器

  -t, --tty                                Allocate a pseudo-TTY
                                            分配伪终端

      --update-delay duration              Delay between updates (ns|us|ms|s|m|h) (default 0s)
                                            更新之间的延迟(ns|us|ms|s|m|h) (default 0s)

      --update-failure-action string       Action on update failure ("pause"|"continue"|"rollback") (default "pause")
                                            更新失败后选项("pause"|"continue"|"rollback") (default "pause")
      --update-max-failure-ratio float     Failure rate to tolerate during an update (default 0)
                                            更新期间容忍的故障率（默认为0）

      --update-monitor duration            Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 5s)
                                            每次更新任务后监视失败的持续时间（ns | us | ms | s | m | h）（默认为5s）

      --update-order string                Update order ("start-first"|"stop-first") (default "stop-first")
                                            更新选项 ("start-first"|"stop-first") (default "stop-first")

      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)
                                            同时更新的最大任务数（0表示一次更新所有任务）（默认值为1）

  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])
      --with-registry-auth                 Send registry authentication details to swarm agents
                                            将注册表验证详细信息发送给swarm代理

  -w, --workdir string                     Working directory inside the container
                                            指定容器内工作目录(workdir)
```

