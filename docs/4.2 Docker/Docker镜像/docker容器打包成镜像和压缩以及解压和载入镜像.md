[TOC]

# 一、docker容器打包成镜像和压缩
## 1.1 将容器保存成镜像
` sudo docker commit -a 'sunmingyang' b4293c3b9202  mask_detectionv2:v2 `

## 1.2 将镜像打包
` docker save -o mask_detection_v5.tar mask_detection:v5 `

## 1.3 将镜像包压缩
` sudo tar -zcvf mask_detection_v5.tar.gz mask_detection_v5.tar `

## 1.4 利用管道打包压缩一步到位
` docker save mask_detection:v5 | gzip > mask_detection_v5.tar.gz `


# 二、docker镜像压缩包解压及镜像载入
## 2.1 压缩包解压
` tar -zxvf mask_detection_v5.tar.gz `

## 2.2 镜像载入
` sudo docker load -i mask_detection_v5.tar  `

# 二、将容器重新打包为镜像
```bash
// 找到运行中的容器 
docker ps
// 选择一个容器,进行打包为镜像
docker commit 容器id 设置打包为镜像的名字
// 找到打包的镜像
docker images
```