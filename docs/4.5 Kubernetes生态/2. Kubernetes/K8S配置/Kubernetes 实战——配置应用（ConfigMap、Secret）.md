[Kubernetes 实战——配置应用（ConfigMap、Secret）](https://www.cnblogs.com/lb477/p/14889395.html)

配置容器化应用的方式：①命令行参数；②环境变量；③文件化配置

## 一、向容器传递命令行参数或环境变量

这两种方式在 Pod 创建后不可被修改

### 1. 在Docker中定义命令与参数

- **ENTRYPOINT**：容器启动时被调用的可执行程序
- **CMD**：传递给 ENTRYPOINT 的默认参数。可被覆盖`docker run <image> <arguments>`

上面两条指令均支持以下两种形式

- **shell**：`ENTRYPOINT node app.js`（`/bin/sh -c node app.js`）
- **exec**：`ENTRYPOINT ["node", "app.js"]`（`node app.js`）

***e.g.***

```dockerfile
FROM ubuntu:latest
ADD test.sh /bin/test.sh  # test.sh每“$1”秒输出一行文本
ENTRYPOINT ["/bin/test.sh"]
CMD ["10"]
docker build -t lb/test:args .
docker push lb/test:args
docker run -it lb/test:args
docker run -it lb/test:args 15  # 传递参数
```

### 2. 向容器传递命令行参数

镜像的 ENTRYPOINT 和 CMD 均可被覆盖

```yml
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]  # 对应ENTRYPOINT，一般情况不覆盖
    args: ["arg1", "arg2"]  # 对应CMD
```

另一种参数表示方式

```yml
args:
- foo  # 字符串无需引号标记
- "15"  # 数值需要
```

### 3. 为容器设置环境变量

K8s 可为 Pod 中的每个容器指定环境变量

```yml
kind: Pod
spec:
  containers:
  - image: some/image
    env:  # 指定环境变量
    - name: FOO
      value: "foo"
    - name: BAR
      value: "$(INTERVAL)bar"  # 引入其他环境变量（command和args属性值也可以借此引用环境变量）
```

> 在每个容器中，K8s 会自动暴露相同命名空间下每个 service 对应的环境变量

## 二、ConfigMap

### 1. 介绍

- 本质为键值对映射，值可以为字面量或配置文件
- 应用无需读取 ConfigMap，映射的内容通过环境变量或卷文件的形式传递给容器
- Pod 通过名称引用 ConfigMap

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616102938814-1797883046.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616102938814-1797883046.png)

### 2. 创建 ConfigMap

```yml
apiVersion: v1
kind: ConfigMap
data:
  sleep-interval: "25"  # 配置条目
metadata:
  name: my-config
```

***直接创建***

```bash
# 可指定字面量或配置文件
kubectl create configmap my-config \
--from-file=foo.json \  # 单独的文件
--from-file=bar=foobar.conf \  # 自定义键名目录下的文件
--from-file=config-opts/ \  # 完整文件夹
--from-literal=some=thing  # 字面量
```

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616103928916-1743187133.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616103928916-1743187133.png)

> ConfigMap 键名需仅包含数字、字母、破折号、下划线、圆点，可首位圆点。键名不合法则不会映射

### 3. 传递 ConfigMap 作为环境变量

```yml
kind: Pod
spec:
  containers:
  - image: some/image
    env:
    - name: INTERVAL
      valueFrom:  # 使用ConfigMap中的key初始化
        configMapKeyRef:
          name: my-config
          key: sleep-interval
```

> 启动 Pod 时若无 ConfigMap：Pod 正常调度，容器启动失败。后续创建 ConfigMap，失败容器会重启。可设置`configMapKeyRef.optional=true`，这样即使 ConfigMap 不存在，容器也能启动

***一次性传递所有***

```yml
spec:
  containers:
  - image: some/image
    envFrom:  # 传递所有
    - prefix: CONFIG_  # 为所有环境变量设置前缀（可选）
      configMapKeyRef:
        name: my-config
```

> 若不是合法的环境变量名称，K8s 不会自动转换键名（如 CONFIG_FOO-BAR）

### 4. 传递 ConfigMap 作为命令行参数

使用 ConfigMap 初始化某个环境变量，然后在参数字段中引用该环境变量

```yml
apiVersion: v1
kind: Pod
spec:
  containers:
  - image: some/image
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: sleep-interval
    args: ["$(INTERVAL)"]  # 参数中引用环境变量
```

### 5. 传递 ConfigMap 作为配置文件

ConfigMap 卷会将每个条目暴露成一个文件，运行在容器中的进程可通过读取文件内容获取相应的值

***将 ConfigMap 卷挂载到某个文件夹***

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-configmap
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d/  # 挂载到文件夹会隐藏该文件夹中已存在的文件
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: fortune-config
```

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616111230851-618433494.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616111230851-618433494.png)

```bash
$ kubectl exec test-configmap -c web-server ls /etc/nginx/conf.d
my-nginx-config.conf
sleep-interval
```

***暴露指定的 ConfigMap 条目***

```yml
spec:
  volumes:
  - name: config
    configMap:
      name: fortune-config
      items:  # 只暴露指定条目为文件
      - key: my-nginx-config.conf
        path: test.conf
