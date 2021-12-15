- [Velero官网](https://velero.io/docs/v1.5/contributions/minio/)
- [基于Velero的集群备份与迁移技术](https://www.tangyuecan.com/2021/02/03/%e5%9f%ba%e4%ba%8evelero%e7%9a%84%e9%9b%86%e7%be%a4%e5%a4%87%e4%bb%bd%e4%b8%8e%e8%bf%81%e7%a7%bb%e6%8a%80%e6%9c%af/)



# 一、原始技术选型

首先是集群必须要使用至少一项备份技术，一方面基于NFS存储在小规模集群下的应用已经可以得到比较稳定表现。但是存储终归是在单一节点上，一旦出现硬盘问题或者其他软硬件问题便会导致数据丢失。即便是之前我们使用的各种分布式存储技术都可以做到数据冗余备份，一定程度上数据还要安全一点。特别是数据存储在本地服务器时，数据安全性便是首要需要解决的问题。

基于这些考虑目前的各种云原生技术的发展，在K8S集群技术体系下已经有很多备份技术，比较突出的有两个：

1. **velero**：这是一套异地备份方案具备直接增量同步任意PVC存储卷的能力，可以按照服务的PVC实时进行增量备份，并且可以集成`S3`对象存储，可以与云平台同步。
2. **Portworx**：这也是一套异地备份方案，相对于`velero`，他更为强大实现了容器服务与存储都作为容器存在，不光是备份可以轻易实现，恢复与迁移都很简单，也支持`S3`对象存储协议。

其中两者有一个核心的不同，`protworx`为了去实现将存储、配置、服务全部做为自定义容器的一部分进行控制，虽然功能上十分强大，但是也导致其后端支持要求比较高，一方面需要云供应商提供CSI服务，同时也需要本地的可拓展硬盘进行支持，使其过于依赖云能力。我们的备份技术选型不仅仅是公司内部使用同时也是我们技术输出的一部分。这一块会作为我们的项目方案进行实施，基于这些考虑反倒认为功能上没有那么强悍的`velero`是比较不错的选型。

一方面`velero`后端存储可以使用`S3`协议的对象存储服务器，这一块阿里或者华为都有提供可以直接使用，并且开源社区也存在像`Minio`这样的兼容`S3`协议的对象存储项目可以本地化，使得无论是基于云还是不适用云我们都可以灵活应用；另一方面`velero`在存储PV快照时需要CSI服务作为存储后端，但是这一块其实可以关闭我们可以使用`Restic`组件将PV数据作为资源的一部分存储到对象存储服务器，这样一来后端仅仅通过`S3`对象存储服务便可以实现备份与还原；最后由于`S3`存储是独立的终端服务器，这样一个不同集群其实完全可以使用同一个对象存储服务器这样一来跨集群迁移（包括PV数据）本质上就是一次备份与一次恢复，这样可以极大的降低迁移工作量。

# 二、部署安装（集群服务端）

这里简直是一个大坑，虽然`velero`项目知名度很高，但是可以看出实际使用者相对较少。毕竟行业绝大部分都没有使用`K8S`，即便是使用了很少去考虑备份的，大部分都是基于云直接做云硬盘快照。这样一来导致这一块的中文文档相当稀少，即使是英文文档也不算详细官方文档也没有系统性的说明，所以一下内容可以说是我自己在测试环境上各种尝试得到的。我们是基于`Rancher`进行部署，所以可以基于`Rancher`的商店进行部署，本质上与`K8S`的原生`API`没有什么区别。

## 2.1 配置商场的helm地址

在应用商店设置里面添加一个新的商店，名称可以自行随意写，`URL`地址使用 https://vmware-tanzu.github.io/helm-charts，分支为master。

## 2.2 配置S3的AK秘钥

对象存储访问需要使用`AK`对作为秘钥，这里我们最好的实现方式是在K8S集群之中选创建好一个密文，一定要在`velero`命名空间下，密文的名称最好为`cloud-credentials`我之前试过不同名字有失败。密文的内容是一个`Key/Value`对，`key`固定填写`cloud`，`value`里面就是`AK`对了，内容如下：

```yaml
[default]
aws_access_key_id=<你的AccessKeyId>
aws_secret_access_key=<你的AccessKey>
```

## 2.3 配置velero应答设置

启用商店之后找到新建的`velero`项目，点击创建可以进入到项目的配置页面，最重要的是下面的应答配置，这里面的东西本质上与`helm`使用`yaml`一样，这里为了方便我们不适用表单直接编辑`yaml`设置即可，具体的配置如下：

```yaml
image: 
  repository: velero/velero #这里是使用的Docker镜像（来源是docker hub）
  tag: v1.4.3 #这里是镜像的版本，最新的1.5.3有一些问题，所以选择上一个版本
configuration: 
  backupStorageLocation: 
    bucket: "xxxxxx" #这里是你S3对象存储的bucket名称
    config: 
      region: "xxxxxx" #这里是S3对象存储的区域
      s3ForcePathStyle: "false" #这个有点复杂，如果设置为true的话访问地址便是http://<endpoint>/<bucket>反之则是http://<bucket>.<endpoint>
      s3Url: "xxxxxx" #这里就是对象存储的endpoint，但是必须加http或者https
    name: "default" #这个便无所谓了，就是这个存储卷的名字，自己可以随意
  provider: "aws" #这里是云服务的供应商，因为aws亚马孙云就是S3的提出者，所以我们使用aws，这样可以接入任何兼容S3的对象存储
credentials: 
  existingSecret: "cloud-credentials" #这里是k8s的秘钥配置，意思是使用我们集群之中的已经存在的秘钥
  useSecret: true #是否使用秘钥，S3的秘钥就是AK对，不用的话就是公共读写的bucket
metrics: 
  enabled: true #是否启用metrics，这里是K8S的一个监控服务，可以开也可以不开
  serviceMonitor: 
    enabled: true #是否启用serviceMonitor，这里是监控服务的集群通信地址，可以开也可以不开
snapshotsEnabled: false #是否启用PV快照，因为后端我们没有提供CSI服务，所以不启用，PV的备份我们使用Restic实现
initContainers: #这里比较重要，因为velero可以支持各种插件拓展，在启动核心镜像之前需要初始化插件镜像，这里是去设置需要初始化的镜像数组
  - name: "velero-plugin-for-aws" #velero 1.1.0版本之后把aws组件从核心里面移除了，所以只能依靠插件实现
    image: "velero/velero-plugin-for-aws:v1.1.0" #插件版本可以在GITHUB上找到与velero的版本对应表
    volumeMounts: 
      - mountPath: "/target" #插件的容器挂载卷（固定配置）
        name: "plugins" #挂载卷的名字（固定配置）
deployRestic: true #这里就是比较重要的部署并且启用Restic，必须有他不然我们无法备份PV
```

# 三、部署安装（客户端）

## 3.1 下载安装客户端二进制文件

没错你不能直接操作服务器使用，你还需要一个客户端。这里比较扯淡，因为客户端使用的是和服务端一样的[项目](https://github.com/vmware-tanzu/velero)很具有迷惑性，首先便是下载，我们不需要拿着源码编译可以直接下载`Relese`版本（`Linux AMD64`[下载地址](https://github.com/vmware-tanzu/velero/releases/download/v1.4.3/velero-v1.4.3-linux-amd64.tar.gz)）。

安装就很简单了，就只有一个可执行文件，其他什么运行库与依赖都没有，只需要把解压之后的`velero`文件放置到系统环境变量目录下就可以了，比如说`/usr/local/bin`下面

## 3.2 配置K8S授权

这里是标准的`K8S`方案，每一个`K8S`集群都可以提供一个`kubeconfig`文件，这个文件里面涉及集群的授权、验证、控制等等一堆东西。简而言之只要有了这个文件，理论上便可以对集群做任何事情。

将自己`K8S`的`kubeconfig`文件下载到任何一台`linux`服务器上，在用户根目录（比如`root`用户就是`/root`）下创建隐藏文件夹`.kube`，然后在这个文件夹下面放一个`config`文件把`kubeconfig`里面的东西写进去就可以了（如果是`root`用户，那么完整地址就是`/root/.kube/config`）

# 四、使用说明

基本上没有什么文档把这个东西的使用说清楚了的，都他妈的是在一个非常具体的场景之下之下怎么用，没有哪里在系统的介绍，甚至官方文档也是仅仅只有几个例子，可能他们认为我这个设计一用就会不需要文档，操，大部分确实是我自己试出来。

首先一旦服务端部署完成之后我们之后几乎所有的备份操作全部都是在客户端上面去做，服务端只负责执行偶尔上去看看日志什么的。客户端就是一个`cli`工具，就我目前使用来看，它的命令分为三个级别，分别是**对象级**、**操作级**、**配置级**。

## 4.1 对象级指令

这部分意思是你需要操作的是什么类型的对象，目前在`velero`里面有三种对象类型：

- **backup**：很明显这个是备份的对象类型
- **restore**：很明显这个是恢复的对象类型
- **schedule**：很明显这个是定时规则对象，这个类型本质上周期性的备份对象类型

## 4.2 操作级指令

这部分意思是你要干什么事，指令只有四个比较简单分别为：

- **create**：创建一个对象，后面跟对象名字
- **delete**：删除一个对象，后面跟对象名字
- **get**：获取对象列表
- **describe**：获取一个对象详情，后面跟对象名字

## 4.3 参数级指令

这部分的指令就比较复杂，意思是通过参数去限制你操作行为的范围、条件等等。这里我只列一下我们会使用到的参数

- **-–include-namespaces**：这里是指你的操作应用命名空间的范围，默认情况下操作会影响整个集群，如果指定了这个参数那么操作行为就仅限于被指定的命名空间之中，当然这里可以指定多个命名空间（用逗号隔开）。
- **-–include-resources**：这里是吃你的操作涉及到哪些K8S资源，比如说`configmaps`（配置映射）、`secrets`（秘文），如果不指定那就会包含所有资源，一旦指定那么只会包含指定的资源，当也可以指定多种不同的资源（用逗号隔开）。
- **–-exclude-namespaces**：与-–include-namespaces正好相反，是在全局的基础上不包含某些命名空间。
- **–-exclude-resources**：与-–include-resources相反，是在所有资源类型的基础上不包含某些资源类型。
- **–schedule**：这个比较好理解就是schedule表达式，表明的是周期执行策略如`--schedule="0 1 * * *"`（每日1点执行）或者是`--schedule="@every 5h“`（每5小时执行）
- **–ttl**：这个时对象的存活时间，一旦超过这个时间之后对象会被自动删除，如`--ttl=72h`（保存72小时）`--ttl=1m`（保存一分钟）
- **–from-backup：**这个是最简单的，只在执行恢复时使用，指定从哪一个备份进行恢复

## 4.4 命令行举例说明

命令行组成其实就是由上面的三个级别指令组装出来的，常规的命令如下：

```bash
velero <对象级指令> <操作级指令> <参数级指令>
```

这个结构就是他命令行的组成了，所有操作行为都是这个一套东西下面可以举几个例子：

\#对象级是backup；操作级create mybackup创建一个叫mybackup的对象；参数级是--include-namespaces=mynamespace说明范围是mynamespace这个命名空间

```bash
#对象级是backup；操作级create mybackup创建一个叫mybackup的对象；参数级是--include-namespaces=mynamespace说明范围是mynamespace这个命名空间
velero backup create mybackup --include-namespaces=mynamespace 

#如果是需要操作多个命名空间也是一样
velero backup create mybackup --include-namespaces=mynamespaceA,mynamespaceB

#如果是多个参数级也是没有问题的，创建一小时后自动删除的备份
velero backup create mybackup --include-namespaces=mynamespace --ttl=1h

#对象级是backup；操作级是get；这里就是获取备份列表，但是他的后面就不能再加参数级
velero backup get

#对象级是backup；操作级是describe mybackup；这里就是获取mybackup这个备份的详情信息
velero backup describe mybackup

#对象级是backup；操作级是delete mybackup；这里就是删除mybackup这个存储了
velero backup delete mybackup

#如果是恢复的话也是一样的，只是恢复的创建指令后面不需要跟名字，需要补一个参数--from-backup从某一个备份上恢复，如果备份有多个命名空间也可以只恢复其中一个
velero restore create --from-backup=mybackup --include-namespaces=mynamespace

#周期性备份的话与backup对象的用法是一样的，对象级是schedule；操作级是create myschedule；后面是参数，每天1点创建一个72小时删除的mynamespace备份
velero schedule create myschedule --schedule="0 1 * * *" --ttl 72h --include-namespaces=mynamespace
```

### 4.5 PVC存储备份

这个必须要单独拿出来说，因为我们没有启用快照，毕竟快照需要使用云服务提供的插件我目前知道国内只有阿里云有其他统统不行。所以要想备份PV里面存储的数据那就只有使用`Restic`，不过这个加密压缩技术本身也是一个[开源项目](https://github.com/restic/restic)有兴趣可以去看看。

如果我们想要去备份`PVC`那么就必须告诉`velero`我们需要备份哪些`PVC`，在`Restic`里面目前只能做到对`pod`所挂载的PVC进行备份，没有错，不能直接对Server进行备份而是对`pod`。具体的方式不算复杂，只需要给需要`PVC`备份的`pod`打一个注释就可以了。如果是原生的`K8S`的话可以直接使用`kubectl`指定，`rancher`管理的集群也可以直接对`Server`打注释最后这个注释也会进入到`pod`；需要定义的注释是：

```bash
backup.velero.io/backup-volumes=<挂载卷名字>,<挂载卷名字>,<挂载卷名字>
```



当然可以上一个卷也可以上多个卷，原生`kubectl`的添加注释的命令为：

```bash
kubectl -n <命名空间> annotate pod/<pod名字> backup.velero.io/backup-volumes=<挂载卷名字>,<挂载卷名字>
```



一旦`pod`里面有了这个注释，后续直接备份的时候`velero`会自动去调用`Restic`获取`PVC`里面的数据进行压缩之后写入到对象存储里面去，这个东西还是比较高级的，最终存储的一大堆块文件，可以添加新的块实现增量备份。

# 五、特别注意

- 创建备份时如果不使用命名空间限制那么便会备份整个集群，甚至是K8S里面的核心组件，千万不要这么干。
- 创建备份的时候备份文件便会自动存储到`OSS`，删除也会删除。
- 恢复时如果本地`PVC`存在则不会去恢复`PVC`数据。
- 周期性定时对象一旦删除便会停止周期执行。
- 任何情况下不要去手动删除`OSS`上面的`restic`文件夹，这里面是所有结构文件，有变动会出现问题。