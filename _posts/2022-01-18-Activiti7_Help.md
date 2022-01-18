---
layout: post
title: Activiti7工作流指南
date: 2022-01-18
Author: hhw
comments: true
toc: true
tags:  [java,activiti]
---

# Activiti工作流笔记

# 一、基础操作

## 1、介绍&使用步骤 

### 启动一个流程实例

流程实例也叫：Processinstance
启动一个流程实例表示开始一次业务流程的运行。
在员工请假流程定义部署完成后，如果张三要请假就可以启动一个流程实例，如果李四要请假也启动一个流程实例，两个流程的执行互相不影响。

### 用户查询待办任务(Task)

因为现在系统的业务流程已经交给activiti管理，通过activiti就可以查询当前流程执行到哪了，当前用户需要办理什么任务了，这些activit帮我们管理了，而不需要开发人员自己编写在sql语句查询。

### 用户办理任务

用户查询待办任务后，就可以办理某个任务，如果这个任务办理完成还需要其它用户办理，比如采购单创建后由部门经理审核，这个过程也是由activiti帮我们完成了。

### 流程结束

当任务办理完成没有下一个任务结点了，这个流程实例就完成了。

## 2、初始化Activiti

### 1.配置maven

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.7</maven.compiler.source>
  <maven.compiler.target>1.7</maven.compiler.target>
  <slf4j.version>1.6.6</slf4j.version>
  <log4j.version>1.2.12</log4j.version>
</properties>

<dependencies>

  <!-- activiti 工作流相关包、mysql、mybatis、log4j相关包 -->

  <dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-engine</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-bpmn-model</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-bpmn-converter</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-json-converter</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>org.activiti.cloud</groupId>
    <artifactId>activiti-cloud-services-api</artifactId>
    <version>7.0.0.Beta1</version>
  </dependency>

  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.40</version>
  </dependency>

  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
  </dependency>

  <!-- log start -->
  <dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>${log4j.version}</version>
  </dependency>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>${slf4j.version}</version>
  </dependency>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>${slf4j.version}</version>
  </dependency>
  <!-- log end -->

  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
  </dependency>

  <dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
  </dependency>
</dependencies>
```

### 2.配置activiti.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Activiti配置专区 -->
    <!--配置Activiti使用的processEngine对象   默认命名为processEngineConfiguration-->
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <!--注入数据源-->
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/activiti?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>
        <property name="jdbcUsername" value="root"/>
        <property name="jdbcPassword" value="123456"/>
        <property name="jdbcMaxActiveConnections" value="3"/>
        <property name="jdbcMaxIdleConnections" value="1"/>
        <!--指定数据库生成策略-->
        <property name="databaseSchemaUpdate" value="true"/>
    </bean>
</beans>

```

### 3.自动生成表结构

```java
// 获取ProcessEngine对象，这里要求activiti.cfg.xml文件必须在resource下，如果要自定义文件，也可以采用另外的自定义方式
// 这里会自动往数据库里生成25张activiti所需要用的表
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
// 输出ProcessEngine对象
System.out.println(processEngine);
```

## 3、表结构介绍

看到刚才创建的25张表，我们发现Activiti的表都以ACT_开头。

第二部分是表示表的用途的两个字母标识。用途也和服务的API对应。

**ACT_RE：**'RE'表示repository。这个前缀的表包含了流程定义和流程静态资源（图片，规则，等等）。

**ACT_RU：**'RU'表示runtime。这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。Activiti只在流程实例执行过程中保存这些数据，<u>在流程结束时就会删除这些记录</u>。这样运行时表可以一直很小速度很快。

**ACT_HI：**'HI'表示history。这些表包含历史数据，比如历史流程实例，变量，任务等等。

**ACT_GE：**GE表示general。通用数据，用于不同场景下

![image-20220108110735196](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108110743.png)

1. RU开头的运行实例记录表，在流程结束后，该流程的相关记录都会被删除

## 4、ProcessEngine

### 1.Service服务接口创建

通过ProcessEngine对象创建Service服务接口

方式如下：

```java
 @Test
 public void testCreateService() {
   ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
   // 获取RuntimeService
   RuntimeService runtimeService = processEngine.getRuntimeService();
}
```

### 2.Service总览

![image-20220108111732930](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108111735.png)

简单介绍： 

**RepositoryService** 

- 是activiti的资源管理类，提供了<font color="blue">管理和控制流程发布包和流程定义的操作</font>。使用工作流建模工具设计的业务流程图需要<font color="red">使用此service将流程定义文件的内容部署到计算机。</font>

- 除了部署流程定义以外还可以：查询引擎中的发布包和流程定义。 

- 暂停或激活发布包，对应全部和特定流程定义。 暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。获得多种资源，像是包含在发布包里的文件， 或引擎自动生成的流程图。 

- 获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

**RuntimeService**

