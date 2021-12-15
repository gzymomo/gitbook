- [HTTPS认证和授权机制学习](https://www.cnblogs.com/kevingrace/p/9882026.html)



**Kubernetes经过一系列认证授权机制来实现集群的安全机制，包括API Server的认证、授权、准入控制机制等。集群的安全性必须考虑以下的几个目标：**

- 保证容器与其所在宿主机的隔离；
-  限制容器给基础设施及其他容器带来消极影响的能力；
- 最小权限原则，合理限制所有组件权限，确保组件只执行它被授权的行为，通过限制单个组件的能力来限制他所能达到的权限范围；
- 明确组件间边界的划分；
- 划分普通用户和管理员角色；
- 在必要的时候允许将管理员权限赋给普通用户；
- 允许拥有Secret数据（Keys、Certs、Passwords）的应用在集群中运行；

# 一、HTTPS 数字证书认证

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624104426349-2038655769.png)

**HTTPS工作流程**

1. 浏览器发起https请求，将自己支持的一套加密规则发送给服务端。
2. 服务端从中选出一组加密算法与HASH算法，并**将自己的身份信息以证书的形式发回给浏览器**。证书里面包含了服务端地址，加密公钥，以及证书的颁发机构等信息。
3. 获得服务端证书之后浏览器要做以下工作：
   - 验证证书的合法性（颁发证书的机构是否合法，证书中包含的服务端地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏里面会显示一个小锁头，否则会给出证书不受信的提示。
   - 如果证书受信任，或者是用户接受了不受信的证书，浏览器会生成一串随机数的密码，并用证书中提供的公钥加密。
   -  使用约定好的HASH计算握手消息，并使用生成的随机数对消息进行加密，最后将之前生成的所有信息发送给服务端。
4. 服务端接收浏览器发来的数据之后要做以下的操作：
   - 使用自己的私钥将信息解密取出密码，使用密码解密浏览器发来的握手消息，并验证HASH是否与浏览器发来的一致。
   - 使用密码加密一段握手消息，发送给浏览器。
5. 浏览器解密并计算握手消息的HASH，如果与服务端发来的HASH一致，则此时握手过程结束，之后所有通信数据将由之前浏览器生成的随机密码并利用加密算法进行加密。

# 二、Kubernetes加密解密原理

kubernetes内部常用的加解密算法为非对称加密算法RSA。

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624105156940-128884966.png)

1. 每个用户都有一对私钥和公钥。
   - 私钥用来进行解密和签名，是给自己用的。
   - 公钥由本人公开，用于加密和验证签名，是给别人用的。
2. 当该用户发送文件时，用私钥签名，别人用他给的公钥解密，可以保证该信息是由他发送的。即数字签名。
3. 当该用户接受文件时，别人用他的公钥加密，他用私钥解密，可以保证该信息只能由他看到。即安全传输

# 三、 数字证书

数字证书则是由证书认证机构（CA）对证书申请者真实身份验证之后，用CA的根证书对申请人的一些基本信息以及申请人的公钥进行签名（相当于加盖发证书机 构的公章）后形成的一个数字文件。CA完成签发证书后，会将证书发布在CA的证书库（目录服务器）中，任何人都可以查询和下载，因此数字证书和公钥一样是公开的。实际上，数字证书就是经过CA认证过的公钥

# 四、CA认证流程

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624105450476-2034140988.png)

## 4.1 SSL双向认证步骤

1. HTTPS通信双方的服务器端向CA机构申请证书，CA机构是可信的第三方机构，它可以是一个公认的权威的企业，也可以是企业自身。企业内部系统一般都使用企业自身的认证系统。CA机构下发根证书、服务端证书及私钥给申请者；
2. HTTPS通信双方的客户端向CA机构申请证书，CA机构下发根证书、客户端证书及私钥个申请者；
3.  客户端向服务器端发起请求，服务端下发服务端证书给客户端。客户端接收到证书后，通过私钥解密证书，并利用服务器端证书中的公钥认证证书信息比较证书里的消息，例如域名和公钥与服务器刚刚发送的相关消息是否一致，如果一致，则客户端认为这个服务器的合法身份；
4. 客户端发送客户端证书给服务器端，服务端接收到证书后，通过私钥解密证书，获得客户端的证书公钥，并用该公钥认证证书信息，
5. 客户端通过随机秘钥加密信息，并发送加密后的信息给服务端。服务器端和客户端协商好加密方案后，客户端会产生一个随机的秘钥，客户端通过协商好的加密方案，加密该随机秘钥，并发送该随机秘钥到服务器端。服务器端接收这个秘钥后，双方通信的所有内容都都通过该随机秘钥加密；

