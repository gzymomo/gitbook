# Docker，Helm和Kubernetes的简化容器管理

Nexus Repository建立在广泛的企业存储功能之上，是适用于所有Docker镜像和Helm Chart存储库的强大的注册表。Nexus  Repository由广泛的用户社区支持，部署了超过500万个实例，支持全球1,200多家组织-600多家大型企业客户。团队可以选择使用Nexus Repository OSS高性能和完全免费的容器注册表，或者在需要企业可伸缩性和功能时选择Nexus Repository Pro。

与Docker Hub或Helm不同，开发团队将Nexus Repository作为所有公共注册表的中央访问点，从而为容器管理提供了更高效，更稳定的解决方案。除了在整个CI/CD构建管道中进行集成之外，使用完全支持的企业级容器注册表还具有许多好处。

# 多种存储库类型

Nexus存储库通过Proxy，Hosted和Group存储库支持Docker镜像和Helm 3存储库，从而使用户可以跨开发团队使用高级容器管理功能。

*代理存储库* -通过为Docker Hub或任何其他Docker镜像的远程注册表设置*代理存储库*，减少重复下载并提高开发人员和CI服务器的下载速度。在本地缓存图像，以加快上市时间并确保本地访问控制。

*托管存储库*-使用Nexus存储库将您自己的容器映像以及第三方映像上载到私有Docker注册表。这些注册表的细粒度权限为开发团队和组织提供了增强的安全性。

*存储库组*-允许用户从组中的所有存储库中提取映像，而无需在初始设置后进行任何其他客户端配置。组存储库使您可以使用工具的一个URL来访问多个代理和托管存储库的聚合内容。

# 创建Docker镜像仓库

创建一个Hosted类型的仓库，设置HTTP模式访问，端口为8090。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTO8DiaPficsIAgeianlfFibkuWocoZTKsmKLSNJ2fc5k3BqFSNy0fXviaxMEAicxWO8L6gd86mrHLtialFow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



更新Neuxs Service,添加8090端口。

```yaml
apiVersion: v1
kind: Service
metadata:
 name: nexus3
 namespace: devops
 labels:
   k8s-app: nexus3
spec:
 selector:
   k8s-app: nexus3
 ports:
 - name: web
   port: 8081
   targetPort: 8081
 - name: web2
   port: 8083
   targetPort: 8083
 - name: docker
   port: 8090
   targetPort: 8090
```

更新Neuxs Ingress，设置域名为`registry.idevops.site`

```yaml
- host: registry.idevops.site
    http:
     paths:
     - path: /
       backend:
          serviceName: nexus3
          servicePort: 8090
```

查看Nexus pod日志会发现已经启动了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTO8DiaPficsIAgeianlfFibkuWoaqwbgPrkVkecu2u6Haq1Ju44h8zcWrubT7vFeRWDDDkvmUGx1niajaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



登录镜像仓库

```bash
## 默认HTTPS会提示错误
[root@zeyang-nuc-service ~]# docker login registry.idevops.site
Username: admin
Password:
Error response from daemon: Get https://registry.idevops.site/v2/: dial tcp 192.168.1.230:443: connect: connection refused


## 更新docker配置
[root@zeyang-nuc-service ~]# vim /etc/docker/daemon.json
{
 "exec-opts":["native.cgroupdriver=systemd"],
 "registry-mirrors": ["https://c9ojlmr5.mirror.aliyuncs.com"],
 "insecure-registries" : ["192.168.1.200:8088","registry.idevops.site"]
}

[root@zeyang-nuc-service ~]# systemctl daemon-reload
[root@zeyang-nuc-service ~]# systemctl restart docker


## 再次登录
[root@zeyang-nuc-service ~]# docker login registry.idevops.site
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

测试上传镜像

```bash
[root@zeyang-nuc-service ~]# docker tag mysql:5.7 registry.idevops.site/library/mysql:5.7
[root@zeyang-nuc-service ~]# docker push registry.idevops.site/library/mysql:5.7
The push refers to repository [registry.idevops.site/library/mysql]
c187f0dccfe2: Pushed
a45abaac81d1: Pushed
71c5f5690aef: Pushed
8df989cb6670: Pushed
f358b00d8ce7: Pushed
ae39983d39c4: Pushed
b55e8d7c5659: Pushed
e8fd11b2289c: Pushed
e9affce9cbe8: Pushed
316393412e04: Pushed
d0f104dc0a1f: Pushed
5.7: digest: sha256:55638620c5a206833217dff4685e0715fb297a8458aa07c5fe5d8730cc6c872f size: 2621
```

在nexus中验证.

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTO8DiaPficsIAgeianlfFibkuWoyuwCgRPT2Qia95iaX7Htqibc8UaBth4QIeALWGoVYqwzpN7LqjnctRShA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Nexus作为容器注册表，通过用于容器存储管理和K8s部署的Docker和Helm注册表为企业提供动力。随着DevOps团队规模的扩大，至关重要的是要依靠有关应用程序中开源组件质量的精确报告。Nexus Lifecycle向开发人员和安全专家提供有关安全漏洞，许可风险和体系结构质量的开源组件智能。寻求完全集成的通用容器管理注册表以及最精确的组件智能的组织，可以使用Nexus平台来满足不断增长的容器化和开源治理的需求。