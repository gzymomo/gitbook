- [基于Kubernetes构建Jenkins微服务发布平台](https://www.cnblogs.com/yuezhimi/p/13091889.html)
- [kubernetes-jenkins CI/CD平台（十八）](https://www.cnblogs.com/yuezhimi/p/11083362.html)
- [Kubernetes+Helm+Jenkins 自动化发布项目](https://mp.weixin.qq.com/s?__biz=MzAwNTM5Njk3Mw==&mid=2247500119&idx=1&sn=c7234b0e7715aeb122a018b514ac6c5e&chksm=9b1fc1d5ac6848c3086a896b5ed536a14f1c9fb0c79c7c4773c5a1e9fdcd3b076c54b2afdf13&mpshare=1&scene=24&srcid=0421hIDd1CAsuqVkKdGqU2c5&sharer_sharetime=1618962188418&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)



# 一、发布流程设计

软件环境：Jenkins + Kubernetes + Gitlab + Harbor+helm

工作流程：手动/自动构建-> Jenkins 调度K8S API-＞动态生成Jenkins Slave pod -＞Slave pod 拉取Git 代码／编译／打包镜像-＞推送到镜像仓库Harbor -＞Slave 工作完成，Pod 自动销毁-＞helm部署到测试或生产Kubernetes平台。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611112650105-784913644.png)

## 1.1 准备基础前提环境

1. K8s（Ingress Controller，CoreDNS，PV自动供给）
2. Helm v3
3. Gitlab
4. Harbor，并启用Chart存储功能
5. MySQL（微服务数据库）
6. Eureka（注册中心）



# 二、在Kubernetes中部署Jenkins

参考：https://github.com/jenkinsci/kubernetes-plugin/tree/fc40c869edfd9e3904a9a56b0f80c5a25e988fa1/src/main/kubernetes

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611113039566-491460879.png)

安装插件: Git Parameter/Git/Pipeline/Config File Provider/kubernetes/Extended Choice Parameter

由于默认插件源在国外服务器，大多数网络无法顺利下载，需修改国内插件源地址：

```bash
cd jenkins_home/updates
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && \
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json
```

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611120306974-541048066.png)

## 2.1 Jenkins Pipeline 及参数化构建

参考：https://jenkins.io/doc/book/pipeline/syntax/

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611120514221-1592842527.png)

• Jenkins Pipeline是一套插件，支持在Jenkins中实现集成和持续交付管道；
• Pipeline通过特定语法对简单到复杂的传输管道进行建模；
• 声明式：遵循与Groovy相同语法。pipeline { }
• 脚本式：支持Groovy大部分功能，也是非常表达和灵活的工具。node { }
• Jenkins Pipeline的定义被写入一个文本文件，称为Jenkinsfile。

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611120603561-1796434446.png)

## 2.2 Jenkins在Kubernetes中动态创建代理

Jenkins Master/Slave架构

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611120742177-722336486.png)

在K8S中Jenkins Master/Slave架构

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611120837425-1424236806.png)

Kubernetes插件：Jenkins在Kubernetes集群中运行动态代理
插件介绍：https://github.com/jenkinsci/kubernetes-plugin

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611121642928-779182373.png)

## 2.3 自定义构建Jenkins Slave镜像

参考：https://github.com/jenkinsci/docker-jnlp-slave

```bash
FROM centos:7
LABEL maintainer ops

RUN yum install -y java-1.8.0-openjdk maven curl git libtool-ltdl-devel && \
    yum clean all && \
    rm -rf /var/cache/yum/* && \
    mkdir -p /usr/share/jenkins

COPY slave.jar /usr/share/jenkins/slave.jar
COPY jenkins-slave /usr/bin/jenkins-slave
COPY settings.xml /etc/maven/settings.xml
RUN chmod +x /usr/bin/jenkins-slave
COPY helm kubectl /usr/bin/

ENTRYPOINT ["jenkins-slave"]
```

构建jenkins-slave推送至harbor仓库

```bash
docker build . -t 192.168.0.241/library/jenkins-slave:jdk-1.8
docker push 192.168.0.241/library/jenkins-slave:jdk-1.8
```

##  2.4 基于Kubernetes构建Jenkins CI系统

添加凭据

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611141133483-2117797446.png)

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611141146944-1683638467.png)

**jenkinsfile**

**拉取代码 => 代码编译 => 单元测试 => 构建镜像 => Helm部署到K8S => 测试**