# 五、Kubernetes几个重要的认证凭据

## 5.1 ca.pem & ca-key.pem & ca.csr

有上文我们可以知道，建立完整TLS加密通信，需要有一个CA认证机构，会向客户端下发根证书、服务端证书以及签名私钥给客户端。ca.pem & ca-key.pem & ca.csr组成了一个自签名的CA机构。

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624110255916-1262481450.png)

## 5.2 token.csv

该文件为一个用户的描述文件，基本格式为 Token,用户名,UID,用户组；这个文件在 apiserver 启动时被 apiserver 加载，然后就相当于在集群内创建了一个这个用户；接下来就可以用 RBAC 给他授权

## 5.3 bootstrap.kubeconfig

该文件中内置了 token.csv 中用户的 Token，以及 apiserver CA 证书；kubelet 首次启动会加载此文件，使用 apiserver CA 证书建立与 apiserver 的 TLS 通讯，使用其中的用户 Token 作为身份标识像 apiserver 发起 CSR 请求

## 5.4 kubelet-client-current.pem

这是一个软连接文件，当 kubelet 配置了 --feature-gates=RotateKubeletClientCertificate=true选项后，会在证书总有效期的 70%~90% 的时间内发起续期请求，请求被批准后会生成一个 kubelet-client-时间戳.pem；kubelet-client-current.pem 文件则始终软连接到最新的真实证书文件，除首次启动外，kubelet 一直会使用这个证书同 apiserver 通讯

## 5.5 kubelet-server-current.pem

同样是一个软连接文件，当 kubelet 配置了 --feature-gates=RotateKubeletServerCertificate=true 选项后，会在证书总有效期的 70%~90% 的时间内发起续期请求，请求被批准后会生成一个 kubelet-server-时间戳.pem；kubelet-server-current.pem 文件则始终软连接到最新的真实证书文件，该文件将会一直被用于 kubelet 10250 api 端口鉴权

# 六、组件证书 及 配置参数

所有客户端的证书首先要经过Kubernetes集群CA的签署，否则不会被集群认可 .

## 6.1 kubectl

kubectl只是个go编写的可执行程序，只要为kubectl配置合适的kubeconfig，就可以在集群中的任意节点使用 。kubectl的权限为admin，具有访问kubernetes所有api的权限。

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624110910047-1355157739.png)

1. --certificate-authority=/etc/kubernetes/ssl/ca.pem 设置了该集群的根证书路径， --embed-certs为true表示将--certificate-authority证书写入到kubeconfig中；
2. --client-certificate=/etc/kubernetes/ssl/admin.pem 指定kubectl证书；
3. --client-key=/etc/kubernetes/ssl/admin-key.pem 指定kubectl私钥；

## 6.2 kubelet

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624111126779-551231297.png)

当成功签发证书后，目标节点的 kubelet 会将证书写入到 --cert-dir= 选项指定的目录中；此时如果不做其他设置应当生成上述除ca.pem以外的4个文件。

- kubelet-client.crt 该文件在 kubelet 完成 TLS bootstrapping 后生成，此证书是由 controller manager 签署的，此后 kubelet 将会加载该证书，用于与 apiserver 建立 TLS 通讯，同时使用该证书的 CN 字段作为用户名，O 字段作为用户组向 apiserver 发起其他请求。
- kubelet.crt 该文件在 kubelet 完成 TLS bootstrapping 后并且没有配置 --feature-gates=RotateKubeletServerCertificate=true 时才会生成；这种情况下该文件为一个独立于 apiserver CA 的自签 CA 证书，有效期为 1 年；被用作 kubelet 10250 api 端口。

