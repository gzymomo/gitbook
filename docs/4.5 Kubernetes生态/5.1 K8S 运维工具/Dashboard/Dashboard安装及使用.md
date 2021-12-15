- [Dashboard安装及使用](https://www.cnblogs.com/v-fan/p/13950268.html)



## Helm安装Dashboard

#### 简介

Dashboard 是 kubernetes 的图形化管理工具，可直观的看到k8s中各个类型控制器的当前运行情况，以及Pod的日志，另外也可直接在 dashboard 中对已有的资源进行资源清单的修改。

 ![img](https://img2020.cnblogs.com/blog/1715041/202011/1715041-20201109191041545-1446222735.png)

 

#### 安装

> 注意：前提是已经安装好helm工具，如未安装还可请参考本人其他随笔：https://www.cnblogs.com/v-fan/p/13949025.html

```bash
# 安装 helm 的 repo源
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard

# 安装Dashboard，注意：要安装到kube-system名称空间下才可以操控整个集群
[root@Centos8 ~]# helm install k8s-dashboard/kubernetes-dashboard --version 2.6.0 -n k8s-dashboard --namespace kube-system
NAME:   k8s-dashboard
LAST DEPLOYED: Sat Sep 12 11:43:46 2020
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                        AGE
k8s-dashboard-kubernetes-dashboard-metrics  0s

==> v1/ClusterRoleBinding
NAME                                        AGE
k8s-dashboard-kubernetes-dashboard-metrics  0s

==> v1/ConfigMap
NAME                                         DATA  AGE
k8s-dashboard-kubernetes-dashboard-settings  0     1s

==> v1/Deployment
NAME                                READY  UP-TO-DATE  AVAILABLE  AGE
k8s-dashboard-kubernetes-dashboard  0/1    1           0          1s

==> v1/Pod(related)
NAME                                                 READY  STATUS             RESTARTS  AGE
k8s-dashboard-kubernetes-dashboard-6d5c6c747f-zgz79  0/1    ContainerCreating  0         1s

==> v1/Role
NAME                                AGE
k8s-dashboard-kubernetes-dashboard  0s

==> v1/RoleBinding
NAME                                AGE
k8s-dashboard-kubernetes-dashboard  0s

==> v1/Secret
NAME                                      TYPE    DATA  AGE
k8s-dashboard-kubernetes-dashboard-certs  Opaque  0     1s
kubernetes-dashboard-csrf                 Opaque  0     1s
kubernetes-dashboard-key-holder           Opaque  0     1s

==> v1/Service
NAME                                TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
k8s-dashboard-kubernetes-dashboard  ClusterIP  10.111.75.108  <none>       443/TCP  1s

==> v1/ServiceAccount
NAME                                SECRETS  AGE
k8s-dashboard-kubernetes-dashboard  1        1s


NOTES:
*********************************************************************************
*** PLEASE BE PATIENT: kubernetes-dashboard may take a few minutes to install ***
*********************************************************************************

Get the Kubernetes Dashboard URL by running:
  export POD_NAME=$(kubectl get pods -n kube-system -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=k8s-dashboard" -o jsonpath="{.items[0].metadata.name}")
  echo https://127.0.0.1:8443/
  kubectl -n kube-system port-forward $POD_NAME 8443:8443
```



> 回显中可以看到，在kube-system名称空间下为k8s集群创建ClusterRole、ClusterRoleBinding、ConfigMap、Deployment、Pod(related)、Role、RoleBinding、Secret、Service和ServiceAccount等资源
>
> 详细可参考helm官方手册：https://hub.helm.sh/charts/k8s-dashboard/kubernetes-dashboard

 

##### 查看是Pod是否正常启动

```bash
[root@Centos8 dashboard]# kubectl get pod -n kube-system
NAME                                                 READY   STATUS        RESTARTS   AGE
k8s-dashboard-kubernetes-dashboard-6d5c6c747f-zgz79   0/1     ImagePullBackOff   0  2m59s
```

发现 Pod 状态为 **ImagePullBackOff** ，因为镜像的原因，查看下所需镜像：

```bash
[root@Centos8 dashboard]# kubectl describe pod k8s-dashboard-kubernetes-dashboard-6d5c6c747f-zgz79 -n kube-system
  Normal   BackOff    72s (x7 over 3m13s)   kubelet, testcentos7  Back-off pulling image "kubernetesui/dashboard:v2.0.3"
```

 

自己手动导入镜像，并传入所有的node节点即可：

```bash
[root@Centos8 dashboard]# docker pull kubernetesui/dashboard:v2.0.3
v2.0.3: Pulling from kubernetesui/dashboard
d5ba0740de2a: Pull complete 
Digest: sha256:45ef224759bc50c84445f233fffae4aa3bdaec705cb5ee4bfe36d183b270b45d
Status: Downloaded newer image for kubernetesui/dashboard:v2.0.3
```

再次查看Pod状态，正常运行：

```bash
[root@Centos8 ~]# kubectl get pod -n kube-system
NAME                                                  READY   STATUS    RESTARTS   AGE
k8s-dashboard-kubernetes-dashboard-6d5c6c747f-zgz79   1/1     Running   0          102s
```

 

#### 配置

修改service的type类型为Nodeport，使其可以外部访问



```bash
# 默认为 ClusterIp
[root@Centos8 ~]# kubectl get svc -n kube-system
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   
k8s-dashboard-kubernetes-dashboard   ClusterIP   10.111.75.108    <none>        443/TCP   

# 修改
[root@Centos8 ~]# kubectl edit svc k8s-dashboard-kubernetes-dashboard -n kube-system
...
ports:
  - name: https
    nodePort: 30001
type: NodePort
...
service/k8s-dashboard-kubernetes-dashboard edited

#修改成功
[root@Centos8 ~]# kubectl get svc -n kube-system
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   
k8s-dashboard-kubernetes-dashboard   NodePort    10.111.75.108    <none>    443:30001/TCP
```



> Dashboard默认为https

 

#### 访问

[https://192.168.152.53:3000](https://192.168.152.53:30001)

![img](https://img2020.cnblogs.com/blog/1715041/202011/1715041-20201109191852563-458482418.png)

 

提示选择是使用Token方式连接还是Kubeconfig方式连接，看个人心情。

在此使用Token连接，查看Token方法：



```bash
[root@Centos8 ~]# kubectl get secret -n kube-system |grep dashboard
k8s-dashboard-kubernetes-dashboard-certs         Opaque                                0 
k8s-dashboard-kubernetes-dashboard-token-xpjj8   kubernetes.io/service-account-token   3 

[root@Centos8 ~]# kubectl describe secret k8s-dashboard-kubernetes-dashboard-token-xpjj8 -n kube-system | grep token
Name:         k8s-dashboard-kubernetes-dashboard-token-xpjj8
Type:  kubernetes.io/service-account-token
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrOHMtZGFzaGJvYXJkLWt1YmVybmV0ZXMtZGFzaGJvYXJkLXRva2VuLXhwamo4Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Ims4cy1kYXNoYm9hcmQta3ViZXJuZXRlcy1kYXNoYm9hcmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1YjRjMzViYi02Mzc3LTRhY2EtYWY0Yy1mZmQyYjg2OWFmM2YiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06azhzLWRhc2hib2FyZC1rdWJlcm5ldGVzLWRhc2hib2FyZCJ9.WTibcCYSOqTpfyTBT6vqsHULTfmWh3TU3NcQHIf-yZw-r5pdd2H5Edz4VqG6d_Ef1zwCzD6Burvdq80gQps7Ju9FdxLl_cjNgq6r9fycaYUMIedrgof7w43BIyBiwh064f3SFpJuZToVxErdHBnLToDpiNjJ0rbsn79oRufA6VRbqA0ogstcFfZ55lWGuEZ7JoDOUH_vno1geZQvk8LJLfd75EeMEBaq_F7I_7go5cydPvi11Sm3hKigOY53wwsBlvNJ3FlTfZMAxPb5IP024cJB-zXXdZjiUDGzeagcwAqrKdKwZl78RW1q0VXM5QwtL08dOBDgoOHMFeiSkeEjyw
```



将Token的值粘贴进网页，点击登录即可。

 

但是，默认情况下，直接进入，它本身没有访问整个集群的权限，所以要先对 dashborad 的SA进行一个ClusterRoleBinding 的操作：

vim dashbindins.yaml



```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-1
subjects:
- kind: ServiceAccount
  name: k8s-dashboard-kubernetes-dashboard
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```



> 将 cluster-admin  的权限赋予给名称为 k8s-dashboard-kubernetes-dashboard 的  SA，cluster-admin是集群默认的的角色，具有整个集群的所有权限，如果有个性化需求，自己定义一个ClusterRole也是可以的

 

进行绑定：

```bash
[root@Centos8 dashboard]# kubectl create -f dashbindins.yaml 
clusterrolebinding.rbac.authorization.k8s.io/dashboard-1 created
```

绑定完成后，再次刷新 dashboard 的界面，就可以看到整个集群的资源情况。

 

#### 个性化定制参数

Dashboard 默认采用 https 的形式进行访问，众所周知，https是需要绑定证书的，咱们以上直接通过 helm 方法安装的是自动绑定了config文件里的证书：

```bash
crt：grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d
kry:grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key
```

 

但是如果我们想要定义自己的https证书，我们可以在创建 dashboard 的时候采用指定变量的方法：

先将 dashboard 文件下载下来:



```bash
[root@Centos8 dashboard]# helm fetch k8s-dashboard/kubernetes-dashboard
[root@Centos8 dashboard]# ls
kubernetes-dashboard-2.6.0.tgz

[root@Centos8 dashboard]# tar zxvf kubernetes-dashboard-2.6.0.tgz
[root@Centos8 kubernetes-dashboard]# ls
charts  Chart.yaml  README.md  requirements.lock  requirements.yaml  templates  values.yaml
```



创建变量文件：

vim dashboardvaluse.yaml

```yaml
image: 
  repository: k8s.gcr.io/kubernetes-dashboard-amd64 # 指定存储库
  tag: v1.10.1  #指定版本
ingress:
  enabled: true #ingress是否开启
  hosts:
  - k8s.vfancloud.com   #指定域名
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  tls:  #指定secret，也就是指定你的证书
  - secretName: repository-ssl
    hosts:
    - k8s.vfancloud.com
rbac:
  clusterAdminRole: true
```



>  创建tls：kubectl create secret tls repository-ssl --key server.key --cert server.crt 

 

编辑完毕后，创建时使用 -f 指定此变量文件即可：



```bash
[root@Centos8 kubernetes-dashboard]# helm install . --version 2.6.0 -n k8s-dashboard --namespace kube-system -f dashboardvaluse.yaml
NAME:   k8s-dashboard
LAST DEPLOYED: Wed Sep 23 21:58:42 2020
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                        AGE
k8s-dashboard-kubernetes-dashboard-metrics  0s

==> v1/ClusterRoleBinding
NAME                                        AGE
k8s-dashboard-kubernetes-dashboard-metrics  0s

==> v1/ConfigMap
NAME                                         DATA  AGE
k8s-dashboard-kubernetes-dashboard-settings  0     2s

==> v1/Deployment
NAME                                READY  UP-TO-DATE  AVAILABLE  AGE
k8s-dashboard-kubernetes-dashboard  0/1    1           0          2s

==> v1/Pod(related)
NAME                                                 READY  STATUS             RESTARTS  AGE
k8s-dashboard-kubernetes-dashboard-6d5c6c747f-5dkhj  0/1    ContainerCreating  0         1s

==> v1/Role
NAME                                AGE
k8s-dashboard-kubernetes-dashboard  0s

==> v1/RoleBinding
NAME                                AGE
k8s-dashboard-kubernetes-dashboard  0s

==> v1/Secret
NAME                                      TYPE    DATA  AGE
k8s-dashboard-kubernetes-dashboard-certs  Opaque  0     2s
kubernetes-dashboard-csrf                 Opaque  0     2s
kubernetes-dashboard-key-holder           Opaque  0     2s

==> v1/Service
NAME                                TYPE       CLUSTER-IP   EXTERNAL-IP  PORT(S)  AGE
k8s-dashboard-kubernetes-dashboard  ClusterIP  10.97.71.25  <none>       443/TCP  2s

==> v1/ServiceAccount
NAME                                SECRETS  AGE
k8s-dashboard-kubernetes-dashboard  1        2s

==> v1beta1/Ingress
NAME                                HOSTS              ADDRESS  PORTS  AGE
k8s-dashboard-kubernetes-dashboard  hub.vfancloud.com  80, 443  2s


NOTES:
*********************************************************************************
*** PLEASE BE PATIENT: kubernetes-dashboard may take a few minutes to install ***
*********************************************************************************
From outside the cluster, the server URL(s) are:
     https://hub.vfancloud.com
```



> 至此，后边的步骤就和上边演示的一样了，绑定ClusterRole即可使用https://hub.vfancloud.com进行访问，以上变量文件仅仅是指定了部分值，具体其他可选变量可以去官网上查看。

 

使用域名访问：https://hub.vfancloud.com:31087