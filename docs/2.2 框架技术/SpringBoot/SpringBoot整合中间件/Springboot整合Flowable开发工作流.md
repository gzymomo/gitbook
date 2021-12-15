官方地址：`https://www.flowable.org/docs/userguide/index.html`，分享的只是简单应用，深入研究还得看官方文档。

## Flowable 核心依赖

```xml
<!--flowable工作流依赖-->
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-spring-boot-starter</artifactId>
    <version>6.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 流程设计

工作流开发的核心是任务流程的设计，`Flowable` 官方建议采用业界标准`BPMN2.0`的 `XML`来描述需要定义的工作流。

我们需要在 `resource` 目录下创建 `processes`路径，存放相关的 `XML`流程配置文件。`Flowable` 框架会默认加载此目录下的工作流文件并解析 `XML`，并将解析后的流程配置信息持久化到数据库。



![img](https://img-blog.csdnimg.cn/20200827111825323.png)



`Flowable` 是依赖于数据库的，但它并不需要我们手动的创建表，而是在程序第一次启动时，自动的向`MySQL` 中创建它所需要的一系列表。

```yaml
spring:
  datasource:
    url: jdbc:mysql://47.93.6.5:3306/order?serverTimezone=UTC
    username: root
    password: 123455
```



![img](https://img-blog.csdnimg.cn/20200826152345138.png)



看到项目启动成功一共生成了60个表，数量还是比较多的，建议使用专门的数据库存在这些工作流表。



![img](https://img-blog.csdnimg.cn/20200826152402283.png)



举个栗子：假如一个请假流程，需要经理审核通过，请假才能生效，如果他驳回流程结束。



![img](https://img-blog.csdnimg.cn/20200827164539573.png#pic_center)



接下来我们用 `XML` 翻译下上边的请假流程图，整体非常简单只要够细心就行了，一起看看每个标签都是什么含义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:flowable="http://flowable.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
             xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
             typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath"
             targetNamespace="http://www.flowable.org/processdef">
    <process id="Leave" name="LeaveProcess" isExecutable="true">
        <userTask id="leaveTask" name="请假" flowable:assignee="${leaveTask}"/>
        <userTask id="managerTask" name="经理审核"/>
        <exclusiveGateway id="managerJudgeTask"/>
        <endEvent id="endLeave" name="结束"/>
        <startEvent id="startLeave" name="开始"/>
        <sequenceFlow id="modeFlow" sourceRef="leaveTask" targetRef="managerTask"/>
        <sequenceFlow id="flowStart" sourceRef="startLeave" targetRef="leaveTask"/>
        <sequenceFlow id="jugdeFlow" sourceRef="managerTask" targetRef="managerJudgeTask"/>
        <endEvent id="endLeave2"/>
        <sequenceFlow id="flowEnd" name="通过" sourceRef="managerJudgeTask" targetRef="endLeave">
            <conditionExpression xsi:type="tFormalExpression">
                <![CDATA[${checkResult=='通过'}]]>
            </conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="rejectFlow" name="驳回" sourceRef="managerJudgeTask"
                      targetRef="endLeave2">
            <conditionExpression xsi:type="tFormalExpression">
                <![CDATA[${checkResult=='驳回'}]]>
            </conditionExpression>
        </sequenceFlow>
    </process>
    <bpmndi:BPMNDiagram id="BPMNDiagram_process">
        <bpmndi:BPMNPlane bpmnElement="Leave" id="BPMNPlane_process">
            <bpmndi:BPMNShape bpmnElement="leaveTask" id="BPMNShape_leaveTask">
                <omgdc:Bounds height="79.99999999999999" width="100.0" x="304.60807973558974" y="122.00000000000001"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="managerTask" id="BPMNShape_managerTask">
                <omgdc:Bounds height="80.0" width="100.0" x="465.0" y="122.0"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="managerJudgeTask" id="BPMNShape_managerJudgeTask">
                <omgdc:Bounds height="40.0" width="40.0" x="611.5" y="142.0"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="endLeave" id="BPMNShape_endLeave">
                <omgdc:Bounds height="28.0" width="28.0" x="696.5" y="148.0"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="startLeave" id="BPMNShape_startLeave">
                <omgdc:Bounds height="30.0" width="30.0" x="213.2256558149128" y="147.0"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNShape bpmnElement="endLeave2"
                              id="BPMNShape_endLeave2">
                <omgdc:Bounds height="28.0" width="28.0" x="617.5" y="73.32098285753572"/>
            </bpmndi:BPMNShape>
            <bpmndi:BPMNEdge bpmnElement="flowEnd" id="BPMNEdge_flowEnd">
                <omgdi:waypoint x="651.1217948717949" y="162.37820512820514"/>
                <omgdi:waypoint x="696.5002839785394" y="162.0891701657418"/>
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="rejectFlow" id="BPMNEdge_rejectFlow">
                <omgdi:waypoint x="631.866093577786" y="142.36609357778607" />
                <omgdi:waypoint x="631.5931090276993" y="101.32067323657485" />
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="modeFlow" id="BPMNEdge_modeFlow">
                <omgdi:waypoint x="404.60807973558974" y="162.0" />
                <omgdi:waypoint x="465.0" y="162.0" />
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="flowStart" id="BPMNEdge_flowStart">
                <omgdi:waypoint x="243.2256558149128" y="162.0" />
                <omgdi:waypoint x="304.60807973558974" y="162.0" />
            </bpmndi:BPMNEdge>
            <bpmndi:BPMNEdge bpmnElement="jugdeFlow" id="BPMNEdge_jugdeFlow">
                <omgdi:waypoint x="565.0" y="162.21367521367523" />
                <omgdi:waypoint x="611.9141630901288" y="162.41416309012877" />
            </bpmndi:BPMNEdge>
        </bpmndi:BPMNPlane>
    </bpmndi:BPMNDiagram>
</definitions>
```

