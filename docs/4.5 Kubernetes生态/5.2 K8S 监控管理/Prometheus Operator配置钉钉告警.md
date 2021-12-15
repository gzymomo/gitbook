- [Prometheus Operator配置钉钉告警](https://www.jianshu.com/p/f0987acb929a)



## 配置钉钉告警

1、注册钉钉账号->机器人管理->自定义（通过webhook接入自定义服务）->添加->复制webhook

![](https://upload-images.jianshu.io/upload_images/15900165-48616a18935f82da.png?imageMogr2/auto-orient/strip|imageView2/2/w/994)

 上述配置好群机器人，获得这个机器人对应的Webhook地址，记录下来，后续配置钉钉告警插件要用，格式如下
[https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx](https://links.jianshu.com/go?to=https%3A%2F%2Foapi.dingtalk.com%2Frobot%2Fsend%3Faccess_token%3Dxxxxxxxx)
 2、创建钉钉告警插件（dingtalk-webhook.yaml），并修改文件中 access_token=xxxxxx 为上一步你获得的机器人认证 token
 到安装包的路径下创建告警信息。

```yaml
cd /k8s-cmp/yaml/prometheus_Operator/kube-prometheus/manifests
$ vim dingtalk-webhook.yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    run: dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  replicas: 1
  template:
    metadata:
      labels:
        run: dingtalk
    spec:
      containers:
      - name: dingtalk
        image: timonwong/prometheus-webhook-dingtalk:v0.3.0
        imagePullPolicy: IfNotPresent
        # 设置钉钉群聊自定义机器人后，使用实际 access_token 替换下面 xxxxxx部分
        args:
          - --ding.profile=webhook1=https://oapi.dingtalk.com/robot/send?access_token=94c9f3664df1a928cb59550ac88caf504ca1808a22e7018fdcf92c50d9960fab
        ports:
        - containerPort: 8060
          protocol: TCP

---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: dingtalk
  name: webhook-dingtalk
  namespace: monitoring
spec:
  ports:
  - port: 8060
    protocol: TCP
    targetPort: 8060
  selector:
    run: dingtalk
  sessionAffinity: None
```

3、应用dingtalk-webhook.yaml

```bash
$ kubectl apply -f dingtalk-webhook.yaml
```

4、添加告警接收器
 到安装包的路径下创建告警接收器。（alertmanager.yaml ）

```yaml
cd /k8s-cmp/yaml/prometheus_Operator/kube-prometheus/manifests
vim alertmanager.yaml 
global:
  resolve_timeout: 5m
route:
  group_by: ['job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: webhook
receivers:
- name: 'webhook'
  webhook_configs:
  - url: 'http://webhook-dingtalk.monitoring.svc.cluster.local:8060/dingtalk/webhook1/send'
    send_resolved: true
```

注：上述配置url: '[http://webhook-dingtalk.monitoring.svc.cluster.local:8060/dingtalk/webhook1/send'](https://links.jianshu.com/go?to=http%3A%2F%2Fwebhook-dingtalk.monitoring.svc.cluster.local%3A8060%2Fdingtalk%2Fwebhook1%2Fsend') 是dingtalk-webhook.yaml文件中svc的地址。
 5、替换原有secret

```bash
cd /k8s-cmp/yaml/prometheus_Operator/kube-prometheus/manifests
kubectl delete  secret alertmanager-main -n monitoring
kubectl create  secret generic alertmanager-main --from-file=alertmanager.yaml -n monitoring
```