- Activiti的流程运行管理类。可以从这个服务类中获取很多关于流程执行相关的信息

**TaskService** 

- Activiti的任务管理类。可以从这个类中获取任务的信息。

**HistoryService** 

- Activiti的历史管理类，可以查询历史信息，执行流程时，引擎会保存很多数据（根据配置），比如流程 实例启动时间，任务的参与者， 完成任务的时间，每个流程实例的执行路径等等。 这个服务主要通过查询功能来获得这些数据。

**ManagementService**

-  Activiti的引擎管理类，提供了对 Activiti 流程引擎的管理和维护功能，这些功能不在工作流驱动的应用 程序中使用，主要用于 Activiti 系统的日常维护。

## 5、流程操作

### 1.流程绘制

#### 1.下载BPMN流程设计插件

![image-20220108120148228](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108120149.png)

#### 2.新建BPMN文件

![image-20220108120247629](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108120249.png)

#### 3.进入设计器

![image-20220108120337090](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108120339.png)

#### 4.简单流程设计

<img src="Activiti工作流笔记.assets/image-20220108120408635.png" alt="image-20220108120408635" style="zoom: 50%;" />

- 选择起始事件：start event

- 选择userTask，并指定操作人和流程名称

  <img src="Activiti工作流笔记.assets/image-20220108120518802.png" alt="image-20220108120518802" style="zoom: 33%;" />

- 最后选择结束事件

- 点击空白处，指定该流程的id

  ![image-20220108120606469](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108120607.png)

#### 5.结束

结束后，将该文件放到resource下

#### 6.网关&图标介绍

BPMN2.0的基本符合主要包含：

 **事件 Event** 

![image-20220108121007733](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108121008.png)

**活动 Activity** 

活动是工作或任务的一个通用术语。一个活动可以是一个任务，还可以是一个当前流程的子处理流程；其次，你还可以为活动指定不同的类型。常见活动如下：

![image-20220108121014563](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108121015.png)

**网关 GateWay** 

网关用来处理决策，有几种常用网关需要了解：

![image-20220108121045043](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108121046.png)

**排他网关 (x)**

 ——只有一条路径会被选择。流程执行到该网关时，按照输出流的顺序逐个计算，当条件的计算结果为 true时，继续执行当前网关的输出流；

 	如果多条线路计算结果都是 true，则会执行第一个值为 true 的线路。如果所有网关计算结果没有 true，则引擎会抛出异常。

​	 排他网关需要和条件顺序流结合使用，default 属性指定默认顺序流，当所有的条件不满足时会执行默 认顺序流。 

**并行网关 (+)** 

——所有路径会被同时选择

​	 拆分 —— 并行执行所有输出顺序流，为每一条顺序流创建一个并行执行线路。 

​	合并 —— <font color="blue">所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。 </font> 会签？

**包容网关 (+)**

 —— 可以同时执行多条线路，也可以在网关上设置条件

​	 拆分 —— 计算每条线路上的表达式，当表达式计算结果为true时，创建一个并行线路并继续执行

​	 合并 —— 所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下 执行。 

**事件网关 (+)** 

​	—— 专门为中间捕获事件设置的，允许设置多个输出流指向多个不同的中间捕获事件。当流程执行到事 件网关后，流程处于等待状态，需要等待抛出事件才能将等待状态转换为活动状态。

#### 7.画板功能

```
Connection—连接

Event---事件

Task---任务 

Gateway---网关

Container—容器 

Boundary event—边界事件

Intermediate event- -中间事件
```

### 2.流程部署

#### 2.1单个文件部署

```JAVA
@Test
public void testLoadBPMN() {
    // 1.获取ProcessEngine对象
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2.获取RepositoryService进行部署操作
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3.添加部署文件
    Deployment deploy = repositoryService.createDeployment()
            .addClasspathResource("bpmn/ActivitiDemoProject.bpmn20.xml")
            .name("请假申请流程")
            .deploy();

    // 输出刚才部署流程的信息
    String id = deploy.getId();
    System.out.println(id);
}
```

可以在日志中查看到相关的输出信息，往下图3张表插入了流程信息

- act_re_deployment: 流程定义部署表，每部署一次就增加一条记录

- act_re_procdef ：流程定义表，部署每个新的流程定义都会在这张表中增加一条记录，包含流程的key

- act_ge_bytearray ：流程资源表，流程部署的 bpmn文件和png图片会保存在该表中

![image-20220108132526078](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220108132527.png)

#### 2.2压缩包部署

暂略

#### 2.3查询所有部署的流程

