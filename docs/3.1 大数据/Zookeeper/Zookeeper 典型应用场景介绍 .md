# Zookeeper 典型应用场景介绍

来源：微信公众号：芋道源码

# 2.具体应用

### 2.1.一致性配置管理

我们在开发的时候，有时候需要获取一些公共的配置，比如数据库连接信息等，并且偶然可能需要更新配置。如果我们的服务器有N多台的话，那修改起来会特别的麻烦，并且还需要重新启动。这里Zookeeper就可以很方便的实现类似的功能。

#### 2.1.1.思路

将公共的配置存放在Zookeeper的节点中

应用程序可以连接到Zookeeper中并对Zookeeper中配置节点进行读取或者修改（对于写操作可以进行权限验证设置），下面是具体的流程图：

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfeRHVYVOzsKcRxpFTHyDLMder3KWH3CC7tMklQgYma8996a3f6t7Kc3w0uWvh7nY8RRsDFibEUYEoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

#### 2.1.2.事例

数据库配置信息一致性的维护

配置类：

```
public class CommonConfig implements Serializable{
 // 数据库连接配置
 private String dbUrl;
 private String username;
 private String password;
 private String driverClass;

 public CommonConfig() {}

 public CommonConfig(String dbUrl, String username, String password, String driverClass) {
  super();
  this.dbUrl = dbUrl;
  this.username = username;
  this.password = password;
  this.driverClass = driverClass;
 }

 public String getDbUrl() {
  return dbUrl;
 }

 public void setDbUrl(String dbUrl) {
  this.dbUrl = dbUrl;
 }

 public String getUsername() {
  return username;
 }

 public void setUsername(String username) {
  this.username = username;
 }

 public String getPassword() {
  return password;
 }

 public void setPassword(String password) {
  this.password = password;
 }

 public String getDriverClass() {
  return driverClass;
 }

 public void setDriverClass(String driverClass) {
  this.driverClass = driverClass;
 }

 @Override
 public String toString() {
  return "CommonConfig:{dbUrl:" + this.dbUrl +
    ", username:" + this.username +
    ", password:" + this.password +
    ", driverClass:" + this.driverClass + "}";
 }
}
```

配置管理中心

- 获取本地配置信息
- 修改配置，并同步

同步配置信息到Zookeeper服务器

```
public class ZkConfigMng {
 private String nodePath = "/commConfig";
 private CommonConfig commonConfig;
 private ZkClient zkClient;

 public CommonConfig initConfig(CommonConfig commonConfig) {
  if(commonConfig == null) {
   this.commonConfig = new CommonConfig("jdbc:mysql://127.0.0.1:3306/mydata?useUnicode=true&characterEncoding=utf-8",
     "root", "root", "com.mysql.jdbc.Driver");
  } else {
   this.commonConfig = commonConfig;
  }
  return this.commonConfig;
 }

 /**
  * 更新配置
  *
  * @param commonConfig
  * @return
  */
 public CommonConfig update(CommonConfig commonConfig) {
  if(commonConfig != null) {
   this.commonConfig = commonConfig;
  }
  syncConfigToZookeeper();
  return this.commonConfig;
 }

 public void syncConfigToZookeeper() {
  if(zkClient == null) {
   zkClient = new ZkClient("127.0.0.1:2181");
  }
  if(!zkClient.exists(nodePath)) {
   zkClient.createPersistent(nodePath);
  }
  zkClient.writeData(nodePath, commonConfig);
 }
}
```

以上是提供者，下面我们需要一个客户端获取这些配置

```
public class ZkConfigClient implements Runnable {

 private String nodePath = "/commConfig";

 private CommonConfig commonConfig;

 @Override
 public void run() {
  ZkClient zkClient = new ZkClient(new ZkConnection("127.0.0.1:2181", 5000));
  while (!zkClient.exists(nodePath)) {
   System.out.println("配置节点不存在!");
   try {
    TimeUnit.SECONDS.sleep(1);
   } catch (InterruptedException e) {
    e.printStackTrace();
   }
  }
  // 获取节点
  commonConfig = (CommonConfig)zkClient.readData(nodePath);
  System.out.println(commonConfig.toString());
  zkClient.subscribeDataChanges(nodePath, new IZkDataListener() {

   @Override
   public void handleDataDeleted(String dataPath) throws Exception {
    if(dataPath.equals(nodePath)) {
     System.out.println("节点：" + dataPath + "被删除了！");
    }
   }

   @Override
   public void handleDataChange(String dataPath, Object data) throws Exception {
    if(dataPath.equals(nodePath)) {
     System.out.println("节点：" + dataPath + ", 数据：" + data + " - 更新");
     commonConfig = (CommonConfig) data;
    }
   }
  });
 }

}
```

