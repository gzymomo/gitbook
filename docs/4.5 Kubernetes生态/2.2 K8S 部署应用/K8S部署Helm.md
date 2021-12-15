- [容器编排系统K8s之包管理器Helm基础使用](https://www.cnblogs.com/qiuhom-1874/p/14305902.html)



# Helm是什么？

没有使用Helm之前，在Kubernetes部署应用，我们要依次部署deployment、service等，步骤比较繁琐。况且随着很多项目微服务化，复杂的应用在容器中部署以及管理显得较为复杂。

<font color='red'>helm通过打包的方式，支持发布的版本管理和控制，很大程度上简化了Kubernetes应用的部署和管理。</font>

Helm本质就是让k8s的应用管理（Deployment、Service等）可配置，能动态生成。通过动态生成K8S资源清单文件（deployment.yaml、service.yaml）。然后kubectl自动调用K8S资源部署。

<font color='red'>Helm是官方提供类似于YUM的包管理，是部署环境的流程封装，Helm有三个重要的概念：chart、release和Repository</font>

- chart是创建一个应用的信息集合，包括各种Kubernetes对象的配置模板、参数定义、依赖关系、文档说明等。可以将chart想象成apt、yum中的软件安装包。
- release是chart的运行实例，代表一个正在运行的应用。当chart被安装到Kubernetes集群，就生成一个release。chart能多次安装到同一个集群，每次安装都是一个release【根据chart赋值不同，完全可以部署出多个release出来】。
- Repository用于发布和存储 Chart 的存储库。



# Helm部署

helm的GitHub地址

```
https://github.com/helm/helm
```





