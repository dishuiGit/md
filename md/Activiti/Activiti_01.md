[TOC]

### 自动创建表

**activiti-context.xml**
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/activit_01" />
        <property name="jdbcDriver" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="123" />
        <!--自动创建表-->
        <property name="databaseSchemaUpdate" value="true" />
        
    </bean>
    <!--配置流程引擎工厂bean-->
    <bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
        <!--注入配置对象-->
        <property name="processEngineConfiguration" ref="processEngineConfiguration"></property>
    </bean>
</beans>
```
**Demo01.java**
```java
public class Demo01 {
    @Test
    public void test01(){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    }
}
```

### 了解23张表
----------------
>Activiti的后台是有数据库的支持，所有的表都以ACT_开头。
第二部分是表示表的用途的两个字母标识。 用途也和服务的API对应.

--------------------

*  `ACT_RE_*`: <font color="red">**RE**</font>表示`repository`。

    >这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。

--------------------

* `ACT_RU_*`: <font color="red">**RU**</font>表示`runtime`。  

    >这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。
    Activiti只在流程实例执行过程中保存这些数据，流程结束时就会删除这些记录。
    这样运行时表可以一直很小速度很快。

--------------------

* `ACT_ID_*`: <font color="red">**ID**</font>表示`identity`。 

    >这些表包含身份信息，比如用户，组等等。

--------------------

* ` ACT_HI_*`: <font color="red">**HI**</font>表示`history`。 

    >这些表包含历史数据，比如历史流程实例， 变量，任务等等。

--------------------

* `ACT_GE_*`: 通用数据， 用于不同场景下。

--------------------
## API操作

### 流程
#### 1.部署流程定义

```java
/**
     * 部署流程定义方式： 方式一：加载单个的流程定义文件部署 方式二：加载zip文件部署
     * 
     * @throws Exception
     */
    @Test
    public void test1() throws Exception {
        // 方式一：加载单个的流程定义文件部署
          DeploymentBuilder deploymentBuilder = pe.getRepositoryService().createDeployment();
          deploymentBuilder.addClasspathResource("test1.bpmn");
          deploymentBuilder.addClasspathResource("test1.png");
          Deployment deployment = deploymentBuilder.deploy();
          System.out.println(deployment.getId());

        // 方式二：加载zip文件部署
        /*DeploymentBuilder deploymentBuilder = pe.getRepositoryService()
                .createDeployment();*/

        // zip文件在类路径下
        /*
         * ZipInputStream zipInputStream = new ZipInputStream(this.getClass()
         * .getClassLoader().getResourceAsStream("process.zip"));
         */

        // zip不在类路径下
        /*ZipInputStream zipInputStream = new ZipInputStream(new FileInputStream(
                new File("d:\\test1.zip")));
        deploymentBuilder.addZipInputStream(zipInputStream);
        deploymentBuilder.name("部署名称");
        deploymentBuilder.deploy();*/
    }
```
#### 2. 查询部署信息

```java
    /**
     * 查询部署信息 对应查询部署表act_re_deployment
     */
    @Test
    public void test2() {
        DeploymentQuery query = pe.getRepositoryService()
                .createDeploymentQuery();
        query.orderByDeploymenTime().desc();
        List<Deployment> list = query.list();
        for (Deployment deployment : list) {
            System.out.println(deployment.getId() + " " + deployment.getName()
                    + " " + deployment.getDeploymentTime());
        }
    }
```
#### 3. 删除部署信息
```java
    /**
     * 删除部署信息
     */
    @Test
    public void test3() {
        String deploymentId = "801";
        // 级联删除
        pe.getRepositoryService().deleteDeployment(deploymentId, true);
    }
```
### 流程定义
#### 1. 查询流程定义列表
```java
    /**
     * 查询流程定义列表 对应查询流程定义表:act_re_procdef
     */
    @Test
    public void test4() {
        ProcessDefinitionQuery query = pe.getRepositoryService()
                .createProcessDefinitionQuery();
        String processDefinitionKey = "qjlc";
        query.processDefinitionKey(processDefinitionKey);
        query.orderByProcessDefinitionVersion().desc();
        query.listPage(0, 6);// 参数一：从指定索引记录查询 参数二：查询几条数据
        List<ProcessDefinition> list = query.list();
        for (ProcessDefinition processDefinition : list) {
            System.out.println(processDefinition.getId());
        }
    }
```
#### 2. 删除流程定义
```java
    /**
     * 删除流程定义---通过删除部署信息达到删除流程定义的目的
     */
    @Test
    public void test5() {
        // 通过页面传递流程定义的id到服务端
        String pdId = "qjlc:6:904";
        ProcessDefinition pd = pe.getRepositoryService()
                .createProcessDefinitionQuery().processDefinitionId(pdId)
                .singleResult();
        String deploymentId = pd.getDeploymentId();
        pe.getRepositoryService().deleteDeployment(deploymentId);
    }
