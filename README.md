# 一、简介

- Activiti：是领先的轻量级，基于Java的开源BPMN引擎，可满足ERP、OA等项目中的流程自动化需求
- 工作流：是将一组任务组织起来以完成某个经营过程。为了实现某个业务目标，利用计算机在多个参与者之间按某种预定规则自动传递文档、信息或者任务。可以修改配置业务流程，减少代码的修改

# 二、说明

## 1、Activiti的常用概念

- 部署(deployment)：一次资源的部署
- 实例(instance):一个流程每发起一次就是一个实例
- 执行(execution):一个流程实例的执行路径
- 任务(task)：一个人的代办
- 执行变量(variable)：作用域在execution上
- 本地变量(LocalVariable)：作用域在task上
- 临时变量(TransVariable)：不存储数据库，流程进入等待节点变量自动清除

## 2、Activiti生成的数据库表说明

- ACT_RE: RE表示repository（仓库），存储流程静态资源，如流程模型文件等
- ACT_RU: RU表示runtime（运行时），存储activiti运行时产生的数据，比如实例信息，用户任务信息，job信息等。当流程结束后，运行时数据将会被删除，以保证数据量尽可能少，保证性能
- ACT_HI: HI表示history（历史），存储流程历史数据，比如实例信息，变量数据等
- ACT_GE: GE表示general（通用），存储通用数据
- ACT_EVT_LOG：存储事件处理日志

|  表名  | 说明 |
| ------------ | ------------ |
| act_evt_log  | 存储事件处理日志 |
| act_ge_bytearray  | 二进制数据表，存储通用的流程定义和流程部署 |
| act_ge_property  | 属性数据表，存储整个流程引擎级别的数据,初始化表结构时，会默认插入三条记录  |
| act_hi_actinst  | 历史节点表  |
| act_hi_attachment | 历史附件表  |
| act_hi_comment  | 历史意见表 |
| act_hi_detail  | 历史详情表，提供历史变量的查询  |
| act_hi_identitylink  | 历史流程人员表  |
| act_hi_procinst  | 历史流程实例表  |
| act_hi_taskinst  | 历史任务实例表  |
| act_hi_varinst  | 历史变量表  |
| act_procdef_info  | 流程定义信息表  |
| act_re_deployment  | 部署信息表  |
| act_re_model  | 流程设计模型部署表  |
| act_re_procdef  | 流程定义数据表  |
| act_ru_deadletter_job  | 作业死亡信息表，作业失败超过重试次数  |
| act_ru_event_subscr  | 运行时事件表  |
| act_ru_execution  | 运行时流程执行实例表  |
| act_ru_identitylink  | 运行时用户信息表  |
| act_ru_integration  | 运行时积分表  |
| act_ru_job | 运行时作业信息表  |
| act_ru_suspended_job  | 运行时作业暂停表  |
| act_ru_task  | 运行时任务信息表  |
| act_ru_timer_job  | 运行时定时器作业表  |
| act_ru_variable  | 运行时变量信息表  |

## 3、Activiti的接口设计

- Service：负责执行动作，数据的增、删、改
    - Service可以通过注入获取，也可以通过ProcessEngine获取

    - 分为以下几种Service：
        - RepositoryService--对资源进行操作，比如部署文件，附件
        - RuntimeService--对运行时流程进行修改，如增加变量，移除变量等
        - TaskService--对用户任务进行操作和查询
        - HistoryService--对审批历史进行操作
        - DynamicBpmnService--可动态修改流程
        - ManagementService--管理服务

- Query：负责执行查询，数据的查
    - Query可以通过相应的Service获取

# 三、使用

## 1、新建一个Spring Initializr项目

## 2、添加Activiti依赖

```xml
<dependency>
	<groupId>org.activiti</groupId>
	<artifactId>activiti-spring-boot-starter</artifactId>
	<version>7.1.0.M1</version>
</dependency>
```

## 3、修改配置文件

```
#自动部署验证设置:true-开启（默认），false-关闭
spring.activiti.check-process-definitions=true
#表示启动时检查数据库表，不存在则创建
spring.activiti.database-schema-update=true
#表示哪种情况下使用历史表，这里配置为full表示全部记录历史，方便绘制流程图
spring.activiti.history-level=full
#true表示使用历史表
spring.activiti.db-history-used=true
```

