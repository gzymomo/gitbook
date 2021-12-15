# 一、Jenkins的安装与部署

## 1.1 Docker方式部署Jenkins

1. 拉取镜像

```bash
docker pull jenkins/jenkins
```

2. 创建Jenkins挂载目录并授权权限

```bash
mkdir -p /var/jenkins_home
chmod 777 /var/jenkins_home
```

3. 启动jenkins容器

```bash
docker run -d -p 10240:8080 -p 10241:50000 -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/mv:/usr/bin/mv -v /usr/local/java/jdk1.8.0_271/bin:/usr/local/java/jdk1.8.0_271/bin -v /usr/local/java/jdk1.8.0_271:/usr/local/java/jdk1.8.0_271 -v /usr/local/maven3.6:/usr/local/maven3.6 -v /etc/localtime:/etc/localtime -v /var/jenkins_home:/var/jenkins_home   --name myjenkins  jenkins/jenkins
```

<font color='bule'> tips:需先在宿主机安装jdk和maven环境，此处不要挂载git，挂载git时，发现无法拉取代码。</font>

命令说明：

>​    **-d 后台运行镜像**
>
>　**-p 10240:8080 将镜像的8080端口映射到服务器的10240端口。**
>
>　**-p 10241:50000 将镜像的50000端口映射到服务器的10241端口**
>
>　**-v /var/jenkins_home:/var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_home目录**
>
>　**-v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。**
>
>   **-v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock  设置docker**
>
>   **-v /usr/bin/mv:/usr/bin/mv -v /usr/local/java/jdk1.8.0_271/bin:/usr/local/java/jdk1.8.0_271/bin 设置jdk**
>
>   **-v /usr/local/maven3.6:/usr/local/maven3.6 设置maven**
>
>　**--name myjenkins 给容器起一个别名**



4. 查看是否启动成功及查看日志

```bash
# 查看是否启动成功，若容器STATUS处于Up状态，在表示启动成功
docker ps -a

# 查看日志
docker logs myjenkins
```

5. 配置镜像加速

```bash
# 进入 cd /var/jenkins_mount/ 目录
cd /var/jenkins_home/
# 修改 vim hudson.model.UpdateCenter.xml里的内容
将 url 修改为 清华大学官方镜像：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

6. 访问Jenkins

```bash
ip:端口
```

7. 管理员密码位置

```bash
# 查看管理员密码
vi /var/jenkins_home/secrets/initialAdminPassword
```



# 二、Jenkins部署SpringBoot应用

## 2.1 构建Spring Boot 项目

在构建之前还需要配置一些开发环境，比如`JDK`，`Maven`等环境。

### 配置JDK、maven、Git环境

`Jenkins`集成需要用到`maven`、`JDK`、`Git`环境，下面介绍如何配置。

首先打开`系统管理`->`全局工具配置`，如下图：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/8.png)

分别配置`JDK`，`Git`，`Maven`的路径，根据你的实际路径来填写。

**「注意」**：这里的`JDK`、`Git`、`Maven`环境一定要挂载到`docker`容器中，否则会出现以下提示：

```
 xxxx is not a directory on the Jenkins master (but perhaps it exists on some agents)
```

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/9.png)

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/10.png)

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/11.png)

配置成功后，点击保存。

### 安装插件

除了初始化配置中安装的插件外，还需要安装如下几个插件：

1. `Maven Integration`
2. `Publish Over SSH`

打开`系统管理` -> `插件管理`，选择`可选插件`，勾选中 `Maven Integration` 和 `Publish Over SSH`，点击`直接安装`。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/12.png)

在安装界面勾选上安装完成后重启 `Jenkins`。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/24.png)

### 添加 SSH Server

`SSH Server` 是用来连接部署服务器的，用于在项目构建完成后将你的应用推送到服务器中并执行相应的脚本。

打开 `系统管理` -> `系统配置`，找到 `Publish Over SSH` 部分，选择`新增`

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/13.png)

点击 `高级` 展开配置

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/14.png)

最终配置如下：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/15.png)

配置完成后可点击 `Test Configuration` 测试连接，出现 `success` 则连接成功。

### 添加凭据

凭据 是用来从 `Git` 仓库拉取代码的，打开 `凭据` -> `系统` -> `全局凭据` -> `添加凭据`

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/16.png)

这里配置的是`Github`，直接使用`用户名`和`密码`，如下图：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/17.png)

创建成功，点击保存。

## 2.2新建Maven项目

以上配置完成后即可开始构建了，首先需要新建一个`Maven`项目，步骤如下。

#### 创建任务

首页点击`新建任务`->`构建一个maven项目`，如下图：![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/18.png)

#### 源码管理

在源码管理中，选择`Git`，填写`仓库地址`，选择之前添加的`凭证`。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/19.png)

#### 构建环境

勾选 `Add timestamps to the Console Output`，代码构建的过程中会将日志打印出来。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/20.png)

#### 构建命令

在`Build`中，填写 `Root POM` 和 `Goals and options`，也就是你构建项目的命令。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/21.png)

#### Post Steps

选择`Run only if build succeeds`，添加 `Post` 步骤，选择 `Send files or execute commands over SSH`。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/22.png)

上图各个选项解析如下：

1. `name`:选择前面添加的`SSH Server`
2. `Source files`:要推送的文件
3. `Remove prefix`:文件路径中要去掉的前缀，
4. `Remote directory`:要推送到目标服务器上的哪个目录下
5. `Exec command`:目标服务器上要执行的脚本

`Exec command`指定了需要执行的脚本，如下：

```bash
# jdk环境，如果全局配置了，可以省略
export JAVA_HOME=/xx/xx/jdk
export JRE_HOME=/xx/xx/jdk/jre
export CLASSPATH=/xx/xx/jdk/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
 
