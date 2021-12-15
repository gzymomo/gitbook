- [Docker容器管理平台Rancher高可用部署](https://www.cnblogs.com/wellful/p/14368038.html)



Rancher 是为使用容器的公司打造的容器管理平台。

也许有人又会问：用K8s不香吗，为什么要加一层RANCHER？其实K8s是比较复杂的，尤其是各组件的版本选择以及大规模的使用场景上，需要一个管理平台降低容器化落地的复杂度。



**【说明】：**

- 当前Rancher版本为：V2.5.5 ;
- 若只想通过Docker安装单机版体验，可执行：

```
docker run -d --privileged --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher``/rancher``:latest 
```

- 下面部署的Rancher集群证书均使用自签名私有证书。
- 中文官网文档：https://docs.rancher.cn/rancher2/ 。
- 通过Rancher导入K8s集群时，需要预先在目标服务器上安装Docker。



**【规划】：**

- 用2台centos7部署k3s+rancher（做高可用），IP分别为：192.168.21.30，192.168.21.31；
- 用1台centos部署mysql5.7存储rancher数据，同时部署nginx代理2台rancher，IP为：192.168.20.101；
- 配置dns域名：rancher.test.cn - 192.168.20.101 ;



**【安装K3s（K8s）】**

　　说明：什么是K3s？K3s是K8s的精简版，很多时候我们并不需要用到K8s的所有功能，Rancher公司帮我们做了一款精简版的K8s，即K3s，推荐在K3s上部署Rancher。　　

　　**一、准备工作**

　　1. **内核升级：**

　　　　建议所有K8s节点（包括K3s、k8s master worker节点）内核均升级到最新稳定版（目前是kernel5.4），好处请自行Google。

　　　　参考：https://github.com/easzlab/kubeasz/blob/master/docs/guide/kernel_upgrade.md

　　　　centos7内核升级步骤：

```bash
# 载入公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# 安装ELRepo
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
# 载入elrepo-kernel元数据
yum --disablerepo=\* --enablerepo=elrepo-kernel repolist
# 查看可用的rpm包
yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*
# 安装长期支持版本的kernel
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-lt.x86_64
# 删除旧版本工具包
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64 -y
# 安装新版本工具包
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-lt-tools.x86_64

#查看默认启动顺序
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg  
CentOS Linux (4.4.183-1.el7.elrepo.x86_64) 7 (Core)  
CentOS Linux (3.10.0-327.10.1.el7.x86_64) 7 (Core)  
CentOS Linux (0-rescue-c52097a1078c403da03b8eddeac5080b) 7 (Core)
#默认启动的顺序是从0开始，新内核是从头插入（目前位置在0，而4.4.4的是在1），所以需要选择0。
grub2-set-default 0  
#重启并检查
reboot
```

　2. **修改主机名：**

```bash
hostnamectl set-hostname k3s01
echo "127.0.0.1  k3s01" >> /etc/hosts
```

 

　　3. **配置dns或者在每台服务器及个人电脑上添加hosts解析**（192.168.20.101 rancher.test.cn）；

 

　　4. **2台rancher节点上安装[helm](http://mirror.cnrancher.com/)：**

```bash
wget http://rancher-mirror.cnrancher.com/helm/v3.4.2/helm-v3.4.2-linux-amd64.tar.gz
tar zxf helm-v3.4.2-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/sbin/
```

 

　　5. **部署mysql5.7**，创建"rancher"库，创建用户rancher，密码为rancher，mysql端口为61306。

 

　　6. **部署nginx**（我是直接在mysql5.7服务器上部署，即IP为192.168.20.101这台机器），代理配置如下：

```
[root@rancher-proxy ~]# cat /etc/nginx/nginx.conf
worker_processes 4;
worker_rlimit_nofile 40000;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 8192;
}

stream {
    upstream rancher_servers_http {
        least_conn;
        server 192.168.21.30:80 max_fails=3 fail_timeout=5s;
        server 192.168.21.31:80 max_fails=3 fail_timeout=5s;
    }
    server {
        listen 80;
        proxy_pass rancher_servers_http;
    }

    upstream rancher_servers_https {
        least_conn;
        server 192.168.21.30:443 max_fails=3 fail_timeout=5s;
        server 192.168.21.31:443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     443;
        proxy_pass rancher_servers_https;
    }

    upstream rancher_servers_k8sapi {
        least_conn;
        server 192.168.21.30:6443 max_fails=3 fail_timeout=5s;
        server 192.168.21.31:6443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     6443;
        proxy_pass rancher_servers_k8sapi;
    }
}
```

 

　　**二、部署K3s**

　　1. 分别在2台Rancher节点上，运行以下命令以**启动 K3s Server 并将其连接到外部数据库**:

```bash
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_EXEC="--tls-san rancher.test.cn" INSTALL_K3S_MIRROR=cn sh -s - server --datastore-endpoint="mysql://rancher:rancher@tcp(192.168.20.101:61306)/rancher"
```

　　**说明：**需要**增加 INSTALL_K3S_EXEC="--tls-san rancher.test.cn" 这段，否则集群证书不认rancher.test.cn或者对应的IP**。

　　参考：https://github.com/k3s-io/k3s/issues/1381

 

　　2. **k3s创建成功确认**：

　　分别在2台节点上确认k3s是否创建成功：

```bash
k3s kubectl get nodes
```

　　测试集群容器的运行状况：

```bash
k3s kubectl get pods --all-namespaces
```

　　**此时会发现系统一直在创建镜像（网络超时），需要对容器加速。请参考第4步（镜像加速）。**

 

　　3. **保存并使用 kubeconfig 文件：**

　　备份/etc/rancher/k3s/k3s.yaml，并将其拷贝一份命名成~/.kube/config：

```bash
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

　　修改~/.kube/config配置文件中的server参数为rancher.test.cn，如下：

```yaml
[root@k3s02 ~]# cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    ……
    server: https://rancher.test.cn:6443
  ……
```

 　结果： 您现在可以使用kubectl来管理您的 K3s 集群。如果您有多个 kubeconfig 文件，可以在使用kubectl时通过传递文件路径来指定要使用的kubeconfig文件：

```bash
kubectl --kubeconfig ~/.kube/config get pods --all-namespaces
```

 

　　4. **镜像加速**

```yaml
cat >> /etc/rancher/k3s/registries.yaml <<EOF
mirrors:
  "docker.io":
    endpoint:
      - "https://fogjl973.mirror.aliyuncs.com"
      - "https://registry-1.docker.io"
EOF

systemctl restart k3s
```

　　再次执行下面命令，查看pod状态，发现各镜像开始下载：



```bash
k3s kubectl get pods --all-namespaces
```

 

 【**安装RANCHER】**

　　1. 在2台k3s节点上确保安装下面CLI工具：

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl) - Kubernetes 命令行工具；
- [helm](http://mirror.cnrancher.com/)，可到helm官网下载二进制包直接使用；

 

　　2. 添加 Helm Chart 仓库:

```bash
helm repo add rancher-stable http://rancher-mirror.oss-cn-beijing.aliyuncs.com/server-charts/stable
```

 

　　3.为 Rancher 创建 Namespace:

```bash
kubectl create namespace cattle-system
```

 

　　4.使用私有自签名证书:

　　使用自签名证书可以不用安装cert-manager，从而避免安装cert-manager很慢。

　　创建自动创建证书的脚本（脚本内容摘抄自Rancher官网，命名为create_self-signed-cert.sh）：

```bash
#!/bin/bash -e

help ()
{
    echo  ' ================================================================ '
    echo  ' --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；'
    echo  ' --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；'
    echo  ' --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（SSL_TRUSTED_DOMAIN）,多个扩展域名用逗号隔开；'
    echo  ' --ssl-size: ssl加密位数，默认2048；'
    echo  ' --ssl-cn: 国家代码(2个字母的代号),默认CN;'
    echo  ' 使用示例:'
    echo  ' ./create_self-signed-cert.sh --ssl-domain=www.test.com --ssl-trusted-domain=www.test2.com \ '
    echo  ' --ssl-trusted-ip=1.1.1.1,2.2.2.2,3.3.3.3 --ssl-size=2048 --ssl-date=3650'
    echo  ' ================================================================'
}

case "$1" in
    -h|--help) help; exit;;
esac

if [[ $1 == '' ]];then
    help;
    exit;
fi

CMDOPTS="$*"
for OPTS in $CMDOPTS;
do
    key=$(echo ${OPTS} | awk -F"=" '{print $1}' )
    value=$(echo ${OPTS} | awk -F"=" '{print $2}' )
    case "$key" in
        --ssl-domain) SSL_DOMAIN=$value ;;
        --ssl-trusted-ip) SSL_TRUSTED_IP=$value ;;
        --ssl-trusted-domain) SSL_TRUSTED_DOMAIN=$value ;;
        --ssl-size) SSL_SIZE=$value ;;
        --ssl-date) SSL_DATE=$value ;;
        --ca-date) CA_DATE=$value ;;
        --ssl-cn) CN=$value ;;
    esac
