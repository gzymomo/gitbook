- [Traefik-v2.x快速入门](https://www.cnblogs.com/xiao987334176/p/12447783.html)



# 一、概述

`traefik` 与 `nginx` 一样，是一款优秀的反向代理工具，或者叫 `Edge Router`。至于使用它的原因则基于以下几点

- 无须重启即可更新配置
- 自动的服务发现与负载均衡
- 与 `docker` 的完美集成，基于 `container label` 的配置
- 漂亮的 `dashboard` 界面
- `metrics` 的支持，对 `prometheus` 和 `k8s` 的集成

接下来讲一下它的安装，基本功能以及配置。`traefik` 在 `v1` 与 `v2` 版本间差异过大，本篇文章采用了 `v2`

![img](https://img2020.cnblogs.com/blog/1341090/202003/1341090-20200309114646111-513828825.png)

traefik官方文档：https://docs.traefik.io/

注意:Traefikv2.0之后的版本在修改了很多bug之后也增加了新的特性，比如增加了TCP的支持，并且更换了新的WEB UI界面

# 二、快速开始

## 环境介绍

操作系统：centos7.6

数量：1台

docker版本：19.03.6

docker-compose版本：1.24.1

ip地址：192.168.28.218

## docker-compose启动

新建yaml文件

```
vi traefik-v2.1.yaml
```

内容如下：

```
version: '3'
services:
  reverse-proxy:
    image: traefik:2.1.6
    # Enables the web UI and tells Traefik to listen to docker
    # 启用webUI 并告诉Traefile去监听docker的容器实例
    command: --api.insecure=true --providers.docker
    ports:
      # traefik暴露的http端口
      - "80:80"
      # webUI暴露的端口(必须制定--api.insecure=true才可以访问)
      - "8080:8080"
    volumes:
      # 指定docker的sock文件来让traefik获取docker的事件，从而实现动态负载均衡
      - /var/run/docker.sock:/var/run/docker.sock
```

使用docker-compose创建集群

```
# docker-compose -f traefik-v2.1.yaml up -d reverse-proxy
Creating network "opt_default" with the default driver
Creating opt_reverse-proxy_1 ... done
```

查看使用docker-compose启动的应用

```
# docker-compose -f traefik-v2.1.yaml ps
       Name                      Command               State                     Ports                   
---------------------------------------------------------------------------------------------------------
opt_reverse-proxy_1   /entrypoint.sh --api.insec ...   Up      0.0.0.0:80->80/tcp, 0.0.0.0:8080->8080/tcp
```

直接访问traefik对外暴露的http接口

```
curl -s  "http://localhost:8080/api/rawdata" | python -m json.tool
```

输出如下：

```json
{
    "middlewares": {
        "dashboard_redirect@internal": {
            "redirectRegex": {
                "permanent": true,
                "regex": "^(http:\\/\\/[^:\\/]+(:\\d+)?)\\/$",
                "replacement": "${1}/dashboard/"
            },
            "status": "enabled",
            "usedBy": [
                "dashboard@internal"
            ]
        },
        "dashboard_stripprefix@internal": {
            "status": "enabled",
            "stripPrefix": {
                "prefixes": [
                    "/dashboard/",
                    "/dashboard"
                ]
            },
            "usedBy": [
                "dashboard@internal"
            ]
        }
    },
    "routers": {
        "api@internal": {
            "entryPoints": [
                "traefik"
            ],
            "priority": 2147483646,
            "rule": "PathPrefix(`/api`)",
            "service": "api@internal",
            "status": "enabled",
            "using": [
                "traefik"
            ]
        },
        "dashboard@internal": {
            "entryPoints": [
                "traefik"
            ],
            "middlewares": [
                "dashboard_redirect@internal",
                "dashboard_stripprefix@internal"
            ],
            "priority": 2147483645,
            "rule": "PathPrefix(`/`)",
            "service": "dashboard@internal",
            "status": "enabled",
            "using": [
                "traefik"
            ]
        },
        "reverse-proxy-opt@docker": {
            "rule": "Host(`reverse-proxy-opt`)",
            "service": "reverse-proxy-opt",
            "status": "enabled",
            "using": [
                "http",
                "traefik"
            ]
        }
    },
    "services": {
        "api@internal": {
            "status": "enabled",
            "usedBy": [
                "api@internal"
            ]
        },
        "dashboard@internal": {
            "status": "enabled",
            "usedBy": [
                "dashboard@internal"
            ]
        },
        "reverse-proxy-opt@docker": {
            "loadBalancer": {
                "passHostHeader": true,
                "servers": [
                    {
                        "url": "http://172.18.0.2:80"
                    }
                ]
            },
            "serverStatus": {
                "http://172.18.0.2:80": "UP"
            },
            "status": "enabled",
            "usedBy": [
                "reverse-proxy-opt@docker"
            ]
        }
    }
}
```

## 查看Traefik官方Dashboard

```
http://192.168.28.218:8080/
```

效果如下：

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309143801837-1005863445.png)

 

 

# 三、创建一个路由

Traefik来检测新服务并为你创建一个路由

### 创建一个新服务

```
vi test-service.yaml
```

内容如下：

```
version: '3'
services:
  whoami:
    image: containous/whoami
    labels:
      - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"
```

### 创建服务

```
# docker-compose -f test-service.yaml up -d whoami
WARNING: Found orphan containers (opt_reverse-proxy_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Creating opt_whoami_1 ... done
```

查看新创建的服务

```
# docker-compose  -f test-service.yaml ps
    Name       Command   State   Ports 
---------------------------------------
opt_whoami_1   /whoami   Up      80/tcp
```

再次查看traefik中的路由信息(就会发现服务自动加载进去了)
其实有点儿类似kong 的路由，只是traefik会自动监听docker的事件

```
curl -s  "http://localhost:8080/api/rawdata" | python -m json.tool
```

输出如下：

```json
{
    "middlewares": {
        "dashboard_redirect@internal": {
            "redirectRegex": {
                "permanent": true,
                "regex": "^(http:\\/\\/[^:\\/]+(:\\d+)?)\\/$",
                "replacement": "${1}/dashboard/"
            },
            "status": "enabled",
            "usedBy": [
                "dashboard@internal"
            ]
        },
        "dashboard_stripprefix@internal": {
            "status": "enabled",
            "stripPrefix": {
                "prefixes": [
                    "/dashboard/",
                    "/dashboard"
                ]
            },
            "usedBy": [
                "dashboard@internal"
            ]
        }
    },
    "routers": {
        "api@internal": {
            "entryPoints": [
                "traefik"
            ],
            "priority": 2147483646,
            "rule": "PathPrefix(`/api`)",
            "service": "api@internal",
            "status": "enabled",
            "using": [
                "traefik"
            ]
        },
        "dashboard@internal": {
            "entryPoints": [
                "traefik"
            ],
            "middlewares": [
                "dashboard_redirect@internal",
                "dashboard_stripprefix@internal"
            ],
            "priority": 2147483645,
            "rule": "PathPrefix(`/`)",
            "service": "dashboard@internal",
            "status": "enabled",
            "using": [
                "traefik"
            ]
        },
        "reverse-proxy-opt@docker": {
            "rule": "Host(`reverse-proxy-opt`)",
            "service": "reverse-proxy-opt",
            "status": "enabled",
            "using": [
                "http",
                "traefik"
            ]
        },
        "whoami@docker": {
            "rule": "Host(`whoami.docker.localhost`)",
            "service": "whoami-opt",
            "status": "enabled",
            "using": [
                "http",
                "traefik"
            ]
        }
    },
    "services": {
        "api@internal": {
            "status": "enabled",
            "usedBy": [
                "api@internal"
            ]
        },
        "dashboard@internal": {
            "status": "enabled",
            "usedBy": [
                "dashboard@internal"
            ]
        },
        "reverse-proxy-opt@docker": {
            "loadBalancer": {
                "passHostHeader": true,
                "servers": [
                    {
                        "url": "http://172.19.0.2:80"
                    }
                ]
            },
            "serverStatus": {
                "http://172.19.0.2:80": "UP"
            },
            "status": "enabled",
            "usedBy": [
                "reverse-proxy-opt@docker"
            ]
        },
        "whoami-opt@docker": {
            "loadBalancer": {
                "passHostHeader": true,
                "servers": [
                    {
                        "url": "http://172.19.0.3:80"
                    }
                ]
            },
            "serverStatus": {
                "http://172.19.0.3:80": "UP"
            },
           "status": "enabled",
            "usedBy": [
                "whoami@docker"
            ]
        }
    }
}
```

## 查看http反向代理记录

查看Traefik中的http反向代理记录，点击HTTP

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309144828660-1648565098.png)

 

