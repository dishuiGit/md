[TOC]

## 历史数据查询
#### 查询历史流程实例列表数据的表为: `act_hi_procinst`

```java
    @Test
    public void test4() {
        // 历史流程实例查询对象,对应查询数据表为act_hi_procinst
        HistoricProcessInstanceQuery query = pe.getHistoryService()
                .createHistoricProcessInstanceQuery();
        List<HistoricProcessInstance> list = query.list();
        for (HistoricProcessInstance pi : list) {
            System.out.println(pi.getId());
        }
    }
```

#### 查询历史活动列表数据,对应查询的表为：`act_hi_actinst`

```java
    @Test
    public void test5() {
        HistoricActivityInstanceQuery query = pe.getHistoryService()
                .createHistoricActivityInstanceQuery();
        query.orderByProcessInstanceId().asc();
        query.orderByHistoricActivityInstanceEndTime().asc();
        List<HistoricActivityInstance> list = query.list();
        for (HistoricActivityInstance ai : list) {
            System.out.println(ai.getActivityId());
        }
    }
```

#### 查询历史任务列表数据,对应查询的表为：`act_hi_taskinst`

```java
    @Test
    public void test6() {
        HistoricTaskInstanceQuery query = 
                pe.getHistoryService().createHistoricTaskInstanceQuery();
        query.orderByProcessInstanceId().desc();
        query.orderByHistoricTaskInstanceEndTime().asc();
        List<HistoricTaskInstance> list = query.list();
        for (HistoricTaskInstance task : list) {
            System.out.println(task.getAssignee() + " " + task.getName());
        }
    }
```

## 流程变量
### 设置流程变量

>1.启动流程实例时设置流程变量

```java
    @Test
    public void test2() {
        String processDefinitionKey = "qjlc";
        Map<String, Object> variables = new HashMap<String, Object>();
        // variables.put("qjts", 3);
        // variables.put("qjyy", "旅游");
        variables.put("user", new User(10, 20, "北京"));
        ProcessInstance pi = pe.getRuntimeService().startProcessInstanceByKey(
                processDefinitionKey, variables);
        System.out.println(pi.getId());
    }
```

>2.办理任务时设置

```java
    @Test
    public void test3() {
        String taskId = "1702";
        Map<String, Object> variables = new HashMap<String, Object>();
        variables.put("key1", "value1");
        variables.put("k2", 300);
        pe.getTaskService().complete(taskId, variables);
    }
```

>3.使用RuntimeService的方法设置

```java
    @Test
    public void test4() {
        String executionId = "1101";// 流程实例的id
        String variableName = "k3";
        Object value = "v3";
        pe.getRuntimeService().setVariable(executionId, variableName, value);
        // pe.getRuntimeService().setVariables(executionId, variables);
    }
```

>4.使用TaskService的方法设置

```java
    @Test
    public void test5() {
        String taskId = "1204";
        Map<String, Object> variables = new HashMap<String, Object>();
        variables.put("k4", "value4");
        // pe.getTaskService().setVariable(taskId, variableName, value);
        pe.getTaskService().setVariables(taskId, variables);
    }
```

-------------------------------------------------------------

### 获取流程变量

>1.使用RuntimeService
    
```java    
    @Test
    public void test6() {
        String executionId = "1101";// 流程实例id
        // 获取指定流程实例下所有的流程变量
        Map<String, Object> map = pe.getRuntimeService().getVariables(
                executionId);
        Set<String> keySet = map.keySet();
        for (String key : keySet) {
            Object value = map.get(key);
            // System.out.println(key + " = " + value);
        }
        // 获取指定流程实例下指定的一个流程变量
        Object value = pe.getRuntimeService().getVariable(executionId, "k3");
        // System.out.println(value);
        // 获取指定流程实例下多个name对应的多个值
        Collection<String> variableNames = new ArrayList<>();
        variableNames.add("k2");
        variableNames.add("k3");
        variableNames.add("k4");
        Map<String, Object> map2 = pe.getRuntimeService().getVariables(
                executionId, variableNames);
        Set<String> keySet2 = map2.keySet();
        for (String key2 : keySet2) {
            Object value2 = map.get(key2);
            System.out.println(key2 + " = " + value2);
        }
    }
```

>2.使用TaskService