done

# CA相关配置
CA_DATE=${CA_DATE:-3650}
CA_KEY=${CA_KEY:-cakey.pem}
CA_CERT=${CA_CERT:-cacerts.pem}
CA_DOMAIN=cattle-ca

# ssl相关配置
SSL_CONFIG=${SSL_CONFIG:-$PWD/openssl.cnf}
SSL_DOMAIN=${SSL_DOMAIN:-'www.rancher.local'}
SSL_DATE=${SSL_DATE:-3650}
SSL_SIZE=${SSL_SIZE:-2048}

## 国家代码(2个字母的代号),默认CN;
CN=${CN:-CN}

SSL_KEY=$SSL_DOMAIN.key
SSL_CSR=$SSL_DOMAIN.csr
SSL_CERT=$SSL_DOMAIN.crt

echo -e "\033[32m ---------------------------- \033[0m"
echo -e "\033[32m       | 生成 SSL Cert |       \033[0m"
echo -e "\033[32m ---------------------------- \033[0m"

if [[ -e ./${CA_KEY} ]]; then
    echo -e "\033[32m ====> 1. 发现已存在CA私钥，备份"${CA_KEY}"为"${CA_KEY}"-bak，然后重新创建 \033[0m"
    mv ${CA_KEY} "${CA_KEY}"-bak
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
else
    echo -e "\033[32m ====> 1. 生成新的CA私钥 ${CA_KEY} \033[0m"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
