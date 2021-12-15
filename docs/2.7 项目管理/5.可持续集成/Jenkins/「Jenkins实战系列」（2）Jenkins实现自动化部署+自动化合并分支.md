# 「Jenkins实战系列」（2）Jenkins实现自动化部署+自动化合并分支

原文地址：https://blog.51cto.com/alex4dream/3006525



## 技术资源推荐

[jenkins官方文档（中文版）](https://www.jenkins.io/zh/doc/)

[jenkins官方网站](https://plugins.jenkins.io/git/#GitPlugin-AdvancedFeatures)

## 自动定时构建

### 定时构建语法：

```html
* * * * *
1.
```

- 第一个 * 表示分钟，取值0~59，若其他值不做设定，则表示每个设定的分钟都会构建
- 第二个 * 表示小时，取值0~23， 若其他值不做设定，则表示每个设定小时的每分钟都会构建
- 第三个 * 表示一个月的第几天，取值1~31，若其他值不做设定，则表示每个月的那一天每分钟都会构建一次
- 第四个 * 表示第几月，取值1~12，若其他值不做设定，则表示每年的那个月每分钟都会构建一次
- 第五个 * 表示一周中的第几天，取值0~7，其中0和7代表的都是周日，若其他值不做设定，则表示每周的那一天几每分钟都会构建一次

### 常用定时构建举例：

- **由于项目的代码一般存在放SVN/GIT中，而一个SVN/GIT往往是有多个项目组在提交代码，而每个项目组又有多人组成，其中每个人也都在对自己的那块代码不停地在进行维护**。
- **所以说对于一个公司而言，SVN/GIT的提交记录往往是很频繁的，正因为如此，Jenkins在执行自动化构建时往往是以天为单位来执行的，下面举的例子就是在一天中常用的定时构建示例**。

#### 每隔5分钟构建一次

> H/5 * * * *

#### 每两小时构建一次

> H H/2 * * *

#### 每天中午下班前定时构建一次

> 0 12 * * *

#### 每天下午下班前定时构建一次

> 0 18 * * *

### 定时构建位置

> **在“配置”->“构建触发器”中，如下图所示**：

![img](https://oscimg.oschina.net/oscnet/up-37467665b5c5d96dd26638ff6f84fdfb100.png)

> **Build after other projects are built：在其他项目触发的时候触发，里面有分为三种情况，也就是其他项目构建成功、失败、或者不稳定的时候触发项目；**

#### **Poll SCM和Build periodically来进行定时自动构建项目**；

- **Poll SCM：定时检查源码变更（根据SCM软件的版本号），如果有更新就checkout最新code下来，然后执行构建动作。如下图配置**：

在 构建触发器 中选择“Poll SCM”

![img](https://oscimg.oschina.net/oscnet/up-a9f7b379807473ba2a28d81d9e028cacb3f.png)

##### Build periodically

![img](https://oscimg.oschina.net/oscnet/up-322c7f17e83fe3a808dc57aaa65925a84e5.png)

##### Poll SCM

![img](https://oscimg.oschina.net/oscnet/up-81d020d00027c1ed175bb6a636fa22f7c37.png)

> **输入框为 分，时，日，月，星期，\* 号代表不限制，我设置的是每天早8点自动部署**。

![img](https://oscimg.oschina.net/oscnet/up-444f81f11292bde498170578e7c792d078b.png)

> - / 5 * * * * （**每5分钟检查一次源码变化**）

- **Build periodically：周期进行项目构建（它不关心源码是否发生变化），如下图配置**：

> H 2 * * * （每天2:00 必须build一次源码）

#### 在 Schedule 中填写

> 0 * * * *

- **第一个参数代表的是分钟 minute，取值 0~59；**
- **第二个参数代表的是小时 hour，取值 0~23；**
- **第三个参数代表的是天 day，取值 1~31；**
- **第四个参数代表的是月 month，取值 1~12；**
- **最后一个参数代表的是星期 week，取值 0~7，0 和 7 都是表示星期天。**

> **所以 0 \* \* \* \* 表示的就是每个小时的第 0 分钟执行构建。**

### 最后补充几个案例

- 在每个小时的前半个小时内的每10分钟

> H(0-29)/10 * * * *

- **每两小时45分钟，从上午9:45开始，每天下午3:45结束**

> 45 9-16/2 * * 1-5

- 每两小时一次，每个工作日上午9点到下午5点(也许是上午10:38，下午12:38，下午2:38，下午4:38)

> H H(9-16)/2 * * 1-5

- **每月(除了12月)从1号到15号这段时间内某刻**。

> H H 1,15 1-11 *

- **每周四，19点30分**

> 30 19 * * 4

- **每天早晨8点**

> 0 8 * * *

- **每周六日的8点12点18点构建**

> H 8,12,18 * * 6,7

- **每周1到周5，8点到23点，每小时构建一次**

> H 8-23 * * 1-5

- **每周1到周5，8点到23点，每两小时构建一次**

> H 8-23/2 * * 1-5

- **每周1到周5，8点到23点，每30分钟构建一次**

> H/30 8-23 * * 1-5

## Jenkins自动合并分支

通过jenkins发布项目时，可能需要合并分支，如：发布测试环境的时候需要把dev分支合并到test分支，此时就可以用jenkins自动实现。

![img](https://oscimg.oschina.net/oscnet/up-0b7182d811a6c454ba931c02e0b0bf85224.png)

![img](https://oscimg.oschina.net/oscnet/up-d62244e8784d081ff3ac708fa813f67795b.png)

![img](https://oscimg.oschina.net/oscnet/up-28ceac1db06c275d985ad410d725056b0db.png)

## 配置构建的参数

![img](https://oscimg.oschina.net/oscnet/up-5a5f7dfac4fcbbf4c72a67b95a2b758931d.png)

- Source files     项目构建后的目录
- Remove prefix    去前缀
- Remote directoty 发布的目录
- Exec command     发布完执行的命令，我这边写的是发布完会重启tomcat

```bash
#! /bin/bash
tomcat_home=/usr/local/tomcat-8
SHUTDOWN=$tomcat_home/bin/shutdown.sh
STARTTOMCAT=$tomcat_home/bin/startup.sh
echo "关闭$tomcat_home"
$SHUTDOWN
#杀死tomcat进程

ps -ef | grep $path | grep java | awk '{print $2}' |  xargs kill -9

#删除日志文件，如果你不先删除可以不要下面一行
rm  $tomcat_home/logs/* -rf
#删除tomcat的临时目录
rm  $tomcat_home/work/* -rf
sleep 5
echo "启动$tomcat_home"
$STARTTOMCAT
#看启动日志
#tail -f $tomcat_home/logs/catalina.out
```