```java
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine(); RepositoryService repositoryService = engine.getRepositoryService();  
// 获取一个 ProcessDefinitionQuery对象 用来查询操作
ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery(); List<ProcessDefinition> list = processDefinitionQuery.processDefinitionKey("evection") .orderByProcessDefinitionVersion() // 安装版本排序 .desc() // 倒序 .list(); 
// 输出流程定义的信息 
for (ProcessDefinition processDefinition : list) {
System.out.println("流程定义的ID：" + processDefinition.getId());    System.out.println("流程定义的name：" + processDefinition.getName());  System.out.println("流程定义的key:" + processDefinition.getKey()); System.out.println("流程定义的version:" + processDefinition.getVersion()); System.out.println("流程部署的id:" + processDefinition.getDeploymentId()); 
}

输出：
流程定义的ID：evection:1:12504 
流程定义的name：
出差申请单 流程定义的key:evection
流程定义的version:1 流程部署的id:12501
```

#### 2.4删除部署的流程

```java
ProcessEngine engine = ProcessEngines.getDefaultProcessEngine(); RepositoryService repositoryService = engine.getRepositoryService();
// 删除流程定义，如果该流程定义已经有了流程实例启动则删除时报错
// ACT_RE_DEPLOYMENT的id
repositoryService.deleteDeployment("12501"); 
// 设置为TRUE 级联删除流程定义，及时流程有实例启动，也可以删除，设置为false 非级联删 除操作。
//repositoryService.deleteDeployment("12501",true);
```



### 3.启动流程实例

> 流程定义部署在Activiti后就可以通过工作流管理业务流程，也就是说上边部署的请假申请流程可以使用了。
>
> 针对该流程，<font color="blue">启动一个流程表示发起一个新的请假申请单，这就相当于Java类和Java对象的关系</font>，类定
>
> 义好了后需要new创建一个对象使用，当然可以new出多个对象来，对于出差申请流程，张三可以发起
>
> 一个出差申请单需要启动一个流程实例。

1. 启动流程实例

```java
// 1.获取ProcessEngine对象
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
// 2.获取RuntimeService对象
RuntimeService runtimeService = processEngine.getRuntimeService();
// 3.根据流程key启动流程（ACT_RE_PROCDEF流程定义表的key）
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("StaffLeave");
// 从实例可以获取一些相关信息
System.out.println("流程定义的ID：" + processInstance.getProcessDefinitionId());
System.out.println("流程实例的ID：" + processInstance.getId());
System.out.println("当前活动的ID：" + processInstance.getActivityId());
 /**
  流程定义的ID：StaffLeave:1:3
  流程实例的ID：2501 (act_hi_procinst表)
  当前活动的ID：null
*/
```

<font color="blue"> 启动流程实例涉及到的表结构</font>

- act_hi_actinst  流程实例执行历史

- act_hi_identitylink 流程的参与用户的历史信息

- act_hi_procinst 流程实例历史信息<font color="red">（记录了当前流程实例ID）</font>

- act_hi_taskinst 流程任务历史信息

- act_ru_execution 流程执行信息

- act_ru_identitylink 流程的参与用户信息

- act_ru_task 任务信息

> <font color="blue"> 其中，act_ru_task 记录了当前的任务（即实例）。act_ru_identitylink记录当前流程的待操作用户信息。这两张表之后查询待办任务有用</font>

### 4.查找待办&已办

流程启动后，任务的负责人就可以查询自己当前能够处理的任务了，即待办列表

#### 4.1直接使用api查询

```java
// 这里写死当前操作的用户
String assignee = "staff";
// 1.获取ProcessEngine对象
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
// 2.获取taskService对象
TaskService taskService = processEngine.getTaskService();
// 3.根据流程key和任务负责人来查询待办任务 （流程key指明了当前是什么流程，任务负责人就好比当前登录的用户）
List<Task> taskList = taskService.createTaskQuery()
  .processDefinitionKey("StaffLeave")
  .taskAssignee(assignee)
  .list();
for (Task task : taskList) {
  System.out.println("流程实例id：" + task.getProcessInstanceId());
  System.out.println("任务id:" + task.getId());
  System.out.println("任务负责人：" + task.getAssignee());
  System.out.println("任务名称：" + task.getName());
}

/**
  输出：
  流程实例id：2501
  任务id:2505
  任务负责人：staff
  任务名称：发起请假单
*/
```

从日志可以看到具体执行的sql：

```sql
select distinct RES.*
from ACT_RU_TASK RES
         inner join ACT_RE_PROCDEF D on RES.PROC_DEF_ID_ = D.ID_
WHERE RES.ASSIGNEE_ = ?
  and D.KEY_ = ?
order by RES.ID_ asc
LIMIT ? OFFSET ?

=> Parameters: staff(String), StaffLeave(String), 2147483647(Integer), 0(Integer)
```

<font color="red">一个流程实例有多个流程节点，每个节点都会生成一个任务，每个任务都会有指定的用户来操作。由此可以看出 task表的ASSIGNEE_字段指向当前任务能操作的用户名称。task表只展示当前节点的任务，如果任务完成了，该节点的任务会从task表删除并插入下一节点的task</font>

