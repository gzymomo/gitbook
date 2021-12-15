- [基于SpringBoot集成线程池，实现线程的池的动态监控（超级详细，建议收藏）](https://www.cnblogs.com/mic112/p/15424574.html)

线程池的监控很重要，对于前面章节讲的动态参数调整，其实还是得依赖于线程池监控的数据反馈之后才能做出调整的决策。还有就是线程池本身的运行过程对于我们来说像一个黑盒，我们没办法了解线程池中的运行状态时，出现问题没有办法及时判断和预警。

对于监控这类的场景，核心逻辑就是要拿到关键指标，然后进行上报，只要能实时拿到这些关键指标，就可以轻松实现监控以及预警功能。

ThreadPoolExecutor中提供了以下方法来获取线程池中的指标。

- getCorePoolSize()：获取核心线程数。
- getMaximumPoolSize：获取最大线程数。
- getQueue()：获取线程池中的阻塞队列，并通过阻塞队列中的方法获取队列长度、元素个数等。
- getPoolSize()：获取线程池中的工作线程数（包括核心线程和非核心线程）。
- getActiveCount()：获取活跃线程数，也就是正在执行任务的线程。
- getLargestPoolSize()：获取线程池曾经到过的最大工作线程数。
- getTaskCount()：获取历史已完成以及正在执行的总的任务数量。

除此之外，ThreadPoolExecutor中还提供了一些未实现的钩子方法，我们可以通过重写这些方法来实现更多指标数据的获取。

- beforeExecute，在Worker线程执行任务之前会调用的方法。
- afterExecute，在Worker线程执行任务之后会调用的方法。
- terminated，当线程池从状态变更到TERMINATED状态之前调用的方法。

比如我们可以在`beforeExecute`方法中记录当前任务开始执行的时间，再到`afterExecute`方法来计算任务执行的耗时、最大耗时、最小耗时、平均耗时等。

# 线程池监控的基本原理

我们可以通过Spring  Boot提供的Actuator，自定义一个Endpoint来发布线程池的指标数据，实现线程池监控功能。当然，除了Endpoint以外，我们还可以通过JMX的方式来暴露线程池的指标信息，不管通过什么方法，核心思想都是要有一个地方看到这些数据。

了解对于Spring  Boot应用监控得读者应该知道，通过Endpoint发布指标数据后，可以采用一些主流的开源监控工具来进行采集和展示。如图10-9所示，假设在Spring  Boot应用中发布一个获取线程池指标信息的Endpoint，那么我们可以采用Prometheus定时去抓取目标服务器上的Metric数据，Prometheus会将采集到的数据通过Retrieval分发给TSDB进行存储。这些数据可以通过Prometheus自带的UI进行展示，也可以使用Grafana图表工具通过PromQL语句来查询Prometheus中采集的数据进行渲染。最后采用AlertManager这个组件来触发预警功能。

![在这里插入图片描述](https://img2020.cnblogs.com/other/1666682/202110/1666682-20211019142556569-1470425450.png)

图10-9 线程池指标监控

图10-9中所涉及到的工具都是比较程度的开源监控组件，大家可以自行根据官方教程配置即可，而在本章节中要重点讲解的就是如何自定义Endpoint发布线程池的Metric数据。

# 在Spring Boot应用中发布线程池信息

对于线程池的监控实现，笔者开发了一个相对较为完整的小程序，主要涉及到几个功能：

- 可以通过配置文件来构建线程池。
- 扩展了ThreadPoolExecutor的实现。
- 发布一个自定义的Endpoint。

该小程序包含的类以及功能说明如下：

- ThreadPoolExecutorForMonitor：扩展ThreadPoolExecutor的实现类。
- ThreadPoolConfigurationProperties：绑定application.properties的配置属性。
- ThreadPoolForMonitorManager：线程池管理类，实现线程池的初始化。
- ThreadPoolProperties：线程池基本属性。
- ResizeLinkedBlockingQueue：这个类是直接复制了LinkedBlockingQueue，提供了`setCapacity`方法，在前面有讲解到，源码就不贴出来。
- ThreadPoolEndpoint：自定义Endpoint。

# ThreadPoolExecutorForMonitor

继承了ThreadPoolExecutor，实现了`beforeExecute`和`afterExecute`，在原有线程池的基础上新增了最短执行时间、最长执行时间、平均执行耗时的属性。

```java
public class ThreadPoolExecutorForMonitor extends ThreadPoolExecutor {

  private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();

  private static final String defaultPoolName="Default-Task";

  private static ThreadFactory threadFactory=new MonitorThreadFactory(defaultPoolName);

  public ThreadPoolExecutorForMonitor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,threadFactory,defaultHandler);
  }
  public ThreadPoolExecutorForMonitor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue,String poolName) {
    super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,new MonitorThreadFactory(poolName),defaultHandler);
  }
  public ThreadPoolExecutorForMonitor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler,String poolName) {
    super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,threadFactory,handler);
  }

  //最短执行时间
  private long minCostTime;
  //最长执行时间
  private long maxCostTime;
  //总的耗时
  private AtomicLong totalCostTime=new AtomicLong();

  private ThreadLocal<Long> startTimeThreadLocal=new ThreadLocal<>();

  @Override
  public void shutdown() {
    super.shutdown();
  }

  @Override
  protected void beforeExecute(Thread t, Runnable r) {
    startTimeThreadLocal.set(System.currentTimeMillis());
    super.beforeExecute(t, r);
  }

  @Override
  protected void afterExecute(Runnable r, Throwable t) {
    long costTime=System.currentTimeMillis()-startTimeThreadLocal.get();
    startTimeThreadLocal.remove();
    maxCostTime=maxCostTime>costTime?maxCostTime:costTime;
    if(getCompletedTaskCount()==0){
      minCostTime=costTime;
    }
    minCostTime=minCostTime<costTime?minCostTime:costTime;
    totalCostTime.addAndGet(costTime);
    super.afterExecute(r, t);
  }

  public long getMinCostTime() {
    return minCostTime;
  }

  public long getMaxCostTime() {
    return maxCostTime;
  }

  public long getAverageCostTime(){//平均耗时
    if(getCompletedTaskCount()==0||totalCostTime.get()==0){
      return 0;
    }
    return totalCostTime.get()/getCompletedTaskCount();
  }

  @Override
  protected void terminated() {
    super.terminated();
  }

  static class MonitorThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    MonitorThreadFactory(String poolName) {
      SecurityManager s = System.getSecurityManager();
      group = (s != null) ? s.getThreadGroup() :
      Thread.currentThread().getThreadGroup();
      namePrefix = poolName+"-pool-" +
        poolNumber.getAndIncrement() +
        "-thread-";
    }

    public Thread newThread(Runnable r) {
      Thread t = new Thread(group, r,
                            namePrefix + threadNumber.getAndIncrement(),
                            0);
      if (t.isDaemon())
        t.setDaemon(false);
      if (t.getPriority() != Thread.NORM_PRIORITY)
        t.setPriority(Thread.NORM_PRIORITY);
      return t;
    }
  }
}
```

# ThreadPoolConfigurationProperties

提供了获取application.properties配置文件属性的功能，

```java
@ConfigurationProperties(prefix = "monitor.threadpool")
@Data
public class ThreadPoolConfigurationProperties {

    private List<ThreadPoolProperties>  executors=new ArrayList<>();

}
```

线程池的核心属性声明。

```java
@Data
public class ThreadPoolProperties {

    private String poolName;
    private int corePoolSize;
    private int maxmumPoolSize=Runtime.getRuntime().availableProcessors();
    private long keepAliveTime=60;
    private TimeUnit unit= TimeUnit.SECONDS;
    private int queueCapacity=Integer.MAX_VALUE;
}
```

上述配置类要生效，需要通过@EnableConfigurationProperties开启，我们可以在Main方法上开启，代码如下。

```java
@EnableConfigurationProperties(ThreadPoolConfigurationProperties.class)
@SpringBootApplication
public class ThreadPoolApplication {

    public static void main(String[] args) {
        SpringApplication.run(ThreadPoolApplication.class, args);
    }
}
```

# application.properties

配置类创建好之后，我们就可以在application.properties中，通过如下方式来构建线程池。

```properties
monitor.threadpool.executors[0].pool-name=first-monitor-thread-pool
monitor.threadpool.executors[0].core-pool-size=4
monitor.threadpool.executors[0].maxmum-pool-size=8
monitor.threadpool.executors[0].queue-capacity=100

monitor.threadpool.executors[1].pool-name=second-monitor-thread-pool
monitor.threadpool.executors[1].core-pool-size=2
monitor.threadpool.executors[1].maxmum-pool-size=4
monitor.threadpool.executors[1].queue-capacity=40
```

# ThreadPoolForMonitorManager

用来实现线程池的管理和初始化，实现线程池的统一管理，初始化的逻辑是根据application.properties中配置的属性来实现的。

- 从配置类中获得线程池的基本配置。
- 根据配置信息构建ThreadPoolExecutorForMonitor实例。
- 把实例信息保存到集合中。

```java
@Component
public class ThreadPoolForMonitorManager {

  @Autowired
  ThreadPoolConfigurationProperties poolConfigurationProperties;

  private final ConcurrentMap<String,ThreadPoolExecutorForMonitor> threadPoolExecutorForMonitorConcurrentMap=new ConcurrentHashMap<>();

  @PostConstruct
  public void init(){
    poolConfigurationProperties.getExecutors().forEach(threadPoolProperties -> {
      if(!threadPoolExecutorForMonitorConcurrentMap.containsKey(threadPoolProperties.getPoolName())){
        ThreadPoolExecutorForMonitor executorForMonitor=new ThreadPoolExecutorForMonitor(
          threadPoolProperties.getCorePoolSize(),
          threadPoolProperties.getMaxmumPoolSize(),
          threadPoolProperties.getKeepAliveTime(),
          threadPoolProperties.getUnit(),
          new ResizeLinkedBlockingQueue<>(threadPoolProperties.getQueueCapacity()),
          threadPoolProperties.getPoolName());
        threadPoolExecutorForMonitorConcurrentMap.put(threadPoolProperties.getPoolName(),executorForMonitor);
      }
    });
  }

  public ThreadPoolExecutorForMonitor getThreadPoolExecutor(String poolName){
    ThreadPoolExecutorForMonitor threadPoolExecutorForMonitor=threadPoolExecutorForMonitorConcurrentMap.get(poolName);
    if(threadPoolExecutorForMonitor==null){
      throw new RuntimeException("找不到名字为"+poolName+"的线程池");
    }
    return threadPoolExecutorForMonitor;
  }

  public ConcurrentMap<String,ThreadPoolExecutorForMonitor> getThreadPoolExecutorForMonitorConcurrentMap(){
    return this.threadPoolExecutorForMonitorConcurrentMap;
  }
}
```

# ThreadPoolEndpoint

使用Spring-Boot-Actuator发布Endpoint，用来暴露当前应用中所有线程池的Metric数据。

> 读者如果不清楚在Spring Boot中自定义Endpoint，可以直接去Spring官方文档中配置，比较简单。

```java
@Configuration
@Endpoint(id="thread-pool")
public class ThreadPoolEndpoint {
  @Autowired
  private ThreadPoolForMonitorManager threadPoolForMonitorManager;

  @ReadOperation
  public Map<String,Object> threadPoolsMetric(){
    Map<String,Object> metricMap=new HashMap<>();
    List<Map> threadPools=new ArrayList<>();
    threadPoolForMonitorManager.getThreadPoolExecutorForMonitorConcurrentMap().forEach((k,v)->{
      ThreadPoolExecutorForMonitor tpe=(ThreadPoolExecutorForMonitor) v;
      Map<String,Object> poolInfo=new HashMap<>();
      poolInfo.put("thread.pool.name",k);
      poolInfo.put("thread.pool.core.size",tpe.getCorePoolSize());
      poolInfo.put("thread.pool.largest.size",tpe.getLargestPoolSize());
      poolInfo.put("thread.pool.max.size",tpe.getMaximumPoolSize());
      poolInfo.put("thread.pool.thread.count",tpe.getPoolSize());
      poolInfo.put("thread.pool.max.costTime",tpe.getMaxCostTime());
      poolInfo.put("thread.pool.average.costTime",tpe.getAverageCostTime());
      poolInfo.put("thread.pool.min.costTime",tpe.getMinCostTime());
      poolInfo.put("thread.pool.active.count",tpe.getActiveCount());
      poolInfo.put("thread.pool.completed.taskCount",tpe.getCompletedTaskCount());
      poolInfo.put("thread.pool.queue.name",tpe.getQueue().getClass().getName());
      poolInfo.put("thread.pool.rejected.name",tpe.getRejectedExecutionHandler().getClass().getName());
      poolInfo.put("thread.pool.task.count",tpe.getTaskCount());
      threadPools.add(poolInfo);
    });
    metricMap.put("threadPools",threadPools);
    return metricMap;
  }
}
```

如果需要上述自定义的Endpoint可以被访问，还需要在application.properties文件中配置如下代码，意味着thread-pool Endpoint允许被访问。

```properties
management.endpoints.web.exposure.include=thread-pool
```

# TestController

提供使用线程池的方法，用来实现在调用之前和调用之后，通过Endpoint获取到Metric数据的变化。

```java
@RestController
public class TestController {

  private final String poolName="first-monitor-thread-pool";
  @Autowired
  ThreadPoolForMonitorManager threadPoolForMonitorManager;

  @GetMapping("/execute")
  public String doExecute(){
    ThreadPoolExecutorForMonitor tpe=threadPoolForMonitorManager.getThreadPoolExecutor(poolName);
    for (int i = 0; i < 100; i++) {
      tpe.execute(()->{
        try {
          Thread.sleep(new Random().nextInt(4000));
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      });
    }
    return "success";
  }
}
```

# 效果演示

访问自定义Endpoint： http://ip:8080/actuator/thread-pool，就可以看到如下数据。我们可以把这个Endpoint配置到Prometheus中，Prometheus会定时抓取这些指标存储并展示，从而完成线程池的整体监控。

```json
{
    "threadPools":[
        {
            "thread.pool.queue.name":"com.concurrent.demo.ResizeLinkedBlockingQueue",
            "thread.pool.core.size":2,
            "thread.pool.min.costTime":0,
            "thread.pool.completed.taskCount":0,
            "thread.pool.max.costTime":0,
            "thread.pool.task.count":0,
            "thread.pool.name":"second-monitor-thread-pool",
            "thread.pool.largest.size":0,
            "thread.pool.rejected.name":"java.util.concurrent.ThreadPoolExecutor$AbortPolicy",
            "thread.pool.active.count":0,
            "thread.pool.thread.count":0,
            "thread.pool.average.costTime":0,
            "thread.pool.max.size":4
        },
        {
            "thread.pool.queue.name":"com.concurrent.demo.ResizeLinkedBlockingQueue",
            "thread.pool.core.size":4,
            "thread.pool.min.costTime":65,
            "thread.pool.completed.taskCount":115,
            "thread.pool.max.costTime":3964,
            "thread.pool.task.count":200,
            "thread.pool.name":"first-monitor-thread-pool",
            "thread.pool.largest.size":4,
            "thread.pool.rejected.name":"java.util.concurrent.ThreadPoolExecutor$AbortPolicy",
            "thread.pool.active.count":4,
            "thread.pool.thread.count":4,
            "thread.pool.average.costTime":1955,
            "thread.pool.max.size":8
        }
    ]
}
```

# 总结

线程池的整体实现并不算太复杂，但是里面涉及到的一些思想和理论是可以值得我们去学习和借鉴，如基于阻塞队列的生产者消费者模型的实现、动态扩容的思想、如何通过AQS来实现安全关闭线程池、降级方案（拒绝策略）、位运算等。实际上越底层的实现，越包含更多技术层面的思想和理论。

线程池在实际使用中，如果是新手，不建议直接用Executors中提供的工厂方法，因为线程池中的参数会影响到内存以及CPU资源的占用，我们可以自己集成ThreadPoolExecutor这个类，扩展一个自己的实现，也可以自己构造ThreadPoolExecutor实例，这样能够更好的了解线程池中核心参数的意义避免不必要的生产问题。