## 6.3 kube-apiserver

kube-apiserver是我们在部署kubernetes集群是最需要先启动的组件，也是我们和集群交互的核心组件。
以下是kube-apiserver所使用的证书：

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624111742636-855209636.png)

1. --token-auth-file=/etc/kubernetes/token.csv 指定了token.csv的位置，用于kubelet 组件 第一次启动时没有证书如何连接 apiserver 。 Token 和 apiserver 的 CA 证书被写入了 kubelet 所使用的 bootstrap.kubeconfig 配置文件中；这样在首次请求时，kubelet 使用 bootstrap.kubeconfig 中的 apiserver CA 证书来与 apiserver 建立 TLS 通讯，使用 bootstrap.kubeconfig 中的用户 Token 来向 apiserver 声明自己的 RBAC 授权身份
2. --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem 指定kube-apiserver证书地址
3. --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem 指定kube-apiserver私钥地址
4. --client-ca-file=/etc/kubernetes/ssl/ca.pem 指定根证书地址
5. --service-account-key-file=/etc/kubernetes/ssl/ca-key.pem 包含PEM-encoded x509 RSA公钥和私钥的文件路径，用于验证Service Account的token，如果不指定，则使用--tls-private-key-file指定的文件
6. --etcd-cafile=/etc/kubernetes/ssl/ca.pem 到etcd安全连接使用的SSL CA文件
7. --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem 到etcd安全连接使用的SSL 证书文件
8. --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem 到etcd安全连接使用的SSL 私钥文件

## 6.4 kube-controller-manager

kubelet 发起的 CSR 请求都是由 kube-controller-manager 来做实际签署的,所有使用的证书都是根证书的密钥对 。由于kube-controller-manager是和kube-apiserver部署在同一节点上，且使用非安全端口通信，故不需要证书。

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624114014278-173207272.png)

1. --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem 指定签名的CA机构根证书，用来签名为 TLS BootStrap 创建的证书和私钥；
2. --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem 指定签名的CA机构私钥，用来签名为 TLS BootStrap 创建的证书和私钥；
3. --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem 同上；
4. --root-ca-file=/etc/kubernetes/ssl/ca.pem 根CA证书文件路径 ，用来对 kube-apiserver 证书进行校验，指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件
5. --kubeconfig kubeconfig 配置文件路径，在配置文件中包括Master的地址信息及必要认证信息；

## 6.5 kube-scheduler && kube-proxy

kube-scheduler是和kube-apiserver一般部署在同一节点上，且使用非安全端口通信，故启动参参数中没有指定证书的参数可选 。 若分离部署，可在kubeconfig文件中指定证书，使用kubeconfig认证，kube-proxy类似
配置示例:

```bash
设置集群参数
# kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig
 
设置客户端认证参数
# kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem \
  --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
 
设置上下文参数
# kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
 
设置默认上下文
# kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
# mv kube-proxy.kubeconfig /etc/kubernetes/
```

## 6.6 [TLS Bootstrapping](https://mritd.me/2018/01/07/kubernetes-tls-bootstrapping-note/)

每个 Kubernetes 集群都有一个集群根证书颁发机构（CA）。 集群中的组件通常使用 CA 来验证 API server 的证书，由API服务器验证 kubelet 客户端证书等。为了支持这一点，CA 证书包被分发到集群中的每个节点，并作为一个 secret 附加分发到默认 service account 上 。

- 想要与 apiserver 通讯就必须采用由 apiserver CA 签发的证书，这样才能形成信任关系，建立 TLS 连接；
- 证书的 CN、O 字段来提供 RBAC 所需的用户与用户组；

## 6.7 kubelet首次启动流程

