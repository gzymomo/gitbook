# 静态Pod

由kubelet管理的仅存在于特定Node上的Pod，不能通过API Service进行管理，无法与RC、deployment或DaemonSet进行关联，并且kubelet也无法对他们进行健康检查，有kubelet创建并运行在kubelet所在的Node上运行。

静态Pod的yaml文件在修改之后，kubelet会进行自动重启该Pod至配置文件生效

创建静态Pod有两种方式：配置文件或者HTTP方式。

下面说一下配置文件的创建方式：

# 配置文件

需要设置kubelet启动参数“--config”，指定kubelet需要监控的配置文件所在的目录，kubelet会定期扫描该目录，并根据目录中的yaml或json文件进行创建操作

（1）如果集群是通过kubeadm创建的，那么已经配置好了静态pod的路径

查看kubelet的启动参数配置文件路径：

```bash
systemctl status kubelet
```

　　

![img](https://p6-tt-ipv6.byteimg.com/img/pgc-image/db0057f0e75a46fcaa4859fc8777995f~tplv-tt-shrink:640:0.image)

 

查看配置文件：

![img](https://p6-tt-ipv6.byteimg.com/img/pgc-image/98dd934f549c4a43adc9282724052112~tplv-tt-shrink:640:0.image)

 

启动参数配置在一个叫/var/lib/kubelet/config.yaml的文件中

在此文件中会发现由下图中的配置，也就是静态Pod路径配置为/etc/kubernetes/manifests路径

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/2aa0258034f54502846a6ab2e6441c66~tplv-tt-shrink:640:0.image)

 

所以只需要将静态Pod的yaml文件放置在此目录下即可。

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/539a24ef5ed04092960797c0ec067541~tplv-tt-shrink:640:0.image)

 

这四个Master上运行的核心组件就是通过此方式进行创建的。

例如上图中我将static-nginx.yaml放到/etc/kubernetes/manifests目录下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
  labels:
    name: static-nginx
spec:
  containers:
  - name: static-nginx
    image: nginx
    ports:
    - containerPort: 80
```

　　

此时使用kubelet get pods就可以查看到相应的Pod

![img](https://p6-tt-ipv6.byteimg.com/img/pgc-image/b4fd5cb6fd6e42f4b67728a55bc63e7c~tplv-tt-shrink:640:0.image)

 

（2）如果不是由kubeadm创建的集群，则需要在kubelet启动参数配置文件中添加如下一行：

```
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true
```

　　

修改配置之后需要重启kubelet

```bash
systemctl stop kubelet
systemctl daemon-reload
systemctl start kubelet
```

　　

例如我在cnode-2上配置了kubelet的启动参数，将静态Pod文件目录设置为/usr/soft/k8s/yaml/staticPod，然后重启kubelet

![img](https://p9-tt-ipv6.byteimg.com/img/pgc-image/9b4c3ffb4e5f46639c4c28d51d7a0a78~tplv-tt-shrink:640:0.image)

 

此时在目录下放置一个yaml文件

![img](https://p26-tt.byteimg.com/img/pgc-image/38f9166354464990a9f1d3c3f6f2dc49~tplv-tt-shrink:640:0.image)

 

保存后就可以查看到相应的Pod是否已创建

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/ef3fb357ab714e2fb839b9f377e49a50~tplv-tt-shrink:640:0.image)

 

【注意】如果Pod没创建成功，可以使用如下命令查看日志

```bash
systemctl status kubelet -l
```

　　

![img](https://p1-tt-ipv6.byteimg.com/img/pgc-image/35fb5da448564bf0b3d8b706d5167150~tplv-tt-shrink:640:0.image)

# Http方式

通过设置kubelet的启动参数“--manifest-url”，kubelet将会定期从该URL地址下载Pod的定义文件，并以.yaml或.json文件的格式进行解析， 然后创建Pod。其实现方式与配置文件方式是一致的。

【注意】静态Pod无法通过kubectl delete进行删除，只能删除对应的yaml文件