## 测试traefik相关功能

 测试访问

```
# curl -H Host:whoami.docker.localhost http://localhost
Hostname: c334de4bc3c8
IP: 127.0.0.1
IP: 172.19.0.3
RemoteAddr: 172.19.0.2:57632
GET / HTTP/1.1
Host: whoami.docker.localhost
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.19.0.1
X-Forwarded-Host: whoami.docker.localhost
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 1ee8d25b3aac
X-Real-Ip: 172.19.0.1
```

## 单机扩容

```
# docker-compose -f test-service.yaml up -d --scale whoami=2
WARNING: Found orphan containers (opt_reverse-proxy_1) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
Starting opt_whoami_1 ... done
Creating opt_whoami_2 ... done
```

再次访问(就会发现自动负载到两个不同的实例上去了)

```
# curl -H Host:whoami.docker.localhost http://localhost
Hostname: c334de4bc3c8
IP: 127.0.0.1
IP: 172.19.0.3
RemoteAddr: 172.19.0.2:57632
GET / HTTP/1.1
Host: whoami.docker.localhost
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.19.0.1
X-Forwarded-Host: whoami.docker.localhost
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 1ee8d25b3aac
X-Real-Ip: 172.19.0.1
```

查看Traefike后端每个service的详情信息:

 ![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309150113026-995096663.png)

 

 就会看到2个service

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309150141245-1835274444.png)

 

 