下面启动Main函数

配置管理服务启动

```
public static void main(String[] args) throws InterruptedException {
  SpringApplication.run(ZookeeperApiDemoApplication.class, args);

  ZkConfigMng zkConfigMng = new ZkConfigMng();
  zkConfigMng.initConfig(null);
  zkConfigMng.syncConfigToZookeeper();
  TimeUnit.SECONDS.sleep(10);

  // 修改值
  zkConfigMng.update(new CommonConfig("jdbc:mysql://192.168.1.122:3306/mydata?useUnicode=true&characterEncoding=utf-8",
    "root", "wxh", "com.mysql.jdbc.Driver"));
 }
}
```

客户端启动：

```
public static void main(String[] args) throws InterruptedException {
  SpringApplication.run(ZookeeperApiDemoApplication.class, args);

  ExecutorService executorService = Executors.newFixedThreadPool(3);
  // 模拟多个客户端获取配置
  executorService.submit(new ZkConfigClient());
  executorService.submit(new ZkConfigClient());
  executorService.submit(new ZkConfigClient());
 }
}
```

### 2.2.分布式锁

在我们日常的开发中，如果是单个进程中对共享资源的访问，我们只需要用synchronized或者lock就能实现互斥操作。但是对于跨进程、跨主机、跨网络的共享资源似乎就无能为力了。

#### 2.1.1.思路

- 首先zookeeper中我们可以创建一个`/distributed_lock`持久化节点
- 然后再在`/distributed_lock`节点下创建自己的临时顺序节点，比如：`/distributed_lock/task_00000000008`
- 获取所有的`/distributed_lock`下的所有子节点，并排序
- 判读自己创建的节点是否最小值（第一位）
- 如果是，则获取得到锁，执行自己的业务逻辑，最后删除这个临时节点。
- 如果不是最小值，则需要监听自己创建节点前一位节点的数据变化，并阻塞。
- 当前一位节点被删除时，我们需要通过递归来判断自己创建的节点是否在是最小的，如果是则执行5）；如果不是则执行6）（就是递归循环的判断）

下面是具体的流程图：