**第一次启动时没有证书如何连接 apiserver ?**   这个问题实际上可以去查看一下 bootstrap.kubeconfig 和 token.csv, 可以得到如下答案：
在 apiserver 配置中指定了一个 token.csv 文件，该文件中是一个预设的用户配置；同时该用户的 Token 和 apiserver 的 CA 证书被写入了 kubelet 所使用的 bootstrap.kubeconfig 配置文件中；这样在首次请求时，kubelet 使用 bootstrap.kubeconfig 中的 apiserver CA 证书来与 apiserver 建立 TLS 通讯，使用 bootstrap.kubeconfig 中的用户 Token 来向 apiserver 声明自己的 RBAC 授权身份。如下图所示：

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624114946731-189201306.png)

在首次启动时，可能与遇到 kubelet 报 401 无权访问 apiserver 的错误；这是因为在默认情况下，kubelet 通过 bootstrap.kubeconfig 中的预设用户 Token 声明了自己的身份，然后创建 CSR 请求；但是不要忘记这个用户在我们不处理的情况下他没任何权限的，包括创建 CSR 请求；所以需要如下命令创建一个 ClusterRoleBinding，将预设用户 kubelet-bootstrap 与内置的 ClusterRole system:node-bootstrapper 绑定到一起，使其能够发起 CSR 请求。

```bash
kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --user=kubelet-bootstrap
```

## 6.8 CSR请求类型

kubelet 发起的 CSR 请求都是由 controller manager 来做实际签署的，对于 controller manager 来说，TLS bootstrapping 下 kubelet 发起的 CSR 请求大致分为以下三种

- nodeclient: kubelet 以 O=system:nodes 和 CN=system:node:(node name) 形式发起的 CSR 请求
- selfnodeclient: kubelet client renew 自己的证书发起的 CSR 请求(与上一个证书就有相同的 O 和 CN)
- selfnodeserver: kubelet server renew 自己的证书发起的 CSR 请求

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624115538545-1683267808.png)

## 6.9 手动签发

在 kubelet 首次启动后，如果用户 Token 没问题，并且 RBAC 也做了相应的设置，那么此时在集群内应该能看到 kubelet 发起的 CSR 请求 ，必须通过后kubernetes 系统才会将该 Node 加入到集群。查看未授权的CSR 请求:

```bash
# kubectl get csr
NAME                                                   AGE       REQUESTOR           CONDITION
node-csr--k3G2G1EoM4h9w1FuJRjJjfbIPNxa551A8TZfW9dG-g   2m        kubelet-bootstrap   Pending
 
# kubectl get nodes
No resources found.
```

通过CSR 请求：

```bash
# kubectl certificate approve node-csr--k3G2G1EoM4h9w1FuJRjJjfbIPNxa551A8TZfW9dG-g
certificatesigningrequest "node-csr--k3G2G1EoM4h9w1FuJRjJjfbIPNxa551A8TZfW9dG-g" approved
 
# kubectl get nodes
NAME            STATUS    ROLES     AGE       VERSION
172.30.195.89   Ready     <none>    48s       v1.10.0
```

自动生成了kubelet kubeconfig 文件和公私钥：

```bash
# ls -l /etc/kubernetes/kubelet.kubeconfig
-rw------- 1 root root 2280 Nov  7 10:26 /etc/kubernetes/kubelet.kubeconfig
 
# ls -l /etc/kubernetes/ssl/kubelet*
-rw-r--r-- 1 root root 1046 Nov  7 10:26 /etc/kubernetes/ssl/kubelet-client.crt
-rw------- 1 root root  227 Nov  7 10:22 /etc/kubernetes/ssl/kubelet-client.key
-rw-r--r-- 1 root root 1115 Nov  7 10:16 /etc/kubernetes/ssl/kubelet.crt
-rw------- 1 root root 1675 Nov  7 10:16 /etc/kubernetes/ssl/kubelet.key
```

当成功签发证书后，目标节点的 kubelet 会将证书写入到 --cert-dir= 选项指定的目录中；注意此时如果不做其他设置应当生成四个文件 .kubelet 与 apiserver 通讯所使用的证书为 kubelet-client.crt，剩下的 kubelet.crt 将会被用于 kubelet server(10250) 做鉴权使用；注意，此时 kubelet.crt 这个证书是个独立于 apiserver CA 的自签 CA，并且删除后 kubelet 组件会重新生成它。

