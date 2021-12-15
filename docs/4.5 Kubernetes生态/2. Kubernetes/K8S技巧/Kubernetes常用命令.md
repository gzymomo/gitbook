# Kubernetes常用命令

## 查看：

```bash
# 查看Pod
kubectl get pods
# 查看services
kubectl get services
# 查看pvc
kubectl get pvc
# 查看ConfigMap
kubectl get configmaps
# 查看Ingress
kubectl get ingress
```

## 查看扩展信息：

```bash
kubectl get pods -o wide
```

## 删除Pod：

```bash
kubectl delete pod podName
# 删除失败的Pod
kubectl get pods | grep Error | awk '{print $1}' | xargs kubectl delete pod
# 删除不在运行的Pod
kubectl delete pod --field-selector=status.phase!=Running
```

## 查看详情：

```bash
kubectl describe quota
kubectl describe pod podName
kubectl describe node docker-desktop
```

## API方式操作资源：

```bash
# 指定文件
kubectl apply -f pkslow-springboot-deployment.yaml
# 指定目录
kubectl apply -f /opt/pkslow/k8s
```

## 查看日志：

```bash
kubectl logs podName
kubectl logs -f podName
```

## 进入Pod执行命令：

```bash
kubectl exec
```

## 指定命令空间：

```bash
kubectl get pods -n pkslow-namespace
```

## 修改默认命令空间：

```bash
kubectl config set-context --current --namespace=pkslow-namespace
```

## 命令空间基本操作：

```bash
# 查看
kubectl get namespaces
# 创建
kubectl create namespace pkslow
# 删除
kubectl delete namespace pkslow
```

## 文件管理：

```bash
# 从客户端复制文件或目录到Pod
kubectl cp pkslow.txt h2-db-5967bf999f-8qr87:/opt/h2-data
kubectl cp pkslow h2-db-5967bf999f-8qr87:/opt/h2-data

# 从Pod复制文件回来
# 目标目录要指定，与源文件类型匹配
$ kubectl cp default/h2-db-5967bf999f-8qr87:/opt/h2-data/pkslow ./pkslow
# 目标文件要指定，与源文件类型匹配
$ kubectl cp default/h2-db-5967bf999f-8qr87:/opt/h2-data/pkslow.txt ./pkslow.txt
```

## 查看集群的Resources：

```bash
kubectl api-resources
```