[![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

#### 2.1.3.事例

```
public class DistributedLock {

 // 常亮
 static class Constant {
  private static final int SESSION_TIMEOUT = 10000;
  private static final String CONNECTION_STRING = "127.0.0.1:2181";
  private static final String LOCK_NODE = "/distributed_lock";
  private static final String CHILDREN_NODE = "/task_";
 }

 private ZkClient zkClient;

 public DistributedLock() {
  // 连接到Zookeeper
  zkClient = new ZkClient(new ZkConnection(Constant.CONNECTION_STRING));
  if(!zkClient.exists(Constant.LOCK_NODE)) {
   zkClient.create(Constant.LOCK_NODE, "分布式锁节点", CreateMode.PERSISTENT);
  }
 }

 public String getLock() {
  try {
   // 1。在Zookeeper指定节点下创建临时顺序节点
   String lockName = zkClient.createEphemeralSequential(Constant.LOCK_NODE + Constant.CHILDREN_NODE, "");
   // 尝试获取锁
   acquireLock(lockName);
   return lockName;
  } catch(Exception e) {
   e.printStackTrace();
  }

  return null;
 }

 /**
  * 获取锁
  * @throws InterruptedException
  */
 public Boolean acquireLock(String lockName) throws InterruptedException {
  // 2.获取lock节点下的所有子节点
  List<String> childrenList = zkClient.getChildren(Constant.LOCK_NODE);
  // 3.对子节点进行排序，获取最小值
  Collections.sort(childrenList, new Comparator<String>() {
   @Override
   public int compare(String o1, String o2) {
    return Integer.parseInt(o1.split("_")[1]) - Integer.parseInt(o2.split("_")[1]);
   }

  });
  // 4.判断当前创建的节点是否在第一位
  int lockPostion = childrenList.indexOf(lockName.split("/")[lockName.split("/").length - 1]);
  if(lockPostion < 0) {
   // 不存在该节点
   throw new ZkNodeExistsException("不存在的节点：" + lockName);
  } else if (lockPostion == 0) {
   // 获取到锁
   System.out.println("获取到锁：" + lockName);
   return true;
  } else if (lockPostion > 0) {
   // 未获取到锁，阻塞
   System.out.println("...... 未获取到锁，阻塞等待 。。。。。。");
   // 5.如果未获取得到锁，监听当前创建的节点前一位的节点
   final CountDownLatch latch = new CountDownLatch(1);
   IZkDataListener listener = new IZkDataListener() {

    @Override
    public void handleDataDeleted(String dataPath) throws Exception {
     // 6.前一个节点被删除,当不保证轮到自己
     System.out.println("。。。。。。前一个节点被删除  。。。。。。");
     acquireLock(lockName);
     latch.countDown();
    }

    @Override
    public void handleDataChange(String dataPath, Object data) throws Exception {
     // 不用理会
    }
   };
   try {
    zkClient.subscribeDataChanges(Constant.LOCK_NODE + "/" + childrenList.get(lockPostion - 1), listener);
    latch.await();
   } finally {
    zkClient.unsubscribeDataChanges(Constant.LOCK_NODE + "/" + childrenList.get(lockPostion - 1), listener);
   }
  }
  return false;
 }

 /**
  * 释放锁（删除节点）
  *
  * @param lockName
  */
 public void releaseLock(String lockName) {
  zkClient.delete(lockName);
 }

 public void closeZkClient() {
  zkClient.close();
 }
}

@SpringBootApplication
public class ZookeeperDemoApplication {

 public static void main(String[] args) throws InterruptedException {
  SpringApplication.run(ZookeeperDemoApplication.class, args);

  DistributedLock lock = new DistributedLock();
  String lockName = lock.getLock();
  /**
   * 执行我们的业务逻辑
   */
  if(lockName != null) {
   lock.releaseLock(lockName);
  }

  lock.closeZkClient();
 }
}
```

### 2.3.分布式队列

在日常使用中，特别是像生产者消费者模式中，经常会使用BlockingQueue来充当缓冲区的角色。但是在分布式系统中这种方式就不能使用BlockingQueue来实现了，但是Zookeeper可以实现。

#### 2.1.1.思路

- 首先利用Zookeeper中临时顺序节点的特点
- 当生产者创建节点生产时，需要判断父节点下临时顺序子节点的个数，如果达到了上限，则阻塞等待；如果没有达到，就创建节点。
- 当消费者获取节点时，如果父节点中不存在临时顺序子节点，则阻塞等待；如果有子节点，则获取执行自己的业务，执行完毕后删除该节点即可。
- 获取时获取最小值，保证FIFO特性。

#### 2.1.2.事例

这个是一个消费者对一个生产者，如果是多个消费者对多个生产者，对代码需要调整。

```
public interface AppConstant {
 static String ZK_CONNECT_STR = "127.0.0.1:2181";
 static String NODE_PATH = "/mailbox";
 static String CHILD_NODE_PATH = "/mail_";
 static int MAILBOX_SIZE = 10;
}

public class MailConsumer implements Runnable, AppConstant{

 private ZkClient zkClient;
 private Lock lock;
 private Condition condition;

 public MailConsumer() {
  lock = new ReentrantLock();
  condition = lock.newCondition();
  zkClient = new ZkClient(new ZkConnection(ZK_CONNECT_STR));
  System.out.println("sucess connected to zookeeper server!");
  // 不存在就创建mailbox节点
  if(!zkClient.exists(NODE_PATH)) {
   zkClient.create(NODE_PATH, "this is mailbox", CreateMode.PERSISTENT);
  }
 }

 @Override
 public void run() {
  IZkChildListener listener = new IZkChildListener() {
   @Override
   public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
    System.out.println("Znode["+parentPath + "] size:" + currentChilds.size());
    // 还是要判断邮箱是否为空
    if(currentChilds.size() > 0) {
     // 唤醒等待的线程
     try {
      lock.lock();
      condition.signal();
     } catch (Exception e) {
      e.printStackTrace();
     } finally {
      lock.unlock();
     }
    }
   }
  };
  // 监视子节点的改变，不用放用while循环中，监听一次就行了，不需要重复绑定
  zkClient.subscribeChildChanges(NODE_PATH, listener);
  try {
   //循环随机发送邮件模拟真是情况
   while(true) {
    // 判断是否可以发送邮件
    checkMailReceive();
    // 接受邮件
    List<String> mailList = zkClient.getChildren(NODE_PATH);
    // 如果mailsize==0,也没有关系；可以直接循环获取就行了
    if(mailList.size() > 0) {
     Collections.sort(mailList, new Comparator<String>() {
      @Override
      public int compare(String o1, String o2) {
       return Integer.parseInt(o1.split("_")[1]) - Integer.parseInt(o2.split("_")[1]);
      }
     });
     // 模拟邮件处理(0-1S)
     TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
     zkClient.delete(NODE_PATH + "/" + mailList.get(0));
     System.out.println("mail has been received:" + NODE_PATH + "/" + mailList.get(0));
    }
   }
  }catch (Exception e) {
   e.printStackTrace();
  } finally {
   zkClient.unsubscribeChildChanges(NODE_PATH, listener);
  }
 }

 private void checkMailReceive() {
  try {
   lock.lock();
   // 判断邮箱是为空
   List<String> mailList = zkClient.getChildren(NODE_PATH);
   System.out.println("mailbox size: " + mailList.size());
   if(mailList.size() == 0) {
    // 邮箱为空，阻塞消费者，直到邮箱有邮件
    System.out.println("mailbox is empty, please wait 。。。");
    condition.await();
    // checkMailReceive();
   }
  } catch (Exception e) {
   e.printStackTrace();
  } finally {
   lock.unlock();
  }
 }
}

public class MailProducer implements Runnable, AppConstant{

 private ZkClient zkClient;
 private Lock lock;
 private Condition condition;

 /**
  * 初始化状态
  */
 public MailProducer() {
  lock = new ReentrantLock();
  condition = lock.newCondition();
  zkClient = new ZkClient(new ZkConnection(ZK_CONNECT_STR));
  System.out.println("sucess connected to zookeeper server!");
  // 不存在就创建mailbox节点
  if(!zkClient.exists(NODE_PATH)) {
   zkClient.create(NODE_PATH, "this is mailbox", CreateMode.PERSISTENT);
  }
 }

 @Override
 public void run() {
  IZkChildListener listener = new IZkChildListener() {
   @Override
   public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
    System.out.println("Znode["+parentPath + "] size:" + currentChilds.size());
    // 还是要判断邮箱是否已满
    if(currentChilds.size() < MAILBOX_SIZE) {
     // 唤醒等待的线程
     try {
      lock.lock();
      condition.signal();
     } catch (Exception e) {
      e.printStackTrace();
     } finally {
      lock.unlock();
     }
    }
   }
  };
  // 监视子节点的改变，不用放用while循环中，监听一次就行了，不需要重复绑定
  zkClient.subscribeChildChanges(NODE_PATH, listener);
  try {
   //循环随机发送邮件模拟真是情况
   while(true) {
    // 判断是否可以发送邮件
    checkMailSend();
    // 发送邮件
    String cretePath = zkClient.createEphemeralSequential(NODE_PATH + CHILD_NODE_PATH, "your mail");
    System.out.println("your mail has been send:" + cretePath);
    // 模拟随机间隔的发送邮件(0-10S)
    TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
   }
  }catch (Exception e) {
   e.printStackTrace();
  } finally {
   zkClient.unsubscribeChildChanges(NODE_PATH, listener);
  }
 }

 private void checkMailSend() {
  try {
   lock.lock();
   // 判断邮箱是否已满
   List<String> mailList = zkClient.getChildren(NODE_PATH);
   System.out.println("mailbox size: " + mailList.size());
   if(mailList.size() >= MAILBOX_SIZE) {
    // 邮箱已满，阻塞生产者，直到邮箱有空间
    System.out.println("mailbox is full, please wait 。。。");
    condition.await();
    checkMailSend();
   }
  } catch (Exception e) {
   e.printStackTrace();
  } finally {
   lock.unlock();
  }
 }
}
```

### 2.4.均衡负载

首先我们需要简单的理解分布式和集群，通俗点说：分布式就是将一个系统拆分到多个独立运行的应用中（有可能在同一台主机也有可能在不同的主机上），集群就是将单个独立的应用复制多分放在不同的主机上来减轻服务器的压力。

而Zookeeper不仅仅可以作为分布式集群的服务注册调度中心（例如dubbo），也可以实现集群的负载均衡。

#### 2.4.1.思路

首先我们要理解，如果是一个集群，那么他就会有多台主机。所以，他在Zookeeper中信息的存在应该是如下所示：

[![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&chksm=fa496f8ecd3ee698f4954c00efb80fe955ec9198fff3ef4011e331aa37f55a6a17bc8c0335a8&scene=21&token=899450012&lang=zh_CN#wechat_redirect)图片

如上的结构，当服务调用方调用服务时，就可以根据特定的均衡负载算法来实现对服务的调用（调用前需要监听/service/serviceXXX节点，以更新列表数据）

#### 2.4.2.事例

```
/**
 * 服务提供者
 *
 * @author Administrator
 *
 */
public class ServiceProvider {
 // 静态常量
 static String ZK_CONNECT_STR = "127.0.0.1:2181";
 static String NODE_PATH = "/service";
 static String SERIVCE_NAME = "/myService";

 private ZkClient zkClient;

 public ServiceProvider() {
  zkClient = new ZkClient(new ZkConnection(ZK_CONNECT_STR));
  System.out.println("sucess connected to zookeeper server!");
  // 不存在就创建NODE_PATH节点
  if(!zkClient.exists(NODE_PATH)) {
   zkClient.create(NODE_PATH, "this is mailbox", CreateMode.PERSISTENT);
  }
 }

 public void registryService(String localIp, Object obj) {
  if(!zkClient.exists(NODE_PATH + SERIVCE_NAME)) {
   zkClient.create(NODE_PATH + SERIVCE_NAME, "provider services list", CreateMode.PERSISTENT);
  }
  // 对自己的服务进行注册
  zkClient.createEphemeral(NODE_PATH + SERIVCE_NAME + "/" + localIp, obj);
  System.out.println("注册成功！[" + localIp + "]");
 }
}

/**
 * 消费者，通过某种均衡负载算法选择某一个提供者
 *
 * @author Administrator
 *
 */
public class ServiceConsumer {
 // 静态常量
 static String ZK_CONNECT_STR = "127.0.0.1:2181";
 static String NODE_PATH = "/service";
 static String SERIVCE_NAME = "/myService";

 private List<String> serviceList = new ArrayList<String>();

 private ZkClient zkClient;

 public ServiceConsumer() {
  zkClient = new ZkClient(new ZkConnection(ZK_CONNECT_STR));
  System.out.println("sucess connected to zookeeper server!");
  // 不存在就创建NODE_PATH节点
  if(!zkClient.exists(NODE_PATH)) {
   zkClient.create(NODE_PATH, "this is mailbox", CreateMode.PERSISTENT);
  }
 }

 /**
  * 订阅服务
  */
 public void subscribeSerivce() {
  serviceList = zkClient.getChildren(NODE_PATH + SERIVCE_NAME);
  zkClient.subscribeChildChanges(NODE_PATH + SERIVCE_NAME, new IZkChildListener() {
   @Override
   public void handleChildChange(String parentPath, List<String> currentChilds) throws Exception {
    serviceList = currentChilds;
   }
  });
 }

 /**
  * 模拟调用服务
  */
 public void consume() {
  //负载均衡算法获取某台机器调用服务
  int index = new Random().nextInt(serviceList.size());
  System.out.println("调用[" + NODE_PATH + SERIVCE_NAME + "]服务：" + serviceList.get(index));
 }
}
```