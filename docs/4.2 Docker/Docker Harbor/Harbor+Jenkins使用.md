将Jenkins和Harbor 相互结合，下图是比较理想的状态，当然还缺一下管理工具等等

![img](https://img2018.cnblogs.com/blog/1339436/201901/1339436-20190124174901450-932377594.png)



java代码构建之后进行的操作

脚本内容：（其实参考这个博主方法，觉得有些麻烦，可以直接利用SpringBoot+IDEA+Maven+docker maven plugin插件打包镜像），

然后直接构建和push镜像。



```bash
#!/bin/bash
#获取镜像id
imagesid=`docker images|grep -i docker-harbor|awk '{print $3}'`
project=/harbor_repo/
#dockerid=`docker ps -a|grep -i docker-test|awk '{print $1}' `
echo $project
#判断镜像是否存在如果存在则删除，否则不删除
if  [ ! -n "$imagesid" ];then
   echo $imagesid "is null"
else
    docker rmi -f $imagesid 
fi
#进入工作目录
cd $project
#生成新的镜像

docker build -t docker-harbor .

#登录docker仓库 
docker login -u admin -p Harbor12345 192.168.10.110

#上传镜像到镜像仓库

docker tag  docker-harbor 192.168.10.110/my_data/docker-harbor:1

docker push 192.168.10.110/my_data/docker-harbor:1
```

