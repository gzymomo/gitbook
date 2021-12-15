## 1. K8S证书

### 1.1. 证书知识储备

此处为语雀文档，点击链接查看：https://www.yuque.com/duduniao/nginx/smgh7e

### 1.2. k8s 证书

该图细节不是很充分，仅作为参考使用，可以对照 k8s 集群安装章节各个证书进行比对和研究。

![未命名绘图.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1601213950451-6df7eedf-638d-44e6-8118-2b6472050616.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_TGludXgt5rih5rih6bif%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

## 2. RBAC

### 2.1. RBAC概述

#### 2.1.1. RBAC实现的原理

RBAC(Role Base Access Controller)，基于角色的访问控制，是目前Kubernetes最常用的权限控制插件。

Kubernetes的用户分为两种：User Account，用户账号，给Kubernetes操作人员使用的账号；Service Account，服务账号，给Kubernetes中Pod使用的账号。Kubernetes管理员查看和操作Kubernetes对象都是通过User Account 账号实现，而Pod去访问集群中的资源时使用的是Service Account。Kubernetes中一切皆对象，权限其实是对特定对象操作，如对某个名称空间中Pod的 GET/DELETE/POST 等操作。

不同的权限就是不同Permission的集合，将权限关联到Role上，再通过RoleBinding关联账户和权限，这就是RBAC实现方式。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580275583257-5d9621a1-e981-44a1-981f-23fb5901d758.png)



#### 2.1.2. 不同类型的授权方式

Role和RoleBinding属于名称空间资源，而Cluster和ClusterRoleBinding是集群层面的资源，Kubernetes允许三种绑定权限的方式(如图)：

- RoleBinding 关联 Role 和 User

User 具备当前NameSpace空间中的权限，不具备跨名称空间权限

- RoleBinding 关联 ClusterRole 和 User

ClusterRole权限降级，只能对当前名称空间中的资源具备权限，不具备跨名称空间的权限。该方式的意义在于：对不同名称空间管理员授权时，只需要定义一个ClusterRole即可，不需要定义多个基于名称空间的Role

- ClusterRoleBinding 关联 ClusterRole 和 User

User 具备集群级别权限，可以跨名称空间操作资源对象

