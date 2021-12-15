# Go 语言特色

- 简洁、快速、安全
- 并行、有趣、开源
- 内存管理、数组安全、编译迅速



# Go 语言用途

Go 语言被设计成一门应用于搭载 Web 服务器，存储集群或类似用途的巨型中央服务器的系统编程语言。

对于高性能分布式系统领域而言，Go 语言无疑比大多数其它语言有着更高的开发效率。它提供了海量并行的支持，这对于游戏服务端的开发而言是再好不过了。



Go 语言的基础组成有以下几个部分：

- 包声明
- 引入包
- 函数
- 变量
- 语句 & 表达式
- 注释



# 环境搭建

- [搭建Go语言开发环境](https://www.pkslow.com/archives/go-setup-env)

## 下载软件包

通过命令行下载：

```bash
$ curl https://dl.google.com/go/go1.15.darwin-amd64.tar.gz -o go1.15.darwin-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  116M  100  116M    0     0   283k      0  0:07:02  0:07:02 --:--:-- 7941k
```

或到网页下载：https://golang.org/dl/

[![img](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/08/go-setup.download-page.png)](https://pkslow.oss-cn-shenzhen.aliyuncs.com/images/2020/08/go-setup.download-page.png)

## 解压缩

解压：

```bash
$ tar -C /Users/pkslow/Software/ -xzf go1.15.darwin-amd64.tar.gz
```

## 配置环境变量

配置环境变量到`.bash_profile`：

```bash
export GO_HOME=/Users/pkslow/Software/go
export PATH=$PATH:$GO_HOME/bin
```

使配置生效：

```bash
$ source .bash_profile 
$ go version
go version go1.15 darwin/amd64
```

## 测试

编辑一个文件：

```bash
$ vi pkslow.go
```

内容如下：

```go
package main
import "fmt"

func main() {
	fmt.Printf("welcome to www.pkslow.com\n")
}
```

编译并执行：

```bash
$ go build pkslow.go
$ ./pkslow 
welcome to www.pkslow.com
```

一切正常，说明成功安装。