<font color="blueyellow">表关系 </font>

ACT_RE_PROCDEF 流程定义表 1：n  ↓

ACT_HI_PROCINST 流程实例表 1：n  ↓

ACT_RU_TASK 任务表

ACT_HI_ACTINST 流程节点（含头尾开始和结束节点）

ACT_HI_TASKINST 流程节点（和ACT_HI_ACTINST的区别而是 只包括中间节点，只记录usertask内容）

#### 4.2结合业务查询【见进阶4.2.1】

结合业务查询需要自己写sql进行，不使用api，任务表会记录

【这里留到后面去补充，有可能是创建流程实例的时候指定businessKey，之后根据businessKey去查询，需要自己关联相关的表】



### 5.流程处理

任务负责人查询出来了待办任务，可以去完成该任务。

```java
// 这里写死当前操作的用户（通过更换用户即可实现流程的每一步审批）
String assignee = "staff";
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
TaskService taskService = processEngine.getTaskService();
// 根据流程Key、流程实例id、操作人查询任务
Task task = taskService.createTaskQuery()
  .processDefinitionKey("StaffLeave")
  .taskAssignee(assignee)
  .processInstanceId("2501")
  .singleResult();
if (Objects.isNull(task)) {
  throw new RuntimeException("当前流程不存在或无操作权限");
}
// 完成任务
taskService.complete(task.getId());
// 一个流程实例的每个节点都会生成一个任务，每个任务都会有指定的用户来操作
```

- task表只展示当前节点的任务，任务执行完成后，该节点的任务会从task表删除并插入下一节点的task。

  例如此时staff发起请假task结束，节点流转到组长审批，此时task表的记录只会有组长审批的任务，因此我们可以通过task表来根据用户查询当前待办任务。

- 然后就是不同的用户登录，然后查询待办任务，直到流程走完。

### 6.查询实例的流程节点

即使流程定义已经被删除了，流程执行的实例信息通过前面的分析，依然保存在Activiti的act_hi_* 的相关表结构中，所以我们还是可以查询流程的执行的历史信息，可以通过HistoryService来查看。

**使用API查询：**

查询某个实例下的所有流程节点

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
HistoryService historyService = processEngine.getHistoryService();
// 获取 actinst 表的查询对象，可根据流程实例id查询
HistoricActivityInstanceQuery historicActivityInstanceQuery = historyService.createHistoricActivityInstanceQuery();
historicActivityInstanceQuery.processInstanceId("2501");
historicActivityInstanceQuery.orderByHistoricActivityInstanceEndTime().desc();
List<HistoricActivityInstance> list = historicActivityInstanceQuery.list();
```

**自己写SQL查询：**

> ACT_HI_ACTINST 流程节点（含头尾开始和结束节点）
>
> ACT_HI_TASKINST 流程节点（和ACT_HI_ACTINST的区别而是 只包括中间节点，只记录usertask内容）

可以通过这2张表去查询某个实例的历史流程节点信息

 ```sql
 SELECT T.NAME_ AS '节点', T.ASSIGNEE_ AS '审批人', T.END_TIME_ AS '审批时间'
 FROM ACT_HI_PROCINST P
     LEFT JOIN ACT_HI_TASKINST T ON T.PROC_INST_ID_ = P.ID_
 ```

# 二、进阶内容

## 1、开发流程概述

1. 使用流程符号画出流程图
2. 把流程定义部署到数据库中
3. 启动流程，创建流程实例
4. 操作流程中的人物

## 2、BusinessKey

- <font color="blue">在实际业务中，需要将activiti表与业务表进行结合</font>
- <font color="blue">在activiti中预留了字段，BusinessKey，专门用来保存业务记录的主键</font>

启动流程实例，传入业务表主键

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RuntimeService runtimeService = processEngine.getRuntimeService();
// 启动流程实例，bs001业务表主键
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("StaffLeave", "bs001");
// 输出该实例的businessKey
System.out.println(processInstance.getBusinessKey());
```

启动后，可在ACT_RU_TASK任务表关联ACT_HI_PROCINST实例表查询到businessKey

## 3、流程实例暂停与激活

- 在实际场景中，可能由于流程变更需要将当前运行的流程暂停而不是删除，流程暂停后将不能继续执行

### 3.1全部流程挂起

- 操作流程的定义为挂起状态，该流程定义下边所有的流程实例全部暂停。
- 流程定义为挂起状态，该流程定义将不允许启动新的流程实例，同时该流程定义下的所有的流程实例都将全部挂起暂停执行。

![image-20220110211746270](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220110211754.png)

- <font color="blue">挂起流程定义后，ACT_RU_TASK中的SUSPENSION_STATE_状态会修改为2 </font>

- 此时再去操作该流程实例会提示异常：Cannot complete a  suspended task

