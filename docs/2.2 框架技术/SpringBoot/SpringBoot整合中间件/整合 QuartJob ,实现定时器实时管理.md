[TOC]

# 1、QuartJob简介
Quartz是一个完全由java编写的开源作业调度框架，形式简易，功能强大。

## 1.1 核心API
1. Scheduler
代表一个 Quartz 的独立运行容器，Scheduler 将 Trigger 绑定到特定 JobDetail， 这样当 Trigger 触发时， 对应的 Job 就会被调度。
2. Trigger
描述 Job 执行的时间触发规则。主要有 SimpleTrigger 和 CronTrigger 两个子类，通过一个 TriggerKey 唯一标识。
3. Job
定义一个任务，规定了任务是执行时的行为。JobExecutionContext 提供了调度器的上下文信息，Job 的数据可从 JobDataMap 中获取。
4. JobDetail
Quartz 在每次执行 Job 时，都重新创建一个 Job 实例，所以它不直接接受一个 Job 的实例，相反它接收一个 Job 实现类。描述 Job 的实现类及其它相关的静态信息，如 Job 名字、描述等。

# 2、SpringBoot整合QuartJob
## 2.1 项目结构
![](https://mmbiz.qpic.cn/mmbiz_jpg/uUIibyNXbAvDRUpHW5ylriakWLCvERx1tsokuDeQX82SpZZvfkCYqQWEm7ya28CMYBGcYxv4NPZeNicuKEbxWEakQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

版本描述
```java
spring-boot：2.1.3.RELEASE
quart-job：2.3.0
```

## 2.2 定时器配置
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;
import javax.sql.DataSource;
import java.util.Properties;
@Configuration
public class ScheduleConfig {
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean(DataSource dataSource) {
        // Quartz参数配置
        Properties prop = new Properties();
        // Schedule调度器的实体名字
        prop.put("org.quartz.scheduler.instanceName", "HuskyScheduler");
        // 设置为AUTO时使用，默认的实现org.quartz.scheduler.SimpleInstanceGenerator是基于主机名称和时间戳生成。
        prop.put("org.quartz.scheduler.instanceId", "AUTO");
        // 线程池配置
        prop.put("org.quartz.threadPool.class", "org.quartz.simpl.SimpleThreadPool");
        prop.put("org.quartz.threadPool.threadCount", "20");
        prop.put("org.quartz.threadPool.threadPriority", "5");
        // JobStore配置:Scheduler在运行时用来存储相关的信息
        // JDBCJobStore和JobStoreTX都使用关系数据库来存储Schedule相关的信息。
        // JobStoreTX在每次执行任务后都使用commit或者rollback来提交更改。
        prop.put("org.quartz.jobStore.class", "org.quartz.impl.jdbcjobstore.JobStoreTX");
        // 集群配置：如果有多个调度器实体的话则必须设置为true
        prop.put("org.quartz.jobStore.isClustered", "true");
        // 集群配置：检查集群下的其他调度器实体的时间间隔
        prop.put("org.quartz.jobStore.clusterCheckinInterval", "15000");
        // 设置一个频度(毫秒)，用于实例报告给集群中的其他实例
        prop.put("org.quartz.jobStore.maxMisfiresToHandleAtATime", "1");
        // 触发器触发失败后再次触犯的时间间隔
        prop.put("org.quartz.jobStore.misfireThreshold", "12000");
        // 数据库表前缀
        prop.put("org.quartz.jobStore.tablePrefix", "qrtz_");
        // 从 LOCKS 表查询一行并对这行记录加锁的 SQL 语句
        prop.put("org.quartz.jobStore.selectWithLockSQL", "SELECT * FROM {0}LOCKS UPDLOCK WHERE LOCK_NAME = ?");

        // 定时器工厂配置
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setQuartzProperties(prop);
        factory.setSchedulerName("HuskyScheduler");
        factory.setStartupDelay(30);
        factory.setApplicationContextSchedulerContextKey("applicationContextKey");
        // 可选，QuartzScheduler 启动时更新己存在的Job
        factory.setOverwriteExistingJobs(true);
        // 设置自动启动，默认为true
        factory.setAutoStartup(true);
        return factory;
    }
}
```
## 2.3 定时器管理工具
```java
import com.quart.job.entity.ScheduleJobBean;
import org.quartz.*;
/**
 * 定时器工具类
 */
public class ScheduleUtil {
    private ScheduleUtil (){}
    private static final String SCHEDULE_NAME = "HUSKY_" ;
    /**
     * 触发器 KEY
     */
    public static TriggerKey getTriggerKey(Long jobId){
        return TriggerKey.triggerKey(SCHEDULE_NAME+jobId) ;
    }
    /**
     * 定时器 Key
     */
    public static JobKey getJobKey (Long jobId){
        return JobKey.jobKey(SCHEDULE_NAME+jobId) ;
    }
    /**
     * 表达式触发器
     */
    public static CronTrigger getCronTrigger (Scheduler scheduler,Long jobId){
        try {
            return (CronTrigger)scheduler.getTrigger(getTriggerKey(jobId)) ;
        } catch (SchedulerException e){
            throw new RuntimeException("getCronTrigger Fail",e) ;
        }
    }
    /**
     * 创建定时器
     */
    public static void createJob (Scheduler scheduler, ScheduleJobBean scheduleJob){
        try {
            // 构建定时器
            JobDetail jobDetail = JobBuilder.newJob(TaskJobLog.class).withIdentity(getJobKey(scheduleJob.getJobId())).build() ;
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder
                    .cronSchedule(scheduleJob.getCronExpression())
                    .withMisfireHandlingInstructionDoNothing() ;
            CronTrigger trigger = TriggerBuilder.newTrigger()
                    .withIdentity(getTriggerKey(scheduleJob.getJobId()))
                    .withSchedule(scheduleBuilder).build() ;
            jobDetail.getJobDataMap().put(ScheduleJobBean.JOB_PARAM_KEY,scheduleJob);
            scheduler.scheduleJob(jobDetail,trigger) ;
            // 如果该定时器处于暂停状态
            if (scheduleJob.getStatus() == 1){
                pauseJob(scheduler,scheduleJob.getJobId()) ;
            }
        } catch (SchedulerException e){
            throw new RuntimeException("createJob Fail",e) ;
        }
    }
    /**
     * 更新定时任务
     */
    public static void updateJob(Scheduler scheduler, ScheduleJobBean scheduleJob) {
        try {
            // 构建定时器
            TriggerKey triggerKey = getTriggerKey(scheduleJob.getJobId());
            CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(scheduleJob.getCronExpression())
                    .withMisfireHandlingInstructionDoNothing();
            CronTrigger trigger = getCronTrigger(scheduler, scheduleJob.getJobId());
            trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
            trigger.getJobDataMap().put(ScheduleJobBean.JOB_PARAM_KEY, scheduleJob);
            scheduler.rescheduleJob(triggerKey, trigger);
            // 如果该定时器处于暂停状态
            if(scheduleJob.getStatus() == 1){
                pauseJob(scheduler, scheduleJob.getJobId());
            }
        } catch (SchedulerException e) {
            throw new RuntimeException("updateJob Fail",e) ;
        }
    }
    /**
     * 停止定时器
     */
    public static void pauseJob (Scheduler scheduler,Long jobId){
        try {
            scheduler.pauseJob(getJobKey(jobId));
        } catch (SchedulerException e){
            throw new RuntimeException("pauseJob Fail",e) ;
        }
    }
    /**
     * 恢复定时器
     */
    public static void resumeJob (Scheduler scheduler,Long jobId){
        try {
            scheduler.resumeJob(getJobKey(jobId));
        } catch (SchedulerException e){
            throw new RuntimeException("resumeJob Fail",e) ;
        }
    }
    /**
     * 删除定时器
     */
    public static void deleteJob (Scheduler scheduler,Long jobId){
        try {
            scheduler.deleteJob(getJobKey(jobId));
        } catch (SchedulerException e){
            throw new RuntimeException("deleteJob Fail",e) ;
        }
    }
    /**
     * 执行定时器
     */
    public static void run (Scheduler scheduler, ScheduleJobBean scheduleJob){
        try {
            JobDataMap dataMap = new JobDataMap() ;
            dataMap.put(ScheduleJobBean.JOB_PARAM_KEY,scheduleJob);
            scheduler.triggerJob(getJobKey(scheduleJob.getJobId()),dataMap);
        } catch (SchedulerException e){
            throw new RuntimeException("run Fail",e) ;
        }
    }
}
```
## 2.4 定时器执行和日志
```java
import com.quart.job.entity.ScheduleJobBean;
import com.quart.job.entity.ScheduleJobLogBean;
import com.quart.job.service.ScheduleJobLogService;
import org.quartz.JobExecutionContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;
import java.lang.reflect.Method;
import java.util.Date;
/**
 * 定时器执行日志记录
 */
public class TaskJobLog extends QuartzJobBean {

    private static final Logger LOG = LoggerFactory.getLogger(TaskJobLog.class) ;

    @Override
    protected void executeInternal(JobExecutionContext context) {
        ScheduleJobBean jobBean = (ScheduleJobBean)context.getMergedJobDataMap().get(ScheduleJobBean.JOB_PARAM_KEY) ;
        ScheduleJobLogService scheduleJobLogService = (ScheduleJobLogService)SpringContextUtil.getBean("scheduleJobLogService") ;
        // 定时器日志记录
        ScheduleJobLogBean logBean = new ScheduleJobLogBean () ;
        logBean.setJobId(jobBean.getJobId());
        logBean.setBeanName(jobBean.getBeanName());
        logBean.setParams(jobBean.getParams());
        logBean.setCreateTime(new Date());
        long beginTime = System.currentTimeMillis() ;
        try {
            // 加载并执行定时器的 run 方法
            Object target = SpringContextUtil.getBean(jobBean.getBeanName());
            Method method = target.getClass().getDeclaredMethod("run", String.class);
            method.invoke(target, jobBean.getParams());
            long executeTime = System.currentTimeMillis() - beginTime;
            logBean.setTimes((int)executeTime);
            logBean.setStatus(0);
            LOG.info("定时器 === >> "+jobBean.getJobId()+"执行成功,耗时 === >> " + executeTime);
        } catch (Exception e){
            // 异常信息
            long executeTime = System.currentTimeMillis() - beginTime;
            logBean.setTimes((int)executeTime);
            logBean.setStatus(1);
            logBean.setError(e.getMessage());
        } finally {
            scheduleJobLogService.insert(logBean) ;
        }
    }
}
```

# 3、定时器服务封装
## 3.1 定时器初始化
```java
@Service
public class ScheduleJobServiceImpl implements ScheduleJobService {

    @Resource
    private Scheduler scheduler ;
    @Resource
    private ScheduleJobMapper scheduleJobMapper ;

    /**
     * 定时器初始化
     */
    @PostConstruct
    public void init (){
        ScheduleJobExample example = new ScheduleJobExample() ;
        List<ScheduleJobBean> scheduleJobBeanList = scheduleJobMapper.selectByExample(example) ;
        for (ScheduleJobBean scheduleJobBean : scheduleJobBeanList) {
            CronTrigger cronTrigger = ScheduleUtil.getCronTrigger(scheduler,scheduleJobBean.getJobId()) ;
            if (cronTrigger == null){
                ScheduleUtil.createJob(scheduler,scheduleJobBean);
            } else {
                ScheduleUtil.updateJob(scheduler,scheduleJobBean);
            }
        }
    }
}
```
## 3.2 添加定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public int insert(ScheduleJobBean record) {
    ScheduleUtil.createJob(scheduler,record);
    return scheduleJobMapper.insert(record);
}
```

## 3.3 立即执行一次定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public void run(Long jobId) {
    ScheduleJobBean scheduleJobBean = scheduleJobMapper.selectByPrimaryKey(jobId) ;
    ScheduleUtil.run(scheduler,scheduleJobBean);
}
```

## 3.4 更新定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public int updateByPrimaryKeySelective(ScheduleJobBean record) {
    ScheduleUtil.updateJob(scheduler,record);
    return scheduleJobMapper.updateByPrimaryKeySelective(record);
}
```
## 3.5 停止定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public void pauseJob(Long jobId) {
    ScheduleJobBean scheduleJobBean = scheduleJobMapper.selectByPrimaryKey(jobId) ;
    ScheduleUtil.pauseJob(scheduler,jobId);
    scheduleJobBean.setStatus(1);
    scheduleJobMapper.updateByPrimaryKeySelective(scheduleJobBean) ;
}
```
## 3.6 恢复定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public void resumeJob(Long jobId) {
    ScheduleJobBean scheduleJobBean = scheduleJobMapper.selectByPrimaryKey(jobId) ;
    ScheduleUtil.resumeJob(scheduler,jobId);
    scheduleJobBean.setStatus(0);
    scheduleJobMapper.updateByPrimaryKeySelective(scheduleJobBean) ;
}
```
## 3.7 删除定时器
```java
@Override
@Transactional(rollbackFor = Exception.class)
public void delete(Long jobId) {
    ScheduleUtil.deleteJob(scheduler, jobId);
    scheduleJobMapper.deleteByPrimaryKey(jobId) ;
}
```

# 4、配置一个测试的定时器
## 4.1 定时接口封装
```java
public interface TaskService {
    void run(String params);
}
```
## 4.2 测试定时器
```java
@Component("getTimeTask")
public class GetTimeTask implements TaskService {
    private static final Logger LOG = LoggerFactory.getLogger(GetTimeTask.class.getName()) ;
    private static final SimpleDateFormat format =
            new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") ;
    @Override
    public void run(String params) {
        LOG.info("Params === >> " + params);
        LOG.info("当前时间::::"+format.format(new Date()));
    }
}
```