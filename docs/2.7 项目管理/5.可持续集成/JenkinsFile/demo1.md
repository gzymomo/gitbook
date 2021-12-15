# 一、后端SpringBoot

## 1.1 jenkinsfile

```json
pipeline {
  agent {
    node {
      label 'maven'
    }
  }
  stages {
    stage('拉取代码') {
      steps {
        git(url: 'http://xxx', credentialsId: 'git-glab', branch: 'dev', changelog: true, poll: false)
      }
    }
    stage('构建镜像') {
      steps {
        container('maven') {
          sh '''docker login -u admin -p 123456 ip:port
                cd project
                mvn package
                cd project-system
                docker build -t project:1.0 .
                docker push ip:port/path'''
        }

      }
    }
    stage('部署') {
      steps {
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: 'admin-config', configs: 'project/deploy/project-dev.yaml', dockerCredentials: [[url: 'ip:port', credentialsId: 'docker-admin']])
      }
    }
  }
}

```



## 2.2 project-dev.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  namespace: ns-dev
  name: projectA
  labels:
    k8s-app: projectA
  annotations:
    deployment.kubernetes.io/revision: '5'
spec:
  replicas: 1  # 副本数量
  selector:
    matchLabels:
      k8s-app: projectA
  template:
    metadata:
      name: projectA
      labels:
        k8s-app: projectA
    spec:
      imagePullSecrets:
        - name: docker-admin
      containers:
        - name: projectA
          image: 'ip:port/path/projectA:1.0.0'
          env:
            - name: JAVA_OPTS
              valueFrom:
                configMapKeyRef:
                  key: jvm.options
                  name: projectA-config
          ports:
            - name: web
              containerPort: 9000
              protocol: TCP
          resources:
            requests:
              cpu: '100m'
              memory: 256Mi
          imagePullPolicy: Always
      restartPolicy: Always

---
kind: Service
apiVersion: v1
metadata:
  namespace: ns-dev
  name: projectA
spec:
  ports:
    - name: web
      protocol: TCP
      port: 80
      targetPort: 9000
  selector:
    k8s-app: projectA
  type: NodePort
  sessionAffinity: None
  publishNotReadyAddresses: true


---
apiVersion: v1
data:
  jvm.options: |-
    -Xmx4G
    -Xms4G
    -server
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=50
    -XX:InitiatingHeapOccupancyPercent=35
    -XX:+ExplicitGCInvokesConcurrent
    -Djava.awt.headless=true
    -Xloggc:/gc.log
    -verbose:gc
    -XX:+PrintGCDetails
    -XX:+PrintGCDateStamps
    -XX:+PrintGCTimeStamps
    -XX:+UseGCLogFileRotation
    -XX:NumberOfGCLogFiles=20
    -XX:GCLogFileSize=10M
    -XX:+UnlockExperimentalVMOptions
    -XX:+UseCGroupMemoryLimitForHeap
    -Dspring.profiles.active=dev
kind: ConfigMap
metadata:
  namespace: ns-dev
  name: projectA-config
```





# 二、前端

## 2.1 jenkinsfile

```json
pipeline {
  agent {
    node {
      label 'nodejs'
    }

  }
  stages {
    stage('stage-ogcq2') {
      steps {
        git(url: 'xxx', credentialsId: 'git-glab', branch: 'dev', changelog: true, poll: false)
      }
    }
    stage('stage-uc94l') {
      steps {
        container('nodejs') {
          sh '''cd project
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install
npm run build
docker login -u admin -p 123456 ip:port
docker build -t project:1.0 .
docker push ip:port/path:1.0'''
        }

      }
    }
    stage('stage-43lrd') {
      steps {
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: 'admin-config', configs: 'project/deploy/assembly.yaml')
      }
    }
  }
}
```

## 2.2 assembly.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: manage
  namespace: ns-dev
  labels:
    k8s-app: manage
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: manage
  template:
    metadata:
      name: manage
      labels:
        k8s-app: manage
      annotations:
        kubesphere.io/containerSecrets: '{"manage":"nexus-admin"}'
    spec:
      imagePullSecrets:
        - name: docker-admin
      containers:
        - name: manage
          image: 'ip:port/path/manage:1.0.0'
          ports:
            - name: web
              containerPort: 80
              protocol: TCP
          imagePullPolicy: Always
      restartPolicy: Always

---
kind: Service
apiVersion: v1
metadata:
  name: manage
  namespace: ns-dev
spec:
  ports:
    - name: web
      protocol: TCP
      port: 80
      targetPort: 80
  selector:
    k8s-app: manage
  type: NodePort
  sessionAffinity: None
  publishNotReadyAddresses: true
```

## 2.3 dockerfile

```bash
FROM nginx
VOLUME /tmp
ENV LANG en_US.UTF-8
RUN echo "server {  \
                      listen       80; \
                      location ^~ /jeecg-boot { \
                      proxy_pass              http://ip:/port/jeecg-boot/; \
                      proxy_set_header        Host ip; \
                      proxy_set_header        X-Real-IP \$remote_addr; \
                      proxy_set_header        X-Forwarded-For \$proxy_add_x_forwarded_for; \
                  } \
                  #解决Router(mode: 'history')模式下，刷新路由地址不能找到页面的问题 \
                  location / { \
                     root   /var/www/html/; \
                      index  index.html index.htm; \
                      if (!-e \$request_filename) { \
                          rewrite ^(.*)\$ /index.html?s=\$1 last; \
                          break; \
                      } \
                  } \
                  access_log  /var/log/nginx/access.log ; \
              } " > /etc/nginx/conf.d/default.conf \
    &&  mkdir  -p  /var/www \
    &&  mkdir -p /var/www/html

ADD dist/ /var/www/html/
EXPOSE 80
EXPOSE 443
```