其实就是把流程图的各种线条逻辑，用不同的`XML`标签描绘出来了。

`<process>`  ： 表示一个完整的工作流

`<documentation>`  ： 对工作流的描述

`<startEvent>`  ： 工作流中起点位置（开始）

`<endEvent >`  ： 工作流中结束位置（结束）

`<userTask>`  ： 代表一个任务审核节点（组长、经理等角色）

`<exclusiveGateway>`  ： 逻辑判断节点，相当于流程图中的菱形框

`<sequenceFlow>` ：链接各个节点的线条，`sourceRef` 属性表示线的起始节点，`targetRef` 属性表示线指向的节点。

上边这一大坨`XML`是不是看着超级麻烦，要是有自动生成工具就好了，我发现`IDEA`自带设计工具，但实在是太难用了。

![img](https://img-blog.csdnimg.cn/20200827154808132.png)



作为一个面向百度编程的程序员，别的不行上网找答案的能力还是可以的，既然我都觉得写`XML`麻烦，那么想来官方肯定也想到了，说不定有现成的工具，逛了一圈官网`https://www.flowable.org/downloads.html` ，居然真的找到了。

`github`下载地址：[github.com/flowable/fl…](https://github.com/flowable/flowable-engine/releases/download/flowable-6.4.0/flowable-6.4.0.zip，下载速度那是相当感人，而且这个工具需要自己安装).......



![img](https://img-blog.csdnimg.cn/20200827171441437.png)

又找了个在线编辑的工具： [www.learun.cn:8090/home_online…](http://www.learun.cn:8090/home_online.htm，各种折腾~，设计完流程后，直接复制自动生成的代码即可。)

![img](https://img-blog.csdnimg.cn/20200827171617712.png)



## 流程审批

流程设计完后剩下的就是对工作流的审批和生成流程图。

首先启动一个请假的流程，以员工ID `staffId` 作为唯一标识，`XML`文件中会接收变量 `leaveTask`，`Flowable`内部会进行数据库持久化，并返回一个流程Id `processId` ，用它可以查询工作流的整体情况，任务Id `task`为员工具体的请假任务。

**注意**：一个请假流程 `processId`中可以包含多个请假任务 `taskId`。

```java
/**
     * @author xiaofu
     * @description 启动流程
     * @date 2020/8/26 17:36
     */
    @RequestMapping(value = "startLeaveProcess")
    @ResponseBody
    public String startLeaveProcess(String staffId) {
        HashMap<String, Object> map = new HashMap<>();
        map.put("leaveTask", staffId);
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("Leave", map);
        StringBuilder sb = new StringBuilder();
        sb.append("创建请假流程 processId：" + processInstance.getId());
        List<Task> tasks = taskService.createTaskQuery().taskAssignee(staffId).orderByTaskCreateTime().desc().list();
        for (Task task : tasks) {
            sb.append("任务taskId:" + task.getId());
        }
        return sb.toString();
    }
```

用启动流程时返回的 `processId` 看一下一下当前的流程图

```java
http://localhost:4000/leave/createProcessDiagramPic?processId=37513
```



![img](https://img-blog.csdnimg.cn/20200827184422961.png#pic_center)



接下来将请假申请进行驳回 ，传入相应的 `taskId` 后执行驳回，再看看整个工作流的效果。

```java
http://localhost:4000/leave/rejectTask?taskId=10086
复制代码
 /**
     * @param taskId
     * @author xinzhifu
     * @description 驳回
     * @date 2020/8/27 14:30
     */
    @ResponseBody
    @RequestMapping(value = "rejectTask")
    public String rejectTask(String taskId) {
        HashMap<String, Object> map = new HashMap<>();
        map.put("checkResult", "驳回");
        taskService.complete(taskId, map);
        return "申请审核驳回~";
    }
```

看到整个请假流程在经理审核这成功阻断了。

```javascript
http://localhost:4000/leave/createProcessDiagramPic?processId=37513
```



![img](https://img-blog.csdnimg.cn/20200827185029709.png#pic_center)



## 总结

开发工作流一般多用在OA系统等传统项目中，我也是第一次尝试做此类功能，收获还是蛮多的，技术栈又压进了一个知识点。今天分享的是个超级简单的`demo`，因为也是刚开始接触，等我用的贼溜的时候，再给小伙伴们做更成熟更深入的分享。

```
demo`的`github` 地址:`https://github.com/chengxy-nds/Springboot-Notebook/tree/master/springboot-work-flowable
```



作者：程序员内点事
链接：https://juejin.im/post/6866965996750667790
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。