## 6.10 自动签发

上面提到，kubelet首次启动时会发起CSR请求，如果我们未做任何配置，则需要手动签发，若集群庞大，那么手动签发的请求就会很多，来了解一下自动签发

## 6.11 RBAC授权

kubelet 所发起的 CSR 请求是由 controller manager 签署的；如果想要是实现自动签发，就需要让 controller manager 能够在 kubelet 发起证书请求的时候自动帮助其签署证书；那么 controller manager 不可能对所有的 CSR 证书申请都自动签署，这时候就需要配置 RBAC 规则，保证 controller manager 只对 kubelet 发起的特定 CSR 请求自动批准即可；针对上面 提出的 3 种 CSR 请求分别给出了 3 种对应的 ClusterRole，如下所示:

```bash
# A ClusterRole which instructs the CSR approver to approve a user requesting
# node client credentials.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: approve-node-client-csr
rules:
- apiGroups: ["certificates.k8s.io"]
  resources: ["certificatesigningrequests/nodeclient"]
  verbs: ["create"]
---
# A ClusterRole which instructs the CSR approver to approve a node renewing its
# own client credentials.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: approve-node-client-renewal-csr
rules:
- apiGroups: ["certificates.k8s.io"]
  resources: ["certificatesigningrequests/selfnodeclient"]
  verbs: ["create"]
---
# A ClusterRole which instructs the CSR approver to approve a node requesting a
# serving cert matching its client cert.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: approve-node-server-renewal-csr
rules:
- apiGroups: ["certificates.k8s.io"]
  resources: ["certificatesigningrequests/selfnodeserver"]
  verbs: ["create"]
```

RBAC 中 ClusterRole 只是描述或者说定义一种集群范围内的能力，这三个 ClusterRole 在 1.7 之前需要自己手动创建，在 1.8 后 apiserver 会自动创建前两个；以上三个 ClusterRole 含义如下：

- approve-node-client-csr: 具有自动批准 nodeclient 类型 CSR 请求的能力;
- approve-node-client-renewal-csr: 具有自动批准 selfnodeclient 类型 CSR 请求的能力;
- approve-node-server-renewal-csr: 具有自动批准 selfnodeserver 类型 CSR 请求的能力;

