YAML 是 "YAML Ain't a Markup Language"  的缩写，是一种可读性高的数据序列化语言，常用于配置管理中。在云原生时代，很多流行的开源项目、云平台等都是 YAML 格式表达的，比如  Kubernetes 中的资源对象、Ansible/Terraform 的配置文件以及流行 CI/CD 平台的配置文件等等。

## 基本格式

首先需要理解的是 YAML 主要是面向数据而非表达能力，所以 YAML 本身的语法非常简洁，其最基本的语法规则为：

- 大小写敏感
- 使用空格缩进表示层级关系（不可以使用 TAB）
- 以 # 表示注释，行内注释 # 前面必须要有空格
- 基本数据类型包括 Null、布尔、字符串、整数、浮点数、日期和时间等
- 基本数据结构包括字典、列表以及纯量（即单个基本类型的值），其他复杂数据结构都是通过这些基本数据结构组合而成
- 单个文件包括多个 YAML 数据结构时使用 `---` 分割

## 字典

字典有两种表达方式，两种方式是对等的，一般方式二用的多一些：

```
# 方式一（行内表达法）
foo: { thing1: huey, thing2: louie, thing3: dewey }

# 方式二
foo:
  thing1: huey
  thing2: louie
  thing3: dewey
```

## 列表

列表也有两种表达方式，一般方式一用的多一些：

```
# 方式一
files:
- foo.txt
- bar.txt
- baz.txt

# 方式二（行内表达法）
lists: [foo.txt, bar.txt, baz.txt]
```

## 字符串

YAML 中默认字符串不需要添加任何引号，但在容易导致混淆的地方则是需要添加引号的。比如

- 字符串格式的数字必须加上引号，比如 "20"
- 字符串格式的布尔值必须加上引号，比如 "true"

YAML 也支持多行字符串，可以使用 `>`（折叠换行） 或者 `|`（保留换行符）：

比如

```
# 折叠换行符，即等同于 "bar : this is not a normal string it spans more than one line see?"
bar: >
this is not a normal string it
spans more than
one line
see?

# 保留换行符
bar: |
this is not a normal string it
spans more than
one line
see?
```

## 片段和引用（Snippet）

YAML 也支持片段和引用，这些构造复杂数据类型时非常有用。你可以用 `&` 来定义一个片段，随后使用 `*` 来引用这个片段。

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibUvYDg8ZxjPITia4dJibMAVVPUCatWJJfqjPcibrvic4IJ1owSV6zZVpxa6qFsPFOy1yhodN1UYk9yfAIPHBKqhKLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Kubernetes 资源对象

Kubernetes 资源对象格式可以参考其官方 API 文档 https://kubernetes.io/docs/reference/kubernetes-api/。需要注意的是，每个资源对象在定义的时候必须包含以下的字段：

- apiVersion - 创建该对象所使用的 Kubernetes API 的版本
- kind - 想要创建的对象的类别
- metadata - 帮助唯一性标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespace

通常，你也需要提供对象的 spec 字段。对象 spec 的详细格式对每个 Kubernetes 对象来说是不同的，包含了特定于该对象的嵌套字段。比如，一个 Nginx Pod 的定义如下所示：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-6799fc88d8-tpx29
  namespace: default
  labels:
    app: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
  restartPolicy: Always
```

## 辅助工具

最后再推荐几个常用的 YAML 工具，包括 yamllint、yq、以及 kube-score。

### yamllit

yamllint 是一个用来检查 YAML 语法的工具，你可以通过 `pip install --user yamllint` 命令来安装该工具。

比如，对上面的 Nginx Pod 运行 yamllint 会得到如下的警告和错误：

```
$ yamllint nginx.yaml
nginx.yaml
  1:1       warning  missing document start "---"  (document-start)
  10:3      error    wrong indentation: expected 4 but found 2  (indentation)
```

根据这两个警告和错误，可以把其修改成如下的格式：

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-6799fc88d8-tpx29
  namespace: default
  labels:
    app: nginx
spec:
  containers:
    - image: nginx
      imagePullPolicy: Always
      name: nginx
  restartPolicy: Always
```

### yq

yq 是一个 YAML 数据处理以及高亮显示的工具，你可以通过 `brew install yq` 来安装该工具（类似于 JSON 数据处理的 `jq` 工具）。

比如，你可以高亮显示上面的 Nginx Pod YAML：

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibUvYDg8ZxjPITia4dJibMAVVPUCatWJJfqU6N3Gsro0sdLb4uMUicpiapfZ4R9pLYHgsKJu1ZXdsDwd4N3VORet3tA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

或者提取上述 YAML 的部分内容：

```
$ yq eval '.metadata.name' nginx.yaml
nginx-6799fc88d8-tpx29
```

或者修改 YAML 中 `metadata.name` 字段：

```
# yq eval '.metadata.name = "nginx"' nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  containers:
    - image: nginx
      imagePullPolicy: Always
      name: nginx
  restartPolicy: Always
```

### kube-score

kube-score 是一个 Kubernetes YAML 静态分析工具，用来检查 Kubernetes 资源对象的配置是否遵循了最佳实践。你可以通过 `kubectl krew install score` 来安装该工具。

比如，还是上述的 Nginx Pod，运行 `kubectl score nginx.yaml` 可以得到如下的错误：

![图片](https://mmbiz.qpic.cn/mmbiz_png/ibUvYDg8ZxjPITia4dJibMAVVPUCatWJJfqKYTtUiaTx4EqKfrQ1TTlx5oGKSMfhESAzzxCFw0Hr4tfWKPXwYqcquw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

根据这些错误，可以发现容器资源、镜像标签、网络策略以及容器安全上下文等四个配置没有遵循最佳实践。

## 参考资料

更多 YAML 的使用细节可以参考如下的资料：

- YAML 语言规范 https://yaml.org/spec/1.2/spec.html
- Kubernetes 资源对象 https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/