![image.png](https://cdn.nlark.com/yuque/0/2020/png/378176/1580276021342-340b25eb-225a-40e9-9e1e-f59aefd86c60.png)

### 2.2. 模板

#### 2.2.1. Role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name            <string>                # 在一个名称空间不能重复
    namespace       <string>                # 指定名称空间，默认defalut
    labels          <map[string]string>     # 标签
    annotations     <map[string]string>     # 注释
rules:              <[]Object               # role权限
    resources       <[]string>              # 指定资源名称资源对象列表
    apiGroups       <[]string>              # 指定API资源组
    resourceNames   <[]string>              # 指定具体资源的白名单，默认允许所有
    nonResourceURLs <[]string>              # 一种特殊的k8s对象
    verbs           <[]string> -required-   # 权限列表(actions)
```

#### 2.2.2. ClusterRole

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
    name            <string>                        # 在一个集群内不能重复
    labels          <map[string]string>             # 标签
    annotations     <map[string]string>             # 注释
rules               <[]Object                       # 定义权限
    resources       <[]string>                      # 指定资源名称资源对象列表
    apiGroups       <[]string>                      # 指定API资源组
    resourceNames   <[]string>                      # 指定具体资源的白名单，默认允许所有
    nonResourceURLs <[]string>                      # 一种特殊的k8s对象
    verbs           <[]string> -required-           # 权限列表(actions)
aggregationRule     <Object>                        # 定义聚合规则
    clusterRoleSelectors    <[]Object>              # 集群角色选择器
        matchLabels         <map[string]string>     # key/value 选择器
        matchExpressions    <[]Object>              # 表达式选择器,参考 deployment.spec.selector
```

#### 2.2.3. RoleBinding

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name            <string>                # 在一个名称空间不能重复
    namespace       <string>                # 指定名称空间，默认defalut
    labels          <map[string]string>     # 标签
    annotations     <map[string]string>     # 注释
roleRef             <Object> -required-     # 待绑定的角色
    kind            <string> -required-     # 资源类型
    name            <string> -required-     # 资源名称
    apiGroup        <string> -required-     # 资源组的APIGroup
subjects            <[]Object>              # 账户
    apiGroup        <string>                # 账户的api组名
        # "" 空字串表示serviceAccount
        # rbac.authorization.k8s.io 表示User或者Group
    kind            <string> -required-     # 账户类型，如User/Group/serviceAccount
    name            <string> -required-     # 账户名称
    namespace       <string>                # 账户的名称空间
```

#### 2.2.4. ClusterRoleBinding

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
    name            <string>                # 在一个集群内不能重复
    labels          <map[string]string>     # 标签
    annotations     <map[string]string>     # 注释
roleRef             <Object> -required-     # 待绑定的角色
    kind            <string> -required-     # 资源类型
    name            <string> -required-     # 资源名称
    apiGroup        <string> -required-     # 资源组的APIGroup
subjects            <[]Object>              # 账户
    apiGroup        <string>                # 账户的api组名
        # "" 空字串表示serviceAccount
        # rbac.authorization.k8s.io 表示User或者Group
    kind            <string> -required-     # 账户类型，如User/Group/serviceAccount
    name            <string> -required-     # 账户名称
    namespace       <string>                # 账户的名称空间
```

#### 2.2.5. ServiceAccount

```
# 一般只需要指定serviceaccount的名称即可
# 除了基础字段之外的其它字段很少使用，需要时手动查询
apiVersion: v1
kind: ServiceAccount        
metadata
    name            <string>                # 在一个名称空间不能重复
    namespace       <string>                # 指定名称空间，默认defalut
    labels          <map[string]string>     # 标签
    annotations     <map[string]string>     # 注释
```

### 2.3. 访问APIServer方式

Kubernetes 账号访问 APIServer 时，可以通过两种方式实现，一种是服务账号的 Token，一种是用户账号的config配置文件，其中用户账户的config中需要配置相关的证书信息。 

### 2.4. 案例

#### 2.4.1. 授权User Account

- RBAC授权

```
[root@hdss7-200 rbac]# cat /data/k8s-yaml/base_resource/rbac/duduniao-user.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: duduniao-role
  namespace: app
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  verbs:
  - get
  - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: duduniao-rolebinding
  namespace: app
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: duduniao-role
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: duduniao
```

- 创建必要的用户证书

```
# kubectl 的配置文件读取顺序：--kubeconfig 参数指定 > $KUBECONFIG 环境变量指定 > ${HOME}/.kube/config
# 此次以集群的CA证书为本，签署并创建一个新的 user account：
# 10.4.7.201 只拷贝了一个二进制命令 kubectl 到 /root/bin/ 下，未拷贝其它任何内容K8S配置
[root@hdss7-200 ~]# cd /opt/certs
[root@hdss7-200 certs]# cat duduniao-csr.json  # 模仿 kubelet-csr.json 来修改
{
    "CN": "duduniao",
    "hosts": [
    "127.0.0.1",
    "10.4.7.10",
    "10.4.7.21",
    "10.4.7.22",
    "10.4.7.23",
    "10.4.7.24",
    "10.4.7.25",
    "10.4.7.26",
    "10.4.7.27",
    "10.4.7.28"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "beijing",
            "L": "beijing",
            "O": "od",
            "OU": "ops"
        }
    ]
}
[root@hdss7-200 certs]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=client duduniao-csr.json | cfssl-json -bare duduniao
[root@hdss7-200 certs]# scp duduniao* ca.pam 10.4.7.201:/root/certs
```

- 创建config文件

```
[root@operate-7-201 ~]# kubectl config set-cluster myk8s \ # 集群名称可以自定义
--certificate-authority=/root/certs/ca.pem \
--embed-certs=true \
--server=https://10.4.7.10:7443 \
--kubeconfig=/root/.kube/config

[root@operate-7-201 ~]# kubectl config set-credentials duduniao \
--client-certificate=/root/certs/duduniao.pem \
--client-key=/root/certs/duduniao-key.pem \
--embed-certs=true \
--kubeconfig=/root/.kube/config

[root@operate-7-201 ~]# kubectl config set-context myk8s-context \
--cluster=myk8s \
--user=duduniao \
--kubeconfig=/root/.kube/config

[root@operate-7-201 ~]# kubectl config use-context myk8s-context --kubeconfig=/root/.kube/config

[root@operate-7-201 ~]# kubectl get pod # 没有访问default名称空间
Error from server (Forbidden): pods is forbidden: User "duduniao" cannot list resource "pods" in API group "" in the namespace "default"
[root@operate-7-201 ~]# kubectl get svc -n app  # 账户创建成功
NAME     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)       AGE
slb-s1   ClusterIP   192.168.219.15   <none>        80/TCP        6d21h
slb-s2   ClusterIP   192.168.10.100   <none>        80/TCP        6d9h
slb-s3   NodePort    192.168.1.125    <none>        80:3080/TCP   6d8h
slb-s4   ClusterIP   None             <none>        80/TCP        6d6h
[root@operate-7-201 ~]# rm -fr /root/certs
```

#### 2.4.2. 授权Service Account

参考: https://www.yuque.com/duduniao/ww8pmw/myrwhq#X4UlI

## 3. DashBoard账户授权

Dashboard的登陆有两种形式，一种是使用 Token，另一种是使用 kube-config 文件，但是两种方式都需要使用ServiceAccount。

### 3.1. 两种登陆方法

#### 3.1.1. 使用Token

```
[root@hdss7-200 ~]# cat /data/k8s-yaml/dashboard/dashboard_1.10.1/admin-rbac.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```



```
[root@hdss7-21 ~]# kubectl apply -f http://k8s-yaml.od.com/dashboard/dashboard_1.10.1/admin-rbac.yaml
[root@hdss7-21 ~]# kubectl get secret -n kube-system | grep kubernetes-dashboard-admin
kubernetes-dashboard-admin-token-lz9rl   kubernetes.io/service-account-token   3      28s
[root@hdss7-21 ~]# kubectl describe secret kubernetes-dashboard-admin-token-lz9rl -n kube-system | grep ^token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1sejlybCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQzYzY0NWFiLTc0MGMtNDNlOS05ZWZmLTUwMTZmMmQwMWM3NSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.Ixik7h42Zfavi_RjgOOw3Exq0TS4IAEjs_mlTWXBeOQ3zxu-hFy4Y2BMSrk2hCHQRxac8Lqz-3L0e7NPMOqu_jK8M6J65xPok4apgzcDWLu17fmH32TvmezvGi0NVT_3EtxsmDfneKFpSZ-XDNwR2TUsNcpQSMMZm32Jj3ohvqXTPeW1gQBFjTY2SdbzLIxFJqFICo_du67m7Gm0N6XugSOSs9pVDz5ucoANsTMsjJ_FAznorT54Xzo8B0aHpkTSRb7Jzz-iLt9QIK2WBDaRbNVdVMlhAFoyMoqG4e1kE-LA5i4912VBqGhmKmduLhDK-z2QyjbyX6qZE5VhWGh3Gg
```

#### 3.1.2. 使用kube-config

dashboard的kube-config需要使用serviceaccount账户，创建方式和 User Account 类似，可以参考: https://www.yuque.com/duduniao/ww8pmw/myrwhq#0iMsc。不过除了可以使用证书来创建kube-config文件，还可以使用Token来创建，以下以Token方式创建kube-config文件：

```
# 获取token信息，注意：使用describe获取的token末尾可能缺少字符串"=="导致解码失败
[root@hdss7-21 ~]# server_token=$(kubectl get secret kubernetes-dashboard-admin-token-lz9rl -n kube-system -o jsonpath={.data.token}|base64 -d)
# 创建config文件
[root@hdss7-21 ~]# kubectl config set-cluster kubernetes --certificate-authority=/opt/apps/kubernetes/server/bin/certs/ca.pem --embed-certs=true --server=https://10.4.7.10:7443 --kubeconfig=/root/dashboard-admin.config
[root@hdss7-21 ~]# kubectl config set-credentials kubernetes-dashboard-admin --token=$server_token --kubeconfig=/root/dashboard-admin.config
[root@hdss7-21 ~]# kubectl config set-context dashboard-admin@kubernetes --cluster=kubernetes --user=kubernetes-dashboard-admin --kubeconfig=/root/dashboard-admin.config
[root@hdss7-21 ~]# kubectl config use-context dashboard-admin@kubernetes --kubeconfig=/root/dashboard-admin.config
# 清除临时变量
[root@hdss7-21 ~]# unset server_token
[root@hdss7-21 ~]# kubectl config view --kubeconfig=/root/dashboard-admin.config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.4.7.10:7443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-dashboard-admin
  name: dashboard-admin@kubernetes
current-context: dashboard-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1sejlybCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQzYzY0NWFiLTc0MGMtNDNlOS05ZWZmLTUwMTZmMmQwMWM3NSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.Ixik7h42Zfavi_RjgOOw3Exq0TS4IAEjs_mlTWXBeOQ3zxu-hFy4Y2BMSrk2hCHQRxac8Lqz-3L0e7NPMOqu_jK8M6J65xPok4apgzcDWLu17fmH32TvmezvGi0NVT_3EtxsmDfneKFpSZ-XDNwR2TUsNcpQSMMZm32Jj3ohvqXTPeW1gQBFjTY2SdbzLIxFJqFICo_du67m7Gm0N6XugSOSs9pVDz5ucoANsTMsjJ_FAznorT54Xzo8B0aHpkTSRb7Jzz-iLt9QIK2WBDaRbNVdVMlhAFoyMoqG4e1kE-LA5i4912VBqGhmKmduLhDK-z2QyjbyX6qZE5VhWGh3Gg
```

### 3.2. 常用dashboard角色设置

Dashboard 权限设置中，有两个主要集群角色： cluster-admin、view，一个是管理员权限，一个是查看权限。

#### 3.2.1. 集群管理员

参考：https://www.yuque.com/duduniao/ww8pmw/myrwhq#X4UlI

#### 3.2.2. 名称空间管理员

- 创建能列出名称空间的集群角色

因为dashboard默认进入的界面是default，名称空间级别用户无法查看和切换到其它名称空间

```
[root@hdss7-200 ~]# cat /data/k8s-yaml/dashboard/dashboard_1.10.1/list-namespace.yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata: 
  name: list-namespace
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - list
```

- RBAC授权

```
[root@hdss7-200 ~]# cat /data/k8s-yaml/dashboard/dashboard_1.10.1/namespace-admin.yaml 
# 创建service account 账户
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: ns-admin
  namespace: kube-system
---
# 授权default名称空间的管理员权限
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ns-admin-default
  namespace: default
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: ns-admin
  namespace: kube-system
---
# 授予app名称空间的管理员权限
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ns-admin-app
  namespace: app
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: ns-admin
  namespace: kube-system
---
# 可以查到到其它名称空间，方便切换。不受该权限时，需要手动修改URL中namespace完成切换
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: ns-admin-list-namespace
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: list-namespace
subjects:
- kind: ServiceAccount
  name: ns-admin
  namespace: kube-system
```

#### 3.2.3. 名称空间浏览权限

```
[root@hdss7-200 ~]# cat /data/k8s-yaml/dashboard/dashboard_1.10.1/namespace-view.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
  name: ns-view
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ns-view-default
  namespace: default
  labels:
    k8s-app: kubernetes-dashboard
    addonmanager.kubernetes.io/mode: Reconcile
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- kind: ServiceAccount
  name: ns-view
  namespace: kube-system
```