# 一、Label

Label是附着到object上（例如Pod）的键值对。可以在创建object的时候指定，也可以在object创建后随时指定。Labels的值对系统本身并没有什么含义，只是对用户才有意义。在一个资源中，key是唯一的。

## 1.1 标签的查看

- 显示pod的所有标签

```bash
[root@hdss7-21 ~]# kubectl get pods -n app --show-labels
NAME     READY   STATUS    RESTARTS   AGE     LABELS
pod-01   1/1     Running   0          95s     app=nginx,release=connary,tier=frontend,version=v1.12
pod-02   1/1     Running   2          2d      app=nginx,release=stable,version=v1.12
pod-03   1/1     Running   0          4m29s   app=nginx,release=alpha,version=v1.12
pod-04   1/1     Running   0          3m5s    app=nginx,release=stable,version=v1.13
```

- 显示包含tier标签的pod

```bash
[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l tier
NAME     READY   STATUS    RESTARTS   AGE     LABELS
pod-01   1/1     Running   0          2m42s   app=nginx,release=connary,tier=frontend,version=v1.12
```

- 显示v1.13稳定版本的pod

```bash
[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l release=stable,version=v1.13
NAME     READY   STATUS    RESTARTS   AGE     LABELS
pod-04   1/1     Running   0          4m52s   app=nginx,release=stable,version=v1.13
```

## 1.2 常用的标签字段

| Key         | Values                   | 说明                                             |
| ----------- | ------------------------ | ------------------------------------------------ |
| release     | stable,canary,alpha,bate | 版本。常用的是稳定版、金丝雀版本、内测版、公测版 |
| environment | dev,qa,gray,production   | 环境。开发环境、测试环境、灰度环境、生产环境     |
| tier        | frontend,backend,cache   | 层级。前端、后端、缓存                           |
| partition   | customerA,customerB      | 模块。业务A，业务B                               |
| version     | v1.1,v1.2                | 版本号。v1.1，v1.2                               |

## 1.3 标签的语法

**Key的语法规则：**

- Key可以有两部分组成：prefix和name，使用 / 连接prefix和name。其中prefix必须为集群内dns能解析的域，不得超过253个字符；name长度不超过63的字符。kubernetes.io 、k8s.io 为kubernetes集群组件保留的prefix。大部分资源很少使用prefix，一般仅由name作为key。
- name必须由字母或数字开头和结尾，中间可以用 中划线、下划线和点连接

**Value的语法规则：**

- 长度不超过63个字符
- 必须由字母或数字开头和结尾，中间可以用 中划线、下划线和点连接

# 二、 Annotations

Label和Annotation都可以将元数据关联到Kubernetes资源对象。Label主要用于选择对象，可以挑选出满足特定条件的对象。相比之下，annotation 不能用于标识及选择对象。annotation中的元数据可多可少，可以是结构化的或非结构化的，也可以包含label中不允许出现的字符。annotation和label一样都是key/value键值对映射结构。

声明配置层管理的字段。使用annotation关联这类字段可以用于区分以下几种配置来源：

- 客户端或服务器设置的默认值，自动生成的字段或自动生成的 auto-scaling 和 auto-sizing 系统配置的字段。
- 创建信息、版本信息或镜像信息。例如时间戳、版本号、git分支、PR序号、镜像哈希值以及仓库地址。
- 记录日志、监控、分析或审计存储仓库的指针
- 可以用于debug的客户端（库或工具）信息，例如名称、版本和创建信息。
- 用户信息，以及工具或系统来源信息、例如来自非Kubernetes生态的相关对象的URL信息。
- 轻量级部署工具元数据，例如配置或检查点。
- 负责人的电话或联系方式，或能找到相关信息的目录条目信息，例如团队网站。

```bash
[root@hdss7-21 ~]# kubectl get node hdss7-21.host.com -o yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
```



# 三、选择器

## 3.1 标签选择器

Label不是唯一的，很多object可能有相同的label。通过label selector，客户端／用户可以指定一个object集合，通过label selector对object的集合进行操作。**多个条件使用“,”连接，表示AND的含义。**

#### 3.1.1. 标签选择器类型

- 基于等值的选择器

| Operators  | Meanings                         |
| ---------- | -------------------------------- |
| Key1=var1  | 存在key1，且key1的值为var1       |
| Key2==var2 | 存在key2，且key2的值为var2       |
| Key3!=var3 | 不存在key3，或者key3的值不为var3 |

- 基于集合的选择器

| Operators                | Meanings                                   |
| ------------------------ | ------------------------------------------ |
| Key1   in (var1,var2)    | 存在key1，且key1的值为var1或者var2         |
| Key2   notin (var1,var2) | 不存在key2，或者key2的值不为var1且不为var2 |
| Key3                     | 存在key3                                   |
| ! key4                   | 不存在key4                                 |

#### 3.1.2. 标签选择器的使用

```bash
[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l tier # 包含tier的pod
NAME     READY   STATUS    RESTARTS   AGE   LABELS
pod-01   1/1     Running   0          45m   app=nginx,release=connary,tier=frontend,version=v1.12
[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l "! tier" # 不包含tier的pod
NAME     READY   STATUS    RESTARTS   AGE    LABELS
pod-02   1/1     Running   2          2d1h   app=nginx,release=stable,version=v1.12
pod-03   1/1     Running   0          48m    app=nginx,release=alpha,version=v1.12
pod-04   1/1     Running   0          47m    app=nginx,release=stable,version=v1.13

[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l "release in (alpha,connary)" # 版本为alpha和connary的pod
NAME     READY   STATUS    RESTARTS   AGE   LABELS
pod-01   1/1     Running   0          47m   app=nginx,release=connary,tier=frontend,version=v1.12
pod-03   1/1     Running   0          50m   app=nginx,release=alpha,version=v1.12

[root@hdss7-21 ~]# kubectl get pods -n app --show-labels -l "tier notin (backend,frontend)" # 不是前后端的pod
NAME     READY   STATUS    RESTARTS   AGE    LABELS
pod-02   1/1     Running   2          2d1h   app=nginx,release=stable,version=v1.12
pod-03   1/1     Running   0          52m    app=nginx,release=alpha,version=v1.12
pod-04   1/1     Running   0          51m    app=nginx,release=stable,version=v1.13
```

## 3.2 字段选择器

字段选择器是基于字段来筛选资源的，平时用的很少，且支持的字段类型很少，目前仅支持：metadata.name, metadata.namespace, status.phase。操作符支持 =, ==, != 三种。

```bash
[root@hdss7-21 ~]# kubectl get pod -A --field-selector metadata.namespace!=app,status.phase=Running
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
default       nginx-ds-7cs4q                          0/1     ImagePullBackOff   2          7d17h
default       nginx-ds-jdp7q                          0/1     ImagePullBackOff   2          7d17h
kube-system   coredns-6b6c4f9648-nmc9v                1/1     Running            1          18h
kube-system   kubernetes-dashboard-76dcdb4677-z46pd   1/1     Running            3          6d4h
kube-system   traefik-ingress-h5pvj                   1/1     Running            3          6d5h
kube-system   traefik-ingress-vtlch                   1/1     Running            3          6d5h
```