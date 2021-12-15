- 博客园：叶剑峰：[grafana 的主体架构是如何设计的？](https://www.cnblogs.com/yjf512/p/14169141.html)





# 一、Grafana介绍

Grafana 是非常强大的可视化项目，它最早从 kibana 生成出来，渐渐也已经形成了自己的生态了。研究完 grafana 生态之后，只有一句话：可视化，grafana 就够了。



Grafana是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示。Grafana提供了对prometheus的友好支持，各种工具帮助你构建更加炫酷的数据可视化。

## 1.1 Grafana特点

- 可视化：快速和灵活的客户端图形具有多种选项。面板插件为许多不同的方式可视化指标和日志。
- 报警：可视化地为最重要的指标定义警报规则。Grafana将持续评估它们，并发送通知。
- 通知：警报更改状态时，它会发出通知。接收电子邮件通知。
- 动态仪表盘：使用模板变量创建动态和可重用的仪表板，这些模板变量作为下拉菜单出现在仪表板顶部。
- 混合数据源：在同一个图中混合不同的数据源!可以根据每个查询指定数据源。这甚至适用于自定义数据源。
- 注释：注释来自不同数据源图表。将鼠标悬停在事件上可以显示完整的事件元数据和标记。
- 过滤器：过滤器允许您动态创建新的键/值过滤器，这些过滤器将自动应用于使用该数据源的所有查询。



## 1.2 Grafana UI

可以通过第三方提供可视化JSON文件来帮助我们快速实现服务器、Elasticsearch、MYSQL等等监控。这里我们在grafana提供的第三方dashboards的地址https://grafana.com/grafana/dashboards来下载对应的json文件然后导入到grafana实现服务器的监控。





## 1.3入口代码

grafana 的最外层就是一个 build.go，它并不是真正的入口，它只是用来编译生成 grafana-server 工具的。

grafana 会生成两个工具，grafana-cli 和 grafana-server。

go run build.go build-server 其实就是运行

```goalng
go build ./pkg/cmd/grafana-server -o ./bin/xxx/grafana-server
```

如果你的项目要生成多个命令行工具，又或者有多个参数，又或者有多个操作，使用 makefile 已经很复杂了，我们是可以这样直接写个 build.go 或者 main.go 在最外层，来负责编译的事情。

所以真实的入口在 ./pkg/cmd/grafana-server/main.go 中。可以跟着这个入口进入。



# 二、设计结构

grafana 中最重要的结构就是 Service。 grafana 设计的时候希望所有的功能都是 Service。是的，所有，包括用户认证 UserAuthTokenService，日志 LogsService， 搜索 LoginService，报警轮训 Service。 所以，这里需要设计出一套灵活的 Service 执行机制。

## 2.1 注册机制

首先，需要有一个 Service 的注册机制。

grafana 提供的是一种有优先级的，服务注册机制。grafana 提供了 pkg/registry 包。

在 Service 外层包了一个结构，包含了服务的名字和服务的优先级。

```golang
type Descriptor struct {
	Name         string
	Instance     Service
	InitPriority Priority
}
```

这个包提供的三个注册方法：

```golang
RegisterServiceWithPriority
RegisetrService
Register
```

这三个注册方法都是把 Descriptior（本质也就是 Service）注册到一个全局的数组中。

取的时候也很简单，就是把这个全局数组按照优先级排列就行。

那么什么时候执行注册操作呢？答案就是在每个 Service 的 init() 函数中进行注册操作。所以我们可以看到代码中有很多诸如：

```golang
_ "github.com/grafana/grafana/pkg/services/ngalert"
_ "github.com/grafana/grafana/pkg/services/notifications"
_ "github.com/grafana/grafana/pkg/services/provisioning"
```

的 import 操作，就是为了注册服务的。

## 2.2 Service 的类型

如果我们自己定义 Service，差不多定义一个 interface 就好了，但是实际这里是有问题的。我们有的服务需要的是后端启动，有的服务并不需要后端启动，而有的服务需要先创建一个数据表才能启动，而有的服务需要根据配置文件判断是否开启。要定义一个 Service 接口满足这些需求，其实也是可以的，只是比较丑陋，而 grafana 的写法就非常优雅了。

grafana 定义了基础的 Service 接口，仅仅需要实现一个 Init() 方法：

```golang
type Service interface {
	Init() error
}
```

而定义了其他不同的接口，比如需要后端启动的服务：

```golang
type BackgroundService interface {
	Run(ctx context.Context) error
}
```

需要数据库注册的服务：

```golang
type DatabaseMigrator interface {
	AddMigration(mg *migrator.Migrator)
}
```

需要根据配置决定是否启动的服务：

```golang
type CanBeDisabled interface {
	IsDisabled() bool
}
```

在具体使用的时候，根据判断这个 Service 是否符合某个接口进行判断。

```golang
service, ok := svc.Instance.(registry.BackgroundService)
if !ok {
    continue
}
```

这样做的优雅之处就在于在具体定义 Service 的时候就灵活很多了。不会定义很多无用的方法实现。

## 2.3 Service 的依赖

这里还有一个麻烦的地方，Service 之间是有互相依赖的。比如 sqlstore.SQLStore 这个服务，是负责数据存储的。它会在很多服务中用到，比如用户权限认证的时候，需要去数据存储中获取用户信息。那么这里如果在每个 Service 初始化的时候进行实例化，也是颇为痛苦的事情。

grafana 使用的是 facebook 的 inject.Graph 包处理这种依赖的问题的。

这个 inject 包使用的是依赖注入的解决方法，把一堆实例化的实例放进包里面，然后使用反射技术，对于一些结构中有指定 tag 标签的字段，就会把对应的实例注入进去。

比如 grafana 中的：

```golangß
type UserAuthTokenService struct {
	SQLStore          *sqlstore.SQLStore            `inject:""`
	ServerLockService *serverlock.ServerLockService `inject:""`
	Cfg               *setting.Cfg                  `inject:""`
	log               log.Logger
}
```

这里可以看到 SQLStore 中有额外的注入 tag。那么在 pkg/server/server.go 中的

```golang
services := registry.GetServices()
if err := s.buildServiceGraph(services); err != nil {
    return err
}
```

这里会把所有的 Service （包括这个 UserAuthTokenService） 中的 inject 标签标记的字段进行依赖注入。

这样就完美解决了 Service 的依赖问题。

## 2.4 Service 的运行

Service 的运行在 grafana 中使用的是 errgroup, 这个包是 “golang.org/x/sync/errgroup”。

使用这个包，不仅仅可以并行 go 执行 Service，也能获取每个 Service 返回的 error，在最后 Wait 的时候返回。

大体代码如下：

```golang
s.childRoutines.Go(func() error {
		...
		err := service.Run(s.context)
		...
	})
}

defer func() {
	if waitErr := s.childRoutines.Wait(); waitErr != nil && !errors.Is(waitErr, context.Canceled) {
		s.log.Error("A service failed", "err", waitErr)
		if err == nil {
			err = waitErr
		}
	}
}()
```

![img](https://img2020.cnblogs.com/blog/136188/202012/136188-20201221181011144-791162279.png)