# jenkins编译之后的jar包位置，在挂载docker的目录下
JAR_PATH=/data/jenkins_home/workspace/test/target
# 自定义的jar包位置
DIR=/data/test

## jar包的名称
JARFILE=swagger-demo-0.0.1-SNAPSHOT.jar

if [ ! -d $DIR/backup ];then
   mkdir -p $DIR/backup
fi

ps -ef | grep $JARFILE | grep -v grep | awk '{print $2}' | xargs kill -9

if [ -f $DIR/backup/$JARFILE ]; then
 rm -f $DIR/backup/$JARFILE
fi

mv $JAR_PATH/$JARFILE $DIR/backup/$JARFILE


java -jar $DIR/backup/$JARFILE > out.log &
if [ $? = 0 ];then
        sleep 30
        tail -n 50 out.log
fi

cd $DIR/backup/
ls -lt|awk 'NR>5{print $NF}'|xargs rm -rf
```

以上脚本大致的意思就是将`kill`原有的进程，启动新构建`jar`包。

> 脚本可以自己定制，比如备份`Jar`等操作。

## 2.3 构建任务

项目新建完成之后，一切都已准备就绪，点击`立即构建`可以开始构建任务，控制台可以看到`log`输出，如果构建失败，在`log`中会输出原因。

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/23.png)

任务构建过程会执行脚本启动项目。

## 2.4 如何构建托管在GitLab的项目？

上文介绍的例子是构建`Github`仓库的项目，但是企业中一般都是私服的`GitLab`，那么又该如何配置呢？

其实原理是一样的，只是在构建任务的时候选择的是`GitLab`的凭据，下面将详细介绍。

### 安装插件

在`系统管理`->`插件管理`->`可选插件`中搜索`GitLab Plugin`并安装。

### 添加GitLab API token

首先打开 `凭据` -> `系统` -> `全局凭据` -> `添加凭据`，如下图：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/25.png)

上图中的`API token`如何获取呢？

打开`GitLab`（例如公司内网的`GitLab`网站），点击个人设置菜单下的`setting`，再点击`Account`，复制`Private token`，如下：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/26.png)

上图的`Private token`则是`API token`，填上即可。

### 配置GitLab插件

打开`系统管理`->`系统配置`->`GitLab`，如下图：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/27.png)

配置成功后，点击`Test Connection`，如果提示`Success`则配置成功。

### 新建任务

新建一个Maven任务，配置的步骤和上文相同，唯一区别则是配置`Git`仓库地址的地方，如下图：

![img](https://gitee.com/chenjiabing666/BlogImage/raw/master/Spring%20Boot%20%E9%9B%86%E6%88%90%20Jenkins/28.png)

仓库地址和凭据需要填写`Gitlab`相对应的。





# 三、Jenkins+GitLab自动化部署

在gitlab上配置jenkins的webhook，当有代码变更时自动触发jenkins构建job，job内的shell脚本负责把覆盖率报告以钉钉群通知的方法发送出去。

![img](https://img2020.cnblogs.com/blog/907091/202007/907091-20200701223555441-513722191.png)

## 3.1 配置jenkins

### 添加 GitLab 凭据

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LWYxZGYxOTBlNWMxODY0OWIucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

首页 -> 凭据 -> 系统 -> 全局凭据 -> 添加凭据, 把上面 GitLab 中生成的 access token 填进去

### 配置 GitLab 连接

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LTM0ZmExNzBhY2YxNDVhNzgucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

首页 -> 系统管理 -> 系统设置 -> Gitlab 配置项, 填入 GitLab 相关的配置, 后面配置项目时用到

### 新建项目 test

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LTE3ZDFlNjM2NzMxOGExM2YucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

Jenkins项目完整配置

- **勾选** 参数化构建过程, 添加 **Git Parameter** 类型的参数 **ref** , 这样构建的时候就可以指定分支进行构建。
- **Source Code Management** 选择 **Git** , 添加项目地址和授权方式 ( **帐号密码** 或者 **ssh key** ) , 分支填写构建参数 **$ref**。
- **Build Triggers** 选择 **Generic Webhook Trigger** 方式用于解析 **GitLab** 推过来的详细参数 ( [jsonpath 在线测试](https://links.jianshu.com/go?to=http%3A%2F%2Fjsonpath.com) ) 。其他触发方式中: [Trigger builds remotely](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fjwentest%2Fp%2F8204421.html) 是 **Jenkins** 自带的, **Build when a change is pushed to GitLab** 是 **GitLab 插件** 提供的, 都属于简单的触发构建, 无法做复杂的处理。
- 虽然 **Generic Webhook Trigger** 提供了 **Token** 参数进行鉴权, 但为了避免不同项目进行混调 ( 比如 A 项目提交代码却触发了 B 项目的构建) , 还要对请求做下过滤。**Optional filter** 中 **Text** 填写需要校验的内容 ( 可使用变量 ) , **Expression** 使用正则表达式对 **Text** 进行匹配, 匹配成功才允许触发构建。
- **Build** 内容按自己实际的项目类型进行调整, 使用 **Maven 插件** 或 **脚本** 等等。
- **GitLab Connection** 选择上面添加的 **GitLab 连接 ( `Jenkins` )** , **Post-build Actions** 添加 **Publish build status to GitLab** 动作, 实现构建结束后通知构建结果给 **GitLab**。

- 回到 **GitLab** 的项目页面中, 添加一个 **Webhook** ( [http://JENKINS_URL/generic-webhook-trigger/invoke?token=](https://links.jianshu.com/go?to=http%3A%2F%2FJENKINS_URL%2Fgeneric-webhook-trigger%2Finvoke%3Ftoken%3D)<上面 **Jenkins** 项目配置中的 **token**> ) , 触发器选择 **标签推送事件**。因为日常开发中 **push** 操作比较频繁而且不是每个版本都需要构建, 所以只针对需要构建的版本打上 **Tag** 就好了。

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LTkyY2Q2ZmJkNmEzYjMyMzMucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

  gitlab添加 Webhook

  创建完使用 **test 按钮** 先测试下, 可能会出现下面的错误



### Jenkins job配置

![img](https://img2020.cnblogs.com/blog/907091/202007/907091-20200701223624718-1428314014.png)

点击上图中的“高级”，出现下图后，点击“Generate”，生成Secret token。

![img](https://img2020.cnblogs.com/blog/907091/202007/907091-20200701223720422-645966943.png)



### Jenkins报错：[Hook executed successfully but returned HTTP 403](https://www.cnblogs.com/chenglc/p/11174530.html)

jenkins配置gitlab的webhook，完成配置，测试结果显示 Hook executed successfully but returned HTTP 403

![img](https://img2018.cnblogs.com/blog/1155586/201907/1155586-20190712101545678-1103423976.png)

<font color='red'> 解决：</font>

#### 1、进入jenkins：

[**Manage Jenkins**](http://192.168.6.124:8888/manage)- >Configure Global Security -> 授权策略 -> Logged-in users can do anything （登录用户可以做任何事情） 点选 -> 匿名用户具有可读权限 点选

![img](https://img2018.cnblogs.com/blog/1155586/201907/1155586-20190712101915779-1578915428.png)

![img](https://img2018.cnblogs.com/blog/1155586/201907/1155586-20190712101949571-675353417.png)

#### 2、去掉跨站点请求伪造 点选 放开

[**Manage Jenkins**](http://192.168.6.124:8888/manage)- >Configure Global Security -> CSRF Protection（跨站请求伪造保护）

 

 ![img](https://img2018.cnblogs.com/blog/1155586/201907/1155586-20190712102234565-1992099827.png)

#### 3、去掉Gitlab enable authentication 点选 放开

系统管理 -> 系统设置 -> Enable authentication for '/project' end-point

 ![img](https://img2018.cnblogs.com/blog/1155586/201907/1155586-20190712102607231-511175613.png)





## 3.2 配置GitLab

### 配置账号的access token

创建账号的 **access token** , 用于 **Jenkins** 调用 **GitLab** 的 **API**

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LTg5ZTk3OTg1YTVhMjhlYzkucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

创建 access token

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xODM2NTM0LTA2NjVlZDlkNTcxMzRhOTgucG5nP2ltYWdlTW9ncjIvYXV0by1vcmllbnQvc3RyaXB8aW1hZ2VWaWV3Mi8yL3cvMTIwMC9mb3JtYXQvd2VicA?x-oss-process=image/format,png)

获取 access token



### Gitlab配置webhook

![img](https://img2020.cnblogs.com/blog/907091/202007/907091-20200701223745917-1335362327.png)

### gitlab使用webhook向jenkins发送请求，报错 Requests to the local network are not allowed



gitlab 10.6 版本以后为了安全，不允许向本地网络发送webhook请求，如果想向本地网络发送webhook请求，则需要使用管理员帐号登录，默认管理员帐号是admin@example.com，密码就是你gitlab搭建好之后第一次输入的密码，登录之后， 点击Configure Gitlab ，如下图所示



![img](https://img-blog.csdn.net/20180621100639420?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1a2FuZ2thbmcxaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

即可进入Admin area，在Admin area中，在settings标签下面，找到OutBound Request，勾选上Allow requests to the local network from hooks and services ，保存更改即可解决问题

![img](https://img-blog.csdn.net/20180621100942473?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1a2FuZ2thbmcxaGFv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)