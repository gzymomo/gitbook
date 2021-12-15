[TOC]

使用Velero对k8s进行备份与恢复（带持久化卷）

## 1.什么是Velero

Velero 是一个云原生的灾难恢复和迁移工具，它本身也是开源的, 采用 Go 语言编写，可以安全的备份、恢复和迁移Kubernetes集群资源和持久卷。

Velero 是西班牙语，意思是帆船，非常符合 Kubernetes 社区的命名风格。Velero 的开发公司 Heptio，之前已被 VMware 收购，其创始人2014就职于Google，当时被认为是 Kubernetes 核心成员。

Velero 是一种云原生的Kubernetes优化方法，支持标准的K8S集群，既可以是私有云平台也可以是公有云。除了灾备之外它还能做资源移转，支持把容器应用从一个集群迁移到另一个集群。

Heptio Velero ( 以前的名字为 ARK) 是一款用于 Kubernetes 集群资源和持久存储卷（PV）的备份、迁移以及灾难恢复等的开源工具。

使用velero可以对集群进行备份和恢复，降低集群DR造成的影响。velero的基本原理就是将集群的数据备份到对象存储中，在恢复的时候将数据从对象存储中拉取下来。可以从官方文档查看可接收的对象存储，本地存储可以使用Minio。下面演示minio作为对象存储使用velero将k8s集群备份、迁移。

## 2.下载velero
1.在可以访问到需要备份的k8s集群的机器上下载并解压velero,并将velero命令添加到linux
```bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.5.4/velero-v1.5.4-linux-amd64.tar.gz
tar -zxvf velero-v1.5.4-linux-amd64.tar.gz
cp velero-v1.5.4-linux-amd64/velero /usr/local/bin
```
2.创建好minio需要使用的认证,这里以velero自带的测试minio部署文件为例
```bash
cd velero-v1.5.4-linux-amd64
kubectl apply -f examples/minio/00-minio-deployment.yaml
vi credentials-velero
[default]
aws_access_key_id = minio
aws_secret_access_key = minio123
```

3.在需要备份的k8s集群上安装velero服务端
```bash
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.0.0 \
    --bucket velero \
    --secret-file ./credentials-velero \
    --use-restic \
    --use-volume-snapshots=false \
    --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio.velero.svc:9000 
```
4.创建任意带持久化存储卷的服务进行备份测试（可自行使用helm创建）这里以备份backup-test命名空间下所有资源为例

```bash
velero backup create test-volum --include-namespaces=backup-test --default-volumes-to-restic
```
5.查看备份状态

```bash
velero backup get|grep test-volum
```
结果如下表示已备份完成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210410110858271.png)
更详细内容可使用**describe**命令

```bash
velero backup describe test-volum --details
```
![备份详细状态](https://img-blog.csdnimg.cn/20210410111425541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZpenpYMQ==,size_16,color_FFFFFF,t_70)


6.删除命名空间,并使用备份恢复
```bash
kubectl delete ns backup-test
velero restore create --from-backup test-volum
```
7.若要每天凌晨定时进行备份、设置过期时间为72小时并使用其他的BackupStorageLocation
```bash
  velero schedule create all-daily-restic --schedule="@midnight" --exclude-namespaces kube-system,back-test,velero --default-volumes-to-restic --ttl=72h --storage-location fizz-minio-backup
```
8.velero添加其他的backuplocation可以使用如下yaml进行创建

```yaml
apiVersion: velero.io/v1
kind: BackupStorageLocation
metadata:
  name: fizz-minio-backup
  namespace: velero
spec:
  backupSyncPeriod: 2m0s
  provider: aws
  objectStorage:
    bucket: velero
  config:
    region: minio
    s3ForcePathStyle: "true"
    s3Url: http://xxx.test.com:33000
```
9.若要迁移到其他集群，需要相同方式在迁移集群部署velero，指定相同的backuplocation执行恢复命令即可