[TOC]

- [一文全面总结MongoDB知识体系](https://www.cnblogs.com/pengdai/p/14515673.html)

- [MongoDB 运维实战总结](https://www.jianshu.com/p/f05f65d3a1dc)

# 一、MangoDB简介

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

![](https://www.runoob.com/wp-content/uploads/2013/10/crud-annotated-document.png)

## 1.1 主要特点
MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。
可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。
可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。
MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。

## 1.2 MongoDB解析
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/9dc25f93a1987aa71266b75465cca31f?showdoc=.jpg)

# 二、Docker搭建MongDB

```bash
docker pull mongo:latest

$ docker run -itd --name mongo -p 27017:27017 mongo --auth

参数说明：

-p 27017:27017 ：映射容器服务的 27017 端口到宿主机的 27017 端口。外部可以直接通过 宿主机 ip:27017 访问到 mongo 的服务。
--auth：需要密码才能访问容器服务。

docker run -itd --name mongo -p 27017:27017 mongo
```

接着使用以下命令添加用户和设置密码，并且尝试连接。

```bash
$ docker exec -it mongodb mongo admin
# 创建一个名为 admin，密码为 123456 的用户。
>  db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
# 尝试使用上面创建的用户信息进行连接。
> db.auth('admin', '123456')
```

# 三、MongDB使用

## 3.1 MongoDB 创建数据库

以下实例我们创建了数据库 runoob:

```sql
> use runoob
switched to db runoob
> db
runoob
```

想查看所有数据库，可以使用 **show dbs** 命令：

```sql
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

可以看到，我们刚创建的数据库 runoob 并不在数据库的列表中， 要显示它，我们需要向 runoob 数据库插入一些数据。

```sql
> db.runoob.insert({"name":"菜鸟教程"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB
```

MongoDB 中默认的数据库为 test，如果你没有创建新的数	库，集合将存放在 test 数据库中。