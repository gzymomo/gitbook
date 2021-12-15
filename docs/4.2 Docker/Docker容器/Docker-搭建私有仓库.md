[TOC]

# 1、拉取私有仓库的docker 镜像
` docker pull registry `

# 2、运行这个镜像
` docker run -d -p 5000:5000 -v /var/project/docker/registry/:/var/lib/registry --restart always --name registry registry:latest `

# 3、推送镜像
如果是现有的先改一个名字
比如 的hello
` docker  tag hello 127.0.0.1/hello `

如果是重新打就是
` docker build -t 127.0.0.1/xxx . `

打好镜像之后,就可以push了。
` docker push 127.0.0.1:5000/hello `

# 4、查看私有仓库中的镜像
curl ip:5000/v2/_catalog