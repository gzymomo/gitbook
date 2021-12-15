- [docker已创建的容器如何修改环境变量](https://www.yuque.com/u1300481/kb/bzltu6)



# 1、前提

在生产环境中可能会使用docker run -e 传入相关参数，但是如果传的是密码相关信息，后期做了改动也要修改相关的环境变量，那该怎么办呢？

# 2、根据容器id找到容器相关的配置文件信息

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1557017/1614778227614-4b5002a0-f9bd-4f71-885e-1320c0404a08.png)

这个容器id为`6ac065232e94` ，默认docker的落盘文件位置为/var/lib/docker，此目录下文件结构如下：

- containers  容器相关
- image   镜像相关
- network  桥接网络映射相关
- overlay2  swarm网络相关
- plugins  插件相关
- swarm  swarm网络相关
- tmp  缓存
- trust  
- volumes 数据卷

## 2.1、containers下为所有创建的容器文件夹

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1557017/1614778449541-cd24136c-ff27-4692-b31e-8f0989177a52.png)

## 2.2、找到`6ac065232e94 `开头的打开，找到`config.v2.json`文件

下面为格式化后数据，env下为创建容器时传进去的值，对其进行修改

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1557017/1614778635159-00a05624-b674-40c5-9195-83596c6562bc.png)

## 2.3、关闭docker

```
systemctl stop docker
```

## 2.4、修改后重新copy到`config.v2.json` 启动docker

```
systemctl stop docker
```

## 2.5、重启容器

```
docker start 6ac065232e94
```