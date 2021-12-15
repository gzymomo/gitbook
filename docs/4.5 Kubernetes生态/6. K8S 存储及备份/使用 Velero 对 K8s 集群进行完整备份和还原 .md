- [使用 Velero 对 K8s 集群进行完整备份和还原](https://mp.weixin.qq.com/s/CzfbBywuYCL24pODKnm5fA)

## 导语

本文介绍使用青云 QingStor 作为 Velero 的后端存储，将 KubeSphere K8s 集群的资源进行备份，并在模拟灾难后快速从备份还原出应用。

KubeSphere 是在 Kubernetes 之上构建的面向云原生应用的容器混合云，支持多云与多集群管理，提供全栈的 IT 自动化运维的能力，简化企业的  DevOps 工作流。KubeSphere 提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。

Velero（以前称为 Heptio Ark）是一个开源工具，可以安全地备份和还原，执行灾难恢复以及迁移 Kubernetes 集群资源和持久卷。Velero 可以部署在 KubeSphere 集群或自建 Kubenetes 集群中。Velero 可用于：

- 备份集群资源并在丢失的情况下进行还原
- 将集群资源迁移到其他集群
- 将生产集群资源复制到开发和测试集群

Velero 包含运行在集群中的服务端软件和一个命令行的客户端。

我们将在 KubeSphere 集群的 wordpress 项目（命名空间）中部署一个 wordpress 的应用，我们先使用 Velero  备份这个命名空间下的应用和数据到 QingStor，然后删除这个项目（命名空间），最后从 QingStor  恢复应用和数据到这个命名空间。下文为具体的讲解。

## 实验环境和前提条件

- 青云容器引擎 QKE 上的 KubeSphere，已配置集群可正常使用 DNS 和访问互联网
- 已开通青云 QingStor 服务

## 创建 QingStor bucket

登录 QingCloud 控制台，进入“产品与服务”→“存储服务”→“对象存储”，点击“创建 Bucket”按钮。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEX4GusR4oHoYES0xo3BEvqPiaibCffVARI0sQ7oYNHrW8YlV1ypj0ofFa0DyUa43BecqONWnDPYjjnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注意这里“Url (API 访问用)”中的 pek3b ，这是 bucket 所在的区域，后面我们将会用到（也可以在创建 bucket 之前点击页面上部 banner 切换区域）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEX4GusR4oHoYES0xo3BEvqPhrtWfWa3y8Uy4k1oSfvVymbxvBqS1vic4SxUQwS2FPQmWHFZ64tauibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 配置 API 秘钥

登录 QingCloud 控制台，进入“产品与服务”→“访问与授权”→“API 秘钥”，点击“创建”按钮。

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEX4GusR4oHoYES0xo3BEvqPic89uoic7vaen4mKPMkk2bM05xe7B5QAIg5EGib2TxIlPTL7azPEmeb0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

创建完成之后会提示下载，在弹出的“下载密钥”窗口，点击下载按钮，将密钥文件：access_key.csv 下载到本地。access_key.csv 文件包含了 access key 和 secret key，例如:

```
qy_access_key_id: 'AJFIOJFEAFDFSLGTRFDD'
qy_secret_access_key: '7902jfCjlkjsJLJIOLK32ljkYsCfzzpPAMNKo9FW'
```

其中:

AJFIOJFEAFDFSLGTRFDD --------  access key id, 7902jfCjlkjsJLJIOLK32ljkYsCfzzpPAMNKo9FW -------  secret access key

## 安装 Velero

### 安装 Velero 客户端

先下载安装 Velero 的客户端工具，然后通过客户端命令在集群中安装服务端软件：

- macOS - HomeBrew

```
brew install velero
```

- Linux - 直接从网站下载对应操作系统的客户端，解压缩后仅一个执行文件。

```
tar -xvf <RELEASE-TARBALL-NAME>.tar.gz
```

然后将解压缩出来的 Velero 可执行文件移动到 $PATH 所指向的路径。

- Windows - Chocolatey

```
choco install velero
```

### 安装服务端软件到集群中:

- 配置 kubectl 访问 KubeSphere 集群

Velero 命令默认读取 kubectl 配置的集群上下文，所以我们先配置 kubectl 对 KubeSphere 集群的访问 具体请参阅 QKE 文档。

小提示：Velero 也支持--kubeconfig 参数和 kubeconfig 环境变量。

- 准备秘钥文件

在当前目录建立一个空白的文本文件，然后编辑内容为如下：

```
[default]
aws_access_key_id=<access key id>
aws_secret_access_key=<secret access key>
```

替换和为之前步骤中得到的对应 access key id 和 secret access key。（注：因为 QingStor 是 AWS S3 兼容对象存储，所以我们将使用 AWS 的 velero 存储后端插件）

- 执行安装程序

在当前目录执行如下命令安装服务端软件，其中 <qingstor_bucket_name> 为之前步骤中创建的 bucket 名，<bucket_region> 为之前创建 bucket 步骤中的区域（例如 pek3b）。

```
export S3_CONFIG_BUCKET=<qingstor_bucket_name>
export S3_CONFIG_REGION=<bucket_region>
export S3_CONFIG_S3URL=https://s3.${S3_CONFIG_REGION}.qingstor.com
velero install --provider aws --bucket ${S3_CONFIG_BUCKET} --secret-file ./credentials-velero --use-volume-snapshots=true --use-restic --backup-location-config region=${S5_CONFIG_REGION},s3ForcePathStyle="true",s3Url=${S3_CONFIG_S3URL} --prefix velero --plugins=velero/velero-plugin-for-aws:v1.2.0,velero/velero-plugin-for-csi:v0.1.2 --features=EnableCSI
```

参数说明：

- --provider: 声明使用的 Velero 插件类型
- --secret-file: 提供访问 QingStor 的秘钥
- --use-volume-snapshots=true: 是否使用 Kubernetes 的卷快照功能
- --use-restic: 使用开源免费备份工具 restic 备份和还原持久卷数据
- --backup-location-config: 备份 bucket 的一些配置
- --prefix: 备份存储在 QingStor bucket 中的前缀，相当于指定备份存放在 bucket 的目录
- --plugins: 使用的 Velero 插件，我们使用 AWS S3 兼容插件和 CSI 插件
- --feature=EnableCSI: 安装 Velero 中 CSI 的支持

Velero 支持对持久卷以建立快照的方式备份，也支持使用 restic 将持久卷里的文件备份到对象存储，本文的实验中使用 restic 来备份到 QingStor。

## 部署 wordpress 应用

```
kubectl create namespace wordpress # 也可以在 Kubesphere 控制台创建名为 wordpress 的项目
kubectl apply -f mysql-deployment.yaml
kubectl apply -f wordpress-deployment.yaml
```

文后提供示例 yaml 文件的下载，下载后请根据自己的 QKE 环境修改 `wordpress-deployment-qke.yaml` 中 wordpress 服务的负载均衡器配置的 Annotation，详情可参考**青云负载均衡器配置指南**[1]。

等待应用起来，我们可以进入浏览器访问 wordpress 的地址执行 wordpress 的安装过程，并发布一篇文章。

## 执行备份

```
velero backup create wordpress-backup --include-namespaces wordpress --default-volumes-to-restic
Backup request "wordpress-backup" submitted successfully.
Run `velero backup describe wordpress-backup` or `velero backup logs wordpress-backup` for more details.
```

`--default-volumes-to-restic` 参数指示使用 restic 备份持久卷到 QingStor。

上面的命令请求创建一个对项目（命名空间）的备份，备份请求发送之后可以用命令查看备份状态，等到 STATUS 列变为 Completed 表示备份完成。

```
velero backup get
NAME                       STATUS            ERRORS   WARNINGS   CREATED                         EXPIRES   STORAGE LOCATION   SELECTOR
wordpress-backup           Completed         0        0          2021-08-23 13:47:01 +0800 CST   29d       default            <none>
```

在 QingStor 上可以看到备份。

```
s3cmd ls s3://shaofeng66-velero-test/velero/backups/wordpress-backup/
2021-08-23 05:47      2187   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/velero-backup.json
2021-08-23 05:47        29   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-csi-volumesnapshotcontents.json.gz
2021-08-23 05:47        29   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-csi-volumesnapshots.json.gz
2021-08-23 05:47      5146   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-logs.gz
2021-08-23 05:47      1185   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-podvolumebackups.json.gz
2021-08-23 05:47       522   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-resource-list.json.gz
2021-08-23 05:47        29   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup-volumesnapshots.json.gz
2021-08-23 05:47     16044   s3://shaofeng66-velero-test/velero/backups/wordpress-backup/wordpress-backup.tar.gz
```

## 模拟灾难

完成备份后，我们删除应用所在的项目（命名空间）以模拟生产环境发生灾难或运维错误导致应用失败。

```
kubectl delete namespace wordpress
```

删除以后再尝试使用浏览器访问应用，提示无法访问：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/u5Pibv7AcsEX4GusR4oHoYES0xo3BEvqPL46UYPuPy0lXBMQLzCrhNYY4ibj1jMQf0Ecm5hNV6guyYBXLBba2KVA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 执行恢复

这时候我们可以用一条命令，使用 Velero 从 QingStor 恢复应用和数据:

```
velero --kubeconfig kubeconfig.yaml restore create --from-backup wordpress-backup
Restore request "wordpress-backup-20210823204907" submitted successfully.
Run `velero restore describe wordpress-backup-20210823204907` or `velero restore logs wordpress-backup-20210823204907` for more details.
```

此命令请求从一个备份恢复，恢复请求发送后可以用命令查看恢复状态，等到 STATUS 变为 Completed 并且 ERRORS 个数为 0，则恢复完成：

```
velero restore get
NAME                                      BACKUP                     STATUS            STARTED                         COMPLETED                       ERRORS   WARNINGS   CREATED                         SELECTOR
wordpress-backup-20210823134946           wordpress-backup           Completed         2021-08-23 13:49:47 +0800 CST   2021-08-23 13:51:30 +0800 CST   0        1          2021-08-23 13:49:47 +0800 CST   <none> 
```

此时再尝试使用浏览器访问应用，可以看到应用与数据都已经恢复：

![图片](https://mmbiz.qpic.cn/mmbiz_png/u5Pibv7AcsEX4GusR4oHoYES0xo3BEvqPMBcZU2dQ6HkJz7jib6CsHKlEnYpxd64p1fAP9iaICWnGNHGnskGOsO9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 总结

在本文中，我们简单介绍了 Kubernetes 集群资源备份工具 Velero，展示了如何配置青云 QingStor 来作为 Velero 的后端存储，并成功实践了对 KubeSphere 中应用和数据的备份和还原操作。

## 参考

- **Velero 官网**[2]
- **KubeSphere 官网**[3]
- **使用 minio 体验 velero 的备份与恢复**[4]
- **青云 QKE 文档**[5]
- **部署 wordpress 的 yaml 文件下载**[6]

## 作者简介

Felix, 80 后研发工程师，现任上海骥步科技有限公司高级软件研发工程师，骥步科技 K8s  商业化灾备产品“银数多云数据管家”核心研发人员。曾供职过蓝色巨人 IBM  近十年，也混迹过互联网创业公司。有十余年存储研发经验，对存储产品、容器技术、Velero 等有深入的研究和理解。

### 脚注

[1]青云负载均衡器配置指南: *https://github.com/yunify/qingcloud-cloud-controller-manager/blob/master/docs/configure.md#%E6%89%8B%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%85%AC%E7%BD%91ip*[2]Velero 官网: *https://velero.io*[3]KubeSphere 官网: *https://kubesphere.io/*[4]使用 minio 体验 velero 的备份与恢复: *https://velero.io/docs/v1.6/contributions/minio/*[5]青云 QKE 文档: *https://docs.qingcloud.com/product/container/qke/*[6]部署 wordpress 的 yaml 文件下载: *https://github.com/jibutech/examples/tree/master/mysql-wordpress-pd*