- [使用Lens部署nginx 服务](https://fizzz.blog.csdn.net/article/details/118555396)

### Lens的使用

每一个集群都可在最左侧，一个菜单按钮， 每一个集群也都有一个Workload的概览面板，
 显示有多少个pod， deployment，daemonsets， statefulSets。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708102619951.png)

删除资源 可以选中复选框，然后在右下角有一个`－`号的按钮，点击就可以删除。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708102941969.png)

编辑资源是在表格的最后一栏，点击`...` 可以显示操作按钮， 重启，编辑，删除，所有的编辑都是编辑资源的yaml文件，

**除了namespace资源外，其他资源的新增和编辑，都是使用yaml文件进行操作的。**

创建资源，比如创建一个 deployment， 需要点击最底部的一个 `+`号
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708103258411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dpdGh1Yl8zNTYzMTU0MA==,size_16,color_FFFFFF,t_70)
 说实话，这个新增资源的按钮，放这里，是不是担心别人找到这个新增功能啊，而且按钮颜色没有明显区分。我也是找了很久才找到。

点击加号按钮，可以打开两种终端一个是 进入cmd终端，一个是编写yaml文件的编辑器。
 在创建资源时Lens预设了很多资源模板
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708103537561.png)

下面启动一个nginx，并将服务暴露出来。

#### 创建的deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-0708
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-fizz
  template:
    metadata:
      labels:
        app: nginx-fizz
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

#### 创建Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-nginx2
  namespace: default
  labels:
    app: hello-nginx1
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: nginx-fizz
  type: NodePort
```

创建service是 type必须是 `NodePort`
 此外 几处的 `nginx-fizz` 是必须保持一致的。
 创建完成后，点击service 的详情，可以看到
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708104737961.png)

点击蓝色字体，就可以自动打开nginx的服务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210708104819701.png)