```java
    @Test
    public void test7() {
        String taskId = "1204";//任务id,获取当前任务所在流程实例下的所有流程变量
        Map<String, Object> map = pe.getTaskService().getVariables(taskId);
        Set<String> keySet = map.keySet();
        for (String key : keySet) {
            Object value = map.get(key);
            System.out.println(key + " = " + value);
        }
    }
```

## 监听器
#### 任务监听器
```java
public class MyTaskListener1 implements TaskListener {
    /**
     * delegateTask为当前监听到的任务对象的代理对象
     */
    public void notify(DelegateTask delegateTask) {
        //动态指定任务的办理人
        //delegateTask.setAssignee(assignee);
        //当前任务的办理人
        String assignee = delegateTask.getAssignee();
        //任务的创建时间
        Date createTime = delegateTask.getCreateTime();
        //事件名称
        String eventName = delegateTask.getEventName();
        //当前任务所在流程实例的id
        String processInstanceId = delegateTask.getProcessInstanceId();
        //当前监听到的任务的名称
        String name = delegateTask.getName();
        //流程变量
        Set<String> variableNames = delegateTask.getVariableNames();
        for (String key : variableNames) {
            Object value = delegateTask.getVariable(key);
            System.out.println(key + " = " + value);
        }
    }
```
#### 执行(流程)监听器
```java
//自定义一个执行监听器，监听流程实例相关的事件
public class MyExecutionListener1 implements ExecutionListener {
    /**
     * execution为当前流程实例的代理对象
     */
    public void notify(DelegateExecution execution) throws Exception {
        System.out.println("自定义的执行监听器执行了。。。");
        String currentActivityId = execution.getCurrentActivityId();
        String currentActivityName = execution.getCurrentActivityName();
        String id = execution.getId();
        Set<String> variableNames = execution.getVariableNames();
        for (String key : variableNames) {
            Object value = execution.getVariable(key);
            System.out.println(key + " = " + value);
        }
        String processDefinitionId = execution.getProcessDefinitionId();
    }
}
```
## 组任务

```java
public class GroupTaskTest {
    ProcessEngine pe = ProcessEngines.getDefaultProcessEngine();

    // 部署流程定义
    @Test
    public void test1() {
        DeploymentBuilder deploymentBuilder = pe.getRepositoryService()
                .createDeployment();
        deploymentBuilder .addClasspathResource("cn/itcast/activiti/task/group/bxlc.bpmn");
        deploymentBuilder .addClasspathResource("cn/itcast/activiti/task/group/bxlc.png");
        deploymentBuilder.deploy();
    }
    
    //启动流程实例
    @Test
    public void test2(){
        String processDefinitionKey = "bxlc";
        ProcessInstance pi = pe.getRuntimeService().startProcessInstanceByKey(processDefinitionKey );
        System.out.println(pi.getId());
    }
    
    //查询[个人任务]列表
    @Test
    public void test3(){
        TaskQuery query = pe.getTaskService().createTaskQuery();
        String assignee = "王五";
        query.taskAssignee(assignee);
        String processDefinitionKey = "bxlc";
        query.processDefinitionKey(processDefinitionKey );
        List<Task> list = query.list();
        for (Task task : list) {
            System.out.println(task.getId());
        }
    }
    
    //查询[组任务]列表
    @Test
    public void test4(){
        TaskQuery query = pe.getTaskService().createTaskQuery();
        String candidateUser = "李四";
        //使用候选人过滤
        query.taskCandidateUser(candidateUser);
        List<Task> list = query.list();
        for (Task task : list) {
            System.out.println(task.getId());
        }
    }
    
    //办理个人任务
    @Test
    public void test5(){
        String taskId = "2104";
        pe.getTaskService().complete(taskId );
    }
    
    //拾取任务（将组任务变为个人任务）
    @Test
    public void test6(){
        String taskId = "2202";
        String userId = "王五";
        pe.getTaskService().claim(taskId , userId);
    }
    
    //退回任务（将个人任务变为组任务）
    @Test
    public void test7(){
        String taskId = "2202";
        pe.getTaskService().setAssignee(taskId , null);
    }
    
    //分配任务
    @Test
    public void test8(){
        String taskId = "2202";
        pe.getTaskService().setAssignee(taskId , "李四");
    }
    
    //办理任务
    @Test
    public void test9(){
        String taskId = "2202";
        pe.getTaskService().complete(taskId );
    }
}
```


```java

```


```java

```


```java

```


```java

```


```java

```