# 此时 /etc/nginx/conf.d/ 中只有 test.conf
```

***不隐藏文件夹中的其他文件***

subPath 可用作挂载卷中的某个独立文件或文件夹，无需挂载整个卷

```yml
spec:
  containers:
  - image: some/image
    volumeMounts:
    - name: config
      mountPath: /etc/someconfig.conf  # 挂载到某个文件
      subPath: myconfig.conf  # 仅挂载条目myconfig.conf
```

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616113047769-1145160007.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616113047769-1145160007.png)

> 这种文件挂载方式会有文件更新的缺陷

***为 ConfigMap 卷中的文件设置权限***

```yml
volumes:
- name: config
  configMap:
    name: fortune-config
    defaultMode: "6600"  # 默认644
```

### 6. 更新配置且不重启应用程序

更新 ConfigMap 后，卷中引用它的文件也会相应更新，进程发现文件改变后进行重载。K8s 也支持文件更新后手动通知容器

```bash
kubectl edit configmap fortune-config
kubectl exec test-configmap -c web-server cat /etc/nginx/conf.d/my-nginx-config.conf
kubectl exec test-configmap -c web-server -- nginx -s reload
```

- 所有文件会被一次性更新：ConfigMap 更新后，K8s 会创建一个文件夹写入所有文件，最终将符号链接转为该文件夹
- 如果挂载的是容器中的单个文件而不是完整的卷，ConfigMap 更新后对应的文件不会被更新

## 三、Secret

与 ConfigMap 类似。区别：①K8s 仅将 Secret 分发到需要访问 Secret 的 Pod 所在的机器节点；②只存在于节点的内存中；③etcd 以加密形式存储 Secret

### 1. 默认令牌 Secret

默认被挂载到所有容器（可通过设置 Pod 定义中或 Pod 使用的服务账户中的`automountServiceAccountToken=false`来关闭该默认行为）

```bash
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-zns7b   kubernetes.io/service-account-token   3      2d
$ kubectl describe secrets
...
Data  # 包含从 Pod 内部安全访问 K8s API 服务器所需的全部信息
====
ca.crt:     570 bytes
namespace:  7 bytes
token:      eyJhbGciO...
$ kubectl describe pod
...
Mounts:
  /var/run/secrets/kubernetes.io/serviceaccount from default-token-zns7b
$ kubectl exec mypod ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt
namespace
token
```

### 2. 创建 Secret

```bash
$ kubectl create secret generic fortune-https --from-file=https.key --from-file=https.cert --from-file=foo
$ kubectl get secret fortune-https -o yaml
apiVersion: v1
kind: Secret
data:
  foo: YmFyCg==
  https.cert: HtD4SF...
  https.key: PJgF0G...
```

Secret 条目的内容会被 Base64 编码：Secret 条目可涵盖二进制数据，Base64 编码可将二进制数据转为纯文本（但 Secret 大小限于 1MB）

***向 Secret 添加纯文本条目***

```yml
stringData:  # 该字段只写，查看时不会显示，而是被编码后展示在data字段下
  foo: bar
```

### 3. 使用 Secret

将 Secret 卷暴露给容器后，条目的值会被自动解码并以真实形式写入对应文件或环境变量

```yml
apiVersion: v1
kind: Pod
metadata:
  name: test-secret
spec:
  containers:
  - image: luksa/fortune:env
    name: html-generator
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
    - name: certs
      mountPath: /etc/nginx/certs/
      readOnly: true
    ports:
    - containerPort: 80
    - containerPort: 443
  volumes:
  - name: html
    emptyDir: {}
  - name: config
    configMap:
      name: fortune-config
      items:
      - key: my-nginx-config.conf
        path: https.conf
  - name: certs
    secret:
      secretName: fortune-https
```

> Secret 也支持通过 defaultModes 指定卷中文件的默认权限

[![img](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616142647050-592085701.png)](https://img2020.cnblogs.com/blog/1614145/202106/1614145-20210616142647050-592085701.png)

```bash
# Secret 卷采用内存文件系统列出容器的挂载点，不会写入磁盘
$ kubectl exec test-secret -c web-server -- mount | grep certs
tmpfs on /etc/nginx/certs type tmpfs (ro,relatime)
```

***暴露 Secret 为环境变量***

不推荐，该方式可能无意中暴露 Secret 信息

```yml
env:
- name: FOO_SECRET
  valueFrom:
    secretKeyRef:
      name: fortune-https
      key: foo
```

### 4. 镜像拉取 Secret

从私有镜像仓库拉取镜像时，K8s 需拥有拉取镜像所需的证书

①创建包含 Docker 镜像仓库证书的 Secret

```bash
kubectl create secret docker-registry mydockerhubsecret \
--docker-username=myusername --docker-password=mypassword --docker-email=myemail
```

②Pod 中字段`imagePullSecrets`引用该 Secret

```yml
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  imagePullSecrets:
  - name: mydockerhubsecret
  containers:
  - image: username/private:tag
    name: test
```

后续可添加 Secret 到 ServiceAccount 使所有 Pod 都能自动添加上该 Secret