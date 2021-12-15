- [jenkins发布k8s应用](https://blog.51cto.com/luoguoling/2952077)



准备：

1.k8s harbor git已经提前搭建好。

2.jdk jenkins都是通过yum安装,且k8s和jenkins在同一个机器上

开始构建

1.配置参数化构建

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202106/28/ade0732c6c2b443deb668e7e1fb79301.jpg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

2.构建环境

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202106/28/f4b618f003b3c235a12e7d6a3774a599.jpg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

3.脚本用法

 

```bash
#!/bin/bash
URL=http://gitlab.aaa.com/system_admin/h5game.xxxx.com.git
Starttime=`date +"%Y-%m-%d_%H-%M-%S"`
Method=$1
Version=$2
t1=`date +"%Y-%m-%d %H:%M:%S"`
#代码克隆至jenkins后端
clone_code(){
    cd /var/lib/jenkins/workspace && git clone  ${URL}&& echo "Clone Finished"
}
#代码打包压缩并远程推送至k8s-master-1的nginx镜像制作目录
Pack_scp(){
    cd /var/lib/jenkins/workspace/h5game.xxxx.com && tar cvzf h5game.xxxx.com.tar.gz * && mv h5game.xxxx.com.tar.gz /root/docker/fronted/www.h5sdk.xxxx.com/ && echo Package Finished
}
#远程操作k8s-master-1节点，进行镜像制作并推送至harbor镜像仓库
build_iamge(){
    cd /root/docker/fronted/www.h5sdk.xxxx.com/ && docker build -t 10.206.16.4/fronted/www.h5sdk.xxxx.com:${BUILD_NUMBER} .
    docker push 10.206.16.4/fronted/www.h5sdk.xxxx.com:${BUILD_NUMBER}
    echo 'build_image and push_harbor success!'
}
#对k8s集群中的nginx的pod应用进行升级
app_update(){
    sed -ri 's@image: .*@image: 10.206.16.4/fronted/www.h5sdk.xxxx.com:${BUILD_NUMBER}@g'  /root/fronted/www.h5sdk.xxxx.com/deployment.yaml
    kubectl set image deployment/h5sdk nginx=10.206.16.4/fronted/www.h5sdk.xxxx.com:${BUILD_NUMBER} -n fronted --record=true
                t2=`date +"%Y-%m-%d %H:%M:%S"`
    start_T=`date --date="${t1}" +%s`
    end_T=`date --date="${t2}" +%s`
    total_time=$((end_T-start_T))
    echo "deploy success,it has been spent ${total_time} seconds"   
}
#k8s集群中的pod应用进行回滚
app_rollback(){
  #根据build_number获取对应的k8s回滚版本号revision
    revision=`kubectl rollout history deployment/h5sdk -n fronted|grep $Version|awk -F " " {'print $1'}`
    kubectl rollout undo deployment/h5sdk  -n fronted --to-revision=$revision
}
#进行k8s集群自动部署的主函数
main(){
    case $Method in
    deploy)
        clone_code
        Pack_scp
        build_iamge
        app_update
    ;;
    rollback)
        app_rollback
    ;;
    esac
}
#执行主函数命令
main $1 $2
```

4.具体操作，可以直接部署，根据build_number进行回滚

![watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=](https://s4.51cto.com/images/blog/202106/28/fc3e05e03032551eae9301862226dc43.jpg?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 