```
#### 3. 获取一次具体的部署对应的流程
```java
    /**
     * 获取一次具体的部署对应的流程定义文件名称(bpmn png)和输入流
     * 
     * @throws Exception
     */
    @Test
    public void test6() throws Exception {
        String deploymentId = "701";
        List<String> names = pe.getRepositoryService()
                .getDeploymentResourceNames(deploymentId);
        for (String name : names) {
            System.out.println(name);
            // 流程定义文件对应的输入流
            InputStream in = pe.getRepositoryService().getResourceAsStream(
                    deploymentId, name);
            FileUtils.copyInputStreamToFile(in, new File("d:\\" + name));
            // 提供一个输出流，将文件内容写到指定的文件中
            /*
             * OutputStream out = new FileOutputStream(new File("d:\\" + name));
             * byte[] b = new byte[1024]; int len = 0; while((len = in.read(b))
             * != -1){ out.write(b, 0, len); out.flush(); } out.close();
             * in.close();
             */
        }
    }

    /**
     * 直接获得png图片对应的输入流
     * 
     * @throws Exception
     */
    @Test
    public void test7() throws Exception {
        String processDefinitionId = "qjlc:4:704";
        InputStream pngStream = pe.getRepositoryService().getProcessDiagram(
                processDefinitionId);
        FileUtils.copyInputStreamToFile(pngStream, new File("d:\\test.png"));
    }
```
## 启动流程实例

```java
    /**
     * 启动流程实例： 方式一：根据流程定义的id启动流程实例 方式二：根据流程定义的key启动流程实例（自动选择最新版本的流程定义启动流程实例）
     */
    @Test
    public void test8() {
        String processDefinitionId = "qjlc:4:704";// 流程定义的id,对应于流程定义表act_re_procdef主键字段的值
        // 方式一：根据流程定义的id启动流程实例
        /*
         * ProcessInstance pi = pe.getRuntimeService().startProcessInstanceById(
         * processDefinitionId); System.out.println(pi.getId());
         */

        String processDefinitionKey = "qjlc";// 流程定义的key
        // 方式二：根据流程定义的key启动流程实例
        ProcessInstance pi = pe.getRuntimeService().startProcessInstanceByKey(
                processDefinitionKey);
        System.out.println(pi.getId());
    }
```
#### 2. 查询流程实例列表
```java
    /**
     * 查询流程实例列表
     */
    @Test
    public void test9() {
        ProcessInstanceQuery query = pe.getRuntimeService()
                .createProcessInstanceQuery();
        String processDefinitionKey = "qjlc";
        query.processDefinitionKey(processDefinitionKey);
        List<ProcessInstance> list = query.list();
        for (ProcessInstance processInstance : list) {
            System.out.println(processInstance.getId());
        }
    }
```
#### 3. 删除流程实例
```java
    /**
     * 删除流程实例
     */
    @Test
    public void test10() {
        String processInstanceId = "1101";// 流程实例id
        String deleteReason = "我愿意";// 删除原因
        pe.getRuntimeService().deleteProcessInstance(processInstanceId,
                deleteReason);
    }
```
## 任务
#### 1. 查询任务列表
```java
    /**
     * 查询任务列表
     */
    @Test
    public void test11() {
        TaskQuery query = pe.getTaskService().createTaskQuery();
        String assignee = "李四";
        // 根据办理人过滤
        query.taskAssignee(assignee);
        // 根据任务的创建时间降序
        query.orderByTaskCreateTime().desc();
        List<Task> list = query.list();
        for (Task task : list) {
            System.out.println(task.getId() + " " + task.getName() + " "
                    + task.getAssignee());
        }
    }
```
#### 2. 办理任务
```java
    /**
     * 办理任务
     */
    @Test
    public void test12(){
        String taskId = "1602";
        pe.getTaskService().complete(taskId);
    }
```
#### 3. 查询最新版本的流程定义列表
```java
    /**
     * 查询最新版本的流程定义列表
     */
    @Test
    public void test13(){
        ProcessDefinitionQuery query = pe.getRepositoryService().createProcessDefinitionQuery();
        //按照版本升序
        query.orderByProcessDefinitionVersion().asc();
        List<ProcessDefinition> list = query.list();
        
        Map<String, ProcessDefinition> map = new HashMap<String ,ProcessDefinition>();
        for (ProcessDefinition pd : list) {
            String key = pd.getKey();//流程定义的key
            map.put(key, pd);
        }
        
        List<ProcessDefinition> newList = new ArrayList<>(map.values());
        System.out.println(map);
    }


```


### hello

```java
package cn.dishui.test;

public class TestJk {

  public void dao(){
    System.out.println("dao---------");
  }
}

```
















