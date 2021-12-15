[TOC]

# 1、删除本地已有的arthas-demo docker container（非必要）
` $ docker stop arthas-demo || true && docker rm arthas-demo || true `

# 2、启动arthas-demo
` $ docker run --name arthas-demo -it hengyunabc/arthas:latest /bin/sh -c "java -jar /opt/arthas/arthas-demo.jar"  `

# 3、启动arthas-boot来进行诊断
```bash
$ docker exec -it arthas-demo /bin/sh -c "java -jar /opt/arthas/arthas-boot.jar"
* [1]: 9 jar
 
[INFO] arthas home: /opt/arthas
[INFO] Try to attach process 9
[INFO] Attach process 9 success.
[INFO] arthas-client connect 127.0.0.1 3658
,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.
/  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'
 
wiki: https://alibaba.github.io/arthas
version: 3.0.5
pid: 9
time: 2018-12-18 11:30:36
```

# 4、诊断Docker里的Java进程
`  docker exec -it  ${containerId} /bin/bash -c "wget https://alibaba.github.io/arthas/arthas-boot.jar && java -jar arthas-boot.jar"   `

# 5、诊断k8s里容器里的Java进程
`  kubectl exec -it ${pod} --container ${containerId} -- /bin/bash -c "wget https://alibaba.github.io/arthas/arthas-boot.jar && java -jar arthas-boot.jar"   `

# 6、把Arthas安装到基础镜像里
可以很简单把Arthas安装到你的Docker镜像里。
```bash
FROM openjdk:8-jdk-alpine

# copy arthas
COPY --from=hengyunabc/arthas:latest /opt/arthas /opt/arthas
```