# 四、Traefik配置介绍

## traefik配置结构图

![img](https://img2020.cnblogs.com/blog/1341090/202003/1341090-20200309150314398-389431203.png)

 

在traefik中的配置，会涉及到两方面内容:

- 动态的路由配置(即由k8s-api或docker相关api来自动发现服务的endpoint而进行路由的配置描述)
- 静态的启动配置(即traefik标准的启动配置参数)

注意:使用docker run traefik[:version] --help可查看traefik的配置参数

 

# 五、k8s部署Traefik

## 环境介绍

| 操作系统   | ip             | 主机名     | 配置  | 备注             |
| ---------- | -------------- | ---------- | ----- | ---------------- |
| centos 7.6 | 192.168.31.150 | k8s-master | 2核4G | Kubernetes1.16.3 |
| centos 7.6 | 192.168.31.178 | k8s-node01 | 2核8G | Kubernetes1.16.3 |

## yaml文件介绍

```
mkdir /opt/traefik
```

目录结构如下：

```
./
├── traefik-config.yaml
├── traefik-ds-v2.1.6.yaml
├── traefik-rbac.yaml
└── ui.yaml
```

traefik-config.yaml

```json
apiVersion: v1
kind: ConfigMap
metadata:
  name: traefik-config
  namespace: kube-system
data:
  traefik.toml: |
    defaultEntryPoints = ["http","https"]
    debug = false
    logLevel = "INFO"
    # Do not verify backend certificates (use https backends)
    InsecureSkipVerify = true
    [entryPoints]
      [entryPoints.http]
      address = ":80"
      compress = true
      [entryPoints.https]
      address = ":443"
        [entryPoints.https.tls]
    #Config to redirect http to https
    #[entryPoints]
    #  [entryPoints.http]
    #  address = ":80"
    #  compress = true
    #    [entryPoints.http.redirect]
    #    entryPoint = "https"
    #  [entryPoints.https]
    #  address = ":443"
    #    [entryPoints.https.tls]
    [web]
      address = ":8080"
    [kubernetes]
    [metrics]
      [metrics.prometheus]
      buckets=[0.1,0.3,1.2,5.0]
      entryPoint = "traefik"
    [ping]
    entryPoint = "http"
```

traefik-ds-v2.1.6.yaml

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: apps/v1
#apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller-v2
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  selector:
    matchLabels:
      name: traefik-ingress-lb-v2
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb-v2
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik:2.1.6
        name: traefik-ingress-lb-v2
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
          hostPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --api.insecure=true
        - --providers.kubernetesingress=true
        - --log.level=INFO
        #- --configfile=/config/traefik.toml
        #volumeMounts:
        #- mountPath: /config
        #  name: config
      volumes:
      - configMap:
          name: traefik-config
        name: config
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service-v2
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb-v2
spec:
  selector:
    k8s-app: traefik-ingress-lb-v2
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
```

traefik-rbac.yaml

```yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
    - extensions
    resources:
    - ingresses/status
    verbs:
    - update
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```

ui.yaml

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  - host: prod-traefik-ui.bgbiao.cn
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

## 部署

```
cd /opt/traefik
kubectl apply -f .
```

查看pod

```
# kubectl get pods -n kube-system | grep traefik
traefik-ingress-controller-v2-hz82b        1/1     Running   0          8m4s
```

 查看svc

```
# kubectl get svc -n kube-system | grep traefik
traefik-ingress-service-v2   ClusterIP   10.1.188.71    <none>        80/TCP,8080/TCP          8m56s
traefik-web-ui               ClusterIP   10.1.239.107   <none>        80/TCP                   46m
```

查看ingresses

```
# kubectl get ingresses.extensions -n kube-system
NAME             HOSTS                       ADDRESS   PORTS   AGE
traefik-web-ui   prod-traefik-ui.bgbiao.cn             80      48m
```

## 查看traefik的dashboard

### 域名访问

由于没有dns服务器，这里直接修改hosts来测试。windows 10添加一条hosts记录

```
192.168.31.178 prod-traefik-ui.bgbiao.cn 
```

注意：这里的192.168.31.178是node节点ip

效果如下：

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309203302013-523412256.png)

 

 

## ip方式

直接通过node ip+8080方式，比如：

```
http://192.168.31.178:8080
```

效果同上！

点击http

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309203415888-2083531292.png)

 

 查看 http service

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309203515930-2036088269.png)

 

 效果如下：

![img](https://img2020.cnblogs.com/i-beta/1341090/202003/1341090-20200309203530073-265369690.png)

 

 

注意:虽然traefikv2.x改动了很多，但是还是向下兼容一些内容的，比如我重新创建traefik-v2.0.1之后，之前创建的ingress规则会自动导入

 

本文参考链接：

https://www.jianshu.com/p/0fc6df85d00d

https://zhuanlan.zhihu.com/p/97420459