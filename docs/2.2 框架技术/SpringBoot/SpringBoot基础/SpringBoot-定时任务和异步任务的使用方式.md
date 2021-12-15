[TOC]

# 1、定时任务
按照指定时间执行的程序。
**使用场景**

1. 数据分析
2. 数据清理
3. 系统服务监控

# 2、同步和异步
- 同步调用
程序按照代码顺序依次执行，每一行程序都必须等待上一行程序执行完成之后才能执行；

- 异步调用
顺序执行时，不等待异步调用的代码块返回结果就执行后面的程序。

**使用场景**

1. 短信通知
2. 邮件发送
3. 批量数据入缓存

# 3、定时器的使用
## 3.1 定时器执行规则注解
```java
@Scheduled(fixedRate = 5000) ：上一次开始执行时间点之后5秒再执行
@Scheduled(fixedDelay = 5000) ：上一次执行完毕时间点之后5秒再执行
@Scheduled(initialDelay=1000, fixedRate=5000) ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
@Scheduled(cron="/5") ：通过cron表达式定义规则
```

## 3.2 定义时间打印定时器
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.text.SimpleDateFormat;
import java.util.Date;
/**
 * 时间定时任务
 */
@Component
public class TimeTask {
    Logger LOG = LoggerFactory.getLogger(TimeTask.class.getName()) ;
    private static final SimpleDateFormat format =
            new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") ;
    /**
     * 每3秒打印一次系统时间
     */
    @Scheduled(fixedDelay = 3000)
    public void systemDate (){
        LOG.info("当前时间::::"+format.format(new Date()));
    }
}
```

## 3.3 启动类开启定时器注解
```j
@EnableScheduling   // 启用定时任务
@SpringBootApplication
public class TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaskApplication.class,args) ;
    }
}
```

# 4、异步任务
## 4.1 编写异步任务类
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;
@Component
public class AsyncTask {
    private static final Logger LOGGER = LoggerFactory.getLogger(AsyncTask.class) ;
    /*
     * [ asyncTask1-2] com.boot.task.config.AsyncTask : ======异步任务结束1======
     * [ asyncTask1-1] com.boot.task.config.AsyncTask : ======异步任务结束0======
     */
    // 只配置了一个 asyncExecutor1 不指定也会默认使用
    @Async
    public void asyncTask0 () {
        try{
            Thread.sleep(5000);
        }catch (Exception e){
            e.printStackTrace();
        }
        LOGGER.info("======异步任务结束0======");
    }
    @Async("asyncExecutor1")
    public void asyncTask1 () {
        try{
            Thread.sleep(5000);
        }catch (Exception e){
            e.printStackTrace();
        }
        LOGGER.info("======异步任务结束1======");
    }
}
```
## 4.2 指定异步任务执行的线程池
这里可以不指定，指定执行的线城池，可以更加方便的监控和管理异步任务的执行。
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;
/**
 * 定义异步任务执行的线程池
 */
@Configuration
public class TaskPoolConfig {
    @Bean("asyncExecutor1")
    public Executor taskExecutor1 () {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数10：线程池创建时候初始化的线程数
        executor.setCorePoolSize(10);
        // 最大线程数20：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(20);
        // 缓冲队列200：用来缓冲执行任务的队列
        executor.setQueueCapacity(200);
        // 允许线程的空闲时间60秒：当超过了核心线程出之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便定位处理任务所在的线程池
        executor.setThreadNamePrefix("asyncTask1-");
        /*
        线程池对拒绝任务的处理策略：这里采用了CallerRunsPolicy策略，
        当线程池没有处理能力的时候，该策略会直接在 execute 方法的调用线程中运行被拒绝的任务；
        如果执行程序已关闭，则会丢弃该任务
         */
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        executor.setWaitForTasksToCompleteOnShutdown(true);
        // 设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住。
        executor.setAwaitTerminationSeconds(600);
        return executor;
    }
}
```

## 4.3 启动类添加异步注解
```java
@EnableAsync        // 启用异步任务
@SpringBootApplication
public class TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaskApplication.class,args) ;
    }
}
4、异步调用的测试接口
@RestController
public class TaskController {
    @Resource
    private AsyncTask asyncTask ;
    @RequestMapping("/asyncTask")
    public String asyncTask (){
        asyncTask.asyncTask0();
        asyncTask.asyncTask1();
        return "success" ;
    }
}
```