fi

if [[ -e ./${CA_CERT} ]]; then
    echo -e "\033[32m ====> 2. 发现已存在CA证书，先备份"${CA_CERT}"为"${CA_CERT}"-bak，然后重新创建 \033[0m"
    mv ${CA_CERT} "${CA_CERT}"-bak
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
else
    echo -e "\033[32m ====> 2. 生成新的CA证书 ${CA_CERT} \033[0m"
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
fi

echo -e "\033[32m ====> 3. 生成Openssl配置文件 ${SSL_CONFIG} \033[0m"
cat > ${SSL_CONFIG} <<EOM
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
EOM

if [[ -n ${SSL_TRUSTED_IP} || -n ${SSL_TRUSTED_DOMAIN} ]]; then
    cat >> ${SSL_CONFIG} <<EOM
subjectAltName = @alt_names
[alt_names]
EOM
    IFS=","
    dns=(${SSL_TRUSTED_DOMAIN})
    dns+=(${SSL_DOMAIN})
    for i in "${!dns[@]}"; do
      echo DNS.$((i+1)) = ${dns[$i]} >> ${SSL_CONFIG}
    done


    if [[ -n ${SSL_TRUSTED_IP} ]]; then
        ip=(${SSL_TRUSTED_IP})
        for i in "${!ip[@]}"; do
          echo IP.$((i+1)) = ${ip[$i]} >> ${SSL_CONFIG}
        done
    fi
fi

echo -e "\033[32m ====> 4. 生成服务SSL KEY ${SSL_KEY} \033[0m"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE}