## 4、启动类添加如下配置可取消访问验证

```java
@SpringBootApplication(exclude = {org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class,
        org.springframework.boot.actuate.autoconfigure.security.servlet.ManagementWebSecurityAutoConfiguration.class})
```

## 5、创建BPMN文件

可以用IDEA插件actiBPM或者bpmn-js完成制作

## 6、在resources下新建目录processes

Activiti默认装载bpmn流程的目录，将需要启动时初始化的流程文件放入此目录下

## 7、编写测试代码

### 流程定义

- 流程部署
```java
@Test
public void deployProcess(){
  //获取ProcessEngine对象 默认配置文件名称：activiti.cfg.xml 并且configuration的Bean实例ID为processEngineConfiguration
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  //获取RepositoryService对象进行流程部署
  RepositoryService repositoryService = processEngine.getRepositoryService();
  //进行部署,将对应的流程定义文件生成到数据库当中，作为记录进行保存
  Deployment deployment = repositoryService.createDeployment()
          .addClasspathResource("flowchart/activiti.bpmn")            //加载流程文件
          .addClasspathResource("flowchart/activiti.png")
          .name("XX流程")                                         //设置流程名称
          .key("activitiKey")
          .deploy();                                              //部署
  //输出部署信息
  System.out.println("流程名称："+deployment.getName());
  System.out.println("流程ID："+deployment.getId());
  System.out.println("流程Key："+deployment.getKey());
}
```

- 流程查询
```java
@Test
public void queryProcess(){
  //获取ProcessEngine对象 默认配置文件名称：activiti.cfg.xml 并且configuration的Bean实例ID为processEngineConfiguration
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  //获取RepositoryService对象进行流程部署
  RepositoryService repositoryService = processEngine.getRepositoryService();
  //进行部署,将对应的流程定义文件生成到数据库当中，作为记录进行保存
  ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();                                             //部署
  List<ProcessDefinition> activitiKey = processDefinitionQuery.processDefinitionKey("activitiKey")
          .orderByProcessDefinitionVersion().desc().list();
  for (ProcessDefinition processDefinition : activitiKey) {
      System.out.println("------------------");
      //输出部署信息
      System.out.println("流程部署ID：" + processDefinition.getDeploymentId());
      System.out.println("流程名称：" + processDefinition.getName());
      System.out.println("流程ID：" + processDefinition.getId());
      System.out.println("流程Key：" + processDefinition.getKey());
      System.out.println("流程版本：" + processDefinition.getVersion());
  }
}
```

- 流程删除
```java
@Test
public void deleteProcess(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  RepositoryService repositoryService = processEngine.getRepositoryService();
  repositoryService.deleteDeployment("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018", true);
}
```

### 流程实例

- 启动实例
```java
@Test
public void startInstance(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  RuntimeService runtimeService = processEngine.getRuntimeService();
  //创建流程实例
  ProcessInstance holiday = runtimeService.startProcessInstanceByKey("activitiKey");
  //输出实例信息
  System.out.println("流程部署ID："+holiday.getDeploymentId());
  System.out.println("流程实例ID："+holiday.getId());
  System.out.println("活动ID："+holiday.getActivityId());
}
```

- 查询流程实例
```java
@Test
public void queryProcessInstance(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  RuntimeService runtimeService = processEngine.getRuntimeService();
  List<ProcessInstance> activitiKey = runtimeService.createProcessInstanceQuery().processDefinitionKey("activitiKey")
          .list();
  for (ProcessInstance processInstance : activitiKey) {
      System.out.println("-------------------------");
      System.out.println("流程实例ID:" + processInstance.getProcessInstanceId());
      System.out.println("流程定义ID:" + processInstance.getProcessDefinitionId());
      System.out.println("是否执行完成:" + processInstance.isEnded());
      System.out.println("是否暂停:" + processInstance.isSuspended());
  }
}
```

### 用户任务

- 分配任务
```java
@Test
public void assignTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  RuntimeService runtimeService = processEngine.getRuntimeService();
  ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("activitiKey", Collections.singletonMap("assignee", "用户"));
  System.out.println("流程实例ID:" + processInstance.getProcessInstanceId());
}
```

- 设置监听器
```java
public class MyTaskListener implements TaskListener {
  @Override
  public void notify(DelegateTask delegateTask) {
      delegateTask.setAssignee("用户");
  }
}
```

