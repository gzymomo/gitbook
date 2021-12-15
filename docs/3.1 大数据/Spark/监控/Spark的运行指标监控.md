[TOC]

spark的运行指标监控：https://www.e-learn.cn/en/share/3399060
spark的运行指标监控：https://www.cnblogs.com/hejunhong/p/12318921.html

![](https://www.showdoc.cc/server/api/common/visitfile/sign/09f435c1354e947420a6f87f46c1808e?showdoc=.jpg)

# 一、Spark指标说明
```
".driver.BlockManager.disk.diskSpaceUsed_MB")//使用的磁盘空间
".driver.BlockManager.memory.maxMem_MB") //使用的最大内存
".driver.BlockManager.memory.memUsed_MB")//内存使用情况
".driver.BlockManager.memory.remainingMem_MB") //闲置内存
//#####################stage###################################
".driver.DAGScheduler.job.activeJobs")//当前正在运行的job
".driver.DAGScheduler.job.allJobs")//总job数
".driver.DAGScheduler.stage.failedStages")//失败的stage数量
".driver.DAGScheduler.stage.runningStages")//正在运行的stage
".driver.DAGScheduler.stage.waitingStages")//等待运行的stage
//#####################StreamingMetrics###################################
".driver.query.StreamingMetrics.streaming.lastCompletedBatch_processingDelay")// 最近批次执行的延迟时间
".driver.query.StreamingMetrics.streaming.lastCompletedBatch_processingEndTime")//最近批次执行结束时间（毫秒为单位）
".driver.query.StreamingMetrics.streaming.lastCompletedBatch_processingStartTime")//最近批次开始执行时间
//执行时间
val lastCompletedBatch_processingTime = (lastCompletedBatch_processingEndTime - lastCompletedBatch_processingStartTime)
".driver.query.StreamingMetrics.streaming.lastReceivedBatch_records")//最近批次接收的数量
".driver.query.StreamingMetrics.streaming.runningBatches")//正在运行的批次
".driver.query.StreamingMetrics.streaming.totalCompletedBatches")//完成的数据量
".driver.query.StreamingMetrics.streaming.totalProcessedRecords")//总处理条数
".driver.query.StreamingMetrics.streaming.totalReceivedRecords")//总接收条数
".driver.query.StreamingMetrics.streaming.unprocessedBatches")//为处理的批次
".driver.query.StreamingMetrics.streaming.waitingBatches")//处于等待状态的批次
```

# 二、程序task的运行情况做表格，用来对job的监控：
```java
 val fieldMap = scala.collection.mutable.Map(
      //TODO=================表格，做链路统计=================================
      "applicationId" -> monitorIndex._3.toString,
      "endTime" -> new DateTime(monitorIndex._1).toDateTime.toString("yyyy-MM-dd HH:mm:ss"),
      "applicationUniqueName" -> monitorIndex._2.toString,
      "sourceCount" -> monitorIndex._4.toString, //当前处理了多条数据
      "costTime" -> monitorIndex._5.toString,//花费的时间
      "countPerMillis" -> monitorIndex._6.toString,
      "serversCountMap" -> serversCountMap ,
      //TODO=================做饼图，用来对内存和磁盘的监控=================================
      "diskSpaceUsed_MB" -> diskSpaceUsed_MB ,//磁盘使用空间
      "maxMem_MB" -> maxMem_MB ,//最大内存
      "memUsed_MB" -> memUsed_MB ,//使用的内寸
      "remainingMem_MB" -> remainingMem_MB ,//闲置内存
      //TODO =================做表格，用来对job的监控=================================
      "activeJobs" -> activeJobs ,//当前正在运行的job
      "allJobs" -> allJobs ,//所有的job
      "failedStages" -> failedStages ,//是否出现错误的stage
      "runningStages" -> runningStages ,//正在运行的 stage
      "waitingStages" -> waitingStages ,//处于等待运行的stage
      "lastCompletedBatch_processingDelay" -> lastCompletedBatch_processingDelay ,//最近批次的延迟啥时间
      "lastCompletedBatch_processingTime" -> lastCompletedBatch_processingTime ,//正在处理的 批次的时间
      "lastReceivedBatch_records" -> lastReceivedBatch_records ,//最近接收到的数据量
      "runningBatches" -> runningBatches ,//正在运行的批次
      "totalCompletedBatches" -> totalCompletedBatches ,//所有完成批次
      "totalProcessedRecords" -> totalProcessedRecords ,//总处理数据条数
      "totalReceivedRecords" -> totalReceivedRecords ,//总接收数据
      "unprocessedBatches" -> unprocessedBatches ,//未处理的批次
      "waitingBatches" -> waitingBatches//处于等待的批次
    )
```
# 三、Prometheus收集到的指标
## 3.1 DAG_scheduler
```yml
DAG_scheduler{application="ZyStreamingInput",qty="activeJobs",type="job"} 0
DAG_scheduler{application="ZyStreamingInput",qty="allJobs",type="job"} 4560
DAG_scheduler{application="ZyStreamingInput",qty="count",type="messageProcessingTime"} 41915
DAG_scheduler{application="ZyStreamingInput",qty="failedStages",type="stage"} 0
DAG_scheduler{application="ZyStreamingInput",qty="m15_rate",type="messageProcessingTime"} 2.31
DAG_scheduler{application="ZyStreamingInput",qty="m1_rate",type="messageProcessingTime"} 2.22
DAG_scheduler{application="ZyStreamingInput",qty="m5_rate",type="messageProcessingTime"} 2.37
DAG_scheduler{application="ZyStreamingInput",qty="max",type="messageProcessingTime"} 51.19
DAG_scheduler{application="ZyStreamingInput",qty="mean",type="messageProcessingTime"} 3.28
DAG_scheduler{application="ZyStreamingInput",qty="mean_rate",type="messageProcessingTime"} 2.35
DAG_scheduler{application="ZyStreamingInput",qty="min",type="messageProcessingTime"} 0
DAG_scheduler{application="ZyStreamingInput",qty="p50",type="messageProcessingTime"} 0.18
DAG_scheduler{application="ZyStreamingInput",qty="p75",type="messageProcessingTime"} 0.55
DAG_scheduler{application="ZyStreamingInput",qty="p95",type="messageProcessingTime"} 26.45
DAG_scheduler{application="ZyStreamingInput",qty="p98",type="messageProcessingTime"} 36.56
DAG_scheduler{application="ZyStreamingInput",qty="p99",type="messageProcessingTime"} 40.22
DAG_scheduler{application="ZyStreamingInput",qty="p999",type="messageProcessingTime"} 43.33
DAG_scheduler{application="ZyStreamingInput",qty="runningStages",type="stage"} 0
DAG_scheduler{application="ZyStreamingInput",qty="stddev",type="messageProcessingTime"} 8.31
DAG_scheduler{application="ZyStreamingInput",qty="waitingStages",type="stage"} 0
```
## 3.2 block_manager
```yml
block_manager{application="ZyStreamingInput",executor_id="driver",qty="diskSpaceUsed_MB",type="disk"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="maxMem_MB",type="memory"} 2557
block_manager{application="ZyStreamingInput",executor_id="driver",qty="maxOffHeapMem_MB",type="memory"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="maxOnHeapMem_MB",type="memory"} 2557
block_manager{application="ZyStreamingInput",executor_id="driver",qty="memUsed_MB",type="memory"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="offHeapMemUsed_MB",type="memory"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="onHeapMemUsed_MB",type="memory"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="remainingMem_MB",type="memory"} 2557
block_manager{application="ZyStreamingInput",executor_id="driver",qty="remainingOffHeapMem_MB",type="memory"} 0
block_manager{application="ZyStreamingInput",executor_id="driver",qty="remainingOnHeapMem_MB",type="memory"} 2557
```
## 3.3 jvm_memory_pools
```yml
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Code-Cache",qty="committed"} 6.4487424e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Code-Cache",qty="init"} 2.555904e+06
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Code-Cache",qty="max"} 2.5165824e+08
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Code-Cache",qty="usage"} 0.25
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Code-Cache",qty="used"} 6.2888768e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Compressed-Class-Space",qty="committed"} 1.31072e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Compressed-Class-Space",qty="init"} 0
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Compressed-Class-Space",qty="max"} 1.073741824e+09
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Compressed-Class-Space",qty="usage"} 0.01
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Compressed-Class-Space",qty="used"} 1.2711176e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Metaspace",qty="committed"} 9.6690176e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Metaspace",qty="init"} 0
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Metaspace",qty="max"} -1
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Metaspace",qty="usage"} 0.98
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="Metaspace",qty="used"} 9.5155936e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Eden-Space",qty="committed"} 1.92413696e+08
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Eden-Space",qty="init"} 6.6060288e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Eden-Space",qty="max"} 1.059586048e+09
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Eden-Space",qty="usage"} 0.05
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Eden-Space",qty="used"} 5.7294024e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Old-Gen",qty="committed"} 4.72383488e+08
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Old-Gen",qty="init"} 1.75112192e+08
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Old-Gen",qty="max"} 2.147483648e+09
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Old-Gen",qty="usage"} 0.14
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Old-Gen",qty="used"} 2.90347768e+08
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Survivor-Space",qty="committed"} 5.24288e+06
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Survivor-Space",qty="init"} 1.048576e+07
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Survivor-Space",qty="max"} 5.24288e+06
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Survivor-Space",qty="usage"} 1
jvm_memory_pools{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Survivor-Space",qty="used"} 5.224408e+06
```
## 3.4 jvm_memory_usage
```yml
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="PS-MarkSweep",qty="count"} 12
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="PS-MarkSweep",qty="time"} 5541
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Scavenge",qty="count"} 750
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="PS-Scavenge",qty="time"} 16387
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="direct",qty="capacity"} 272906
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="direct",qty="count"} 27
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="direct",qty="used"} 272907
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="heap",qty="committed"} 6.70040064e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="heap",qty="init"} 2.62144e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="heap",qty="max"} 2.863661056e+09
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="heap",qty="usage"} 0.12
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="heap",qty="used"} 3.52707968e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="mapped",qty="capacity"} 0
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="mapped",qty="count"} 0
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="mapped",qty="used"} 0
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="non-heap",qty="committed"} 1.742848e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="non-heap",qty="init"} 2.555904e+06
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="non-heap",qty="max"} -1
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="non-heap",qty="usage"} -1.7075588e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="non-heap",qty="used"} 1.7075588e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="total",qty="committed"} 8.44324864e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="total",qty="init"} 2.64699904e+08
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="total",qty="max"} 2.863661055e+09
jvm_memory_usage{application="ZyStreamingInput",executor_id="driver",mem_type="total",qty="used"} 5.2362208e+08
```
## 3.5 task_info
```yml
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_processingDelay",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 53
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_processingEndTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190061e+12
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_processingStartTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190008e+12
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_schedulingDelay",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_submissionTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190007e+12
task_info{application="ZyStreamingInput",qty="lastCompletedBatch_totalDelay",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 54
task_info{application="ZyStreamingInput",qty="lastReceivedBatch_processingEndTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190061e+12
task_info{application="ZyStreamingInput",qty="lastReceivedBatch_processingStartTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190008e+12
task_info{application="ZyStreamingInput",qty="lastReceivedBatch_records",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 0
task_info{application="ZyStreamingInput",qty="lastReceivedBatch_submissionTime",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1.585119190007e+12
task_info{application="ZyStreamingInput",qty="receivers",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 0
task_info{application="ZyStreamingInput",qty="retainedCompletedBatches",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1000
task_info{application="ZyStreamingInput",qty="runningBatches",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 0
task_info{application="ZyStreamingInput",qty="size",task="LiveListenerBus",type1="queue",type2="appStatus"} 0
task_info{application="ZyStreamingInput",qty="size",task="LiveListenerBus",type1="queue",type2="eventLog"} 0
task_info{application="ZyStreamingInput",qty="size",task="LiveListenerBus",type1="queue",type2="executorManagement"} 0
task_info{application="ZyStreamingInput",qty="totalCompletedBatches",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 1784
task_info{application="ZyStreamingInput",qty="totalProcessedRecords",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 786
task_info{application="ZyStreamingInput",qty="totalReceivedRecords",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 786
task_info{application="ZyStreamingInput",qty="unprocessedBatches",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 0
task_info{application="ZyStreamingInput",qty="waitingBatches",task="ZyStreamingInput",type1="StreamingMetrics",type2="streaming"} 0
```