```yaml
#!/usr/bin/env groovy// 公共
def registry = "192.168.0.241"
// 项目
def project = "microservice"
def git_url = "http://192.168.0.138:12580/root/ms.git"
def gateway_domain_name = "gateway.xxxx.com"
def portal_domain_name = "portal.xxxx.com"
// 认证
def image_pull_secret = "registry-pull-secret"
def harbor_registry_auth = "c176b2b2-7eaf-4968-a1e0-8988d6e04057"
def git_auth = "03e08854-c546-43c1-82c9-d053a75953d2"
// ConfigFileProvider ID
def k8s_auth = "6e3f7dd8-6e26-42de-91c3-5b52ff3117ec"

pipeline {
  agent {
    kubernetes {
        label "jenkins-slave"
        yaml """
kind: Pod
metadata:
  name: jenkins-slave
spec:
  containers:
  - name: jnlp
    image: "${registry}/library/jenkins-slave:jdk-1.8"
    imagePullPolicy: Always
    volumeMounts:
      - name: docker-cmd
        mountPath: /usr/bin/docker
      - name: docker-sock
        mountPath: /var/run/docker.sock
      - name: maven-cache
        mountPath: /root/.m2
  volumes:
    - name: docker-cmd
      hostPath:
        path: /usr/bin/docker
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
    - name: maven-cache
      hostPath:
        path: /tmp/m2
"""
        }
      
      }
    parameters {
        gitParameter branch: '', branchFilter: '.*', defaultValue: 'master', description: '选择发布的分支', name: 'Branch', quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH'        
        extendedChoice defaultValue: 'none', description: '选择发布的微服务', \
          multiSelectDelimiter: ',', name: 'Service', type: 'PT_CHECKBOX', \
          value: 'gateway-service:9999,portal-service:8080,product-service:8010,order-service:8020,stock-service:8030'
        choice (choices: ['ms', 'demo'], description: '部署模板', name: 'Template')
        choice (choices: ['1', '3', '5', '7'], description: '副本数', name: 'ReplicaCount')
        choice (choices: ['ms'], description: '命名空间', name: 'Namespace')
    }
    stages {
        stage('拉取代码'){
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: "${params.Branch}"]], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [], submoduleCfg: [], 
                userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_url}"]]
                ])
            }
        }
        stage('代码编译') {
            // 编译指定服务
            steps {
                sh """
                  mvn clean package -Dmaven.test.skip=true
                """
            }
        }
        stage('构建镜像') {
          steps {
              withCredentials([usernamePassword(credentialsId: "${harbor_registry_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                sh """
                 docker login -u ${username} -p '${password}' ${registry}
                 for service in \$(echo ${Service} |sed 's/,/ /g'); do
                    service_name=\${service%:*}
                    image_name=${registry}/${project}/\${service_name}:${BUILD_NUMBER}
                    cd \${service_name}
                    if ls |grep biz &>/dev/null; then
                        cd \${service_name}-biz
                    fi
                    docker build -t \${image_name} .
                    docker push \${image_name}
                    cd ${WORKSPACE}
                  done
                """
                configFileProvider([configFile(fileId: "${k8s_auth}", targetLocation: "admin.kubeconfig")]){
                    sh """
                    # 添加镜像拉取认证
                    kubectl create secret docker-registry ${image_pull_secret} --docker-username=${username} --docker-password=${password} --docker-server=${registry} -n ${Namespace} --kubeconfig admin.kubeconfig |true
                    # 添加私有chart仓库
                    helm repo add  --username ${username} --password ${password} myrepo http://${registry}/chartrepo/${project}
                    """
                }
              }
          }
        }
        stage('Helm部署到K8S') {
          steps {
              sh """
              common_args="-n ${Namespace} --kubeconfig admin.kubeconfig"
              
              for service in  \$(echo ${Service} |sed 's/,/ /g'); do
                service_name=\${service%:*}
                service_port=\${service#*:}
                image=${registry}/${project}/\${service_name}
                tag=${BUILD_NUMBER}
                helm_args="\${service_name} --set image.repository=\${image} --set image.tag=\${tag} --set replicaCount=${replicaCount} --set imagePullSecrets[0].name=${image_pull_secret} --set service.targetPort=\${service_port} myrepo/${Template}"

                # 判断是否为新部署
                if helm history \${service_name} \${common_args} &>/dev/null;then
                  action=upgrade
                else
                  action=install
                fi

                # 针对服务启用ingress
                if [ \${service_name} == "gateway-service" ]; then
                  helm \${action} \${helm_args} \
                  --set ingress.enabled=true \
                  --set ingress.host=${gateway_domain_name} \
                   \${common_args}
                elif [ \${service_name} == "portal-service" ]; then
                  helm \${action} \${helm_args} \
                  --set ingress.enabled=true \
                  --set ingress.host=${portal_domain_name} \
                   \${common_args}
                else
                  helm \${action} \${helm_args} \${common_args}
                fi
              done
              # 查看Pod状态
              sleep 10
              kubectl get pods \${common_args}
              """
          }
        }
    }
}
```



![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611185036202-1098491929.png) 

Pipeline 集成Helm 发布微服务项目

![img](https://img2020.cnblogs.com/blog/1156961/202006/1156961-20200611160116832-369627885.png)

小结：

❖使用Jenkins的插件
 •Git & gitParameter
 •Kubernetes
 •Pipeline
 •Kubernetes Continuous Deploy
 •Config File Provider
 •Extended Choice Parameter
❖CI/CD环境特点
 •Slave弹性伸缩
 •基于镜像隔离构建环境
 •流水线发布，易维护
❖Jenkins参数化构建可帮助你完成更复杂环境CI/CD