### 3.2单个实例挂起

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RuntimeService runtimeService = processEngine.getRuntimeService();
// 查询出要挂起的流程实例对象
ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
        .processDefinitionKey("StaffLeave")
        .singleResult();
// 判断实例是否挂起
boolean suspended = processInstance.isSuspended();
if (suspended) {
    // 激活
    System.out.println("当前是挂起状态，正在激活中");
    runtimeService.activateProcessInstanceById(processInstance.getId());
} else {
    // 挂起
    System.out.println("当前是激活状态，正在挂起");
    runtimeService.suspendProcessInstanceById(processInstance.getId());
}
```

- <font color="blue">挂起流程定义后，ACT_RU_TASK中的SUSPENSION_STATE_状态会修改为2 </font>

- 此时再去操作该流程实例会提示异常：Cannot complete a  suspended task

## 4、动态分配任务审批人（UEL表达式）

### 4.1分配任务审核人

- 之前演示的内容是固定任务的审批人（固定Assignee值）
- 此处改写为UEL表达式，可以根据实际业务<font color="blue">动态</font>指定审批人

#### 4.1.1表达式分配

> 在Activiti中支持使用UEL表达式，UEL表达式是Java EE6 规范的一部分， UEL(Unified Expression Language) 即 统一表达式语音， Activiti支持两种UEL表达式： UEL-value 和UEL-method

##### UEL-value

**①绘制UEL流程：**

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220110220100.png" alt="image-20220110220055300" style="zoom:50%;" />

**②部署流程（略）**

**③创建实例**

创建时设置UEL的值

```java
// 1.获取ProcessEngine对象
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
// 2.获取RuntimeService对象
RuntimeService runtimeService = processEngine.getRuntimeService();
// 3.根据流程key启动流程（ACT_RE_PROCDEF流程定义表的key），设置Assignee的取值（UEL）
Map<String, Object> map = new HashMap<>(16);
map.put("staff", "Tom");
map.put("grouper", "Jane");
map.put("manager", "Lily");
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("StaffLeaveUEL", map);
// 从实例可以获取一些相关信息
System.out.println("流程定义的ID：" + processInstance.getProcessDefinitionId());
System.out.println("流程实例的ID：" + processInstance.getId());
System.out.println("当前活动的ID：" + processInstance.getActivityId());
```

- <font color="blue">创建实例后，ACT_RU_VARIABLE 表会记录创建该实例时传入的相关信息</font>

  ![image-20220110215955365](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220110215957.png)

##### UEL-method

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220110220550.png" alt="image-20220110220547618" style="zoom:50%;" />

- <font color="blue">userBean 是 spring 容器中的一个 bean，表示调用该 bean 的 getUserId()方法。相当于UEL中可以执行Service方法</font>

##### UEL-value与UEL-method

- 再比如：${IManageService.findManagerForEmployee(emp)}

  IManageService 是 spring 容器的一个 bean，findManagerForEmployee 是该 bean 的一个方法，emp 是activiti流程变量， emp 作为参数传到 IManageService.findManagerForEmployee 方法中。

##### 其他

- 表达式支持解析基础类型、 bean、 list、 array 和 map，也可作为条件判断。

  如：${order.price > 100 && order.price < 250}

#### 4.1.2监听器分配

- 设计流程时可给userTask添加监听器
- 使用一个java类当做监听器，可在流程启动或执行时给相应的节点设置属性，例如Assignee，在流程设计的时候就不需要指定Assignee
- 共有4种类型：
  - create:任务创建后触发 
  - assignment:任务分配后触发 
  - Delete:任务完成后触发 
  - All：所有事件都触发 
- 监听器可以获取task的name和event的name，以此来进行多种判断赋值
- 其他略

### 4.2查询任务

#### 4.2.1关联BusinessKey查询待办

**需求：**

在 activiti 实际应用时，查询待办任务可能要显示出业务系统的一些相关信息。

比如：查询待审批请假任务列表需要将出差单的日期、 请假天数等信息显示出来。

出差天数等信息在业务系统中存在，而并没有在 activiti 数据库中存在，所以是无法通过 activiti 的 api查询到请假天数等信息。

**实现：**

在查询待办任务列表时，通过 businessKey（业务标识 ）关联查询业务系统的请假单表，查询出请假天数等信息。

这里我们使用自己编写的SQL查询：

**思路：**可在ACT_RU_TASK任务表关联ACT_HI_PROCINST实例表查询到businessKey，再关联业务表主键

```SQL
SELECT L.name as 请假人, L.day, T.NAME_, T.PROC_INST_ID_
FROM ACT_RU_TASK T
    JOIN ACT_HI_PROCINST P ON P.ID_ = T.PROC_INST_ID_
    JOIN ACT_RE_PROCDEF F ON F.ID_ = P.PROC_DEF_ID_
    JOIN T_STAFF_LEAVE L ON L.id = P.BUSINESS_KEY_