echo -e "\033[32m ====> 5. 生成服务SSL CSR ${SSL_CSR} \033[0m"
openssl req -sha256 -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/C=${CN}/CN=${SSL_DOMAIN}" -config ${SSL_CONFIG}

echo -e "\033[32m ====> 6. 生成服务SSL CERT ${SSL_CERT} \033[0m"
openssl x509 -sha256 -req -in ${SSL_CSR} -CA ${CA_CERT} \
    -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_DATE} -extensions v3_req \
    -extfile ${SSL_CONFIG}

echo -e "\033[32m ====> 7. 证书制作完成 \033[0m"
echo
echo -e "\033[32m ====> 8. 以YAML格式输出结果 \033[0m"
echo "----------------------------------------------------------"
echo "ca_key: |"
cat $CA_KEY | sed 's/^/  /'
echo
echo "ca_cert: |"
cat $CA_CERT | sed 's/^/  /'
echo
echo "ssl_key: |"
cat $SSL_KEY | sed 's/^/  /'
echo
echo "ssl_csr: |"
cat $SSL_CSR | sed 's/^/  /'
echo
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 9. 附加CA证书到Cert文件 \033[0m"
cat ${CA_CERT} >> ${SSL_CERT}
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 10. 重命名服务证书 \033[0m"
echo "cp ${SSL_DOMAIN}.key tls.key"
cp ${SSL_DOMAIN}.key tls.key
echo "cp ${SSL_DOMAIN}.crt tls.crt"
cp ${SSL_DOMAIN}.crt tls.crt
```

 　生成证书：

```bash
./create_self-signed-cert.sh --ssl-domain=rancher.test.cn --ssl-trusted-domain=rancher.test.cn --ssl-trusted-ip=192.168.20.101,192.168.21.30,192.168.21.31 --ssl-size=2048 --ssl-date=3650
```

　　生效：

```bash
kubectl -n cattle-system create secret tls tls-rancher-ingress   --cert=tls.crt   --key=tls.key
kubectl -n cattle-system create secret generic tls-ca   --from-file=cacerts.pem=./cacerts.pem
```

 

　　5.通过Helm安装Rancher

```bash
helm install rancher rancher-stable/rancher   --namespace cattle-system   --set hostname=rancher.test.cn   --set ingress.tls.source=secret   --set privateCA=true
```

　　说明：若执行该步骤遇到报错：

```
Error: Kubernetes cluster unreachable: Get "http://localhost:8080/version?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
```

　　可以执行下面步骤：

```bash
export KUBECONFIG=~/.kube/config
```

 　检查：

```bash
kubectl get pods --all-namespaces
helm ls --all-namespaces
```

 

　　6.验证 Rancher Server 是否已成功部署:

　　检查 Rancher Server 是否运行成功：

```bash
kubectl -n cattle-system rollout status deploy/rancher
```

　　若出现下面信息（successfully rolled out）则表示成功，否则继续等待：

```bash
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

 　检查 deployment 的状态：

```bash
kubectl -n cattle-system get deploy rancher
```

　　若出现下面信息则表示安装成功（DESIRED和AVAILABLE应该显示相同的个数）：

```bash
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
rancher 3 3 3 3 3m
```

 

**恭喜！**

　　到此安装完成，可以通过浏览器访问 [http://rancher.test.cn](http://rancher.leke.cn)

　　激动人心的时刻！

![img](https://img2020.cnblogs.com/blog/1227485/202102/1227485-20210203224144667-2050052851.png)

 感谢RANCHER公司为我们提供了这款开源、简洁易用、功能强大的容器管理平台！

 

**【附录】**

安装Docker：

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/epel-7.repo
yum install docker-ce -y
docker --version
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```



容器加速：

```json
[root@rancher-work02 ~]# cat /etc/docker/daemon.json 
{
 "registry-mirrors":["https://6kx4zyno.mirror.aliyuncs.com"]
}
```

生效容器加速配置：

```bash
systemctl daemon-reload 
systemctl restart docker
```