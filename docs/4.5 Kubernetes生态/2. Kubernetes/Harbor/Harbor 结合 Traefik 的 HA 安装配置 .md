Harbor 是一个 CNCF 基金会托管的开源的可信的云原生 docker registry 项目，可以用于存储、签名、扫描镜像内容，Harbor  通过添加一些常用的功能如安全性、身份权限管理等来扩展 docker registry 项目，此外还支持在 registry  之间复制镜像，还提供更加高级的安全功能，如用户管理、访问控制和活动审计等，在新版本中还添加了 Helm 仓库托管的支持。

Harbor 最核心的功能就是给 `docker registry` 添加上一层权限保护的功能，要实现这个功能，就需要我们在使用 docker login、pull、push 等命令的时候进行拦截，先进行一些权限相关的校验，再进行操作，其实这一系列的操作 `docker registry v2` 就已经为我们提供了支持，v2 集成了一个安全认证的功能，将安全认证暴露给外部服务，让外部服务去实现。

## Harbor 认证原理

上面我们说了 `docker registry v2` 将安全认证暴露给了外部服务使用，那么是怎样暴露的呢？我们在命令行中输入 `docker login https://registry.qikqiak.com` 为例来为大家说明下认证流程：

1. docker client 接收到用户输入的 docker login 命令，将命令转化为调用 engine api 的 RegistryLogin 方法
2. 在 RegistryLogin 方法中通过 http 调用 registry 服务中的 auth 方法
3. 因为我们这里使用的是 v2 版本的服务，所以会调用 loginV2 方法，在 loginV2 方法中会进行 /v2/ 接口调用，该接口会对请求进行认证
4. 此时的请求中并没有包含 token 信息，认证会失败，返回 401 错误，同时会在 header 中返回去哪里请求认证的服务器地址
5. registry client 端收到上面的返回结果后，便会去返回的认证服务器那里进行认证请求，向认证服务器发送的请求的 header 中包含有加密的用户名和密码
6. 认证服务器从 header 中获取到加密的用户名和密码，这个时候就可以结合实际的认证系统进行认证了，比如从数据库中查询用户认证信息或者对接 ldap 服务进行认证校验
7. 认证成功后，会返回一个 token 信息，client 端会拿着返回的 token 再次向 registry 服务发送请求，这次需要带上得到的 token，请求验证成功，返回状态码就是200了
8. docker client 端接收到返回的200状态码，说明操作成功，在控制台上打印 Login Succeeded 的信息 至此，整个登录过程完成，整个过程可以用下面的流程图来说明：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy7QFyFfpZIhZ2p3hqU3NShPZIV89gPR36upCeick5ibTia6UvtIEM6Fs9Ow/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

要完成上面的登录认证过程有两个关键点需要注意：怎样让 registry 服务知道服务认证地址？我们自己提供的认证服务生成的 token 为什么 registry 就能够识别？

对于第一个问题，比较好解决，registry 服务本身就提供了一个配置文件，可以在启动 registry 服务的配置文件中指定上认证服务地址即可，其中有如下这样的一段配置信息：

```
......
auth:
  token:
    realm: token-realm
    service: token-service
    issuer: registry-token-issuer
    rootcertbundle: /root/certs/bundle
......
```

其中 realm 就可以用来指定一个认证服务的地址，下面我们可以看到 Harbor 中该配置的内容。

> 关于 registry 的配置，可以参考官方文档：https://docs.docker.com/registry/configuration/

第二个问题，就是 registry 怎么能够识别我们返回的 token 文件？如果按照 registry 的要求生成一个 token，是不是 registry  就可以识别了？所以我们需要在我们的认证服务器中按照 registry 的要求生成 token，而不是随便乱生成。那么要怎么生成呢？我们可以在  docker registry 的源码中可以看到 token 是通过 `JWT（JSON Web Token）` 来实现的，所以我们按照要求生成一个 JWT 的 token 就可以了。

对 golang 熟悉的同学可以去 clone 下 Harbor 的代码查看下，Harbor 采用 beego 这个 web 开发框架，源码阅读起来不是特别困难。我们可以很容易的看到 Harbor 中关于上面我们讲解的认证服务部分的实现方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy7Foa22pKqCHjH4VSiagU8L5qBbHdyy09jz8GNPPFfHID2p4PfWAIleOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 安装

Harbor 涉及的组件比较多，我们可以使用 Helm 来安装一个高可用版本的 Harbor，也符合生产环境的部署方式。在安装高可用的版本之前，我们需要如下先决条件：

