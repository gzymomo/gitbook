- [高可用集群篇（一）-- k8s集群部署](https://juejin.cn/post/6940887645551591455)
- [高可用集群篇（二）-- KubeSphere安装与使用](https://juejin.cn/post/6942282048107347976)
- [高可用集群篇（三）-- MySQL主从复制&ShardingSphere读写分离分库分表](https://juejin.cn/post/6944142563532079134)
- [高可用集群篇（四）-- Redis、ElasticSearch、RabbitMQ集群](https://juejin.cn/post/6945360597668069384)
- [高可用集群篇（五）-- k8s部署微服务](https://juejin.cn/post/6946396542097948702)



# 一、K8S有状态服务

## 1.1 什么是有状态服务

- 无状态服务

  - 1、是指该服务运行的实例不会在本地存储需要持久化的数据，并且多个实例对于同一个请求响应的结果是完全一致的
  - 2、多个实例可以共享相同的持久化数据。例如：nginx实例，tomcat实例等
  - 3、相关的k8s资源有：ReplicaSet、ReplicationController、Deployment等，由于是无状态服务，所以这些控制器创建的pod序号都是随机值。并且在缩容的时候并不会明确缩容某一个pod，而是随机的，因为所有实例得到的返回值都是一样，所以缩容任何一个pod都可以

- 有状态服务

  - 1、宠物和牛的类比，农场主的牛如果病了可以丢掉再重新买一头，如果宠物主的宠物病死了是没法找到一头一模一样的宠物的；有状态服务 可以说是 **需要数据存储功能的服务、或者指多线程类型的服务，队列等**（mysql数据库、kafka、zookeeper等）

  - 2、每个实例都需要有自己独立的持久化存储，并且在k8s中是通过申明模板来进行定义；持久卷申明模板在创建pod之前创建，绑定到pod中，模板可以定义多个

    说明： 有状态的 pod是用来运行有状态应用的，所以其在数据卷上存储的数据非常重要，在 Statefulset缩容时删除这个声明将是灾难性的，特别是对于 Statefulset来说，缩容就像减少其 replicas 数值一样简单。基于这个原因，当你需要释放特定的持久卷时，需要手动删除对应的持久卷声明

  - 3、相关的k8s资源为：statefulSet，由于是有状态的服务，所以每个pod都有特定的名称和网络标识。比如pod名是由statefulSet名+有序的数字组成（0、1、2..）

  - 4、在进行缩容操作的时候，可以明确知道会缩容哪一个pod，从数字最大的开始。并且Stat巳fulset 在有实例不健康的情况下是不允许做缩容操作的

## 1.2 k8s部署MySQL

- 可以使用kubesphere，快速搭建MySQL环境

  - 有状态服务抽取配置为ConfigMap

    ![image-20210323102239013](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/847ff3c793e04b8286d61803a5e3f6a5~tplv-k3u1fbpfcp-zoom-1.image)

  - 有状态服务必须使用pvc持久化数据

    ![image-20210323102326626](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/670c565f97c7489685a548ce43b7c1e2~tplv-k3u1fbpfcp-zoom-1.image)

  - 服务集群内访问使用DNS提供的稳定域名

  ![image-20210323100739205](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81b7ecc4ea81424dbd5d051a0880d3e6~tplv-k3u1fbpfcp-zoom-1.image)

### 1.2.1 创建MySQL主从服务

- 第一步：创建存储卷pvc和配置configMap

  configMap的内容就是之前docker方式启动的 my.cnf

  ```bash
  #mysql 主节点
  [client]
  default-character-set=utf8
  [mysql]
  default-character-set=utf8
  
  [mysqld]
  init_connect='SET collation_connection=utf8_unicode_ci'
  init_connect='SET NAMES utf8'
  character-set-server=utf8
  collation-server=utf8_unicode_ci
  skip-character-set-client-handshake
  skip-name-resolve
  
  server-id=1
  log-bin=mysql-bin
  read-only=0
  binlog-do-db=touch-air-mall-ums
  binlog-do-db=touch-air-mall-pms
  binlog-do-db=touch-air-mall-oms
  binlog-do-db=touch-air-mall-sms
  binlog-do-db=touch-air-mall-wms
  binlog-do-db=demo_ds_0
  binlog-do-db=demo_ds_1
  
  replicate-ignore-db=mysql
  replicate-ignore-db=sys
  replicate-ignore-db=information_schema
  replicate-ignore-db=performance_schema
  ```
  
- 第二步：创建有状态服务

  - 设置副本
  - 拉取mysql镜像，高级设置修改内存，使用默认端口
  - 设置环境变量
  - 挂载数据、挂载配置

  ![k8s部署mysql.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6014ea1b6ce94a17a3adddb2fd897b99~tplv-k3u1fbpfcp-watermark.image)

- 第三步：配置主从

  - 进入master容器，连接mysql，授权账户

    ```sql
    mysql -uroot -p
    GRANT REPLICATION SLAVE ON *.* TO 'backup'@'%' IDENTIFIED BY '123456';
    ```
    
    ![image-20210323144851610](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70700fd296324d32bb2907822201f8c9~tplv-k3u1fbpfcp-zoom-1.image)
    
  - 进入slaver容器，配置同步数据
  
    ```shell
    #host使用master的域名（DNS）
    CHANGE MASTER TO MASTER_HOST='mysql-master.touch-air-mall', 
    MASTER_USER='backup',
    MASTER_PASSWORD='123456',
    MASTER_LOG_FILE='mysql-bin.000005',MASTER_LOG_POS=439,master_port=3306;
    #开启从节点
    start slave;
    #查看slave状态
    show slave status\G;
    ```
    
  - - ![image-20210323145541403](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cb9e0a233254c7280a2a662d50c165e~tplv-k3u1fbpfcp-zoom-1.image)
    
  

### 1.2.2 测试主从配置

- 在主库master中创建数据库

  ```sql
  CREATE DATABASE demo_ds_0;
  ```

- 创建表

  ```mysql
  SET NAMES utf8mb4;
  SET FOREIGN_KEY_CHECKS = 0;
  
  -- ----------------------------
  -- Table structure for sys_user
  -- ----------------------------
  DROP TABLE IF EXISTS `user`;
  CREATE TABLE `user`  (
    `id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '主键',
    `been_deleted` int(11) NULL DEFAULT 0 COMMENT '逻辑删除 0表示逻辑未删除 1表示逻辑删除',
    `create_by` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '创建者',
    `created` datetime(0) NULL DEFAULT NULL COMMENT '创建时间',
    `remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '备注',
    `update_by` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '更新者',
    `updated` datetime(0) NULL DEFAULT NULL COMMENT '修改时间',
    `version` int(11) NULL DEFAULT NULL COMMENT '版本号',
    `last_logged_in` datetime(0) NULL DEFAULT NULL COMMENT '上次登录时间',
    `nick_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '用户昵称',
    `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '密码',
    `phone` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '手机号',
    `first_sign_in` int(11) NULL DEFAULT NULL COMMENT '首次登录：0表示未登录，1表示首次登录，2表示非首次登录',
    `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '账号',
    `email` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '邮箱',
    `sys_dept_id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '部门id',
    `sys_dept_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '部门名称',
    PRIMARY KEY (`id`) USING BTREE,
    INDEX `idx_username_password`(`username`, `password`) USING BTREE
  ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = '系统用户表' ROW_FORMAT = Dynamic;
  
  SET FOREIGN_KEY_CHECKS = 1;
  ```
  
- 插入数据

  ```mysql
  INSERT INTO `demo_ds_0`.`user`(`id`, `been_deleted`, `create_by`, `created`, `remark`, `update_by`, `updated`, `version`, `last_logged_in`, `nick_name`, `password`, `phone`, `first_sign_in`, `username`, `email`, `sys_dept_id`, `sys_dept_name`) VALUES ('402881f773b383ae0173b383d019001b', 0, NULL, '2020-08-03 16:50:27', NULL, NULL, '2021-03-23 15:01:08', 11007, '2021-03-23 15:01:08', '管理员', '$2a$10$H57iIdDN5QigR7QOxjFtceRA1l4MSjPSJSm3t3AtyW9RaoIuc4m5y', NULL, 2, 'admin', NULL, NULL, NULL);
  ```
  
  ![image-20210323151234589](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/119e662a97774fc98973a84f0876da8a~tplv-k3u1fbpfcp-zoom-1.image)
  
- 确认从库是否同步

  - ![image-20210323151726965](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf37d3a9f38044ef9db8e9f6c747f934~tplv-k3u1fbpfcp-zoom-1.image)

  #### 

### 1.2.3 k8s部署总结（*）

- 1、每一个MySQL、Redis必须是有状态服务
- 2、每一个MySQL、Redis都必须挂载自己配置文件（ConfigMap）和存储卷（PVC）
- 3、docker方式启动时，配置里的master的IP地址，都改为DNS域名即可

> 由于测试机内存有限，接下来的服务就都不以集群的形式启动了，但大同小异，步骤都和MySQL主从一样

## 1.3 k8s部署Redis

- 第一步：创建redis的配置文件configMap

  ```shell
   redis-conf: appendonly yes
  ```
  
- 第二步：创建存储卷 pvc

  ![image-20210324084736652](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2be7ec93da7c48e29d81f5494cec2a99~tplv-k3u1fbpfcp-zoom-1.image)

- 第三步：创建有状态服务，拉取redis镜像、设置内存、挂载数据、挂载配置文件

  ![image-20210324085856705](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6906b0a1029c440cb9d2edb8bbd658ec~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324090005106](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6639eaead5494cbca6797e2927fb9e55~tplv-k3u1fbpfcp-zoom-1.image)

  - ![image-20210324090005106](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6639eaead5494cbca6797e2927fb9e55~tplv-k3u1fbpfcp-zoom-1.image)

  

## 1.4 k8s部署ElasticSearch&Kibana

### 1.4.1 部署ElasticSearch

- 第一步：创建es的配置文件 configMap

  ![image-20210324091903787](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d00e420d1783441cbd6e8818278ca3bd~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324091926499](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8d71ec53ebc4501a37f5a722f7c67e2~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324091926499](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8d71ec53ebc4501a37f5a722f7c67e2~tplv-k3u1fbpfcp-zoom-1.image)

- 第二步：创建es的存储卷pvc

- 第三步：创建有状态服务，拉取es镜像...

  ![image-20210324092443507](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80b9376c6844ff4a524e69ac7c3a8ad~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324093153529](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/355a355f884347b7970dffd15e012fc9~tplv-k3u1fbpfcp-zoom-1.image)

  

  - 测试ES是否创建成功

    - 1、使用admin账户登录，console控制台测试
    - 2、使用之前安装好的 **wordpress**测试

    ```shell
    #curl 访问es的9200端口，k8s中使用域名进行访问
    curl mall-es.touch-air-mall:9200
    ```
    
- - ![image-20210324093042544](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e45d0cde04541389a12d594c77a0f51~tplv-k3u1fbpfcp-zoom-1.image)
    


## 1.5 部署kibana

- kibana是ElasticSearch的可视化服务，无需挂载数据与配置，因此只需要启动无状态服务

  - 添加环境变量

    ```shell
    SERVER_HOST  0.0.0.0
    ELASTICSEARCH_HOSTS  http://mall-es.touch-air-mall:9200
    ```
    
  - ![image-20210324105432923](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55ac20ca8eac4dfdae76e40f0f3ad53e~tplv-k3u1fbpfcp-zoom-1.image)
    
  ![image-20210324095346260](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df1bc54f4f0c4017905ca75da649ed7b~tplv-k3u1fbpfcp-zoom-1.image)
    
  ![image-20210324105342179](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0af82a67c78426395c527b3bba540de~tplv-k3u1fbpfcp-zoom-1.image)
  
- 浏览器访问暴露端口

  ```
  http://192.168.83.133:31061/
  ```
  
- ![image-20210324111015986](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f4932de10644df19c8da919ecfd0c9f~tplv-k3u1fbpfcp-zoom-1.image)
  


## 1.6 k8s部署RabbitMQ

- 同上

- 第一步：创建mq的存储卷pvc

- 创建有状态服务，拉取镜像，挂载存储卷

  - ![image-20210324143322444](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5d6b85350004cdb86cc6613090b7ae9~tplv-k3u1fbpfcp-zoom-1.image)

  

## 1.7 k8s部署Nacos

### 1.7.1 无状态服务的两种部署方式

- 1、直接创建无状态服务

- 2、有状态服务，删除服务不删除工作副本，然后创建指定工作负载

  ![image-20210324151808372](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/faeefc702b7748aa90e7e64821fd6f69~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324151831839](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6bff0cd76d548eb95a26be61810a355~tplv-k3u1fbpfcp-zoom-1.image)

- 浏览器访问测试

  ```
  http://192.168.83.134:31384/nacos
  ```
  
- ![image-20210324151920642](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e94ea47c1de47acac8943c5fa909996~tplv-k3u1fbpfcp-zoom-1.image)
  


## 1.8 k8s部署Zipkin

- 创建无状态服务

  ![image-20210324152358995](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39b139406aa44e43a68534fcaba27671~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324152439266](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b42dd25cf7a421c955e1a81fbe54192~tplv-k3u1fbpfcp-zoom-1.image)

- 访问测试

  ```
  http://192.168.83.133:31768/zipkin/
  ```
  
- ![image-20210324152525690](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42a1db772b6f4b0fbbb7ce033f9c11df~tplv-k3u1fbpfcp-zoom-1.image)
  


## 1.9 k8s部署Sentinel

- 使用开源镜像

  ```
  bladex/sentinel-dashboard:1.6.3
  ```
  
- 创建一个无状态服务

  ![image-20210324153155984](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ba33af04fff4a9484741da67169b1f7~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210324153228972](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8943444c41524fc2a2b4ec69ad2c41c7~tplv-k3u1fbpfcp-zoom-1.image)

- 访问测试

  ```
  http://192.168.83.133:30823/
  ```
  
- ![image-20210324153336088](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad9dbbf425d44ff3930558297256b723~tplv-k3u1fbpfcp-zoom-1.image)
  


## 1.10 创建指定工作负载的服务

- 场景：已经无状态的sentinel服务，想要再创建一个有状态的sentinel服务

  - ![image-20210330135055783](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e27e22dca614dd39d88c86d3cb19ba6~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210330135152608](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b1f68f2e2f3484090f5be7e359616f7~tplv-k3u1fbpfcp-zoom-1.image)

  

# 二、K8S部署微服务

## 2.1 部署流程

- 部署流程图

  ![image-20210324153917383](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bbe2b70fdb74f5f8575d05f39707bad~tplv-k3u1fbpfcp-zoom-1.image)

- 操作步骤

  - 第一步：为每一个项目准备一个`Dockerfile`；`Docker`按照这个`Dockerfile`将项目制作成镜像
  - 第二步：为每一个项目生成k8s部署描述文件
  - 将上述操作串联起来：编写好`jenkinsfile`

### 2.1.1 生产环境配置抽取

- 基于NodePort的工作负载，创建供集群内访问的服务

  ```
  mall-nacos-service
  mall-sentinel-service
  mall-zipkin-service
  ```
  
  ![image-20210325155813974](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/806297686c604871bbe70a92e5070d28~tplv-k3u1fbpfcp-zoom-1.image)
  
  ![image-20210325161605402](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0442e4e864948858d6a0f9d8e6ddbbd~tplv-k3u1fbpfcp-zoom-1.image)
  
- 添加生产环境配置文件

  将之前本地调试的所有服务host与端口号信息，切换成集群中的自定义的域名

  ```
  spring.rabbitmq.host=mall-rabbitmq.touch-air-mall
  spring.datasource.url=jdbc:mysql://mysql-master.touch-air-mall:3306/touch_air_mall_sms?serverTimezone=Asia/Shanghai
  spring.cloud.nacos.discovery.server-addr=mall-nacos-service.touch-air-mall:8848
  spring.cloud.sentinel.transport.dashboard=mall-sentinel-service:8858
  spring.zipkin.base-url=http://mall-zipkin.touch-air-mall:9411/
  spring.redis.host=mall-redis.touch-air-mall
  ```

#### 2.1.2 制作项目镜像

- `renren-fast`微服务

  - 1、生成`jar`包

    ```
    mvn clean package -Dmaven.test.skip=true
    ```
    
    ![image-20210325150740489](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/996afe194d73457199c9a3409cbf0b1f~tplv-k3u1fbpfcp-zoom-1.image)
    
  - 2、将`jar`包和`dockerfile`拷贝进docker服务所在的服务器中
  
    ```shell
    docker build -f Dockerfile -t docker.io/touch/admin:v1.0 .
    ```
    
  - - ![image-20210325154348150](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/977f793bfb2b483d8416a575fc1922d9~tplv-k3u1fbpfcp-zoom-1.image)

#### 2.1.3 创建Dockerfile

- 每个微服务的根目录下新建`Dockerfile`

  ```shell
  FROM java:8
  #容器内暴露8080，所有微服务都一样，互不影响
  EXPOSE 8080
  
  VOLUME /tmp
  ADD target/*.jar  /app.jar
  RUN bash -c 'touch /app.jar'
  ENTRYPOINT ["java","-jar","/app.jar","--spring.profiles.active=prod"]
  ```

#### 2.1.4 创建微服务k8s部署描述文件

- [参考文件](https://gitee.com/OK12138/devops-java-sample/blob/master/deploy/prod-ol/devops-sample.yaml)

- 部署`Deployment`描述文件

  ```yaml
  kind: Deployment
  apiVersion: apps/v1
  metadata:
    name: mall-auth-server
    namespace: touch-air-mall
    labels:
      app: mall-auth-server
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: mall-auth-server
    template:
      metadata:
        labels:
          app: mall-auth-server
      spec:
        containers:
          - name: mall-auth-server
            image: $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest
            ports:
              - name: tcp-mall-auth-server
                containerPort: 8080
                protocol: TCP
            resources:
              limits:
                cpu: 500m
                memory: 512Mi
              requests:
                cpu: 10m
                memory: 10Mi
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            imagePullPolicy: IfNotPresent
        restartPolicy: Always
        terminationGracePeriodSeconds: 30
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxUnavailable: 25%
        maxSurge: 25%
    revisionHistoryLimit: 10
    progressDeadlineSeconds: 600
  
  ---
  kind: Service
  apiVersion: v1
  metadata:
    name: mall-auth-server
    namespace: touch-air-mall
    labels:
      app: mall-auth-server
    annotations:
      kubesphere.io/alias-name: 认证服务
      kubesphere.io/serviceType: statelessservice
  spec:
    ports:
      - name: http
        protocol: TCP
        port: 8080
        targetPort: 8080
        nodePort: 20001
    selector:
      app: mall-auth-server
    type: NodePort
    sessionAffinity: None
  
  ```

#### 2.1.5  理解`TargetPort`、`Port`、`NodePort`

![image-20210326084219748](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f0803217a1e4adba214fdd22fb3bae9~tplv-k3u1fbpfcp-zoom-1.image)

- `port`：【clusterIP:port】，service暴露在clusterIP

- `targerPort`：【容器映射端口】在pod上相当于（Dockerfile中Expose）

- `nodePort`：【nodeIP:nodePort】提供给外部流量访问k8s集群中service的入口

  ![image-20210326084851753](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd24ca85cde74dbabbfd066a011353f7~tplv-k3u1fbpfcp-zoom-1.image)

> 除了`NodePort`不可用重复，`Port`与`TargertPort`都是可以重复的

## 2.2 流水线

- 编写Jenkinsfile

  - 第一步：拉取代码

  - 第二步：参数化构建&环境变量

    ![image-20210327102739577](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6df3985219b244f3889e660bb5b81077~tplv-k3u1fbpfcp-zoom-1.image)

  - 第三步：sonar代码质量分析

  - 第四步：build & push（快照、最新版）

  - 第五步：部署到dev环境

  - 第六步：打上TAG，发布版本

  ```shell
  pipeline {
    agent {
      node {
        label 'maven'
      }
    }
    environment {
          DOCKER_CREDENTIAL_ID = 'dockerhub-id'
          GITHUB_CREDENTIAL_ID = 'gitee-id'
          KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-id'
          REGISTRY = 'docker.io'
          DOCKERHUB_NAMESPACE = 'wlfctothemoon'
          GITHUB_ACCOUNT = 'OK12138'
          SONAR_CREDENTIAL_ID='sonar-qube'
          BRANCH_NAME='main'
      }
    stages {
      stage('拉取代码') {
        steps {
          git(url: 'https://gitee.com/OK12138/touch-air-mall.git', credentialsId: 'gitee-id', branch: 'main', changelog: true, poll: false)
          sh 'echo 正在构建 $PROJECT_NAME 版本号：$PROJECT_VERSION 将会提交给 $REGISTRY 镜像仓库'
          container ('maven') {
            sh "mvn clean install -Dmaven.test.skip=true -gs `pwd`/mvn-settings.xml"
            }
        }
      }
      stage('sonar代码质量分析') {
            steps {
              container ('maven') {
                withCredentials([string(credentialsId: "$SONAR_CREDENTIAL_ID", variable: 'SONAR_TOKEN')]) {
                  withSonarQubeEnv('sonar') {
                   sh "echo 当前目录 `pwd` "
                   sh "mvn sonar:sonar -gs `pwd`/mvn-settings.xml -Dsonar.login=$SONAR_TOKEN"
                  }
                }
              }
            }
          }
      stage ('build & push 构建镜像并推送') {
              steps {
                  container ('maven') {
                      sh 'mvn -Dmaven.test.skip=true -gs `pwd`/mvn-settings.xml clean package'
                      sh 'cd $PROJECT_NAME && docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
                      withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                          sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:latest '
                          sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:latest '
                      }
                  }
              }
          }
     stage('部署到k8s') {
            steps {
              input(id: "deploy-to-dev-$PROJECT_NAME", message: "是否将$PROJECT_NAME部署到集群中?")
              kubernetesDeploy(configs: "$PROJECT_NAME/deploy/**", enableConfigSubstitution: true, kubeconfigId: "$KUBECONFIG_CREDENTIAL_ID")
            }
          }
     stage('push with tag 打上TAG标签，发布版本'){
            when{
              expression{
                return params.PROJECT_VERSION =~ /v.*/
              }
            }
            steps {
                container ('maven') {
                  input(id: 'release-image-with-tag', message: '发布当前版本镜像吗?')
                    withCredentials([usernamePassword(credentialsId: "$GITHUB_CREDENTIAL_ID", passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                      sh 'git config --global user.email "kubesphere@yunify.com" '
                      sh 'git config --global user.name "kubesphere" '
                      sh 'git tag -a $PROJECT_VERSION -m "$PROJECT_VERSION" '
                      sh 'git push http://$GIT_USERNAME:$GIT_PASSWORD@gitee.com/$GITHUB_ACCOUNT/touch-air-mall.git --tags --ipv4'
                    }
                  sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:$PROJECT_VERSION '
                  sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$PROJECT_NAME:$PROJECT_VERSION '
              }
            }
          }
    }
  }
  
  ```

  - ![image-20210327150055019](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b64b621729e844e8813789420c01dec5~tplv-k3u1fbpfcp-zoom-1.image)

  

#### 2.2.1 移植数据库

- 之前启动的MySQL 主从节点，都是有状态服务，供集群内访问，未暴露端口

  可以使用指定工作负载创建一个无状态MySQL服务，NodePort暴露访问

  ![image-20210327152012465](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5144712bb438424c99695e05a7126d80~tplv-k3u1fbpfcp-zoom-1.image)

  拷贝完成后，删除暴露，保证集群内部数据安全

#### 2.2.2 部署效果

- 流程图

  - ![image-20210327152147807](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cf29f4bbb714e369f9ad6e9448107c9~tplv-k3u1fbpfcp-zoom-1.image)

  

#### 2.2.3 移植Nginx（上线静态资源无法访问）

- 使用Dockerfile生成自定义的nginx镜像

  - 准备自定义Dockerfile，以及文件

    ![image-20210331090441728](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16ee28ade8f44ca08928dd2ee6903028~tplv-k3u1fbpfcp-zoom-1.image)

  - 使用Dockerfile构建镜像

    ```shell
    docker build -t touch-air/mall-nginx:v1.2 .
    ```
    
  ![image-20210331090859126](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/021395c07e1b4dbeb70dd71515d673db~tplv-k3u1fbpfcp-zoom-1.image)
  
- 可以结合阿里云镜像仓库，将本地（虚拟机的docker镜像），部署至k8s集群中

  推送镜像到dockerhub（阿里云镜像仓库）

  ```shell
  #标记镜像
  docker tag local-image:tag username/new-repo:tag
  #上传镜像
  docker push username/new-repo:tag
  ```
  
- 修改nginx的上游服务器，保证在k8s集群中，路由到网关

  - 先备份原先的`nginx.conf`，再进行修改

    ![image-20210328115344457](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a883468e2294e7babaa743fa4fe59c7~tplv-k3u1fbpfcp-zoom-1.image)

#### 2.2.4 整合阿里云镜像仓库

- 登录阿里云镜像容器服务，开通、创建个人镜像仓库

  ![image-20210328121239271](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc2aa966879e4ae291683cb83ee678c5~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20210328121411248](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c997cd08a7fb4be8adaed278eb30f998~tplv-k3u1fbpfcp-zoom-1.image)

- 推送镜像至阿里云

  - 1、登录阿里云，密码就是开通镜像仓库时的密码

    ```shell
    docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
    ```
    
  - 2、推送镜像
  
    ```shell
    #标记镜像
    docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/tothemoon/mall-nginx:[镜像版本号]
    #推送
    docker push registry.cn-hangzhou.aliyuncs.com/tothemoon/mall-nginx:[镜像版本号]
    ```
    
  - - ![image-20210331091137281](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fef82202357497d8664fd7cc2fe6062~tplv-k3u1fbpfcp-zoom-1.image)
    
      ![image-20210328122120131](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dc4368374da42529460353a3ce47809~tplv-k3u1fbpfcp-zoom-1.image)
    
  

#### 2.2.5 Jenkins修改阿里云镜像仓库

- 修改仓库凭证以及仓库地址

#### 2.2.6 流水线部署所有微服务

- 以`mall-gateway`网关服务为例

- 上面的参数化构建很重要，这样我们部署服务，可以选择我们想要部署的一个或多个服务

  部署不同服务时，只需要修改`PROJECT_NAME`变为想要部署的项目名称（`mall-gateway、mall-cart、mall-seckill`）

  ![流水线构建微服务.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e27a61c07094d128930280eea35fd1b~tplv-k3u1fbpfcp-watermark.image)

- 重复运行流水线构建所有商城微服务

  ![image-20210330142331966](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fdda0cdd3a34034a74822b4db767151~tplv-k3u1fbpfcp-zoom-1.image)

  

  - 最终效果，浏览器访问暴露出来的端口，出现标准`springboot`工程的404界面即可；待后续nginx服务部署之后，就可以查看整体商城服务

  - k8s集群中的nacos服务注册中心

    ![image-20210330162325517](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdf68173a02e4cb3a1f6d8b7f20f79b6~tplv-k3u1fbpfcp-zoom-1.image)

  - k8s集群中sentinel流量监控

    - ![image-20210330162357903](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddae174c564b48c9a1b2cd6b1794b430~tplv-k3u1fbpfcp-zoom-1.image)

- 将阿里云仓库的微服务权限修改为公开，方便k8s集群访问

  ![image-20210330142617240](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7573dd87ff4447af887df83b92783d93~tplv-k3u1fbpfcp-zoom-1.image)

- 接口测试

  ```
  #商品三级分类数据
  http://192.168.83.133:30007/index/catalog.json
  ```
  
- ![image-20210401161047977](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99f3d95ba04341598efb41cf006abff9~tplv-k3u1fbpfcp-zoom-1.image)



## 2.3 最终部署

#### 2.3.1 部署前置nginx

- 利用我们之前打包推送到阿里云上的nginx镜像，在k8s集群中部署nginx服务

  - 创建个人仓库地址

    ![image-20210330163540204](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81b26b1fb0a84cab87c11766b82d2984~tplv-k3u1fbpfcp-zoom-1.image)

  - 选择个人阿里云上的nginx镜像

    ![image-20210330163938349](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a37d7c3d2fad41afb114519be5e6fbc2~tplv-k3u1fbpfcp-zoom-1.image)

  - 测试访问

    ![image-20210330164027831](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1c51b2ecfe34df18d61533958a8af11~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210330164045797](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdb9dee93c734121806a2e5aefdfa645~tplv-k3u1fbpfcp-zoom-1.image)

  - 当前使用ip地址＋暴露端口号的访问访问方式，显然不能满足我们的需求

#### 2.3.2 创建网关与应用路由

##### 2.3.2.1 外网访问网关

- 在创建应用路由之前，需要先启用外网访问入口，即网关。这一步是创建对应的应用路由控制器，用来负责将请求转发到对应的后端服务

  **注意：由于使用 Load Balancer 需要在安装前配置与安装与云服务商对接的 cloud-controller-manage 插件，参考** [安装负载均衡器插件](https://v2-1.docs.kubesphere.io/docs/zh-CN/installation/qingcloud-lb) **来安装和使用负载均衡器插件**（暂不演示）

- 以项目管理员的身份，开启**应用路由**

  ![image-20210330164823778](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/838cab5b393849f0a27b0e8d86a5f549~tplv-k3u1fbpfcp-zoom-1.image)

  

  - 创建`NodePort`网关

    - - ![image-20210330173404832](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f20f6bb2b834ab4a3f36cc5e575b0dc~tplv-k3u1fbpfcp-zoom-1.image)

    

##### 2.3.2.2 应用路由

- 创建应用路由

  - ![image-20210330165314905](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bacd4ae2dcb4ee489f3a6a4cbedbb0d~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210330165508984](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc1f0b9820584057a5951347a5924c1b~tplv-k3u1fbpfcp-zoom-1.image)

  

#### 2.3.3 部署Vue项目

- 第一步：修改线上网关环境

  ![image-20210401153310874](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d851c38aa67b4f40959345f6062e63ad~tplv-k3u1fbpfcp-zoom-1.image)

  修改 `index-prod.js`中的api请求地址

  ![image-20210401153628617](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75567a9770b447739d5f0664fefaf920~tplv-k3u1fbpfcp-zoom-1.image)

- 第二步：编译打包，生成dist目录

  ```shell
  npm run build
  ```

  ![image-20210401152914142](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8388d6a2e404f8f9d7ae206327f1264~tplv-k3u1fbpfcp-zoom-1.image)

- 第三步：编写Dockerfile

  ```shell
  FROM nginx
  MAINTAINER leifengyang
  ADD dist.tar.gz /usr/share/nginx/html
  EXPOSE 80
  ENTRYPOINT nginx -g "daemon off;"
  ```

- 第四步：打包成镜像

  ```shell
  docker build -t touch-air-mall-vue-app:v1.0 -f Dockerfile .
  ```

  ![image-20210401155813661](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c82c82e4a1ae49afa79f04e754519186~tplv-k3u1fbpfcp-zoom-1.image)

- 上传阿里云镜像仓库，kubesphere创建无状态服务

  - 上传

    ```shell
    #登录
    docker login --username=xxx registry.cn-hangzhou.aliyuncs.com
    #标记镜像
    docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/tothemoon/mall-nginx:[镜像版本号]
    #推送
    docker push registry.cn-hangzhou.aliyuncs.com/tothemoon/mall-nginx:[镜像版本号]
    ```

    ![image-20210401160259625](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59660b36cbfb46389c3e077804634874~tplv-k3u1fbpfcp-zoom-1.image)

  - 创建无状态服务

    ![image-20210401160612785](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/336fd8a355884a03b53200376e6c861a~tplv-k3u1fbpfcp-zoom-1.image)

    ![image-20210401160657528](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3a08fe2ce084b38a12c5b08ab395c7a~tplv-k3u1fbpfcp-zoom-1.image)