WHERE F.KEY_ = 'StaffLeave'
     AND T.ASSIGNEE_ = 'staff'
```

查询出来后再根据实际进行任务处理

## <font color="blue">5、流程变量【重要】</font>

流程变量在 activiti 中是一个非常重要的角色，流程运转有时需要靠流程变量，业务系统和 activiti

结合时少不了流程变量，流程变量就是 activiti 在管理工作流时根据管理需要而设置的变量。

<font color="blue">比如：在出差申请流程流转时如果出差天数大于 3 天则由总经理审核，否则由人事直接审核， 出差天数就可以设置为流程变量，在流程流转时使用。</font>



**注意：虽然流程变量存储业务的数据可以通过activiti的api查询流程变量从而实现 查询业务数据，但是不建议这样使用，因为业务数据查询由业务系统负责，activiti设置流程变量是为了流程执行需要而创建。（解释：比如请假天数，应该有单独的业务表去存和取，而不是直接通过activiti去查）**

### 5.1流程变量类型

> 如果将pojo存储到流程变量中，实体必须实现序列化（implements Serializable 和 设置serialVersionUID）

<img src="https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220111215925.png" alt="image-20220111215908577" style="zoom:50%;" />

### 5.2流程变量作用域

流程变量的作用域可以是一个流程实例(processInstance)，或一个任务(task)，或一个执行实例(execution)

#### 5.2.1global变量

- 第一种：可以在实例<u>启动时</u>进行变量设置

  ```JAVA
  ProcessEngine engine = ProcessEngines.getDefaultProcessEngine();
  RuntimeService runtimeService = engine.getRuntimeService();
  Map<String, Object> map = new HashMap<>(16);
  Evection evection = new Evection();
  evection.setDay(3);
  // 设置对象变量（全局变量）注意：对象实体需要实现序列化 同时UEL表达式参考：${evection.day}
  map.put("evection", evection);
  map.put("staff", "Tom");
  map.put("grouper", "Jane");
  map.put("manager", "Lily");
  // 流程启动时设置
  ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("ActivitiVariableDemo", map);
  ```

- 第二种：可以在流程<u>执行途中</u>进行变量设置

  ```JAVA
  // 这里写死当前操作的用户，目前是staff用户操作
  String assignee = "Tom";
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  // 根据流程Key、流程实例id、操作人查询任务
  Task task = taskService.createTaskQuery()
    .processDefinitionKey("ActivitiVariableDemo")
    .taskAssignee(assignee)
    .singleResult();
  if (Objects.isNull(task)) {
    throw new RuntimeException("当前流程不存在或无操作权限");
  }
  // 完成任务，【同时设置流程变量：请假天数】，注意：这里设置的是全局变量（流程变量也可以设置一个对象（需实现序列化），同时UEL表达式参考：${user.day}）
  Map<String, Object> map = new HashMap<>(16);
  map.put("day", 2);
  taskService.complete(task.getId(), map);
  ```

- 第三种：通过流程实例**执行id**（executionId）设置

  ```java
  String executionId = "2601";
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  RuntimeService runtimeService = processEngine.getRuntimeService();
  runtimeService.setVariable(executionId, "day", 3);
  // 可设置多个
  runtimeService.setVariable(executionId, "man", "jim");
  ```

  备注：executionId必须是当前为结束的流程实例执行id，通常使用此id设置的流程变量，可以通过`runtimeService.getVariable()`方法获取流程变量

- 第四种：通过**taskId**设置

  ```java
  String taskId = "23122";
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  TaskService taskService = processEngine.getTaskService();
  taskService.setVariable(taskId, "day", 3);
  // 可设置多个
  taskService.setVariable(taskId, "man", "jim");
  ```

  备注：任务id必须是当前待办任务的id（在ac_ru_task表存在），如果该任务已结束，会报错，也可以通过`taskService.getVariable()`方法获取流程变量

- 设置流程变量后，activiti会保存到ACT_RU_VARIABLE表，流程实例结束后从该表删除

- <font color="blue">同一个流程实例中map的key相同，那么后者会覆盖前者</font>

#### 5.2.1local变量

- 第一种：任务办理时设置

  任务办理时设置local流程变量，在任务complete之前使用，任务结束该变量失效，因此无法在当前流程实例使用，可以通过查询历史任务查询。

  因此作用域为任务的local变量，每个任务可以设置相同的变量名，不互相影响。

  ![image-20220112220303070](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220112220304.png)

- 第二种：通过当前任务设置			![image-20220112220333310](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220112220334.png)

​		与通过taskId设置全局变量类似，只是调用的方法不同。

- 查询历史local变量

  ![image-20220112220552449](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220112220554.png)

## 6、组任务(多候选人模式)

### 6.1需求

在流程定义中在任务结点的 assignee 固定设置任务负责人，在流程定义时将参与者固定设置在.bpmn文件中，<font color="blue">如果临时任务负责人变更则需要修改流程定义，系统可扩展性差。</font>

针对这种情况可以给任务设置多个候选人，可以从候选人中选择参与者来完成任务。

### 6.2流程设计

在流程图中任务节点配置candidate-users(候选人)，多个候选人之间用逗号分开。

注意：由于IDEA的bpmn流程设计插件未找到candidate-users选项，只能通过在节点手动添加的方式。或使用eclipse进行设置

<font color="blue">手动在节点设置多候选人，支持UEL表达式</font>，assignee不必设置：

```xml
<!-- 常量写法 格式为多个逗号隔开-->
<userTask id="sid-1c25b116-c78a-4c7c-8fb1-1561c7e134aa" name="组长审核" activiti:candidateUsers="Tom,Jone,Henry"/>
<!-- UEL写法 -->
<userTask id="sid-1c25b116-c78a-4c7c-8fb1-1561c7e134aa" name="组长审核" activiti:candidateUsers="${grouper}"/
```

### 6.3执行

> 部署略

#### 6.3.1启动流程实例

> 启动流程实例，同时设置candidate-user变量（设置候选人）

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
RuntimeService runtimeService = processEngine.getRuntimeService();
Map<String, Object> map = new HashMap<>( 16);
map.put("staff", "Tom");

// 在此设置【设置多候选人变量】*，设置完成流程到了这一节点生成任务后，这些人都可以查出该任务
map.put("grouper", "Jane,Kim,Henry");
map.put("manager", "Lily");

ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("ActivitiCandidateDemo", map);
System.out.println("流程定义的ID：" + processInstance.getProcessDefinitionId());
System.out.println("流程实例的ID：" + processInstance.getId());
System.out.println("当前活动的ID：" + processInstance.getActivityId());
```