- Kubernetes 集群 1.10+ 版本
- Helm 2.8.0+ 版本
- 高可用的 Ingress 控制器
- 高可用的 PostgreSQL 9.6+（Harbor 不进行数据库 HA 的部署）
- 高可用的 Redis 服务（Harbor 不处理）
- 可以跨节点或外部对象存储共享的 PVC

Harbor 的大部分组件都是无状态的，所以我们可以简单增加 Pod 的副本，保证组件尽量分布到多个节点上即可，在存储层，需要我们自行提供高可用的  PostgreSQL、Redis 集群来存储应用数据，以及存储镜像和 Helm Chart 的 PVC 或对象存储。

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy764hJicoiaZgicDmUYe5ajdlL9vEdWCyCxyh8SbfFqexz1dABJ6b4OnTng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

首先添加 Chart 仓库地址：

```
# 添加 Chart 仓库
helm repo add harbor https://helm.goharbor.io
# 更新
helm repo update
# 拉取1.6.2版本并解压
helm pull harbor/harbor --untar --version 1.6.2
```

在安装 Harbor 的时候有很多可以配置的参数，可以在 harbor-helm 项目上进行查看，在安装的时候我们可以通过 `--set` 指定参数或者 `values.yaml` 直接编辑 Values 文件即可：

- Ingress 配置通过 `expose.ingress.hosts.core` 和 `expose.ingress.hosts.notary`
- 外部 URL 通过配置 `externalURL`
- 外部 PostgreSQL 通过配置 `database.type` 为 `external`，然后补充上 `database.external` 的信息。需要我们手动创建3个空的数据：`Harbor core`、`Notary server` 以及 `Notary signer`，Harbor 会在启动时自动创建表结构
- 外部 Redis 通过配置 `redis.type` 为 `external`，并填充 `redis.external` 部分的信息。Harbor 在 2.1.0 版本中引入了 redis 的 `Sentinel` 模式，你可以通过配置 `sentinel_master_set` 来开启，host 地址可以设置为 `<host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>`。还可以参考文档https://community.pivotal.io/s/article/How-to-setup-HAProxy-and-Redis-Sentinel-for-automatic-failover-between-Redis-Master-and-Slave-servers 在 Redis 前面配置一个 HAProxy 来暴露单个入口点。
- 存储，默认情况下需要一个默认的 `StorageClass` 在 K8S 集群中来自动生成 PV，用来存储镜像、Charts 和任务日志。如果你想指定 `StorageClass`，可以通过 `persistence.persistentVolumeClaim.registry.storageClass`、`persistence.persistentVolumeClaim.chartmuseum.storageClass` 以及 `persistence.persistentVolumeClaim.jobservice.storageClass` 进行配置，另外还需要将 accessMode 设置为 `ReadWriteMany`，确保 PV 可以跨不同节点进行共享存储。此外我们还可以通过指定存在的 PVCs 来存储数据，可以通过 `existingClaim` 进行配置。如果你没有可以跨节点共享的 PVC，你可以使用外部存储来存储镜像和 Chart（外部存储支持：azure，gcs，s3 swift 和 oss），并将任务日志存储在数据库中。将设置为 `persistence.imageChartStorage.type` 为你要使用的值并填充相应部分并设置 `jobservice.jobLogger` 为 `database`
- 副本：通过设置 `portal.replicas`，`core.replicas`，`jobservice.replicas`，`registry.replicas`，`chartmuseum.replicas`，`notary.server.replicas` 和 `notary.signer.replicas` 为 n（n> = 2）

比如这里我们将主域名配置为 `harbor.k8s.local`，通过前面的 NFS 的 StorageClass 来提供存储（生产环境不建议使用 NFS），又因为前面我们在安装 GitLab 的时候就已经单独安装了  postgresql 和 reids 两个数据库，所以我们也可以配置 Harbor  使用这两个外置的数据库，这样可以降低资源的使用（我们可以认为这两个数据库都是 HA  模式）。但是使用外置的数据库我们需要提前手动创建数据库，比如我们这里使用的 GitLab 提供的数据库，则进入该 Pod 创建 `harbor`、`notary_server`、`notary_signer` 这3个数据库：

```
$ kubectl get pods -n kube-ops -l name=postgresql
NAME                          READY   STATUS    RESTARTS   AGE
postgresql-566846fd86-9kps9   1/1     Running   1          2d
$ kubectl exec -it postgresql-566846fd86-9kps9 /bin/bash -n kube-ops
root@postgresql-566846fd86-9kps9:/var/lib/postgresql# sudo su - postgres
postgres@postgresql-566846fd86-9kps9:~$ psql
psql (12.3 (Ubuntu 12.3-1.pgdg18.04+1))
Type "help" for help.

postgres=# CREATE DATABASE harbor OWNER postgres;  # 创建 harbor 数据库
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE harbor to postgres;  # 授权给 postgres 用户
GRANT
postgres=# GRANT ALL PRIVILEGES ON DATABASE harbor to gitlab;  # 授权给 gitlab 用户
GRANT
# Todo: 用同样的方式创建其他两个数据库：notary_server、notary_signer
......
postgres-# \q  # 退出
```

