- [Helm v3](https://www.cnblogs.com/yuezhimi/p/12981950.html)
- [Kubernets包管理工具—>Helm介绍与使用](https://www.cnblogs.com/v-fan/p/13949025.html)



# 为什么需要Helm？

K8S上的应用对象，都是由特定的资源描述组成，包括deployment、service等。都保存各自文件中或者集中写到一个配置文件。然后kubectl apply –f 部署。

如果应用只由一个或几个这样的服务组成，上面部署方式足够了。

而对于一个复杂的应用，会有很多类似上面的资源描述文件，例如微服务架构应用，组成应用的服务可能多达十个，几十个。如果有更新或回滚应用的需求，可能要修改和维护所涉及的大量资源文件，而这种组织和管理应用的方式就显得力不从心了。

且由于缺少对发布过的应用版本管理和控制，使Kubernetes上的应用维护和更新等面临诸多的挑战，主要面临以下问题：

1. **如何将这些服务作为一个整体管理**
2. **这些资源文件如何高效复用**
3. **不支持应用级别的版本管理**



# 一、Helm 介绍

Helm是一个Kubernetes的包管理工具，就像Linux下的包管理器，如yum/apt等，可以很方便的将之前打包好的yaml文件部署到kubernetes上。

Helm有3个重要概念：

- **helm：**一个命令行客户端工具，主要用于Kubernetes应用chart的创建、打包、发布和管理。一个helm程序包，是创建一个应用的信息集合，包含各种Kubernetes对象的配置模板、参数定义、依赖关系、文档说明等。可以将Chart比喻为yum中的软件安装包；
- **Repository**：Charts仓库，用于集中存储和分发Charts；
- **Chart：**应用描述，一系列用于描述 k8s 资源相关文件的集合。
- **Config**：应用程序实例化安装运行时所需要的配置信息；
- **Release：**基于Chart的部署实体，一个 chart 被 Helm 运行后将会生成对应的一个 release；将在k8s中创建出真实运行的资源对象。



# 二、Helm客户端

## 2.1 部署helm客户端

Helm客户端下载地址：https://github.com/helm/helm/releases

解压移动到/usr/bin/目录即可。

```bash
wget https://get.helm.sh/helm-vv3.2.1-linux-amd64.tar.gz
tar zxvf helm-v3.2.1-linux-amd64.tar.gz 
mv linux-amd64/helm /usr/bin/
```

helm常用命令

| 命令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| create     | 创建一个chart并指定名字                                      |
| dependency | 管理chart依赖                                                |
| get        | 下载一个release。可用子命令：all、hooks、manifest、notes、values |
| history    | 获取release历史                                              |
| install    | 安装一个chart                                                |
| list       | 列出release                                                  |
| package    | 将chart目录打包到chart存档文件中                             |
| pull       | 从远程仓库中下载chart并解压到本地 # helm pull stable/mysql --untar |
| repo       | 添加，列出，移除，更新和索引chart仓库。可用子命令：add、index、list、remove、update |
| rollback   | 从之前版本回滚                                               |
| search     | 根据关键字搜索chart。可用子命令：hub、repo                   |
| show       | 查看chart详细信息。可用子命令：all、chart、readme、values    |
| status     | 显示已命名版本的状态                                         |
| template   | 本地呈现模板                                                 |
| uninstall  | 卸载一个release                                              |
| upgrade    | 更新一个release                                              |
| version    | 查看helm客户端版本                                           |

## 2.2 配置国内chart仓库

- 微软仓库（http://mirror.azure.cn/kubernetes/charts/）这个仓库推荐，基本上官网有的chart这里都有。
- 阿里云仓库（https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts ）
- 官方仓库（https://hub.kubeapps.com/charts/incubator）官方chart仓库，国内有点不好使。

添加存储库

```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
helm repo update
```

查看配置的存储库

```bash
helm repo list
helm search repo stable
```

删除存储库：

```bash
helm repo remove aliyun
```

## 2.3 helm基本使用

主要介绍三个命令：

- chart install
- chart upgrade
- chart rollback

### 1. 使用chart部署一个应用

```bash
#查找chart
# helm search repo weave
NAME                  CHART VERSION    APP VERSION    DESCRIPTION                                       
aliyun/weave-cloud    0.1.2                           Weave Cloud is a add-on to Kubernetes which pro...
aliyun/weave-scope    0.9.2            1.6.5          A Helm chart for the Weave Scope cluster visual...
stable/weave-cloud    0.3.7            1.4.0          Weave Cloud is a add-on to Kubernetes which pro...
stable/weave-scope    1.1.10           1.12.0         A Helm chart for the Weave Scope cluster visual...

#查看chrt信息
# helm show chart stable/mysql

#安装包
# helm install ui stable/weave-scope

#查看发布状态
# helm list
NAME    NAMESPACE    REVISION    UPDATED                                    STATUS      CHART                 APP VERSION
ui      default      1           2020-05-28 17:45:01.696109626 +0800 CST    deployed    weave-scope-1.1.10    1.12.0     
[root@k8s-master ~]# helm status ui
NAME: ui
LAST DEPLOYED: Thu May 28 17:45:01 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
You should now be able to access the Scope frontend in your web browser, by
using kubectl port-forward:

kubectl -n default port-forward $(kubectl -n default get endpoints \
ui-weave-scope -o jsonpath='{.subsets[0].addresses[0].targetRef.name}') 8080:4040

then browsing to http://localhost:8080/.
For more details on using Weave Scope, see the Weave Scope documentation:

https://www.weave.works/docs/scope/latest/introducing/#修改service Type: NodePort 即可访问ui
```



### 2. 安装前自定义chart配置选项

自定义选项是因为并不是所有的chart都能按照默认配置运行成功，可能会需要一些环境依赖，例如PV。

所以我们需要自定义chart配置选项，安装过程中有两种方法可以传递配置数据：

- --values（或-f）：指定带有覆盖的YAML文件。这可以多次指定，最右边的文件优先
- --set：在命令行上指定替代。如果两者都用，--set优先级高

**--values使用，先将修改的变量写到一个文件中**



```bash
# helm show values stable/mysql
# cat config.yaml 
persistence:
  enabled: true
  storageClass: "managed-nfs-storage"
  accessMode: ReadWriteOnce
  size: 8Gi
mysqlUser: "k8s"
mysqlPassword: "123456"
mysqlDatabase: "k8s"
# helm install db -f config.yaml stable/mysql
# kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
db-mysql-57485b68dc-4xjhv                 1/1     Running   0          8m51s

# kubectl run -it db-client --rm --restart=Never --image=mysql:5.7 -- bash
If you don't see a command prompt, try pressing enter.
root@db-client:/# mysql -hdb-mysql -uk8s -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 36
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| k8s                |
+--------------------+
```



以上将创建具有名称的默认MySQL用户k8s，并授予此用户访问新创建的k8s数据库的权限，但将接受该图表的所有其余默认值。

**命令行替代变量：**

```
# helm install db --set persistence.storageClass="managed-nfs-storage" stable/mysql
```

也可以把chart包下载下来查看详情：

```
# helm pull stable/mysql --untar
```

values yaml与set使用：

 ![img](https://img2020.cnblogs.com/blog/1156961/202005/1156961-20200529134739515-981521937.png)

**该helm install命令可以从多个来源安装：**

- chart存储库
- 本地chart存档（helm install foo-0.1.1.tgz）
- chart目录（helm install path/to/foo）
- 完整的URL（helm install https://example.com/charts/foo-1.2.3.tgz）

# 三、构建一个Helm Chart



```bash
# helm create mychart
Creating mychart
# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml
```



- Chart.yaml：用于描述这个 Chart的基本信息，包括名字、描述信息以及版本等。
- values.yaml ：用于存储 templates 目录中模板文件中用到变量的值。
- Templates： 目录里面存放所有yaml模板文件。
- charts：目录里存放这个chart依赖的所有子chart。
- NOTES.txt ：用于介绍Chart帮助信息， helm install 部署后展示给用户。例如：如何使用这个 Chart、列出缺省的设置等。
- _helpers.tpl：放置模板助手的地方，可以在整个 chart 中重复使用

创建Chart后，接下来就是将其部署：

```
helm install web mychart/
```

也可以打包推送的charts仓库共享别人使用。

```
# helm package mychart/
mychart-0.1.0.tgz
```

## 3.1 chart模板

Helm最核心的就是模板，即模板化的K8S manifests文件。

它本质上就是一个Go的template模板。Helm在Go template模板的基础上，还会增加很多东西。如一些自定义的元数据信息、扩展的库以及一些类似于编程形式的工作流，例如条件语句、管道等等。这些东西都会使得我们的模板变得更加丰富。

有了模板，我们怎么把我们的配置融入进去呢？用的就是这个values文件。这两部分内容其实就是chart的核心功能。

接下来，部署nginx应用，熟悉模板使用



```bash
# helm create nginx
# vim nginx/Chart.yaml 
apiVersion: v2
name: nginx
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: 1.15

# vim nginx/values.yaml
replicas: 3
image: nginx
tag: 1.15
serviceport: 80
targetport: 80
label: nginx

# vim nginx/templates/NOTES.txt 
hello

# vim nginx/templates/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: {{ .Values.label }}
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.label }}
  template:
    metadata:
      labels:
        app: {{ .Values.label }}
    spec:
      containers:
      - image: {{ .Values.image }}:{{ .Values.tag }}
        name: web

# vim nginx/templates/service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app: {{ .Values.label }}
  name: {{ .Release.Name }}
spec:
  ports:
  - port: {{ .Values.serviceport }}
    protocol: TCP
    targetPort: {{ .Values.targetport }}
  selector:
    app: {{ .Values.label }}
  type: NodePort

#查看实际的模板被渲染过后的资源文件
# helm get manifest web
# helm install web nginx/
NAME: web
LAST DEPLOYED: Fri May 29 16:09:46 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
hello
# helm list
NAME    NAMESPACE    REVISION    UPDATED                                    STATUS      CHART          APP VERSION
web     default      1           2020-05-29 16:09:46.608457282 +0800 CST    deployed    nginx-0.1.0    1.15
# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
web-5675686b8-7wtqk                       1/1     Running   0          25s
web-5675686b8-f72hk                       1/1     Running   0          25s
web-5675686b8-k4kqr                       1/1     Running   0          25s
```



这个deployment就是一个Go template的模板，这里定义的Release模板对象属于Helm内置的一种对象，是从values文件中读取出来的。这样一来，我们可以将需要变化的地方都定义变量。

## 3.2 调试

Helm也提供了`--dry-run --debug`调试参数，帮助你验证模板正确性。在执行`helm install`时候带上这两个参数就可以把对应的values值和渲染的资源清单打印出来，而不会真正的去部署一个release。

比如我们来调试上面创建的 chart 包：

```
# helm install web --dry-run nginx/
```

## 3.3 内置对象

刚刚我们使用 `{{.Release.Name}}`将 release 的名称插入到模板中。这里的 Release 就是 Helm 的内置对象，下面是一些常用的内置对象：

| Release.Name      | release 名称                    |
| ----------------- | ------------------------------- |
| Release.Name      | release 名字                    |
| Release.Namespace | release 命名空间                |
| Release.Service   | release 服务的名称              |
| Release.Revision  | release 修订版本号，从1开始累加 |

## 2.4 Values

Values对象是为Chart模板提供值，这个对象的值有4个来源：

- chart 包中的 values.yaml 文件
- 父 chart 包的 values.yaml 文件
- 通过 helm install 或者 helm upgrade 的 `-f`或者 `--values`参数传入的自定义的 yaml 文件
- 通过 `--set` 参数传入的值

chart 的 values.yaml 提供的值可以被用户提供的 values 文件覆盖，而该文件同样可以被 `--set`提供的参数所覆盖。



```bash
# helm upgrade web --set replicas=5 nginx/
Release "web" has been upgraded. Happy Helming!
NAME: web
LAST DEPLOYED: Fri May 29 16:34:17 2020
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
hello

# helm history web
REVISION    UPDATED                     STATUS        CHART          APP VERSION    DESCRIPTION     
1           Fri May 29 16:33:56 2020    superseded    nginx-0.1.0    1.15           Install complete
2           Fri May 29 16:34:17 2020    deployed      nginx-0.1.0    1.15           Upgrade complete

# kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
web-5675686b8-7n7bg                       1/1     Running   0          54s
web-5675686b8-9vf28                       1/1     Running   0          33s
web-5675686b8-9wkgz                       1/1     Running   0          54s
web-5675686b8-jdrhr                       1/1     Running   0          54s
web-5675686b8-rrrxc                       1/1     Running   0          33s
```



## 2.5 升级、回滚和删除

发布新版本的chart时，或者当您要更改发布的配置时，可以使用该`helm upgrade` 命令。

```
# helm upgrade --set imageTag=1.17 web nginx
# helm upgrade -f values.yaml web nginx
```

如果在发布后没有达到预期的效果，则可以使用`helm rollback`回滚到之前的版本。

例如将应用回滚到第一个版本：

```
# helm rollback web 1
```

卸载发行版，请使用以下`helm uninstall`命令：

```
# helm uninstall web
```

查看历史版本配置信息

```
# helm get all --revision 1 web
```

## 2.6  管道与函数

前面讲的模块，其实就是将值传给模板引擎进行渲染，模板引擎还支持对拿到数据进行二次处理。

例如从.Values中读取的值变成字符串，可以使用`quote`函数实现：

```yaml
# vi templates/deployment.yaml
app: {{ quote .Values.label.app }}
# helm install --dry-run web ../mychart/ 
        project: ms
        app: "nginx"
```

quote .Values.label.app 将后面的值作为参数传递给quote函数。

模板函数调用语法为：functionName arg1 arg2...

另外还会经常使用一个default函数，该函数允许在模板中指定默认值，以防止该值被忽略掉。

例如忘记定义，执行helm install 会因为缺少字段无法创建资源，这时就可以定义一个默认值。



```yaml
# cat values.yaml 
replicas: 2
# cat templates/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
```



```
- name: {{ .Values.name | default "nginx" }}
```

其他函数：

缩进：{{ .Values.resources | indent 12 }}

大写：{{ upper .Values.resources }}

首字母大写：{{ title .Values.resources }}

## 2.7 流程控制

流程控制是为模板提供了一种能力，满足更复杂的数据逻辑处理。

Helm模板语言提供以下流程控制语句：

- `if/else` 条件块
- `with` 指定范围
- `range` 循环块

### if

`if/else`块是用于在模板中有条件地包含文本块的方法，条件块的基本结构如下：



```
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```



示例



```yaml
# cat values.yaml 
devops: k8

# cat templates/deployment.yaml 
...
  template:
    metadata:
      labels:
        app: nginx
        {{ if eq .Values.devops "k8s" }}
        devops: 123
        {{ else }}
        devops: 456
        {{ end }}
```



在上面条件语句使用了`eq`运算符判断是否相等，除此之外，还支持`ne`、 `lt`、 `gt`、 `and`、 `or`等运算符。

注意数据类型。

通过模板引擎来渲染一下，会得到如下结果：



```
# helm install --dry-run web ../mychart/ 
...
      labels:
        app: nginx
        
        devops: 456
```



可以看到渲染出来会有多余的空行，这是因为当模板引擎运行时，会将控制指令删除，所有之前占的位置也就空白了，需要使用{{- if ...}} 的方式消除此空行：



```yaml
# cat templates/deploymemt.yaml
...
        env:
        {{- if eq .Values.env.hello "world" }}
          - name: hello
            value: 123
        {{- end }}
```



现在是不是没有多余的空格了，如果使用`-}}`需谨慎，比如上面模板文件中：



```yaml
# cat templates/deploymemt.yaml
...
       env:
        {{- if eq .Values.env.hello "world" -}}
           - hello: true
        {{- end }}
```



这会渲染成：

```
        env:- hello: true
```

因为`-}}`它删除了双方的换行符。

条件判断就是判断条件是否为真，如果值为以下几种情况则为false：

- 一个布尔类型的 `false`
- 一个数字 `零`
- 一个 `空`的字符串
- 一个空的集合（ `map`、 `slice`、 `tuple`、 `dict`、 `array`）

除了上面的这些情况外，其他所有条件都为 `真`。

例如，判断一个空的数组



```yaml
# cat values.yaml 
resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

# cat templates/deploymemt.yaml
...
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
        {{- if .Values.resources }}
        resources:
{{ toYaml .Values.resources | indent 10 }}
        {{- end }}
```



例如，判断一个布尔值



```yaml
# cat values.yaml 
service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true 
  host: example.ctnrs.com

# cat templates/ingress.yaml 
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        backend:
          serviceName: {{ .Release.Name }}
          servicePort: {{ .Values.service.port }}
{{ end }}
```



### range

在 Helm 模板语言中，使用 `range`关键字来进行循环操作。

我们在 `values.yaml`文件中添加上一个变量列表：

```yaml
# cat values.yaml 
test:
  - 1
  - 2
  - 3
```

循环打印该列表：



```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}
data:
  test: |
  {{- range .Values.test }}
    {{ . }}
  {{- end }}
```



循环内部我们使用的是一个 `.`，这是因为当前的作用域就在当前循环内，这个 `.`引用的当前读取的元素。

### with

with ：控制变量作用域。

还记得之前我们的 `{{.Release.xxx}}`或者 `{{.Values.xxx}}`吗？其中的 `.`就是表示对当前范围的引用， `.Values`就是告诉模板在当前范围中查找 `Values`对象的值。而 `with`语句就可以来控制变量的作用域范围，其语法和一个简单的 `if`语句比较类似：

```
{{ with PIPELINE }}
  #  restricted scope
{{ end }}
```

`with`语句可以允许将当前范围 `.`设置为特定的对象，比如我们前面一直使用的 `.Values.label`，我们可以使用 `with`来将 `.`范围指向 `.Values.label`：



```yaml
# cat values.yaml 
...
nodeSelector:
  team: a
  gpu: yes

# cat templates/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      {{- with .Values.nodeSelector }}
      nodeSelector:
        team: {{ .team }}
        gpu: {{ .gpu }}
      {{- end }}
      containers:
      - image: nginx:1.16
        name: nginx
```



优化后：

```
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

上面增加了一个{{- with .Values.nodeSelector}} xxx {{- end }}的一个块，这样的话就可以在当前的块里面直接引用 `.team`和 `.gpu`了。

**with**是一个循环构造。使用`.Values.nodeSelector`中的值：将其转换为Yaml。

toYaml之后的点是循环中`.Values.nodeSelector`的当前值

## 变量

**变量**，在模板中，使用变量的场合不多，但我们将看到如何使用它来简化代码，并更好地利用with和range。

**问题1：获取数组键值**



```yaml
# cat ../values.yaml
env:
  NAME: "gateway"
  JAVA_OPTS: "-Xmx1G"
  
# cat deployment.yaml 
...
        env:
        {{- range $k, $v := .Values.env }}
           - name: {{ $k }}
             value: {{ $v | quote }}
        {{- end }}
```



结果如下

```
    env:
       - name: JAVA_OPTS
         value: "-Xmx1G"
       - name: NAME
         value: "gateway"
```

上面在 `range`循环中使用 `$key`和 `$value`两个变量来接收后面列表循环的键和值`。`

**问题2：with中不能使用内置对象**

`with`语句块内不能再 `.Release.Name`对象，否则报错。

我们可以将该对象赋值给一个变量可以来解决这个问题：



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicas }}
  template:
    metadata:
      labels:
        project: {{ .Values.label.project }}
        app: {{ quote .Values.label.app }}
      {{- with .Values.label }}
        project: {{ .project }}
        app: {{ .app }}
        release: {{ .Release.Name }}
      {{- end }}
```



上面会出错



```
      {{- $releaseName := .Release.Name -}}
      {{- with .Values.label }}
        project: {{ .project }}
        app: {{ .app }}
        release: {{ $releaseName }}
        # 或者可以使用$符号,引入全局命名空间
        release: {{ $.Release.Name }}
      {{- end }}
```



可以看到在 `with`语句上面增加了一句 `{{-$releaseName:=.Release.Name-}}`，其中 `$releaseName`就是后面的对象的一个引用变量，它的形式就是 `$name`，赋值操作使用 `:=`，这样 `with`语句块内部的 `$releaseName`变量仍然指向的是 `.Release.Name`

## 命名模板

需要复用代码的地方用。

命名模板：使用define定义，template引入，在templates目录中默认下划线*开头的文件为公共模板(*helpers.tpl)



```
# cat _helpers.tpl
{{- define "demo.fullname" -}}
{{- .Chart.Name -}}-{{ .Release.Name }}
{{- end -}}

# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ template "demo.fullname" . }}
...
```



template指令是将一个模板包含在另一个模板中的方法。但是，template函数不能用于Go模板管道。为了解决该问题，增加include功能。



```
# cat _helpers.tpl
{{- define "demo.labels" -}}
app: {{ template "demo.fullname" . }}
chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
release: "{{ .Release.Name }}"
{{- end -}}

# cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "demo.fullname" . }}
  labels:
    {{- include "demo.labels" . | nindent 4 }}
...
```



上面包含一个名为 `demo.labels` 的模板，然后将值 `.` 传递给模板，最后将该模板的输出传递给 `nindent` 函数。

## 开发自己的chart

1、先创建模板

2、修改Chart.yaml，Values.yaml，添加常用的变量

3、在templates目录下创建部署镜像所需要的yaml文件，并变量引用yaml里经常变动的字段

# 四、使用Harbor作为Chart仓库

**启用Harbor的Chart仓库服务**

```
# ./install.sh --with-chartmuseum
```

**安装push插件**

```
helm plugin install https://github.com/chartmuseum/helm-push
```

**添加repo**

```
# helm repo add --username admin --password Harbor12345 myrepo http://192.168.0.241/chartrepo/library
```

推送到harbor仓库

```
# helm push mysql-1.4.0.tgz --username=admin --password=Harbor12345 http://192.168.0.241/chartrepo/library
# helm install web --version 1.4.0 myrepo/demo
```

# 五、helm命令说明

##### helm的使用

```
helm常用命令：
- helm search:    搜索charts
- helm fetch:     下载charts到本地目录
- helm install:   安装charts
- helm list:      列出charts的所有版本

用法:
  helm [command]

命令可用选项:
  completion  为指定的shell生成自动补全脚本（bash或zsh）
  create      创建一个新的charts
  delete      删除指定版本的release
  dependency  管理charts的依赖
  fetch       下载charts并解压到本地目录
  get         下载一个release
  history     release历史信息
  home        显示helm的家目录
  init        在客户端和服务端初始化helm
  inspect     查看charts的详细信息
  install     安装charts
  lint        检测包的存在问题
  list        列出release
  package     将chart目录进行打包
  plugin      add(增加), list（列出）, or remove（移除） Helm 插件
  repo        add(增加), list（列出）, remove（移除）, update（更新）, and index（索引） chart仓库
  reset       卸载tiller
  rollback    release版本回滚
  search      关键字搜索chart
  serve       启动一个本地的http server
  status      查看release状态信息
  template    本地模板
  test        release测试
  upgrade     release更新
  verify      验证chart的签名和有效期
  version     打印客户端和服务端的版本信息
```