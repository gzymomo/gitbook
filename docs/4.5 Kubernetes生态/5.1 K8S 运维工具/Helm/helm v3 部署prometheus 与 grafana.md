- [helm v3 部署prometheus 与 grafana](https://blog.51cto.com/flyfish225/3569682)

# helm v3 部署prometheus 与 grafana

## 一： k8s 的环境简介

```html
系统： CentOS7.9x64
k8s 的版本： k8s 1.18.20
```

![image_1fdcbjog118la14d69gk1v595s5m.png-43.7kB](https://s4.51cto.com/images/blog/202108/20/00e9b27b6e19f0a93a6cc896db86d2cc.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdcbj68n17aietg1s7710rpv19.png-69.8kB](https://s4.51cto.com/images/blog/202108/20/4e02c017c9b455641ab90bca5b0edf87.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 二： helm 3.4.2 部署

```html
# helm3 下载地址
  https://github.com/helm/helm/releases
# 下载后解压获取二进制文件即可使用

tar -zxvf helm-v3.4.2-linux-amd64.tar.gz
cd linux-amd64/
mv helm /usr/bin/
chmod +x /usr/bin/helm
helm version 
```

![image_1fdcbnopojobi9k1ggm1h041jca13.png-152.3kB](https://s4.51cto.com/images/blog/202108/20/f70e38debb4bf739d0986433ba13dd73.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

------

## 三：添加promethues 的阿里云helm 仓库

```html
helm  repo add aliyuncs https://apphub.aliyuncs.com

helm repo list
```

![image_1fdcbrfi8135f14or1fiegfstl21g.png-79.5kB](https://s4.51cto.com/images/blog/202108/20/70b4a074c12f30a303dc9f9cdbaf9473.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 四：安装promtheus

```html
建一个namespaces:

kubectl create ns monitoring

helm install prometheus aliyuncs/prometheus-operator \
--set prometheus.service.type=NodePort \
--set prometheus.service.nodePort=30090 \
--namespace monitoring 
```

![image_1fdcbvj99iisd4q1il09d61i8i1t.png-184.9kB](https://s4.51cto.com/images/blog/202108/20/1622ed06632cff340608efacbbbe7604.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl get pod -n monitoring 
```

![image_1fdcc05og1sj2vmbfv9jo0sl2a.png-137.5kB](https://s4.51cto.com/images/blog/202108/20/8aa06d9bb9fd911486281186a20b8d6a.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl get svc -n monitoring
```

![image_1fdcc2e0bfrm1fckhbr1c821bfo34.png-145.6kB](https://s4.51cto.com/images/blog/202108/20/d5e36e4664b15e7c74c4b32d1902f046.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
打开web

http://192.168.100.11:30090
```

![image_1fdcc41tv1abd1kc21n653o17bm3h.png-318.3kB](https://s4.51cto.com/images/blog/202108/20/3a0d9ed1c8f4be770742b886a9433d9c.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdcc5qg61nhhef21q1s12112k03u.png-437.5kB](https://s4.51cto.com/images/blog/202108/20/5d3078b4f96ab0fcf93873bbea9a9500.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

## 五： 关于内置grafana 的升级与端口暴露：

```html
kubectl get pod -n monitoring
```

![image_1fdh036v421lgo2m8t14aif1e9.png-40.1kB](https://s4.51cto.com/images/blog/202108/20/e8c2943dd154a81b52f2ccd1e999f601.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubeclt get deploy -n monitoring
```

![image_1fdh05tbf1df71itr96j1f5mam3m.png-21kB](https://s4.51cto.com/images/blog/202108/20/e49ed989dad825351e9bd12f66652938.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl edit deploy prometheus-grafana -n monitoring
---
grafana的镜像调整为最新版本：
 image: grafana/grafana：6.5.2 
 改为 image: grafana/grafana
---
```

![image_1fdh094ln1lbi9611it7deu4bb13.png-13.3kB](https://s4.51cto.com/images/blog/202108/20/1eb2921a3ff123c2e0491f5c443c0397.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
会从新初始化grafana 的pod
```

![image_1fdh0cpv8i1212p6124q1t3j1prc1g.png-42.4kB](https://s4.51cto.com/images/blog/202108/20/ef653aebb198fedf5f35116c1932bb40.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdh0f048top59d120p177vl831t.png-43.6kB](https://s4.51cto.com/images/blog/202108/20/e2f4fad931ff7e39660f9d1aab78cfc3.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdh0fknm1qgot6nfm94431usf2a.png-37.5kB](https://s4.51cto.com/images/blog/202108/20/ac904522c5e0d74457e1401852e0ba1b.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
关于grafana svc 的端口暴露
kubectl get svc -n monitoring 
kubectl edit svc prometheus-grafana -n monitoring
```

![image_1fdh328ttir4b8s7m6qbt12hd2n.png-45.3kB](https://s4.51cto.com/images/blog/202108/20/4b5fa2265e98d55549eeb22a15d006f1.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdh32o2r148eb3m1j8a1ok7u3a34.png-35.6kB](https://s4.51cto.com/images/blog/202108/20/f2f7e9069cd7e247afb9eb56e996b3e2.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl edit svc prometheus-grafana -n monitoring
type: ClusterIP 改为：
type: NodePort
```

![image_1fdh33c1a19c21hrg1hglm2h15dr3h.png-21kB](https://s4.51cto.com/images/blog/202108/20/3a5ed44ce652b3cce5ed12b4c0815201.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl get svc -n monitoring 
```

![image_1fdhaeu2lk48kg140851419on3u.png-45.5kB](https://s4.51cto.com/images/blog/202108/20/ca43ba85ac9f5f678d4ec2d1485b6a61.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdhahdaf1nj38071qhg3utvr44b.png-147.3kB](https://s4.51cto.com/images/blog/202108/20/5ec8083195d23c5859c2de1910fa826a.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
这个地方的用户名密码：
  kubectl get secret -n monitoring 
```

![image_1fdhaj9ka1a3f97a1n34dii314o.png-67.6kB](https://s4.51cto.com/images/blog/202108/20/36b9102dc4cb351c677a10edf5349022.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
kubectl edit secret prometheus-grafana -n monitoring
---
  admin-password: cHJvbS1vcGVyYXRvcg==
  admin-user: YWRtaW4=
---
base64 位的转码：
echo -n YWRtaW4= | base64 --decode
echo -n cHJvbS1vcGVyYXRvcg== | base64 --decode
```

![image_1fdhapakt1olf19deicf1oo612pt55.png-40.5kB](https://s4.51cto.com/images/blog/202108/20/6301dbf6e6681239774b60e6e6223d03.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdhatp1g1kcr1ek977m1q851j4b5i.png-25.2kB](https://s4.51cto.com/images/blog/202108/20/9e6d135e43497979b11d85fe7af11f7b.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

```html
用户名：admin
密码： prom-operator
```

![image_1fdhc665s1uhq1ih31u2e10se17qk6p.png-114.5kB](https://s4.51cto.com/images/blog/202108/20/6ef6c373924802bcd38780a6a10d624e.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdhb03ve1k641l9e1bd01muj176r5v.png-101.4kB](https://s4.51cto.com/images/blog/202108/20/a229316d52ebdd02b1997547d49e1c9e.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

![image_1fdhb0rpv1ujurk5pju174c1a2p6c.png-121.2kB](https://s4.51cto.com/images/blog/202108/20/5f961900f837c4a58afd1439312bd93a.png?x-oss-process=image/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

​        