数据库准备过后，就可以使用我们自己定制的 values 文件来进行安装了，完整的定制的 values 文件如下所示：

```
# values-prod.yaml
externalURL: https://harbor.k8s.local
harborAdminPassword: Harbor12345
logLevel: debug

expose:
  type: ingress
  tls:
    enabled: true
  ingress:
    hosts:
      core: harbor.k8s.local
      notary: notary.k8s.local
    annotations:
      # 因为我们使用的 Traefik2.x 作为 Ingress 控制器
      kubernetes.io/ingress.class: traefik
      traefik.ingress.kubernetes.io/router.entrypoints: websecure
      traefik.ingress.kubernetes.io/router.tls: "true"

persistence:
  enabled: true
  resourcePolicy: "keep"
  persistentVolumeClaim:
    registry:
      # 如果需要做高可用，多个副本的组件则需要使用支持 ReadWriteMany 的后端
      # 这里我们使用nfs，生产环境不建议使用nfs
      storageClass: "nfs-storage"
      # 如果是高可用的，多个副本组件需要使用 ReadWriteMany，默认为 ReadWriteOnce
      accessMode: ReadWriteMany
      size: 5Gi
    chartmuseum:
      storageClass: "nfs-storage"
      accessMode: ReadWriteMany
      size: 5Gi
    jobservice:
      storageClass: "nfs-storage"
      accessMode: ReadWriteMany
      size: 1Gi
    trivy:
      storageClass: "nfs-storage"
      accessMode: ReadWriteMany
      size: 2Gi

database:
  type: external
  external:
    host: "postgresql.kube-ops.svc.cluster.local"
    port: "5432"
    username: "gitlab"
    password: "passw0rd"
    coreDatabase: "harbor"
    notaryServerDatabase: "notary_server"
    notarySignerDatabase: "notary_signer"

redis:
  type: external
  external:
    addr: "redis.kube-ops.svc.cluster.local:6379"

# 默认为一个副本，如果要做高可用，只需要设置为 replicas >= 2 即可
portal:
  replicas: 1
core:
  replicas: 1
jobservice:
  replicas: 1
registry:
  replicas: 1
chartmuseum:
  replicas: 1
trivy:
  replicas: 1
notary:
  server:
    replicas: 1
  signer:
    replicas: 1
```

由于我们这里使用的 Ingress 控制器是 traefik2.x 版本，在配置 Ingress 的时候，我们需要重新配置 `annotations`（如果你使用的是其他 Ingress 控制器，请参考具体的使用方式）。这些配置信息都是根据 Harbor 的 Chart 包默认的 values 值进行覆盖的，现在我们直接安装即可：

```
$ cd harbor
$ helm upgrade --install harbor . -f values-prod.yaml -n kube-ops
Release "harbor" does not exist. Installing it now.
NAME: harbor
LAST DEPLOYED: Sat May 29 16:22:13 2021
NAMESPACE: kube-ops
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Please wait for several minutes for Harbor deployment to complete.
Then you should be able to visit the Harbor portal at https://harbor.k8s.local
For more details, please visit https://github.com/goharbor/harbor
```

正常情况下隔一会儿就可以安装成功了：

```
$ helm ls -n kube-ops
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
harbor  kube-ops        1               2021-05-29 16:22:13.495394 +0800 CST    deployed        harbor-1.6.2    2.2.2
$ kubectl get pods -n kube-ops -l app=harbor
NAME                                           READY   STATUS    RESTARTS   AGE
harbor-harbor-chartmuseum-6996f677d4-dhkgd     1/1     Running   0          91s
harbor-harbor-core-84b8db479-rpwml             1/1     Running   0          91s
harbor-harbor-jobservice-5584dccd6-gpv79       1/1     Running   0          91s
harbor-harbor-notary-server-7d79b7d46d-dlgt7   1/1     Running   0          91s
harbor-harbor-notary-signer-69d8fdd476-7pt44   1/1     Running   0          91s
harbor-harbor-portal-559c4d4bfd-8x2pp          1/1     Running   0          91s
harbor-harbor-registry-758f67dbbb-nl729        2/2     Running   0          91s
harbor-harbor-trivy-0                          1/1     Running   0          50s
```

