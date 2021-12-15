[TOC]

# 1、Zookeeper基础简介
Zookeeper是一个Apache开源的分布式的应用，为系统架构提供协调服务。从设计模式角度来审视：该组件是一个基于观察者模式设计的框架，负责存储和管理数据，接受观察者的注册，一旦数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

## 1.1 基本理论
- 数据结构
ZooKeeper记录数据的结构与Linux文件系统相似，整体可以看作一棵树，每个节点称ZNode。每个Znode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvBu010xkia3PRhYRrQj5bQWghhDoWMQANjIoON84HS1KIXPfKELKJANHJ8IG36L0icicIDTlJetiboImg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 节点类型
短暂（ephemeral）：客户端和服务器端断开连接后，创建的节点自动删除。
持久（persistent）：客户端和服务器端断开连接后，创建的节点持久化保存。

- 集群服务
在Zookeeper集群服务是由一个领导者（leader），多个跟随者（follower）组成的集群。领导者负责进行投票的发起和决议，更新集群服务状态。跟随者用于接收客户请求并向客户端返回结果，在选举Leader过程中参与投票。集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。

- 数据一致性
每个server保存一份相同的数据拷贝，客户端无论请求到被集群中哪个server处理，得到的数据都是一致的。

## 1.2 应用场景
- 经典应用：Dubbo框架的服务注册和发现；
- 分布式消息同步和协调机制；
- 服务器节点动态上下线；
- 统一配置管理、负载均衡、集群管理；

# 2、安全管理操作
## 2.1 操作权限
ZooKeeper的节点有5种操作权限：CREATE(增)、READ(查)、WRITE(改)、DELETE(删)、ADMIN(管理)等相关权限，这5种权限集合可以简写为crwda，每个单词的首字符拼接而成。

## 2.2 认证方式：
- world
默认方式，开放的权限，意解为全世界都能随意访问。

- auth
已经授权且认证通过的用户才可以访问。

- digest
用户名:密码方式认证，实际业务开发中最常用的方式。

- IP白名单
授权指定的Ip地址，和指定的权限点，控制访问。

## 2.3 Digest授权流程
- 添加认证用户
addauth digest 用户名:密码

- 设置权限
setAcl /path auth:用户名:密码:权限

- 查看Acl设置
getAcl /path

- 完整操作流程
```bash
-- 添加授权用户
[zk: localhost:2181] addauth digest smile:123456
-- 创建节点
[zk: localhost:2181] create /cicada cicada
-- 节点授权
[zk: localhost:2181] setAcl /cicada auth:smile:123456:cdrwa
-- 查看授权
[zk: localhost:2181] getAcl /cicada
```

# 3、SpringBoot整合Zookeeper
## 3.1 核心依赖
> Curator是Apache开源的一个Zookeeper客户端连接和操作的组件，Curator框架在Zookeeper原生API接口上进行二次包装。提供ZooKeeper各种应用场景：比如：分布式锁服务、集群领导选举、共享计数器、缓存机制、分布式队列等API封装。

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>2.12.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.12.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>2.12.0</version>
</dependency>
```
## 3.2 Zookeeper参数
```yml
zoo:
  keeper:
    #开启标志
    enabled: true
    #服务器地址
    server: 127.0.0.1:2181
    #命名空间，被称为ZNode
    namespace: cicada
    #权限控制，加密
    digest: smile:123456
    #会话超时时间
    sessionTimeoutMs: 3000
    #连接超时时间
    connectionTimeoutMs: 60000
     #最大重试次数
    maxRetries: 2
    #初始休眠时间
    baseSleepTimeMs: 1000