![img](https://img2018.cnblogs.com/blog/907596/201906/907596-20190624122821443-2018224117.png)

所以，如果想要 kubelet 能够自动签发，那么就应当将适当的 ClusterRole 绑定到 kubelet 自动续期时所所采用的用户或者用户组身上。

## 6.12 CluserRole 绑定

要实现自动签发，创建的 RBAC 规则，则至少能满足四种情况:
自动批准 kubelet 首次用于与 apiserver 通讯证书的 CSR 请求(nodeclient)
自动批准 kubelet 首次用于 10250 端口鉴权的 CSR 请求(实际上这个请求走的也是 selfnodeserver 类型 CSR)

基于以上2种情况，实现自动签发需要创建 2个 ClusterRoleBinding，创建如下:

```bash
自动批准 kubelet 的首次 CSR 请求(用于与 apiserver 通讯的证书)
# kubectl create clusterrolebinding node-client-auto-approve-csr --clusterrole=approve-node-client-csr --group=system:bootstrappers
 
自动批准 kubelet 发起的用于 10250 端口鉴权证书的 CSR 请求(包括后续 renew)
# kubectl create clusterrolebinding node-server-auto-renew-crt --clusterrole=approve-node-server-renewal-csr --group=system:nodes
```

**证书轮换**
**开启证书轮换下的引导过程**

- kubelet 读取 bootstrap.kubeconfig，使用其 CA 与 Token 向 apiserver 发起第一次 CSR 请求(nodeclient)；
- apiserver 根据 RBAC 规则自动批准首次 CSR 请求(approve-node-client-csr)，并下发证书(kubelet-client.crt)；
- kubelet 使用刚刚签发的证书(O=system:nodes, CN=system:node:NODE_NAME)与 apiserver 通讯，并发起申请 10250 server 所使用证书的 CSR 请求；
- apiserver 根据 RBAC 规则自动批准 kubelet 为其 10250 端口申请的证书(kubelet-server-current.crt)；
- 证书即将到期时，kubelet 自动向 apiserver 发起用于与 apiserver 通讯所用证书的 renew CSR 请求和 renew 本身 10250 端口所用证书的 CSR 请求；
- apiserver 根据 RBAC 规则自动批准两个证书；
- kubelet 拿到新证书后关闭所有连接，reload 新证书，以后便一直如此；

**从以上流程我们可以看出，实现证书轮换创建 的RBAC 规则，则至少能满足四种情况:**

- 自动批准 kubelet 首次用于与 apiserver 通讯证书的 CSR 请求(nodeclient)；
- 自动批准 kubelet 首次用于 10250 端口鉴权的 CSR 请求(实际上这个请求走的也是 selfnodeserver 类型 CSR)；
- 自动批准 kubelet 后续 renew 用于与 apiserver 通讯证书的 CSR 请求(selfnodeclient)；
- 自动批准 kubelet 后续 renew 用于 10250 端口鉴权的 CSR 请求(selfnodeserver)；

基于以上四种情况，我们只需在开启了自动签发的基础增加一个ClusterRoleBinding:

```bash
自动批准 kubelet 后续 renew 用于与 apiserver 通讯证书的 CSR 请求
# kubectl create clusterrolebinding node-client-auto-renew-crt --clusterrole=approve-node-client-renewal-csr --group=system:nodes
```

## 6.13 开启证书轮换的配置

kubelet 启动时增加 --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true 选项，则 kubelet 在证书即将到期时会自动发起一个 renew 自己证书的 CSR 请求；增加--rotate-certificates 参数，kubelet 会自动重载新证书；

同时 controller manager 需要在启动时增加 --feature-gates=RotateKubeletServerCertificate=true 参数，再配合上面创建好的 ClusterRoleBinding，kubelet client 和 kubelet server 证才书会被自动签署；

## 6.14 证书过期时间

TLS bootstrapping 时的证书实际是由 kube-controller-manager 组件来签署的，也就是说证书有效期是 kube-controller-manager 组件控制的；kube-controller-manager 组件提供了一个 --experimental-cluster-signing-duration 参数来设置签署的证书有效时间；默认为 8760h0m0s，将其改为 87600h0m0s 即 10 年后再进行 TLS bootstrapping 签署证书即可。

# 七、TLS Bootstrapping总结

**=========================== 流程总结 =========================**

1. kubelet 首次启动通过加载 bootstrap.kubeconfig 中的用户 Token 和 apiserver CA 证书发起首次 CSR 请求，这个 Token 被预先内置在 apiserver 节点的 token.csv 中，其身份为 kubelet-bootstrap 用户和 system:bootstrappers 用户组；想要首次 CSR 请求能成功(成功指的是不会被 apiserver 401 拒绝)，则需要先将 kubelet-bootstrap 用户和 system:node-bootstrapper 内置 ClusterRole 绑定；
2. 对于首次 CSR 请求可以手动签发，也可以将 system:bootstrappers 用户组与 approve-node-client-csr ClusterRole 绑定实现自动签发(1.8 之前这个 ClusterRole 需要手动创建，1.8 后 apiserver 自动创建，并更名为 system:certificates.k8s.io:certificatesigningrequests:nodeclient)
3. 默认签署的的证书只有 1 年有效期，如果想要调整证书有效期可以通过设置 kube-controller-manager 的 --experimental-cluster-signing-duration 参数实现，该参数默认值为 8760h0m0s。
4. 对于证书轮换，需要通过协调两个方面实现；第一，想要 kubelet 在证书到期后自动发起续期请求，则需要在 kubelet 启动时增加 --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true 来实现；第二，想要让 controller manager 自动批准续签的 CSR 请求需要在 controller manager 启动时增加 --feature-gates=RotateKubeletServerCertificate=true 参数，并绑定对应的 RBAC 规则；同时需要注意的是 1.7 版本的 kubelet 自动续签后需要手动重启 kubelet 以使其重新加载新证书，而 1.8 后只需要在 kublet 启动时附带 --rotate-certificates 选项就会自动重新加载新证书。

# 八、kubernetes配置总结

apiserver 预先放置 token.csv，内容样例如下:

```bash
6df3c701f979cee17732c30958745947,kubelet-bootstrap,10001,"system:bootstrappers"
```

允许 kubelet-bootstrap 用户创建首次启动的 CSR 请求 和RBAC授权规则

```bash
# kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --user=kubelet-bootstrap
   
# kubectl create clusterrolebinding kubelet-nodes \
  --clusterrole=system:node \
  --group=system:nodes
```

配置 kubelet 自动续期，RotateKubeletClientCertificate 用于自动续期 kubelet 连接 apiserver 所用的证书(kubelet-client-xxxx.pem)，RotateKubeletServerCertificate 用于自动续期 kubelet 10250 api 端口所使用的证书(kubelet-server-xxxx.pem)，--rotate-certificates 选项使得 kubelet 能够自动重载新证书。

```bash
KUBELET_ARGS="--cgroup-driver=cgroupfs \
              --cluster-dns=10.254.0.2 \
              --resolv-conf=/etc/resolv.conf \
              --experimental-bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
              --feature-gates=RotateKubeletClientCertificate=true,RotateKubeletServerCertificate=true \
              --rotate-certificates \
              --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \
              --fail-swap-on=false \
              --cert-dir=/etc/kubernetes/ssl \
              --cluster-domain=cluster.local. \
              --hairpin-mode=promiscuous-bridge \
              --serialize-image-pulls=false \
              --pod-infra-container-image=gcr.io/google_containers/pause-amd64:3.0"
```

配置 controller manager 自动批准相关 CSR 请求，如果不配置 --feature-gates=RotateKubeletServerCertificate=true 参数，则即使配置了相关的 RBAC 规则，也只会自动批准 kubelet client 的 renew 请求：

```bash
KUBE_CONTROLLER_MANAGER_ARGS="--address=0.0.0.0 \
                              --service-cluster-ip-range=10.254.0.0/16 \
                              --cluster-name=kubernetes \
                              --cluster-signing-cert-file=/etc/kubernetes/ssl/k8s-root-ca.pem \
                              --cluster-signing-key-file=/etc/kubernetes/ssl/k8s-root-ca-key.pem \
                              --service-account-private-key-file=/etc/kubernetes/ssl/k8s-root-ca-key.pem \
                              --feature-gates=RotateKubeletServerCertificate=true \
                              --root-ca-file=/etc/kubernetes/ssl/k8s-root-ca.pem \
                              --leader-elect=true \
                              --experimental-cluster-signing-duration 10m0s \
                              --node-monitor-grace-period=40s \
                              --node-monitor-period=5s \
                              --pod-eviction-timeout=5m0s"
```

创建自动批准相关 CSR 请求的 ClusterRole，1.8 的 apiserver 自动创建了前两条 ClusterRole，所以只需要创建一条就行了。

```bash
# A ClusterRole which instructs the CSR approver to approve a node requesting a
# serving cert matching its client cert.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
rules:
- apiGroups: ["certificates.k8s.io"]
  resources: ["certificatesigningrequests/selfnodeserver"]
  verbs: ["create"]
```

将 ClusterRole 绑定到适当的用户组，以完成自动批准相关 CSR 请求

```bash
自动批准 system:bootstrappers 组用户 TLS bootstrapping 首次申请证书的 CSR 请求
# kubectl create clusterrolebinding node-client-auto-approve-csr --clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient --group=system:bootstrappers
 
自动批准 system:nodes 组用户更新 kubelet 自身与 apiserver 通讯证书的 CSR 请求
# kubectl create clusterrolebinding node-client-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient --group=system:nodes
 
自动批准 system:nodes 组用户更新 kubelet 10250 api 端口证书的 CSR 请求
# kubectl create clusterrolebinding node-server-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeserver --group=system:nodes
```