安装完成后，我们就可以将域名 `harbor.k8s.local` 解析到 Ingress Controller 所在的节点，我们这里使用的仍然是 Traefik，由于我们开启了  KubernetesIngress 支持的，所以我们只需要将域名解析到 Traefik 的 Pod  所在节点即可，然后就可以通过该域名在浏览器中访问了：

```
$ kubectl get ingress -n kube-ops
NAME                           CLASS    HOSTS              ADDRESS   PORTS     AGE
harbor-harbor-ingress          <none>   harbor.k8s.local             80, 443   115s
harbor-harbor-ingress-notary   <none>   notary.k8s.local             80, 443   115s
```

用户名使用默认的 admin，密码则是上面配置的默认 `Harbor12345`，需要注意的是要使用 https 进行访问，否则登录可能提示用户名或密码错误：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy7YrmGh6BdhJmMvvuEa6NKwoKDTcvjBOkDstk8wE2LoeqJlJibEUz2CDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

但是这里也需要注意的是，由于我们这里使用的 traefik2.x 版本的 Ingress 控制器，所以对于 Ingress 资源的支持不是很友好，由于我们添加了 `traefik.ingress.kubernetes.io/router.tls: "true"` 这个注解，导致我们的 http 服务又失效了，为了解决这个问题，我们这里手动来创建一个 http 版本的 Ingress 对象：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    meta.helm.sh/release-name: harbor
    meta.helm.sh/release-namespace: kube-ops
    traefik.ingress.kubernetes.io/router.entrypoints: web
    traefik.ingress.kubernetes.io/router.middlewares: kube-system-redirect-https@kubernetescrd
  labels:
    app: harbor
    app.kubernetes.io/managed-by: Helm
    chart: harbor
    heritage: Helm
    release: harbor
  name: harbor-harbor-ingress-http
  namespace: kube-ops
spec:
  rules:
  - host: harbor.k8s.local
    http:
      paths:
      - backend:
          serviceName: harbor-harbor-portal
          servicePort: 80
        path: /
        pathType: Prefix
      - backend:
          serviceName: harbor-harbor-core
          servicePort: 80
        path: /api
        pathType: Prefix
      - backend:
          serviceName: harbor-harbor-core
          servicePort: 80
        path: /service
        pathType: Prefix
      - backend:
          serviceName: harbor-harbor-core
          servicePort: 80
        path: /v2
        pathType: Prefix
      - backend:
          serviceName: harbor-harbor-core
          servicePort: 80
        path: /chartrepo
        pathType: Prefix
      - backend:
          serviceName: harbor-harbor-core
          servicePort: 80
        path: /c
        pathType: Prefix
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
    meta.helm.sh/release-name: harbor
    meta.helm.sh/release-namespace: kube-ops
    traefik.ingress.kubernetes.io/router.entrypoints: web
    traefik.ingress.kubernetes.io/router.middlewares: kube-system-redirect-https@kubernetescrd
  labels:
    app: harbor
    app.kubernetes.io/managed-by: Helm
    chart: harbor
    heritage: Helm
    release: harbor
  name: harbor-harbor-ingress-notary-http
  namespace: kube-ops
spec:
  rules:
  - host: notary.k8s.local
    http:
      paths:
      - backend:
          serviceName: harbor-harbor-notary-server
          servicePort: 4443
        path: /
        pathType: Prefix
```

为了让能够跳转到 https，我们还需要创建如下所示的一个 Middleware（如果你使用的是其他 Ingress 控制器，请参考具体的使用方式）：

```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: redirect-https
  namespace: kube-system
spec:
  redirectScheme:
    scheme: https
```

需要注意的是在 Ingress 的 annotations 中配置中间件的格式为 `<namespace>-redirect-https@kubernetescrd`。

登录过后即可进入 Harbor 的 Dashboard 页面：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy704riaibNanrfRNfLtZ4IevuFneovjkuInPErZjlLA33K0uN0Pkrz7tRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可以看到有很多功能，默认情况下会有一个名叫 `library` 的项目，该项目默认是公开访问权限的，进入项目可以看到里面还有 `Helm Chart` 包的管理，可以手动在这里上传，也可以对该项目里面的镜像进行一些其他配置。

## 推送镜像

现在我们来测试下使用 `docker cli` 来进行 `pull/push` 镜像，直接使用 `docker login` 命令登录：

```
$ docker login harbor.k8s.local
Username: admin
Password:
Error response from daemon: Get https://harbor.k8s.local/v2/: x509: certificate signed by unknown authority
```

可以看到会登录失败，这是因为在使用 `docker login` 登录的时候会使用 https 的服务，而我们这里是自签名的证书，所以就报错了，我们可以将使用到的 CA 证书文件复制到 `/etc/docker/certs.d/harbor.k8s.local` 目录下面来解决这个问题（如果该目录不存在，则创建它）。`ca.crt` 这个证书文件我们可以通过 Ingress 中使用的 Secret 资源对象来提供：

```
$ kubectl get secret harbor-harbor-ingress -n kube-ops -o yaml
apiVersion: v1
data:
  ca.crt: <ca.crt>
  tls.crt: <tls.crt>
  tls.key: <tls.key>