```
## 3.3 服务初始化配置
```java
@Configuration
public class ZookeeperConfig {
    private static final Logger LOGGER = LoggerFactory.getLogger(ZookeeperConfig.class) ;
    @Resource
    private ZookeeperParam zookeeperParam ;
    private static CuratorFramework client = null ;
    /**
     * 初始化
     */
    @PostConstruct
    public void init (){
        //重试策略，初试时间1秒，重试10次
        RetryPolicy policy = new ExponentialBackoffRetry(
                zookeeperParam.getBaseSleepTimeMs(),
                zookeeperParam.getMaxRetries());
        //通过工厂创建Curator
        client = CuratorFrameworkFactory.builder()
                .connectString(zookeeperParam.getServer())
                .authorization("digest",zookeeperParam.getDigest().getBytes())
                .connectionTimeoutMs(zookeeperParam.getConnectionTimeoutMs())
                .sessionTimeoutMs(zookeeperParam.getSessionTimeoutMs())
                .retryPolicy(policy).build();
        //开启连接
        client.start();
        LOGGER.info("zookeeper 初始化完成...");
    }
    public static CuratorFramework getClient (){
        return client ;
    }
    public static void closeClient (){
        if (client != null){
            client.close();
        }
    }
}
```
## 3.4 封装系列接口
```java
public interface ZookeeperService {
    /**
     * 判断节点是否存在
     */
    boolean isExistNode (final String path) ;
    /**
     * 创建节点
     */
    void createNode (CreateMode mode,String path ) ;
    /**
     * 设置节点数据
     */
    void setNodeData (String path, String nodeData) ;
    /**
     * 创建节点
     */
    void createNodeAndData (CreateMode mode, String path , String nodeData) ;
    /**
     * 获取节点数据
     */
    String getNodeData (String path) ;
    /**
     * 获取节点下数据
     */
    List<String> getNodeChild (String path) ;
    /**
     * 是否递归删除节点
     */
    void deleteNode (String path,Boolean recursive) ;
    /**
     * 获取读写锁
     */
    InterProcessReadWriteLock getReadWriteLock (String path) ;
}
```
## 3.5 接口实现
```java
@Service
public class ZookeeperServiceImpl implements ZookeeperService {
    private static final Logger LOGGER = LoggerFactory.getLogger(ZookeeperServiceImpl.class);
    @Override
    public boolean isExistNode(String path) {
        CuratorFramework client = ZookeeperConfig.getClient();
        client.sync() ;
        try {
            Stat stat = client.checkExists().forPath(path);
            return client.checkExists().forPath(path) != null;
        } catch (Exception e) {
            LOGGER.error("isExistNode error...", e);
            e.printStackTrace();
        }
        return false;
    }
    @Override
    public void createNode(CreateMode mode, String path) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        try {
            // 递归创建所需父节点
            client.create().creatingParentsIfNeeded().withMode(mode).forPath(path);
        } catch (Exception e) {
            LOGGER.error("createNode error...", e);
            e.printStackTrace();
        }
    }
    @Override
    public void setNodeData(String path, String nodeData) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        try {
            // 设置节点数据
            client.setData().forPath(path, nodeData.getBytes("UTF-8"));
        } catch (Exception e) {
            LOGGER.error("setNodeData error...", e);
            e.printStackTrace();
        }
    }
    @Override
    public void createNodeAndData(CreateMode mode, String path, String nodeData) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        try {
            // 创建节点，关联数据
            client.create().creatingParentsIfNeeded().withMode(mode)
                  .forPath(path,nodeData.getBytes("UTF-8"));
        } catch (Exception e) {
            LOGGER.error("createNode error...", e);
            e.printStackTrace();
        }
    }
    @Override
    public String getNodeData(String path) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        try {
            // 数据读取和转换
            byte[] dataByte = client.getData().forPath(path) ;
            String data = new String(dataByte,"UTF-8") ;
            if (StringUtils.isNotEmpty(data)){
                return data ;
            }
        }catch (Exception e) {
            LOGGER.error("getNodeData error...", e);
            e.printStackTrace();
        }
        return null;
    }
    @Override
    public List<String> getNodeChild(String path) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        List<String> nodeChildDataList = new ArrayList<>();
        try {
            // 节点下数据集
            nodeChildDataList = client.getChildren().forPath(path);
        } catch (Exception e) {
            LOGGER.error("getNodeChild error...", e);
            e.printStackTrace();
        }
        return nodeChildDataList;
    }
    @Override
    public void deleteNode(String path, Boolean recursive) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        try {
            if(recursive) {
                // 递归删除节点
                client.delete().guaranteed().deletingChildrenIfNeeded().forPath(path);
            } else {
                // 删除单个节点
                client.delete().guaranteed().forPath(path);
            }
        } catch (Exception e) {
            LOGGER.error("deleteNode error...", e);
            e.printStackTrace();
        }
    }
    @Override
    public InterProcessReadWriteLock getReadWriteLock(String path) {
        CuratorFramework client = ZookeeperConfig.getClient() ;
        // 写锁互斥、读写互斥
        InterProcessReadWriteLock readWriteLock = new InterProcessReadWriteLock(client, path);
        return readWriteLock ;
    }
}
```
## 3.6 基于Swagger2接口
```java
@Api("Zookeeper接口管理")
@RestController
public class ZookeeperApi {
    @Resource
    private ZookeeperService zookeeperService ;
    @ApiOperation(value="查询节点数据")
    @GetMapping("/getNodeData")
    public String getNodeData (String path) {
        return zookeeperService.getNodeData(path) ;
    }
    @ApiOperation(value="判断节点是否存在")
    @GetMapping("/isExistNode")
    public boolean isExistNode (final String path){
        return zookeeperService.isExistNode(path) ;
    }
    @ApiOperation(value="创建节点")
    @GetMapping("/createNode")
    public String createNode (CreateMode mode, String path ){
        zookeeperService.createNode(mode,path) ;
        return "success" ;
    }
    @ApiOperation(value="设置节点数据")
    @GetMapping("/setNodeData")
    public String setNodeData (String path, String nodeData) {
        zookeeperService.setNodeData(path,nodeData) ;
        return "success" ;
    }
    @ApiOperation(value="创建并设置节点数据")
    @GetMapping("/createNodeAndData")
    public String createNodeAndData (CreateMode mode, String path , String nodeData){
        zookeeperService.createNodeAndData(mode,path,nodeData) ;
        return "success" ;
    }
    @ApiOperation(value="递归获取节点数据")
    @GetMapping("/getNodeChild")
    public List<String> getNodeChild (String path) {
        return zookeeperService.getNodeChild(path) ;
    }
    @ApiOperation(value="是否递归删除节点")
    @GetMapping("/deleteNode")
    public String deleteNode (String path,Boolean recursive) {
        zookeeperService.deleteNode(path,recursive) ;
        return "success" ;
    }
}
```