- 查询任务
```java
@Test
public void queryTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  /**
   * 查询代办业务 createTaskQuery查询任务 taskCandidateOrAssigned查询任务执行者 processDefinitionKey：查询流程
   * taskCandidateOrAssigned匹配规则:1.Assigned 2.配置bpmn文件中定义的值
   * taskAssignee匹配规则:1.Assigned
   */
  List<Task> list = taskService.createTaskQuery()
          .processDefinitionKey("activitiKey")
          .includeProcessVariables()
          .taskAssignee("用户")
          .list();
  for (Task task : list) {
      System.out.println("-------------------------");
      System.out.println("流程实例ID:" + task.getProcessInstanceId());
      System.out.println("任务ID:" + task.getId());
      System.out.println("任务负责人:" + task.getAssignee());
      System.out.println("任务名称:" + task.getName());
  }
}
```

- 用户任务处理
```java
@Test
public void completeTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  //任务处理
  taskService.complete("10305");
}
```

###  组任务

- 查询组任务
```java
@Test
public void queryGroupTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  List<Task> list = taskService.createTaskQuery()
          .processDefinitionKey("activitiKey")
          .taskCandidateUser("用户")
          .list();
  for (Task task : list) {
      System.out.println("-------------------------");
      System.out.println("流程实例ID:" + task.getProcessInstanceId());
      System.out.println("任务ID:" + task.getId());
      System.out.println("任务负责人:" + task.getAssignee());
      System.out.println("任务名称:" + task.getName());
  }
}
```

- 拾取组任务
```java
@Test
public void claimGroupTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  Task task = taskService.createTaskQuery()
          .taskId("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018")
          .taskCandidateUser("用户")
          .singleResult();
  if (null != task) {
      taskService.claim("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018", "用户");
      System.out.println("任务拾取成功");
  }
}
```

- 办理个人任务
```java
@Test
public void completeTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  Task task = taskService.createTaskQuery()
          .processDefinitionKey("activitiKey")
          .taskCandidateUser("用户")
          .singleResult();
  if (null != task) {
      taskService.complete(task.getId());
      System.out.println("用户任务执行完毕");
  }
}
```

- 归还组任务
```java
@Test
public void pushGroupTask(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  Task task = taskService.createTaskQuery()
          .taskId("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018")
          .taskAssignee("用户")
          .singleResult();
  if (null != task) {
      taskService.setAssignee(task.getId(), null);
      System.out.println("归还组任务完成");
  }
}
```

- 交接任务
```java
@Test
public void pushToCandateUser(){
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  Task task = taskService.createTaskQuery()
          .taskId("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018")
          .taskAssignee("用户")
          .singleResult();
  if (null != task) {
      Task task1 = taskService.createTaskQuery()
              .taskCandidateUser("用户2")
              .singleResult();
      if (null != task1) {
          taskService.setAssignee("981a5f2e-e83f-11ea-8f9d-2a7fcfb37018", "用户2");
      }
  }
}
```

### 历史流程查询
```java
@Test
public void getHistory() {
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  HistoryService historyService = processEngine.getHistoryService();
  //获取历史任务
  HistoricActivityInstanceQuery historicActivityInstanceQuery = historyService.createHistoricActivityInstanceQuery();
  //获取指定流程实例的任务
  historicActivityInstanceQuery.processInstanceId("2501");
  //获取任务列表
  List<HistoricActivityInstance> list = historicActivityInstanceQuery.list();
  for (HistoricActivityInstance ai : list) {
      System.out.println("任务节点ID："+ai.getActivityId());
      System.out.println("任务节点名称："+ai.getActivityName());
      System.out.println("流程实例ID信息："+ai.getProcessDefinitionId());
      System.out.println("流程实例ID信息："+ai.getProcessInstanceId());
      System.out.println("==============================");
  }
}
```

### 8、启动生成数据表、进行流程测试

# 四、源码

- Service和Query
- 命令模式
- BPMN解析
- 表的生成

# 五、链接

[Activiti官网](https://www.activiti.org/ "Activiti官网")

[spring-boot-activiti官网](https://spring.io/blog/2015/03/08/getting-started-with-activiti-and-spring-boot "spring-boot-activiti官网")