kind: Secret
metadata:
  ......
  name: harbor-harbor-ingress
  namespace: kube-ops
  resourceVersion: "450460"
  selfLink: /api/v1/namespaces/kube-ops/secrets/harbor-harbor-ingress
  uid: 0c44425c-8258-407a-a0a7-1c7e50d29404
type: kubernetes.io/tls
```

其中 data 区域中 `ca.crt` 对应的值就是我们需要证书，不过需要注意还需要做一个 base64 的解码，这样证书配置上以后就可以正常访问了。

不过由于上面的方法较为繁琐，所以一般情况下面我们在使用 `docker cli` 的时候是在 docker 启动参数后面添加一个 `--insecure-registry` 参数来忽略证书的校验的，在 docker 启动配置文件 `/usr/lib/systemd/system/docker.service` 中修改ExecStart的启动参数：

```
ExecStart=/usr/bin/dockerd --insecure-registry harbor.k8s.local
```

或者在 `Docker Daemon` 的配置文件中添加：

```
$ cat /etc/docker/daemon.json
{
  "insecure-registries" : [
    "harbor.k8s.local"
  ],
  "registry-mirrors" : [
    "https://ot2k4d59.mirror.aliyuncs.com/"
  ]
}
```

然后保存重启 docker，再使用 `docker cli` 就没有任何问题了：

```
$ docker login harbor.k8s.local
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

比如我们本地现在有一个名为 `busybox:1.28.4` 的镜像，现在我们想要将该镜像推送到我们的私有仓库中去，应该怎样操作呢？首先我们需要给该镜像重新打一个具有 `harbor.k8s.local` 前缀的镜像，然后推送的时候就可以识别到推送到哪个镜像仓库：

```
$ docker tag busybox:1.28.4 harbor.k8s.local/library/busybox:1.28.4
$ docker push harbor.k8s.local/library/busybox:1.28.4
The push refers to repository [harbor.k8s.local/library/busybox]
432b65032b94: Pushed
1.28.4: digest: sha256:74f634b1bc1bd74535d5209589734efbd44a25f4e2dc96d78784576a3eb5b335 size: 527
```

推送完成后，我们就可以在 Portal 页面上看到这个镜像的信息了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/z9BgVMEm7YuQmlc1fCBzZxgwdL5KBOy7X9VbQgeqojshgEDKRYD9aIfE4uiafxGoUoPIvkricqzDZ5WXg4v4iay3g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

镜像 push 成功，同样可以测试下 pull：

```
$ docker rmi harbor.k8s.local/library/busybox:1.28.4
Untagged: harbor.k8s.local/library/busybox:1.28.4
Untagged: harbor.k8s.local/library/busybox@sha256:74f634b1bc1bd74535d5209589734efbd44a25f4e2dc96d78784576a3eb5b335
$ docker rmi busybox:1.28.4
Untagged: busybox:1.28.4
Untagged: busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
Deleted: sha256:8c811b4aec35f259572d0f79207bc0678df4c736eeec50bc9fec37ed936a472a
Deleted: sha256:432b65032b9466b4dadcc5c7b11701e71d21c18400aae946b101ad16be62333a
$ docker pull harbor.k8s.local/library/busybox:1.28.4
1.28.4: Pulling from library/busybox
07a152489297: Pull complete
Digest: sha256:74f634b1bc1bd74535d5209589734efbd44a25f4e2dc96d78784576a3eb5b335
Status: Downloaded newer image for harbor.k8s.local/library/busybox:1.28.4
harbor.k8s.local/library/busybox:1.28.4

$ docker images |grep busybox
harbor.k8s.local/library/busybox                       1.28.4     8c811b4aec35   3 years ago     1.15MB
```

到这里证明上面我们的私有 docker 仓库搭建成功了，大家可以尝试去创建一个私有的项目，然后创建一个新的用户，使用这个用户来进行 pull/push  镜像，Harbor 还具有其他的一些功能，比如镜像复制，Helm Chart 包托管等等，大家可以自行测试，感受下 Harbor 和官方自带的  registry 仓库的差别。