此处设置3位候选人Jane、Kim、Henry

#### 6.3.2候选人查询任务

> 查询候选人的待办任务，也称作组任务

```java
String user = "Jane";
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
TaskService taskService = processEngine.getTaskService();
List<Task> taskList = taskService.createTaskQuery()
        .processDefinitionKey("ActivitiCandidateDemo")
        // 查询这里使用 【taskCandidateUser】
        .taskCandidateUser(user)
        .list();
for (Task task : taskList) {
    System.out.println("流程实例id：" + task.getProcessInstanceId());
    System.out.println("任务id:" + task.getId());
    System.out.println("任务负责人：" + task.getAssignee());
    System.out.println("任务名称：" + task.getName());
}
```

#### 6.3.3拾取与完成任务

> 多候选人模式的任务（也称组任务），需要先【拾取】，才能complete

```java
// 根据任务id和user查询组待办任务
String user = "Jane";
String taskId = "95003";
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
TaskService taskService = processEngine.getTaskService();
Task task = taskService.createTaskQuery()
        .taskId(taskId)
        .taskCandidateUser(user)
        .singleResult();
if (Objects.isNull(task)) {
    throw new RuntimeException("当前流程不存在或无操作权限");
}
// 拾取任务
taskService.claim(taskId, user);
// 根据用户查询个人任务
Task personTask = taskService.createTaskQuery()
        .taskAssignee(user)
        .singleResult();
System.out.println("流程实例id：" + personTask.getProcessInstanceId());
System.out.println("任务id:" + personTask.getId());
System.out.println("任务负责人：" + personTask.getAssignee());
System.out.println("任务名称：" + personTask.getName());
// 完成任务
taskService.complete(personTask.getId());
```

#### 6.3.4归还任务

> 归还任务：如果当前用户拾取了任务又不想处理，可以进行归还，归还后，该候选人无法再拾取和处理该任务

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
TaskService taskService = processEngine.getTaskService();
String user = "Jane";
String taskId = "95003";
Task task = taskService.createTaskQuery()
        .taskId(taskId)
        .taskCandidateUser(user)
        .singleResult();
if (Objects.isNull(task)) {
    throw new RuntimeException("当前流程不存在或无操作权限");
}
// 归还任务
taskService.setAssignee(taskId, null);
```

#### 6.3.5任务交接

> 任务交接：指定另外一个负责人，如果当前用户无法处理，可以指定另外一个人来处理

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
TaskService taskService = processEngine.getTaskService();
String user = "Jane";
String taskId = "95003";
Task task = taskService.createTaskQuery()
        .taskId(taskId)
        .taskCandidateUser(user)
        .singleResult();
if (Objects.isNull(task)) {
    throw new RuntimeException("当前流程不存在或无操作权限");
}
// 任务交接,指定新负责人
String newAssignee = "Henry";
taskService.setAssignee(taskId, newAssignee);
```

### 6.4涉及的表

- 查询当前任务执行表

  ```sql
  SELECT * FROM act_ru_task
  ```

  记录当前待办任务，由于该任务是组任务，在任务拾取之前，Assignee为空，当拾取后，该字段会赋值任务负责人，之后该用户就可以通过个人任务查询到该任务。

