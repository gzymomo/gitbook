# [日志聚合工具loki](https://www.cnblogs.com/ssgeek/p/11584870.html)

## 1、loki是什么

[Loki](https://github.com/grafana/loki)是一个水平可扩展，高可用性，多租户的日志聚合系统，受到Prometheus的启发。它的设计非常经济高效且易于操作，因为它不会为日志内容编制索引，而是为每个日志流编制一组标签。官方介绍说到：Like Prometheus, but for logs.

## 2、loki特点

与其他日志聚合系统相比，`Loki`：

- 不对日志进行全文索引。通过存储压缩的非结构化日志和仅索引元数据，`Loki`操作更简单，运行更便宜。
- 索引和组使用与`Prometheus`已使用的相同标签记录流，使您可以使用与`Prometheus`已使用的相同标签在指标和日志之间无缝切换。
- 特别适合存放`Kubernetes Pod`日志; 诸如`Pod`标签之类的元数据会被自动删除和编入索引。
- 在`Grafana`有本机支持（已经包含在Grafana 6.0或更新版本中）。

## 3、loki组成

Loki由3个组成部分组成：

- loki 是主服务器，负责存储日志和处理查询。
- promtail 是代理，负责收集日志并将其发送给loki。
- 用户界面的Grafana。

## 4、loki安装

`loki`的安装方式包含如下：使用官方的`docker`镜像单独运行、使用`helm`工具在`kubernetes`上安装、使用源码构建。
本文采用的方法是基于`kubernetes`环境使用`helm`安装。所以前提是需要有一个`kubernetes`环境并且安装好了`helm`。

```
[root@k8s-qa-master-01 ~]# helm version  
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
[root@k8s-qa-master-01 ~]# helm repo list
NAME    URL                                                   
stable  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
local   http://127.0.0.1:8879/charts
```

### 4.1、添加helm的chart库

```
[root@k8s-qa-master-01 ~]# helm repo add loki https://grafana.github.io/loki/charts
"loki" has been added to your repositories
[root@k8s-qa-master-01 ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "loki" chart repository
Update Complete.
[root@k8s-qa-master-01 ~]# helm repo list
NAME    URL                                                   
stable  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
local   http://127.0.0.1:8879/charts                          
loki    https://grafana.github.io/loki/charts
```

### 4.2、安装loki及promtail

分为以下多种安装方式：

- 默认安装

```
helm upgrade --install loki loki/loki-stack
```

- 自定义`namespaces`安装

```
helm upgrade --install loki --namespace=loki-stack loki/loki-stack
```

- 选择安装，只安装`loki`或只安装`promtail`

```
helm upgrade --install loki loki/loki
helm upgrade --install promtail loki/promtail --set "loki.serviceName=loki"
```

- 自定义配置安装

方法一：在`helm`命令中使用`--set`参数覆盖默认`chart`中的配置或者是`chart`中子`chart`的配置。

```
helm upgrade --install loki loki/loki-stack --set "key1=val1,key2=val2,..."
```

方法二：只下载相应的`chart`包，下载后修改默认配置再安装。默认下载的`chart`包在~/.helm/cache/archive目录下，例如修改默认promtail收集日志的目录

```
[root@k8s-qa-master-01 archive]# pwd
/root/.helm/cache/archive
[root@k8s-qa-master-01 archive]# ls
loki-stack-0.16.3.tgz
[root@k8s-qa-master-01 archive]# tar xf loki-stack-0.16.3.tgz 
[root@k8s-qa-master-01 archive]# vim loki-stack/charts/promtail/values.yaml
···
# Extra volumes to scrape logs from
volumes:
- name: docker
  hostPath:
    path: /data/docker/containers
- name: pods
  hostPath:
    path: /var/log/pods

volumeMounts:
- name: docker
  mountPath: /data/docker/containers
  readOnly: false
- name: pods
  mountPath: /var/log/pods
  readOnly: false
···
```

修改完配置后进行安装

```
[root@k8s-qa-master-01 archive]# helm install ./loki-stack --name=loki --namespace=kube-system
NAME:   loki
LAST DEPLOYED: Wed Sep 25 14:29:07 2019
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                       AGE
loki-promtail-clusterrole  42s

==> v1/ClusterRoleBinding
NAME                              AGE
loki-promtail-clusterrolebinding  42s

==> v1/ConfigMap
NAME                  DATA  AGE
loki-loki-stack-test  1     44s
loki-promtail         1     44s

==> v1/DaemonSet
NAME           DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
loki-promtail  5        5        0      5           0          <none>         39s

==> v1/Pod(related)
NAME                 READY  STATUS             RESTARTS  AGE
loki-0               0/1    Running            0         37s
loki-promtail-2lqzz  0/1    Running            0         36s
loki-promtail-8rpdj  0/1    ContainerCreating  0         37s
loki-promtail-h4lrm  0/1    Running            0         37s
loki-promtail-mbjws  0/1    Running            0         37s
loki-promtail-nj7k4  0/1    Running            0         36s

==> v1/Role
NAME           AGE
loki           41s
loki-promtail  41s

==> v1/RoleBinding
NAME           AGE
loki           40s
loki-promtail  40s

==> v1/Secret
NAME  TYPE    DATA  AGE
loki  Opaque  1     44s

==> v1/Service
NAME           TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
loki           ClusterIP  10.68.3.79  <none>       3100/TCP  39s
loki-headless  ClusterIP  None        <none>       3100/TCP  40s

==> v1/ServiceAccount
NAME           SECRETS  AGE
loki           1        43s
loki-promtail  1        42s

==> v1/StatefulSet
NAME  READY  AGE
loki  0/1    39s

==> v1beta1/PodSecurityPolicy
NAME           PRIV   CAPS      SELINUX           RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
loki           false  RunAsAny  MustRunAsNonRoot  MustRunAs  MustRunAs  true      configMap,emptyDir,persistentVolumeClaim,secret
loki-promtail  false  RunAsAny  RunAsAny          RunAsAny   RunAsAny   true      secret,configMap,hostPath


NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.

See http://docs.grafana.org/features/datasources/loki/ for more detail.
[root@k8s-qa-master-01 archive]# kubectl get pods -n kube-system|grep loki
loki-0                                   1/1     Running   0          2m11s
loki-promtail-2lqzz                      1/1     Running   0          2m10s
loki-promtail-8rpdj                      1/1     Running   0          2m11s
loki-promtail-h4lrm                      1/1     Running   0          2m11s
loki-promtail-mbjws                      1/1     Running   0          2m11s
loki-promtail-nj7k4                      1/1     Running   0          2m10s
```

### 4.3、安装grafana

本步骤的前提是还没有安装`grafana`，如果在安装`loki`前已经安装好，此步骤可忽略
安装

```
helm install stable/grafana -n loki-grafana
```

获取`grafana`密码

```
kubectl get secret --namespace <YOUR-NAMESPACE> loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

访问`grafana`，开启端口转发

```
kubectl port-forward --namespace <YOUR-NAMESPACE> service/loki-grafana 3000:80
```

如果将Loki和Promtail部署在不同的群集上，则可以在Loki前面添加一个Ingress。通过添加证书，您可以创建一个https端点。为了提高安全性，请在Ingress上启用基本身份验证,可参考[地址](https://github.com/grafana/loki/tree/master/production/helm#run-loki-behind-https-ingress)

## 5、配置和使用

登录到`gfafana`,配置`loki`的数据源
![img](http://image.ssgeek.com/20190925-02.png)
切换到`grafana`左侧区域的`Explore`，即可进入到`loki`的页面
![img](http://image.ssgeek.com/20190925-03.png)
点击`Log labels`就可以把当前系统采集的日志标签给显示出来，可以根据这些标签进行日志的过滤查询,也可直接输入过滤表达式，如图所示，过滤出`container`名称为`jenkins`的日志
![img](http://image.ssgeek.com/20190925-04.png)

## 6、日志选择和过滤

### 6.1、日志选择器

对于查询表达式的标签部分，将其用大括号括起来{}，然后使用键值语法选择标签。多个标签表达式用逗号分隔：

```
{app="mysql",name="mysql-backup"}
```

当前支持以下标签匹配运算符：

- = 完全相等。
- != 不相等。
- =~ 正则表达式匹配。
- !~ 不进行正则表达式匹配。

例子：

```
{name=~"mysql.+"}
{name!~"mysql.+"}
```

### 6.2、日志过滤器

编写日志流选择器后，您可以通过编写搜索表达式来进一步过滤结果。搜索表达式可以只是文本或正则表达式。
查询示例：

```
{job="mysql"} |= "error"
{name="kafka"} |~ "tsdb-ops.*io:2003"
{instance=~"kafka-[23]",name="kafka"} != kafka.server:type=ReplicaManager
```

过滤器运算符可以被链接，并将顺序过滤表达式-结果日志行将满足每个过滤器。例如：

```repl
{job="mysql"} |= "error" != "timeout"
```

已实现以下过滤器类型：

- |= 行包含字符串。
- != 行不包含字符串。
- |~ 行匹配正则表达式。
- !~ 行与正则表达式不匹配。
  regex表达式接受RE2语法。默认情况下，匹配项区分大小写，并且可以将regex切换为不区分大小写的前缀(?i)。

更多内容可参考[官方说明](https://github.com/grafana/loki/blob/master/docs/querying.md#grafana)