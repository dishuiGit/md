```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/context 
						http://www.springframework.org/schema/context/spring-context-2.5.xsd
						http://www.springframework.org/schema/tx 
						http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
	<!-- 加载jdbc属性文件 -->
	<context:property-placeholder location="classpath:jdbc.properties"/>

	<bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="${driverClassName}"></property>
		<property name="url" value="${url}"></property>
		<property name="username" value="${username}"></property>
		<property name="password" value="${password}"></property>
	</bean>
	
	<!-- 配置会话工厂对象 ,整合hibernate-->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
		<property name="dataSource" ref="ds"/>
		<!-- 注入hibernate相关属性 -->
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
				<prop key="hibernate.hbm2ddl.auto">update</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
			</props>
		</property>
		<!-- 注入hbm映射文件 -->
		<property name="mappingDirectoryLocations">
			<list>
				<value>classpath:cn/itcast/domain</value>
			</list>
		</property>
	</bean>
	
	<!-- 配置一个spring提供的事务管理器 -->
	<bean id="dataSourcetransactionManager"
		 class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<!-- 配置流程引擎配置对象 -->
	<bean id="processEngineConfiguration"
		class="org.activiti.spring.SpringProcessEngineConfiguration">
		<property name="dataSource" ref="ds"></property>
		<property name="transactionManager" ref="dataSourcetransactionManager"></property>
		<property name="databaseSchemaUpdate" value="true"></property>
	</bean>
	<!-- 配置一个流程引擎工厂bean，用于创建一个流程引擎对象 -->
	<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
		<!-- 注入配置对象 -->
		<property name="processEngineConfiguration" ref="processEngineConfiguration"></property>
	</bean>
	
	<!-- 使用上面配置的流程引擎对象processEngine创建一个TaskService对象 -->
	<bean id="taskService" factory-bean="processEngine" factory-method="getTaskService"></bean>
	
	<!-- 组件扫描 -->
	<context:component-scan base-package="cn.itcast"/>
	<!-- 支持注解（加入注解解析器） -->
	<context:annotation-config/>
	<!-- 支持事务注解 -->
	<tx:annotation-driven transaction-manager="dataSourcetransactionManager"/>
</beans>
```

### 纯注解开发,如果找不到Action
>![找不到struts类](../pic/struts_01.png)

>检查`<context:component-scan base-package="cn.itcast"/>`