- 查询任务参与者

  ```sql
  SELECT * FROM act_ru_identitylink
  ```

  - 任务参与者，记录当前参考任务用户或组，当前任务如果设置了候选人，会向该表插入候选人记录，有几个候选就插入几个。
  - 与act_ru_identitylink对应的还有一张历史表act_hi_identitylink，向act_ru_identitylink插入记录的同时也会向历史表插入记录。任务完成

- 查询实例任务候选人

  如果实例结束了，想找某个实例下的所有任务和对应的候选人信息，可以通过以下方式

  ```sql
  SELECT K.*
  FROM ACT_HI_PROCINST P
       JOIN ACT_HI_TASKINST T ON T.PROC_INST_ID_ = P.ID_
       JOIN ACT_HI_IDENTITYLINK K ON K.TASK_ID_ = T.ID_
  WHERE  P.ID_ = '92501'
  ```

  

## 7、网关

网关用来控制流程的流向

### 7.1排他网关ExclusiveGateway

#### 7.1.1什么是排他网关

> 排他网关，用来在流程中实现决策。 当流程执行到这个网关，所有分支都会判断条件是否为true，如果为true则执行该分支。
>
> <font color="blue">**注意**：排他网关只会选择一个为true的分支执行。如果有两个分支条件都为true，排他网关会选择id值较小的一条分支去执行。</font>

<font color="blue">排他网关与直接设置流程变量的区别：</font>

在使用连线的condition条件时，不使用排他网关也可以实现一样的流程效果。但缺点是如果条件都为false时，流程就结束（异常结束），若使用了排他网关，则会抛出异常。相当于排他网关使得整个的流程会更加规范。异常信息：

```log
org.activiti.engine.ActivitiException: No outgoing sequence flow of the exclusive gateway 'exclusivegateway1' could be selected for continuing the process at ]org.activiti.engine.impl.bpmn.behavior.ExclusiveGatewayActivityBehavior.leave(Ex clusiveGatewayActivityBehavior.java:85)
```

#### 7.1.2流程设计

- 新增排他网关
- 在设置Default flow element时要注意，这是设置默认走的连线，注意：默认走的连线不能设置条件变量，否则部署时会报异常。

![image-20220116173542458](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116173551.png)

- 排他网关后的连线依然要设置变量

![image-20220116173527342](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116173548.png)

#### 7.1.3执行

基本与之前流程执行类似，略

### 7.2并行网关ParallelGateway

会签使用

#### 7.2.1什么是并行网关

 并行网关允许将流程分成多条分支，也可以把多条分支汇聚到一起，并行网关的功能是基于进入和外出顺序流的：

l fork分支：并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。

l join汇聚：所有到达并行网关，在此等待的进入分支， 直到所有进入顺序流的分支都到达以后， 流程就会通过汇聚网关。

注意，如果同一个并行网关有多个进入和多个外出顺序流， 它就同时具有分支和汇聚功能。 这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支。

**与其他网关的主要区别是，并行网关不会解析连线的条件。 即使定义了条件，也会被忽略。**

#### 7.2.2流程设计

![image-20220116215629581](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116215630.png)



- 使用并行网关，实现请假需要组长和项目经理会签。
- 并行结束后重新汇聚，经过排他网关，根据请假天数判断是否需要进行总经理审批。
- 流程执行略

#### 7.2.3涉及的表

1. 在到达并行网关后，task表会生成2条任务，execution表一共会有3条记录（正常是2条）：

   task：组长审核和项目经理审核

   ![image-20220116215851769](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116215853.png)

   execution：指向当前并行的2个节点，如果其中一条办结，会删除该办结的节点，生成新的节点指向并行网关（汇聚），未办结的节点还会存在。

   ![image-20220116221601376](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116221602.png)

2. 会签中其中一个任务完成，并不会生成下一个任务，还需等待另一个任务完成

   ![image-20220116220157082](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220116220158.png)

3. 并行执行完成后，execution记录变回2条，task表记录恢复。execution表记录实例当前正常执行的节点，并且会冗余一条节点在实例全流程中都存在。

### 7.3包含网关InclusiveGateway

#### 7.3.1什么是包含网关

> 包含网关可以看作是排他网关和并行网关的结合。
>
> 包含网关可以实现排他网关的功能（2条路径任选其一）也可以实现并行网关的功能（路径分支与汇聚）

#### 7.3.2流程设计

![image-20220118210953143](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220118210955.png)

- 此处使用包含网关，根据天数判断流程路径。以day=3为例，此时会途径开发经理审核与人事经理审核。
- 带有条件的分支需要满足条件才会进入，不带条件的分支一定会进入。
- 最分支流程走完后会进行汇聚。
- 相关的变更与并行网关类似，暂略











































































