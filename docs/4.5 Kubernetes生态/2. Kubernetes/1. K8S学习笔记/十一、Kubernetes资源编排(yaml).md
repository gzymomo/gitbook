# 1、YAML文件概述

k8s集群中对资源管理和资源对象编排部署都可以通过声明样式(YAML)文件来解决，也就是可以把需要对资源对象操作编辑到YAML格式文件中，我们把这种文件叫做资源清单文件，通过kubectl命令直接使用资源清单文件就可以实现对大量的资源对象进行编排部署了。



# 2、YAML文件书写格式

## 2.1 YAML介绍

YAML :仍是一种标记语言。为了强调这种语言以数据做为中心，而不是以标记语言为重点。

YAML是一个可读性高，用来表达数据序列的格式。



## 2.2 YAML基本语法，语法格式

- 使用空格做为缩进

- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

- 低版本缩进时不允许使用Tab键，只允许使用空格

- 使用#标识注释，从这个字符一直到行尾，都会被解释器忽略
- 通过缩进表示层级关系
- 一般开头缩进两个空格
- 字符后缩进一个空格，比如冒号，逗号等后面
- 使用---表示新的yaml文件开始



# 3、yaml文件组成部分

![](..\..\img\yaml.png)



## 3.1 控制器定义



## 3.2 被控制对象





# 4、常用字段含义



- apiVersion：API版本
- kind：资源类型
- metadata：资源元数据
- spec：资源规格
- replicas：副本数量
- selector：标签选择器
- template：Pod模板
- metadata：Pod元数据
- spec：Pod规格
- containers：容器配置

# 5、如何快速编写yaml文件

## 5.1 使用kubectl create命令生成yaml文件

```bash
kubectl create deployment web --image=nginx -o yaml --dry-run
```

将yaml输入到一个文件中

```bash
kubectl create deployment web --image=nginx -o yaml --dry-run > my1.yaml
```



## 5.2 使用kubectl get命令导出yaml文件

```bash
kubectl get deploy nginx -o=yaml --export > my2.yaml
```

