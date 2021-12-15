- [Flowable引擎的使用](https://www.tangyuecan.com/2020/10/20/flowable%e5%bc%95%e6%93%8e%e7%9a%84%e4%bd%bf%e7%94%a8/)



## 技术选型

工作流引擎目前开源上可以选的就只有`activiti`、`flowable`、`camunda`三种，当然除了`activiti`它们都有商用版本，而且`flowable`与`camunda`都是从`activiti`之中分裂出来的子项目，也是NB。但是目前从技术选型角度考虑，我们公司后续发展或者项目情况来看`BPMN 2.0`标准已经完全足够使用了，更加复杂的流程控制确实没有什么必要性对我们而言也不是重点所以。比如像`CMMN`、`DMN`这些动态的流程控制（都TM不叫工作流了）太高档了我们这些项目确实用不上。

商用上面临和ETL一样的情况，几家大型的软件技术公司都形成了自身的完整Pass平台，整体使用确实很NB但是单独拿出来就比较难受了，目前没有考虑的必要。

这样一来选型就非常明确`camunda`这种东西重点在于对各种`BPM`的整合，搞不定`BPMN`，`activiti`后续版本重心全部去搞`CMMN`这种流程控制了，并且了由于项目最主要的开发人员都跑了基本现在是一个烂尾项目（所以现在都还在用`activiti 5`这个版本），那么选型就只能是`flowable`了。但是基本上`flowable`在6之后的版本维护情况比较糟糕（全部都去搞商用版了）所以我们定的版本是`6.5.0`

## 项目集成

集成上`flowable`可以说继承所有`activiti`的优点加几个包就可以了。

```xml
<dependency>
  <groupId>org.flowable</groupId>
  <artifactId>flowable-spring-boot-starter</artifactId>
  <version>6.5.0</version>
</dependency>
```

还有一种集成方式是独立部署然后通过统一授权进行流程的动态发布、加载与管理，使用HTTP接口与各个项目进行沟通。这种方式其实也可以但是使用上比较反直觉同时由于我们项目部署上可能是内网部署（中石油、华虹等）也可能是封闭服务器部署（别人的服务器环境）还有就是我们自己的服务器（rancher）所以说独立部署工作流然后通过接口进行通信有点过于重量级了不是很灵活。

使用上无论是数据库还是上层的接口封装和`activiti`都比较像（其实两者在`BPMN`部分真的差不多），我根据网上的代码自己封装了几个接口个人认为基本上满足使用了，如果需要直接加就是了。接口定义如下：

```java
public interface FlowableService {
   /**
    * 启动流程
    *
    * @param processKey  流程定义key(流程图ID)
    * @param businessKey 业务key(可以为空)
    * @param map         参数键值对
    * @return 流程实例ID
    */
   String start(String processKey, String businessKey, Map<String, Object> map);

   /**
    * 停止流程
    *
    * @param processInstanceId 流程实例ID
    * @param reason            终止理由
    */
   void stop(String processInstanceId, String reason);

   /**
    * 获取指定用户的任务列表（创建时间倒序）
    *
    * @param userId 用户ID
    * @return 任务列表
    */
   List<TaskObject> getListByUserId(String userId);

   /**
    * 获取指定用户组的任务列表
    *
    * @param group 用户组
    * @return 任务列表
    */
   List<TaskObject> getListByGroup(String group);

   /**
    * 完成指定任务
    *
    * @param taskId 任务ID
    * @param map    变量键值对
    * @throws Exception 任务不存在
    */
   void complete(String taskId, Map<String, Object> map) throws Exception;

   /**
    * 获取指定任务列表中的特定任务
    *
    * @param list        任务列表
    * @param businessKey 业务key
    * @return 任务
    */
   Task getOneByBusinessKey(List<Task> list, String businessKey);

   /**
    * 创建流程并完成第一个任务
    *
    * @param processKey  流程定义key(流程图ID)
    * @param businessKey 业务key
    * @param map         变量键值对
    */
   void startAndComplete(String processKey, String businessKey, Map<String, Object> map);

   /**
    * 退回到指定任务节点
    *
    * @param currentTaskId 当前任务ID
    * @param targetTaskKey 目标任务节点key
    *
    * @throws Exception 当前任务节点不存在
    */
   void backToStep(String currentTaskId, String targetTaskKey) throws Exception;

   /**
    * 返回尚未结束的流程实例
    *
    * @param processKey 流程的名称（传入空返回所有）
    * @return 流程列表
    */
   List<ProcessInstanceObject> getProcessList(String processKey);
   /**
    * 返回所有的流程实例
    *
    * @param processKey 流程的名称（传入空返回所有）
    * @return 流程列表
    */
   List<ProcessInstanceObject> getHistoryProcessList(String processKey);
   /**
    * 获取尚未结束的流程图
    *
    * @param outputStream 流程图的输出流（传入空返回所有）
    * @param processId 流程的ID
    * @throws IOException 流无法读取或写入
    */
   void getProcessChart(OutputStream outputStream,String processId) throws IOException;
}
```

实现代码如下：

```java
@Service
public class FlowableServiceImpl implements FlowableService {

   private static final Logger logger = LoggerFactory.getLogger("flowable-service-log");

   @Resource
   RuntimeService runtimeService;
   @Resource
   TaskService taskService;
   @Resource
   HistoryService historyService;
   @Resource
   ProcessEngine processEngine;
   @Resource
   RepositoryService repositoryService;

   @Override
   public String start(String processKey, String businessKey, Map<String, Object> map) {
      ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processKey, businessKey, map);
      return processInstance.getId();
   }

   @Override
   public void stop(String processInstanceId, String reason) {
      runtimeService.deleteProcessInstance(processInstanceId, reason);
   }

   @Override
   public List<TaskObject> getListByUserId(String userId) {
      List<Task> tasks = taskService.createTaskQuery().taskAssignee(userId).orderByTaskCreateTime().desc().list();
      return taskToTaskObject(tasks);
   }

   @Override
   public List<TaskObject> getListByGroup(String group) {
      List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup(group).orderByTaskCreateTime().desc().list();
      return taskToTaskObject(tasks);
   }

   private List<TaskObject> taskToTaskObject(List<Task> tasks) {
      List<TaskObject> taskObjects = new ArrayList<>();
      for (Task task : tasks) {
         taskObjects.add(TaskObject.fromTask(task));
      }
      return taskObjects;
   }

   @Override
   public void complete(String taskId, Map<String, Object> map) throws Exception {
      Task task = taskService.createTaskQuery().taskId(taskId).singleResult();
      if (task == null) {
         logger.error(taskId + "：指定的任务不存在");
         throw new Exception("任务不存在");
      }
      taskService.complete(taskId, map);
   }

   @Override
   public Task getOneByBusinessKey(List<Task> list, String businessKey) {
      Task task = null;
      for (Task t : list) {
         // 通过任务对象获取流程实例
         ProcessInstance pi = runtimeService.createProcessInstanceQuery().processInstanceId(t.getProcessInstanceId()).singleResult();
         if (businessKey.equals(pi.getBusinessKey())) {
            task = t;
         }
      }
      return task;
   }

   @Override
   public void startAndComplete(String processKey, String businessKey, Map<String, Object> map) {
      String processInstanceId = start(processKey, businessKey, map);
      Task task = processEngine.getTaskService().createTaskQuery().processInstanceId(processInstanceId).singleResult();
      taskService.complete(task.getId(), map);
   }

   @Override
   public void backToStep(String currentTaskId, String targetTaskKey) throws Exception {

      Task currentTask = taskService.createTaskQuery().taskId(currentTaskId).singleResult();
      if (currentTask == null) {
         logger.error(currentTaskId + "：指定的任务不存在");
         throw new Exception("当前任务节点不存在");
      }
      List<String> currentTaskKeys = new ArrayList<>();
      currentTaskKeys.add(currentTask.getTaskDefinitionKey());
      runtimeService.createChangeActivityStateBuilder().processInstanceId(currentTask.getProcessInstanceId()).moveActivityIdsToSingleActivityId(currentTaskKeys, targetTaskKey);
   }

   @Override
   public List<ProcessInstanceObject> getProcessList(String processKey) {
      if (processKey == null) {
         return processToProcessObject(runtimeService.createProcessInstanceQuery().orderByStartTime().desc().list());
      }
      return processToProcessObject(runtimeService.createProcessInstanceQuery().processDefinitionKey(processKey).orderByStartTime().desc().list());
   }

   @Override
   public List<ProcessInstanceObject> getHistoryProcessList(String processKey) {
      if (processKey == null) {
         return historyProcessToProcessObject(historyService.createHistoricProcessInstanceQuery().orderByProcessInstanceStartTime().desc().list());
      }
      return historyProcessToProcessObject(historyService.createHistoricProcessInstanceQuery().orderByProcessInstanceStartTime().desc().processDefinitionKey(processKey).list());
   }

   private List<ProcessInstanceObject> processToProcessObject(List<ProcessInstance> instances) {
      List<ProcessInstanceObject> instanceObjects = new ArrayList<>();
      for (ProcessInstance instance : instances) {
         instanceObjects.add(ProcessInstanceObject.fromProcessInstance(instance));
      }
      return instanceObjects;
   }

   private List<ProcessInstanceObject> historyProcessToProcessObject(List<HistoricProcessInstance> instances) {
      List<ProcessInstanceObject> instanceObjects = new ArrayList<>();
      for (HistoricProcessInstance instance : instances) {
         instanceObjects.add(ProcessInstanceObject.fromHistoryProcessInstance(instance));
      }
      return instanceObjects;
   }

   @Override
   public void getProcessChart(OutputStream out, String processId) throws IOException {
      ProcessInstance pi = runtimeService.createProcessInstanceQuery().processInstanceId(processId).singleResult();

      //流程走完的不显示图
      if (pi == null) {
         return;
      }
      Task task = taskService.createTaskQuery().processInstanceId(pi.getId()).singleResult();
      //使用流程实例ID，查询正在执行的执行对象表，返回流程实例对象
      String InstanceId = task.getProcessInstanceId();
      List<Execution> executions = runtimeService
         .createExecutionQuery()
         .processInstanceId(InstanceId)
         .list();

      //得到正在执行的Activity的Id
      List<String> activityIds = new ArrayList<>();
      List<String> flows = new ArrayList<>();
      for (Execution exe : executions) {
         List<String> ids = runtimeService.getActiveActivityIds(exe.getId());
         activityIds.addAll(ids);
      }

      //获取流程图
      BpmnModel bpmnModel = repositoryService.getBpmnModel(pi.getProcessDefinitionId());
      ProcessEngineConfiguration engconf = processEngine.getProcessEngineConfiguration();
      ProcessDiagramGenerator diagramGenerator = engconf.getProcessDiagramGenerator();
      InputStream in = diagramGenerator.generateDiagram(bpmnModel, "png",
         activityIds, flows, "宋体", "宋体", "宋体", null, 1.0, true);
      byte[] buf = new byte[1024];
      int legth = 0;
      try {
         while ((legth = in.read(buf)) != -1) {
            out.write(buf, 0, legth);
         }
      } finally {
         if (in != null) {
            in.close();
         }
         if (out != null) {
            out.close();
         }
      }
   }
}
```

这些接口基本上足够我们使用了，一般的流程都是涉及到具体的个人（用户ID）、角色（用户组ID）上我们可以通过接口进行定义；同时流程之中需要参数通过`map`进行传递其他就是开始流程、结束流程、跳转任务这种`BPMN`基础节点。实际使用上还是比较明确也很直觉的。

还剩一个东西就是定义的`BPMN`流程的`xml`文件，这个东西可以直接放在`resources/processes`里面，只要不冲突和重复就不会有问题。

## 流程编辑

编辑流程可以说太简单了，我们没有独立部署所以编辑器就不会放置在一个统一地方。但是与项目集成的好处是我们一旦编辑完成之后只需要`BPMN`文件便可以运行。所以编辑器每人自己搞一个就可以了，这里我提供一个官方镜像，基本上集成了所有的`flowable`组件：[flowable/all-in-one:6.5.0](https://hub.docker.com/r/flowable/all-in-one)运行说明如下：

```
docker run -p 8080:8080 flowable/all-in-one:6.5.0
```

启动之访问http://localhost:8080/flowable-modeler即可，默认的用户名是：admin，密码：test，进去了之后就是编辑页面了编辑完成之后导出为xml文件就可以放在项目里面使用了，算是比较简单。

还有一个[中文文档](http://www.shareniu.com/flowable6.5_zh_document/bpm/index.html)上手使用是足够了，不过我水平也比较有限不可能全部搞得明明白白，大部分情况问题都可以在论坛、文档这些地方找到。不过flowable-6.5.0这个版本还是非常优秀的，无论稳定性还是技术市场保有量都不错。